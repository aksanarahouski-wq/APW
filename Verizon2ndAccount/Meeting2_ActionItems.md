# Action Items: Verizon Second Account Support - Post-Meeting 2

**Meeting Date:** 2026-02-17
**Document Created:** 2026-02-17
**Last Updated:** 2026-02-17
**Status:** Active

---

## 📋 Scope Update (2026-02-17)

**Features Moved Out of Scope:**
Two features discussed in Meeting 2 have been moved out of scope for this initiative and will be handled as separate features:
1. **Service Plan Filtering on Device Index** - Service plan filter dropdown on device list page
2. **Device Group Mismatch Reporting** - Scheduled job to detect and report device group mismatches

**Note:** Service plan dropdown filtering in device EDIT form and Assign Device page remains IN SCOPE as part of validation logic.

---

## CRITICAL - Immediate Action Required

### 1. UI Design Decision
**Owner:** Aksana
**Deadline:** This Week
**Status:** ✅ COMPLETE

**Task:** Make final decision on account selector UI design

**DECISION MADE: Option C - Radio Buttons**

**Rationale:**
- **Scalability:** Radio buttons can accommodate future additions (3rd Verizon account, T-Mobile, etc.) without code changes
- **User Experience:** Clear visual distinction between mutually exclusive options (only one account per device)
- **Accessibility:** Better than checkbox for semantics (radio = one choice, checkbox = multiple choices)
- **Middle Ground:** Combines simplicity of checkbox with scalability of dropdown
- **Vertical Space:** Takes slightly more space than checkbox/dropdown but acceptable trade-off for clarity

**Implementation Details:**
- Device create/edit form: Radio button group for Verizon account selection
- Labels: ⚪ "Regular Account (PPU)" | ⚪ "Unlimited Internet (FWA)"
- Default selection: "Regular Account (PPU)" for new devices
- Visual styling: Standard radio buttons with clear labels and spacing

**Next Steps:**
- [x] Choose UI pattern (radio buttons)
- [x] Document rationale
- [ ] Update wireframes/mockups with radio button design
- [ ] Communicate decision to development team

---

### 2. Confirm Development Timeline
**Owner:** Aksana + Laura
**Deadline:** This Week
**Status:** 🔴 Not Started

**Task:** Establish clear timeline for Verizon second account development

**Questions to Answer:**
- [ ] When can development start?
- [ ] What's the estimated duration?
- [ ] When is target deployment date?
- [ ] What are the dependencies/blockers?

**Context from Meeting:**
- Team confirmed Verizon second account is top priority before configs or credit card (Meeting2.md lines 183-220)
- Aksana's goal was to finalize scope in this meeting (achieved)
- Devon is okay with 3-month delay on credit card work

**Action Items:**
- [ ] Review current sprint capacity
- [ ] Check developer availability
- [ ] Identify any blocking dependencies
- [ ] Communicate timeline to Devon/Adam
- [ ] Add to roadmap with dates

---

### 3. Update PRD with Meeting 2 Decisions
**Owner:** Aksana
**Deadline:** This Week
**Status:** ✅ COMPLETE

**Task:** Update PRD to reflect all decisions made in Meeting 2

**Changes Required:**

**ADD - New Requirements:**
- [x] Dynamic form behavior: gray out usage limits and device group names when FWA selected (Meeting2.md lines 483-487)

**MOVED OUT OF SCOPE (Separate Features):**
- [x] Service plan filter on device index page - Will be handled as separate feature
- [x] Device group mismatch reporting job - Will be handled as separate feature

**UPDATE - Changed Decisions:**
- [x] Terminology: "Unlimited Internet (FWA)" and "Regular Account (PPU)" (Meeting2.md lines 1347-1362)
- [x] UI Design: Radio buttons for account selection (decided 2026-02-17)
- [x] Migration strategy: Manual post-deployment for 15 devices (Meeting2.md lines 1095-1111)
- [x] Confirmation dialogs: NOT needed (Meeting2.md lines 741-787)
- [x] Service plan form fields: Keep WiFi, Firewall, Show Usage; remove/gray out usage limits and device group names

**DOCUMENT - Known Limitations:**
- [ ] Mid-cycle billing gap (Phase 2) - devices switching accounts mid-cycle require manual billing adjustment (Meeting2.md lines 803-1064)
- [ ] 5G device support not included (will revisit when needed) (Meeting2.md lines 302-348)

**REMOVE - Out of Scope:**
- [x] Customer self-service upgrade (already removed in Meeting 1)
- [x] Admin-automated account migration (already removed in Meeting 1)

---

## HIGH PRIORITY - This Week

### 4. Document Mid-Cycle Billing Workaround Strategy
**Owner:** Aksana + Devon
**Deadline:** Before Development Starts
**Status:** 🟡 In Discussion

**Task:** Document strategy for handling devices that switch accounts mid-billing cycle

**Problem Statement:**
When a device switches from pay-per-use to unlimited (or vice versa) mid-cycle, the data usage accrued on the old account isn't automatically billed.

**Agreed Approach (from Meeting):**
Volume is too low (5-10 devices/month) to build full automation now. Manual handling acceptable.

**Workaround Options Discussed:**
1. **Restrict to 1st of month only** - Only allow account changes on billing cycle start
2. **Charge penalty fee** - Cover potential lost revenue with flat fee
3. **Manual billing adjustment** - Ops team manually reviews and adjusts invoices

**Action Items:**
- [ ] Choose primary workaround strategy
- [ ] Document process for ops team
- [ ] Create internal KB article
- [ ] Add warning message in portal when changing account mid-cycle
- [ ] Document as Phase 2 enhancement in PRD
- [ ] Estimate impact: How much revenue at risk? How much time for manual process?

---

### 5. Design Device Group Mismatch Reporting Job
**Owner:** Aksana + Development Team
**Deadline:** TBD (Separate Feature)
**Status:** ⚪ MOVED OUT OF SCOPE

**Decision:** This feature will be handled as a separate feature outside the Verizon Second Account Support scope.

**Context from Meeting:**
- 47 devices currently have device groups that don't match their service plans (Meeting2.md lines 1379-1420)
- Reason: Manual workarounds where device is on higher tier at Verizon but lower tier in portal for customer satisfaction
- Service plan changes are rare, so event-based syncing doesn't work

**Next Steps:**
- [ ] Create separate PRD/requirements document for Device Group Mismatch Reporting
- [ ] Schedule as future feature after Verizon Second Account Support is deployed

---

### 6. Create Wireframes/Mockups for Key Screens
**Owner:** Aksana
**Deadline:** Before Development Starts
**Status:** 🟡 Planning

**Screens Needing Design:**
- [ ] Device create/edit form with **radio button** account selector
- [ ] Service plan create/edit form with FWA checkbox and conditional field behavior
- [ ] Device view page with account badge
- [ ] Device list page with service plan filter
- [ ] Device list page with account column

**Design Specifications:**
- [x] Account selector UI pattern: **Radio buttons** ⚪ Regular Account (PPU) | ⚪ Unlimited Internet (FWA)
- [x] Terminology: "Unlimited Internet (FWA)" vs "Regular Account (PPU)"
- [ ] Badge design for account display (colors, icons, placement)
- [ ] Gray-out behavior for usage limits/device group names when FWA selected
- [ ] Radio button styling: alignment, spacing, label formatting

---

## MEDIUM PRIORITY - Next Week

### 7. Technical Design Document
**Owner:** Development Team Lead
**Deadline:** Next Week
**Status:** 🔴 Not Started

**Task:** Create detailed technical design document

**Sections Required:**
- [ ] Database schema changes (devices.verizon_account_name, service_plans.verizon_account_name)
- [ ] Migration scripts (add columns, default to 'regular', handle 15 existing FWA devices)
- [ ] VerizonApi class modifications (accept account parameter, route to correct credentials)
- [ ] Locations requiring VerizonApi updates (15+ places identified in codebase)
- [ ] Service plan form conditional logic (gray out fields when FWA selected)
- [ ] Validation logic (single device edit, bulk assignment, import)
- [ ] Device group mismatch reporting job architecture
- [ ] Error handling and messaging

**Dependencies:**
- Pending UI design decision
- Pending PRD updates

---

### 8. Update Test Plan
**Owner:** QA Team
**Deadline:** Next Week
**Status:** 🔴 Not Started

**Task:** Update test plan to reflect scope changes

**Test Cases to ADD:**
- [ ] Device create with account selection
- [ ] Device edit - change account field
- [ ] Service plan create with FWA checkbox
- [ ] Service plan form conditional behavior (fields gray out)
- [ ] Single device validation (account mismatch with service plan)
- [ ] Bulk device assignment validation (mixed accounts)
- [ ] Import validation (invalid account values)
- [ ] API routing (verify correct credentials used per device account)
- [ ] Device group mismatch reporting job
- [ ] Service plan filter on device index

**Test Cases to REMOVE:**
- [x] Customer self-service upgrade
- [x] Admin automated account migration

**Test Cases to MODIFY:**
- [ ] Update terminology in all test cases to "Unlimited Internet (FWA)" and "Regular Account (PPU)"

---

### 9. Create KB Article: Manual Verizon Account Migration Process
**Owner:** Devon + Operations Team
**Deadline:** Before Production Deployment
**Status:** 🔴 Not Started

**Task:** Document the 5-step manual Verizon account migration process

**Content Required:**
1. **Overview:** When and why devices need to be moved between Verizon accounts
2. **Prerequisites:** Authorization, access requirements
3. **Step-by-Step Process:**
   - Step 1: Log into physical device
   - Step 2: Change APN configuration
   - Step 3: Deactivate SIM in Verizon account #1
   - Step 4: Reactivate SIM in Verizon account #2 (within 2-3 minute window)
   - Step 5: Verify IP address changed
4. **Portal Updates:**
   - Update device.verizon_account_name field
   - Assign device to compatible service plan
   - Verify device connectivity
5. **Rollback Procedure:** What to do if something goes wrong
6. **Common Issues:** Troubleshooting guide
7. **Who Can Perform:** List of authorized personnel

**Context from Meeting:**
- Only 3 people authorized to perform migrations (Meeting2.md lines 741-787)
- Process is "too risky to automate" due to tight timing window and complexity
- Volume is low (5-10 devices per month)

---

### 10. Ship Card 1965 (Checkbox Feature)
**Owner:** Development Team
**Deadline:** ASAP
**Status:** 🟢 Dev Complete, Ready to Ship

**Task:** Deploy card 1965 that customer urgently requested

**Context from Meeting:**
- Devon asked when it can be released (Meeting2.md lines 1203-1231)
- Aksana confirmed it's dev-complete
- High priority for customer satisfaction

**Action Items:**
- [ ] QA regression testing
- [ ] Schedule deployment
- [ ] Notify Devon when shipped
- [ ] Notify customer

---

## LOWER PRIORITY - Before Development

### 11. Define Error Message Standards
**Owner:** Aksana + UX
**Deadline:** Before Development Starts
**Status:** 🔴 Not Started

**Task:** Define standard error messages for account mismatch scenarios

**Scenarios Requiring Messages:**

**Scenario 1: Single Device Edit**
- User tries to assign FWA service plan to regular account device
- Error: "This service plan requires an Unlimited Internet account, but this device is configured for Regular account. Please select a compatible service plan or contact support to upgrade this device's account."

**Scenario 2: Bulk Device Assignment**
- User tries to assign FWA plan to mix of 100 devices (80 regular, 20 FWA)
- Error: "Cannot assign service plan: 80 device(s) are incompatible. This service plan requires Unlimited Internet account. Incompatible devices: [list of serial numbers]. Please select only Unlimited Internet devices or choose a compatible service plan."

**Scenario 3: Import Validation**
- Import includes invalid account value
- Error: "Row 15: Device with serial number 'ABC123' has invalid Verizon account 'business'. Must be 'regular' or 'fwa'. This row will be skipped."

**Scenario 4: Mid-Cycle Account Change**
- Admin changes account field mid-billing cycle
- Warning: "This device is being changed mid-billing cycle. Data usage accrued on the [old account name] will need to be manually billed. Refer to [KB article link] for billing adjustment process."

**Standards to Define:**
- [ ] Tone (technical vs. user-friendly)
- [ ] Level of detail
- [ ] Remediation guidance
- [ ] When to show warnings vs. errors vs. info messages

---

### 12. Database Migration Strategy for 15 FWA Devices
**Owner:** Development Team
**Deadline:** Before Development Starts
**Status:** 🟡 Planning

**Task:** Plan migration approach for 15 existing FWA devices

**Context from Meeting:**
- 15 devices already manually migrated to Verizon FWA account (Meeting2.md lines 1069-1198)
- Currently in "mess" state: no service plans, flat rate billing, Verizon identifiers removed to stop API failures
- Decision: Manual one-by-one update after deployment (Meeting2.md lines 1095-1111)

**Migration Plan:**
1. **Pre-Deployment:**
   - [ ] Identify all 15 devices in portal (already visible)
   - [ ] Document current state (company, billing status, missing data)
   - [ ] Create checklist for post-deployment updates

2. **Post-Deployment:**
   - [ ] Update each device: Set verizon_account_name = 'fwa'
   - [ ] Re-add Verizon identifiers (IMEIs) if missing
   - [ ] Assign to appropriate FWA service plan
   - [ ] Verify API calls route to correct FWA account
   - [ ] Monitor for any errors

3. **Fail-Safe:**
   - [ ] Prevent config updates to devices until account field is explicitly set
   - [ ] Add validation to block NULL account values for Verizon devices

**Owner:** Devon + Adam (identify devices), Development (implement fail-safe)

---

### 13. Estimate Timeline and Create Jira Tickets
**Owner:** Aksana + Development Team Lead
**Deadline:** Before Development Starts
**Status:** 🔴 Not Started

**Task:** Break down work into stories, estimate, and create Jira tickets

**Epic Breakdown:**

**Epic 1: Database & Core Infrastructure**
- [ ] Add verizon_account_name field to devices table
- [ ] Add verizon_account_name field to service_plans table
- [ ] Create migration scripts
- [ ] Update VerizonApi class to accept account parameter
- [ ] Update 15+ locations that instantiate VerizonApi

**Epic 2: Device Management UI**
- [ ] Device create/edit form: Add account selector
- [ ] Device view page: Add account badge
- [ ] Device list page: Add account column
- [ ] Device list page: Add service plan filter
- [ ] Device import: Add account column to template

**Epic 3: Service Plan Management UI**
- [ ] Service plan create/edit: Add FWA checkbox
- [ ] Service plan form: Conditional field behavior (gray out usage limits/device groups)
- [ ] Service plan view: Display account compatibility

**Epic 4: Validation & Business Logic**
- [ ] Single device edit validation
- [ ] Bulk device assignment validation (with partial assignment support)
- [ ] Import validation
- [ ] Error message implementation
- [ ] Mid-cycle account change warning (if implemented)

**Epic 5: Device Group Mismatch Reporting (NEW)**
- [ ] Design scheduled job
- [ ] Implement detection logic
- [ ] Generate report
- [ ] Create download UI
- [ ] Schedule job (monthly/quarterly)

**Epic 6: Testing & Documentation**
- [ ] QA test plan execution
- [ ] KB article for manual migration process
- [ ] User documentation updates
- [ ] API documentation updates

**Estimated Effort:** TBD (need technical design first)

---

## DEFERRED - Phase 2

### 14. Mid-Cycle Billing Automation
**Owner:** TBD
**Deadline:** Phase 2
**Status:** 🔵 Deferred

**Task:** Build automated solution for mid-cycle account switching billing

**Context from Meeting:**
- Current volume (5-10 devices/month) too low to justify automation (Meeting2.md lines 950-1014)
- Manual workarounds acceptable for now
- Should revisit if volume increases or manual process becomes too burdensome

**Requirements for Phase 2:**
- [ ] Prorate data usage from pay-per-use account
- [ ] Handle billing cycle split between two accounts
- [ ] Generate accurate invoices with usage from both accounts
- [ ] Reuse logic from existing customer-to-customer transfer process (Meeting2.md lines 866-880)

**Trigger to Revisit:**
- Volume increases above 10-15 devices/month
- Manual process becomes error-prone or time-consuming
- Customer complaints about billing accuracy

---

### 15. 5G Device Support
**Owner:** TBD
**Deadline:** Phase 2 / Future
**Status:** 🔵 Deferred

**Task:** Add support for 5G FWA devices (requires different activation process)

**Context from Meeting:**
- Current scope only includes 4G devices (Meeting2.md lines 302-348)
- 5G devices require address validation and tower capacity checks
- No 5G devices in inventory currently

**Requirements for Phase 2:**
- [ ] Research Verizon 5G activation requirements
- [ ] Implement address validation
- [ ] Implement tower capacity checking
- [ ] Update device activation workflow for 5G

**Trigger to Revisit:**
- When 5G FWA devices are added to inventory
- When customer requests 5G service

---

## Meeting Follow-Up Items

### 16. Send Meeting Summary to Team
**Owner:** Aksana
**Deadline:** This Week
**Status:** 🔴 Not Started

**Task:** Distribute meeting summary and action items

**Recipients:**
- Devon D'Andrea
- Adam Curcie
- Stone (if applicable)
- Development Team Lead
- QA Lead

**Content:**
- [ ] Link to this action item document
- [ ] Key decisions made
- [ ] Timeline expectations
- [ ] Next steps
- [ ] Request for feedback/questions

---

### 17. Schedule Follow-Up Meeting (If Needed)
**Owner:** Aksana
**Deadline:** TBD
**Status:** ⚪ Pending

**Task:** Determine if additional alignment meeting is needed before development

**Potential Topics:**
- Review updated PRD
- Review technical design
- Review wireframes/mockups
- Final approval to proceed

**Decision Point:** After PRD updated and technical design complete

---

## Summary Dashboard

### By Priority
- 🔴 **CRITICAL (3):** UI design, timeline, PRD update
- 🟡 **HIGH (7):** Mid-cycle billing, device group reporting, wireframes, technical design, test plan, KB article, card 1965
- 🔵 **DEFERRED (2):** Mid-cycle automation, 5G support

### By Status
- 🔴 **Not Started (9)**
- 🟡 **In Progress (1)**
- 🟢 **Complete (3)**
- 🔵 **Deferred (2)**
- ⚪ **Out of Scope/Separate Feature (2)**

### By Owner
- **Aksana (6):** UI design, timeline, PRD update, mid-cycle billing doc, wireframes, meeting summary
- **Development Team (4):** Technical design, device group reporting, migration strategy, Jira tickets
- **QA Team (1):** Test plan
- **Devon/Operations (1):** KB article
- **TBD (2):** Phase 2 items

---

**Last Updated:** 2026-02-17
**Next Review:** Weekly until development complete
