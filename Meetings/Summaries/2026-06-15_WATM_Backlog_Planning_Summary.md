# WATM Backlog Planning — June 15, 2026

**Date:** June 15, 2026, 3:30 PM ET
**Duration:** ~51 minutes
**Organizer:** Laura Perry
**Recording:** [tldv](https://tldv.io/app/meetings/6a3052b04bc8250013fe58ca)

## Attendees

- Aksana Rahouski (Orases)
- Noah Bratzel (Orases)
- Aaron Diefes (Orases)
- Richard Sacco (Orases)
- Stone Marballie (Orases) — joined late
- Laura Perry (Orases) — joined late

## Meeting Overview

Internal backlog planning session covering two main topics: (1) the CakePHP 5 release strategy, testing plan, and timeline for pushing to the review environment, and (2) a billing and device status sync issue where deactivated devices were incorrectly appearing on invoices due to unreliable SIM status data from carriers. The team agreed on a soft development freeze during CakePHP 5 testing next week and decided to create a spike ticket to investigate and document the billing/device status sync process before implementing a fix.

## Topics Discussed

### CakePHP 5 Testing Plan and Timeline

- Aaron outlined the existing QA test plan, which covers every main module in the app with P0/P1/P2 priority levels. A shorter, higher-level version also exists for experienced testers who do not need step-by-step instructions.
- Both documents are stored in GitHub under the CakePHP 4 folder.
- Aaron estimated a full end-to-end test of the site would take 1-2 full days (8+ hours minimum). He noted he has never done a complete E2E test on the APW site before.
- The team agreed to target pushing CakePHP 5 to the review environment early Monday of the following week, since the current week is short (Aaron out starting Tuesday, multiple people out Thursday for the holiday).
- Testing goal: complete CakePHP 5 testing by end of the following week.

### CakePHP 5 Release Strategy and Development Freeze

- Once CakePHP 5 is pushed to review, a **soft development freeze** takes effect — no new work can be merged into review or shipped to production until CakePHP 5 testing is complete and it goes to production.
- **Hotfix option**: If a critical client bug arises during the freeze, the team can revert the review environment to the normal branch, apply a hotfix directly, or test fixes on beta — multiple options exist to handle emergencies without disrupting the CakePHP 5 testing.
- Developers can continue working on new features in **feature branches** during the freeze, but nothing goes to QA or review until CakePHP 5 is released to production. Noah emphasized avoiding stacking features on top of the CakePHP 5 branch to prevent compounding test scope and ambiguous bug sources.
- Aksana noted that larger items like multi-outlet power relay have enough planning and TRD work to keep developers busy during the freeze without needing to push code.

### Stone's In-Flight Bug Tickets

- Stone confirmed two bug tickets (not part of the CakePHP 5 bundle) are done. He is finishing ETank merges and created a pull request, but the build failed — he needs to investigate the failure.
- Plan: get these two tickets through review and to the client before pushing CakePHP 5 to review, so they are not blocked by the freeze.

### Billing and Device Status Issue — Deactivated Devices on Invoices

- **The problem**: Deactivated devices were incorrectly appearing on invoices for the most recent billing cycle (May 10 - June 11). The client discovered devices on their invoices that should not have been billed because they were deactivated.
- **Root cause**: The billing process checks the device SIM status table to determine whether a device is active. For Verizon devices, SIM statuses in that table were not reliably reflecting the actual carrier-side deactivation status. A prior code change attempted to use SIM statuses to catch billing discrepancies, but the underlying SIM status data was inaccurate.
- **Richard's manual fix**: He identified all devices deactivated before the billing cycle start date (May 10), filtered to those still flagged as `charge_for_current_cycle`, ran a command to check actual SIM statuses via carrier APIs, compiled a report, got Devon's approval, then set `charge_for_current_cycle` to zero in the `company_device_usages` table so the client could regenerate invoices. He deliberately avoided modifying the devices table to keep the hotfix minimal.
- **Scale**: Approximately 2,000 devices need to be marked as not charged, indicating this is a widespread data quality issue, not an edge case.

### Device Status Sync — Solution Discussion

- **Richard's view**: The device SIM status table was introduced as a logging mechanism during T-Mobile/test-ready work — it records the last API response for a SIM but was never designed as a source of truth. He advocated querying carrier APIs directly at billing time for deactivated devices and reporting anomalies (e.g., a device marked deactivated in the portal but still showing active on the carrier) to the client for review.
- **Noah's view**: The team should not accept unreliable internal statuses. Device statuses in the portal are supposed to reflect reality. He advocated for a periodic sync mechanism (e.g., a nightly cron job) that queries carrier APIs and updates portal statuses to ensure they stay accurate. If a full periodic sync is too heavy, at minimum a verification step should run as part of the billing process before anything gets billed.
- **Stone's view**: The SIM status concept is sound but needs to be tightened up. He suggested adding validation checks during SIM status transitions — for example, before reverting a pending deactivation (due to no carrier response), first check the carrier API to confirm the actual state before rolling back. Timing edge cases (carrier accepts but response arrives late, causing a false revert) are a known risk.
- **Carrier API constraints**: Verizon supports batch queries (up to 1,000 devices at a time). AT&T and T-Mobile only support one-by-one queries, making bulk status checks expensive in API calls.
- **Stone's existing command**: He previously built a command to identify gaps between portal and carrier statuses, but it relied on the same device SIM status table — so it is also affected by the unreliable data.

### Billing Process Complexity

- The billing logic is nuanced: suspended devices still get charged, only deactivated devices (or those deactivated for a certain period) are excluded. Even deactivated devices with usage may still be charged unless the client explicitly requests otherwise via the `charge_for_current_cycle` flag.
- Aksana noted the team needs a clear diagram documenting which devices should and should not make the billing cycle cut, since this logic comes up repeatedly every few months and is not well-documented.
- The team agreed this is a two-part problem: (1) fix the billing process to not rely on unreliable SIM status data for deactivated device exclusion, and (2) separately improve the SIM status sync to make the data reliable long-term.

### Verizon Data Grabbing Bug (WATM-2174)

- Richard reported a minor bug: the Verizon data grab works, but a CakePHP logging library throws errors when the request does not originate from a standard URL request (e.g., from a command/cron). Ticket WATM-2174 created and added to the sprint.
- Assessed as low priority — can be picked up when someone has capacity.

### Ticket Quality Improvement

- Aksana praised the significant improvement in Richard's ticket quality, noting they now include detailed context, root cause analysis, and clear descriptions — a major change from six months ago. Richard credited using Claude to process Devon's emails and generate structured tickets.

## Action Items

- **Stone**: Complete ETank merges and investigate/fix the failed build on his pull request. Target: get both bug tickets to review before the CakePHP 5 push.
- **Noah**: Push CakePHP 5 to the review environment early Monday of the following week.
- **All devs (Aaron, Noah, Stone, Aksana)**: Complete CakePHP 5 end-to-end testing by end of the following week, dividing the test plan among the team.
- **Aksana**: Create a spike ticket for investigating and documenting the billing and device status sync process.
- **Stone**: Pick up the spike ticket — investigate the billing process, document how device statuses flow between portal and carriers, identify gaps, and propose where to patch or change logic.
- **Team**: Communicate findings to the client (Devon/Adam) before implementing billing changes, since billing accuracy directly affects their operations.
- **Richard**: WATM-2174 (Verizon data grab logging bug) available in the sprint for pickup when capacity allows.

## Key Decisions

- **CakePHP 5 push to review** scheduled for early Monday of the following week (June 22), with a soft development freeze taking effect at that point.
- **CakePHP 5 testing target**: complete by end of the following week (June 27). Bug tickets in flight will be pushed ahead of the freeze.
- **Feature development during freeze**: allowed in feature branches only; no QA or review submissions until CakePHP 5 is in production.
- **Hotfixes during freeze**: approved as needed — team can revert review, apply fixes on beta, or bypass the CakePHP 5 branch for critical client issues.
- **Billing/device status issue**: will not be fixed immediately. A spike ticket will be created first to document the process, identify root causes, and propose a solution before any code changes. The team has until the next billing cycle (~July 11) to implement a fix.
- **Two-track approach** agreed for the billing/status problem: (1) fix the deactivated device billing logic to not depend on unreliable SIM status data, and (2) separately improve SIM status reliability through better sync mechanisms and transition validation.
