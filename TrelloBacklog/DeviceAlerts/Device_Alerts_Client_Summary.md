# Device Alerts — Client Summary

**Date:** 2026-04-06
**Related Ticket:** WATM-1659 — New alert types and notifications
**Epic:** WATM-1680 — New Device Alerts

---

## 1. What's Done

The foundation for device alerts is in place:

- **Alert reception** — The system can receive alert packets from InHand routers (I-22, I-52) and route them separately from regular device check-ins
- **Alert processing** — Incoming alerts are automatically parsed, matched to the originating device by serial number, and stored in the database
- **Error tracking** — If an alert can't be processed (e.g., unknown device, unrecognized alert type), it's logged with the reason for review
- **One alert type configured** — "LAN 2 Link Up/Down" is fully set up as a proof of concept
- **Database ready** — All the tables needed to store alerts, alert types, and processing failures are created

---

## 2. What's Remaining

### 2.1 Complete Alert Type Setup

Only 1 of 9+ alert types is configured. Once we have the confirmed list and the exact identifiers from InHand firmware, we can set up the rest:

| Alert Type | Status |
|---|---|
| WAN/LAN1 Link Up/Down | Needs firmware identifier |
| LAN2 Link Up/Down | Configured |
| Cellular Up/Down | Needs firmware identifier |
| Traffic Alarm | Needs firmware identifier |
| Traffic Disconnect Alarm | Needs firmware identifier |
| SIM/UIM Card Switch | Needs firmware identifier |
| Active Link Switch | Needs firmware identifier |
| SIM/UIM Card Fault | Needs firmware identifier |
| Signal Quality Fault | Needs firmware identifier |

The InHand firmware may support additional types beyond this list (e.g., service faults, low memory). We need to confirm the final scope.

### 2.2 Alert Viewing Interface

There is currently no way to view alerts in the portal. See our recommendations in Section 4 below for the proposed approach.

### 2.3 Alert Email Notifications

The system stores alerts but does not send any email notifications when they occur. The notification setup, email templates, and delivery pipeline all need to be built. See our recommendations in Section 4 below for the proposed approach.

---

## 3. Questions We Need Answered

### Before We Can Proceed

1. **InHand firmware alert identifiers** — We need the exact identifiers the InHand firmware sends for each alert type. Either sample alert data from devices or InHand documentation would work. We have one confirmed (for LAN2 Link Up/Down) but need the rest.

2. **Are devices already sending alerts?** — Has the firmware update been pushed to InHand devices? If so, alerts may already be hitting our system and being logged as unrecognized types, which would actually help us map the identifiers.

3. **Final alert type list** — The requirements list 9 types, but InHand firmware may support more. Should we configure all types the firmware supports, or only the 9 listed?

### Notification Setup Decisions

4. **Grouped or separate notification rules?** — We recommend a single "Device Alert" notification type where the user checks which alert types trigger it (see Section 4.2). Does that fit how your customers would configure this?

5. **Different email content per alert type?** — Does each alert type need its own email template wording, or is a generic "Alert: [type] occurred on device [serial] at [time]" sufficient for most cases?

6. **Company-level vs device-level?** — The existing notification system lets users apply to all devices or pick specific ones. Should alert notifications work the same way?

### Access & Scope

7. **Who can view alert history?** — We recommend all portal users (Super Admin and Admin) can see alerts scoped to their company. Should any roles be excluded?

8. **Who can set up alert notifications?** — We recommend the same roles that can currently set up device outage notifications. Any changes needed?

9. **Is the alert viewing interface in scope for initial delivery?** — The original requirements flagged this as potentially deferred. Should we include it in the first release, or deliver notification emails first and the viewing interface later?

10. **Combine old and new alerts in the view?** — The system already tracks some alerts (data spikes, devices coming back online, low signal). We recommend showing these alongside the new firmware alerts in a single view (see Section 4.3). Any concerns with that approach?

---

## 4. UI & UX Recommendations

### 4.1 Alert Viewing — Two Entry Points

We recommend two places for users to view alert history. Both follow the same interface patterns already used throughout the portal.

#### Device Detail Page — New "Alerts" Tab

The device detail page already has tabs for Signal Strength, Data Usage, and Daily Usage. We recommend adding an **"Alerts" tab** to this same area:

- Shows a filterable list of alerts for that specific device
- Columns: Date/Time, Alert Type, Details, Whether a Notification Was Sent
- Filters: Alert type dropdown, date range
- This is the natural place to look when troubleshooting a specific device

#### Global "Device Alerts" Page — New Menu Item

A new page under **Modules > Devices** in the main navigation:

```
Modules > Devices
  Browse Devices
  Device Alerts    <-- NEW
  Add Device
  ...
```

This page shows alerts across all devices within a company:

- Columns: Date/Time, Device (linked to device detail page), Company, Alert Type, Details, Notification Sent
- Filters: Company (for distributors managing sub-companies), Device (search by serial), Alert Type, Date Range
- Export to Excel
- Useful for operations teams monitoring alert trends across their entire fleet

**Who can access:** All portal roles (Super Admin and Admin), scoped to their own company and sub-companies.

### 4.2 Alert Notification Setup — Extending Company Notifications

Rather than building a new notification system, we recommend extending the **existing Company Notifications** feature that customers already use for device outage emails. This keeps the experience familiar and reuses all existing functionality.

#### How It Would Work

**Today:** When setting up a notification, the only option is "Device Outage."

**Proposed:** A new option — "Device Alert" — appears alongside "Device Outage" in the notification type dropdown.

When a user selects "Device Alert," the form shows:

- **Alert type checkboxes** — The user selects which alert types should trigger this notification. For example:
  - WAN/LAN1 Link Up/Down
  - LAN2 Link Up/Down
  - Cellular Up/Down
  - SIM/UIM Card Switch
  - Signal Quality Fault
  - _(etc. — populated from the configured alert types)_

- **Email subject and body** — Same editor as outage notifications, with new template variables available:
  - Alert type name (e.g., "LAN 2 Link Down")
  - Time the alert occurred on the device
  - Network interface involved (e.g., "LAN2", "WAN", "Cellular")
  - Alert detail summary

- **Device targeting** — Same as today: apply to all devices, or select specific devices

- **Scheduling** — Same as today: set active date ranges and operating hours (day-of-week + time windows)

**Note:** Unlike outage notifications, alert notifications would not have a "Restored" email — alerts are one-time events, not a status that recovers.

#### Multiple Rules for Different Content

If a customer wants different email wording for SIM switches vs. link failures, they simply create two notification rules — one with SIM-related types checked, one with link-related types checked. Each rule has its own email subject and body. This matches how outage notifications work today.

#### Why This Approach

- Customers already know the Company Notifications interface
- Same permission model — the same users who manage outage notifications can manage alert notifications
- Same audit trail — all sent notifications are logged and viewable
- Same scheduling and device targeting — no new concepts to learn
- Minimal new UI — the notification setup form gets one new conditional section

### 4.3 Unified Alert Timeline

The portal already tracks some system-generated alerts (data usage spikes, devices coming back online after extended downtime, low signal strength). These are separate from the new firmware-generated alerts, but we recommend **combining both into a single timeline** in the UI:

| Source Label | Example Alerts |
|---|---|
| **System Alert** | Data Spike, Device Back Online, Low Signal |
| **Device Alert** | LAN2 Link Down, SIM Switch, Cellular Down, Signal Quality Fault |

Users would see one combined, date-sorted list with a source label to distinguish where the alert came from. This gives users a single place to see all alert activity for a device or across their fleet, without needing to check multiple screens.

### 4.4 Notification Flow Summary

Here's what happens when an alert comes in, end to end:

1. InHand router detects an event (e.g., LAN2 link goes down)
2. Router sends an alert packet to the WATM system
3. System receives and stores the alert, matched to the correct device
4. System checks: does this alert type have email notifications enabled?
5. If yes: finds all matching notification rules for this device's company
6. Checks scheduling (is this within the configured active hours?)
7. Sends the notification email with the alert details filled in
8. Logs the sent notification for audit purposes

All of this happens automatically — no manual intervention needed once the notification rules are configured.
