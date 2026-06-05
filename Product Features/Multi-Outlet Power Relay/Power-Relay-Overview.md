# Power Relay Feature — Problem Statement & Roadmap

**Document Version:** 1.0
**Date:** 2026-06-05
**Author:** Aksana Rahouski (Orases)
**Status:** Draft
**Document Owner:** Aksana Rahouski
**Last Updated:** 2026-06-05

---

## Problem Statement

APW sells power relay accessories that connect to InHand routers, allowing customers to remotely restart equipment (ATMs, gaming terminals, kiosks) without a physical site visit. This is one of APW's core differentiators — competitors require a separate internet connection per device, while APW can manage equipment power from the same router that provides connectivity.

**The problem: APW cannot tell whether a power relay is physically connected to a device.**

Field technicians install routers and relays at customer sites. They frequently do not document whether a relay was connected. The WATM portal has a manual checkbox ("Power Relay Installed?") that someone is supposed to set, but it comes with its own tooltip warning: *"Power Relay Installed is a manual flag for tracking purposes. We do not validate that a power relay is actually connected to the device."*

The portal explicitly tells users not to trust its own data.

### Real-World Impact

**Silent power cycle failures destroy customer confidence.**

A support agent sees a customer's ATM is frozen. They open the WATM portal, find the device, and click "Restart Power Cycler." The command goes to the router. The router toggles an empty IO port. Nothing happens — no power cycle, no error, no feedback. The agent thinks the system is broken. The customer thinks APW's service doesn't work. In reality, there's simply no relay plugged in.

This scenario plays out regularly because:
- Field techs don't take notes during installs (Devon D'Andrea: *"People would reach out to us all the time 'cause they don't take notes when they go do installs."*)
- The manual checkbox is wrong more often than it's right
- There is no way to verify relay presence without physically visiting the site

**Trade shows expose the gap.**

APW attends trade shows where the multi-outlet power relay is a flagship product. Prospective customers pick up the relay hardware and ask: "Can you detect if this is connected?" For six years, the answer has been "no." This undermines the sales pitch for a product that is otherwise a genuine competitive advantage.

**The unreliable flag creates operational waste.**

- Support agents waste time troubleshooting "failed" power cycles that were never going to work
- Operations staff cannot audit the fleet to know which devices actually have relays installed
- Power schedules get assigned to devices without relays, executing commands into the void
- New device types (i52 with 4-outlet relay) will make the problem worse — customers could toggle "I have a 4-outlet relay" when they don't, then wonder why none of the 4 outlet controls work

### The Discovery That Changes Everything

During the June 2, 2026 Sponsor Update call, Devon D'Andrea and Adam Curcie demonstrated a discovery: **InHand routers can detect whether a relay is physically connected.**

The technique is simple — temporarily switch the router's IO port from output mode to input mode, read the circuit level, then switch back:
- `io_level = low` → relay IS physically connected (closed circuit)
- `io_level = high` → relay is NOT connected (open circuit)

This takes approximately 2 seconds and provides a definitive, hardware-verified answer. The capability was always present in the router firmware — no one had ever utilized it.

This discovery enables the portal to:
1. **Verify** relay presence with a single button click
2. **Auto-update** the `is_power_relay_installed` flag based on hardware truth
3. **Gate** power control buttons — only show them when a relay is confirmed
4. **Extend** the same detection to all 4 ports on i52 devices for the multi-outlet relay

---

## Business Value

### For APW Sales & Marketing
- **Closes the trade show gap**: "Can you detect the relay?" becomes "Yes — one click, instant verification"
- **Strengthens competitive positioning**: No competitor offers hardware-verified relay detection with automated power control gating
- **Enables confident multi-outlet sales**: The 4-outlet relay (i52) becomes a reliable product, not a "trust us, it works" pitch

### For APW Operations
- **Eliminates silent failures**: Power buttons only appear when a relay is confirmed — no more commands into empty IO ports
- **Enables fleet auditing**: Operations can verify relay presence across devices during normal workflows
- **Reduces support burden**: Agents can verify relay status before attempting power cycles, eliminating wasted troubleshooting time

### For APW Customers
- **Power cycles that actually work**: When the button is there, it means the relay is there
- **Self-service verification**: Customers can check their own relay status without calling support
- **Reliable scheduling**: Power schedules only execute on devices with confirmed relays

---

## Solution: Two-Phase Approach

The work is split into two phases because the discovery of automated detection changes the architecture. Before building multi-outlet controls (Phase 2), we first need to fix foundational problems with how the portal tracks and verifies relay hardware on existing single-outlet devices (Phase 1).

### Phase 1 — Single-Outlet Power Relay Enhancements

**Scope:** I-22, I-4500, Systex (existing single-outlet models)
**Source:** [WATM-2163](https://orases.atlassian.net/browse/WATM-2163) — Devon's IO detection discovery

Fixes the foundation:

| Layer | What It Does | Why It Matters |
|-------|-------------|----------------|
| **Model-Level Capability** | Migrates relay capability from Config Groups to Device Models with a `relay_type` enum (none / single / multi) | Data model accuracy — relay capability is determined by hardware model, not configuration group |
| **Automated Detection** | Adds a "Check Relay" button that runs a 3-step IO detection sequence to verify relay presence (1 port) | Eliminates guesswork — confirm relay in seconds instead of trusting a manual checkbox |
| **Gated Power Controls** | Hides power buttons until relay presence is hardware-verified | Prevents silent failures — no more commands to devices without relays |

**Full PRD:** [PRD-single-outlet-power-relay-enhancements.md](PRD-single-outlet-power-relay-enhancements.md)

### Phase 2 — Multi-Outlet Power Relay (i52)

**Scope:** I-52 (four-outlet relay accessory)
**Dependency:** Phase 1 must be complete before Phase 2 development begins

Adds multi-outlet power management:

- Per-outlet on/off/restart controls with labeled outlets
- Per-outlet power scheduling (e.g., games off at night, ATM always on)
- Outlet status from check-in DIO data
- Relay activation toggle with hardware detection validation (extending Phase 1 pattern to 4 ports)

**Business Impact:** Competitive differentiator — one i52 router managing up to 4 terminals with independent power control. No competitor offers this. APW has 200 relay units in inventory.

**Full PRD:** [PRD-multi-outlet-power-relay.md](PRD-multi-outlet-power-relay.md)

### How Phase 1 Feeds Phase 2

| Phase 1 Foundation | Phase 2 Extension |
|--------------------|-------------------|
| `relay_type` enum on model (None / Single / Multi) | Activates the `multi` value — I-52 devices get four-outlet toggle and power management module |
| Relay detection via IO input mode (1 port) | Extends detection to 4 IO ports on I-52 devices |
| Power controls gated behind `is_power_relay_installed` | Replaces single button with four-outlet module when relay activated on I-52 |
| Config Group → Model migration for relay capability | Phase 2 depends on this migration being complete |

### Execution Order

Phase 1 → Phase 2

Phase 1 is a hard prerequisite for Phase 2. The model-level data model, detection API methods, and gating patterns established in Phase 1 are directly reused and extended in Phase 2. Developing Phase 2 without Phase 1 would require duplicating foundational work and would leave single-outlet devices with the existing unreliable manual tracking.

---

## References

- [Phase 1 PRD — Single-Outlet Power Relay Enhancements](PRD-single-outlet-power-relay-enhancements.md)
- [Phase 2 PRD — Multi-Outlet Power Relay (i52)](PRD-multi-outlet-power-relay.md)
- [WATM-2163 — New Digital IO Feature](https://orases.atlassian.net/browse/WATM-2163)
- [June 2, 2026 Sponsor Update Summary](../../Meetings/Summaries/2026-06-02_APW_Project_Sponsor_Update_Summary.md)
- [Multi-Outlet Power Relay Discovery Document](Multi_Outlet_Power_Relay_IR315.md)
- [UI Prototype — Option B Table](prototypes/option-b-table.html)

---

END OF DOCUMENT
