# Configuration Engine V2 — Requirements

## Overview

The Configuration Engine V2 replaces the current file-based configuration system with a schema-driven parameter management system. Instead of uploading configuration files per device model/carrier/service plan combination, administrators define parameters in a central schema and configure values through a layered inheritance model.

This document defines the scope, architecture, and requirements for the V2 implementation based on client discussions (Meeting 2, March 2026).

---

## Inheritance Hierarchy

Configuration values resolve through 5 levels. Each level can override values from the level above it. The final compiled configuration for a device is the result of merging all applicable layers top-down.

```
1. Schema Defaults
       ↓
2. Model Defaults
       ↓
3. 3-Way Rule (Model + Carrier + Service Plan)
       ↓
4. Company Overrides
       ↓
5. Device Overrides
```

### Level 1: Schema (with Defaults)

The schema is the single source of truth for all configuration parameters. It replaces the need for a separate "global layer" — default values are defined directly on the schema.

**What it stores per parameter:**
- Parameter name / key
- Data type (string, integer, boolean, IP address, etc.)
- Default value (or null if no default)
- Whether the parameter is mandatory
- Validation rules / constraints (e.g., allowed values, min/max)
- Description

**Key behavior:**
- Schema defaults apply system-wide as the baseline
- If no other layer overrides a parameter, the schema default is used
- Null defaults mean the parameter must be explicitly set at a lower level (or left unconfigured)
- Administered via a new admin page ("Grand Schema")

**Parameter metadata (per schema entry):**

| # | Field | Type | Required | Purpose |
|---|-------|------|----------|---------|
| 1 | `parameter_name` | VARCHAR(255), unique | Yes | Unique key identifier (e.g., `wan0_proto`, `wl0_ssid`) |
| 2 | `display_name` | VARCHAR(255) | No | Human-readable label for UI |
| 3 | `description` | TEXT | No | Explanation of what this parameter does |
| 4 | `category` | VARCHAR(100) | Yes | Grouping for UI (Network, Firewall, DHCP, DNS, WiFi, etc.) |
| 5 | `data_type` | ENUM | Yes | `string`, `integer`, `boolean`, `ip_address`, `encrypted` |
| 6 | `default_value` | TEXT | No | Schema-level default — replaces the global layer. Null if no default. |
| 7 | `is_required` | BOOLEAN | Yes | If true, final compiled config must have a non-null value |
| 8 | `is_encrypted` | BOOLEAN | Yes | Sensitive field (passwords), stored with encryption |
| 9 | `validation_regex` | VARCHAR(500) | No | Pattern validation for string types (e.g., IP format, MAC address) |
| 10 | `min_value` | DECIMAL(10,2) | No | Minimum for numeric types (e.g., port numbers, timeouts) |
| 11 | `max_value` | DECIMAL(10,2) | No | Maximum for numeric types |
| 12 | `available_at_company` | BOOLEAN | Yes | Can be overridden at company level |
| 13 | `available_at_device` | BOOLEAN | Yes | Can be overridden at device level (customer-facing) |

**Design decisions — what was removed from V1 and why:**

- **`allowed_at_levels` (JSON)** → Replaced by `available_at_company` and `available_at_device` booleans. Schema level is implicit (all params are in the schema). 3-way rule availability is controlled by model parameter selection, not a schema flag.
- **`has_two_way_rules`, `has_three_way_rules`, `has_four_way_rules`** → Removed. The V1 conditional rule system (11-level priority with 6 two-way combinations) is replaced by the single 3-way rule screen. No per-parameter rule flags needed.
- **`usage_frequency`, `ui_priority`** → Removed. These were analytical metadata from the V1 file audit. In V2, model parameter selection controls what's relevant — not a priority flag on the schema.
- **`related_parameters` (JSON)** → Deferred. Conditional parameter visibility adds UI complexity. Not needed for V2.
- **`help_text`** → Consolidated into `description`. One field is sufficient.

### Level 2: Model Defaults

Model-level default values that apply to ALL configurations for a given model, regardless of carrier or service plan.

**Why this is needed:**

Some parameters are model-specific but constant across all carrier/service plan combinations. For example, `io_enable = 1` for the I-22 — it should always be on for every I-22, whether it's on Verizon or T-Mobile. Without model defaults, you'd have to repeat this value in every 3-way rule for that model, which is redundant and error-prone.

**What it stores:**
- Optional default values for any parameter selected for this model

**Key behavior:**
- Only parameters in the model's selected parameter list can have model defaults
- Model defaults override schema defaults
- A 3-way rule value overrides the model default (carrier/service plan specifics win)
- Not every parameter needs a model default — only set values here when they should be constant across all carrier/service plan combinations for this model
- Managed in the Configuration Admin section (see UI Placement below)

### Level 3: 3-Way Rule (Model + Carrier + Service Plan)

A single configuration screen where administrators select a Model + Carrier + one or more Service Plans and set parameter values that apply to that combination.

**Prerequisites:**
- The model must be flagged as "is configurable" (see Model Configuration below)
- The model must have its parameter list selected from the schema
- Only parameters selected for the model are available to configure on this screen

**What it stores:**
- The model / carrier / service plan(s) combination
- Parameter values that override schema defaults and model defaults for this combination

**Key behavior:**
- Only models configured for the configuration engine appear in the model dropdown
- The parameter list shown is filtered to only what was selected on the model screen
- This replaces file uploads — no more config files, just value entry
- **Service plan is a one-to-many selection.** A single 3-way rule can apply to multiple service plans. For example, one rule for I-22 + Verizon can cover both ATM and Standard service plans if they share the same configuration values. This reduces redundancy — instead of creating identical rules for each service plan, the admin selects all applicable service plans in one rule.
- Multiple 3-way rules can exist for the same model + carrier as long as their service plan selections do not overlap. A service plan cannot belong to more than one 3-way rule for the same model + carrier combination.
- Examples: I22+Verizon+[ATM, Standard], I22+T-Mobile+[ATM], 4100+Verizon+[ATM, IoT Basic, Enterprise]

**Configuration Preview:**

The 3-way rule editor includes a preview function that shows the full compiled configuration as it would exist at this level of the hierarchy — before any company or device overrides. The preview displays ALL schema parameters (all ~697), not just the model's selected subset:

- Parameters with a 3-way rule value show that value (source: 3-Way Rule)
- Parameters with a model default but no 3-way rule value show the model default (source: Model Default)
- Parameters with only a schema default show that default (source: Schema Default)
- Parameters with no value at any level are flagged (and marked as errors if required)

The preview distinguishes between parameters in the model's selected list (shown prominently) and non-model parameters (shown dimmed with their schema defaults). This gives the admin a complete picture of what a device's configuration would look like at this level, and supports export to key-value format for comparison with existing .DAT config files.

### Level 4: Company Overrides (via Override Sets)

Company-level configuration overrides are managed through **named override sets** — reusable collections of parameter overrides that can be assigned to one or many companies.

**Why override sets instead of per-company overrides:**

In the current system, if 10 companies need the same firewall rules (e.g., Cord Financial RMS access), each company gets its own copy of those overrides. Changing one value means updating 10 places. Override sets solve this: create one set, assign it to all 10 companies, edit once.

**What an override set stores:**
- A name (descriptive, e.g., "Cord Financial RMS Access")
- A reference to the 3-way rule it overrides (model + carrier + service plan)
- Parameter values that override the 3-way rule output for the assigned companies

**Key behavior:**
- An override set is always scoped to a specific 3-way rule. The parameter list available in the override set is the same parameter list the model carries — determined by model parameter selection.
- An override set can be assigned to one company (unique override) or many companies (shared override).
- **One override set per 3-way rule per company.** A company cannot have multiple override sets for the same 3-way rule. This avoids needing to resolve conflicts between competing override sets.
- If a company needs mostly the same overrides as a shared set but with a few differences, a new override set must be created for that company.
- Editing an override set updates the configuration for ALL companies assigned to it.
- Sub-companies inherit their parent company's override set assignment by default.
- Sub-companies can be unlinked from the parent and assigned a different override set (or none — falling back to the 3-way rule values).

**Configuration Preview:**

The override set editor includes the same configuration preview function as the 3-way rule editor, but resolves one level deeper. The preview displays ALL ~697 schema parameters with values resolved through four layers: schema default → model default → 3-way rule → company override. This shows the admin exactly what a device's configuration would look like for companies assigned to this override set (before device-level overrides). Supports the same export to key-value format for comparison with existing config files.

**Where it's managed:**
- **Configuration Admin Section:** Primary management — create, edit, browse override sets. View which companies are assigned to each set.
- **Company Edit Page:** Assignment interface — view which override sets the company uses, link to an existing override set, or trigger creation of a new one. The company page is a consumer of override sets, not the owner.

### Level 5: Device Overrides

Device-level parameter overrides. Limited scope — primarily customer-facing settings.

**What it stores:**
- Parameter values that override company-level settings for a specific device

**Key behavior:**
- Limited set of parameters (Wi-Fi, cellular settings, etc.)
- Customers can modify these themselves
- Most parameters are NOT available at this level — only those explicitly allowed
- This level already exists in the current system and remains largely unchanged for V2 scope

---

## Device-Level Configuration Rendering and Legacy Coexistence

### Overview

The V2 configuration engine and the legacy file-based configuration system (.DAT files) will coexist for an extended transition period. Devices will be manually moved from the old system to the new system one at a time. During this period, the device page must support previewing configurations from both systems and clearly indicate which system a device is using.

### Configuration Source Selector

Each device needs a setting indicating which configuration system it uses:

- **Legacy Configuration** — the device uses the current file-based system (.DAT files uploaded per model/carrier/service plan, with custom company configuration files and device-level `custom_configurations` JSON overrides)
- **V2 Configuration** — the device uses the new schema-driven configuration engine (schema defaults → model defaults → 3-way rule → company override set → device overrides)

**Key behavior:**
- The selector is a per-device setting, stored on the device record
- An admin manually switches a device to "V2" when the new configuration has been fully built in the V2 engine for that device's model/carrier/service plan combination
- No automated migration — configurations are manually built in the V2 engine, and devices are manually switched over
- Both systems remain fully operational during the transition period — switching the selector does not delete or modify the device's legacy configuration files
- Switching back to Legacy is always possible since legacy files are preserved

### Device Page Configuration Preview

The device page must be able to preview the device's configuration from both systems:

**Legacy Configuration Preview:**
- A read-only preview of the compiled .DAT file content — showing the key-value pairs from the legacy file after all legacy layers are applied (base config file + custom company config + device custom_configurations)
- This is a temporary preview built to give visibility into the legacy configuration while the old system is still in use

**V2 Configuration Preview:**
- The fully resolved configuration from the V2 engine: all ~697 schema parameters resolved through all 5 levels (schema default → model default → 3-way rule → company override set → device override)
- Uses the same preview pattern as the 3-way rule and override set editors, but resolved through all levels including device overrides
- Each parameter shows its final value and source layer (color-coded badge: schema default, model default, 3-way rule, company override, device override)
- Support the same filters: search, show only overridden, show only missing required
- Support export to key-value format

**Visual indicator on device page:**
- The current configuration source should be prominently displayed (e.g., a dropdown or radio selection: "Legacy" or "V2 Engine")
- The active preview corresponds to whichever system the device is set to use

**Side-by-Side Comparison:**
- Both configuration systems always render a configuration for the device — the source selector only determines which one is actually applied
- A comparison view shows legacy and V2 configurations side by side, with match/difference indicators per parameter
- This allows the admin to see exactly how the two systems differ for this device, regardless of which one is active
- Useful for verifying V2 configuration completeness before switching a device over, and for ongoing auditing

---

## Model Configuration

Model configuration is split across two locations to keep concerns separated:

### Model Edit Page (existing, minor change)

**"Is Configurable" checkbox only:**
- New boolean field on the model record
- When unchecked: model is not part of the configuration engine
- When checked: makes this model available for configuration setup in the Config Admin section
- This is the only config-related change on the model edit page

### Configuration Admin Section (parameter selection + model defaults)

All configuration-specific model setup lives in the Configuration Admin section alongside the Grand Schema and 3-way rules. This keeps all config management in one place rather than splitting it between model admin and config admin.

**Why not on the model edit page?**
- Model management serves many purposes (device specs, manufacturer info, etc.) — configuration is just one concern
- An admin building configurations works in one place: schema → model config → 3-way rules. No jumping between admin sections.
- The model edit page would get bloated with ~700 parameter checkboxes
- Separation of concerns: model admin owns model attributes, config admin owns everything config-related

**What it provides:**

1. **Parameter Selection** — Select which parameters from the schema apply to this model. Different models can have different parameter subsets (e.g., I-22 needs IO parameters that other models don't). This selection determines what appears on the 3-way rule screen and company override screen for this model.

2. **Model Default Values** — For each selected parameter, optionally set a model-level default value. These apply to all configurations for this model regardless of carrier/service plan. Use this for values that are constant across all combinations (e.g., `io_enable = 1` for all I-22s). Parameters without a model default fall through to the schema default.

**Important:** Parameter selection and model defaults are model-only. There is no per-carrier or per-service-plan parameter selection or defaults. The carrier and service plan influence the _values_ (in the 3-way rule), not which parameters exist.

---

## Resolution Logic

When compiling the final configuration for a device, the system resolves each parameter by walking the hierarchy top-down:

```
For each parameter in the device's model parameter list:
  1. Start with schema default
  2. If model default has a value → override
  3. If 3-way rule (device's model + carrier + service plan) has a value → override
  4. If device's company (or inherited parent company) is assigned an override set
     for this 3-way rule, and it has a value for this parameter → override
  5. If device itself has an override → override
  6. Result = final compiled value
```

If a parameter is mandatory (per schema) and still null after resolution, this should be flagged as a configuration error.

---

## Configuration Change Log

All changes to configuration data must be tracked with an audit trail. Every time a configuration entity is created or modified, the system records:

- **Timestamp** — when the change was made
- **User** — who made the change (admin user ID and name)
- **Entity type** — what was changed (Grand Schema, Model Configuration, 3-Way Rule, Company Override Set)
- **Entity identifier** — which specific record (e.g., "I-22 model defaults", "I-22 + Verizon + [ATM, Standard]", "Cord Financial RMS Access")
- **Change summary** — what changed (e.g., "Updated 3 parameter values", "Added service plan: Enterprise", "Assigned 2 companies")

### What is tracked

| Entity | Tracked Changes |
|--------|----------------|
| Grand Schema | Parameter creation, parameter edits (default value, data type, validation, flags), parameter deletion |
| Model Configuration | Parameter selection changes (added/removed parameters), model default value changes |
| 3-Way Rule | Creation, parameter value changes, service plan additions/removals |
| Company Override Set | Creation, parameter override value changes, company assignment/unassignment changes |

### Where change logs are displayed

Each configuration entity's edit/detail page — including the Grand Schema parameter edit page — should show a "Change History" section or tab with a chronological list of changes. Most recent changes first. The Grand Schema browse page should also show a "Recent Schema Activity" feed for at-a-glance visibility into who is modifying the schema.

Each log entry shows:
- Date/time
- User name
- Description of the change

The Configuration Management landing page may also show a "Recent Activity" feed with the latest changes across all configuration entities.

---

## Screens / UI Changes

### New Screens (Configuration Admin Section)

| Screen | Purpose |
|--------|---------|
| Grand Schema | Define all parameters with defaults, types, validation |
| Model Configuration | Per-model parameter selection + model default values |
| Company Override Sets | Create, edit, browse named override sets. View company assignments per set. |

### Modified Screens

| Screen | Changes |
|--------|---------|
| Model Edit | Add "is configurable" checkbox only (simple boolean flag) |
| Configuration Browse/Create | Replace file upload with model+carrier+service plan picker + parameter value editor (3-way rule screen). Includes configuration preview showing all ~697 schema parameters with resolved values and export to key-value format. |
| Company Edit | Add override set assignment section — link to existing override sets or create new ones. Shows which override sets are active for this company, grouped by 3-way rule. |

### Modified Screens (Device)

| Screen | Changes |
|--------|---------|
| Device Edit/View | Add configuration source selector (Legacy vs. V2). Show legacy .DAT preview when on Legacy. Show full 5-level resolved configuration preview with source layer badges when on V2. |

---

## Out of Scope (V2)

Items discussed but explicitly deferred:

- Per-carrier parameter selection (Adam confirmed not needed)
- Per-service-plan parameter selection (Adam confirmed not needed)
- Device-level configuration overrides redesign (the override mechanism itself stays as-is; what changes is the configuration source toggle, preview, and migration workflow)
- Affiliate/distributor commission model (mentioned as future business need)
- Navigation/menu restructuring (acknowledged as needed but separate effort)

---

## Open Questions

1. **Parameter import:** Will the initial ~700 parameters be imported from Adam's spreadsheet, or manually entered via the Grand Schema screen?
2. **Migration path:** How do existing config files map to the new schema? Is there a migration, or do existing configs stay as-is until they're rebuilt in V2?
3. **Mandatory parameter enforcement:** When should mandatory-but-null parameters be flagged? At config creation time? At device provisioning time? Both?
4. ~~**Company config linking:**~~ → Resolved. Override sets replace company-to-company linking. One set, many companies. Edits propagate automatically.
5. **Sub-company unlink behavior:** When a sub-company unlinks from parent's override set, does it fall back to the 3-way rule values (no company override), or does it get a copy of the parent's override set values as a new standalone set?
6. **Schema versioning:** If a parameter is removed from the schema, what happens to existing 3-way rules and override sets that reference it?
7. **`available_at_company` flag:** Is this schema flag still needed as a guardrail to restrict which parameters can appear in company override sets? Or is the model parameter selection sufficient (override sets inherit the model's parameter list from the 3-way rule)?
8. **Override set cross-3-way-rule reuse:** Can an override set apply across multiple 3-way rules (e.g., Cord RMS access needed for both I-22+Verizon+ATM and 4100+Verizon+ATM)? Or must separate override sets be created per 3-way rule, even if the values are identical? (Current proposal: one override set per 3-way rule.)
