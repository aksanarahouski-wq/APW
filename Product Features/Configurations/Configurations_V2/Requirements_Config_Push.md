# Configuration V2 — Config Push, Delivery & Verification

**Phase:** Phase 2 (after Config Builder is finalized)
**Last Updated:** April 13, 2026

---

## Purpose

This document defines how V2 configurations get delivered to devices and how we verify they were actually applied. This is the critical bridge between "building configs in the portal" (Phase 1) and "configs running on devices."

---

## How the Legacy System Delivers Configs

1. **Trigger:** Device checks in via UDP → hostname comparison detects mismatch → queues a `DeviceConfigurationUpdateJob`
2. **Delivery:** Job downloads .DAT file from S3 → applies custom overlays → uploads to device via RouterApi (HTTP POST to port 4444)
3. **Post-delivery:** Device is rebooted regardless of success/failure → result logged to `device_configuration_logs`
4. **Manual option:** Admins can trigger config pushes via queue worker (`bin/cake worker -p update-configs`)

### Known Gaps in Legacy System

| Gap | Description |
|-----|-------------|
| **No delivery verification** | The system pushes a config but has no way to confirm the device received and applied it |
| **Device can be unreachable** | API access on the device could be disabled, or device could be offline at push time |
| **No retry intelligence** | If a push fails, there's no smart retry — it waits for next check-in hostname mismatch |
| **No push status dashboard** | No centralized view showing which devices have/haven't received the latest config |
| **Reboot always happens** | Device reboots even if upload failed — unnecessary disruption |
| **No rollback mechanism** | If a bad config is pushed, there's no way to roll back to a known-good state |
| **No batch push control** | Can't say "push to these 50 devices first, then the rest if it goes well" |
| **Single trigger mechanism** | Only check-in hostname mismatch or manual queue trigger — no on-demand push from portal |

---

## Delivery Model

Enhanced push model that leverages the existing check-in infrastructure as a trigger mechanism, plus a new on-demand manual push from the portal. The portal always initiates the push — devices never download configs on their own.

| Component | Mechanism | New vs. Existing |
|-----------|-----------|-----------------|
| **Check-in triggered push** | Device checks in → hostname/version mismatch detected → system queues config push → pushes via RouterApi | Existing flow, enhanced for V2 compiled configs |
| **On-demand manual push** | Admin selects specific devices in the portal → triggers immediate push via RouterApi, bypassing check-in wait | **New** |
| **Retry on check-in** | If a push fails (device unreachable, API error), the device is flagged → next check-in triggers a retry push | Enhanced existing (smarter retry tracking) |

---

## Verification Strategy

Two-layer verification using push result (immediate) and hostname match on check-in (ongoing). The goal is to know whether the whole configuration was successfully pushed and accepted by the device. Hostname update is part of the config — if the hostname matches after check-in, the full config landed. If it doesn't match, the device needs attention. No firmware changes required.

| Layer | When | What Happens | What It Tells You |
|-------|------|-------------|-------------------|
| **Push result** | Immediately on push | RouterApi returns SUCCESS or an error (unreachable, API disabled, upload error, etc.) | Whether the device was reachable and accepted the upload |
| **Hostname verification** | Next device check-in | Device reports its hostname via UDP check-in. System compares against the expected hostname for the current config version. | Whether the device actually applied the config and is running it |

### Per-Device Push Status Tracking

Every config push — whether triggered by check-in or on-demand — creates a status record per device. This gives the client visibility into the state of every device's configuration at any time.

**Per-device record:**

| Field | Description |
|-------|-------------|
| Device | Which device received the push |
| Config version | Which compiled configuration was pushed |
| Push timestamp | When the push was attempted |
| Push trigger | Check-in triggered / On-demand manual / Retry |
| Push result | Success / Failed: unreachable / Failed: API disabled / Failed: upload error |
| Hostname status | Verified (hostname matched on next check-in) / Pending (awaiting next check-in) / Mismatch (hostname didn't match — config may not have applied) |
| Hostname verified at | Timestamp of the check-in that confirmed hostname match |

**Device config status lifecycle:**

```
Config push triggered
  → Push succeeds     → Status: Pushed, hostname pending
                          → Next check-in: hostname matches    → Status: Verified
                          → Next check-in: hostname mismatch   → Status: Mismatch (needs attention)
                          → No check-in within expected window → Status: Unverified (stale)
  → Push fails        → Status: Failed (with reason)
                          → Queued for retry on next check-in
```

**Fleet-level visibility:**

Admins can view push status across all devices to see at a glance how a config rollout is progressing:

| Status | Meaning |
|--------|---------|
| **Verified** | Push succeeded AND hostname confirmed on subsequent check-in |
| **Pushed, pending verification** | Push succeeded, awaiting next check-in to confirm hostname |
| **Failed** | Push failed (with reason: unreachable, API disabled, upload error) |
| **Retry queued** | Previous push failed, will retry on next check-in |
| **Mismatch** | Push reported success but hostname didn't update on check-in — needs investigation |

---

## Hostname Versioning

Devon and Adam proposed a hostname-based version tracking scheme that aligns with the verification strategy:

- When a config change is made, iterate the hostname with the date
- Example: `VZW22_03272026` → next change → `VZW22_04032026`
- Naming scheme: `{carrier}{model}_{date}` or `{carrier}{model}_{customer}_{date}`
- When device checks in and reports an old hostname, the portal knows the push didn't land
- This already partially exists in the legacy system — hostname mismatch triggers the update job

---

## Config Deployment Pipeline

Config delivery follows a pipeline with defined stages:

```
  → Stage 1: COMPILE
      Resolve all 5 inheritance levels into a final compiled .DAT file per device
      Store compiled config with version ID and expected hostname

  → Stage 2: VALIDATE (pre-push)
      Run parameter validation (data types, required fields, constraints)
      Diff against current running config — show what's changing
      Admin reviews and confirms push

  → Stage 3: DELIVER
      Push compiled .DAT file to device(s) via RouterApi
      Log push attempt with timestamp, result, and any error
      If device unreachable, queue for retry

  → Stage 4: VERIFY
      Immediate: Check RouterApi response (SUCCESS/ERROR)
      Ongoing: Hostname check on every subsequent check-in — match = verified

  → Stage 5: REPORT
      Dashboard shows per-device config status:
        Verified — push succeeded, hostname confirmed on check-in
        Pushed — push succeeded, awaiting hostname verification on next check-in
        Pending — push queued, awaiting delivery
        Failed — push failed, needs attention
        Mismatch — push succeeded but hostname didn't update, needs investigation
        Retry Queued — previous attempt failed, will retry on check-in
```

### When Does Compilation Happen? (Needs Team Discussion)

There are two approaches to when the system resolves the 5-level hierarchy and produces the compiled .DAT file for each device. This is a backend architecture decision that affects push speed, change visibility, and system complexity.

**Option A: Compile at push time**
- Compilation happens when a push is triggered (check-in or on-demand)
- The system resolves all 5 levels, generates the .DAT file, and pushes it in one flow
- No compiled files are stored until a push is needed
- Simpler to build — compilation is a single codepath triggered by push
- Tradeoff: on-demand push is slower (compile first, then push), and you don't know what a config change affects until you trigger compilation

**Option B: Pre-compile on every config change**
- Every time anything changes in the hierarchy (schema default, model default, 3-way rule, override set, company assignment), the system immediately recompiles the .DAT file for every affected device
- The compiled .DAT is always sitting there, ready to push — no compilation at push time
- On-demand push is fast — just grab the latest file and send it
- You get immediate visibility into how a change affects the fleet (the compiled files are already generated, so you can count affected devices and diff against previous versions)
- Tradeoff: more complex — the system must track which config entities affect which devices and trigger recompilation on every change. Storage for compiled files per device (~1,500+ files).

**Option C: Pre-compile parameters, generate .DAT at push time**
- On every config change, resolve the 5-level hierarchy and store the compiled parameter set in the database (not as a .DAT file)
- At push time, generate the .DAT file from the stored parameter set — fast, since resolution is already done
- Gives the change visibility benefits of Option B without managing .DAT files on disk
- Tradeoff: two-step process (parameter resolution on change, .DAT generation on push), but each step is simpler than doing everything at once

Note: The Requirements document already specifies "Automatic Recompilation on Device Attribute Changes" — the system must recompile when a device's carrier, service plan, company, or model changes. If recompilation on device attribute changes is already required, extending it to also recompile on config value changes (Option B or C) may be a natural fit.

**This needs to be discussed with the dev team** — it affects backend architecture but does not change the admin-facing UI or the prototype.

### Config Status Per Device (Database Design Concept)

A new table or extended field tracking the delivery state for each device:

```
device_config_deployments
  device_id
  config_version_id (links to the versioned compiled config)
  expected_hostname (the hostname the device should report after applying this config)
  deployment_status: pending | pushing | pushed | verified | failed | mismatch | retry_queued
  push_trigger: checkin | on_demand | retry
  push_attempts: integer (count of tries)
  last_push_at: datetime
  last_push_result: success | unreachable | api_disabled | upload_error | reboot_error
  hostname_verified_at: datetime (when check-in confirmed hostname match)
  created / modified
```

### Push Dashboard

A dedicated admin page showing config deployment status across the fleet:

```
Config Deployment Status — v2.3 (approved March 27, 2026)

  Verified              1,247    82%
  Pushed (pending)        142     9%
  Failed                   47     3%
  Mismatch                 31     2%
  Retry Queued             53     4%

  [Retry All Failed]  [Push to Pending]  [View Details]
```

With drill-down to see individual device status, push history, and error details.

### Retry Strategy

Retry behavior depends on the failure reason — not a one-size-fits-all escalation timer.

| Failure Reason | What Happened | Retry Approach |
|---|---|---|
| **Unreachable** | Device is offline or no network path to the device | Wait for next check-in. The check-in is the signal that the device is back online and reachable — that's when the system retries the push. No point retrying before then. |
| **Upload Error** | Device was reachable, API responded, but the upload failed (timeout, malformed response, transient error) | Retry immediately (once). If the immediate retry also fails, queue for next check-in. |
| **Reboot Error** | Config uploaded successfully but the reboot command failed | Retry reboot immediately (once). If the retry also fails, flag for admin — the config may have been applied but the device didn't restart to pick it up. |
| **API Disabled** | Device is reachable but API access is turned off on the device | Do not auto-retry. This requires manual intervention on the device itself — no amount of retrying from the portal will fix it. Flag for admin immediately. |

Admin can always manually trigger a retry from the push dashboard or device page, regardless of failure reason.

### V2 Config Compilation → .DAT Output

The V2 configuration engine outputs the same .DAT file format that InHand devices consume today. Devices don't need to know or care that the config was built by the V2 engine — they receive a .DAT file via RouterApi, same as the legacy system.

```
V2 Parameter Set (DB)
  → Compiler → .DAT file (device-compatible format)
  → Push .DAT via RouterApi (same as legacy)
  → Device receives and applies .DAT file (no change from device's perspective)
```

This means **V2 doesn't require any device firmware changes.**

---

## Failure Handling (Open)

What happens when a config push fails?

| Scenario | Action |
|----------|--------|
| Device unreachable (offline/NAT) | Queue for retry on next check-in; flag device in dashboard |
| API access disabled on device | Flag for admin; no auto-retry — requires manual intervention on the device |
| Upload error (transient) | Retry immediately once; if still fails, queue for next check-in |
| Reboot fails after successful upload | Retry reboot once; if still fails, flag for admin |
| Config applied but causes device malfunction | Fix the configuration in the portal and push the corrected config |
| Push succeeds but hostname doesn't update on next check-in | Re-push; flag for admin to investigate device-side issue |
| Partial fleet failure (some succeed, some fail) | Dashboard shows per-device status; admin fixes issues and retries failed devices |

**No rollback mechanism.** If a bad config lands on a device, the fix path is: correct the configuration values in the portal (schema, 3-way rule, override set — wherever the issue is) and push the corrected config. There is no "revert to previous version" feature. This keeps the system simple — the config builder is the single source of truth, and the fix-and-push workflow is already fast enough.

---

## Push Scope and Staging

The approval gate for config changes lives in the **config builder**, not in the push system.

3-way rules and override sets — the two levels with the widest blast radius — use a draft/publish workflow with two-person approval (see Requirements document, "Versioning and Approval" section). An admin makes changes, which are saved as a draft. A second admin reviews and approves the draft, at which point it becomes the published version. Only published versions are pushed to devices.

This means:
- **No staging or gating needed at the push level.** The safety gate is the approval step in the config builder. Once a config change is approved, it propagates to affected devices on their next check-in — same as the legacy system today.
- **On-demand push is available** for admins who want to push to specific devices immediately after approval, rather than waiting for check-in.
- **The push dashboard shows rollout progress** as devices check in and receive the approved config.

The push system's job is delivery and verification — not approval. Approval happens before a config ever reaches the push pipeline.

---

## Relationship to Other Phases

| Phase | Scope | Dependency |
|-------|-------|------------|
| **Phase 1** (current) | Config Builder — schema, inheritance, override sets, resolution logic | None — standalone |
| **Phase 2** (this document) | Config Push, Delivery, Verification, Reporting | Depends on Phase 1 (needs compiled configs to push) |
| **Legacy Coexistence** | Per-device toggle between legacy and V2 config systems | Spans all phases |

**On versioning:** There is no config versioning or rollback system in scope. If a pre-compilation approach is adopted (see Compilation Timing discussion), the system will naturally have a "current" and "previous" compiled state per device. Whether to retain previous versions — and how many — can be revisited once the compilation architecture is decided. For now, the recovery path is fix-and-push.

---

## Outstanding Questions

### Device & Technical Questions (Need Adam / InHand)

| # | Question | Priority |
|---|----------|----------|
| 1 | What happens when a device receives a config via RouterApi but the config has an error? Does it reject it, apply it and malfunction, or silently ignore bad parameters? | High — affects pre-push validation requirements |
| 3 | What is the typical check-in interval for devices? How long between check-ins? | High — determines hostname verification SLA |
| 4 | If a device's API access is disabled, what caused it and can it be remotely re-enabled? | High — affects failure handling and retry strategy |
| 6 | What is the device reboot time? How long is the device offline during a config apply? | High — affects push scheduling and acceptable push windows |
| 8 | Are there devices behind firewalls or NAT where inbound connections (push) are impossible? | High — affects on-demand push feasibility and retry strategy |
| 2 | Can we query a device for its currently running configuration? (e.g., via `download.cgi` or `getinfo.cgi`) | Low — useful for future debugging and support troubleshooting |
| 5 | Can a device report its config version or hash during check-in (beyond hostname)? | Low — good to know for future enhancements |
| 7 | Does the InHand API support any kind of config diff or validation endpoint? | Low — could support pre-push validation in the deployment pipeline |

### Business & Process Questions (Need Devon / Adam)

| # | Question | Why It Matters |
|---|----------|---------------|
| 9 | Should config changes be pushed immediately on approval, or should there be a "schedule push" option? | Determines if we need scheduling infrastructure |
| 10 | What's the acceptable window for a device to be running an outdated config? Hours? Days? | Defines our SLA for config delivery |
| 11 | Who should be notified when a push fails? Just admins? The company? | Drives notification design |
| 12 | Should there be a distinction between "critical" config changes (push ASAP) and "routine" changes (push on next check-in)? | Affects push priority and urgency classification |
| 13 | ~~Do we need to support "push to one device" for testing before fleet-wide rollout?~~ | Resolved — on-demand push supports this. Staging is handled by draft/approval in the config builder. |
| 14 | How should the legacy config push coexist with V2 push? Separate mechanisms or unified? | Affects implementation approach during transition |

### Design Questions (Internal)

| # | Question | Options |
|---|----------|---------|
| D1-1 | Where does the manual push trigger live in the UI? | Device page / Push dashboard / Both |
| D1-2 | What is the selection model for on-demand push? | Individual device selection / Scope by config change / Both |
| D1-3 | Should on-demand push show a confirmation before executing? | Always / Only for >N devices / No |
| D1-4 | Should on-demand push be available to all admin roles or restricted? | All admins / Super admins only / Configurable |
| D1-5 | What happens if a device already has a check-in push queued when an on-demand push is triggered? | Replace / Skip / Allow duplicate |
| D2-1 | How long after a push should we wait for hostname verification before flagging as "unverified/stale"? | Based on expected check-in interval (need answer to question #3) |
| D2-2 | Should the push status history retain all attempts or only the latest per config version? | All attempts (full audit trail) / Latest only / Configurable retention |
| D2-3 | Should hostname mismatch auto-trigger a retry push, or just flag for admin review? | Auto-retry / Flag only / Configurable |

### Architecture Questions (Need Dev Team)

| # | Question | Options |
|---|----------|---------|
| A1 | When does compilation happen — at push time, on every config change, or a hybrid? | Compile at push time / Pre-compile on change / Pre-compile parameters + generate .DAT at push |

### Open Decisions

| Decision | Depends On |
|----------|------------|
| Failure Handling | Device behavior questions #1, #4, #8 |
| Compilation Timing | Architecture question A1 — dev team discussion |

---

## Next Steps

1. **Get answers to high-priority device behavior questions (#1, #3, #4, #6, #8)** — Adam is the primary source; some may require InHand input
2. **Review this document with Devon and Adam** — Align on remaining open decisions (failure handling)
3. **Resolve design questions (D1-1 through D2-3)** — Needed before UI design
4. **Decide on failure handling strategy** — Informed by device behavior answers
5. **Prototype the deployment pipeline** — Can be designed in parallel with Phase 1 config builder work
