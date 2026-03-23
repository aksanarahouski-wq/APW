# PRD: "Admin Deactivated" Device Status

**Version:** 1.0 | **Date:** 16.03.2026 | **Author:** Aksana Rahouski | **Status:** Draft

---

## Problem

When WATM admins need to deactivate a specific device on a customer's account (e.g., due to non-payment on a secondary payment method), the only option today is "Deactivated" — which the customer can reverse themselves. There is no way to deactivate a device that the customer/parent/sub-company cannot undo.

"Admin Suspension" exists but only suspends SIMs (doesn't fully deactivate at the carrier level), and doesn't follow the same billing/deactivation logic.

## Solution

Add a new device primary status: **"Admin Deactivated"**

- Behaves like "Deactivated" (same carrier API calls, same billing logic, same emails)
- Cannot be modified by any customer role — only WATM admins (Orases/system-level)
- Allows the customer account to remain active while a specific device is admin-controlled

---

## Requirements

### New Status: "Admin Deactivated"

| Property | Value |
|----------|-------|
| Status name | Admin Deactivated |
| `company_can_modify` | `0` (false) |
| Carrier API behavior | Same as "Deactivated" — Verizon: deactivate SIM, AT&T: suspend SIM, T-Mobile: deactivate SIM |
| IP address | Cleared (same as Deactivated) |
| Billing | Same as "Deactivated" — device can be exempted from current cycle charges, with ability to waive billing |
| Deactivation notice email | Yes — same email as "Deactivated", sent to all parties |
| Mass deactivation email | Yes — included in bulk deactivation mailer |

### Access Control

| Action | Who Can Do It |
|--------|---------------|
| Set a device to "Admin Deactivated" | WATM admins only (Orases/system-level) |
| Remove / change "Admin Deactivated" | WATM admins only (Orases/system-level) |
| Customer/company users | Cannot see "Admin Deactivated" in status dropdown; dropdown is disabled when device is in this status (same pattern as "Admin Suspension") |

### Status Transitions

**From "Admin Deactivated", a WATM admin can transition to:**
- Active — Yes
- Deactivated — Yes
- Admin Suspension — No (cannot change to any suspend status)
- Data Suspension — No
- Customer Suspension — No

**To "Admin Deactivated" from any other status:**
- Only WATM admins can set this status
- Customer role users cannot transition any device into "Admin Deactivated"

### Bulk Actions

- "Admin Deactivated" is available as an option in bulk status change actions (admin only)

### Device Removal Requests

- Customers can still submit device removal requests for devices in "Admin Deactivated" status (allowed, not blocked)

### Dashboard / Reporting

- "Admin Deactivated" devices are counted **separately** on the dashboard (new count)
- "Admin Deactivated" devices are **also included** in the existing total deactivated count
- Both: separate breakdown + included in aggregate

---

## Scope

### In Scope

- New `device_statuses` record: "Admin Deactivated" with `company_can_modify = 0`
- Carrier API integration: same deactivation logic as "Deactivated"
- Billing logic: same as "Deactivated" (exemption from current cycle, waive billing option)
- Email notifications: same deactivation notice emails
- UI: status dropdown disabled for customer roles when device is Admin Deactivated
- Bulk status change: include "Admin Deactivated" as option for admin users
- Status transition rules: block transitions to any suspend status
- Dashboard: add separate "Admin Deactivated" count, include in total deactivated count

### Out of Scope

- Changes to existing "Deactivated" or "Admin Suspension" behavior
- Changes to device removal request workflow (already allowed)
- New email templates specific to "Admin Deactivated" (uses same as "Deactivated")

---

## Technical Notes

### Database

- Add new record to `device_statuses` table: name = "Admin Deactivated", `company_can_modify = 0`
- Follow the same pattern as "Admin Suspension" for the `company_can_modify` flag

### Carrier API

- Route through the same carrier API calls as "Deactivated" in `DevicesTable::afterSave()`:
  - Verizon: deactivate SIM
  - AT&T: suspend SIM
  - T-Mobile: deactivate SIM
- IP address cleared on status change (same as Deactivated)

### Billing

- `charge_for_current_cycle` behavior: same as "Deactivated" — not forced to `true` when status is deactivated
- Waive billing option available (same logic as standard deactivation)

### UI Changes

- Status dropdown: include "Admin Deactivated" for WATM admin role users
- Status dropdown: disable/hide for customer role users when device is in this status (same as Admin Suspension pattern)
- Bulk actions: add to status change options (admin only)

### Status Transition Logic

Add validation in `DevicesTable::beforeSave()` or status change logic:
- If current status is "Admin Deactivated", block transitions to: Data Suspension, Admin Suspension, Customer Suspension
- Allow transitions to: Active, Deactivated

### Dashboard

- Add `admin_deactivated_count` to dashboard query
- Include admin deactivated devices in existing `deactivated_count` aggregate

---

## Client Responses Reference

All answers sourced from inline comments on Confluence page: [Goals & Questions: "Admin Deactivated" Device Status](https://orases.atlassian.net/wiki/spaces/WATM/pages/2982346754)

| # | Question | Client Answer |
|---|----------|---------------|
| 1 | How is "Admin Deactivated" different? | Keeps customer account active while disabling a unit on a secondary payment method that the customer/parent/sub cannot control |
| 2 | Same carrier API calls as Deactivated? | Yes |
| 3 | Who can set it? | Only WATM admins (Orases/system-level). Not company admins. Any WATM admin. |
| 4 | Who can remove/change it? | Only WATM admins/Orases/system level |
| 5a | Can transition back to Active? | Yes |
| 5b | Transition restrictions? | Cannot change from Admin Deactivated to any type of suspend status |
| 6 | Billing behavior? | Logic works just like a typical deactivation with the ability to waive billing as well |
| 7 | Deactivation notice emails? | Yes, all parties should receive email regarding unit deactivation |
| 8 | Bulk actions? | Yes |
| 9 | Device removal requests? | Yes (allowed) |
| 10 | Dashboard reporting? | Both — counted separately and included in existing deactivated total |
