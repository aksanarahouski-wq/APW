# WATM-2038: Additional Status Configuration per Device Model
## Feature Summary

**Date:** April 7, 2026
**Ticket:** WATM-2038
**Status:** Requirements Finalized

---

## What We're Building

We're adding the ability for admins to control which **Additional Statuses** (e.g., Misplaced, Offline) are available for each device model. This is configured through the existing Device Model settings page in the admin panel — no code changes or deployments needed when you want to adjust status availability for any model going forward.

**Immediate use case:** Remove "Offline" from the Additional Status options for SIM/Systech M5 and M5-302 models.

---

## How It Works

### For Admins

1. **Navigate** to the Device Model edit page (e.g., Admin > Device Models > M5 > Edit)
2. A new **"Available Additional Statuses"** section appears with checkboxes for each status
3. **Uncheck** any status you want to remove for that model (e.g., uncheck "Offline")
4. **Save** — if any devices of that model currently have the removed status, you'll see a confirmation showing how many devices will be affected
5. **Confirm** — the configuration saves and affected devices have their Additional Status cleared automatically

You can repeat this for any model at any time. If you need to re-enable a status later, simply check it again and save.

### For Customer Portal Users

- Customer portal users will **only see the statuses that are enabled** for a device's model when editing a device
- No configuration access — status management is admin-only

### What Stays the Same

- **All existing models continue to work exactly as they do today** until you explicitly change their configuration
- **Primary Status** (Active, Deactivated, etc.) is not affected — this only applies to Additional Status
- **Bulk status updates** are not affected

---

## Decisions Confirmed

| Topic | Decision |
|-------|----------|
| Existing devices with a removed status | Automatically cleared when the admin saves the model configuration |
| Who sees the restriction | Both admin panel and customer portal |
| Who can configure it | Admin only |
| Bulk status updates | Not affected |

---

## What to Expect

- **After deployment**, no behavior changes — everything works as it does today
- **To apply restrictions**, an admin goes to the Device Model edit page and unchecks the statuses that shouldn't be available
- **Changes take effect immediately** across admin panel, customer portal, and the API
- **Future models or status changes** can be managed the same way without involving the development team

---

## Scope Boundaries

**Included:**
- Additional Status configuration per device model
- Enforcement on admin panel, customer portal, and API
- Automatic cleanup of affected devices

**Not included at this time:**
- Primary Status restrictions per model
- Manufacturer-level restrictions (configuration is per model)
- Bulk status update restrictions
