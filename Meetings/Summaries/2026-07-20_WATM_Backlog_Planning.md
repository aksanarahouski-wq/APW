# WATM Backlog Planning — 2026-07-20

**Date:** July 20, 2026, 3:30 PM EDT
**Duration:** ~32 minutes
**Organizer:** Laura Perry
**Attendees:** Aksana Rahouski, Laura Perry, Noah Bratzel, Richard Sacco, Aaron Diefes, Stone Marballie

---

## Summary

Internal team sync to align on current priorities, assignments, and upcoming work. Key topics: cellular backup feature (new, urgent), branding/rebrand rollout strategy, power relay spike, and bug fixes.

---

## Current Priorities (in order)

1. **Bug fixes** — Immediate priority for the team
2. **Branding / Rebrand** — High client priority; pre-cutover work can begin soon
3. **Cellular Backup Service Plans** — Top client priority once requirements are defined (targeting early September for trade shows)
4. **Power Relay Phase 1** — Spike in progress

---

## Assignments

| Person | Current Work | Next Up |
|--------|-------------|---------|
| **Aksana** | Draft cellular backup feature requirements (before tomorrow's meeting); ping stakeholders on tickets 1212 & 2205 for alignment | Report back to team once requirements are packaged |
| **Aaron** | Testing — UAT ticket 2181 by end of week; owns scope/deliverables for Power Relay Phase 1 | Handoff to Stone on power relay; may need separate meeting for scope discussion |
| **Richard** | Fix ACH amount discrepancy bug; review & clean up branding tickets, do another pass on pre-cutover vs cutover labeling | Move ready branding tickets into sprint; plan rollout strategy (testing, domain setup, rollback) |
| **Noah** | Finish chard autosync bug → move to testing; pick up bug WATM-2244 | Available for branding or cellular backup once bugs are cleared; 50% capacity (CP project) |
| **Stone** | Power Relay spike (WATM-2198) — write TRD | After spike: create dev tickets to complete the feature |
| **Laura** | Close ticket 2163 (Jira + Trello) with comment linking to 2196; assign 2196 to Aaron as Trello bridge card | Sprint board maintenance |

---

## Key Decisions & Discussion Points

### Cellular Backup Feature
- Client wants it within ~4 weeks (not a hard deadline) — tied to trade show schedule, real target is early September (6-7 weeks)
- MVP will start with **Verizon only**, even though client wants all carriers
- Medium-to-large feature, carries risk
- Aksana drafting requirements doc; will share with client for validation before dev handoff
- **This pushes Configurations V2 work again**

### Branding / Rebrand
- Most tickets are small (<1 hour each) — Richard broke them down granularly for easier testing
- **Highest risk area:** Server configuration, DNS, domain redirects, SSL
- **WooCommerce dependency:** Callback URL from checkyourbox.com → allpointcommand.com must keep working post-cutover; requires coordination with Javier
- **Noah's suggestion (agreed):** Three-phase approach:
  1. **Redesign work** — code changes in a separate feature branch, rolled out all at once
  2. **Domain infrastructure** — set up new domain (idmcontrol.com) to serve alongside existing domains without redirects ("stealth launch") to enable testing
  3. **Redirect cutover** — flip redirects from old → new domains only after everything is verified
- Richard to do a final pass on tickets and move pre-cutover items into the sprint

### Power Relay
- WATM-2163 closed — was just contextual info (curl commands), not a deliverable
- WATM-2196 is the Trello bridge card (client-facing), assigned to Aaron
- WATM-2198 is Stone's spike — will produce TRD and dev tickets
- Phase 2 currently blocked but expected to unblock soon; not building immediately

### Backlog Items
- Tickets 1212 and 2205 are in "Ready for Development" in Trello but still need stakeholder approval — Aksana to ping again
- Support tickets available in queue for when capacity opens up

---

## Next Steps

1. **Aksana** — Complete cellular backup requirements draft before tomorrow's client meeting
2. **Richard** — Fix ACH bug → do final pass on branding tickets → move ready items to sprint
3. **Noah** — Finish autosync bug → pick up WATM-2244
4. **Stone** — Continue power relay spike → deliver TRD
5. **Aaron** — Complete UAT testing on 2181 by end of week
6. **Laura** — Close 2163, update 2196 assignment
7. **Team** — Regroup after current work completes to assess whether branding or cellular backup is next
8. **Tomorrow's client meeting** — Validate priority order with stakeholders
