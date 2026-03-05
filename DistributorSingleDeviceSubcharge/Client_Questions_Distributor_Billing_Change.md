# Client Clarification Questions: Distributor Single Device Billing Change

**Date:** 2025-10-27
**Re:** Implementation Questions for Single Device Minimum Change
**Priority:** High - Blocking Development

---

## Request Summary

We're implementing the requested changes to distributor billing:

✅ **Change 1:** Enforce $4.25 minimum distributor wholesale price for payment methods with exactly 1 device
✅ **Change 2:** Remove $7.95 minimum charge for all subcompanies

**Example Use Case:**
- Subcompany: EZMONEY MACHINES LLC (Device X02807) - **1 device per payment method**
- Distributor: Merchant Money Services LLC (Contact: Tyler)
- Current: Charge EZMONEY $7.95, commission based on $5.99 - $3.75 = $2.24
- New: Charge EZMONEY $5.99, commission based on $5.99 - $4.25 = $1.74

**Business Rationale:**
- Merchant Money Services LLC has ~50 subcompanies with only 1 device each
- Base price of $3.75 doesn't cover ACH fees + carrier costs for single-device transactions
- When payment methods have multiple devices, ACH fees are distributed, so $3.75 base is acceptable
- The $4.25 minimum ensures adequate margin on single-device-per-payment-method accounts

**Key Rule:** Applied **per payment method**. A company with 2 devices on separate payment methods would have the $4.25 minimum applied to EACH payment method independently.

Before we proceed with development, we need clarification on **5 critical scenarios** that could significantly impact the implementation.

---

## Critical Questions Requiring Client Decision

### Question 1: Negative Commission Scenario ⚠️ HIGHEST PRIORITY

**Scenario:**
- Subcompany has 1 device
- Subcompany price: $3.50
- Current distributor wholesale: $2.50
- New distributor minimum: $4.25

**The Problem:** Distributor minimum ($4.25) exceeds subcompany price ($3.50)

**Commission Calculation:** $3.50 - $4.25 = **-$0.75** (negative!)

**Question:** How should we handle this situation?

**Option A: Allow Negative Commission**
- Distributor would owe money back
- Rarely makes business sense
- ❌ **Not Recommended**

**Option B: Cap Commission at $0.00**
- Distributor earns nothing
- Subcompany still pays $3.50
- Protects distributor from losses
- ⚠️ **Possible, but may not solve pricing issue**

**Option C: Raise Subcompany Price to Match Minimum**
- Subcompany price: $3.50 → $4.25
- Distributor commission: $4.25 - $4.25 = $0.00
- Ensures no negative values
- ✅ **Recommended - maintains pricing integrity**

**Option D: Don't Apply $4.25 Minimum When It Creates Negative Commission**
- Only enforce minimum if it results in positive commission
- Most flexible option
- May not achieve intended pricing goals
- ⚠️ **Possible, but defeats purpose of minimum**

**❓ CLIENT DECISION NEEDED:** Which option should we implement?

**Our Recommendation:** Option C (raise subcompany price) or clarify that this scenario won't occur in practice.

---

### Question 2: Device Count Per Payment Method Definition

**Scenario:** Determining if a payment method has "exactly 1 device" for the rule.

**What counts as a "device" for the per-payment-method calculation?**

**A. Service Plan Devices (Standard Active Devices)**
- Devices with active service plan charges on this payment method
- ✅ **We assume YES - these count as devices**

**B. Delayed Billing Devices**
- Devices marked with `delayed_billing = true` (charged $0.00 temporarily) on this payment method
- ❓ **Do these count toward the payment method's device count?**

**C. Devices with Flat Rate Only**
- Devices on payment method with flat rate charges (no service plan)
- ❓ **Do these count as devices?**

**D. Devices with Financing Only**
- Devices on payment method with only financing charges
- ❓ **Do these count as devices?**

**Example Scenario:**
```
Payment Method 1 (Bank Account):
  - 1 active service plan device: $5.99
  - 1 delayed billing device: $0.00 (temporary)

Does this payment method have "1 device" or "2 devices"?
Should $4.25 minimum apply to this payment method?
```

**❓ CLIENT DECISION NEEDED:** What counts toward the device count per payment method?

**Our Recommendation:** Count only active service plan devices from `company_device_usages` table per payment method. Exclude delayed billing, flat rate only, and financing only.

---

### Question 3: Dual SIM Upcharge Interaction

**Scenario:**
- Subcompany has 1 dual SIM device
- Base tier price: $3.00
- Dual SIM upcharge: $2.00
- Total device charge: $5.00
- Distributor wholesale (base): $2.50

**Question:** Does the $4.25 minimum apply to:

**Option A: Base Price Only**
- Apply minimum to base tier: $3.00 → check if distributor wholesale needs adjustment
- Dual SIM fee added after
- Distributor base wholesale: $2.50 → $4.25
- Final price: $4.25 (base) + $2.00 (dual SIM) = $6.25

**Option B: Total Price (Base + Dual SIM)**
- Apply minimum to total: $5.00 (already above $4.25)
- Distributor wholesale: $2.50 (no change needed)
- Final price: $5.00

**Impact on Commission:**
```
Option A:
  - Subcompany pays: $6.25
  - Distributor commission: $6.25 - ($4.25 + dual SIM parent) = varies

Option B:
  - Subcompany pays: $5.00
  - Distributor commission: $5.00 - ($2.50 + dual SIM parent) = varies
```

**❓ CLIENT DECISION NEEDED:** Apply $4.25 minimum to base price or total including dual SIM?

**Our Recommendation:** Apply to base price only (Option A) - consistent with tiered pricing structure.

---

### Question 4: Dual SIM Upcharge Interaction

**Scenario:**
- Subcompany has 1 dual SIM device
- Base tier price: $3.00
- Dual SIM upcharge: $2.00
- Total device charge: $5.00
- Distributor wholesale (base): $2.50

**Question:** Does the $4.25 minimum apply to:

**Option A: Base Price Only**
- Apply minimum to base tier: $3.00 → check if distributor wholesale needs adjustment
- Dual SIM fee added after
- Distributor base wholesale: $2.50 → $4.25
- Final price: $4.25 (base) + $2.00 (dual SIM) = $6.25

**Option B: Total Price (Base + Dual SIM)**
- Apply minimum to total: $5.00 (already above $4.25)
- Distributor wholesale: $2.50 (no change needed)
- Final price: $5.00

**Impact on Commission:**
```
Option A:
  - Subcompany pays: $6.25
  - Distributor commission: $6.25 - ($4.25 + dual SIM parent) = varies

Option B:
  - Subcompany pays: $5.00
  - Distributor commission: $5.00 - ($2.50 + dual SIM parent) = varies
```

**❓ CLIENT DECISION NEEDED:** Apply $4.25 minimum to base price or total including dual SIM?

**Our Recommendation:** Apply to base price only (Option A) - consistent with tiered pricing structure.

---

### Question 4: Single Device Per Payment Method on Credit Card

**Scenario:**
- Payment method: Credit Card (3% processing fee)
- 1 device on this payment method
- Device price: $3.50
- Distributor wholesale: $2.50

**With New Rule:**
```
Base charge: $3.50
Credit card fee (3%): +$0.11
Total subcompany pays: $3.61

Distributor minimum: $4.25

Options for handling:
A) Commission: $3.50 - $4.25 = -$0.75 (negative before CC fee)
B) Commission: $3.61 - $4.25 = -$0.64 (negative after CC fee)
C) Adjust price up: Subcompany pays $4.25 + 3% = $4.38
```

**Question:** When a payment method with 1 device uses credit card:
- Is commission calculated on base price or total with CC fee?
- Should we adjust pricing to avoid negative commissions?

**❓ CLIENT DECISION NEEDED:** How to handle credit card processing fees in commission calculation?

**Our Recommendation:** Option C - Raise subcompany price to match $4.25 minimum (same as Question 1 recommendation), then add 3% CC fee.

---

**Scenario:**
- Subcompany has `is_surcharge_excluded = true` flag
- Payment method has 1 device at $3.00
- Distributor wholesale: $2.50

**Current Behavior:** This flag exempts company from $7.95 minimum

**Question:** Should the $4.25 distributor minimum still apply to payment methods with 1 device?

**Option A: YES - Apply $4.25 Minimum**
- `is_surcharge_excluded` only affects subcompany minimums ($7.95)
- Distributor minimum ($4.25) is separate business rule for commission calculation
- Distributor wholesale: $2.50 → $4.25

**Option B: NO - Respect Exclusion Flag**
- `is_surcharge_excluded` means NO minimum charges of any kind
- Distributor wholesale: $2.50 (unchanged)

**❓ CLIENT DECISION NEEDED:** Does `is_surcharge_excluded` also exclude the $4.25 distributor minimum?

**Our Recommendation:** Option A (still apply $4.25) - `is_surcharge_excluded` affects customer-facing charges, while $4.25 is a distributor commission calculation rule.

---

## Summary of Decisions Needed

| # | Question | Priority | Blocking? |
|---|----------|----------|-----------|
| 1 | Negative commission handling | 🔴 Critical | ✅ Yes |
| 2 | Device count per payment method definition | 🟡 High | ✅ Yes |
| 3 | Dual SIM upcharge interaction | 🟡 High | ⚠️ Partial |
| 4 | Single device per payment method + credit card | 🟡 High | ⚠️ Partial |
| 5 | is_surcharge_excluded interaction | 🟢 Medium | No |

---

## Next Steps

### Option 1: Email Response
Please reply to this document with your decisions for each question (1-6).

### Option 2: Meeting
We can schedule a 30-minute call to walk through these scenarios and make decisions together.

### Option 3: Assumptions
If you'd like us to proceed with our recommended options, please confirm and we'll begin development:

**Our Recommended Defaults:**
1. ✅ Raise subcompany price to match $4.25 minimum (no negative commissions)
2. ✅ Count only active service plan devices for "1 device per payment method" check
3. ✅ Apply $4.25 to base price (before dual SIM)
4. ✅ Raise payment method price to $4.25 + 3% CC fee when needed
5. ✅ Apply $4.25 minimum even if is_surcharge_excluded = true (separate rules)

---

## Timeline Impact

- ⏱️ **With clarifications today:** Development starts immediately → 10-15 days
- ⏱️ **With clarifications this week:** Development starts this week → 12-17 days
- ⏱️ **With assumptions approved:** Development starts immediately → 10-15 days + potential rework if assumptions wrong

---

## Contact Information

**For Questions:**
- [Your Name]
- [Your Email]
- [Your Phone]

**Response Deadline:** [Date] to maintain timeline

---

**Thank you for your time in clarifying these scenarios. Getting these details right will ensure the billing changes work exactly as intended for your distributor relationships.**
