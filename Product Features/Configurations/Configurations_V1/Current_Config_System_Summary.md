# How Device Configurations Work Today (Legacy System)

## The Big Picture

Every WATM device needs a configuration file (.DAT) that tells it how to behave — firewall rules, DNS settings, DHCP, power schedules, hostname, etc. The current system uses a **file-based approach** with a 3-tier override hierarchy and automatic delivery triggered by device check-ins.

---

## Configuration Hierarchy (3 Levels)

```
Priority 1 (Highest):  Custom Company Configuration
                        → Company-specific .DAT file overriding the base
                        → Can be set to "apply_to_children" for child companies

Priority 2:            Parent Company Configuration
                        → Inherited from parent company if apply_to_children = true

Priority 3 (Default):  Base Configuration
                        → Standard .DAT file matched by device model + carrier + service plan + cellular backup
```

When the system needs a config for a device, it checks in that order and uses the first match.

---

## How Devices Get Matched to a Base Configuration

A base configuration is uniquely defined by **four criteria**:

| Criteria | Source | Example Values |
|----------|--------|----------------|
| **Device Model** | ConfigGroup → DeviceModel | IR615, IR302, I-22 |
| **Carrier** | Active SIM status | vz, att, tmo, dual_vz_att, dual_vz_tmo |
| **Service Plan** | Device's assigned plan | Basic, Premium, Backup |
| **Cellular Backup** | Boolean flag | true / false |

When a configuration is created or approved, the system **automatically scans all devices** and assigns it to every device that matches all four criteria. The carrier match uses the `device_sim_statuses` table to verify which SIMs are actually active.

---

## How Configurations Get Delivered to Devices

### Trigger: Device Check-in

```
Device powers on or checks in via UDP
  → Node.js server receives check-in on port 8002
  → Batched and sent to AWS SQS (process-checkins queue)
  → CakePHP queue worker picks it up
  → Compares device's REPORTED hostname vs. EXPECTED hostname from config
  → If they DON'T MATCH → queue a DeviceConfigurationUpdateJob
```

The hostname acts as a version indicator. If a device reports a different hostname than what the assigned configuration expects, the system knows the device is running an outdated config.

### The Update Job (DeviceConfigurationUpdateJob)

```
1. FETCH   → Get the device's configuration using the 3-tier hierarchy
             (custom company → parent company → base config)

2. DOWNLOAD → Pull the .DAT config file from AWS S3

3. OVERLAY  → Apply device-level custom_configurations (JSON key-value pairs)
              - For each key: find matching line in .DAT file via regex
              - If found: replace the value
              - If not found: append new line
              Example: fw_acl=custom_value replaces fw_acl=default_value

4. UPLOAD   → Push modified .DAT file to device via RouterApi
              - HTTP POST to device IP on port 4444
              - Endpoint: /upload.cgi?type=config
              - Uses HTTP Basic Auth with device credentials

5. REBOOT   → Reboot device to apply the new configuration
              (happens regardless of upload success/failure)

6. LOG      → Create entry in device_configuration_logs table
              - Records success or failure with details
              - Full audit trail of every config push attempt
```

### Manual Trigger

Admins can also force config updates via the queue worker:
```bash
bin/cake worker -p update-configs -c update-configs -r 295
```

---

## Hostname Matching — How Config Pushes Get Triggered

The hostname is the mechanism that connects device check-ins to config delivery. It acts as a **manual version indicator** — the admin changes the hostname string when they want devices to receive a new config.

### Where the expected hostname is stored

The "Anticipated Host Name" is a manually entered field on both the base configuration and custom company configuration screens. It's stored in two tables:

| Table | Column | Example |
|-------|--------|---------|
| `configurations` | `host_name` | `VZW22_03272026` |
| `custom_company_configurations` | `host_name` | `VZW22_CORD_04032026` |

The naming convention is typically `{carrier}{model}_{date}` or `{carrier}{model}_{customer}_{date}`. There is no auto-generation — the admin types this value manually when creating or editing a config.

### Where the reported hostname comes from

When a device checks in via UDP, the payload includes key-value pairs. One of those keys is `hostname` — the hostname the device is currently running. This gets parsed in `DeviceCheckinsTable::parseAndSave()` and saved to the `device_checkins` table.

### The comparison flow

In `DeviceCheckinsTable::afterSaveCommit()`, immediately after a check-in is saved:

```
1. Get the device's expected configuration:
   → getDeviceConfiguration(deviceId) resolves using the 3-tier hierarchy:
     Priority 1: Custom company configuration (direct)
     Priority 2: Parent company configuration (if apply_to_children = true)
     Priority 3: Base configuration

2. Compare:
   → checkin.hostname  ≠  configuration.host_name ?

3. If mismatch → queue DeviceConfigurationUpdateJob
```

### Double-check in the push job

The `DeviceConfigurationUpdateJob` does a **second hostname comparison** before actually pushing to the device. This handles the race condition where a device may have already received the config between when the job was queued and when it executes. If hostnames now match, the job skips the push.

### The `ignore_hostname` device flag

Individual devices have an `ignore_hostname` boolean flag. When set to `true`:
- Hostname comparison is bypassed entirely
- The config push always proceeds regardless of match/mismatch
- Used for manual intervention when a device needs a forced config push

### How a config update propagates to devices

The full sequence when an admin updates a config:

```
1. Admin edits a configuration and changes the "Anticipated Host Name"
   (e.g., VZW22_03272026 → VZW22_04032026)

2. All devices on this configuration still report the old hostname (VZW22_03272026)

3. On each device's next check-in:
   → Reported hostname: VZW22_03272026
   → Expected hostname: VZW22_04032026
   → MISMATCH → DeviceConfigurationUpdateJob queued

4. Job runs:
   → Second hostname check (still mismatched)
   → Downloads .DAT file from S3
   → Applies device-level custom_configurations overlay
   → Uploads to device via RouterApi
   → Reboots device
   → Logs result

5. Device comes back up with new config and new hostname (VZW22_04032026)

6. On next check-in:
   → Reported hostname: VZW22_04032026
   → Expected hostname: VZW22_04032026
   → MATCH → no action needed
```

### Key code locations for hostname matching

| Component | File |
|-----------|------|
| Check-in payload parsing | `DeviceCheckinsTable.php::parseAndSave()` (line ~237) |
| Hostname comparison trigger | `DeviceCheckinsTable.php::afterSaveCommit()` (line ~176) |
| Config resolution (3-tier) | `DevicesTable.php::getDeviceConfiguration()` (lines ~2799-2896) |
| Double-check before push | `DeviceConfigurationUpdateJob.php` (hostname check before upload) |
| Admin hostname entry (UI) | `SystemManagement/templates/Admin/Configurations/edit.php` (line ~70, field: "Anticipated Host Name") |

---

## Per-Device Customizations

Beyond company-level overrides, individual devices can have **device-level** customizations stored in a JSON field (`devices.custom_configurations`). These are applied as a final overlay on top of whatever configuration file the device receives.

Common use cases:
- Firewall ACL rules (`fw_acl`)
- Custom IP settings
- Device-specific DNS or DHCP overrides

---

## Approval Workflow

Configurations go through a status lifecycle:

```
pending → approved → (active/deployed)
pending → denied
approved → pending (auto-reset if any field is modified)
approved → pending_delete → deleted
```

- Any modification to an approved config **automatically resets it to pending**
- Approval triggers auto-assignment to matching devices
- Deletion is blocked if devices are still assigned to the config

---

## Key Database Tables

| Table | Purpose |
|-------|---------|
| `configurations` | Base config files with matching criteria |
| `config_groups` | Links device models to their configurations (1:1) |
| `configuration_service_plans` | Many-to-many join: configs ↔ service plans |
| `custom_company_configurations` | Company-level overrides referencing base configs |
| `devices.custom_configurations` | JSON field for per-device key-value overrides |
| `device_sim_statuses` | Tracks active/inactive SIM status per carrier |
| `device_configuration_logs` | Audit trail of every config push |
| `o_files` | S3 file storage metadata (Orases package) |

---

## Key Code Locations

| Component | File |
|-----------|------|
| Config matching & auto-assignment | `plugins/SystemManagement/src/Model/Table/ConfigurationsTable.php` (afterSave) |
| 3-tier config resolution | `plugins/Devices/src/Model/Table/DevicesTable.php:1671-1768` |
| Config update job | `plugins/Devices/src/Job/DeviceConfigurationUpdateJob.php` |
| Check-in hostname comparison | `plugins/Devices/src/Model/Table/DeviceCheckinsTable.php:169-185` |
| Device → router communication | `plugins/Devices/src/Util/RouterApi.php` |
| Custom config overlay logic | `DeviceConfigurationUpdateJob.php:100-110` |
| Admin UI | `plugins/SystemManagement/src/Controller/Admin/ConfigurationsController.php` |

---

## Limitations of the Current System

These are the pain points that V1 aims to solve:

1. **File-based configs** — The entire configuration is a monolithic .DAT file. Changing one parameter means re-uploading the whole file.

2. **Flat override model** — Only company-level and device-level overrides. No concept of reusable "override sets" or modular building blocks.

3. **No parameter-level control** — The system works at the file level, not the parameter level. You can't see which parameters differ between configs without diffing .DAT files.

4. **No verification** — No way to confirm a device actually received and applied the pushed config (Devon's concern from today's meeting).

5. **No versioning/rollback** — When a config is updated, the previous version is gone. No history, no rollback capability.

6. **Rigid matching** — Configs must match the full 4-way criteria (model + carrier + service plan + cellular backup). No "any model" or "any carrier" flexibility (Adam's feedback about override sets being agnostic to model/carrier/plan).

7. **Single override per company** — A company gets one custom config file. Can't stack multiple independent override sets (Cord Financial needing both firewall AND DNS overrides as separate modules).
