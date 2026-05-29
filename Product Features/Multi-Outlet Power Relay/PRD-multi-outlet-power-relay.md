# Product Requirements Document
## Multi-Outlet Power Relay (i52)

**Document Version:** 1.0
**Date:** 2026-05-26
**Author:** Aksana Rahouski (Orases)
**Status:** Draft
**Related Tickets:** None yet
**Document Owner:** Aksana Rahouski
**Last Updated:** 26.05.2026

---

## Table of Contents

1. [Overview](#overview)
2. [Goals and Success Criteria](#goals-and-success-criteria)
3. [Users](#users)
4. [Scope](#scope)
5. [Functional Requirements and Business Rules](#functional-requirements-and-business-rules)
6. [Technical Requirements](#technical-requirements)
7. [Testing Requirements](#testing-requirements)
8. [Dependencies and Risks](#dependencies-and-risks)
9. [Open Questions](#open-questions)
10. [References](#references)
11. [Notes](#notes)

---

## Overview

The Multi-Outlet Power Relay feature adds support for a four-outlet power relay accessory that connects to InHand i52 routers via DIO (Digital Input/Output) ports. This enables WATM portal users to remotely and independently power-cycle up to four pieces of equipment connected to a single i52 router at a site. The relay is a "dumb" hardware accessory with no network connectivity of its own — all control is exercised through API calls sent to the i52 router.

For i52 devices where the customer has installed the relay hardware and activated it in the portal, this feature replaces the existing single-outlet "Restart Power Cycler" button with a full power management module supporting per-outlet control, labeling, status display, and scheduling. i52 devices without the relay activated continue to use the existing single restart button.

### Key Features

- **Per-outlet power control**: Independently turn on, turn off, or restart (10-second power cycle) each of four outlets
- **Outlet labeling**: Name each outlet to identify connected equipment (e.g., "ATM", "Jukebox", "Game 1")
- **Per-outlet scheduling**: Set power on/off schedules at the individual outlet level
- **Outlet status display**: Show current on/off state of each outlet based on DIO state from check-ins
- **Relay activation toggle**: Self-service enablement on the device page when a customer installs the relay hardware

### Business Impact

- **Competitive differentiation**: Competitors require separate connections per device at a site. APW can offer one i52 router managing up to four devices with independent power control — a capability no competitor offers.
- **New revenue from accessory sales**: The relay is a one-time hardware purchase. APW has 200 units in inventory. The portal toggle ("Are you utilizing a four-outlet power relay?") serves as a discovery/sales prompt for customers who see it.
- **Customer retention**: Customers with multiple terminals at a site (gaming, kiosk, ATM) get consolidated management from a single router, reducing their operational complexity and monthly costs.

---

## Goals and Success Criteria

### Primary Goals

1. **Enable independent remote power control of up to 4 outlets** per i52 device through the WATM portal
2. **Provide per-outlet scheduling** so customers can automate power on/off times for individual pieces of equipment
3. **Migrate power relay capability from configuration groups to the model level** to align the data model with how relay capability is actually determined (by hardware model, not config group)

### Success Criteria

- Portal users can activate the four-outlet relay on any i52 device and control each outlet independently (on/off/restart)
- Each outlet can be labeled, and labels persist across sessions
- Per-outlet power schedules can be created, edited, and deleted independently of other outlets on the same device
- Outlet status (on/off) displayed on the device page reflects actual DIO state from the most recent check-in
- Existing power relay functionality for i22, i4500, and Systex models remains unchanged
- When the four-outlet relay is activated on an i52 device, the single "Restart Power Cycler" button is replaced by the multi-outlet power management module. i52 devices without the relay activated retain the existing single restart button.

---

## Users

### Primary Users

**Company Device Manager** (e.g., APW customer operations staff)
- **Role**: Manages deployed devices and equipment at customer sites through the WATM portal
- **Need**: Remotely power-cycle individual pieces of equipment without visiting the site and without affecting other equipment at the same location
- **Pain Point**: Currently, the single-outlet relay (i22) can only restart one device. Sites with multiple terminals require either multiple routers or a physical site visit to restart individual equipment.
- **Benefit**: One i52 router with the four-outlet relay provides independent power control over up to 4 terminals, reducing site visits, monthly costs, and operational complexity

**Company Administrator** (e.g., APW operations lead like Devon)
- **Role**: Configures devices, manages schedules, and oversees device fleet
- **Need**: Set up power schedules per outlet (e.g., turn off game terminals at night, keep ATM on 24/7) and activate/deactivate relay capability on devices
- **Pain Point**: Existing scheduling works at the device level only — no ability to schedule power differently for different equipment sharing a single router
- **Benefit**: Per-outlet scheduling provides granular control, reducing energy costs and enabling business-hours-only operation of specific equipment

---

## Scope

### In Scope

**Portal UI — Device Detail Page**
- Relay activation toggle ("Are you utilizing a four-outlet power relay?") on i52 device pages
- Four-outlet power management module with per-outlet controls (on/off toggle, restart button)
- Per-outlet labeling with editable text fields
- Per-outlet status display (on/off) based on DIO state
- Simple four-outlet visual diagram representing the relay layout
- Replacement of the legacy single "Restart Power Cycler" button with the multi-outlet module when the relay is activated on an i52 device

**Portal UI — Scheduling**
- Per-outlet power on/off scheduling, extending the existing power management scheduling feature
- Ability to set the same schedule across all four outlets or configure each independently

**Data Model / Architecture**
- Migration of power relay capability flag from configuration groups to the model level
- Model-level column indicating relay capability (which models support power relay)
- Storage for per-outlet labels, schedules, and relay activation state per device

**API / Device Communication**
- Per-outlet DIO control commands sent to the i52 router via API (inverted logic: high-to-low, back-to-high)
- DIO state reporting through check-in data (requires IO config entry in POM 1.2 configuration)
- Optional direct API query to i52 router to validate current DIO state before sending commands

### Out of Scope

- LAN link-up/link-down monitoring as outlet state validation (explicitly deferred by Adam — depends on alarm management feature)
- Alarm management and notification system (separate backlog item)
- Twilio SMS integration for power event notifications (separate backlog item, not yet started)
- Changes to existing power relay behavior on i22, i4500, or Systex models
- Billing changes — the relay is a one-time hardware purchase with no recurring portal fees
- Per-outlet user permissions (all users with device access can control all outlets)
- Firmware changes to the i52 router
- Support for relay accessories from manufacturers other than the current Electronics Saloon product

---

## Functional Requirements and Business Rules

### FR-1: Model-Level Power Relay Capability

**FR-1.1: The system shall define power relay capability at the model level, not the configuration group level.**
- A new attribute on the device model record indicates whether that model supports power relay
- For i52 models, this attribute shall indicate "multi-outlet power relay" (4 outlets)
- For i22, i4500, and Systex models, this attribute shall indicate "single-outlet power relay" (1 outlet, existing behavior)
- Models without relay capability (e.g., i4100) shall have no relay attribute

**Business Rules:**
- The relay capability attribute determines which power management UI appears on the device detail page
- Migration: Existing power-cycler-capable flags on configuration groups shall be migrated to model-level attributes. The configuration group flag shall be deprecated.

**FR-1.2: The model management page shall display a column indicating relay capability.**
- The column shall show "Multi-Outlet (4)" for i52, "Single-Outlet" for i22/i4500/Systex, and blank/none for models without relay support

---

### FR-2: Relay Activation on Device Page

**FR-2.1: The i52 device detail page shall display a relay activation toggle: "Are you utilizing a four-outlet power relay?"**
- The toggle shall be positioned in the lower section of the device detail page, in the same area as the existing "Use this device as backup internet?" toggle
- Default state for all i52 devices: No (deactivated)
- The toggle is a yes/no control, not a multi-step wizard

**Business Rules:**
- The relay is a "dumb" hardware accessory with no electronic handshake — the portal cannot detect whether a relay is physically connected. The toggle is a user declaration.
- When toggled to "Yes": The four-outlet power management module becomes visible on the device page
- When toggled to "No": The power management module is hidden
- See [Q-4](PRD-multi-outlet-power-relay-QA.md#q-4-relay-activation-toggle--who-can-enabledisable-it) for open question on role permissions for this toggle
- See [Q-7](PRD-multi-outlet-power-relay-QA.md#q-7-what-happens-to-outlet-data-when-relay-is-deactivated) for open question on data retention when deactivated

**Interaction & UI Details:**
- The toggle follows the same visual pattern as the existing backup internet toggle on the device page
- When toggled to "Yes," the power management module shall appear below the toggle without a page reload (inline expand)

---

### FR-3: Per-Outlet Power Control Actions

**FR-3.1: The power management module shall display four outlets, each with independent controls.**

Each outlet shall have:
- **On/Off toggle**: Sets the outlet to a persistent on or off state
- **Restart button**: Executes a 10-second power cycle (turns outlet off, waits 10 seconds, turns back on)
- **Current status indicator**: Shows whether the outlet is currently on or off

**FR-3.1a: The power management module shall include a "Restart All" button that restarts all four outlets simultaneously.**
- Clicking "Restart All" sends DIO LOW commands to all **active** (in-use) outlets simultaneously, waits 10 seconds, then sends DIO HIGH commands simultaneously. Inactive outlets are not affected. Outlets are not queued sequentially.
- A confirmation dialog shall appear before execution (same pattern as individual restart — see Interaction & UI Details below). The dialog reflects the number of active outlets.
- During the restart cycle, all active outlets show "Restarting" state and all individual restart buttons are disabled

**Business Rules:**
- "On" means the DIO port is set to HIGH (default/powered state for the four-outlet relay)
- "Off" means the DIO port is set to LOW (power cut)
- "Restart" means: set DIO to LOW, wait 10 seconds, set DIO to HIGH. This matches existing single-relay restart behavior but with inverted logic.
- The i52 has 4 usable DIO ports (ports 1-4) plus 1 common ground. Each outlet maps to one DIO port.
- See [Q-6](PRD-multi-outlet-power-relay-QA.md#q-6-dio-port-to-outlet-mapping-documentation) for open question on exact port-to-outlet mapping
- See [Q-1](PRD-multi-outlet-power-relay-QA.md#q-1-confirmation-dialog-before-power-actions) for open question on confirmation dialogs
- See [Q-2](PRD-multi-outlet-power-relay-QA.md#q-2-restart-all-capability) for open question on "Restart All" button

**FR-3.2: The system shall send DIO control commands to the i52 router via API to execute power actions.**
- Commands target the i52 router's IP address, specifying the DIO port number for the target outlet
- The command logic is **inverted** from the existing i22 single relay:
  - i22 (existing): LOW-to-HIGH triggers relay, HIGH-to-LOW resets
  - i52 (new): HIGH-to-LOW triggers relay, LOW-to-HIGH resets
- Each outlet is controlled independently; commanding one outlet shall not affect the state of other outlets

**FR-3.3: After a power action is executed, the system shall update the outlet status display.**
- The system shall optimistically update the UI to reflect the expected new state
- The system shall validate the actual DIO state on the next check-in and correct the display if it differs from expected state
- Optionally, the system may make a direct API call to the router to query current DIO state immediately after sending a command for faster validation

**Interaction & UI Details:**
- The four outlets shall be displayed as a simple visual diagram — four labeled boxes arranged to represent the physical relay layout
- Each outlet box shows: outlet number (1-4), custom label (if set), current status (on/off), and action controls
- **Restart confirmation dialog**: When a user clicks the Restart button on an outlet, the system shall display a confirmation dialog before executing the action. The dialog shall include the outlet's custom label (or default name if no label is set) so the user can verify they are restarting the correct outlet. Dialog text: "Are you sure you want to restart '{label}' (Outlet {n})? This will power off the outlet for 10 seconds, then power it back on." with Cancel and "Yes, Restart" buttons. On/Off toggles do not require a confirmation dialog.
- **Restart All confirmation dialog**: When a user clicks the "Restart All" button, a confirmation dialog shall appear: "Are you sure you want to restart all {n} active outlets? This will power off the outlets for 10 seconds, then power them back on." with Cancel and "Yes, Restart All" buttons. The count reflects only active (in-use) outlets.
- While a restart is in progress (10-second cycle), the outlet's status indicator shall show a "restarting" state and the restart button shall be disabled for that outlet
- Action buttons for other outlets remain active during a single outlet's restart

---

### FR-4: Outlet Labeling

**FR-4.1: Each outlet shall have an editable text label field.**
- Labels are optional — outlets without labels display as "Outlet 1", "Outlet 2", etc.
- Labels are saved per-device and persist across sessions
- See [Q-5](PRD-multi-outlet-power-relay-QA.md#q-5-outlet-label-character-limit-and-validation) for open question on character limits and validation

**Business Rules:**
- Labels are unique identifiers for the customer's convenience only — they have no effect on system behavior
- Any user who can view the device page can edit outlet labels
- Example labels: "ATM", "Jukebox", "Game 1", "Redemption Terminal"

**Interaction & UI Details:**
- Labels shall be editable inline on the outlet diagram (click to edit, or edit icon)
- The label appears prominently on each outlet box in the visual diagram

---

### FR-5: Per-Outlet Scheduling

**FR-5.1: The system shall support independent power on/off schedules for each outlet.**
- Scheduling extends the existing power management scheduling feature to operate at the outlet level rather than the device level
- Each outlet can have its own schedule or no schedule
- Users can set the same schedule across all four outlets or configure each independently

**Business Rules:**
- A schedule defines times when an outlet should be powered on and times when it should be powered off (e.g., "Game terminal on at 8:00 AM, off at 10:00 PM daily")
- Schedules operate in the device's configured timezone
- When a scheduled event fires, the system sends the appropriate DIO command to the i52 router for that specific outlet
- See [Q-3](PRD-multi-outlet-power-relay-QA.md#q-3-scheduling-override-behavior) for open question on how manual actions interact with active schedules

**Interaction & UI Details:**
- Scheduling UI shall be accessible from the power management module on the device page
- The UI shall support per-outlet schedule configuration — potentially as tabs per outlet, or a combined view with per-outlet rows
- A "set same schedule for all outlets" shortcut shall be available to avoid repetitive configuration
- Each outlet's current schedule (if any) shall be visible in summary form on the outlet diagram (e.g., "On: 8AM-10PM daily")

---

### FR-6: Outlet Status from Check-In Data

**FR-6.1: The system shall read DIO port states from i52 check-in data and display them as outlet statuses.**
- When an IO config entry is present in the device's POM 1.2 configuration, check-ins report the state (HIGH/LOW) of all four DIO ports
- The portal shall map DIO port states to outlet on/off status and display them on the device page
- HIGH = outlet is ON (powered), LOW = outlet is OFF (power cut)

**Business Rules:**
- Outlet status is only as current as the most recent check-in. The status display shall show the timestamp of the last check-in that included DIO data.
- If no DIO data has been received (e.g., IO config not yet applied), outlet statuses shall display as "Unknown"
- If a check-in DIO state contradicts the expected state (e.g., portal shows "On" but check-in reports LOW), the system shall update the display to match the actual check-in data

**FR-6.2: The IO configuration entry required for DIO state reporting shall be automatically applied when the relay is activated.**
- When a user toggles the relay activation to "Yes" (FR-2.1), the system shall ensure the device's configuration includes the IO config entry in the POM 1.2 text field so that subsequent check-ins report DIO state
- This configuration push follows existing configuration management patterns

---

### FR-8: Per-Outlet Active/In-Use State

**FR-8.1: Each outlet shall have an "active" (in use) toggle indicating whether equipment is physically connected to that outlet.**
- When the relay is first activated on a device, all four outlets default to **inactive** (not in use)
- The user marks each outlet as active when they connect equipment to it
- Active outlets display the full set of controls: on/off toggle, restart button, status indicator, label, and scheduling
- Inactive outlets are visually dimmed or collapsed, displaying only the outlet number and an "Activate" or "Mark as in use" control

**Business Rules:**
- Only active outlets are included in the "Restart All" action. If outlets 1 and 3 are active and outlets 2 and 4 are inactive, "Restart All" restarts only outlets 1 and 3.
- Only active outlets can have schedules assigned. If a user deactivates an outlet that has an active schedule, the schedule is paused (not deleted) and resumes if the outlet is reactivated.
- Only active outlets show power status from check-in DIO data. Inactive outlets do not display a status indicator.
- Labeling is only available on active outlets.
- The "Restart All" confirmation dialog shall reflect the number of active outlets: "Are you sure you want to restart all {n} active outlets? This will power off the outlets for 10 seconds, then power them back on."

**Interaction & UI Details:**
- The active/inactive toggle is a simple control on each outlet box (e.g., a switch or "Mark as in use" / "Mark as not in use" link)
- Transitioning an outlet from inactive to active expands it to show full controls
- Transitioning from active to inactive collapses it to a dimmed/minimal state
- The outlet number (1-4) is always visible regardless of active state, so users can identify which physical port maps to which outlet

---

### FR-7: Legacy Power Cycler Button Behavior (i52)

**FR-7.1: The i52 device detail page shall conditionally display the power management interface based on relay activation state.**
- When the relay is **activated** (toggle set to "Yes"): The single "Restart Power Cycler" button is hidden and replaced by the four-outlet power management module
- When the relay is **not activated** (toggle set to "No"): The existing single "Restart Power Cycler" button remains visible and functional, along with the relay activation toggle
- See [Q-8](PRD-multi-outlet-power-relay-QA.md#q-8-existing-i52-devices--migration-path) for open question on verifying current single-relay usage on i52

**Business Rules:**
- The single "Restart Power Cycler" button shall continue to appear on i22, i4500, and Systex device pages — no changes to those models
- i52 devices that have not activated the relay (toggle set to "No") shall continue to show the existing single "Restart Power Cycler" button plus the relay activation toggle
- The single button and the four-outlet module are mutually exclusive on a given i52 device — activating the relay switches from one to the other

---

### Key User Flows

- **Activation Flow**: Customer installs relay hardware on i52 -> Opens device page in portal -> Toggles "Are you utilizing a four-outlet power relay?" to Yes -> Power management module appears with all 4 outlets inactive -> Customer marks connected outlets as active (e.g., outlets 1-3) -> Customer labels active outlets -> Customer optionally sets schedules
- **Power Cycle Flow**: User opens device page -> Sees four-outlet module -> Clicks "Restart" on Outlet 2 ("Jukebox") -> System sends DIO LOW command to port 2, waits 10 seconds, sends DIO HIGH -> Outlet status shows "Restarting..." then updates to "On" -> Next check-in confirms DIO state
- **Scheduling Flow**: User opens device page -> Opens scheduling for Outlet 3 ("Game 1") -> Sets schedule: On 8:00 AM, Off 10:00 PM, Mon-Sat -> Saves -> Schedule summary appears on outlet box -> System automatically sends DIO commands at scheduled times

---

## Technical Requirements

### API / DIO Commands

- Power control commands shall be sent to the i52 router via API call to the router's IP address
- Each command specifies the target DIO port number (1-4) and the desired state (HIGH or LOW)
- The command logic is inverted from the existing i22 relay:
  - **To power OFF an outlet**: Set DIO port to LOW
  - **To power ON an outlet**: Set DIO port to HIGH
  - **To restart an outlet**: Set DIO port to LOW, wait 10 seconds, set DIO port to HIGH
- The system shall handle API call failures gracefully (router offline, timeout) and display an appropriate error to the user

### Check-In Integration

- DIO state reporting requires an IO config entry in the device's POM 1.2 configuration field
- When present, check-in payloads include the state of all DIO ports as HIGH/LOW values
- The check-in processing pipeline shall parse DIO state data and store it for display on the device page

### Direct API State Query (Optional Enhancement)

- The system may make a direct API call to the i52 router (outside of check-in flow) to query current DIO port states
- This provides faster validation than waiting for the next check-in
- This is an optimization, not a requirement — check-in-based validation is the baseline

### Performance

- Power control commands (on/off/restart) shall be sent to the router within 2 seconds of the user clicking the action button
- Outlet status display shall update from check-in data within the normal check-in processing pipeline (no special priority)

### Data Model Overview

**Device Relay Settings** (per device)

| Field | Type | Required | Validation | Notes |
|-------|------|----------|------------|-------|
| device_id | FK (Device) | Yes | Must reference i52 device | Links to parent device |
| relay_activated | Boolean | Yes | true/false | User toggle state |
| outlet_1_label | String | No | See Q-5 | Custom name for outlet 1 |
| outlet_2_label | String | No | See Q-5 | Custom name for outlet 2 |
| outlet_3_label | String | No | See Q-5 | Custom name for outlet 3 |
| outlet_4_label | String | No | See Q-5 | Custom name for outlet 4 |
| outlet_1_active | Boolean | Yes | true/false, default false | Whether outlet 1 is in use |
| outlet_2_active | Boolean | Yes | true/false, default false | Whether outlet 2 is in use |
| outlet_3_active | Boolean | Yes | true/false, default false | Whether outlet 3 is in use |
| outlet_4_active | Boolean | Yes | true/false, default false | Whether outlet 4 is in use |

**Model Relay Capability** (per model)

| Field | Type | Required | Validation | Notes |
|-------|------|----------|------------|-------|
| model_id | FK (Model) | Yes | Existing model | Links to device model |
| relay_type | Enum | No | none/single/multi | none=no relay, single=1 outlet, multi=4 outlets |

**Outlet Schedules** — Extend existing power schedule data model to include outlet number (1-4) as a discriminator. Exact schema depends on current scheduling table structure.

---

## Testing Requirements

### Test Plan Overview

**Testing Phases:**
1. Unit Testing (Development team) — API command logic, DIO state parsing, data model
2. Integration Testing (QA team) — End-to-end with physical i52 + relay hardware
3. User Acceptance Testing (UAT) (APW — Devon, Adam) — Real-world site scenarios
4. Regression Testing (QA team) — Existing i22/i4500/Systex relay functionality unchanged

### Key Test Scenarios

**IT-1: Relay Activation Toggle**
1. Navigate to an i52 device detail page
2. Toggle "Are you utilizing a four-outlet power relay?" to Yes
3. **Verify:** Four-outlet power management module appears with 4 outlets, each showing "Unknown" status
4. Toggle back to No
5. **Verify:** Power management module is hidden

**IT-2: Individual Outlet Restart**
1. Activate relay on an i52 device with physical relay connected
2. Click "Restart" on Outlet 1
3. **Verify:** API sends DIO LOW command to port 1
4. **Verify:** After 10 seconds, API sends DIO HIGH command to port 1
5. **Verify:** Outlet 1 status shows "Restarting" during the cycle, then "On" after
6. **Verify:** Physical relay outlet 1 powers off then on; outlets 2-4 are unaffected

**IT-3: Outlet On/Off Toggle**
1. Toggle Outlet 3 to "Off"
2. **Verify:** API sends DIO LOW command to port 3
3. **Verify:** Outlet 3 status updates to "Off"
4. **Verify:** Physical relay outlet 3 loses power; outlets 1, 2, 4 unaffected
5. Toggle Outlet 3 back to "On"
6. **Verify:** API sends DIO HIGH command to port 3
7. **Verify:** Outlet 3 status updates to "On"

**IT-4: Outlet Labeling**
1. Click edit on Outlet 2 label
2. Enter "Jukebox"
3. Save
4. **Verify:** Outlet 2 displays "Jukebox" as its label
5. Navigate away and return to device page
6. **Verify:** Label "Jukebox" persists

**IT-5: Per-Outlet Scheduling**
1. Create a schedule for Outlet 1: On at 8:00 AM, Off at 10:00 PM, daily
2. Create a different schedule for Outlet 2: On at 6:00 AM, Off at 11:00 PM, weekdays only
3. Leave Outlets 3 and 4 with no schedule
4. **Verify:** At 10:00 PM, Outlet 1 receives DIO LOW command; Outlet 2 remains on until 11:00 PM
5. **Verify:** Outlets 3 and 4 are unaffected by any schedule

**IT-6: DIO State from Check-In**
1. Activate relay on device, ensure IO config is in POM 1.2
2. Manually set Outlet 1 to Off via portal
3. Wait for next check-in
4. **Verify:** Check-in data includes DIO port 1 = LOW
5. **Verify:** Portal outlet 1 status shows "Off" with check-in timestamp

**IT-7: Inverted Logic Validation**
1. On an i52 with relay, send "On" command to Outlet 1
2. **Verify:** API sends DIO HIGH (not LOW) — confirming inverted logic from i22
3. On an i22 with existing single relay, click "Restart Power Cycler"
4. **Verify:** API sends LOW-to-HIGH (existing behavior unchanged)

**UAT-1: Full Site Setup**
- **Persona:** Company Device Manager (APW operations)
- **Scenario:** Customer deploys i52 router at a gaming location with 3 terminals. They install the four-outlet relay, connect terminals to outlets 1-3, and leave outlet 4 unused.
- **Steps:** Activate relay in portal, label outlets 1-3 with terminal names, set weekday schedule for all 3, manually restart outlet 2 when a terminal freezes
- **Success Criteria:** All 3 outlets are independently controllable, labels and schedules persist, unused outlet 4 causes no issues

**RT-1: i22 Single Relay Regression**
- **Verify:** i22 devices still show the single "Restart Power Cycler" button
- **Verify:** Clicking the button still triggers the existing LOW-to-HIGH power cycle
- **Verify:** No four-outlet UI appears on i22 devices

**RT-2: i4500 / Systex Relay Regression**
- **Verify:** Existing power relay behavior on i4500 and Systex models is unchanged

### Testing Notes

- Integration and UAT testing requires physical hardware: i52 router + four-outlet relay + wiring harness
- Adam is building ~10 prototype units; one should be provided to Orases for testing
- DIO state validation via check-ins requires the IO config entry to be present in the device's POM 1.2 configuration

---

## Dependencies and Risks

### Dependencies

**Must Exist Before Development:**
1. **Physical hardware for testing** — i52 router + four-outlet relay + wiring harness from Adam/APW | **Mitigation:** Adam building ~10 prototypes; request one be shipped to Orases before dev starts
2. **DIO command documentation** — Exact API call format, DIO port numbering, and inverted logic specification | **Mitigation:** Adam confirmed API is tested and working; request formal documentation
3. **DIO port-to-outlet mapping** — Which DIO port number controls which physical outlet position | **Mitigation:** See Q-6; request from Adam/InHand

**Required for Testing:**
- IO config entry format for POM 1.2 to enable DIO state reporting in check-ins

**Integrates With:**
- Existing check-in processing pipeline — DIO state parsing
- Existing power management scheduling system — extended to per-outlet granularity
- Existing device configuration management — IO config push to POM 1.2
- Device model management — new relay capability attribute

### Risks

**MEDIUM RISK: Inverted DIO Command Logic**
- **Description:** The i52 four-outlet relay uses inverted DIO logic (HIGH-to-LOW) compared to the existing i22 single relay (LOW-to-HIGH). Incorrect implementation could leave outlets in the wrong state.
- **Impact:** High — Could leave customer equipment powered off or prevent power cycling
- **Probability:** Low — Adam has tested and confirmed the inverted logic works
- **Mitigation:** Document exact command sequences. Test extensively with physical hardware before release. Include IT-7 (inverted logic validation) in test plan.

**MEDIUM RISK: No Hardware Detection**
- **Description:** The relay is a "dumb" accessory — the portal cannot detect whether it's physically connected. Users self-declare via the activation toggle.
- **Impact:** Medium — Users could activate the relay toggle without hardware connected, then be confused when power commands have no effect
- **Probability:** Low — APW controls hardware sales and customer onboarding
- **Mitigation:** Display a note/tooltip on the activation toggle explaining that the physical relay must be installed first. Outlet status showing "Unknown" (no DIO data) serves as an indirect indicator.

**LOW RISK: Legacy Button Replacement on i52**
- **Description:** When a user activates the four-outlet relay, the existing single "Restart Power Cycler" button is replaced by the multi-outlet module. Users must understand the transition.
- **Impact:** Low — The single button remains for i52 devices without the relay activated, so no one loses existing functionality
- **Probability:** Low — Relay activation is an explicit user action; users who activate it are aware they have the hardware
- **Mitigation:** Clear UI messaging on the activation toggle. Adam to verify current single-relay usage on i52 (see Q-8) to inform any migration guidance needed.

**LOW RISK: Scheduling Complexity**
- **Description:** Extending device-level scheduling to per-outlet granularity adds complexity to the scheduling data model and UI.
- **Impact:** Medium — Could increase development effort and introduce edge cases (e.g., schedule conflicts between outlets)
- **Probability:** Medium — Scheduling is a known complex area
- **Mitigation:** Start with simple per-outlet independent schedules. Defer complex features like inter-outlet dependencies or conflict resolution.

---

## Open Questions

See [PRD-multi-outlet-power-relay-QA.md](PRD-multi-outlet-power-relay-QA.md) for the full Q&A document with 8 open questions requiring stakeholder input.

### Summary of Open Items

| ID | Topic | Decision Owner | Status |
|----|-------|---------------|--------|
| Q-1 | Confirmation dialog before power actions | Devon / Adam (APW) | **Resolved** — Confirmation on Restart only, showing outlet label |
| Q-2 | "Restart All" button | Devon / Adam (APW) | **Resolved** — Yes, included. All outlets restart simultaneously. |
| Q-3 | Scheduling override behavior (manual vs. scheduled) | Devon / Adam (APW) | Open |
| Q-4 | Role permissions for relay activation toggle | Devon (APW) | Open |
| Q-5 | Outlet label character limit and validation | Aksana / Dev team | Open |
| Q-6 | DIO port-to-outlet mapping documentation | Adam (APW) / InHand | Open |
| Q-7 | Data retention when relay is deactivated | Devon / Adam (APW) | Open |
| Q-8 | Existing i52 single relay usage verification | Adam (APW) | Open |

---

## References

### Related Features
- [Existing Power Relay Schedule](link) — Current device-level scheduling feature (built for single outlet, one customer)
- [Alarm Management](link) — Future feature for device alerts and notifications (backlog item, not yet started)
- [Twilio SMS Integration](link) — Future feature for notification delivery (backlog item, above alarm management in priority)

### Supporting Documentation
- [Multi-Outlet Power Relay Discovery Document](Multi_Outlet_Power_Relay_IR315.md) — Original discovery notes from March 25, 2026 meeting
- [APW Check-in Transcript — May 11, 2026](../../Meetings/Transcripts/2026-05-11_APW_Check-in.md) — Meeting where detailed requirements were discussed
- [Q&A Document](PRD-multi-outlet-power-relay-QA.md) — Open questions requiring stakeholder answers

---

## Notes

### Evidence Sources
- **Multi_Outlet_Power_Relay_IR315.md** — Discovery document from March 25, 2026 APW x InHand x Orases meeting. Source for hardware specs, business context, and initial discovery questions.
- **2026-05-11_APW_Check-in.md** — May 11, 2026 meeting transcript. Source for activation flow, UX decisions, technical details (inverted DIO logic, check-in integration), scope decisions (LAN monitoring out, scheduling in), and stakeholder positions.

### Key Decisions

| Decision | Rationale | Source |
|----------|-----------|--------|
| i52 only — no multi-outlet on other models | i52 has 4 DIO ports; i22 has only 2 IO ports and cannot support 4-outlet relay | May 11 meeting (Adam) |
| Relay activation is a manual toggle, not auto-detected | Relay is a "dumb" accessory with no electronic handshake | May 11 meeting (Devon, Adam) |
| No billing changes | Relay is one-time hardware purchase; no recurring portal fee | May 11 meeting (Devon) |
| Replace single "Restart Power Cycler" with multi-outlet module when relay is activated on i52 | New module provides per-outlet control; devices without relay keep existing single button | May 11 meeting (Adam) |
| Capability moves from config groups to model level | Config group flag was a historical workaround; model is the correct level | May 11 meeting (Devon confirmed "100%") |
| LAN monitoring is NOT MVP | Not all customer equipment connects to router via ethernet; depends on alarm management feature | May 11 meeting (Adam: "I definitely don't feel good about it for MVP") |
| Per-outlet scheduling IS in scope | Customers need different schedules per equipment type (e.g., games off at night, ATM always on) | May 11 meeting (Devon, Adam) |
| UX pattern: toggle similar to backup internet toggle | Consistent with existing portal patterns; Adam proposed this approach | May 11 meeting (Adam) |
| Power management module in lower device page section | Same area as wifi/cellular backup toggle, not in upper device info | May 11 meeting (Adam) |

### Confidence Levels
- FR-1 through FR-4, FR-6, FR-7: **High confidence** — Explicitly discussed and agreed in May 11 meeting
- FR-5 (Scheduling): **Medium confidence** — Per-outlet scheduling was confirmed in scope, but the detailed UX (tabs vs. rows, "set all" shortcut) is inferred from discussion, not explicitly designed
- Technical Requirements (DIO commands): **High confidence** — Adam confirmed API tested and working; inverted logic explicitly stated

---

END OF DOCUMENT
