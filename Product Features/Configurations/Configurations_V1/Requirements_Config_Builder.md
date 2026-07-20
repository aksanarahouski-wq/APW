# Configuration Engine V1 — Requirements

## Overview

The Configuration Engine V1 replaces the current file-based configuration system with a schema-driven parameter management system. Instead of uploading configuration files per device model/carrier/service plan combination, administrators define parameters in a central schema and configure values through a layered inheritance model.

The current system manages approximately 700 configuration parameters. This document defines the scope, architecture, and requirements for the V1 implementation based on client discussions (Meeting 2, March 2026).

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

> **Decision:** Schema is append-only for MVP. Parameters cannot be removed from the schema. If removal is ever needed, the client will consult with the dev team. (Source: Adam, Confluence Apr 17 2026)

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
| 8 | `is_encrypted` | BOOLEAN | Yes | Sensitive field (passwords), stored with encryption. Values are masked ('********') in all previews, change logs, and exports — never readable after save, only editable. |
| 9 | `validation_regex` | VARCHAR(500) | No | Pattern validation for string types (e.g., IP format, MAC address) |
| 10 | `min_value` | DECIMAL(10,2) | No | Minimum for numeric types (e.g., port numbers, timeouts) |
| 11 | `max_value` | DECIMAL(10,2) | No | Maximum for numeric types |
| 12 | `available_at_company` | BOOLEAN | Yes | Can be overridden at company level |
| 13 | `available_at_device` | BOOLEAN | Yes | Can be overridden at device level (customer-facing) |

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

> **Decision:** Cellular backup is NOT a matching dimension. The legacy system used it as a 4th criterion, but APW stopped utilizing that flag and switched to a customer-driven, device-page-initiated approach. The 3-way rule (Model + Carrier + Service Plan) is confirmed correct. (Source: Adam, Confluence Apr 17 2026)

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
- **Splitting is manual.** When service plans in a grouped 3-way rule need to diverge (e.g., ATM and Standard previously shared values but now need different configs), the admin manually removes the service plan from the existing rule and creates a new rule for it. No clone/split tooling is needed.
- Examples: I22+Verizon+[ATM, Standard], I22+T-Mobile+[ATM], 4100+Verizon+[ATM, IoT Basic, Enterprise]

**Configuration Preview:**

The 3-way rule editor includes a preview function that shows the full compiled configuration as it would exist at this level of the hierarchy — before any company or device overrides. The preview displays all schema parameters (currently ~700), not just the model's selected subset:

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
- A scope definition: Model, Carrier, and Service Plan selectors — each can be a specific value or "Any" (applies regardless of that dimension)
- Parameter values that override any value from higher levels (schema defaults, model defaults, or 3-way rule values) for the assigned companies' matching devices

**Override Set Scoping:**

Each override set defines which devices it applies to using three scope selectors:

| Selector | Options | Effect |
|----------|---------|--------|
| Model | Specific model (e.g., I-22) or **Any** | When specific: applies only to devices of that model. When Any: applies to all models. |
| Carrier | Specific carrier (e.g., Verizon) or **Any** | When specific: applies only to devices on that carrier. When Any: applies to all carriers. |
| Service Plan | Specific plan(s) (multi-select, e.g., ATM, Standard) or **Any** | When specific: applies only to devices on those plans. When Any: applies to all service plans. |

**"Any" is a perpetual wildcard:** The "Any" option matches all current AND future values for that dimension. If a new service plan, carrier, or model is added to the system, it automatically falls under existing override sets that use "Any" for that selector. There is no snapshot behavior — "Any" always means "everything that exists at resolution time."

**Examples of override set scoping:**

| Override Set | Model | Carrier | Service Plan | Real-World Use |
|---|---|---|---|---|
| Cord Firewall | Any | Any | Any | Cord's firewall rules apply to all their devices regardless of type |
| Baltech Scheduler | Any | Any | Any | Nightly power cycle at 3am for all Baltech devices |
| Allen Hoover Data Limit | Any | Any | Tier 3 | Daily usage cap for one specific plan across all devices |
| I-22 Power Config | I-22 | Any | Any | IO-specific override for one model, all carriers/plans |
| Miele VZW ATM Config | I-22 | Verizon | ATM | Fully scoped like a traditional 3-way rule override |

**Parameter list availability:**
- When the override set specifies a **specific model**: the parameter list comes from that model's selected parameters (same as today). Only parameters selected for that model can be overridden.
- When the override set specifies **Any model**: the parameter list is all schema parameters flagged as `available_at_company = true`. Since there is no single model to inherit from, the schema flag controls which parameters are available. At resolution time, only parameters that belong to the device's actual model are applied — others are silently ignored. *(Open question: should the editor surface which models each parameter applies to? See Open_Questions.md Q2.)*

**Key behavior:**
- An override set can be assigned to one company (unique override) or many companies (shared override).
- **Multiple override sets per company are allowed**, as long as their parameters do not conflict for overlapping device scopes (see Conflict Detection below).
- Editing an override set updates the configuration for ALL companies assigned to it.
- Sub-companies inherit their parent company's override set assignments by default (see Distributor Sub-Company Inheritance below).
- Sub-companies can be unlinked from the parent and assigned different override sets (or none — falling back to the 3-way rule values).

**Distributor Sub-Company Inheritance:**

When a distributor company is assigned an override set, an "Apply to sub-companies" flag controls whether sub-companies automatically inherit the assignment:

- **When enabled:** All current and future sub-companies automatically inherit the parent's override set. New sub-companies added under the distributor get the override set automatically.
- **Unlink:** Any individual sub-company can be unlinked from the parent's override set. When unlinked, the sub-company falls back to the 3-way rule values (no company-level override) unless it has its own separate override set assigned.
- **Re-link:** An unlinked sub-company can be put back on the parent's override set.
- **Own override set:** A sub-company can be assigned a different override set while the parent's "apply to sub-companies" is active. This acts as an implicit unlink — the sub-company's explicit assignment wins over the parent's inheritance.
- **No nesting:** The system does not support nested distributor hierarchies. Inheritance is one level only: distributor → direct sub-companies.
- **Scope filtering at resolution time:** Inheritance is blanket — all sub-companies inherit the assignment. Whether the override set actually affects a sub-company's devices depends on scope matching at resolution time (e.g., if the override set is scoped to Verizon, only Verizon devices in the sub-company are affected). The UI should show effective reach as **companies**, not devices (e.g., "applies to 12 of 47 companies"). (Source: Adam, Confluence Apr 17 2026)

**Build vs. Assign Separation:**

Override set lifecycle is split into two distinct phases: **building** (defining scope + parameters + values) and **assigning** (attaching companies). This separation enables clean conflict validation.

- **Building** — An override set can be created and edited without any company assignments. An unassigned override set is inactive — it has no effect on any device. No validation is needed during the build phase because there are no companies to validate against.
- **Assigning** — Companies are attached to an override set one at a time. Each assignment triggers conflict validation against that specific company's existing override sets. This is the enforcement point.
- **Minimum assignment** — An override set with no companies assigned is valid but inactive. It will not be compiled or pushed to any device. This allows admins to build and review override sets before activating them.

**Conflict Detection (at assignment time):**

When assigning an override set to a company, the system checks all other override sets already assigned to that company. The assignment is **rejected** if:
1. The new override set's scope overlaps with an existing override set's scope (i.e., there exists at least one possible device that matches both scopes), AND
2. Both override sets contain at least one parameter in common.

Scope overlap is determined by comparing each selector:
- "Any" overlaps with everything (Any overlaps with Any, Any overlaps with specific values)
- A specific value overlaps with the same specific value or with "Any"
- Two different specific values do NOT overlap (e.g., Model=I-22 and Model=4100 never match the same device)

**Example — assignment allowed:**
- Company Cord has: "Cord Firewall" (Any + Any + Any) → sets `fw_acl`, `fw_enable`, `fw_rule_count`
- Admin assigns: "Baltech Scheduler" (Any + Any + Any) → sets `scheduler`, `sched_time`
- Scopes overlap (both are Any/Any/Any), but parameters are completely different → **allowed**

**Example — assignment rejected:**
- Company Cord has: "Cord Firewall" (Any + Any + Any) → sets `fw_acl`, `fw_enable`
- Admin tries to assign: "I-22 Power Config" (I-22 + Any + Any) → sets `io_enable`, `fw_acl`
- Scopes overlap (Any/Any/Any overlaps with I-22/Any/Any for any I-22 device), and both set `fw_acl` → **rejected** with error: "Parameter 'fw_acl' conflicts with existing override set 'Cord Firewall' for overlapping device scopes."

**Example — assignment allowed (no scope overlap):**
- Company has: "VZW Firewall" (Any + Verizon + Any) → sets `fw_acl`
- Admin assigns: "TMO Firewall" (Any + T-Mobile + Any) → sets `fw_acl`
- Scopes do NOT overlap (Verizon and T-Mobile never match the same device) → **allowed**, even though both set `fw_acl`

**Conflict Detection (on override set edits):**

When editing an override set that already has companies assigned (changing parameters, values, or scope), the edit saves successfully — the override set itself is always valid. After saving, the system runs a re-validation across all assigned companies:

1. For each assigned company, check their OTHER override sets for parameter + scope overlap against the updated set.
2. If conflicts are found, the conflicting companies are flagged in a validation report showing: which company, which parameter conflicts, and which other override set causes the conflict.
3. Flagged companies remain assigned but are marked as "conflict — needs resolution." The override set will not be compiled or pushed for those companies until the conflict is resolved.
4. The admin must resolve conflicts by either: editing one of the conflicting override sets to remove the parameter overlap, or unassigning the company from one of the conflicting sets.

This approach allows edits without blocking — the admin can save their work and resolve conflicts as a separate step, rather than being blocked from saving by a conflict in a company they weren't thinking about.

**Conflict Detection (on scope changes):**

Changing an override set's scope (e.g., narrowing from Any + Any + Any to I-22 + Any + Any) follows the same re-validation flow. A scope change can either create new conflicts (widening scope) or resolve existing ones (narrowing scope). The system re-validates all assigned companies after any scope change.

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
- This level already exists in the current system and remains largely unchanged for V1 scope

---

## Device-Level Configuration Rendering and Legacy Coexistence

### Overview

The V1 configuration engine and the legacy file-based configuration system (.DAT files) will coexist for an extended transition period. Devices will be manually moved from the old system to the new system one at a time. During this period, the device page must support previewing configurations from both systems and clearly indicate which system a device is using.

### Configuration Source Selector

Each device needs a setting indicating which configuration system it uses:

- **Legacy Configuration** — the device uses the current file-based system (.DAT files uploaded per model/carrier/service plan, with custom company configuration files and device-level `custom_configurations` JSON overrides)
- **V1 Configuration** — the device uses the new schema-driven configuration engine (schema defaults → model defaults → 3-way rule → company override set → device overrides)

**Key behavior:**
- The selector is a per-device setting, stored on the device record
- An admin manually switches a device to "V1" when the new configuration has been fully built in the V1 engine for that device's model/carrier/service plan combination
- No automated migration — configurations are manually built in the V1 engine, and devices are switched over one at a time (no bulk migration path — intentionally conservative)
- Both systems remain fully operational during the transition period — switching the selector does not delete or modify the device's legacy configuration files
- Switching back to Legacy is always possible since legacy files are preserved

> **Decision:** Migration is manual, one company/device at a time. Existing configs stay as-is until rebuilt in V1. No automated migration tooling for MVP. (Source: Adam, Confluence Mar 27 + Apr 16 2026)

### Device Page Configuration Preview

The device page must be able to preview the device's configuration from both systems:

**Legacy Configuration Preview:**
- A read-only preview of the compiled .DAT file content — showing the key-value pairs from the legacy file after all legacy layers are applied (base config file + custom company config + device custom_configurations)
- This is a temporary preview built to give visibility into the legacy configuration while the old system is still in use

**V1 Configuration Preview:**
- The fully resolved configuration from the V1 engine: all schema parameters resolved through all 5 levels (schema default → model default → 3-way rule → company override set → device override)
- Uses the same preview pattern as the 3-way rule and override set editors, but resolved through all levels including device overrides
- Each parameter shows its final value and source layer (color-coded badge: schema default, model default, 3-way rule, company override, device override)
- Support the same filters: search, show only overridden, show only missing required
- Support export to key-value format

**Visual indicator on device page:**
- The current configuration source should be prominently displayed (e.g., a dropdown or radio selection: "Legacy" or "V1 Engine")
- The active preview corresponds to whichever system the device is set to use

**Side-by-Side Comparison:**
- Both configuration systems always render a configuration for the device — the source selector only determines which one is actually applied
- A comparison view shows legacy and V1 configurations side by side, with match/difference indicators per parameter
- This allows the admin to see exactly how the two systems differ for this device, regardless of which one is active
- Useful for verifying V1 configuration completeness before switching a device over, and for ongoing auditing

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
- The model edit page would get bloated with hundreds of parameter checkboxes
- Separation of concerns: model admin owns model attributes, config admin owns everything config-related

**What it provides:**

1. **Parameter Selection** — Select which parameters from the schema apply to this model. Different models can have different parameter subsets (e.g., I-22 needs IO parameters that other models don't). This selection determines what appears on the 3-way rule screen and company override screen for this model. Adding a new parameter to the Grand Schema does not auto-select it for any model — the admin must go to each model's configuration and add it explicitly.

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
  4. Find ALL override sets assigned to the device's company where:
     - Override set model = device's model OR "Any"
     - Override set carrier = device's carrier OR "Any"
     - Override set service plan includes device's service plan OR "Any"
     For each matching override set, if it has a value for this parameter → override
  5. If device itself has an override → override
  6. Result = final compiled value
```

If a parameter is mandatory (per schema) and still null after resolution, this should be flagged as a configuration error. Mandatory parameter enforcement happens at the time MCS Rules are published. (Source: Adam, Confluence Apr 17 2026)

**Missing 3-Way Rule:** If no 3-way rule exists for a device's model + carrier + service plan combination, Level 3 is a pass-through — the resolution continues with schema defaults and model defaults, then applies any matching company override sets and device overrides. This is a valid state, not an error. However, the device's configuration preview should indicate that no 3-way rule exists for this combination (e.g., a notice: "No 3-way rule configured for I-22 + T-Mobile + Standard").

**Why Step 4 is safe with multiple matching override sets:**

Conflict detection at assignment time (see Level 4) guarantees that no two override sets assigned to the same company can contain the same parameter for overlapping device scopes. This means that for any given device and any given parameter, **at most one override set will provide a value**. There is no ambiguity or "who wins" scenario — if an override set matches the device and contains the parameter, it is the only one that does.

**Resolution Example — Single override set matches:**

```
Device: I-22, Verizon, ATM, belongs to Cord Financial

Cord has one override set:
  "Cord Firewall" — Scope: Any + Any + Any → fw_acl = "cord_rules"

Resolution for fw_acl:
  1. Schema default:     fw_acl = null
  2. Model default:      (none set)
  3. 3-Way Rule:         I-22 + Verizon + ATM → fw_acl = "default_rules"
  4. Company override:   "Cord Firewall" matches (Any/Any/Any) → fw_acl = "cord_rules"
  5. Device override:    (none)
  Result: fw_acl = "cord_rules"
```

**Resolution Example — Multiple override sets, no conflict:**

```
Device: I-22, Verizon, ATM, belongs to Cord Financial

Cord has three override sets:
  A) "Cord Firewall"     — Any + Any + Any    → fw_acl = "cord_rules", fw_enable = true
  B) "Baltech Scheduler" — Any + Any + Any    → scheduler = "3am_cycle"
  C) "I-22 Power Config" — I-22 + Any + Any   → io_enable = 1

All three match this device. But each provides different parameters:
  - fw_acl comes from A only
  - fw_enable comes from A only
  - scheduler comes from B only
  - io_enable comes from C only

No conflicts — each parameter has exactly one source.
```

**Resolution Example — Different carrier scopes, same parameter (no conflict):**

```
Device: I-22, Verizon, ATM, belongs to TelCorp

TelCorp has two override sets:
  A) "VZW Firewall" — Any + Verizon + Any → fw_acl = "vzw_rules"
  B) "TMO Firewall" — Any + T-Mobile + Any → fw_acl = "tmo_rules"

For this Verizon device:
  - Only A matches (Verizon scope)
  - B does not match (T-Mobile scope)
  Result: fw_acl = "vzw_rules"

For a T-Mobile device in the same company:
  - Only B matches
  Result: fw_acl = "tmo_rules"
```

---

## Versioning and Approval (3-Way Rules & Override Sets)

3-way rules and override sets are the two configuration levels with the widest blast radius — a single change can affect hundreds or thousands of devices. Changes to these levels require a second set of eyes before going live.

### How It Works

When an admin edits a 3-way rule or an override set, the changes are saved as a **draft version** rather than going live immediately. The current **published version** remains active — devices continue using it. A second admin reviews and approves the draft, at which point it becomes the new published version and propagates to affected devices.

```
Admin makes changes → Draft version created (published version stays active)
  → Second admin reviews draft → Approves
    → Draft becomes published version
    → Affected devices receive new config on next check-in or via on-demand push
```

### Draft Behavior

- **One draft at a time.** Only one draft can exist per 3-way rule or override set. If a draft is already pending approval, it must be approved or discarded before new changes can be made.
- **Draft does not affect devices.** While a draft is pending, all devices continue using the published version. The draft is invisible to the resolution engine.
- **Draft preview.** The draft includes a preview showing what the compiled configuration would look like with the proposed changes, compared against the current published version. This shows the reviewer exactly what will change and how many devices are affected.
- **Draft can be discarded.** If the changes aren't needed, the draft can be discarded and the published version remains unchanged.

### Approval

- **Two-person approval.** The admin who created the draft cannot approve their own changes. A different admin must review and approve. This follows the existing pattern in the system for approving other entities.
- **What the approver sees:**
  - Summary of what changed (parameters added, modified, or removed)
  - Number of affected devices
  - Preview comparing draft vs. published version (parameter-level diff)
  - Who made the change and when
- **Approval is recorded** in the change log — both who made the change and who approved it.

**Access:** No role-based permissions for the configuration engine. All admins have equal access to all configuration sections. The only access control is the two-person approval rule (creator cannot approve their own draft).

### What Gets Versioned

| Entity | Versioned? | Approval Required? | Rationale |
|--------|-----------|-------------------|-----------|
| **3-Way Rule** | Yes | Yes | A single rule can affect all devices for a model + carrier + service plan combination — potentially hundreds of devices |
| **Company Override Set** | Yes | Yes | Override sets can be assigned to many companies — a change propagates to all assigned companies' devices |
| Grand Schema | No | No | Schema defaults are the lowest priority level and are overridden by everything. Changes are low risk. |
| Model Defaults | No | No | Changed infrequently, narrower scope (one model). Lower blast radius. |
| Device Overrides | No | No | Single device scope. No blast radius. |

### Version History

Each 3-way rule and override set maintains a version history showing:
- Version number
- Who created the draft
- Who approved it
- When it was published
- Summary of changes

This history is part of the change log (see Configuration Change Log section) and visible on the entity's edit/detail page.

### Impact on Configuration Compilation

If a pre-compilation approach is adopted (see Config Push & Verification document), only **published versions** are used for compilation. Drafts are never compiled or pushed to devices. When a draft is approved and becomes the published version, recompilation is triggered for all affected devices.

---

## Automatic Recompilation on Device Attribute Changes

When a device's attributes change, the resolved configuration may shift — a different 3-way rule may apply, different override sets may match, or the device may move to a company with different override set assignments. The system must automatically recompile the device's resolved configuration when any of the following changes:

- **Carrier change** (e.g., SIM swap from Verizon to T-Mobile) — may match a different 3-way rule and different carrier-scoped override sets
- **Service plan change** — may match a different 3-way rule and different service-plan-scoped override sets
- **Company reassignment** — entirely different set of override sets applies
- **Model change** (rare — device replacement scenarios) — different model defaults, parameter list, 3-way rule, and model-scoped override sets

Recompilation updates the device's configuration preview on the device page. Once config push is implemented, recompilation would also trigger the delivery pipeline to push the updated config to the device. Admins should see a summary of what changed when recompilation occurs, and historical changes should be tracked in a change log / audit trail. (Source: Adam, Confluence Apr 17 2026)

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
| 3-Way Rule | Creation, draft creation, draft approval, draft discard, parameter value changes, service plan additions/removals |
| Company Override Set | Creation, draft creation, draft approval, draft discard, parameter override value changes, company assignment/unassignment changes |

### Where change logs are displayed

Each configuration entity's edit/detail page — including the Grand Schema parameter edit page — should show a "Change History" section or tab with a chronological list of changes. Most recent changes first. The Grand Schema browse page should also show a "Recent Schema Activity" feed for at-a-glance visibility into who is modifying the schema.

Each log entry shows:
- Date/time
- User name
- Description of the change

Change logs capture what/who/when only. There is no optional "reason" or "notes" field for change entries.

The Configuration Management landing page may also show a "Recent Activity" feed with the latest changes across all configuration entities.

---

## Screens / UI Changes

### New Screens (Configuration Admin Section)

| Screen | Purpose |
|--------|---------|
| Grand Schema | Define all parameters with defaults, types, validation |
| Model Configuration | Per-model parameter selection + model default values |
| Company Override Sets | Create, edit, browse named override sets. View company assignments per set. Browse page includes assignment status column and conflict indicators. |

### Modified Screens

| Screen | Changes |
|--------|---------|
| Model Edit | Add "is configurable" checkbox only (simple boolean flag) |
| Configuration Browse/Create | Replace file upload with model+carrier+service plan picker + parameter value editor (3-way rule screen). Includes configuration preview showing all schema parameters with resolved values and export to key-value format. |
| Company Edit | Add override set assignment section — link to existing override sets or create new ones. Shows which override sets are active for this company, grouped by 3-way rule. |

### Modified Screens (Device)

| Screen | Changes |
|--------|---------|
| Device Edit/View | Add configuration source selector (Legacy vs. V1). Show legacy .DAT preview when on Legacy. Show full 5-level resolved configuration preview with source layer badges when on V1. |

---

## Out of Scope (V1)

Items discussed but explicitly deferred:

- Per-carrier parameter selection (Adam confirmed not needed)
- Per-service-plan parameter selection (Adam confirmed not needed)
- Device-level configuration overrides redesign (the override mechanism itself stays as-is; what changes is the configuration source toggle, preview, and migration workflow)
- Affiliate/distributor commission model (mentioned as future business need)
- Navigation/menu restructuring (acknowledged as needed but separate effort)
- Reverse impact analysis ("which devices are affected by this parameter") — forward resolution (device → config) is sufficient for now; reverse lookup is a future iteration

---

## Open Questions

All open questions and client feedback are tracked in `Open_Questions.md`. See that file for the full list of open items, answered-but-not-yet-applied client responses, and design questions needing internal decisions.

---

## Change Log

### Version 1 — Initial Requirements (March 20, 2026)

**Decision:** Initial requirements document created based on client discussions (Meeting 2, March 2026). Established the 5-level inheritance hierarchy, override set concept, and 3-way rule design.

**Source:** Internal design sessions and early client feedback.

### Version 2 — Override Set Flexible Scoping (April 7, 2026)

**Decision:** Override sets are decoupled from specific 3-way rules and now support flexible scoping with "Any" option for model, carrier, and service plan selectors.

**Source:** Client feedback from Meeting 1 (March 27, 2026) with Devon D'Andrea and Adam Curcie. Adam identified that the vast majority of their customer customizations (especially firewall rules for ATM customers) are agnostic to model, carrier, and service plan. Requiring override sets to be scoped to a specific 3-way rule forced unnecessary duplication.

**What changed:**

| Area | Before (V1 Requirements) | After (V2 Requirements) |
|------|--------------------------|------------------------|
| Override set scoping | Tied to a specific 3-way rule (one model + one carrier + service plans) | Independent scope with Model/Carrier/Service Plan selectors, each supporting "Any" or a specific value |
| Override sets per company | One override set per 3-way rule per company | Multiple override sets allowed per company, subject to conflict detection |
| Parameter list source | Always inherited from the 3-way rule's model | Model-specific: from model's parameter list. Model=Any: from schema `available_at_company` flag |
| Conflict resolution | Not needed (one set per rule per company) | Strict conflict detection at assignment time — no parameter overlap allowed for overlapping device scopes |
| Resolution logic (Level 4) | Direct lookup: find override set for this company + this 3-way rule | Matching: find all override sets assigned to company where scope matches device's model/carrier/service plan |

**What did NOT change:**
- The 3-way rule (Level 3) remains unchanged — still requires specific model + carrier + service plan(s)
- The 5-level hierarchy order is unchanged (Schema → Model → 3-Way Rule → Company Override → Device)
- Company override still wins over 3-way rule; device override still wins over company override
- Sub-company inheritance behavior is unchanged

**Design decision — conflict resolution approach:**
- **Option 1 (chosen): Strict no-overlap.** No two override sets assigned to the same company can contain the same parameter for overlapping device scopes. Conflicts are rejected at assignment time.
- Option 2 (deferred): Specificity-based resolution where more-specific scopes win over less-specific. This adds power but also complexity. May be reconsidered in a future version if strict no-overlap proves too limiting in practice.

*This change introduced questions about conflict detection UX, soft conflicts, and browse/filter UI — these have since been resolved. See Open_Questions.md for details.*

### Version 3 — Requirements Review Gap Fixes (April 12, 2026)

**Source:** Internal requirements review session identifying gaps in dynamic behavior (what happens when data changes over time).

**What was added:**

| Addition | Section | Details |
|----------|---------|---------|
| Conflict detection on edits | Level 4: Company Overrides | Editing an override set triggers conflict checks against all assigned companies. Blocked if new parameter conflicts. |
| "Any" is perpetual | Level 4: Override Set Scoping | "Any" matches all current AND future values. New service plans/carriers/models auto-fall under existing "Any"-scoped sets. |
| Encrypted value display | Level 1: Schema | Encrypted parameters are masked ('********') in all previews, change logs, and exports. Never readable after save. |
| Missing 3-way rule behavior | Resolution Logic | No matching 3-way rule is valid — Level 3 is a pass-through. Preview shows a notice. |
| Auto-recompile on device changes | New section | Device config auto-recompiles when carrier, service plan, company, or model changes. |

### Version 4 — Versioning and Approval for 3-Way Rules & Override Sets (April 13, 2026)

**Source:** Internal requirements review — identifying the need for a safety gate on high-blast-radius config changes before they reach devices.

**What was added:**

| Addition | Section | Details |
|----------|---------|---------|
| Draft/publish workflow | New section: Versioning and Approval | 3-way rules and override sets require draft → review → approval before going live. One draft at a time per entity. |
| Two-person approval | Versioning and Approval | The admin who creates a draft cannot approve their own changes. A different admin must review and approve. |
| Draft preview with diff | Versioning and Approval | Drafts include a preview comparing proposed changes against the current published version, with affected device count. |
| Version history | Versioning and Approval | Each 3-way rule and override set maintains a version history (who changed, who approved, when published). |
| Compilation impact | Versioning and Approval | Only published versions are used for compilation. Drafts are never compiled or pushed to devices. |
| Change log updates | Configuration Change Log | Draft creation, approval, and discard are now tracked events for 3-way rules and override sets. |

### Version 5 — Consolidated Open Questions & Review Decisions (April 13, 2026)

**Source:** Internal cleanup — consolidating the separate Outstanding Client Questions document into the requirements, and documenting review decisions that were tracked separately.

**What changed:**

| Change | Details |
|--------|---------|
| Consolidated open questions | Merged 4 new client questions (silent parameter filtering, cellular backup, distributor inheritance granularity, auto-recompile notification) into the Open Questions section. Enriched conflict detection UX question with edit-blocking detail. |
| Resolved parameter import question | Initial ~700 parameters will be seeded directly into the database by the dev team. Moved to resolved questions. |
| Documented review decisions inline | Added: manual per-model parameter selection (D), per-device migration with no bulk switch (G), no change reason field (H), equal admin access (I), manual 3-way rule splitting (L), reverse lookup deferred (N). |
| Removed V1 comparison content | Removed "Design decisions — what was removed from V1 and why" block from Level 1 schema section. |
| Retired Outstanding_Client_Questions.md | All content merged into this document. |

### Version 6 — Build vs. Assign Separation for Override Sets (July 19, 2026)

**Decision:** Override set lifecycle is split into two distinct phases: building (defining scope + parameters + values) and assigning (attaching companies). Conflict validation is separated accordingly.

**Source:** Internal design review analyzing real company config data (Altech, Cord, Baum override analysis against Verizon I-22 base rule). Analysis revealed 15-35 parameter overrides per company with significant parameter key overlap across companies but different values — confirming the many-to-many relationship complexity.

**What changed:**

| Area | Before | After |
|------|--------|-------|
| Override set creation | Required company context or implied immediate assignment | Override sets can be built independently with no companies. Unassigned sets are inactive. |
| Conflict validation on build | Edits blocked if any assigned company would have a conflict | Edits always save. Re-validation runs after save, flagging conflicting companies. |
| Conflict validation on assign | Checked at assignment time | Unchanged — still checked per-company at assignment time. |
| Conflicting companies after edit | Edit blocked entirely | Flagged as "conflict — needs resolution." Override set not compiled/pushed for those companies until resolved. |
| Scope change validation | Implicitly part of edit blocking | Explicit re-validation after scope changes. Narrowing scope may resolve existing conflicts. |

**What did NOT change:**
- Conflict detection logic (scope overlap + parameter overlap) is unchanged
- Assignment examples (allowed, rejected) are unchanged
- The 5-level hierarchy order is unchanged
- Override set scoping with "Any" option is unchanged
