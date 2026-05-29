# Product Requirements Document
## T-Mobile FWA (Fixed Wireless Access) Support

**Document Version:** 1.0
**Date:** 2026-05-04
**Author:** Aksana Rahouski
**Status:** Draft
**Related Tickets:** TBD (pending Jira ticket creation)
**Document Owner:** Aksana
**Last Updated:** 05.04.2026

---

## Table of Contents

1. [Overview](#overview)
2. [Goals and Success Criteria](#goals-and-success-criteria)
3. [Users](#users)
4. [Scope](#scope)
5. [Functional Requirements and Business Rules](#functional-requirements-and-business-rules)
6. [Technical Requirements](#technical-requirements)
7. [Testing Requirements](#testing-requirements)
8. [Dependencies and Risks](#dependencies-and-risks)
9. [Open Questions](#open-questions)
10. [References](#references)
11. [Notes](#notes)

---

## Overview

This feature extends the FWA (Fixed Wireless Access) business account support -- originally built for Verizon under epic WATM-2027 -- to T-Mobile devices. APW has begun onboarding customers onto T-Mobile's FWA unlimited plan ($75/month), but the portal has no way to differentiate T-Mobile FWA devices from standard pay-per-use (PPU) T-Mobile devices. Operations is currently using flat-rate overrides, admin notes, and off-system tracking as workarounds -- resulting in billing errors (a $400 credit was recently issued to a customer).

The T-Mobile implementation is significantly simpler than Verizon because T-Mobile FWA operates under the **same account and API** as regular T-Mobile SIMs. No separate provider account, API endpoints, or credentials are needed. The portal change is primarily a **labeling and validation layer**: a "Rate Plan Type" flag on the device (PPU vs. FWA) that controls which service plans can be assigned.

### Key Features

- **Rate Plan Type field on T-Mobile devices** -- PPU (default) or FWA designation, super admin only
- **Shared FWA service plan** -- T-Mobile FWA devices use the same FWA service plan as Verizon, with carrier-specific pricing tiers ($75/month for T-Mobile)
- **Service plan validation** -- FWA devices restricted to FWA plans only; PPU devices restricted to PPU plans only
- **Device import support** -- Rate plan type selectable during T-Mobile device import

### Business Impact

- **Revenue accuracy**: Enables proper billing for T-Mobile FWA devices at $75/month through the service plan system rather than manual flat-rate workarounds
- **Operational efficiency**: Eliminates flat-rate overrides, admin notes, and off-system tracking for T-Mobile FWA devices
- **Configuration management**: FWA devices can be assigned to service plans, enabling configuration file delivery to devices that currently receive none
- **Billing integrity**: Prevents assigning wrong plan type to devices (PPU plan on FWA device or vice versa)
- **Scalability**: Current workaround covers only 2 devices and will not scale as T-Mobile FWA adoption grows

---

## Goals and Success Criteria

### Primary Goals

1. **Enable T-Mobile FWA Device Identification**: Add a rate plan type field to T-Mobile devices so operations can distinguish PPU from FWA devices in the portal
2. **Enable Service Plan Assignment for T-Mobile FWA**: Allow T-Mobile FWA devices to be assigned to the shared FWA service plan with T-Mobile-specific pricing ($75/month)
3. **Enforce Plan-Device Compatibility**: Validate that T-Mobile FWA devices can only use FWA plans and T-Mobile PPU devices can only use PPU plans
4. **Eliminate Manual Workarounds**: Remove the need for flat-rate overrides, admin notes, and off-system tracking for T-Mobile FWA devices

### Success Criteria

- ✅ All T-Mobile devices display a rate plan type field with two options: PPU (default) and FWA
- ✅ Only super admin users can view and modify the rate plan type field
- ✅ T-Mobile FWA devices can be assigned to the shared FWA service plan with $75/month T-Mobile pricing
- ✅ System validates and blocks incompatible service plan/rate plan type combinations on save
- ✅ Device import supports rate plan type selection for T-Mobile (PPU default if not specified)
- ✅ Existing T-Mobile devices default to PPU with no disruption to current operations
- ✅ Funcrest's 2 FWA devices can be migrated from flat-rate workaround to proper FWA service plan assignment

---

## Users

### Primary Users

**Operations Team (Devon D'Andrea, Dan)**
- **Role**: Day-to-day device management, SIM provisioning, service plan assignment
- **Need**: Identify which T-Mobile devices are on FWA vs. PPU; assign correct service plans; deliver configurations
- **Pain Point**: Currently using flat-rate workarounds and admin notes to track FWA devices; T-Mobile SIMs not entered in portal SIM field because behavior is unknown; no service plan assigned means no configuration delivery
- **Benefit**: Proper device flagging; service plan assignment; configuration delivery; accurate billing through the existing system

**Admin Users (Adam Curcie)**
- **Role**: Account management, billing oversight, rate plan type designation
- **Need**: Set rate plan type on T-Mobile devices; ensure billing accuracy; manage FWA customer onboarding
- **Pain Point**: Manual billing adjustments required; recently issued ~$400 credit to Funcrest customer due to workaround-related billing errors; no visibility into which devices are on which plan type
- **Benefit**: Automated billing through service plan system; clear device labeling; no manual overrides needed

---

## Scope

### In Scope

**Device-Level Rate Plan Type**
- ✅ Add "Rate Plan Type" field to T-Mobile devices with two options: PPU (default) and FWA
- ✅ Display field on device view and edit pages for T-Mobile devices
- ✅ Restrict field visibility and editing to super admin access level only
- ✅ Allow freely changing between PPU and FWA (no lock-after-save behavior)
- ✅ Default existing T-Mobile devices to PPU

**Shared FWA Service Plan**
- ✅ Add T-Mobile as a carrier option in the existing FWA service plan's pricing tiers
- ✅ T-Mobile FWA pricing configurable per tier (initial: $75/month flat)
- ✅ No new service plan type needed -- uses existing FWA plan structure built for Verizon

**Service Plan Validation**
- ✅ T-Mobile FWA devices can only be assigned FWA service plans
- ✅ T-Mobile PPU devices can only be assigned PPU service plans
- ✅ Validation on save with clear error messages (same pattern as Verizon)
- ✅ Applies to single device edit and bulk device assignment

**Device Import**
- ✅ Update T-Mobile device import template to support rate plan type column
- ✅ Valid values: PPU, FWA; default if not specified: PPU
- ✅ Same approach as Verizon device import update

**Device Assignment**
- ✅ Service plan assignment follows same rules as Verizon FWA
- ✅ Bulk assignment with partial assignment support (compatible devices succeed, incompatible listed in error)

### Out of Scope

- ❌ **T-Mobile FWA plan automation via API** -- T-Mobile does not support setting SIM cards to the FWA plan via API or self-service. Plan changes require emailing T-Mobile; they confirm via email. This is a carrier limitation.
- ❌ **T-Mobile callback/webhook for plan change notifications** -- Investigating whether T-Mobile can notify the portal when a SIM plan changes is a future consideration (similar to Verizon callbacks).
- ❌ **Automated Verizon FWA plan switching** -- Separate scope, not addressed here.
- ❌ **Different T-Mobile API accounts or endpoints** -- Not needed. T-Mobile FWA uses the same account and API as regular T-Mobile SIMs.
- ❌ **Restricting rate plan type edits after initial save** -- The field is freely editable. Changing it does not affect the carrier side, billing, or device performance. It is purely administrative.
- ❌ **Cellular backup line item for FWA** -- Devon mentioned a potential 2 GB line item for cellular backup devices on FWA plans. Confirmed not needed for this scope.

---

## Functional Requirements and Business Rules

### FR-1: Rate Plan Type Field

**FR-1.1: Field Definition**
- The system shall add a "Rate Plan Type" field to all T-Mobile devices
- The field shall have two options: PPU (Pay Per Use) and FWA (Fixed Wireless Access)
- The field shall default to PPU when no selection is made
- The field shall be stored at the device level, associated with the T-Mobile SIM card

**Business Rules:**
- Only users with super admin access level can view and modify this field
- Non-super-admin users (distributors, sub-companies, customers) shall not see or interact with this field
- The field is freely editable with no restrictions on changing between PPU and FWA
- Changing the rate plan type does not trigger any API call to T-Mobile
- Changing the rate plan type does not affect device billing calculations or device performance
- The rate plan type is an administrative label that controls which service plans can be assigned

**Interaction & UI Details:**
- Field location: Under the T-Mobile SIM card section on the device edit page (same placement pattern as Verizon's "Provider Account Type")
- Input type: Radio button selector (PPU | FWA), matching Verizon's implementation
- Device view page: Display rate plan type as a badge or label
- For Verizon devices, the existing "Provider Account Type" field remains unchanged

**FR-1.2: Default and Migration Behavior**
- All existing T-Mobile devices shall default to PPU (no data migration required -- PPU is the assumed state when no value is set)
- New T-Mobile devices shall default to PPU unless explicitly set to FWA during creation or import

**Business Rules:**
- NULL or empty rate plan type shall be treated as PPU for all validation and display purposes

### FR-2: Service Plan Compatibility

**FR-2.1: Validation Rules**
- The system shall validate that a device's rate plan type is compatible with the assigned service plan before saving
- If device.rate_plan_type = 'fwa', only service plans marked as FWA shall be allowed
- If device.rate_plan_type = 'ppu' (or NULL/default), only service plans marked as PPU/regular shall be allowed
- If an incompatible combination is attempted, the system shall display an error message and block the save

**Business Rules:**
- Validation shall occur on save, not in dropdown filtering
- Same validation logic as Verizon FWA (reuse existing implementation)
- Applies to both single device edit and bulk device assignment workflows
- Customers cannot self-assign a PPU device to an FWA plan or vice versa

**Interaction & UI Details:**
- Service plan dropdown: Show all service plans (no filtering based on device rate plan type)
- Error message on save: Clear message identifying that the device's rate plan type is incompatible with the selected service plan
- For bulk assignment: List incompatible devices by serial number in the error message

**FR-2.2: Shared FWA Service Plan with T-Mobile Pricing**
- T-Mobile FWA devices shall use the same FWA service plan as Verizon FWA devices
- The existing FWA service plan shall support T-Mobile as a carrier in the pricing tier configuration
- Each pricing tier shall allow a T-Mobile-specific price (initial configuration: $75/month)

**Business Rules:**
- No new service plan type needs to be created
- Carrier-specific pricing tiers are the mechanism for differentiating T-Mobile ($75/month) from Verizon pricing
- When a T-Mobile FWA device is billed, the system uses the T-Mobile carrier pricing from the assigned service plan tier

### FR-3: API Behavior

**FR-3.1: No API Differentiation**
- All T-Mobile API calls shall use the same endpoint and credentials regardless of rate plan type
- Status changes (activate, suspend, deactivate, resume) shall behave identically for PPU and FWA devices
- No API routing logic changes are required (unlike Verizon, which requires separate credentials per account)

**Business Rules:**
- T-Mobile FWA plan changes at the carrier level are handled outside the portal via email:
  1. APW emails T-Mobile requesting a SIM be moved to the FWA plan
  2. T-Mobile confirms via email
  3. Admin updates the portal's rate plan type field to match
- The portal does not automate or initiate any T-Mobile plan changes

### FR-4: Device Import

**FR-4.1: Import Template Update**
- The T-Mobile device import template shall include a rate plan type column
- Valid values: PPU, FWA (case-insensitive)
- Default if column is empty or not specified: PPU

**Business Rules:**
- Invalid rate plan type values shall generate an import error for that row
- Same import approach as the Verizon device import update

### FR-5: Bulk Device Assignment

**FR-5.1: Bulk Validation with Partial Assignment**
- When assigning a service plan to multiple T-Mobile devices via the Assign Device page, the system shall validate each device's rate plan type against the selected service plan
- Devices that pass validation shall be successfully assigned
- Devices that fail validation shall be skipped and listed in an error message with serial numbers
- Same partial assignment behavior as Verizon FWA bulk assignment

**Business Rules:**
- Success message shall show both successful and failed counts
- Admins can review failed devices, correct rate plan types, and re-submit

### Key User Flows

- **Flow 1 -- New T-Mobile FWA Device Setup**: Admin creates device -> enters T-Mobile SIM -> sets rate plan type to FWA -> assigns company -> assigns FWA service plan -> device receives configuration and is billed at T-Mobile FWA pricing
- **Flow 2 -- Migrate Existing Workaround Device**: Admin edits existing Funcrest device -> enters T-Mobile SIM in SIM field -> sets rate plan type to FWA -> assigns FWA service plan -> removes flat-rate override -> device now managed through normal portal workflow
- **Flow 3 -- Import T-Mobile FWA Devices**: Admin imports CSV with T-Mobile devices and FWA rate plan type -> devices created with FWA flag -> admin assigns FWA service plan in bulk

---

## Technical Requirements

> Technical complexity is low. No separate TRD required. Changes extend existing Verizon FWA patterns.

### Integration
- **T-Mobile API**: No changes required. All existing T-Mobile API integrations continue to function identically for both PPU and FWA devices. Same endpoints, same credentials, same request/response formats.

### Security & Compliance
- **Access control**: Rate plan type field restricted to super admin role. Follows the same access control pattern as Verizon's provider account type field.
- **Audit logging**: Rate plan type changes shall be logged with user info, timestamp, and old/new value. Same logging pattern as existing device field changes.

### Data Model Overview

**Device (T-Mobile SIM extension)**
| Field | Type | Required | Validation | Notes |
|-------|------|----------|------------|-------|
| rate_plan_type | Enum | No | 'ppu' or 'fwa' | Defaults to 'ppu' if NULL |

**Rate Plan Type Enum:**
- `ppu` -- Pay Per Use (default). Standard T-Mobile SIM on usage-based billing.
- `fwa` -- Fixed Wireless Access. T-Mobile SIM on unlimited $75/month plan.

---

## Testing Requirements

### Test Plan Overview
**Testing Phases:**
1. Integration Testing (QA team)
2. User Acceptance Testing (UAT) (Devon/Adam)
3. Regression Testing (QA team)

### Key Test Scenarios

**IT-1: Rate Plan Type Default**
1. Create a new T-Mobile device
2. Do not select a rate plan type
3. **Verify:** Rate plan type defaults to PPU

**IT-2: Rate Plan Type Edit -- Super Admin**
1. Log in as super admin
2. Edit a T-Mobile device, change rate plan type from PPU to FWA
3. **Verify:** Field saves successfully
4. Change rate plan type back from FWA to PPU
5. **Verify:** Field saves successfully (no lock-after-save restriction)

**IT-3: Rate Plan Type -- Access Control**
1. Log in as distributor user
2. View a T-Mobile device with FWA rate plan type
3. **Verify:** Rate plan type field is not visible
4. Repeat for sub-company and customer user roles
5. **Verify:** Field is not visible or editable for any non-super-admin role

**IT-4: Service Plan Validation -- FWA Device Blocked from PPU Plan**
1. Set a T-Mobile device's rate plan type to FWA
2. Attempt to assign a PPU service plan
3. Click save
4. **Verify:** Error message displayed; save is blocked

**IT-5: Service Plan Validation -- FWA Device Accepts FWA Plan**
1. Set a T-Mobile device's rate plan type to FWA
2. Assign an FWA service plan
3. Click save
4. **Verify:** Saves successfully

**IT-6: Service Plan Validation -- PPU Device Blocked from FWA Plan**
1. Set a T-Mobile device's rate plan type to PPU
2. Attempt to assign an FWA service plan
3. Click save
4. **Verify:** Error message displayed; save is blocked

**IT-7: Service Plan Validation -- PPU Device Accepts PPU Plan**
1. Set a T-Mobile device's rate plan type to PPU
2. Assign a PPU service plan
3. Click save
4. **Verify:** Saves successfully

**IT-8: Shared FWA Plan -- T-Mobile Carrier Pricing**
1. Open the FWA service plan configuration
2. **Verify:** T-Mobile is available as a carrier in pricing tiers
3. Set T-Mobile pricing to $75/month
4. Assign the FWA plan to a T-Mobile FWA device
5. **Verify:** Billing for that device reflects the $75 T-Mobile tier pricing, not the Verizon tier pricing

**IT-9: Device Import -- Rate Plan Type**
1. Import T-Mobile devices with rate plan type column set to "PPU"
2. **Verify:** Devices created with PPU rate plan type
3. Import T-Mobile devices with rate plan type column set to "FWA"
4. **Verify:** Devices created with FWA rate plan type
5. Import T-Mobile devices with rate plan type column blank/empty
6. **Verify:** Devices default to PPU
7. Import T-Mobile devices with invalid rate plan type value (e.g., "INVALID")
8. **Verify:** Import error generated for that row

**IT-10: Bulk Assignment -- Partial Assignment**
1. Select a mix of T-Mobile PPU and FWA devices on the Assign Device page
2. Select an FWA-only service plan
3. Click Assign
4. **Verify:** FWA devices are successfully assigned
5. **Verify:** PPU devices are listed in error message with serial numbers
6. **Verify:** Success message shows correct counts (X succeeded, Y failed)

**IT-11: API Operations Unchanged**
1. Set a T-Mobile device to FWA rate plan type
2. Activate the device
3. **Verify:** Activation uses the same T-Mobile API endpoint as PPU devices; no errors
4. Suspend the device
5. **Verify:** Suspension uses the same endpoint; no errors

**UAT-1: Funcrest Migration**
- **Persona:** Devon (Operations)
- **Scenario:** Migrate Funcrest's 2 existing FWA devices from flat-rate workaround to portal-managed service plan
- **Steps:**
  1. Edit device, enter T-Mobile SIM in SIM field
  2. Set rate plan type to FWA
  3. Assign FWA service plan with $75 T-Mobile pricing tier
  4. Remove flat-rate override
  5. Verify device appears correctly in portal with FWA badge
  6. Verify next billing cycle generates correct $75 charge through service plan
- **Success Criteria:** Both devices billed correctly at $75/month through service plan system; no manual flat-rate override needed; configuration files delivered to devices

### Regression Testing

**RT-1: Existing T-Mobile PPU Devices**
- **Verify:** All existing T-Mobile devices continue operating normally with no changes to behavior, billing, or API calls

**RT-2: Verizon FWA Functionality**
- **Verify:** Verizon FWA feature (provider account type, API routing, service plan validation) is not affected by T-Mobile changes

**RT-3: Non-T-Mobile Devices**
- **Verify:** AT&T and Verizon devices are not affected by the rate plan type field (field should not appear on non-T-Mobile devices)

**RT-4: Existing Service Plan Assignments**
- **Verify:** All existing device-to-service-plan assignments remain intact after deployment

---

## Dependencies and Risks

### Dependencies

**Must Be Complete Before Development:**
1. **WATM-2106: Verizon FWA UAT Feedback** -- The unlimited display logic fixes, FWA plan scoping, and "Service Plan Usage Limit" field removal must be finalized. T-Mobile FWA reuses the same service plan structure, so it must be correct first. | **Mitigation:** T-Mobile rate plan type field and validation can be developed in parallel; only the service plan pricing tier addition depends on WATM-2106 completion.
2. **Verizon FWA Service Plan** -- The shared FWA service plan must exist and be functional in production before T-Mobile can be added as a carrier pricing tier. | **Mitigation:** Verify FWA plan is deployed and tested on review environment.

**Integrates With:**
- Existing T-Mobile API integration (no changes needed)
- Existing FWA service plan structure (add T-Mobile carrier pricing tier)
- Device import system (add rate plan type column for T-Mobile)
- Service plan validation system (extend existing Verizon FWA validation to T-Mobile)
- Bulk device assignment system (extend partial assignment validation)

### Risks

**LOW RISK: T-Mobile API Behavior Changes**
- **Description:** The current assumption is that T-Mobile API behavior is identical for FWA and PPU devices. If T-Mobile introduces API-level differentiation in the future, the portal would need API routing changes similar to Verizon.
- **Impact:** Low -- Would require additional development but the rate plan type field is already in place to support routing.
- **Probability:** Low -- Adam confirmed same API, same endpoints, same account.
- **Mitigation:** The rate plan type field provides the data foundation. API routing can be added later using the same pattern implemented for Verizon (credential switching based on device field).

**LOW RISK: Manual Plan Change Process Bottleneck**
- **Description:** T-Mobile FWA plan changes require emailing T-Mobile and waiting for confirmation. As FWA adoption grows, this manual process could become a bottleneck.
- **Impact:** Medium -- Increased operational overhead for the APW team.
- **Probability:** Low -- Currently 2 devices; growth expected to be gradual.
- **Mitigation:** Investigate T-Mobile callback capability for automated plan change notification (documented as open question, out of scope for this version).

---

## Open Questions

### 1. T-Mobile Callback Capability
- **Question:** Can T-Mobile (via Simple) initiate a callback/webhook to the portal when they change a SIM to the FWA plan? This would automate the portal-side rate plan type update.
- **Decision Needed By:** Not urgent -- can be investigated independently
- **Decision Owner:** Adam / development team
- **Status:** Open -- not yet investigated with T-Mobile/Simple. Currently only 2 FWA devices; manual update is acceptable.

---

## Notes

### Verizon FWA vs. T-Mobile FWA Comparison

| Aspect | Verizon FWA | T-Mobile FWA |
|--------|-------------|--------------|
| Separate API account | Yes (separate ThingSpace profile) | No (same account) |
| Separate API credentials | Yes | No |
| API endpoint differences | Yes (different routing) | No (same endpoints) |
| Field name | Provider Account Type | Rate Plan Type |
| Options | Regular (PPU) / Unlimited Internet (FWA) | PPU / FWA |
| Default | Regular (PPU) | PPU |
| Who can change | Super admin only | Super admin only |
| Freely editable | Yes | Yes |
| Plan change automation | Not automated (Phase 2 consideration) | Not possible via API currently |
| Plan change process | Manual Verizon process (5 steps) | Email T-Mobile; they confirm |
| Service plan | Shared FWA plan | Shared FWA plan |
| Pricing mechanism | Carrier-specific tiers | Carrier-specific tiers ($75/month) |
| Service plan validation | FWA device -> FWA plan only | FWA device -> FWA plan only |
| Device import | Supports account type column | Supports rate plan type column |
| Billing impact of field change | Yes (wrong API credentials = wrong billing) | No (purely administrative label) |
| Devices in production | ~15 | 2 (Funcrest customer) |

---

END OF DOCUMENT
