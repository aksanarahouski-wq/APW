# Configuration Engine V2 — Meeting 1 Summary

**Date:** March 27, 2026
**Attendees:** Aksana Rahouski (Orases), Devon D'Andrea (APW), Adam Curcie (APW)
**Duration:** ~54 minutes
**Topic:** Initial client feedback on Configuration Engine V2 prototype and requirements

---

## Overall Feedback

The client response to the V2 prototype and requirements was **very positive**:

- Devon: *"The interface, the way you interact with it — I love all of it"* and *"way far closer to the finish line than I would have ever expected"*
- Both Devon and Adam confirmed the UI flow, button placement, and general user experience feel right
- Adam noted they'd started reviewing but estimate about 4 more days to get through all the documentation
- Devon: *"I would have never expected us to be as far as we are given how gigantic of a lift this is"*

---

## Key Discussion Topics

### 1. Multiple Override Sets Per Company (Change Request)

**Current design:** One override set per 3-Way Rule per company.

**Client feedback:** A company needs to be able to have **multiple override sets** on the same 3-Way Rule, as long as the parameters don't conflict.

**Real-world examples given:**
- **Cord Financial** has a firewall override set. But Cord might also need a separate override set for something unrelated (e.g., DNS, DHCP). These should coexist as modular building blocks.
- **Altech** has a unique combination of firewall rules, DHCP scope, AND DNS — packaged as one override set that only applies to them and their subs.
- **Baltech** has a scheduler override (devices power cycle nightly at 3am). Adam wants to be able to apply Baltech's scheduler override to Cord without having to rebuild Cord's entire override set.

**Proposed behavior:**
- Override sets should work **modularly** — stack multiple on the same company for the same 3-Way Rule
- System should **reject or warn** if two override sets assigned to the same company contain conflicting (overlapping) parameters
- Think of override sets as reusable "building blocks" that can be mixed and matched

### 2. Override Sets Decoupled from 3-Way Rules ("Any/All" Concept)

**Current design:** Every override set is scoped to a specific 3-Way Rule (specific model + carrier + service plan).

**Client feedback:** Many company overrides (especially firewall rules) are **agnostic** to model, carrier, and service plan. The parameters apply universally to a company regardless of device type.

**Adam's point:** *"None of these configurations, none of the differences and none of the customizations are related to the service plan, the model, or the carrier really"* — referring to the vast majority of their current custom config files.

**What they want:**
- When creating an override set, the model/carrier/service plan selectors should support an **"Any" or "All" option** in addition to specific values
- This allows creating override sets like:
  - Firewall override → applies to **all models, all carriers, all service plans** for a company
  - Scheduler override → applies to **all models, all carriers, ATM plan only**
  - Power cycling override → applies to **I-22 only, all carriers, all service plans**
- Some overrides will still be scoped to a specific 3-Way Rule; others won't need that specificity

**The complexity acknowledged:** This brings back elements of the V1 two-way rule concept. The team recognized this during the meeting — Aksana: *"We're back to two-way rules and three-way rules and who wins"*. The resolution logic needs to account for varying levels of specificity when multiple overrides could apply.

### 3. Config Push Verification and Delivery Tracking (Future Phase)

Devon raised the critical concern that pushed configurations may not actually reach devices:
- API access to a device could be disabled
- Device could be offline when the push happens
- No current mechanism to verify the device received the new config

**Hostname versioning idea (Devon/Adam):** When a config change is made, iterate the hostname with the date (e.g., `VZW22_03272026`). When a device checks in, if it reports an old hostname, the portal knows the config push didn't land.

**Agreed:** This is the **next phase** after the config builder is finalized — how configs are applied, verified, and reported on.

### 4. Versioning and Rollback

Aksana raised the need for configuration versioning:
- When a config is updated, the previous version should be preserved
- Admins should be able to review, approve, and then publish a new version
- If something goes wrong, roll back to a previous version
- Agreed this is important but not part of the immediate V2 builder scope

### 5. Legacy System Coexistence

Client confirmed the legacy/.DAT system must remain fully functional:
- Both systems coexist during transition
- Per-device switching between Legacy and V2
- Adam was emphatic about not removing legacy: *"Don't get rid of it. Just disable it."*
- Legacy removal would only happen after extended confidence period (years)
- Everyone agreed this is critical for safety given how sensitive device configurations are

---

## Action Items

| # | Action | Owner | Due |
|---|--------|-------|-----|
| 1 | Send 3 sample custom config files (Altech, Kervin Martin, Baltech) to Aksana for analysis | Adam | ASAP |
| 2 | Analyze the sample config files to understand the actual scope of customer customizations and where they do/don't depend on model/carrier/service plan | Aksana | Before next meeting |
| 3 | Prepare real-world scenarios/examples of override rules that need different levels of specificity (model-only, carrier-only, service-plan-only, agnostic, full 3-way) | Devon & Adam | Before next meeting |
| 4 | Rethink the override set design to address: (a) multiple override sets per company, (b) "Any/All" scoping instead of strict 3-Way Rule binding, (c) conflict detection for overlapping parameters | Aksana | Before next meeting |
| 5 | Book follow-up working session for Friday April 3 | Aksana | This week |
| 6 | Sponsor call April 7 — align on priorities considering this week's feedback | All | April 7 |
| 7 | Continue reviewing the prototype and requirements documentation (Adam estimated 4 more days) | Adam & Devon | Ongoing |

---

## Decisions Made

1. **UI/UX direction confirmed** — The prototype's interface, layout, and interaction patterns are approved. Changes will be to mechanics/logic, not the overall UX approach.
2. **Legacy system preservation** — The legacy configuration system will be disabled but never removed from the codebase.
3. **Phased approach confirmed** — Phase 1: config builder (current). Phase 2: config delivery/push verification. Phase 3: versioning/rollback. Each phase discussed separately.

---

## Open Design Questions (Arising from This Meeting)

1. **Multiple override sets per company:** How do we handle conflict detection when assigning multiple override sets to the same company? At assignment time? Real-time validation? What's the UX for resolving conflicts?

2. **"Any/All" override scoping:** If an override set says "all models" and another says "I-22 specifically" for the same company and same parameter — which wins? Do we need a specificity-based resolution (more specific wins)?

3. **Resolution complexity:** Adding flexible override scoping (1-way, 2-way, 3-way combinations) significantly increases the resolution algorithm complexity. Need to determine: is the simplicity trade-off worth it, or can we find a middle ground?

4. **Override set modularity vs. usability:** Adam wants modular building blocks (firewall set + scheduler set + DNS set). But more granularity = more objects to manage. Where's the balance between flexibility and usability?

---

## Next Meeting

**Target:** Friday, April 3, 2026 — Working session to:
- Review analyzed sample config files
- Walk through real-world override scenarios
- Align on the override set design changes (multiple per company + flexible scoping)
