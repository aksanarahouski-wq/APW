# WATM-2176: Maintain Existing Pricing During Device Transfer

**Status:** Draft
**Priority:** Normal
**Jira:** [WATM-2176](https://orases.atlassian.net/browse/WATM-2176)
**Date:** June 28, 2026
**Source:** Client request (Devon D'Andrea) — discussed in APW Check-in, June 23, 2026

---

## Overview

When transferring devices between customers using the "Assign Devices" feature, there is a "maintain existing service plan" option that preserves the device's service plan. However, this option only preserves the service plan — it does not preserve the flat rate. Devices can be priced via a service plan, a flat rate, or both. If a device has a flat rate but no service plan, using "maintain existing service plan" during transfer silently wipes the flat rate, leaving the device with no pricing at all.

This was discovered when Vince transferred a device that had a $275/month flat rate (no service plan). He clicked "maintain existing service plan" per standard procedure. The system accepted it with a success message, but the device now had no pricing — flat rate wiped, no service plan to maintain.

This enhancement consolidates the option into **"maintain existing pricing"** — a catch-all that preserves whichever pricing model the device has (service plan, flat rate, or both). It also adds validation to prevent transfers that would result in a device with no pricing.

---

## Current Behavior

### How Device Pricing Works

Devices can be priced in two ways (not mutually exclusive):

| Pricing Model | Storage | Description |
|--------------|---------|-------------|
| **Service Plan** | `devices.service_plan_id` (FK) | Tiered, usage-based pricing. Price determined by data usage matched to plan tiers. |
| **Flat Rate** | `devices.flat_rate` (decimal) | Fixed monthly charge regardless of usage. |

- A device can have a service plan only, a flat rate only, or both
- A device with neither is not billable and will be skipped during invoice generation

### The Transfer Flow (Assign Devices)

1. Admin enters serial numbers for devices to transfer
2. Selects target company and payment method
3. Can check **"Maintain existing service plan"** to preserve the current service plan
4. Can manually enter a flat rate value in a separate field
5. Clicks Assign — devices are transferred

### What Goes Wrong

- **"Maintain existing service plan" only preserves `service_plan_id`** — flat rate is not included
- **Flat rate field defaults to blank** on the assign form — if user doesn't explicitly re-enter it, it's wiped to null
- **No validation on the maintain option** — system accepts "maintain existing service plan" even when the device has no service plan, showing a success message while pricing is silently lost
- **Vince's workflow:** He clicks "maintain existing service plan" on every transfer per SOP. He has no way to know a device has a flat rate vs. a service plan without checking each one individually. Transfers happen daily.

---

## Scope

### In Scope

1. **Consolidate "maintain existing service plan" into "maintain existing pricing"** — a single option that preserves whichever pricing the device currently has: service plan, flat rate, or both
2. **Validation** — when "maintain existing pricing" is checked, if the device has neither a service plan nor a flat rate, display an error rather than silently accepting
3. **Rename the option** — from "Maintain existing service plan" to "Maintain existing pricing" to reflect its broader scope

### Out of Scope

- Changes to how flat rate or service plan pricing is calculated during billing
- Changes to the flat rate input field on the assign form (it remains available for manually setting a new flat rate)
- Changes to unassignment behavior (unassignment always clears all billing fields — that's expected)

---

## Functional Requirements

### FR-1: Rename and Consolidate the Maintain Option

- The "Maintain existing service plan" checkbox on the Assign Devices form must be renamed to **"Maintain existing pricing"**
- When checked, the system must preserve **all existing pricing fields** on the device:
  - `service_plan_id` — if the device has one, keep it
  - `flat_rate` — if the device has one, keep it
- This replaces the current behavior where only `service_plan_id` was preserved

### FR-2: Validation — No Pricing Detected

- When "Maintain existing pricing" is checked during a transfer, the system must validate that the device has at least one form of pricing (service plan or flat rate)
- If a device has **neither** a service plan **nor** a flat rate, the system must:
  - Display an error message identifying the device (by serial number)
  - Skip that device (do not transfer it without pricing)
  - Continue processing other devices in the batch

### FR-3: Manual Override Still Works

- If the user does **not** check "Maintain existing pricing" and instead manually enters a service plan and/or flat rate on the form, that manual entry should be used (current behavior)
- The "Maintain existing pricing" option and manual entry remain mutually exclusive for the fields they control (same as today with service plan)

### FR-4: Both Pricing Types Preserved

- If a device has **both** a service plan and a flat rate, checking "Maintain existing pricing" must preserve both
- If a device has **only** a flat rate (no service plan), checking "Maintain existing pricing" must preserve the flat rate
- If a device has **only** a service plan (no flat rate), checking "Maintain existing pricing" must preserve the service plan (current behavior for this case)

---

## Current Implementation (Technical Context)

### Key Files

| Layer | File | Purpose |
|-------|------|---------|
| **Assign Controller** | `plugins/Devices/src/Controller/Admin/DevicesController.php` (line ~1535) | `assign()` — device transfer logic |
| **Assign Template** | `plugins/Devices/templates/Admin/Devices/assign.php` | Transfer form with maintain checkboxes |
| **Device Model** | `plugins/Devices/src/Model/Table/DevicesTable.php` | Validation, beforeSave, afterSave |
| **Device Entity** | `plugins/Devices/src/Model/Entity/Device.php` | `flat_rate` property (line 65) |
| **Billing Usage** | `plugins/Devices/src/Model/Table/CompanyDeviceUsagesTable.php` (line ~341) | `transferCompanyUsage()` |

### Flat Rate Storage

- **Device record:** `devices.flat_rate` (decimal 16,2, nullable)
- **Billing snapshot:** `company_device_usages.flat_rate` (decimal 16,2, nullable) — copied from device when billing record is created
- **Added in:** Migration `20230303151739_AddFlatRateAndFinancingToDevices.php`

### Current "Maintain" Logic

The assign form has a `maintain_existing_service_plan` checkbox. When checked, the controller preserves `service_plan_id` from the existing device entity. The `flat_rate` field is handled separately — it comes from the form input, and if blank, is set to null during the transfer.

### What Needs to Change

1. **Template:** Rename checkbox label from "Maintain existing service plan" to "Maintain existing pricing". Update the form field name if needed (e.g., `maintain_existing_pricing`).
2. **Controller (`assign()`):** When maintain option is checked, preserve both `service_plan_id` AND `flat_rate` from the existing device entity.
3. **Validation:** Before saving, if maintain option is checked and the device has neither `service_plan_id` nor `flat_rate`, add an error and skip the device.

---

## Acceptance Criteria

### AC-1: Option Renamed

**Given** an admin is on the Assign Devices page
**When** the form loads
**Then** the checkbox reads "Maintain existing pricing" (not "Maintain existing service plan")

### AC-2: Flat Rate Preserved During Transfer

**Given** a device has a flat rate of $275 and no service plan
**When** an admin transfers it to another customer with "Maintain existing pricing" checked
**Then** the device's flat rate of $275 is preserved after the transfer
**And** the transfer completes successfully

### AC-3: Service Plan Preserved During Transfer

**Given** a device has a service plan and no flat rate
**When** an admin transfers it to another customer with "Maintain existing pricing" checked
**Then** the device's service plan is preserved after the transfer (current behavior maintained)

### AC-4: Both Preserved During Transfer

**Given** a device has both a service plan and a flat rate of $50
**When** an admin transfers it to another customer with "Maintain existing pricing" checked
**Then** both the service plan and the flat rate are preserved after the transfer

### AC-5: Error When No Pricing Exists

**Given** a device has no service plan and no flat rate
**When** an admin tries to transfer it with "Maintain existing pricing" checked
**Then** an error is displayed: the device serial number is identified and the user is informed that no existing pricing was found
**And** the device is not transferred

### AC-6: Batch Transfer — Mixed Devices

**Given** a batch of 5 devices being transferred with "Maintain existing pricing" checked
**And** 4 devices have pricing (some service plan, some flat rate, some both)
**And** 1 device has neither
**When** the transfer is processed
**Then** the 4 devices with pricing are transferred successfully with their pricing preserved
**And** the 1 device without pricing shows an error and is skipped

### AC-7: Manual Entry Still Works

**Given** an admin is transferring a device without "Maintain existing pricing" checked
**When** they manually enter a flat rate and/or select a service plan on the form
**Then** the manually entered values are applied to the device (no change to current behavior)

### AC-8: Billing Reflects Maintained Pricing

**Given** a device with a flat rate was transferred with "Maintain existing pricing" checked
**When** the next billing cycle runs
**Then** the device is billed at the maintained flat rate under the new customer
