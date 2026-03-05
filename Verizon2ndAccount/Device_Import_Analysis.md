# Device Import Process Analysis - Verizon SIM Active Field

**Date:** 2026-01-28
**Question:** What happens when a device is imported with "Verizon SIM Active YES"?

## Quick Answer

**NO Verizon API call is made during device import.** The "Verizon SIM Active" field only creates a database record indicating the desired status - it does NOT activate the SIM via the Verizon API.

## Detailed Import Flow

### 1. Import Template Structure

**File:** `plugins/Devices/templates/Admin/Devices/import.php`

**Template Columns (15 total):**
1. Manufacturer
2. Model
3. Serial Number
4. Manufacturer Serial Number
5. IMEI/ESN
6. **Verizon SIM Number**
7. **Verizon SIM Active** (yes/no, y/n, 1/0)
8. AT&T SIM Number
9. AT&T SIM Active
10. T-Mobile SIM Number
11. T-Mobile SIM Active
12. Warranty Start Date
13. Warranty End Date
14. IP Address
15. WiFi Capable

**Notable Absence:** No Service Plan column

### 2. Import Processing Logic

**File:** `plugins/Devices/src/Controller/Admin/DevicesController.php:2085-2407`

#### Step 1: Parse Spreadsheet (lines 2146-2375)

```php
// Line 2154: Extract Verizon fields from spreadsheet
$verizon_sim_number, $verizon_sim_active,

// Line 2234: Boolean parser for yes/no/y/n/1/0/true/false
$parseBoolean = fn($value) =>
    is_string($value)
    && in_array(strtolower(trim($value)), ['yes', 'y', '1', 'true'], true);
```

#### Step 2: Validate SIM Active Flag (lines 2246-2254)

```php
// Validation: Cannot mark as active without SIM number
if (empty($verizon_sim_number) && $parseBoolean($verizon_sim_active)) {
    $this->Flash->error(
        __(
            'Device with serial number "{0}": Verizon SIM is marked as active but no Verizon SIM number was provided.',
            $serial_number
        )
    );
    $anyMissing = true;
    continue;
}
```

#### Step 3: Create DeviceSimStatus Record (lines 2294-2299)

```php
if (!empty($verizon_sim_number)) {
    $verizonActive = $parseBoolean($verizon_sim_active);
    $deviceSimStatuses[] = [
        'carrier' => 'verizon',
        'sim_status' => $verizonActive ? 'active' : 'inactive',
        // NOTE: No provider_status set for Verizon (unlike AT&T/T-Mobile which get 'TEST_READY')
    ];
}
```

**Key Observations:**
- Creates a `device_sim_statuses` record with:
  - `carrier` = 'verizon'
  - `sim_status` = 'active' or 'inactive'
  - `provider_status` = NULL (not set during import)
- **Does NOT call Verizon API**
- **Does NOT validate with Verizon that SIM exists or is active**

#### Step 4: Compare to AT&T and T-Mobile (lines 2301-2317)

```php
// AT&T SIM (lines 2301-2308)
if (!empty($att_sim_number)) {
    $attActive = $parseBoolean($att_sim_active);
    $deviceSimStatuses[] = [
        'carrier' => 'att',
        'sim_status' => $attActive ? 'active' : 'inactive',
        'provider_status' => 'TEST_READY',  // ← Sets provider_status!
    ];
}

// T-Mobile SIM (lines 2310-2317)
if (!empty($tmo_sim_number)) {
    $tmoActive = $parseBoolean($tmo_sim_active);
    $deviceSimStatuses[] = [
        'carrier' => 'tmobile',
        'sim_status' => $tmoActive ? 'active' : 'inactive',
        'provider_status' => 'TEST_READY',  // ← Sets provider_status!
    ];
}
```

**Discrepancy:** AT&T and T-Mobile SIMs get `provider_status = 'TEST_READY'`, but Verizon does not get any provider_status during import.

#### Step 5: Save Device (lines 2388)

```php
if ($this->Devices->save($device, ['creation_method' => 'import', 'atomic' => false])) {
    $savedCount++;
}
```

**Important:** Passes `'creation_method' => 'import'` option to save.

### 3. Device Save Callbacks

#### DevicesTable::beforeSave (lines 1277-1434)

**Group Assignment Logic (lines 1316-1340):**
```php
if ($entity->isDirty('service_plan_id') && !empty($entity->service_plan_id)) {
    // Assigns device to Verizon group if service plan has verizon_provider_group
    $vzApi->assignGroupToDevice(
        $entity->verizon_sim_number,
        $servicePlan->last_approved_historical_service_plan->verizon_provider_group
    );
}
```

**Impact on Import:**
- This code only runs if `service_plan_id` is dirty (changed) and not empty
- During import, devices typically have NO service plan assigned
- Therefore, **NO group assignment API call is made**

#### DevicesTable::afterSave (lines 1436-1650+)

**Logging Only:**
```php
if ($entity->isNew()) {
    $creationMethod = $options['creation_method'] ?? '';
    $logMessage = __('Device: {0} created by {1}', $entity->serial_number, $creationMethod);
    // Logs to o_logs table
}
```

**Impact on Import:**
- Only creates audit log entry
- **NO API calls made**

### 4. DeviceSimStatus Save Callbacks

#### DeviceSimStatusesTable::beforeSave (lines 192-333)

**Critical Early Return (line 195-196):**
```php
// Only process if sim_status changed
if (!$entity->isDirty('sim_status') || $entity->isNew()) {
    return;  // ← EXITS EARLY FOR NEW RECORDS
}
```

**Impact on Import:**
- When device_sim_statuses records are created during import, they are NEW entities
- `$entity->isNew()` returns TRUE
- Method returns immediately
- **NO API calls are made**
- **NO activation/deactivation/suspension logic executes**

**Later Logic (would execute if not new):**
- Line 315-326: Handles sim_status changes
  - 'inactive' → calls `deactivateSimViaApi()`
  - 'active' → calls `activateSimViaApi()` or `suspendSimViaApi()`
- Line 343-456: API call methods
  - `deactivateSimViaApi()` - Verizon deactivation
  - `activateSimViaApi()` - Verizon activation/resume
  - `suspendSimViaApi()` - Verizon suspension

**But none of this executes during import because isNew() returns true.**

## Summary: What Actually Happens During Import

### When "Verizon SIM Active YES" is imported:

1. ✓ Device record created with `verizon_sim_number` populated
2. ✓ DeviceSimStatus record created:
   - carrier = 'verizon'
   - sim_status = 'active'
   - provider_status = NULL
3. ✓ Audit log entry created
4. ✗ **NO Verizon API call made**
5. ✗ **NO actual activation on Verizon network**
6. ✗ **NO validation that SIM exists in Verizon system**
7. ✗ **NO group assignment (no service plan during import)**

### Database State After Import:

```sql
-- devices table
verizon_sim_number: '89148000012345678901'
vz_activate_id: NULL  -- Not populated during import
vz_group_applied: NULL  -- Not assigned without service plan

-- device_sim_statuses table
carrier: 'verizon'
sim_status: 'active'
provider_status: NULL  -- Not set during import
last_api_sync: NULL  -- No API call made
```

### Key Difference from AT&T/T-Mobile:

**AT&T and T-Mobile:**
- Set `provider_status = 'TEST_READY'` during import
- This special status prevents accidental activation later (see line 247-310)
- Requires explicit action to move from TEST_READY to active

**Verizon:**
- Sets `sim_status = 'active'` but `provider_status = NULL`
- No protective "ready state"
- Status is purely informational, not synced with carrier

## When API Calls ARE Made

Verizon API calls happen in these scenarios (all AFTER initial import):

### 1. Manual Device Status Change
**File:** `DevicesTable.php:1690-1726`

When a user manually changes device status:
- Active → Deactivated: calls `deactivateDevice()`
- Active → Suspended: calls `suspendDevice()`
- Suspended → Active: calls `resumeDevice()`
- Deactivated → Active: calls `activateDevice()`

**Note:** No service plan check before these calls!

### 2. SIM Status Toggle (After Import)
**File:** `DeviceSimStatusesTable.php:192-333`

When a user changes sim_status on an existing DeviceSimStatus record:
- Only if `isDirty('sim_status')` AND NOT `isNew()`
- Calls appropriate API method based on change

### 3. Service Plan Assignment
**File:** `DevicesTable.php:1316-1340`

When a service plan is assigned to a device:
- Calls `assignGroupToDevice()` to assign Verizon group
- Only if service plan has `verizon_provider_group` configured

### 4. Verizon Callbacks
**File:** `VerizonCallbacksController.php`

Asynchronous callbacks from Verizon API update status after API calls complete.

## Impact on Second Verizon Account Implementation

### Critical Finding:

Since import does NOT call Verizon API, the second account implementation must handle:

1. **Import Process:**
   - Need to determine which account the imported SIM belongs to
   - Cannot validate SIM existence during import
   - Must store account information before any API calls are made

2. **Post-Import Operations:**
   - First API call after import will need to determine correct account
   - If using service plan approach, devices imported without service plans cannot make API calls
   - If using device-level approach, need to add account field to import template

### Recommendations:

**For Import Template:**

Option A: Add "Verizon Account" column to import template
```
Column 16: Verizon Account (optional: 'default', 'secondary', or account name)
```

Option B: Use existing "Verizon SIM Number" to auto-determine account
- Parse SIM number prefix/range to determine account
- Requires maintaining SIM range mapping

Option C: Default all imports to primary account
- Allow manual reassignment post-import
- Requires bulk update feature

## Questions This Analysis Raises

1. **Why is `vz_activate_id` not populated during import?**
   - This field is critical for Verizon API calls
   - Currently must be set manually or via API response
   - Should import populate this from verizon_sim_number?

2. **Why no `provider_status` set for Verizon during import?**
   - AT&T/T-Mobile get 'TEST_READY' as protective status
   - Verizon just gets NULL
   - Could lead to confusion about actual SIM state

3. **What happens if user tries to activate imported "active" SIM?**
   - Device has sim_status='active' in database
   - But never actually activated via API
   - User might assume it's already active when it's not

4. **How to handle SIM number → Account mapping?**
   - During import, how to determine which account SIM belongs to?
   - Manual specification vs. automatic detection
   - Validation of SIM ownership by account

## Code References

- **Import Controller:** `plugins/Devices/src/Controller/Admin/DevicesController.php:2085-2407`
- **Device Import Template:** `plugins/Devices/templates/Admin/Devices/import.php`
- **DevicesTable beforeSave:** `plugins/Devices/src/Model/Table/DevicesTable.php:1277-1434`
- **DevicesTable afterSave:** `plugins/Devices/src/Model/Table/DevicesTable.php:1436-1650+`
- **DeviceSimStatusesTable beforeSave:** `plugins/Devices/src/Model/Table/DeviceSimStatusesTable.php:192-333`
- **VerizonApi class:** `plugins/Devices/src/Util/VerizonApi.php`

## Recommended Next Steps

1. **Clarify Import Behavior:**
   - Document that "SIM Active" during import is informational only
   - Add warning in UI that import does not activate SIMs
   - Consider adding validation step after import

2. **Add Account to Import:**
   - Add Verizon Account column to template (if using device-level approach)
   - Validate account selection during import parsing
   - Default to primary account if not specified

3. **Consider Post-Import Validation:**
   - Add bulk validation feature to check SIM status with Verizon API
   - Report discrepancies between database and actual carrier status
   - Option to sync database status with carrier

4. **Update Documentation:**
   - Update import guide to clarify SIM Active behavior
   - Document when API calls are made vs. when records are created
   - Explain difference between sim_status and provider_status
