# Product Requirements Document - Simple Feature
## Admin-Configurable Additional Status per Device Model

**Document Version:** 1.0
**Date:** 2026-04-07
**Author:** Aksana / Orases
**Status:** Draft
**Related Tickets:** [WATM-2038](https://orases.atlassian.net/browse/WATM-2038)
**Document Owner:** Aksana
**Last Updated:** 07.04.2026

---

## Table of Contents

1. [Overview](#overview)
2. [Goals and Success Criteria](#goals-and-success-criteria)
3. [Users](#users)
4. [Scope](#scope)
5. [Functional Requirements and Business Rules](#functional-requirements-and-business-rules)
6. [Technical Requirements](#technical-requirements)
7. [Testing Requirements](#testing-requirements)
8. [Dependencies and Risks](#dependencies-and-risks)
9. [Open Questions](#open-questions)
10. [References](#references)
11. [Notes](#notes)

---

## Overview

This feature extends the Device Model management page in the WATM admin panel to allow administrators to configure which Additional Statuses are available for each device model. Currently, all Additional Statuses (Misplaced, Offline) are shown for every device regardless of manufacturer or model. The client needs the ability to restrict specific statuses from specific models — starting with removing "Offline" from SIM/Systech M5 and M5-302 models — without requiring code deployments for each change.

### Key Features
- **Model-Level Status Configuration**: Checkbox interface on the Device Model edit page to enable/disable Additional Statuses per model
- **Automatic Device Cleanup**: Devices with a newly-disallowed status are automatically cleared when the admin saves the configuration
- **Cross-Portal Enforcement**: Status restrictions are enforced on both admin panel and customer portal device edit forms
- **Dynamic Dropdown Refresh**: Additional Status dropdown updates dynamically when a device's model is changed

### Business Impact
- Eliminates the need for code deployments when the client wants to restrict statuses for additional models in the future
- Gives the client direct control over device status options per model, reducing turnaround time from days to minutes
- Prevents incorrect status assignments that may cause confusion in device monitoring workflows

---

## Goals and Success Criteria

### Primary Goals
1. **Enable admin-managed status restrictions per model**: Administrators can control which Additional Statuses are available for each Device Model through the admin panel without developer involvement
2. **Enforce restrictions across all device edit surfaces**: Both admin panel and customer portal device edit forms respect the configured restrictions
3. **Maintain backward compatibility**: Existing behavior is preserved for all models until an admin explicitly configures restrictions

### Success Criteria

- ✅ Admin can navigate to any Device Model edit page and see checkboxes for all Additional Statuses
- ✅ Unchecking a status and saving removes it from the Additional Status dropdown on all device edit forms for that model
- ✅ Devices currently assigned a removed status are automatically cleared (set to null) upon saving the model configuration
- ✅ Models with no explicit configuration continue to show all Additional Statuses (backward compatible)
- ✅ API device update endpoint rejects disallowed Additional Statuses with a validation error

---

## Users

### Primary Users

**WATM Administrator (APW Team)**
- **Role**: Admin panel user responsible for managing device models, manufacturers, and system configuration
- **Need**: Ability to control which Additional Statuses are available for each device model without requesting code changes
- **Pain Point**: Currently, all Additional Statuses appear for all devices regardless of model. Removing an irrelevant status (e.g., "Offline" for SIM-only models) requires a developer to hardcode the restriction and deploy
- **Benefit**: Self-service configuration that takes effect immediately, with the flexibility to adjust as new models or statuses are added

**Customer Portal User**
- **Role**: End user who manages their company's devices through the customer-facing portal
- **Need**: See only the Additional Statuses that are relevant to their device's model
- **Pain Point**: Presented with status options that don't apply to their device type, which can lead to incorrect status assignments
- **Benefit**: Cleaner, model-appropriate status options that reduce confusion and prevent invalid selections

---

## Scope

### In Scope

**Admin Configuration UI**
- ✅ "Available Additional Statuses" checkbox section on the Device Model edit page
- ✅ Save handler that persists the model-to-status associations
- ✅ Confirmation warning with affected device count when unchecking a status currently in use

**Device Edit Forms**
- ✅ Additional Status dropdown filtering on admin panel device edit form
- ✅ Additional Status dropdown filtering on customer portal device edit form
- ✅ AJAX refresh of Additional Status dropdown when device model is changed
- ✅ Graceful handling of devices with a currently-disallowed status (clear on save)

**Data & API**
- ✅ New join table (`device_models_addl_statuses`) and CakePHP model associations
- ✅ Seed data to maintain current behavior for all existing models
- ✅ API validation on device update endpoint for disallowed statuses

**Device Cleanup**
- ✅ Automatic nullification of disallowed statuses on affected devices when admin saves model config

### Out of Scope
- ❌ Restricting **primary statuses** per model (potential future enhancement)
- ❌ Bulk status update filtering (client confirmed not relevant)
- ❌ Manufacturer-level status restrictions (configuration is at the model level; models already belong to manufacturers)
- ❌ Adding new Additional Status values (existing seeded values are sufficient)

---

## Functional Requirements and Business Rules

### FR-1: Data Model — Model-to-Status Association

**FR-1.1: Join Table**
- The system shall store the many-to-many relationship between device models and additional statuses in a `device_models_addl_statuses` join table
- Each record links one `device_model_id` to one `device_addl_status_id`

**Business Rules:**
- If a model has **no entries** in the join table, all Additional Statuses are available for that model (backward compatible default)
- If a model has **one or more entries** in the join table, only the linked statuses are available
- All checkboxes checked and no checkboxes checked are treated equivalently — both mean "all statuses available"

**FR-1.2: Seed Data**
- The system shall seed the join table with all existing Additional Statuses linked to all existing Device Models
- This ensures no behavior change upon deployment for any existing model

### FR-2: Admin Configuration UI

**FR-2.1: Status Checkbox Section**
- The Device Model edit page shall display an "Available Additional Statuses" section
- The section shall contain one checkbox per record in the `device_addl_statuses` table
- Each checkbox shall be labeled with the Additional Status title
- Checked = status is available for devices of this model; Unchecked = status is not available

**Interaction & UI Details:**
- Checkboxes appear within the existing Device Model edit form, below existing fields
- Standard CakePHP form helper checkbox rendering, consistent with existing admin UI patterns
- The section is only visible to admin users; customer portal users cannot access Device Model edit pages

**FR-2.2: Save with Device Cleanup**
- When the admin unchecks a status that is currently assigned to one or more devices of this model and clicks Save, the system shall display a confirmation dialog
- The confirmation dialog shall state the number of devices that will have their Additional Status cleared
- If the admin confirms, the system shall:
  1. Save the updated status associations
  2. Set `device_addl_status_id` to NULL on all devices of this model that have the removed status
- If the admin cancels, the save is aborted and no changes are made

**Business Rules:**
- Only admin users can modify status configuration; this is enforced by existing RBAC on the Device Models controller
- The confirmation warning is only shown when unchecking a status that has active device assignments; unchecking an unused status saves silently

### FR-3: Device Edit — Status Dropdown Filtering

**FR-3.1: Admin Panel Device Edit**
- When editing a device in the admin panel, the Additional Status dropdown shall only contain statuses allowed by the device's current model configuration
- If the device's current Additional Status is not in the allowed list, it shall not appear as a selectable option
- On save, if the device's Additional Status is not in the allowed list, the system shall set it to NULL

**FR-3.2: Customer Portal Device Edit**
- The same filtering logic from FR-3.1 applies to the customer portal device edit form
- The customer portal user sees only the allowed statuses for the device's model

**FR-3.3: Dynamic Dropdown on Model Change**
- When the user changes the device's model on the edit form (via the existing manufacturer/model cascade), the Additional Status dropdown shall refresh via AJAX to reflect the new model's allowed statuses
- If the previously selected Additional Status is not allowed by the new model, the dropdown shall reset to the empty/null option

**Business Rules:**
- Bulk status updates are excluded from this filtering — no changes to bulk update functionality
- The null/empty option ("No Additional Status") is always available regardless of model configuration

### FR-4: API Validation

**FR-4.1: Device Update Endpoint**
- The API device update endpoint (`Api/V1/DevicesController`) shall validate that the submitted `device_addl_status_id` is allowed for the device's model
- If the submitted status is not in the allowed list, the API shall return a validation error with an appropriate message (e.g., "Additional status is not available for this device model")

**Business Rules:**
- NULL/empty Additional Status values are always accepted (clearing the status is always valid)
- If the model has no configuration (no join table entries), all statuses pass validation

### FR-5: Model Change Handling

**FR-5.1: Status Clearance on Model Change**
- When a device's model is changed (via the device edit form), if the device's current Additional Status is not allowed by the new model, the system shall clear the Additional Status (set to NULL) on save

**Business Rules:**
- This applies to both admin panel and customer portal device edit forms
- The user is informed via the dropdown resetting to empty (FR-3.3) before they save

---

## Technical Requirements

### Data Model

**`device_models_addl_statuses` (new join table)**

| Field | Type | Required | Validation | Notes |
|-------|------|----------|------------|-------|
| id | INT (auto-increment) | Yes | Auto-generated | Primary key |
| device_model_id | INT | Yes | FK → device_models.id | Device model reference |
| device_addl_status_id | INT | Yes | FK → device_addl_statuses.id | Additional status reference |
| created | DATETIME | Yes | Auto-set | Record creation timestamp |
| modified | DATETIME | Yes | Auto-set | Record modification timestamp |

**Unique constraint:** (`device_model_id`, `device_addl_status_id`) — prevent duplicate entries

### CakePHP Associations
- `DeviceModelsTable` → `BelongsToMany` → `DeviceAddlStatuses` (through `device_models_addl_statuses`)
- `DeviceAddlStatusesTable` → `BelongsToMany` → `DeviceModels` (through `device_models_addl_statuses`)
- New `DeviceModelsAddlStatusesTable` class for the join table

### Performance & System
- The status configuration query (join table lookup) adds one additional query per device edit page load; this is negligible given existing page complexity
- The AJAX endpoint for refreshing statuses on model change should respond within existing page interaction performance expectations

### Security & Compliance
- Admin-only configuration is enforced by existing RBAC on `DeviceModelsController` actions
- No new user-facing permissions are required; the feature leverages existing admin role checks
- Audit logging via existing `o_logs` integration for model configuration changes

### Files Impacted

| File | Change |
|------|--------|
| `config/Migrations/` | New migration: create `device_models_addl_statuses` table |
| `config/Seeds/` | Seed join table with all statuses for all existing models |
| `plugins/Devices/src/Model/Table/DeviceModelsTable.php` | Add `BelongsToMany` association |
| `plugins/Devices/src/Model/Table/DeviceModelsAddlStatusesTable.php` | New table class |
| `plugins/Devices/src/Model/Table/DeviceAddlStatusesTable.php` | Add `BelongsToMany` association |
| `plugins/Devices/src/Controller/Admin/DeviceModelsController.php` | Edit action: add status checkboxes, save handler with device cleanup |
| `plugins/Devices/templates/Admin/DeviceModels/edit.php` | Add "Available Additional Statuses" section |
| `plugins/Devices/src/Controller/Admin/DevicesController.php` | Filter Additional Status dropdown by model config |
| `plugins/Devices/templates/Admin/Devices/edit.php` | Dropdown rendering, AJAX refresh support |
| `plugins/Devices/src/Controller/Api/V1/DevicesController.php` | Validate Additional Status against model config |
| `webroot/js/` | AJAX handler to refresh Additional Status dropdown on model change |

---

## Testing Requirements

### Test Plan Overview
**Testing Phases:**
1. Integration Testing (Development team)
2. User Acceptance Testing (UAT) (APW team / Laura Perry)
3. Regression Testing (QA team)

### Key Test Scenarios

**IT-1: Admin Configures Status Restrictions**
1. Navigate to a Device Model edit page (e.g., M5)
2. Uncheck "Offline" from the Available Additional Statuses section
3. Save the model
4. **Verify:** "Offline" no longer appears in the Additional Status dropdown when editing a device of that model

**IT-2: Device Cleanup on Status Removal**
1. Ensure at least one device of model M5 has Additional Status = "Offline"
2. Navigate to Device Model M5 edit page
3. Uncheck "Offline" and click Save
4. **Verify:** Confirmation dialog appears showing the count of affected devices
5. Confirm the save
6. **Verify:** All M5 devices previously set to "Offline" now have Additional Status = NULL

**IT-3: Customer Portal Enforcement**
1. Configure model M5 to exclude "Offline"
2. Log in as a customer portal user with an M5 device
3. Edit the device
4. **Verify:** "Offline" does not appear in the Additional Status dropdown

**IT-4: Dynamic Dropdown on Model Change**
1. Edit a device currently assigned to a model with all statuses enabled
2. Change the model to one with "Offline" disabled
3. **Verify:** The Additional Status dropdown refreshes and "Offline" is removed
4. **Verify:** If "Offline" was selected, the dropdown resets to empty

**IT-5: API Validation**
1. Configure model M5 to exclude "Offline"
2. Send an API request to update an M5 device's Additional Status to "Offline"
3. **Verify:** The API returns a validation error

**IT-6: Backward Compatibility**
1. Verify a model with no explicit configuration (or all statuses checked) still shows all Additional Statuses
2. Edit a device of that model
3. **Verify:** All Additional Statuses are available in the dropdown

**UAT-1: End-to-End Admin Workflow**
- **Persona:** WATM Administrator (APW Team)
- **Scenario:** Admin wants to remove "Offline" from M5 and M5-302 models
- **Steps:** Navigate to each model's edit page → uncheck "Offline" → confirm affected device count → save
- **Success Criteria:** "Offline" is no longer selectable for M5/M5-302 devices on either portal; affected devices are cleared

**RT-1: Existing Device Edit Regression**
- **Verify:** Editing a device of a model with no status restrictions works exactly as before
- **Verify:** Primary Status dropdown behavior is unchanged
- **Verify:** Bulk status updates function without restriction

### Testing Notes
- Test with both admin and customer portal user roles
- Test the confirmation dialog cancel path (ensure no changes are saved)
- Test AJAX refresh with slow network conditions to ensure no race conditions with the manufacturer/model cascade

---

## Dependencies and Risks

### Dependencies

**Must Exist Before Development:**
1. **Existing Device Model admin CRUD** - Already exists; this feature extends the edit action | **Status:** Available
2. **Existing manufacturer/model cascade AJAX** - The dynamic model dropdown on device edit already exists; the Additional Status AJAX refresh will follow the same pattern | **Status:** Available

**Integrates With:**
- Device edit forms (admin panel and customer portal)
- API V1 device update endpoint
- Existing `device-manufacturer-model-cascade.js` AJAX pattern

### Risks

**LOW RISK: Additional Status Values Change**
- **Description:** If new Additional Status values are added to the seed data in the future, they won't automatically be enabled for models with explicit configurations
- **Impact:** Low — admin would need to manually enable the new status for each configured model
- **Probability:** Low — Additional Statuses rarely change
- **Mitigation:** Document this behavior; consider a notification or auto-enable option in a future iteration

**LOW RISK: Confirmation Dialog UX**
- **Description:** If a model has hundreds of devices with the status being removed, the cleanup operation could take noticeable time
- **Impact:** Low — the device count for any single model/status combination is expected to be manageable
- **Probability:** Low
- **Mitigation:** Run cleanup as a background operation if performance becomes an issue; for now, synchronous save is acceptable

---

## Open Questions

No open questions. All critical decisions were resolved during client Q&A.

---

