# Requirements

**Purpose:** Business requirements, functional specifications, and PRD materials
**Status:** Requirements Complete, Design Validated
**Last Updated:** March 1, 2026

---

## Overview

This directory contains all requirements documentation for the WATM Hierarchical Configuration Management System. Requirements are derived from client discovery sessions, production configuration analysis, and technical research.

---

## Folder Structure

### PRD/
**Purpose:** Official Product Requirements Documents and specifications

**Key Documents:**

#### Config_Management_System_PRD.md
**Status:** Version 1.5 - COMPLETE
**Contains:**
- Executive summary and problem statement
- Six-layer configuration hierarchy specification
- Conditional Rules Framework (Two-Way, Three-Way, Four-Way)
- 11-level priority resolution engine
- User stories and use cases
- Success criteria and metrics

**Recent Updates (v1.5):**
- Expanded Two-Way Rules to support ALL 6 two-factor combinations
- Validated Three-Way Rules as ONLY Model+Carrier+ServicePlan
- Added sub-priority ordering for Two-Way Rules
- Real-world examples for each rule type

#### Config_Management_System_PRD_Functional_Requirements.md
**Status:** COMPLETE
**Contains:**
- FR-1: Config Key Schema Management (600+ parameters)
- FR-2: Six-Layer Configuration Hierarchy
- FR-3: Config Resolution Engine (11-level priority)
- FR-3a: Conditional Rules Framework
- FR-4: Layer Configuration Interfaces (Global, Model, Carrier, Plan, Company, Device)
- FR-5: Device Effective Config View
- FR-6: Customer Self-Service Portal
- FR-7: Validation & Safety Systems
- FR-8: Audit Trail & Logging
- FR-9: Change Detection & Versioning
- FR-10: Database Schema Requirements

#### Config_Management_System_Presentation.md
**Status:** COMPLETE
**Purpose:** Stakeholder presentation materials

#### Priority_Hierarchy_Test_Cases.md
**Status:** COMPLETE
**Purpose:** Test cases for 11-level priority resolution validation

#### Bulk_Change_Service_Plan_Flow.md
**Status:** COMPLETE
**Purpose:** Workflow for bulk service plan changes affecting configurations

### PRD/Outstanding_Issues/
**Purpose:** Open design questions requiring resolution

**Current Issues:**

#### 1. Device_Configuration_Grouping_and_Comparison.md
**Status:** Solution Proposed - Awaiting Review
**Priority:** HIGH - Critical for admin usability

**Problem:** In old system, admins could tell if devices shared configs by filename. In new dynamic system, need alternative method to identify identical configurations.

**Proposed Solution:** Hybrid approach using:
- Profile Hash (fast grouping by device attributes)
- Effective Config Hash (accurate validation of resolved values)

**Next Steps:**
- Stakeholder review
- Engineering performance validation
- Prioritize Phase 1 vs Phase 2 features

#### 2. Configuration_Validation_Strategy.md
**Status:** Solution Proposed - Awaiting Review
**Priority:** CRITICAL - Prevents bricking devices at scale

**Problem:** Configuration changes can affect thousands of devices simultaneously. Invalid configs can permanently damage devices. Need comprehensive validation strategy to ensure every device receives valid configuration after any layer/rule change.

**Proposed Solution:** 5-Level Defense-in-Depth Strategy:
- Level 1: Input Validation (instant feedback on parameter entry)
- Level 2: Layer Validation (before saving changes)
- Level 3: Impact Calculation (show affected devices before apply)
- Level 4: Effective Config Validation (CRITICAL - validate fully resolved configs)
- Level 5: Device-Side Validation (final safety net)

**Next Steps:**
- Engineering review of validation logic
- QA validation of test coverage
- Performance testing of effective config validation at scale

#### 3. Configuration_Versioning_and_Change_Management.md
**Status:** Solution Proposed - Awaiting Review
**Priority:** HIGH - Critical for safe bulk changes and production rollback

**Problem:** Need ability to group related configuration changes together, review/approve before deployment, and rollback if issues occur. Currently no way to make bulk edits as "draft", no approval workflow, and no quick rollback capability.

**Proposed Solution:** Git-like versioning system with:
- Draft/Review/Approval workflow
- Version comparison (side-by-side before/after)
- Atomic activation (all devices updated together)
- Instant rollback to previous version
- Multi-stakeholder review process
- Gradual rollout capability (test on small subset first)

**Next Steps:**
- Database schema review by engineering
- Workflow review by operations team
- UI/UX mockups for version management interface
- Performance testing of atomic activation at scale

---

### Client_Requirements/
**Purpose:** Client-specific requirements and pain points

**Key Document:**

#### Client_Requirements_Summary.md
**Status:** COMPLETE
**Source:** Meeting1.md, Meeting2.md (04_Meetings_and_Discovery/)

**Contains:**
- Multi-level configuration hierarchy requirements
- Company-level custom configurations (admin-managed, not customer-accessible)
- Device-level configurations (limited customer access)
- Migration strategy requirements (no hard cutover, gradual migration)
- Phased development approach
- Access control requirements

**Critical Client Quotes:**
- "Keep the current configuration management system in place while we integrate or move away"
- "They will take themselves out of business and blame us" (re: customer self-service safety)
- Configuration management is the "heartbeat" of their business

---

### Data_Structures/
**Purpose:** Database schema and data model requirements

**Key Document:**

#### Data_Structure_Requirements.md
**Status:** COMPLETE

**Contains:**
- Entity-Relationship Diagrams (ERD)
- Table structures for all layers
- Conditional rules tables (Two-Way, Three-Way, Four-Way)
- Configuration key schema table
- Device configuration tracking tables
- Audit logging tables
- Indexes and performance optimization

**Key Tables:**
- `config_keys` - Schema for 600+ configuration parameters
- `config_global_values`, `config_model_values`, etc. - Layer-specific values
- `config_conditional_rules` - Multi-factor conditional logic
- `devices` - Device attributes and config hashes
- `device_config_overrides` - Device-level custom values
- `config_audit_log` - Complete change history

---

## Requirements Traceability

### Requirements Sources

**Research and Analysis** (`02_Research_and_Analysis/`)
- Multi-Layer Parameters Analysis → Layer hierarchy design
- Rule Combinations Analysis → Conditional rules framework
- Configuration Dependencies → Priority resolution logic

**Client Documents** (`01_Client_Provided_Documents/`)
- Production config files → 600+ parameter schema
- Customer configs → Four-Way Rules requirements
- Service plan configs → Three-Way Rules requirements

**Discovery Sessions** (`04_Meetings_and_Discovery/`)
- Meeting1.md → Multi-level hierarchy, migration strategy
- Meeting2.md → Goals, objectives, user stories
- InternalMeeting.md → Technical constraints, implementation approach

**Current System Analysis** (`06_Documentation/System_Documentation/`)
- Device_configs_application.md → Integration requirements
- Custom_Company_Configurations.md → Company layer requirements

---

## Key Requirements Summary

### Validated Architecture

**Six-Layer Configuration Hierarchy:**
1. Global Layer - System-wide defaults
2. Model Layer - Device model-specific settings
3. Carrier Layer - Cellular carrier-specific settings
4. Service Plan Layer - Service tier features and policies
5. Company Layer - Customer portfolio-wide settings
6. Device Layer - Individual device overrides

**Conditional Rules Framework:**
- **Two-Way Rules (6 types):** Model+Carrier, Carrier+ServicePlan, Model+ServicePlan, Carrier+Customer, Model+Customer, ServicePlan+Customer
- **Three-Way Rules (1 type):** Model+Carrier+ServicePlan (service plan baselines)
- **Four-Way Rules (1 type):** Model+Carrier+ServicePlan+Customer (customer exceptions)

**11-Level Priority Resolution:**
1. Device Override
2. Four-Way Rule (customer exception)
3. Company Override
4. Three-Way Rule (service plan baseline)
5. Service Plan Layer
6. Two-Way Rule (6 sub-priorities)
7. Carrier Layer
8. Model Layer
9. Global Layer
10. Schema Default
11. Required Validation

### User Access Requirements

**System Administrators:**
- Full access to all layers and all parameters
- Conditional Rules management
- Bulk operations
- Migration tools
- Complete audit trail

**End Customers (Phase 1: Device-Level Only):**
- Device-level configuration only (not company-wide)
- Customer-configurable parameters only
- UI-driven controls with strict validation
- Cannot download raw config files
- Preview and confirmation required

**End Customers (Phase 2+: Company-Wide):**
- Company-level configuration (portfolio-wide settings)
- Apply settings across entire device fleet
- Same validation and safety controls

### Migration Requirements

**Critical Constraints:**
- NO hard cutover - cannot lose configurations during migration
- Gradual device-by-device migration
- Both old and new systems operational in parallel
- Safe rollback capability
- Start small (5-10 devices), expand gradually (100 → 500 → 1,000+)
- Per-device migration status tracking

### Performance Requirements

**Target Metrics:**
- Config resolution: <100ms per device
- Global config change: <10 minutes for 100,000 devices
- Bulk operations: <5 minutes for 10,000 devices
- Admin UI response: <2 seconds for grouping queries
- Profile hash calculation: <1ms per device
- Effective hash calculation: <100ms per device (async)

### Safety Requirements

**Validation:**
- Schema-level validation (data types, formats, ranges)
- Layer-level validation (only allowed parameters at each layer)
- Required parameter validation (cannot generate invalid config)
- Customer permission validation (cannot modify restricted parameters)

**Audit Trail:**
- Log all configuration changes (who, what, when, why)
- Log effective config generation events
- Log customer access and modifications
- Searchable audit log interface
- Export for compliance

---

## Requirements Status

| Requirement Category | Status | Document | Confidence |
|---------------------|--------|----------|------------|
| Core PRD | ✅ COMPLETE | Config_Management_System_PRD.md | HIGH |
| Functional Requirements | ✅ COMPLETE | Config_Management_System_PRD_Functional_Requirements.md | HIGH |
| Data Structures | ✅ COMPLETE | Data_Structure_Requirements.md | HIGH |
| Client Requirements | ✅ COMPLETE | Client_Requirements_Summary.md | HIGH |
| Test Cases | ✅ COMPLETE | Priority_Hierarchy_Test_Cases.md | HIGH |
| Outstanding Issues | 🔄 IN REVIEW | 3 issues documented (see Outstanding_Issues folder) | MEDIUM |

**Overall Status:** Requirements phase COMPLETE, design validated, ready for implementation

---

## Success Criteria

### Phase 1 MVP Success Criteria
- ✅ Config Key Schema Management built and tested
- ✅ Full schema imported (600+ configuration parameters)
- ✅ Hierarchical config engine built and tested for all 6 layers
- ✅ Conditional Rules Framework implemented (Two-Way, Three-Way, Four-Way)
- ✅ Both config management systems operational in parallel
- ✅ 1,000 devices successfully migrated
- ✅ Zero critical incidents related to new config system

### Phase 2 Success Criteria
- ✅ Customer-facing device configuration UI with validation
- ✅ Customers can edit device-level configs (not only Admins)
- ✅ Customer access restricted to device-level only
- ✅ 50,000 devices migrated

### Phase 3 Success Criteria
- ✅ All devices migrated to new config system
- ✅ Old config management system retired
- ✅ Code cleanup completed

---

## Open Questions & Issues

### Outstanding Design Questions (Solutions Proposed)

See `PRD/Outstanding_Issues/` folder for detailed solution documents.

**1. Device Configuration Grouping (HIGH PRIORITY)**
- **Question:** How do admins identify devices with identical configs in dynamic resolution system?
- **Proposed Solution:** Hybrid approach (Profile Hash + Effective Config Hash)
- **Status:** Solution documented, awaiting stakeholder review
- **Document:** `PRD/Outstanding_Issues/Device_Configuration_Grouping_and_Comparison.md`

**2. Configuration Validation Strategy (CRITICAL PRIORITY)**
- **Question:** How to validate configs at scale without bricking thousands of devices?
- **Proposed Solution:** 5-Level Defense-in-Depth validation strategy
- **Status:** Solution documented, awaiting engineering/QA review
- **Document:** `PRD/Outstanding_Issues/Configuration_Validation_Strategy.md`

**3. Configuration Versioning & Change Management (HIGH PRIORITY)**
- **Question:** How to safely manage bulk config changes with review/approval and rollback?
- **Proposed Solution:** Git-like versioning system with Draft→Review→Approve→Activate workflow
- **Status:** Solution documented, awaiting stakeholder review
- **Document:** `PRD/Outstanding_Issues/Configuration_Versioning_and_Change_Management.md`

### Minor Implementation Details (To Be Decided)

**4. Configuration Template Naming**
- **Question:** Should system auto-generate template names or require manual naming?
- **Status:** To be determined in Phase 1 implementation

**5. Effective Hash Recalculation Timing**
- **Question:** Real-time vs queued recalculation when layers change?
- **Status:** To be validated during performance testing

**6. Virtual Filename Display**
- **Question:** Show always or only during migration?
- **Status:** To be decided based on user feedback during migration

---

## Related Documentation

**Supporting Analysis:**
- `02_Research_and_Analysis/` - Evidence supporting requirements
- `02_Research_and_Analysis/Rule_Combinations/` - Validated rule design decisions

**Implementation:**
- `05_Implementation/Implementation_Plans/` - Phased implementation strategy
- `05_Implementation/Technical_Updates/` - Technical clarifications

**User Guidance:**
- `06_Documentation/User_Guides/` - Target users, goals and objectives
- `06_Documentation/System_Documentation/` - Current system behavior

**Discovery:**
- `04_Meetings_and_Discovery/Meeting_Notes/` - Original client requirements

---

## Next Steps

**Requirements Phase:** ✅ COMPLETE

**Ready For:**
1. Technical design (detailed database schema, API design)
2. UI/UX design (mockups, wireframes, user flows)
3. Implementation planning (sprint planning, task breakdown)
4. Development (Phase 1 MVP)

**Outstanding Items:**
1. **Stakeholder review of three outstanding issues:**
   - Device Configuration Grouping solution
   - Configuration Validation Strategy
   - Configuration Versioning & Change Management
2. Engineering validation of performance assumptions (all three solutions)
3. QA validation of test coverage (especially validation strategy)
4. Operations team review of versioning workflow and rollback procedures
5. UI/UX mockups for version management interface
6. Database schema review by engineering (versioning tables)
7. Prioritization of Phase 1 vs Phase 2+ features
8. Creation of implementation tickets

---

**Requirements Status:** COMPLETE - All Design Decisions Validated
**Last Updated:** March 1, 2026
**Document Owner:** Aksana Rahouski
