# Product Requirements Document
## Company Invitations: Visibility & Status Tracking (Phase 1 + Phase 2)

**Document Version:** 1.1
**Date:** 2026-03-05
**Author:** Aksana
**Status:** Draft
**Related Tickets:** TBD
**Document Owner:** Aksana
**Last Updated:** 05.03.2026

---

## Table of Contents

1. [Overview](#overview)
2. [Background and Problem Statement](#background-and-problem-statement)
3. [Goals and Success Criteria](#goals-and-success-criteria)
4. [Users and User Stories](#users-and-user-stories)
5. [Acceptance Criteria](#acceptance-criteria)
6. [Scope](#scope)
7. [Functional Requirements and Business Rules](#functional-requirements-and-business-rules)
8. [Technical Requirements](#technical-requirements)
9. [UI/UX Requirements](#uiux-requirements)
10. [Testing Requirements](#testing-requirements)
11. [Dependencies and Risks](#dependencies-and-risks)
12. [Open Questions](#open-questions)
13. [References](#references)
14. [Notes](#notes)

---

## Overview

The Company Invitations feature adds a read-only dashboard to APCommand that exposes the existing `company_invites` table data through a searchable, filterable UI. Today, invitation data is stored in the database but never displayed — admins have no visibility into who has been invited, whether they accepted, or which invitations are still outstanding. Phase 1 delivers a basic invitation list. Phase 2 adds status tracking (Pending / Accepted / Expired), summary metrics, filtering, and a direct link from an accepted invitation to the created company record.

### Key Features
- **Invitation List (Phase 1):** Paginated table of all sent invitations with email, account type, sent date, and sender
- **Status Tracking (Phase 2):** Real-time status per invitation (Pending or Accepted) computed from database state
- **Summary Dashboard (Phase 2):** Aggregate metrics — total sent, pending count, accepted count, acceptance rate
- **Filtering and Search (Phase 2):** Filter by status, account type, and date range; search by email

### Business Impact
- Eliminates the "blind spot" in the customer onboarding process by surfacing data that already exists in the system
- Enables admins and Customer Super Admins to identify invitations that need follow-up without manual database queries
- Provides measurable conversion visibility (invitation → company account) to support onboarding decisions

---

## Background and Problem Statement

### Current State

The `company_invites` table stores every invitation sent since the feature was introduced (migration `20240425172444_CreateCompanyInvites.php`). The table captures email address, account type, parent company (for subcustomer invitations), a UUID token, and timestamps. When an invitation is sent, the record is saved and an email is dispatched automatically via the `afterSave` hook in `CompanyInvitesTable`.

There is no index action in `CompanyInvitesController` — the controller exposes only the `invite` (send) action. No UI page exists to view, search, or filter sent invitations. There is also no field tracking whether an invitation was accepted or which company was created from it.

### Problems

**Problem 1: No Invitation Visibility**
- Admins cannot see the list of invitations they have sent without direct database access
- There is no way to answer "Did we invite this person?" or "Who have we invited recently?" from within APCommand
- Affected users: System Administrators, Customer Super Admins
- Business impact: Duplicate invitations, missed follow-ups, inability to audit the onboarding pipeline

**Problem 2: No Conversion Status**
- Even with a list, there is currently no way to determine whether a given invitation was accepted (i.e., led to a company registration) or is still awaiting a response
- Identifying conversions requires a manual JOIN across `company_invites`, `o_users`, and `companies` tables, matching on email — a fragile, manual process
- Affected users: System Administrators
- Business impact: No measurable acceptance rate, inability to identify stale/outstanding invitations without engineering involvement

### Impact if Not Addressed

Admins continue to operate blind: they cannot track onboarding effectiveness, may send duplicate invitations, and have no way to prioritize follow-up outreach. The data to solve this already exists; failure to expose it means unnecessary manual work and reduced confidence in the onboarding process.

---

## Goals and Success Criteria

### Primary Goals

1. **Visibility:** Provide a single page in APCommand where all sent invitations are visible without database access
2. **Status Awareness:** Make it immediately clear whether each invitation is Pending, Accepted, or Expired
3. **Conversion Traceability:** Allow a user to navigate directly from an accepted invitation to the company account it created

### Success Criteria

- A user can navigate to the Company Invitations page and see all invitations sent from the system (or their own, depending on role) without any database query
- Each invitation displays its current status (Pending or Accepted) derived from live data
- Accepted invitations include a clickable link to the associated company record
- The summary bar shows correct total, pending, accepted, and acceptance-rate counts
- A user can filter the list by status, account type, and date range, and search by partial email address
- Historical invitations (sent before this feature ships) display as "Pending" by default and update to "Accepted" if a matching company registration is found
- Invitations do not expire; all unaccepted invitations remain "Pending" indefinitely

### Out of Scope

- Resend invitation action (Phase 3)
- Revoke/cancel invitation action (Phase 3)
- Bulk invitation upload via CSV (Phase 3)
- Export to Excel (Phase 3)
- Detailed analytics dashboard beyond the summary bar (Phase 3)
- Changes to the invitation sending flow or email template
- Changes to the recipient registration experience

---

## Users and User Stories

### Primary Users

**System Administrator (Admin)**
- **Role:** Platform-level administrator with access to all companies and all invitations
- **Need:** See every invitation sent across the entire system, identify pending invitations, and navigate to accepted companies
- **Pain Point:** Currently has zero UI visibility into the invitation pipeline; must query the database directly
- **Benefit:** Can monitor the entire onboarding funnel from one page

**Customer Super Admin**
- **Role:** Company-level administrator who can invite Subcustomers under their own company
- **Need:** See the invitations they personally sent to Subcustomers
- **Pain Point:** Cannot confirm whether their Subcustomer invitations were received, pending, or accepted
- **Benefit:** Can track their own subcustomer onboarding without contacting system admins

### User Stories

**As a System Administrator:**
- As a System Administrator, I need to view a list of all company invitations sent through APCommand so that I can monitor the onboarding pipeline without database access
- As a System Administrator, I need to see the status (Pending or Accepted) for each invitation so that I can identify which invitations require follow-up
- As a System Administrator, I need a summary of total, pending, and accepted invitation counts so that I can quickly assess onboarding conversion health
- As a System Administrator, I need to filter invitations by status, account type, and date range so that I can focus on specific segments of the invitation pipeline
- As a System Administrator, I need to search for an invitation by email address so that I can quickly look up a specific invitee
- As a System Administrator, I need to click through from an accepted invitation to the associated company record so that I can navigate seamlessly between invitation and company data

**As a Customer Super Admin:**
- As a Customer Super Admin, I need to see the subcustomer invitations I have sent so that I can track my own onboarding activity
- As a Customer Super Admin, I need to see the status (Pending or Accepted) of my subcustomer invitations so that I can know whether to follow up externally

---

## Acceptance Criteria

**Phase 1 — Invitation List:**
- [ ] Navigating to `/admin/companies/company-invites/index` renders a paginated table of sent invitations
- [ ] The table displays: Email Address, Account Type, Sent Date, and Sent By (user who created the invitation)
- [ ] The table is sortable by each column (ascending/descending)
- [ ] A search field filters visible rows by partial or full email address match
- [ ] Pagination controls allow navigating through all records (page size: 25 records per page)
- [ ] A System Administrator sees all invitations in the system
- [ ] A Customer Super Admin sees only invitations where `parent_company_id` matches their company
- [ ] The page includes a "+ Send New Invite" button that links to the existing invite form at `/admin/companies/company-invites/invite`
- [ ] Users without the appropriate role receive a 403 response when accessing the page

**Phase 2 — Status Tracking:**
- [ ] The table includes a Status column displaying one of: Pending or Accepted
- [ ] Status values are color-coded: Pending = blue badge, Accepted = green badge
- [ ] An invitation with a linked `company_id` (accepted) shows status "Accepted"
- [ ] An invitation without a linked `company_id` shows status "Pending" regardless of age (invitations do not expire)
- [ ] Historical invitations (pre-feature, no `company_id`) that match a company registration by email show status "Accepted" with the linked company
- [ ] Historical invitations with no matching company registration show status "Pending"
- [ ] Accepted invitations display a "View Company" link in the Action column that navigates to the company detail page
- [ ] Pending invitations display "Follow up" as a non-interactive label in the Action column (no action; label only)
- [ ] A summary bar above the table shows: Total, Pending count, Accepted count, and Acceptance Rate (Accepted / Total, as a percentage)
- [ ] Summary bar counts always reflect the full scope of invitations for the current user and are NOT affected by active filters
- [ ] Filter controls allow filtering by: Status (All / Pending / Accepted), Account Type (All / Distributor / Customer / Subcustomer), and Date Range (Sent Date)
- [ ] Filters and search can be combined (AND logic)
- [ ] The page URL updates to reflect active filters so filtered views can be bookmarked or shared

---

## Scope

### In Scope

**UI Changes**
- New index page: `/admin/companies/company-invites/index`
- Summary metrics bar (Phase 2)
- Filterable, searchable, sortable data table
- Status badge column (Phase 2)
- Action column with "View Company" link for Accepted invitations (Phase 2)

**Backend Changes**
- New `index` action on `CompanyInvitesController`
- Status computation logic (Pending / Accepted / Expired) based on `company_id`, `accepted_at`, `expires_at`
- Scope/permission enforcement: admins see all; Customer Super Admins see only their `parent_company_id` rows
- Backfill logic: on first load (or via migration), attempt to match historical invitations to companies by email to populate `company_id` and `accepted_at`

**Database Changes (Phase 2)**
- Migration adding columns to `company_invites`: `company_id`, `accepted_at`, `status`, `sent_by_user_id`
- Update to `CompaniesController::invitation()` to set `company_id`, `accepted_at`, and `status = 'accepted'` on successful registration

**Permission Changes**
- Add index route to RBAC permissions for admin role and customer role (scoped)

---

## Functional Requirements and Business Rules

### FR-1: Invitation List Display

**FR-1.1: Page Access**
- The system shall provide an index page at `/admin/companies/company-invites/index` accessible to authenticated users with the admin or customer role
- The page shall be linked from the existing invite form and from the Companies navigation menu

**Business Rules:**
- A user with role `admin` sees all `company_invites` records regardless of `parent_company_id`
- A user with role `customer` (Customer Super Admin) sees only records where `parent_company_id` = their company's ID
- Any other authenticated role receives a 403 Forbidden response

**FR-1.2: Table Columns — Phase 1**
The table shall display the following columns:

| Column | Source | Notes |
|---|---|---|
| Email Address | `company_invites.email` | Linked to search |
| Account Type | `account_types.title` via `account_type_id` FK | |
| Sent Date | `company_invites.created` | Formatted as MMM D, YYYY |
| Sent By | `o_users.first_name + last_name` via `sent_by_user_id` | "Unknown" for historical records with no `sent_by_user_id` |

**Business Rules:**
- All columns shall be sortable (ascending/descending) with sort state persisted in URL query parameters
- Default sort: Sent Date descending (most recent first)
- Pagination: 25 records per page; total record count displayed

**FR-1.3: Email Search**
- The system shall provide a text search field that filters invitations by partial or full email address match (case-insensitive LIKE query)
- Search executes on form submission (not on keypress)

**Business Rules:**
- Search applies within the current user's permitted scope (admin: all; customer: their parent_company_id only)
- Search combines with active filters using AND logic

---

### FR-2: Status Tracking (Phase 2)

**FR-2.1: Status Column**
- The system shall compute and display a status for each invitation record

**Status Definitions:**

| Status | Display | Condition |
|---|---|---|
| Accepted | Green badge | `company_id` IS NOT NULL (invitation was used to create a company) |
| Pending | Blue badge | `company_id` IS NULL (invitation has not yet been used; invitations do not expire) |

**Business Rules:**
- Status is derived at query time from `company_id`; it is not stored as a separate column to avoid sync drift
- The `status` column added to the database (per the migration) shall be updated only when transitions occur (invite accepted via registration flow) to support indexed queries; runtime status display overrides this with the derived logic above
- An invitation that is "Accepted" cannot transition back to Pending
- Invitations do not expire; all unaccepted invitations remain Pending indefinitely

**FR-2.2: Accepted Invitation — Company Link**
- For invitations with status Accepted, the Action column shall display a "View Company" link
- Clicking "View Company" navigates the user to the company detail page for the associated company (`/admin/companies/companies/view/{company_id}`)

**Business Rules:**
- The "View Company" link respects existing company view permissions; if the user does not have access to that company record, standard access control applies
- Customer Super Admins with role `customer` can only view companies under their hierarchy; the link will still navigate but access control is enforced at the destination

**FR-2.3: Summary Metrics Bar**
- The system shall display a summary bar above the table showing:
  - Total: count of all invitations in scope
  - Pending: count of Pending-status invitations in scope
  - Accepted: count of Accepted-status invitations
  - Acceptance Rate: `Accepted / Total * 100`, formatted as a percentage (0 decimal places; e.g., "68%")

**Business Rules:**
- Summary counts reflect the current user's permitted scope (admin: all; customer: their records only)
- Summary counts are NOT affected by active filters — they always show totals for the full scope
- If Total = 0, Acceptance Rate displays as "0%"

**FR-2.4: Filters**
- The system shall provide filter controls for:
  - **Status:** Dropdown — All (default), Pending, Accepted
  - **Account Type:** Dropdown — All (default), Distributor, Customer, Subcustomer
  - **Date Range:** Date range picker filtering on `company_invites.created` (Sent Date); both start and end are optional
- Filters apply to the table and pagination but NOT to the summary bar totals
- Active filter state is reflected in URL query parameters to support bookmarking

**Business Rules:**
- Filter combinations use AND logic (all selected filters must match)
- Clearing all filters returns the full scoped list

---

### FR-3: Registration Flow Update (Phase 2 — Backend)

**FR-3.1: Invitation Acceptance Recording**
- When a recipient successfully completes company registration via the invitation flow (`CompaniesController::invitation()`), the system shall update the originating `company_invites` record to set:
  - `company_id` = the newly created company's ID
  - `accepted_at` = current timestamp
  - `status` = `'accepted'`

**Business Rules:**
- The update shall occur within the same database transaction as company creation
- If the transaction fails and is rolled back, the `company_invites` record shall not be updated
- If the invitation token is not found (invalid or already used), the system behavior is unchanged from the current implementation (BadRequestException)

---

### FR-4: Historical Data Handling

**FR-4.1: Backfill on Migration**
- The database migration that adds `company_id`, `accepted_at`, `status`, and `sent_by_user_id` shall include a one-time backfill step that:
  1. For each existing `company_invites` record, attempts to find a matching company by joining `o_users.email = company_invites.email` and `companies.primary_user_id = o_users.id` where `companies.created >= company_invites.created`
  2. If exactly one match is found, sets `company_id` and `accepted_at` on the invite record
  3. If no match or multiple ambiguous matches exist, leaves `company_id` NULL (status = Pending)

**Business Rules:**
- The backfill is a best-effort operation; it does not guarantee 100% accuracy for historical data (email matching covers ~95% of cases per analysis)
- Backfill runs once in the migration; it is not re-run on subsequent deploys
- `sent_by_user_id` cannot be backfilled for historical records and will remain NULL; these display "Unknown" in the Sent By column

---

## Technical Requirements

### Performance & System
- The Company Invitations index page shall load in under 2 seconds for datasets up to 10,000 invitation records
- All filter and search queries shall execute in under 1 second
- The `company_invites` table should have indexes on: `email`, `parent_company_id`, `account_type_id`, `created`, and the new `company_id` column

### Security & Compliance
- Page access is gated by CakeDC/Auth RBAC rules defined in `config/permissions.php`
- The `index` action must be explicitly added to the allowed actions for `admin` and `customer` roles
- Customer Super Admin scope enforcement must be applied at the query level (not just the view layer) to prevent URL manipulation exposing other companies' invitations
- No sensitive data beyond email addresses is exposed on this page; existing data classification standards apply
- All queries use CakePHP ORM parameterized queries (no raw SQL in controllers)

### Data Model — New Columns on `company_invites`

**Migration:** Add to existing `company_invites` table

| Column | Type | Nullable | Default | Notes |
|---|---|---|---|---|
| `company_id` | INT(11) UNSIGNED | Yes | NULL | FK to `companies.id`; set on acceptance |
| `accepted_at` | DATETIME | Yes | NULL | Timestamp when invitation was accepted |
| `status` | ENUM('pending','accepted') | No | `'pending'` | Updated to `'accepted'` when company registers; runtime display derived from `company_id` |
| `sent_by_user_id` | INT(11) UNSIGNED | Yes | NULL | FK to `o_users.id`; set at invite creation |

**Note:** Status is derived at runtime from `company_id` for display accuracy. The stored `status` column supports indexed queries and is kept in sync by the registration flow update (FR-3.1). Confirm with engineering during technical design.

**Existing Schema (unchanged):**
- `id`, `token`, `email`, `account_type_id`, `parent_company_id`, `created`, `modified`

**FK References:**
```sql
ADD CONSTRAINT fk_company_invites_company
    FOREIGN KEY (company_id) REFERENCES companies(id) ON DELETE SET NULL;
ADD CONSTRAINT fk_company_invites_sent_by
    FOREIGN KEY (sent_by_user_id) REFERENCES o_users(id) ON DELETE SET NULL;
```

**Indexes to add:**
```sql
ADD INDEX idx_company_invites_email (email);
ADD INDEX idx_company_invites_company_id (company_id);
ADD INDEX idx_company_invites_created (created);
```

---

## UI/UX Requirements

### Key Interface Components

**Page: Company Invitations** (`/admin/companies/company-invites/index`)

**Phase 1 Layout:**
```
Company Invitations                              [+ Send New Invite]

Search: [__________________________]  [Search]

┌──────────────────────────────────────────────────────────────────┐
│ Email Address     | Account Type | Sent Date    | Sent By        │
├──────────────────────────────────────────────────────────────────┤
│ john@customer.com | Customer     | Feb 15, 2026 | Admin User     │
│ jane@dist.com     | Distributor  | Feb 10, 2026 | Admin User     │
│ bob@sub.com       | Subcustomer  | Feb 8, 2026  | Customer User  │
└──────────────────────────────────────────────────────────────────┘
[< Prev]  Page 1 of 4  [Next >]
```

**Phase 2 Layout (adds to Phase 1):**
```
Company Invitations                              [+ Send New Invite]

Summary:  Total: 47  |  Pending: 15  |  Accepted: 32 (68%)

Filters:  [Status: All ▼]  [Account Type: All ▼]  [Date Range: __ to __]
Search:   [__________________________]  [Search]  [Clear Filters]

┌────────────────────────────────────────────────────────────────────────┐
│ Email Address     | Account Type | Status        | Sent Date  | Action │
├────────────────────────────────────────────────────────────────────────┤
│ john@cust.com     | Customer     | [Pending]     | Feb 15     | -      │
│ jane@dist.com     | Distributor  | [Accepted]    | Feb 10     | View Co│
│ old@inactive.com  | Customer     | [Pending]     | Jan 1      | -      │
└────────────────────────────────────────────────────────────────────────┘
[< Prev]  Page 1 of 4  [Next >]
```

- Status badges use the Limitless theme's label classes: `label-info` (Pending), `label-success` (Accepted)
- "View Company" in the Action column is a standard anchor link; no modal or AJAX
- "Sent By" column is present in Phase 1 but not shown in the Phase 2 mockup — engineering to confirm column visibility with UX; recommendation is to keep it

### Key User Flows

- **View all invitations:** Admin navigates to Companies menu → Company Invitations → table renders with all records sorted by most recent
- **Find a specific invitee:** Admin types partial email in search field → submits → table filters to matching records
- **Identify pending invitations needing follow-up:** Admin selects "Pending" in Status filter → table shows only outstanding invitations sorted by age
- **Navigate to an accepted company:** Admin clicks "View Company" on an Accepted row → routed to company detail page

### Accessibility
- All table headers shall have `scope="col"` attributes
- Status badges shall include text content (not icon-only) so screen readers convey status
- Filter and search controls shall have associated `<label>` elements
- Keyboard navigation shall work for all interactive controls (tab order, enter to submit search/filters)

---

## Testing Requirements

### Test Plan Overview

**Testing Phases:**
1. Unit Testing (Development team) — model/table methods, status computation logic, scope enforcement
2. Integration Testing (QA team) — full page render, filter combinations, permission enforcement
3. User Acceptance Testing (Business stakeholders) — real-world scenarios with seeded data
4. Regression Testing (QA team) — existing invitation send flow unaffected

### Key Test Scenarios

**IT-1: Admin Sees All Invitations**
1. Seed 10 invitations across 3 different parent companies
2. Log in as admin role user
3. Navigate to `/admin/companies/company-invites/index`
4. **Verify:** All 10 invitations are displayed

**IT-2: Customer Super Admin Sees Only Own Invitations**
1. Seed 5 invitations for Company A and 3 for Company B
2. Log in as Customer Super Admin for Company A
3. Navigate to `/admin/companies/company-invites/index`
4. **Verify:** Exactly 5 invitations displayed; Company B invitations not visible

**IT-3: Status — Accepted**
1. Create an invitation; complete company registration using that invitation's token
2. Navigate to Company Invitations index
3. **Verify:** Invitation shows "Accepted" status with "View Company" link pointing to the correct company

**IT-4: Status — Pending (no expiration)**
1. Create an invitation; do not complete registration
2. Navigate to Company Invitations index
3. **Verify:** Invitation shows "Pending" status regardless of age

**IT-6: Filter by Status**
1. Seed invitations in both statuses (Pending and Accepted)
2. Select "Pending" from Status filter; submit
3. **Verify:** Only Pending invitations appear in table; summary bar unchanged (still shows full totals)

**IT-7: Email Search**
1. Seed invitations including `test@example.com` and `other@domain.com`
2. Search for "example"
3. **Verify:** Only `test@example.com` is returned

**IT-8: Summary Bar Accuracy**
1. Seed: 10 total, 4 Accepted, 6 Pending
2. Apply a filter (e.g., status = Accepted)
3. **Verify:** Table shows 4 rows; summary bar still shows Total: 10, Pending: 6, Accepted: 4, Rate: 40%

**IT-9: Registration Flow Updates Invite Record**
1. Create invitation; note the `company_invites.id`
2. Complete registration via the invitation token
3. Query `company_invites` where id = [id]
4. **Verify:** `company_id` is set to the new company's ID; `accepted_at` is populated; no other invitation records affected

**UAT-1: Admin Monitors Onboarding Pipeline**
- **Persona:** System Administrator
- **Scenario:** Admin wants to see which Distributor invitations from last month are still pending
- **Steps:** Navigate to Company Invitations; set Account Type = Distributor, Status = Pending, Date Range = Feb 1–28, 2026; review results
- **Success Criteria:** List shows only Distributor invitations from February that have not been accepted; admin can identify which to follow up on

**UAT-2: Customer Super Admin Tracks Subcustomer Invitations**
- **Persona:** Customer Super Admin
- **Scenario:** Super Admin sent 3 subcustomer invitations last week and wants to see if any were accepted
- **Steps:** Navigate to Company Invitations; review Status column for their invitations
- **Success Criteria:** All 3 invitations are visible with correct status; any accepted ones show "View Company" link

**RT-1: Existing Invitation Send Flow Unaffected**
- **Verify:** Navigating to `/admin/companies/company-invites/invite` and sending an invitation still works exactly as before
- **Verify:** Invitation email is still sent; record is created in `company_invites`; new columns are populated (`sent_by_user_id`, `status = 'pending'`)
- **Verify:** The recipient's registration flow still works end-to-end

**RT-2: Unauthorized Access Blocked**
- **Verify:** A user without admin or customer role receives 403 when accessing the index page
- **Verify:** A Customer Super Admin cannot access another company's invitations by manipulating URL parameters

### Testing Notes
- Test data should include invitations created before the migration (historical records) to validate backfill and Pending-fallback behavior
- UAT should use a staging environment with realistic (anonymized) invitation data seeded

---

## Dependencies and Risks

### Dependencies

**Must Exist Before Development:**
1. **Existing `company_invites` table** — all Phase 1 display logic reads from this table; no changes required before Phase 1 begins
2. **CakePHP RBAC permissions** (`config/permissions.php`) — `index` action must be added to allowed actions for admin and customer roles before the page is accessible | **Mitigation:** Permission change is low-risk and can be added independently
3. **`o_users` table / `orases/users` package** — `sent_by_user_id` FK references this table; package must be compatible with new FK | **Mitigation:** Existing FKs to `o_users` already exist in the schema; no new risk

**Required for Testing:**
- Company registration flow (existing feature) must be functional in the test environment to validate IT-9 and RT-1

**Integrates With:**
- `Companies` plugin — company detail page link for "View Company" action
- `orases/users` (`o_users` table) — Sent By display and `sent_by_user_id` FK
- CakeDC/Auth RBAC — permission enforcement for index action

### Risks

**LOW RISK: Historical Data Accuracy**
- **Description:** Backfill matching (by email) for pre-existing invitations may incorrectly link or fail to link some records if emails changed or multiple invitations were sent to the same address
- **Impact:** Low — affects historical display accuracy only; no new data is at risk
- **Probability:** Low — email matching covers ~95% of cases per analysis; edge cases are acceptable
- **Mitigation:** Document the limitation in the Notes section of the page; historical records marked Pending can be manually reviewed if accuracy is critical

**LOW RISK: Invitation Sending Flow Regression**
- **Description:** Updates to `CompaniesController::invitation()` to record `company_id` and `accepted_at` could introduce a regression in the registration flow
- **Impact:** High — registration is a core onboarding path; any breakage blocks new company creation
- **Probability:** Low — the change is additive (one update call added to an existing transaction)
- **Mitigation:** RT-1 regression test covers the full registration path end-to-end; deploy to staging first

**LOW RISK: Scope Enforcement Bypass**
- **Description:** A Customer Super Admin could attempt to enumerate other companies' invitation data by manipulating query parameters
- **Impact:** Medium — data privacy; invitation email addresses of other companies' invitees exposed
- **Probability:** Low — scoping must be enforced at the query/model layer, not the view layer
- **Mitigation:** IT-2 and RT-2 test cases explicitly validate scope enforcement; code review must confirm query-level scoping

---

## Open Questions

All questions have been resolved. No open items remain.

### Resolved Decisions

| # | Question | Decision | Date |
|---|---|---|---|
| 1 | Invitation expiration threshold | No expiration — invitations remain Pending indefinitely until accepted | 2026-03-05 |
| 2 | Access permissions scope | Option B — Admins see all; Customer Super Admins see only their own (scoped by `parent_company_id`) | 2026-03-05 |
| 3 | Expired invitation link behavior | N/A — removed; no expiration concept exists in this implementation | 2026-03-05 |
| 4 | Summary bar counts | Option A — always reflects full-scope totals; not affected by active filters | 2026-03-05 |

---

## References

### Supporting Documentation
- Client Proposal: `TrelloBacklog/CompanyInvites/CompanyInvite_ClientProposal.md`
- Technical Flow Analysis: `TrelloBacklog/CompanyInvites/CompanyInvite.md`
- Client Request Analysis: `TrelloBacklog/CompanyInvites/CompanyInvite_ClientRequest_Analysis.md`
- Quick Reference Summary: `TrelloBacklog/CompanyInvites/CompanyInvite_Summary.md`

### Related Code
- Controller: `plugins/Companies/src/Controller/Admin/CompanyInvitesController.php`
- Table: `plugins/Companies/src/Model/Table/CompanyInvitesTable.php`
- Registration Controller: `plugins/Companies/src/Controller/CompaniesController.php` (method: `invitation()`)
- Existing Migration: `watm/config/Migrations/20240425172444_CreateCompanyInvites.php`
- Permissions: `watm/config/permissions.php`

---

## Notes

### Evidence Sources
- All functional requirements derived from `CompanyInvite_ClientProposal.md` (client-facing proposal, March 3, 2026) and `CompanyInvite.md` (technical flow analysis)
- Database schema and column details drawn directly from `CompanyInvite.md` schema section and the analysis document's recommended SQL
- Status definitions, UI mockups, and permission model drawn from `CompanyInvite_ClientRequest_Analysis.md` (Option 2 recommendation)
- Backfill SQL approach validated against existing queries documented in `CompanyInvite.md` (Conversion Rate Analysis section)

### Key Decisions Made

- **PRD Type:** Implementation PRD — the feature design was approved in the client proposal; this document captures detailed requirements ready for development
- **Phase Scope:** Phase 1 (visibility) and Phase 2 (status tracking) are specified together per client recommendation ("Build Phase 1 + 2 together — most efficient")
- **No expiration:** Client confirmed invitations do not expire. Statuses are Pending and Accepted only. `expires_at` column is not added. No Expired status, no expiration threshold configuration, and no link-behavior logic needed. (2026-03-05)
- **Access permissions — Option B confirmed:** Admins see all invitations; Customer Super Admins see only invitations they sent (scoped by `parent_company_id`). Scope enforcement is at the query level. (2026-03-05)
- **Summary bar — Option A confirmed:** Summary counts always reflect the full scope for the current user and are never filtered by active filter selections. (2026-03-05)
- **Status derivation:** Status is computed at runtime from `company_id` (Accepted if set, Pending otherwise). A stored `status` ENUM column (`pending`, `accepted`) is included for indexed queries and kept in sync by the registration flow.
- **Historical records:** Backfill via email matching is best-effort; no manual remediation required; display as Pending if unmatched.

### Confidence Levels
- ✅ High confidence: All Phase 1 column definitions (data exists in table today, confirmed in technical analysis)
- ✅ High confidence: Role-based scoping (admin vs. customer) — confirmed by client (Option B, 2026-03-05)
- ✅ High confidence: Registration flow update requirements — directly informed by existing `CompaniesController::invitation()` code review
- ✅ High confidence: No expiration — confirmed by client (2026-03-05)
- ✅ High confidence: Summary bar shows unfiltered totals — confirmed by client (2026-03-05)
- ❓ Low confidence: `sent_by_user_id` for historical records — cannot be backfilled; "Unknown" fallback is an assumption pending client review

---

END OF DOCUMENT
