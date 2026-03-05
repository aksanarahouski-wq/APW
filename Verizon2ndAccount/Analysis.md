# Verizon Second Account Integration - Technical Analysis

**Date:** 2026-01-28
**Analyst:** Claude Code
**Client Request:** Support for managing devices across two Verizon Business Internet accounts

## Executive Summary

The client needs to support a second Verizon account in the WATM portal. The proposed solution of storing the Verizon Account at the Service Plan level **will NOT work in all cases** due to the following critical finding:

**Devices can exist and operate without assigned service plans**, yet they still require Verizon API interactions for activation, deactivation, suspension, and status checks.

## Current Architecture Analysis

### 1. Service Plan Relationship

**Device Entity (`devices` table):**
- `service_plan_id` (int|null) - **NULLABLE field**
- `service_plan_tier_id` (int|null) - **NULLABLE field**

**Validation Rules (DevicesTable.php:401-402):**
```php
$validator
    ->nonNegativeInteger('service_plan_id')
    ->allowEmptyString('service_plan_id');
```

**Key Finding:** Service plans are **optional** for devices, not required.

### 2. Verizon API Integration

**Current Implementation (`plugins/Devices/src/Util/VerizonApi.php`):**

All Verizon API methods accept an optional `$accountName` parameter and fall back to global configuration:

```php
public function activateDevice(
    $deviceIds,
    string $servicePlan = null,
    string $zipCode = null,
    string $accountName = null  // Optional parameter
): array {
    // ...
    $accountName = $accountName ?? Configure::read('Verizon.accountName');
    // ...
}
```

**Critical Issue:** In **ALL current usages** throughout the codebase, the `$accountName` parameter is **NEVER passed** - all calls rely on the default configuration value.

### 3. Places Where Verizon API is Called

#### DevicesTable.php (lines 1690-1726)
**Context:** Device status changes (Active, Deactivated, Suspended)

```php
$vzApi = new VerizonApi();
switch ($newStatus) {
    case 'Deactivated':
        $vzApi->deactivateDevice($entity->vz_activate_id);  // NO accountName passed
        break;
    case 'Admin Suspension':
    case 'Customer Suspension':
    case 'Data Suspension':
        $vzApi->suspendDevice($entity->vz_activate_id);  // NO accountName passed
        break;
    case 'Active':
        if (in_array($previousStatus, ['Admin Suspension', ...])) {
            $vzApi->resumeDevice($entity->vz_activate_id);  // NO accountName passed
        } elseif ($previousStatus === 'Deactivated') {
            $vzApi->activateDevice($entity->vz_activate_id);  // NO accountName passed
        }
}
```

**Service Plan Check:** None. The code does NOT verify if the device has a service plan before making API calls.

#### DeviceSimStatusesTable.php (lines 343-456)
**Context:** SIM status management (deactivate/activate)

```php
private function deactivateSimViaApi(string $carrier, string $simNumber, $device): ?string
{
    switch ($carrier) {
        case 'verizon':
            if (!empty($device->vz_activate_id)) {
                $vzApi = new VerizonApi();
                $vzApi->deactivateDevice($device->vz_activate_id);  // NO accountName passed
                return 'pending_deactivated';
            }
            return null;
    }
}
```

**Service Plan Check:** None.

#### VerizonCallbacksController.php (lines 318, 371, 406)
**Context:** Handling asynchronous callbacks from Verizon

```php
$attApi->resumeDevice($device->att_sim_number);  // NO accountName passed
$attApi->suspendDevice($device->att_sim_number);  // NO accountName passed
```

**Service Plan Check:** None.

### 4. Edge Cases Where Service Plan is NOT Available

#### Case 1: Device Import
**File:** `plugins/Devices/templates/Admin/Devices/import.php`

**Import Template Columns:**
- Manufacturer
- Model
- Serial Number
- Manufacturer Serial Number
- IMEI/ESN
- **Verizon SIM Number** ✓
- Verizon SIM Active
- AT&T SIM Number
- AT&T SIM Active
- T-Mobile SIM Number
- T-Mobile SIM Active
- Warranty Start/End Date
- IP Address
- WiFi Capable

**Missing Column:** Service Plan

**Impact:** Devices can be bulk imported with Verizon SIM numbers but NO service plan assigned. If these devices need activation/deactivation, the Verizon API call would fail without a fallback mechanism.

#### Case 2: Manual Device Creation/Editing
**File:** `plugins/Devices/templates/Admin/Devices/edit.php:321-325`

```php
<?= $this->Form->control('service_plan_id', [
    'type' => 'select',
    'options' => $servicePlans,
    'empty' => true,  // Service plan is OPTIONAL
]) ?>
```

**Impact:** Users can create or edit devices without assigning a service plan.

#### Case 3: RMA/Device Replacement
**File:** `plugins/Devices/src/Model/Table/RmasTable.php`

When devices are replaced via RMA process, the new device may not immediately have a service plan assigned, but might need SIM activation.

#### Case 4: Initial Device Setup
When a new device is first added to the system:
1. Device is created with SIM number
2. Status might be set to "Active" to trigger activation
3. Service plan may not be assigned yet (assigned later during onboarding)

#### Case 5: Legacy Data
Existing devices in the database may have:
- `verizon_sim_number` populated
- `service_plan_id` = NULL

**Database Query Results:**
```sql
SELECT COUNT(*) FROM devices
WHERE verizon_sim_number IS NOT NULL
AND service_plan_id IS NULL;
```
If this query returns > 0, those devices cannot use service-plan-based account routing.

## Existing Service Plan Provider Group Fields

**HistoricalServicePlan Entity** already contains:
- `verizon_provider_group` (string)
- `att_provider_group` (string)
- `tmobile_provider_group` (string)

**Current Usage:** These fields appear to store carrier-specific grouping information, but their exact purpose needs clarification. They might be:
- Carrier plan identifiers
- Device grouping for carrier APIs
- Rate plan codes

**Question:** Can `verizon_provider_group` be repurposed or extended to include account information?

## Analysis of Proposed Solution

### Proposed: Store Verizon Account on Service Plan Level

**Pros:**
✓ Logical grouping - different service plans could be associated with different accounts
✓ Minimal schema changes if using existing `verizon_provider_group` field
✓ Easy to manage via admin interface

**Cons:**
✗ **CRITICAL:** Devices can exist without service plans
✗ **CRITICAL:** All current Verizon API calls would fail for devices without service plans
✗ Requires service plan to be mandatory for all devices with Verizon SIMs (breaking change)
✗ Requires refactoring of all API call sites to check for service plan first
✗ Cannot handle edge cases (import, initial setup, RMA)
✗ Legacy data with NULL service_plan_id would need migration

**Risk Assessment:** HIGH - This approach will break existing functionality

## Alternative Solutions

### Option 1: Device-Level Verizon Account (RECOMMENDED)

**Implementation:**
1. Add new field to `devices` table: `verizon_account_name` (string, nullable)
2. Modify `VerizonApi` instantiation to accept account credentials:
   ```php
   $accountName = $entity->verizon_account_name ?? Configure::read('Verizon.accountName');
   $vzApi = new VerizonApi(null, null, null, null, $accountName);
   ```
3. Add account selection dropdown in device edit form
4. Store multiple account configurations in config

**Pros:**
✓ Works for ALL devices regardless of service plan
✓ Minimal code changes
✓ Backward compatible (defaults to main account if null)
✓ Handles all edge cases
✓ Direct, explicit control per device

**Cons:**
✗ Adds field to devices table (potential performance impact on large table)
✗ Requires manual selection per device during creation/import
✗ Need to add account field to import template

**Risk Assessment:** LOW

### Option 2: Company-Level Verizon Account

**Implementation:**
1. Add `verizon_account_name` to `companies` table
2. Devices inherit account from their company
3. Allow company-level configuration override

**Pros:**
✓ Works for all devices in a company
✓ Simplifies management - one setting per company
✓ Leverages existing multi-tenant architecture
✓ Handles edge cases automatically

**Cons:**
✗ No per-device flexibility
✗ Cannot mix accounts within a single company
✗ May not match client's use case if devices in same company use different accounts

**Risk Assessment:** MEDIUM

### Option 3: Hybrid Approach (MOST ROBUST)

**Implementation:**
1. Add `verizon_account_name` to service plans (optional)
2. Add `verizon_account_name` to devices (optional)
3. Add `verizon_account_name` to companies (optional)
4. Implement fallback chain:
   ```php
   $accountName = $device->verizon_account_name           // Device level (highest priority)
                  ?? $device->service_plan->verizon_account_name  // Service plan level
                  ?? $device->company->verizon_account_name       // Company level
                  ?? Configure::read('Verizon.accountName');      // Global default (fallback)
   ```

**Pros:**
✓ Maximum flexibility
✓ Handles ALL edge cases
✓ Backward compatible
✓ Allows granular control when needed
✓ Graceful degradation to global config

**Cons:**
✗ More complex implementation
✗ Multiple places to configure
✗ Requires clear documentation of precedence rules
✗ More fields to maintain

**Risk Assessment:** LOW (but higher complexity)

### Option 4: Service Plan with Fallback (BALANCED)

**Implementation:**
1. Add `verizon_account_name` to `historical_service_plans` table
2. Add fallback to global config for devices without service plans:
   ```php
   $accountName = $device->service_plan?->last_approved_historical_service_plan?->verizon_account_name
                  ?? Configure::read('Verizon.accountName');
   ```
3. Update all Verizon API call sites to use this pattern

**Pros:**
✓ Uses proposed service plan approach when possible
✓ Handles edge cases via fallback
✓ Backward compatible
✓ Moderate complexity

**Cons:**
✗ Devices without service plans always use default account (may not be desired)
✗ Need to update ~15+ call sites in code
✗ Cannot differentiate between "no preference" and "use default account"

**Risk Assessment:** LOW-MEDIUM

## Implementation Checklist (for any solution)

### Code Changes Required:

1. **VerizonApi.php modifications:**
   - [ ] Update constructor to accept dynamic account credentials
   - [ ] Ensure all methods properly use passed account name
   - [ ] Fix bug in `changeCustomFields()` method (missing $accountName parameter usage at line 498-501)

2. **All Verizon API call sites (~15 locations):**
   - [ ] DevicesTable.php (lines 1702, 1707, 1715, 1717)
   - [ ] DeviceSimStatusesTable.php (lines 350, 478, 480, 608)
   - [ ] VerizonCallbacksController.php (lines 318, 371, 406)
   - [ ] Various Command classes
   - [ ] Update each to determine and pass correct account name

3. **Configuration Management:**
   - [ ] Create multiple Verizon account configurations in app_local.php:
     ```php
     'Verizon' => [
         'default' => [
             'key' => 'xxx',
             'secret' => 'xxx',
             'username' => 'xxx',
             'password' => 'xxx',
             'accountName' => 'xxx',
         ],
         'secondary' => [
             'key' => 'eb64fb35-0b07-4636-9278-258045cdde5e',
             'secret' => '54c3c486-ac0f-45a0-bcb5-9b1171f2a11c',
             'username' => 'Orases5gAPI',
             'password' => '2AerKID*90Okdjh3@!',
             'accountName' => '542647743-00001',
         ],
     ]
     ```
   - [ ] Update VerizonApi to support multi-account configuration

4. **Database Schema:**
   - [ ] Create migration for chosen solution (device/service plan/company/hybrid)
   - [ ] Add appropriate indexes
   - [ ] Migrate existing data

5. **UI/UX Updates:**
   - [ ] Add account selection dropdown in device edit form
   - [ ] Add account column to device import template
   - [ ] Add account field to service plan edit form (if using that approach)
   - [ ] Add account indicator to device view page
   - [ ] Update bulk device operations to handle account selection

6. **Testing:**
   - [ ] Test device activation with both accounts
   - [ ] Test device deactivation with both accounts
   - [ ] Test device suspension/resume with both accounts
   - [ ] Test device import with account specification
   - [ ] Test devices without service plans
   - [ ] Test callback handling for both accounts
   - [ ] Test status changes across different accounts
   - [ ] Test edge cases (RMA, bulk updates, etc.)

7. **Documentation:**
   - [ ] Update CLAUDE.md with multi-account architecture
   - [ ] Document account selection rules/precedence
   - [ ] Create admin guide for managing multiple accounts
   - [ ] Update API documentation

## Recommended Solution: Option 1 (Device-Level)

**Justification:**
1. **Lowest Risk:** Handles all edge cases without breaking existing functionality
2. **Clearest Intent:** Explicit per-device control makes debugging easier
3. **Simplest Implementation:** Single field, straightforward lookup, minimal code changes
4. **Most Flexible:** Allows per-device account assignment regardless of service plan status
5. **Backward Compatible:** Existing devices default to main account automatically

**Implementation Steps:**
1. Add `verizon_account_name` field to `devices` table (nullable string)
2. Create migration to add the field
3. Update Device entity to include the field
4. Add account configuration in app_local.php
5. Modify all Verizon API call sites (15 locations) to pass account name:
   ```php
   $accountConfig = $this->getVerizonAccountConfig($device->verizon_account_name);
   $vzApi = new VerizonApi(
       $accountConfig['key'],
       $accountConfig['secret'],
       $accountConfig['username'],
       $accountConfig['password']
   );
   ```
6. Add UI dropdown for account selection in device forms
7. Add account column to import template
8. Test thoroughly

## Security Considerations

1. **API Credentials Storage:**
   - All account credentials currently stored in app_local.php (not in database)
   - Consider using environment variables or encrypted configuration
   - Limit access to account configuration to admin users only

2. **Account Validation:**
   - Validate account name before making API calls
   - Handle invalid/missing account gracefully
   - Log account-specific API failures separately for debugging

3. **Audit Logging:**
   - Log which account is used for each API operation
   - Track account changes on devices
   - Monitor API usage per account for billing purposes

## Questions for Client

1. **Account Distribution:**
   - How will devices be distributed between the two accounts?
   - Is there a pattern (by company, by device type, by region)?

2. **Migration:**
   - Should all existing devices default to the original account?
   - Are there specific devices that should immediately use the new account?

3. **Billing:**
   - Does the Verizon account affect billing/invoicing in WATM?
   - Do different accounts have different rate plans?

4. **Future Scalability:**
   - Possibility of more than 2 accounts in the future?
   - Need for automatic account selection based on rules?

## Conclusion

**The proposed solution of storing Verizon Account at the Service Plan level will NOT work reliably** because:

1. Service plans are optional for devices
2. Devices can be imported, created, and operated without service plans
3. All Verizon API operations can occur on devices without service plans
4. Current code makes no checks for service plan existence before API calls

**Recommended approach:** Store Verizon account at the **Device level** with fallback to global configuration. This ensures all edge cases are handled while maintaining backward compatibility and providing clear, explicit control.

**Alternative viable approach:** Hybrid solution with precedence chain (device → service plan → company → global) for maximum flexibility at the cost of additional complexity.
