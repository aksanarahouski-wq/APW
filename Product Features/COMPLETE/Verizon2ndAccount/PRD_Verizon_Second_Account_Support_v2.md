# Verizon Second Account Support - PRD
## Verizon Second Account Support - FWA Integration

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Background and Problem Statement](#background-and-problem-statement)
3. [Goals and Objectives](#goals-and-objectives)
4. [Target Users](#target-users)
5. [User Stories](#user-stories)
6. [Scope](#scope)
7. [Functional Requirements](#functional-requirements)
8. [Technical Design](#technical-design)
9. [Testing Requirements](#testing-requirements)
10. [Dependencies and Risks](#dependencies-and-risks)
11. [Implementation Plan](#implementation-plan)
12. [Success Metrics](#success-metrics)
13. [Known Limitations and Phase 2 Considerations](#known-limitations-and-phase-2-considerations)
14. [Open Questions](#open-questions)
15. [Document History](#document-history)

---

## Executive Summary

The WATM portal currently supports a single Verizon account for managing devices with Verizon SIM cards. Verizon has mandated that Fixed Wireless Access (FWA) devices with unlimited plans must be managed through a separate account profile due to billing and pricing model differences. This PRD outlines the implementation of multi-account Verizon support to enable management of devices across both the original Regular Account (Pay Per Use) and the new Unlimited Internet (FWA) account.

The solution will store Verizon account configuration at the device level, allowing each device to be explicitly assigned to the appropriate account. Service plans will be configured with Verizon account restrictions, enabling enforcement of compatibility rules—Unlimited Internet (FWA) plans will only be available to devices on the FWA account, and Regular Account (PPU) plans only to devices on the regular account. This prevents billing errors and ensures compliance with Verizon's pricing tier separation.

**Important Scope Note:** The portal's role is **tracking-only**. The portal tracks which Verizon account each device belongs to but does NOT automate the process of moving devices between accounts. Account migration is a complex manual process requiring specific operational procedures outside the portal. Only authorized operations personnel will manually update the portal's account field after completing the manual Verizon migration process.

### Key Features

**1. Device-Level Verizon Account Configuration**
- Every device with a Verizon SIM explicitly specifies which Verizon account it belongs to (Regular Account (PPU) or Unlimited Internet (FWA))
- Field stored at device level, independent of service plan
- Device uses this field to determine which Verizon API credentials to use for all operations (activation, deactivation, status changes, etc.)
- Implementation touchpoints:
  - Device create/edit form with radio button selector
  - Device view page with account badge display
  - Device import template (Column 16 for Verizon account)
  - Device update import with account column support

**2. Service Plan Account Restrictions**
- Service plans marked as compatible with specific Verizon accounts (FWA-only OR Regular-only)
- Every service plan MUST select one of two options: FWA-only or Regular-only 
- Unlimited Internet (FWA) service plans can ONLY be used by devices on FWA account
- Regular Account (PPU) service plans can ONLY be used by devices on regular account
- Service plans control MORE than billing - they also control device configurations (APNs, etc.)
- Wrong service plan = wrong APN config = device won't connect
- Dynamic form behavior: Gray out usage limits and device group names when FWA account selected

**3. Service Plan Compatibility Validation**
- Single device edit: Validates service plan matches device's Verizon account before save
- Bulk device assignment: Validates ALL selected devices are compatible with chosen service plan
- Clear error messages identifying incompatible devices by serial number
- Service plan dropdowns show ALL service plans (no filtering)
- Validation occurs on save, not in dropdown display
- Validation rules:
  - If device.verizon_account = 'fwa' → only allow service plans marked as 'fwa'
  - If device.verizon_account = 'regular' → only allow service plans marked as 'regular'
  - If incompatible combination selected → display error message on save and block save

**4. Bulk Device Assignment with Partial Assignment Support**
- Assign Device page validates service plan compatibility for each device in the batch
- System supports **partial assignment** (not all-or-nothing)
- Devices that pass validation are successfully assigned
- Devices that fail validation are skipped and listed in error message with serial numbers
- Admins can review failed devices, correct issues, and re-submit subset
- Leverages existing robust error handling foundation

**5. API Call Routing Based on Device Account**
- All Verizon API calls automatically route to correct account based on device configuration
- 15+ locations in codebase that instantiate VerizonApi updated to pass account parameter
- Device account field determines which credentials to use
- Fixes current problem: 15 existing FWA devices causing API failures

**6. Tracking-Only Portal Role for Account Management**
- Portal displays account status and allows authorized admins to update account field after manual migration
- No automated account switching (too complex/risky - requires 5-step manual Verizon process)
- Admin can manually update device.verizon_account_name field post-migration
- All account changes logged for audit trail

### Business Impact
- **Compliance**: Enables compliance with Verizon's mandatory account separation for FWA devices
- **Revenue Opportunity**: Unlocks ability to offer unlimited FWA plans ($70-80+ MRC per device)
- **Billing Accuracy**: Prevents incompatible service plan/account pairings that could cause billing errors
- **Operational Efficiency**: Automates account routing, preventing manual errors and API failures

---

## Background and Problem Statement

### Problems

**Problem 1: Verizon-Mandated Account Separation**
- Verizon requires separate account profiles for Fixed Wireless Access (FWA) devices
- FWA devices use flat-rate unlimited plans ($70-80+ MRC) categorized as "Unlimited Internet"
- Original account uses pay-per-use pricing with low MRC plans (usage-based billing)
- Verizon makes it difficult to switch between unlimited and pay-per-use plans (to combat SIM banking, network abuse, fraud)
- **Who is affected**: Operations team managing device configurations
- **Business impact**: Cannot offer FWA unlimited plans, losing revenue opportunity; risk of API failures if wrong credentials used

**Problem 2: No Account Differentiation Mechanism**
- Current system has no way to specify which Verizon account a device belongs to
- All API calls use single global account configuration
- No validation that device is on correct account before API operations
- **Who is affected**: Operations team, development team, API reliability
- **Business impact**: Cannot support second account; risk of billing errors; manual workarounds required

**Problem 3: No Service Plan - Verizon Account Compatibility Enforcement**
- Service plans are not currently linked to Verizon account requirements
- FWA unlimited service plans ($70-80+ MRC) can only exist on the FWA Verizon account
- Regular pay-per-use service plans can only exist on the regular Verizon account
- No mechanism to prevent incompatible service plan/account combinations in either single device edit or bulk assignment workflows
- **Device Edit Page**: No validation when assigning service plan to single device
- **Assign Device Page (Bulk Assignment)**: Can assign incompatible service plans to entire batches of devices simultaneously, magnifying the impact of configuration errors
- System does not validate that a device's service plan matches its Verizon account
- Admins could misconfigure service plans without knowing which account they require
- Bulk assignment workflow particularly risky - single mistake affects multiple devices
- **Who is affected**: Operations team configuring plans (especially bulk operations), billing team
- **Business impact**:
  - Potential billing errors if device assigned to incompatible plan/account combination
  - **Bulk assignment errors can create widespread billing issues affecting many devices at once**
  - Risk of API failures when device account doesn't match service plan expectations
  - Manual intervention required to fix incompatible assignments (time-consuming for bulk errors)
  - Inability to enforce Verizon's pricing tier separation (unlimited vs. pay-per-use)
  - **Wrong service plan = wrong APN configuration = device connectivity failure**

### Impact if Not Addressed

**Business Impact:**
- Cannot offer FWA unlimited plans, losing potential $70-80+ MRC per device revenue
- Manual workarounds required, increasing operational overhead and error risk
- Cannot onboard new FWA customers
- Currently 10-15 FWA devices in manual management limbo
- **Billing errors from incompatible service plan/account pairings**
- **Cannot enforce Verizon's mandatory pricing tier separation**

**Technical Impact:**
- API calls may fail or use wrong account
- Billing discrepancies between WATM and Verizon
- Data integrity issues (database shows wrong account/status)
- **Service plans could be assigned to devices on incompatible Verizon accounts**
- **No validation to prevent mismatched configurations**
- **Wrong APN configurations causing device connectivity failures**

**Customer Impact:**
- Devices may lose connectivity due to wrong configurations
- Risk of devices being activated on wrong account with incorrect billing
- **Unexpected billing charges from incompatible plan assignments**

---

## Goals and Objectives

### Primary Goals

1. **Enable Multi-Account Verizon Support**: Implement device-level Verizon account configuration to support both Regular Account (PPU) and Unlimited Internet (FWA) accounts simultaneously
2. **Ensure API Call Routing**: Guarantee all Verizon API calls route to the correct account based on device configuration
3. **Implement Service Plan - Account Compatibility Enforcement**: Define service plans with required Verizon account (FWA or regular), validate compatibility, and prevent incompatible service plan/account combinations
4. **Provide Tracking-Only Portal Interface**: Allow authorized operations staff to track and update device account status after manual migration processes
5. **Maintain Backward Compatibility**: Ensure existing devices default to original account with no disruption

### Success Criteria

- ✅ All devices can be explicitly assigned to a Verizon account (regular or FWA)
- ✅ All 15+ Verizon API call locations route to correct account based on device configuration
- ✅ Service plans must be marked as either FWA-only or Regular-only (every service plan requires account selection)
- ✅ System validates and prevents incompatible service plan/device account combinations
- ✅ Device import template includes Verizon account specification
- ✅ Authorized admins can update account field after manual migration (tracking-only, no automation)
- ✅ Dynamic service plan form behavior (gray out fields when FWA selected)
- ✅ Existing devices continue functioning with no disruption (default to original account)
- ✅ All account assignments and changes are logged for audit
- ✅ Zero API failures due to incorrect account credentials
- ✅ Zero billing errors from incompatible service plan/account pairings

### Non-Goals (Out of Scope)

- ❌ Automated account migration via portal (manual process only)
- ❌ Customer self-service account upgrades (operations team only)
- ❌ Support for more than 2 Verizon accounts (can be future enhancement)
- ❌ Automatic account determination based on SIM number patterns
- ❌ Real-time sync of device status from Verizon API on page load
- ❌ AT&T or T-Mobile multi-account support (Verizon only)
- ❌ Service plan automatic account assignment (explicit configuration required)
- ❌ 5G device support (requires different activation process - Phase 2)
- ❌ Mid-cycle billing automation (manual workaround acceptable for now - Phase 2)

---

## Target Users

### Primary Users

**1. Operations Team / Admin Users**
- **Role**: Internal staff managing device inventory and configuration
- **Need**: Assign devices to correct Verizon account during setup and import; track account status after manual migrations
- **Pain Point**: Manual tracking of which devices are on FWA vs. regular account; no system support; 15 devices currently causing API failures
- **Benefit**: Clear device-level configuration; bulk import support; tracking interface for account status

---

## User Stories

### Epic 1: Device-Level Verizon Account Configuration

**Story 1.1: Assign Verizon Account to Device**
- **As an** Operations Admin
- **I want to** assign a Verizon account to a device when creating or editing it
- **So that** the correct account credentials are used for all API operations

**Acceptance Criteria:**
- Device edit form includes "Verizon Account" radio button selector (⚪ Regular Account (PPU) | ⚪ Unlimited Internet (FWA))
- Options: "Regular Account (PPU)", "Unlimited Internet (FWA)"
- Default selection is "Regular Account (PPU)" (maps to original account)
- Selection is saved to device record
- Device view page displays Verizon account badge
- Device index table includes filterable "Verizon Account" column

**Story 1.2: Bulk Import with Verizon Account**
- **As an** Operations Admin
- **I want to** specify Verizon account in bulk device import
- **So that** new devices are configured correctly from the start

**Acceptance Criteria:**
- Import template includes Column 16: "Verizon Account"
- Accepts values: empty (defaults to "regular"), "regular", "fwa"
- Invalid values show error and prevent import
- Import summary shows count by account
- All imported devices have verizon_account_name populated

### Epic 2: Service Plan Account Restrictions

**Story 2.1: Configure Service Plan Account Restriction**
- **As an** Operations Admin
- **I want to** mark service plans as compatible with specific Verizon accounts
- **So that** only appropriate service plans can be assigned to devices

**Acceptance Criteria:**
- Service plan edit form includes "Verizon Account Compatibility" selector
- Options: "No Restriction", "Regular Account Only", "Unlimited Internet (FWA) Only"
- NULL/empty means no restriction (available to all devices)
- Service plan view page displays account restriction
- When FWA selected, usage limits and device group names are grayed out (not required)

**Story 2.2: Dynamic Service Plan Form Behavior**
- **As an** Operations Admin
- **I want** irrelevant fields to be grayed out when creating FWA service plans
- **So that** I focus only on the fields that matter for FWA plans

**Acceptance Criteria:**
- When "Unlimited Internet (FWA)" selected, gray out: Usage Limits, Device Group Names
- Keep enabled: Service Plan Name, WiFi checkbox, Firewall checkbox, Show Usage checkbox
- Help text explains which fields are required for FWA plans
- Grayed-out fields are not required for form submission

**Story 2.3: Validate Service Plan Compatibility**
- **As the** System
- **I want to** prevent saving incompatible service plan assignments (device assignment, device update flows)
- **So that** data integrity is maintained even if dropdown is bypassed

**Acceptance Criteria:**
- Validation rule checks device.verizon_account_name matches service_plan.verizon_account_name
- If service plan has no restriction, allow any device
- If mismatch, show error: "This service plan requires [account] but device is configured for [account]"
- Validation runs on create and update
- API/bulk updates also validated

**Story 2.4: Bulk Device Assignment with Account Compatibility**
- **As an** Operations Admin
- **I want to** bulk assign multiple devices to a service plan with automatic account compatibility validation
- **So that** I can efficiently assign devices while preventing incompatible plan/account combinations

**Acceptance Criteria:**
- Assign Device page allows selecting multiple devices and a service plan
- Service plan dropdown shows ALL service plans (no filtering based on device accounts)
- Validation occurs when "Assign" is clicked
- If selected service plan is incompatible with any devices, those devices fail validation and assignment is skipped for them (partial assignment)
- Validation occurs before save, checking each device's compatibility
- System supports **partial assignment**: Compatible devices succeed, incompatible devices fail with detailed error
- If any device incompatible, show error: "Cannot assign service plan: {count} device(s) incompatible. Serial numbers: {list}"
- Error message lists serial numbers of incompatible devices
- Success message shows count of devices assigned

### Epic 3: Admin Account Tracking (Portal Role)

**Story 3.1: Admin Updates Device Account After Manual Migration**
- **As an** Operations Admin with authorization
- **I want to** update a device's Verizon account field in the portal after completing manual migration
- **So that** the portal reflects the current state and API calls route correctly

**Acceptance Criteria:**
- Admin users can edit device.verizon_account_name field
- NO confirmation dialog required (only 3 authorized people)
- Change is logged to o_logs: "Admin updated device Verizon account from [old] to [new]"
- Admin receives success message confirming update

**Story 3.2: Manual Migration Process Documentation**
- **As an** Operations Admin
- **I want** clear documentation of the manual Verizon account migration process
- **So that** I know how to properly migrate devices between accounts outside the portal


**Story 3.3: Audit Trail for Account Changes**
- **As the** System
- **I want to** log all device account changes
- **So that** we have an audit trail for compliance and troubleshooting

**Acceptance Criteria:**
- All changes to device.verizon_account_name logged to o_logs
- Log entry includes: device_id, serial_number, old_account, new_account, user_id, timestamp
- Log category: "DEVICE VERIZON ACCOUNT CHANGE"
- Logs viewable in audit trail reports

### Epic 4: API Call Routing

**Story 4.1: Route API Calls to Correct Account**
- **As the** System
- **I want to** automatically use the correct Verizon account credentials for API calls
- **So that** operations succeed without manual configuration

**Acceptance Criteria:**
- All 15+ Verizon API call locations updated to determine account from device
- VerizonApi class accepts account key in constructor
- If device.verizon_account_name is 'fwa', use FWA credentials
- If device.verizon_account_name is 'regular' or NULL, use regular account credentials
- API call logging includes which account was used
- No hardcoded account selection - always determined from device configuration

**Story 4.2: Handle Missing Account Configuration**
- **As the** System
- **I want to** gracefully handle devices without explicit account configuration
- **So that** existing devices and edge cases continue functioning

**Acceptance Criteria:**
- If device.verizon_account_name is NULL or empty, default to 'regular' account
- Log warning if API call made for device without explicit account
- Existing devices (before migration) continue using original account
- Import without account specified defaults to 'regular'

---

## Scope

### In Scope

**1. Device-Level Verizon Account Configuration**
- ✅ Device create/edit form with **radio button selector** (⚪ Regular Account (PPU) | ⚪ Unlimited Internet (FWA))
- ✅ Device view page with account badge display
- ✅ Device import template: Support Verizon account column
- ✅ Device update import: Support Verizon account column
- ✅ Every device uses this field to determine which Verizon API credentials to use
- ✅ Create migration to populate existing devices with 'regular' account (default)
- ✅ Manual post-deployment update strategy for 15 existing FWA devices

**2. Service Plan Account Restrictions**
- ✅ Service plan edit form: Verizon Account Restriction selector (Regular-only, FWA-only)
- ✅ **Dynamic form behavior: Gray out usage limits and device group names when FWA account selected**
- ✅ Service plan view page: Account restriction indicator badge
- ✅ Unlimited Internet (FWA) service plans can ONLY be used by devices on FWA account
- ✅ Regular Account (PPU) service plans can ONLY be used by devices on regular account
- ✅ Service plans control device configurations (APNs) - wrong service plan = wrong APN = device won't connect

**3. Service Plan Compatibility Validation**
- ✅ **Service plan dropdown behavior**: Show ALL service plans (no filtering based on device account)
- ✅ **Single device edit**: Validate service plan matches device's Verizon account on save
- ✅ **Validation blocks save**: If incompatible combination, display error message and prevent save

**4. Bulk Device Assignment with Partial Assignment Support**
- ✅ **Service plan dropdown behavior**: Show ALL service plans (no filtering based on selected devices)
- ✅ **Bulk device assignment (Assign Device page)**: Validate ALL selected devices are compatible with chosen service plan when "Assign" is clicked
- ✅ **Partial assignment support** (not all-or-nothing):
  - Devices that pass validation are successfully assigned
  - Devices that fail validation are skipped and listed in error message
- ✅ Clear error messages identifying incompatible devices by serial number
- ✅ Success messages show both successful and failed counts
- ✅ Admins can review failed devices, correct issues, and re-submit subset
- ✅ Leverages existing robust error handling foundation (recently improved)

**5. API Call Routing Based on Device Account**
- ✅ VerizonApi class accepts account key parameter in constructor
- ✅ VerizonApi loads account-specific credentials from configuration
- ✅ Helper method to determine account from device
- ✅ Update all 15+ API call locations to pass correct account
- ✅ If device.verizon_account_name = 'fwa' → use FWA credentials
- ✅ If device.verizon_account_name = 'regular' or NULL → use regular account credentials
- ✅ API call logging includes which account was used (for debugging)
- ✅ Fixes current problem: 15 existing FWA devices causing API failures

**6. Admin Account Tracking (Portal Role)**
- ✅ Allow authorized admins to update device verizon_account field
- ✅ NO confirmation dialog (only 3 authorized people - Meeting 2 decision)
- ✅ Account change logging audit trail

**Audit Logging**
- ✅ Log device Verizon account assignment/changes
- ✅ Log admin account updates with user info
- ✅ Log API calls with account used (for debugging)
- ✅ Log import with account distribution summary (e.g., "Imported 50 devices: 45 Regular, 5 FWA")

**Reporting**
- ✅ Device list export includes Verizon account column


### Out of Scope (Future Enhancements)

**Automated Account Migration**
- ❌ Portal automation of account switching (too complex/risky - requires 5-step manual process)
- ❌ Customer self-service upgrade/downgrade
- ❌ Admin "migrate" button to automate account switching
- ❌ API calls triggered by portal to move device between accounts
- ❌ Confirmation dialogs for account changes (only 3 authorized people - not needed)

**Mid-Cycle Billing Automation (Phase 2)**
- ❌ Automated prorating for devices switching accounts mid-cycle
- ❌ Billing cycle split handling
- ❌ Automatic invoice adjustments for mid-cycle changes
- **Current approach**: Manual billing adjustments (5-10 devices/month - acceptable volume)
- **Revisit when**: Volume increases or manual process becomes too burdensome

**5G Device Support (Phase 2)**
- ❌ 5G FWA device activation (requires address validation and tower capacity checks)
- ❌ Different activation workflow for 5G vs. 4G
- **Current scope**: 4G devices only
- **Revisit when**: 5G FWA devices added to inventory

**Automatic Account Determination**
- ❌ Auto-detect account from SIM number patterns/ranges
- ❌ Import account suggestion based on SIM prefix
- ❌ Machine learning for account prediction

**Advanced Account Migration**
- ❌ Bulk account migration wizard
- ❌ Preview/dry-run before account change
- ❌ Automated revert workflows with approval chains

**Multi-Account Expansion**
- ❌ Support for 3+ Verizon accounts
- ❌ Dynamic account configuration (add accounts via UI)
- ❌ Account-specific rate plans and billing rules

**Cross-Carrier Multi-Account**
- ❌ AT&T multi-account support
- ❌ T-Mobile multi-account support
- ❌ Unified carrier account management

**Real-Time Validation**
- ❌ Check device actually exists in Verizon account via API during assignment
- ❌ Real-time SIM ownership validation
- ❌ Automatic account correction if mismatch detected

---

## Functional Requirements

### FR-1: Device Verizon Account Configuration

**FR-1.1: Device Account Field**
- System SHALL add `verizon_account_name` field to devices table
- Field SHALL be VARCHAR(100), nullable, with default value NULL
- Field SHALL have index for performance (filtering/searching)
- Field SHALL accept values: NULL, 'regular', 'fwa'

**FR-1.2: Device Edit Form Account Selection**
- Device edit form SHALL include "Verizon Account" radio button selector with two options:
  - ⚪ Regular Account (PPU) [Pay Per Use]
  - ⚪ Unlimited Internet (FWA) [Fixed Wireless Access]
- Selector options SHALL be: "Regular Account (PPU)", "Unlimited Internet (FWA)"
- Form SHALL save selected value to device.verizon_account_name
- Empty selection SHALL default to 'regular' in database


**FR-1.3: Device View Account Display**
- Device view page SHALL display Verizon account as badge
- Badge SHALL be color-coded: Blue for "Regular Account (PPU)", Orange for "Unlimited Internet (FWA)"
- Badge SHALL be prominent and easy to identify
- Tooltip SHALL show full account details

**FR-1.4: Device Import Account Column**
- Import template SHALL include Column 16: "Verizon Account"
- Column SHALL accept: empty (defaults to 'regular'), "regular", "fwa"
- Invalid values SHALL trigger validation error with clear message
- Import parser SHALL validate and normalize values
- Devices imported without account SHALL default to 'regular'
- Import summary SHALL show count by account: "Imported 50 devices: 45 Regular Account (PPU), 5 Unlimited Internet (FWA)"

**FR-1.6: Manual Migration Strategy for 15 Existing FWA Devices**
- 15 devices already on FWA account SHALL be visible in portal post-deployment
- Portal SHALL NOT automatically migrate these devices in database (manual update strategy)
- Operations team SHALL manually update each device post-deployment:
  - Set device.verizon_account_name = 'fwa'
  - Assign to appropriate FWA service plan
  - Verify API calls route to FWA account
- System SHALL prevent config updates to devices until account field explicitly set (fail-safe)

### FR-2: Service Plan Verizon Account Restrictions

**FR-2.1: Service Plan Account Restriction Field**
- System SHALL add `verizon_account_name` field to historical_service_plans table
- Field SHALL be VARCHAR(100), nullable, with default value NULL
- Field SHALL have index for performance
- Field SHALL accept values: NULL, 'regular', 'fwa'
- NULL SHALL mean "no restriction - available to all devices"

**FR-2.2: Service Plan Edit Form Restriction Selection**
- Service plan edit form SHALL include "Verizon Account Restriction" selector
- Selector options SHALL be: "No Restriction", "Regular Account (PPU) Only", "Unlimited Internet (FWA) Only"
- Empty/NULL SHALL mean no restriction
- Form SHALL save to historical_service_plans.verizon_account_name
- Help text SHALL explain: "Restricts which devices can use this service plan based on their Verizon account"

**FR-2.3: Dynamic Service Plan Form Behavior (NEW)**
- When "Unlimited Internet (FWA) Only" is selected in account restriction:
  - System SHALL gray out: Usage Limits fields, Device Group Names fields
  - System SHALL keep enabled: WiFi checkbox, Firewall checkbox, Show Usage checkbox
  - Help text SHALL explain: "Unlimited Internet (FWA) plans do not require usage limits or device group names"
- Grayed-out fields SHALL NOT be required for form submission
- Grayed-out fields MAY still be filled for edge cases (system remains flexible)

**FR-2.4: Service Plan View Restriction Display**
- Service plan view page SHALL display account restriction if set
- Display SHALL be prominent badge near plan name
- Badge SHALL match device account badge styling

**FR-2.5: Service Plan Dropdown Display (No Filtering)**
- Device edit form service plan dropdown SHALL display ALL service plans
- Assign Device page service plan dropdown SHALL display ALL service plans
- Dropdowns SHALL NOT filter based on device Verizon account or selected devices' accounts
- All service plans remain visible regardless of account compatibility
- **Rationale**: Simpler implementation; validation on save provides clear feedback; allows flexibility for edge cases

**FR-2.6: Service Plan Compatibility Validation**
- System SHALL validate service plan compatibility on device save
- If service plan has account restriction AND device has account configured:
  - If they match OR plan has no restriction → allow save
  - If they don't match → block save with error
- Error message SHALL be: "This service plan requires Verizon {plan_account} account, but device is configured for {device_account} account."
- Validation SHALL apply to: manual edits, API calls, bulk updates

**FR-2.7: Bulk Device Assignment Validation (Assign Device Page)**
- Assign Device page SHALL allow selecting multiple devices for bulk service plan assignment
- Service plan dropdown SHALL display ALL service plans (no filtering based on selected devices' accounts)
- Validation occurs when user clicks "Assign" button (not in dropdown display)
- System SHALL validate compatibility for EACH device in batch before save
- System SHALL support **PARTIAL ASSIGNMENT** (not all-or-nothing):
  - Compatible devices succeed and are assigned
  - Incompatible devices fail with detailed error message
- If ANY device incompatible with selected service plan:
  - Error message SHALL list incompatible devices: "Cannot assign service plan to {count} device(s). These devices are incompatible: {serial1}, {serial2}, {serial3}..."
  - Error message SHALL specify reason if possible: "Device {serial} is on {account} account but plan requires {plan_account} account"
  - Compatible devices SHALL still be assigned successfully
- On successful batch assignment:
  - Success message SHALL show count: "Successfully assigned {plan_name} to {count} devices. {failed_count} devices were incompatible."
  - o_logs entry SHALL record bulk assignment with device count

### FR-3: Admin Account Tracking (Portal Role)

**IMPORTANT SCOPE NOTE:** Portal role is **tracking-only**. The portal does NOT automate account migration. Account migration is a complex 5-step manual process performed outside the portal by authorized operations personnel.

**FR-3.1: Admin Updates Account Field After Manual Migration**
- Admin users SHALL be able to edit device.verizon_account_name field
- Field SHALL be editable in device edit form
- NO confirmation dialog required (only 3 authorized people perform migrations)
- On save:
  - System SHALL log to o_logs: "Admin updated device {serial} Verizon account from {old_account} to {new_account}"
  - System SHALL show success message: "Device Verizon account updated successfully"

**FR-3.3: Portal Does NOT Automate Migration**
- Portal SHALL NOT provide "Migrate" or "Upgrade" buttons
- Portal SHALL NOT trigger Verizon API calls to move device between accounts
- Portal SHALL NOT show customer-facing upgrade options
- Portal role: Display current account status and allow manual tracking updates only

**FR-3.4: Audit Trail**
- All changes to device.verizon_account_name SHALL be logged to o_logs
- Log entry SHALL include: device_id, serial_number, old_account, new_account, user_id, timestamp
- Log category SHALL be: "DEVICE VERIZON ACCOUNT CHANGE"
- Logs SHALL be viewable in audit trail reports

### FR-4: Verizon API Call Routing

**FR-4.1: VerizonApi Account Configuration**
- VerizonApi constructor SHALL accept optional `$accountKey` parameter (default: 'regular')
- Constructor SHALL load credentials from Configure::read("Verizon.accounts.{$accountKey}")
- If account key not found, SHALL fall back to 'regular' account
- Constructor SHALL log warning if fallback occurs

**FR-4.2: Device Account Determination Helper**
- VerizonApi class SHALL provide static method: `getAccountKeyFromDevice(Device $device): string`
- Method SHALL return:
  - 'fwa' if device.verizon_account_name = 'fwa'
  - 'regular' if device.verizon_account_name = 'regular' OR NULL
  - Value of device.verizon_account_name for other configured accounts
- Method SHALL never return NULL (always has fallback)

**FR-4.3: API Call Location Updates**
- ALL locations that instantiate VerizonApi SHALL be updated to:
  1. Determine account key from device using helper method
  2. Pass account key to VerizonApi constructor
- Affected locations (15+):
  - DevicesTable.php: beforeSave group assignment, status change logic
  - DeviceSimStatusesTable.php: SIM activation/deactivation
  - VerizonCallbacksController.php: callback processing
  - Various Command classes
- Pattern SHALL be:
  ```php
  $accountKey = VerizonApi::getAccountKeyFromDevice($device);
  $vzApi = new VerizonApi($accountKey);
  ```

**FR-4.4: API Call Logging**
- System SHALL log all Verizon API calls with account used
- Log entry SHALL include: device_id, serial_number, account_key, api_method, timestamp
- Logging scope SHALL be 'verizon_api' in logs/verizon-api.log
- Format SHALL be: "[{timestamp}] Device {serial} ({id}): Called {method} on {account} account"

**FR-4.5: Fallback Behavior**
- If device has no verizon_account_name (NULL), SHALL default to 'regular' account
- If device.verizon_account_name has invalid value, SHALL default to 'regular' account and log warning
- Warning SHALL include: device_id, serial_number, invalid value
- System SHALL never fail API call due to missing account configuration

### FR-5: Data Validation and Integrity

**FR-5.1: Import Validation**
- Import parser SHALL validate Verizon account values
- Valid values: empty, "regular", "fwa" (case-insensitive)
- Invalid values SHALL show error: "Device with serial number '{serial}': Invalid Verizon account '{value}'. Must be 'regular' or 'fwa'."
- Invalid rows SHALL be skipped, import continues with remaining valid rows
- Import summary SHALL list skipped rows with reasons

**FR-5.2: Bulk Update Validation with Partial Assignment Support**
- Bulk update operations SHALL validate account values before applying
- System SHALL support **PARTIAL ASSIGNMENT**:
  - Devices that pass validation are updated successfully
  - Devices that fail validation are skipped and listed in error message
- Validation SHALL check:
  - Account value is valid ('regular', 'fwa', or NULL)
  - If changing to 'fwa', device has verizon_sim_number
- Bulk update SHALL show summary of changes:
  - "Successfully updated {success_count} devices"
  - "Failed to update {fail_count} devices: {error_details}"

**FR-5.3: Configuration Validation**
- System SHALL validate Verizon account configuration on application startup
- Required fields for each account: key, secret, username, password, accountName
- Missing required fields SHALL log error and use fallback configuration
- Invalid configuration SHALL not prevent application startup (graceful degradation)

### FR-6: Role-Based Permissions Matrix

**FR-6.1: Write Access — Super Admin Only**
- Only super admin users SHALL be able to set or change the `provider_account_type` field on a device (via edit form or import)
- The account type radio button on the device create/edit form SHALL be hidden for all non-super-admin roles
- Server-side validation SHALL reject any `provider_account_type` change submitted by a non-super-admin user, regardless of how it was submitted (form, API, import)
- Device import (which includes the account type column) is already restricted to super admins; no additional permission change needed

**FR-6.2: Read Access — All Roles**
- The account type badge on the device view page SHALL be visible to all roles (super admin, company admin, customer)
- The account type column on the device list/index page SHALL be visible and filterable by all roles
- The account type value SHALL be included in device exports for all roles that have export access

**FR-6.3: Service Plan Account Type Restriction**
- Editing the `provider_account_type` restriction on service plans follows existing service plan edit permissions (no change)
- The validation layer (compatibility check between device account type and service plan restriction) is the guardrail — not role-based restriction on the service plan field
- Whoever can assign a service plan to a device today can continue to do so; the system enforces that only compatible plans can be assigned based on the device's account type

**FR-6.4: Permissions Summary**

| Action | Super Admin | Company Admin | Customer |
|--------|:-----------:|:-------------:|:--------:|
| Set/edit device account type (radio button) | Yes | No | No |
| Set device account type via import | Yes | No | No |
| View account type badge (device view) | Yes | Yes | Yes |
| View/filter account type column (device list) | Yes | Yes | Yes |
| Account type included in device export | Yes | Yes | Yes |
| Edit service plan account type restriction | Existing permissions — no change |
| Assign service plan to device (with validation) | Existing permissions — no change |

---

## Technical Design

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      WATM Portal (CakePHP)                  │
│                                                             │
│  ┌──────────────┐         ┌──────────────┐                │
│  │   Device     │         │  Service     │                │
│  │  Management  │         │   Plans      │                │
│  │              │         │              │                │
│  │ - Edit Form  │         │ - Edit Form  │                │
│  │ - View Page  │         │ - Filtering  │                │
│  │ - Import     │         │ - Dynamic    │                │
│  │              │         │   Behavior   │                │
│  └──────┬───────┘         └──────┬───────┘                │
│         │                        │                         │
│         ▼                        ▼                         │
│  ┌─────────────────────────────────────┐                  │
│  │   DevicesTable / ServicePlansTable  │                  │
│  │                                     │                  │
│  │  - Validation Rules                 │                  │
│  │  - beforeSave Hooks                 │                  │
│  │  - Account Compatibility Check      │                  │
│  │  - Partial Assignment Support       │                  │
│  └──────────────┬──────────────────────┘                  │
│                 │                                          │
│                 ▼                                          │
│  ┌──────────────────────────────────────┐                 │
│  │        VerizonApi Class              │                 │
│  │                                      │                 │
│  │  constructor($accountKey)            │                 │
│  │  getAccountKeyFromDevice($device)    │                 │
│  │  - Loads account config              │                 │
│  │  - Routes API calls                  │                 │
│  └───────┬────────────────┬─────────────┘                 │
│          │                │                               │
│          │                │                               │
└──────────┼────────────────┼───────────────────────────────┘
           │                │
           ▼                ▼
   ┌───────────────┐  ┌───────────────┐
   │   Verizon     │  │   Verizon     │
   │Regular Account│  │  FWA Account  │
   │    (PPU)      │  │  Unlimited    │
   │               │  │   Internet    │
   │ Pay-Per-Use   │  │   (FWA)       │
   │ Pricing       │  │               │
   └───────────────┘  └───────────────┘
```

## Testing Requirements

### Test Plan Overview

**Testing Phases:**
1. Unit Testing (PHPUnit - developer)
2. Integration Testing (End-to-end workflows - QA)
3. User Acceptance Testing (UAT - business stakeholders)
4. Regression Testing (Existing features - QA)

### Integration Test Cases

#### End-to-End Workflows

**Test Case IT-1: Import Devices with Mixed Accounts**
1. Create import file with 10 devices:
   - 7 devices with verizon_account_name = "regular"
   - 3 devices with verizon_account_name = "fwa"
2. Upload import file
3. Review import summary
4. **Verify:**
   - Import succeeds
   - Summary shows "Imported 10 devices: 7 Regular Account (PPU), 3 Unlimited Internet (FWA)"
   - Database correctly populated
   - All devices have verizon_account_name set

**Test Case IT-2: API Call Routes to Correct Account**
1. Create test device with verizon_account_name = 'fwa'
2. Change device status to trigger API call (e.g., Active → Suspended)
3. Monitor verizon-api.log file
4. **Verify:**
   - VerizonApi instantiated with 'fwa' account key
   - API call uses FWA credentials
   - Log entry includes "Called suspendDevice on fwa account"
   - API call succeeds
   - Device status updated

**Test Case IT-3: Service Plan Restriction Enforced**
1. Create service plan with verizon_account_name = 'fwa' (FWA only)
2. Create device with verizon_account_name = 'regular'
3. Edit device, attempt to assign FWA-only service plan
4. **Verify:**
   - Service plan not visible in dropdown OR
   - Validation blocks save with error message
   - Error explains account mismatch
   - Device unchanged

**Test Case IT-4: Bulk Device Assignment with Compatible Accounts**
1. Create 5 test devices all with verizon_account_name = 'fwa'
2. Navigate to Assign Device page
3. Select all 5 FWA devices
4. Observe service plan dropdown shows ALL service plans
5. Select FWA-compatible service plan
6. Click "Assign" button
7. **Verify:**
   - Service plan dropdown displays ALL service plans (no filtering)
   - Assignment succeeds for all 5 devices
   - Success message shows: "Successfully assigned {plan} to 5 devices"
   - All devices updated with service_plan_id
   - o_logs entry created for bulk assignment

**Test Case IT-5: Bulk Assignment with Partial Success (Mixed Accounts)**
1. Create 3 test devices: 2 with verizon_account_name = 'fwa', 1 with verizon_account_name = 'regular'
2. Create FWA-only service plan (verizon_account_name = 'fwa')
3. Navigate to Assign Device page
4. Select all 3 devices (mixed accounts)
5. Observe service plan dropdown shows ALL service plans (no filtering)
6. Select FWA-only plan
7. Click "Assign" button
8. **Verify:**
   - Service plan dropdown displays ALL service plans despite mixed account selection
   - 2 FWA devices assigned successfully
   - 1 regular device fails with error message
   - Error message lists serial number of failed device
   - Error explains: "Device {serial} is on Regular Account (PPU) but plan requires Unlimited Internet (FWA)"
   - Success message shows: "Successfully assigned to 2 devices. 1 device was incompatible."

**Test Case IT-6: Bulk Assignment with All Devices Types (No Dropdown Filtering)**
1. Create devices: 3 FWA devices, 2 regular devices
2. Create service plans: 1 FWA-only, 1 regular-only
3. Navigate to Assign Device page
4. **Test Scenario A: Select only FWA devices**
   - Select 3 FWA devices
   - Observe dropdown shows ALL service plans (FWA-only, regular-only)
   - Select FWA-only plan and assign
   - Verify: All 3 devices assigned successfully
5. **Test Scenario B: Select only regular devices**
   - Select 2 regular devices
   - Observe dropdown shows ALL service plans (no filtering)
   - Select regular-only plan and assign
   - Verify: All 2 devices assigned successfully
6. **Test Scenario C: Select mixed devices with FWA-only plan**
   - Select all 5 devices (mixed accounts)
   - Observe dropdown shows ALL service plans (no filtering)
   - Select FWA-only plan and assign
   - Verify: 3 FWA devices succeed, 2 regular devices fail with error (partial assignment)
7. **Test Scenario D: Select mixed devices with regular-only plan**
   - Select all 5 devices (mixed accounts)
   - Select regular-only plan and assign
   - Verify: 2 regular devices succeed, 3 FWA devices fail with error (partial assignment)

**Test Case IT-7: Dynamic Service Plan Form Behavior**
1. Navigate to service plan create/edit form
2. Select "Unlimited Internet (FWA) Only" for Verizon Account Restriction
3. **Verify:**
   - Usage Limits fields are grayed out
   - Device Group Names fields are grayed out
   - WiFi, Firewall, Show Usage checkboxes remain enabled
   - Help text explains FWA requirements
   - Form can be saved without grayed-out fields

**Test Case IT-8: Admin Updates Device Account (Tracking)**
1. Log in as authorized admin
2. Navigate to device with verizon_account_name = 'regular'
3. Edit device, change account to 'fwa'
4. Save device
5. **Verify:**
   - No confirmation dialog appears (only 3 authorized people)
   - Device saved successfully
   - device.verizon_account_name changed to 'fwa'
   - o_logs entry created with category DEVICE VERIZON ACCOUNT CHANGE
   - Success message displayed

### User Acceptance Test Cases

#### UAT-1: Operations Admin Configures New FWA Device
**Persona:** Operations Admin
**Scenario:** Company receives new device order for customer requesting unlimited plan. Admin needs to configure device for FWA account during initial setup.

**Steps:**
1. Navigate to Devices → Add Device
2. Fill in device details (manufacturer, model, serial, Verizon SIM)
3. Set "Verizon Account" selector to "Unlimited Internet (FWA)"
4. Set "Service Plan" to FWA unlimited plan
5. Save device

**Success Criteria:**
- ✅ Device created successfully
- ✅ Device view shows orange "Unlimited Internet (FWA)" badge
- ✅ Device appears in FWA filter on device list
- ✅ Subsequent API operations use FWA credentials
- ✅ Process feels intuitive and clear

#### UAT-2: Operations Admin Bulk Imports Mixed Devices
**Persona:** Operations Admin
**Scenario:** Quarterly device shipment arrives with 50 regular devices and 5 FWA devices. Admin needs to import all at once.

**Steps:**
1. Prepare import file with 55 devices
2. Set Column 16 (Verizon Account) to "regular" for 50 devices
3. Set Column 16 to "fwa" for 5 devices
4. Navigate to Devices → Import
5. Upload file
6. Review import summary
7. Confirm import

**Success Criteria:**
- ✅ All 55 devices imported successfully
- ✅ Import summary clearly shows "50 Regular Account (PPU), 5 Unlimited Internet (FWA)"
- ✅ Device list correctly shows account badges
- ✅ No errors or confusion
- ✅ Process faster than manual entry

#### UAT-3: Operations Admin Bulk Assignment with Account Compatibility
**Persona:** Operations Admin
**Scenario:** After importing 20 new FWA devices, admin needs to quickly assign all of them to the FWA unlimited service plan without doing each one individually.

**Steps:**
1. Log in as operations admin
2. Navigate to Devices → Device List
3. Filter by "Verizon Account: Unlimited Internet (FWA)"
4. Select all 20 FWA devices (checkboxes)
5. Click "Assign Device" action
6. Observe service plan dropdown shows ALL service plans
7. Select FWA unlimited plan
8. Review confirmation details
9. Click Save/Assign

**Success Criteria:**
- ✅ Can select multiple devices efficiently (checkboxes work smoothly)
- ✅ Service plan dropdown shows ALL service plans (no filtering)
- ✅ Assignment completes in single action (no need to repeat 20 times)
- ✅ Success message clearly indicates 20 devices assigned
- ✅ Process significantly faster than individual assignment
- ✅ Can verify all 20 devices now have correct service plan

#### UAT-4: Operations Admin Bulk Assignment with Mixed Account Devices (Partial Assignment)
**Persona:** Operations Admin
**Scenario:** Admin accidentally selects a mix of FWA and regular devices, then tries to assign an FWA-only plan. System should provide partial assignment with clear error message.

**Steps:**
1. Log in as operations admin
2. Navigate to Devices → Device List
3. Select 10 devices: 7 FWA devices + 3 regular devices (mixed)
4. Click "Assign Device" action
5. Observe service plan dropdown shows ALL service plans (no filtering)
6. Select FWA-only plan
7. Click "Assign"

**Success Criteria:**
- ✅ Service plan dropdown shows ALL service plans (no filtering based on mixed accounts)
- ✅ System allows admin to select any service plan
- ✅ Upon clicking "Assign", validation occurs and system assigns 7 compatible FWA devices
- ✅ Error message lists the 3 incompatible devices by serial number
- ✅ Success message shows: "Successfully assigned to 7 devices. 3 devices were incompatible."
- ✅ Admin understands which devices failed and why
- ✅ Can review failed devices and fix individually

### Regression Test Cases

**Test Case RT-1: Existing Devices Continue Functioning**
- **Verify:** Devices without verizon_account_name (NULL) continue working
- **Verify:** API calls for these devices use regular account
- **Verify:** Service plan assignment works as before
- **Verify:** No breaking changes to device edit/view

**Test Case RT-2: Legacy Service Plans Continue Working**
- **Verify:** Existing service plans continue functioning with new validation
- **Verify:** No breaking changes to service plan assignment workflow
- **Verify:** Service plan edit form requires account selection (FWA-only or Regular-only)

**Test Case RT-3: Device Import Without Account Column**
- **Verify:** Import template without Column 16 still works
- **Verify:** Devices imported without account default to 'regular'
- **Verify:** No errors or failures

**Test Case RT-4: Verizon API Operations Still Function**
- **Verify:** Device activation works on regular account
- **Verify:** Device suspension works on regular account
- **Verify:** Group assignment works on regular account
- **Verify:** Callback processing works correctly

**Test Case RT-5: Bulk Device Assignment Continues Working**
- **Verify:** Assign Device page still functions for devices without verizon_account_name
- **Verify:** Bulk assignment works for devices with NULL service plans
- **Verify:** Service plan dropdown shows all available plans
- **Verify:** No breaking changes to bulk assignment workflow

---

## Known Limitations and Phase 2 Considerations

### Mid-Cycle Account Switching Billing Gap (Phase 2)

**Issue:** When a device switches from Regular Account to Unlimited Internet (or vice versa) mid-billing cycle, the data usage accrued on the old account is not automatically billed.

**Volume:** 5-10 devices per month (low volume - manual approach acceptable)

**Current Approach:** Manual billing adjustments by operations team

**Workaround Options:**
1. **Restrict to 1st of billing cycle**: Only allow account changes on 1st of month
2. **Charge penalty/prorating fee**: Cover potential lost revenue with flat fee or prorated charge
3. **Continue manual billing adjustment**: Operations team manually reviews and adjusts invoices

**Phase 2 Solution:**
- Automated prorating and billing cycle split handling
- Reuse logic from existing customer-to-customer transfer process
- Calculate usage on old account through day of switch
- Calculate usage on new account from day of switch forward
- Generate accurate split invoices

**Decision Needed:** Choose workaround strategy before deployment (manual adjustment recommended for now)

**Trigger to Revisit:**
- Volume increases above 10-15 devices/month
- Manual process becomes error-prone or too time-consuming
- Customer complaints about billing accuracy

---
