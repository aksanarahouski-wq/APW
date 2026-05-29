# ATM Service Plan Requirements

**Document Date:** 2025-11-10
**Source:** Service Plan Change Meeting Notes

## Overview

The ATM service plan needs to be restructured to support carrier and device model-based pricing differentiation, replacing the current single global pricing model.

---

## 1. Carrier + Model-Based Pricing Structure

### Current State
- Single global price: **$4.95** for all devices

### Required State
Different prices for each carrier/model combination:

| Model | Carrier | Price |
|-------|---------|-------|
| i-22 | Verizon | $4.95 |
| i-22 | AT&T | $4.95 |
| i-22 | T-Mobile | $3.50 |
| Origin | Verizon | $4.00 (example) |
| Origin | T-Mobile | Not supported initially |

**Note:** Origin models will only support T-Mobile carrier initially (no Verizon/AT&T options for Origin)

---

## 2. Two Additional Pricing Dimensions

Add two new layers to the service plan structure:

1. **Model** - Device model (i-22, Origin, etc.)
2. **Carrier** - Network carrier (Verizon, AT&T, T-Mobile)

These dimensions replace or extend the current usage-limit-only attribute system.

---

## 3. Custom Service Plan Support

### Requirement
Any custom ATM plans created by distributors or customers must also support the same carrier/model granularity.

### Example Scenario
If distributor Tyler has a base discount bringing $4.95 → $3.75:
- He should be able to set different custom prices for:
  - T-Mobile i-22: Custom price (e.g., $3.50 → adjusted to Tyler's discount)
  - Verizon i-22: Custom price (e.g., $4.95 → adjusted to Tyler's discount)
  - AT&T i-22: Custom price (e.g., $4.95 → adjusted to Tyler's discount)

### Challenge
- Many distributors have existing custom discounts (Tyler: $3.75, Chord Base: $3.50)
- Regular customers also have custom ATM plans
- Need migration strategy for all existing custom plans

---

## 4. Backwards Compatibility Approach

### Critical Requirement
Make changes backwards compatible to avoid breaking existing plans:

- **Keep default pricing** that doesn't depend on carrier/model
- **Allow optional addition** of carrier/model-specific pricing
- **Nothing breaks** until the extra attributes are explicitly added
- Existing service plans continue to work as-is

### Benefits
- Avoids immediate requirement to update all existing custom service plans
- Provides flexibility for phased rollout
- Reduces risk of data corruption or orphaned records

---

## 5. Prerequisites - Service Plan ID Bug Fix

### Critical Blocker
**Must fix the service plan ID bug before implementing carrier/model pricing changes**

### Current Bug Behavior
- Editing a base service plan creates a **new service plan ID**
- All child custom service plans become **orphaned** (no longer associated with parent)
- Breaks all custom pricing for distributors and customers
- Requires manual database patching to fix

### Previous Impact
- Occurred when adding micro-tiers to tier 3 service plan
- Created hours of manual database repair work
- Cannot risk this happening again with carrier/model changes

### Resolution Required
Service plan edits (adding attributes, price tiers, etc.) should:
- **NOT create new service plan IDs**
- Maintain parent-child relationships with custom plans
- Allow safe editing without orphaning child records

---

## 6. Migration Strategy (Decision Required)

### Questions to Answer
1. How to handle existing custom ATM plans when carrier/model pricing goes live?
2. Should changes auto-apply to all custom plans day one?
3. Or should each distributor/customer plan be manually adjusted?

### Current State
- Dozens of distributors with custom ATM pricing
- Multiple regular customers with custom ATM plans
- Most devices are i-22 on Verizon/AT&T
- Very few T-Mobile devices currently exist in system

### Options Under Consideration
- **Option A:** Auto-apply carrier/model structure with default pricing matching current discounts
- **Option B:** Manually adjust each custom plan individually
- **Option C:** Phased approach - apply to new plans only, migrate existing plans on schedule

---

## 7. Technical Implementation Notes

### Scope
- **Only ATM service plan** needs this feature initially
- Other service plans (tier-based plans) remain unchanged

### Current Device Distribution
- Majority: i-22 devices on Verizon/AT&T networks
- Minimal: T-Mobile i-22 devices (small double-digit number)
- None: Origin devices (new product line)

### Implementation Approach
- Changes should be made to **price tiers** structure
- Avoid editing the base service plan definition directly
- Prevent service plan ID changes that break parent-child relationships

### UI/UX Considerations
- When creating service plans, allow selection of which carriers to support
- Add model + carrier selection to price tier configuration
- Mirror structure in both base and custom service plan interfaces
- Ensure distributors can see and adjust pricing for each carrier/model combination

---

## Related Requirements

### Single Device Minimum Charge ($4.25)
While not directly part of the carrier/model pricing structure, this related requirement impacts ATM service plans:

- All single-device customers must be charged minimum $4.25 (ACH processing cost)
- Applies regardless of custom pricing below this threshold
- Requires messaging on "Managed Customer Service Plan" page to inform distributors
- Impacts commission calculations for distributors with heavy discounts

---

## Action Items

### Development Team
1. Estimate cost/effort to fix the service plan ID bug
2. Design backwards-compatible approach for carrier/model pricing
3. Implement optional carrier/model attributes for service plans
4. Update custom service plan UI to support granular pricing

### Business Team
1. Decide migration strategy for existing custom ATM plans
2. Pull list of all distributors and customers with custom ATM plans
3. Determine if changes should auto-apply or require manual adjustment
4. Define communication plan for distributors about pricing changes

### Timeline
- Next billing cycle: ~1 week away (Tuesday/Wednesday)
- Follow-up meeting: Next week at regular check-in time
- Both teams to come prepared with solutions and decisions

---

## Questions for Resolution

1. Should carrier/model pricing be available only for ATM plan, or build as generic feature for all service plans?
2. How to communicate pricing changes to existing distributors with custom plans?
3. What happens if a device switches carriers (e.g., T-Mobile → Verizon)?
4. Should there be validation preventing prices below operational minimums ($4.25 for single device)?
5. How to handle devices on carriers not yet configured in the pricing matrix?
