# Requirement: Distributor Single Device Minimum Change

**Date:** 2025-10-27
**Status:** Requirements Documentation
**Priority:** TBD
**Complexity:** High - Impacts core billing logic

---

## Executive Summary

Modify the billing system to implement a special minimum pricing rule for distributors' subcompanies with single devices per payment method. This change affects both the minimum charge enforcement and distributor commission calculations.

---

## Client Decision Summary (Updated 2025-10-31)

Three critical implementation questions were resolved with client input:

### 1. Negative Commission Handling ✅
**Decision:** Raise subcompany price to match distributor minimum

When the $4.25 distributor minimum would create a negative commission (distributor minimum > subcompany price), the system will automatically raise the subcompany's price to match the distributor minimum. This ensures:
- No negative commissions occur
- Distributor commission = $0.00 in these edge cases
- Pricing integrity maintained across the system

**Example:** Subcompany price $3.50 with distributor minimum $4.25 → Raise subcompany price to $4.25, commission = $0.00

### 2. Dual SIM Upcharge Application ✅
**Decision:** Apply $4.25 minimum to distributor aggregate (base + dual SIM fee)

The minimum applies to the distributor's total wholesale cost, not just the base price:
- Calculate: distributor base price + distributor's dual SIM upcharge = distributor aggregate
- If distributor aggregate < $4.25, raise to $4.25
- Then apply negative commission rule if needed (raise subcompany price if necessary)

**Example:** Distributor base $2.50 + dual SIM $1.00 = $3.50 aggregate → Raise to $4.25 minimum

### 3. Credit Card Payment Methods ✅
**Decision:** Minimum charge rules do NOT apply to credit card payments

The $4.25 distributor minimum and related pricing rules only apply to ACH/NACHA payment methods:
- Credit card payments are exempt from these minimums
- Different fee structure (percentage-based vs fixed ACH fees) justifies exemption
- Currently, subcompanies under distributors don't use credit cards, but implementation is future-proofed

---

## Current Behavior

### Current Minimum Charge Rule

**Current Logic:**
- All companies pay a minimum of $7.95 per payment method
- Exception: Companies with `is_surcharge_excluded = true` are exempt from the minimum
- This minimum applies to both direct customers and subcompanies under distributors
- Distributor commissions are calculated as: subcompany price minus distributor wholesale price

### Current Example: EZMONEY MACHINES LLC (Device X02807)

**Scenario:**
- Subcompany: EZMONEY MACHINES LLC
- Distributor: Merchant Money Services LLC (Contact: Tyler)
- Device count: 1 device per payment method
- Subcompany price: $5.99
- Distributor wholesale: $3.75

**Current Billing:**
```
1. Device base charge: $5.99
2. Minimum $7.95 applied to subcompany
3. EZMONEY pays: $7.95
4. Merchant Money Services LLC commission: $5.99 - $3.75 = $2.24
```

**Problem:**
- Subcompany pays $7.95 but distributor commission based on $5.99
- Inconsistent pricing creates customer confusion

---

## Requested Changes

### Change 1: Distributor Minimum Wholesale Price

**Rule:** When a distributor's subcompany has exactly **1 device per payment method**, enforce a **$4.25 minimum distributor wholesale price** for commission calculations.

**Conditions:**
- Company has `parent_company_id` (is a subcompany)
- Parent company has `can_upcharge = 1` (is a distributor)
- **Payment method has exactly 1 device assigned**

**Logic:**
```
IF company.parent_company_id IS NOT NULL
AND parent_company.account_type.can_upcharge == 1
AND device_count_for_payment_method == 1
AND distributor_wholesale_price < 4.25
THEN
    distributor_wholesale_price = 4.25 (for commission calculation)
```

**Business Rationale (from client):**
- Merchant Money Services LLC has ~50 subcompanies with only 1 device each
- Base price of $3.75 doesn't make sense for single-device transactions when factoring in ACH fees + carrier costs
- When payment methods have multiple devices, ACH fees are distributed, so $3.75 base is acceptable
- The $4.25 minimum adjusts the commission calculation to ensure adequate margin on single-device-per-payment-method accounts

**Important:** This is applied **per payment method**. A company with 2 devices on separate payment methods would have the $4.25 minimum applied to each payment method independently.

### Change 2: Configuration Changes

#### 2a. Hide 'Exclude Device Surcharge' for Subcompanies

**Current Behavior:**
- All companies can see and control the 'Exclude Device Surcharge' checkbox
- This checkbox controls whether the company is excluded from the $7.95 minimum charge

**New Behavior:**
- Hide this checkbox for companies that have a distributor parent
- Only show if the company has no parent OR the parent is not a distributor
- The field should not be accessible/editable by subcompanies with distributor parents

**Rationale:**
- Subcompanies should not control their own surcharge settings
- This prevents conflicts with distributor-level pricing rules
- Distributor manages pricing for all their subcompanies

#### 2b. Add Distributor Configuration Option

**New Field Required:**
- Database field: `companies.exclude_single_device_minimum`
- Field type: Boolean (default: false)
- Visibility: Distributor edit page only (companies with distributor/can_upcharge capability)
- UI Label: "Exclude minimum distro price for 1-off paying subs"
- Help Text: "When checked, the $4.25 minimum distributor wholesale price will NOT apply to payment methods with exactly 1 device"

**Purpose:**
- Allows distributors to opt-out of the $4.25 minimum rule for single-device payment methods
- Gives distributors flexibility to manage their own commission structures
- Some distributors may want to keep lower wholesale prices even for single devices

**Behavior:**
- When a distributor enables this option, the $4.25 minimum enforcement is skipped for all their subcompanies
- This affects only the specific distributor and their subcompanies, not the entire system
- The check should occur before applying the $4.25 minimum during invoice generation

### Change 3: Remove $7.95 Minimum for ALL Subcompanies

**Rule:** Paying subcompanies (companies with a parent company) should **NOT** have the $7.95 minimum charge applied.

**Current Behavior:**
- All companies (except those with `is_surcharge_excluded = true`) pay a minimum of $7.95 per payment method
- This applies regardless of whether they are direct customers or subcompanies under a distributor

**New Behavior:**
- Only direct customers (companies with NO parent) should have the $7.95 minimum applied
- All subcompanies (companies with a parent distributor) are exempt from the $7.95 minimum
- The `is_surcharge_excluded` flag check should still be respected for direct customers

**Logic:**
- Apply $7.95 minimum charge ONLY when:
  - Company does NOT have `is_surcharge_excluded = true` AND
  - Company does NOT have a parent company (is not a subcompany)

**Rationale from Client:**
> "We do NOT need to have the minimum $7.95 apply to any paying subcompanies."

---

## New Behavior Examples

### Example 1: Single Device Subcompany (Main Use Case)

**Scenario:**
- Subcompany: EZMONEY MACHINES LLC
- Distributor: Merchant Money Services LLC (Contact: Tyler)
- Device count: 1 device per payment method
- Subcompany price: $5.99
- Distributor wholesale (original): $3.75

**New Billing:**
```
1. Device base charge: $5.99
2. Distributor wholesale bumped: $3.75 → $4.25 (minimum enforcement)
3. No $7.95 minimum applied (subcompany exemption)
4. EZMONEY pays: $5.99
5. Merchant Money Services LLC commission: $5.99 - $4.25 = $1.74
```

**Changes:**
- ✅ Subcompany pays actual price ($5.99 instead of $7.95)
- ✅ Distributor commission reduced ($1.74 instead of $2.24)
- ✅ Consistent pricing (subcompany pays what's shown)

### Example 2: Multiple Devices, Separate Payment Methods

**Scenario:**
- Subcompany: ABC Company
- Distributor: XYZ Distributor
- Device A: $4.00 (Distributor wholesale: $3.00) → Payment Method 1
- Device B: $3.50 (Distributor wholesale: $2.50) → Payment Method 2
- Total devices: 2 (but 1 per payment method each)

**New Billing:**
```
Payment Method 1:
  - Original device charge: $4.00
  - Device count: 1 (qualifies for $4.25 minimum)
  - Distributor wholesale bumped: $3.00 → $4.25
  - Since subcompany price ($4.00) < distributor minimum ($4.25):
    → Subcompany price raised: $4.00 → $4.25
  - ABC pays: $4.25
  - XYZ commission: $4.25 - $4.25 = $0.00

Payment Method 2:
  - Original device charge: $3.50
  - Device count: 1 (qualifies for $4.25 minimum)
  - Distributor wholesale bumped: $2.50 → $4.25
  - Since subcompany price ($3.50) < distributor minimum ($4.25):
    → Subcompany price raised: $3.50 → $4.25
  - ABC pays: $4.25
  - XYZ commission: $4.25 - $4.25 = $0.00

Total ABC pays: $8.50
Total XYZ commission: $0.00
```

**✅ RESOLVED:** When distributor minimum exceeds subcompany price, raise the subcompany price to match the minimum. This prevents negative commissions and ensures pricing integrity.

### Example 3: Multiple Devices, Same Payment Method

**Scenario:**
- Subcompany: DEF Company
- Distributor: GHI Distributor
- Device A: $3.00 (Distributor wholesale: $2.00) → Payment Method 1
- Device B: $3.00 (Distributor wholesale: $2.00) → Payment Method 1
- Total devices: 2 on same payment method
- Total charge: $6.00

**New Billing:**
```
Payment Method 1:
  - Device A + B: $6.00 total to subcompany
  - Device count: 2 (does NOT qualify for $4.25 minimum - more than 1 device on this payment method)
  - No $7.95 minimum applied (subcompany exemption)
  - DEF pays: $6.00
  - GHI commission per device: $3.00 - $2.00 = $1.00 each
  - Total commission: $2.00
```

**✅ NO ISSUE:** The $4.25 minimum does NOT apply because this payment method has more than 1 device.

### Example 4: Subcompany with High-Priced Device

**Scenario:**
- Subcompany: JKL Company
- Distributor: MNO Distributor
- Device: $25.00 → Payment Method 1
- Distributor wholesale: $20.00

**New Billing:**
```
Payment Method 1:
  - Device charge: $25.00
  - Distributor wholesale: $20.00 (no change, above $4.25)
  - No $7.95 minimum applied (subcompany exemption)
  - JKL pays: $25.00
  - MNO commission: $25.00 - $20.00 = $5.00
```

### Example 5: Non-Subcompany (Direct Customer)

**Scenario:**
- Company: PQR Company (NO parent_company_id)
- Device: $4.00 → Payment Method 1

**New Billing:**
```
Payment Method 1:
  - Device charge: $4.00
  - $7.95 minimum STILL APPLIES (not a subcompany)
  - PQR pays: $7.95
```

**No Change:** Direct customers still pay $7.95 minimum.

---

## Edge Cases and Questions for Client

### Edge Case 1: Negative Commissions ✅ RESOLVED

**Scenario:** Subcompany price ($3.50) < Distributor minimum ($4.25)

**Question:** What happens when distributor wholesale minimum ($4.25) exceeds subcompany price?

**Client Decision:** **Option C: Raise Subcompany Price to Match Minimum**

**Implementation:**
- When $4.25 distributor minimum would exceed the subcompany's calculated price, raise the subcompany price to match the distributor minimum
- This ensures no negative commissions occur
- Distributor commission will be $0.00 in these cases
- Maintains pricing integrity across the system

**Example:**
```
Original subcompany price: $3.50
Distributor wholesale minimum: $4.25
Result: Raise subcompany price to $4.25
Commission: $4.25 - $4.25 = $0.00
```

### Edge Case 2: Credit Application ✅ APPLIED RECOMMENDATION

**Question:** If subcompany has credits, are they applied before or after the $4.25 minimum calculation?

**Decision:** Credits are applied AFTER all pricing calculations and minimums have been determined.

**Implementation:**
The order of operations for billing calculations should be:
1. Calculate base device charges
2. Apply $4.25 distributor minimum (for single-device ACH payment methods)
3. Apply subcompany price adjustment if needed (prevent negative commissions)
4. Skip $7.95 minimum for subcompanies (only apply to direct customers)
5. Apply account credits to final invoice total

**Rationale:**
- Minimums and pricing rules should be calculated on the actual service charges
- Credits are a payment/adjustment mechanism applied to the final invoice amount
- This maintains consistency with current credit application behavior

### Edge Case 3: Delayed Billing Devices ✅ APPLIED RECOMMENDATION

**Question:** Do devices with `delayed_billing = true` count toward the "1 device per payment method" threshold?

**Decision:** YES, delayed billing devices count toward the device count per payment method.

**Implementation:**
- Include all active devices in the count, regardless of their `delayed_billing` status
- If a payment method has 1 active device (even if it has delayed billing enabled), the $4.25 minimum applies
- The device count check should consider all devices associated with a payment method for that billing cycle

**Rationale:**
- Delayed billing is a billing cycle timing feature, not a device exclusion
- The device still exists on the payment method and will eventually be billed
- ACH fees are still incurred regardless of delayed billing status
- Consistent counting logic prevents edge case exploits

### Edge Case 4: Dual SIM Upcharges ✅ RESOLVED

**Question:** Does the $4.25 minimum apply to base price or total price including dual SIM fees?

**Client Decision:** Apply $4.25 minimum to **distributor aggregate (base price plus their dual fee)**

**Implementation:**
- The $4.25 minimum applies to the distributor's total wholesale cost
- Distributor wholesale = base distributor price + distributor's dual SIM upcharge
- If (base distributor price + distributor dual SIM fee) < $4.25, raise to $4.25
- Then apply the same logic as Edge Case 1 if needed (raise subcompany price if it falls below distributor minimum)

**Example:**
```
Scenario 1: Distributor aggregate below $4.25
  Base tier price to distributor: $2.50
  Distributor's dual SIM upcharge: $1.00
  Distributor aggregate: $2.50 + $1.00 = $3.50
  Apply minimum: $3.50 → $4.25

  Subcompany base price: $3.00
  Subcompany dual SIM upcharge: $2.00
  Subcompany total: $5.00

  Since distributor minimum ($4.25) < subcompany price ($5.00):
    Commission: $5.00 - $4.25 = $0.75

Scenario 2: Distributor aggregate already above $4.25
  Base tier price to distributor: $3.00
  Distributor's dual SIM upcharge: $2.00
  Distributor aggregate: $3.00 + $2.00 = $5.00 (already above $4.25)
  No adjustment needed

  Subcompany total: $7.00
  Commission: $7.00 - $5.00 = $2.00
```

### Edge Case 5: Multiple Billing Cycles ✅ APPLIED RECOMMENDATION

**Question:** If a device is transferred mid-cycle, how do we count devices for the "1 device per payment method" check?

**Decision:** Count devices per payment method at the time of invoice generation for that specific billing cycle.

**Implementation:**
- Device count should reflect the actual state at invoice generation time
- Use the billing cycle context to determine which devices are associated with which payment methods
- If a device was transferred mid-cycle, count it based on its final assignment for that billing cycle
- The check is performed per billing cycle, not across multiple cycles

**Rationale:**
- Invoice generation is a point-in-time snapshot
- Mid-cycle transfers should be reflected in the device count for that cycle's invoice
- This approach maintains consistency with how other billing calculations work
- Prevents confusion from historical device assignments

### Edge Case 6: Flat Rate and Financing Charges ✅ APPLIED RECOMMENDATION

**Question:** Do flat rate or financing charges count as "devices" for the 1-device-per-payment-method check?

**Decision:** NO, flat rate and financing charges do NOT count as devices for the single-device check.

**Implementation:**
- Only count actual service plan devices when determining if a payment method has exactly 1 device
- Flat rate charges are excluded from the device count
- Financing charges are excluded from the device count
- The device count should only include devices with active service plans that generate recurring monthly charges

**Rationale:**
- The $4.25 minimum is specifically designed for single service plan devices
- Flat rate and financing charges are one-time or special billing line items, not devices
- These charges don't incur per-device carrier costs that the $4.25 minimum is meant to cover
- Counting only service plan devices provides clear, unambiguous business logic

### Edge Case 7: Credit Card Payment Methods ✅ RESOLVED

**Question:** Do we apply the same minimum charge rules for credit card payments?

**Client Decision:** These minimum charge rules do **NOT** apply to credit card payment methods.

**Implementation:**
- The $4.25 distributor minimum only applies to ACH/NACHA payment methods
- Credit card payments are excluded from this minimum logic
- Current system: Subcompanies under distributors don't use credit card payment methods
- Future-proofing: If credit cards are enabled for distributor subcompanies in the future, skip minimum enforcement

**Business Rationale:**
- The $4.25 minimum exists to cover ACH fees + carrier costs for single-device transactions
- Credit card processing has different fee structures that don't require this minimum
- ACH fees are fixed per transaction, making the minimum necessary
- Credit card fees are percentage-based, so no fixed minimum needed

---

## Testing Requirements

### Test Case 1: Basic Single Device Subcompany
**Setup:**
- Create subcompany with parent distributor
- Assign 1 device at $5.99
- Distributor wholesale: $3.75

**Expected:**
- Distributor wholesale adjusted to $4.25
- Subcompany charged $5.99 (no $7.95 minimum)
- Distributor commission: $1.74

### Test Case 2: Multiple Devices, One Payment Method
**Setup:**
- Subcompany with 2 devices
- Both on same payment method
- Each $4.00, distributor wholesale $3.00 each

**Expected:**
- NO $4.25 minimum applied (payment method has 2 devices, not 1)
- Subcompany charged $8.00 (no $7.95 minimum)
- Distributor wholesale: $3.00 (unchanged)
- Commission per device: $4.00 - $3.00 = $1.00 each
- Total commission: $2.00

### Test Case 3: Multiple Devices, Separate Payment Methods
**Setup:**
- Subcompany with 2 devices
- Each on different payment method (both ACH)
- Each device originally priced at $3.50, distributor wholesale $2.50 each

**Expected:**
- $4.25 minimum applied to BOTH payment methods (each has 1 device)
- Distributor wholesale per payment method: $2.50 → $4.25
- Since subcompany price ($3.50) < distributor minimum ($4.25):
  → Subcompany price raised to $4.25 per device
- Subcompany charged: $4.25 × 2 = $8.50
- Distributor commission per device: $4.25 - $4.25 = $0.00 each
- Total commission: $0.00

### Test Case 4: Direct Customer (Non-Subcompany)
**Setup:**
- Company with NO parent_company_id
- 1 device at $4.00

**Expected:**
- NO $4.25 minimum (not a subcompany)
- $7.95 minimum STILL APPLIES
- Customer charged $7.95

### Test Case 5: Subcompany with is_surcharge_excluded
**Setup:**
- Subcompany with is_surcharge_excluded = true
- 1 device at $3.00
- Distributor wholesale: $2.50
- Payment method: ACH

**Expected (per recommendation):**
- $4.25 minimum SHOULD STILL APPLY (distributor wholesale: $2.50 → $4.25)
- Since subcompany price ($3.00) < distributor minimum ($4.25):
  → Subcompany price raised to $4.25
- No $7.95 minimum (flag excludes this)
- Subcompany charged: $4.25
- Commission: $4.25 - $4.25 = $0.00

**Note:** This test case applies the recommendation. Awaiting final client confirmation on is_surcharge_excluded interaction.

### Test Case 6: High-Price Single Device
**Setup:**
- Subcompany with 1 device at $25.00
- Distributor wholesale: $20.00

**Expected:**
- No $4.25 minimum (already above)
- Subcompany charged $25.00
- Distributor commission: $5.00

### Test Case 7: Credit Card Payment Method
**Setup:**
- Subcompany with 1 device at $5.99
- Distributor wholesale: $3.75
- Payment method: Credit Card
- Single device on payment method

**Expected:**
- NO $4.25 minimum applied (credit card payment method excluded)
- NO $7.95 minimum applied (subcompany exemption)
- Subcompany charged: $5.99 (plus any credit card processing fees per normal logic)
- Distributor wholesale: $3.75 (unchanged)
- Commission: $5.99 - $3.75 = $2.24

**Note:** Currently, subcompanies under distributors don't use credit card payment methods in the system. This test case is for future-proofing if that changes.

### Test Case 8: Distributor Opt-Out of $4.25 Minimum
**Setup:**
- Distributor with `exclude_single_device_minimum = true`
- Subcompany with 1 device at $5.99
- Distributor wholesale: $3.75

**Expected:**
- NO $4.25 minimum applied (distributor opted out)
- Subcompany charged $5.99 (no $7.95 minimum)
- Distributor wholesale: $3.75 (unchanged)
- Commission: $5.99 - $3.75 = $2.24

### Test Case 9: Subcompany UI - Surcharge Exclusion Hidden
**Setup:**
- Subcompany with parent distributor
- Access Companies edit page

**Expected:**
- 'Exclude Device Surcharge' checkbox NOT visible
- Other fields visible and editable normally

### Test Case 10: Distributor UI - New Configuration Visible
**Setup:**
- Distributor company with `can_upcharge = 1`
- Access Companies edit page

**Expected:**
- 'Exclude minimum distro price for 1-off paying subs' checkbox visible
- Can toggle checkbox and save successfully
- New field value persists in database

### Test Case 11: Dual SIM Device with Aggregate Below $4.25
**Setup:**
- Subcompany with 1 dual SIM device
- Payment method: ACH
- Distributor base wholesale: $2.50
- Distributor dual SIM upcharge: $1.00
- Distributor aggregate: $3.50 (below minimum)
- Subcompany base price: $3.00
- Subcompany dual SIM upcharge: $2.00
- Subcompany total: $5.00

**Expected:**
- Distributor aggregate: $3.50 → raised to $4.25
- Subcompany price ($5.00) > distributor minimum ($4.25), so no adjustment needed
- Subcompany charged: $5.00
- Distributor wholesale: $4.25
- Commission: $5.00 - $4.25 = $0.75

### Test Case 12: Dual SIM Device - Subcompany Price Adjustment Needed
**Setup:**
- Subcompany with 1 dual SIM device
- Payment method: ACH
- Distributor base wholesale: $2.00
- Distributor dual SIM upcharge: $2.50
- Distributor aggregate: $4.50 (above minimum, no adjustment)
- Subcompany base price: $2.50
- Subcompany dual SIM upcharge: $1.50
- Subcompany total: $4.00 (below distributor aggregate!)

**Expected:**
- Distributor aggregate: $4.50 (no adjustment needed, already above $4.25)
- Subcompany price ($4.00) < distributor aggregate ($4.50)
- Apply negative commission rule: Raise subcompany price to $4.50
- Subcompany charged: $4.50
- Distributor wholesale: $4.50
- Commission: $4.50 - $4.50 = $0.00

---

## Migration and Rollout Strategy

### Phase 1: Development
1. Database schema changes for distributor configuration options
2. Implement single-device-per-payment-method detection
3. Implement $4.25 distributor minimum enforcement (ACH only, with opt-out capability)
4. Remove $7.95 minimum for all subcompanies
5. Update commission calculations to reflect new minimums
6. Update Companies configuration UI for visibility and new options
7. Implement subcompany price adjustment when below distributor minimum

### Phase 2: Testing
1. Unit tests for all edge cases (see Test Cases section)
2. Integration tests with full invoice generation
3. Manual QA with real distributor scenarios (especially Merchant Money Services LLC use case)
4. Verify commission calculations across various scenarios
5. Test distributor opt-out functionality

### Phase 3: Data Analysis
1. Identify all affected distributors and subcompanies
2. Calculate financial impact on current invoices
3. Estimate commission changes for distributors
4. Prepare customer communication materials
5. Identify any distributors who may want to opt-out of the $4.25 minimum

### Phase 4: Deployment
1. Deploy to staging environment
2. Run parallel billing calculations (old vs new logic) for validation
3. Verify results with finance team
4. Communicate changes to affected distributors and subcompanies
5. Deploy to production
6. Monitor first billing cycle closely for anomalies

---

## Impact Analysis

### Financial Impact

**Affected Parties:**
1. **Subcompanies** - Pay less (no $7.95 minimum)
2. **Distributors** - Earn less commission (higher wholesale minimum)
3. **WATM** - Revenue neutral (commissions adjust)

**Example Impact (EZMONEY MACHINES LLC):**
- Subcompany saves: $7.95 - $5.99 = **$1.96 per month**
- Distributor (Merchant Money Services LLC) commission reduced: $2.24 - $1.74 = **$0.50 per month**

---

## Client Clarifications Needed

### ✅ RESOLVED - Client Responses Received

1. **Negative Commission Handling:** ✅ **CLIENT DECISION**
   - **Decision:** Raise subcompany price to match distributor minimum
   - When distributor minimum ($4.25) > subcompany price, raise subcompany price to $4.25
   - This prevents negative commissions; distributor commission will be $0.00 in these cases

2. **Credit Card Payment Methods:** ✅ **CLIENT DECISION**
   - **Decision:** Minimum charge rules do NOT apply to credit card payment methods
   - Only apply $4.25 minimum to ACH/NACHA payments
   - Credit cards exempt due to different fee structure (percentage-based vs fixed ACH fees)

3. **Dual SIM Upcharge:** ✅ **CLIENT DECISION**
   - **Decision:** Apply $4.25 minimum to distributor aggregate (base price + dual SIM fee)
   - Calculate distributor's total wholesale cost (base + their dual SIM upcharge)
   - If aggregate < $4.25, raise to $4.25
   - Then check if subcompany price needs adjustment per negative commission rule

### ✅ APPLIED RECOMMENDATIONS (Business Logic Decisions)

4. **Credit Application Order:** ✅ **APPLIED**
   - **Decision:** Credits applied after all pricing calculations and minimums
   - See Edge Case 2 for full details

5. **Delayed Billing Devices:** ✅ **APPLIED**
   - **Decision:** Delayed billing devices DO count toward device count per payment method
   - See Edge Case 3 for full details

6. **Multiple Billing Cycles:** ✅ **APPLIED**
   - **Decision:** Count devices at invoice generation time for that specific billing cycle
   - See Edge Case 5 for full details

7. **Flat Rate and Financing Charges:** ✅ **APPLIED**
   - **Decision:** These charges do NOT count as devices for the single-device check
   - Only actual service plan devices are counted
   - See Edge Case 6 for full details

### Outstanding Questions (Need Client Input)

8. **is_surcharge_excluded Interaction:**
   - Does $4.25 minimum apply if subcompany has this flag?
   - **Recommendation:** Yes, still apply the $4.25 minimum even if `is_surcharge_excluded = true`
   - The `is_surcharge_excluded` flag controls the $7.95 minimum, not distributor wholesale pricing

9. **Effective Date:**
   - Apply retroactively or only to future invoices?
   - **Recommendation:** Apply to future billing cycles only (not retroactive)
   - Retroactive changes could create confusion and financial disputes

---

## Success Criteria

✅ Single device subcompanies no longer pay $7.95 minimum
✅ Distributor wholesale never below $4.25 for single device scenarios (unless distributor opts out)
✅ Multi-device subcompanies unaffected by $4.25 rule
✅ Direct customers still pay $7.95 minimum
✅ Commission calculations accurate with new minimums
✅ No negative commissions (or handled appropriately)
✅ All edge cases tested and documented
✅ Earnings reports reflect new calculation method
✅ 'Exclude Device Surcharge' checkbox hidden for subcompanies with distributors
✅ New 'Exclude minimum distro price for 1-off paying subs' checkbox visible for distributors only
✅ Distributors can opt-out of $4.25 minimum using the new configuration

---

## Related Documentation

- Current Distributor Model: `/Documentation/Distributor_Subcompany_Pricing_Commissions_Model.md`
- Billing Caveats: `/watm/claude/apw_concepts/billing_caveats.md`
- Invoice Entity: `plugins/Billing/src/Model/Entity/Invoice.php`
- Invoice Generation: `plugins/Billing/src/Model/Table/InvoicesTable.php`

---

## Timeline Estimate (Preliminary)

- **Requirements Clarification:** 1-2 days
- **Database Migration:** 0.5 days
- **Development:** 4-6 days (includes configuration UI changes)
- **Testing:** 2-3 days
- **QA and Review:** 2 days
- **Deployment:** 1 day

**Total:** ~11-16 business days

---

## Notes

- This change significantly affects core billing logic
- Thorough testing required to avoid commission calculation errors
- Customer communication important (subcompanies will see lower bills)
- Monitor distributor feedback after rollout
- Consider grandfather clause for existing contracts if needed

---

**Document Version:** 1.4
**Last Updated:** 2025-10-31
**Changelog:**
- v1.4: Applied recommendations to all unresolved edge cases:
  - Edge Case 2 (Credit Application): Applied recommendation - credits applied after all calculations
  - Edge Case 3 (Delayed Billing Devices): Applied recommendation - delayed billing devices DO count
  - Edge Case 5 (Multiple Billing Cycles): Applied recommendation - count devices at invoice generation time
  - Edge Case 6 (Flat Rate/Financing): Applied recommendation - these do NOT count as devices
  - Updated "Client Clarifications Needed" section to separate client decisions, applied recommendations, and outstanding questions
  - Updated Test Case 5 to reflect recommendation for is_surcharge_excluded interaction
  - All edge cases now have clear decisions and implementation guidance (4 resolved by client, 4 applied from recommendations, 2 outstanding)
- v1.3: Removed technical implementation details:
  - Removed "Technical Implementation Requirements" section (code examples, function implementations, specific file locations)
  - Removed "Code Locations to Modify" subsection from Impact Analysis
  - Simplified Configuration Changes sections to focus on business requirements rather than code
  - Updated Migration and Rollout Strategy to be more process-oriented
  - Document now focuses on WHAT needs to be done rather than HOW to implement it
- v1.2: Updated with client clarification responses:
  - Resolved Edge Case 1: Raise subcompany price to match distributor minimum (prevent negative commissions)
  - Resolved Edge Case 4: Apply $4.25 minimum to distributor aggregate (base + dual SIM)
  - Added Edge Case 7: Credit card payments exempt from minimum charge rules
  - Updated test cases 3 and 7 to reflect client decisions
  - Updated examples to show price adjustment behavior
- v1.1: Added configuration changes - hide surcharge exclusion for subcompanies, add distributor opt-out option
- v1.0: Initial requirements document

**Next Review:** After outstanding questions clarified (is_surcharge_excluded interaction, effective date)
