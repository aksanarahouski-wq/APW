# Scope Change Summary: Verizon Second Account Support

**Date:** 2026-02-17
**Change Type:** Scope Reduction - Features Moved to Separate Initiatives
**Approved By:** Aksana Rahouski
**Status:** Implemented in PRD v2.2

---

## Summary

Two features that were identified in Meeting 2 and included in PRD v2.0-2.1 have been removed from the Verizon Second Account Support scope. These features will be handled as separate initiatives.

---

## Features Moved Out of Scope

### 1. Service Plan Filtering on Device Index
**What It Was:**
- Add a service plan filter dropdown to the device index/list page
- Allow users to filter devices by their assigned service plan
- Improve device list navigation and management

**Why It Was Removed:**
- Not directly related to Verizon account management
- Adds complexity to an already substantial feature
- Can be delivered independently without blocking Verizon second account functionality
- Better to keep focused scope for faster delivery

**Where It Was Mentioned:**
- Meeting 2 discussion (lines 293-295)
- PRD v2.0-2.1 sections: Goals, User Stories (Story 5.1), FR-6, Test Cases (IT-7, UAT-5)
- Action Items document (Item #5)

**Impact:**
- Reduces Verizon Second Account Support scope
- No impact on core Verizon account functionality
- Will be prioritized as separate feature

---

### 2. Device Group Mismatch Reporting
**What It Was:**
- Scheduled job (monthly/quarterly) to detect device group mismatches
- Generate CSV reports showing devices where device.device_group != service_plan.device_group
- Help operations team maintain data integrity
- Manual review and correction workflow

**Why It Was Removed:**
- Not directly related to Verizon account management
- Addresses a separate data quality issue (47 devices currently affected)
- Adds significant complexity (scheduled jobs, report generation, storage)
- Can be delivered independently without blocking Verizon second account functionality
- Better to keep focused scope for faster delivery

**Where It Was Mentioned:**
- Meeting 2 discussion (lines 1379-1420, 1495-1611)
- PRD v2.0-2.1 sections: Goals, User Stories (Story 5.2), FR-7, Technical Design, Test Cases (UT-4, IT-8, UAT-6)
- Action Items document (Item #5)

**Impact:**
- Reduces Verizon Second Account Support scope
- No impact on core Verizon account functionality
- Operations team continues current manual review process
- Will be prioritized as separate feature

---

## What Remains IN SCOPE

### Service Plan Validation and Filtering
**IMPORTANT:** The following service plan filtering functionality remains IN SCOPE:

✅ **Device Edit Form:**
- Service plan dropdown filtered by device's Verizon account
- Shows only compatible service plans for the selected device
- Part of validation logic to prevent incompatible assignments

✅ **Assign Device Page (Bulk Assignment):**
- Service plan dropdown filtered by selected devices' Verizon accounts
- If all devices have same account: Show plans for that account
- If devices have mixed accounts: Show only unrestricted plans
- Part of validation logic to prevent bulk incompatible assignments
- Partial assignment support (compatible devices succeed, incompatible fail)

❌ **Device Index/List Page:**
- Service plan filter dropdown - REMOVED (separate feature)

---

## Updated Documents

### 1. PRD v2.2
**File:** `/Users/aksana/Documents/Projects/WATM/Verizon2ndAccount/PRD_Verizon_Second_Account_Support_v2.md`

**Changes:**
- Removed from Key Features, Business Impact
- Removed from Goals and Objectives (Goal 5)
- Removed Story 5.1 and Story 5.2 from User Stories
- Removed from Scope "In Scope" section
- Removed FR-1.5 (Device List Service Plan Filter)
- Removed FR-6 (Service Plan Filtering) entirely
- Removed FR-7 (Device Group Mismatch Reporting) entirely
- Updated FR-2.5 with clarification about what remains in scope
- Removed from Technical Design architecture diagram
- Removed from Technical Design code implementation sections
- Removed test cases: UT-3, UT-4, IT-7, IT-8, UAT-5, UAT-6
- Removed from Dependencies and Risks
- Removed from Implementation Plan (Phase 3 tasks)
- Removed from Success Metrics
- Removed Question 4 from Open Questions
- Updated Question 7 resolution to remove service plan filter reference
- Updated Document History with v2.2 entry

### 2. Action Items Document
**File:** `/Users/aksana/Documents/Projects/WATM/Verizon2ndAccount/Meeting2_ActionItems.md`

**Changes:**
- Added Scope Update section at top of document
- Updated Action Item #3 (Update PRD) to show features moved out of scope
- Updated Action Item #5 (Device Group Mismatch Reporting) status to "Out of Scope"
- Updated summary dashboard to reflect status changes

---

## Rationale for Scope Reduction

### 1. Focus on Core Functionality
The primary goal of Verizon Second Account Support is to:
- Enable device-level Verizon account assignment
- Prevent billing errors from account mismatches
- Route API calls to correct Verizon account
- Support FWA unlimited plans

Both removed features are **operational improvements** that enhance usability but are not core to achieving these goals.

### 2. Faster Time to Value
Reducing scope allows:
- Faster development (estimated 3-3.5 weeks → likely faster)
- Simpler testing and QA
- Lower risk deployment
- Quicker access to FWA revenue opportunity
- Less complex rollback if issues occur

### 3. Independent Delivery
Both features can be delivered independently:
- **Service Plan Filter:** Pure UI enhancement, no data model changes
- **Device Group Mismatch Reporting:** Standalone reporting job, no integration with Verizon accounts

### 4. Prioritization Flexibility
Separating these features allows:
- Re-evaluation of priority after Verizon Second Account deployment
- Assessment of actual user need vs. assumed need
- Resource allocation based on other competing priorities
- Opportunity to combine with other reporting/filtering enhancements

---

## Timeline Impact

### Original Estimate (PRD v2.0-2.1): 3-3.5 weeks (15-17 days)
- Phase 1: Core Infrastructure (5 days)
- Phase 2: Service Plan Restrictions (5 days)
- Phase 3: UI Updates (4 days) - **Included device group reporting**
- Phase 4: Testing (5 days)
- Phase 5: Deployment (2 days)

### Revised Estimate (PRD v2.2): 2.5-3 weeks (13-15 days)
- Phase 1: Core Infrastructure (5 days) - No change
- Phase 2: Service Plan Restrictions (5 days) - No change
- Phase 3: UI Updates (2 days) - **Removed 2 days for reporting and filtering**
- Phase 4: Testing (3 days) - **Reduced 2 days (fewer test cases)**
- Phase 5: Deployment (2 days) - No change

**Net Impact:** 2-2.5 days faster (~10-15% reduction)

---

## Next Steps

### Immediate (This Week):
- ✅ Update PRD to v2.2 (complete)
- ✅ Update Action Items document (complete)
- ✅ Document scope change (this file)
- [ ] Communicate scope change to stakeholders:
  - Devon D'Andrea
  - Adam Curcie
  - Development Team Lead
  - QA Lead

### Short-Term (Before Development):
- [ ] Confirm revised timeline estimate with development team
- [ ] Update any existing Jira tickets to reflect scope
- [ ] Review and approve PRD v2.2
- [ ] Green light to proceed with development

### Future (After Verizon Second Account Deployment):
- [ ] Create separate requirements document for Service Plan Filtering feature
- [ ] Create separate requirements document for Device Group Mismatch Reporting feature
- [ ] Prioritize both features in product roadmap
- [ ] Estimate and schedule for future sprint

---

## Communication Template

**Subject:** Scope Update - Verizon Second Account Support

**Message:**

Hi team,

We've made a scope adjustment to the Verizon Second Account Support initiative to keep the project focused and accelerate delivery.

**What Changed:**
Two features identified in Meeting 2 have been moved out of scope and will be handled as separate features:
1. Service Plan filtering on device index page
2. Device Group Mismatch Reporting (scheduled job)

**What Remains:**
All core Verizon account functionality remains in scope:
- Device-level account assignment (radio buttons)
- Service plan account restrictions
- Validation to prevent incompatible assignments (including filtered dropdowns in edit forms)
- Dynamic service plan form behavior
- API call routing
- Admin account tracking
- Bulk assignment with partial assignment support

**Why This Change:**
- Faster delivery of core Verizon account functionality
- Lower risk, simpler testing
- These features can be delivered independently
- Accelerates access to FWA revenue opportunity

**Timeline Impact:**
Estimated timeline reduced from 3-3.5 weeks to 2.5-3 weeks.

**Next Steps:**
- PRD v2.2 is ready for review
- Ready to proceed with development once timeline confirmed
- Removed features will be prioritized separately

Please review the updated PRD v2.2 and let me know if you have any questions.

Thanks,
Aksana

---

**Document Owner:** Aksana Rahouski
**Last Updated:** 2026-02-17
**Related Documents:**
- PRD v2.2: `/Users/aksana/Documents/Projects/WATM/Verizon2ndAccount/PRD_Verizon_Second_Account_Support_v2.md`
- Action Items: `/Users/aksana/Documents/Projects/WATM/Verizon2ndAccount/Meeting2_ActionItems.md`
- Meeting 2 Transcript: `/Users/aksana/Documents/Projects/WATM/Verizon2ndAccount/Meeting2.md`
