# Product Requirements Document (PRD) - Client Summary
## Verizon Second Account Support - FWA Integration

**Document Version:** 2.5 (Short Version)
**Date:** 2026-02-18
**Author:** Development Team
**Status:** Ready for Development
**Related Tickets:** [WATM-XXXX](https://orases.atlassian.net/browse/WATM-XXXX)
**Document Owner:** Aksana

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Background and Problem Statement](#background-and-problem-statement)
3. [Goals and Objectives](#goals-and-objectives)
4. [Target Users](#target-users)
5. [Scope](#scope)
6. [Known Limitations and Phase 2 Considerations](#known-limitations-and-phase-2-considerations)
7. [Document History](#document-history)

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
