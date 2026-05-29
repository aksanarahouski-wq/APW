# Client Response & Implementation Plan - Verizon Second Account

## Client Response Summary

### 1. Why Two Accounts?
- **Verizon-mandated separation** for "Fixed Wireless Access" (FWA) business use case
- **Different pricing models:**
  - **FWA Account:** Flat rate/unlimited plans ($70-80+ MRC), no overages, categorized as "Business Internet"
  - **Original Account:** Per-byte pricing, super low MRC plans, usage-based
- **Verizon restriction:** Makes it difficult to switch between unlimited and low MRC plans (to combat SIM banking, network abuse, fraud)
- **Different pricing tiers** rather than service agreements

### 2. Device Differentiation
- **Through service plan assignments**
- **One-way customer action:** Customers can change TO FWA plans once, but CANNOT revert
- **Manual override required:** Company must manually process any device reversion from FWA → Regular account
- **Implies:** This is a carefully controlled process with significant operational implications

### 3. Expected Distribution
- **FWA Account:** 10-15 devices currently active, no proven growth path
- **Original Account:** Vast majority of devices for "a very long time"
- **No 50/50 split:** Heavily weighted toward original account

### 4. Movement Between Accounts
- **Manual process** with many steps
- **Cumbersome,** especially after devices leave office
- **Limited by Verizon restrictions**

## Client's Preferred Solution

**Device Level Storage** with the following workflow:
1. Only customer-requested "unlimited" devices go on FWA account
2. Activate and upload FWA devices separately
3. System automatically knows which account based on device configuration
4. **Question:** Can we restrict service plan access based on Verizon account?
   - Prevent FWA account devices from accessing regular service plans
   - Prevent regular account devices from accessing FWA service plans

---

## Implementation Plan

### Solution: Device-Level Verizon Account with Service Plan Restrictions

Yes, we can implement service plan restrictions based on Verizon account assignment. Here's how:

### Phase 1: Core Infrastructure

#### 1.1 Database Schema Changes

**Migration: Add `verizon_account_name` to `devices` table**
```sql
ALTER TABLE devices
ADD COLUMN verizon_account_name VARCHAR(100) NULL DEFAULT NULL
COMMENT 'Verizon account identifier (default, fwa, or account number)';

-- Create index for filtering
CREATE INDEX idx_devices_verizon_account ON devices(verizon_account_name);
```

**Migration: Add `verizon_account_name` to `historical_service_plans` table**
```sql
ALTER TABLE historical_service_plans
ADD COLUMN verizon_account_name VARCHAR(100) NULL DEFAULT NULL
COMMENT 'Required Verizon account for devices using this service plan';

CREATE INDEX idx_historical_service_plans_verizon_account
ON historical_service_plans(verizon_account_name);
```

#### 1.2 Configuration Setup

**File: `config/app_local.php`**
```php
'Verizon' => [
    'accounts' => [
        'default' => [
            'key' => env('VERIZON_DEFAULT_KEY'),
            'secret' => env('VERIZON_DEFAULT_SECRET'),
            'username' => env('VERIZON_DEFAULT_USERNAME'),
            'password' => env('VERIZON_DEFAULT_PASSWORD'),
            'accountName' => env('VERIZON_DEFAULT_ACCOUNT_NAME'),
            'display_name' => 'Regular Account (Per-Byte Pricing)',
        ],
        'fwa' => [
            'key' => 'eb64fb35-0b07-4636-9278-258045cdde5e',
            'secret' => '54c3c486-ac0f-45a0-bcb5-9b1171f2a11c',
            'username' => 'Orases5gAPI',
            'password' => '2AerKID*90Okdjh3@!',
            'accountName' => '542647743-00001',
            'display_name' => 'FWA Account (Unlimited Plans)',
        ],
    ],
    // Fallback to default account if not specified
    'default_account' => 'default',
],
```

#### 1.3 VerizonApi Class Updates

**File: `plugins/Devices/src/Util/VerizonApi.php`**

Update constructor to accept account configuration:
```php
public function __construct(
    string $accountKey = 'default'  // New parameter: account config key
) {
    $accounts = Configure::read('Verizon.accounts');
    $accountConfig = $accounts[$accountKey] ?? $accounts['default'];

    $this->key = $accountConfig['key'];
    $this->secret = $accountConfig['secret'];
    $this->username = $accountConfig['username'];
    $this->password = $accountConfig['password'];
    $this->accountName = $accountConfig['accountName'];
    // ... rest of constructor
}
```

Helper method to get account from device:
```php
public static function getAccountKeyFromDevice(Device $device): string
{
    return $device->verizon_account_name ?? Configure::read('Verizon.default_account');
}
```

### Phase 2: Service Plan Restrictions

#### 2.1 Service Plan Configuration

**UI: Service Plan Edit Form**
- Add "Verizon Account Restriction" dropdown
- Description: "Restricts which devices can use this service plan based on their Verizon account"
- Options:
  - **Empty (NULL):** "No Restriction - Available to All Devices" (existing behavior, default)
  - **'default':** "Regular Account Only" (per-byte pricing devices)
  - **'fwa':** "FWA Account Only" (unlimited plan devices)

**Important:** A device can only be on ONE Verizon account at a time. This setting controls which account's devices are allowed to select this service plan.

**Example Configuration:**
- Regular per-byte pricing service plans → Set to "Regular Account Only"
- Unlimited FWA service plans → Set to "FWA Account Only"
- Generic/neutral service plans (if any) → Leave as "No Restriction"

#### 2.2 Validation Rules

**File: `plugins/Devices/src/Model/Table/DevicesTable.php`**

Add validation in `buildRules()`:
```php
// Validate service plan is compatible with device's Verizon account
$rules->add(
    function ($entity, $options) {
        if (empty($entity->service_plan_id)) {
            return true; // No service plan, no restriction
        }

        $servicePlan = $this->ServicePlans->get($entity->service_plan_id, [
            'contain' => 'LastApprovedHistoricalServicePlans'
        ]);

        $planVzAccount = $servicePlan->last_approved_historical_service_plan->verizon_account_name ?? null;

        // If service plan has no account restriction, allow any device
        if (empty($planVzAccount)) {
            return true;
        }

        $deviceVzAccount = $entity->verizon_account_name ?? 'default';

        // Check if accounts match
        if ($planVzAccount !== $deviceVzAccount) {
            $entity->setError('service_plan_id', [
                'verizon_account_mismatch' => sprintf(
                    'This service plan requires Verizon %s account, but device is configured for %s account.',
                    $planVzAccount,
                    $deviceVzAccount
                )
            ]);
            return false;
        }

        return true;
    },
    'verizonAccountServicePlanCompatibility'
);
```

#### 2.3 Service Plan Filtering

**File: `plugins/Devices/src/Controller/Admin/DevicesController.php`**

Update service plan dropdown to filter by device's Verizon account:
```php
private function getAvailableServicePlansForDevice(Device $device): array
{
    $deviceVzAccount = $device->verizon_account_name ?? 'default';

    $query = $this->Devices->ServicePlans->find('list')
        ->innerJoinWith('LastApprovedHistoricalServicePlans')
        ->where([
            'OR' => [
                'LastApprovedHistoricalServicePlans.verizon_account_name IS' => null, // No restriction
                'LastApprovedHistoricalServicePlans.verizon_account_name' => $deviceVzAccount, // Matches device
            ],
            'LastApprovedHistoricalServicePlans.status' => 'approved'
        ]);

    return $query->toArray();
}
```

### Phase 3: Verizon API Integration Updates

Update all 15+ locations where VerizonApi is instantiated:

**Pattern to apply everywhere:**
```php
// OLD CODE:
$vzApi = new VerizonApi();

// NEW CODE:
$accountKey = VerizonApi::getAccountKeyFromDevice($device);
$vzApi = new VerizonApi($accountKey);
```

**Files to update:**
1. `DevicesTable.php` (lines 1330, 1698)
2. `DeviceSimStatusesTable.php` (lines 348, 472)
3. `VerizonCallbacksController.php` (multiple locations)
4. Various Command classes
5. Test files

### Phase 4: UI Updates

#### 4.1 Device Edit Form

**File: `plugins/Devices/templates/Admin/Devices/edit.php`**

Add Verizon Account dropdown:
```php
<?= $this->Form->control('verizon_account_name', [
    'label' => __('Verizon Account'),
    'type' => 'select',
    'options' => [
        'default' => 'Regular Account (Per-Byte Pricing)',
        'fwa' => 'FWA Account (Unlimited Plans)'
    ],
    'empty' => '-- Use Default Account --',
    'templateVars' => [
        'help' => 'Select FWA account only for customer-requested unlimited devices. Cannot be changed by customers after activation.'
    ]
]) ?>
```

#### 4.2 Device Import Template

**File: `plugins/Devices/templates/Admin/Devices/import.php`**

Add Column 16: Verizon Account
- Values: empty (default), "default", "fwa", or account number
- Parser will default to 'default' if empty

**File: `plugins/Devices/src/Controller/Admin/DevicesController.php`**

Update import parsing (around line 2158):
```php
list(
    $manufacturer, $model, $serial_number,
    $manufacturer_serial_number, $imei_esn,
    $verizon_sim_number, $verizon_sim_active,
    $att_sim_number, $att_sim_active,
    $tmo_sim_number, $tmo_sim_active,
    $warranty_start_date, $warranty_end_date, $ip, $is_wifi_capable,
    $verizon_account_name  // NEW COLUMN
) = $spreadsheetRow;

// Normalize and default
$verizon_account_name = $normalizeEmpty($verizon_account_name) ?? 'default';

// Validate account name
if (!in_array($verizon_account_name, ['default', 'fwa'])) {
    $this->Flash->error(
        __('Device with serial number "{0}": Invalid Verizon account "{1}". Must be "default" or "fwa".',
            $serial_number, $verizon_account_name)
    );
    $anyMissing = true;
    continue;
}

$deviceData = compact(
    // ... existing fields
    'verizon_account_name'  // Add to compact
);
```

#### 4.3 Device View/Index Pages

Add Verizon Account indicator:
- Show badge: "Regular Account" or "FWA Account"
- Color code: Blue for regular, Orange for FWA
- Add to device index table as filterable column

#### 4.4 Bulk Update Feature

**File: `plugins/Devices/src/Controller/Admin/DevicesController.php`**

Add bulk update action for Verizon account:
```php
public function bulkUpdateVerizonAccount()
{
    // Allow bulk changing of Verizon account
    // Include warning about service plan compatibility
    // Log all changes to o_logs table
}
```

### Phase 5: Customer Access Restrictions

#### 5.1 Service Plan Change Restrictions

**Business Rule:** Customers can change TO FWA plans once, but cannot revert

**Implementation:**

**File: `plugins/Devices/src/Model/Table/DevicesTable.php`**

Add in `beforeSave()`:
```php
// Check for service plan changes by customers
if ($entity->isDirty('service_plan_id') && !$entity->isNew()) {
    $identity = Router::getRequest()?->getAttribute('identity');
    $isCustomer = $identity?->get('role') === 'customer';

    if ($isCustomer) {
        // Get old and new service plans
        $oldPlanId = $entity->getOriginal('service_plan_id');
        $newPlanId = $entity->service_plan_id;

        if (!empty($oldPlanId) && !empty($newPlanId)) {
            $oldPlan = $this->ServicePlans->get($oldPlanId, ['contain' => 'LastApprovedHistoricalServicePlans']);
            $newPlan = $this->ServicePlans->get($newPlanId, ['contain' => 'LastApprovedHistoricalServicePlans']);

            $oldAccount = $oldPlan->last_approved_historical_service_plan->verizon_account_name ?? 'default';
            $newAccount = $newPlan->last_approved_historical_service_plan->verizon_account_name ?? 'default';

            // Block customer from reverting FWA → Regular
            if ($oldAccount === 'fwa' && $newAccount === 'default') {
                $entity->setError('service_plan_id', [
                    'fwa_revert_blocked' => 'You cannot change from an unlimited plan back to a regular plan. Please contact support for assistance.'
                ]);
                return false; // Prevent save
            }

            // Log FWA account activation by customer
            if ($oldAccount === 'default' && $newAccount === 'fwa') {
                $this->getEventManager()->dispatch(
                    new Event('App.captureLog', $this, [
                        __('Customer upgraded device {0} to FWA unlimited plan (one-way change)', $entity->serial_number),
                        'identity' => $identity,
                        'options' => [
                            'category' => 'DEVICE FWA UPGRADE',
                            'device_id' => $entity->id,
                            'company_id' => $entity->company_id,
                            'old_account' => $oldAccount,
                            'new_account' => $newAccount,
                        ],
                    ])
                );
            }
        }
    }
}
```

#### 5.2 Admin Override

**Add permission:** `Devices.Devices.revertFromFwa`

Admins with this permission can:
- Revert devices from FWA → Regular account
- Change service plan regardless of restrictions
- See special "Admin Override" checkbox when editing devices with FWA account

### Phase 6: Reporting & Monitoring

#### 6.1 Account Usage Dashboard

Create dashboard widget showing:
- Total devices per Verizon account
- Active devices per account
- Recent account changes
- Service plan distribution per account

#### 6.2 Audit Logging

Log all Verizon account-related events:
- Device created with FWA account
- Customer upgraded to FWA plan
- Admin reverted from FWA account
- API calls per account (for billing reconciliation)

### Phase 7: Data Migration

#### 7.1 Existing Devices

**Migration script:**
```php
// Set all existing devices to 'default' account
UPDATE devices
SET verizon_account_name = 'default'
WHERE verizon_sim_number IS NOT NULL;

// Explicitly set the 10-15 FWA devices (client provides list)
UPDATE devices
SET verizon_account_name = 'fwa'
WHERE id IN (123, 456, 789, ...); -- List of FWA device IDs
```

#### 7.2 Service Plans

**Configuration:**
- Mark FWA-only service plans with `verizon_account_name = 'fwa'`
- Leave regular plans with `verizon_account_name = NULL` (available to all)
- Client provides list of which service plans are FWA-only

---

## Implementation Timeline

### Phase 1: Core Infrastructure (Week 1)
- Database migrations
- VerizonApi class updates
- Configuration setup
- Update all 15+ API call locations

### Phase 2: Service Plan Restrictions (Week 1-2)
- Validation rules
- Service plan filtering
- UI updates for restrictions

### Phase 3: UI Updates (Week 2)
- Device edit form
- Import template (add column 16)
- Device view/index pages
- Bulk update feature

### Phase 4: Customer Restrictions (Week 2-3)
- One-way FWA upgrade logic
- Admin override permission
- Audit logging

### Phase 5: Testing (Week 3)
- Unit tests for validation rules
- Integration tests for API routing
- End-to-end testing:
  - Import with FWA account
  - Customer upgrade to FWA
  - Customer blocked from reverting
  - Admin override
  - API calls route to correct account

### Phase 6: Migration & Deployment (Week 3-4)
- Data migration script
- Service plan configuration
- Production deployment
- Monitoring setup

---

## Answering Your Specific Question

> "Maybe we can eliminate customers from accessing our other service plans if we restrict devices on this Verizon account from those service plans? Explain."

**Yes, we can implement this restriction in multiple ways:**

### Method 1: Service Plan Filtering (Recommended)
When a device has `verizon_account_name = 'fwa'`:
- Service plan dropdown ONLY shows FWA-compatible service plans
- Hides all regular account plans from the dropdown
- Customer never sees the option to select incompatible plans

**User Experience:**
- Device on FWA account → Customer only sees unlimited FWA plans
- Device on regular account → Customer only sees regular per-byte plans
- Clean, prevents confusion

### Method 2: Validation Block
If customer somehow tries to assign incompatible service plan:
- Validation error: "This service plan is not available for FWA devices"
- Save is blocked
- Error message explains restriction

### Method 3: One-Way Upgrade Only
- Allow customers to move FROM regular TO FWA plans
- Block any movement FROM FWA TO regular plans
- Requires admin intervention to revert

**Combined Approach (Recommended):**
1. Filter service plan dropdown by account compatibility (prevent confusion)
2. Add validation as safety net (in case of API/direct access)
3. Implement one-way upgrade restriction (business rule enforcement)
4. Provide admin override for manual reversions

This gives you:
✅ Customer can upgrade to unlimited (FWA) once
✅ Customer CANNOT revert to regular plans
✅ Customer doesn't see incompatible plans (clean UX)
✅ Admin can manually revert when needed
✅ All changes are logged for audit

---

## Security & Safety Considerations

### 1. Prevent Account Confusion
- Clear labeling in UI
- Color-coded badges
- Confirmation prompts for FWA upgrades

### 2. API Call Routing
- Always use device's `verizon_account_name` for API calls
- Never assume account based on service plan alone
- Fallback to 'default' if NULL

### 3. Import Safety
- Validate account name during import
- Log all FWA device imports separately
- Require explicit account specification (no accidental FWA assignments)

### 4. Audit Trail
- Log every account assignment/change
- Track who made the change (customer vs. admin)
- Log all API calls with account used

---

## Questions for Client

Before we begin implementation, please confirm:

1. **List of existing FWA devices:** Can you provide the serial numbers or device IDs of the 10-15 devices currently on the FWA account?

2. **List of FWA service plans:** Which service plans should be marked as "FWA only"? (We'll configure these to require FWA account)

3. **Customer communication:** Do you want us to add UI warnings/prompts when customer upgrades to FWA plan? (e.g., "This is a one-way change. You cannot revert to a regular plan without contacting support.")

4. **Bulk import process:** When you "upload them separately" for FWA devices, will you use a separate import file? Or same file with Column 16 marked as "fwa"?

5. **Admin users:** Who should have permission to revert devices from FWA → Regular account? (All admins or specific role?)

6. **Naming:** Are "Regular Account" and "FWA Account" the right display names for the UI? Or would you prefer different labels?

---

## Next Steps

1. **Client confirms** answers to questions above
2. **We provide** detailed estimate (hours/cost) for implementation
3. **Client approves** estimate and timeline
4. **We begin** Phase 1 development

Please review and let us know if this implementation plan meets your needs!
