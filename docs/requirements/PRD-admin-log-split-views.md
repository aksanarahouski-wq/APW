# PRD: Admin Logs — Split User and System Log Views

**Version:** 1.0 | **Date:** 05.03.2026 | **Author:** Aksana Rahouski | **Status:** Draft

---

## Problem

The `/admin/logging` page is flooded with system-generated entries (primarily "Device Configuration Log Id changed" — generated every ~2 hours per active device). The last 100 entries can span only 15 minutes, burying actual user actions. The page is slow to load due to the volume.

## Solution

Replace the single "Browse Logs" view with two separate views:

- **Browse User Logs** — only user-initiated actions (the primary use case for ~90% of log access)
- **Browse System Logs** — only system-generated events (preserved for troubleshooting)

No data is deleted. Classification is applied at query time via WHERE clause filtering on existing `o_logs` fields.

---

## Log Classification Rules

### User Logs (has an authenticated user)

A log entry is classified as a **User Log** when:
- The `user` field contains an authenticated user identity (not NULL/N/A), OR
- The event is an explicitly user-initiated action (device assignment, device creation/deletion, configuration change via UI, payment method change, service plan change, billing status change by a user)

**Rule:** If a log entry has an authenticated user, it goes to User Logs regardless of the field that changed.

### System Logs (no user, system-generated)

A log entry is classified as a **System Log** when:
- `user` is NULL/N/A AND `ip_address` is NULL/N/A, OR
- The event matches a known system pattern:
  - "Device Configuration Log Id changed from X to Y"
  - Device status changed from offline to null (check-in event)
  - Hostname/TID changed via config push
  - Password rotated by system (SysTech devices)
  - Other internal tracking field updates with no user action

**Fallback rule:** Any entry that doesn't clearly match User Logs defaults to System Logs. Every entry must appear in exactly one view.

### Important Distinction

`configuration_id` changes made by a user via UI → **User Logs**
`device_configuration_log_id` changes made by system config push → **System Logs**

---

## Scope

### In Scope

- Two new navigation links replacing "Browse Logs": "Browse User Logs" and "Browse System Logs"
- Query-level filtering on `o_logs` to separate views
- Both views keep the same columns, filters (date, company, device, log type), pagination, and sort order as current page
- Classification ruleset defined in a maintainable, centralized location (PHP config array)

### Out of Scope

- Deleting/archiving historical entries
- Changes to `o_logs` table schema
- Adding new log types or changing what gets logged
- Per-device log filtering (separate feature)

---

## Technical Notes

- Read from existing `o_logs` table — no schema changes
- Classification logic lives in the CakePHP admin logging controller/query layer, not in the `orases/logs` package
- Filter at DB query level (WHERE clause), not in PHP after loading
- Existing RBAC permissions apply to both views — no permission changes needed
- Check `watm/config/permissions.php` for new route entries

### Existing `o_logs` Fields Used for Classification

| Field | Classification Use |
|-------|-------------------|
| `user` | NULL/N/A → candidate for System Log |
| `ip_address` | NULL/N/A → corroborates system origin |
| `log_type` | Category (e.g., DEVICE MODIFICATION) |
| `message` | Pattern matching for specific event types |

---

## Acceptance Criteria

- [ ] "Browse User Logs" and "Browse System Logs" links appear in admin logging navigation
- [ ] User Logs view contains zero system-generated entries (User = N/A config log changes, check-in state transitions, etc.)
- [ ] System Logs view contains all system-generated entries
- [ ] Sum of entries in both views equals total entries in `o_logs` for any given time range (no entries lost)
- [ ] Existing filters work independently in both views
- [ ] User Logs page loads faster than current unified view for the same time range
- [ ] RBAC unchanged — same access controls apply

---

## Open Questions

1. **Categorization completeness:** Are there system-generated log types beyond the identified list? (e.g., billing-cycle-generated, notification-triggered entries). **Recommendation:** Launch with known list, default unknowns to System Logs, refine after observation.

2. **"Billing active changed" / "pending changed" classification:** These may be user-triggered or system-triggered depending on context. **Recommendation:** Route based on whether `user` field is populated — user present → User Logs, N/A → System Logs.

3. **Navigation placement:** Do the two new links replace "Browse Logs" entirely, or appear as sub-items? Decision needed before UI implementation.

---

## References

- [Decision Summary](../../TrelloBacklog/AdminLogs/Decision_Summary.md)
- [Client Request Analysis](../../TrelloBacklog/AdminLogs/ClientRequest_ConfigLogSpam_Analysis.md)
- [Meeting Transcript](../../TrelloBacklog/AdminLogs/meeting.md) (March 5, 2026)
- **Fallback option:** Add `device_configuration_log_id` to `$dirtyFilterArray` in `DevicesTable.php:1476` — one-line change that delivers ~80% of the benefit if split views are deferred.
