# WATM Backlog Planning

## Meeting Info
- **Date:** July 6, 2026
- **Duration:** ~60 minutes
- **Organizer:** Laura Perry
- **Attendees:** Aksana Rahouski, Noah Bratzel, Aaron Diefes, Stone Marballie, Richard Sacco
- **Recording:** [tl;dv](https://tldv.io/app/meetings/6a4c0229491ee600135943c8)

---

## Key Topics Discussed

### 1. CakePHP 5 Upgrade - Testing & Rollout Plan
Aaron is actively testing and found one datetime-related bug in test usage generation (already fixed). Noah completed CI cleanup work on the CAKE 5 branch. The team agreed to move CAKE 5 to beta by Thursday, ask the client to validate (especially billing-related features), then push to production early the following week. Key concerns: cron jobs and background processes are nearly impossible to test outside production; rollback is possible if critical issues arise (Noah will tag the release). Richard will send UDP packets to test check-in processing on the review environment.

### 2. Noah's Workload & Other Projects
Noah has no remaining APW/CAKE tasks and will focus on the CP project this week. He will remain available if CAKE-related bugs surface.

### 3. Stone's Smaller Tickets
Stone's smaller tickets are all branched off CAKE 5 and sitting in review. They will not be merged until CAKE 5 merges to beta to avoid introducing other changes during verification.

### 4. Upcoming Feature Work
- **Multi-outlet power relay enhancements (Phase 2):** Stone will lead development, Aaron will lead with requirements already written.
- **Configurations rework:** Aksana and Noah will brainstorm the roadmap after the CAKE release.

### 5. Verizon Device Status Syncing Issues (Spike)
Stone walked through findings from his spike investigation:

- **Pending-as-success bug:** When Verizon's callback shows "pending activation," the current code incorrectly treats it as successful and marks the device as active. The device may never actually activate on Verizon's side. Fix: leave pending devices in pending state and let the reconciliation cleanup job handle updates.
- **Cleanup job gap:** The cleanup job runs every 15 minutes checking devices with the `is_vzw_pending` flag against Verizon's API, but does not update the `provider_status` field, creating gaps in local device state tracking. Patching this to also update provider status will fix downstream billing and status issues.
- **Scale:** 96 activation failures over a 12-hour period on a single day, indicating Verizon system issues.
- **Verizon-only issue:** Only Verizon uses callback-based activation; AT&T and T-Mobile return success or failure immediately.

### 6. Billing Discrepancies
Two billing issues caused by portal-carrier status desync:
- **First issue (earlier):** Inactive devices in portal were active on carrier = free service (no billing).
- **Recent issue:** Devices marked active in portal are inactive on carrier = incorrect charges.
- The `charge_per_current_cycle` job marks deactivated devices so they're not billed, but relies on accurate status data.
- Billing cycle ends this week, making the Verizon status syncing fix urgent.

### 7. Device Export Bug (Ticket #92)
Device export failing for one specific company. Theory: email delivery for that company fails, causing the export job to fail. Testing requires pulling production database locally. Lower priority than Verizon status syncing but client keeps requesting it.

### 8. Jira Ticket Hygiene
- "Support 2026" Epic should be assigned to all non-feature-grouped tickets to avoid orphans.
- Ticket #92 added to sprint and marked critical.
- Service plans management bug (already deployed from production-head branch) marked Done.

---

## Decisions Made

1. CAKE 5 to beta by Thursday, then production early next week.
2. Richard to send UDP packets to test check-in processing on review environment before beta rollout.
3. Aaron to identify specific modules/features for client to test in beta (billing is top priority).
4. Verizon status syncing fix is Stone's top priority -- must be in production by Friday (before billing cycle ends).
5. Leave pending devices in pending state rather than treating pending activation as successful.
6. Email admins immediately if any devices remain pending after cleanup job runs (count > 0 triggers email with list; no email on success).
7. Device export (ticket #92) is second priority after Verizon fix.
8. Multi-outlet power relay Phase 2: Aaron leads requirements, Stone leads development.
9. Configurations rework roadmap to be developed after CAKE release (Aksana + Noah).
10. Consolidate the two spike tickets into a single actionable ticket.

---

## Action Items

| Who | Action | Deadline |
|-----|--------|----------|
| **Aaron** | Complete CAKE 5 testing and roll out to beta | Thursday (July 10) |
| **Aaron** | Identify specific modules and features for client testing in beta | Before client meeting (Tue July 7) |
| **Richard** | Send UDP packets to test check-in processing with CAKE upgrade on review env | When Aaron requests |
| **Stone** | Create consolidated ticket for Verizon device status syncing issues | ASAP |
| **Stone** | Implement Verizon status syncing fix and deploy to production | By Friday (July 11) |
| **Stone** | Work on device export bug (ticket #92) after Verizon fix | After Verizon fix |
| **Aksana** | Create ticket consolidating device export and status syncing work (client-visible) | After meeting |
| **Aksana** | Communicate CAKE 5 beta timeline and testing expectations to client | Tue client meeting (July 7) |
| **Noah** | Focus on CP project; remain available for CAKE-related bugs | This week |

---

## Notable Details

- 96 activation failures over 12 hours on one day -- unprecedented, suggests Verizon internal issues.
- Cleanup job runs every 15 minutes, checks `is_vzw_pending` flag against Verizon API.
- `charge_per_current_cycle` job runs at start of new billing cycle to mark deactivated devices.
- Service plans management bug fix was branched off production head and deployed directly (bypassed normal pipeline).
- Beta vs. review environment differences are not fully documented -- creates testing blind spots.
- Production is the real test for cron jobs and background processes (don't run on lower envs).
- Stone estimated half a day for the Verizon status syncing fix implementation.
- Stone is exclusively allocated to APW with no other projects.
