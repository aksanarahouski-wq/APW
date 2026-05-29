# Multi-Outlet Power Relay (IR315 / I52)

**Status:** Discovery / Pre-Development
**Date Created:** March 26, 2026
**Source:** APW x InHand x Orases Joint Meeting — March 25, 2026

---

## Overview

Allpoint Wireless wants to support independent power cycling of up to 4 connected devices from a single InHand IR315 router (referred to as the "I52" in the APW portal). This is an upgrade from the current IR302 (I22) router, which only supports a single power relay outlet. The feature is primarily targeting kiosk and gaming customers who have multiple terminals at one location and currently need separate connections for each one.

Devon noted during the meeting that this feature will **jump ahead of most backlog items** in priority.

---

## What We Know About the Hardware

### Current State: IR302 / I22 (Single Outlet)

- The existing IR302 router (called I22 in the portal) has **1 digital I/O** that connects to a power relay
- The portal has a single "Restart Power Cycler" button that triggers the relay — turns the outlet off, waits 10 seconds, turns it back on
- A power relay schedule feature was also built for a specific customer (though that customer reportedly does not use it)

### New Hardware: IR315 / I52 (Four Outlets)

- The IR315 router (called I52 in the portal) has **4 independent digital I/O ports** plus a common neutral
- Each I/O port can independently control a separate power relay outlet
- The I/O commands are similar to what the IR302 uses today, but inverted (current: low-to-high/high-to-low; IR315: flipped)
- **No web GUI changes on the router itself** — this is about how the portal sends I/O commands and presents the controls to the user

### Third-Party Power Relay Hardware

- Adam sourced a multi-outlet power relay product from a third-party manufacturer (not InHand)
- It supports independently switching up to 4 (or more) outlets
- Adam has tested a prototype — **it works as expected**
- The manufacturer confirmed they can fulfill up to **20,000 units/year**
- Adam is in communication with the manufacturer about making the outlet connectors **modular** (plug-in/plug-out rather than fixed)

### Wiring Harness

- InHand (Ziming) quoted the custom wiring harness at **$11/unit**
- **Minimum order:** 100 units
- **Lead time:** 3 weeks for manufacturing + a few days for air shipping
- Different wiring harness configurations may be needed depending on how many outlets a customer wants connected (1, 2, 3, or 4)
- Ziming suggested color-coded connectors to make installation foolproof

### Prototype Availability

- Adam is ordering a prototype off Amazon to send to Orases (~2 days delivery)
- The show prototype is in transit from Las Vegas back to APW

---

## Business Context

### Why This Feature

- **Competitive differentiator:** Any competitor can sell 4 separate internet connections for 4 terminals. APW can offer 1 connection that manages all 4 — something competitors don't offer
- **Customer demand:** Every person at a recent trade show picked up the multi-outlet relay and asked about it. Strong organic interest
- **Revenue math:** APW may sell fewer connections per site, but the unique capability wins entire new customers — net more connections overall
- **Rick (CEO):** "We're trying to set ourselves apart to get more customers"

### Target Use Cases

**Primary: Gaming terminals**
- Gaming operators have 4+ terminals lined up at a single location
- Terminals frequently need hard resets/restarts remotely
- Currently would need 4 separate APW routers with 4 monthly fees to individually power cycle each terminal
- With the IR315 + multi-outlet relay: 1 router, 1 monthly fee, independent control of all 4

**Secondary: Kiosks**
- Similar multi-terminal setups at retail locations
- Bitcoin ATMs, lottery kiosks, vending machines with connectivity

**Tertiary: ATMs (future)**
- Locations with multiple ATMs could benefit from consolidated power management

### Cross-Sell with Security Device

Devon noted there is crossover with the anti-theft VT210 tracker — gaming and kiosk locations are strong candidates for both the multi-outlet relay and the security tracker. These could be sold together.

---

## Portal Changes Required (Discovery Needed)

### Current Portal Behavior (IR302 / I22)
- Single "Restart Power Cycler" button on the device detail page
- Sends one I/O command to the router
- Router triggers the single relay → outlet turns off → waits → turns back on
- Power relay schedule feature exists (built for one customer) — allows scheduling on/off times

### What Needs to Change for IR315 / I52

The following questions need to be answered during discovery with the client:

#### Device Detection and Enablement
- [ ] Does the portal auto-detect that an I52 has multi-outlet capability, or does it need to be manually enabled?
- [ ] Is multi-outlet available on all I52 devices, or only when a customer purchases the relay hardware?
- [ ] How do we know how many outlets are actually connected (1, 2, 3, or 4)?

#### UI/UX — Device Detail Page
- [ ] What does the button layout look like for 4 independent outlets?
- [ ] Can customers name/label each outlet? (e.g., "Terminal 1", "Lottery Kiosk", "Bitcoin ATM")
- [ ] Do we show status per outlet (on/off/unknown)?
- [ ] Is there a "restart all" option in addition to individual controls?
- [ ] How does this coexist with the existing single-outlet "Restart Power Cycler" button for I22 devices?

#### I/O Commands
- [ ] Richard needs the exact I/O command structure for the IR315's 4 DIOs
- [ ] Commands are reportedly similar to current I/O commands but with inverted logic (need to confirm with InHand/Adam)
- [ ] Each outlet maps to a specific DIO port — need the mapping documented
- [ ] What is the command sequence? (e.g., set DIO2 low → wait 10 seconds → set DIO2 high)

#### Power Relay Schedule
- [ ] Does the existing power relay schedule feature need to be extended to support per-outlet schedules?
- [ ] Or is scheduling not needed for the initial release?

#### Configuration Management
- [ ] Are there any router-side configurations that need to be pushed to enable multi-outlet mode?
- [ ] Does the I52 need specific firmware to support the 4-outlet relay?

#### Customer Permissions
- [ ] Can company admins control all 4 outlets, or should there be per-outlet permissions?
- [ ] Are there any safety considerations (e.g., confirmation dialog before cutting power to an outlet)?

#### Billing
- [ ] Is multi-outlet capability included in the standard I52 service plan?
- [ ] Or is it an add-on feature with additional monthly cost?
- [ ] Does having 4 devices on one connection change how usage/billing is calculated?

---

## What Orases Needs to Begin Development

| Item | Source | Status |
|------|--------|--------|
| Prototype hardware (multi-outlet relay + IR315) | Adam (APW) | Adam ordering from Amazon, ~2 days |
| I/O command documentation for IR315 4-DIO | Adam / InHand (Ziming) | Not yet provided |
| Wiring diagram (which DIO maps to which outlet) | InHand (Ziming) | Not yet provided |
| UI/UX requirements (button layout, naming, status display) | APW (Devon/Adam) | Not yet defined |
| Decision on scheduling (per-outlet schedules or not for V1) | APW (Devon/Adam) | Not yet decided |
| Decision on billing model | APW (Devon/Adam) | Not yet decided |
| Confirmation of I/O command logic (inverted from IR302) | Adam | Mentioned in meeting, needs formal documentation |

---

## Next Steps

### Immediate
1. **Adam to send prototype hardware to Orases** — Multi-outlet relay + IR315 so Richard can begin testing I/O commands
2. **Adam/InHand to provide I/O command documentation** — Exact DIO mapping, command structure, and any differences from the IR302

### Discovery (APW + Orases)
3. **Define the UI/UX for multi-outlet management** — What does the device page look like? How do customers interact with 4 outlets? Naming, status indicators, button layout
4. **Decide on scope for V1** — Is it just the buttons (restart individual outlets), or does it include scheduling, naming, status, etc.?
5. **Clarify billing** — Is multi-outlet bundled or add-on?

### Development (After Discovery)
6. **Portal changes** — Update device detail page for I52 devices, implement per-outlet I/O commands, extend or replace power cycler UI
7. **Testing** — Requires physical hardware to validate I/O commands end-to-end
8. **Timeline:** Devon indicated this is a high-priority item that will jump most of the backlog. Portal work can begin as soon as discovery is complete and hardware is in hand. Adam suggested it should not be a huge project from a backend perspective since the I/O interaction is similar to what exists today — the bulk of the work is UI redesign.

---

## Dependencies and Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Prototype not yet in Orases' hands | Cannot test or validate I/O commands | Adam ordering from Amazon — expected within days |
| I/O command logic is inverted from IR302 | Could cause issues if not properly documented | Need formal documentation from Adam/InHand before dev starts |
| Wiring harness configurations vary by customer (1-4 outlets) | Portal needs to handle variable number of active outlets | Design UI to be flexible — show only connected/enabled outlets |
| Modular connectors not yet confirmed by manufacturer | Could affect hardware packaging and installation experience | Adam is in active communication with manufacturer |
| Existing power relay schedule feature may need rework | Scope creep if we need per-outlet scheduling | Clarify with APW if scheduling is V1 or V2 |
| Competing priorities (security device, CakePHP upgrade, configs V2, 5G API) | Could delay start despite high priority designation | Devon indicated this jumps most backlog items — need formal prioritization confirmation |
