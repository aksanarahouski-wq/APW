# Product Requirements Document (PRD)
## Dual Carrier Fee Handling - Move to Company Level

**Document Version:** 1.0
**Date:** January 22, 2026
**Author:** Aksana Rahouski, Stone Marballie
**Status:** Client Review Complete - Ready for Implementation
**Related Tickets:** [To be created]
**Document Owner:** Aksana Rahouski

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Background and Problem Statement](#background-and-problem-statement)
3. [Goals and Objectives](#goals-and-objectives)
4. [Scope](#scope)
5. [Functional Requirements](#functional-requirements)
6. [Testing Requirements](#testing-requirements)
7. [Open Questions](#open-questions)
8. [Document History](#document-history)

---

## Executive Summary

This feature moves dual carrier fee pricing from the service plan level to the company level, introducing separate pricing for AT&T and T-Mobile dual carrier configurations. This change provides more granular control over dual carrier pricing and aligns with the business strategy of incentivizing T-Mobile dual carrier adoption with lower pricing for the new Origin device.

Currently, all dual carrier devices are charged a single fee ($4.95 default) regardless of which secondary carrier they use. With the introduction of the Origin device (Verizon/T-Mobile dual carrier), the business needs differentiated pricing to offer competitive rates for T-Mobile dual carrier while maintaining existing AT&T dual carrier pricing.

### Key Features
- **Separate Dual Carrier Fees**: Two distinct fee fields for AT&T ($4.95) and T-Mobile ($3.50) dual carrier configurations
- **Company-Level Customization**: Per-company pricing flexibility for both carrier types
- **Intelligent Billing**: Automatic fee selection based on device secondary carrier configuration
- **Backward Compatible**: Seamless migration of existing customer pricing

### Business Impact
- **Revenue Optimization**: Enables competitive pricing for Origin device adoption while maintaining margins on traditional dual carrier devices
- **Customer Flexibility**: Allows customized pricing negotiations for different carrier configurations
- **Strategic Alignment**: Supports business strategy to incentivize T-Mobile dual carrier adoption with $3.50 pricing vs $4.95 for AT&T

**Production Deadline:** February 6, 2026

### ✅ Client Review Complete (January 26, 2026)

All critical decisions have been received and documented:
- **Migration Strategy:** Option C - Preserve existing values as AT&T fees, set T-Mobile to $3.50 default
- **Default Values:** Confirmed - AT&T $4.95, T-Mobile $3.50
- **Special Handling:** None required - standard migration for all customers
- **Null Values:** None exist in current data (verified by client export)

**Status:** Ready for Implementation

---

## Background and Problem Statement

### Current State

The system currently has:
- Single "Dual Carrier Fee" field at the company level
- Default value: $4.95
- Applied uniformly to all dual carrier devices regardless of secondary carrier (AT&T or T-Mobile)
- Fee is independent of data usage pricing and billed per device, per billing cycle
- Customizable per company for special pricing arrangements

### Problems

**Problem 1: Inability to Price Differentiate by Carrier**
- With the Origin device launch (Verizon/T-Mobile dual carrier), business wants to offer $3.50 T-Mobile dual carrier fee
- Current system only supports single dual carrier fee, preventing carrier-specific pricing
- **Who is affected**: Sales team, customers purchasing Origin devices, billing operations
- **Business impact**: Cannot implement competitive pricing strategy for new Origin device, limiting market competitiveness

**Problem 2: Pricing Strategy Misalignment**
- Business strategy aims to incentivize T-Mobile dual carrier adoption with lower pricing
- Traditional AT&T dual carrier pricing ($4.95) should remain unchanged
- **Who is affected**: Product management, sales, customers
- **Business impact**: Missing opportunity to drive Origin device adoption through strategic pricing

### Impact if Not Addressed

- **Lost Revenue Opportunity**: Cannot launch Origin devices with competitive $3.50 T-Mobile dual carrier pricing
- **Customer Confusion**: Forced to charge Origin customers same $4.95 fee as traditional devices, reducing value proposition
- **Competitive Disadvantage**: Inability to offer differentiated pricing limits market positioning
- **Timeline Risk**: February 6, 2026 production deadline for Origin device billing

---

## Goals and Objectives

### Primary Goals

1. **Enable Carrier-Specific Dual Carrier Pricing**: Support separate pricing for AT&T and T-Mobile dual carrier configurations at the company level
2. **Maintain Pricing Flexibility**: Preserve ability for companies to have custom dual carrier fees, now for both carriers
3. **Ensure Billing Accuracy**: Automatically apply correct dual carrier fee based on device secondary carrier configuration

### Success Criteria

_Measurable criteria that define when this project is successful_

- ✅ Two separate dual carrier fee fields visible and editable on company settings page
- ✅ Default values correctly set: $4.95 for AT&T, $3.50 for T-Mobile
- ✅ Invoice generation automatically selects correct fee based on device configuration
- ✅ Existing custom pricing preserved according to approved migration strategy
- ✅ Invoice display clearly shows which dual carrier fee was applied (AT&T or T-Mobile)
- ✅ All existing automated tests pass; new tests cover dual carrier fee logic
- ✅ No impact on single-carrier device billing
- ✅ Production deployment by February 6, 2026

### Non-Goals (Out of Scope)

_What this project will NOT include_

- ❌ Changes to service plan pricing structure (handled in separate ticket)
- ❌ Dual carrier device configuration or provisioning changes
- ❌ Carrier selection logic modifications
- ❌ Historical invoice adjustments or recalculation
- ❌ AT&T/T-Mobile dual carrier combination support (confirmed will never exist)
- ❌ Changes to how data usage is calculated or billed

---

## Scope

### In Scope

**Database Changes**
- ✅ Add new field: `dual_carrier_fee_tmobile` to companies table
- ✅ Rename/repurpose existing field: `dual_carrier_fee` → `dual_carrier_fee_att` (or add new field)
- ✅ Data migration script to populate both fields using **Option C approach**:
  - `dual_carrier_fee_att`: Preserve existing `dual_carrier_fee` values (custom AT&T pricing)
  - `dual_carrier_fee_tmobile`: Set to $3.50 default for all companies
- ✅ Migration script verification: Check for null values (client reports none exist)
- ✅ Default value configuration for new companies: AT&T = $4.95, T-Mobile = $3.50

**User Interface Updates**
- ✅ Company settings page: Add second dual carrier fee input field
- ✅ Label existing field as "Dual Carrier Fee (AT&T)"
- ✅ Add new field labeled "Dual Carrier Fee (T-Mobile)"
- ✅ Update form validation to handle both fields
- ✅ Maintain per-company customization capability

**Billing Logic Updates**
- ✅ Modify invoice generation to check device secondary carrier configuration
- ✅ Implement logic to select appropriate dual carrier fee based on carrier
- ✅ Update invoice display to show carrier-specific fee label
- ✅ Ensure backward compatibility during transition period

### Out of Scope (Future Enhancements)

**Service Plan Changes**
- ❌ Changes to service plan pricing structure (separate ticket)
- ❌ Model-based pricing variations

**Device Configuration**
- ❌ Dual carrier device configuration changes
- ❌ Carrier selection logic or device provisioning
- ❌ Support for AT&T/T-Mobile dual carrier combination

**Historical Data**
- ❌ Historical invoice adjustments
- ❌ Recalculation of past billing cycles
- ❌ Retroactive pricing changes

---

## Functional Requirements

### FR-1: Company Settings Page

**FR-1.1: Display Two Dual Carrier Fee Fields**
- Company settings page must display two separate input fields for dual carrier fees
- Fields must be clearly labeled: "Dual Carrier Fee (AT&T)" and "Dual Carrier Fee (T-Mobile)"
- Fields must accept decimal values with two decimal places (e.g., 4.95)
- Fields must be positioned adjacently or in a logical grouping

**FR-1.2: Default Values for New Companies**
- When a new company is created, Dual Carrier Fee (AT&T) defaults to $4.95
- When a new company is created, Dual Carrier Fee (T-Mobile) defaults to $3.50
- Defaults can be overridden by administrator during company setup

**FR-1.3: Update Existing Company Settings**
- Administrators can modify either or both dual carrier fee values
- Changes save successfully to database when form is submitted
- Changes take effect immediately for future billing cycles
- No impact on invoices already generated

### FR-2: Billing Logic

**FR-2.1: Fee Selection Based on Secondary Carrier**
- During invoice generation, system retrieves device secondary carrier (AT&T or T-Mobile)
- If secondary carrier is AT&T: apply company's Dual Carrier Fee (AT&T)
- If secondary carrier is T-Mobile: apply company's Dual Carrier Fee (T-Mobile)
- If device is single carrier: no dual carrier fee is applied

**FR-2.2: Primary Carrier Billing Rule**
- All dual carrier devices have Verizon as primary carrier (business rule)
- Data usage is billed at Verizon rate regardless of which SIM transmitted data
- Dual carrier fee is in addition to data usage charges
- Total data usage = Primary carrier usage + Secondary carrier usage

**FR-2.3: Fee Application Rules**
- Dual carrier fee is applied once per device per billing cycle
- Fee is independent of actual data usage on secondary carrier
- Fee amount is determined solely by secondary carrier type and company configuration

### FR-3: Data Validation and Integrity

**FR-3.1: Input Validation**
- Dual carrier fee fields must accept values >= $0.00
- Negative values are rejected with error message
- Values must be numeric with up to 2 decimal places
- Maximum value: $999.99 (reasonable business constraint)
- Empty/null values: If encountered (client reports none exist), use system defaults:
  - AT&T field: $4.95
  - T-Mobile field: $3.50
- Note: Sub-customers with $0.00 values inherit parent company pricing (not null/empty)

**FR-3.2: Business Rule Validation**
- No validation prevents setting both fees to $0.00 (valid for promotional pricing)
- System warns if fees are set significantly different from defaults (e.g., >$10 difference)
- No cross-validation between AT&T and T-Mobile fees (they are independent)

**FR-3.3: Data Migration Validation**
- Migration script implements **Option C** approach:
  - Preserve existing `dual_carrier_fee` values as `dual_carrier_fee_att` (AT&T fees)
  - Set all `dual_carrier_fee_tmobile` values to $3.50 default
- Migration script validates all companies have values after migration
- Migration script logs any null values found (client reports none exist, but verify)
- Migration verifies custom AT&T pricing preserved (not all set to $4.95 default)
- Migration verifies all T-Mobile fees set to $3.50 default
- Migration produces summary report: number of companies migrated, range of AT&T fee values, any anomalies
- Rollback capability if validation fails

### FR-4: Invoice Display

**FR-4.1: Line Item Format**
- Invoice must show dual carrier fee with carrier designation
- Format: "Dual Carrier Fee (AT&T): $X.XX" or "Dual Carrier Fee (T-Mobile): $X.XX"
- Line item appears in appropriate section of invoice (fees section)
- Amount displayed matches company's configured fee for that carrier

**FR-4.2: Invoice Accuracy**
- Carrier designation on invoice matches device secondary carrier configuration
- Amount charged matches company's configured fee at time of invoice generation
- Dual carrier fee appears for dual carrier devices only
- No dual carrier fee for single carrier devices

---

## Testing Requirements

### Test Plan Overview

**Testing Phases:**
1. User Acceptance Testing (UAT) (Business stakeholders)
2. Regression Testing (QA)

**Target Completion:** Before February 3, 2026 (Beta deployment)

### Integration Test Cases

#### End-to-End Workflows

**Test Case IT-1: New Company with Origin Device (T-Mobile Dual)**
1. Create new company in system
2. Verify Dual Carrier Fee (AT&T) = $4.95, Dual Carrier Fee (T-Mobile) = $3.50
3. Add Origin device (Verizon/T-Mobile dual carrier) to company
4. Device uses 5GB Verizon + 5GB T-Mobile data in billing cycle
5. Generate invoice
6. **Verify:** Invoice shows "Dual Carrier Fee (T-Mobile): $3.50"
7. **Verify:** Data usage charged at Verizon rate for 10GB total
8. **Verify:** Total = Data charges + $3.50

**Test Case IT-2: Existing Company with Traditional Dual Carrier (AT&T)**
1. Select existing company with traditional dual carrier device (Verizon/AT&T)
2. After migration, verify Dual Carrier Fee (AT&T) field has correct value
3. Device uses 8GB data in billing cycle
4. Generate invoice
5. **Verify:** Invoice shows "Dual Carrier Fee (AT&T): $[company_rate]"
6. **Verify:** Fee matches company's configured AT&T dual carrier fee

**Test Case IT-3: Custom Pricing Configuration**
1. Navigate to company settings page
2. Set Dual Carrier Fee (AT&T) = $5.00
3. Set Dual Carrier Fee (T-Mobile) = $4.00
4. Save changes
5. Add one device with AT&T dual carrier, one with T-Mobile dual carrier
6. Generate invoice
7. **Verify:** AT&T device charged $5.00 dual fee
8. **Verify:** T-Mobile device charged $4.00 dual fee
9. **Verify:** Both fees display correct carrier designation on invoice

**Test Case IT-4: Edge Case - Zero Dollar Dual Carrier Fee**
1. Set company Dual Carrier Fee (T-Mobile) = $0.00
2. Add Origin device to company
3. Generate invoice
4. **Verify:** Invoice shows "Dual Carrier Fee (T-Mobile): $0.00" or omits line item (per business rule)
5. **Verify:** Total invoice amount does not include dual carrier fee

**Test Case IT-5: Single Carrier Device (No Dual Fee)**
1. Add single carrier Verizon device to company
2. Device uses 10GB data
3. Generate invoice
4. **Verify:** No dual carrier fee line item appears
5. **Verify:** Only data usage charges applied

### User Acceptance Test Cases

#### UAT-1: Origin Device Purchase and Billing
**Persona:** Sales Team / Account Manager
**Scenario:** Customer purchases Origin device with T-Mobile dual carrier, wants to verify competitive pricing

**Steps:**
1. Set up new customer company in system
2. Verify default T-Mobile dual carrier fee is $3.50
3. Add Origin device to customer account
4. Simulate one billing cycle with device usage
5. Generate invoice for review
6. Review invoice with customer

**Success Criteria:**
- ✅ Setup process is straightforward and clear
- ✅ Invoice clearly shows $3.50 T-Mobile dual carrier fee
- ✅ Customer understands pricing breakdown
- ✅ Fee is competitive and aligns with sales pitch

#### UAT-2: Custom Pricing Negotiation
**Persona:** Sales Team / Operations
**Scenario:** High-value customer negotiates custom dual carrier pricing for both AT&T and T-Mobile devices

**Steps:**
1. Navigate to customer company settings
2. Update Dual Carrier Fee (AT&T) to $3.00
3. Update Dual Carrier Fee (T-Mobile) to $2.50
4. Save configuration
5. Add mix of AT&T and T-Mobile dual carrier devices
6. Generate invoice

**Success Criteria:**
- ✅ Both fees can be customized independently
- ✅ Custom pricing applied correctly to respective devices
- ✅ Invoice clearly differentiates the two fee types
- ✅ Finance team can verify correct pricing

#### UAT-3: Migration Validation
**Persona:** Billing Operations / Finance
**Scenario:** Verify existing custom pricing preserved after migration

**Steps:**
1. Identify 5-10 companies with custom dual carrier fees before migration
2. Document current custom fee values
3. Run migration
4. Verify each company's fees after migration
5. Generate test invoices for sample devices

**Success Criteria:**
- ✅ Custom pricing values match expected values per migration strategy
- ✅ No unexpected changes to customer pricing
- ✅ Invoices generate without errors
- ✅ Finance team confirms pricing accuracy

### Regression Test Cases

**Test Case RT-1: Single Carrier Billing Unchanged**
- **Verify:** Single carrier devices still billed correctly with no dual carrier fee
- **Verify:** Data usage calculations unchanged
- **Verify:** Invoice format for single carrier devices unchanged

**Test Case RT-2: Data Usage Calculation Unchanged**
- **Verify:** Total data usage = Primary + Secondary carrier usage (same as before)
- **Verify:** Billing rate based on primary carrier (Verizon) only
- **Verify:** Dual carrier fee is separate line item (not part of data charges)

**Test Case RT-3: Existing Automation and Integrations**
- **Verify:** Automated billing processes complete successfully
- **Verify:** Invoice generation scheduled jobs run without errors
- **Verify:** Any downstream systems consuming invoice data still function

---

## Open Questions

### ✅ ALL QUESTIONS ANSWERED - Client Review Complete (January 26, 2026)

---

### 1. Data Migration Strategy ✅ DECIDED

**Question:** How should we migrate existing dual carrier fee data from single field to two fields (AT&T and T-Mobile)?

**CLIENT DECISION: Option C - Repurpose existing for AT&T only**

**Migration Approach:**
- Keep current `dual_carrier_fee` values as AT&T fees (preserves existing AT&T custom pricing)
- Set T-Mobile field to $3.50 default for all companies
- This aligns with the fact that most existing dual carrier devices are AT&T-based

**Decision Date:** January 26, 2026
**Decision By:** Client (Devon D'Andrea, Adam Curcie)

---

### 2. Default Values Confirmation ✅ CONFIRMED

**Question:** Confirm the default values for new dual carrier fee fields?

**CLIENT DECISION: Values Confirmed**

**Default Values:**
- Dual Carrier Fee (AT&T): **$4.95** ✅
- Dual Carrier Fee (T-Mobile): **$3.50** ✅

**Decision Date:** January 26, 2026
**Decision By:** Client (Devon D'Andrea, Adam Curcie)

---

### 3. Special Customer Handling ✅ DECIDED

**Question:** Are there any specific customers or customer segments requiring special handling during migration?

**CLIENT DECISION: No special customer handling required**

**Confirmation:**
- No specific customers require special handling
- Standard migration approach (Option C) applies to all customers uniformly
- All customers will receive:
  - AT&T dual carrier fee: Current value preserved
  - T-Mobile dual carrier fee: $3.50 default

**Decision Date:** January 26, 2026
**Decision By:** Client (Devon D'Andrea, Adam Curcie)

---

### 4. Null/Empty Value Handling ✅ ADDRESSED

**Question:** If a dual carrier fee field is empty/null in the database, what should the system do?

**CLIENT FEEDBACK: No null values exist in current data**

**Data Verification:**
- Client reviewed export of all companies from APC
- Dual sim upcharge field has values for all customers and sub-customers
- **No null values found in current data**
- If any null values are discovered during implementation, they can be addressed at that time

**Important Note on Sub-Customer $0.00 Values:**
- Sub-customers with dual sim upcharge of "$0.00" are **still paying** the dual sim upcharge they inherit from their distributor
- They are not paying "$0.00" (free service)
- They are simply not paying anything **in addition** to what the distributor charges
- This is inheritance behavior, not a null/empty value scenario

**Implementation Action Required:**
- ✅ Verify no null values exist during migration script execution
- ✅ Log any null values found (if any) for client review
- ✅ If null values are found, apply Option A (Use system default: $4.95 for AT&T, $3.50 for T-Mobile)

**Decision Date:** January 26, 2026
**Decision By:** Client (Devon D'Andrea, Adam Curcie)

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | January 22, 2026 | Aksana Rahouski | Initial PRD created following template, converted from requirements doc |
| 1.1 | January 26, 2026 | Aksana Rahouski | Updated with client decisions on all open questions. Migration strategy: Option C (repurpose existing for AT&T, default $3.50 for T-Mobile). Default values confirmed ($4.95 AT&T, $3.50 T-Mobile). No special customer handling needed. No null values exist in current data. Document status changed to "Ready for Implementation". |

---

**Document Status:** Client Review Complete - Ready for Implementation
**Client Review Date:** January 26, 2026
**Implementation Start Date:** TBD

---

## Appendix

### A. Wireframes/Mockups

**Company Settings Page - Current UI:**
```
Company Configuration
┌─────────────────────────────────────────┐
│ Dual Carrier Fee:  [ $4.95 ]           │
└─────────────────────────────────────────┘
```

**Company Settings Page - Proposed UI:**
```
Company Configuration
┌─────────────────────────────────────────┐
│ Dual Carrier Fee (AT&T):    [ $4.95 ]  │
│ Dual Carrier Fee (T-Mobile): [ $3.50 ]  │
└─────────────────────────────────────────┘
```

**Invoice Display - Before:**
```
Device XYZ123
- Data Usage (10GB Verizon): $40.00
- Dual Carrier Fee: $4.95
Total: $44.95
```

**Invoice Display - After:**
```
Device XYZ123 (Verizon/T-Mobile Dual)
- Data Usage (10GB Verizon): $40.00
- Dual Carrier Fee (T-Mobile): $3.50
Total: $43.50
```

### B. Related Documents
- [Service Plan Management Enhancements PRD](https://orases.atlassian.net/wiki/spaces/WATM/pages/2537717778/)
- [Client Feedback - January 22, 2026](https://orases.atlassian.net/wiki/spaces/WATM/pages/2751791125/)
- Meeting Transcript: `/Users/aksana/Documents/Projects/WATM/ServicePlanChanges/Meetings/meeting.md`
- Email Follow-up Draft: `/Users/aksana/Documents/Projects/WATM/ServicePlanChanges/Meetings/Email Draft - Jan 22 Follow-up.md`

### C. Business Context

**Origin Device Background:**
- New device model with Verizon/T-Mobile dual carrier capability
- Strategic product for Q1 2026 launch
- Competitive pricing ($3.50 T-Mobile dual fee) is key selling point
- Traditional dual carrier devices (Verizon/AT&T) remain at $4.95

**Pricing Philosophy:**
- Primary carrier is always Verizon for dual carrier devices
- Customers purchase "Verizon service" with backup SIM
- Failover to secondary carrier doesn't change billing rate
- Dual carrier fee is for having "hot" backup SIM, not for data routing
- Data usage billed at primary carrier (Verizon) rate regardless of which SIM transmitted data

---

END OF DOCUMENT
