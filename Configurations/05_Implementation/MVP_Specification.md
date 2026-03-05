# MVP Specification - Configuration System Phase 1
## Business-Focused Scope Definition

**Document Purpose:** High-level MVP scope for client alignment and roadmap planning
**Status:** Proposed - Awaiting Approval
**Last Updated:** March 4, 2026
**Document Owner:** Aksana Rahouski

---

## Executive Summary

This document defines the **minimum viable product** needed to solve the #1 operational pain point: manually editing hundreds of configuration files for global changes.

**Business Goal:** Enable admins to change configuration settings globally in 5 minutes instead of 5 hours.

**MVP Purpose:** Build and validate the core configuration framework with a limited pilot (up to 10 devices). This is a **proof-of-concept and foundation-building phase**, NOT a production rollout.

**Post-MVP:** Use learnings to build next milestone with enhanced features. Mass device migration will only be considered after the complete feature set is production-ready.

**Risk Level:** LOW - Limited pilot scope, no commitment to mass migration, instant rollback capability

---

## Problem Statement

### Current State Pain Point

**Manual Global Changes Take Hours:**
When an admin needs to change a configuration setting globally (e.g., DNS servers, NTP server, security settings):
- Must open 200+ individual device configuration files
- Manually find and update each parameter
- Update version control timestamps manually
- High risk of errors and inconsistencies
- **Time required:** 5+ hours per change

**Business Impact:**
- Operations team spends excessive time on routine changes
- Configuration drift and inconsistencies across devices
- Delayed response to infrastructure changes
- Error-prone manual process

### Target State (MVP)

**Global Changes in Minutes:**
Admin makes changes through a centralized interface:
1. Open Global Configuration page
2. Update settings once
3. Save
4. Changes automatically propagate to all affected devices

**Business Impact:**
- **95% time reduction** for global configuration changes
- Consistent configuration across all devices
- Reduced operational burden on admin team
- Faster response to infrastructure changes

---

## MVP Implementation Approach

The MVP focuses on building the foundation for dynamic configuration management:

**Four Core Components:**
1. **Configuration Schema Management** - Define and catalog all 600+ configuration parameters with scope classification
2. **Global Configuration Layer** - Create centralized global defaults for truly global parameters
3. **Dynamic Device Configuration Generation** - Build device configs using legacy files + schema + global values + device overrides
4. **Configuration Application Mechanism** - Determine and implement how configs are delivered and applied to devices

**How Device Configuration Works:**
- **Legacy configuration files serve as foundation** - Each device's existing `.dat` file provides complete baseline with all model/carrier/plan-specific settings
- Schema definitions add any new parameters not in legacy files
- Global configuration layer applies changes to "global-only" parameters (scope-aware)
- Legacy file values preserved for "layer-specific" parameters (model/carrier/plan settings)
- Device-specific overrides (Wi-Fi, equipment settings) layer on top as highest priority
- System generates complete, valid configuration guaranteed to have all required parameters
- Configuration must be reliably delivered and applied to devices using validated mechanism

**Why Legacy Files Are Critical:**
Without model/carrier/service plan/customer configuration layers in MVP, legacy files provide the specialized settings that vary by these dimensions. This ensures generated configurations are complete and safe to apply while still enabling global changes.

**Critical MVP Component - Configuration Application:**
The MVP **must** fully design and validate the configuration application mechanism. This includes:
- Understanding current triggers: device updates and device check-ins
- Determining if new system uses same triggers or requires different approach
- Testing and validating that configurations apply safely to devices
- Documenting the chosen mechanism for production use

Without validated configuration application, the MVP cannot prove the system works end-to-end.

---

## MVP Scope: What We're Building

### Core Capability 1: Configuration Schema Management
**Business Value:** Single source of truth for all configuration parameters

Build a centralized parameter schema system that defines all 600+ configuration options:
- Parameter catalog (name, description, purpose)
- Data types and validation rules
- Default values and constraints
- Required vs. optional parameters
- **Parameter scope classification: "global-only" vs. "layer-specific"**

**Parameter Scope Classification:**
- **Global-only parameters**: Can have a single value across all devices (DNS servers, NTP servers, logging endpoints, etc.)
  - These parameters will be available in the global configuration interface
  - When set globally, they override legacy file values
- **Layer-specific parameters**: Vary by model, carrier, service plan, or customer (APN settings, hardware timeouts, carrier-specific configurations, etc.)
  - These parameters are NOT available in global configuration interface (prevents incorrect global overrides)
  - Values are preserved from legacy configuration files
  - Will be managed through future model/carrier/service plan layers (post-MVP)

**User Benefit:** Admins can see all available configuration options in one place with clear definitions, valid value ranges, and understanding of which parameters can be configured globally vs. which require specialized configuration.

---

### Core Capability 2: Global Configuration Layer
**Business Value:** Change settings once, apply everywhere (for truly global parameters)

Build the global configuration layer with admin interface to:
- **Display only "global-only" parameters** (scope-aware interface based on schema)
- Set global default values for global-only parameters
- Prevent setting global values for layer-specific parameters (UI enforces correctness)
- View which devices will be affected by changes
- Track change history (who changed what and when)
- Add notes documenting why changes were made

**Scope Enforcement:**
The global configuration interface will ONLY show parameters marked as `scope: "global-only"` in the schema. This prevents admins from accidentally setting global values for parameters that should vary by model, carrier, service plan, or customer.

**Examples of Global-Only Parameters:**
- DNS servers (primary/secondary)
- NTP servers
- Logging endpoints
- Company-wide security settings
- Network monitoring configurations

**User Benefit:** Admins can make global changes in minutes through a simple, safe interface that prevents incorrect overrides of model/carrier/plan-specific settings.

---

### Core Capability 3: Dynamic Device Configuration Generation
**Business Value:** Automated configuration building using legacy files, schema, and global values

Build configuration generation engine that:
- **Starts with device's legacy configuration file as foundation** (provides complete, working baseline)
- Parses legacy `.dat` files into key-value pairs
- Compares against schema to identify any new parameters
- Applies global values for "global-only" parameters (scope-aware)
- Preserves legacy values for "layer-specific" parameters
- Layers in device-specific overrides (Wi-Fi, equipment settings)
- Generates complete, valid configuration files for devices

**Why Legacy File as Foundation:**
Legacy configuration files contain all model-specific, carrier-specific, service plan-specific, and customer-specific settings that the MVP doesn't have layers for. By starting with the legacy file, we ensure generated configurations are complete and valid while still enabling global changes.

**Configuration Generation Process:**

**Step 1: Load Legacy File**
- Load device's existing configuration file from old system (.dat format)
- Parse into key-value pairs
- This provides complete baseline with ALL required parameters already set correctly

**Step 2: Schema Comparison & Merge**
- Compare legacy file parameters against schema definition
- If schema has NEW parameters not in legacy file: Add with schema default values
- If legacy file has parameters not in schema: Preserve them (backward compatibility)
- Result: Complete parameter set

**Step 3: Apply Global Overrides (Scope-Aware)**
- For parameters marked `scope: "global-only"` in schema:
  - If global configuration has value set: Override legacy file value
  - If not set globally: Keep legacy file value
- For parameters marked `scope: "layer-specific"` in schema:
  - Always keep legacy file value (contains correct model/carrier/plan setting)
  - Global configuration cannot set these (UI prevents it)

**Step 4: Apply Device Overrides**
- For any parameter with device-specific override set:
  - Use device override value (highest priority)
  - Works for both global-only and layer-specific parameters

**Configuration Priority:**

**For Global-Only Parameters (highest to lowest):**
1. Device-specific override - highest priority
2. Global configuration value
3. Legacy file value
4. Schema default value - lowest priority

**For Layer-Specific Parameters (highest to lowest):**
1. Device-specific override - highest priority
2. Legacy file value (contains correct model/carrier/plan setting)
3. Schema default value - lowest priority

**User Benefit:** Configurations are complete, valid, and safe to apply while enabling global changes. The system intelligently preserves specialized settings while allowing global updates where appropriate.

---

### Core Capability 4: Configuration Preview & Comparison (NEW)
**Business Value:** Test and validate before switching devices

Build dual-preview capability for each device:
- **Old Configuration Preview** - Shows existing config file currently in use
- **New Configuration Preview** - Shows dynamically generated config from new system
- **Side-by-side comparison** - Highlight differences between old and new
- **Device flag controls active config** - Toggle between old and new systems

**User Benefit:**
- Test new configuration system without risk
- Validate that generated configs are correct
- Identify any issues before migration
- Build confidence in new system

**Implementation:**
On device detail page, show:
```
┌─────────────────────────────────────────────────────────┐
│ Device #12345 Configuration                              │
├─────────────────────────────────────────────────────────┤
│ Current System: ○ Old Config File  ● New Config System  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│ [Old Configuration]        [New Configuration]          │
│ ┌────────────────────┐    ┌────────────────────┐       │
│ │ dns_primary=8.8.8.8│    │ dns_primary=10.0.1.53      │
│ │ dns_secondary=8.8.4.4   │ dns_secondary=10.0.2.53    │
│ │ ntp_server=time... │    │ ntp_server=time...│       │
│ │ wifi_ssid=Custom   │    │ wifi_ssid=Custom  │       │
│ │ (600+ more params) │    │ (600+ more params)│       │
│ └────────────────────┘    └────────────────────┘       │
│                                                          │
│ [Download Old] [Download New] [Compare Differences]     │
└─────────────────────────────────────────────────────────┘
```

---

### Core Capability 5: Pilot Device Management System
**Business Value:** Zero-risk validation with instant rollback

Enable controlled pilot testing with limited device set:
- Toggle device flag: old config file vs. new config system
- Preview new config before switching
- **Limited to pilot group only (up to 10 devices)**
- Instant rollback if issues detected (just toggle flag back)
- Old system continues working for all non-pilot devices

**User Benefit:** Admin team can thoroughly validate new system with minimal risk. Pilot serves as proof-of-concept, not production rollout.

---

### Core Capability 6: Configuration Application Mechanism
**Business Value:** Reliable, safe delivery of configurations to devices

Design, implement, and validate how configurations are applied to devices in the new system.

**Current System Understanding:**
Today's system has two primary triggers for configuration application:
1. **Device Update Trigger** - Manual or scheduled configuration push to device
2. **Device Check-in Trigger** - Device contacts server during periodic check-in, receives config if changed

**MVP Requirements:**
This capability must be **fully designed and validated** during MVP, including:

**Phase 1: Investigation & Design**
- Document current configuration application process in detail
- Identify how device update triggers work today
- Identify how check-in triggers work today
- Map configuration delivery flow end-to-end
- Identify dependencies and integration points

**Phase 2: Design Decision**
- Determine if new system can use existing triggers (most likely approach)
- Or identify if new triggers/mechanisms are needed (requires justification)
- Document chosen approach with rationale
- Design integration between dynamic config generation and application triggers

**Phase 3: Implementation & Integration**
- Implement configuration application for new system
- Integrate with existing device update trigger mechanism
- Integrate with existing check-in trigger mechanism
- Ensure backward compatibility (old system devices unaffected)

**Phase 4: Validation with Pilot Devices**
- Test device update trigger with pilot devices
- Test check-in trigger with pilot devices
- Verify configurations apply correctly and safely
- Validate device behavior after config application
- Confirm rollback process works (can revert to old config)

**Success Criteria:**
✅ Configuration application mechanism fully documented
✅ Design decision made and documented (use existing triggers vs. new approach)
✅ Integration implemented and tested
✅ Pilot devices successfully receive and apply configurations from new system
✅ Both device update and check-in triggers work reliably
✅ No impact to devices still on old system
✅ Rollback mechanism validated

**User Benefit:** Admin team has confidence that configurations will be delivered reliably and safely to devices. The end-to-end system is proven, not just the configuration generation.

**Why This is Critical:**
Without this capability, the MVP only proves we can *generate* configurations, not that we can *deliver and apply* them to devices. The full value chain must be validated to consider the MVP successful.

---

### Legacy Configuration File Integration
**Business Value:** Ensures complete, valid configurations without building all configuration layers

**Critical Component for MVP Success:**
Legacy configuration files (`.dat` format) are the foundation that makes the MVP viable without building model/carrier/service plan/customer-specific configuration layers.

**Implementation Requirements:**

**1. Legacy File Parser**
- Parse `.dat` files (key=value format) into structured data
- Handle various `.dat` file formats and variations
- Validate file integrity and completeness
- Example file: `ATT_22_01282025.dat`, `VZW_15_02152025.dat`, etc.

**2. Device-to-File Mapping**
- Each device must have a legacy configuration file mapped to it
- Maintain mapping table: device_id → legacy_config_file_path
- For new devices added during MVP: assign appropriate legacy file based on model/carrier/plan
- Legacy files are read-only (never modified by new system)

**3. Schema Comparison Logic**
- Compare legacy file parameters against schema definition
- Identify new parameters in schema not present in legacy file
- Identify deprecated parameters in legacy file not in schema (preserve for backward compatibility)
- Generate complete parameter set combining both sources

**4. Scope-Aware Override Logic**
- Apply global overrides ONLY for parameters marked `scope: "global-only"`
- Preserve legacy file values for parameters marked `scope: "layer-specific"`
- Implement priority hierarchy correctly for each scope type

**5. Legacy File Management**
- Store legacy files in version control or secure storage
- Track which devices use which legacy files
- Provide admin interface to view device's legacy file
- Support updating device's legacy file assignment if needed

**New Devices During MVP:**
All new devices added to the system during MVP must have a legacy configuration file assigned following the same pattern as existing devices. This ensures consistency and completeness.

**Post-MVP Path:**
When model/carrier/service plan/customer layers are built in future milestones, the system will gradually migrate away from legacy file dependency. The specialized layers will replace the values currently sourced from legacy files.

---

## What We're NOT Building (MVP)

To keep scope minimal and delivery fast, these features are **explicitly excluded** from MVP:

❌ **Device Model-Specific Defaults Layer** - Model-specific settings preserved from legacy files, but no dedicated model configuration layer (add in future milestone)
❌ **Carrier-Specific Defaults Layer** - Carrier-specific settings preserved from legacy files, but no dedicated carrier configuration layer (add in future milestone)
❌ **Company-Specific Defaults Layer** - Company-specific settings preserved from legacy files, but no dedicated company configuration layer (add in future milestone)
❌ **Service Plan Defaults Layer** - Service plan settings preserved from legacy files, but no dedicated service plan configuration layer (add in future milestone)
❌ **Conditional Configuration Rules** - No complex multi-way rules (add in Phase 5+)
❌ **Draft/Review/Approve Workflow** - Direct publish only (add in 2027)
❌ **Customer Self-Service Portal** - Admin-only system (add in 2027)
❌ **Advanced Validation & Testing** - Basic validation only (enhance later)
❌ **Device Grouping/Profiles** - All devices managed individually (add in 2027)

---

## User Experience

### For Administrators

#### Today's Workflow (5+ hours)
```
Need to change DNS servers globally:
→ Open git repository
→ Find 200+ device config files
→ Edit each file manually
→ Update version timestamp in each
→ Commit changes
→ Wait for devices to check in (24 hours)
→ Manually verify changes
```

#### Tomorrow's Workflow (5 minutes)
```
Need to change DNS servers globally:
→ Open Global Configuration page
→ Update "Primary DNS Server" field
→ Update "Secondary DNS Server" field
→ Click "Save"
→ Done - system handles the rest automatically
```

---

### For System (Behind the Scenes)

**Dynamic Configuration Generation Process:**

```
Device #12345 requests configuration:

Step 1: Load device's legacy configuration file
  → Locate device's .dat file from old system (e.g., ATT_22_01282025.dat)
  → Parse .dat file into key-value pairs
  → Result: Complete baseline with ALL 600+ parameters already set
  → Includes model-specific, carrier-specific, service plan settings

Step 2: Load and compare schema
  → Load schema definition (600+ parameters with scope classification)
  → Compare schema parameters to legacy file parameters
  → If schema has NEW parameters not in legacy file:
    - Add new parameters with schema default values
  → Result: Complete parameter set including any new capabilities

Step 3: Apply global overrides (scope-aware)
  → For "global-only" parameters (marked in schema):
    - dns_primary: 8.8.8.8 (legacy file) → 10.0.1.53 (global override)
    - dns_secondary: 8.8.4.4 (legacy file) → 10.0.2.53 (global override)
    - ntp_server: (legacy value) → time.google.com (global override)
  → For "layer-specific" parameters (marked in schema):
    - apn_carrier_setting: Keep legacy file value (correct for this carrier)
    - model_timeout: Keep legacy file value (correct for this model)
    - service_plan_limit: Keep legacy file value (correct for this plan)

Step 4: Apply device-specific overrides
  → wifi_ssid: (legacy/global) → "CustomerNetwork" (device override - highest)
  → wifi_password: (legacy/global) → "********" (device override - highest)
  → connected_equipment: (legacy/global) → "ATM_Model_X" (device override)

Step 5: Generate final configuration
  → All parameters resolved using scope-aware priority hierarchy
  → Validation checks applied
  → Complete, valid configuration file generated
  → Configuration is guaranteed complete (legacy file provided baseline)

Step 6: Deliver configuration to device
  → Trigger: Device update request OR device check-in
  → System delivers generated configuration
  → Device receives configuration file

Step 7: Device applies configuration
  → Device validates configuration format
  → Device applies settings
  → Device confirms successful application
  → Device operates with new configuration
```

**Key Insight:**
The MVP validates the complete flow from admin change through device application. Configuration is built using legacy files as foundation (ensuring completeness), then intelligently applies global changes for truly global parameters while preserving model/carrier/plan-specific settings. This ensures configurations are both complete AND allow global management.

**Example End-to-End Flow:**
1. Admin changes global DNS server from 8.8.8.8 to 10.0.1.53
2. System generates new configuration for Device #12345 (AT&T carrier, Model X):
   - Start with device's legacy file (ATT_22_01282025.dat) - all 600+ parameters set
   - Add any new schema parameters not in legacy file
   - Apply global DNS change (dns_primary: 10.0.1.53) - scope: "global-only"
   - Preserve AT&T APN settings from legacy file - scope: "layer-specific"
   - Preserve Model X timeout settings from legacy file - scope: "layer-specific"
   - Apply device-specific Wi-Fi and equipment overrides
3. Device #12345 checks in (or receives update trigger)
4. System delivers new configuration to device
5. Device applies new DNS setting (global change worked!)
6. Device retains correct AT&T and Model X settings (preserved from legacy)
7. Admin verifies DNS change took effect without breaking carrier/model settings

**MVP Success = End-to-End Validation:**
The MVP is successful when we can prove the complete flow works: admin makes change → system generates config → system delivers config → device applies config → change is verified working.

---

## Migration Strategy

### MVP Pilot Approach (No Mass Migration)

**Important Clarification:**
The MVP is **NOT intended for full device migration**. The goal is to build and validate the core configuration framework with a small pilot group, then use lessons learned to build future enhancements before considering broader adoption.

**Phase 1: Build & Test**
- Develop configuration management system
- Import existing configuration parameters
- Set up global defaults
- Internal testing on staging environment

**Phase 2: Limited Pilot (10 Devices Maximum)**
- Select up to 10 low-risk test devices
- Migrate to new configuration system
- **Monitor and validate functionality**
- Admin team tests and provides feedback
- **Pilot devices remain on new system for duration of MVP**

**After MVP Completion:**
- **No Phase 3 expansion** - We will NOT expand beyond the pilot devices
- **No mass migration** - We will NOT migrate additional devices onto MVP system
- **Decision point:** Based on MVP learnings, determine whether to:
  - **Option A:** Migrate pilot devices back to old system while building next milestone
  - **Option B:** Leave pilot devices on new system while building next milestone
  - This decision will be made after MVP validation to minimize risk

**Future State:**
- Next milestone will incorporate MVP learnings and additional required features
- Mass migration will only be considered after full feature set is built
- Old configuration system remains the production standard during MVP phase

**Safety Mechanism:**
At any point during pilot:
- Toggle device back to old system instantly
- No data loss, no device downtime
- Fix issues and retry
- Pilot serves as validation, not as production rollout

---

## Success Criteria

### Functional Success
✅ Admin can change global configuration parameter in <5 minutes (global-only parameters)
✅ Configuration schema includes scope classification (global-only vs. layer-specific) for all 600+ parameters
✅ Legacy `.dat` file parser successfully processes device configuration files
✅ Global configuration interface only shows global-only parameters (scope-aware UI)
✅ Configuration application mechanism fully designed and documented
✅ Changes propagate to pilot devices via device update trigger
✅ Changes propagate to pilot devices via check-in trigger
✅ Global overrides applied correctly (only override global-only parameters)
✅ Legacy file values preserved correctly (layer-specific parameters not overridden)
✅ Device-specific overrides (Wi-Fi, equipment) continue working correctly (highest priority)
✅ Configuration files generated for devices are complete (all 600+ parameters set)
✅ Configuration files generated for devices are valid and accepted by devices
✅ Devices apply configurations successfully without errors
✅ End-to-end flow validated: admin change → legacy file + schema merge → scope-aware override → generation → delivery → application → device verification

### Operational Success
✅ Up to 10 pilot devices successfully validated on new system
✅ Old system continues working for all non-pilot devices (production standard)
✅ Admin team trained and comfortable using new system
✅ Zero production incidents caused by new system
✅ Pilot provides clear learnings for next milestone development

### Business Success
✅ **95% time reduction** for global configuration changes (5 hours → 5 minutes)
✅ Reduced configuration errors and inconsistencies
✅ Admin team satisfaction with new system
✅ Foundation established for future enhancements (model/carrier/company layers)

---

## Risks & Mitigation

### Risk 1: Configuration Errors Impact Devices
**Likelihood:** Low (legacy files provide proven baseline)
**Impact:** High
**Mitigation:**
- Start with legacy configuration files (already working on devices)
- Only override truly global parameters (scope-aware logic prevents incorrect overrides)
- Start with only 10 devices for pilot
- Thorough testing before each expansion
- Instant rollback capability
- Device-side validation rejects invalid configs
- Configuration comparison tool shows exactly what changed

### Risk 2: Admin Team Adoption
**Likelihood:** Low
**Impact:** Medium
**Mitigation:**
- Hands-on training sessions
- Clear documentation with examples
- Demonstrate time savings
- Gradual migration builds confidence

### Risk 3: Configuration Application Mechanism Design
**Likelihood:** Low (using existing triggers reduces risk)
**Impact:** High (must work reliably for MVP success)
**Mitigation:**
- Early investigation of existing trigger mechanisms
- Document current system thoroughly before designing integration
- Design review with technical team before implementation
- Phased validation: test in staging before pilot devices
- Extensive testing with pilot devices before declaring success
- Document chosen approach with detailed rationale

### Risk 4: Legacy File Parsing and Integration
**Likelihood:** Medium
**Impact:** Medium
**Mitigation:**
- Analyze multiple legacy `.dat` file formats early
- Build robust parser that handles format variations
- Test parser with diverse sample files
- Validate parsed data against schema
- Maintain device-to-file mapping carefully

### Risk 5: Parameter Scope Miscategorization
**Likelihood:** Medium
**Impact:** Medium
**Mitigation:**
- Conservative approach: Default to "layer-specific" when uncertain
- Review categorization with subject matter experts
- Test with pilot devices to validate correct behavior
- Allow recategorization in future milestones if needed
- Document rationale for each categorization decision

---

## Deliverables

### Development Complete
- Configuration schema management system functional with scope classification (global-only vs. layer-specific)
- Legacy `.dat` file parser implemented and tested
- Device-to-legacy-file mapping system functional
- Global configuration layer with admin interface (scope-aware - only shows global-only parameters)
- Dynamic device configuration generation engine with legacy file integration
- Configuration preview and comparison capability (old vs. new side-by-side)
- Migration management interface
- **Configuration application mechanism fully designed, documented, and implemented**
- **Integration with device update and check-in triggers complete**

### Pilot Ready
- Up to 10 devices migrated and validated
- **Configuration application tested via device update trigger**
- **Configuration application tested via check-in trigger**
- **End-to-end flow validated (generate → deliver → apply → verify)**
- Admin team trained on basic operations
- Monitoring and rollback procedures documented

### MVP Complete
- Pilot devices operating successfully on new configuration system
- **Configuration application mechanism proven in production with pilot devices**
- **Both device update trigger and check-in trigger validated and working**
- **Legacy file integration validated** - configurations complete and devices operating correctly
- **Scope-aware global overrides validated** - global changes applied without breaking layer-specific settings
- Old system remains operational as production standard
- Schema with scope classification, global layer, legacy file integration, dynamic generation, and application delivery validated end-to-end
- Global change tested and verified on pilot devices (change made, applied, validated on devices, layer-specific settings preserved)
- Full documentation delivered including:
  - Configuration application design and rationale
  - Legacy file integration approach
  - Parameter scope categorization decisions
  - Device-to-file mapping procedures
- Admin team fully trained and confident
- **Post-MVP decision:** Determine whether to migrate pilot devices back to old system or leave on new system while building next milestone
- Lessons learned documented for next milestone development, including:
  - Configuration application insights
  - Legacy file integration challenges and solutions
  - Parameter scope categorization learnings

---

## Post-MVP Direction

### MVP as Foundation, Not Final Product

**Key Understanding:**
The MVP is designed to **validate core concepts** and **build the foundation**, but is **NOT the complete production system**. After MVP validation, we will build the next milestone with:
- Enhanced features based on MVP learnings
- Additional configuration layers (model/carrier/company-specific)
- More sophisticated rule capabilities
- Production-ready feature set

**Post-MVP Options for Pilot Devices:**

**Option A: Migrate Back to Old System**
- Move pilot devices back to old configuration system
- Keeps all devices on consistent production standard
- Eliminates need to maintain two systems during development
- Minimizes risk during next milestone development

**Option B: Leave on New System**
- Pilot devices stay on MVP configuration system
- Provides ongoing validation during next milestone development
- Requires maintaining both systems in parallel
- May create support complexity

**Decision Criteria:**
- Stability of pilot devices on MVP system
- Risk tolerance during next milestone development
- Resource availability to maintain dual systems
- Business priority for consistency vs. ongoing validation

**This decision will be made after MVP validation, not now.**

### Development Path Forward

After MVP completion:
1. **Evaluate MVP results** - What worked well? What needs improvement?
2. **Document lessons learned** - Capture insights for next milestone
3. **Define next milestone scope** - What additional features are needed?
4. **Decide pilot device fate** - Option A or Option B based on evaluation
5. **Build next milestone** - Incorporate MVP learnings and expand capabilities
6. **Future consideration:** Mass migration only after full feature set is production-ready

**Mass device migration is NOT planned as part of MVP or immediate post-MVP phases.**

---

## Next Steps

### Immediate
1. **Client review and approval** of this MVP scope
2. **Answer open questions:**
   - Which devices should be in pilot group?
   - Who from admin team should be involved in testing?
   - What is the ideal timeframe for rollout?

### Following
3. **Detailed planning:**
   - Technical design document (for development team)
   - UI/UX mockups (for client review)
   - Test plan (for QA team)

4. **Kickoff:**
   - Development sprint planning
   - Set up development environment
   - Begin build phase

---

## Appendix: Why This MVP Scope is Right-Sized

### Too Small Would Be:
- Just documenting parameters without ability to change them
- Just building UI without connecting to devices
- Just global defaults without respecting device overrides

### Too Large Would Be:
- Adding model/carrier/company layers (significant complexity increase)
- Building conditional rule system (adds complexity)
- Creating draft/review/approve workflow (not needed yet)
- Planning for mass device migration (not MVP goal - validation only)

### This MVP is Just Right Because:
✅ Solves the #1 pain point (5 hours → 5 minutes for global changes)
✅ Builds the foundation: schema + global layer + dynamic generation
✅ Low risk approach (limited pilot with up to 10 devices)
✅ Validates core concepts before building complete feature set
✅ Provides learnings for next milestone development
✅ No commitment to mass migration - pilot serves as validation only

---

**Document Status:** Business-Focused MVP Scope - Ready for Client Review
**Next Action:** Client approval of scope and timeline
**Document Owner:** Aksana Rahouski
**Last Updated:** March 4, 2026
