# WATM-2038: Additional Status — Technical Analysis

**Ticket:** WATM-2038 — Additional Status
**Date:** 2026-04-03
**Status:** Draft
**Creator:** Laura Perry

## Client Request

> Remove the "Offline" option from "Additional Status" for devices with manufacturer "SIM" or "Systech", and models "M5" or "M5-302".

## Current System Architecture

### Status Model

Devices have two status fields:

| Field | Table | FK Column | Values | Required |
|-------|-------|-----------|--------|----------|
| Primary Status | `device_statuses` | `devices.device_status_id` | Active, Deactivated, Data Suspension, Admin Suspension, Customer Suspension | Yes |
| Additional Status | `device_addl_statuses` | `devices.device_addl_status_id` | Misplaced, Offline | No (nullable) |

- Statuses are **seeded into the database** via `DeviceStatusesSeed.php` and `DeviceAddlStatusesSeed.php`
- There is **no admin UI** for managing status records — they are hardcoded at deployment
- Primary statuses have a `company_can_modify` flag for role-based filtering
- Additional statuses have **no filtering logic** — all options are shown to all users for all devices

### How Statuses Are Presented

- **Edit form:** `plugins/Devices/templates/Admin/Devices/edit.php` (lines 56-71)
- **Controller:** `plugins/Devices/src/Controller/Admin/DevicesController.php` (lines 1264-1463)
- Primary status dropdown is filtered by user role (admin vs. customer) and device state (is_vz_pending, current status)
- Additional status dropdown is **unfiltered** — always shows all values regardless of manufacturer/model

### Manufacturer & Model Architecture

- `device_manufacturers` table — has boolean capability flags (`is_systech_api`, `sim_only`, `is_inhand_api`, etc.)
- `device_models` table — has boolean capability flags (`is_cellular_backup_capable`, `is_power_cycler_capable`)
- Both already support **per-manufacturer and per-model feature flags** pattern
- Admin CRUD exists for both: `DeviceManufacturersController` and `DeviceModelsController`
- Cascading dropdown in UI: selecting manufacturer filters available models via AJAX (`device-manufacturer-model-cascade.js`)

### Key Finding

The system already uses a **feature flag pattern on models** (e.g., `is_cellular_backup_capable`, `is_power_cycler_capable`) to control device capabilities. There is **no existing mechanism** to control which statuses are available per manufacturer or model. The additional status dropdown is always the full list.

## Options Analysis

### Option 1: Hardcoded Filtering

**Approach:** Add server-side logic in `DevicesController::edit()` that checks the device's manufacturer/model and removes "Offline" from the additional status dropdown for specific manufacturers (SIM, Systech) and models (M5, M5-302).

**Implementation:**
- Modify `DevicesController.php` edit action (~lines 1264-1463) to filter `deviceAddlStatuses` list based on device manufacturer title or model name
- Could use manufacturer flags (`is_systech_api`, `sim_only`) or match by title string
- Similar filtering needed in bulk status update action (~line 2630+)
- API controller (`Api/V1/DevicesController.php`) status mapping would also need guards

**Pros:**
- Fast to implement (hours, not days)
- Minimal database changes
- Low risk — small, targeted code change
- Directly addresses the client's exact request

**Cons:**
- Any future changes require a code deployment
- Client cannot self-manage — must request changes from dev team
- Logic is buried in controller code, not visible in admin UI
- If more manufacturers/models need status restrictions later, the hardcoded list grows and becomes harder to maintain
- Doesn't scale well if the client wants per-status control across more dimensions

### Option 2: Model-Level Status Configuration (Admin-Managed)

**Approach:** Add a many-to-many relationship between `device_models` and `device_addl_statuses` (and optionally `device_statuses`) so the admin can configure which statuses are available per model.

**Implementation:**
- New join table: `device_models_addl_statuses` (`device_model_id`, `device_addl_status_id`)
- New admin UI on the Device Model edit page: checkboxes for enabled additional statuses
- Modify `DevicesController::edit()` to filter the additional status dropdown based on the device's model's enabled statuses
- Default behavior: if no statuses are configured for a model, show all (backward compatible)
- Seed the join table for existing models with all statuses enabled

**Pros:**
- Client fully controls which statuses are available per model — no dev involvement needed
- Follows existing pattern (model already has capability flags, this extends the concept)
- Admin UI is already built for models — just adds a section
- Scales to any future status restrictions without code changes
- Visible and auditable — admin can see exactly what's configured
- Could extend to primary statuses later if needed

**Cons:**
- More upfront development time (migration, join table, admin UI, controller logic)
- Slightly more complex data model
- Requires client training on the new admin feature
- Need to handle edge cases: what happens to devices already set to a status that gets disabled for their model?

## Recommendation

**Option 2 is the stronger long-term investment.** The system already follows the pattern of per-model capability flags. This request signals the client wants model-level control over device behavior, and status restrictions are likely just the beginning. Building the configurable approach now avoids repeated small deployments for each future restriction.

However, **Option 1 is valid if timeline is tight** or the client confirms this is a one-off request with no plans to expand.

## Files Impacted

| File | Purpose |
|------|---------|
| `plugins/Devices/src/Controller/Admin/DevicesController.php` | Edit action status filtering |
| `plugins/Devices/templates/Admin/Devices/edit.php` | Status dropdown rendering |
| `plugins/Devices/src/Model/Table/DeviceAddlStatusesTable.php` | Status list query |
| `plugins/Devices/src/Model/Table/DeviceModelsTable.php` | Model associations (Option 2) |
| `plugins/Devices/src/Controller/Admin/DeviceModelsController.php` | Model admin UI (Option 2) |
| `plugins/Devices/templates/Admin/DeviceModels/edit.php` | Model edit form (Option 2) |
| `plugins/Devices/src/Controller/Api/V1/DevicesController.php` | API status validation |
| `config/Migrations/` | New migration (Option 2) |
| `config/Seeds/` | Seed join table (Option 2) |
