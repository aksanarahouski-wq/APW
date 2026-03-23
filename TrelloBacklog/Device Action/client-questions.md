# Client Questions: "Admin Deactivated" Status

## Background from Research

The current device primary statuses are:

| # | Status | Company Can Modify | Carrier API Action |
|---|--------|--------------------|--------------------|
| 1 | Active | Yes | Activate/Resume SIMs |
| 2 | Deactivated | Yes | Deactivate SIMs (Verizon), Suspend (AT&T), Deactivate (T-Mobile) |
| 3 | Data Suspension | Yes | Suspend SIMs |
| 4 | Admin Suspension | No | Suspend SIMs |
| 5 | Customer Suspension | Yes | Suspend SIMs |

### What "Deactivated" Currently Does

When a device is set to **Deactivated**:
1. **Carrier SIM deactivation** — Verizon SIM is deactivated via API; AT&T SIM is suspended; T-Mobile SIM is deactivated (T-Mobile has no suspend concept)
2. **IP cleared** — The device's IP address is removed
3. **Billing exemption** — The `charge_for_current_cycle` flag is NOT forced to true (unlike all other statuses), meaning the device can avoid charges for the current cycle
4. **Deactivation notice email** — A notification email is sent to the company (unless the company itself is deactivated)
5. **Mass deactivation email** — Bulk deactivations trigger a separate mass notification mailer
6. **Customers can self-deactivate** — `company_can_modify = 1` means customers can set this status themselves

### Existing Precedent: "Admin Suspension"

The system already has **Admin Suspension** (`company_can_modify = 0`), which:
- Customers cannot change or remove
- Suspends (but does not deactivate) SIMs at the carrier level
- Customer role users see the status dropdown as disabled when a device is in Admin Suspension

---

## Questions for Client

### 1. How is "Admin Deactivated" different from "Deactivated"?

The existing "Deactivated" status deactivates the device's SIMs at the carrier level and customers can set/unset it themselves. **What specific problem does "Admin Deactivated" solve?**

Possible goals:
- **a)** Prevent customers from reactivating a device that was deactivated by an admin (audit/control reason)
- **b)** A more permanent deactivation vs. "Admin Suspension" which only suspends SIMs
- **c)** Tracking/reporting — knowing whether deactivation was initiated by admin vs. customer
- **d)** Something else?

### 2. Carrier API Behavior

Should "Admin Deactivated" trigger the **same carrier API calls** as "Deactivated" (i.e., actually deactivate the SIMs at Verizon/AT&T/T-Mobile)?

Or should it behave more like "Admin Suspension" (suspend but not fully deactivate)?

> This matters because deactivation at some carriers (especially Verizon) can be harder to reverse than suspension.

### 3. Who Can Set "Admin Deactivated"?

- Only WATM admins (Orases/system-level)?
- Company admins (parent company level)?
- Any admin-role user?

### 4. Who Can Remove / Change "Admin Deactivated"?

The request says customers cannot modify it. Can the same admins who set it also remove it, or does it require a specific role/permission?

### 5. What Statuses Can "Admin Deactivated" Transition To?

- Can an admin move a device from "Admin Deactivated" directly back to "Active"?
- Should there be any restrictions on what status it can transition to?

### 6. Billing Behavior

Currently "Deactivated" exempts the device from `charge_for_current_cycle`. Should "Admin Deactivated" also exempt billing, or should the device still be charged?

### 7. Deactivation Notice Emails

Should the system send the same deactivation notice email to the company when a device is set to "Admin Deactivated"? Or should it be suppressed/different since the customer didn't initiate it?

### 8. Bulk Actions Support

Should "Admin Deactivated" be available as an option in the bulk status change actions?

### 9. Device Removal Requests

If a customer submits a device removal request for a device that is "Admin Deactivated", should it be blocked or allowed?

### 10. Reporting / Dashboard

The dashboard currently tracks `deactivated_count`. Should "Admin Deactivated" devices be:
- Counted separately?
- Included in the existing deactivated count?
- Both (separate + total)?
