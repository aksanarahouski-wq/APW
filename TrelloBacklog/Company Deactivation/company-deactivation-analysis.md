# Company Deactivation Flow Analysis & Bug Assessment

## Client Report (Beach Market - 2/17/26)

> On 2/17/26 at 11:25AM we finalized a company deactivation (Beach Market) and when we did that we elected to waive the final billing. This process also deactivated the single device on that account at that same timestamp. However, the customer was still invoiced for that device this month.

---

## Company Deactivation Flow (As Designed)

### Step 1: Admin Initiates Deactivation

**File:** `Companies/Controller/Admin/CompaniesController.php:744`

Admin clicks "Deactivate Company" button, which opens a confirmation modal with a **"Waive Final Bill"** checkbox.

### Step 2: beforeSave Intercepts — Forces Pending Status

**File:** `Companies/Model/Table/CompaniesTable.php:647-662`

When status is set to `deactivated` but was NOT previously `pending_deactivation` or `suspended`:
- **Status is redirected** to `pending_deactivation` (not deactivated)
- `waive_final_bill` is preserved (only cleared if new status is NOT deactivated)
- A **DeactivationApproval email** is sent to admin superusers

**Devices are NOT deactivated yet at this step** — pending_deactivation only sends the approval email.

### Step 3: Admin Approves — Finalizes Deactivation

Admin returns to the company edit page and changes status from `pending_deactivation` → `deactivated`. This time the beforeSave allows it through because previous status is `pending_deactivation`.

### Step 4: afterSave Deactivates Devices

**File:** `Companies/Model/Table/CompaniesTable.php:780-841`

When status becomes `deactivated` or `suspended`:
1. Gets ALL devices for the company
2. Saves `CompanyDeviceDeactivationStatuses` records (preserves previous device status for potential restore)
3. Sets all non-deactivated devices to `Deactivated` status
4. This triggers `DevicesTable::afterSave()` which:
   - Calls carrier APIs to deactivate SIMs
   - Clears the device IP

### What Happens to `charge_for_current_cycle` on Device Deactivation

**File:** `Devices/Model/Table/DevicesTable.php:1280-1285` (beforeSave)

```php
$isDeactivated = $statuses[$entity->device_status_id] === 'Deactivated';
if (!$entity->charge_for_current_cycle && !$isDeactivated) {
    $entity->charge_for_current_cycle = true;
}
```

This code **does NOT set `charge_for_current_cycle = 0`** when deactivating. It only prevents non-deactivated devices from having `charge_for_current_cycle = false`. When a device is deactivated, `charge_for_current_cycle` remains at whatever value it had (typically `1`).

---

## Billing Pipeline — Two Separate Protections

### Protection 1: `waive_final_bill` on Company (NACHA only)

**File:** `Billing/Model/Table/BillingCyclesTable.php:272, 326`

```php
->innerJoinWith('Companies', function (Query $q) {
    return $q->where(['waive_final_bill' => 0]);
})
```

This **only** filters companies during NACHA file generation (ACH payment collection). It does NOT prevent invoice generation. The invoice is still created and visible — it just won't be sent to the bank.

### Protection 2: `charge_for_current_cycle` on CompanyDeviceUsages (Invoice Generation)

**File:** `Devices/Model/Table/CompanyDeviceUsagesTable.php:799-803`

```php
->where([
    'CompanyDeviceUsages.billing_cycle_id' => $billingCycleId,
    'CompanyDeviceUsages.charge_for_current_cycle' => 1,
    'CompanyDeviceUsages.waive_final_billing' => 0
])
```

Invoice generation checks BOTH `charge_for_current_cycle` and `waive_final_billing` on the **usage record**, not on the company.

### Protection 3: `CleanUpDeactivatedDevicesForCurrentCycleCommand` (Scheduled Cleanup)

**File:** `Billing/Command/CleanUpDeactivatedDevicesForCurrentCycleCommand.php`

This command is designed to clean up deactivated devices by setting `charge_for_current_cycle = 0` on both the device and its usage records. **However**, it has a critical filter:

```php
// Lines 67-82: EXCLUDE devices that were deactivated during the current cycle
$oLogExclusions = $this->OLogs->find()->where([
    'device_id IS NOT NULL',
    'OLogs.created >' => $currentBillingCycle->cycle_start,
    'OLogs.message LIKE' => '%to Deactivated%'
])->matching('OLogCategories', ...)->all()->extract('device_id')->toArray();
```

**This intentionally EXCLUDES devices deactivated in the current cycle.** The logic is: if a device was deactivated THIS cycle, it was active for part of the cycle and should still be billed.

### Protection 4: `waive_final_billing` on Device → Usage (Device-Level Waive)

**File:** `Devices/Model/Table/DevicesTable.php:2558-2581`

The `waive_final_billing` flag on a device propagates to `company_device_usages.waive_final_billing` — but ONLY when the device is **unassigned** (company_id cleared). Company deactivation does NOT unassign devices; it only changes their status to Deactivated.

---

## Bug or Expected Behavior?

### The Gap: Company `waive_final_bill` ≠ Device/Usage `waive_final_billing`

There are **two separate fields** that sound similar but work differently:

| Field | Location | Set During Company Deactivation? | Effect |
|-------|----------|----------------------------------|--------|
| `waive_final_bill` | `companies` table | Yes (checkbox in modal) | Excludes from NACHA file only |
| `waive_final_billing` | `devices` table / `company_device_usages` table | **NO** | Excludes from invoice generation |

**The company deactivation flow does NOT:**
1. Set `charge_for_current_cycle = 0` on the device
2. Set `waive_final_billing = 1` on the device
3. Set `waive_final_billing = 1` on the `company_device_usages` record
4. Set `charge_for_current_cycle = 0` on the `company_device_usages` record

**The cleanup command also won't help** because it intentionally skips devices deactivated in the current billing cycle.

### Verdict: This is a BUG

The client selected "waive final bill" during company deactivation. The system saved `waive_final_bill = 1` on the company record. However:

1. **Invoice was still generated** — because the `company_device_usages` record for device W90326 still had `charge_for_current_cycle = 1` and `waive_final_billing = 0`
2. **The company-level `waive_final_bill` only blocks NACHA** — so the invoice appears in the system and was likely visible/sent to the customer, even if payment wouldn't actually be collected via ACH

The client's expectation is reasonable: checking "waive final bill" during company deactivation should mean the company isn't billed at all — not just that the ACH payment isn't collected.

---

## Questions for Client Follow-Up

### 1. Clarify "Invoiced" — Invoice Generated vs. Payment Collected?

Was the customer actually **charged** (money collected from their bank), or did they just see an invoice in the portal/receive an invoice PDF?

- If **invoice was generated but not collected**: The NACHA protection worked, but the invoice visibility is the issue
- If **payment was actually collected**: The NACHA protection also failed, which is a more severe bug

### 2. Was This a Two-Step or One-Step Deactivation?

The audit log shows the company went to deactivated with `waive_final_bill` set in a single edit. Was there a prior `pending_deactivation` step, or did they go directly to deactivated?

> This matters because `waive_final_bill` is cleared if the status isn't `deactivated` (line 658-659). If the beforeSave redirected to `pending_deactivation`, the flag might have been cleared.

**This could be the root cause**: If the flow was:
1. Admin selects "Deactivate" + checks "Waive Final Bill"
2. beforeSave forces status to `pending_deactivation`
3. beforeSave checks: `newStatus !== 'deactivated'` → **clears `waive_final_bill`**
4. Company is saved with `pending_deactivation` and `waive_final_bill = false`

Then even the NACHA protection would fail because the flag got cleared.

### 3. Expected Behavior When "Waive Final Bill" Is Checked

Does the client expect:
- **a)** No invoice generated at all (device removed from billing entirely)
- **b)** Invoice generated but $0 amount
- **c)** Invoice generated but not collected (current NACHA-only behavior, if it worked)

### 4. Timeline Confirmation

The logs show the deactivation happened on 2/17/26. When was the billing cycle for "this month"? Was 2/17 within the current billing cycle at the time invoices were generated?

---

## Identified Issues (For Dev Team)

### Issue 1: `waive_final_bill` May Be Cleared During Two-Step Deactivation (LIKELY ROOT CAUSE)

**File:** `Companies/Model/Table/CompaniesTable.php:654-659`

```php
if ($newStatus === 'deactivated' && !in_array($previousStatus, ['pending_deactivation', 'suspended'])) {
    $pendingDeactivationId = array_flip($statusKeys)['pending_deactivation'];
    $entity->company_status_id = $pendingDeactivationId; // Changed to pending_deactivation
}
if ($newStatus !== 'deactivated' && $entity->waive_final_bill) {
    $entity->waive_final_bill = false; // THIS CHECK USES THE ORIGINAL newStatus, NOT the redirected one
}
```

Wait — actually the `$newStatus` variable was set from the original `company_status_id` BEFORE the redirect. So `$newStatus` would still be `'deactivated'` even after the redirect. **This means the flag is NOT cleared by this logic.**

BUT: the `$entity->company_status_id` was changed to `pending_deactivation`. So the entity saves as `pending_deactivation` with `waive_final_bill = true`. On the SECOND save (approval to `deactivated`), does `waive_final_bill` get re-submitted? Or does it come from the database? Need to verify this.

### Issue 2: Company `waive_final_bill` Not Propagated to Device/Usage Level

Even if `waive_final_bill` is correctly saved on the company, the company deactivation `afterSave` (lines 780-841) never:
- Sets `waive_final_billing = 1` on devices
- Sets `waive_final_billing = 1` on `company_device_usages`
- Sets `charge_for_current_cycle = 0` on devices or usages

This means invoices will still be generated for the devices.

### Issue 3: NACHA Filter Only Checks Company, Not Usage

The NACHA generation filters by `Companies.waive_final_bill = 0`, but invoice generation filters by `CompanyDeviceUsages.waive_final_billing = 0`. These are disconnected — there's no mechanism to propagate the company flag to the usage records during deactivation.

---

## Recommended Fix Direction

When a company is deactivated with `waive_final_bill = true`, the `afterSave` should also update the `company_device_usages` records for the current billing cycle to set either:
- `waive_final_billing = 1`, OR
- `charge_for_current_cycle = 0`

This would prevent invoices from being generated for devices belonging to the waived company.
