# APW Check-in - June 23, 2026 - Summary

**Date:** June 23, 2026
**Duration:** ~41 minutes
**Attendees:** Devon D'Andrea (APW), Laura Perry (Orases PM), Aksana Rahouski (Orases BA/PM), Richard Sacco (Orases Dev), Aaron Diefes (Orases Dev), Stone Marballie (Orases Dev), Adam Curcie (APW), Jon (APW)
**Recording:** [tldv](https://tldv.io/app/meetings/6a3abb85816b3d0013634324)

---

## Key Discussion Points

### Marketing and Customer Communication
Devon sent an 11,000-recipient email blast showcasing portal features built over the past year, priced at $250 per box. Within five minutes, it received a 64 click rate and 307 total clicks. Devon excluded approximately 150 email addresses affiliated with a company involved in a trademark dispute. Ten boxes were sold during the meeting itself. Aksana recommended sending a separate, targeted blast to distributors highlighting features relevant to scaling their networks (invitations, service plan pricing). Devon agreed and will use the company export to pull the distributor list.

### Trello Card Description Save Issue
Devon reported that Trello card descriptions he wrote appeared unsaved -- the content was sitting in a draft/cached state and only persisted after manually hitting Save again. Aksana confirmed this is an intermittent issue seen across other clients as well, possibly related to the Trello-to-Jira sync tool. The root cause has not been reliably reproduced.

### Power Relay Requirements
Aksana clarified that the power relay work is split into two phases. Phase 1 focuses on enhancing the single-outlet relay: moving the flag to the model level, adding device detection (whether something is attached), and enabling restart capability. Phase 2 covers multi-outlet (four outlets) with additional functionality. Devon and Adam reviewed the Confluence requirements pages and had questions about a grid of dependencies -- Aksana explained those items are phase 2 dependencies that will be unblocked once phase 1 is complete. Devon will review the phase 1 requirements and add inline comments for anything unclear. Adam is on vacation this week.

### T-Mobile Work Launch
Laura confirmed that device status changes and signal strength display have been launched. The SignR ticket and Checkit bug (server time sync issue) are also completed. Additional T-Mobile tickets remain queued and will be picked up after the Cake 5 upgrade is through QA.

### Cake 5 Upgrade Status
All Cake 5 development is complete and deployed to the review environment for testing. Aaron is aggressively pushing through QA this week; both Aaron and Noah are out the following week (returning June 29). A soft code freeze is in effect -- no other work will be deployed to the review environment until Cake 5 testing is done, to avoid conflating issues from the major framework upgrade (CakePHP 4 to 5). Bug fixes can still go through a passing lane to production. Once Cake 5 ships, devs will pick up larger feature work (power relays and configurations).

### W-9 Tax Forms
Devon confirmed the W-9 tax form tickets remain low priority and will continue to be deferred in favor of higher-priority work.

### Device Billing and Deactivation Issue
Deactivated devices appearing on invoices was mostly resolved for the current billing cycle by Richard manually identifying and patching incorrect data. However, the root cause needs a permanent fix before the next billing cycle. The core problem is that device status and carrier SIM status can be out of sync, leading to false positives in either direction. Stone is currently analyzing the billing process to identify gaps and implement a durable solution.

### Sub-Company Invitation Streamlining
Devon described a multi-step pain point: after a sub-company accepts an invitation, the "allow payment method" flag must be manually checked, the sub-company must then go back to add a payment method, and in some cases devices need to be reassigned from a distributor's placeholder billing method. Aksana proposed moving the payment method flag to the invitation level, creating two invitation types -- one that allows payment methods and one that restricts them. When the sub-company accepts, the flag is automatically set. Devon agreed this is the right scope. Device reassignment automation was discussed but deemed an edge case not worth investing in now.

### Update Devices Bulk Import Issues
Devon described the bulk import process as cumbersome: leaving fields blank in the CSV wipes existing data (e.g., location name), the address format differs between the device export (single field) and the import template (separate columns for address, state, zip), and using a stale export risks overwriting changes made in the interim. Aksana proposed adding a "partial update" mode that only updates fields present in the CSV, alongside the existing full update. She will design the improvement.

### Flat Rate Pricing Transfer Issue
When devices with a flat rate (but no service plan) are transferred between customers using "maintain existing service plan," the flat rate is silently lost because the option only preserves service plans. Vince encountered this when transferring devices -- the portal accepted the transfer without warning, but the flat rate pricing was wiped. Richard confirmed this is expected behavior given the current naming but agreed the option should be broadened. The team agreed to rename/consolidate the option to "maintain existing pricing," which would preserve both service plans and flat rates during transfers.

---

## Action Items

- [ ] **Devon** -- Send a separate email blast to the distributor list highlighting distributor-relevant features (invitations, service plan pricing)
- [ ] **Devon** -- Review phase 1 power relay requirements in Confluence and add comments for any unclear items
- [ ] **Aksana** -- Implement payment method flag at the invitation level for sub-company invitations (two types: with and without payment method)
- [ ] **Aksana** -- Design and improve bulk device import to support partial updates (only update fields included in CSV)
- [ ] **Aksana** -- Consolidate "maintain existing service plan" logic to "maintain existing pricing" covering both service plans and flat rates during device transfers
- [ ] **Stone** -- Complete root cause analysis on deactivated devices appearing on invoices; implement permanent fix before next billing cycle
- [ ] **Aaron/Noah** -- Complete Cake 5 QA testing; address any bugs that come back before both return June 29

## Decisions Made

- Power relay work will proceed in two phases: phase 1 (single outlet enhancement) first, phase 2 (multi-outlet) after
- Soft code freeze in effect during Cake 5 QA -- no new features deployed to review environment; bug fixes can bypass via passing lane
- Sub-company invitation will support a payment method flag set at invitation time, creating two invitation types
- Device reassignment during sub-company onboarding is deferred as an edge case
- "Maintain existing service plan" option will be broadened to "maintain existing pricing" to include flat rates

## Items Deferred

- W-9 tax form tickets -- deprioritized; lower priority than all newly submitted tickets
- Automated device reassignment during sub-company invitation -- edge case, not worth investment at this time
- Distributor placeholder billing optimization (manual invoice adjustments for Great Lakes scenario) -- tabled as infrequent occurrence
