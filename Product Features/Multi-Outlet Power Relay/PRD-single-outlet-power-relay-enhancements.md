# Product Requirements Document
## Single-Outlet Power Relay Enhancements (Phase 1)

**Document Version:** 1.0
**Date:** 2026-06-03
**Author:** Aksana Rahouski (Orases)
**Status:** Draft
**Source Ticket:** [WATM-2163 — New Digital IO Feature](https://orases.atlassian.net/browse/WATM-2163)
**Related:** [PRD — Multi-Outlet Power Relay (Phase 2)](PRD-multi-outlet-power-relay.md)
**Document Owner:** Aksana Rahouski
**Last Updated:** 2026-06-03

---

## Table of Contents

1. [Overview](#overview)
2. [Current State and Problems](#current-state-and-problems)
3. [Goals and Success Criteria](#goals-and-success-criteria)
4. [Users](#users)
5. [Scope](#scope)
6. [Functional Requirements and Business Rules](#functional-requirements-and-business-rules)
7. [Technical Requirements](#technical-requirements)
8. [Testing Requirements](#testing-requirements)
9. [Dependencies and Risks](#dependencies-and-risks)
10. [Open Questions](#open-questions)
11. [Relationship to Phase 2 (Multi-Outlet)](#relationship-to-phase-2-multi-outlet)
12. [References](#references)
13. [Notes](#notes)

---

## Overview

This document defines Phase 1 enhancements to the WATM portal's power relay management system. The work addresses long-standing reliability and usability issues with how the portal tracks, detects, and gates power relay functionality on single-outlet devices (I-22, I-4500, Systex).

Today, the portal has no way to verify whether a power relay is physically connected to a device. A manual checkbox (`is_power_relay_installed`) is the only indicator, and it's frequently wrong — field technicians don't document installs, customers don't know their hardware state, and support agents can't confirm relay presence before attempting a remote power cycle. When a power cycle command is sent to a device without a relay, it silently does nothing, wasting time and eroding customer confidence.

During the June 2, 2026 Sponsor Update call, Devon D'Andrea and Adam Curcie revealed a discovery that solves this problem: InHand routers can detect whether a relay is physically connected by temporarily switching the IO port from output mode to input mode. This enables automated, hardware-verified relay detection with ~100% accuracy — a capability that has been a customer and trade-show pain point for approximately six years.

This Phase 1 work establishes the foundation for the Multi-Outlet Power Relay feature (Phase 2), which extends these same patterns to support four independent outlets on I-52 devices.

### Key Features

- **Model-level relay type**: Replaces the boolean `is_power_cycler_capable` flag with a `relay_type` enum (none / single / multi) on device models, with proper admin UI on Device Model pages
- **Automated relay detection**: A "Check Relay" button on the device page runs a 3-step IO detection sequence to verify whether a power relay is physically connected — replacing the unreliable manual checkbox
- **Gated power controls**: Power action buttons (Restart, On, Off) are only shown when relay presence has been hardware-verified, eliminating silent failures

### Business Impact

- **Eliminates a 6-year trade show pain point**: Automated relay detection answers the #1 question prospective customers ask — "can you tell if the relay is connected?"
- **Reduces support waste**: No more failed power cycle attempts on devices without relays, saving support time and improving customer trust
- **Establishes foundation for Phase 2**: The model-level data model, detection API methods, and gating patterns are directly reused by the Multi-Outlet Power Relay feature (i52)

### Layers of Work

| Layer | What It Does | Why It Matters |
|-------|-------------|----------------|
| **Layer 1: Model-Level Relay Capability** | Moves relay capability management from Config Group pages to Device Model pages and evolves the flag into a relay type enum | Data model accuracy — relay capability is determined by hardware model, not configuration group |
| **Layer 2: Automated Relay Detection** | Replaces the manual `is_power_relay_installed` checkbox with a hardware-verified "Check Relay" button | Eliminates guesswork — support, customers, and the system itself can confirm relay presence in seconds |
| **Layer 3: Gate Power Actions Behind Detection** | Only shows power control buttons (restart, on, off) when relay presence is hardware-verified | Prevents silent failures — no more sending power cycle commands to devices without relays |

---

## Current State and Problems

### How Relay Capability Is Tracked Today

**Model-Level: `is_power_cycler_capable` (boolean on `device_models` table)**

The flag already lives on the device model record (added via migration `20250626192136`), and is set to `true` for I-22 and I-52 models. However, the UI for viewing and editing this flag is on the **Configuration Group** pages (`ConfigGroups/edit.php`, `view.php`, `index.php`), not on the Device Model pages. The Device Model Browse, View, and Edit pages have no awareness of relay capability.

This creates confusion: the data is on the model, but the admin experience suggests it's a config group setting. It also means you can't see at a glance which models support power relay when browsing device models.

**Device-Level: `is_power_relay_installed` (boolean on `devices` table)**

A manual checkbox on the device edit page, labeled "Power Relay Installed?" with a tooltip: *"Power Relay Installed is a manual flag for tracking purposes. We do not validate that a power relay is actually connected to the device."*

The portal explicitly warns users that this flag is not reliable.

### Problems With the Current Approach

**Problem 1: Manual tracking is unreliable**

Field technicians install routers and relays at customer sites. They frequently do not document whether a relay was connected. The `is_power_relay_installed` checkbox depends on someone manually setting it, and there is no enforcement or verification. Devon D'Andrea stated during the June 2 call: *"People would reach out to us all the time 'cause they don't take notes when they go do installs."*

**Problem 2: Silent power cycle failures**

The "Restart Power Cycler" button appears on any device whose model has `is_power_cycler_capable = true`, regardless of whether a relay is physically connected. When a user clicks "Restart Power Cycler" on a device without a relay:
- The portal sends the DIO command to the router
- The router executes the command on an empty IO port
- Nothing happens — no power cycle, no error, no feedback
- The user thinks the service is broken or the device is unresponsive

This wastes support time and erodes customer trust.

**Problem 3: No hardware verification for 6 years**

APW has attended trade shows for years where potential customers ask "can you detect if the relay is connected?" The answer has always been "no." Devon and Adam discovered during testing (reported June 1-2, 2026) that InHand routers can detect relay presence via IO input mode circuit detection — a capability that was available the entire time but never utilized.

**Problem 4: Relay capability UI is in the wrong place**

Admins who want to understand which device models support power relay must navigate to Config Group pages. The Device Model pages — the logical place to see hardware capabilities — show no relay information. This will become more confusing when multi-outlet relay (Phase 2) introduces different relay types per model.

---

## Goals and Success Criteria

### Primary Goals

1. **Establish relay capability as a model-level attribute** with proper UI on Device Model pages, replacing the current Config Group-based management
2. **Enable automated hardware detection** of whether a power relay is physically connected to a device, replacing the unreliable manual flag
3. **Gate power control actions** (restart, on, off) behind verified relay detection to eliminate silent failures

### Success Criteria

- ✅ Admins can view and edit relay capability (None / Single-Outlet / Multi-Outlet) on Device Model pages
- ✅ The Config Group pages no longer serve as the primary UI for relay capability management
- ✅ A "Check Relay" action on the device page runs the IO detection sequence and reports whether a relay is physically connected
- ✅ The `is_power_relay_installed` flag is automatically updated based on detection results
- ✅ Power control buttons (Restart Power Cycler, On, Off) only appear on devices where relay presence has been confirmed
- ✅ No changes to how power commands are actually sent to routers — only to when and whether they are offered in the UI
- ✅ Existing power relay functionality on I-22, I-4500, and Systex models continues to work with no regression

---

## Users

### Primary Users

**Company Device Manager / Support Agent** (e.g., APW operations staff)
- **Need:** Quickly verify whether a device has a relay connected before attempting a power cycle
- **Pain Point:** Currently relies on a manual checkbox that may be wrong, leading to failed power cycles with no feedback
- **Benefit:** One-click relay detection provides a definitive answer in seconds

**Company Administrator** (e.g., Devon D'Andrea, APW operations lead)
- **Need:** Audit the fleet to know which devices actually have relays installed, and trust that power management features only appear where they'll work
- **Pain Point:** The manual flag is unreliable; no way to bulk-verify relay presence across the fleet
- **Benefit:** Detection results auto-update the flag, making the device list an accurate reflection of hardware state

**System Administrator / Orases Admin** (e.g., portal admins managing device models)
- **Need:** Configure which device models support relay capability and what type (single vs. multi-outlet)
- **Pain Point:** Relay capability is managed on Config Group pages, disconnected from the Device Model pages where it logically belongs
- **Benefit:** Model-level relay type is visible and editable directly on Device Model pages

---

## Scope

### In Scope

**Layer 1 — Model-Level Relay Capability**
- Add `relay_type` enum field to Device Model (None / Single-Outlet / Multi-Outlet)
- Add "Power Relay" column to Browse Models table
- Add "Power Relay Capability" row to View Model page
- Add "Power Relay Capability" dropdown to Edit Model form
- Migrate existing `is_power_cycler_capable = true` records to appropriate `relay_type` values
- Deprecate relay capability display/editing on Config Group pages

**Layer 2 — Automated Relay Detection**
- "Check Relay" button on device detail page for devices whose model has relay capability
- 3-step IO detection sequence via RouterApi (set input mode → read io_level → restore output mode)
- Pre-check: verify the router has a physical IO port before attempting detection
- Auto-update `is_power_relay_installed` flag based on detection result
- Remove manual editing of `is_power_relay_installed` — flag becomes system-managed
- Display detection result and timestamp to the user

**Layer 3 — Gate Power Actions Behind Detection**
- Conditionally show/hide power control buttons based on `is_power_relay_installed` state
- When `is_power_relay_installed = false` or never checked: show "Check Relay" button, hide power controls
- When `is_power_relay_installed = true`: show power controls (Restart Power Cycler, On, Off), keep "Check Relay" available for re-verification

### Out of Scope

- Multi-outlet relay UI, activation toggle, per-outlet controls, labeling, or scheduling (Phase 2)
- Changes to how power commands are sent to routers (DIO command logic stays the same)
- Bulk relay detection across multiple devices (potential future enhancement)
- Alarm management or notification systems
- Billing changes
- Firmware changes to InHand routers

---

## Functional Requirements and Business Rules

### FR-1: Model-Level Relay Capability

**FR-1.1: The system shall define power relay capability at the device model level using a relay type attribute.**
- The `relay_type` field replaces the existing `is_power_cycler_capable` boolean
- Values: `none` (default), `single` (single-outlet relay), `multi` (multi-outlet relay, 4 outlets)
- For Phase 1, only `none` and `single` are operationally relevant; `multi` is stored but its behavior is implemented in Phase 2

**FR-1.2: The Browse Models page shall display a "Power Relay" column.**
- Column values: blank/dash for `none`, "Single-Outlet" for `single`, "Multi-Outlet (4)" for `multi`

**FR-1.3: The View Model page shall display a "Power Relay Capability" row.**
- Shows the relay type value for the model

**FR-1.4: The Edit Model page shall include a "Power Relay Capability" dropdown.**
- Options: None (default), Single-Outlet, Multi-Outlet (4)

**FR-1.5: Existing `is_power_cycler_capable` data shall be migrated to `relay_type`.**
- `is_power_cycler_capable = true` on I-22, I-4500 → `relay_type = single`
- `is_power_cycler_capable = true` on I-52 → `relay_type = multi`
- All others → `relay_type = none`
- The `is_power_cycler_capable` column is deprecated; existing references in the codebase are updated to use `relay_type`

**FR-1.6: The Config Group pages shall no longer serve as the primary UI for relay capability.**
- The checkbox on Config Group Edit is removed or made read-only with a note directing admins to the Device Model page
- The Config Group View and Browse pages may continue to display relay capability as read-only for reference during the transition period

**Business Rules:**
- The relay type is a hardware characteristic of the model — it does not change per-device or per-configuration
- Changing a model's relay type affects all devices of that model (e.g., controls shown/hidden on device pages)

---

### FR-2: Automated Relay Detection

**FR-2.1: The device detail page shall display a "Check Relay" button for devices whose model has relay capability (`relay_type` = `single` or `multi`).**
- The button is available regardless of the current `is_power_relay_installed` state
- Clicking the button initiates the detection sequence

**FR-2.2: The detection sequence shall execute the following steps via the device's RouterApi:**

1. **Pre-check: Verify the router has a physical IO port**
   - Command: TBD (Devon to provide the specific API call)
   - If the router does not have an IO port → display "This device does not have an IO port" and abort
   - If the router is unreachable → display appropriate error and abort

2. **Step 1: Set IO to input mode**
   - API call: `POST /iodigital.cgi` with body `type=config&io=1&mode=in`
   - Expected response: `SUCCESS`

3. **Step 2: Read the IO level**
   - API call: `POST /iodigital.cgi` with body `type=get&io=1`
   - Response: `io_level=low` (relay IS connected) or `io_level=high` (relay is NOT connected)

4. **Step 3: Restore IO to output mode**
   - API call: `POST /iodigital.cgi` with body `type=config&io=1&mode=out`
   - Expected response: `SUCCESS`
   - This step is critical — the device must be in output mode for normal power relay commands to function

**FR-2.3: The detection result shall automatically update the `is_power_relay_installed` flag on the device record.**
- `io_level=low` → set `is_power_relay_installed = true`
- `io_level=high` → set `is_power_relay_installed = false`

**FR-2.4: The detection result shall be displayed to the user on the device page.**
- Success with relay detected: "Relay detected — power relay is connected to this device." with a green indicator
- Success with no relay: "No relay detected — no power relay is connected to this device." with a gray/neutral indicator
- Failure (router unreachable, IO port not present, command failed): appropriate error message with guidance

**FR-2.5: The system shall record a timestamp of the last relay detection check.**
- Displayed on the device page: "Last checked: [date/time]"
- This helps users understand how current the detection result is

**FR-2.6: The `is_power_relay_installed` field shall no longer be manually editable by users.**
- The checkbox on the device edit page is removed
- The field is now system-managed, updated only by the detection sequence
- The device view page continues to display the value, now with the "Last checked" timestamp

**Business Rules:**
- The detection sequence temporarily switches the IO port to input mode. During this brief window (~1-2 seconds), the device cannot send power relay commands. This is acceptable because detection is a deliberate user-initiated action, not a background process.
- Detection should be available on-demand via a button, not run automatically on every check-in or page load (it involves direct API calls to the router, which has performance and reliability implications).
- If Step 3 (restore to output mode) fails, the system should retry and/or alert the user that the IO port may be in the wrong mode. This is a critical safety concern.

**Interaction & UI Details:**
- The "Check Relay" button should be positioned near the current "Power Relay Installed?" display on the device view page
- During the detection sequence, the button should show a loading/spinner state and be disabled to prevent duplicate requests
- The result should appear inline without a page reload

---

### FR-3: Gate Power Actions Behind Verified Detection

**FR-3.1: Power control buttons shall only be displayed when `is_power_relay_installed = true`.**

- **Relay confirmed (`is_power_relay_installed = true`):** Show "Restart Power Cycler" button (and On/Off controls if applicable). The "Check Relay" button remains available for re-verification.
- **Relay not confirmed (`is_power_relay_installed = false`):** Hide power control buttons. Show a message: "No relay detected on this device. Use 'Check Relay' to verify." The "Check Relay" button is prominently displayed.
- **Never checked (`is_power_relay_installed = null` or no detection timestamp):** Hide power control buttons. Show a message: "Relay status unknown. Use 'Check Relay' to verify whether a power relay is connected." The "Check Relay" button is prominently displayed.

**FR-3.2: The power management pages (Company Power Schedules, Power Cycler Capable Devices table) shall reflect relay detection state.**
- Devices without a confirmed relay should be visually distinguished in the power management device list (e.g., grayed out, flagged, or filtered)
- This prevents admins from assigning schedules to devices that may not have relays

**Business Rules:**
- Gating is based on the `is_power_relay_installed` flag value, which is now hardware-verified via FR-2
- If a device previously had a confirmed relay but the relay is later physically removed, the flag will remain `true` until someone runs "Check Relay" again. This is acceptable — the flag represents the last known state.
- Power commands continue to work exactly as they do today when gating allows them. No changes to DIO command logic, scheduling, or state management.

**Interaction & UI Details:**
- The transition from "unknown/no relay" state to "relay confirmed" should be seamless — after a successful "Check Relay" that finds a relay, power control buttons appear immediately without a page reload
- For devices where relay was previously confirmed, opening the device page shows power controls immediately (no need to re-check every time)

---

### Key User Flows

**Flow 1: First-Time Relay Verification**
1. Admin opens device page for an I-22 device
2. Sees "Relay status unknown" message and "Check Relay" button — no power control buttons visible
3. Clicks "Check Relay"
4. System runs detection sequence (2-3 seconds)
5. Result: "Relay detected" → `is_power_relay_installed` set to `true`
6. Power control buttons (Restart Power Cycler) appear immediately
7. "Last checked" timestamp is displayed

**Flow 2: Device Without a Relay**
1. Admin opens device page for an I-22 device
2. Clicks "Check Relay"
3. Result: "No relay detected"
4. Power control buttons remain hidden
5. Admin knows not to attempt a power cycle and can arrange for relay installation if needed

**Flow 3: Re-Verification After Hardware Change**
1. A field tech removes the relay from a device
2. Admin suspects relay was removed, opens device page
3. Power controls are still visible (flag was `true` from previous check)
4. Clicks "Check Relay" to re-verify
5. Result: "No relay detected" → `is_power_relay_installed` set to `false`
6. Power control buttons disappear

**Flow 4: Admin Configuring Model Relay Capability**
1. Admin navigates to Device Models → Browse
2. Sees "Power Relay" column showing capability per model
3. Clicks Edit on a model
4. Sets "Power Relay Capability" to "Single-Outlet"
5. All devices of that model now show the "Check Relay" button on their device pages

---

## Technical Requirements

### Layer 1: Data Model Changes

**Device Models table (`device_models`):**
- Add `relay_type` enum column: `none` (default), `single`, `multi`
- Migrate existing data: `is_power_cycler_capable = true` → appropriate `relay_type` based on model name
- Deprecate `is_power_cycler_capable` column (retain for backward compatibility during transition, remove in a subsequent release)

**Devices table (`devices`):**
- `is_power_relay_installed` remains but becomes system-managed (not user-editable)
- Add `relay_last_checked_at` datetime column (nullable) to store the timestamp of the last detection check

### Layer 2: RouterApi Changes

**New methods in `RouterApi.php`:**
- `checkIoPortExists()` — Pre-check whether the router has a physical IO port (command TBD from Devon)
- `setIoMode($port, $mode)` — Sets IO port mode to 'in' or 'out' via `type=config&io={port}&mode={mode}`
- `getIoLevel($port)` — Reads IO port level via `type=get&io={port}`, returns 'low' or 'high'
- `detectRelay($port)` — Orchestrates the full 3-step detection sequence with error handling and guaranteed restore to output mode

### Layer 3: Controller/View Changes

**DevicesController:**
- New action: `checkRelay($deviceId)` — Runs detection, updates device record, returns result (AJAX endpoint)

**Device View template:**
- Remove manual `is_power_relay_installed` checkbox from edit form
- Add "Check Relay" button with inline result display
- Conditionally show/hide power control buttons based on `is_power_relay_installed` state
- Display "Last checked" timestamp when available

**Device Model templates:**
- Browse: Add "Power Relay" column
- View: Add "Power Relay Capability" row
- Edit: Add "Power Relay Capability" dropdown

**Config Group templates:**
- Remove or make read-only the `is_power_cycler_capable` checkbox
- Add note directing admins to Device Model pages for relay capability management

### Performance

- The relay detection sequence involves 3-4 API calls to the router (pre-check + 3 steps). Each call has network latency. Expected total time: 2-5 seconds.
- Detection is user-initiated (button click), not background. No impact on check-in processing or scheduled operations.
- The `setIoMode` restore-to-output call (Step 3) should have retry logic to prevent leaving the device in input mode.

### Error Handling

- If the router is unreachable: display "Unable to connect to device. Ensure the device is online and try again."
- If Step 1 or Step 2 fails: abort and attempt to restore output mode (Step 3). Display error to user.
- If Step 3 (restore to output mode) fails: retry up to 3 times. If still failing, display a warning: "Warning: The device IO port may be in input mode. Power relay commands may not work until the IO port is restored to output mode. Contact support."
- Log all detection attempts (success and failure) for troubleshooting.

---

## Testing Requirements

### Test Plan Overview

**Testing Phases:**
1. Unit Testing (Dev team) — RouterApi methods, data model migration, conditional UI logic
2. Integration Testing (QA team) — End-to-end with physical router + relay hardware
3. User Acceptance Testing (APW — Devon, Adam) — Real-world verification on production devices
4. Regression Testing (QA team) — Existing power cycler functionality unchanged

### Key Test Scenarios

**Layer 1 Tests:**

**T-1.1: Model Relay Type Display**
1. Navigate to Device Models → Browse
2. **Verify:** "Power Relay" column is visible
3. **Verify:** I-22 shows "Single-Outlet", I-52 shows "Multi-Outlet (4)", I-4100 shows blank/dash

**T-1.2: Model Relay Type Editing**
1. Navigate to Device Models → Edit for I-22
2. **Verify:** "Power Relay Capability" dropdown is present with options: None, Single-Outlet, Multi-Outlet (4)
3. **Verify:** Current value is "Single-Outlet"
4. Change to "None", save
5. **Verify:** I-22 devices no longer show "Check Relay" button or power controls
6. Revert to "Single-Outlet"

**T-1.3: Migration Verification**
1. After migration runs, query `device_models` table
2. **Verify:** I-22 has `relay_type = single`
3. **Verify:** I-52 has `relay_type = multi`
4. **Verify:** I-4100, CR202, etc. have `relay_type = none`

**Layer 2 Tests:**

**T-2.1: Relay Detection — Relay Connected**
1. Navigate to device page for a device with a physical relay connected
2. Click "Check Relay"
3. **Verify:** Loading/spinner state appears
4. **Verify:** Result displays "Relay detected"
5. **Verify:** `is_power_relay_installed` is set to `true`
6. **Verify:** "Last checked" timestamp is displayed and current

**T-2.2: Relay Detection — No Relay Connected**
1. Navigate to device page for a device without a relay
2. Click "Check Relay"
3. **Verify:** Result displays "No relay detected"
4. **Verify:** `is_power_relay_installed` is set to `false`

**T-2.3: Relay Detection — Router Offline**
1. Navigate to device page for an offline device
2. Click "Check Relay"
3. **Verify:** Error message displays: "Unable to connect to device"
4. **Verify:** `is_power_relay_installed` is not changed

**T-2.4: Relay Detection — IO Port Restoration**
1. Run relay detection on a device with a relay
2. **Verify:** After detection, the device's IO port is in output mode
3. **Verify:** Normal power relay commands (Restart Power Cycler) work immediately after detection

**T-2.5: Manual Flag Removed**
1. Navigate to device edit page
2. **Verify:** "Power Relay Installed?" checkbox is no longer editable
3. **Verify:** Device view page still displays the value as read-only with "Last checked" timestamp

**Layer 3 Tests:**

**T-3.1: Power Controls Hidden When Relay Not Confirmed**
1. Navigate to device page for a device where `is_power_relay_installed = false`
2. **Verify:** "Restart Power Cycler" button is NOT visible
3. **Verify:** Message displays: "No relay detected. Use 'Check Relay' to verify."
4. **Verify:** "Check Relay" button IS visible

**T-3.2: Power Controls Shown When Relay Confirmed**
1. Navigate to device page for a device where `is_power_relay_installed = true`
2. **Verify:** "Restart Power Cycler" button IS visible and functional
3. **Verify:** "Check Relay" button remains available for re-verification

**T-3.3: Power Controls Appear After Successful Detection**
1. Start on device page with no relay confirmed (power controls hidden)
2. Click "Check Relay" — relay IS detected
3. **Verify:** Power controls appear immediately without page reload

**T-3.4: Power Controls Disappear After Failed Detection**
1. Start on device page with relay previously confirmed (power controls visible)
2. Physically disconnect relay
3. Click "Check Relay" — relay is NOT detected
4. **Verify:** Power controls disappear immediately

**Regression Tests:**

**RT-1: Existing Power Cycler Functionality**
- **Verify:** Devices with confirmed relays can still restart, turn on, turn off as before
- **Verify:** Power schedules continue to execute on devices with confirmed relays
- **Verify:** No changes to DIO command logic (same API calls to router)

**RT-2: Non-Relay Models Unaffected**
- **Verify:** Devices with `relay_type = none` (e.g., I-4100) show no relay-related UI elements
- **Verify:** No "Check Relay" button on non-relay-capable models

---

## Dependencies and Risks

### Dependencies

**Must Exist Before Development:**

| Dependency | Source | Status |
|-----------|--------|--------|
| IO port pre-check command — the API call to verify whether a router has a physical IO port | Devon D'Andrea (APW) | **Not yet provided** — Devon mentioned during the June 2 call that this command needs to be added to WATM-2163 |
| Confirmation of detection sequence on multiple device models (I-22, I-52, I-4500) | Devon / Adam (APW) | **Partially tested** — Devon tested on I-22 and I-4100 during the call; broader testing across models needed |

**Integrates With:**
- Existing `RouterApi.php` — new methods added alongside existing power cycler methods
- Existing device detail page — new "Check Relay" button and conditional power controls
- Existing Device Model management pages — new relay type column/field
- Existing power management / scheduling system — devices without confirmed relay should be flagged

### Risks

**MEDIUM RISK: IO Port Restore Failure**
- **Description:** If Step 3 (restore IO to output mode) fails after detection, the device's IO port is stuck in input mode and power relay commands won't work
- **Impact:** High — device becomes unable to power cycle until manually fixed
- **Probability:** Low — the restore command is a simple API call that should succeed if the device is reachable
- **Mitigation:** Implement retry logic (up to 3 attempts). If still failing, display a prominent warning to the user. Log the failure for support investigation.

**LOW RISK: Existing Devices Without Detection History**
- **Description:** All existing devices with `is_power_relay_installed = true` (set manually) have no detection timestamp. After this change, these devices would still show power controls based on the old manual flag, but with no "Last checked" timestamp.
- **Impact:** Low — the flag value is likely correct for most devices (someone manually set it), and users can re-verify with "Check Relay" at any time
- **Probability:** High — all existing devices will be in this state initially
- **Mitigation:** Consider a communication to APW suggesting they run "Check Relay" on devices during normal operations to build up verified state. No forced migration needed.

**LOW RISK: Config Group Deprecation**
- **Description:** Removing the relay capability checkbox from Config Group pages changes an admin workflow
- **Impact:** Low — this is an internal admin function, not customer-facing, and the new location (Device Model pages) is more intuitive
- **Probability:** Low — the change is straightforward
- **Mitigation:** Add a note on the Config Group page directing admins to the Device Model page. Leave read-only display during transition.

---

## Open Questions

| ID | Question | Decision Owner | Status |
|----|----------|---------------|--------|
| Q-1 | What is the specific API command to check whether a router has a physical IO port? | Devon D'Andrea (APW) | **Open** — Devon to provide |
| Q-2 | Should there be a "Check All Relays" bulk action on the company/fleet level, or is per-device detection sufficient for Phase 1? | Aksana / Devon | **Open** — leaning per-device only for Phase 1 |
| Q-3 | For devices with `is_power_relay_installed = true` from the old manual flag, should we auto-show power controls (grandfather them in) or require a fresh "Check Relay" verification? | Aksana / Devon | **Open** — leaning grandfather to avoid disruption |
| Q-4 | Does the detection sequence work identically across all single-outlet models (I-22, I-4500, Systex), or are there model-specific differences? | Adam Curcie (APW) | **Open** — needs testing |
| Q-5 | Should we display a relay detection history/log on the device page, or just the last check result and timestamp? | Aksana | **Open** — leaning last check only for simplicity |

---

## Relationship to Phase 2 (Multi-Outlet)

This Phase 1 work establishes three foundational patterns that Phase 2 (Multi-Outlet Power Relay) builds upon:

| Phase 1 Foundation | Phase 2 Extension |
|--------------------|-------------------|
| `relay_type` enum on model (None / Single / Multi) | Phase 2 activates the `multi` value — I-52 devices get the four-outlet activation toggle and power management module |
| Relay detection via IO input mode (1 port) | Phase 2 extends detection to 4 IO ports on I-52 devices |
| Power controls gated behind `is_power_relay_installed` | Phase 2 replaces the single button with the four-outlet module when relay is activated on I-52 |
| Config Group → Model migration for relay capability | Phase 2 depends on this migration being complete |

**The existing Phase 2 PRD ([PRD-multi-outlet-power-relay.md](PRD-multi-outlet-power-relay.md)) should be updated to:**
- Reference this Phase 1 PRD as a hard dependency
- Note that FR-1 (model-level capability migration) is now handled in Phase 1
- Note that the "dumb accessory with no electronic handshake" statement in FR-2.1 is no longer fully accurate — Phase 1 introduces hardware detection that Phase 2 should leverage for multi-outlet relay presence verification
- Update the risk section to reflect that "No Hardware Detection" (currently listed as MEDIUM RISK) is mitigated by Phase 1

---

## References

### Source Materials
- **[WATM-2163](https://orases.atlassian.net/browse/WATM-2163)** — Original Jira ticket from Devon D'Andrea describing the IO detection discovery and 3-step API sequence
- **[June 2, 2026 Sponsor Update Call](../../Meetings/Summaries/2026-06-02_APW_Project_Sponsor_Update_Summary.md)** — Meeting where Devon and Adam presented the discovery and discussed prioritization
- **[June 2, 2026 Call Transcript](../../Meetings/Transcripts/2026-06-02_APW_Project_Sponsor_Update_Transcript.md)** — Full transcript, signal strength and relay detection discussion at [00:19:36]-[00:23:31]
- **[PRD — Multi-Outlet Power Relay (Phase 2)](PRD-multi-outlet-power-relay.md)** — Phase 2 PRD that builds on this foundation
- **[Multi-Outlet UI Mockup Ideas](UI-Mockup-Ideas.md)** — Includes Part 3 (Model Management) and Part 1B (Single-Outlet Enhancement) mockups relevant to this Phase 1

### Key Code Locations (Current State)

| File | What's There Today |
|------|-------------------|
| `plugins/Devices/src/Util/RouterApi.php` | Existing power cycler methods (powerCyclerOn, powerCyclerOff, getPowerCyclerStatus, restartPowerCycler) — new detection methods go here |
| `plugins/Devices/src/Model/Entity/DeviceModel.php` | `is_power_cycler_capable` property — to be replaced by `relay_type` |
| `plugins/Devices/src/Model/Entity/Device.php` | `is_power_relay_installed` property — to become system-managed |
| `plugins/Devices/templates/Admin/Devices/view.php` (line ~184) | Manual "Power Relay Installed?" display — to be replaced with detection UI |
| `plugins/Devices/templates/Admin/Devices/edit.php` (lines 169, 390) | Manual checkbox — to be removed |
| `plugins/SystemManagement/templates/Admin/ConfigGroups/edit.php` (line 48) | Config Group relay capability checkbox — to be deprecated |
| `config/Migrations/20250626192136_AddIsPowerCyclerCapableToDeviceModels.php` | Migration that added `is_power_cycler_capable` to device_models |
| `config/Migrations/20260115140000_AddPowerRelayInstalledToDevices.php` | Migration that added `is_power_relay_installed` to devices |

---

## Notes

### Evidence Sources
- **WATM-2163 Jira ticket** — Devon D'Andrea's original description of the 3-step IO detection discovery, including curl command examples
- **June 2, 2026 Sponsor Update call transcript** — Devon and Adam demonstrated detection live on an I-22 and I-4100 during the call. Key quotes on relay pain point and detection discovery at [00:19:36]-[00:23:31]
- **June 2, 2026 Sponsor Update summary** — Action items and key decisions related to WATM-2163 prioritization
- **Codebase analysis** — Current state of `is_power_cycler_capable`, `is_power_relay_installed`, RouterApi power cycler methods, device view/edit templates, and Config Group templates reviewed to document existing behavior

### Key Decisions

| Decision | Rationale | Source |
|----------|-----------|--------|
| Replace boolean `is_power_cycler_capable` with `relay_type` enum | Enum supports future multi-outlet value; boolean cannot distinguish single vs. multi | Analysis of Phase 2 requirements + May 11 meeting (Devon: "100%" on model-level capability) |
| Detection is user-initiated (button), not automatic on check-in | Direct API calls to router have performance and reliability implications; should not run on every check-in | Engineering analysis |
| Remove manual `is_power_relay_installed` checkbox | With automated detection, manual editing becomes a source of data inconsistency | Logical consequence of FR-2 |
| Gate power controls behind detection | Prevents silent failures that have wasted support time and eroded customer trust | Problem 2 analysis, Devon's trade show feedback |
| Grandfather existing `is_power_relay_installed = true` devices | Avoid disrupting existing workflows; users can re-verify with "Check Relay" at any time | Leaning decision, pending final confirmation (Q-3) |

### Confidence Levels
- FR-1 (Model-Level Capability): **High confidence** — `is_power_cycler_capable` already exists on `device_models`; migration to enum is straightforward
- FR-2 (Automated Detection): **High confidence** — Devon demonstrated the 3-step detection sequence live during June 2 call; API commands are documented in WATM-2163
- FR-3 (Gated Power Controls): **High confidence** — Logical consequence of FR-2; existing conditional UI patterns in the portal provide implementation precedent
- IO port pre-check command: **Low confidence** — Devon mentioned this is needed but has not yet provided the specific API call (Q-1)
- Cross-model detection consistency: **Medium confidence** — Tested on I-22 and I-4100 during the call; I-4500 and Systex need verification (Q-4)

### Q&A Document
- [PRD-single-outlet-power-relay-enhancements-QA.md](PRD-single-outlet-power-relay-enhancements-QA.md) — Not yet created; open questions are tracked in the Open Questions section above

---

END OF DOCUMENT
