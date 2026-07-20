# Custom Company Configurations Management

## Overview

Custom Company Configurations allow companies to override base device configurations with company-specific configuration files. This enables multi-tenant customization where different companies can have different device settings even when using the same device model and base configuration.

## Database Structure

### Table: `custom_company_configurations`

Key columns:
- `id` - Primary key (auto-increment integer)
- `configuration_id` - Foreign key to `configurations` table
- `company_id` - Foreign key to `companies` table
- `o_file_id` - UUID foreign key to `o_files` (S3-stored config file)
- `host_name` - Expected hostname for devices using this config
- `apply_to_children` - Boolean flag for hierarchical inheritance
- `pending_delete` - Boolean flag for soft deletion workflow
- `created`, `modified` - Timestamps

Source: Migration `20230417214725_CreateCustomCompanyConfigurations.php`

### Relationships

```
custom_company_configurations
    ├─ belongsTo configurations (configuration_id)
    ├─ belongsTo companies (company_id)
    └─ belongsTo o_files (o_file_id)
```

### Unique Constraint

**Composite Unique Key:** `(company_id, configuration_id)`
- Each company can have only ONE custom configuration per base configuration
- Enforced at database and application level

Source: `CustomCompanyConfigurationsTable.php:88`

## Mapping to Companies

### Direct Mapping

Custom Company Configurations are mapped to companies through a **direct foreign key relationship**:

```php
$table->belongsTo('Companies', [
    'className' => 'Companies.Companies'
]);
```

The `company_id` field establishes a one-to-one relationship between a custom configuration and a specific company.

### Multi-Tenant Inheritance with `apply_to_children`

Added in migration `20240411172221`:

```sql
apply_to_children BOOLEAN DEFAULT 0 NOT NULL
```

**Behavior:**
- When `apply_to_children = 1` (true): Parent company's custom configuration applies to all child companies
- When `apply_to_children = 0` (false): Configuration applies only to the specific company

**Hierarchy Resolution Logic** (from `DevicesTable.php:1732-1760`):

```php
// Priority 1: Direct company custom configuration
if (isset($deviceEntity->custom_company_configuration)) {
    return $deviceEntity->custom_company_configuration;
}

// Priority 2: Parent company configuration with apply_to_children=1
if (!empty($deviceEntity->company->parent_company_id)) {
    $parentCompanyConfiguration = CustomCompanyConfigurations::find()
        ->where([
            'configuration_id' => $deviceEntity->configuration_id,
            'apply_to_children' => 1,
            'company_id' => $deviceEntity->company->parent_company_id
        ])
        ->first();

    if (isset($parentCompanyConfiguration)) {
        return $parentCompanyConfiguration;
    }
}

// Priority 3: Base configuration (fallback)
return $deviceEntity->configuration;
```

### Device Lookup via Composite Foreign Key

Devices are associated with Custom Company Configurations through a **composite foreign key**:

```php
$this->belongsTo('CustomCompanyConfigurations', [
    'className' => 'SystemManagement.CustomCompanyConfigurations'
])->setForeignKey([
    'company_id',
    'configuration_id'
]);
```

This means a device automatically gets the custom configuration when:
- Device's `company_id` matches custom config's `company_id`
- Device's `configuration_id` matches custom config's `configuration_id`

Source: `DevicesTable.php:242-250`

## Management Workflow

### Creating/Editing Custom Company Configurations

**URL:** `/admin/system-management/custom-company-configurations/edit/{configurationId}/{id?}`

**Controller:** `SystemManagement\Controller\Admin\CustomCompanyConfigurationsController`

#### Process Flow:

1. **Access Control Check** (lines 19-33)
   - Verify base configuration is not marked `pending_delete`
   - If marked for deletion, redirect with error message
   - Prevents adding custom configs to configurations being phased out

2. **Load or Create Entity** (lines 36-57)
   - If `id` provided: Load existing custom configuration
   - If no `id`: Create new entity with `configuration_id` pre-set
   - Contains `OFiles` and `Companies` associations

3. **Form Submission** (lines 64-100)
   - Patch entity with form data
   - Validate required fields:
     - `company_id` (required)
     - `host_name` (required)
     - `o_file_id` (required, file upload)
   - Save entity
   - On success: Redirect to configuration view page

4. **Post-Save Hook** (lines 70-82 in Table)
   - Automatically sets parent configuration status to `pending`
   - Forces re-approval workflow when custom configs are added/modified
   - Ensures admins review changes before deployment

Source: `CustomCompanyConfigurationsController.php:17-104`

### Form Fields

From template `edit.php`:

```php
// Company selection (Select2 enhanced dropdown)
$this->Form->control('company_id', [
    'type' => 'select',
    'id' => 'company-select-2',
    'options' => $selectedCompany
]);

// Expected hostname
$this->Form->control('host_name', [
    'label' => __('Anticipated Host Name')
]);

// Hierarchical inheritance flag
$this->Form->control('apply_to_children', [
    'label' => __('Apply to Children'),
    'type' => 'checkbox'
]);

// Configuration file upload (.dat file)
$this->Form->fileUpload('o_file', [
    'accept' => '.dat'
]);
```

### Deleting Custom Company Configurations

**URL:** `/admin/system-management/custom-company-configurations/delete/{id}`

**Workflow:** Soft deletion with approval process

```php
public function delete(string $id)
{
    $customCompanyConfiguration = CustomCompanyConfigurations::get($id);

    // Mark for deletion (soft delete)
    $customCompanyConfiguration->pending_delete = true;

    if (save($customCompanyConfiguration)) {
        Flash::success('Configuration marked for deletion for company: ' . $company->title);
    }

    // Redirect back to parent configuration view
    return redirect(['controller' => 'Configurations', 'action' => 'view', $configuration_id]);
}
```

**Actual Deletion:** Occurs when parent configuration is approved

```php
private function processRelatedCustomCompanyConfigurations($configuration, $status)
{
    $customConfigsPendingDelete = CustomCompanyConfigurations::find()
        ->where([
            'configuration_id' => $configuration->id,
            'pending_delete' => true
        ])
        ->all();

    // Delete when parent configuration is approved
    if ($status === 'approved') {
        CustomCompanyConfigurations::deleteMany($customConfigsPendingDelete);
    }
}
```

Source: `CustomCompanyConfigurationsController.php:106-136` and `ConfigurationsController.php:326-338`

## Viewing Custom Company Configurations

Custom Company Configurations are displayed within the parent Configuration view page.

**URL:** `/admin/system-management/configurations/view/{id}`

The view page contains (loads with):
```php
'CustomCompanyConfigurations' => [
    'Companies',
    'OFiles'
]
```

This shows:
- All custom configurations linked to the base configuration
- Which companies have custom overrides
- The configuration files associated with each company
- `apply_to_children` status
- `pending_delete` status

Source: `ConfigurationsController.php:106-116`

## Pushing Configurations to Devices

### Manual Push from Configuration View

Admins can manually push configurations to specific devices from the Configuration view page:

**Endpoint:** `/admin/system-management/configurations/push-config/{configurationId}/{serialNumber}/{customConfigId?}`

**Process:**
1. Load device by serial number
2. Verify device is assigned to the configuration
3. If `customConfigId` provided, use custom company configuration
4. Otherwise, use base configuration or device's matched custom config
5. Queue `DeviceConfigurationUpdateJob` with override flag

Source: `ConfigurationsController.php:177-254`

### Automatic Push on Device Check-in

When devices check in via UDP:
1. System compares device's `hostname` with expected `host_name` from configuration
2. `getDeviceConfiguration()` resolves which config to use (custom → parent → base)
3. If hostname mismatch, queue configuration update job
4. Job downloads appropriate custom or base configuration file

Source: `DeviceCheckinsTable.php:172` and `DevicesTable.php:1671-1768`

## Validation Rules

### Required Fields
- `company_id` - Company must be selected
- `host_name` - Hostname cannot be blank
- `o_file_id` - Configuration file must be uploaded

### Business Rules
- Company must exist in `companies` table
- Configuration must exist in `configurations` table
- Combination of `(company_id, configuration_id)` must be unique
- Cannot add custom config if parent configuration status is `pending_delete`

Source: `CustomCompanyConfigurationsTable.php:50-91`

## Multi-Tenant Scenarios

### Scenario 1: Company with Direct Custom Configuration

```
Configuration A (Base)
    └─ CustomCompanyConfiguration (Company X)
        ├─ company_id: X
        ├─ o_file_id: custom-file-123
        └─ apply_to_children: false

Device belonging to Company X
    → Uses CustomCompanyConfiguration for Company X
```

### Scenario 2: Child Company Inheriting Parent Configuration

```
Configuration A (Base)
    └─ CustomCompanyConfiguration (Company Parent)
        ├─ company_id: Parent
        ├─ o_file_id: parent-custom-file
        └─ apply_to_children: TRUE

Company Parent
    └─ Company Child (parent_company_id: Parent)
        └─ Device
            → Uses Parent's CustomCompanyConfiguration (inherited)
```

### Scenario 3: Child Overriding Parent Configuration

```
Configuration A (Base)
    ├─ CustomCompanyConfiguration (Company Parent)
    │   ├─ company_id: Parent
    │   └─ apply_to_children: TRUE
    │
    └─ CustomCompanyConfiguration (Company Child)
        ├─ company_id: Child
        └─ apply_to_children: false

Company Parent
    └─ Company Child
        └─ Device
            → Uses Child's CustomCompanyConfiguration (direct mapping takes precedence)
```

### Scenario 4: No Custom Configuration (Base Only)

```
Configuration A (Base)
    (no custom company configurations)

Company X
    └─ Device
        → Uses Configuration A (base configuration)
```

## File Storage

Configuration files are stored in S3 via the **Orases Files package**:

- Files uploaded through `FileUpload` widget
- Stored with UUID in `o_files` table
- Path format: `s3://bucket/path/to/file.dat`
- Associated with custom company configuration via `o_file_id`

**File Format:** `.dat` files (text-based configuration format for InHand routers)

## Key Workflows Summary

### 1. Add Custom Configuration for Company
```
Admin → Configuration View
    → "Add Custom Company Configuration"
    → Select Company
    → Upload .dat file
    → Set hostname
    → Toggle "apply_to_children" (optional)
    → Save
    → Parent configuration status → "pending"
    → Admin approves parent configuration
    → Custom config becomes active
```

### 2. Delete Custom Configuration
```
Admin → Configuration View
    → Find Custom Company Configuration
    → Click "Delete"
    → Status → pending_delete = true
    → Admin approves parent configuration
    → Custom config actually deleted
    → Devices revert to base or parent config
```

### 3. Device Configuration Resolution
```
Device checks in
    → System looks up device's company_id + configuration_id
    → Check 1: Direct CustomCompanyConfiguration exists?
        → YES: Use it
        → NO: Continue to Check 2
    → Check 2: Parent has CustomCompanyConfiguration with apply_to_children=1?
        → YES: Use parent's
        → NO: Continue to Check 3
    → Check 3: Use base Configuration
```

## Security Considerations

- Custom configurations can only be managed by admins with appropriate permissions
- Soft deletion prevents accidental immediate removal
- Approval workflow ensures changes are reviewed
- Foreign key constraints prevent orphaned records
- Composite unique constraint prevents duplicate configurations per company

## Related Files

- **Table:** `plugins/SystemManagement/src/Model/Table/CustomCompanyConfigurationsTable.php`
- **Controller:** `plugins/SystemManagement/src/Controller/Admin/CustomCompanyConfigurationsController.php`
- **Template:** `plugins/SystemManagement/templates/Admin/CustomCompanyConfigurations/edit.php`
- **Migrations:**
  - `20230417214725_CreateCustomCompanyConfigurations.php`
  - `20240411172221_AddApplyToChildrenToCustomCompanyConfigurations.php`
  - `20240709161646_AddPendingDeleteToCustomCompanyConfigurations.php`
- **Configuration Resolution:** `plugins/Devices/src/Model/Table/DevicesTable.php:1671-1768`
