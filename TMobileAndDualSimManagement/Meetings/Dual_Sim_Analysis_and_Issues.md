# Dual SIM Functionality Analysis & Issues

**Date:** October 29, 2025
**Status:** Investigation Complete - Awaiting Client Feedback

## Executive Summary

This document provides a comprehensive analysis of the WATM Dual SIM functionality, including how device active SIM tracking, company assignment, and configuration mapping work. Several critical issues have been identified that may be causing problems for the client.

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Current Architecture](#current-architecture)
3. [Identified Issues](#identified-issues)
4. [Recommended Fixes](#recommended-fixes)
5. [Questions for Client](#questions-for-client)
6. [Technical Reference](#technical-reference)

---

## System Overview

The Dual SIM system manages IoT devices that can have multiple cellular carriers (Verizon, AT&T, T-Mobile). The system needs to:

1. Track which SIM cards are active/inactive on each device
2. Automatically assign appropriate configuration files based on carrier combination
3. Integrate with carrier APIs for SIM activation/deactivation
4. Support both single-carrier and dual-carrier devices
5. Map devices to configurations based on device model, service plan, and carrier type

### Supported Carrier Combinations

- **Single Carrier:** `vz`, `att`, `tmo`
- **Dual Carrier:** `dual_vz_att` (Verizon + AT&T), `dual_vz_tmo` (Verizon + T-Mobile)

---

## Current Architecture

### Component 1: DeviceSimStatuses Table

**Purpose:** Track the active/inactive status of each SIM card per device

**Database Schema:**
```sql
device_sim_statuses (
    id INT PRIMARY KEY,
    device_id INT (FK to devices),
    carrier ENUM('att', 'tmobile', 'verizon'),
    sim_status ENUM('active', 'inactive') DEFAULT 'inactive',  -- User-controlled
    provider_status VARCHAR(50) NULL,                          -- API-controlled
    last_api_sync DATETIME NULL,
    created DATETIME,
    modified DATETIME,
    UNIQUE (device_id, carrier)
)
```

**Key Fields:**
- `sim_status`: What the user/system wants the SIM to be ('active' or 'inactive')
- `provider_status`: Actual status from carrier API ('pending_active', 'activated', 'deactivated', 'suspended', etc.)
- `last_api_sync`: When the status was last confirmed with carrier API

**File Location:** `plugins/Devices/src/Model/Table/DeviceSimStatusesTable.php`

---

### Component 2: Configuration Assignment Logic

**Purpose:** Automatically assign configuration files to devices based on their attributes

**Matching Criteria (in order):**
1. ✅ **Device Model Match** - `device.device_model_id` = `configuration.config_group.device_model_id`
2. ✅ **Service Plan Match** - `device.service_plan_id` IN `configuration.service_plans`
3. ✅ **Cellular Backup Match** - `device.is_cellular_backup` = `configuration.cellular_backup`
4. ✅ **Carrier Type Match** - Uses `device_sim_statuses` to determine active carriers
5. ✅ **Dual SIM Flag Match** - Dual configs only match devices with `is_dual_sim = 1`

**Trigger:** Runs automatically when a **configuration** is saved (`ConfigurationsTable::afterSave()`)

**File Location:** `plugins/SystemManagement/src/Model/Table/ConfigurationsTable.php:315-407`

---

### Component 3: Device Carrier Detection

**Purpose:** Dynamically determine device carrier type based on active SIMs

**Virtual Properties on Device Entity:**

```php
// File: plugins/Devices/src/Model/Entity/Device.php

protected function _getIsVz(): bool
{
    // Verizon SIM present AND device_sim_statuses.sim_status = 'active'
    return !empty($this->verizon_sim_number) &&
           isset($this->sim_status_by_carrier['verizon']) &&
           $this->sim_status_by_carrier['verizon']->sim_status === 'active';
}

protected function _getIsAtt(): bool
{
    // AT&T SIM present AND device_sim_statuses.sim_status = 'active'
    return !empty($this->att_sim_number) &&
           isset($this->sim_status_by_carrier['att']) &&
           $this->sim_status_by_carrier['att']->sim_status === 'active';
}

protected function _getIsTmo(): bool
{
    // T-Mobile SIM present AND device_sim_statuses.sim_status = 'active'
    return !empty($this->tmo_sim_number) &&
           isset($this->sim_status_by_carrier['tmobile']) &&
           $this->sim_status_by_carrier['tmobile']->sim_status === 'active';
}

protected function _getCarrierType(): string
{
    // Returns: 'dual_vz_att', 'dual_vz_tmo', 'vz', 'att', 'tmo', or ''
    if ($this->is_vz && $this->is_att && $this->is_dual_sim) {
        return 'dual_vz_att';
    } elseif ($this->is_vz && $this->is_tmo && $this->is_dual_sim) {
        return 'dual_vz_tmo';
    } elseif ($this->is_vz) {
        return 'vz';
    } elseif ($this->is_att) {
        return 'att';
    } elseif ($this->is_tmo) {
        return 'tmo';
    }
    return '';
}
```

**Critical Dependency:** These properties require `device_sim_statuses` records to exist and be loaded with the device.

---

### Component 4: API Integration

**Purpose:** Synchronize SIM activation/deactivation with carrier APIs

**Workflow:**
1. User checks/unchecks SIM checkbox in device edit form
2. `DeviceSimStatusesTable::beforeSave()` detects `sim_status` change
3. Calls appropriate carrier API method:
   - **Activate:** `VerizonApi::activateDevice()`, `AttApi::resumeDevice()`, `TMobileApi::editConnectionStatus('ACTIVE')`
   - **Deactivate:** `VerizonApi::deactivateDevice()`, `AttApi::suspendDevice()`, `TMobileApi::editConnectionStatus('Deactivated')`
4. Updates `provider_status` to reflect API response
5. Sets `last_api_sync` timestamp

**File Location:** `plugins/Devices/src/Model/Table/DeviceSimStatusesTable.php:84-254`

---

## Identified Issues

### 🔴 CRITICAL Issue #1: No Company-Based Configuration Assignment

**Problem:** Configuration auto-assignment logic does **NOT** filter by company.

**Current Code (`ConfigurationsTable.php:322-333`):**
```php
$deviceQuery = $this->Devices->find()
    ->where([
        'Devices.device_model_id IS' => $config->config_group->device_model_id,
        'Devices.service_plan_id IN' => collection($config->service_plans)->extract('id')->toList(),
        'Devices.is_cellular_backup IS' => $config->cellular_backup ? 1 : 0,
    ]);
// NO COMPANY FILTER!
```

**Impact:**
- Configuration created for Company A can be auto-assigned to Company B's devices
- No multi-tenant isolation for configurations
- Violates multi-tenant architecture principles

**Expected Behavior:**
Configurations should only match devices within the same company or child companies (respecting parent-child relationships).

**Severity:** HIGH - Breaks multi-tenant isolation

---

### 🔴 CRITICAL Issue #2: DeviceSimStatuses Not Auto-Created

**Problem:** When devices are created or updated with SIM numbers, `device_sim_statuses` records are NOT automatically created.

**Evidence:**
- Manual command exists: `bin/cake populate_sim_statuses`
- No `afterSave` hook in `DevicesTable` creates sim status records
- Virtual properties (`is_vz`, `is_att`, `is_tmo`) return `false` if records don't exist

**Impact:**
- New devices cannot match configurations (carrier_type = '' because no sim statuses)
- Configuration assignment fails silently
- Admin must manually run populate command after importing/creating devices
- Users see "No matching configuration was found" validation error

**Reproduction Steps:**
1. Create new device with `verizon_sim_number = '1234567890'`
2. Set `service_plan_id`, `device_model_id`, `is_dual_sim = 0`
3. Save device
4. Check `device_sim_statuses` table - **no records exist**
5. Device's `carrier_type` property returns `''` (empty)
6. No configuration assigned despite matching criteria

**Severity:** CRITICAL - Prevents configuration assignment

---

### 🔴 HIGH Issue #3: No Configuration Reassignment on Device Changes

**Problem:** When device attributes change, configuration is NOT automatically updated.

**Scenarios That Fail:**

1. **Service Plan Change:**
   - Device starts with "Basic Plan" → assigned Config A
   - Admin changes to "Premium Plan"
   - Device still has Config A (incorrect)
   - **Expected:** Should reassign to Config B (Premium)

2. **SIM Status Change:**
   - Device has VZ SIM (active) + AT&T SIM (inactive) → assigned single VZ config
   - Admin activates AT&T SIM + checks `is_dual_sim`
   - Device still has single VZ config (incorrect)
   - **Expected:** Should reassign to dual_vz_att config

3. **Carrier Switch:**
   - Device changes from AT&T to T-Mobile SIM
   - Configuration unchanged (incorrect)
   - **Expected:** Should reassign from 'att' config to 'tmo' config

**Root Cause:** Only `ConfigurationsTable::afterSave()` triggers assignment, not `DevicesTable::afterSave()`

**Severity:** HIGH - Devices get wrong configurations

---

### 🟡 MEDIUM Issue #4: Service Plan Mapping Ambiguity

**Problem:** Relationship between configurations, service plans, and companies is unclear.

**Questions Needing Clarification:**

1. **Are service_plans global or company-specific?**
   - Table `service_plans` has no `company_id` column
   - But `company_service_plans` table exists (join table?)
   - Unclear how company-specific pricing relates to configurations

2. **Can different companies share the same service plan but need different configs?**
   - Example: Company A and Company B both use "Premium Plan"
   - But Company A uses custom APN settings vs Company B uses default
   - How is this handled?

3. **Configuration-ServicePlan mapping:**
   - `configuration_service_plans` is many-to-many
   - One configuration can support multiple service plans
   - But service plan might be company-specific - conflict?

**Impact:**
- Configuration assignment may match wrong service plan for company
- Potential for devices to get configurations meant for different companies

**Severity:** MEDIUM - Needs clarification but may not be broken

---

### 🟡 MEDIUM Issue #5: Missing Dual SIM Configuration Availability Validation

**Problem:** No validation that appropriate dual configuration exists before marking device as dual SIM.

**Current Validation (`DevicesTable.php:578-646`):**
- ✅ Validates 2 active SIMs requires `is_dual_sim = 1`
- ✅ Validates only VZ+ATT or VZ+TMO combinations allowed
- ✅ Validates 3+ SIMs not supported

**Missing Validation:**
- ❌ Does NOT check if dual configuration exists for device's service plan
- ❌ Does NOT warn if dual config unavailable
- ❌ Does NOT verify device model supports dual SIM

**Scenario:**
1. Admin sets device to `is_dual_sim = 1`
2. Activates both VZ and AT&T SIMs
3. Device passes validation
4. **BUT:** No `dual_vz_att` configuration exists for this device model + service plan
5. Device saves with `configuration_id = NULL`
6. Device is broken - no config file

**Severity:** MEDIUM - Allows invalid device states

---

### 🟡 MEDIUM Issue #6: API Sync Status Not Surfaced in UI

**Problem:** `provider_status` field tracks actual carrier API status but may not be visible to users.

**Missing UI Elements:**
- No indicator showing `sim_status` vs `provider_status` mismatch
- No visual alert when API call fails
- No display of `last_api_sync` age (stale data indicator)
- No retry button for failed activations

**Example Hidden Failure:**
1. User checks "Activate Verizon SIM" checkbox
2. `sim_status` changes to 'active'
3. API call to Verizon fails (network timeout)
4. `provider_status` remains 'deactivated'
5. **User sees:** Checkbox checked, assumes SIM is active
6. **Reality:** SIM is still deactivated at carrier

**Current Handling:**
- API failures logged to event system only
- No user-facing notification
- `provider_status` stores failure info but not displayed

**Severity:** MEDIUM - Users unaware of API failures

---

### 🟢 LOW Issue #7: No Periodic API Sync

**Problem:** SIM statuses can drift from actual carrier state over time.

**Current State:**
- `last_api_sync` field exists
- `DeviceSimStatus::needsSync(hoursOld = 6)` method exists
- Custom finder `findNeedingSync()` exists
- **BUT:** No scheduled job to actually perform periodic sync

**Impact:**
- Carrier could deactivate SIM externally (billing failure, etc.)
- `provider_status` never updates
- Device appears active in WATM but is actually inactive

**Severity:** LOW - Edge case but could cause confusion

---

## Recommended Fixes

### Priority 1: Auto-Create DeviceSimStatuses Records

**Goal:** Automatically create `device_sim_statuses` records when device is saved with SIM numbers.

**Implementation:**

**File:** `plugins/Devices/src/Model/Table/DevicesTable.php`

**Add method:**
```php
/**
 * Ensure device_sim_statuses records exist for populated SIM numbers
 *
 * @param Device $device Device entity
 * @return void
 */
protected function ensureSimStatusRecords(Device $device): void
{
    $simMapping = [
        'verizon_sim_number' => 'verizon',
        'att_sim_number' => 'att',
        'tmo_sim_number' => 'tmobile',
    ];

    foreach ($simMapping as $simField => $carrier) {
        if (!empty($device->{$simField})) {
            // Check if record already exists
            $exists = $this->DeviceSimStatuses->exists([
                'device_id' => $device->id,
                'carrier' => $carrier,
            ]);

            if (!$exists) {
                // Determine initial status
                $initialStatus = $this->determineInitialSimStatus($device, $carrier);

                $simStatus = $this->DeviceSimStatuses->newEntity([
                    'device_id' => $device->id,
                    'carrier' => $carrier,
                    'sim_status' => $initialStatus,
                ]);

                $this->DeviceSimStatuses->save($simStatus);
            }
        }
    }
}

/**
 * Determine initial SIM status for newly created record
 *
 * @param Device $device Device entity
 * @param string $carrier Carrier name
 * @return string 'active' or 'inactive'
 */
protected function determineInitialSimStatus(Device $device, string $carrier): string
{
    // For dual SIM devices, both carriers should be active
    if ($device->is_dual_sim) {
        return 'active';
    }

    // For single SIM devices, activate the primary carrier only
    // Priority: Verizon > AT&T > T-Mobile
    if ($carrier === 'verizon' && !empty($device->verizon_sim_number)) {
        return 'active';
    }

    if ($carrier === 'att' && !empty($device->att_sim_number) && empty($device->verizon_sim_number)) {
        return 'active';
    }

    if ($carrier === 'tmobile' && !empty($device->tmo_sim_number) &&
        empty($device->verizon_sim_number) && empty($device->att_sim_number)) {
        return 'active';
    }

    return 'inactive';
}
```

**Add to `afterSave()` callback:**
```php
public function afterSave(EventInterface $event, EntityInterface $entity, ArrayObject $options)
{
    // Existing code...

    // Ensure sim status records exist
    if ($entity->id) {
        $this->ensureSimStatusRecords($entity);
    }
}
```

**Testing:**
1. Create device with `verizon_sim_number`
2. Verify `device_sim_statuses` record created with `carrier='verizon'`, `sim_status='active'`
3. Add `att_sim_number` to existing device
4. Verify second record created

---

### Priority 2: Trigger Configuration Reassignment on Device Changes

**Goal:** Automatically reassign configuration when device attributes change.

**Implementation:**

**File:** `plugins/Devices/src/Model/Table/DevicesTable.php`

**Add method:**
```php
/**
 * Reassign configuration based on current device attributes
 *
 * @param Device $device Device entity
 * @return void
 */
protected function reassignConfiguration(Device $device): void
{
    // Reload device with all necessary associations
    $device = $this->get($device->id, [
        'contain' => ['DeviceSimStatuses', 'DeviceModel', 'ServicePlan']
    ]);

    // Find matching configuration
    $configuration = $this->Configurations->find()
        ->contain(['ConfigGroups', 'ServicePlans'])
        ->matching('ServicePlans', function ($q) use ($device) {
            return $q->where(['ServicePlans.id' => $device->service_plan_id]);
        })
        ->where([
            'ConfigGroups.device_model_id' => $device->device_model_id,
            'Configurations.cellular_backup' => $device->is_cellular_backup ? 1 : 0,
            'Configurations.carrier' => $device->carrier_type,
            'Configurations.status' => 'approved',
        ])
        ->first();

    if ($configuration && $configuration->id !== $device->configuration_id) {
        // Update configuration without triggering infinite loop
        $this->updateAll(
            ['configuration_id' => $configuration->id],
            ['id' => $device->id]
        );

        $this->log("Device {$device->id} reassigned to configuration {$configuration->id}");
    }
}
```

**Add to `afterSave()` callback:**
```php
public function afterSave(EventInterface $event, EntityInterface $entity, ArrayObject $options)
{
    // Existing code...

    // Check if relevant fields changed
    $relevantFields = [
        'service_plan_id',
        'device_model_id',
        'is_cellular_backup',
        'is_dual_sim',
        'verizon_sim_number',
        'att_sim_number',
        'tmo_sim_number',
    ];

    $hasRelevantChanges = false;
    foreach ($relevantFields as $field) {
        if ($entity->isDirty($field)) {
            $hasRelevantChanges = true;
            break;
        }
    }

    if ($hasRelevantChanges) {
        $this->reassignConfiguration($entity);
    }
}
```

**Testing:**
1. Device with "Basic Plan" + VZ SIM → Config A assigned
2. Change to "Premium Plan"
3. Verify Config B auto-assigned

---

### Priority 3: Add Company Filtering to Configuration Assignment

**Goal:** Only assign configurations to devices within the same company hierarchy.

**Implementation:**

**File:** `plugins/SystemManagement/src/Model/Table/ConfigurationsTable.php`

**Modify `afterSave()` method (line 322):**

**Current:**
```php
$deviceQuery = $this->Devices->find()
    ->where([
        'Devices.device_model_id IS' => $config->config_group->device_model_id,
        'Devices.service_plan_id IN' => collection($config->service_plans)->extract('id')->toList(),
        'Devices.is_cellular_backup IS' => $config->cellular_backup ? 1 : 0,
    ]);
```

**Updated:**
```php
$deviceQuery = $this->Devices->find()
    ->contain(['Companies'])
    ->where([
        'Devices.device_model_id IS' => $config->config_group->device_model_id,
        'Devices.service_plan_id IN' => collection($config->service_plans)->extract('id')->toList(),
        'Devices.is_cellular_backup IS' => $config->cellular_backup ? 1 : 0,
    ]);

// Add company filtering if configuration is company-specific
if (!empty($config->company_id)) {
    $deviceQuery->where(function($exp) use ($config) {
        return $exp->or([
            'Devices.company_id' => $config->company_id,  // Direct match
            'Companies.parent_id' => $config->company_id  // Child companies
        ]);
    });
}
```

**Prerequisites:**
- Verify if `configurations` table has `company_id` column
- If not, may need to rely on `custom_company_configurations` instead

**Testing:**
1. Create configuration for Company A
2. Create devices for Company A and Company B with matching attributes
3. Verify only Company A devices get configuration assigned

---

### Priority 4: Add Dual SIM Configuration Availability Validation

**Goal:** Prevent users from enabling dual SIM if no matching configuration exists.

**Implementation:**

**File:** `plugins/Devices/src/Model/Table/DevicesTable.php`

**Add validation rule:**
```php
/**
 * Validation rules
 */
public function validationDefault(Validator $validator): Validator
{
    // Existing validation...

    // Add dual SIM configuration check
    $validator->add('is_dual_sim', 'hasMatchingConfiguration', [
        'rule' => function ($value, $context) {
            if ($value != 1) {
                return true; // Not dual SIM, skip check
            }

            // Determine expected carrier_type
            $carrierType = $this->calculateCarrierType($context['data']);

            // Check if matching dual configuration exists
            $configExists = $this->Configurations->find()
                ->matching('ServicePlans', function ($q) use ($context) {
                    return $q->where(['ServicePlans.id' => $context['data']['service_plan_id']]);
                })
                ->matching('ConfigGroups', function ($q) use ($context) {
                    return $q->where(['DeviceModels.id' => $context['data']['device_model_id']]);
                })
                ->where([
                    'Configurations.carrier' => $carrierType,
                    'Configurations.status' => 'approved',
                ])
                ->count() > 0;

            return $configExists;
        },
        'message' => __('No approved dual SIM configuration exists for this device model and service plan combination.'),
    ]);

    return $validator;
}
```

**Testing:**
1. Try to enable dual SIM on device
2. If no dual config exists, validation fails with helpful message
3. User knows to create config first

---

### Priority 5: Surface API Sync Status in UI

**Goal:** Show users the actual SIM status from carrier APIs.

**Implementation:**

**File:** `plugins/Devices/templates/Admin/Devices/edit.php`

**Add status indicator next to each SIM checkbox:**

```php
<div class="form-group">
    <label>
        <?= $this->Form->checkbox('device_sim_statuses.0.sim_status') ?>
        Verizon SIM Active
    </label>

    <?php if (!empty($device->sim_status_by_carrier['verizon'])): ?>
        <?php $vzStatus = $device->sim_status_by_carrier['verizon']; ?>
        <div class="api-status">
            <strong>Provider Status:</strong>
            <span class="badge badge-<?= $vzStatus->status_badge_class ?>">
                <?= h($vzStatus->provider_status ?? 'Unknown') ?>
            </span>

            <?php if ($vzStatus->last_api_sync): ?>
                <small class="text-muted">
                    Last synced: <?= $vzStatus->last_api_sync->timeAgoInWords() ?>
                </small>
            <?php else: ?>
                <small class="text-warning">
                    Never synced with carrier API
                </small>
            <?php endif; ?>
        </div>
    <?php endif; ?>
</div>
```

**Add retry button:**
```php
<?php if (!empty($vzStatus->provider_status) && $vzStatus->provider_status !== 'active'): ?>
    <?= $this->Form->postLink(
        'Retry Activation',
        ['action' => 'retrySimActivation', $device->id, 'verizon'],
        ['class' => 'btn btn-sm btn-warning']
    ) ?>
<?php endif; ?>
```

**Testing:**
1. View device edit form
2. See current `sim_status` (checkbox) and `provider_status` (badge)
3. If mismatch, see visual indicator
4. Click retry button to re-attempt API call

---

## Questions for Client

### Company & Service Plan Questions

**Q1: Company-Specific Configurations**
- Are configurations intended to be company-specific, or shared globally across all companies?
- If company-specific, should child companies inherit parent configurations?
- Current implementation: No company filtering in configuration assignment

**Q2: Service Plan Scope**
- Are `service_plans` global (system-wide) or company-specific?
- Can Company A and Company B both have devices on "Premium Plan" but need different configurations?
- How does `company_service_plans` table relate to base `service_plans`?

**Q3: Configuration Sharing**
- Can multiple companies share the same base configuration?
- Or does each company need their own copy (via `custom_company_configurations`)?

---

### SIM Status Initialization Questions

**Q4: Default SIM Status**
- When a new device is created with SIM numbers, should SIMs default to 'active' or 'inactive'?
- Current implementation: Manual population via CLI command

**Q5: Import Behavior**
- When devices are bulk imported with SIM numbers, should sim status records be auto-created?
- Should imported devices trigger carrier API calls immediately?

---

### Configuration Assignment Questions

**Q6: Assignment Priority**
- If multiple configurations match a device (rare but possible), which should win?
  - Most recently created?
  - Specific priority field?
  - First match?

**Q7: Reassignment Triggers**
- Should configuration reassignment happen automatically when device attributes change?
- Or should it require manual admin action?
- Current implementation: Only when configuration is saved, not when device changes

**Q8: Missing Configuration Handling**
- What should happen if no matching configuration exists?
- Current: Validation error prevents device save
- Alternative: Allow save but flag device as "needs configuration"?

---

### API Integration Questions

**Q9: API Failure Handling**
- When carrier API calls fail during SIM activation, what should happen?
  - Retry automatically?
  - Show error to user?
  - Queue for later retry?
- Current: Logs error but no user notification

**Q10: API Sync Frequency**
- Should there be a scheduled job to periodically sync SIM statuses with carrier APIs?
- How often? (Recommendation: Daily for active devices)
- Current: No periodic sync, only during manual changes

---

### Dual SIM Questions

**Q11: Dual SIM Validation**
- Should system prevent enabling dual SIM if no matching configuration exists?
- Or allow it and show warning?
- Current: No validation, device can be saved without config

**Q12: Dual SIM Transitions**
- What's the workflow for converting single-carrier device to dual-carrier?
  1. Add second SIM number
  2. Create/approve dual configuration
  3. Activate second SIM
  4. Enable `is_dual_sim` flag
- Is this the correct order? Should system guide users?

---

## Technical Reference

### Key Files by Component

#### Configuration Assignment Logic
- **`plugins/SystemManagement/src/Model/Table/ConfigurationsTable.php`**
  - Lines 315-407: `afterSave()` - Auto-assignment logic
  - Lines 191-217: Unique constraint validation
  - Lines 222-235: `beforeSave()` - Status reset on modification

#### Device SIM Status Management
- **`plugins/Devices/src/Model/Table/DeviceSimStatusesTable.php`**
  - Lines 84-110: `beforeMarshal()` - Checkbox value conversion
  - Lines 119-254: `beforeSave()` - API integration
  - Lines 260-303: `deactivateSimViaApi()`
  - Lines 312-378: `activateSimViaApi()`
  - Lines 387-426: `suspendSimViaApi()`

- **`plugins/Devices/src/Model/Entity/DeviceSimStatus.php`**
  - Lines 45-60: Virtual properties (status_badge_class, carrier_display)
  - Lines 69-75: `needsSync()` method

#### Device Carrier Detection
- **`plugins/Devices/src/Model/Entity/Device.php`**
  - Lines 235-241: `_getIsVz()` - Verizon SIM active check
  - Lines 243-249: `_getIsAtt()` - AT&T SIM active check
  - Lines 251-257: `_getIsTmo()` - T-Mobile SIM active check
  - Lines 259-262: `_getIsDual()` - Dual SIM check
  - Lines 264-279: `_getCarrierType()` - Carrier type calculation
  - Lines 361-376: `_getSimStatusByCarrier()` - SIM status mapping

#### Device Validation & Processing
- **`plugins/Devices/src/Model/Table/DevicesTable.php`**
  - Lines 578-646: `matchesActiveSimCount` validation rule
  - Lines 2963-3060: `beforeMarshal()` - SIM detection and config assignment
  - Lines 3069-3085: `isSimActive()` helper
  - Lines 3096-3109: `buildSimStatusData()` helper
  - Lines 3120-3151: `handleSimNumberDeactivation()`
  - Lines 3340-3394: `updateSimStatusForCarrier()`

#### CLI Commands
- **`plugins/Devices/src/Command/PopulateSimStatusesCommand.php`**
  - Initial population of device_sim_statuses records
  - Usage: `bin/cake populate_sim_statuses [--device-id=X] [--dry-run] [--verbose]`

#### API Callbacks
- **`plugins/Devices/src/Controller/Api/VerizonCallbacksController.php`**
  - Lines 84-189: Webhook handler for Verizon activation/deactivation responses
  - Updates `provider_status` and `last_api_sync`

---

### Database Tables

#### device_sim_statuses
```sql
CREATE TABLE device_sim_statuses (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    device_id INT UNSIGNED NOT NULL,
    carrier ENUM('att', 'tmobile', 'verizon') NOT NULL,
    sim_status ENUM('active', 'inactive') DEFAULT 'inactive' NOT NULL,
    provider_status VARCHAR(50) NULL,
    last_api_sync DATETIME NULL,
    created DATETIME NOT NULL,
    modified DATETIME NOT NULL,
    INDEX idx_device_id (device_id),
    UNIQUE INDEX idx_device_carrier (device_id, carrier),
    INDEX idx_sim_status (sim_status),
    INDEX idx_last_api_sync (last_api_sync),
    FOREIGN KEY (device_id) REFERENCES devices(id) ON DELETE CASCADE ON UPDATE RESTRICT
);
```

**Migration:** `config/Migrations/20250905185829_CreateDeviceSimStatuses.php`

#### configurations
```sql
CREATE TABLE configurations (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    config_group_id INT UNSIGNED NOT NULL,
    o_file_id CHAR(36) NOT NULL,
    title VARCHAR(255) NOT NULL UNIQUE,
    host_name VARCHAR(255) NOT NULL,
    carrier ENUM('vz', 'att', 'tmo', 'dual_vz_att', 'dual_vz_tmo') NOT NULL,
    cellular_backup TINYINT(1) DEFAULT 0 NOT NULL,
    status ENUM('pending', 'approved', 'denied', 'pending_delete') NOT NULL,
    created_by_user_id INT UNSIGNED NULL,
    modified_by_user_id INT UNSIGNED NULL,
    created DATETIME NOT NULL,
    modified DATETIME NOT NULL,
    FOREIGN KEY (config_group_id) REFERENCES config_groups(id) ON DELETE CASCADE,
    FOREIGN KEY (o_file_id) REFERENCES o_files(id) ON DELETE RESTRICT
);
```

**Migrations:**
- `20221109223106_CreateConfigurations.php` - Initial creation
- `20250904180411_UpdateConfigurationsCarrierEnum.php` - Added 'tmo' carrier
- `20250904180921_RenameConfigurationsDualCarrierEnum.php` - Added dual variants

#### configuration_service_plans
```sql
CREATE TABLE configuration_service_plans (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    configuration_id INT UNSIGNED NOT NULL,
    service_plan_id INT UNSIGNED NOT NULL,
    FOREIGN KEY (configuration_id) REFERENCES configurations(id) ON DELETE CASCADE,
    FOREIGN KEY (service_plan_id) REFERENCES service_plans(id) ON DELETE CASCADE
);
```

**Migration:** `20221110194041_CreateConfigurationGlobalDataTierGroups.php` (renamed in `20221209141247`)

---

### Configuration Assignment Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    Configuration Saved/Updated                   │
│                 (ConfigurationsTable::afterSave)                 │
└────────────────────────────────┬────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│  Find Devices Matching:                                          │
│  1. device_model_id = config.config_group.device_model_id       │
│  2. service_plan_id IN config.service_plans                     │
│  3. is_cellular_backup = config.cellular_backup                 │
│  4. DeviceSimStatuses indicate matching carrier                 │
└────────────────────────────────┬────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│  Filter by Carrier Type:                                         │
│  • 'vz': Has VZ SIM + active VZ status + NOT dual_sim           │
│  • 'att': Has ATT SIM + active ATT status + NOT dual_sim        │
│  • 'tmo': Has TMO SIM + active TMO status + NOT dual_sim        │
│  • 'dual_vz_att': is_dual_sim + active ATT status               │
│  • 'dual_vz_tmo': is_dual_sim + active TMO status               │
└────────────────────────────────┬────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│  Final Verification:                                             │
│  Filter where device.carrier_type === config.carrier            │
│  (Uses virtual property that checks active SIM statuses)        │
└────────────────────────────────┬────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│  Update Matching Devices:                                        │
│  UPDATE devices SET configuration_id = X WHERE id IN (...)      │
└─────────────────────────────────────────────────────────────────┘
```

---

### Device Carrier Type Calculation Flow

```
Device Entity Loaded
        │
        ▼
┌──────────────────────────────────────────────────────────────────┐
│  Load DeviceSimStatuses Association                               │
│  (hasMany relationship, eager or lazy loaded)                    │
└───────────────────────────┬──────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────────┐
│  Virtual Property: sim_status_by_carrier                         │
│  Returns: ['verizon' => SimStatus, 'att' => SimStatus, ...]     │
└───────────────────────────┬──────────────────────────────────────┘
                            │
        ┌───────────────────┴───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│   is_vz      │   │   is_att     │   │   is_tmo     │
│              │   │              │   │              │
│ VZ SIM +     │   │ ATT SIM +    │   │ TMO SIM +    │
│ Active Status│   │ Active Status│   │ Active Status│
└──────┬───────┘   └──────┬───────┘   └──────┬───────┘
       │                  │                  │
       └──────────────────┴──────────────────┘
                          │
                          ▼
              ┌──────────────────────┐
              │   carrier_type       │
              │                      │
              │ if (vz && att && ds) │
              │   → 'dual_vz_att'    │
              │ if (vz && tmo && ds) │
              │   → 'dual_vz_tmo'    │
              │ if (vz)              │
              │   → 'vz'             │
              │ if (att)             │
              │   → 'att'            │
              │ if (tmo)             │
              │   → 'tmo'            │
              │ else                 │
              │   → ''               │
              └──────────────────────┘

ds = is_dual_sim flag
```

---

## Next Steps

### Immediate Actions

1. **Schedule Client Meeting**
   - Review identified issues
   - Get answers to questions above
   - Prioritize fixes based on business impact

2. **Data Assessment**
   - Run query to find devices without `device_sim_statuses` records:
     ```sql
     SELECT d.id, d.serial_number, d.verizon_sim_number, d.att_sim_number, d.tmo_sim_number
     FROM devices d
     LEFT JOIN device_sim_statuses dss ON d.id = dss.device_id
     WHERE (d.verizon_sim_number IS NOT NULL OR d.att_sim_number IS NOT NULL OR d.tmo_sim_number IS NOT NULL)
     AND dss.id IS NULL;
     ```
   - Estimate impact: How many devices affected?

3. **Quick Fix Option**
   - Run `bin/cake populate_sim_statuses` to create missing records
   - This is non-destructive and can be done immediately
   - May resolve configuration assignment issues for existing devices

### Development Roadmap

**Phase 1: Critical Fixes (1-2 weeks)**
- [ ] Implement auto-creation of DeviceSimStatuses records
- [ ] Add device-triggered configuration reassignment
- [ ] Fix company filtering in configuration assignment

**Phase 2: Validation & UX (1 week)**
- [ ] Add dual SIM configuration availability validation
- [ ] Surface API sync status in device edit UI
- [ ] Add retry mechanism for failed API calls

**Phase 3: Maintenance & Monitoring (Ongoing)**
- [ ] Implement periodic API sync job
- [ ] Add monitoring/alerts for configuration mismatches
- [ ] Create admin dashboard for SIM status health

---

## Testing Checklist

### Manual Testing Scenarios

**Scenario 1: New Device with Single SIM**
- [ ] Create device with Verizon SIM only
- [ ] Verify `device_sim_statuses` record auto-created
- [ ] Verify `sim_status = 'active'`
- [ ] Verify correct 'vz' configuration assigned
- [ ] Verify `provider_status` updated after API call

**Scenario 2: New Device with Dual SIM**
- [ ] Create device with VZ + AT&T SIMs
- [ ] Set `is_dual_sim = 1`
- [ ] Verify two `device_sim_statuses` records created
- [ ] Verify both `sim_status = 'active'`
- [ ] Verify 'dual_vz_att' configuration assigned

**Scenario 3: Change Service Plan**
- [ ] Device starts with "Basic Plan" + VZ SIM
- [ ] Change to "Premium Plan"
- [ ] Verify configuration auto-updated to Premium config
- [ ] Verify device functionality maintained

**Scenario 4: Add Second SIM (Single → Dual)**
- [ ] Start with device having VZ SIM only (single config)
- [ ] Add AT&T SIM number
- [ ] Activate AT&T SIM status
- [ ] Set `is_dual_sim = 1`
- [ ] Verify configuration changes to dual_vz_att
- [ ] Verify both SIMs show correct API status

**Scenario 5: Company Isolation**
- [ ] Create configuration for Company A
- [ ] Create matching devices for Company A and Company B
- [ ] Verify only Company A devices get configuration
- [ ] Verify Company B devices do not get assigned

**Scenario 6: API Failure Handling**
- [ ] Simulate Verizon API failure (disconnect network)
- [ ] Attempt to activate SIM
- [ ] Verify user sees error message
- [ ] Verify `provider_status` reflects failure
- [ ] Verify retry button appears

**Scenario 7: Missing Configuration**
- [ ] Try to enable dual SIM on device
- [ ] No dual_vz_att config exists for device model
- [ ] Verify validation error with helpful message
- [ ] Verify device not saved in invalid state

---

## Glossary

**Terms:**

- **Base Configuration:** Standard configuration stored in `configurations` table, applies to multiple devices
- **Custom Company Configuration:** Company-specific override of base configuration
- **Carrier Type:** Computed value indicating active carrier combination ('vz', 'att', 'tmo', 'dual_vz_att', 'dual_vz_tmo')
- **SIM Status:** User-controlled desired state ('active' or 'inactive')
- **Provider Status:** Actual state reported by carrier API
- **Configuration Group:** Bridge between device models and configurations
- **Service Plan:** Billing plan that determines device pricing and data allowances
- **Dual SIM Device:** Device with two cellular carriers, typically Verizon + (AT&T or T-Mobile)

**Carrier Codes:**
- `vz` = Verizon
- `att` = AT&T
- `tmo` = T-Mobile
- `dual_vz_att` = Dual carrier (Verizon + AT&T)
- `dual_vz_tmo` = Dual carrier (Verizon + T-Mobile)

---

## Document History

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2025-10-29 | 1.0 | Claude Code | Initial analysis and documentation |

---

## Appendix: Related Documentation

- **Base Device Configurations:** `Meetings/Dual_Sim_meeting.md`
- **Configuration System:** `claude/apw_concepts/configurations.md` (if exists)
- **Device Status Switching:** `claude/apw_concepts/device_status_switching.md`
- **Billing System:** `claude/apw_concepts/billing.md`
- **Migration Guidelines:** `claude/migrations.md`

---

**END OF DOCUMENT**
