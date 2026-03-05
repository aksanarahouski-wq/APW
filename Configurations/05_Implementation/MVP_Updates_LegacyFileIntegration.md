# MVP Specification Updates - Legacy File Integration & Scope-Aware Configuration

**Date:** March 4, 2026
**Change Type:** Major Architecture Update
**Document Updated:** `MVP_Specification.md`

---

## Executive Summary

Major update to MVP configuration generation approach to solve the **completeness problem**: configurations generated from schema + global + device overrides alone are incomplete because MVP doesn't include model/carrier/service plan/customer-specific configuration layers.

**Solution:** Use existing legacy configuration files (`.dat` format) as the foundation, providing complete baseline configurations that include all specialized settings. Apply global changes scope-aware manner to enable global management without breaking specialized configurations.

---

## The Problem Identified

### Original MVP Approach Gap

**Original approach:**
```
Configuration = Schema Defaults + Global Values + Device Overrides
```

**Problem:**
- Many parameters vary by model, carrier, service plan, or customer
- These layers are NOT being built in MVP (future milestone)
- Configurations generated would be **incomplete** (missing critical parameters)
- **Cannot safely apply incomplete configurations to devices**

**Example gaps:**
- Carrier-specific APN settings
- Model-specific timeout values
- Service plan bandwidth limits
- Customer-specific security configurations

### Root Cause

MVP scope intentionally excludes:
- ❌ Model-specific defaults layer
- ❌ Carrier-specific defaults layer
- ❌ Service plan defaults layer
- ❌ Customer/company-specific defaults layer

Without these layers, there's no way to set the parameters that vary by these dimensions.

---

## The Solution: Legacy File Integration

### New Configuration Generation Approach

**Updated approach:**
```
Configuration = Legacy File (baseline) + Schema (new params) + Global (scope-aware) + Device Overrides
```

### How It Works

**Step 1: Start with Legacy Configuration File**
- Every device has an existing `.dat` file from the old system
- File contains ALL 600+ parameters with working values
- Includes model-specific, carrier-specific, service plan-specific settings
- Provides complete, proven baseline

**Step 2: Merge with Schema**
- Compare legacy file to schema definition
- If schema has NEW parameters not in legacy file: Add with schema defaults
- If legacy file has parameters not in schema: Preserve for backward compatibility
- Result: Complete parameter set

**Step 3: Apply Global Overrides (Scope-Aware)**
- Parameters categorized as `scope: "global-only"` → Apply global override
- Parameters categorized as `scope: "layer-specific"` → Keep legacy file value
- This prevents incorrect global overrides of specialized settings

**Step 4: Apply Device Overrides**
- Device-specific overrides always win (highest priority)
- Works for both global-only and layer-specific parameters

---

## Key Innovation: Parameter Scope Classification

### Schema Enhancement

Every parameter in schema now has a `scope` attribute:

```json
{
  "dns_primary": {
    "type": "string",
    "default": "8.8.8.8",
    "scope": "global-only",  // ← Safe to apply globally
    "description": "Primary DNS server"
  },
  "apn_carrier_setting": {
    "type": "string",
    "default": "internet",
    "scope": "layer-specific",  // ← Varies by carrier
    "description": "Carrier APN configuration"
  }
}
```

### Scope Types

**Global-Only Parameters:**
- Can have single value across all devices
- Examples: DNS servers, NTP servers, logging endpoints, company-wide security settings
- Displayed in global configuration UI
- Global values override legacy file values

**Layer-Specific Parameters:**
- Vary by model, carrier, service plan, or customer
- Examples: APN settings, carrier timeouts, hardware configurations, plan limits
- NOT displayed in global configuration UI (prevents mistakes)
- Legacy file values preserved (contain correct specialized settings)

### UI Enforcement

Global configuration interface is **scope-aware**:
- Only shows parameters marked `scope: "global-only"`
- Admin cannot accidentally set global value for layer-specific parameter
- Prevents incorrect overrides that would break device functionality

---

## Priority Hierarchies

### For Global-Only Parameters
1. **Device-specific override** (if set) - highest priority
2. **Global configuration value** (if set)
3. **Legacy file value**
4. **Schema default value** - lowest priority

### For Layer-Specific Parameters
1. **Device-specific override** (if set) - highest priority
2. **Legacy file value** (contains correct model/carrier/plan setting)
3. **Schema default value** - lowest priority

Note: Global configuration CANNOT set layer-specific parameters (UI prevents it)

---

## Updated Core Capabilities

### Core Capability 1: Configuration Schema Management
**Added:**
- Parameter scope classification (global-only vs. layer-specific)
- Categorization of all 600+ parameters during schema creation
- Scope-based access control for global configuration

### Core Capability 2: Global Configuration Layer
**Added:**
- Scope-aware UI (only shows global-only parameters)
- Prevention of global overrides for layer-specific parameters
- Examples of global-only vs. layer-specific parameters

### Core Capability 3: Dynamic Device Configuration Generation
**Completely restructured:**
- Legacy file as foundation (Step 1)
- Schema comparison and merge (Step 2)
- Scope-aware global override logic (Step 3)
- Device override application (Step 4)
- Separate priority hierarchies by scope type

### New: Legacy Configuration File Integration
**Brand new capability:**
- Parse `.dat` files into key-value pairs
- Device-to-file mapping system
- Schema comparison logic
- Scope-aware override engine
- Legacy file management

---

## Technical Implementation Requirements

### 1. Legacy File Parser
- Parse `.dat` file format (key=value pairs)
- Handle format variations across different file types
- Validate file integrity
- Convert to structured format for processing

### 2. Device-to-File Mapping
- Maintain mapping: device_id → legacy_config_file_path
- Support file assignment for new devices
- Track which devices use which files
- Provide admin interface to view/update mappings

### 3. Scope Classification System
- Schema definition includes scope attribute
- Categorization decisions for all 600+ parameters
- Documentation of categorization rationale
- Ability to recategorize in future milestones

### 4. Scope-Aware Override Engine
- Different logic paths for global-only vs. layer-specific
- Global configuration only processes global-only parameters
- Legacy file values protected for layer-specific parameters
- Device overrides work for both scope types

### 5. Configuration Comparison
- Side-by-side view: old (legacy file) vs. new (generated)
- Highlight what changed (global overrides, device overrides, new parameters)
- Validate completeness before application

---

## Updated System Flow

### Old Flow (Original MVP)
```
Admin change → Schema defaults + Global values + Device overrides → Generate config → Apply
```
**Problem:** Generated config is incomplete

### New Flow (Legacy File Integration)
```
Admin change
  ↓
Load device's legacy .dat file (complete baseline)
  ↓
Compare to schema (add any new parameters)
  ↓
Apply global overrides (scope-aware: only global-only parameters)
  ↓
Preserve legacy values (layer-specific parameters)
  ↓
Apply device overrides (highest priority, all scopes)
  ↓
Generate complete, valid configuration
  ↓
Apply to device
```
**Result:** Complete configuration that enables global management

---

## Example Scenario

### Scenario: Change DNS Globally for AT&T Device

**Device: #12345**
- Carrier: AT&T
- Model: Model X
- Service Plan: Premium
- Legacy file: `ATT_22_01282025.dat`

**Admin Action:**
Change global DNS from 8.8.8.8 to 10.0.1.53

**Configuration Generation Process:**

**Step 1: Load legacy file**
```
dns_primary=8.8.8.8           (will be overridden - global-only)
dns_secondary=8.8.4.4         (will be overridden - global-only)
apn_setting=broadband         (preserved - layer-specific, AT&T)
timeout_value=120             (preserved - layer-specific, Model X)
bandwidth_limit=1000          (preserved - layer-specific, Premium plan)
... (600+ more parameters)
```

**Step 2: Compare to schema**
```
Schema has new parameter: security_log_endpoint
Add: security_log_endpoint=logs.company.com (schema default)
```

**Step 3: Apply global overrides (scope-aware)**
```
dns_primary: 8.8.8.8 → 10.0.1.53 (scope: global-only, override!)
dns_secondary: 8.8.4.4 → 10.0.2.53 (scope: global-only, override!)
apn_setting: Keep broadband (scope: layer-specific, preserve!)
timeout_value: Keep 120 (scope: layer-specific, preserve!)
bandwidth_limit: Keep 1000 (scope: layer-specific, preserve!)
```

**Step 4: Apply device overrides**
```
wifi_ssid: (legacy/global) → "CustomerNetwork" (device override)
wifi_password: (legacy/global) → "********" (device override)
```

**Result:**
- ✅ DNS changed globally (worked!)
- ✅ AT&T APN setting preserved (not broken!)
- ✅ Model X timeout preserved (not broken!)
- ✅ Premium plan limit preserved (not broken!)
- ✅ Device-specific Wi-Fi settings applied
- ✅ New security logging enabled
- ✅ All 600+ parameters present (complete!)

---

## Risk Mitigation Updates

### Risk 1: Configuration Errors Impact Devices
**Before:** Medium likelihood
**After:** Low likelihood

**Why risk reduced:**
- Legacy files provide proven baseline (already working on devices)
- Scope-aware logic prevents incorrect overrides
- Only changing truly global parameters

### New Risk: Legacy File Parsing and Integration
**Likelihood:** Medium
**Impact:** Medium

**Mitigation:**
- Analyze multiple file formats early
- Build robust parser with format variation handling
- Test with diverse sample files
- Validate parsed data against schema

### New Risk: Parameter Scope Miscategorization
**Likelihood:** Medium
**Impact:** Medium

**Mitigation:**
- Conservative approach: Default to "layer-specific" when uncertain
- Review with subject matter experts
- Test with pilot devices
- Allow recategorization in future milestones
- Document rationale for decisions

---

## Success Criteria Updates

### Functional Success - Added
✅ Configuration schema includes scope classification for all 600+ parameters
✅ Legacy `.dat` file parser successfully processes device configuration files
✅ Global configuration interface only shows global-only parameters
✅ Global overrides applied correctly (only override global-only parameters)
✅ Legacy file values preserved correctly (layer-specific parameters not overridden)
✅ Configuration files generated are complete (all 600+ parameters set)

### MVP Complete - Added
✅ Legacy file integration validated
✅ Scope-aware global overrides validated
✅ Global changes applied without breaking layer-specific settings
✅ Documentation includes parameter scope categorization decisions
✅ Lessons learned include legacy file integration challenges and solutions

---

## What We're NOT Building (Updated)

**Original exclusions:**
- ❌ Model-Specific Defaults - All models share same global defaults
- ❌ Carrier-Specific Defaults - All carriers share same global defaults
- ❌ Company-Specific Defaults - All companies share same global defaults
- ❌ Service Plan Defaults - All service plans share same global defaults

**Updated exclusions (clarified):**
- ❌ Model-Specific Defaults **Layer** - Model-specific settings **preserved from legacy files**, but no dedicated model configuration layer
- ❌ Carrier-Specific Defaults **Layer** - Carrier-specific settings **preserved from legacy files**, but no dedicated carrier configuration layer
- ❌ Company-Specific Defaults **Layer** - Company-specific settings **preserved from legacy files**, but no dedicated company configuration layer
- ❌ Service Plan Defaults **Layer** - Service plan settings **preserved from legacy files**, but no dedicated service plan configuration layer

**Key difference:**
We're not building the layers, but we're not losing the specialized settings either. Legacy files bridge the gap.

---

## New Devices During MVP

**Requirement:** All new devices added during MVP must have a legacy configuration file assigned.

**Process:**
1. Identify appropriate legacy file template based on model/carrier/plan
2. Assign file to new device in device-to-file mapping
3. Follow same configuration generation process as existing devices

**Rationale:**
Ensures consistency - all devices have complete baseline configurations.

---

## Post-MVP Path

### When Model/Carrier/Service Plan Layers Are Built

Future milestones will add dedicated configuration layers:
- Model-specific defaults layer
- Carrier-specific defaults layer
- Service plan defaults layer
- Customer/company-specific defaults layer

**Migration from Legacy Files:**
As these layers are built, settings currently sourced from legacy files will be migrated to appropriate layer:
- APN settings → Carrier layer
- Timeout values → Model layer
- Bandwidth limits → Service Plan layer
- Security configs → Customer layer

**Legacy File Sunset:**
Eventually, legacy files will no longer be needed as foundation. They'll serve as reference/validation only.

---

## Implementation Complexity Impact

### Additional Work Required

**Schema Development:**
- Categorize all 600+ parameters as global-only vs. layer-specific
- Document rationale for each categorization decision
- Review with subject matter experts
- **Estimated effort:** 2-3 weeks

**Parser Development:**
- Analyze `.dat` file format variations
- Build robust parser
- Test with diverse samples
- **Estimated effort:** 2 weeks

**Mapping System:**
- Build device-to-file mapping system
- Admin interface for viewing/managing mappings
- **Estimated effort:** 1 week

**Scope-Aware Logic:**
- Implement different override logic for each scope type
- Build scope-aware UI filters
- **Estimated effort:** 2 weeks

**Total additional effort:** 7-8 weeks (some parallel work possible)

### Benefits vs. Complexity

**Benefits:**
- ✅ Generated configurations are complete and safe to apply
- ✅ Enables global configuration changes (core MVP value)
- ✅ Preserves specialized settings without building complex layers
- ✅ Reduces risk of device errors
- ✅ Provides proven baseline for each device

**Complexity Added:**
- ⚠️ Parameter categorization effort (one-time)
- ⚠️ Parser implementation
- ⚠️ Device-to-file mapping system
- ⚠️ Scope-aware override logic

**Verdict:** Additional complexity is justified. Without this, MVP cannot generate complete configurations, which means it cannot be validated with real devices. The alternative (building full model/carrier/plan layers) would add 6+ months of work.

---

## Key Decisions Made

### Decision 1: Use Legacy Files as Foundation
**Rationale:** Provides complete configurations without building all specialized layers

### Decision 2: Scope Classification in Schema
**Rationale:** Enables safe global management while protecting specialized settings

### Decision 3: UI Enforcement of Scope
**Rationale:** Prevents admin mistakes by only showing global-only parameters in global UI

### Decision 4: Conservative Categorization Default
**Rationale:** When uncertain, default to "layer-specific" (safer - preserves legacy value)

### Decision 5: Allow Recategorization Post-MVP
**Rationale:** Acknowledges learning curve; enables corrections in future milestones

---

## Documentation Requirements

### Must Document

1. **Parameter Scope Categorization:**
   - List of all 600+ parameters with scope classification
   - Rationale for each categorization decision
   - Review and approval by subject matter experts

2. **Legacy File Format:**
   - `.dat` file format specification
   - Parsing rules and edge cases
   - Sample files for testing

3. **Device-to-File Mapping:**
   - How mappings are created and maintained
   - Process for new device file assignment
   - Admin procedures for viewing/updating mappings

4. **Scope-Aware Override Logic:**
   - Detailed flowcharts for each scope type
   - Examples showing how priority works
   - Edge cases and how they're handled

5. **Testing Procedures:**
   - How to validate scope categorization
   - How to verify legacy file integration
   - How to test global changes don't break specialized settings

---

## Questions Answered

**Q1: Legacy file per device?**
✅ Yes, every device has its own legacy file

**Q2: File format?**
✅ Yes, `.dat` files need to be parsed into key-value pairs

**Q3: New keys in schema?**
✅ Yes, add missing keys with schema defaults

**Q4: Global override behavior?**
✅ Scope-aware: only override global-only parameters, preserve layer-specific

**Q5: Modification to legacy files?**
✅ Correct, legacy files are read-only inputs

**Q6: Fallback for new devices?**
✅ New devices must have legacy file assigned (same pattern as existing)

---

## Document Control

**Changes Made By:** Aksana Rahouski
**Date:** March 4, 2026
**Reason:** Client clarification that configurations must be complete and safe to apply; legacy files provide specialized settings that MVP layers won't cover
**Review Status:** Updated based on client requirements
**Next Action:** Client review and approval of legacy file integration approach
