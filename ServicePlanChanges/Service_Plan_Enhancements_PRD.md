# Product Requirements Document (PRD)
## Service Plan Carrier and Model Pricing Enhancements

**Document Version:** 1.0
**Date:** November 12, 2025
**Author:** Product Team
**Status:** Draft for Review

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Background and Problem Statement](#background-and-problem-statement)
3. [Goals and Objectives](#goals-and-objectives)
4. [Target Users](#target-users)
5. [User Stories](#user-stories)
6. [Scope](#scope)
7. [Functional Requirements](#functional-requirements)
8. [Testing Requirements](#testing-requirements)
9. [Dependencies and Risks](#dependencies-and-risks)

---

## Executive Summary

WATM requires the ability to set different pricing for service plans based on device model and carrier combinations. This enhancement enables flexible pricing strategies for new device lines (e.g., Origin devices on T-Mobile at $3.50 vs. i-22 devices on Verizon/AT&T at $4.95) while maintaining backwards compatibility with existing service plans and custom pricing arrangements.

### Key Features
- **Optional pricing overrides** for specific carrier and device model combinations
- **Backwards compatible** implementation that preserves existing service plans
- **Applies to all service plans** (future-proofed, not ATM-only)
- **Custom service plan support** allowing distributors to set granular pricing
- **Critical bug fix** for service plan ID changes that orphan custom plans

### Business Impact
- Enables competitive pricing for T-Mobile devices
- Supports new Origin device line launch
- Future-proofs pricing model for additional carriers/device types
- Maintains distributor profit margins with transparent pricing
- Prevents data corruption from service plan edits

---

## Background and Problem Statement

### Current State

The WATM system currently uses a single base price per service plan (e.g., ATM plan = $4.95 for all devices). This flat pricing model cannot accommodate:

1. **Different carrier costs**: T-Mobile service costs less than Verizon/AT&T
2. **Device-specific pricing**: New Origin devices have different economics than i-22 devices
3. **Market competitive pricing**: Need to offer $3.50 pricing for T-Mobile devices vs. $4.95 for others

### Problems

**Problem 1: Inflexible Pricing Model**
- Cannot price T-Mobile i-22 devices at $3.50 while maintaining Verizon/AT&T pricing at $4.95
- New Origin device line requires different pricing structure
- Cannot differentiate pricing by carrier within same service plan

**Problem 2: Critical Service Plan ID Bug**
- Editing a base service plan creates a new service plan ID
- All custom service plans become "orphaned" (lose parent relationship)
- Breaks custom pricing for all distributors and customers
- Requires manual database patching to repair
- Occurred previously when adding micro-tiers to Tier 3 plan

**Problem 3: Impact on Distributor Economics**
- Distributors with $3.75 pricing cannot profitably resell $4.95-based services
- No visibility into carrier/model-specific costs
- Custom pricing must be manually adjusted for new device/carrier combinations

---

## Goals and Objectives

### Primary Goals

1. **Enable granular pricing** for carrier and device model combinations
2. **Maintain backwards compatibility** with existing service plans and custom pricing
3. **Fix service plan ID bug** to allow safe editing of base service plans
4. **Support new product launches** (T-Mobile devices, Origin line)
5. **Preserve distributor economics** with transparent, adjustable pricing

### Success Criteria

- ✅ Admins can set different prices per carrier/model combination for any service plan
- ✅ Default pricing applies when no override is defined (backwards compatible)
- ✅ Editing base service plans does NOT create new IDs or orphan custom plans
- ✅ Distributors can customize pricing for each carrier/model combination
- ✅ Billing system correctly applies carrier/model-specific pricing
- ✅ All existing custom service plans continue to work without modification
- ✅ Zero downtime migration with no impact to current billing

### Non-Goals (Out of Scope)

- ❌ Automatic migration/repricing of existing custom service plans
- ❌ Reporting interface for listing all custom plans per base plan
- ❌ Automatic price adjustments based on wholesale cost changes
- ❌ Multi-currency support
- ❌ Time-based pricing (promotional pricing, seasonal rates)
- ❌ Volume-based carrier/model pricing discounts

---

## Target Users

### Primary Users

**1. WATM Administrators**
- **Role**: Create and manage base service plans
- **Need**: Set different prices for carrier/model combinations
- **Pain Point**: Cannot differentiate pricing by device type or carrier
- **Benefit**: Flexible pricing strategy aligned with business costs

**2. Distributors (Resellers)**
- **Role**: Create custom service plans for their sub-customers
- **Need**: Understand their cost per device/carrier and set profitable markups
- **Pain Point**: Lack visibility into carrier/model-specific costs
- **Benefit**: Transparent pricing enables accurate profit margin calculations

**3. WATM Finance/Billing Team**
- **Role**: Manage pricing strategy and billing operations
- **Need**: Accurate, auditable pricing per device/carrier
- **Pain Point**: Flat pricing doesn't reflect actual carrier costs
- **Benefit**: Pricing aligned with costs improves margin management

### Secondary Users

**4. Technical Operations Team**
- **Role**: Maintain system health and data integrity
- **Need**: Safe service plan edits without data corruption
- **Pain Point**: Service plan edits create orphaned custom plans
- **Benefit**: Reduced manual database repair work

**5. Customer Support Team**
- **Role**: Answer customer questions about pricing
- **Need**: Understand why prices differ by device/carrier
- **Pain Point**: Cannot explain pricing variations
- **Benefit**: Clear pricing structure improves support quality

---

## User Stories

### Epic 1: Base Service Plan Management

**US-1.1: Create Service Plan with Carrier/Model Pricing**
```
As a WATM Administrator
I want to create a service plan with optional carrier and model pricing overrides
So that I can set different prices based on device and carrier combinations

Acceptance Criteria:
- Can create service plan with default pricing (existing functionality)
- Can optionally add carrier + model pricing overrides
- Can select from available device models (i-22, Origin, etc.)
- Can select from available carriers (Verizon, AT&T, T-Mobile)
- Can set custom price for each model/carrier combination
- Can add multiple carrier/model combinations
- Can save without any overrides (backwards compatible)
- Default price applies when no override matches
```

**US-1.2: Edit Service Plan Without Breaking Custom Plans**
```
As a WATM Administrator
I want to edit a base service plan without creating a new service plan ID
So that existing custom service plans remain associated with the parent plan

Acceptance Criteria:
- Editing base service plan retains original service plan ID
- All custom service plans remain linked to parent
- No orphaned custom plans after edit
- Audit log tracks what was changed
- Approval workflow still functions correctly
```

**US-1.3: View Service Plan with Carrier/Model Pricing**
```
As a WATM Administrator or Distributor
I want to view a service plan and see all carrier/model pricing overrides
So that I understand the complete pricing structure

Acceptance Criteria:
- View page shows default price clearly labeled
- View page shows all carrier/model overrides in organized list
- Format: "Model + Carrier / Price" (e.g., "I-22 + T-Mobile / 3.50")
- Easy to distinguish between default and override pricing
- Can see when overrides were added/modified
```

**US-1.4: Delete Carrier/Model Pricing Override**
```
As a WATM Administrator
I want to remove a carrier/model pricing override
So that those device/carrier combinations revert to default pricing

Acceptance Criteria:
- Can delete individual carrier/model override
- Deletion removes override (devices revert to default price)
- Confirmation prompt before deletion
- Cannot delete if custom plans depend on it (warning)
```

### Epic 2: Custom Service Plan Management

**US-2.1: Create Custom Plan with Carrier/Model Pricing**
```
As a WATM Administrator creating distributor pricing
I want to set custom prices for each carrier/model combination
So that distributors receive appropriate discounts per device type

Acceptance Criteria:
- Custom plan form shows base service plan carrier/model structure
- For each carrier/model override in base plan, can set custom price
- Shows "Base Price" vs "Adjusted Price" for each override
- Can apply bulk discount to all prices (increase/decrease by amount or %)
- Default price adjustment applies to combinations without overrides
- Can save with partial overrides (some combinations use default adjustment)
```

**US-2.2: Edit Custom Plan Pricing**
```
As a WATM Administrator or Distributor
I want to modify custom pricing for specific carrier/model combinations
So that I can adjust margins for different device types

Acceptance Criteria:
- Edit form shows existing custom prices per carrier/model
- Can modify individual carrier/model prices
- Can modify default price adjustment
- Shows base price for reference when editing
- Validates prices (minimum thresholds, valid numbers)
```

**US-2.3: View Custom Plan with Granular Pricing**
```
As a Distributor
I want to see my custom pricing for each carrier/model combination
So that I can set appropriate markups for my sub-customers

Acceptance Criteria:
- View clearly shows default custom price
- View shows all carrier/model override prices
- Format matches base plan view for consistency
- Can easily identify which combinations have custom pricing
- Shows relationship to base service plan
```

**US-2.4: Understand Pricing Inheritance**
```
As a Distributor viewing custom service plan
I want to understand how my pricing relates to base plan pricing
So that I can explain costs to my sub-customers

Acceptance Criteria:
- View shows base plan name and default price
- For carrier/model overrides, shows both base and custom price
- Clear visual distinction between inherited and customized pricing
- Tooltip or help text explains pricing hierarchy
```

### Epic 3: Billing and Pricing Application

**US-3.1: Apply Carrier/Model Pricing in Billing**
```
As the Billing System
I want to apply the correct price based on device model and carrier
So that customers are charged accurately

Acceptance Criteria:
- Billing checks device model and carrier
- If override exists for model/carrier combo, use override price
- If no override exists, use default service plan price
- For custom plans, check custom override first, then custom default, then base
- Billing records show which price was applied and why
- Audit trail for pricing decisions
```

**US-3.2: Handle Missing Carrier/Model Pricing**
```
As the Billing System
I want to gracefully handle devices with no specific pricing override
So that billing doesn't fail for edge cases

Acceptance Criteria:
- If device model/carrier not in overrides, use default price
- Log warning if unusual combination encountered
- No billing failures due to missing overrides
- Support team notified of unexpected combinations
```

### Epic 4: Migration and Backwards Compatibility

**US-4.1: Preserve Existing Service Plans**
```
As a WATM Administrator
I want all existing service plans to continue working after deployment
So that current customers experience no disruption

Acceptance Criteria:
- All existing service plans function identically pre/post deployment
- No price changes unless explicitly configured
- No data migration required for existing plans
- Custom plans maintain current pricing
- Distributors see no changes unless base plan updated
```

**US-4.2: Gradual Adoption of New Pricing**
```
As a WATM Administrator
I want to add carrier/model pricing only to specific service plans
So that I can roll out changes incrementally

Acceptance Criteria:
- Can add overrides to ATM plan without affecting Tier plans
- Other service plans unaffected by changes to one plan
- Can test pricing on subset of devices/customers
- Can revert changes by removing overrides
```

---

## Scope

### In Scope

**Service Plan Management Pages**
- ✅ Create Base Service Plan page (add carrier/model pricing section)
- ✅ Edit Base Service Plan page (add carrier/model pricing section)
- ✅ View Base Service Plan page (display carrier/model pricing)

**Custom Service Plan Management Pages**
- ✅ Create Custom Service Plan page (add carrier/model pricing section)
- ✅ Edit Custom Service Plan page (add carrier/model pricing section)
- ✅ View Custom Service Plan page (display carrier/model pricing)

**Backend Changes**
- ✅ Fix service plan ID bug (prevent new IDs on edit)
- ✅ Database schema for carrier/model pricing overrides
- ✅ Billing logic to apply carrier/model-specific pricing
- ✅ API endpoints for managing carrier/model pricing

**Data**
- ✅ Device model reference data (i-22, Origin, etc.)
- ✅ Carrier reference data (Verizon, AT&T, T-Mobile)
- ✅ Pricing override records

**Business Logic**
- ✅ Pricing resolution algorithm (override → default)
- ✅ Custom plan pricing inheritance
- ✅ Validation rules for pricing overrides

### Out of Scope (Future Enhancements)

**Reporting and Analytics**
- ❌ Report showing all custom plans derived from base plan
- ❌ Pricing comparison report across distributors
- ❌ Revenue analysis by carrier/model combination

**Advanced Features**
- ❌ Time-based pricing (promotional rates, seasonal pricing)
- ❌ Automatic price adjustments based on wholesale costs
- ❌ Tiered volume discounts per carrier/model
- ❌ Geographic pricing variations
- ❌ Contract-based pricing

**UI Enhancements**
- ❌ Bulk pricing import from CSV/Excel
- ❌ Pricing preview/calculator tool
- ❌ Visual pricing comparison charts
- ❌ Pricing history timeline

**Migration Tools**
- ❌ Automatic repricing of existing custom plans
- ❌ Migration wizard for updating distributor pricing
- ❌ Price change notification system

---

## Functional Requirements

### FR-1: Service Plan Pricing Structure

**FR-1.1: Default Pricing**
- Every service plan MUST have a default price per usage tier
- Default price applies when no carrier/model override exists
- Default price displayed with clear label: "Default Price - Applied unless carrier + model override is defined"
- Backward compatible: existing plans without overrides function identically

**FR-1.2: Carrier + Model Pricing Overrides**
- Service plans MAY include optional carrier + model pricing overrides
- Each override consists of: Device Model + Carrier + Price
- Multiple overrides can be defined for same service plan
- Overrides are optional; plans can have zero, one, or many overrides

**FR-1.3: Pricing Resolution Logic**
```
For a given device on a service plan:
1. Identify device model (e.g., i-22, Origin)
2. Identify device carrier (e.g., Verizon, AT&T, T-Mobile)
3. Check for carrier + model override matching both attributes
4. IF override exists: USE override price
5. ELSE: USE default price
```

**FR-1.4: Custom Plan Pricing Inheritance**
```
For custom service plans:
1. Inherit carrier/model structure from base plan
2. Apply discount/adjustment to each base price
3. Custom plan can have:
   - Custom default price (adjusted from base default)
   - Custom override prices (adjusted from base overrides)
4. If base plan adds new override, custom plan shows base override price until manually adjusted
```

### FR-2: Service Plan Management

**FR-2.1: Create Service Plan**
- Admin can create service plan with standard fields (name, device groups, usage limits, etc.)
- Admin can optionally add "Carrier + Model Pricing" section
- Section is collapsed by default (opt-in for cleaner UI)
- Admin can expand section to add overrides
- Admin can add multiple overrides using "+ Add Carrier + Model Combination" button
- Each override has dropdowns for: Model, Carrier, and Price input field
- Admin can remove individual overrides with delete button
- Form validates: no duplicate model/carrier combinations, prices > 0, required fields

**FR-2.2: Edit Service Plan (WITH BUG FIX)**
- Admin can edit any service plan field without creating new service plan ID
- System updates existing service plan record in place
- All custom service plans remain linked via original parent ID
- Approval workflow (if enabled) functions correctly with in-place updates
- Audit log tracks all changes with timestamps and user
- Warning displayed if editing base plan with many custom children

**FR-2.3: View Service Plan**
- Display standard service plan information
- Display "Price Tiers" section showing usage limits and default prices
- If carrier/model overrides exist, display section: "Optional pricing overrides for specific carrier and model combinations"
- List each override as: "Model + Carrier / Price" (e.g., "I-22 + T-Mobile / 3.50")
- Clear visual distinction between default and override pricing
- Edit button available to authorized users

**FR-2.4: Delete Service Plan**
- Cannot delete base service plan with active custom child plans (validation)
- Warning message: "This service plan has X custom service plans. Delete all custom plans first."
- Can delete base plan with no children
- Soft delete preserves historical data for billing/reporting

### FR-3: Custom Service Plan Management

**FR-3.1: Create Custom Service Plan**
- Select company and base service plan
- Form displays base service plan's price tier structure
- For each base price tier, input adjusted price
- If base plan has carrier/model overrides, display table:
  - Column 1: Model + Carrier
  - Column 2: Base Price (read-only, from base plan)
  - Column 3: Adjusted Price (editable)
- Can use "Apply" feature to bulk adjust all prices (increase/decrease by amount or %)
- Validation: adjusted prices must be valid numbers, can be lower or higher than base

**FR-3.2: Edit Custom Service Plan**
- Display current custom pricing for all tiers
- Display current custom pricing for all carrier/model overrides
- Can modify individual prices
- Can modify adjustment method (discount vs markup)
- Shows base prices for reference
- Validation prevents prices below operational minimums (warning, not blocker)

**FR-3.3: View Custom Service Plan**
- Display company, base service plan name
- Display custom default pricing per tier
- If carrier/model overrides exist, display table:
  - Column: Model + Carrier
  - Column: Base Price
  - Column: Adjusted Price
- Clear labeling of what is custom vs inherited from base
- Edit button for authorized users

**FR-3.4: Custom Plan Synchronization**
- When base plan adds new carrier/model override:
  - Custom plan automatically shows new override with base price
  - Admin must manually adjust custom price if discount desired
  - Default adjustment does NOT automatically apply to new overrides (explicit action required)
- When base plan removes carrier/model override:
  - Custom plan override becomes inactive but preserved (audit trail)
  - Devices revert to custom default price
- When base plan changes base price:
  - Custom plans unaffected (preserve explicit custom pricing)
  - Admin must manually adjust custom prices if desired

### FR-4: Billing Integration

**FR-4.1: Price Calculation**
- Billing system identifies service plan, device model, and carrier for each device
- Apply pricing resolution logic (FR-1.3)
- For custom plans, check custom override prices first
- Store applied price and reason in billing records (audit trail)
- Handle edge cases: missing model/carrier data, unknown combinations

**FR-4.2: Billing Reports**
- Invoices show per-device pricing (if detailed billing enabled)
- Invoices group by service plan and carrier/model if applicable
- Admin billing reports show price applied per device
- Distributor commission calculations use correct carrier/model pricing

**FR-4.3: Historical Pricing**
- Billing uses pricing in effect at time of billing cycle
- Changing prices affects future billing only (no retroactive changes)
- Audit trail preserves historical pricing decisions

### FR-5: Data Validation and Integrity

**FR-5.1: Input Validation**
- Prices must be numeric, positive values
- Prices support up to 2 decimal places
- Model and Carrier must be selected from valid reference data
- Cannot create duplicate model/carrier combinations on same service plan
- Warning (not blocker) if price below $4.25 (ACH minimum)

**FR-5.2: Referential Integrity**
- Device models must exist in device_models table
- Carriers must exist in carriers table
- Cannot delete model/carrier if pricing overrides exist (orphan protection)
- Custom plans must reference valid base service plan ID

**FR-5.3: Business Rule Validation**
- Warning if creating pricing substantially different from default (potential error)
- Warning if custom price lower than base price for distributor (negative margin alert)
- Validation errors prevent save; warnings allow save with confirmation

---

## Testing Requirements

### Test Plan Overview

**Testing Phases:**
1. Unit Testing (Developer responsibility)
2. Integration Testing (QA + Developer)
3. User Acceptance Testing (UAT) (Business stakeholders)
4. Performance Testing (QA + DevOps)
5. Regression Testing (QA)

### Integration Test Cases

#### End-to-End Workflows

**Test Case IT-1: Complete Service Plan Creation Flow**
1. Admin creates ATM service plan with default $4.95
2. Admin adds i-22 + T-Mobile override at $3.50
3. Admin adds Origin + Verizon override at $4.00
4. Admin saves service plan
5. View service plan page displays all pricing correctly
6. **Verify:** Database has service_plan record + 2 override records

**Test Case IT-2: Complete Custom Service Plan Creation Flow**
1. Admin selects base ATM plan (with 2 overrides from IT-1)
2. Admin sets adjusted default price to $3.75
3. Admin adjusts i-22 + T-Mobile to $3.00
4. Admin adjusts Origin + Verizon to $3.50
5. Admin saves custom service plan
6. **Verify:** Database has custom_service_plan record + 2 custom override records

**Test Case IT-3: Complete Billing Cycle with Overrides**
1. Create test company with custom service plan from IT-2
2. Assign 3 devices:
   - Device 1: i-22 on T-Mobile
   - Device 2: Origin on Verizon
   - Device 3: i-22 on AT&T
3. Run billing cycle
4. **Verify:** Billing line items show:
   - Device 1: $3.00 (custom override)
   - Device 2: $3.50 (custom override)
   - Device 3: $3.75 (custom default)
5. **Verify:** Invoice total = $10.25

**Test Case IT-4: Edit Base Plan Without Breaking Custom Plans**
1. Base ATM plan has 5 custom service plans as children
2. Admin edits base ATM plan (changes name, adds 3rd override)
3. Admin saves base plan
4. **Verify:** Base plan ID unchanged
5. **Verify:** All 5 custom plans still linked (query custom_service_plans.service_plan_id)
6. **Verify:** All 5 custom plans show new 3rd override (with base price)

**Test Case IT-5: Distributor Creates Sub-Customer Custom Plan**
1. Distributor logs in (has custom ATM plan at $3.75)
2. Distributor creates custom plan for sub-customer
3. Distributor sets sub-customer default price to $6.99
4. Distributor adjusts sub-customer overrides to $6.50 and $6.75
5. Distributor saves
6. **Verify:** Sub-customer plan created and linked to distributor's custom plan as base

### User Acceptance Test Cases

#### UAT-1: Admin Creates Service Plan with Carrier/Model Pricing
**Persona:** WATM Administrator
**Scenario:** Create new service plan for upcoming Origin device launch

**Steps:**
1. Navigate to Service Plans → Create Service Plan
2. Enter service plan name: "Origin Special"
3. Set default price: $4.50
4. Expand "Carrier + Model Pricing" section
5. Add override: Origin + T-Mobile = $3.50
6. Add override: Origin + Verizon = $4.00
7. Click Save
8. Navigate to View Service Plan page
9. Verify all pricing displayed correctly

**Success Criteria:**
- ✅ Override section easy to find and use
- ✅ Adding overrides intuitive
- ✅ Form validation prevents errors
- ✅ View page clearly displays pricing structure
- ✅ No confusion about default vs override pricing

#### UAT-2: Admin Edits Service Plan Without Fear
**Persona:** WATM Administrator
**Scenario:** Need to add new carrier/model override to existing ATM plan with many customers

**Steps:**
1. Navigate to ATM service plan (has 50 custom child plans)
2. Click Edit
3. Notice warning: "This plan has 50 custom service plans"
4. Add new override: Origin + AT&T = $4.25
5. Click Save
6. Navigate away and back to ATM plan
7. Verify override added
8. Check one custom service plan
9. Verify custom plan still works and shows new override

**Success Criteria:**
- ✅ Warning message provides confidence (not breaking children)
- ✅ Edit process smooth and fast
- ✅ No errors during save
- ✅ Custom plans still functional
- ✅ Admin comfortable editing service plans going forward

#### UAT-3: Distributor Adjusts Custom Pricing for New Override
**Persona:** Distributor (Tyler)
**Scenario:** WATM added T-Mobile pricing to ATM plan, Tyler needs to adjust his custom plan

**Steps:**
1. Distributor logs in
2. Navigates to Manage Customer Service Plans
3. Edits their custom ATM plan
4. Sees new override: i-22 + T-Mobile = $3.50 (base price)
5. Understands this is WATM's cost
6. Adjusts to $3.00 (maintaining their $0.50 discount)
7. Saves custom plan
8. Views custom plan to verify

**Success Criteria:**
- ✅ Clear that $3.50 is base price (WATM cost)
- ✅ Obvious where to enter custom price
- ✅ Can calculate margin easily
- ✅ Save process smooth
- ✅ Distributor confident they set correct pricing

#### UAT-4: Finance Reviews Billing with Mixed Pricing
**Persona:** WATM Finance Manager
**Scenario:** Review monthly billing to ensure carrier/model pricing applied correctly

**Steps:**
1. Run monthly billing cycle
2. Export admin billing report
3. Filter for company with mixed device types
4. Review per-device pricing
5. Verify i-22 T-Mobile devices charged $3.50
6. Verify i-22 Verizon devices charged $4.95
7. Verify totals correct
8. Spot check invoices sent to customers

**Success Criteria:**
- ✅ Report clearly shows device model and carrier
- ✅ Pricing varies correctly by model/carrier
- ✅ Totals accurate
- ✅ No billing errors or disputes
- ✅ Finance confident in new pricing model

### Regression Test Cases

**Test Case RT-1: Existing Service Plans Unaffected**
- **Verify:** All service plans without overrides function identically pre/post deployment
- **Verify:** Billing for these plans unchanged
- **Verify:** No price changes unless explicitly configured

**Test Case RT-2: Existing Custom Plans Unaffected**
- **Verify:** All custom service plans maintain current pricing
- **Verify:** No orphaned custom plans
- **Verify:** No unexpected price changes in billing

**Test Case RT-3: Other System Features Unaffected**
- **Verify:** Device management functions normally
- **Verify:** User management functions normally
- **Verify:** Reports generate correctly
- **Verify:** APIs respond correctly

---

## Dependencies and Risks

### Dependencies

**Internal Dependencies:**
1. **Device Model Data Accuracy**
   - Device records must have correct device_model_id
   - Devices must have correct carrier_id
   - **Mitigation:** Audit and clean device data before launch

2. **Billing System Integration**
   - Billing system must be able to query device model and carrier
   - Billing system must support new pricing resolution logic
   - **Mitigation:** Thorough integration testing

3. **Approval Workflow**
   - If service plan approval system exists, must work with ID immutability fix
   - **Mitigation:** Review approval workflow code, implement versioning if needed

**External Dependencies:**
1. **Carrier Data Updates**
   - If new carriers onboarded, need to add to carriers table
   - **Mitigation:** Document process for adding new carriers

2. **Device Model Updates**
   - If new device models released, need to add to device_models table
   - **Mitigation:** Document process for adding new models

### Risks

#### HIGH RISK: Service Plan ID Bug Not Fully Fixed

**Description:** The service plan ID bug is complex and may have edge cases not covered
**Impact:** High - Could orphan custom plans, major data integrity issue
**Probability:** Medium
**Mitigation:**
- Extensive testing of edit scenarios
- Code review focused on ID immutability
- Test with approval workflow (if exists)
- Rollback plan ready
- Monitor closely in production for first 2 weeks

#### HIGH RISK: Billing Errors from Incorrect Pricing

**Description:** Pricing resolution logic has bugs, causing incorrect billing
**Impact:** High - Revenue loss, customer disputes, refunds required
**Probability:** Low
**Mitigation:**
- Comprehensive unit and integration tests
- Parallel run billing in UAT vs production data (compare results)
- Billing audit report to catch discrepancies
- Gradual rollout (ATM plan first, then others)
- Easy rollback to default pricing (feature flag)

#### MEDIUM RISK: Performance Degradation in Billing

**Description:** Pricing resolution queries slow down billing significantly
**Impact:** Medium - Billing takes too long, delays invoicing
**Probability:** Low
**Mitigation:**
- Performance testing before launch
- Database query optimization
- Caching strategy implementation
- Monitor billing cycle duration closely
- Optimize queries if needed post-launch

#### MEDIUM RISK: User Confusion About Pricing Structure

**Description:** Admins/distributors confused by default vs override pricing
**Impact:** Medium - Incorrect pricing set, support burden
**Probability:** Medium
**Mitigation:**
- Clear UI labels and help text
- User training/documentation
- Tooltips explaining concepts
- Support team training
- Monitor support tickets for confusion patterns

#### LOW RISK: Migration Issues with Existing Custom Plans

**Description:** Existing custom plans don't display correctly after deployment
**Impact:** Medium - Distributors see incorrect pricing, lose trust
**Probability:** Low
**Mitigation:**
- Backwards compatibility testing
- No automatic migration (reduces risk)
- Spot check custom plans post-deployment
- Communication to distributors about changes
- Support team ready to assist

#### LOW RISK: Reference Data Management

**Description:** Device models or carriers not kept up to date
**Impact:** Low - New devices/carriers can't use override pricing
**Probability:** Low
**Mitigation:**
- Document process for adding models/carriers
- Admin UI for managing reference data (future enhancement)
- Alert if device has unknown model or carrier
- Regular audit of reference data

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-12 | Product Team | Initial PRD created |

---

**Document Status:** Draft for Review
**Next Review Date:** [To be scheduled after stakeholder review]
**Approvals Required:**
- [ ] Product Manager
- [ ] Engineering Lead
- [ ] QA Lead
- [ ] Business Stakeholders (Devon, Adam)
- [ ] Finance Manager

---

END OF DOCUMENT
