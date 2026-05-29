# WATM Trello Backlog — Review Report

**Query:** `project = WATM AND status = Draft AND labels = Trello`
**Date Pulled:** 2026-04-03
**Last Updated:** 2026-04-06
**Total Draft Tickets:** 18
**Additional Tickets Reviewed (Non-Draft):** 2

## Purpose

This report consolidates the Trello-labeled backlog for client review. It includes the full ticket list, plus summaries of research completed on specific tickets to support prioritization and scoping discussions.

---

## Tickets Reviewed — Research Summaries

These tickets have been analyzed in detail. Full research documents are linked for each.

### Device Alerts — WATM-1659 (In Progress, Flagged as Impediment)

**What it is:** New device alert types and email notifications for InHand router events (link changes, SIM switches, signal faults, etc.).

**What's done:**
- Alert reception from InHand routers via UDP — working
- Alert processing and database storage — working
- Error tracking for failed alerts — working
- 1 of 9+ alert types configured (LAN2 Link Up/Down)

**What's remaining:**
- Complete alert type setup (need firmware identifiers from InHand for remaining 8+ types)
- Alert viewing interface in the portal (no UI exists yet)
- Email notification triggering (alerts are stored but no emails are sent)
- Integration with existing Company Notifications system

**Key questions for client:**
- Are InHand devices already sending alert packets? If so, we can use failure logs to map identifiers.
- Final confirmed list of alert types to support?
- Should alert notifications extend the existing Company Notifications UI (recommended) or be a separate system?
- Who can view alert history? Who can configure alert notifications?
- Is the alert viewing UI in scope for initial delivery, or email notifications first?

**Confluence:**
- [Device Alerts — Client Summary](https://orases.atlassian.net/wiki/spaces/WATM/pages/3095855106/Device+Alerts+Client+Summary)
- [Device Alerts Processing and Notifications scope](https://orases.atlassian.net/wiki/spaces/WATM/pages/1955037199/Device+Alerts+Processing+and+Notifications+scope.)
- [New Device Alerts Processing and Notifications](https://orases.atlassian.net/wiki/spaces/WATM/pages/2132672597/New+Device+Alerts+Processing+and+Notifications)
- [Discovery - Alerts, TMO API - May 20, 2025](https://orases.atlassian.net/wiki/spaces/WATM/pages/2142666756/Discovery+-+Alerts+TMO+API+-+May+20+2025)
- [Check In (Alerts & Notifications) - March 5, 2025](https://orases.atlassian.net/wiki/spaces/WATM/pages/1955168282/Check+In+Alerts+Notifications+-+March+5+2025)

**Jira:** [WATM-1659](https://orases.atlassian.net/browse/WATM-1659)

**Local Research:** [DeviceAlerts/Research_WATM-1659.md](DeviceAlerts/Research_WATM-1659.md) | [DeviceAlerts/Device_Alerts_Client_Summary.md](DeviceAlerts/Device_Alerts_Client_Summary.md)

---

### Permissions & Roles Revisions

**What it is:** Review of all roles, permissions, and authorization rules in the WATM portal.

**Current state:** 4 roles exist (WATM Super Admin, WATM Admin, Company Super Admin, Company Admin). All admin accounts currently use Super Admin — the WATM Admin role is not actively used.

**Gaps identified:**
- No read-only customer role (e.g., for field technicians who just need to monitor)
- No granular admin roles (billing-only, config-only, etc.) — it's all-or-nothing
- WATM Admin role exists but needs review before it can be used to restrict access
- Company Admin has limited write access — may need adjustment

**Potential new roles to discuss:**
- Admin Viewer (read-only across all sections)
- Billing Admin / Config Admin (specialized admin access)
- Customer Viewer (read-only customer portal)
- Customer Technician (device management only)

**Confluence:**
- [Roles and permissions expansion](https://orases.atlassian.net/wiki/spaces/WATM/pages/2132836404/Roles+and+permissions+expansion)
- [Roles and Permissions: Legacy documentation](https://orases.atlassian.net/wiki/spaces/WATM/pages/3100966915/Roles+and+Permissions+Legacy+documentation)
- [Permissions Overview](https://orases.atlassian.net/wiki/spaces/WATM/pages/459735080/Permissions+Overview)
- [CUB Account Types and User Roles](https://orases.atlassian.net/wiki/spaces/WATM/pages/367067273/CUB+Account+Types+and+User+Roles)
- [User Permissions](https://orases.atlassian.net/wiki/spaces/WATM/pages/334102557/User+Permissions)

**Local Research:** [Permissions-Revisions/Permissions_Roles_Analysis.md](Permissions-Revisions/Permissions_Roles_Analysis.md) | [Permissions-Revisions/Permissions_Roles_Client_Summary.md](Permissions-Revisions/Permissions_Roles_Client_Summary.md)

---

### WATM-1501 — Add the ability to resend W9 documents yearly (Draft)

**What it is:** Enable annual W9 re-collection from distributors via Adobe Sign, with both individual and bulk send capabilities.

**Key findings:**
- W9 is currently bundled with Distributor Agreement + 1099 Consent in a single Adobe Sign envelope — not sent independently
- "Resend" today sends a reminder on an existing agreement, not a new W9
- A fresh yearly W9 requires a new Adobe Sign envelope per distributor (costs 1 transaction each, ~33/year currently)
- No bulk send capability exists — today it's one-at-a-time from the company view page
- WATM-1491 is a duplicate ticket — should be consolidated

**Key questions for client:**
- W9 only, or full 3-document package for yearly renewal?
- Is bulk send worth the effort at current scale (33 distributors)?
- Should old signed W9s be kept on file or replaced?
- Manual trigger or automated annual process?

**Confluence:**
- [W9 document enhancements](https://orases.atlassian.net/wiki/spaces/WATM/pages/3095560194/W9+document+enhancements)
- [W9 Form Document](https://orases.atlassian.net/wiki/spaces/WATM/pages/1198718980/W9+Form+Document)
- [Distributor Agreement](https://orases.atlassian.net/wiki/spaces/WATM/pages/1313767453/Distributor+Agreement)

**Jira:** [WATM-1501](https://orases.atlassian.net/browse/WATM-1501) | [WATM-1491 (duplicate)](https://orases.atlassian.net/browse/WATM-1491)

**Local Research:** [WATM-1501/Analysis.md](WATM-1501/Analysis.md)

---

### WATM-1953 — Deactivated Device CleanUp Command (Needs Review and Estimate)

**What it is:** Bug — the cleanup command incorrectly removed an active device from billing.

**What happened:** On 11/12/25, `CleanUpDeactivatedDevicesForCurrentCycleCommand` set `charge_for_current_cycle = 0` on device RF3022109011248 (Digital Music Systems, Inc). The device was NOT actually deactivated — it remained active and consumed 47GB of unbilled data.

**The problem:** The command does not verify a device's actual `device_status_id` before removing it from billing. It assumes its query criteria are sufficient.

**What needs investigation:**
- What query/condition does the command use to identify devices? Why was this device included?
- Was there a failed carrier deactivation or a status that was set and reverted?
- Device status history in `o_logs` around 11/12/25

**Recommended fix:** Add status verification before flipping billing flags; log/alert when a device matches cleanup criteria but isn't actually deactivated.

**Jira:** [WATM-1953](https://orases.atlassian.net/browse/WATM-1953) — ticket description updated with investigation notes

**Local Research:** [WATM-1953/Investigation.md](WATM-1953/Investigation.md)

---

### WATM-1981 — Annual Commissions Report: Add Tax ID and Update Addresses (Draft)

**What it is:** Add Tax ID column and split address into separate columns on the Annual Commissions report.

**Key findings:**
- Address split is trivial — data is already stored in separate columns, just needs the CSV export changed
- Tax ID is NOT stored in the database — it's only available via Adobe Sign API from signed W9 forms
- Two options: (A) fetch from Adobe Sign at report time (slow, API-dependent) or (B) store encrypted Tax ID in the companies table (recommended)
- HTML view and CSV export currently show different column sets

**Key questions for client:**
- Which report format — HTML view, CSV export, or both?
- Should we add an encrypted Tax ID field to the company record?
- What about distributors without a W9 on file?
- Include Address Line 2 as a separate column?

**Note:** If WATM-1501 (yearly W9 resend) is built with Tax ID extraction, it would solve this ticket's Tax ID requirement at the same time.

**Confluence:**
- [W9 document enhancements](https://orases.atlassian.net/wiki/spaces/WATM/pages/3095560194/W9+document+enhancements)
- [W9 Form Document](https://orases.atlassian.net/wiki/spaces/WATM/pages/1198718980/W9+Form+Document)

**Jira:** [WATM-1981](https://orases.atlassian.net/browse/WATM-1981)

**Local Research:** [WATM-1981/Analysis.md](WATM-1981/Analysis.md)

---

### WATM-2038 — Additional Status (Draft)

**What it is:** Remove the "Offline" option from Additional Status for devices with manufacturer SIM/Systech and models M5/M5-302.

**Key findings:**
- The system already uses per-model capability flags but has no mechanism to control which statuses are available per manufacturer/model
- Additional status dropdown is always the full list — no filtering logic exists

**Two options:**
- **Option 1 — Hardcoded filtering:** Quick fix in the controller. Fast to implement but requires code deployment for any future changes.
- **Option 2 — Model-level configuration (recommended):** Admin-managed join table so the client can control which statuses are available per device model without dev involvement. Follows existing capability flag patterns.

**Key question:** Is this a one-off request, or will more status restrictions be needed? That determines which option is appropriate.

**Jira:** [WATM-2038](https://orases.atlassian.net/browse/WATM-2038)

**Local Research:** [WATM-2038/Analysis.md](WATM-2038/Analysis.md)

---

## All Draft Tickets (Full List)

| # | Key | Summary | Type | Priority | Status | Labels | Creator | Parent | Created |
|---|-----|---------|------|----------|--------|--------|---------|--------|---------|
| 1 | WATM-2038 | Additional Status | Trello Card | Normal | Draft | Trello | Laura Perry | — | 2026-03-27 |
| 2 | WATM-2004 | Device Group Mismatch Reporting | Story | Normal | Draft | Trello | Aksana Rahouski | WATM-1964: APW Support 2026 | 2026-02-24 |
| 3 | WATM-1981 | Annual Commissions Report: Add Tax ID and Update Addresses | Story | Normal | Draft | Trello, WATM_grooming | Laura Perry | WATM-2020: W9 document enhancements | 2026-01-22 |
| 4 | WATM-1806 | Investigate Twilio account | Task | Normal | Draft | Trello | Laura Perry | WATM-1964: APW Support 2026 | 2025-05-19 |
| 5 | WATM-1796 | Make Duplicates Impossible | Story | Normal | Draft | Trello | Laura Perry | WATM-1964: APW Support 2026 | 2025-05-08 |
| 6 | WATM-1714 | Revise Manage Device Page | Story | Normal | Draft | Trello | Laura Perry | WATM-1789: UX Improvements | 2025-01-28 |
| 7 | WATM-1713 | RMA Blank Slate | Story | Normal | Draft | Trello | Laura Perry | WATM-1787: APC Support 2024 | 2025-01-23 |
| 8 | WATM-1556 | Add SMS back as an option for 2FA and notifications | Story | Normal | Draft | Trello | Denise Schnabel | WATM-1964: APW Support 2026 | 2024-08-12 |
| 9 | WATM-1549 | Enhance white labeling feature | Story | Normal | Draft | Trello | Jon Mack | WATM-1489: Commissions P2 - Companies | 2024-08-07 |
| 10 | WATM-1501 | Add the ability to resend W9 documents yearly | Story | Normal | Draft | Trello | Jon Mack | WATM-2020: W9 document enhancements | 2024-07-05 |
| 11 | WATM-1491 | Add ability to resend W9 document to distributors | Story | Normal | Draft | Trello | Jon Mack | WATM-2020: W9 document enhancements | 2024-06-20 |
| 12 | WATM-1183 | Log-in "as" company | Story | Normal | Draft | Trello | Denise Schnabel | WATM-1964: APW Support 2026 | 2023-10-09 |
| 13 | WATM-1112 | Adjust 2FA codes timing | Story | Normal | Draft | Trello | Jon Mack | WATM-1964: APW Support 2026 | 2023-08-09 |
| 14 | WATM-994 | IDEA FOR MANUAL CARRIER SELECTION | Story | Normal | Draft | Trello | Jon Mack | WATM-1964: APW Support 2026 | 2023-07-06 |
| 15 | WATM-942 | Ability to add specific companies for a Global Notification | Story | Low | Draft | Trello | Jon Mack | WATM-1964: APW Support 2026 | 2023-06-02 |
| 16 | WATM-938 | make things on dashboard "clickable" | Story | Normal | Draft | Trello | Jon Mack | WATM-1789: UX Improvements | 2023-05-31 |
| 17 | WATM-930 | Optimize Mobile UX | Story | Low | Draft | Trello | Jon Mack | WATM-1789: UX Improvements | 2023-05-26 |
| 18 | WATM-758 | Update left nav options | Story | Low | Draft | Trello | Jon Mack | WATM-1789: UX Improvements | 2023-03-31 |

### Additional Tickets Reviewed (Not in Draft)

| Key | Summary | Type | Priority | Status | Notes |
|-----|---------|------|----------|--------|-------|
| WATM-1953 | Deactivated Device CleanUp Command | Bug | Normal | Needs Review and Estimate | Billing flag set on active device — needs investigation |
| WATM-1388 | Slowness with filter section on Browse Devices page | Story | Normal | Needs Review and Estimate | Search performance optimization — skipped for now |

---

## By Year Created

### 2026 (3 tickets)
- WATM-2038 — Additional Status
- WATM-2004 — Device Group Mismatch Reporting
- WATM-1981 — Annual Commissions Report: Add Tax ID and Update Addresses

### 2025 (4 tickets)
- WATM-1806 — Investigate Twilio account
- WATM-1796 — Make Duplicates Impossible
- WATM-1714 — Revise Manage Device Page
- WATM-1713 — RMA Blank Slate

### 2024 (4 tickets)
- WATM-1556 — Add SMS back as an option for 2FA and notifications
- WATM-1549 — Enhance white labeling feature
- WATM-1501 — Add the ability to resend W9 documents yearly
- WATM-1491 — Add ability to resend W9 document to distributors

### 2023 (7 tickets)
- WATM-1183 — Log-in "as" company
- WATM-1112 — Adjust 2FA codes timing
- WATM-994 — IDEA FOR MANUAL CARRIER SELECTION
- WATM-942 — Ability to add specific companies for a Global Notification
- WATM-938 — make things on dashboard "clickable"
- WATM-930 — Optimize Mobile UX
- WATM-758 — Update left nav options

---

## Summary Stats

- **By Status:** Draft: 18
- **By Priority:** Normal: 15, Low: 3 (WATM-942, WATM-930, WATM-758)
- **By Creator:** Jon Mack: 9, Laura Perry: 6, Denise Schnabel: 2, Aksana Rahouski: 1
- **With Parent Epic:** 17 tickets
- **Without Parent:** 1 ticket (WATM-2038)
- **Tickets with completed research:** 6 (Device Alerts, Permissions, WATM-1501, WATM-1953, WATM-1981, WATM-2038)
- **Duplicate tickets identified:** WATM-1491 duplicates WATM-1501
