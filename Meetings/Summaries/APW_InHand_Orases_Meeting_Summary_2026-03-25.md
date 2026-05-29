## APW x InHand x Orases — Meeting Summary
**Date:** March 25, 2026

### Attendees
- **Allpoint Wireless (APW):** Devon D'Andrea, Adam Curcie, Rick (President/CEO)
- **InHand Networks:** Kenneth Hunter (VP Sales, Americas), Ziming Wang (Head Engineer, US)
- **Orases:** Aksana Rahouski (Sr. Product Manager), Laura Perry (Project Manager), Richard Sacco (Developer)

### Purpose
First joint meeting between all three partner companies to align on upcoming hardware + software initiatives. Was originally planned as an in-person meeting but moved to virtual due to weather.

---

### Topic 1: Anti-Theft / Security Device (VT210 Tracker)

**What it is:** InHand's Vehicle Tracker 210 (VT210) — a small, phone-sized IoT device with 4G cellular, GPS, battery backup, and multiple digital I/O ports. It connects to door sensors, vibration sensors, a siren, and a relay (to cut power). It communicates via MQTT.

**Hardware status:**
- The VT210 is an existing, market-proven product — not new hardware
- InHand has completed proof-of-concept testing with sensors (door, vibration, siren, relay)
- **Outstanding item:** Local logic (device-side automation when cellular is lost) — Ziming committed to delivering this within ~2 weeks
- MQTT interface to cloud is ready for testing — sends JSON payloads via MQTT broker

**Business model discussion:**
- Rick emphasized this is **not a standalone revenue product** — it's an add-on to drive more connections (their core business)
- Market research/surveys showed price sensitivity is the main barrier. Retail ATM operators do the math: $10/month x 500 ATMs = $60K/year, but losses may only be $5-6K/year. The sweet spot is $2-3/month with hardware under ~$150 upfront
- The product should be **a la carte** with tiers:
  - **Base (Deterrent):** Door sensor + siren + notifications after hours. Configurable business hours (armed/disarmed by schedule, not manually)
  - **Enhanced (future):** Add monitoring, additional sensors, relay/anti-jackpotting
- **Key insight from Rick:** Cash loaders come at unpredictable times — cannot rely on manual arm/disarm. Must be timer/schedule-based
- Ken highlighted the **upsell opportunity** — thousands of existing APW customers could add this as a second device + second data plan alongside their existing router

**Software/portal implications discussed:**
- Device will live in the portal similar to existing router devices
- Needs its own MQTT broker (likely self-hosted on AWS for reliability at scale). Ziming confirmed building a lightweight broker is straightforward — open source options exist, and InHand has built brokers before
- Two-way communication needed: portal must be able to send commands (e.g., silence siren, push configs)
- Alerts/notifications are critical — real-time notification when events trigger
- Data usage is negligible (<10MB/month), so carrier cost is not a factor

**Agreed next steps:**
- APW (Devon/Adam) needs to define the **MVP business vision** first — what the default product looks like, how it's sold, default configs, provisioning flow
- Orases recommended starting with the 80/20 rule — build for the 80% use case first
- No development timeline set yet — Devon estimated fall at earliest given other priorities
- InHand to deliver local logic firmware update in ~2 weeks

---

### Topic 2: Multi-Outlet Power Relay (IR315 / I52)

**What it is:** A new product opportunity using InHand's IR315 router (called I52 in APW's portal), which has 4 independent digital I/O ports. This allows independently power cycling up to 4 connected devices from a single router — versus the current IR302 (I22) which only has 1 outlet.

**Use case:** Primarily for kiosk and gaming customers who have 4 terminals at one location. Instead of buying 4 separate connections, they get 1 router that can independently restart any of the 4 machines.

**Business rationale (Rick):** Competitive differentiator — any competitor can sell 4 connections, but APW can offer 1 connection managing 4 devices. The net connections won from new customers outweighs selling fewer connections per site.

**Status:**
- Adam has tested a prototype — it works as expected
- Hardware harness from InHand quoted at $11/unit, 100 minimum order, 3-week lead time
- Manufacturer confirmed they can fulfill up to 20,000/year
- Adam is working with the outlet manufacturer on making connectors modular

**Portal implications:**
- Current UI has a single "Restart Power Cycler" button — needs redesign for 4 independent outlets
- Need to determine: auto-detect vs. enable per device, naming/labeling of outlets, button layout
- Devon noted this will **jump ahead of most backlog items** in priority
- Richard requested the prototype hardware be sent to Orases once available so dev can begin

**Agreed next steps:**
- Adam to follow up with manufacturer on modular connectors
- Adam to order and send prototype to Orases (available off Amazon in ~2 days)
- APW to define UI/UX requirements for multi-outlet management
- Orases to begin scoping the portal changes once requirements and hardware are in hand

---

### Topic 3: Camera/Video Integration (Tabled)

- Rick and Ken briefly raised the idea of integrating low-cost cameras (e.g., Blink) for visual monitoring
- Adam noted the challenges: power (PoE), proximity, storage, and complexity of setup
- Ken mentioned InHand has partners using low-cost cameras with edge logic for retail monitoring
- **Decision: Tabled for the next quarterly meeting.** Camera is a separate conversation from theft deterrent

---

### Topic 4: 5G Business Internet API Integration

- Brief discussion after InHand/Rick dropped off
- Noah (Orases dev) was blocked on 5G account API testing due to a missing character in credentials — resolved during the call
- APW (Adam) to provide a SIM card for testing ASAP
- Noah is actively working on this

---

### Standing Action Items

| Item | Owner | Status |
|------|-------|--------|
| Define MVP business vision for security device (default config, pricing tiers, provisioning) | Devon / Adam | Not started |
| Deliver local logic firmware for VT210 | InHand (Ziming) | ~2 weeks |
| Send VT210 MQTT documentation to Orases | InHand (Ziming) | Ready to share |
| Multi-outlet prototype to Orases | Adam | Pending (ordering from Amazon) |
| Define multi-outlet UI/UX requirements | Devon / Adam | Not started |
| Provide 5G SIM card for testing | Adam / Dan | Imminent |
| Schedule quarterly joint meetings (APW + InHand + Orases) | All | Agreed — next meeting suggested ~June 25 |
