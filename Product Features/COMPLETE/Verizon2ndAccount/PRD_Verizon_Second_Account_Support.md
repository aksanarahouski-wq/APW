# Product Requirements Document (PRD)
## Verizon Second Account Support - FWA Integration

**Document Version:** 1.0
**Date:** 2026-01-29
**Author:** Development Team
**Status:** Draft for Review
**Related Tickets:** [WATM-XXXX](https://orases.atlassian.net/browse/WATM-XXXX)
**Document Owner:** Aksana

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
13. [Open Questions](#open-questions)
14. [Document History](#document-history)

---

## Executive Summary

The WATM portal currently supports a single Verizon Business Internet account for managing devices with Verizon SIM cards. Verizon has mandated that Fixed Wireless Access (FWA) devices with unlimited plans must be managed through a separate account profile due to billing and pricing model differences. This PRD outlines the implementation of multi-account Verizon support to enable management of devices across both the original per-byte pricing account and the new FWA unlimited account.

The solution will store Verizon account configuration at the device level, allowing each device to be explicitly assigned to the appropriate account. Service plans will be configured with Verizon account restrictions, enabling enforcement of compatibility rules—FWA unlimited plans will only be available to devices on the FWA account, and regular per-byte plans only to devices on the regular account. This prevents billing errors and ensures compliance with Verizon's pricing tier separation. Customers will be able to upgrade devices to FWA unlimited plans (a one-way change), but will not be able to revert without administrative assistance, matching Verizon's billing restrictions.

### Key Features
- **Device-Level Verizon Account Assignment**: Each device explicitly configured with the correct Verizon account (default or FWA)
- **Service Plan Account Restrictions**: Service plans marked as compatible with specific Verizon accounts, preventing mismatched assignments
- **One-Way FWA Upgrade for Customers**: Customers can upgrade to unlimited FWA plans but cannot revert (requires admin override)
- **Bulk Import Support**: Import template includes Verizon account column for initial device configuration
- **API Call Routing**: All Verizon API calls automatically route to the correct account based on device configuration

### Business Impact
- **Compliance**: Enables compliance with Verizon's mandatory account separation for FWA devices
- **Revenue Opportunity**: Unlocks ability to offer unlimited FWA plans ($70-80+ MRC per device)
- **Billing Accuracy**: Prevents incompatible service plan/account pairings that could cause billing errors
- **Operational Efficiency**: Automates account routing, preventing manual errors and API failures
- **Customer Control**: Allows customers to self-service upgrade to unlimited plans while protecting against accidental downgrades

---

## Background and Problem Statement

### Current State

**Verizon API Integration:**
- WATM integrates with Verizon's M2M API for device activation, deactivation, suspension, and status management
- Single Verizon account credentials stored in global configuration (`config/app_local.php`)
- VerizonApi class instantiated without account parameter, always uses default credentials
- 15+ locations in codebase make Verizon API calls (device status changes, SIM management, callbacks, group assignments)

**Service Plan Structure:**
- Service plans are optional for devices (`service_plan_id` is nullable)
- Devices can exist without service plans (during import, RMA, initial setup)
- No current mechanism to differentiate devices by Verizon account
- **Service plans have no Verizon account association or restriction**
- **All service plans visible to all devices regardless of account compatibility**
- **No validation that service plan matches device's Verizon account requirements**

**Device Import Process:**
- Bulk import template includes Verizon SIM number and "SIM Active" status
- Import does NOT call Verizon API - only creates database records
- Devices imported without service plans assigned
- No validation of SIM ownership or account during import

**Device-to-Service-Plan Assignment:**
- **Device Edit Page**: Single device service plan assignment with dropdown selection
- **Assign Device Page**: Bulk assignment allowing multiple devices to be assigned to a service plan and company simultaneously
- No current validation that service plan is compatible with device's Verizon account in either workflow
- Bulk assignment can assign incompatible plans to entire batches of devices
- No filtering of service plans based on device Verizon account compatibility

### Problems

**Problem 1: Verizon-Mandated Account Separation**
- Verizon requires separate account profiles for Fixed Wireless Access (FWA) devices
- FWA devices use flat-rate unlimited plans ($70-80+ MRC) categorized as "Business Internet"
- Original account uses per-byte pricing with low MRC plans (usage-based billing)
- Verizon makes it difficult to switch between unlimited and per-byte plans (to combat SIM banking, network abuse, fraud)
- **Who is affected**: Operations team, customers requesting unlimited plans
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
- Regular per-byte service plans can only exist on the regular Verizon account
- No mechanism to prevent customers from assigning incompatible service plan/account combinations in either single device edit or bulk assignment workflows
- **Device Edit Page**: No validation when assigning service plan to single device
- **Assign Device Page (Bulk Assignment)**: Can assign incompatible service plans to entire batches of devices simultaneously, magnifying the impact of configuration errors
- System does not validate that a device's service plan matches its Verizon account
- Customers could accidentally select wrong service plan type for their device's account
- Admins could misconfigure service plans without knowing which account they require
- Bulk assignment workflow particularly risky - single mistake affects multiple devices
- **Who is affected**: Customers selecting service plans, operations team configuring plans (especially bulk operations), billing team
- **Business impact**:
  - Potential billing errors if device assigned to incompatible plan/account combination
  - **Bulk assignment errors can create widespread billing issues affecting many devices at once**
  - Customer confusion when seeing all service plans regardless of device account compatibility
  - Risk of API failures when device account doesn't match service plan expectations
  - Manual intervention required to fix incompatible assignments (time-consuming for bulk errors)
  - Inability to enforce Verizon's pricing tier separation (unlimited vs. per-byte)

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

**Customer Impact:**
- Customers requesting unlimited plans cannot be serviced
- Risk of devices being activated on wrong account with incorrect billing
- **Customer confusion seeing service plans they cannot actually use**
- **Accidental downgrades from FWA unlimited to per-byte plans**
- **Unexpected billing charges from incompatible plan assignments**

---

## Goals and Objectives

### Primary Goals

1. **Enable Multi-Account Verizon Support**: Implement device-level Verizon account configuration to support both regular (per-byte) and FWA (unlimited) accounts simultaneously
2. **Ensure API Call Routing**: Guarantee all Verizon API calls route to the correct account based on device configuration
3. **Implement Service Plan - Account Compatibility Enforcement**: Define service plans with required Verizon account (FWA or regular), validate compatibility, and prevent customers from assigning incompatible service plan/account combinations
4. **Support One-Way FWA Upgrades**: Allow customers to upgrade to unlimited FWA plans while preventing downgrades (matching Verizon restrictions)
5. **Maintain Backward Compatibility**: Ensure existing devices default to original account with no disruption

### Success Criteria

- ✅ All devices can be explicitly assigned to a Verizon account (default or FWA)
- ✅ All 15+ Verizon API call locations route to correct account based on device configuration
- ✅ Service plans can be restricted to specific Verizon accounts (FWA-only, regular-only, or unrestricted)
- ✅ System validates and prevents incompatible service plan/device account combinations
- ✅ Customers only see service plans compatible with their device's Verizon account
- ✅ Device import template includes Verizon account specification
- ✅ Customers can upgrade to FWA plans but cannot downgrade without admin approval
- ✅ Existing devices continue functioning with no disruption (default to original account)
- ✅ All account assignments and changes are logged for audit
- ✅ Zero API failures due to incorrect account credentials
- ✅ Zero billing errors from incompatible service plan/account pairings

### Non-Goals (Out of Scope)

- ❌ Support for more than 2 Verizon accounts (can be future enhancement)
- ❌ Automatic account determination based on SIM number patterns
- ❌ Real-time sync of device status from Verizon API on page load
- ❌ Migration of devices between accounts via UI (Phase 1 - admin override only)
- ❌ AT&T or T-Mobile multi-account support (Verizon only)
- ❌ Service plan automatic account assignment (explicit configuration required)

---

## Target Users

### Primary Users

**1. Operations Team / Admin Users**
- **Role**: Internal staff managing device inventory and configuration
- **Need**: Assign devices to correct Verizon account during setup and import
- **Pain Point**: Manual tracking of which devices are on FWA vs. regular account; no system support
- **Benefit**: Clear device-level configuration; bulk import support; 

**2. End Customers (Company Admins)**
- **Role**: External customers managing their company's device fleet
- **Need**: Upgrade devices to unlimited FWA plans when needed
- **Pain Point**: Cannot request unlimited plans; unclear which plans are available for which devices
- **Benefit**: Self-service FWA upgrade; clear service plan restrictions; prevented from accidental downgrades

---

## User Stories

### Epic 1: Device-Level Verizon Account Configuration

**Story 1.1: Assign Verizon Account to Device**
- **As an** Operations Admin
- **I want to** assign a Verizon account to a device when creating or editing it
- **So that** the correct account credentials are used for all API operations

**Acceptance Criteria:**
- Device edit form includes "Verizon Account" dropdown with options: "Default (empty)", "Regular Account", "FWA Account"
- Default selection is "Regular Account" (maps to original account)
- Selection is saved to device record
- Device view page displays Verizon account badge
- Device index table includes filterable "Verizon Account" column

**Story 1.2: Bulk Import with Verizon Account**
- **As an** Operations Admin
- **I want to** specify Verizon account in bulk device import
- **So that** new devices are configured correctly from the start

**Acceptance Criteria:**
- Import template includes Column 16: "Verizon Account"
- Accepts values: empty (defaults to "default"), "default", "fwa"
- Invalid values show error and prevent import
- Import summary shows count by account
- All imported devices have verizon_account_name populated

### Epic 2: Service Plan Account Restrictions

**Story 2.1: Configure Service Plan Account Restriction**
- **As an** Operations Admin
- **I want to** mark service plans as compatible with specific Verizon accounts
- **So that** customers only see appropriate service plans for their devices

**Acceptance Criteria:**
- Service plan edit form includes "Verizon Account Restriction" dropdown
- Options: "No Restriction", "Regular Account Only", "FWA Account Only"
- NULL/empty means no restriction (available to all devices)
- Service plan view page displays account restriction


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
- Service plan dropdown shows only plans compatible with ALL selected devices
- If selected devices have mixed Verizon accounts (some FWA, some regular), only show unrestricted service plans
- If all selected devices have same account, show plans for that account + unrestricted plans
- Validation occurs before save, checking each device's compatibility
- If any device incompatible, show error: "Cannot assign service plan: {count} device(s) incompatible. Serial numbers: {list}"
- Error message lists serial numbers of incompatible devices
- Assignment succeeds only if ALL devices compatible
- Success message shows count of devices assigned

### Epic 3: One-Way FWA Upgrade

**Story 3.1: Customer Upgrades Device to FWA Plan**
- **As a** Customer
- **I want to** upgrade my device to an unlimited FWA service plan
- **So that** I can use unlimited data

**Acceptance Criteria:**
- Customer can select FWA service plan for their device
- Confirmation prompt: "Upgrading to unlimited plan is permanent. You cannot revert without contacting support."
- On save, device.verizon_account_name automatically changes to 'fwa'
- Change is logged to o_logs: "Customer upgraded device to FWA unlimited plan"
- Customer receives success message confirming upgrade

**Story 3.2: Prevent Customer Downgrade from FWA**
- **As the** System
- **I want to** prevent customers from changing FWA devices back to regular plans
- **So that** Verizon's billing restrictions are enforced

**Acceptance Criteria:**
- If customer role attempts to change service plan from FWA → regular, validation blocks save
- Error message: "You cannot change from an unlimited plan back to a regular plan. Please contact support for assistance."
- Customer service plan dropdown does not show regular plans for FWA devices
- Admin users can override with special permission

**Story 3.3: Admin Override for FWA Reversion**
- **As an** Operations Admin with special permission
- **I want to** revert a device from FWA account to regular account
- **So that** I can manually process customer requests and correct errors

**Acceptance Criteria:**
- Admin users with "Devices.Devices.revertFromFwa" permission see override option
- Device edit form includes "Admin Override" checkbox for FWA devices
- Checking override allows changing verizon_account_name from 'fwa' to 'default'
- Allows assigning regular service plans to FWA devices
- Change logged to o_logs: "Admin reverted device from FWA to regular account" with reason field
- Requires confirmation: "This is a manual override of Verizon restrictions. Confirm reason: [text field]"

### Epic 4: API Call Routing

**Story 4.1: Route API Calls to Correct Account**
- **As the** System
- **I want to** automatically use the correct Verizon account credentials for API calls
- **So that** operations succeed without manual configuration

**Acceptance Criteria:**
- All 15+ Verizon API call locations updated to determine account from device
- VerizonApi class accepts account key in constructor
- If device.verizon_account_name is 'fwa', use FWA credentials
- If device.verizon_account_name is 'default' or NULL, use regular account credentials
- API call logging includes which account was used
- No hardcoded account selection - always determined from device configuration

**Story 4.2: Handle Missing Account Configuration**
- **As the** System
- **I want to** gracefully handle devices without explicit account configuration
- **So that** existing devices and edge cases continue functioning

**Acceptance Criteria:**
- If device.verizon_account_name is NULL or empty, default to 'default' account
- Log warning if API call made for device without explicit account
- Existing devices (before migration) continue using original account
- Import without account specified defaults to 'default'

---

## Scope

### In Scope

**Database Schema Changes**
- ✅ Add `verizon_account_name` column to `devices` table (VARCHAR 100, nullable, indexed)
- ✅ Add `verizon_account_name` column to `historical_service_plans` table (VARCHAR 100, nullable, indexed)
- ✅ Create migration to populate existing devices with 'default' account
- ✅ Create migration to set FWA devices (from provided list) to 'fwa' account

**Configuration Management**
- ✅ Multi-account configuration structure in app_local.php
- ✅ FWA account credentials configuration
- ✅ Account display names and descriptions

**VerizonApi Class Updates**
- ✅ Accept account key parameter in constructor
- ✅ Load account-specific credentials from configuration
- ✅ Helper method to determine account from device
- ✅ Update all 15+ API call locations to pass correct account

**UI Changes - Device Management**
- ✅ Device edit form: Verizon Account dropdown
- ✅ Device edit form: Service plan dropdown filtered by device account compatibility
- ✅ Device view page: Verizon Account badge/indicator
- ✅ Device index table: Verizon Account column (filterable)
- ✅ Device bulk update: Option to change Verizon account (admin only)
- ✅ Device import: Add Column 16 for Verizon account
- ✅ **Assign Device page: Validation prevents assigning incompatible service plans to batch of devices**
- ✅ **Assign Device page: Error message if attempting to assign incompatible plan to any device in batch**

**UI Changes - Service Plan Management**
- ✅ Service plan edit form: Verizon Account Restriction dropdown
- ✅ Service plan view page: Account restriction indicator
- ✅ Service plan index: Filter by account restriction

**Business Logic - Validation**
- ✅ Validate service plan compatibility with device account (single device edit)
- ✅ **Validate service plan compatibility for bulk device assignment (Assign Device page)**
- ✅ **Batch validation: Check all devices in assignment for service plan compatibility**
- ✅ Prevent customer downgrade from FWA to regular plans
- ✅ Validate Verizon account name during import (only 'default' or 'fwa')
- ✅ Validation error messages with clear explanations
- ✅ **Bulk assignment error messages identify incompatible devices by serial number**

**Business Logic - Admin Override**
- ✅ Permission: `Devices.Devices.revertFromFwa`
- ✅ Admin override checkbox for FWA device reversion
- ✅ Confirmation prompt with reason field
- ✅ Override logging with reason to o_logs

**Audit Logging ideas (TBD)**
- ✅ Log device Verizon account assignment/changes
- ✅ Log customer FWA upgrades
- ✅ Log admin FWA reversions with reason
- ✅ Log API calls with account used (for debugging)
- ✅ Log import with account distribution summary

**Reporting**
- ✅ Device list export includes Verizon account column
- ✅ Filter devices by Verizon account in all device lists

### Out of Scope (Future Enhancements)

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
- Field SHALL accept values: NULL, 'default', 'fwa', or account number

**FR-1.2: Device Edit Form Account Selection**
- Device edit form SHALL include "Verizon Account" dropdown
- Dropdown options SHALL be: "Use Default Account (empty)", "Regular Account (Per-Byte)", "FWA Account (Unlimited)"
- Form SHALL save selected value to device.verizon_account_name
- Empty selection SHALL save NULL to database
- Help text SHALL explain: "Select FWA account only for customer-requested unlimited devices. Cannot be changed by customers after activation."

**FR-1.3: Device View Account Display**
- Device view page SHALL display Verizon account as badge
- Badge SHALL be color-coded: Blue for "Regular Account", Orange for "FWA Account", Gray for "Default (unspecified)"
- Badge SHALL be prominent and easy to identify
- Tooltip SHALL show full account details

**FR-1.4: Device List Account Column**
- Device index table SHALL include "Verizon Account" column
- Column SHALL be filterable (dropdown: All, Regular, FWA, Unspecified)
- Column SHALL be sortable
- Column SHALL display badge matching device view style

**FR-1.5: Device Import Account Column**
- Import template SHALL include Column 16: "Verizon Account"
- Column SHALL accept: empty (defaults to 'default'), "default", "fwa"
- Invalid values SHALL trigger validation error with clear message
- Import parser SHALL validate and normalize values
- Devices imported without account SHALL default to 'default'
- Import summary SHALL show count by account: "Imported 50 devices: 45 Regular, 5 FWA"

### FR-2: Service Plan Verizon Account Restrictions

**FR-2.1: Service Plan Account Restriction Field**
- System SHALL add `verizon_account_name` field to historical_service_plans table
- Field SHALL be VARCHAR(100), nullable, with default value NULL
- Field SHALL have index for performance
- Field SHALL accept values: NULL, 'default', 'fwa'
- NULL SHALL mean "no restriction - available to all devices"

**FR-2.2: Service Plan Edit Form Restriction Selection**
- Service plan edit form SHALL include "Verizon Account Restriction" dropdown
- Dropdown options SHALL be: "No Restriction", "Regular Account Only", "FWA Account Only"
- Empty/NULL SHALL mean no restriction
- Form SHALL save to historical_service_plans.verizon_account_name
- Help text SHALL explain: "Restricts which devices can use this service plan based on their Verizon account"

**FR-2.3: Service Plan View Restriction Display**
- Service plan view page SHALL display account restriction if set
- Display SHALL be prominent badge near plan name
- Badge SHALL match device account badge styling

**FR-2.4: Service Plan Dropdown Filtering**
- Device edit form service plan dropdown SHALL filter by device account compatibility
- If device.verizon_account_name = 'fwa', show only plans with verizon_account_name = 'fwa' OR NULL
- If device.verizon_account_name = 'default' OR NULL, show only plans with verizon_account_name = 'default' OR NULL
- Tooltip SHALL explain when plans are filtered: "Only showing plans compatible with this device's Verizon account"

**FR-2.5: Service Plan Compatibility Validation**
- System SHALL validate service plan compatibility on device save
- If service plan has account restriction AND device has account configured:
  - If they match OR plan has no restriction → allow save
  - If they don't match → block save with error
- Error message SHALL be: "This service plan requires Verizon {plan_account} account, but device is configured for {device_account} account."
- Validation SHALL apply to: manual edits, API calls, bulk updates

**FR-2.6: Bulk Device Assignment Filtering and Validation (Assign Device Page)**
- Assign Device page SHALL allow selecting multiple devices for bulk service plan assignment
- Service plan dropdown SHALL intelligently filter based on selected devices:
  - If ALL selected devices have same verizon_account_name (e.g., all 'fwa'), show plans for that account + unrestricted plans
  - If selected devices have MIXED accounts (some 'fwa', some 'default'), show ONLY unrestricted plans (no account restriction)
  - If ANY selected device has NULL account, treat as 'default'
- System SHALL validate compatibility for EACH device in batch before save
- If ANY device incompatible with selected service plan:
  - Block entire batch assignment (all-or-nothing)
  - Error message SHALL list incompatible devices: "Cannot assign service plan: {count} device(s) are incompatible with this plan. Incompatible devices: {serial1}, {serial2}, {serial3}..."
  - Error message SHALL specify reason if possible: "Device {serial} is on {account} account but plan requires {plan_account} account"
- On successful batch assignment:
  - Success message SHALL show count: "Successfully assigned {plan_name} to {count} devices"
  - o_logs entry SHALL record bulk assignment with device count
- Tooltip SHALL explain filtering: "Service plans filtered based on selected devices' Verizon accounts. {count} devices selected: {breakdown}"
  - Breakdown example: "5 FWA devices, 3 Regular devices - showing unrestricted plans only"

### FR-3: Customer FWA Upgrade and Downgrade Prevention

**FR-3.1: Customer FWA Upgrade Flow**
- Customer users SHALL be able to assign FWA service plans to their devices
- When customer selects FWA service plan, system SHALL show confirmation modal
- Confirmation message SHALL be: "Upgrading to an unlimited plan is a permanent change. You cannot revert to a per-byte plan without contacting support. Continue?"
- On confirmation and save:
  - System SHALL automatically change device.verizon_account_name to 'fwa'
  - System SHALL log to o_logs: "Customer upgraded device {serial} to FWA unlimited plan (one-way change)"
  - System SHALL show success message: "Device upgraded to unlimited plan successfully"

**FR-3.2: Customer FWA Downgrade Prevention**
- System SHALL detect when customer user attempts to change from FWA service plan to regular service plan
- System SHALL block save operation
- Error message SHALL be: "You cannot change from an unlimited plan back to a regular plan. Please contact support for assistance."
- Customer users SHALL NOT see regular account service plans in dropdown for FWA devices

**FR-3.3: Admin Override Permission**
- System SHALL define permission: "Devices.Devices.revertFromFwa"
- Only admin users with this permission SHALL see override options
- Admin users WITH permission SHALL see "Admin Override" checkbox on device edit form for FWA devices
- Checkbox label SHALL be: "Allow reverting to regular account (admin override)"

**FR-3.4: Admin FWA Reversion Flow**
- When admin checks override checkbox, system SHALL show reason field
- Reason field SHALL be required to proceed
- Confirmation modal SHALL warn: "This is a manual override of Verizon restrictions. This may require manual processes with Verizon. Document reason:"
- On save:
  - System SHALL allow changing verizon_account_name from 'fwa' to 'default'
  - System SHALL allow assigning regular service plans
  - System SHALL log to o_logs: "Admin reverted device {serial} from FWA to regular account. Reason: {reason}"
  - System SHALL show success message: "Device reverted to regular account. Manual Verizon processes may be required."

### FR-4: Verizon API Call Routing

**FR-4.1: VerizonApi Account Configuration**
- VerizonApi constructor SHALL accept optional `$accountKey` parameter (default: 'default')
- Constructor SHALL load credentials from Configure::read("Verizon.accounts.{$accountKey}")
- If account key not found, SHALL fall back to 'default' account
- Constructor SHALL log warning if fallback occurs

**FR-4.2: Device Account Determination Helper**
- VerizonApi class SHALL provide static method: `getAccountKeyFromDevice(Device $device): string`
- Method SHALL return:
  - 'fwa' if device.verizon_account_name = 'fwa'
  - 'default' if device.verizon_account_name = 'default' OR NULL
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
- If device has no verizon_account_name (NULL), SHALL default to 'default' account
- If device.verizon_account_name has invalid value, SHALL default to 'default' account and log warning
- Warning SHALL include: device_id, serial_number, invalid value
- System SHALL never fail API call due to missing account configuration

### FR-5: Data Validation and Integrity

**FR-5.1: Import Validation**
- Import parser SHALL validate Verizon account values
- Valid values: empty, "default", "fwa" (case-insensitive)
- Invalid values SHALL show error: "Device with serial number '{serial}': Invalid Verizon account '{value}'. Must be 'default' or 'fwa'."
- Invalid rows SHALL be skipped, import continues with remaining valid rows
- Import summary SHALL list skipped rows with reasons

**FR-5.2: Bulk Update Validation**
- Bulk update operations SHALL validate account values before applying
- Validation SHALL check:
  - Account value is valid ('default', 'fwa', or NULL)
  - If changing to 'fwa', device has verizon_sim_number
  - If changing from 'fwa', user has override permission (if customer role)
- Bulk update SHALL show preview of changes before applying
- Preview SHALL include warnings for risky changes

**FR-5.3: Configuration Validation**
- System SHALL validate Verizon account configuration on application startup
- Required fields for each account: key, secret, username, password, accountName
- Missing required fields SHALL log error and use fallback configuration
- Invalid configuration SHALL not prevent application startup (graceful degradation)

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
│  │ - Import     │         │              │                │
│  └──────┬───────┘         └──────┬───────┘                │
│         │                        │                         │
│         ▼                        ▼                         │
│  ┌─────────────────────────────────────┐                  │
│  │   DevicesTable / ServicePlansTable  │                  │
│  │                                     │                  │
│  │  - Validation Rules                 │                  │
│  │  - beforeSave Hooks                 │                  │
│  │  - Account Compatibility Check      │                  │
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
└──────────┼────────────────┼───────────────────────────────┘
           │                │
           ▼                ▼
   ┌───────────────┐  ┌───────────────┐
   │   Verizon     │  │   Verizon     │
   │Regular Account│  │  FWA Account  │
   │               │  │               │
   │ Per-Byte      │  │  Unlimited    │
   │ Pricing       │  │   Plans       │
   └───────────────┘  └───────────────┘
```

### Data Model Changes

**Migration 1: Add verizon_account_name to devices**
```sql
-- Add column to devices table
ALTER TABLE devices
ADD COLUMN verizon_account_name VARCHAR(100) NULL DEFAULT NULL
COMMENT 'Verizon account identifier (default, fwa, or account number)'
AFTER vz_group_applied;

-- Create index for filtering and searching
CREATE INDEX idx_devices_verizon_account
ON devices(verizon_account_name);

-- Populate existing devices with 'default' account
UPDATE devices
SET verizon_account_name = 'default'
WHERE verizon_sim_number IS NOT NULL
  AND verizon_account_name IS NULL;
```

**Migration 2: Add verizon_account_name to historical_service_plans**
```sql
-- Add column to historical_service_plans table
ALTER TABLE historical_service_plans
ADD COLUMN verizon_account_name VARCHAR(100) NULL DEFAULT NULL
COMMENT 'Required Verizon account for devices using this service plan'
AFTER tmobile_provider_group;

-- Create index for filtering
CREATE INDEX idx_historical_service_plans_verizon_account
ON historical_service_plans(verizon_account_name);

-- Note: NULL means no restriction - plan available to all devices
-- Specific values ('default', 'fwa') restrict plan to those devices
```

**Migration 3: Set FWA devices (from client-provided list)**
```sql
-- Update specific devices to FWA account
-- Replace IDs with actual device IDs from client
UPDATE devices
SET verizon_account_name = 'fwa'
WHERE id IN (123, 456, 789); -- Client to provide list of 10-15 FWA device IDs
```

**Entity Updates:**

**Device.php:**
```php
/**
 * @property string|null $verizon_account_name
 */
class Device extends Entity
{
    protected $_accessible = [
        // ... existing fields
        'verizon_account_name' => true,
    ];
}
```

**HistoricalServicePlan.php:**
```php
/**
 * @property string|null $verizon_account_name
 */
class HistoricalServicePlan extends Entity
{
    protected $_accessible = [
        // ... existing fields
        'verizon_account_name' => true,
    ];
}
```

### Code Changes

**Configuration: config/app_local.php**
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
            'description' => 'Original Verizon M2M account for usage-based billing',
        ],
        'fwa' => [
            'key' => 'eb64fb35-0b07-4636-9278-258045cdde5e',
            'secret' => '54c3c486-ac0f-45a0-bcb5-9b1171f2a11c',
            'username' => 'Orases5gAPI',
            'password' => '2AerKID*90Okdjh3@!',
            'accountName' => '542647743-00001',
            'display_name' => 'FWA Account (Unlimited Plans)',
            'description' => 'Fixed Wireless Access account for unlimited flat-rate plans',
        ],
    ],
    'default_account' => 'default', // Fallback if device has no account specified

    // Legacy single-account config (deprecated, kept for backward compatibility)
    'key' => env('VERIZON_DEFAULT_KEY'),
    'secret' => env('VERIZON_DEFAULT_SECRET'),
    'username' => env('VERIZON_DEFAULT_USERNAME'),
    'password' => env('VERIZON_DEFAULT_PASSWORD'),
    'accountName' => env('VERIZON_DEFAULT_ACCOUNT_NAME'),
],
```

**VerizonApi Class Updates: plugins/Devices/src/Util/VerizonApi.php**
```php
class VerizonApi
{
    private Client $httpClient;
    private string $key;
    private string $secret;
    private string $username;
    private string $password;
    private string $accountName;
    private string $cacheConfig = 'default';
    private bool $enableCarrierDeviceUpdates;

    /**
     * Constructor
     *
     * @param string $accountKey Account configuration key ('default', 'fwa', etc.)
     */
    public function __construct(string $accountKey = 'default')
    {
        $accounts = Configure::read('Verizon.accounts');

        // Fallback to default account if key not found
        if (!isset($accounts[$accountKey])) {
            Log::warning("Verizon account key '{$accountKey}' not found, using default account");
            $accountKey = Configure::read('Verizon.default_account', 'default');
        }

        $accountConfig = $accounts[$accountKey];

        $this->key = $accountConfig['key'];
        $this->secret = $accountConfig['secret'];
        $this->username = $accountConfig['username'];
        $this->password = $accountConfig['password'];
        $this->accountName = $accountConfig['accountName'];

        $this->enableCarrierDeviceUpdates = Configure::read('Device.enableCarrierApiDeviceUpdates');
        $this->httpClient = new Client([
            'host' => 'thingspace.verizon.com',
            'scheme' => 'https',
        ]);
    }

    /**
     * Get Verizon account key from device configuration
     *
     * @param Device $device Device entity
     * @return string Account key ('default', 'fwa', etc.)
     */
    public static function getAccountKeyFromDevice(Device $device): string
    {
        // If device has explicit account configuration, use it
        if (!empty($device->verizon_account_name)) {
            return $device->verizon_account_name;
        }

        // Otherwise, fall back to default account
        return Configure::read('Verizon.default_account', 'default');
    }

    // ... rest of existing methods unchanged
}
```

**DevicesTable Validation: plugins/Devices/src/Model/Table/DevicesTable.php**
```php
public function buildRules(RulesChecker $rules): RulesChecker
{
    // ... existing rules

    // Validate service plan is compatible with device's Verizon account
    $rules->add(
        function ($entity, $options) {
            // Skip if no service plan assigned
            if (empty($entity->service_plan_id)) {
                return true;
            }

            /** @var \Devices\Model\Table\ServicePlansTable $servicePlansTable */
            $servicePlansTable = $this->ServicePlans;
            $servicePlan = $servicePlansTable->get($entity->service_plan_id, [
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
        'verizonAccountServicePlanCompatibility',
        [
            'errorField' => 'service_plan_id',
            'message' => 'Service plan is not compatible with device Verizon account'
        ]
    );

    return $rules;
}

public function beforeSave(EventInterface $event, EntityInterface $entity, ArrayObject $options)
{
    // ... existing beforeSave logic

    // Prevent customer downgrade from FWA to regular account
    if ($entity->isDirty('service_plan_id') && !$entity->isNew()) {
        $identity = Router::getRequest()?->getAttribute('identity');
        $isCustomer = $identity?->get('role') === 'customer';

        if ($isCustomer) {
            $oldPlanId = $entity->getOriginal('service_plan_id');
            $newPlanId = $entity->service_plan_id;

            if (!empty($oldPlanId) && !empty($newPlanId)) {
                /** @var \Devices\Model\Table\ServicePlansTable $servicePlansTable */
                $servicePlansTable = $this->ServicePlans;

                $oldPlan = $servicePlansTable->get($oldPlanId, [
                    'contain' => 'LastApprovedHistoricalServicePlans'
                ]);
                $newPlan = $servicePlansTable->get($newPlanId, [
                    'contain' => 'LastApprovedHistoricalServicePlans'
                ]);

                $oldAccount = $oldPlan->last_approved_historical_service_plan->verizon_account_name ?? 'default';
                $newAccount = $newPlan->last_approved_historical_service_plan->verizon_account_name ?? 'default';

                // Block customer from reverting FWA → Regular
                if ($oldAccount === 'fwa' && $newAccount === 'default') {
                    $entity->setError('service_plan_id', [
                        'fwa_revert_blocked' => __('You cannot change from an unlimited plan back to a regular plan. Please contact support for assistance.')
                    ]);
                    return false;
                }

                // Log FWA upgrade by customer
                if ($oldAccount === 'default' && $newAccount === 'fwa') {
                    // Auto-update device Verizon account
                    $entity->verizon_account_name = 'fwa';

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

    // ... rest of existing beforeSave logic
}
```

**Update All VerizonApi Instantiations (15+ locations):**

Example pattern to apply everywhere:
```php
// OLD CODE:
$vzApi = new VerizonApi();

// NEW CODE:
$accountKey = VerizonApi::getAccountKeyFromDevice($device);
$vzApi = new VerizonApi($accountKey);
```

**Affected files:**
1. `plugins/Devices/src/Model/Table/DevicesTable.php` (lines 1330, 1698, etc.)
2. `plugins/Devices/src/Model/Table/DeviceSimStatusesTable.php` (lines 348, 472, 608)
3. `plugins/Devices/src/Controller/Api/VerizonCallbacksController.php` (multiple)
4. `plugins/Devices/src/Command/TestVerizonCommand.php`
5. `plugins/Devices/src/Command/ClearUpVerizonDeviceStatusesCommand.php`
6. Any other files instantiating VerizonApi

### Integration Points

**Verizon M2M API:**
- Endpoints: thingspace.verizon.com/api/m2m/v1/
- Authentication: OAuth2 + session token
- Methods used: activateDevice, deactivateDevice, suspendDevice, resumeDevice, assignGroupToDevice, getDeviceInfo
- Impact: API calls now route to correct account based on device configuration

**Device Import Process:**
- Import template gains new column
- Parser updated to handle verizon_account_name
- No breaking changes - column optional

**Service Plan Management:**
- Service plans gain account restriction field
- Filtering logic added to device edit forms
- No breaking changes - restriction optional (NULL = no restriction)

**Audit Logging (o_logs table):**
- New log categories: DEVICE FWA UPGRADE, DEVICE FWA REVERT
- Logs include old/new account values
- API call logging includes account used

### Technical Constraints

**Performance:**
- Indexes added to support filtering by verizon_account_name
- Service plan filtering adds JOIN but limited to active plans
- No significant performance impact expected

**Scalability:**
- Solution supports 2 accounts currently
- Can extend to N accounts by adding to configuration
- No hard-coded limits

**Security:**
- Account credentials stored in app_local.php (not database)
- Credentials not exposed to frontend
- API call logging does not include credentials
- Permission required for FWA reversion

**Backward Compatibility:**
- Existing devices default to 'default' account (migration)
- NULL verizon_account_name treated as 'default'
- Service plans with NULL restriction work for all devices
- No breaking changes to existing functionality

---

## Testing Requirements

### Test Plan Overview

**Testing Phases:**
1. Unit Testing (PHPUnit - developer)
2. Integration Testing (End-to-end workflows - QA)
3. User Acceptance Testing (UAT - business stakeholders)
4. Regression Testing (Existing features - QA)

### Unit Test Cases

**UT-1: VerizonApi Account Configuration**
- Test VerizonApi constructor with 'default' account key loads correct credentials
- Test VerizonApi constructor with 'fwa' account key loads correct credentials
- Test VerizonApi constructor with invalid account key falls back to default
- Test VerizonApi::getAccountKeyFromDevice() with various device configurations

**UT-2: Device Validation Rules**
- Test service plan compatibility validation (matching accounts → pass)
- Test service plan compatibility validation (mismatched accounts → fail)
- Test service plan compatibility validation (no restriction → pass)
- Test customer FWA downgrade blocked
- Test admin FWA downgrade allowed with permission

**UT-3: Service Plan Filtering**
- Test getAvailableServicePlansForDevice() with regular account device
- Test getAvailableServicePlansForDevice() with FWA account device
- Test getAvailableServicePlansForDevice() with no account specified

### Integration Test Cases

#### End-to-End Workflows

**Test Case IT-1: Import Devices with Mixed Accounts**
1. Create import file with 10 devices:
   - 7 devices with verizon_account_name = "default"
   - 3 devices with verizon_account_name = "fwa"
2. Upload import file
3. Review import summary
4. **Verify:**
   - Import succeeds
   - Summary shows "Imported 10 devices: 7 Regular, 3 FWA"
   - Database correctly populated
   - All devices have verizon_account_name set

**Test Case IT-2: Customer Upgrades Device to FWA Plan**
1. Log in as customer user
2. Navigate to device with regular account
3. Edit device, select FWA service plan
4. Confirm upgrade warning
5. Save device
6. **Verify:**
   - Device saved successfully
   - device.verizon_account_name changed to 'fwa'
   - service_plan_id updated
   - o_logs entry created with category DEVICE FWA UPGRADE
   - Success message displayed

**Test Case IT-3: Customer Blocked from FWA Downgrade**
1. Log in as customer user
2. Navigate to device with FWA account and FWA service plan
3. Edit device, attempt to select regular service plan
4. **Verify:**
   - Regular service plans not in dropdown OR
   - If API/direct access, validation blocks save
   - Error message displayed
   - Device unchanged

**Test Case IT-4: Admin Reverts FWA Device to Regular**
1. Log in as admin user with revertFromFwa permission
2. Navigate to device with FWA account
3. Edit device, check "Admin Override" checkbox
4. Enter reversion reason
5. Change to regular service plan
6. Confirm warning
7. Save device
8. **Verify:**
   - Device saved successfully
   - device.verizon_account_name changed to 'default'
   - service_plan_id updated
   - o_logs entry created with category DEVICE FWA REVERT and reason
   - Success message with manual process warning

**Test Case IT-5: API Call Routes to Correct Account**
1. Create test device with verizon_account_name = 'fwa'
2. Change device status to trigger API call (e.g., Active → Suspended)
3. Monitor verizon-api.log file
4. **Verify:**
   - VerizonApi instantiated with 'fwa' account key
   - API call uses FWA credentials
   - Log entry includes "Called suspendDevice on fwa account"
   - API call succeeds
   - Device status updated

**Test Case IT-6: Service Plan Restriction Enforced**
1. Create service plan with verizon_account_name = 'fwa' (FWA only)
2. Create device with verizon_account_name = 'default'
3. Edit device, attempt to assign FWA-only service plan
4. **Verify:**
   - Service plan not visible in dropdown OR
   - Validation blocks save with error message
   - Error explains account mismatch
   - Device unchanged

**Test Case IT-7: Bulk Device Assignment with Compatible Accounts**
1. Create 5 test devices all with verizon_account_name = 'fwa'
2. Navigate to Assign Device page
3. Select all 5 FWA devices
4. Observe service plan dropdown
5. Select FWA-compatible service plan
6. Save assignment
7. **Verify:**
   - Service plan dropdown shows FWA plans + unrestricted plans
   - Assignment succeeds for all 5 devices
   - Success message shows: "Successfully assigned {plan} to 5 devices"
   - All devices updated with service_plan_id
   - o_logs entry created for bulk assignment

**Test Case IT-8: Bulk Assignment Blocked for Mixed Accounts**
1. Create 3 test devices: 2 with verizon_account_name = 'fwa', 1 with verizon_account_name = 'default'
2. Create FWA-only service plan (verizon_account_name = 'fwa')
3. Navigate to Assign Device page
4. Select all 3 devices
5. Observe service plan dropdown
6. Attempt to select FWA-only plan (via API if not visible)
7. **Verify:**
   - Service plan dropdown shows ONLY unrestricted plans (no account-specific plans)
   - If FWA plan somehow selected, validation blocks save
   - Error message lists incompatible device serial number
   - Error explains: "Device {serial} is on default account but plan requires fwa account"
   - No devices updated (all-or-nothing validation)

**Test Case IT-9: Bulk Assignment Pre-Filtering Logic**
1. Create devices: 3 FWA devices, 2 regular devices
2. Create service plans: 1 FWA-only, 1 regular-only, 1 unrestricted
3. Navigate to Assign Device page
4. **Test Scenario A: Select only FWA devices**
   - Select 3 FWA devices
   - Observe dropdown shows: FWA-only + unrestricted (not regular-only)
5. **Test Scenario B: Select only regular devices**
   - Select 2 regular devices
   - Observe dropdown shows: regular-only + unrestricted (not FWA-only)
6. **Test Scenario C: Select mixed devices**
   - Select all 5 devices (mixed accounts)
   - Observe dropdown shows: unrestricted only (no account-specific)
7. **Verify:**
   - Filtering logic matches documented requirements
   - Tooltip explains filtering with device breakdown
   - UI updates dynamically as selection changes

### User Acceptance Test Cases

#### UAT-1: Operations Admin Configures New FWA Device
**Persona:** Operations Admin
**Scenario:** Company receives new device order for customer requesting unlimited plan. Admin needs to configure device for FWA account during initial setup.

**Steps:**
1. Navigate to Devices → Add Device
2. Fill in device details (manufacturer, model, serial, Verizon SIM)
3. Set "Verizon Account" dropdown to "FWA Account (Unlimited)"
4. Set "Service Plan" to FWA unlimited plan
5. Save device

**Success Criteria:**
- ✅ Device created successfully
- ✅ Device view shows orange "FWA Account" badge
- ✅ Device appears in FWA filter on device list
- ✅ Subsequent API operations use FWA credentials
- ✅ Process feels intuitive and clear

#### UAT-2: Operations Admin Bulk Imports Mixed Devices
**Persona:** Operations Admin
**Scenario:** Quarterly device shipment arrives with 50 regular devices and 5 FWA devices. Admin needs to import all at once.

**Steps:**
1. Prepare import file with 55 devices
2. Set Column 16 (Verizon Account) to "default" for 50 devices
3. Set Column 16 to "fwa" for 5 devices
4. Navigate to Devices → Import
5. Upload file
6. Review import summary
7. Confirm import

**Success Criteria:**
- ✅ All 55 devices imported successfully
- ✅ Import summary clearly shows "50 Regular, 5 FWA"
- ✅ Device list correctly shows account badges
- ✅ No errors or confusion
- ✅ Process faster than manual entry

#### UAT-3: Customer Self-Service FWA Upgrade
**Persona:** End Customer (Company Admin)
**Scenario:** Customer has device on regular plan but needs unlimited data for a project. Wants to upgrade without contacting support.

**Steps:**
1. Log in as customer
2. Navigate to My Devices
3. Click Edit on target device
4. Change Service Plan to unlimited FWA plan
5. Read confirmation warning
6. Confirm upgrade
7. Save device

**Success Criteria:**
- ✅ Upgrade completes successfully without support intervention
- ✅ Warning message is clear and understandable
- ✅ Device badge changes to reflect FWA account
- ✅ Billing updates appropriately (out of scope for this test)
- ✅ Customer understands change is permanent

#### UAT-4: Customer Prevented from Accidental Downgrade
**Persona:** End Customer (Company Admin)
**Scenario:** Customer has FWA device but mistakenly tries to change to regular plan.

**Steps:**
1. Log in as customer
2. Navigate to device with FWA unlimited plan
3. Click Edit
4. Observe service plan dropdown
5. Attempt to select regular plan (if visible via API/direct URL)

**Success Criteria:**
- ✅ Regular plans not shown in dropdown
- ✅ If somehow accessed, validation prevents save
- ✅ Error message is clear and helpful
- ✅ Directs customer to contact support
- ✅ Device remains on FWA plan

#### UAT-5: Operations Admin Bulk Assignment with Account Compatibility
**Persona:** Operations Admin
**Scenario:** After importing 20 new FWA devices, admin needs to quickly assign all of them to the FWA unlimited service plan without doing each one individually.

**Steps:**
1. Log in as operations admin
2. Navigate to Devices → Device List
3. Filter by "Verizon Account: FWA"
4. Select all 20 FWA devices (checkboxes)
5. Click "Assign Device" action
6. Observe service plan dropdown options
7. Select FWA unlimited plan
8. Review confirmation details
9. Click Save/Assign

**Success Criteria:**
- ✅ Can select multiple devices efficiently (checkboxes work smoothly)
- ✅ Service plan dropdown shows only compatible plans (FWA + unrestricted)
- ✅ Assignment completes in single action (no need to repeat 20 times)
- ✅ Success message clearly indicates 20 devices assigned
- ✅ Process significantly faster than individual assignment
- ✅ No errors or confusion about which plans are available
- ✅ Can verify all 20 devices now have correct service plan

#### UAT-6: Operations Admin Protected from Bulk Assignment Errors
**Persona:** Operations Admin
**Scenario:** Admin accidentally selects a mix of FWA and regular devices, then tries to assign an FWA-only plan. System should prevent this error before it causes widespread issues.

**Steps:**
1. Log in as operations admin
2. Navigate to Devices → Device List
3. Select 10 devices: 7 FWA devices + 3 regular devices (mixed)
4. Click "Assign Device" action
5. Observe service plan dropdown
6. Attempt to assign FWA-only plan (if visible)

**Success Criteria:**
- ✅ Service plan dropdown shows ONLY unrestricted plans (no FWA-only or regular-only)
- ✅ Tooltip explains: "Selected devices have mixed accounts, showing unrestricted plans only"
- ✅ If FWA-only plan somehow selected, validation blocks with clear error
- ✅ Error message lists the 3 incompatible devices by serial number
- ✅ No devices updated (all-or-nothing protection)
- ✅ Admin understands they need to separate devices by account first
- ✅ UI guides admin toward correct action rather than just blocking

### Regression Test Cases

**Test Case RT-1: Existing Devices Continue Functioning**
- **Verify:** Devices without verizon_account_name (NULL) continue working
- **Verify:** API calls for these devices use default account
- **Verify:** Service plan assignment works as before
- **Verify:** No breaking changes to device edit/view

**Test Case RT-2: Service Plans Without Restrictions Work for All**
- **Verify:** Service plans with NULL verizon_account_name appear for all devices
- **Verify:** Both regular and FWA devices can select unrestricted plans
- **Verify:** No validation errors for unrestricted plans

**Test Case RT-3: Device Import Without Account Column**
- **Verify:** Import template without Column 16 still works
- **Verify:** Devices imported without account default to 'default'
- **Verify:** No errors or failures

**Test Case RT-4: Verizon API Operations Still Function**
- **Verify:** Device activation works on default account
- **Verify:** Device suspension works on default account
- **Verify:** Group assignment works on default account
- **Verify:** Callback processing works correctly

**Test Case RT-5: Bulk Device Assignment Continues Working**
- **Verify:** Assign Device page still functions for devices without verizon_account_name
- **Verify:** Bulk assignment works for devices with NULL service plans
- **Verify:** Service plan dropdown shows all unrestricted plans for legacy devices
- **Verify:** No breaking changes to bulk assignment workflow

---

## Dependencies and Risks

### Dependencies

**Internal Dependencies:**

1. **Orases Logging Package**
   - Required for audit logging (o_logs table)
   - **Mitigation:** Standard package, already integrated

2. **CakePHP Queue Plugin**
   - Not directly required but API callbacks may use queues
   - **Mitigation:** Existing dependency, no new requirements

3. **Client-Provided FWA Device List**
   - Need list of 10-15 existing FWA devices for data migration
   - **Mitigation:** Request from client during implementation; can migrate later if not available

**External Dependencies:**

1. **Verizon M2M API Availability**
   - Both accounts must have valid API credentials
   - Both accounts must remain active
   - **Mitigation:** Validate credentials during deployment; fallback to default account if FWA fails

2. **Verizon Account Restrictions**
   - Verizon's enforcement of FWA account separation must remain stable
   - Verizon's billing rules must not change unexpectedly
   - **Mitigation:** System designed to be flexible; can adjust restrictions if Verizon changes rules

### Risks

#### HIGH RISK: API Calls to Wrong Account During Transition

**Description:** During deployment and migration, API calls might route to wrong account, causing billing issues or failures

**Impact:** High - Could result in incorrect billing, failed operations, customer impact

**Probability:** Medium

**Mitigation:**
- Phased deployment: migrate database first, then code
- Test in staging with both account credentials before production
- Deploy during low-traffic window
- Monitor API call logs closely after deployment
- Have rollback plan ready (revert code, devices remain migrated)
- Start with small batch of FWA devices, expand gradually

#### MEDIUM RISK: Customer Confusion About Account Types

**Description:** Customers may not understand difference between regular and FWA accounts, leading to support tickets

**Impact:** Medium - Increased support burden, potential customer dissatisfaction

**Probability:** Medium

**Mitigation:**
- Clear labeling in UI: "Regular Account (Per-Byte)" vs "FWA Account (Unlimited)"
- Help text and tooltips explain differences
- Confirmation warnings for FWA upgrades
- Create knowledge base article
- Train customer support team before launch

#### MEDIUM RISK: Service Plan Misconfiguration

**Description:** Admin might incorrectly mark service plans with wrong account restriction, preventing legitimate assignments

**Impact:** Medium - Operational friction, customer complaints, manual fixes required

**Probability:** Low-Medium

**Mitigation:**
- Clear UI with help text
- Default to "No Restriction" for new service plans
- Audit existing service plans during configuration
- Provide admin report of service plan restrictions
- Easy to fix: just change restriction and re-save

#### LOW RISK: Missing FWA Credentials

**Description:** FWA account credentials might be invalid or change without notice

**Impact:** Medium - FWA API calls fail, but regular account unaffected

**Probability:** Low

**Mitigation:**
- Validate credentials during deployment
- API fallback to default account with warning log
- Monitor API failure rates by account
- Alert ops team if FWA account has repeated failures

#### LOW RISK: Performance Impact of Additional Fields

**Description:** New fields and indexes might slow down queries

**Impact:** Low - Slight performance degradation

**Probability:** Low

**Mitigation:**
- Indexes added to support filtering
- Fields nullable, not required in queries
- Test query performance in staging
- Monitor slow query log after deployment

---

## Implementation Plan

### Phase 1: Core Infrastructure (Week 1 - 5 days)

**Day 1-2: Database and Configuration**
- [ ] Create migration for devices.verizon_account_name
- [ ] Create migration for historical_service_plans.verizon_account_name
- [ ] Create migration to populate existing devices with 'default'
- [ ] Create migration to set FWA devices (from client list)
- [ ] Add multi-account configuration to app_local.php
- [ ] Update Device and HistoricalServicePlan entities
- [ ] Run migrations in staging environment
- [ ] Validate data migration results

**Day 3-5: VerizonApi Class and Call Sites**
- [ ] Update VerizonApi constructor to accept account key
- [ ] Add getAccountKeyFromDevice() static helper method
- [ ] Update all 15+ VerizonApi instantiation locations
- [ ] Add API call logging with account information
- [ ] Test API routing in staging environment
- [ ] Validate FWA credentials work correctly

### Phase 2: Service Plan Restrictions (Week 2 - 5 days)

**Day 6-7: Service Plan Configuration**
- [ ] Update ServicePlansTable with account restriction field
- [ ] Update service plan edit form with restriction dropdown
- [ ] Update service plan view page to display restriction
- [ ] Add service plan index filtering by restriction
- [ ] Test service plan configuration UI

**Day 8-10: Device Service Plan Filtering and Validation**
- [ ] Add service plan dropdown filtering logic in DevicesController (single device edit)
- [ ] Add service plan dropdown filtering logic in Assign Device page (bulk assignment)
- [ ] Implement intelligent filtering for mixed device selections (Assign Device page)
- [ ] Implement validation rule for service plan compatibility (single device)
- [ ] Implement batch validation for bulk device assignment (Assign Device page)
- [ ] Add validation in DevicesTable::buildRules()
- [ ] Add batch validation in Assign Device controller action
- [ ] Test validation with various scenarios (single and bulk)
- [ ] Test filtered service plan dropdowns (device edit and Assign Device page)
- [ ] Test error messages for incompatible bulk assignments with device serial lists

### Phase 3: UI Updates (Week 2-3 - 5 days)

**Day 11-12: Device Management UI**
- [ ] Update device edit form with Verizon Account dropdown
- [ ] Update device view page with account badge
- [ ] Update device index table with account column
- [ ] Add device list filtering by Verizon account
- [ ] Style account badges (blue for regular, orange for FWA)

**Day 13-15: Device Import**
- [ ] Update import template to include Column 16
- [ ] Update import parser to handle verizon_account_name
- [ ] Add import validation for account values
- [ ] Update import summary to show account distribution
- [ ] Test import with various scenarios
- [ ] Update import documentation

### Phase 4: Customer Restrictions and Admin Override (Week 3 - 5 days)

**Day 16-17: One-Way FWA Upgrade**
- [ ] Add FWA upgrade detection in DevicesTable::beforeSave()
- [ ] Add confirmation modal for customer FWA upgrades
- [ ] Implement automatic account change on FWA plan selection
- [ ] Add audit logging for customer FWA upgrades
- [ ] Test customer upgrade flow

**Day 18-20: FWA Downgrade Prevention and Admin Override**
- [ ] Add customer downgrade validation in DevicesTable
- [ ] Implement service plan filtering for customer FWA devices
- [ ] Create revertFromFwa permission
- [ ] Add admin override checkbox to device edit form
- [ ] Add override reason field and confirmation
- [ ] Add audit logging for admin FWA reversions
- [ ] Test admin override flow
- [ ] Test customer downgrade blocking

### Phase 5: Testing and Documentation (Week 4 - 5 days)

**Day 21-22: Integration Testing**
- [ ] Run all integration test cases
- [ ] Test end-to-end workflows
- [ ] Test API call routing with both accounts
- [ ] Test edge cases (NULL values, missing configs, etc.)
- [ ] Performance testing (query response times)

**Day 23-24: UAT Preparation**
- [ ] Prepare UAT environment with test data
- [ ] Create UAT test accounts and permissions
- [ ] Document UAT test scenarios
- [ ] Conduct UAT with stakeholders
- [ ] Fix any issues identified in UAT

**Day 25: Documentation and Deployment Prep**
- [ ] Update CLAUDE.md with multi-account architecture
- [ ] Create admin user guide
- [ ] Create customer help article
- [ ] Document rollback procedure
- [ ] Prepare deployment checklist

### Phase 6: Deployment and Monitoring (Week 4 - 2 days)

**Day 26: Staging Deployment**
- [ ] Deploy to staging environment
- [ ] Run smoke tests
- [ ] Validate both accounts work
- [ ] Test with stakeholders

**Day 27: Production Deployment**
- [ ] Schedule deployment window (low traffic)
- [ ] Run database migrations
- [ ] Deploy code changes
- [ ] Validate configuration
- [ ] Run smoke tests in production
- [ ] Monitor API call logs
- [ ] Monitor error rates
- [ ] Verify FWA devices function correctly

**Post-Deployment (Days 28-30):**
- [ ] Monitor system for 3 days
- [ ] Address any issues immediately
- [ ] Gather user feedback
- [ ] Document lessons learned

**Total Estimated Timeline:** 4 weeks (20 business days)

### Rollout Strategy

**Deployment Approach:** Phased deployment with monitoring

**Phases:**
1. **Database Migration**: Deploy schema changes first (can be done ahead of code)
2. **Code Deployment**: Deploy all code changes in single release
3. **Configuration**: Add FWA account credentials to production config
4. **Validation**: Test both accounts work correctly
5. **Monitoring**: Watch API logs and error rates for 3 days

**User Communication:**
- Email to internal ops team 1 week before deployment
- Knowledge base article published before launch
- In-app notification to customers about new FWA upgrade option
- Customer support team briefed on changes

**Training Needed:**
- Ops team training session (1 hour)
- Customer support FAQ document
- Admin user guide in documentation

**Rollback Plan:**
- Code rollback via git revert (5 minutes)
- Database changes can remain (backward compatible)
- FWA devices will default to regular account (functional but suboptimal)
- Manual fixes may be needed for any devices changed during deployment window

---

## Success Metrics

### Key Performance Indicators (KPIs)

**Technical Metrics:**
- **API Success Rate by Account**: Baseline N/A → Target 99.5%+ for both accounts
- **API Call Routing Accuracy**: Baseline N/A → Target 100% (no wrong-account calls)
- **Query Performance Impact**: Baseline current → Target <5% increase in device list query time
- **Deployment Downtime**: Target <5 minutes

**User Adoption Metrics:**
- **FWA Device Count**: Baseline 10-15 → Target growth to 25+ within 3 months
- **Customer Self-Service FWA Upgrades**: Target 80%+ of FWA upgrades done by customers (vs. ops team)
- **Bulk Assignment Usage**: Target 60%+ of service plan assignments use Assign Device page (vs. individual edits)
- **Bulk Assignment Error Rate**: Target <5% of bulk assignments blocked due to incompatibility errors
- **Support Tickets Related to Account Confusion**: Target <5 tickets per month

**Business Metrics:**
- **Revenue from FWA Devices**: Baseline $0 (manual) → Target $70-80+ MRC × device count
- **Time to Onboard FWA Device**: Baseline manual (hours) → Target <10 minutes
- **Account Assignment Errors**: Target 0 billing discrepancies due to wrong account

### Measurement Plan

- **Measurement Period**: 3 months post-launch
- **Review Cadence**:
  - Daily monitoring first week (API success rates, errors)
  - Weekly review for 1 month (adoption, support tickets)
  - Monthly review ongoing (business metrics)
- **Success Threshold**:
  - All technical metrics met within 1 week
  - User adoption metrics trending positive within 1 month
  - Business metrics positive ROI within 3 months

**Monitoring Tools:**
- API call logging (verizon-api.log)
- Error tracking (error.log)
- o_logs audit entries
- Device list reports (count by account)
- Customer support ticket tagging

---

## Open Questions

### 1. FWA Device List for Migration
- **Question:** Which 10-15 devices are currently on the FWA account and need to be marked as such in the migration?
- **Options:**
  - **Option A**: Client provides list of device IDs or serial numbers before deployment
  - **Option B**: Migration initially marks all as 'default', client/ops manually updates FWA devices post-deployment
- **Decision Needed By**: Before Phase 6 (Deployment)
- **Decision Owner**: Client
- **Recommendation**: Option A - Get list before deployment for clean migration

### 2. Service Plan Configuration
- **Question:** Which existing service plans should be marked as "FWA Account Only"?
- **Discussion Points:**
  - Need to identify all unlimited/flat-rate plans
  - Should any plans remain unrestricted (available to both accounts)?
  - Impact on existing device-plan assignments
- **Decision Needed By**: Before Phase 5 (Testing)
- **Decision Owner**: Client + Operations Team
- **Recommendation**: Provide list of service plan names/IDs to configure

### 3. Admin Override Permission Assignment
- **Question:** Which admin users should have the "revertFromFwa" permission?
- **Options:**
  - **Option A**: All super admins users
  - **Option B**: Create new role specifically for this permission
- **Decision Needed By**: Before Phase 4 (Admin Override implementation)
- **Decision Owner**: Client + Management
- **Recommendation**: Option B - Limit to senior ops who understand Verizon manual processes

### 4. Customer Communication Approach
- **Question:** How should we communicate the FWA upgrade option to existing customers?
- **Options:**
  - **Option A**: Proactive email to all customers announcing new unlimited option
  - **Option B**: Passive - just add to UI, let customers discover
  - **Option C**: Targeted outreach to customers with high usage who might benefit
- **Decision Needed By**: Before Phase 6 (Deployment)
- **Decision Owner**: Client + Marketing/Sales
- **Recommendation**: Option C - Targeted approach to avoid overwhelming customers who don't need it

### 5. Display Names and Labeling
- **Question:** Are the proposed display names clear and appropriate?
  - "Regular Account (Per-Byte Pricing)"
  - "FWA Account (Unlimited Plans)"
- **Options:**
  - **Option A**: Use as proposed
  - **Option B**: Simplify to "Standard Account" and "Unlimited Account"
  - **Option C**: Use Verizon account numbers: "Account 1" and "Account 2"
- **Decision Needed By**: Before Phase 3 (UI Updates)
- **Decision Owner**: Client + UX consideration
- **Recommendation**: Option A - Most descriptive and clear for users

### 6. Import Template Backward Compatibility
- **Question:** Should we maintain backward compatibility with import files that don't include Column 16?
- **Options:**
  - **Option A**: Yes - Column 16 optional, defaults to 'default' if missing
  - **Option B**: No - Require Column 16, reject imports without it
- **Decision Needed By**: Before Phase 3 (Import implementation)
- **Decision Owner**: Development Team + Operations
- **Recommendation**: Option A - Maintain compatibility, less disruption

### 7. Assign Device Page Filtering Behavior
- **Question:** When devices with mixed accounts are selected on Assign Device page, should we:
- **Options:**
  - **Option A**: Show only unrestricted plans (no account-specific plans) - forces safe selection
  - **Option B**: Show all plans but display warning icon for incompatible ones - gives visibility
  - **Option C**: Auto-deselect devices with incompatible accounts when plan selected - dynamic adjustment
- **Decision Needed By**: Before Phase 2 (Service Plan Restrictions)
- **Decision Owner**: Operations Team + UX consideration
- **Recommendation**: Option A - Safest approach, prevents user errors entirely. If users need to see what plans are incompatible, they should assign devices separately by account type.
- **Current PRD Spec**: Option A (show only unrestricted plans for mixed selections)

---

## Next Steps

1. **Review and Approve PRD** - Client + Development Lead by [Date]
2. **Provide FWA Device List** - Client by [Date]
3. **Provide FWA Service Plan List** - Client by [Date]
4. **Assign Admin Override Permissions** - Management by [Date]
5. **Schedule Kickoff Meeting** - Development Lead by [Date]
6. **Create Jira Tickets from Implementation Plan** - Development Lead by [Date]
7. **Begin Phase 1 Development** - Development Team by [Date]

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-29 | Development Team | Initial PRD created from client request and technical analysis |

---

**Document Status:** Draft for Review
**Next Review Date:** [Date]
**Approvals Required:**
- [ ] Product Manager / Client
- [ ] Engineering Lead
- [ ] QA Lead
- [ ] Operations Team Lead
- [ ] Customer Support Manager

---

END OF DOCUMENT
