# WATM-2179: Improve "Update Devices" Bulk Import — Partial Update Support

**Status:** Draft
**Priority:** Normal
**Jira:** [WATM-2179](https://orases.atlassian.net/browse/WATM-2179)
**Date:** June 28, 2026
**Source:** Client request (Devon D'Andrea) — discussed in APW Check-in, June 23, 2026

---

## Overview

The "Update Devices" bulk import feature allows admins to upload a spreadsheet to update multiple devices at once. Today, the import uses a fixed 22-column format where every field is always written to the device record — including blank fields, which overwrite existing data with null. This means that to safely change a single field (e.g., toggling AT&T SIM from yes to no), the user must populate every other column with the device's current data to avoid wiping it out.

Devon's example: he needed to switch 400 devices from dual carrier to single carrier Verizon. The only thing he needed to change was the AT&T SIM active flag. But to do that safely, he had to pull in location name, TID, address, and every other field via VLOOKUPs from a fresh device export — a disproportionate amount of work for a simple change.

This enhancement adds a **partial update** mode where only fields with values are updated, leaving blank fields untouched.

---

## Current Behavior

### Workflow

1. Admin navigates to Devices > Update Devices
2. Page shows a sample data row (from a random device) illustrating the 22-column format
3. Admin uploads an Excel (.xlsx) or CSV file with header row + up to 1000 device rows
4. File is stored and set to "pending" status
5. An authorized user approves the bulk update
6. On approval, each row is processed: device is looked up by serial number, fields are patched, and the device is saved

### The 22 Columns (Fixed Format)

| # | Column | Blank Behavior |
|---|--------|----------------|
| 1 | Serial Number | Required (row skipped if blank) |
| 2 | IP Address | **Wipes to null** |
| 3 | Service Plan | Preserves existing |
| 4 | Primary Status | Preserves existing |
| 5 | Additional Status | Can clear (sets null) |
| 6 | Manufacturer Serial Number | **Wipes to null** |
| 7 | Manufacturer | Preserves existing |
| 8 | Model | Preserves existing |
| 9 | IMEI/ESN | **Wipes to null** |
| 10 | Verizon SIM Number | **Wipes to null** |
| 11 | Verizon Active | Preserves existing |
| 12 | AT&T SIM Number | **Wipes to null** |
| 13 | AT&T Active | Preserves existing |
| 14 | T-Mobile SIM Number | **Wipes to null** |
| 15 | T-Mobile Active | Preserves existing |
| 16 | Customer Warranty Date | **Wipes to null** |
| 17 | Location Name | **Wipes to null** |
| 18 | TID | **Wipes to null** |
| 19 | Address | **Wipes to null** |
| 20 | City | **Wipes to null** |
| 21 | State | Preserves existing |
| 22 | Zip | **Wipes to null** |

**12 of 22 fields wipe existing data when left blank.** This is the core problem — the user must populate those fields with current values even when they don't intend to change them.

### Why This Is Painful

- Must start from a fresh device export to get current values
- Export address format (single combined field) doesn't match import format (4 separate columns) — requires manual parsing
- If the export is even slightly stale, re-importing can overwrite changes made by sub-companies or other users in the interim
- VLOOKUPs and data manipulation required just to safely change one field across many devices

---

## Scope

### In Scope

1. **Add a partial update mode** — a new import option where only fields that contain a value are updated on the device; blank/empty fields are skipped and existing data is preserved
2. **Retain the existing full update mode** — current behavior remains available for users who want to overwrite all fields

### Out of Scope

- **Export/import address format mismatch** — the export combines address into a single field while import requires 4 separate columns. This is a known inconvenience but is not being addressed in this ticket.
- **Changes to the import file column structure** — both modes use the same 22-column format
- **New UI for selecting which fields to update** — the partial mode simply skips blank fields; no per-field selection UI

---

## Functional Requirements

### FR-1: Update Mode Selection

- The Update Devices page must offer a choice between two modes: **Full Update** and **Partial Update**
- This selection should be made at upload time (before the file is submitted)
- The selected mode must be stored on the bulk update record so it is known at approval/processing time
- Default: Full Update (preserves current behavior)

### FR-2: Full Update Mode (Existing Behavior)

- Behaves exactly as the system works today
- All 22 columns are processed; blank fields overwrite existing values with null
- No changes to current logic

### FR-3: Partial Update Mode (New Behavior)

- Only fields that contain a non-empty value in the CSV are updated on the device
- Blank/empty fields are skipped entirely — existing device data is preserved
- Serial Number remains required (used to identify the device)
- This applies to all fields in the `$deviceData` array that are currently set unconditionally (IP, manufacturer serial, IMEI, SIM numbers, warranty date, location name, TID, address, city, zip)
- Fields that already have conditional/preserve logic (service plan, state, primary status, manufacturer, model, SIM active status) continue to behave as they do today
- Validation rules still apply to any fields that are provided

### FR-4: Approval Flow — Mode Visibility

- The approval workflow remains the same: file is uploaded, set to pending, then approved
- The **Pending Device Updates** table on the Update Devices page must display the update mode (Full or Partial) for each pending upload, so reviewers know what type of update they are approving before clicking through
- The individual bulk update approval/view page should also display the mode
- No changes to approval permissions or process

### FR-5: Instructions Updated

- The Update Devices page instructions should explain the difference between Full and Partial update modes
- Partial mode description: "Only fields with values will be updated. Leave fields blank to keep existing data unchanged."
- Full mode description: "All fields will be updated. Blank fields will clear existing data."

---

## Current Implementation (Technical Context)

### Key Files

| Layer | File | Purpose |
|-------|------|---------|
| **Upload Page** | `plugins/Devices/templates/Admin/Devices/update.php` | Upload form, sample data, instructions |
| **Upload Controller** | `plugins/Devices/src/Controller/Admin/DevicesController.php` (line ~2461) | `update()` — file upload, creates pending record |
| **Processing Controller** | `plugins/Devices/src/Controller/Admin/DeviceBulkUpdatesController.php` (line ~69) | `approve()` — processes each row on approval |
| **Bulk Update Model** | `plugins/Devices/src/Model/Table/DeviceBulkUpdatesTable.php` | Bulk update record management |
| **Device Model** | `plugins/Devices/src/Model/Table/DevicesTable.php` | Device validation, `beforeMarshal` (config assignment) |
| **Approval View** | `plugins/Devices/templates/Admin/DeviceBulkUpdates/view.php` | Approval page UI |

### Where the Wipe Happens

In `DeviceBulkUpdatesController::approve()`, lines 264-287, the `$deviceData` array is built with **all fields unconditionally**, including null values from blank CSV cells:

```php
$deviceData = [
    'ip' => $ipAddress,                    // null if blank → wipes
    'manufacturer_serial_number' => $manufacturerSerialNumber, // null if blank → wipes
    'imei_esn' => $imei,                   // null if blank → wipes
    'verizon_sim_number' => $verizonSim,   // null if blank → wipes
    'att_sim_number' => $attSim,           // null if blank → wipes
    'tmo_sim_number' => $tmoSim,           // null if blank → wipes
    'warranty_start_date' => $customerWarrantyDate, // null if blank → wipes
    'location_name' => $locationName,      // null if blank → wipes
    'tid' => $tid,                         // null if blank → wipes
    'address_1' => $address,               // null if blank → wipes
    'city' => $city,                       // null if blank → wipes
    'zip_code' => $zip,                    // null if blank → wipes
];
```

When `patchEntity()` is called on line 433 with this data, CakePHP marks these fields as dirty and saves null to the database.

### What Needs to Change for Partial Mode

For partial update mode, the fix is to **conditionally include fields** in `$deviceData` — only add a field if its CSV value is non-empty. The fields that already have conditional logic (service plan, state, primary status, manufacturer, model) can stay as-is.

The mode needs to be:
1. Captured on the upload form (`DevicesController::update()`)
2. Stored on the `device_bulk_updates` record (new column, e.g., `update_mode` enum: 'full'/'partial')
3. Read during processing (`DeviceBulkUpdatesController::approve()`) to determine whether to include blank fields

---

## Acceptance Criteria

### AC-1: Mode Selection Available on Upload

**Given** an admin is on the Update Devices page
**When** they prepare to upload a file
**Then** they can choose between "Full Update" and "Partial Update" modes
**And** "Full Update" is selected by default

### AC-2: Partial Update Preserves Blank Fields

**Given** an admin uploads a file in Partial Update mode with 3 devices
**And** only the Serial Number and AT&T Active columns have values (all other columns are blank)
**When** the bulk update is approved
**Then** only the AT&T Active status is updated for those 3 devices
**And** all other fields (IP, location name, TID, address, SIM numbers, etc.) retain their existing values

### AC-3: Partial Update Applies Provided Values

**Given** an admin uploads a file in Partial Update mode
**And** a row has Serial Number, Location Name, and TID filled in (other fields blank)
**When** the bulk update is approved
**Then** the device's Location Name and TID are updated to the new values
**And** all other fields are unchanged

### AC-4: Full Update Behaves as Today

**Given** an admin uploads a file in Full Update mode
**And** some fields are left blank
**When** the bulk update is approved
**Then** blank fields overwrite existing data with null (current behavior preserved)

### AC-5: Mode Visible on Approval Page

**Given** a bulk update file has been uploaded
**When** an approver views the pending bulk update
**Then** they can see whether it was uploaded as a Full Update or Partial Update

### AC-6: Validation Still Applies

**Given** an admin uploads a file in Partial Update mode
**And** a provided field has an invalid value (e.g., non-existent manufacturer name)
**When** the bulk update is approved
**Then** the device is skipped with an appropriate error message (same as today)

### AC-7: SIM Number Behavior in Partial Mode

**Given** an admin uploads a file in Partial Update mode
**And** SIM number columns are left blank
**When** the bulk update is approved
**Then** existing SIM numbers are preserved (not removed)

**Given** an admin uploads a file in Partial Update mode
**And** a SIM number column has a new value
**When** the bulk update is approved
**Then** the SIM number is updated to the new value

### AC-8: Same File Format

**Given** an admin wants to do a Partial Update
**When** they prepare their spreadsheet
**Then** the same 22-column format is used (no new template needed)
**And** they only need to fill in Serial Number + the fields they want to change
