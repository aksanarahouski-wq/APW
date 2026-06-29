# WATM-1659: Device Alerts — Research & Status

**Ticket:** WATM-1659 — New alert types and notifications
**Parent Epic:** WATM-1680 — New Device Alerts (On-Hold)
**Status:** In Progress (flagged as Impediment)
**Assignee:** Aksana Rahouski
**Date:** 2026-04-05

**Child Tickets:**
- WATM-1717 — Handle New Alert Processing and Data (In Progress)
- WATM-1718 — Company notifications for new alert types (Needs Review and Estimate)
- WATM-1719 — View device alerts (Needs Review and Estimate)

**Requirements Doc:** [Confluence — Device Alerts Processing and Notifications scope](https://orases.atlassian.net/wiki/spaces/WATM/pages/1955037199/Device+Alerts+Processing+and+Notifications+scope.)

---

## 1. What's Already Built

### 1.1 UDP Alarm Ingestion (Node.js) — DONE

The check-ins service (`watm/checkins/index.js`) already separates alarm packets from regular check-ins:
- Detects packets containing `alarm_msg` string
- Routes alarm messages to a separate batch (`deviceAlarmMessages`)
- Sends alarm batches to the `process-device-alarms` SQS queue (separate from `process-checkins`)
- Tracks CloudWatch metrics: `AlarmUDPPackets`, `AlarmBatchSize`
- Batching interval: 15 seconds

### 1.2 Alarm Queue Processing (CakePHP) — DONE

**File:** `plugins/Devices/src/Job/ProcessAlarmJob.php`

- Receives alarm data from SQS
- Calls `DeviceAlarmsTable::parseAndSaveMany()`
- Parsing logic:
  - Splits raw alarm string by newlines, extracts key=value pairs
  - Requires `serialnum` (matched to `devices.manufacturer_serial_number`)
  - Decodes `alarm_msg` as JSON array of alarm objects
  - For each alarm: constructs keyword as `{type}_{interface}` (e.g., `link_change_lan2`)
  - Looks up keyword in `device_alarm_types` table
  - Saves to `device_alarms` or logs to `device_alarm_failures`

### 1.3 Database Schema — DONE (Feb 2025)

Four tables created via migrations:

**`device_alarms`** — Stores processed alarms
| Column | Type | Notes |
|--------|------|-------|
| id | int (PK) | |
| device_id | int (FK) | Links to devices table |
| device_alarm_type_id | int (FK) | Links to device_alarm_types |
| timestamp | datetime | When the alarm occurred on device |
| alarm_data | text | JSON with alarm details |
| created | datetime | When stored in system |

**`device_alarm_types`** — Alarm type definitions
| Column | Type | Notes |
|--------|------|-------|
| id | int (PK) | |
| alarm_keyword | varchar(255) | Unique, e.g. `link_change_lan2` |
| title | varchar(255) | Display name |
| triggers_email | boolean | Default 0 — flag for notification trigger |

**`device_alarm_failures`** — Failed alarm processing
| Column | Type | Notes |
|--------|------|-------|
| id | int (PK) | |
| device_alarm_failure_type_id | int (FK) | |
| alarm_text | text | Raw unparseable data |
| error_text | varchar(255) | Human-readable error |
| created | datetime | |

**`device_alarm_failure_types`** — Failure categories
- `failed_find` — Device not found by serial
- `failed_gather` — Failed to save fields
- `failed_parse` — Failed to parse alarm structure
- `failed_serial` — Missing serial number
- `undefined_type` — Alarm keyword not in device_alarm_types

### 1.4 Alarm Type Seed — MINIMAL

**File:** `config/Seeds/DeviceAlarmTypesSeed.php`

Only **1 alarm type** is seeded:
- `link_change_lan2` — "LAN 2 Link Up/Down" (triggers_email: 1)

**Expected alarm types from InHand firmware config** (from `TestInputCommand.php`):
```
fault-service, memory-low, port0-wan-link-up/down, port1-4-link-up/down,
dialup-up/down, traffic-alarm, traffic-discon, switch-sim-card,
switch-backup-link, fault-sim-card, fault-signal-quality
```

The seed needs to be expanded to cover all alert types from the requirements.

### 1.5 Error Tracking — DONE

Failures are categorized and stored with the original raw alarm text for debugging.

---

## 2. What's NOT Built

### 2.1 Complete Alarm Type Definitions

Only 1 of ~9+ alarm types is seeded. Need the full mapping of InHand firmware alarm keywords to human-readable titles. The requirements doc lists:

| Alert Type | Alarm Keyword (TBD) | Seeded? |
|-----------|---------------------|---------|
| WAN/LAN1 Link-Up/Down | ? | No |
| LAN2 Link-Up/Down | `link_change_lan2` | Yes |
| Cellular Up/Down | ? | No |
| Traffic Alarm | ? | No |
| Traffic Disconnect Alarm | ? | No |
| SIM/UIM Card Switch | ? | No |
| Active Link Switch | ? | No |
| SIM/UIM Card Fault | ? | No |
| Signal Quality Fault | ? | No |

### 2.2 Admin UI for Viewing Device Alarms (WATM-1719)

**Status:** Not started. No controllers, routes, or templates exist.

**Needed:**
- Device alarms list/index page with filtering (by type, device, date range)
- Device alarms view page (alarm details)
- Integration into device view page (alarm history tab/section)
- Searchable alert history

**Open questions from requirements doc:**
- Who should have access: APC Admins only? Customer Admins too?
- Is alert history in scope for Phase 1 or deferred?

### 2.3 Notification Triggering from Alarms (WATM-1718)

**Status:** Not started. The `triggers_email` flag exists on `device_alarm_types` but nothing reads it.

**Current gap:**
- `ProcessAlarmJob` saves alarms but does NOT trigger any notifications
- No integration between `device_alarms` and `company_notifications`
- No job to send alarm-based notifications
- No alarm-specific notification templates

### 2.4 Company Notification Integration

**Existing notification system supports:**
- Type: only `outage` (device goes offline/comes back)
- Delivery: email only (SMS deprecated May 2025)
- Device targeting: all devices OR specific device list
- Scheduling: date ranges + operating hours (day-of-week + time windows)
- Shortcodes: `{{service_plan}}`, `{{billing_cycle}}`, `{{location_name}}`, `{{tid}}`
- Logging: full audit trail of sent notifications

**What needs to be added:**
- New notification type(s) for device alarms (one per alarm type? or a single "device alert" type with sub-configuration?)
- New shortcodes for alarm-specific data (alarm type, interface, timestamp, etc.)
- Alarm type selection UI on the company notification config page
- Trigger mechanism: ProcessAlarmJob -> check triggers_email -> find matching company notification -> queue notification send
- New job: `SendDeviceAlarmNotificationJob` (following pattern of `SendDeviceOutageNotificationJob`)

### 2.5 Legacy Device Alerts vs New Device Alarms

There are **two separate systems**:

1. **`device_alerts`** (2023) — System-generated alerts based on data analysis:
   - `data_spike` — Device uses >10% of monthly usage in one day
   - `back_online` — Device comes back online after >30 days
   - `low_signal` — Signal strength <10%
   - These are NOT connected to notifications

2. **`device_alarms`** (2025) — Device-generated alerts from InHand firmware:
   - Link changes, cellular events, SIM switches, signal faults
   - This is the system WATM-1659 is about

**Question:** Should these two systems be unified or kept separate? They serve different purposes but the naming overlap could cause confusion.

---

## 3. Questions for the Client

### Must-Answer Before Development

1. **Complete alarm keyword mapping:** We need the exact keywords the InHand firmware sends for each alert type. We have `link_change_lan2` confirmed — what are the keywords for the other 8+ types? Can the client provide a sample alarm packet for each type, or firmware documentation?

2. **Are devices already sending alarm packets?** The firmware was supposed to be pushed to InHand devices (I-22, I-52). Is that complete? Are alarm packets already hitting our UDP server and being stored/failing?

3. **Alarm types confirmation:** The requirements list 9 types, but the firmware config string shows additional types (`fault-service`, `memory-low`). What is the final, confirmed list of alarm types we need to support?

### Notification Configuration

4. **One notification rule per alarm type, or grouped?** Should the customer create separate notification configs for each alarm type (e.g., one for "LAN2 Link Down", another for "SIM Switch")? Or should there be a single "Device Alert" notification with checkboxes for which alarm types trigger it?

5. **Notification content per alarm type:** Does each alarm type need different notification body content/shortcodes? For example, a SIM switch notification might include which SIM was switched to, while a link-down notification shows which interface went down. What data should be available as shortcodes?

6. **Company-level vs device-level alert configuration:** The requirements doc asks this explicitly:
   - Do we configure on a company level (all company devices send alerts)?
   - Do we configure on a device level (per-device opt-in)?
   - The existing `company_notifications` system supports both (via `is_all_devices` flag + `company_notification_devices` junction table) — should we reuse this pattern?

### Access & Visibility

7. **Who can view alert history?** The requirements doc asks: APC Admins? Customer Admins? Both? All roles?

8. **Who can configure alert notifications?** Same role that configures device outage notifications today (customer admin), or restricted?

### Scope & Priority

9. **Is alert history/viewing (WATM-1719) in scope for the initial delivery, or deferred?** The requirements doc flags this as "In/Out of scope?"

10. **Should we unify the legacy `device_alerts` (data spike, back online, low signal) with the new `device_alarms` system?** Or keep them separate? They currently live in separate tables with different schemas.

---

## 4. Remaining Requirements

### 4.1 Alarm Type Seed Completion

Only 1 of ~9+ alarm types is seeded. All alarm type keywords need to be confirmed with InHand firmware documentation and seeded via migration. See Section 2.1 for the full list.

### 4.2 Notification Triggering from Alarms

The `triggers_email` flag exists on `device_alarm_types` but nothing reads it. Full notification integration is needed:
- Trigger mechanism in `ProcessAlarmJob` after alarm save
- New notification type (`device_alert`) added to `CompanyNotification::getTypes()`
- New `SendDeviceAlarmNotificationJob` following the outage notification pattern
- Alarm-specific shortcodes for email templates
- Company notification UI updates (alarm type selection checkboxes)

### 4.3 Admin UI for Viewing Device Alarms

No controllers, routes, or templates exist for viewing device alarms. Needed:
- Global "Device Alerts" page (new nav item, new controller)
- Device view page "Alerts" tab integration
- Filtering by type, device, company, date range
- Export capability

### 4.4 Client Questions Still Open

See Section 3 for the full list. Key blockers:
- Complete alarm keyword mapping from InHand firmware
- Confirmation of which alarm types to support
- Notification configuration UX decisions (per-type vs grouped)
- Access/visibility decisions (which roles see what)
- Scope decision on alert history viewing

---

## 5. UI & UX Recommendations

### 5.1 Alert History — Two Entry Points

**Entry Point 1: Device View Page — "Alerts" Tab**

The device view page (`plugins/Devices/templates/Admin/Devices/view.php`) uses a tabbed layout with query-param-driven tab switching. Add an **"Alerts" tab** to the Statistics card section (alongside Signal Strength, Data Usage, Daily Usage):

- Filterable DataTable of alarms for that specific device
- Columns: Date/Time, Alert Type (badge), Details (parsed from `alarm_data` JSON), Notification Sent (yes/no indicator)
- Filters: Alert type dropdown, date range
- Follows existing tab pattern: `?stats-tab=alerts`
- Provides per-device context when troubleshooting a specific device

**Entry Point 2: Global "Device Alerts" Page — New Nav Item**

Add to the Devices plugin navigation (`plugins/Devices/src/Plugin.php`):
```
Modules > Devices
├── Browse Devices
├── Device Alerts    ← NEW
├── Add Device
├── ...
```

New controller: `DeviceAlertsController` in the Devices plugin. This is the global alert browser across all devices within the user's company scope:

- Filters: Company (super admin only), Device (serial search), Alert Type (dropdown), Date Range
- Columns: Date/Time, Device Serial (linked to device view), Company, Alert Type (badge), Details, Notification Sent
- Export capability
- Follows the same DataTable + filter panel pattern as the device index page
- Can unify both `device_alarms` (firmware-generated) and `device_alerts` (system-generated like data_spike, back_online, low_signal) into a single timeline view for the user, querying both tables

**Access:** Both views visible to all roles (super admin, company admin, customer) scoped to their company.

### 5.2 Notification Configuration — Extend Existing Company Notifications

Rather than building a separate notification system, extend the existing Company Notifications UI (`plugins/Notifications/templates/Admin/CompanyNotifications/edit.php`).

**Current state:** The `CompanyNotification` entity has `getTypes()` returning only `['outage' => 'Device Outage']`. The edit form renders fields conditionally based on type.

**Proposed changes:**

1. **Add new notification type** to `CompanyNotification::getTypes()`:
   ```php
   'outage'       => 'Device Outage',      // existing
   'device_alert' => 'Device Alert',       // new
   ```

2. **Conditional form section** — When "Device Alert" is selected as the type, show:
   - **Alert type checkboxes:** Which alarm types trigger this notification (e.g., WAN Link Down, LAN2 Link Down, Cellular Down, SIM Switch, etc.). Populated from `device_alarm_types` table.
   - Title and Body fields (same as outage, with new shortcodes)
   - No "Restored" title/body section (alerts don't have a restored state like outage does)

3. **Reuse everything else as-is:**
   - Device targeting: all devices or specific device list (existing `is_all_devices` + `company_notification_devices`)
   - Operating hours / date range scheduling (existing fields)
   - Email delivery (existing mechanism)
   - Audit logging (existing `company_notification_logs`)

4. **New shortcodes** added to `KeyWordTranslatorTrait`:
   - `{{alert_type}}` — Human-readable alert type name (e.g., "LAN 2 Link Down")
   - `{{alert_time}}` — When the alert occurred on the device
   - `{{alert_interface}}` — Network interface involved (e.g., "LAN2", "WAN", "Cellular")
   - `{{alert_details}}` — Parsed summary of the alarm data

5. **One notification config = one notification rule.** If a customer wants different email content for SIM switches vs link-downs, they create two notification configs — one with SIM types checked, one with link types checked. This matches how outage notifications work (one config per rule).

**Why extend rather than build new:**
- Customers already know the Company Notifications UI
- Same permission model (customer role can manage)
- Same audit trail (`company_notification_logs`)
- Same scheduling/device targeting infrastructure
- The edit form just gets a new conditional section — minimal UI work

### 5.3 Notification Trigger Flow

New processing flow after alarm is saved:

```
ProcessAlarmJob saves alarm
    → Check device_alarm_types.triggers_email = true
    → Query company_notifications WHERE type = 'device_alert'
        AND alarm_type matches notification's selected types
        AND device matches (all devices or in device list)
    → Check scheduling constraints (date range, operating hours)
    → Queue SendDeviceAlarmNotificationJob
        → Translate shortcodes with alarm-specific data
        → Send email via existing SendNotificationJob
        → Log to company_notification_logs
```

New job: `SendDeviceAlarmNotificationJob` — follows the pattern of `SendDeviceOutageNotificationJob` but receives alarm data instead of offline status data.

### 5.4 Unifying Legacy Alerts in the UI

The legacy `device_alerts` (data_spike, back_online, low_signal) and new `device_alarms` (firmware-generated) can be presented in a **unified timeline** in the UI without merging database tables:

- The global Device Alerts page and the device view Alerts tab query both tables
- Display a combined, date-sorted list with a "Source" indicator (System / Device)
- Legacy alerts show as "System Alert: Data Spike", "System Alert: Device Back Online", etc.
- Firmware alerts show as "Device Alert: LAN2 Link Down", "Device Alert: SIM Switch", etc.

This gives users one place to see everything without a database refactor.

### 5.5 Summary of Changes by Area

| Area | What Changes |
|------|-------------|
| Devices Plugin nav | Add "Device Alerts" menu item |
| Device view page | Add "Alerts" tab to statistics section |
| New controller | `DeviceAlertsController` for global alert browser |
| New templates | Alert index (list + filters), alert detail view |
| CompanyNotification entity | Add `'device_alert'` to `getTypes()` |
| CompanyNotifications edit form | Add conditional alert type checkboxes section when type = device_alert |
| KeyWordTranslatorTrait | Add `{{alert_type}}`, `{{alert_time}}`, `{{alert_interface}}`, `{{alert_details}}` |
| New job | `SendDeviceAlarmNotificationJob` |
| ProcessAlarmJob | Add notification trigger logic after save |
| company_notifications table | New junction table or JSON column for selected alarm types per notification |

---

## 6. Key Files Reference

### Existing Implementation
| Area | File Path |
|------|-----------|
| UDP alarm handling | `watm/checkins/index.js` (lines 386-410, 501-507) |
| Alarm processing job | `plugins/Devices/src/Job/ProcessAlarmJob.php` |
| Alarm model/parsing | `plugins/Devices/src/Model/Table/DeviceAlarmsTable.php` |
| Alarm types model | `plugins/Devices/src/Model/Table/DeviceAlarmTypesTable.php` |
| Alarm failures model | `plugins/Devices/src/Model/Table/DeviceAlarmFailuresTable.php` |
| Alarm type seed | `config/Seeds/DeviceAlarmTypesSeed.php` |
| Alarm failure types seed | `config/Seeds/DeviceAlarmFailureTypesSeed.php` |
| DB migrations | `config/Migrations/20250224174024_CreateDeviceAlarms.php` |
| | `config/Migrations/20250224174932_CreateDeviceAlarmTypes.php` |
| | `config/Migrations/20250224175526_CreateDeviceAlarmFailureTypes.php` |
| | `config/Migrations/20250224182357_CreateDeviceAlarmFailures.php` |
| | `config/Migrations/20250224182436_HandleDeviceAlarmForeignKeys.php` |
| InHand alarm config reference | `watm/watm/src/Command/TestInputCommand.php` (line 24) |

### Notification System (to integrate with)
| Area | File Path |
|------|-----------|
| Company notifications controller | `plugins/Notifications/src/Controller/Admin/CompanyNotificationsController.php` |
| Company notifications model | `plugins/Notifications/src/Model/Table/CompanyNotificationsTable.php` |
| Company notification entity | `plugins/Notifications/src/Model/Entity/CompanyNotification.php` |
| Notification send job | `plugins/Notifications/src/Job/SendNotificationJob.php` |
| Device outage notification job | `plugins/Devices/src/Job/SendDeviceOutageNotificationJob.php` |
| Shortcode translation | `plugins/Notifications/src/Traits/KeyWordTranslatorTrait.php` |
| Notification edit template | `plugins/Notifications/templates/Admin/CompanyNotifications/edit.php` |

### Legacy Device Alerts (separate system)
| Area | File Path |
|------|-----------|
| Device alerts model | `plugins/Devices/src/Model/Table/DeviceAlertsTable.php` |
| Device alert types model | `plugins/Devices/src/Model/Table/DeviceAlertTypesTable.php` |
| Alert types seed | `config/Seeds/DeviceAlertTypesSeed.php` |
| Test alert command | `plugins/Devices/src/Command/TestDeviceAlertCommand.php` |
