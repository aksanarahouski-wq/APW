# Configuration Versioning and Change Management

**Issue:** Configuration Versioning, Review/Approval Workflow, and Rollback Capability
**Priority:** HIGH - Critical for safe bulk changes and production rollback
**Status:** Solution Proposed - Awaiting Review
**Last Updated:** March 2, 2026

---

## Problem Statement

### Current Gap in Design

The Hierarchical Configuration Management System supports dynamic configuration resolution across 6 layers and conditional rules, but lacks a formal versioning and change management system.

### Why This Is Critical

**Bulk Changes Without Review:**
- Admins need to make multiple related changes (e.g., update firewall rules across layers, add new rules, adjust priorities)
- Currently no way to group changes together as a "draft" that can be reviewed before activation
- Risk of partial changes being applied if admin is interrupted mid-edit

**No Approval Workflow:**
- Configuration changes affect thousands of devices simultaneously
- Changes can permanently damage devices if invalid
- Need formal review/approval process before changes go live
- Multiple stakeholders may need to sign off (Engineering, QA, Operations)

**Cannot Roll Back:**
- If bad configuration is deployed, no way to quickly revert to last known good state
- Would require manually identifying and reversing all changes
- Time-critical when devices are affected in production

**Audit Trail Gaps:**
- Need to see complete change history with ability to compare versions
- Need to understand WHY changes were made (not just WHAT changed)
- Need to track WHO approved changes

### Real-World Scenarios

**Scenario 1: Service Plan Update**
1. Admin needs to update ATM service plan firewall rules
2. Changes affect: Service Plan Layer, Three-Way Rules, and Company-level exceptions
3. Changes impact 10,000+ devices
4. **Problem:** Admin makes changes over 2 days, forgets one rule → partial deployment breaks devices

**Scenario 2: Emergency Rollback**
1. New global DNS servers deployed via configuration update
2. Devices start failing to connect (DNS servers were incorrectly configured)
3. 50,000 devices affected, production outage
4. **Problem:** No quick rollback → must manually find and revert changes → hours of downtime

**Scenario 3: Multi-Team Approval**
1. Security team requests global firewall rule change
2. Engineering needs to validate technical correctness
3. QA needs to test on staging devices
4. Operations needs to approve production rollout timing
5. **Problem:** No formal workflow → changes applied without all stakeholders approving

---

## Proposed Solution

### Git-Like Configuration Versioning System

Implement a versioning system similar to Git workflows, where configuration changes go through defined stages with clear approval gates and rollback capability.

**Core Concept:**
- Configuration exists as versioned snapshots
- Admins work on "draft" versions without affecting production
- Changes go through formal review and approval process
- Activation is atomic (all devices updated together)
- Previous versions preserved for instant rollback

---

## Solution Overview

### Version Lifecycle

```
CREATE DRAFT → MAKE CHANGES → SUBMIT FOR REVIEW → GET APPROVALS → ACTIVATE → [ROLLBACK IF NEEDED]
    ↓                                                    ↓
 (saved)                                            (rejected)
```

**Five States:**
1. **DRAFT** - Work in progress, visible only to creator
2. **PENDING_REVIEW** - Submitted, awaiting stakeholder approvals
3. **APPROVED** - All stakeholders approved, ready to activate
4. **ACTIVE** - Currently deployed to devices
5. **ARCHIVED** - Previously active, superseded by newer version

---

## User Experience & Workflows

### Workflow 1: Creating a New Configuration Version

**Admin Experience:**

1. **Start New Version**
   - Click "New Configuration Version"
   - Give it a name: "Q1 2026 Firewall Update"
   - Add description: "Adding RMS server whitelist rules for CORD company"
   - System creates draft version, copying current active config as starting point

2. **Make Changes (Over Multiple Sessions)**
   - Edit Global Layer: Update DNS servers
   - Edit Service Plan Layer: Add firewall rules to ATM plan
   - Add Three-Way Rule: Model+Carrier+ServicePlan baseline
   - Add Four-Way Rule: CORD company exception
   - Save draft (can return later to continue editing)

3. **Preview Impact**
   - Click "Preview Affected Devices"
   - System shows: "This version will affect 12,450 devices"
   - Breakdown by change type shown
   - Admin reviews to ensure expected scope

4. **Submit for Review**
   - Click "Submit for Review"
   - System validates all changes
   - System generates side-by-side comparison (current vs proposed)
   - Status changes to PENDING_REVIEW
   - Notifications sent to required reviewers

### Workflow 2: Reviewing a Configuration Version

**Reviewer Experience:**

1. **Receive Notification**
   - Email: "New configuration version pending your review"
   - Subject: "v1.3.0 - DNS Server Migration"

2. **Review Changes**
   - Open version in review interface
   - See side-by-side comparison:
     - **OLD:** dns_primary_server = 8.8.8.8
     - **NEW:** dns_primary_server = 10.0.1.53
   - See affected device count per change
   - See validation results (all checks passed)
   - See other reviewer statuses:
     - ✅ Engineering (John) - Approved
     - ✅ QA (Sarah) - Approved
     - ⏳ Operations (You) - Pending

3. **Make Decision**
   - **APPROVE:** Add comments if desired, click "Approve"
   - **REJECT:** Add explanation, click "Reject" (version status → REJECTED)
   - **REQUEST CHANGES:** Add comments, request modifications

4. **Outcome**
   - If all reviewers approve → status becomes APPROVED
   - If any reviewer rejects → status becomes REJECTED (creator notified)
   - Version cannot activate until all required approvals received

### Workflow 3: Activating a Configuration Version

**Operator Experience:**

1. **Select Approved Version**
   - See list of approved versions ready for activation
   - Select "v1.3.0 - DNS Server Migration"

2. **Choose Deployment Strategy**
   - **Option A: Immediate Deployment**
     - All 52,341 devices updated on next check-in
     - Full deployment within 24 hours

   - **Option B: Gradual Rollout**
     - Phase 1: Deploy to 5% of devices (~2,617 devices)
     - Wait 24 hours, monitor for issues
     - Phase 2: Deploy to 25% of devices (~13,085 devices)
     - Wait 24 hours, monitor for issues
     - Phase 3: Deploy to 100% (all remaining devices)

3. **Review Rollback Plan**
   - System shows: "If activation fails, system will roll back to:"
   - **v1.2.0 - Q1 2026 Firewall Update** (current active version)
   - Confirm understanding

4. **Provide Justification**
   - Enter reason for change: "Migrating to internal DNS servers for improved performance"
   - Justification recorded in audit log

5. **Activate**
   - Click "Activate Version"
   - System performs atomic activation:
     - Current active version (v1.2.0) → ARCHIVED
     - New version (v1.3.0) → ACTIVE
     - All devices marked for config update
   - Confirmation shown: "Version v1.3.0 is now ACTIVE"

### Workflow 4: Emergency Rollback

**Emergency Response Experience:**

1. **Detect Problem**
   - Monitoring alerts: "Device connection failures spiking"
   - Investigation: DNS servers not responding
   - Decision: Need to rollback immediately

2. **Initiate Rollback**
   - Navigate to Configuration Versions
   - Current ACTIVE version: v1.3.0 (DNS Server Migration)
   - Click "Rollback"

3. **Select Previous Version**
   - System shows recent ARCHIVED versions:
     - **v1.2.0** - Last Active: 14 days (no issues reported) ✅ RECOMMENDED
     - **v1.1.0** - Last Active: 14 days (no issues reported)
   - Select v1.2.0

4. **Provide Rollback Reason**
   - Enter: "DNS servers not responding, devices unable to connect - Production outage"
   - Select: "Notify all stakeholders" + "Create incident report"

5. **Execute Rollback**
   - Click "EXECUTE ROLLBACK"
   - System performs atomic rollback:
     - v1.3.0 (problematic) → ARCHIVED
     - v1.2.0 (previous stable) → ACTIVE
     - All devices immediately switched back
   - Incident report created automatically
   - Stakeholders notified

6. **Result**
   - Devices receive v1.2.0 configuration on next check-in
   - Connection issues resolve within 1 hour
   - Total downtime minimized

---

## Key Features

### 1. Version Comparison

**What Users See:**
- Side-by-side view of current ACTIVE vs proposed version
- Changes highlighted by type:
  - 🟢 **ADDED:** New parameters or rules
  - 🟡 **MODIFIED:** Changed values
  - 🔴 **DELETED:** Removed parameters or rules
- Affected device count per change
- Clear visual hierarchy: Global changes shown first (highest impact)

**Example:**
```
GLOBAL LAYER CHANGES:
🟡 dns_primary_server
   OLD: 8.8.8.8
   NEW: 10.0.1.53
   Affects: 52,341 devices

SERVICE PLAN LAYER - ATM:
🟢 connection_timeout
   NEW: 300
   Affects: 12,450 devices (all ATM plan devices)
```

### 2. Multi-Stakeholder Review

**Review Roles:**
- **Engineering:** Technical validation (parameter correctness, syntax)
- **QA:** Testing validation (tested on staging devices)
- **Operations:** Production readiness (timing, rollout strategy)
- **Security:** Security validation (firewall rules, access controls)
- **Management:** Business approval (customer impact)

**Flexibility:**
- Not all roles required for every change
- Can define required reviewers per version
- Can skip review for minor changes (with proper permissions)

### 3. Gradual Rollout

**Why This Matters:**
- Test on small subset before full deployment
- Monitor for issues during rollout
- Reduce blast radius if problems occur
- Build confidence with phased approach

**User Control:**
- Admin chooses rollout percentages
- Admin sets wait time between phases
- System tracks progress and sends notifications
- Can pause/stop rollout at any phase

### 4. Version History

**What's Tracked:**
- Every version ever created (DRAFT, REJECTED, ACTIVE, ARCHIVED)
- Who created, who reviewed, who activated
- When each state transition occurred
- Why changes were made (description, justification)
- What devices were affected
- Validation results

**Searchability:**
- Search by version number, name, creator
- Filter by date range, status, reviewer
- Find versions affecting specific parameters
- Identify who made specific changes

### 5. Rollback Capability

**Key Characteristics:**
- **Instant:** One-click rollback to any previous version
- **Safe:** Previous versions preserved, fully validated
- **Audited:** Rollback reason required and logged
- **Notified:** Stakeholders automatically informed
- **Documented:** Incident report created automatically

---

## User Interface Concepts

### Version List Page

**Primary View:**
- Current ACTIVE version highlighted (green badge)
- Pending review versions with reviewer status (yellow badge)
- Draft versions in progress (blue badge)
- Archived versions (gray badge)

**Actions:**
- ACTIVE version: "View Details" | "Compare" | "Rollback"
- PENDING_REVIEW: "View Details" | "Review" | "Compare"
- DRAFT: "Continue Editing" | "Delete Draft"
- ARCHIVED: "View Details" | "Restore" (creates new draft)

**Quick Stats:**
- Version number and name
- Status badge
- Date activated (if applicable)
- Device count affected
- Last modified date/user

### Draft Editor

**Editing Experience:**
- Tabbed interface: Global | Model | Carrier | Plan | Company | Device
- Each tab shows parameters for that layer
- Changes highlighted (yellow background = modified, green border = new)
- Real-time affected device count updates
- Save draft button (saves progress without submitting)
- Validate button (runs validation checks)
- Submit for Review button (triggers workflow)

**Change Summary:**
- Always visible sidebar showing all changes made
- Organized by layer and rule type
- Click any change to jump to that section
- Shows running total of affected devices

### Review Interface

**Reviewer View:**
- Top section: Version metadata (name, description, creator, date)
- Middle section: Side-by-side comparison (current vs proposed)
- Bottom section: Review actions (Approve | Reject | Request Changes)
- Sidebar: Other reviewers and their status

**Change Navigation:**
- Jump between changes quickly
- Filter by layer or change type
- See only changes relevant to your role (optional)

### Activation Page

**Pre-Activation Checklist:**
- ✅ All required approvals received
- ✅ Validation checks passed
- ✅ Affected devices identified (52,341 devices)
- ✅ Rollback plan confirmed (v1.2.0)

**Deployment Options:**
- Radio buttons: Immediate vs Gradual
- If gradual: Phase percentages and wait times configurable
- Justification text area (required)
- Large "Activate Version" button

**Safety Features:**
- Warning message about device count
- Confirmation dialog (must type version number to confirm)
- Cannot activate if validation fails

---

## Integration with Validation Strategy

**Validation Happens At:**

1. **While Editing (Draft Stage):**
   - Input validation (instant feedback)
   - Layer validation (parameter allowed at this layer?)

2. **Before Submit (Draft → Pending Review):**
   - Complete validation suite runs
   - Impact calculation performed
   - Effective config validation (CRITICAL)
   - Submission blocked if validation fails

3. **During Review:**
   - Reviewers see validation results
   - Can re-run validation if needed
   - Approval contingent on passing validation

4. **Before Activation:**
   - Final validation check
   - Activation blocked if validation fails

**Key Insight:**
Versioning + Validation work together:
- **Versioning** provides workflow and rollback
- **Validation** prevents bad configs from reaching production
- Together they provide safe, controlled configuration management

---

## Benefits Summary

### For Administrators
✅ Make bulk changes without fear of partial deployment
✅ Work on drafts over multiple days without affecting production
✅ Preview impact before submitting for review
✅ Clear audit trail of what changed and why

### For Reviewers
✅ Clear side-by-side comparison of changes
✅ See validation results before approving
✅ Understand business justification
✅ Coordinate with other stakeholders

### For Operations
✅ Control deployment timing and strategy
✅ Test on small subset before full rollout
✅ Instant rollback if issues occur
✅ Complete change history for troubleshooting

### For the Organization
✅ Formal approval process for high-impact changes
✅ Reduced risk of configuration errors
✅ Faster recovery from issues (instant rollback)
✅ Complete audit trail for compliance

---

## Rollout Strategy

### Phase 1: Basic Versioning
- Version creation and draft editing
- Simple activation (immediate deployment only)
- Basic rollback to previous version
- No formal review workflow yet

### Phase 2: Review Workflow
- Multi-stakeholder review process
- Approval requirements
- Version comparison interface
- Review comments and feedback

### Phase 3: Advanced Features
- Gradual rollout capability
- Automated notifications and alerts
- Version branching (create draft from archived version)
- Advanced search and filtering

---

## Open Questions for Stakeholders

### 1. Review Requirements
- Which roles must review which types of changes?
- Can we skip review for minor changes? If so, define "minor"
- What happens if a reviewer is unavailable? (delegate, timeout, skip?)

### 2. Versioning Scheme
- Semantic versioning (v1.2.3) auto-generated OR custom names?
- How long to keep archived versions? (storage considerations)
- Should deleted versions be soft-deleted or hard-deleted?

### 3. Rollback Scope
- Always rollback entire version OR support partial rollback (specific layers only)?
- Who has permission to rollback? (ops only? admins? senior staff?)
- Should rollback require approval or be instant in emergencies?

### 4. Gradual Rollout
- Default rollout strategy per change type? (e.g., global changes default to gradual)
- How to handle device grouping for rollout? (random? by region? by customer?)
- What metrics to monitor during rollout? (error rate? connection rate?)

### 5. Draft Management
- Can multiple admins collaborate on same draft? (concurrent editing)
- Auto-save drafts every N seconds OR manual save only?
- How long to keep abandoned drafts? (auto-delete after 30 days?)

---

## Success Criteria

**Must Have (MVP):**
- ✅ Ability to create draft versions
- ✅ Ability to edit drafts without affecting production
- ✅ Ability to activate a draft (atomic operation)
- ✅ Ability to rollback to previous version
- ✅ Side-by-side comparison of versions
- ✅ Basic audit trail (who, what, when)

**Should Have (Phase 2):**
- ✅ Multi-stakeholder review workflow
- ✅ Approval requirements and tracking
- ✅ Validation integration
- ✅ Gradual rollout capability
- ✅ Rich audit trail (including WHY - justification)

**Nice to Have (Phase 3+):**
- ✅ Version branching (restore archived version)
- ✅ Concurrent editing with conflict resolution
- ✅ Advanced search and filtering
- ✅ Automated rollback triggers (if error rate exceeds threshold)
- ✅ Configuration diff viewer with syntax highlighting

---

## Next Steps

1. **Stakeholder Review:**
   - Engineering: Database design approach and technical feasibility
   - QA: Testing strategy for version workflow
   - Operations: Rollout process and rollback procedures
   - Management: Approval workflow and permissions model

2. **Answer Open Questions:**
   - Define review requirements and approval gates
   - Choose versioning scheme
   - Clarify rollback permissions and scope
   - Define default rollout strategies

3. **UI/UX Design:**
   - Create wireframes for key screens (version list, editor, review interface)
   - User testing with admins and reviewers
   - Refine workflows based on feedback

4. **Technical Design:**
   - Database schema design
   - API design for version operations
   - Performance considerations for large-scale deployments

5. **Implementation Planning:**
   - Break into phases (MVP → Phase 2 → Phase 3)
   - Estimate effort per phase
   - Create implementation tickets

---

**Status:** Solution Proposed - Awaiting Stakeholder Review
**Priority:** HIGH - Critical for production safety and bulk change management
**Document Owner:** Aksana Rahouski
**Last Updated:** March 2, 2026
