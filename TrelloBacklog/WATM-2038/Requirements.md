# WATM-2038: Admin-Configurable Additional Status per Device Model

**Ticket:** WATM-2038
**Date:** 2026-04-07
**Status:** Requirements Approved
**Requested By:** Laura Perry / APW Team
**Approach:** Option 2 — Admin-Configurable Status per Model

---

## Decision Summary

The client selected **Option 2: Admin-Configurable Status per Model**. The admin panel will be extended so administrators can control which Additional Statuses are available for each Device Model.

### Client Answers to Clarifying Questions

| Question | Answer |
|----------|--------|
| Should this restriction apply when editing existing devices that already have "Offline" set? | **Yes** — clear the status on affected devices, don't just prevent new selections |
| Are there other manufacturer/model combinations you'd eventually want status restrictions for? | **Possibly** — confirms the need for a configurable approach |
| Does this apply to both the admin panel and the customer portal? | **Visible by both, only configurable by admin** — both portals respect the restriction, but only admins can manage the configuration |
| Should the restriction also apply to bulk status updates? | **No** — bulk update is not relevant for this restriction |

---

## Requirements

### 1. Data Model

**1.1** Create a join table `device_models_addl_statuses` to represent the many-to-many relationship between `device_models` and `device_addl_statuses`.

| Column | Type | Description |
|--------|------|-------------|
| `id` | INT (PK, auto-increment) | Primary key |
| `device_model_id` | INT (FK → device_models.id) | The device model |
| `device_addl_status_id` | INT (FK → device_addl_statuses.id) | The allowed additional status |
| `created` | DATETIME | Timestamp |
| `modified` | DATETIME | Timestamp |

**1.2** Default behavior: If a model has **no entries** in the join table, **all** additional statuses are available (backward compatible).

**1.3** If a model has **any entries** in the join table, **only** the linked statuses are available for devices of that model.

**1.4** Seed the join table with all additional statuses enabled for all existing models to maintain current behavior.

### 2. Admin Configuration UI

**2.1** Add an "Available Additional Statuses" section to the **Device Model edit page** (`DeviceModelsController::edit`).

**2.2** Display checkboxes for each additional status record from `device_addl_statuses`.

**2.3** When all checkboxes are checked (or none are checked), all statuses are available — both states are treated equivalently for backward compatibility.

**2.4** Only admin users can access and modify this configuration. Customer portal users cannot see or change these settings.

### 3. Device Edit — Status Dropdown Filtering

**3.1** On the **device edit form** (admin panel), the Additional Status dropdown must be filtered based on the device's model configuration.

**3.2** On the **device edit form** (customer portal), the Additional Status dropdown must also be filtered based on the device's model configuration.

**3.3** If the device's current additional status is **not** in the allowed list for its model, the form must still handle this gracefully:
- Do **not** show the disallowed status as a selectable option
- On save, clear the disallowed status (set to null)

**3.4** Bulk status updates are **excluded** from this restriction — no changes needed to bulk update functionality.

### 4. Existing Device Cleanup

**4.1** When an additional status is **removed** from a model's allowed list (unchecked in admin), devices of that model that currently have that status must be updated:
- Set `device_addl_status_id` to `NULL` on affected devices
- This should happen at the time the admin saves the model configuration

**4.2** Display a confirmation warning to the admin before saving when unchecking a status that is currently assigned to devices of that model. The warning should indicate how many devices will be affected.

### 5. API

**5.1** The API device status update endpoint (`Api/V1/DevicesController`) must validate that the submitted additional status is allowed for the device's model.

**5.2** If a disallowed status is submitted via API, return a validation error.

### 6. Model Change Handling

**6.1** When a device's model is changed (on the device edit form), if the device's current additional status is not allowed by the new model, clear the additional status (set to null).

**6.2** The Additional Status dropdown should update dynamically when the model is changed (via the existing manufacturer/model cascade AJAX pattern).

---

## Out of Scope

- Restricting **primary statuses** per model (future consideration)
- Bulk status update filtering
- Manufacturer-level status restrictions (configuration is at the **model** level only; the model already belongs to a manufacturer)

---

## Files Impacted

| File | Change |
|------|--------|
| `config/Migrations/` | New migration for `device_models_addl_statuses` join table |
| `config/Seeds/` | Seed join table with all statuses for all existing models |
| `plugins/Devices/src/Model/Table/DeviceModelsTable.php` | Add `BelongsToMany` association to `DeviceAddlStatuses` |
| `plugins/Devices/src/Model/Table/DeviceModelsAddlStatusesTable.php` | New table class for join table |
| `plugins/Devices/src/Model/Table/DeviceAddlStatusesTable.php` | Add `BelongsToMany` association to `DeviceModels` |
| `plugins/Devices/src/Controller/Admin/DeviceModelsController.php` | Add status checkboxes to edit action, handle save with device cleanup |
| `plugins/Devices/templates/Admin/DeviceModels/edit.php` | Add "Available Additional Statuses" checkbox section |
| `plugins/Devices/src/Controller/Admin/DevicesController.php` | Filter additional status dropdown by model config |
| `plugins/Devices/templates/Admin/Devices/edit.php` | Status dropdown rendering (may need AJAX refresh) |
| `plugins/Devices/src/Controller/Api/V1/DevicesController.php` | Validate additional status against model config |
| `webroot/js/` | AJAX handler to refresh additional statuses on model change |

---

## Acceptance Criteria

1. Admin can navigate to a Device Model edit page and see checkboxes for each additional status
2. Unchecking a status and saving removes it from the Additional Status dropdown on device edit forms for that model
3. Devices of that model that currently have the unchecked status are updated to NULL, with a confirmation warning shown first
4. The restriction is enforced on both admin panel and customer portal device edit forms
5. The restriction is enforced on the API device update endpoint
6. Changing a device's model clears the additional status if it's not allowed by the new model
7. Models with no status configuration (or all statuses checked) show all additional statuses (backward compatible)
8. Bulk status updates are not affected by this restriction
