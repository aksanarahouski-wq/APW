# Base Device Configurations - Storage and Mapping

## Overview

Base Device Configurations define the standard configuration settings for devices based on their device model, carrier type, cellular backup status, and service plan. These configurations serve as templates that can be overridden by Custom Company Configurations.

## Database Structure

### Table: `configurations`

**Core Columns:**
- `id` - Primary key (auto-increment integer)
- `config_group_id` - Foreign key to `config_groups` (device model grouping)
- `o_file_id` - UUID foreign key to `o_files` (S3-stored configuration file)
- `title` - Unique name for the configuration
- `host_name` - Expected hostname pattern for devices
- `carrier` - ENUM: 'vz' (Verizon), 'att' (AT&T), 'tmo' (T-Mobile), 'dual_vz_att' (Dual VZ+AT&T), 'dual_vz_tmo' (Dual VZ+TMO)
- `cellular_backup` - Boolean flag for backup cellular configurations
- `status` - Configuration approval status (pending, approved, denied, pending_delete)
- `created_by_user_id`, `modified_by_user_id` - User tracking
- `created`, `modified` - Timestamps

Source: Migrations `20221109223106_CreateConfigurations.php`, `20250904180411_UpdateConfigurationsCarrierEnum.php`, `20250904180921_RenameConfigurationsDualCarrierEnum.php`

### Join Table: `configuration_service_plans`

Links configurations to service plans (many-to-many relationship):
- `id` - Primary key
- `configuration_id` - Foreign key to `configurations`
- `service_plan_id` - Foreign key to `service_plans` (formerly `global_data_tier_groups`)

Originally named `configuration_global_data_tier_groups`, renamed in migration `20221209141247`.

Source: Migration `20221110194041_CreateConfigurationGlobalDataTierGroups.php`

## Relationships

```
configurations
    ├─ belongsTo config_groups (config_group_id) [INNER JOIN, CASCADE delete]
    ├─ belongsTo o_files (o_file_id) [RESTRICT delete]
    ├─ belongsToMany service_plans (via configuration_service_plans)
    ├─ hasMany devices (configuration_id)
    └─ hasMany custom_company_configurations (configuration_id)
```

Source: `ConfigurationsTable.php:95-120`

## Unique Configuration Constraint

Each configuration must be **unique** based on the combination of:
1. `config_group_id` (device model)
2. `carrier` (vz, att, dual)
3. `cellular_backup` (true/false)
4. Service plan IDs (from join table)

**Business Rule Enforcement** (lines 191-217):
```php
// Custom validation rule in ConfigurationsTable
$rules->add(function (Configuration $entity) {
    $conditions = [
        'config_group_id' => $entity->config_group_id,
        'carrier' => $entity->carrier,
        'cellular_backup' => $entity->cellular_backup,
    ];

    $cnt = $this->find()
        ->matching('ServicePlans', function (Query $q) use ($tierIds) {
            return $q->where(['ServicePlans.id IN' => $tierIds]);
        })
        ->where($conditions)
        ->count();

    return $cnt === 0; // Must be unique
}, 'uniqueCarrierBackupServicePlan', [
    'errorField' => 'carrier',
    'message' => __('The combination of carrier, cellular backup, and service plan ' .
        'must be unique within a Configuration Group.')
]);
```

## Mapping to Devices

### Direct Foreign Key Relationship

Devices are linked to configurations through a **direct foreign key**:

```sql
ALTER TABLE devices
ADD COLUMN configuration_id INTEGER UNSIGNED NULL,
ADD FOREIGN KEY (configuration_id) REFERENCES configurations(id)
    ON UPDATE CASCADE
    ON DELETE SET NULL;
```

Source: Migration `20221110195351_AddConfigurationIdToDevices.php`

**Relationship Definition:**
```php
// In DevicesTable
$this->belongsTo('Configurations', [
    'foreignKey' => 'configuration_id',
    'className' => 'SystemManagement.Configurations',
]);
```

Source: `DevicesTable.php:134-137`

### Automatic Device Assignment

When a configuration is **created or updated**, the system automatically assigns it to matching devices.

**Auto-Assignment Logic** (lines 315-407 in `ConfigurationsTable.php`):

```php
public function afterSave(EventInterface $event, EntityInterface $entity, ArrayObject $options)
{
    // Load configuration with relationships
    $config = $this->find()
        ->contain(['ConfigGroups', 'ServicePlans'])
        ->where(['Configurations.id IS' => $entity->id])
        ->firstOrFail();

    // Build device query with base conditions
    $deviceQuery = $this->Devices->find()
        ->select(['Devices.id', 'Devices.verizon_sim_number', 'Devices.att_sim_number',
                  'Devices.tmo_sim_number', 'Devices.is_dual_sim'])
        ->contain([
            'DeviceSimStatuses' => [
                'fields' => ['DeviceSimStatuses.device_id', 'DeviceSimStatuses.carrier',
                            'DeviceSimStatuses.sim_status']
            ]
        ])
        ->where([
            'Devices.device_model_id IS' => $config->config_group->device_model_id,
            'Devices.service_plan_id IN' => collection($config->service_plans)->extract('id')->toList(),
            'Devices.is_cellular_backup IS' => $config->cellular_backup ? 1 : 0,
        ]);

    // Add carrier-specific conditions using DeviceSimStatuses
    switch ($config->carrier) {
        case 'vz':
            // Single Verizon: has Verizon SIM + active Verizon status + not dual SIM
            $deviceQuery
                ->where(['Devices.verizon_sim_number IS NOT' => null,
                        'TRIM(Devices.verizon_sim_number) !=' => ''])
                ->where(['Devices.is_dual_sim !=' => 1])
                ->matching('DeviceSimStatuses', function($q) {
                    return $q->where(['DeviceSimStatuses.carrier' => 'verizon',
                                     'DeviceSimStatuses.sim_status' => 'active']);
                });
            break;

        case 'att':
            // Single AT&T: has AT&T SIM + active AT&T status + not dual SIM
            $deviceQuery
                ->where(['Devices.att_sim_number IS NOT' => null,
                        'TRIM(Devices.att_sim_number) !=' => ''])
                ->where(['Devices.is_dual_sim !=' => 1])
                ->matching('DeviceSimStatuses', function($q) {
                    return $q->where(['DeviceSimStatuses.carrier' => 'att',
                                     'DeviceSimStatuses.sim_status' => 'active']);
                });
            break;

        case 'tmo':
            // Single T-Mobile: has T-Mobile SIM + active T-Mobile status + not dual SIM
            $deviceQuery
                ->where(['Devices.tmo_sim_number IS NOT' => null,
                        'TRIM(Devices.tmo_sim_number) !=' => ''])
                ->where(['Devices.is_dual_sim !=' => 1])
                ->matching('DeviceSimStatuses', function($q) {
                    return $q->where(['DeviceSimStatuses.carrier' => 'tmobile',
                                     'DeviceSimStatuses.sim_status' => 'active']);
                });
            break;

        case 'dual_vz_att':
            // Dual VZ+ATT: narrow down to devices with active AT&T + dual SIM
            $deviceQuery
                ->where(['Devices.is_dual_sim' => 1])
                ->matching('DeviceSimStatuses', function($q) {
                    return $q->where(['DeviceSimStatuses.carrier' => 'att',
                                     'DeviceSimStatuses.sim_status' => 'active']);
                });
            break;

        case 'dual_vz_tmo':
            // Dual VZ+TMO: narrow down to devices with active T-Mobile + dual SIM
            $deviceQuery
                ->where(['Devices.is_dual_sim' => 1])
                ->matching('DeviceSimStatuses', function($q) {
                    return $q->where(['DeviceSimStatuses.carrier' => 'tmobile',
                                     'DeviceSimStatuses.sim_status' => 'active']);
                });
            break;
    }

    // Final precision check using carrier_type virtual property
    $deviceIds = $deviceQuery->all()
        ->filter(function($device) use ($config) {
            // Virtual property carrier_type checks active SIM statuses
            return $device->carrier_type === $config->carrier;
        })
        ->extract('id')
        ->toArray();

    // Update matching devices
    if (!empty($deviceIds)) {
        $devicesUpdated = $this->Devices->updateAll(
            ['configuration_id' => $entity->id],
            ['id IN' => $deviceIds]
        );

        if ($devicesUpdated > 0) {
            $this->log($devicesUpdated . ' Devices Updated with configuration:' . $entity->id);
        }
    }
}
```

### Device Matching Criteria

A device is automatically assigned a configuration when **ALL** of the following match:

1. **Device Model Match:**
   - Device's `device_model_id` = Configuration's `config_group.device_model_id`

2. **Service Plan Match:**
   - Device's `service_plan_id` IN Configuration's linked `service_plans`

3. **Cellular Backup Match:**
   - Device's `is_cellular_backup` = Configuration's `cellular_backup`

4. **Carrier Match** (uses `device_sim_statuses` table):
   - **Verizon (vz):** Device has Verizon SIM + active Verizon status + not dual_sim
   - **AT&T (att):** Device has AT&T SIM + active AT&T status + not dual_sim
   - **T-Mobile (tmo):** Device has T-Mobile SIM + active T-Mobile status + not dual_sim
   - **Dual VZ+AT&T (dual_vz_att):** Device has both VZ and AT&T SIMs + active AT&T status + `is_dual_sim=1`
   - **Dual VZ+TMO (dual_vz_tmo):** Device has both VZ and T-Mobile SIMs + active TMO status + `is_dual_sim=1`

5. **Final Verification:**
   - Device's `carrier_type` virtual property must match configuration's `carrier` value
   - This virtual property is calculated from active SIM statuses in `device_sim_statuses` table

## DeviceSimStatuses Table

**Added:** Migration `20250905185829_CreateDeviceSimStatuses.php`

The `device_sim_statuses` table tracks the active/inactive status of each SIM card in a device:

**Columns:**
- `id` - Primary key
- `device_id` - Foreign key to `devices`
- `carrier` - ENUM: 'att', 'tmobile', 'verizon'
- `sim_status` - ENUM: 'active', 'inactive' (default: 'inactive')
- `provider_status` - Raw status from carrier API (string, nullable)
- `last_api_sync` - Last successful API status sync (datetime, nullable)
- `created`, `modified` - Timestamps

**Purpose:**
- Tracks which SIM cards are currently active on a device
- Used by configuration matching logic to determine correct carrier type
- Synced with carrier APIs to reflect actual SIM activation status
- Enables precise configuration assignment for dual-SIM devices

**Example:**
```
Device with dual SIM (VZ+AT&T):
- Row 1: device_id=123, carrier='verizon', sim_status='active'
- Row 2: device_id=123, carrier='att', sim_status='active'
→ carrier_type = 'dual_vz_att'

Device with single AT&T SIM:
- Row 1: device_id=456, carrier='att', sim_status='active'
→ carrier_type = 'att'
```

## Configuration Storage

### File Storage

Configuration files (.dat format) are stored in **AWS S3** via the Orases Files package:

- Files uploaded through admin interface
- Stored with UUID in `o_files` table
- Reference stored in `configurations.o_file_id`
- Path accessed via `configuration.o_file.path`

**File Format:** Text-based `.dat` files containing InHand router configuration parameters (key=value format)

### Configuration Versioning

The system supports **multiple versions** of configurations:
- Multiple configurations can exist for the same ConfigGroup
- Distinguished by carrier, cellular_backup, and service_plan combinations
- Each version must have unique `title`
- Status tracking: `pending`, `approved`, `denied`, `pending_delete`

## Configuration Lifecycle

### 1. Creation

**URL:** `/admin/system-management/configurations/add`

**Required Fields:**
- `config_group_id` - Which device model this config applies to
- `title` - Unique name
- `host_name` - Expected hostname pattern
- `carrier` - Verizon, AT&T, or Dual
- `cellular_backup` - Boolean flag
- `o_file_id` - Configuration file upload
- `service_plans` - One or more service plans (many-to-many)

**Initial Status:** `pending` (requires approval)

### 2. Approval Workflow

**Status Values:**
- `pending` - Awaiting admin approval
- `approved` - Active and ready for deployment
- `denied` - Rejected, not used
- `pending_delete` - Marked for deletion, awaiting final approval

**Status Changes Trigger:**
- Email notifications to admins (via `ConfigApproval` mailer)
- Automatic device assignment (on approval)
- Logging of changes to audit trail

Source: `ConfigurationsTable.php:312-314`

### 3. Modification Tracking

**Auto-Status Reset** (lines 222-235):
Any modification to a configuration (except status, modified, modified_by_user_id) automatically resets status to `pending`:

```php
public function beforeSave(EventInterface $event, EntityInterface $entity, ArrayObject $options)
{
    if ($entity->isNew() || $entity->get('status') === 'pending' || $entity->get('status') === 'pending_delete') {
        return;
    }

    $dirtyFields = array_filter($entity->getDirty(), function ($dirtyField) {
        return !in_array($dirtyField, ['status', 'modified', 'modified_by_user_id']);
    });

    if (empty($dirtyFields)) {
        return;
    }

    $entity->set('status', 'pending'); // Force re-approval
}
```

### 4. Change Logging

All configuration changes are logged with detailed before/after values:
- Creation logged as "CONFIGURATION CREATION"
- Modifications logged as "CONFIGURATION MODIFICATION" with field-by-field changes
- Tracks: config_group changes, carrier changes, service plan changes, etc.

Source: `ConfigurationsTable.php:243-310`

### 5. Deletion

Configurations cannot be deleted if:
- Devices are assigned to them
- Custom Company Configurations reference them

**Soft Delete Process:**
1. Mark status as `pending_delete`
2. Admin approves deletion
3. System checks for dependencies
4. If clear, actually deletes the record

Source: `ConfigurationsController.php:434-448`

## ConfigGroups - The Bridge to Device Models

### Purpose

`config_groups` act as a grouping mechanism that links device models to their configurations.

**Key Relationship:**
```
device_model (1) ←→ (1) config_group (1) ←→ (many) configurations
```

Each device model has exactly **one** config_group, which can have **multiple** configurations (different carriers, backup types, service plans).

### Structure

- `id` - Primary key
- `device_model_id` - Foreign key to `device_models` (UNIQUE)
- `title` - Display name
- `description` - Optional description
- `config_count` - Counter cache of linked configurations

Source: `ConfigGroupsTable.php:44-132`

## Resolution Priority

When determining which configuration a device should use:

### Priority 1: Custom Company Configuration
If a `custom_company_configurations` record exists for the device's `company_id` + `configuration_id`, use that custom file.

### Priority 2: Parent Company Custom Configuration
If the device's parent company has a custom configuration with `apply_to_children=1`, use that.

### Priority 3: Base Configuration (THIS)
Use the base configuration from the `configurations` table.

Source: `DevicesTable.php:1671-1768`

## Validation Rules

### Required Fields
- `config_group_id` - Must reference existing ConfigGroup
- `o_file_id` - Must reference uploaded file
- `title` - Required and unique
- `host_name` - Required
- `carrier` - Required, must be 'vz', 'att', or 'dual'
- `status` - Required

### Business Rules
- Title must be globally unique across all configurations
- Combination of (config_group, carrier, cellular_backup, service_plans) must be unique
- ConfigGroup must exist
- File must exist in `o_files`
- Cannot modify if status is `pending_delete` (must be approved first)

Source: `ConfigurationsTable.php:129-220`

## Common Scenarios

### Scenario 1: Single Carrier Device (Verizon)

```
Configuration:
    - config_group_id: 1 (InHand IR615 Router)
    - carrier: 'vz'
    - cellular_backup: false
    - service_plans: [Basic Plan, Premium Plan]

Device Assignment:
    - device_model_id: 5 (matches config_group.device_model_id=5)
    - verizon_sim_number: '1234567890' (populated)
    - att_sim_number: NULL
    - tmo_sim_number: NULL
    - is_dual_sim: 0
    - service_plan_id: 10 (Basic Plan - in configuration's service_plans)
    - is_cellular_backup: 0
    - device_sim_statuses:
        - carrier='verizon', sim_status='active'
    - carrier_type: 'vz'
    → Automatically assigned configuration_id = 1
```

### Scenario 2: Single T-Mobile Device

```
Configuration:
    - config_group_id: 1
    - carrier: 'tmo'
    - cellular_backup: false
    - service_plans: [Basic Plan]

Device Assignment:
    - device_model_id: 5
    - verizon_sim_number: NULL
    - att_sim_number: NULL
    - tmo_sim_number: '5551234567' (populated)
    - is_dual_sim: 0
    - service_plan_id: 10 (Basic Plan)
    - is_cellular_backup: 0
    - device_sim_statuses:
        - carrier='tmobile', sim_status='active'
    - carrier_type: 'tmo'
    → Automatically assigned configuration_id = 2
```

### Scenario 3: Dual Carrier Device (VZ + AT&T)

```
Configuration:
    - config_group_id: 1
    - carrier: 'dual_vz_att'
    - cellular_backup: false
    - service_plans: [Premium Plan]

Device Assignment:
    - device_model_id: 5
    - verizon_sim_number: '1234567890' (populated)
    - att_sim_number: '0987654321' (populated)
    - tmo_sim_number: NULL
    - is_dual_sim: 1 (IMPORTANT!)
    - service_plan_id: 11 (Premium Plan)
    - is_cellular_backup: 0
    - device_sim_statuses:
        - carrier='verizon', sim_status='active'
        - carrier='att', sim_status='active'
    - carrier_type: 'dual_vz_att'
    → Automatically assigned configuration_id = 3
```

### Scenario 4: Dual Carrier Device (VZ + T-Mobile)

```
Configuration:
    - config_group_id: 1
    - carrier: 'dual_vz_tmo'
    - cellular_backup: false
    - service_plans: [Premium Plan]

Device Assignment:
    - device_model_id: 5
    - verizon_sim_number: '1234567890' (populated)
    - att_sim_number: NULL
    - tmo_sim_number: '5551234567' (populated)
    - is_dual_sim: 1 (IMPORTANT!)
    - service_plan_id: 11 (Premium Plan)
    - is_cellular_backup: 0
    - device_sim_statuses:
        - carrier='verizon', sim_status='active'
        - carrier='tmobile', sim_status='active'
    - carrier_type: 'dual_vz_tmo'
    → Automatically assigned configuration_id = 4
```

### Scenario 5: Cellular Backup Configuration

```
Configuration:
    - config_group_id: 1
    - carrier: 'att'
    - cellular_backup: TRUE
    - service_plans: [Backup Plan]

Device Assignment:
    - device_model_id: 5
    - verizon_sim_number: NULL
    - att_sim_number: '1122334455' (populated)
    - tmo_sim_number: NULL
    - is_dual_sim: 0
    - is_cellular_backup: 1 (IMPORTANT!)
    - service_plan_id: 15 (Backup Plan)
    - device_sim_statuses:
        - carrier='att', sim_status='active'
    - carrier_type: 'att'
    → Automatically assigned configuration_id = 5
```

### Scenario 6: Inactive SIM Status - No Match

```
Device:
    - device_model_id: 5
    - verizon_sim_number: '1234567890' (populated)
    - service_plan_id: 10 (matches config)
    - is_cellular_backup: 0
    - device_sim_statuses:
        - carrier='verizon', sim_status='inactive' (INACTIVE!)
    - carrier_type: '' (empty because no active SIMs)

Result:
    → configuration_id remains NULL
    → Device shows validation error: "No matching configuration was found"
    → SIM must be activated before configuration can be assigned
```

### Scenario 7: No Matching Configuration

```
Device:
    - device_model_id: 5
    - verizon_sim_number: '1234567890'
    - service_plan_id: 99 (Not in ANY configuration's service_plans)
    - is_cellular_backup: 0
    - device_sim_statuses:
        - carrier='verizon', sim_status='active'

Result:
    → configuration_id remains NULL
    → Device shows validation error: "No matching configuration was found"
```

Source: `DevicesTable.php:841-855`

## Device Validation

Devices **MUST** have a valid configuration assigned:

```php
$rules->add(
    function (Device $entity) {
        if (!isset($entity->configuration_id) || empty($entity->configuration_id)) {
            return false;
        }
        return true;
    },
    'MustHaveConfig',
    [
        'errorField' => 'configuration_id',
        'message' => __('No matching configuration was found')
    ]
);
```

Exception: New devices or devices with certain manufacturer types may bypass this rule.

## Integration Points

### With Device Check-ins
- Devices report current hostname in check-ins
- System compares with `configuration.host_name`
- Mismatch triggers configuration update job

### With Queue System
- Configuration updates queued via `DeviceConfigurationUpdateJob`
- Queue topic: `update-configs`
- Worker processes: `bin/cake worker -p update-configs`

### With Custom Company Configurations
- Base configurations can be overridden per company
- Custom configs reference base configuration via `configuration_id`
- Device lookup checks custom first, then base

## Related Files

- **Table:** `plugins/SystemManagement/src/Model/Table/ConfigurationsTable.php`
- **Entity:** `plugins/SystemManagement/src/Model/Entity/Configuration.php`
- **Controller:** `plugins/SystemManagement/src/Controller/Admin/ConfigurationsController.php`
- **Migrations:**
  - `20221109223106_CreateConfigurations.php`
  - `20221110194041_CreateConfigurationGlobalDataTierGroups.php`
  - `20221110195351_AddConfigurationIdToDevices.php`
  - `20221214202105_AddStatusToConfigurations.php`
- **Device Matching:** `plugins/Devices/src/Model/Table/DevicesTable.php` (lines 315-378 for auto-assignment)
- **Templates:** `plugins/SystemManagement/templates/Admin/Configurations/`

## Key Takeaways

1. **Base configurations are stored in `configurations` table** with file references in S3
2. **Devices are mapped via `configuration_id` foreign key** (direct one-to-many)
3. **Automatic assignment** occurs when configuration is saved, based on device model, carrier, backup status, and service plan
4. **Unique constraint** ensures no duplicate configurations for the same device model/carrier/backup/service plan combo
5. **Approval workflow** with status tracking ensures reviewed configurations
6. **Many-to-many with service plans** allows one configuration to support multiple billing tiers
7. **ConfigGroups bridge device models to configurations** (one model = one group = many configs)
8. **Carrier matching uses `device_sim_statuses` table** to verify active SIM statuses before assignment
9. **Five carrier types supported**: vz, att, tmo, dual_vz_att, dual_vz_tmo

## Recent Changes (September 2025)

### T-Mobile Support Added
- **New carrier type:** 'tmo' for T-Mobile single-carrier devices
- Migration: `20250904180411_UpdateConfigurationsCarrierEnum.php`

### Dual Carrier Enhancement
- **'dual' renamed to 'dual_vz_att'** to explicitly indicate VZ+AT&T combination
- **'dual_vz_tmo' added** for VZ+T-Mobile dual-carrier devices
- Migration: `20250904180921_RenameConfigurationsDualCarrierEnum.php`

### DeviceSimStatuses Table
- **New table** to track active/inactive status of each SIM per device
- Enables precise carrier matching based on actual SIM activation status
- Syncs with carrier APIs to reflect real-time SIM status
- Migration: `20250905185829_CreateDeviceSimStatuses.php`

### Improved Configuration Matching
- **Hybrid approach:** Database query + virtual property filter
- Uses `device_sim_statuses` to match only devices with active SIMs
- Final verification via `carrier_type` virtual property on Device entity
- More accurate assignment, especially for dual-SIM devices

### Impact on Existing Code
- **Old 'dual' configs auto-migrated to 'dual_vz_att'** during migration
- Matching logic now requires active SIM status to assign configuration
- Devices with inactive SIMs will not match any configuration until SIM is activated
