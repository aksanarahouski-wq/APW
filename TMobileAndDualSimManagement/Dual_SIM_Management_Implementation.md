# JIRA TICKET: Dual-SIM Device Management - Complete Implementation

## Summary
Implement dual-SIM device management enhancements including T-Mobile support, configuration mapping matrix, import/assignment process updates, and smart SIM status handling

## Issue Type
Epic / Story

## Priority
High

## Labels
- dual-sim
- t-mobile
- configuration-mapping
- device-management
- origin-device

## Description

### Overview
Implement comprehensive dual-SIM device management system to support the new Origin device model with T-Mobile carrier, enhance configuration mapping logic to consider both SIM presence and active status, and streamline import/assignment workflows while preserving carrier "test ready" status benefits.

### Background
- New Origin device model with eSIM (Verizon) + physical SIM (T-Mobile/AT&T) capability
- Current system only considers active status for configuration mapping (insufficient for dual-SIM scenarios)
- Need to support devices with 2 SIMs present but only 1 active
- Warehouse pre-configures SIMs in "test ready" status to avoid premature billing charges
- Decided across 3 meetings (see Resources section)

### Business Value
- Support new Origin device model for market expansion
- Reduce billing costs by preserving "test ready" status (AT&T unlimited free, T-Mobile 6-month grace period)
- Eliminate configuration remapping issues when carrier status changes
- Streamline warehouse operations and reduce errors during device assignment

---

## Acceptance Criteria

### 1. Import Template Enhancement
- [ ] Import template includes new columns: `Verizon Active` (Yes/No), `AT&T Active` (Yes/No), `T-Mobile Active` (Yes/No)
- [ ] System validates maximum 2 SIMs per import row (reject if 3 SIMs provided)
- [ ] System auto-sets `is_dual_sim` flag to TRUE if both SIMs imported as active
- [ ] Import process does NOT trigger carrier API calls (no activation on import)
- [ ] SIM numbers and active flags stored in database only
- [ ] Import validation prevents active flag without corresponding SIM value

### 2. Assignment Screen Simplification
- [ ] Carrier selection section removed from assignment screen
- [ ] "Maintain existing SIM" checkbox removed from assignment screen
- [ ] Assignment only updates `company_id` relationship
- [ ] Assignment does NOT modify SIM status or trigger carrier API calls
- [ ] Existing billing cycle transfer logic preserved

### 3. Update Devices Feature Enhancement
- [ ] Update Devices feature supports bulk SIM status changes with new active status columns
- [ ] Can bulk update: Verizon Active, AT&T Active, T-Mobile Active (Yes/No)
- [ ] Preferred method for bulk SIM status modifications (not assignment screen)

### 4. Configuration Mapping Engine Overhaul
- [ ] Implement 18-row configuration mapping matrix (see Implementation Details)
- [ ] Mapping logic considers BOTH SIM presence AND active status (not just active status)
- [ ] Special cases: 2 SIMs present + 1 active = dual carrier config (not single carrier)
  - Example: VZW inactive + ATT active → `dual_vz_att` config
  - Example: VZW active + TMO inactive → `dual_vz_tmo` config
- [ ] Add `dual_vz_tmo` carrier combination to config builder
- [ ] Update `ConfigurationsTable::afterSave()` auto-assignment logic
- [ ] Update device `carrier_type` virtual property calculation

### 5. Device SIM Status Update (Smart Status Logic)
- [ ] **On Import**: No carrier API calls, assume "test ready" status
- [ ] **On Device Edit**: Implement smart status toggle logic:
  - [ ] Query current SIM status via carrier API before update
  - [ ] If current status = "test ready" → keep "test ready" (do not change)
  - [ ] If current status ≠ "test ready" → toggle "active" ↔ "inactive" based on checkbox
  - [ ] AT&T-specific: Once exited "test ready", cannot return (enforce one-way transition)
  - [ ] T-Mobile-specific: 6-month grace period in "test ready"
  - [ ] Verizon: Standard activation (no test ready handling)
- [ ] Update `DeviceSimStatusesTable` records accordingly
- [ ] Log status transitions for audit trail

### 6. Validation Rules
- [ ] Prevent saving device with active SIM flag when no SIM value exists
- [ ] All 18 configuration mapping scenarios validated (see matrix below)
- [ ] Maintain existing validation: devices must have valid `configuration_id`

### 7. Testing
- [ ] Test all 18 configuration mapping scenarios (especially yellow-highlighted special cases)
- [ ] Test import with 3 SIMs (should be rejected)
- [ ] Test import with 2 active SIMs (should set `is_dual_sim=1`)
- [ ] Test device edit preserves "test ready" status
- [ ] Test device edit toggles active/inactive when not in "test ready"
- [ ] Test assignment does not modify SIM status
- [ ] Test bulk update via Update Devices feature

---

## Implementation Details

### 18-Row Configuration Mapping Matrix

#### I-4100/4500/M5 Models (Single SIM only - 3 rows):
1. VZW SIM active, ATT SIM=NO, TMO SIM=NO → `vz` config
2. VZW SIM=NO, ATT SIM active, TMO SIM=NO → `att` config
3. VZW SIM=NO, ATT SIM=NO, TMO SIM active → `tmo` config

#### I-22/I-52 Models (9 rows):
4. VZW SIM active, ATT SIM=NO, TMO SIM=NO → `vz` config
5. VZW SIM active, ATT SIM inactive, TMO SIM=NO → `vz` config
6. VZW SIM active, ATT SIM=NO, TMO SIM inactive → `vz` config
7. VZW SIM=NO, ATT SIM active, TMO SIM=NO → `att` config
8. **⚠️ VZW SIM inactive, ATT SIM active, TMO SIM=NO → `dual_vz_att` config** (SPECIAL CASE)
9. **⚠️ VZW SIM inactive, ATT SIM=NO, TMO SIM active → `dual_vz_tmo` config** (SPECIAL CASE)
10. VZW SIM active, ATT SIM active, TMO SIM=NO → `dual_vz_att` config
11. VZW SIM active, ATT SIM=NO, TMO SIM active → `dual_vz_tmo` config

#### ORIGIN/CR202 Models (6 rows - Verizon always eSIM):
12. **⚠️ VZW eSIM active, ATT SIM=NO, TMO SIM inactive → `dual_vz_tmo` config** (SPECIAL CASE)
13. **⚠️ VZW eSIM active, ATT SIM inactive, TMO SIM=NO → `dual_vz_att` config** (SPECIAL CASE)
14. VZW eSIM inactive, ATT SIM active, TMO SIM=NO → `att` config
15. VZW eSIM inactive, ATT SIM=NO, TMO SIM active → `tmo` config
16. VZW eSIM active, ATT SIM active, TMO SIM=NO → `dual_vz_att` config
17. VZW eSIM active, ATT SIM=NO, TMO SIM active → `dual_vz_tmo` config

**⚠️ = Special cases requiring dual carrier config despite single active SIM**

### Mapping Logic Algorithm
```
1. Determine device model (I-4100/4500/M5, I-22/I-52, or ORIGIN/CR202)
2. Check SIM presence for each carrier (VZW, ATT, TMO)
3. Check active status from device_sim_statuses table
4. Apply matrix rules:
   - If 2 SIMs present (regardless of active status) → dual carrier config
   - If 1 SIM present and active → single carrier config
   - If 1 SIM present and inactive → no config match (validation error)
5. Match to configuration where:
   - config_group.device_model_id = device.device_model_id
   - carrier matches matrix result
   - service_plan_id matches
   - cellular_backup matches
```

### Technical Implementation Areas

#### 1. Database/Models
**Files to modify:**
- `plugins/Devices/src/Model/Table/DevicesTable.php`
  - Update import logic (no API calls)
  - Update device edit save logic (smart status handling)
- `plugins/SystemManagement/src/Model/Table/ConfigurationsTable.php`
  - Update `afterSave()` method with new 18-row mapping logic (lines 315-407)
- `plugins/Devices/src/Model/Entity/Device.php`
  - Update `carrier_type` virtual property calculation
- `plugins/Devices/src/Model/Table/DeviceSimStatusesTable.php`
  - Implement smart status query and update logic

#### 2. Controllers
**Files to modify:**
- `plugins/Devices/src/Controller/Admin/DevicesController.php`
  - Update import action
  - Update edit action (add smart status logic before save)
  - Update assign action (remove carrier selection logic)
- `plugins/SystemManagement/src/Controller/Admin/ConfigurationsController.php`
  - Add `dual_vz_tmo` to carrier dropdown options

#### 3. Templates/Views
**Files to modify:**
- `plugins/Devices/templates/Admin/Devices/import.php`
  - Add import template download with new columns
- `plugins/Devices/templates/Admin/Devices/assign.php`
  - Remove carrier selection section
  - Remove "maintain existing SIM" checkbox
- `plugins/Devices/templates/Admin/Devices/edit.php`
  - Ensure SIM active checkboxes trigger smart status logic
- `plugins/SystemManagement/templates/Admin/Configurations/add.php` & `edit.php`
  - Add `dual_vz_tmo` option to carrier dropdown

#### 4. Import/Export
**Files to modify:**
- Import CSV template file
- Import validation logic
- Bulk update devices CSV template

#### 5. API Integration
**Files to modify:**
- Verizon API service (maintain existing activation logic)
- AT&T API service (add "test ready" status check)
- T-Mobile API service (add "test ready" status check)

---

## Technical Notes

### Dual Carrier vs. Dual SIM Definitions
- **Dual Carrier Config**: Two SIMs physically present (determines config file selection)
- **Dual SIM Billing**: Both SIMs active (determines if customer charged for 2 carriers)
- These are **separate concepts** with different purposes

### Carrier-Specific "Test Ready" Behavior
- **AT&T**: Unlimited free period while in "test ready", once exited cannot return
- **T-Mobile**: 6-month grace period in "test ready"
- **Verizon**: No "test ready" status, standard activation only

### Why 2 SIMs Present = Dual Carrier Config (Even if 1 Inactive)
Device needs dual carrier config structure when 2 SIMs present because:
1. Prevents config remapping if carrier status changes later
2. Config file contains failover logic for both SIMs
3. Inactive SIM can be activated without changing config
4. Avoids device going offline during config transition

### Edge Case Deferred
**Scenario**: Dual carrier device on VZW-only config, VZW goes down, need to switch to ATT remotely
**Decision**: Cannot solve through portal if device offline, handle operationally (extremely rare)

---

## Sub-Tasks / Checklist

### Phase 1: Database & Model Changes
- [ ] Update `device_sim_statuses` table structure (if needed)
- [ ] Add `dual_vz_tmo` to configurations.carrier ENUM
- [ ] Update DevicesTable import method (remove API calls)
- [ ] Update DevicesTable edit method (add smart status logic)
- [ ] Update ConfigurationsTable afterSave mapping logic (18-row matrix)
- [ ] Update Device entity carrier_type virtual property
- [ ] Add validation: prevent active flag without SIM value
- [ ] Add validation: maximum 2 SIMs on import

### Phase 2: Controller Changes
- [ ] Update DevicesController::import() - add new column support
- [ ] Update DevicesController::edit() - implement smart status checks
- [ ] Update DevicesController::assign() - remove carrier selection
- [ ] Update DevicesController::updateDevices() - add bulk SIM status support
- [ ] Update ConfigurationsController - add dual_vz_tmo option

### Phase 3: View/Template Changes
- [ ] Create new import CSV template with active columns
- [ ] Update import.php template with new template link
- [ ] Update assign.php template (remove carrier section)
- [ ] Update edit.php template (ensure smart status triggers)
- [ ] Update configurations add/edit templates (dual_vz_tmo option)

### Phase 4: API Service Updates
- [ ] Implement AT&T "test ready" status query
- [ ] Implement T-Mobile "test ready" status query
- [ ] Update DeviceSimStatusesTable with API response handling
- [ ] Add logging for status transitions

### Phase 5: Testing
- [ ] Unit tests for 18-row mapping matrix logic
- [ ] Integration tests for import (3 SIMs rejection, dual flag auto-set)
- [ ] Integration tests for device edit (test ready preservation)
- [ ] Integration tests for assignment (no SIM status change)
- [ ] Integration tests for bulk update
- [ ] Manual QA for all 18 scenarios (especially special cases 8-9, 12-13)

### Phase 6: Documentation
- [ ] Update user documentation for import process
- [ ] Update admin documentation for configuration mapping rules
- [ ] Document "test ready" status handling for operations team
- [ ] Update API documentation

---

## Dependencies
- Origin device model must be created in system
- T-Mobile API integration must be functional
- Config builder must support `dual_vz_tmo` carrier type
- Migration `20250905185829_CreateDeviceSimStatuses.php` must be applied

---

## Related Resources

### Meeting Notes & Follow-ups
- Meeting 1 Transcript: `/Users/aksana/Documents/Projects/WATM/Meetings/Dual_Sim_meeting_1.md`
- Meeting 2 Transcript: `/Users/aksana/Documents/Projects/WATM/Meetings/Dual_Sim_meeting_2.md`
- Meeting 2 Follow-up: `/Users/aksana/Documents/Projects/WATM/Meetings/FollowUps/FollowUp_Meet2.md`
- Meeting 3 Transcript: `/Users/aksana/Documents/Projects/WATM/Meetings/Dual_Sim_meeting_3.md`
- Meeting 3 Follow-up: `/Users/aksana/Documents/Projects/WATM/Meetings/FollowUps/FollowUp_Meet3.md`

### Diagrams & Documentation
- Configuration Mapping Matrix: `/Users/aksana/Documents/Projects/WATM/Meetings/Images/Configuration Mapping Matrix.png`
- Base Device Configurations Documentation: (existing system docs)

### Code References
- `plugins/SystemManagement/src/Model/Table/ConfigurationsTable.php:315-407` (current mapping logic)
- `plugins/Devices/src/Model/Table/DevicesTable.php` (device management)
- `plugins/Devices/src/Model/Table/DeviceSimStatusesTable.php` (SIM status tracking)
- Migration `20250905185829_CreateDeviceSimStatuses.php`
- Migration `20250904180921_RenameConfigurationsDualCarrierEnum.php`

---

## Assignee
Richard Sacco (Lead Developer)

## Reporter
Aksana Rahouski (Product Owner)

## Story Points
21 (Epic - substantial changes across multiple components)

## Sprint
TBD

---

## Comments / Notes

### Configuration Engine Future Work (Out of Scope)
This ticket implements hard-coded 18-row mapping rules as a temporary solution. Future configuration engine overhaul will use hierarchical parameter system (Global → Model → Carrier), but that is a separate initiative.

### Parameter Documentation (In Progress)
Adam has documented 664 of 700 InHand device parameters (36 pending). This documentation will be critical for future configuration engine work but is not blocking for current ticket.

### Testing Priority
Special cases (rows 8-9, 12-13 in matrix) are highest priority for testing as they represent the most complex scenarios where 2 SIMs are present but only 1 is active.
