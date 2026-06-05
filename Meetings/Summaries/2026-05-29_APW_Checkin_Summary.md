# APW Check-in — Meeting Summary

**Date:** May 29, 2026, 1:30 PM EDT (5:30 PM UTC)
**Duration:** ~65 minutes
**Organizer:** Laura Perry
**Attendees:** Laura Perry, Aksana Rahouski, Richard Sacco, Aaron Diefes, Stone Marballie (Orases); Devon D'Andrea, Adam Curcie, Jon (APW)
**Recording:** https://tldv.io/app/meetings/6a19cd10c4457d001305d4ca

---

## Overview

Biweekly client check-in covering email template changes that went live unexpectedly, signal strength reporting on older 4100 devices, infrastructure upgrade scheduling (MySQL + server), portal rebranding timeline and requirements, recent deployments, check-in timestamp bug, device export billing anomalies, customer losses, a Verizon SIM status discrepancy, SIM terminology rename ticket, and a multi-outlet power relay feature review with prototype walkthrough.

---

## Key Discussion Topics

### Email Template Changes (Unexpected)
- A white-labeled email template went live Tuesday midday without a documented ticket or team awareness
- The Meeli sub-company email now has full branding (logo, footer, custom URL) vs. the previous plain-text style
- CakePHP 4.6 minor upgrade pushed to production last week may have coincided with the change, but team confirmed the upgrade alone wouldn't cause template changes
- Team agreed to investigate commit history to determine who made the change and whether it was billed
- Devon: "I wanna know what we're paying for too"

### Device Signal Strength — 4100 Firmware Issue
- Older 4100 devices with newer firmware report empty/incorrect signal strength values
- InHand's ASU (Arbitrary Strength Unit) value is available as a workaround and appears to report correctly
- **Action:** Richard to create a ticket for using ASU value instead of the current signal strength method when processing check-ins for affected devices

### Infrastructure Upgrades — MySQL & Server
- **CakePHP 4.6 minor upgrade:** Launched Tuesday, complete and live in production
- **MySQL upgrade:** Due July 1st; Richard working on it. **Scheduled for Tuesday (June 3rd)** with expected downtime of 30 min to 2 hours. Offline page will be displayed. Adam requested earliest possible start time (6-7 AM preferred)
- **Server upgrade (PHP 8.2):** Set up and ready. **Scheduled for Thursday (June 5th)** with no expected downtime. Separated from MySQL by a day to isolate any issues
- APW will notify customers Monday about Tuesday's planned downtime

### Portal Rebranding
- APW is under a 60-day rebranding deadline (started ~2 weeks ago) due to a legal/trademark dispute
- Changes are cosmetic only: logos and business name. Colors, styles, and guidelines remain the same
- **apcommand.com** domain will remain for device check-ins; no need to reconfigure devices
- Customer-facing portal will redirect from old URL to new domain (permanent redirect)
- Key unknowns:
  - Whether the other party will accept a 180-day extension (requested by APW's legal)
  - Whether rebranding fully resolves legal exposure (could still be sued for damages)
  - **New company name not finalized** — expected by next week
- White-labeled sub-company URLs may also need updates (TBD)

### Recent Deployments & Feature Updates
- Verizon second account work went live
- T-Mobile testing ongoing
- Invoice template redesign deployed
- Nacha-to-invoice drift fix deployed
- Deactivated device cleanup command deployed
- Company deactivation email ready but not yet pushed to production

### Check-in Timestamp Bug
- A device sent a check-in with an inaccurate device timestamp, causing it to sort incorrectly in the check-in export
- Team agreed to **use server time instead of device-reported time** for check-in timestamps — server time is always more accurate
- Adam noted this could become more common with cellular backup deployments where devices send data over non-cellular WAN interfaces

### Device Export — $0 Price Anomalies
- 15 devices showing $0 price in device export:
  - 10 are SmartVen devices — **intentional**, not meant to be billed
  - 2 are Wireless ATM Store / test warehouse devices — expected, testing artifacts
  - Remaining were explained by bill cycle cut timing
- Client's urgency resolved once the bill cycle explanation was provided

### Customer Losses
- **Bitcoin kiosk company** (publicly traded) filed Chapter 11, shut down ~8,000 kiosks. Direct business loss for APW
- **Republic:** Likely will not continue as a customer by end of year. Republic's CEO initially committed to staying, then reversed the decision a week later. Situation is not 100% confirmed but team is "very confident" they'll leave

### Verizon SIM Status Discrepancy — Device W62620
- Device shows as "deactivated" in the portal but is actually "suspended" in Verizon
- Root cause: Verizon suspended the device for data usage, then APW's deactivation request failed. A subsequent resume also failed. The portal recorded it as deactivated despite the API failures
- The device belongs to Republic (the customer potentially leaving)
- Devon manually deactivated the device during the call to stop further charges
- Richard to investigate the deeper issue of why the portal status drifted from Verizon's actual status

### SIM Terminology Rename Ticket
- Ticket ready for client review to rename "inactive" and "deactivated" SIM terminology
- Current naming confuses both internal team and customers
- Devon approved the approach with a minor tooltip text alteration needed
- **Action:** Devon to provide updated tooltip text so development can proceed

### Multi-Outlet Power Relay Feature — Prototype Review
- Feature is for I-52 routers with a new 4-outlet power relay accessory
- Aksana presented two prototype layouts: **grid view vs. table view**
- **Client chose the table layout** as more consistent with the portal's existing design
- Key capabilities reviewed:
  - Toggle between single-outlet and multi-outlet relay products
  - Individual outlet labeling (e.g., "Jukebox", "ATM", "Game")
  - Per-outlet power on/off/restart with confirmation dialogs
  - Bulk restart option for all outlets
  - Power cycler state and schedule visibility on the device page
  - Schedule manager expanded to support per-outlet scheduling
- Devon sent a Paint sketch of the relay hardware for use in the prototype image
- InHand is sourcing a cheaper 1-2 outlet relay alternative (current single outlet costs ~$37)
- Single and multi-outlet relays will be treated as **separate products** due to different API calls (DIO1 vs DIO4)
- I-52 can technically connect to a single-outlet relay, but it requires different API calls than I-22 — not inherently compatible
- Power cycler enable/disable setting moving from config group level to model level
- **Action:** Aksana to finish the PRD; client to review; then sizing and backlog placement

---

## Action Items

| Owner | Action | Priority |
|-------|--------|----------|
| Richard | Investigate email template change — identify commit/ticket that triggered it | High |
| Richard | Create ticket for using ASU value for signal strength on 4100 devices with newer firmware | Medium |
| Richard | Perform MySQL upgrade on Tuesday (June 3rd), early morning | High |
| Richard | Perform server upgrade (PHP 8.2) on Thursday (June 5th) | High |
| Richard | Investigate device W62620 Verizon SIM status discrepancy (portal vs. actual state) | Medium |
| Devon | Notify customers Monday about Tuesday's planned MySQL downtime | High |
| Devon | Provide updated tooltip text for inactive/deactivated SIM terminology ticket | Medium |
| Aksana | Finish multi-outlet power relay PRD and send to client for review | Medium |
| Team | Investigate rebranding scope for portal once new company name is finalized | Low |

---

## Decisions Made

- **Server time** will be used for check-in timestamps instead of device-reported time
- **Table layout** chosen over grid layout for multi-outlet power relay UI
- **MySQL upgrade:** Tuesday June 3rd; **Server upgrade:** Thursday June 5th
- Single-outlet and multi-outlet relays treated as separate products
- Power cycler enable/disable moves from config group level to model level
- apcommand.com domain retained for device check-ins; portal will use redirects for rebranding
- SmartVen $0 pricing is intentional — no action needed
