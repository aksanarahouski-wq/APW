# Anti-Theft / Security Device (VT210 Tracker)

**Status:** Discovery / Pre-Development
**Date Created:** March 26, 2026
**Source:** APW x InHand x Orases Joint Meeting — March 25, 2026

---

## Overview

Allpoint Wireless is introducing a theft-deterrent product for ATMs and kiosks using InHand's Vehicle Tracker 210 (VT210). This device will be sold as an add-on to existing APW connectivity services, positioned as a low-cost security layer that deters theft through sensors, alarms, and notifications. The product requires portal integration for device management, event monitoring, alerting, and two-way communication.

---

## What We Know About the Device

### Hardware: InHand VT210

- **Form factor:** Small, phone-sized IoT device (MCU-level, not a Linux router)
- **Connectivity:** Standalone 4G cellular module (does not use the existing APW router for connectivity — requires its own SIM card and data plan)
- **GPS:** Built-in GPS for location tracking
- **Battery:** Internal battery backup — continues to send alerts even if external power is cut
- **I/O Ports:** Multiple digital input/output and analog input/output ports for connecting sensors and actuators
- **No web GUI:** Configured via command line (telnet with static IP) or over-the-air (OTA) for firmware and config pushes
- **Communication protocol:** MQTT only — all data to/from the device goes through an MQTT broker
- **Data usage:** Negligible — estimated under 10MB/month (event-based messaging, not continuous streaming)
- **Product maturity:** The VT210 is an existing, market-proven product that InHand has sold for years. The custom work is adding local logic specific to APW's security use case

### Supported Sensors and Actuators

| Component | Type | Purpose |
|-----------|------|---------|
| Door sensor | Digital input | Detects door open/close events |
| Vibration sensor | Digital input | Detects physical tampering or forced entry attempts |
| Siren | Digital output | Audible alarm triggered by events |
| Relay | Digital output | Cuts power to the ATM (anti-jackpotting) |

### How It Communicates

1. **Device → Cloud:** The VT210 publishes MQTT messages (JSON payloads) to a broker whenever an I/O event occurs (e.g., door opened, vibration detected). Messages include the device serial number and I/O state changes
2. **Cloud → Device:** Commands can be sent back via MQTT to the device (e.g., silence siren, update configuration, change armed/disarmed schedule)
3. **Health/keep-alive:** The device sends periodic keep-alive messages to the MQTT broker to confirm it is online
4. **No traditional check-ins:** Unlike existing APW routers, this device does not send UDP check-ins. All communication flows through MQTT

### Local Logic (In Progress)

InHand is developing local logic firmware so the device can act autonomously if cellular connectivity is lost:
- If someone cuts the antenna or cellular goes down, the device can still trigger the siren and relay locally based on sensor inputs
- This is critical — without it, a thief could simply jam or cut cellular before breaking in
- **ETA:** ~2 weeks from meeting date (target: April 8, 2026)

### MQTT Documentation

InHand has a test document showing:
- Broker configuration (currently using HiveMQ public broker for testing)
- Topic structure: subscribes by device serial number (e.g., `200/<serial_number>`)
- Message format: JSON with I/O state (e.g., `DI1` changing from 0 to 1 indicates a sensor trigger)
- Ziming to share this documentation with Orases

---

## Business Context

### Why This Product Exists

- APW is in the **connections business**, not the security business. This product is a means to win and retain more connectivity customers
- Market research and customer surveys confirmed demand but extreme price sensitivity among retail ATM operators
- Existing competitors (DPL, Bullhorn) are too expensive ($500+ upfront, $7-10/month) and/or difficult to install
- The goal is to undercut the market significantly to drive mass adoption

### Target Pricing

| Component | Target |
|-----------|--------|
| Hardware (tracker + sensors + siren, all-in) | Under ~$150 upfront |
| Monthly service | $2-3/month |
| Data plan cost to APW | Negligible (under 10MB/month) |

### Target Customer

- Retail ATM operators with large fleets (100-500+ ATMs)
- Kiosk operators
- Gaming terminal operators
- Primarily urban locations with higher theft risk
- Both existing APW customers (add-on) and new customers (acquisition tool)

### Product Tiers (A La Carte Vision)

**Tier 1 — Deterrent (MVP / Launch Product):**
- Door sensor + siren + notifications after business hours
- Schedule-based arming (configurable business hours — NOT manual arm/disarm)
- Event logging (door open/close history even during business hours)
- SMS/email notifications when events trigger after hours

**Tier 2 — Enhanced (Future):**
- Active monitoring (24/7 with human or automated response)
- Additional sensors (multiple doors, additional vibration points)
- Relay/anti-jackpotting (cut power on tamper detection)
- GPS tracking and map view in portal

### Key Business Rule: Schedule-Based Arming Only

Rick (APW CEO) was clear: **no manual arm/disarm via app or portal**. Reasons:
- Cash loaders arrive at unpredictable times — cannot expect someone to sit at a computer waiting to disarm
- If a new employee forgets to re-arm, the system is useless
- Business hours are set once per location, and the system arms/disarms automatically
- Example: Store closes at 8:00 PM → sensors go live at 8:15 PM. Store opens at 8:00 AM → sensors turn off at 7:00 AM

### Bundling Consideration

- Many customers will already have an APW router at the same location
- The VT210 is a **second device with a second data plan** — it does not connect to or depend on the existing router
- Potential for discounted pricing when bundled with an existing APW connection at the same location
- Portal needs to consider how to present two devices at one location (router + tracker) under the same customer account

---

## Portal Integration Requirements (What We Need to Define)

These items need to be fleshed out with the client before development can begin:

### Device Lifecycle
- [ ] How is the device provisioned/activated in the portal?
- [ ] Does it follow the same flow as existing InHand routers (add device, assign to company, activate)?
- [ ] What is the default configuration when a device is first added?
- [ ] How are firmware updates pushed to the device?

### MQTT Infrastructure
- [ ] Stand up an MQTT broker (self-hosted on AWS recommended for reliability and control at scale)
- [ ] For early testing, can use public broker (HiveMQ) that InHand already has configured
- [ ] Define topic structure and message schema
- [ ] Build subscriber service to consume messages from the broker and process into the portal

### Portal UI/UX
- [ ] Where does this device live in the portal? Same device list as routers or separate section?
- [ ] What does the device detail page look like? (status, sensor states, event history, GPS location)
- [ ] How does a customer configure business hours / armed schedule?
- [ ] How does an admin manage/view alert history?
- [ ] How are two devices at one location visually linked (router + tracker)?

### Alerts and Notifications
- [ ] What events trigger notifications? (door open after hours, vibration, power loss, device offline)
- [ ] Notification channels: SMS, email, push notification, portal alert?
- [ ] Who receives notifications? (device owner, company admin, custom contact list?)
- [ ] Configurable notification rules per device or per company?

### Two-Way Commands
- [ ] Silence siren remotely from portal
- [ ] Push configuration changes (business hours, sensor enable/disable)
- [ ] Any other remote commands needed?

### Billing
- [ ] Flat monthly rate per device
- [ ] Discounted rate when bundled with existing APW router at same location?
- [ ] How does this appear on invoices? Separate line item or bundled?

### Reporting
- [ ] Event history / audit log per device
- [ ] Dashboard showing fleet-wide security status
- [ ] Door open/close logs (valuable even during business hours for internal theft detection)

---

## What InHand Is Delivering

| Deliverable | Status | ETA |
|-------------|--------|-----|
| VT210 hardware with sensor support | Complete | Available now |
| Proof of concept (sensors + siren + relay) | Complete | Done |
| MQTT interface to cloud (JSON over MQTT) | Complete | Ready for testing |
| MQTT test documentation | Complete | Ziming to share with Orases |
| Local logic firmware (autonomous operation without cellular) | In progress | ~April 8, 2026 |
| Custom wiring harness design | Not started | Pending final sensor package definition |
| Color-coded connectors for idiot-proof installation | Discussed | Pending final hardware config |

---

## Next Steps

### Immediate (APW Ownership)
1. **Define the MVP product vision** — Devon and Adam to document: what the default device package includes, how it's sold, default configuration, and the customer journey from purchase to activation
2. **Share MQTT documentation** — Ziming to send the test broker setup and message schema to Orases (Richard)

### Near-Term (Joint APW + Orases)
3. **Discovery session** — Sit down together and define the answers to all the portal integration requirements listed above
4. **Validate hardware capabilities** — Confirm with InHand that the MVP sensor package supports the defined use cases
5. **MQTT broker evaluation** — Richard to research MQTT broker options (self-hosted open source vs. managed) and recommend an approach

### Development (Not Yet Scheduled)
6. **Portal development** — Build device management, MQTT integration, alerting, and billing for the VT210
7. **Timeline:** Devon estimated fall 2026 at earliest, pending prioritization against other backlog items (CakePHP upgrade, Configs V2, multi-outlet power relay, 5G API integration)

---

## Dependencies and Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| MVP product vision not yet defined by APW | Blocks all portal development | Schedule dedicated session with Devon/Adam to define |
| MQTT is a new protocol for the portal (currently uses UDP check-ins + SQS) | Requires new infrastructure and integration pattern | Start with research and small POC using public broker |
| Local logic firmware not yet delivered | Cannot fully test autonomous operation | InHand targeting ~2 weeks; not a blocker for portal work |
| Price point ($2-3/month) may not cover development costs | Business viability risk | APW views this as a customer acquisition tool, not a profit center |
| Schedule-based arming only (no manual override) | Could frustrate some customers | APW made a deliberate business decision based on operational reality |
| Competing priorities on Orases roadmap | Could delay start of development | Need prioritization alignment with client at next monthly meeting |
