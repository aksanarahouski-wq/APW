# Meeting 2 Preparation: Verizon Second Account Support
**Date:** 2026-02-17
**Meeting Type:** Follow-up to Meeting 1 (Feb 5, 2026)
**Attendees:** Aksana, Devon, Adam, Stone, Development Team

---

## Meeting Objectives

1. Finalize portal scope (tracking vs. automation)
2. Resolve UI design decision (checkbox vs. dropdown)
3. Define migration strategy for 15 existing FWA devices
4. Confirm service plan form requirements
5. Agree on next steps and timeline

---

## Quick Reference: What Changed from PRD

### ❌ REMOVED from Scope
- **Customer self-service FWA upgrade** (Story 3.1)
  - Reason: Too complex/risky to automate (requires simultaneous APN change + cross-account SIM activation within 2-3 min window)
  - New approach: At most, a "request upgrade" button

- **Admin-automated account migration** (parts of Story 3.3)
  - Reason: Same complexity - requires manual multi-step process
  - Portal role changes: Track status, not automate switching

### ✅ CONFIRMED Requirements
- Device-level Verizon account assignment
- Service plan account restrictions and validation
- Bulk device assignment compatibility checking
- API routing based on device account
- Import template support

---

## Section 1: Confirmed Requirements

### 1.1 Device-Level Verizon Account Configuration

**Status:** ✅ Confirmed

**What We Agreed:**
- Every device with a Verizon SIM needs to specify which Verizon account it belongs to (regular or FWA/Business Internet)
- This field is stored at the device level, independent of service plan
- Device uses this field to determine which Verizon API credentials to use for all operations (activation, deactivation, status changes, etc.)

**Implementation Touchpoints:**
- Device create/edit form
- Device import template (add column for Verizon account)
- Device view page (display account badge)
- Device list page (filterable column)

**Key Quote from Meeting:**
> "If Verizon grade, we'll have to pick which Verizon, right? So now we have two... with a Verizon SIM, it will always tell him which account to map it to" - Aksana (lines 495-498)

---

### 1.2 Service Plan Account Restrictions

**Status:** ✅ Confirmed (Enhanced Understanding)

**What We Agreed:**
- Service plans must be marked as compatible with specific Verizon accounts
- FWA/Business Internet service plans can ONLY be used by devices on FWA account
- Regular service plans can ONLY be used by devices on regular account
- When assigning a device to a service plan, system validates compatibility

**Why This Matters (New Insight):**
- Service plans control MORE than billing - they also control device configurations (APNs, etc.)
- FWA account uses completely different APN than regular account
- Wrong service plan = wrong config = device won't connect

**Key Quote from Meeting:**
> "This box is on the ATM service plan, so it gets this config. This box is on tier one, it gets this config. And this box is on the business plan, so it gets this config with a completely separate APN, because it's a completely separate Verizon account." - Adam (lines 555-560)

---

### 1.3 Service Plan Compatibility Validation

**Status:** ✅ Confirmed

**What We Agreed:**
- Single device edit: Validate service plan matches device's Verizon account before save
- Bulk device assignment: Validate ALL selected devices are compatible with chosen service plan
- Display clear error messages identifying incompatible devices (by serial number)
- Build on existing good error handling foundation

**Validation Rules:**
- If device.verizon_account = 'fwa' → only allow service plans marked as 'fwa' or unrestricted
- If device.verizon_account = 'regular' → only allow service plans marked as 'regular' or unrestricted
- Bulk assignment: If mixed accounts selected, only show unrestricted service plans

**Key Quote from Meeting:**
> "If device is business, but service plan is not, that match is impossible." - Aksana (line 726)

---

### 1.4 Bulk Device Assignment Validation

**Status:** ✅ Confirmed

**What We Agreed:**
- Assign Device page will validate service plan compatibility for each device in the batch
- Validation occurs when assignment is submitted

**Error Handling:**
- System supports **partial assignment** (not all-or-nothing)
- Devices that pass validation are successfully assigned
- Devices that fail validation are skipped and listed in error message
- Error message displays incompatible devices with serial numbers
- Customer can review failed devices, correct issues, and re-submit that subset
- Leverage existing robust error handling (recently improved)

---

### 1.5 Simplified FWA Service Plan Form

**Status:** ✅ Confirmed

**What We Agreed:**
- FWA service plans don't need most standard fields
- Required: Service plan name + FWA account indicator (checkbox or dropdown)
- Not needed: Device group names, usage limits (mostly)
- Form can be dramatically simplified for FWA plans

**Key Quote from Meeting:**
> "We only will need that checkbox and the name. Everything else is optional and probably won't be used." - Devon & Adam (lines 777-780)

**Open Question:** Exact list of required vs. optional fields (see Section 2.4)

---

### 1.6 API Routing Based on Device Account

**Status:** ✅ Confirmed (Implicit)

**What We Agreed:**
- All Verizon API calls must route to correct account based on device configuration
- 15+ locations in code that instantiate VerizonApi need updating
- Device account field determines which credentials to use

**Current Problem:**
- 15 existing FWA devices causing API failures because portal doesn't know which account they're on
- Missing IMEIs/SIMs for these devices

---

## Section 2: Open Questions for Discussion

### 2.1 🔴 CRITICAL: Device Account Upgrade/Downgrade Process

**Context:**
We have agreed that the portal's role is **tracking-only** - it displays account status but does NOT automate account switching. This applies to:
- Initial 15 devices already on Business Internet account (migration)
- **Future devices** that need to be upgraded (Regular → Business Internet) or downgraded (Business Internet → Regular)

**The Manual Process (From Meeting 1):**
Moving a device between Verizon accounts requires:
1. Log into physical device
2. Change APN configuration
3. Deactivate SIM in Verizon account #1
4. Reactivate SIM in Verizon account #2 (within 2-3 minute window)
5. IP address changes
- This process is "too risky to automate" (Devon)
- "Numerous steps... have to be executed in a very specific order" (Adam)

**Questions to Answer:**

**For Future Upgrades (Regular → Business Internet):**
1. What triggers an upgrade request? (Customer request, sales process, usage pattern?)
2. Who performs the manual Verizon account migration? (Ops team, specific role?)
3. What is the step-by-step admin workflow?
   - Step 1: Receive upgrade request
   - Step 2: Perform manual Verizon account migration (5-step process above)
   - Step 3: Update portal: Change device.verizon_account_name from 'regular' to 'fwa'
   - Step 4: Update portal: Assign device to Business Internet service plan
   - Step 5: Verify device connectivity and configuration
4. Should portal show warning when admin changes account field to confirm manual process completed?
5. Should we require a "reason" field when account is changed?

**For Future Downgrades (Business Internet → Regular):**
1. Under what circumstances would we downgrade? (Customer request, contract change?)
2. Same manual Verizon process required (but reversed)
3. Same portal workflow but reversed direction
4. Additional considerations for downgrade?

**Portal's Role - Confirmed:**
Portal should be **tracking-only** with these capabilities:
- Display current account status (badge on device view)
- Allow admin users to UPDATE account field (after manual Verizon process complete)
- Show confirmation dialog when account field is changed: "Have you completed the manual Verizon account migration? Changing this field should only occur AFTER the device has been moved between Verizon accounts."
- Require confirmation/reason when changing account
- Validate that service plan matches new account (prevent saving incompatible combination)
- Document the manual process steps (link to KB article in confirmation dialog)
- Log all account changes for audit trail

**Workflow Documentation Needed:**
- [ ] Create KB article documenting 5-step manual Verizon migration process
- [ ] Define who is authorized to perform manual migrations
- [ ] Create admin checklist for upgrade/downgrade workflow
- [ ] Document rollback procedure if something goes wrong

**Decision Needed:**
1. Confirm portal scope (tracking-only - no automation)
2. Approve confirmation dialog design
3. Define admin workflow for future upgrades/downgrades

---

### 2.2 🔴 CRITICAL: Initial Migration - 15 Existing Business Internet Devices

**The Problem:**
- 15 devices **already migrated** to Business Internet Verizon account (manual process already completed)
- Portal doesn't know they're on Business Internet account
- Causing API failures (portal trying to use wrong account credentials)
- Missing IMEIs/SIMs for some
- Risk: Portal could push wrong configs before they're properly flagged
- Devon's concern: "Make sure there's no chance these devices are gonna somehow get a configuration"

**This is Different from Future Migrations:**
- These devices have ALREADY been manually moved in Verizon
- Portal just needs to be updated to reflect current reality
- No manual Verizon process needed - just portal data update

**Questions to Answer:**
1. Can we get a list of these 15 devices (serial numbers, IMEIs, etc.)?
2. What's their current state in the portal? (What company? What service plan?)
3. How should we identify them during code deployment?
4. What fail-safes do we need during deployment to prevent config pushes?
5. Timeline: When can we get this list?

**Migration Strategy Options:**

**Option A: Manual Update After Deployment**
- Deploy code with account field defaulting to 'regular'
- Manually update 15 devices to 'fwa' after deployment
- Manually assign to Business Internet service plan
- Pros: Simple, safe
- Cons: Window of time with incorrect data, API failures continue

**Option B: Pre-Migration Data Script**
- Get device list before deployment
- Include in migration script to set these devices to 'fwa'
- Set appropriate Business Internet service plan
- Pros: Clean migration, no data inconsistency window, immediate API fix
- Cons: Requires device list upfront (by end of week)

**Option C: Conservative Default with Admin Flagging**
- Deploy with all devices defaulting to 'regular'
- Add admin tool to bulk-flag Business Internet devices
- Prevent ANY config updates to unflagged devices until manually verified
- Pros: Safest, no risk of wrong configs
- Cons: Most complex, API failures continue until flagged

**Recommendation:** Option B if we can get list this week, otherwise Option C

**Action Items:**
- [ ] **Devon/Adam:** Provide list of 15 Business Internet devices (serial numbers minimum, ideally with current company/service plan)
- [ ] **Development:** Add fail-safe logic to prevent config pushes to unflagged devices during deployment window
- [ ] **Operations:** Document which devices are on which account for ongoing reference

---

### 2.3 🔴 MAJOR: Checkbox vs. Dropdown for Account Selection?

**The Debate:**

**Checkbox Approach:**
- ☑️ "FWA/Business Internet" checkbox
- Checked = FWA account, Unchecked = Regular account
- Pros: Simple, clean UI for 2 accounts
- Cons: Doesn't scale to 3+ accounts

**Dropdown Approach:**
- Dropdown with options: "Regular Account", "FWA Account"
- Pros: Scales to N accounts, more explicit
- Cons: Slightly more complex for simple 2-account case

**Context from Meeting:**
- Meeting leaned toward checkbox for simplicity
- Aksana raised scalability concern: "Checkbox is good for one or two, but is never good for one, two, or three" (lines 1187-1191)
- T-Mobile Business Internet mentioned as future consideration (uses different plan codes, not different accounts)
- Decision was deferred to this meeting

**Questions to Answer:**
1. How likely is a 3rd Verizon account in next 12-24 months?
2. Does T-Mobile BI solution need same treatment (Adam said "different plan codes, not different accounts")?
3. Cost of rework if we do checkbox now and need dropdown later?
4. Is there a hybrid approach? (e.g., radio buttons for scalability + simplicity)

**Options:**

**Option A: Checkbox (Simple Now)**
- Use checkbox for Verizon accounts
- Refactor to dropdown when 3rd account needed
- Pro: Faster to implement, cleaner UI
- Con: Technical debt if accounts expand

**Option B: Dropdown (Future-Proof)**
- Use dropdown from the start
- Works for 2 accounts, scales to N
- Pro: No rework needed later
- Con: Slight overkill for current 2-account case

**Option C: Radio Buttons (Middle Ground)**
- Radio buttons: ⚪ Regular Account  ⚪ FWA Account
- Visual simplicity of checkbox, scalability of multiple options
- Pro: Best of both worlds
- Con: Takes more vertical space

**Recommendation:** Option B (Dropdown) or Option C (Radio Buttons) for future-proofing

**Key Question:** Adam/Devon - How likely is 3rd Verizon account or similar T-Mobile needs in next 1-2 years?

---

### 2.4 🟡 MODERATE: Service Plan Form Field Requirements

**What We Know:**
- FWA service plans need minimal fields
- Don't need: Device group names, usage limits (mostly)
- Do need: Name, Verizon account indicator
- Should stay flexible for edge cases

**Questions to Answer:**
1. Should FWA checkbox/dropdown hide irrelevant fields dynamically?
2. Or just make fields optional and explain they're not needed?
3. Exception: T-Mobile unlimited technically has limits - keep usage limit field available?

**Form Behavior Options:**

**Option A: Dynamic Form (Hide Fields)**
- When FWA account selected, hide unnecessary fields
- Pros: Cleaner UI, less confusion
- Cons: More complex JavaScript, potential bugs

**Option B: Static Form (All Fields Visible, Most Optional)**
- All fields visible, but asterisks removed for FWA plans
- Help text explains which fields needed for FWA
- Pros: Simpler implementation, consistent UI
- Cons: Slightly cluttered, need good help text

**Option C: Separate Form for FWA Plans**
- Completely different form for FWA service plans
- Pros: Optimal UX for each account type
- Cons: Code duplication, harder to maintain

**Recommendation:** Option B (Static form with clear help text) for simplicity

**Proposed Field List for FWA Service Plans:**
- ✅ Required: Service Plan Name
- ✅ Required: Verizon Account (FWA)
- ✅ Required: Enable WiFi (yes/no)
- ✅ Required: Enable Firewall (yes/no)
- ⚪ Optional: Show Usage (probably yes)
- ⚪ Optional: Is Custom (for company-specific plans)
- ❌ Not Needed: Device Group Names
- ❌ Not Needed: Usage Limit (unless T-Mobile edge case)

**Decision Needed:** Confirm field list and form behavior

---

### 2.5 🟡 MODERATE: Terminology Standardization

**Current State:**
Meeting used multiple terms interchangeably:
- "FWA" (Fixed Wireless Access)
- "Business Internet"
- "BI"
- "Unlimited plans"

**Questions to Answer:**
1. What should be the PRIMARY term in the UI?
2. Should we use technical term (FWA) or business term (Business Internet)?
3. Do we need to explain the acronym anywhere?

**Options:**

**Option A: "Business Internet (FWA)"**
- User-facing: "Business Internet" or "Business Internet Account"
- Technical docs: Include "(FWA)" for clarity
- Pros: Business-friendly, self-explanatory
- Cons: Longer label

**Option B: "FWA Account"**
- Short and technical
- Pros: Concise, matches industry terminology
- Cons: May confuse non-technical users

**Option C: "Unlimited Account"**
- Describes the key benefit
- Pros: Customer-centric, clear value prop
- Cons: Doesn't distinguish from other potential unlimited plans

**Recommendation:** Option A - "Business Internet" for UI, "FWA" in technical docs

**Proposed Labels:**
- Device field: "Verizon Account Type"
  - Options: "Regular Account (Per-Byte)", "Business Internet Account (Unlimited)"
- Service plan field: "Verizon Account Compatibility"
  - Options: "Regular Account Only", "Business Internet Only", "Both Accounts"

**Decision Needed:** Confirm terminology for UI labels

---

### 2.6 🟢 MINOR: Error Message Standards

**What We Know:**
- System has good error handling foundation (recently improved)
- Need clear, actionable error messages for account mismatches

**Questions:**
1. How detailed should error messages be?
2. Should they include remediation steps?
3. Tone: technical or user-friendly?

**Example Scenarios:**

**Scenario 1: Single Device Edit**
- User tries to assign FWA service plan to regular device
- Error: "This service plan requires a Business Internet account, but this device is configured for Regular account. Please select a compatible service plan or contact support to upgrade this device's account."

**Scenario 2: Bulk Device Assignment**
- User tries to assign FWA plan to mix of 100 devices (80 regular, 20 FWA)
- Error: "Cannot assign service plan: 80 device(s) are incompatible. This service plan requires Business Internet account. Incompatible devices: [list of serial numbers]. Please select only Business Internet devices or choose a compatible service plan."

**Scenario 3: Import Validation**
- Import includes invalid account value
- Error: "Row 15: Device with serial number 'ABC123' has invalid Verizon account 'business'. Must be 'regular' or 'fwa'. This row will be skipped."

**Decision Needed:** Approve error message tone and format

---

## Section 3: Key Insights from Meeting 1

### Insight 1: Portal Scope is Much Simpler Than PRD Assumed
**What This Means:**
- PRD assumed portal would enable automated account switching
- Reality: Portal just tracks which account device is on
- This is actually GOOD NEWS for development:
  - Simpler implementation
  - Faster delivery
  - Lower risk
  - Fewer edge cases

**Impact:**
- Remove automation features from scope
- Focus on tracking, validation, and reporting
- Document manual process for admins

---

### Insight 2: Service Plans Are Safety-Critical, Not Just Business Rules
**What This Means:**
- Wrong service plan = wrong APN config
- Wrong APN = device won't connect
- This isn't just about billing accuracy - it's about device functionality

**Impact:**
- Validation is a SAFETY mechanism, not nice-to-have
- Must be fail-safe (block invalid assignments, don't just warn)
- Testing must be thorough

---

### Insight 3: 15 Orphaned Devices Are a Deployment Risk
**What This Means:**
- Devices currently in limbo state
- If portal pushes configs before they're properly flagged, devices will break
- Devon explicitly concerned about this

**Impact:**
- Need explicit fail-safes during deployment
- Conservative approach: Don't push ANY configs to unverified devices
- Get device list ASAP

---

### Insight 4: Team Has Good Foundation to Build On
**What This Means:**
- Recent improvements to bulk operations provide robust error handling
- Pattern already exists for validation with detailed error messages
- Don't need to build error handling from scratch

**Impact:**
- Faster development
- Consistent UX
- Leverage existing patterns

---

## Section 4: Updated Scope Summary

### ✅ In Scope (Confirmed)

**Database & Configuration:**
- Add `verizon_account_name` field to devices table
- Add `verizon_account_name` field to service plans table
- Store multiple Verizon account credentials in config
- Migration to populate existing devices with 'regular' default

**UI - Device Management:**
- Device create/edit form: Verizon account selector (checkbox or dropdown - TBD)
- Device view page: Display account with badge
- Device list: Filterable account column
- Device import: Add column for Verizon account

**UI - Service Plan Management:**
- Service plan create/edit: Account compatibility selector
- Simplified form for FWA plans (fewer required fields)
- Service plan view: Display account compatibility

**Business Logic - Validation:**
- Single device edit: Validate service plan compatibility
- Bulk device assignment: Validate all devices compatible
- Import validation: Check account values
- Clear, actionable error messages with device identification

**Business Logic - API Routing:**
- Update VerizonApi class to accept account parameter
- Update 15+ locations that instantiate VerizonApi
- Automatic routing based on device.verizon_account_name

**Audit & Logging:**
- Log device account changes
- Log service plan assignments
- Log API calls with account used (for debugging)

---

### ❌ Out of Scope (Removed/Deferred)

**Removed - Customer Self-Service FWA Upgrade:**
- No button for customers to upgrade themselves
- No confirmation modal for customer upgrade
- No automated account switching
- **Possible future:** "Request upgrade" button that creates ticket

**Removed - Admin Automated Account Migration:**
- No "Admin Override" checkbox to automate account switching
- Admin cannot click button to move device between accounts
- Portal does not trigger Verizon API calls for account migration

**Changed - Admin Account Tracking:**
- Admin CAN update account field in portal (after manual Verizon process)
- Portal tracks/displays account status
- May include warnings/documentation about manual process
- **This is tracking, not automation**

---

### To Be Decided (Pending This Meeting)

- Checkbox vs. dropdown UI design
- Exact service plan form field requirements
- 15 device migration strategy
- Error message standards
- Terminology for UI labels
- Portal's role in account field updates (tracking only? warnings?)

---

## Section 5: Proposed Decisions & Recommendations

### Decision 1: Portal Role
**Recommendation:** Portal is TRACKING-ONLY with these capabilities:
- ✅ Display current Verizon account for each device
- ✅ Allow admin to UPDATE account field (after manual Verizon process)
- ✅ Show confirmation dialog when account changed: "Have you completed the manual Verizon account migration process? This should only be updated after the SIM has been manually moved between Verizon accounts."
- ✅ Require reason/notes field when changing account
- ✅ Log all account changes
- ✅ Link to KB article documenting manual process
- ❌ Do NOT trigger any automated Verizon API calls for account migration
- ❌ Do NOT show "upgrade" or "migrate" buttons to customers

**Rationale:** Matches operational reality, reduces risk, simpler implementation

---

### Decision 2: UI Design
**Recommendation:** Dropdown approach for future-proofing

**Implementation:**
```
Verizon Account Type: [Dropdown]
  - Regular Account (Per-Byte Pricing)
  - Business Internet Account (Unlimited)
```

**Alternative if team prefers:** Radio buttons as middle ground

**Rationale:**
- Scales to 3+ accounts without rework
- Explicit and clear
- Small UI complexity increase for significant future flexibility
- T-Mobile BI may need similar treatment

---

### Decision 3: 15 Device Migration
**Recommendation:** Pre-migration data script (if device list available this week)

**Steps:**
1. Get list of 15 FWA devices from Devon/Adam (by end of week)
2. Create migration script to flag these devices as 'fwa'
3. Add fail-safe: Prevent config updates to devices with NULL account until explicitly set
4. Deploy with devices pre-flagged
5. Monitor API logs after deployment

**Fallback:** If list not available, deploy with all devices as 'regular' and manually update after deployment

---

### Decision 4: Service Plan Form
**Recommendation:** Static form with optional fields and good help text

**Required Fields for FWA Plans:**
- Service Plan Name
- Verizon Account Compatibility: "Business Internet Only"
- Enable WiFi
- Enable Firewall

**Optional Fields (available but not required):**
- Show Usage
- Is Custom
- Usage Limit (for edge cases)

**Hidden/Not Used:**
- Device Group Names (hide or gray out for FWA plans)

---

### Decision 5: Terminology
**Recommendation:** "Business Internet" for user-facing UI, "FWA" in technical docs

**Labels:**
- Devices: "Verizon Account Type"
- Service Plans: "Verizon Account Compatibility"
- Values: "Regular Account" and "Business Internet Account"

---

## Section 6: Action Items from This Meeting

### Immediate (This Week):
- [ ] **Devon/Adam:** Provide list of 15 existing FWA devices (serial numbers, IMEIs)
- [ ] **Team:** Finalize UI design decision (checkbox vs. dropdown)
- [ ] **Team:** Approve updated scope and recommendations
- [ ] **Aksana:** Update PRD to reflect confirmed scope changes
- [ ] **Development:** Begin database schema design based on confirmed requirements

### Short-Term (Next Week):
- [ ] **Development:** Create detailed technical design document
- [ ] **Development:** Update VerizonApi class design
- [ ] **Development:** Plan migration script for 15 devices
- [ ] **QA:** Update test plan to remove automated upgrade tests
- [ ] **Operations:** Document manual account migration process (KB article)

### Medium-Term (Before Development):
- [ ] **Team:** Review and approve updated PRD
- [ ] **Team:** Review technical design
- [ ] **Team:** Create Jira tickets
- [ ] **Team:** Estimate timeline

---

## Section 7: Meeting Agenda (Recommended)

### Part 1: Critical Decisions (25 minutes)
1. **Portal scope confirmation** (10 min)
   - Confirm tracking-only approach
   - Approve confirmation dialog design
   - Discuss KB article for manual process

2. **UI design decision** (10 min)
   - Present options: checkbox, dropdown, radio buttons
   - Consider T-Mobile future needs
   - Make final decision

3. **15 device migration strategy** (5 min)
   - Can we get device list this week?
   - Approve migration approach
   - Assign action item

### Part 2: Details & Refinements (15 minutes)
4. **Service plan form requirements** (5 min)
   - Approve field list
   - Confirm optional vs. required

5. **Terminology & error messages** (5 min)
   - Approve "Business Internet" terminology
   - Quick review of error message examples

6. **Timeline & next steps** (5 min)
   - When can development start?
   - When do we need updated PRD?
   - Next check-in meeting

### Part 3: Open Discussion (10 minutes)
7. Any other concerns or questions
8. Confirm action items
9. Schedule next meeting if needed

---

## Section 8: Success Criteria for This Meeting

By the end of this meeting, we should have:
- ✅ Clear agreement on portal's role (tracking vs. automation)
- ✅ Final UI design decision (checkbox vs. dropdown vs. radio)
- ✅ Migration plan for 15 existing FWA devices
- ✅ Approved service plan form requirements
- ✅ Agreed terminology for UI labels
- ✅ Action items assigned with owners and deadlines
- ✅ Green light to update PRD and begin technical design

---

## Appendix A: Quick Reference - PRD Changes Needed

### Sections to Remove/Revise:
- ❌ Story 3.1: Customer Upgrades Device to FWA Plan (entire story)
- ⚠️ Story 3.3: Admin Override for FWA Reversion (change from automation to tracking)
- ❌ Test Case IT-2: Customer Upgrades Device to FWA Plan
- ❌ Test Case UAT-3: Customer Self-Service FWA Upgrade
- ⚠️ Test Case UAT-4: Customer Prevented from Downgrade (keep but simplify - no dropdown to prevent)

### Sections to Add:
- ✅ Admin account field update process (tracking after manual Verizon process)
- ✅ Confirmation dialog design for account changes
- ✅ Fail-safe logic for 15 existing FWA devices
- ✅ Link to manual migration process documentation

### Sections to Update:
- ⚠️ FR-3: Customer FWA Upgrade (remove automation, add tracking)
- ⚠️ Success Criteria (remove automation metrics)
- ⚠️ Implementation Plan (adjust phases to remove automation features)

---
