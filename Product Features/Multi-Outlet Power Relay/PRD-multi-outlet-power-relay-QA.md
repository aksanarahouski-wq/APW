# Multi-Outlet Power Relay - PRD Questions & Answers

**PRD:** [PRD-multi-outlet-power-relay.md](PRD-multi-outlet-power-relay.md)
**Status:** Awaiting Answers
**Created:** 2026-05-26

_This document captures questions that need answers before finalizing the PRD. The PRD draft has been written with these items flagged as open questions._

---

## Blocking Questions

### Q-1: Confirmation Dialog Before Power Actions

**What decision/data is needed:**
Should the portal display a confirmation dialog before executing a power action (restart, turn off, turn on) on an outlet? If yes, should the confirmation include the outlet label/name to prevent accidental actions on the wrong outlet?

**Why it matters:**
- Impacts FR-3 (Power Control Actions) — determines whether actions are one-click or require confirmation
- Power-cycling the wrong outlet could disrupt a live ATM transaction or game session
- Safety consideration raised in original discovery doc but not resolved in May 11 meeting

**Evidence:**
"Can company admins control all 4 outlets, or should there be per-outlet permissions? Are there any safety considerations (e.g., confirmation dialog before cutting power to an outlet)?" — from Multi_Outlet_Power_Relay_IR315.md, discovery questions
No direct discussion of confirmation dialogs in May 11 transcript.

**Answer:**
**Decision: Confirmation dialog on Restart only.** On/Off toggles do not require confirmation.

When a user clicks the Restart button on an outlet, a confirmation dialog appears with the outlet label so the user can verify they are restarting the correct outlet. Dialog text:

> **Are you sure you want to restart "{label}" (Outlet {n})?**
> This will power off the outlet for 10 seconds, then power it back on.
>
> [Cancel] [Yes, Restart]

If no custom label is set, the dialog shows the default name (e.g., "Outlet 2").

**Rationale:** Restart is the most impactful action (10-second power cycle could disrupt a live transaction). On/Off toggles are lower-risk since the user can immediately reverse them.

**Decided by:** Devon / Adam (APW) — 2026-05-29

---

### Q-2: "Restart All" Capability

**What decision/data is needed:**
Should there be a "Restart All Outlets" button in addition to individual per-outlet restart buttons? Or is individual control sufficient for MVP?

**Why it matters:**
- Impacts FR-3 (Power Control Actions) — determines button layout and API call sequencing
- A "restart all" action would need to define whether outlets restart simultaneously or sequentially
- Common use case: site-wide power issue where all equipment needs a restart

**Evidence:**
"What does the button layout look like for 4 independent outlets?" and "Is there a 'restart all' option in addition to individual controls?" — from Multi_Outlet_Power_Relay_IR315.md, discovery questions
Not discussed in May 11 meeting.

**Answer:**
**Decision: Yes — include a "Restart All" button.** All outlets restart simultaneously (parallel DIO commands, not sequential).

A "Restart All" button shall appear in the power management module alongside the individual per-outlet controls. Clicking it triggers a confirmation dialog:

> **Are you sure you want to restart all 4 outlets?**
> This will power off all outlets for 10 seconds, then power them back on.
>
> [Cancel] [Yes, Restart All]

All four outlets receive DIO LOW commands simultaneously, wait 10 seconds, then all receive DIO HIGH commands simultaneously. No sequential queuing.

**Decided by:** Devon / Adam (APW) — 2026-05-29

---

### Q-3: Scheduling Override Behavior

**What decision/data is needed:**
When a schedule is active for an outlet and a user manually toggles that outlet (e.g., turns it on outside scheduled hours), what should happen?
- Option A: Manual action overrides the schedule until the next scheduled event
- Option B: Manual action is temporary and the schedule resumes at the next trigger
- Option C: Manual action disables the schedule for that outlet (requires re-enabling)

**Why it matters:**
- Impacts FR-5 (Per-Outlet Scheduling) — defines how manual vs. scheduled actions interact
- Without clear rules, users may be confused when a manually-turned-on outlet gets turned off by a schedule
- Devon mentioned overrides briefly in the May 11 meeting but it wasn't resolved

**Evidence:**
"Devon mentioned overrides briefly but it wasn't fully explored" — from meeting extraction summary
Existing scheduling feature works at device level; outlet-level adds complexity to override logic.

**Answer:**
_[To be filled in by user/stakeholders]_

---

### Q-4: Relay Activation Toggle — Who Can Enable/Disable It?

**What decision/data is needed:**
Which user roles can toggle the "Are you utilizing a four-outlet power relay?" setting on a device? Is this restricted to company admins, or can any user with device access enable/disable it?

**Why it matters:**
- Impacts FR-2 (Relay Activation) — determines permission model for the toggle
- Enabling/disabling the relay changes the device page UI for all users viewing that device
- Accidental deactivation could hide outlet schedules and labels

**Evidence:**
"Can company admins control all 4 outlets, or should there be per-outlet permissions?" — from Multi_Outlet_Power_Relay_IR315.md
Not discussed in May 11 meeting.

**Answer:**
_[To be filled in by user/stakeholders]_

---

### Q-5: Outlet Label Character Limit and Validation

**What decision/data is needed:**
What is the maximum character length for outlet labels? Are there any character restrictions (e.g., alphanumeric only, no special characters)? Should labels be required or optional?

**Why it matters:**
- Impacts FR-4 (Outlet Labeling) — determines input validation rules and database field sizing
- Labels that are too long could break the visual layout of the four-outlet diagram
- Labels appear in scheduling UI, status displays, and potentially in notifications (future)

**Evidence:**
"Labeling — text fields so customers can name each outlet (e.g., 'ATM,' 'Jukebox,' 'Game 1,' 'Redemption Terminal')" — from meeting extraction
Examples suggest short labels (3-20 chars) but no explicit limit was discussed.

**Answer:**
_[To be filled in by user/stakeholders]_

---

### Q-6: DIO Port-to-Outlet Mapping Documentation

**What decision/data is needed:**
What is the exact mapping between DIO port numbers (1-4) on the i52 router and physical outlet positions (1-4) on the relay? Is this mapping fixed by the wiring harness, or could it vary by installation?

**Why it matters:**
- Impacts FR-3 (Power Control Actions) and Technical Requirements — the portal must send API commands to the correct DIO port for each outlet
- If the mapping varies by installation, the portal may need a configuration step where the user maps ports to outlets
- If fixed, the mapping can be hard-coded

**Evidence:**
"All four outlets controlled via API calls to same i52 IP, differentiated by DIO port number" — from meeting extraction
"Different wiring harness configurations may be needed depending on how many outlets a customer wants connected (1, 2, 3, or 4)" — from Multi_Outlet_Power_Relay_IR315.md
Adam confirmed API tested and working but exact port mapping not documented in meeting.

**Answer:**
_[To be filled in by user/stakeholders]_

---

### Q-7: What Happens to Outlet Data When Relay Is Deactivated?

**What decision/data is needed:**
If a user toggles the relay activation to "No" (deactivates), what happens to:
- Saved outlet labels?
- Active schedules?
- Historical outlet state data?

Options:
- Option A: Data is preserved (hidden but retained); re-enabling restores everything
- Option B: Data is cleared; re-enabling starts fresh
- Option C: User is warned and must confirm; data is preserved for X days then cleared

**Why it matters:**
- Impacts FR-2 (Relay Activation) — determines data lifecycle and user expectations
- Accidental deactivation should not permanently destroy configuration
- Reactivation UX depends on whether data persists

**Evidence:**
No direct evidence in provided context. This scenario was not discussed in either the March or May meetings.

**Answer:**
_[To be filled in by user/stakeholders]_

---

### Q-8: Existing i52 Devices — Single Relay Usage

**What decision/data is needed:**
Are there existing i52 devices in the portal that currently use the single "Restart Power Cycler" button? If so, how many? Should those customers receive any communication about the new multi-outlet relay option?

**Why it matters:**
- Informational, not blocking — the existing single button is preserved for i52 devices without the relay activated, so no functionality is removed
- Knowing current single-relay usage on i52 helps determine whether migration guidance or proactive outreach is needed when the multi-outlet feature launches
- Customers currently using single relay on i52 may be candidates for the four-outlet relay hardware upgrade

**Evidence:**
"Adam is 'not 100% positive' but 'pretty confident' none do. He'll check with a handful of customers (mentioned Millie)." — from meeting extraction
"Adam is 'very confident nobody is using' the traditional single relay on an i52." — from meeting extraction

**Answer:**
_[To be filled in by user/stakeholders]_

---

END OF DOCUMENT
