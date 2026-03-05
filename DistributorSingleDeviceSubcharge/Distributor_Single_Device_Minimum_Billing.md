# JIRA TICKET: Distributor Single Device Minimum - Billing Rule Changes

  

## Summary

Implement $4.25 minimum distributor wholesale price for single-device-per-payment-method scenarios and remove $7.95 minimum charge for all subcompanies under distributors
  
## Description


### Overview

Modify the billing system to enforce a $4.25 minimum distributor wholesale price for payment methods with exactly one device, while removing the $7.95 minimum charge for all subcompanies. This change affects invoice generation, commission calculations, and distributor configuration options.

  

### Background

- Merchant Money Services LLC has ~50 subcompanies with only 1 device each

- Current system: Subcompanies pay $7.95 minimum but distributor commission based on lower wholesale price (e.g., $3.75)

- Base distributor wholesale of $3.75 doesn't cover ACH fees + carrier costs for single-device transactions

- When payment methods have multiple devices, ACH fees are distributed, making lower wholesale prices acceptable

- Need to adjust commission calculations to ensure adequate margins on single-device-per-payment-method accounts

  

### Business Value

- Ensures distributors receive adequate commission to cover ACH transaction fees and carrier costs for single-device accounts

- Eliminates customer confusion by removing inconsistent pricing (subcompany pays $7.95 but distributor commission based on $5.99)

- Reduces billing for subcompanies (no more $7.95 minimum)

- Provides distributors flexibility via opt-out configuration

- Maintains accurate commission calculations across all scenarios

  

### Use Case Example

**Current State:**

- Subcompany: EZMONEY MACHINES LLC (Device X02807)

- Distributor: Merchant Money Services LLC

- Device price: $5.99, Distributor wholesale: $3.75

- Result: EZMONEY pays $7.95, distributor commission = $5.99 - $3.75 = $2.24

  

**New State:**

- Same scenario

- Result: EZMONEY pays $5.99 (no minimum), distributor wholesale raised to $4.25, commission = $5.99 - $4.25 = $1.74

  

---

  

## Acceptance Criteria

  

### 1. Distributor Minimum Wholesale Price Enforcement

- [ ] When a subcompany has exactly 1 device on a payment method, enforce $4.25 minimum distributor wholesale price

- [ ] Minimum applies ONLY to ACH/NACHA payment methods (NOT credit cards)

- [ ] Minimum calculated on distributor aggregate (base price + distributor's dual SIM upcharge if applicable)

- [ ] Applied independently per payment method (company with 2 devices on separate payment methods gets minimum on each)

- [ ] Check occurs ONLY for companies with `parent_company_id` where parent has `can_upcharge = 1`

  

### 2. Subcompany Price Adjustment (Negative Commission Prevention)

- [ ] If subcompany price < distributor minimum ($4.25), automatically raise subcompany price to match distributor minimum

- [ ] This ensures no negative commissions occur

- [ ] Distributor commission will be $0.00 in these cases

- [ ] Maintains pricing integrity across the system

  

### 3. Remove $7.95 Minimum for Subcompanies

- [ ] $7.95 minimum charge NO LONGER applies to any company with `parent_company_id`

- [ ] Direct customers (no parent company) still pay $7.95 minimum

- [ ] `is_surcharge_excluded` flag still respected for direct customers

  

### 4. Configuration UI Changes

- [ ] Hide "Exclude Device Surcharge" checkbox for subcompanies with distributor parents

- [ ] Show checkbox only when: `parent_company_id IS NULL` OR parent is NOT a distributor

- [ ] Add new "Exclude minimum distro price for 1-off paying subs" checkbox for distributors only

- [ ] New checkbox visible only when `account_type.can_upcharge = 1`

- [ ] New database field: `companies.exclude_single_device_minimum` (boolean, default false)

  

### 5. Distributor Opt-Out Capability

- [ ] When distributor enables opt-out flag, $4.25 minimum is NOT enforced for their subcompanies

- [ ] Opt-out affects only that specific distributor and their subcompanies

- [ ] Check opt-out flag before applying minimum during invoice generation

  

### 6. Device Counting Logic

- [ ] Count only actual service plan devices (NOT flat rate or financing charges)

- [ ] Include delayed billing devices in count

- [ ] Count reflects device state at invoice generation time for that billing cycle

- [ ] Check devices per payment method (not total company devices)

  

### 7. Order of Operations

- [ ] Calculate base device charges

- [ ] Apply $4.25 distributor minimum (for single-device ACH payment methods)

- [ ] Apply subcompany price adjustment if needed (prevent negative commissions)

- [ ] Skip $7.95 minimum for subcompanies (only apply to direct customers)

- [ ] Apply account credits to final invoice total

  

---

  

## Implementation Details

  

### Key Business Rules

  

#### Rule 1: Single Device Detection

A payment method qualifies for $4.25 minimum when ALL conditions met:

1. Company has `parent_company_id` (is a subcompany)

2. Parent company has `account_type.can_upcharge = 1` (is a distributor)

3. Parent company has `exclude_single_device_minimum = 0` (distributor has NOT opted out)

4. Payment method type is ACH/NACHA (NOT credit card)

5. Payment method has exactly 1 service plan device for that billing cycle

  

#### Rule 2: Distributor Minimum Calculation

- Calculate distributor aggregate = base distributor price + distributor's dual SIM upcharge (if applicable)

- If distributor aggregate < $4.25, raise distributor aggregate to $4.25

- This is the distributor wholesale price used for commission calculations

  

#### Rule 3: Subcompany Price Adjustment

- Calculate subcompany price = base price + subcompany upcharge + dual SIM fee (if applicable)

- If subcompany price < distributor minimum ($4.25 from Rule 2), raise subcompany price to match distributor minimum

- This prevents negative commissions

  

#### Rule 4: Payment Method Exclusions

Credit card payment methods are completely exempt from these rules:

- No $4.25 minimum applied

- No subcompany $7.95 minimum removal (current behavior maintained)

- Rationale: Credit card fees are percentage-based, ACH fees are fixed per transaction

  

### Edge Cases & Decisions

  

**Edge Case 1: Negative Commissions** RESOLVED

- Decision: Raise subcompany price to match distributor minimum

- Prevents negative commissions, commission = $0.00 in these cases

  

**Edge Case 2: Credit Application** APPLIED

- Decision: Credits applied AFTER all pricing calculations and minimums

  

**Edge Case 3: Delayed Billing Devices** APPLIED

- Decision: YES, delayed billing devices count toward device count

  

**Edge Case 4: Dual SIM Upcharges** RESOLVED

- Decision: Apply $4.25 minimum to distributor aggregate (base + dual SIM)

  

**Edge Case 5: Multiple Billing Cycles** APPLIED

- Decision: Count devices at invoice generation time for that specific billing cycle

  

**Edge Case 6: Flat Rate/Financing** APPLIED

- Decision: NO, these do NOT count as devices

  

**Edge Case 7: Credit Card Payments** RESOLVED

- Decision: Credit card payments exempt from minimum charge rules

  

**Edge Case 8: is_surcharge_excluded** 🔶 RECOMMENDATION

- Recommendation: $4.25 minimum still applies even if subcompany has this flag

- Flag controls $7.95 minimum, not distributor wholesale pricing

  

**Edge Case 9: Effective Date** 🔶 RECOMMENDATION

- Recommendation: Apply to future billing cycles only (not retroactive)

  

---

  

## Database Changes

  

### New Field Required

**Table:** `companies`

**Field:** `exclude_single_device_minimum`

**Type:** BOOLEAN

**Default:** false

**Description:** When true, distributor opts out of $4.25 minimum for single-device payment methods

  

### Migration Required

- Add `exclude_single_device_minimum` column to `companies` table

- Default all existing records to `false`

- Add appropriate indexes if needed for query performance

  

---

  

## Testing Requirements

  

### Test Case 1: Basic Single Device Subcompany

**Setup:**

- Subcompany with parent distributor

- 1 device at $5.99 on ACH payment method

- Distributor wholesale: $3.75

  

**Expected:**

- Distributor wholesale adjusted to $4.25

- Subcompany charged $5.99 (no $7.95 minimum)

- Distributor commission: $1.74

  

### Test Case 2: Multiple Devices, One Payment Method

**Setup:**

- Subcompany with 2 devices on same ACH payment method

- Each $4.00, distributor wholesale $3.00 each

  

**Expected:**

- NO $4.25 minimum applied (payment method has 2 devices)

- Subcompany charged $8.00 (no $7.95 minimum)

- Commission: $1.00 per device × 2 = $2.00

  

### Test Case 3: Multiple Devices, Separate Payment Methods

**Setup:**

- Subcompany with 2 devices on different ACH payment methods

- Each device $3.50, distributor wholesale $2.50 each

  

**Expected:**

- $4.25 minimum applied to BOTH payment methods

- Since subcompany price ($3.50) < minimum ($4.25): raise to $4.25 per device

- Subcompany charged: $4.25 × 2 = $8.50

- Commission per device: $0.00 × 2 = $0.00

  

### Test Case 4: Direct Customer (Non-Subcompany)

**Setup:**

- Company with NO parent_company_id

- 1 device at $4.00

  

**Expected:**

- NO $4.25 minimum (not a subcompany)

- $7.95 minimum STILL APPLIES

- Customer charged $7.95

  

### Test Case 5: Subcompany with is_surcharge_excluded 🔶

**Setup:**

- Subcompany with `is_surcharge_excluded = true`

- 1 device at $3.00 on ACH payment method

- Distributor wholesale: $2.50

  

**Expected (per recommendation):**

- $4.25 minimum SHOULD STILL APPLY

- Subcompany price raised from $3.00 to $4.25

- No $7.95 minimum (flag excludes this)

- Commission: $0.00

  

### Test Case 6: High-Price Single Device

**Setup:**

- Subcompany with 1 device at $25.00

- Distributor wholesale: $20.00

  

**Expected:**

- No $4.25 minimum (already above)

- Subcompany charged $25.00

- Commission: $5.00

  

### Test Case 7: Credit Card Payment Method

**Setup:**

- Subcompany with 1 device at $5.99 on credit card

- Distributor wholesale: $3.75

  

**Expected:**

- NO $4.25 minimum applied (credit card exempt)

- NO $7.95 minimum applied (subcompany exemption)

- Subcompany charged: $5.99 (+ credit card processing fees per normal logic)

- Commission: $2.24

  

### Test Case 8: Distributor Opt-Out

**Setup:**

- Distributor with `exclude_single_device_minimum = true`

- Subcompany with 1 device at $5.99

- Distributor wholesale: $3.75

  

**Expected:**

- NO $4.25 minimum applied (distributor opted out)

- Subcompany charged $5.99 (no $7.95 minimum)

- Commission: $2.24

  

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

- Subcompany with 1 dual SIM device on ACH

- Distributor base: $2.50, dual SIM upcharge: $1.00 (aggregate: $3.50)

- Subcompany total: $5.00

  

**Expected:**

- Distributor aggregate: $3.50 → raised to $4.25

- Subcompany charged: $5.00 (no adjustment needed)

- Commission: $0.75

  

### Test Case 12: Dual SIM Device - Subcompany Price Adjustment

**Setup:**

- Subcompany with 1 dual SIM device on ACH

- Distributor aggregate: $4.50 (already above minimum)

- Subcompany total: $4.00 (below distributor aggregate)

  

**Expected:**

- Distributor aggregate: $4.50 (no adjustment)

- Subcompany price raised from $4.00 to $4.50

- Commission: $0.00

  

---

