# Multi-Outlet Power Relay — UI Mockup Ideas

**Date:** 2026-05-29
**Author:** Aksana Rahouski
**Status:** Draft / Internal Discussion
**Related PRD:** [PRD-multi-outlet-power-relay.md](PRD-multi-outlet-power-relay.md)

This document captures UI layout ideas for three areas of the multi-outlet power relay feature. Each section shows what the current UI looks like today and proposes how it should change.

---

## Part 1: Four-Outlet Relay Module on the Device Page

### What It Looks Like Today

The bottom of the device detail page has a simple button bar:

```
┌─────────────────────────────────────────────────────────────────────┐
│  [ Restart Device ]   [ Restart Power Cycler ]   [ Received Support Call ]  │
└─────────────────────────────────────────────────────────────────────┘
```

- "Restart Power Cycler" is a single button — one click, one relay, done.
- There is also a "Power Relay Installed?" Yes/No field in the General Info section of the device.

### Proposed: i52 Device Without Relay Activated

No change. The existing button bar stays as-is, plus the relay activation toggle appears in the lower device page section (near the existing backup internet toggle area):

```
┌─────────────────────────────────────────────────────────────────────┐
│  Power Relay                                                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Are you utilizing a four-outlet power relay?    [ No  v ]           │
│                                                                      │
│  (i) If you have installed a four-outlet power relay on this         │
│      device, select "Yes" to enable per-outlet power controls.       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  [ Restart Device ]   [ Restart Power Cycler ]   [ Received Support Call ]  │
└─────────────────────────────────────────────────────────────────────┘
```

The single "Restart Power Cycler" button remains functional for i52 devices that don't have the four-outlet relay.

### Proposed: i52 Device With Relay Activated

When the toggle is set to "Yes", the four-outlet module appears inline (no page reload) and the single "Restart Power Cycler" button is hidden from the bottom bar.

**Option A: Grid Layout (Recommended)**

Four outlet cards in a 2x2 or 4-across grid. Each card is self-contained with label, status, and controls.

```
┌─────────────────────────────────────────────────────────────────────┐
│  Power Relay                                                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Are you utilizing a four-outlet power relay?    [ Yes v ]           │
│                                                                      │
│  ┌─────────────────────┐  ┌─────────────────────┐                   │
│  │  Outlet 1            │  │  Outlet 2            │                   │
│  │  "ATM"          [edit]│  │  "Jukebox"     [edit]│                   │
│  │                      │  │                      │                   │
│  │  Status: ● ON        │  │  Status: ● ON        │                   │
│  │                      │  │                      │                   │
│  │  [ ON/OFF toggle ]   │  │  [ ON/OFF toggle ]   │                   │
│  │  [ Restart ]         │  │  [ Restart ]         │                   │
│  │                      │  │                      │                   │
│  │  Schedule: None      │  │  Schedule: 8AM-10PM  │                   │
│  └─────────────────────┘  └─────────────────────┘                   │
│                                                                      │
│  ┌─────────────────────┐  ┌─────────────────────┐                   │
│  │  Outlet 3            │  │  Outlet 4            │                   │
│  │  "Game 1"      [edit]│  │  "Outlet 4"    [edit]│                   │
│  │                      │  │                      │                   │
│  │  Status: ○ OFF       │  │  Status: ? Unknown   │                   │
│  │                      │  │                      │                   │
│  │  [ ON/OFF toggle ]   │  │  [ ON/OFF toggle ]   │                   │
│  │  [ Restart ]         │  │  [ Restart ]         │                   │
│  │                      │  │                      │                   │
│  │  Schedule: 8AM-10PM  │  │  Schedule: None      │                   │
│  └─────────────────────┘  └─────────────────────┘                   │
│                                                                      │
│  Last check-in with DIO data: 5/29/26 10:15 AM                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  [ Restart Device ]   [ Received Support Call ]                      │
└─────────────────────────────────────────────────────────────────────┘
```

Note: "Restart Power Cycler" is gone from the bottom bar — it's now replaced by per-outlet restart buttons in the module above.

**Key elements per outlet card:**
- **Outlet number** — always visible (Outlet 1, 2, 3, 4)
- **Custom label** — editable inline, defaults to "Outlet N" if blank
- **Status indicator** — green dot ON, gray dot OFF, question mark Unknown
- **ON/OFF toggle** — persistent power state control
- **Restart button** — 10-second power cycle; shows "Restarting..." state with spinner and disables during cycle
- **Schedule summary** — one-line preview of active schedule if any, "None" if not

**Option B: Table/Row Layout**

A more compact table-style layout, similar to the existing Power Cycler Capable Devices table pattern the portal already uses.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  Power Relay                                                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  Are you utilizing a four-outlet power relay?    [ Yes v ]                       │
│                                                                                  │
│  ┌────────┬──────────────┬──────────┬───────────┬──────────────┬───────────┐    │
│  │ Outlet │ Label        │ Status   │ Schedule  │ Power        │ Actions   │    │
│  ├────────┼──────────────┼──────────┼───────────┼──────────────┼───────────┤    │
│  │  1     │ ATM          │ ● ON     │ None      │ [ON] / OFF   │ [Restart] │    │
│  │  2     │ Jukebox      │ ● ON     │ 8AM-10PM  │ [ON] / OFF   │ [Restart] │    │
│  │  3     │ Game 1       │ ○ OFF    │ 8AM-10PM  │  ON / [OFF]  │ [Restart] │    │
│  │  4     │ —            │ ? Unknown│ None      │  ON /  OFF   │ [Restart] │    │
│  └────────┴──────────────┴──────────┴───────────┴──────────────┴───────────┘    │
│                                                                                  │
│  Last check-in with DIO data: 5/29/26 10:15 AM                                  │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

**Comparison:**
| Aspect | Option A (Grid) | Option B (Table) |
|--------|----------------|-----------------|
| Visual clarity | Each outlet is a distinct visual unit | More compact, scannable |
| Matches existing patterns | New pattern for the portal | Closer to the Power Cycler Capable Devices table |
| Label editing | Click edit icon on card | Click label cell inline |
| Schedule visibility | Summary on card | Column in table |
| Mobile/responsive | Cards stack vertically | Table may need horizontal scroll |
| Scalability | Works well for 4 | Would work for any count |

### Restarting State

When a restart is in progress on an outlet (10-second power cycle), the card/row should show:

```
┌─────────────────────┐
│  Outlet 2            │
│  "Jukebox"     [edit]│
│                      │
│  Status: ↻ Restarting│
│                      │
│  [ ON/OFF disabled ] │
│  [ Restart disabled ]│  <-- grayed out with spinner
│                      │
│  Schedule: 8AM-10PM  │
└─────────────────────┘
```

- The restart button shows a spinner and is disabled for that outlet only
- Other outlets remain fully interactive
- After 10 seconds, status returns to ON and controls re-enable

---

## Part 1B: Single-Outlet Power Cycler Enhancement

### The Problem Today

Single-outlet devices (I-22, I-4500, Systex) have a "Restart Power Cycler" button at the bottom of the device page — and that's it. There is **zero visibility** on the device page for:
- Whether the power cycler is currently ON or OFF
- Whether the device is on a power schedule, and which one
- Whether an override is active

This information exists in the system (it's visible in the Power Cycler Capable Devices table on the company power management page), but you can't see it when you're looking at the actual device.

### Proposed: Single-Outlet Power Cycler Module

Replace the bottom-bar "Restart Power Cycler" button with a dedicated power cycler module on the device page that shows state, schedule, and controls in one place:

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│  Power Cycler                                                                     │
├──────────────────────────────────────────────────────────────────────────────────┤
│                                                                                   │
│  ┌──────────────────┬──────────────────────┬─────────────────┬────────────────┐  │
│  │  Power State      │  Schedule            │  Override       │  Actions       │  │
│  │                   │                      │                 │                │  │
│  │  ● ON             │  Business Hours      │  No Override    │  [ON/OFF]      │  │
│  │                   │  Active              │                 │  [Restart]     │  │
│  │  Last check-in:   │  8AM-10PM Mon-Sat ET │                 │                │  │
│  │  5/29/26 10:15 AM │                      │                 │                │  │
│  └──────────────────┴──────────────────────┴─────────────────┴────────────────┘  │
│                                                                                   │
└──────────────────────────────────────────────────────────────────────────────────┘
```

This mirrors the information already available in the power management table but puts it right where the user needs it — on the device they're looking at.

See prototype: `prototypes/single-outlet-device.html` — includes a scenario switcher to demo ON/OFF/Unknown states, with and without schedules, and override states.

---

## Part 2: Power Scheduling — Multi-Outlet Enhancement

### What It Looks Like Today

**Company Power Schedules page** — card-based layout:
- Each schedule is a card showing: Name, Active/Inactive badge, Timezone, Days active, # of Devices, Edit/Delete icons
- "+ New Schedule" button to create
- "Browse Power Management" link from company view

**Power Cycler Capable Devices table** (inside a schedule or company):
- Columns: Serial Number, Location, TID, Model, Status, Power Cycler State (ON/Unknown), Last Action, Power Schedule, Override Status, Actions
- Filters: serial number, location, TID, schedule name
- Bulk Actions, Select All, Refresh Table
- This view shows devices as whole units — one row per device

**Key observation:** Today, the scheduling system operates at the **device level**. A schedule is assigned to a device, and the device has one power cycler state. With multi-outlet, we need to extend this to handle the concept of individual outlets within a device.

### The Challenge

The existing model is: **Schedule -> Device -> single power state**

The new model needs: **Schedule -> Device -> Outlet 1, 2, 3, 4 (each with independent state)**

We need to handle:
1. Devices with a single outlet (i22, i4500, Systex) — behave exactly as today
2. Devices with four outlets (i52 with relay) — each outlet can have its own schedule
3. The power management list/table needs to surface outlet-level information for multi-outlet devices

### Proposed Approaches

**Approach A: Outlet-Aware Schedule Assignment (Per-Outlet Schedules)**

Each outlet on a multi-outlet device can be assigned to a different schedule independently.

*Schedule creation stays the same* — you create a schedule with a name, timezone, days, on/off times. No change to the schedule object itself.

*Assignment changes* — when assigning a device with a relay to a schedule, you pick which outlets get that schedule:

```
Add Device to Schedule: "Business Hours"
┌─────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  Device: W52290 (I-52)    Location: Main St ATM                      │
│                                                                      │
│  This device has a four-outlet power relay.                          │
│  Select which outlets to include in this schedule:                   │
│                                                                      │
│  [x] Outlet 1 — "ATM"                                               │
│  [x] Outlet 2 — "Jukebox"                                           │
│  [ ] Outlet 3 — "Game 1"        (already in schedule: "Night Off")  │
│  [ ] Outlet 4 — (unlabeled)                                         │
│                                                                      │
│  [ Cancel ]                                      [ Add to Schedule ] │
└─────────────────────────────────────────────────────────────────────┘
```

*Power Cycler Capable Devices table* — multi-outlet devices expand to show outlet rows:

```
┌────────────┬──────────┬───────┬──────────┬──────────────┬──────────────┬───────────┐
│ Serial #   │ Location │ Model │ Status   │ Power State  │ Schedule     │ Actions   │
├────────────┼──────────┼───────┼──────────┼──────────────┼──────────────┼───────────┤
│ 99999      │          │ I-22  │ Deact.   │ ● ON         │ Test1        │ Actions v │
├────────────┼──────────┼───────┼──────────┼──────────────┼──────────────┼───────────┤
│ W52290     │ Main St  │ I-52  │ Active   │              │              │           │
│  ├ Outlet 1│          │       │          │ ● ON  "ATM"  │ Biz Hours    │ Actions v │
│  ├ Outlet 2│          │       │          │ ● ON  "Juke" │ Biz Hours    │ Actions v │
│  ├ Outlet 3│          │       │          │ ○ OFF "Game1"│ Night Off    │ Actions v │
│  └ Outlet 4│          │       │          │ ? Unknown    │ No schedule  │ Actions v │
├────────────┼──────────┼───────┼──────────┼──────────────┼──────────────┼───────────┤
│ AD0010     │          │ I-22  │ Active   │ ? Unknown    │ No schedule  │ Actions v │
└────────────┴──────────┴───────┴──────────┴──────────────┴──────────────┴───────────┘
```

- Single-outlet devices (I-22) appear as one row — no change from today
- Multi-outlet devices (I-52 with relay) show a parent row + expandable child rows per outlet
- Each outlet row has its own power state, schedule, and actions
- The parent row for multi-outlet devices doesn't have a single power state — the state is per-outlet

**Approach B: Device-Level Schedule with Outlet Override**

The schedule is still assigned at the device level (like today), but with an option to configure different behavior per outlet within that assignment.

*Schedule assignment stays device-level:*

```
Schedule: "Business Hours" — ON 8AM, OFF 10PM, Mon-Sat

Assigned Devices:
┌─────────────┬──────────┬──────────────────────────────────────────────┐
│ Device      │ Model    │ Outlet Configuration                         │
├─────────────┼──────────┼──────────────────────────────────────────────┤
│ 99999       │ I-22     │ (single outlet — uses schedule as-is)        │
├─────────────┼──────────┼──────────────────────────────────────────────┤
│ W52290      │ I-52     │ ● All outlets follow this schedule           │
│             │          │ ○ Custom per-outlet:                         │
│             │          │   Outlet 1 "ATM": Excluded (always on)       │
│             │          │   Outlet 2 "Jukebox": Follows schedule       │
│             │          │   Outlet 3 "Game 1": Follows schedule        │
│             │          │   Outlet 4: Excluded (always on)             │
└─────────────┴──────────┴──────────────────────────────────────────────┘
```

**Comparison:**

| Aspect | Approach A (Per-Outlet) | Approach B (Device + Override) |
|--------|------------------------|-------------------------------|
| Flexibility | Each outlet can be in a completely different schedule | One schedule per device, outlets can opt in/out |
| Complexity | More complex — outlets are first-class schedule participants | Simpler — extends existing device-level model |
| Matches real use cases | "ATM always on, games on business hours" needs 2 schedules | Same scenario: assign "Business Hours", exclude ATM outlet |
| Table display | Outlet sub-rows required | Device rows with expandable detail |
| Bulk actions | Need to think about outlet-level bulk actions | Device-level bulk actions still work |
| Migration risk | New association model (schedule-to-outlet) | Extends existing association (schedule-to-device + outlet config) |

### Schedule Card Enhancement

The company power schedule cards should indicate when multi-outlet devices are included:

```
┌───────────────────────────────────────┐
│  Business Hours              [Active] │
│                                       │
│  (clock) Eastern Time                 │
│  Days active: Mon-Sat                 │
│                                       │
│  (devices) 3 Devices, 8 Outlets      │
│                                       │
│              [edit]   [delete]         │
└───────────────────────────────────────┘
```

- "3 Devices, 8 Outlets" — makes it clear that some devices contribute multiple outlets to this schedule
- Single-outlet devices count as 1 device / 1 outlet
- A four-outlet device with all outlets in the schedule counts as 1 device / 4 outlets

### "Set Same Schedule for All Outlets" Shortcut

From the device page (Part 1), users should be able to quickly assign all four outlets to the same schedule without going to the scheduling page four times:

```
┌─────────────────────────────────────────────────────────────────────┐
│  Outlet Schedules                                                    │
│                                                                      │
│  ○ Set same schedule for all outlets                                 │
│  ● Configure each outlet independently                               │
│                                                                      │
│  Outlet 1 "ATM":     [ No schedule     v ]                          │
│  Outlet 2 "Jukebox": [ Business Hours  v ]                          │
│  Outlet 3 "Game 1":  [ Business Hours  v ]                          │
│  Outlet 4:           [ No schedule     v ]                          │
│                                                                      │
│  [ Cancel ]                                         [ Save ]         │
└─────────────────────────────────────────────────────────────────────┘
```

When "Set same schedule for all outlets" is selected, a single dropdown applies to all four.

---

## Part 3: Model Management — Power Relay Capability

### What It Looks Like Today

The "Power Cycler Capable" flag currently lives at the **configuration group level**, not on the model. The model pages themselves have no awareness of relay capability:

**Browse Models table** — three columns only:
```
┌─────────────────┬──────────────────┬──────────┬──────────────────────────┐
│ Model Name      │ Manufacturer     │ Devices  │                          │
├─────────────────┼──────────────────┼──────────┼──────────────────────────┤
│ I-22            │ InHand           │ 53       │ [View] [Edit] [Delete]   │
│ I-4100          │ InHand           │ 7        │ [View] [Edit] [Delete]   │
│ I-4500          │ Aaron Test       │ 1        │ [View] [Edit] [Delete]   │
│ I-52            │ InHand           │ 0        │ [View] [Edit] [Delete]   │
│ ...             │                  │          │                          │
└─────────────────┴──────────────────┴──────────┴──────────────────────────┘
```

**View Model** — shows Model Name, Manufacturer, Number of Devices, Created/Modified timestamps, Created By/Modified By. No relay field.

**Edit Model** — only two fields: Model Name and Manufacturer dropdown. No relay capability option.

### What Needs to Change (FR-1)

The relay capability flag needs to move from configuration groups to the model level. This affects three model pages: Browse Models, View Model, and Edit Model.

### Proposed: Browse Models Table

Add a **"Power Relay"** column to the table so admins can see at a glance which models support relay and what type:

```
┌─────────────────┬──────────────────┬──────────┬──────────────────┬──────────────────────────┐
│ Model Name      │ Manufacturer     │ Devices  │ Power Relay      │                          │
├─────────────────┼──────────────────┼──────────┼──────────────────┼──────────────────────────┤
│ Aaron Test      │ Aaron Test       │ 4        │ —                │ [View] [Edit] [Delete]   │
│ ATT SIM         │ SIM              │ 0        │ —                │ [View] [Edit] [Delete]   │
│ CR202           │ InHand           │ 4        │ —                │ [View] [Edit] [Delete]   │
│ I-22            │ InHand           │ 53       │ Single-Outlet    │ [View] [Edit] [Delete]   │
│ I-4100          │ InHand           │ 7        │ —                │ [View] [Edit] [Delete]   │
│ I-4500          │ Aaron Test       │ 1        │ Single-Outlet    │ [View] [Edit] [Delete]   │
│ I-52            │ InHand           │ 0        │ Multi-Outlet (4) │ [View] [Edit] [Delete]   │
│ M5              │ InHand           │ 1        │ —                │ [View] [Edit] [Delete]   │
└─────────────────┴──────────────────┴──────────┴──────────────────┴──────────────────────────┘
```

**Column values:**
- **"—"** or blank — model does not support power relay (e.g., I-4100, CR202)
- **"Single-Outlet"** — existing single-relay capability (I-22, I-4500, Systex)
- **"Multi-Outlet (4)"** — four-outlet relay capability (I-52)

### Proposed: View Model

Add a **"Power Relay Capability"** row to the model detail table:

```
┌─────────────────────────────────────────────────────────────────────┐
│  View Model                                                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Model Name                          I-52                            │
│  ─────────────────────────────────────────────────                   │
│  Manufacturer                        InHand                          │
│  ─────────────────────────────────────────────────                   │
│  Power Relay Capability              Multi-Outlet (4)                │
│  ─────────────────────────────────────────────────                   │
│  Number of Devices                   0                               │
│  ─────────────────────────────────────────────────                   │
│  Created                             12/3/25, 12:52 PM              │
│  ─────────────────────────────────────────────────                   │
│  Modified                            12/3/25, 12:52 PM              │
│  ─────────────────────────────────────────────────                   │
│  ...                                                                 │
│                                                                      │
│  [Back]                    [Edit]                          [Delete]   │
└─────────────────────────────────────────────────────────────────────┘
```

For a model with no relay capability, the row would show "None" or "—".

### Proposed: Edit Model

Add a **"Power Relay Capability"** dropdown to the edit form, below the existing Manufacturer field:

```
┌─────────────────────────────────────────────────────────────────────┐
│  Edit Model                                                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Model Name *                                                        │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │ I-52                                                      │       │
│  └──────────────────────────────────────────────────────────┘       │
│                                                                      │
│  Manufacturer *                                                      │
│  ┌──────────────────────────────────────────────────────┬───┐       │
│  │ InHand                                                │ v │       │
│  └──────────────────────────────────────────────────────┴───┘       │
│                                                                      │
│  Power Relay Capability                                              │
│  ┌──────────────────────────────────────────────────────┬───┐       │
│  │ Multi-Outlet (4)                                      │ v │       │
│  └──────────────────────────────────────────────────────┴───┘       │
│                                                                      │
│  Dropdown options:                                                   │
│    • None (default)                                                  │
│    • Single-Outlet                                                   │
│    • Multi-Outlet (4)                                                │
│                                                                      │
│  [Cancel]                                                  [Save]    │
└─────────────────────────────────────────────────────────────────────┘
```

### Migration Note

Today the "Power Cycler Capable" flag lives on configuration groups. As part of this feature:

1. A new "Power Relay Capability" field is added to the model (None / Single-Outlet / Multi-Outlet)
2. Existing config group flags are migrated to the corresponding model records
3. The config group flag is deprecated (and eventually removed)

This means the device page's existing "Power Relay Installed?" field in General Info (currently Yes/No) also derives its meaning from the model. For i52 devices, the new relay activation toggle (FR-2) on the device page replaces this field's role. For single-outlet models (I-22, etc.), the existing "Power Relay Installed?" behavior remains unchanged.

### Naming Consideration

The existing flag is called "Power Cycler Capable" at the config group level. The new model-level field could be called:
- **"Power Relay Capability"** — aligns with the PRD language and the physical hardware (it's a relay)
- **"Power Cycler Type"** — keeps the legacy "Power Cycler" terminology the portal already uses
- **"Relay Type"** — shortest, used in the PRD data model

Recommendation: Use **"Power Relay"** as the column header in the Browse table (compact) and **"Power Relay Capability"** as the field label in View/Edit (descriptive). This transitions away from the "Power Cycler" terminology toward "Power Relay" which is more accurate for the hardware.

---

## Discussion Points

1. **Model-level relay capability naming** — The existing config group flag is called "Power Cycler Capable." Should the new model-level field keep the "Power Cycler" terminology for consistency, or transition to "Power Relay" which is more accurate? The portal uses "Power Cycler" in many places (button labels, table headers, schedule pages), so renaming everything at once could be a larger scope change. Alternatively, this feature could start the transition by using "Power Relay" on the model pages and device page module, while leaving existing "Power Cycler" references on legacy single-outlet UIs unchanged for now.

2. **Grid vs. Table for outlet display on device page** — Grid (Option A) feels more visual and matches the physical relay layout. Table (Option B) is more compact and consistent with existing portal patterns. Leaning grid for the device page since it's only 4 outlets.

3. **Per-outlet schedules vs. device-level with overrides** — Approach A is more flexible but adds complexity. Approach B is simpler but may not handle the "ATM on 24/7, games on business hours" case as cleanly if those require genuinely different on/off times (not just include/exclude). Need stakeholder input on real-world scheduling patterns.

4. **Power Cycler Capable Devices table** — Regardless of scheduling approach, this table needs to handle multi-outlet devices. The expandable sub-row pattern seems like the right fit.

5. **Where does the scheduling entry point live?** The device page outlet cards show a schedule summary, but the full scheduling configuration likely stays on the Company Power Schedules page. The device page can link to the schedule or offer a quick-assign dropdown.

6. **Migration from config group to model** — The "Power Cycler Capable" flag on configuration groups needs to be migrated to model-level "Power Relay Capability." Dev team needs to audit how this is currently stored before designing the migration script.

---

## References

- [PRD-multi-outlet-power-relay.md](PRD-multi-outlet-power-relay.md) — FR-3 (power control), FR-5 (scheduling)
- Screenshots: `screenshots/` folder — current device page, power management, company views
