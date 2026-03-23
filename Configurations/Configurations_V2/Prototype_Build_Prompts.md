# Configuration Engine V2 — Prototype Build Prompts

Step-by-step prompts for building the Configuration Management administrative area prototype. Each prompt builds on the previous phase. Feed these into your lab environment sequentially.

Before starting: ensure the prototype has a working admin navigation menu where we can add the new Configuration Management section.

---

## Phase 0: Navigation and Landing Page

### Prompt 0.1 — Add Configuration Management to Admin Menu

```
Add a new top-level section to the admin navigation menu called "Configuration Management". It should contain the following sub-menu items:

1. Grand Schema
2. Model Configuration
3. 3-Way Rules
4. Company Override Sets

Each menu item should link to its own page. For now, each page can be a placeholder with just a heading.

Place this section in the admin menu grouped together — not split across different menu areas. This is a self-contained configuration management admin area.
```

### Prompt 0.2 — Configuration Management Landing Page

```
Create a landing page for the Configuration Management section. This is the page an admin sees when they click "Configuration Management" in the menu.

It should show:
- A heading: "Configuration Management"
- A brief description: "Manage the configuration engine — define parameters, configure models, set up rules, and manage company overrides."
- Four card-style links to each sub-section, each with:
  - An icon (use appropriate icons: schema/database for Grand Schema, device/model for Model Configuration, rules/layers for 3-Way Rules, building/company for Company Override Sets)
  - The section name
  - A one-line description:
    - Grand Schema: "Define all configuration parameters, data types, defaults, and validation rules"
    - Model Configuration: "Select parameters and set default values for each device model"
    - 3-Way Rules: "Configure parameter values for model + carrier + service plan combinations"
    - Company Override Sets: "Create and manage reusable configuration overrides assigned to companies"
  - A count/stat if available (e.g., "694 parameters", "5 models configured", "12 rules", "8 override sets") — these can be hardcoded placeholder numbers for now
```

---

## Phase 1: Grand Schema

### Prompt 1.1 — Schema Browse Page

```
Build the Grand Schema browse page. This is a table/list view of all configuration parameters in the system.

The table should have these columns:
- Parameter Name (the key, e.g., "wan0_proto", "wl0_ssid")
- Display Name (human-readable label)
- Category (e.g., Network, Firewall, DHCP, DNS, WiFi, Cellular, Monitor)
- Data Type (string, integer, boolean, ip_address, encrypted)
- Default Value (the schema-level default, can be empty/null)
- Required (yes/no badge)
- Encrypted (yes/no badge, highlighted if yes)

Features:
- Search/filter bar at the top that filters across parameter name and display name
- Category filter dropdown (show all categories, allow selecting one to filter)
- Data type filter dropdown
- Pagination (show 25 per page)
- Click on any row to open the edit view
- "Add Parameter" button in the top right

Populate with sample data — at least 20-30 parameters across different categories. Use realistic WATM parameter names like:
- Network: wan0_proto, wan0_ipaddr, lan0_netmask, lan0_gateway, dns_primary, dns_secondary
- Firewall: fw_enable, fw_table_1, fw_rule_count
- WiFi: wl0_ssid, wl0_security, wl0_channel, wl0_password
- Cellular: cell_apn, cell_carrier_code, cell_signal_threshold
- Monitor: mon_interval, mon_heartbeat, mon_checkin_url
- IO: io_enable, io_port_count, io_alarm_threshold
```

### Prompt 1.2 — Schema Add/Edit Parameter

```
Build the add/edit form for a schema parameter. This form is used both for creating new parameters and editing existing ones.

Form fields:
1. Parameter Name — text input, required, unique. Lowercase with underscores (e.g., wan0_proto). Show validation: only lowercase letters, numbers, and underscores allowed.
2. Display Name — text input, optional. Human-readable label (e.g., "WAN Protocol").
3. Description — textarea, optional. Explanation of what this parameter does.
4. Category — dropdown, required. Options: Network, Firewall, DHCP, DNS, WiFi, Cellular, Monitor, IO, System, Security. Allow typing to filter.
5. Data Type — dropdown, required. Options: string, integer, boolean, ip_address, encrypted.
6. Default Value — text input, optional. The schema-level default value. If data type is boolean, show a toggle instead. If data type is integer, show a number input.
7. Required — checkbox. "This parameter must have a non-null value in the final compiled configuration."
8. Encrypted — checkbox. "This is a sensitive field (e.g., passwords). Values will be stored with encryption."
9. Validation Regex — text input, optional. Only show if data type is string or ip_address. Pattern that values must match.
10. Min Value — number input, optional. Only show if data type is integer.
11. Max Value — number input, optional. Only show if data type is integer.
12. Available at Company Level — checkbox. Default checked. "This parameter can be overridden in company override sets."
13. Available at Device Level — checkbox. Default unchecked. "This parameter can be overridden at the individual device level."

Layout:
- Two-column layout for the main fields (name/display name on left, category/type on right)
- Full-width for description
- Default value and validation fields grouped together
- Checkboxes (required, encrypted, availability) grouped at the bottom

Buttons: Save, Cancel. On edit, also show a "Delete" button with confirmation dialog.
```

---

## Phase 2: Model Configuration

### Prompt 2.1 — Model Configuration Browse Page

```
Build the Model Configuration browse page. This shows all device models that are flagged as "is configurable" and their configuration status.

The table should have these columns:
- Model Name (e.g., "I-22", "4100", "Cradlepoint IBR900")
- Manufacturer (e.g., "CalAmp", "Sierra Wireless", "Cradlepoint")
- Parameters Selected (count, e.g., "142 / 694" meaning 142 of 694 schema parameters are selected for this model)
- Model Defaults Set (count of parameters that have model-level default values)
- 3-Way Rules (count of 3-way rules that exist for this model)
- Status (badge: "Configured" if parameters are selected, "Pending Setup" if flagged as configurable but no parameters selected yet)

Features:
- Click on any row to open the model configuration detail page
- No "Add" button here — models are added via the Model Edit page ("is configurable" checkbox). This page only shows models already flagged.

Populate with 4-5 sample models:
- I-22 (CalAmp) — 142 parameters, 18 defaults, 3 rules, Configured
- 4100 (CalAmp) — 128 parameters, 12 defaults, 2 rules, Configured
- Cradlepoint IBR900 (Cradlepoint) — 95 parameters, 8 defaults, 1 rule, Configured
- Sierra RV55 (Sierra Wireless) — 0 parameters, 0 defaults, 0 rules, Pending Setup
```

### Prompt 2.2 — Model Configuration Detail Page

```
Build the Model Configuration detail page for a single model. This page has two main sections displayed as tabs or collapsible panels:

**Header area:**
- Model name (large heading, e.g., "I-22 Configuration")
- Manufacturer name (subtitle)
- Summary stats: X parameters selected, Y model defaults set, Z 3-way rules using this model

**Tab/Section 1: Parameter Selection**

A two-panel layout:
- Left panel: "Available Parameters" — all schema parameters NOT yet selected for this model. Shown as a searchable, filterable list grouped by category. Each row shows parameter name, display name, category, and data type.
- Right panel: "Selected Parameters" — parameters selected for this model. Same display format.
- Buttons/actions to move parameters between panels (select/deselect). Support multi-select.
- Category filter on both panels to quickly find parameters.
- A "Select All in Category" option for bulk selection.
- Show counts on both panels (e.g., "Available: 552" / "Selected: 142").

**Tab/Section 2: Model Defaults**

A table/form showing only the SELECTED parameters for this model:
- Parameter Name
- Display Name
- Category
- Data Type
- Schema Default (read-only, shown for reference)
- Model Default (editable input field — this is what the admin sets here)

The model default column should show an empty/placeholder state if no model default is set, with the schema default visible so the admin knows what the baseline is. Only parameters where a model default makes sense (constant across all carrier/service plan combos for this model) should have values entered here.

Features:
- Filter/search within the selected parameters
- Group by category with collapsible sections
- Save button to persist all changes
- Visual indicator showing which parameters have model defaults set vs. inheriting schema defaults
```

---

## Phase 3: 3-Way Rules

### Prompt 3.1 — 3-Way Rules Browse Page

```
Build the 3-Way Rules browse page. This shows all existing model + carrier + service plan configuration rules.

The table should have these columns:
- Model (e.g., "I-22")
- Carrier (e.g., "Verizon", "T-Mobile", "AT&T")
- Service Plan (e.g., "ATM", "Standard", "IoT Basic")
- Parameters Configured (count of parameters with values set in this rule)
- Override Sets (count of company override sets that reference this 3-way rule)
- Last Modified (date)

Features:
- Filter dropdowns for Model, Carrier, and Service Plan (each independently filterable)
- Click on any row to open the 3-way rule editor
- "Add 3-Way Rule" button

Populate with sample data:
- I-22 + Verizon + ATM — 45 parameters, 3 override sets
- I-22 + T-Mobile + ATM — 42 parameters, 1 override set
- I-22 + AT&T + Standard — 38 parameters, 0 override sets
- 4100 + Verizon + ATM — 35 parameters, 2 override sets
- 4100 + Verizon + Standard — 30 parameters, 0 override sets
- Cradlepoint IBR900 + T-Mobile + IoT Basic — 28 parameters, 1 override set
```

### Prompt 3.2 — 3-Way Rule Editor (Add/Edit)

```
Build the 3-way rule editor page. This is where an admin configures parameter values for a specific model + carrier + service plan combination.

**Header / Selector area:**
When creating a new rule:
- Model dropdown (only models flagged as configurable with parameters selected)
- Carrier dropdown
- Service Plan dropdown
- When model is selected, the parameter list below populates based on that model's selected parameters

When editing an existing rule:
- Model, Carrier, Service Plan shown as read-only labels (the combination is the identity of this rule)

**Parameter Value Editor:**

A table/form showing all parameters selected for this model:
- Parameter Name
- Display Name
- Category (use as group headers — group parameters by category with collapsible sections)
- Data Type
- Schema Default (read-only, shown in a muted/gray style for reference)
- Model Default (read-only, shown in a muted/gray style for reference — only if one exists)
- Effective Baseline (the value this parameter would have without this 3-way rule — either model default or schema default, whichever applies)
- 3-Way Rule Value (editable input field — this is what the admin sets)

Input behavior per data type:
- string: text input
- integer: number input (respect min/max from schema if set)
- boolean: toggle switch
- ip_address: text input with IP format hint
- encrypted: password input with show/hide toggle

Features:
- If the 3-way rule value is empty/null, the effective baseline is shown as what will be used (visual indicator: "inherited" state)
- If the 3-way rule value is set, highlight it to show it's actively overriding (visual indicator: "overridden" state — e.g., bold text, colored left border, or a small override icon)
- Category-based collapsible groups
- Search/filter within the parameter list
- "Clear" button per parameter to remove the 3-way rule value and revert to baseline
- Summary at top: "X of Y parameters overridden in this rule"
- Save and Cancel buttons
```

---

## Phase 4: Company Override Sets

### Prompt 4.1 — Company Override Sets Browse Page

```
Build the Company Override Sets browse page. This shows all named override sets in the system.

The table should have these columns:
- Override Set Name (e.g., "Cord Financial RMS Access", "Miele Custom Firewall", "Standard DNS Override")
- 3-Way Rule (show as "Model + Carrier + Service Plan", e.g., "I-22 + Verizon + ATM")
- Parameters Overridden (count)
- Companies Assigned (count, with a clickable link or expandable list showing company names)
- Last Modified (date)

Features:
- Search by override set name
- Filter by 3-way rule (model/carrier/service plan dropdowns)
- Click on any row to open the override set editor
- "Create Override Set" button

Populate with sample data:
- "Cord Financial RMS Access" — I-22 + Verizon + ATM — 8 params — 12 companies
- "Miele Network Config" — I-22 + Verizon + ATM — 15 params — 1 company (Miele)
- "Standard DNS Override" — 4100 + Verizon + ATM — 3 params — 25 companies
- "High Security Firewall" — I-22 + T-Mobile + ATM — 11 params — 4 companies
- "Loaded ATMs Custom" — I-22 + Verizon + ATM — 6 params — 1 company (Loaded ATMs)
```

### Prompt 4.2 — Company Override Set Editor (Add/Edit)

```
Build the company override set editor page. This page has two main sections: the parameter overrides and the company assignments.

**Header area:**
- Override Set Name — text input (editable, descriptive name)
- 3-Way Rule selector:
  - When creating: dropdowns for Model, Carrier, Service Plan to select which 3-way rule this override set is based on. Only show combinations that have existing 3-way rules.
  - When editing: shown as a read-only label (cannot change the 3-way rule association after creation)

**Section 1: Parameter Overrides**

Similar layout to the 3-way rule editor but with one additional column in the inheritance chain:

A table/form showing all parameters from this model's selected parameter list:
- Parameter Name
- Display Name
- Category (group by category with collapsible sections)
- Effective 3-Way Rule Value (read-only — the compiled value from schema default → model default → 3-way rule value. This is what companies would get WITHOUT this override set.)
- Override Value (editable input field — what this override set changes)

Input behavior: same as 3-way rule editor (type-appropriate inputs per data type).

Features:
- Empty override values mean "inherit from the 3-way rule" (show inherited state visually)
- Set override values highlighted as active overrides
- Search/filter within parameters
- Category collapsible groups
- Summary: "X of Y parameters overridden"
- "Clear" button per parameter to remove override

**Section 2: Company Assignments**

A two-panel layout:
- Left panel: "Available Companies" — companies whose devices use the selected 3-way rule but are NOT assigned to this override set. Searchable list showing company name and device count.
- Right panel: "Assigned Companies" — companies currently assigned to this override set. Show company name and device count.
- Move buttons between panels (assign/unassign). Support multi-select.
- Show counts: "Available: X" / "Assigned: Y"

Constraint enforcement:
- If a company is already assigned to a DIFFERENT override set for the same 3-way rule, show a warning icon next to that company in the available list. On attempt to assign, show a confirmation: "This company is currently assigned to [Other Override Set Name] for this 3-way rule. Reassign to this override set instead?" (since only one override set per 3-way rule per company is allowed)

Buttons: Save, Cancel. On edit, show Delete with confirmation ("This will remove overrides for X assigned companies").
```

---

## Phase 5: Resolution Preview

### Prompt 5.1 — Configuration Resolution Preview

```
Build a Configuration Resolution Preview page. This is a read-only diagnostic tool that shows the final compiled configuration for a given combination, with full visibility into which layer each value comes from.

**Selector area:**
Four dropdowns in sequence:
1. Model (required)
2. Carrier (required)
3. Service Plan (required)
4. Company (optional — if not selected, shows resolution through 3-way rule only)

A "Resolve" button that compiles and displays the result.

**Results display:**

A table showing the fully resolved configuration:
- Parameter Name
- Display Name
- Category (group by category)
- Final Value (the compiled result — prominently displayed)
- Source Layer (which layer the value came from — shown as a color-coded badge):
  - "Schema Default" (gray)
  - "Model Default" (blue)
  - "3-Way Rule" (green)
  - "Company Override" (orange)
  - "MISSING" (red — for required parameters with no value)

Expandable detail per row: click to see the full resolution chain:
  - Schema Default: [value or null]
  - Model Default: [value or null]
  - 3-Way Rule: [value or null]
  - Company Override Set: [set name and value, or "none"]
  - → Final: [value]

**Summary stats at top:**
- Total parameters: X
- From schema defaults: X (with gray indicator)
- From model defaults: X (with blue indicator)
- From 3-way rule: X (with green indicator)
- From company override: X (with orange indicator)
- Missing/errors: X (with red indicator, if any required parameters are unresolved)

Features:
- Export to CSV or JSON (the final compiled config)
- Search/filter parameters in the results
- Toggle to show only overridden parameters (hide those still at schema default)
- Toggle to show only parameters with errors/missing values
```

---

## Build Order Summary

| Phase | Depends On | Screens |
|-------|-----------|---------|
| 0 | Nothing | Nav menu, Landing page |
| 1 | Phase 0 | Schema browse, Schema add/edit |
| 2 | Phase 1 | Model config browse, Model config detail |
| 3 | Phase 2 | 3-Way rules browse, 3-Way rule editor |
| 4 | Phase 3 | Override sets browse, Override set editor |
| 5 | Phase 4 | Resolution preview |

Each phase is usable on its own — you can demo schema management without having 3-way rules built yet. But the full value shows when you can walk through the complete flow: define a parameter in the schema → select it for a model → set it in a 3-way rule → override it for a company → see the final compiled result in the preview.

---
---

# Part 2: Workflow Prompts — Creating Records End-to-End

These prompts focus on the **creation workflows** — walking through the full process of adding a new 3-way rule and creating a new company override set. Use these after the browse/edit screens from Part 1 are built. These prompts emphasize the step-by-step user experience, validation, and how data flows between screens.

---

## Prompt P2.1 — Add a New 3-Way Rule (Full Workflow)

```
Build the "Add 3-Way Rule" workflow. This is triggered when an admin clicks "Add 3-Way Rule" from the 3-Way Rules browse page. The workflow guides the admin through creating a new configuration rule for a model + carrier + service plan combination.

**Step 1: Select the Combination**

A form with three dependent dropdowns:

1. Model dropdown:
   - Only shows models that are flagged as "is configurable" AND have parameters selected in their model configuration.
   - Models with no parameters selected should NOT appear (they aren't ready for 3-way rules yet).
   - Show model name and manufacturer (e.g., "I-22 (CalAmp)").
   - After selecting a model, show a small info line below: "142 parameters selected for this model"

2. Carrier dropdown:
   - Populates after model is selected.
   - Shows all available carriers (e.g., Verizon, T-Mobile, AT&T, Sprint).
   - If a 3-way rule already exists for this model + carrier + any service plan, show a subtle indicator next to that carrier (e.g., a small count badge "2 rules exist").

3. Service Plan dropdown:
   - Populates after carrier is selected.
   - Shows all available service plans (e.g., ATM, Standard, IoT Basic, Enterprise).
   - If a 3-way rule already exists for this exact model + carrier + service plan combination, DISABLE that option and show "Rule already exists" next to it. Duplicate 3-way rules for the same combination are not allowed.

Validation:
- All three fields are required.
- The combination must be unique — if a rule already exists for this model + carrier + service plan, show an error and link to the existing rule for editing.

A "Continue" or "Next" button to proceed to Step 2.

**Step 2: Configure Parameter Values**

After the combination is confirmed, show the full parameter editor.

Header area (read-only, confirming the selection):
- Model: I-22 (CalAmp)
- Carrier: Verizon
- Service Plan: ATM
- Parameters available: 142

The parameter editor is a table/form grouped by category (collapsible sections). For each parameter:

| Column | Description |
|--------|-------------|
| Parameter Name | The key (e.g., wan0_proto) |
| Display Name | Human-readable label (e.g., "WAN Protocol") |
| Data Type | string, integer, boolean, ip_address, encrypted |
| Schema Default | Read-only, gray text. The default from the grand schema. Shows "—" if no default. |
| Model Default | Read-only, blue text. The model-level default if one exists. Shows "—" if not set. |
| Effective Baseline | Read-only, shown prominently. This is the value that will be used if the admin does NOT set a 3-way rule value. It equals the model default if set, otherwise the schema default. Shows "No default" in red if neither exists and the parameter is required. |
| 3-Way Rule Value | Editable input. This is what the admin fills in to override the baseline for this combination. |

Input types match the parameter's data type:
- string → text input
- integer → number input (enforce min/max from schema validation if defined)
- boolean → toggle switch (on/off)
- ip_address → text input with placeholder "e.g., 192.168.1.1"
- encrypted → password-style input with a show/hide eye icon

Row states:
- "Inherited" state (default): 3-Way Rule Value is empty. The row appears in a standard/muted style. The effective baseline is what will be used.
- "Overridden" state: Admin has entered a 3-Way Rule Value. The row is highlighted (e.g., light green background or bold left border) to make it visually obvious which parameters are being actively configured in this rule.
- "Required - No Value" state: Parameter is marked required in the schema, no baseline exists (schema default and model default are both null), and no 3-way rule value is entered. Show a red warning indicator.

Features:
- Category collapsible groups (e.g., "Network (24 parameters)", "Firewall (8 parameters)") — click to expand/collapse. Show count of overridden parameters per category in the header.
- Search/filter bar to find parameters by name or display name.
- Filter toggle: "Show only overridden" — hides parameters still at baseline, so the admin can focus on what they've actually configured.
- Filter toggle: "Show only required" — shows only required parameters, useful for ensuring all mandatory fields are set.
- "Clear" icon button per row to reset the 3-way rule value back to empty (revert to baseline).
- Running summary bar (sticky at top or bottom): "12 of 142 parameters configured | 0 required parameters missing"

Buttons:
- "Save" — saves the 3-way rule. On success, redirect to the 3-way rule browse page with a success toast message showing the new rule (e.g., "3-Way Rule created: I-22 + Verizon + ATM").
- "Save & Continue Editing" — saves but stays on this page.
- "Cancel" — confirmation dialog if any values were entered ("You have unsaved changes. Discard?"), then redirect back to browse.

Edge cases:
- If the admin saves with required parameters that have no value at any level (no schema default, no model default, no 3-way rule value), show a warning (not a blocker): "3 required parameters have no value set. Devices using this rule may have incomplete configurations." Allow saving anyway — the resolution preview can catch these later.
```

---

## Prompt P2.2 — Create a New Company Override Set (Full Workflow)

```
Build the "Create Company Override Set" workflow. This is triggered when an admin clicks "Create Override Set" from the Company Override Sets browse page. The workflow guides the admin through creating a named set of parameter overrides scoped to a specific 3-way rule, then assigning it to one or more companies.

**Step 1: Define the Override Set**

A form with:

1. Override Set Name — text input, required.
   - Descriptive name for this set (e.g., "Cord Financial RMS Access", "High Security Firewall", "Custom DNS for Retail").
   - Placeholder text: "Enter a descriptive name for this override set"
   - This name is how admins will identify and search for this set later.

2. 3-Way Rule selector — three dependent dropdowns:
   - Model dropdown: only models that have at least one 3-way rule created.
   - Carrier dropdown: populates after model selection — only carriers that have a 3-way rule for the selected model.
   - Service Plan dropdown: populates after carrier selection — only service plans that have a 3-way rule for the selected model + carrier.
   - These dropdowns are filtered to ONLY show combinations where a 3-way rule already exists. You cannot create an override set for a 3-way rule that doesn't exist yet.
   - After all three are selected, show a confirmation line: "Overriding values from: I-22 + Verizon + ATM (45 parameters configured)"

Validation:
- Name is required.
- All three 3-way rule fields are required.
- Name does not need to be unique system-wide, but warn if a similarly named set already exists for the same 3-way rule.

A "Continue" button to proceed to Step 2.

**Step 2: Set Parameter Overrides**

Header area (read-only, confirming the selection):
- Override Set Name: "Cord Financial RMS Access"
- Based on 3-Way Rule: I-22 + Verizon + ATM
- Parameters available: 142

The parameter editor shows all parameters from this model's selected parameter list, grouped by category (collapsible sections). For each parameter:

| Column | Description |
|--------|-------------|
| Parameter Name | The key (e.g., fw_table_1) |
| Display Name | Human-readable label (e.g., "Firewall Table 1") |
| Data Type | string, integer, boolean, ip_address, encrypted |
| Effective 3-Way Rule Value | Read-only. The fully compiled value from the hierarchy up through the 3-way rule (schema default → model default → 3-way rule value). This is what companies get WITHOUT this override set. Shows the value and a small source badge indicating where it came from (e.g., "192.168.1.1" with a "3-Way Rule" badge, or "255.255.255.0" with a "Schema Default" badge). |
| Company Override Value | Editable input. The value this override set will apply. |

Input types match the parameter's data type (same behavior as the 3-way rule editor).

Row states:
- "Inherited" state (default): Company Override Value is empty. The effective 3-way rule value is what assigned companies will get. Row appears in standard/muted style.
- "Overridden" state: Admin has entered a Company Override Value. Row is highlighted (e.g., light orange background or bold left border) to visually distinguish from inherited parameters. The override value is shown prominently.

Features:
- Category collapsible groups with override count per category.
- Search/filter by parameter name or display name.
- Filter toggle: "Show only overridden" — focuses on parameters being actively changed.
- "Clear" icon per row to remove the override value.
- Running summary bar: "8 of 142 parameters overridden"

A "Continue" button to proceed to Step 3. Also a "Save without assigning companies" option — saves the override set with no companies assigned yet (can be assigned later).

**Step 3: Assign Companies**

Header area (read-only):
- Override Set Name: "Cord Financial RMS Access"
- Based on 3-Way Rule: I-22 + Verizon + ATM
- Parameters overridden: 8

Two-panel company assignment layout:

Left panel — "Available Companies":
- Shows companies whose devices use the selected 3-way rule (i.e., companies that have devices with this model + carrier + service plan combination) AND are not already assigned to this override set.
- Each row shows:
  - Company name
  - Device count (how many devices in this company match the 3-way rule)
  - Current override set assignment (if any) — shown as a small badge. If the company is already assigned to a DIFFERENT override set for this 3-way rule, show the name of that set with a warning color (e.g., "Currently: High Security Firewall"). This is critical — only one override set per 3-way rule per company.
- Searchable by company name.
- Multi-select with checkboxes.

Right panel — "Assigned Companies":
- Companies selected to receive this override set.
- Same display format: company name + device count.
- Multi-select for bulk unassign.

Move controls between panels:
- "Assign →" button (move selected from left to right)
- "← Unassign" button (move selected from right to left)
- "Assign All" and "Unassign All" links

Constraint enforcement:
- When moving a company that is already assigned to a different override set for the same 3-way rule, show a modal confirmation:
  "Cord Financial is currently assigned to 'High Security Firewall' for I-22 + Verizon + ATM. Only one override set per 3-way rule per company is allowed. Reassign Cord Financial to 'Cord Financial RMS Access' instead?"
  - Buttons: "Reassign" / "Cancel"
  - If confirmed, the company is moved to this override set and removed from the other one.

Summary below panels:
- "X companies will receive this override set"
- "Y companies will be reassigned from other override sets" (if any conflicts were resolved)

Buttons:
- "Save & Create" — saves the override set with parameter overrides and company assignments. Redirect to the override set browse page with success toast: "Override Set created: Cord Financial RMS Access (assigned to 12 companies)".
- "Back" — go back to Step 2 (parameter overrides) without losing company selections.
- "Cancel" — confirmation if changes were made, then redirect to browse.

Edge case:
- If no companies are assigned, allow saving anyway. Show an info message: "This override set has no companies assigned. You can assign companies later from this page or from individual company edit pages."
- If the admin tries to create an override set but no 3-way rules exist yet, show a friendly empty state on Step 1: "No 3-Way Rules have been created yet. Create a 3-Way Rule first, then come back to create override sets." with a link to the 3-Way Rules page.
```

---
---

# Part 3: Enhancement Prompts

These prompts add features to screens built in Part 1 and Part 2.

---

## Prompt P3.1 — 3-Way Rule Editor: Configuration Preview Button

```
Enhance the 3-Way Rule editor page (both the add and edit views) by adding a "Preview Configuration" button. This button opens a full configuration preview that shows ALL parameters in the entire schema — not just the ones selected for this model — with their resolved values as they would appear at the 3-way rule level (before any company or device overrides).

**Button placement:**
- Add a "Preview Configuration" button in the header/toolbar area of the 3-way rule editor, next to the existing Save and Cancel buttons.
- Style it as a secondary/outline button with an eye or preview icon.
- The button should work whether the rule is being created (new, unsaved) or edited (existing). When previewing an unsaved rule, use the current in-memory values the admin has entered — do not require saving first.

**Preview display:**
When clicked, open a full-screen modal or slide-over panel (not a new page — the admin should be able to close it and return to editing without losing their work).

The preview shows a complete key-value list of ALL parameters in the grand schema (all ~697 parameters), not just the model's selected subset. This represents what the full compiled configuration looks like at this point in the hierarchy.

**For each parameter, show:**

| Column | Description |
|--------|-------------|
| Parameter Name | The key (e.g., wan0_proto) |
| Display Name | Human-readable label |
| Category | For grouping |
| Value | The resolved value at this level of the hierarchy |
| Source | Where the value comes from — shown as a color-coded badge |
| In Model | Whether this parameter is in the model's selected parameter list (yes/no indicator) |

**Value resolution logic for each parameter:**
The preview resolves every schema parameter through the hierarchy up to (and including) the 3-way rule level:

1. If the parameter has a 3-Way Rule value set (from the current editor) → show that value, source badge: "3-Way Rule" (green)
2. Else if the parameter has a Model Default for this model → show that value, source badge: "Model Default" (blue)
3. Else if the parameter has a Schema Default → show that value, source badge: "Schema Default" (gray)
4. Else → show "Not Set", source badge: "No Value" (light/muted). If the parameter is marked as required, show "MISSING" in red instead.

**Important distinction — model-selected vs. non-model parameters:**
- Parameters that ARE in this model's selected parameter list are shown in normal style. These are the parameters that the 3-way rule can configure and that will flow through to company overrides and devices.
- Parameters that are NOT in this model's selected parameter list are shown in a dimmed/muted style. They still appear with their schema default values (if any), but they are visually de-emphasized because they are not actively managed for this model. This gives the admin the full picture while making it clear which parameters are "active" for this configuration.

**Layout and features:**

Header area:
- Title: "Configuration Preview: I-22 + Verizon + ATM"
- Subtitle: "Showing all 697 schema parameters resolved through the 3-way rule level"
- Summary stats bar:
  - Total parameters: 697
  - With values: X (parameters that have a value from any source)
  - From schema defaults: X (gray badge)
  - From model defaults: X (blue badge)
  - From 3-way rule: X (green badge)
  - Not set: X (muted)
  - Missing (required, no value): X (red badge, if any)
  - In model parameter list: X of 697

Parameter list:
- Grouped by category with collapsible sections. Each category header shows: "Network (45 parameters — 24 in model, 6 overridden by 3-way rule)"
- Within each category, show model-selected parameters first (normal style), then non-model parameters below (dimmed style), separated by a subtle divider or label ("Not selected for this model").

Filters/toggles:
- Search by parameter name or display name
- Filter: "Show all" (default) / "Model parameters only" / "Non-model parameters only"
- Filter: "Show only parameters with values" — hides "Not Set" parameters
- Filter: "Show only 3-way rule overrides" — shows only parameters where the 3-way rule is actively setting a value
- Filter: "Show only missing required" — shows only required parameters with no value

Export:
- "Export as Key-Value" button — exports the full 697-parameter list as a plain text key=value format (similar to the current .DAT file format). Only includes parameters that have a value (skip "Not Set" parameters). This is useful for comparison with existing config files.
- "Export as CSV" button — exports all 697 rows with columns: parameter_name, value, source, in_model, category

Footer:
- "Close" button to return to the 3-way rule editor
- Note: "This preview shows resolution through the 3-way rule level. Company overrides and device overrides are not included."
```

---

## Prompt P3.2 — 3-Way Rule: Multi-Select Service Plans

```
Update the 3-Way Rule screens (browse page, add workflow, and edit view) to support multiple service plans per rule. A single 3-way rule applies to one model, one carrier, and ONE OR MORE service plans.

**Changes to the Add 3-Way Rule workflow (Step 1: Select the Combination):**

Replace the Service Plan single-select dropdown with a multi-select component:
- After the admin selects a Model and Carrier, the Service Plan field should be a multi-select (checkboxes, tag-style selector, or a multi-select dropdown — whatever fits the UI best).
- The admin can select one or more service plans to include in this rule.
- Service plans that are already assigned to ANOTHER 3-way rule for the same model + carrier combination should be DISABLED with a note: "Already in rule: [rule name or ID]". A service plan cannot belong to more than one 3-way rule for the same model + carrier.
- After selection, show the selected service plans as tags/chips below the selector (e.g., "ATM", "Standard", "Enterprise").
- The info line should update to reflect the selection: "This rule will apply to I-22 + Verizon for 3 service plans: ATM, Standard, Enterprise"

**Changes to the Edit 3-Way Rule view:**

- The header area should display all associated service plans (e.g., "Service Plans: ATM, Standard, Enterprise" shown as tags/chips).
- On the edit view, service plans should be editable — the admin can add or remove service plans from the rule.
  - Adding a service plan: only service plans not already assigned to another rule for this model + carrier are available.
  - Removing a service plan: show a confirmation if the service plan being removed has company override sets that reference this 3-way rule for that service plan. Removing it would affect those override sets.
- If all service plans are removed, the rule cannot be saved (at least one is required).

**Changes to the 3-Way Rules Browse Page:**

- The "Service Plan" column should display multiple values. Show as comma-separated tags/chips (e.g., "ATM, Standard" or as individual badges).
- The Service Plan filter dropdown should still work — selecting a service plan in the filter shows all rules that INCLUDE that service plan (not exact match).
- Update the sample data to reflect multi-select:
  - I-22 + Verizon + [ATM, Standard] — 45 parameters, 3 override sets
  - I-22 + T-Mobile + [ATM] — 42 parameters, 1 override set
  - I-22 + AT&T + [Standard, Enterprise] — 38 parameters, 0 override sets
  - 4100 + Verizon + [ATM, IoT Basic] — 35 parameters, 2 override sets
  - 4100 + Verizon + [Standard] — 30 parameters, 0 override sets
  - Cradlepoint IBR900 + T-Mobile + [IoT Basic, Enterprise] — 28 parameters, 1 override set

**Changes to the Configuration Preview (P3.1):**

- The preview title should list all service plans: "Configuration Preview: I-22 + Verizon + [ATM, Standard, Enterprise]"
- The preview content itself does not change — the parameter values are the same across all service plans in this rule (that's the whole point of grouping them).

**Validation rules:**
- At least one service plan must be selected.
- No service plan can appear in more than one 3-way rule for the same model + carrier.
- When editing, if a service plan is removed from a rule, any company override sets scoped to this rule still apply to the remaining service plans. If this is the LAST rule a service plan was part of, warn the admin.
```

---

## Prompt P3.3 — Company Override Set Editor: Configuration Preview Button

```
Add a "Preview Configuration" button to the Company Override Set editor page (both add and edit views). This reuses the same preview screen pattern from the 3-Way Rule editor (P3.1) but resolves one level deeper — through the company override layer.

**Button placement:**
- Add a "Preview Configuration" button in the header/toolbar area of the override set editor, next to Save and Cancel.
- Style it identically to the 3-way rule editor's preview button (secondary/outline button with eye/preview icon).
- The button should work on unsaved override sets — use the current in-memory override values the admin has entered. Do not require saving first.

**Preview display:**
Opens as a full-screen modal or slide-over panel (same component/pattern as the 3-way rule preview). The admin can close it and return to editing without losing work.

Shows ALL ~697 schema parameters with their resolved values through the company override level — one layer deeper than the 3-way rule preview.

**For each parameter, show:**

| Column | Description |
|--------|-------------|
| Parameter Name | The key (e.g., wan0_proto) |
| Display Name | Human-readable label |
| Category | For grouping |
| Value | The resolved value at this level of the hierarchy |
| Source | Where the value comes from — shown as a color-coded badge |
| In Model | Whether this parameter is in the model's selected parameter list (yes/no indicator) |

**Value resolution logic for each parameter:**
The preview resolves every schema parameter through the hierarchy up to (and including) the company override level:

1. If the parameter has a Company Override value set (from the current editor) → show that value, source badge: "Company Override" (orange)
2. Else if the parameter has a 3-Way Rule value → show that value, source badge: "3-Way Rule" (green)
3. Else if the parameter has a Model Default for this model → show that value, source badge: "Model Default" (blue)
4. Else if the parameter has a Schema Default → show that value, source badge: "Schema Default" (gray)
5. Else → show "Not Set", source badge: "No Value" (light/muted). If the parameter is marked as required, show "MISSING" in red instead.

**Layout and features:**

Header area:
- Title: "Configuration Preview: Cord Financial RMS Access"
- Subtitle: "Based on: I-22 + Verizon + [ATM, Standard] — Showing all 697 schema parameters resolved through the company override level"
- Summary stats bar:
  - Total parameters: 697
  - With values: X
  - From schema defaults: X (gray badge)
  - From model defaults: X (blue badge)
  - From 3-way rule: X (green badge)
  - From company override: X (orange badge)
  - Not set: X (muted)
  - Missing (required, no value): X (red badge, if any)
  - In model parameter list: X of 697

Parameter list:
- Same layout as the 3-way rule preview: grouped by category with collapsible sections.
- Model-selected parameters shown in normal style, non-model parameters dimmed.
- Within each category header, show: "Firewall (8 parameters — 6 in model, 2 overridden by 3-way rule, 3 overridden by company override)"

Filters/toggles:
- Search by parameter name or display name
- Filter: "Show all" (default) / "Model parameters only" / "Non-model parameters only"
- Filter: "Show only parameters with values"
- Filter: "Show only company overrides" — shows only parameters where this override set is actively setting a value
- Filter: "Show only 3-way rule + company overrides" — shows parameters overridden at either level
- Filter: "Show only missing required"

Export:
- "Export as Key-Value" button — same as 3-way rule preview, exports key=value format. This represents what a device's config would look like for companies assigned to this override set (before device-level overrides).
- "Export as CSV" button — all 697 rows with columns: parameter_name, value, source, in_model, category

Footer:
- "Close" button to return to the override set editor
- Note: "This preview shows resolution through the company override level. Device-level overrides are not included."

**Key difference from the 3-way rule preview:**
The 3-way rule preview shows resolution through 3 layers (schema → model → 3-way rule). This company override preview shows resolution through 4 layers (schema → model → 3-way rule → company override). The additional orange "Company Override" badges make it immediately clear which values this override set is changing on top of the 3-way rule baseline.
```

---
---

# Part 4: Device Page — Configuration Source and Preview

These prompts add configuration framework visibility and preview capabilities to the device detail page. Assumes the prototype already has a device detail/view page.

---

## Prompt P4.1 — Device Page: Configuration Source Selector and Indicator

```
Add a configuration source selector to the device detail/edit page. This tells the admin which configuration framework (Legacy or V2) the device is currently using, and allows the admin to manually switch between them.

**Placement:**
Add the configuration source selector near the top of the device detail page, below the device header/identification info but above the main content area. This should be clearly visible.

**Configuration Source Selector:**

A dropdown or radio button group with two options:
- "Legacy (.DAT File)"
- "V2 Configuration Engine"

Next to the selector, show contextual info based on the current selection:

When set to Legacy:
- Show the config file reference (e.g., "File: VZW_22_01282025.dat")
- Small badge: "Legacy" in amber/yellow

When set to V2 Engine:
- Show the resolved 3-way rule (e.g., "Rule: I-22 + Verizon + [ATM, Standard]")
- Show the company override set if any (e.g., "Override Set: Cord Financial RMS Access" or "No override set")
- Small badge: "V2" in teal/green

**Selector behavior:**
- Changing the selection simply updates the device's configuration source flag. No migration workflow, no prerequisites check — the admin manually decides when a device is ready to switch.
- The Configuration tab/section below updates to show the corresponding preview (Legacy or V2) based on the current selection.
- Switching back to Legacy is always allowed.

**Sample device data for prototype:**
Create 3-4 sample devices that can be navigated between (tabs, dropdown, or a small device list):
- Device "ATM-VZW-001" — Legacy, File: VZW_22_01282025.dat, Company: Cord Financial
- Device "ATM-VZW-002" — V2 Engine, Rule: I-22 + Verizon + [ATM, Standard], Override Set: Cord Financial RMS Access, Company: Cord Financial
- Device "ATM-TMO-003" — V2 Engine, Rule: I-22 + T-Mobile + [ATM], Override Set: none, Company: Loaded ATMs
- Device "ATM-VZW-004" — Legacy, File: VZW_4100_02152025.dat, Company: Miele
```

---

## Prompt P4.2 — Device Page: Legacy Configuration Preview

```
Add a "Configuration" section or tab to the device detail page. When the device's configuration source is set to "Legacy", show a preview of the legacy .DAT file configuration.

This is a temporary preview — it exists to give visibility into what the legacy file-based configuration contains while the old system is still in use.

**Header area:**
- Title: "Device Configuration (Legacy)"
- Subtitle: "File-based configuration — [filename, e.g., VZW_22_01282025.dat]"
- Badge: "Legacy" in amber/yellow

**Configuration display:**
Show the device's compiled legacy configuration as a key-value list, simulating what the .DAT file contains after all legacy layers are applied (base config file + custom company config + device custom_configurations).

Table with columns:
| Column | Description |
|--------|-------------|
| Parameter Name | The key (e.g., wan0_proto) |
| Value | The current value from the legacy configuration |
| Source | Where this value comes from in the legacy system — shown as a badge: "Base Config" (gray), "Company Config" (blue), "Device Override" (purple) |

The legacy source badges map to the old system's hierarchy:
- "Base Config" (gray) — value comes from the base .DAT file (the ConfigGroup configuration)
- "Company Config" (blue) — value was overridden by a custom company configuration file
- "Device Override" (purple) — value was overridden by the device's custom_configurations JSON field

Features:
- Search/filter by parameter name
- Filter: "Show all" / "Company overrides only" / "Device overrides only"
- Summary stats: "Total parameters: 697 | From base config: 650 | Company overrides: 35 | Device overrides: 12"
- Export as Key-Value button (exports the compiled .DAT-style key=value output)

Populate with sample data — ~30-40 visible parameters across categories with a mix of sources. Most from Base Config, some from Company Config (e.g., firewall rules), a few from Device Override (e.g., wifi_ssid, wifi_password).
```

---

## Prompt P4.3 — Device Page: V2 Configuration Preview

```
When the device's configuration source is set to "V2 Configuration Engine", show the V2 configuration preview in the same "Configuration" section/tab. This is the FINAL and COMPLETE resolution — all 5 levels of the hierarchy are resolved, including device-level overrides. This is what would be pushed to the device.

**Header area:**
- Title: "Device Configuration (V2 Engine)"
- Subtitle: "Fully resolved configuration for [device name]"
- Badge: "V2 Engine" in teal/green
- Context info (read-only):
  - Model: I-22 (CalAmp)
  - Carrier: Verizon
  - Service Plan: ATM
  - 3-Way Rule: I-22 + Verizon + [ATM, Standard]
  - Company: Cord Financial
  - Override Set: Cord Financial RMS Access (or "None" if no override set assigned)

**Configuration display:**

All ~697 schema parameters resolved through ALL 5 levels.

Table with columns:
| Column | Description |
|--------|-------------|
| Parameter Name | The key (e.g., wan0_proto) |
| Display Name | Human-readable label |
| Category | For grouping |
| Final Value | The fully resolved value — this is what the device gets |
| Source Layer | Which layer provided this value — color-coded badge |
| In Model | Whether this parameter is in the model's selected parameter list |

**Source layer badges (5 levels — full hierarchy):**
- "Schema Default" (gray) — value comes from the grand schema default
- "Model Default" (blue) — value was set as a model-level default
- "3-Way Rule" (green) — value was set in the model + carrier + service plan rule
- "Company Override" (orange) — value was set in the company's assigned override set
- "Device Override" (purple) — value was set at the individual device level
- "MISSING" (red) — required parameter with no value at any level

**Value resolution logic (all 5 levels):**
For each of the ~697 schema parameters:
1. If the device has a device-level override for this parameter → use it, badge: "Device Override" (purple)
2. Else if the device's company is assigned an override set for this 3-way rule, and it has a value → use it, badge: "Company Override" (orange)
3. Else if the 3-way rule has a value → use it, badge: "3-Way Rule" (green)
4. Else if the model has a default for this parameter → use it, badge: "Model Default" (blue)
5. Else if the schema has a default → use it, badge: "Schema Default" (gray)
6. Else → "Not Set" / "MISSING" if required

**Expandable detail per row:**
Click on any parameter row to expand and see the full resolution chain — every layer's value (or "not set") so the admin can trace exactly how the final value was determined:

```
wan0_proto
├── Schema Default:    "dhcp"          (gray)
├── Model Default:     —               (not set)
├── 3-Way Rule:        "static"        (green) ← overrode schema default
├── Company Override:  —               (not set, inherits 3-way rule)
├── Device Override:   —               (not set, inherits company/3-way rule)
└── Final Value:       "static"        Source: 3-Way Rule
```

Another example with device override:
```
wl0_ssid
├── Schema Default:    "WATM-Default"  (gray)
├── Model Default:     —               (not set)
├── 3-Way Rule:        "WATM-ATM"      (green)
├── Company Override:  "CordFinancial" (orange) ← overrode 3-way rule
├── Device Override:   "CordATM-FL01"  (purple) ← overrode company override
└── Final Value:       "CordATM-FL01"  Source: Device Override
```

**Layout and features:**

Summary stats bar (sticky at top):
- Total parameters: 697
- From schema defaults: X (gray)
- From model defaults: X (blue)
- From 3-way rule: X (green)
- From company override: X (orange)
- From device override: X (purple)
- Not set: X
- Missing (required): X (red, if any)

Parameter list:
- Grouped by category with collapsible sections
- Each category header shows counts per source: "Network (45 params — 20 schema, 5 model, 12 rule, 6 company, 2 device)"
- Model-selected parameters shown in normal style, non-model parameters dimmed (same pattern as other previews)

Filters/toggles:
- Search by parameter name or display name
- Source filter: "Show all" / "Schema defaults only" / "Model defaults only" / "3-Way Rule only" / "Company overrides only" / "Device overrides only"
- "Show only overridden (non-default)" — hides parameters still at schema default
- "Show only device overrides" — shows just the device-level changes
- "Show only missing required" — shows required parameters with no value at any level
- "Model parameters only" / "All parameters" toggle

Export:
- "Export as Key-Value" button — exports the final compiled key=value format (what would be pushed to the device). Only parameters with values.
- "Export as CSV" button — all 697 rows with columns: parameter_name, display_name, final_value, source_layer, category, in_model
- "Export as JSON" button — structured export with full resolution chain per parameter

Populate with sample data for a V2 device showing a realistic mix:
- ~400 parameters at Schema Default (gray) — the bulk of the config
- ~50 parameters at Model Default (blue) — model-specific constants
- ~40 parameters at 3-Way Rule (green) — carrier/service plan specific
- ~8 parameters at Company Override (orange) — company-specific (e.g., firewall, DNS)
- ~5 parameters at Device Override (purple) — device-specific (e.g., wifi SSID, password, hostname)
- ~2 parameters MISSING (red) — to demonstrate error flagging
```

---

## Prompt P4.4 — Device Page: Side-by-Side Legacy vs. V2 Comparison

```
Add a "Compare Legacy vs. V2" view to the device Configuration section/tab. Both configuration frameworks render a configuration for this device at all times — the device source selector only determines which one is actually applied. This comparison view lets the admin see both rendered configurations side by side to understand the differences.

**Access:**
A "Compare Legacy vs. V2" button on the device Configuration tab, always visible regardless of which configuration source the device is currently set to.

**Layout:**
Opens as a full-width view (can be inline on the page or a full-screen modal). Two-column side-by-side comparison:

| Left Column: Legacy Configuration | Right Column: V2 Configuration |
|-----------------------------------|-------------------------------|
| Header: "Legacy (.DAT File)" with amber badge | Header: "V2 Engine" with teal badge |
| File: VZW_22_01282025.dat | Rule: I-22 + Verizon + [ATM, Standard] |
| | Override Set: Cord Financial RMS Access |

A clear indicator showing which column represents the ACTIVE configuration (the one the device is currently set to use). For example, the active side gets a prominent "ACTIVE" badge or highlighted border.

**Comparison table:**
A single merged table where each row shows the same parameter from both systems:

| Column | Description |
|--------|-------------|
| Parameter Name | The key |
| Category | For grouping |
| Legacy Value | Value from the compiled legacy configuration |
| Legacy Source | Base Config / Company Config / Device Override |
| V2 Value | Value from the V2 engine resolution |
| V2 Source | Schema Default / Model Default / 3-Way Rule / Company Override / Device Override |
| Match | Visual indicator: ✓ (green check) if values are identical, ✗ (red X) if different, ⚠ (yellow warning) if parameter exists in one system but not the other |

**Row highlighting:**
- Matching values: no highlight (default/white background)
- Different values: light red/pink background — draws attention to discrepancies
- Parameter only in Legacy (not in V2 schema): light yellow background with note "Not in V2 schema"
- Parameter only in V2 (new schema parameter not in legacy file): light blue background with note "New in V2"

**Summary stats bar:**
- Total parameters compared: X
- Matching: X (green)
- Different: X (red)
- Only in Legacy: X (yellow)
- Only in V2: X (blue)
- Match rate: X%

**Filters/toggles:**
- "Show all" (default)
- "Show only differences" — hides matching parameters, focuses on discrepancies
- "Show only matching" — confirms what's aligned
- "Show only in Legacy" — parameters in legacy but not V2
- "Show only in V2" — parameters in V2 but not legacy
- Search by parameter name
- Group by category with collapsible sections

**Footer:**
- "Close" button to return to the device Configuration tab
- Note: "Both configurations are always rendered. The device source selector determines which configuration is applied to the device."
```

---

## Prompt P4.5 — Device Page V2 Configuration: Preview Button

```
Add a "Preview Configuration" button to the Device Configuration view when the device is on the V2 Configuration Engine. This follows the same preview pattern used on the 3-Way Rule editor (P3.1) and Company Override Set editor (P3.3), but resolves through ALL 5 levels — this is the deepest and final resolution in the hierarchy.

**Button placement:**
- Add a "Preview Configuration" button in the header area of the V2 configuration section on the device page, near the context info (model, carrier, service plan, company, override set).
- Style it identically to the preview buttons on the 3-way rule and override set editors (secondary/outline button with eye/preview icon).

**Preview display:**
Opens as a full-screen modal or slide-over panel (same component/pattern as the other preview screens). The admin can close it and return to the device page.

Shows ALL ~697 schema parameters with their resolved values through ALL 5 levels of the hierarchy — the complete, final configuration that would be pushed to this device.

**For each parameter, show:**

| Column | Description |
|--------|-------------|
| Parameter Name | The key (e.g., wan0_proto) |
| Display Name | Human-readable label |
| Category | For grouping |
| Value | The final resolved value for this device |
| Source | Where the value comes from — shown as a color-coded badge |
| In Model | Whether this parameter is in the model's selected parameter list (yes/no indicator) |

**Value resolution logic (all 5 levels):**
The preview resolves every schema parameter through the complete hierarchy:

1. If the device has a device-level override → show that value, source badge: "Device Override" (purple)
2. Else if the device's company is assigned an override set for this 3-way rule, and it has a value → show that value, source badge: "Company Override" (orange)
3. Else if the 3-way rule has a value → show that value, source badge: "3-Way Rule" (green)
4. Else if the model has a default → show that value, source badge: "Model Default" (blue)
5. Else if the schema has a default → show that value, source badge: "Schema Default" (gray)
6. Else → show "Not Set", source badge: "No Value" (light/muted). If required, show "MISSING" in red.

**Expandable detail per row:**
Click on any parameter row to expand and see the full resolution chain — every layer's value (or "not set"):

```
fw_table_1
├── Schema Default:    ""              (gray)
├── Model Default:     —               (not set)
├── 3-Way Rule:        "DENY ALL"      (green)
├── Company Override:  "ALLOW 10.0.1.0/24"  (orange) ← Cord Financial RMS Access
├── Device Override:   —               (not set, inherits company override)
└── Final Value:       "ALLOW 10.0.1.0/24"  Source: Company Override
```

**Layout and features:**

Header area:
- Title: "Configuration Preview: [device name]"
- Subtitle: "Complete resolution through all 5 levels"
- Context line: "Model: I-22 | Carrier: Verizon | Service Plan: ATM | Company: Cord Financial | Override Set: Cord Financial RMS Access"
- Summary stats bar:
  - Total parameters: 697
  - With values: X
  - From schema defaults: X (gray badge)
  - From model defaults: X (blue badge)
  - From 3-way rule: X (green badge)
  - From company override: X (orange badge)
  - From device override: X (purple badge)
  - Not set: X (muted)
  - Missing (required, no value): X (red badge, if any)
  - In model parameter list: X of 697

Parameter list:
- Grouped by category with collapsible sections.
- Each category header shows: "Firewall (8 parameters — 2 schema, 1 model, 3 rule, 1 company, 1 device)"
- Model-selected parameters shown in normal style, non-model parameters dimmed.

Filters/toggles:
- Search by parameter name or display name
- Filter: "Show all" (default) / "Model parameters only" / "Non-model parameters only"
- Source filter: "All sources" / "Schema defaults only" / "Model defaults only" / "3-Way Rule only" / "Company overrides only" / "Device overrides only"
- "Show only parameters with values"
- "Show only overridden (non-schema-default)" — shows parameters where any layer beyond schema default is providing the value
- "Show only missing required"

Export:
- "Export as Key-Value" button — exports the final compiled key=value format. This is the complete device configuration — what would be pushed to the device.
- "Export as CSV" button — all 697 rows with columns: parameter_name, value, source, in_model, category
- "Export as JSON" button — structured export with the full resolution chain per parameter (all 5 layers)

Footer:
- "Close" button to return to the device page
- Note: "This is the complete device configuration resolved through all 5 levels: Schema Default → Model Default → 3-Way Rule → Company Override → Device Override."

**Key difference from other previews:**
- 3-Way Rule preview resolves 3 levels (schema → model → rule)
- Company Override Set preview resolves 4 levels (schema → model → rule → company override)
- This Device preview resolves ALL 5 levels (schema → model → rule → company override → device override) — it is the final, complete configuration
```

---
---

# Part 5: Configuration Change Log

---

## Prompt P5.1 — Change History on Configuration Screens

```
Add a "Change History" section or tab to the following configuration screens: Model Configuration detail page, 3-Way Rule editor page, and Company Override Set editor page. This tracks who changed what and when for each configuration entity.

**What to add to each screen:**

A "Change History" tab or collapsible section (at the bottom of the page or as a secondary tab alongside the main content) showing a chronological list of changes. Most recent first.

Each log entry displays:
- Timestamp (date and time, e.g., "Mar 15, 2026 at 2:34 PM")
- User (name and avatar/initials badge, e.g., "Adam Curcie" with an "AC" circle badge)
- Action description — a human-readable summary of what changed

**Log entry formats per entity type:**

Model Configuration:
- "Added 12 parameters to model parameter list (Network: 5, Firewall: 3, WiFi: 4)"
- "Removed 2 parameters from model parameter list (IO: io_legacy_port, io_legacy_enable)"
- "Updated model default: io_enable changed from '' to '1'"
- "Updated 3 model default values (Network: 2, Cellular: 1)"

3-Way Rule:
- "Created 3-Way Rule: I-22 + Verizon + [ATM, Standard]"
- "Updated 5 parameter values (Firewall: 2, Network: 3)"
- "Changed wan0_proto from 'dhcp' to 'static'"
- "Added service plan: Enterprise"
- "Removed service plan: Standard"
- "Cleared value for dns_secondary (reverted to baseline)"

Company Override Set:
- "Created override set: Cord Financial RMS Access"
- "Updated 3 parameter override values (Firewall: 2, DNS: 1)"
- "Changed fw_table_1 from 'DENY ALL' to 'ALLOW 10.0.1.0/24'"
- "Assigned 4 companies: Cord Financial, Loaded ATMs, ABC Corp, XYZ Inc"
- "Unassigned 1 company: XYZ Inc"
- "Reassigned Loaded ATMs from 'Standard DNS Override' to this set"

**Layout:**

Each log entry is a row/card with:
- Left: timestamp (smaller, muted text)
- Center: user badge (initials circle) + user name
- Right/below: action description

Features:
- Show the 10 most recent entries by default
- "Show more" / "Load older" button to paginate
- Filter by user (dropdown of users who have made changes)
- Filter by date range

**Sample data:**
Populate each screen with 5-8 sample log entries from different users (Adam Curcie, Devon D'Andrea, Aksana Rahouski) over the past few weeks.

Example for the I-22 Model Configuration:
- Mar 15, 2026 2:34 PM — Adam Curcie — "Updated model default: io_enable changed from '' to '1'"
- Mar 14, 2026 11:15 AM — Aksana Rahouski — "Added 8 parameters to model parameter list (IO: 4, Monitor: 4)"
- Mar 12, 2026 3:45 PM — Adam Curcie — "Updated 5 model default values (Network: 3, Cellular: 2)"
- Mar 10, 2026 9:20 AM — Devon D'Andrea — "Removed 1 parameter from model parameter list (System: sys_legacy_flag)"
- Mar 8, 2026 4:10 PM — Aksana Rahouski — "Created model configuration for I-22. Selected 142 parameters."

Example for the I-22 + Verizon + [ATM, Standard] 3-Way Rule:
- Mar 16, 2026 10:05 AM — Adam Curcie — "Updated 3 parameter values (Firewall: 1, Network: 2)"
- Mar 15, 2026 3:20 PM — Adam Curcie — "Added service plan: Standard"
- Mar 14, 2026 2:00 PM — Aksana Rahouski — "Updated wan0_proto from 'dhcp' to 'static'"
- Mar 12, 2026 11:30 AM — Devon D'Andrea — "Created 3-Way Rule: I-22 + Verizon + [ATM]"

Example for Cord Financial RMS Access override set:
- Mar 17, 2026 9:45 AM — Adam Curcie — "Assigned 2 companies: ABC Corp, Loaded ATMs"
- Mar 16, 2026 4:30 PM — Adam Curcie — "Updated fw_table_1 from 'DENY ALL' to 'ALLOW 10.0.1.0/24'"
- Mar 15, 2026 1:15 PM — Aksana Rahouski — "Created override set: Cord Financial RMS Access. Overriding 8 parameters. Assigned to Cord Financial."
```

---

## Prompt P5.2 — Configuration Management Landing Page: Recent Activity Feed

```
Update the Configuration Management landing page (from Prompt 0.2) to include a "Recent Activity" feed showing the latest configuration changes across all entity types.

**Placement:**
Add a "Recent Activity" section below the four card-style links to the sub-sections. This gives the admin an at-a-glance view of what's been changing in the configuration system.

**Layout:**
A vertical timeline-style feed showing the most recent changes across all configuration entities (model configurations, 3-way rules, company override sets).

Each entry shows:
- Timestamp (date and time)
- User (name with initials badge)
- Entity type icon + label (e.g., model icon + "Model Configuration", rule icon + "3-Way Rule", company icon + "Override Set")
- Entity name (e.g., "I-22", "I-22 + Verizon + [ATM, Standard]", "Cord Financial RMS Access") — clickable, links to that entity's edit page
- Action description (same format as the per-entity change history)

Features:
- Show the 15 most recent entries by default
- "View all activity" link to a full activity log page (or just scroll/load more)
- Filter by entity type (Model / 3-Way Rule / Override Set)
- Filter by user

**Sample data:**
Mix entries from all three entity types, ordered by timestamp (most recent first):

- Mar 17, 2026 9:45 AM — Adam Curcie — [Override Set] Cord Financial RMS Access — "Assigned 2 companies: ABC Corp, Loaded ATMs"
- Mar 16, 2026 4:30 PM — Adam Curcie — [Override Set] Cord Financial RMS Access — "Updated fw_table_1 from 'DENY ALL' to 'ALLOW 10.0.1.0/24'"
- Mar 16, 2026 10:05 AM — Adam Curcie — [3-Way Rule] I-22 + Verizon + [ATM, Standard] — "Updated 3 parameter values"
- Mar 15, 2026 3:20 PM — Adam Curcie — [3-Way Rule] I-22 + Verizon + [ATM, Standard] — "Added service plan: Standard"
- Mar 15, 2026 2:34 PM — Adam Curcie — [Model Config] I-22 — "Updated model default: io_enable = '1'"
- Mar 15, 2026 1:15 PM — Aksana Rahouski — [Override Set] Cord Financial RMS Access — "Created override set. 8 parameters. Assigned to Cord Financial."
- Mar 14, 2026 2:00 PM — Aksana Rahouski — [3-Way Rule] I-22 + Verizon + [ATM] — "Updated wan0_proto to 'static'"
- Mar 14, 2026 11:15 AM — Aksana Rahouski — [Model Config] I-22 — "Added 8 parameters (IO: 4, Monitor: 4)"
- Mar 12, 2026 3:45 PM — Adam Curcie — [Model Config] I-22 — "Updated 5 model default values"
- Mar 12, 2026 11:30 AM — Devon D'Andrea — [3-Way Rule] I-22 + Verizon + [ATM] — "Created 3-Way Rule"
```

---

## Phase 7: Distributor / Sub-Company Override Inheritance

These prompts add distributor sub-company inheritance handling to the override set system. They modify the override set editor (Phase 4), the company edit page, and the browse page. Run these after the base Phase 4 prompts are working.

**Context:** Distributors (like Miele) can have hundreds of sub-companies. When a distributor is assigned an override set, sub-companies inherit it automatically via an "Apply to sub-companies" flag. Individual sub-companies can be unlinked (falling back to the 3-way rule) or assigned their own override set. There are no nested distributors — inheritance is one level only.

### Prompt 7.1 — Distributor Sub-Company Inheritance in Override Set Editor

```
Update the Company Assignments section (Section 2) of the override set editor to handle distributor companies with sub-company inheritance.

**Current behavior to keep:** The two-panel assign/unassign layout for standalone (non-distributor) companies remains unchanged.

**New behavior for distributors:**

When a distributor company is assigned to this override set, it appears in the "Assigned Companies" panel with an expandable row. Add a checkbox below the distributor name: "☑ Apply to sub-companies". When checked, the distributor row expands to show a sub-company inheritance table.

The sub-company table has these columns:
- Sub-Company Name
- Status — one of:
  - "● Inheriting" (green dot) — using this override set via parent inheritance
  - "○ Unlinked" (gray dot) — not using this override set
- Detail — for unlinked sub-companies, show one of:
  - The name of another override set they're assigned to (as a clickable link that opens that override set)
  - "(no override — falls back to 3-way rule)" if they have no override set at all
- Action — a button:
  - [Unlink] for inheriting sub-companies
  - [Re-link] for unlinked sub-companies

Below the sub-company table, show a summary line: "Sub-companies: X inheriting, Y unlinked"

**Collapsed state:** When the distributor row is collapsed (default when many sub-companies), show just the distributor name and the summary count inline: "Miele (Distributor) — 248 sub-companies: 245 inheriting, 3 unlinked"

**Important constraints:**
- Only sub-companies whose devices use the same 3-way rule as this override set appear in the sub-company table. Sub-companies on a different 3-way rule are not shown (they're not eligible for this override set).
- There are no nested distributors. Inheritance is one level only: distributor → direct sub-companies.
- Assigning a sub-company to a DIFFERENT override set (from that other override set's editor) should automatically show it as "Unlinked" here with a reference to the other override set.
- The "Apply to sub-companies" checkbox defaults to checked when a distributor is first assigned. Unchecking it removes the inheritance — all sub-companies stop inheriting and fall back to the 3-way rule (or their own override set if they have one).

**Sample data to populate:**
- Miele (Distributor) assigned with "Apply to sub-companies" checked:
  - Miele Sub-Co 1 through Sub-Co 6: four inheriting, two unlinked
  - Miele Sub-Co 4: unlinked, assigned to "Sub-Co 4 Custom" override set
  - Miele Sub-Co 5: unlinked, no override set (falls back to 3-way rule)
- Cord Financial and Loaded ATMs as standalone (non-distributor) companies, no sub-company table
```

### Prompt 7.2 — Company Edit Page: Override Set Assignment View

```
Build a "Configuration Overrides" section on the Company Edit page. This section shows which override sets are assigned to this company and allows linking/unlinking. This is a CONSUMER view — the company page links to override sets but doesn't own them.

**Layout:** A table showing all 3-way rules relevant to this company's devices, and the override set assigned (if any) for each.

Columns:
- 3-Way Rule (displayed as "Model + Carrier + Service Plan(s)", e.g., "I-22 + Verizon + [ATM]")
- Override Set (name of the assigned override set, or "(none)" if no override)
- Source — how this company got the override set:
  - "Direct" — the company is directly assigned
  - "Inherited from [Parent Name]" — the company inherits from its distributor parent
- Actions:
  - [View] — opens the override set editor (read-only or navigates to the Config Admin section)
  - [Change] — opens a modal to pick a different override set for this 3-way rule (shows available override sets scoped to that 3-way rule)
  - [Remove] — unassigns the override set (falls back to 3-way rule values)

**Distributor-specific additions:**

If this company IS a distributor, for each row that has an override set assigned, show an additional line below:
- "☑ Applied to sub-companies (X of Y inheriting)" — clickable to expand a mini sub-company status list, or links to the override set editor where the full sub-company table lives.

If this company IS a sub-company, for each row where the override set is inherited:
- Source column shows "Inherited from [Parent Name]"
- Actions show:
  - [Use Parent's] — grayed out / current state indicator (already using parent's)
  - [Override] — opens modal to pick a different override set, which unlinks from parent
- If the sub-company was previously unlinked and has its own override set:
  - Source column shows "Direct (Parent uses: [Parent's Override Set Name])"
  - Actions show:
  - [Use Parent's] — re-links to the parent's override set
  - [Change] — pick a different override set

**Sample data for distributor view (Miele):**
- I-22 + Verizon + [ATM] → "Miele Network Config" → Direct → ☑ Applied to subs (245 of 248 inheriting)

**Sample data for sub-company view (Miele Sub-Co 4):**
- I-22 + Verizon + [ATM] → "Sub-Co 4 Custom" → Direct (Parent uses: Miele Network Config) → [Use Parent's] [Change]

**Sample data for sub-company view (Miele Sub-Co 1, inheriting):**
- I-22 + Verizon + [ATM] → "Miele Network Config" → Inherited from Miele → [Override]
```

### Prompt 7.3 — Override Set Browse Page: Distributor Column Update

```
Update the Company Override Sets browse page (Prompt 4.1) to better reflect distributor inheritance in the "Companies Assigned" column.

Current column shows just a count. Change it to show a breakdown:

- For override sets with no distributor assignments: show count as before (e.g., "12 companies")
- For override sets with distributor assignments: show a two-line display:
  - Line 1: "X companies (Y direct, Z inherited)"
  - Line 2: small text showing distributor names, e.g., "via: Miele"

The expandable company list (when clicking the count) should now group companies:

**Directly Assigned:**
- Cord Financial
- Loaded ATMs

**Inherited (via Miele):**
- Miele Sub-Co 1
- Miele Sub-Co 2
- Miele Sub-Co 3
- Miele Sub-Co 6

**Unlinked from Miele (not using this set):**
- Miele Sub-Co 4 → using "Sub-Co 4 Custom"
- Miele Sub-Co 5 → no override (3-way rule)

Update the sample data for "Miele Network Config" to show: "7 companies (3 direct, 4 inherited)" with "via: Miele" underneath.
```

---

## Phase 8: Grand Schema Change History and Activity Feed

These prompts add change tracking to the Grand Schema screens and integrate schema activity into the existing change log infrastructure. Prompts 8.1 and 8.2 are new screens/sections. Prompts 8.3 and 8.4 are incremental enhancements to the already-built P5.1 and P5.2 screens.

### Prompt 8.1 — Grand Schema Browse Page: Recent Schema Activity Feed

```
Update the Grand Schema browse page (from Prompt 1.1) to include a "Recent Schema Activity" section that shows recent changes across all schema parameters.

**Placement:**
Add a collapsible "Recent Schema Activity" panel above the parameter table. Default state: collapsed, showing just the header with the count of changes in the last 7 days (e.g., "Recent Schema Activity (6 changes this week)"). Click to expand.

**Expanded layout:**
A compact activity feed showing the most recent schema changes. Each entry shows:
- Timestamp (date and time)
- User (name with initials badge)
- Parameter name (clickable — navigates to that parameter's edit page)
- Action description (e.g., "Created parameter", "Updated default value", "Changed data type", "Deleted parameter")

Features:
- Show the 10 most recent entries by default
- "View all schema activity" link to expand or paginate
- Filter by user
- Filter by action type (Created / Updated / Deleted)

**Sample data (most recent first):**
- Mar 18, 2026 11:20 AM — Adam Curcie — wan0_proto — "Updated default value from 'dhcp' to 'static'"
- Mar 17, 2026 3:10 PM — Devon D'Andrea — cell_band_lock — "Created parameter (Category: Cellular, Type: string)"
- Mar 16, 2026 2:45 PM — Aksana Rahouski — fw_rule_count — "Updated max_value from 50 to 100"
- Mar 15, 2026 10:30 AM — Adam Curcie — wl0_password — "Changed 'Encrypted' from No to Yes"
- Mar 14, 2026 3:05 PM — Aksana Rahouski — wan0_proto — "Changed 'Available at Company Level' from No to Yes"
- Mar 13, 2026 11:00 AM — Devon D'Andrea — sys_legacy_flag — "Deleted parameter"
- Mar 12, 2026 4:20 PM — Adam Curcie — mon_checkin_url — "Updated validation regex"
- Mar 10, 2026 2:15 PM — Devon D'Andrea — wan0_proto — "Updated description"
```

### Prompt 8.2 — Grand Schema Parameter Edit Page: Change History Tab

```
Update the Grand Schema parameter edit page (from Prompt 1.2) to include a "Change History" tab or section showing the full edit history for that specific parameter.

**Placement:**
Add a "Change History" tab alongside the main edit form. Alternatively, add a collapsible "Change History" section below the form fields. The history should be clearly separated from the editable fields.

**Layout:**
A chronological list of all changes made to this parameter, most recent first. Each entry shows:
- Timestamp (date and time)
- User (name with initials badge)
- Action description with specific field-level detail:

**Log entry formats:**
- "Created parameter: wan0_proto (Category: Network, Type: string, Default: 'dhcp')"
- "Updated default value: changed from 'dhcp' to 'static'"
- "Updated data type: changed from 'string' to 'integer'"
- "Updated category: changed from 'Network' to 'System'"
- "Updated display name: changed from 'WAN Protocol' to 'Primary WAN Protocol'"
- "Updated description"
- "Updated validation regex: added pattern '^[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}$'"
- "Updated min_value: changed from 0 to 1"
- "Updated max_value: changed from 50 to 100"
- "Changed 'Required' from No to Yes"
- "Changed 'Encrypted' from No to Yes"
- "Changed 'Available at Company Level' from Yes to No"
- "Changed 'Available at Device Level' from No to Yes"

When multiple fields are changed in a single save, show them as one log entry with all changes listed:
- "Updated 3 fields: default value ('dhcp' → 'static'), Required (No → Yes), description updated"

Features:
- Show the 10 most recent entries by default
- "Show more" / "Load older" button to paginate
- Filter by user
- Filter by date range

**Sample data for wan0_proto parameter:**
- Mar 18, 2026 11:20 AM — Adam Curcie — "Updated default value: changed from 'dhcp' to 'static'"
- Mar 14, 2026 3:05 PM — Aksana Rahouski — "Changed 'Available at Company Level' from No to Yes"
- Mar 10, 2026 2:15 PM — Devon D'Andrea — "Updated description"
- Mar 7, 2026 10:30 AM — Adam Curcie — "Created parameter: wan0_proto (Category: Network, Type: string, Default: 'dhcp')"

**Sample data for fw_rule_count parameter:**
- Mar 16, 2026 2:45 PM — Aksana Rahouski — "Updated max_value: changed from 50 to 100"
- Mar 11, 2026 9:00 AM — Adam Curcie — "Updated 2 fields: min_value (null → 1), Required (No → Yes)"
- Mar 7, 2026 11:45 AM — Devon D'Andrea — "Created parameter: fw_rule_count (Category: Firewall, Type: integer, Default: 10)"
```

### Prompt 8.3 — Enhance Existing Change History Screens (P5.1) to Include Grand Schema

```
The Change History tabs on the Model Configuration, 3-Way Rule, and Override Set editor pages are already built (from P5.1). Now add the same Change History pattern to the Grand Schema parameter edit page.

**What to add:**
On the existing Grand Schema parameter edit page (built from Prompt 1.2), add a "Change History" tab or collapsible section — using the same component/layout pattern already used on the Model Configuration, 3-Way Rule, and Override Set editor pages.

Use the same log entry row format (timestamp, user initials badge + name, action description). Same "Show more" pagination, same user and date range filters.

**Grand Schema-specific log entry formats:**
- "Created parameter: wan0_proto (Category: Network, Type: string, Default: 'dhcp')"
- "Updated default value: changed from 'dhcp' to 'static'"
- "Updated data type: changed from 'string' to 'integer'"
- "Updated category: changed from 'Network' to 'System'"
- "Updated display name: changed from 'WAN Protocol' to 'Primary WAN Protocol'"
- "Updated description"
- "Updated validation regex: added pattern '^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$'"
- "Updated min_value: changed from 0 to 1"
- "Updated max_value: changed from 50 to 100"
- "Changed 'Required' from No to Yes"
- "Changed 'Encrypted' from No to Yes"
- "Changed 'Available at Company Level' from Yes to No"
- "Changed 'Available at Device Level' from No to Yes"

When multiple fields are changed in a single save, group them into one entry:
- "Updated 3 fields: default value ('dhcp' → 'static'), Required (No → Yes), description updated"

**Sample data for wan0_proto parameter:**
- Mar 18, 2026 11:20 AM — Adam Curcie — "Updated default value: changed from 'dhcp' to 'static'"
- Mar 14, 2026 3:05 PM — Aksana Rahouski — "Changed 'Available at Company Level' from No to Yes"
- Mar 10, 2026 2:15 PM — Devon D'Andrea — "Updated description"
- Mar 7, 2026 10:30 AM — Adam Curcie — "Created parameter: wan0_proto (Category: Network, Type: string, Default: 'dhcp')"
```

### Prompt 8.4 — Enhance Existing Landing Page Activity Feed (P5.2) to Include Grand Schema

```
The "Recent Activity" feed on the Configuration Management landing page is already built (from P5.2). It currently shows activity for Model Configuration, 3-Way Rule, and Override Set entity types. Enhance it to also include Grand Schema changes.

**Changes needed:**

1. Add a new entity type to the feed: "Grand Schema" with a schema/database icon (consistent with the Grand Schema icon used on the landing page card from Prompt 0.2).

2. Add "Grand Schema" as a new option in the existing entity type filter dropdown (alongside Model / 3-Way Rule / Override Set).

3. Grand Schema entries in the feed should show:
   - The schema icon + "Grand Schema" label
   - The parameter name (clickable — links to that parameter's edit page)
   - The action description

4. Insert these sample Grand Schema entries into the existing timeline at the appropriate chronological positions (they should interleave with the existing entries, not appear as a separate block):

   - Mar 18, 2026 11:20 AM — Adam Curcie — [Grand Schema] wan0_proto — "Updated default value from 'dhcp' to 'static'"
   - Mar 17, 2026 3:10 PM — Devon D'Andrea — [Grand Schema] cell_band_lock — "Created parameter (Category: Cellular, Type: string)"
   - Mar 13, 2026 11:00 AM — Devon D'Andrea — [Grand Schema] sys_legacy_flag — "Deleted parameter"

These should appear at the top of the feed (they are the most recent entries) above the existing Mar 17 Override Set entry.
```

---

## Phase 9: Split Company Assignment into Standalone and Distributor Sections

The current override set editor (Prompt 4.2) has a single "Company Assignments" section with a two-panel assign/unassign layout. Prompt 7.1 added distributor sub-company inheritance into that same panel. This phase separates the assignment area into two distinct sections with different UX flows — because assigning a standalone company is a simple pick-and-move operation, while assigning a distributor involves inheritance decisions (apply to sub-companies, unlink/re-link) that need their own dedicated UI.

### Prompt 9.1 — Split Company Assignments into Two Sections

```
The override set editor currently has one "Section 2: Company Assignments" area with a two-panel layout (Available ↔ Assigned). Replace this single section with two separate assignment sections, each with its own header, layout, and workflow. Keep Section 1 (Parameter Overrides) unchanged.

**New layout for the assignment area:**

Replace the current single "Company Assignments" section with two clearly separated sections using tabs, or stacked panels with distinct headers. Tabs are preferred — "Standalone Companies" tab and "Distributors" tab — so only one is visible at a time and the page doesn't get too long.

Each tab should show a count badge: "Standalone Companies (14)" and "Distributors (2)".

---

**Section 2A: Standalone Company Assignments**

This section handles companies that are NOT distributors (no sub-companies). This is the simpler flow — pick companies, move them over.

Keep the existing two-panel layout:
- Left panel: "Available Companies" — standalone companies (non-distributors) whose devices use this 3-way rule and are NOT currently assigned to this override set. Searchable, showing company name and device count.
- Right panel: "Assigned Companies" — standalone companies currently assigned. Show company name and device count.
- Move buttons between panels (assign/unassign). Support multi-select.
- Show counts: "Available: X" / "Assigned: Y"

Same constraint enforcement as before:
- If a standalone company is already assigned to a DIFFERENT override set for the same 3-way rule, show a warning icon. On assign attempt, show confirmation: "This company is currently assigned to [Other Override Set Name]. Reassign to this override set?"

Important: Distributors should NOT appear in this panel at all — they have their own section.

**Sample data:**
Available: 8 standalone companies (e.g., "ABC Corp (12 devices)", "XYZ Inc (5 devices)", etc.)
Assigned: Cord Financial (18 devices), Loaded ATMs (9 devices), Company C (6 devices)

---

**Section 2B: Distributor Assignments**

This section handles distributor companies and their sub-company inheritance. This is the more complex flow — assigning a distributor means making inheritance decisions about potentially hundreds of sub-companies.

Layout is NOT a two-panel picker. Instead, use a card/list-based layout:

**Top area: "Add Distributor" button + search dropdown**
- Clicking "Add Distributor" opens a searchable dropdown showing available distributors whose devices use this 3-way rule
- Only distributor companies appear here (not standalone companies)
- Same conflict warning: if the distributor is already assigned to a different override set, show confirmation before reassigning

**Below: Assigned Distributors list**
Each assigned distributor appears as an expandable card/panel:

┌─────────────────────────────────────────────────────────────┐
│ ▼ Miele                                          [Remove]  │
│   Distributor · 248 sub-companies on this 3-way rule       │
│                                                             │
│   ☑ Apply to sub-companies                                  │
│                                                             │
│   ┌───────────────────────────────────────────────────────┐ │
│   │ Sub-Company          │ Status       │ Action          │ │
│   │──────────────────────│──────────────│─────────────────│ │
│   │ Miele Sub-Co 1       │ ● Inheriting │ [Unlink]        │ │
│   │ Miele Sub-Co 2       │ ● Inheriting │ [Unlink]        │ │
│   │ Miele Sub-Co 3       │ ● Inheriting │ [Unlink]        │ │
│   │ Miele Sub-Co 4       │ ○ Unlinked   │ [Re-link]       │ │
│   │                      │  → "Sub-Co 4 Custom"           │ │
│   │ Miele Sub-Co 5       │ ○ Unlinked   │ [Re-link]       │ │
│   │                      │  → (3-way rule fallback)       │ │
│   │ Miele Sub-Co 6       │ ● Inheriting │ [Unlink]        │ │
│   └───────────────────────────────────────────────────────┘ │
│   Summary: 4 inheriting, 2 unlinked                         │
│                                                             │
│   [Search sub-companies...]                                 │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ ▶ National ATM Services                          [Remove]  │
│   Distributor · 34 sub-companies · ☑ Apply to subs         │
│   32 inheriting, 2 unlinked                    [Expand ▼]  │
└─────────────────────────────────────────────────────────────┘

**Distributor card details:**

Header row (always visible, even when collapsed):
- Distributor name
- "Distributor" label badge
- Sub-company count on this 3-way rule
- If collapsed: inline summary "32 inheriting, 2 unlinked" + [Expand] control
- [Remove] button — removes the distributor (and all sub-company inheritance) from this override set. Show confirmation: "This will remove Miele and stop inheritance for X sub-companies."

Expanded content:
- "Apply to sub-companies" checkbox
  - When checked (default): sub-company table is shown, new sub-companies auto-inherit
  - When unchecked: sub-company table hides, all sub-companies stop inheriting (fall back to 3-way rule or their own override set). Show confirmation before unchecking: "This will stop X sub-companies from inheriting this override set."
- Sub-company table (same columns as described in Prompt 7.1):
  - Sub-Company Name
  - Status: "● Inheriting" (green) or "○ Unlinked" (gray)
  - Detail for unlinked: name of other override set (clickable link) or "(3-way rule fallback)"
  - Action: [Unlink] or [Re-link]
- Search/filter bar within the sub-company table (important for distributors with hundreds of subs)
- Summary line: "X inheriting, Y unlinked"

**Important behaviors:**
- Only sub-companies whose devices use this override set's 3-way rule appear in the table
- No nested distributors — one level only
- Collapsed is the default state for distributors with more than 20 sub-companies
- Expanded is the default for distributors with 20 or fewer sub-companies

**Sample data:**
- Miele: 248 sub-companies, expanded, 4 inheriting / 2 unlinked (use Sub-Co 1-6 as shown above)
- National ATM Services: 34 sub-companies, collapsed, 32 inheriting / 2 unlinked
```

### Prompt 9.2 — Distributor Assignment: Onboarding Flow

```
When an admin clicks "Add Distributor" and selects a distributor from the dropdown in the Distributor Assignments section (Prompt 9.1), show a short onboarding flow before the distributor card appears in the assigned list. This guides the admin through the inheritance decision upfront.

**Step 1: Confirmation + inheritance choice**

Show a modal or inline panel:

┌─────────────────────────────────────────────────────────────┐
│ Assign Distributor: Miele                                   │
│                                                             │
│ Miele has 248 sub-companies with devices on this 3-way rule │
│ (I-22 + Verizon + [ATM]).                                   │
│                                                             │
│ How should sub-companies be handled?                        │
│                                                             │
│ ○ Apply to all sub-companies                                │
│   All 248 sub-companies will inherit this override set.     │
│   You can unlink individual sub-companies afterward.        │
│                                                             │
│ ○ Assign distributor only                                   │
│   Only Miele itself gets this override set.                 │
│   Sub-companies are not affected (they keep their current   │
│   configuration — either their own override set or the      │
│   3-way rule values).                                       │
│                                                             │
│                              [Cancel]  [Assign Distributor] │
└─────────────────────────────────────────────────────────────┘

Default selection: "Apply to all sub-companies" (matches the most common workflow — Adam's Miele example where hundreds of subs get the same config).

**Step 2: Conflict resolution (only if needed)**

If "Apply to all sub-companies" was selected AND some sub-companies are currently assigned to a different override set for this 3-way rule, show a second step before finalizing:

┌─────────────────────────────────────────────────────────────┐
│ Sub-Company Conflicts                                       │
│                                                             │
│ 3 of Miele's 248 sub-companies are currently assigned to    │
│ a different override set for this 3-way rule:               │
│                                                             │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Sub-Company      │ Current Override Set     │ Action    │ │
│ │──────────────────│─────────────────────────│───────────│ │
│ │ Miele Sub-Co 4   │ Sub-Co 4 Custom          │ ○ Keep    │ │
│ │                  │                          │ ○ Replace │ │
│ │ Miele Sub-Co 5   │ Miele Legacy Firewall    │ ○ Keep    │ │
│ │                  │                          │ ○ Replace │ │
│ │ Miele Sub-Co 12  │ Special DNS Config       │ ○ Keep    │ │
│ │                  │                          │ ○ Replace │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ "Keep" = sub-company stays on its current override set      │
│ and appears as "Unlinked" in the inheritance table.         │
│                                                             │
│ "Replace" = sub-company is reassigned to this override set  │
│ and appears as "Inheriting".                                │
│                                                             │
│ Default: Keep (preserves existing sub-company configs)      │
│                                                             │
│                                    [Back]  [Confirm & Add]  │
└─────────────────────────────────────────────────────────────┘

After confirmation, the distributor card appears in the Distributor Assignments section in its expanded state so the admin can immediately see the inheritance status.

**If "Assign distributor only" was selected:** No conflict step needed. The distributor card appears with "Apply to sub-companies" unchecked and no sub-company table.

**Sample scenario to populate:**
- Admin adds Miele with "Apply to all sub-companies"
- 3 sub-companies have existing override sets → conflict screen appears
- Admin keeps Sub-Co 4 on its own set, replaces the other two
- Result: Miele card shows 246 inheriting, 2 unlinked (Sub-Co 4 with its own set, and... actually just 1 unlinked since the other two were replaced)
```

### Prompt 9.3 — Distributor Tab: Two-Panel Picker + Detail Cards

```
The Distributor Assignments tab (Section 2B from Prompt 9.1) currently uses an "Add Distributor" button with a search dropdown to assign distributors. Replace this with a two-panel picker layout — similar to the Standalone Companies tab — so the admin can see all available distributors at a glance and pick which ones to assign. Below the picker, the assigned distributor detail cards remain for managing sub-company inheritance.

**New layout for the Distributors tab — two areas stacked vertically:**

**Area 1: Distributor Picker (top half)**

A two-panel assign/unassign layout, same pattern as the Standalone Companies tab:

- Left panel: "Available Distributors" — distributor companies whose devices use this 3-way rule and are NOT currently assigned to this override set. Each row shows:
  - Distributor name
  - Device count (the distributor's own devices)
  - Sub-company count on this 3-way rule (e.g., "248 sub-companies")
  - Warning icon if already assigned to a different override set (same conflict behavior as standalone)

- Right panel: "Assigned Distributors" — distributors currently assigned. Each row shows:
  - Distributor name
  - Sub-company summary (e.g., "245 of 248 inheriting")

- Move buttons between panels (assign/unassign).
- Show counts: "Available: X" / "Assigned: Y"
- Searchable within each panel.

**When a distributor is moved from Available → Assigned:**
Trigger the onboarding flow from Prompt 9.2 (the inheritance choice modal: "Apply to all sub-companies" vs "Assign distributor only", then conflict resolution if needed). After the onboarding completes, the distributor appears in the right panel AND a detail card appears in Area 2 below.

**When a distributor is moved from Assigned → Available (unassigned):**
Show confirmation: "Remove [Distributor Name]? This will stop inheritance for X sub-companies currently inheriting this override set." On confirm, the distributor moves back to the left panel and its detail card is removed from Area 2.

---

**Area 2: Assigned Distributor Detail Cards (bottom half)**

Below the picker, show the expandable detail cards for each assigned distributor — same cards as described in Prompt 9.1 (the "Apply to sub-companies" checkbox, sub-company inheritance table with status/unlink/re-link, search within subs, collapse/expand).

This area only appears if at least one distributor is assigned. If no distributors are assigned, show a subtle empty state: "No distributors assigned. Use the picker above to assign distributors to this override set."

The detail cards are in the same order as the right panel of the picker. Clicking a distributor name in the right panel scrolls to its detail card below.

---

**Visual separation:**
Use a clear divider or section header between Area 1 (picker) and Area 2 (detail cards). Area 2 header: "Distributor Inheritance Management" or "Sub-Company Configuration" with a brief helper text: "Manage how sub-companies inherit this override set from their parent distributor."

**Sample data:**

Available distributors (left panel):
- Global Vending Corp (3 devices, 42 sub-companies)
- Pacific ATM Group (1 device, 18 sub-companies)

Assigned distributors (right panel):
- Miele (245 of 248 inheriting)
- National ATM Services (32 of 34 inheriting)

Detail cards (Area 2):
- Miele: expanded, showing Sub-Co 1–6 inheritance table (same as Prompt 9.1 sample data)
- National ATM Services: collapsed, summary "32 inheriting, 2 unlinked"
```
