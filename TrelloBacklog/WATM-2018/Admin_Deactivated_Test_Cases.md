# Test Cases: "Admin Deactivated" Device Status

**Ticket:** WATM-2018
**Requirements:** [Confluence PRD](https://orases.atlassian.net/wiki/spaces/WATM/pages/2993324038/Admin+Deactivated+Device+Status)
**Branch:** `stoneM_watm-2018_admin_deactivated`
**Date:** 2026-04-28

---

## Prerequisites

- Access to a WATM admin account (Orases/system-level)
- Access to a customer account (company-level user)
- At least 3 test devices in "Active" status across different carriers (Verizon, AT&T, T-Mobile if possible)
- At least 1 device in each existing status: Active, Deactivated, Admin Suspension, Data Suspension, Customer Suspension
- A test company that is NOT deactivated (for email tests)

---

## 1. Database / Status Setup

### TC-1.1: Admin Deactivated status exists in device_statuses table
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Query `device_statuses` table | "Admin Deactivated" record exists |
| 2 | Check `company_can_modify` field | Value is `0` (false) |

---

## 2. Single Device Status Change (Admin)

### TC-2.1: Admin can set a device to "Admin Deactivated" from Active
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Log in as WATM admin | Admin dashboard loads |
| 2 | Navigate to an Active device's edit page | Device edit form displays |
| 3 | Verify "Admin Deactivated" appears in the status dropdown | Status is listed |
| 4 | Select "Admin Deactivated" | A confirmation button appears (replacing the normal Save button) |
| 5 | Click the confirmation button to save | Device saves successfully, redirects to device view |
| 6 | Verify the device status on the view page | Status shows "Admin Deactivated" |

### TC-2.2: Admin can set a device to "Admin Deactivated" from Deactivated
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to a Deactivated device's edit page | Edit form loads |
| 2 | Verify "Admin Deactivated" is in the dropdown | Status is listed |
| 3 | Select "Admin Deactivated" and save | Device saves successfully |
| 4 | Verify device status | Shows "Admin Deactivated" |

### TC-2.3: Admin can transition from "Admin Deactivated" to "Active"
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to an Admin Deactivated device's edit page | Edit form loads |
| 2 | Verify "Active" is in the dropdown | Yes, it is listed |
| 3 | Select "Active" and save | Device saves successfully |
| 4 | Verify device status | Shows "Active" |

### TC-2.4: Admin can transition from "Admin Deactivated" to "Deactivated"
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to an Admin Deactivated device's edit page | Edit form loads |
| 2 | Select "Deactivated" and save (confirm) | Device saves successfully |
| 3 | Verify device status | Shows "Deactivated" |

### TC-2.5: Admin CANNOT transition from "Admin Deactivated" to suspension statuses
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to an Admin Deactivated device's edit page | Edit form loads |
| 2 | Inspect the status dropdown options | "Admin Suspension", "Customer Suspension", and "Data Suspension" are NOT listed |
| 3 | Verify only Active and Deactivated (and Admin Deactivated) are available | Correct |

### TC-2.6: IP address is cleared when setting to Admin Deactivated
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Note the IP address of an Active device with an IP assigned | Record the IP |
| 2 | Change the device status to "Admin Deactivated" | Save succeeds |
| 3 | Check the device's IP field | IP is cleared (null/empty) |

### TC-2.7: charge_for_current_cycle behavior on Admin Deactivated
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Set a device to "Admin Deactivated" | Save succeeds |
| 2 | Check `charge_for_current_cycle` in the database | Value is allowed to be `0` (the beforeSave logic does NOT force it to `1` for deactivated statuses) |

---

## 3. Single Device Status Change (Customer)

### TC-3.1: Customer does NOT see "Admin Deactivated" in status dropdown
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Log in as a customer role user | Customer dashboard loads |
| 2 | Navigate to an Active device's edit page | Edit form displays |
| 3 | Check the status dropdown options | "Admin Deactivated" is NOT in the list |

### TC-3.2: Customer CANNOT modify a device in "Admin Deactivated" status
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | As admin, set a device to "Admin Deactivated" | Device is Admin Deactivated |
| 2 | Log in as customer who owns this device | Customer dashboard loads |
| 3 | Navigate to that device's edit page | Edit form loads |
| 4 | Check the status dropdown | Dropdown is DISABLED (greyed out, non-interactive) |
| 5 | Verify the customer cannot change the status | No status change possible |

### TC-3.3: Customer can still submit device removal request for Admin Deactivated device
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to an Admin Deactivated device (as customer) | Device page loads |
| 2 | Attempt to submit a device removal request | Request is allowed and submits successfully |

---

## 4. Bulk Status Change (Admin)

### TC-4.1: Admin can bulk change devices to "Admin Deactivated"
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Log in as WATM admin | Admin dashboard loads |
| 2 | Navigate to Bulk Actions page | Bulk actions form loads |
| 3 | Verify "Admin Deactivated" is in the bulk status dropdown | Status is listed |
| 4 | Select "Admin Deactivated" as the target status | Selection accepted |
| 5 | Enter serial numbers of 2+ Active devices | Serial numbers entered |
| 6 | Click Apply | A confirmation modal appears (deactivation warning) |
| 7 | Confirm in the modal | Devices are updated to Admin Deactivated |
| 8 | Verify each device status | All show "Admin Deactivated" |

### TC-4.2: Customer CANNOT bulk change devices to "Admin Deactivated"
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Log in as customer | Customer dashboard loads |
| 2 | Navigate to Bulk Actions page | Bulk actions form loads |
| 3 | Check the bulk status dropdown | "Admin Deactivated" is NOT listed (customer gets filtered status list) |

### TC-4.3: Customer cannot bulk change Admin Deactivated devices to any status except Deactivated
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | As admin, set a device to "Admin Deactivated" | Done |
| 2 | Log in as customer | Customer dashboard |
| 3 | Go to Bulk Actions, enter that device's serial number | Serial entered |
| 4 | Select "Active" as target status and apply | Error: device cannot be updated (added to `cantBeUpdated` list) |
| 5 | Select "Deactivated" as target status and apply | Device is updated to Deactivated (allowed exception) |

### TC-4.4: Bulk deactivation skips devices with pending Verizon status changes
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Find a device with `is_vz_pending = 1` | Device identified |
| 2 | Include it in a bulk "Admin Deactivated" action | Error flash: "This device is still pending a status change: [serial]" |
| 3 | No devices are updated in that batch | Correct — entire batch blocked when any device is pending |

---

## 5. Email Notifications

### TC-5.1: Single device deactivation sends email notification
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Set a single device to "Admin Deactivated" (from Active) | Device saves |
| 2 | Check MailHog / email inbox for the company's super users | Deactivation notice email received |
| 3 | Verify email content: device serial number, deactivation timestamp, user who deactivated | All present |
| 4 | Verify email contains "Device Deactivated" header (red) | Correct |
| 5 | Verify "Reactivate Device" button/link is in the email | Present, links to device edit page |
| 6 | Verify billing note: "You will not be billed for this device going forward" | Present |

### TC-5.2: Bulk deactivation sends mass deactivation email
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Bulk change 3 devices to "Admin Deactivated" | Devices update |
| 2 | Check email for company super users | Mass deactivation notice received |
| 3 | Verify email lists all deactivated device serial numbers | All 3 listed |
| 4 | Verify device count in email | Shows "3" devices |
| 5 | Verify "View Devices" button in email | Present |

### TC-5.3: Deactivation email NOT sent to deactivated company
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Find a device belonging to a deactivated company | Device identified |
| 2 | Set it to "Admin Deactivated" | Device saves |
| 3 | Check emails | No deactivation notice email sent for this company |

### TC-5.4: White-label branding in emails
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Set a device to "Admin Deactivated" for a company with a parent company (white-label) | Device saves |
| 2 | Check the deactivation email | Email uses parent company branding (logo, name) |

---

## 6. Carrier API Integration

### TC-6.1: Verizon device — Admin Deactivated triggers SIM deactivation
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Set a Verizon device to "Admin Deactivated" | Device saves |
| 2 | Verify carrier API call was made | Verizon `deactivateDevice()` API called |
| 3 | Check `is_vz_pending` flag | Set to `1` (pending carrier confirmation) |
| 4 | After carrier confirms | `is_vz_pending` resets, SIM status shows deactivated |

### TC-6.2: AT&T device — Admin Deactivated triggers SIM suspension
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Set an AT&T device to "Admin Deactivated" | Device saves |
| 2 | Verify carrier behavior | AT&T SIM suspension API called (same as standard Deactivated) |

### TC-6.3: T-Mobile device — Admin Deactivated triggers SIM deactivation
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Set a T-Mobile device to "Admin Deactivated" | Device saves |
| 2 | Verify carrier behavior | T-Mobile deactivation API called |

---

## 7. Dashboard / Reporting

### TC-7.1: Admin Deactivated has its own count on the dashboard chart
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Ensure at least 2 devices are in "Admin Deactivated" status | Devices set |
| 2 | Navigate to the dashboard | Dashboard loads with "Devices By Status" chart |
| 3 | Verify "Admin Deactivated" appears as a separate segment | Yes, shown in dark red color |
| 4 | Verify the count matches expected number | Count is correct |

### TC-7.2: Admin Deactivated devices are included in the total deactivated count
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Note: 2 devices are "Admin Deactivated", 3 devices are "Deactivated" | Baseline |
| 2 | Check the "Deactivated" segment on the chart | Shows 3 (NOT 5 — the chart subtracts admin_deactivated from deactivated_count for display) |
| 3 | Check "Admin Deactivated" segment | Shows 2 |
| 4 | Combined total of Deactivated + Admin Deactivated | Equals 5 |

> **Note from code:** The dashboard query counts `deactivated_count` as `Deactivated + Admin Deactivated` combined, but the chart template subtracts `admin_deactivated_count` from `deactivated_count` for display, so they appear as separate slices that sum to the total.

### TC-7.3: Dashboard chart colors
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | View the "Devices By Status" doughnut chart | Chart renders |
| 2 | Verify color coding | Active=green, Deactivated=red, Admin Deactivated=darkred, Admin Suspension=gray, Data Suspension=orange, Customer Suspension=yellow |

---

## 8. Billing

### TC-8.1: Admin Deactivated device can be exempted from current cycle billing
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Set a device to "Admin Deactivated" with `charge_for_current_cycle = 0` | Device saves |
| 2 | Verify the device is excluded from the current billing cycle | Device not billed |

### TC-8.2: Admin Deactivated device included in billing cleanup command
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Run `CleanUpDeactivatedDevicesForCurrentCycleCommand` (readonly mode) | Command runs |
| 2 | Verify Admin Deactivated devices are included in the scope | Command checks both "Deactivated" and "Admin Deactivated" statuses |
| 3 | Verify cleanup report email lists Admin Deactivated devices | Included in report |

---

## 9. Company Deactivation

### TC-9.1: Company deactivation job uses "Deactivated" (NOT "Admin Deactivated")
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Deactivate/suspend a company | Company status changes |
| 2 | Verify the `DeactivateCompanyDevicesJob` runs | Job processes |
| 3 | Check the status of devices that were Active | Devices set to "Deactivated" (not "Admin Deactivated") |

> **Note:** This is expected behavior — company-level deactivation is a different flow from admin-targeted device deactivation.

---

## 10. UI / JavaScript Behavior

### TC-10.1: Confirmation button on single device edit when selecting Admin Deactivated
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Open an Active device's edit page | Normal "Save" button visible |
| 2 | Change status dropdown to "Admin Deactivated" | "Save" button hides, "Confirm" button appears |
| 3 | Change status back to "Active" | "Confirm" button hides, "Save" button reappears |

### TC-10.2: Confirmation button does NOT appear when already deactivated
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Open a Deactivated device's edit page | Normal "Save" button visible |
| 2 | Change status to "Admin Deactivated" | "Save" button remains (no confirm toggle — device is already in a deactivated status) |

### TC-10.3: Bulk action confirmation modal for Admin Deactivated
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | On Bulk Actions page, select "Admin Deactivated" status | Status selected |
| 2 | Enter serial numbers and click Apply | Confirmation modal appears |
| 3 | Click OK in modal | Form submits |

### TC-10.4: Bulk action with no serial numbers does NOT show modal
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Select "Admin Deactivated" but leave serial numbers empty | Status selected, no serials |
| 2 | Click Apply | Form submits directly (no modal) — server-side validation catches the error |

### TC-10.5: Status dropdown disabled for customer viewing Admin Deactivated device
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Log in as customer, navigate to Admin Deactivated device edit page | Page loads |
| 2 | Inspect status dropdown | Dropdown is disabled (non-interactive) |

### TC-10.6: Status dropdown disabled when device has pending Verizon change
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to device with `is_vz_pending = 1` | Edit page loads |
| 2 | Inspect status dropdown | Dropdown is disabled regardless of user role |

---

## 11. Device View Page

### TC-11.1: Admin Deactivated status displays on device view page
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Navigate to a device in "Admin Deactivated" status | View page loads |
| 2 | Verify the status is clearly displayed | "Admin Deactivated" shown |

---

## 12. Edge Cases & Negative Tests

### TC-12.1: Deactivated company device cannot be reactivated from Admin Deactivated
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Find a device in "Admin Deactivated" belonging to a deactivated company | Device identified |
| 2 | Attempt to change status to "Active" | "Active" is NOT available in dropdown (excluded when company is deactivated) |
| 3 | Only "Deactivated" is available as a transition | Correct |

### TC-12.2: Bulk action with mix of updatable and non-updatable devices
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Enter serial numbers for: 1 Active device + 1 device with `is_vz_pending` | Serials entered |
| 2 | Bulk change to "Admin Deactivated" | Error for pending device; entire batch blocked |

### TC-12.3: Customer bulk action on mixed devices (some Admin Deactivated)
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | As customer, enter serials for: 1 Active device + 1 Admin Deactivated device | Serials entered |
| 2 | Select "Active" as target status | Apply |
| 3 | Active device updates, Admin Deactivated device is blocked | Flash error lists the blocked serial number, successful device updates |

### TC-12.4: Serial number not found in bulk action
| Step | Action | Expected Result |
|------|--------|----------------|
| 1 | Enter a mix of valid serials + 1 invalid serial | Serials entered |
| 2 | Bulk change to "Admin Deactivated" | Valid devices update; flash error lists the unfound serial |

---

## Summary Checklist

| Area | # Tests | Priority |
|------|---------|----------|
| Database setup | 1 | P0 |
| Single device - Admin | 7 | P0 |
| Single device - Customer | 3 | P0 |
| Bulk actions - Admin | 4 | P0 |
| Email notifications | 4 | P1 |
| Carrier API | 3 | P1 |
| Dashboard/Reporting | 3 | P1 |
| Billing | 2 | P1 |
| Company deactivation | 1 | P2 |
| UI/JavaScript | 6 | P1 |
| Device view | 1 | P2 |
| Edge cases | 4 | P2 |
| **Total** | **39** | |
