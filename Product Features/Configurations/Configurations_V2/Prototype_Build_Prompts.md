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

---
---

# Part 8: Override Set Flexible Scoping (April 7, 2026)

These prompts update the Company Override Set screens to support flexible scoping — override sets are no longer tied to a specific 3-way rule. Instead, each override set defines its own scope using Model, Carrier, and Service Plan selectors where each can be a specific value or "Any" (applies to all). Multiple override sets can be assigned to the same company as long as their parameters don't conflict for overlapping device scopes.

**Context:** Based on client feedback (Meeting 1, March 27, 2026), the vast majority of customer customizations are agnostic to model, carrier, and service plan. Tying override sets to specific 3-way rules forced unnecessary duplication and didn't match real-world usage patterns.

**What changes:**
- Override set creation: scope selectors replace the 3-way rule picker
- Override set browse page: scope columns replace the 3-way rule column
- Override set editor: parameter list source changes based on scope
- Company assignment: conflict detection replaces the old "one per 3-way rule" constraint
- Resolution preview: updated to show multiple matching override sets

Run these prompts AFTER the base Phase 4 and Part 2 prompts are working. These prompts modify existing screens.

---

## Prompt P8.1 — Update Company Override Sets Browse Page (Flexible Scoping)

```
Update the Company Override Sets browse page to reflect the new flexible scoping model. Override sets are no longer tied to a specific 3-way rule — each has its own scope selectors (Model, Carrier, Service Plan) where each can be a specific value or "Any".

**Changes to the table columns:**

Replace the "3-Way Rule" column with three separate scope columns:

| Column | Description |
|--------|-------------|
| Override Set Name | Same as before (e.g., "Cord Financial RMS Access") |
| Model Scope | Shows the specific model name OR "Any" as a distinct badge. Examples: "I-22" or a gray "Any" badge |
| Carrier Scope | Shows the specific carrier name OR "Any" as a distinct badge. Examples: "Verizon" or gray "Any" badge |
| Service Plan Scope | Shows specific plan(s) as comma-separated tags OR "Any" as a badge. Examples: "ATM, Standard" or gray "Any" badge |
| Parameters Overridden | Count (same as before) |
| Companies Assigned | Count with expandable list (same as before) |
| Last Modified | Date (same as before) |

**Visual treatment for "Any" badges:**
- "Any" should appear as a distinct pill/badge in a muted gray or blue-gray color to differentiate it from specific values
- Specific values appear as regular text or colored badges (matching the style used elsewhere for models, carriers, and service plans)
- The visual contrast should make it immediately clear which override sets are broadly scoped vs. narrowly scoped

**Updated filter dropdowns:**
- Model filter: includes "Any" as a filter option in addition to specific models. Selecting "Any" shows only override sets with Model scope = Any. Selecting a specific model shows override sets scoped to that model OR to Any.
- Carrier filter: same pattern — includes "Any" option
- Service Plan filter: same pattern — includes "Any" option
- Add a "Scope breadth" filter: "All scopes" / "Fully scoped (no Any)" / "Partially scoped (some Any)" / "Fully agnostic (all Any)" — this helps admins quickly find broad vs. narrow override sets

**Updated sample data:**

Replace the existing sample data with these examples that demonstrate the range of scoping:

- "Cord Firewall" — Model: Any, Carrier: Any, Service Plan: Any — 3 params — 12 companies
- "Baltech Scheduler" — Model: Any, Carrier: Any, Service Plan: Any — 2 params — 1 company (Baltech)
- "Miele VZW ATM Config" — Model: I-22, Carrier: Verizon, Service Plan: ATM — 15 params — 1 company (Miele)
- "ATM Data Limit" — Model: Any, Carrier: Any, Service Plan: ATM — 4 params — 25 companies
- "I-22 Power Config" — Model: I-22, Carrier: Any, Service Plan: Any — 3 params — 8 companies
- "High Security Firewall (TMO)" — Model: Any, Carrier: T-Mobile, Service Plan: ATM, Standard — 11 params — 4 companies
- "Standard DNS Override" — Model: 4100, Carrier: Verizon, Service Plan: ATM — 3 params — 25 companies

This sample data should make the scoping concept immediately understandable when looking at the browse page.
```

---

## Prompt P8.2 — Update Company Override Set Creation Workflow (Flexible Scoping)

```
Update the "Create Company Override Set" workflow to use flexible scope selectors instead of the 3-way rule picker. The admin now defines what devices this override set applies to using Model, Carrier, and Service Plan selectors — each with an "Any" option.

**Step 1: Define the Override Set (replace existing Step 1 from P2.2)**

A form with:

1. Override Set Name — text input, required.
   - Same as before: descriptive name (e.g., "Cord Financial RMS Access", "Baltech Scheduler")
   - Placeholder text: "Enter a descriptive name for this override set"

2. Scope selectors — three fields that define which devices this override set applies to:

   a) Model scope:
      - A dropdown with two modes, controlled by a toggle or radio button:
        - "Any Model" (default) — applies to all models. When selected, the dropdown is disabled/hidden and shows an "Any" badge.
        - "Specific Model" — when selected, shows a dropdown of models flagged as configurable with parameters selected. Show model name and manufacturer (e.g., "I-22 (CalAmp)").
      - Below the selector, show contextual info:
        - If Any: "This override set will be available for all device models"
        - If specific: "This override set applies to I-22 devices only (142 parameters selected for this model)"

   b) Carrier scope:
      - Same toggle pattern:
        - "Any Carrier" (default) — applies to all carriers. Shows "Any" badge.
        - "Specific Carrier" — dropdown of available carriers (Verizon, T-Mobile, AT&T, Sprint).
      - Below the selector:
        - If Any: "This override set will apply regardless of carrier"
        - If specific: "This override set applies to Verizon devices only"

   c) Service Plan scope:
      - Same toggle pattern:
        - "Any Service Plan" (default) — applies to all service plans. Shows "Any" badge.
        - "Specific Service Plan(s)" — multi-select of available service plans (ATM, Standard, IoT Basic, Enterprise). Selected plans shown as tags/chips.
      - Below the selector:
        - If Any: "This override set will apply regardless of service plan"
        - If specific: "This override set applies to: ATM, Standard"

   **Scope summary line** — after all three selectors, show a combined summary:
   - "Scope: Any Model + Any Carrier + Any Service Plan" (fully agnostic)
   - "Scope: I-22 + Verizon + ATM" (fully scoped)
   - "Scope: Any Model + Any Carrier + ATM, Standard" (partially scoped)
   - Use the same badge styling as the browse page (gray "Any" badges for agnostic dimensions, normal text for specific values)

Validation:
- Name is required.
- At least... actually, no specific scope is required — "Any + Any + Any" is valid. All three defaulting to "Any" is the simplest and most common case.
- Name does not need to be unique system-wide, but warn if a similarly named set already exists.

A "Continue" button to proceed to Step 2.

**Step 2: Set Parameter Overrides (modified parameter list logic)**

Header area (read-only, confirming the selection):
- Override Set Name: "Cord Firewall"
- Scope: Any Model + Any Carrier + Any Service Plan (with badges)
- Parameters available: [count — depends on scope, see below]

**Parameter list source — this is the key change:**

- If Model scope = specific model (e.g., I-22):
  - The parameter list comes from that model's selected parameters, exactly as before.
  - Show parameters grouped by category, same editor as the existing override set editor.
  - Info line: "Showing 142 parameters selected for I-22"

- If Model scope = Any:
  - The parameter list comes from ALL schema parameters where `available_at_company = true`.
  - This is a broader list than any single model's parameters.
  - Show parameters grouped by category, same editor layout.
  - Info line: "Showing all company-level parameters (X parameters with available_at_company = true)"
  - Add a subtle note: "When Model is 'Any', parameters are sourced from the schema. At resolution time, only parameters that belong to the device's actual model will be applied."

The rest of the parameter editor works identically to the existing override set editor:
- Effective baseline column shows the schema default (since there's no single 3-way rule to reference)
- Override value column is the editable input
- Same category grouping, search, filters, "Show only overridden" toggle
- Same input types per data type

**Step 3: Assign Companies (modified conflict detection)**

Header area (read-only):
- Override Set Name: "Cord Firewall"
- Scope: Any Model + Any Carrier + Any Service Plan
- Parameters overridden: 3 (fw_acl, fw_enable, fw_rule_count)

Two-panel company assignment layout (same visual pattern as before):

Left panel — "Available Companies":
- Shows ALL companies (not filtered by 3-way rule anymore, since override sets are no longer scoped to one).
- Each row shows:
  - Company name
  - Device count (total devices in the company)
  - Existing override sets count — a small badge: "3 override sets" showing how many override sets are already assigned to this company
  - Conflict indicator — if assigning this override set to this company would cause a parameter conflict, show a red warning icon with tooltip: "Conflict: fw_acl conflicts with 'Existing Override Set Name'"

- Searchable by company name.
- Multi-select with checkboxes.
- Companies with conflicts should still be selectable, but attempting to save will show the conflict error (see below).

Right panel — "Assigned Companies":
- Companies selected to receive this override set.
- Same display format: company name + device count.
- Multi-select for bulk unassign.

**Conflict detection on assign:**

When the admin moves a company from Available to Assigned (or clicks Save), the system checks:

1. For each company being assigned, find all OTHER override sets already assigned to that company.
2. For each existing override set, check if its scope overlaps with the new override set's scope:
   - "Any" overlaps with everything
   - A specific value overlaps with the same specific value or with "Any"
   - Two different specific values do NOT overlap
3. If scopes overlap, check if the two override sets share any parameters.
4. If both scopes overlap AND parameters overlap → show a conflict error modal:

   **Conflict modal content:**
   "Cannot assign 'Cord Firewall' to Cord Financial"

   "The following parameter conflicts were detected:"

   | Conflicting Parameter | Existing Override Set | Existing Scope | New Override Set | New Scope |
   |---|---|---|---|---|
   | fw_acl | I-22 Power Config | I-22 + Any + Any | Cord Firewall | Any + Any + Any |

   "Two override sets assigned to the same company cannot contain the same parameter when their scopes overlap (could match the same device)."

   "To resolve: remove the conflicting parameter from one of the override sets, or narrow the scopes so they don't overlap."

   Buttons: "OK" (dismiss, do not assign)

5. If no conflicts → assignment proceeds normally.

Buttons:
- "Save & Create" — runs conflict check, saves if no conflicts. Redirect to browse page with success toast.
- "Back" — go back to Step 2 without losing assignments.
- "Cancel" — confirmation if changes were made, then redirect to browse.
```

---

## Prompt P8.3 — Update Company Override Set Editor (Edit Mode — Flexible Scoping)

```
Update the Company Override Set edit view to show the flexible scope selectors instead of the read-only 3-way rule label.

**Header area changes:**

Replace the read-only "Based on 3-Way Rule: I-22 + Verizon + ATM" label with:

Scope display (read-only on edit — scope cannot be changed after creation):
- Model: "I-22" or "Any" (badge)
- Carrier: "Verizon" or "Any" (badge)
- Service Plan: "ATM, Standard" or "Any" (badge)
- Combined scope line with badges, same styling as the browse page and creation workflow

Add a note: "Scope is set at creation and cannot be changed. To change scope, create a new override set."

**Why scope is read-only on edit:**
Changing the scope of an existing override set could invalidate company assignments (e.g., narrowing from "Any" to "I-22" might create conflicts with other override sets). To keep the system predictable, scope is locked at creation. The admin can always create a new override set with a different scope and reassign companies.

**Parameter editor:**
- Same as creation — parameter list source depends on model scope (specific model's parameters vs. schema `available_at_company` parameters)
- No changes to the editor behavior itself

**Company assignments:**
- Same conflict detection as the creation workflow (Prompt P8.2, Step 3)
- When viewing existing assignments, show each company's current override set list in the detail view so the admin can see what else is assigned
```

---

## Prompt P8.4 — Update Resolution Preview for Multiple Override Sets

```
Update the Configuration Resolution Preview page (Phase 5, Prompt 5.1) to handle the new flexible override set scoping. The preview must now show which override sets matched and contributed values.

**Changes to the selector area:**

No structural changes to the selectors — still four dropdowns (Model, Carrier, Service Plan, Company). But the resolution behavior behind the "Resolve" button changes.

**Changes to the results display:**

When a company is selected, the preview now resolves against ALL override sets assigned to that company whose scope matches the selected model/carrier/service plan combination.

**New info section above the results table (when company is selected):**

"Matching Override Sets for [Company Name]:"

Show a list of all override sets assigned to this company that match the selected device profile:

| Override Set | Scope | Parameters Contributed |
|---|---|---|
| Cord Firewall | Any + Any + Any | 3 (fw_acl, fw_enable, fw_rule_count) |
| Baltech Scheduler | Any + Any + Any | 2 (scheduler, sched_time) |
| I-22 Power Config | I-22 + Any + Any | 1 (io_enable) |

Also show override sets assigned to this company that do NOT match (dimmed, with reason):
- "VZW-Only DNS" — Any + Verizon + Any — "Does not match: carrier is T-Mobile" (if resolving for a T-Mobile device)
- "4100 Specific" — 4100 + Any + Any — "Does not match: model is I-22"

This gives the admin full visibility into which override sets are active and which are not for a given device profile.

**Changes to the source badges:**

The "Company Override" badge (orange) now includes a tooltip or expanded detail showing WHICH override set provided the value. For example:

- Source badge: "Company Override" (orange)
- Tooltip: "From: Cord Firewall (Any + Any + Any)"

In the expandable detail per row, the Company Override line now shows:
```
├── Company Override:  "cord_rules"    (orange) ← Cord Firewall [Any + Any + Any]
```

Instead of just:
```
├── Company Override:  "cord_rules"    (orange)
```

**Changes to the summary stats:**

Add a line: "Override sets matched: X of Y assigned" (e.g., "3 of 4 assigned override sets matched this device profile")

**Updated sample resolution for preview:**

When resolving for: I-22 + Verizon + ATM + Cord Financial

Show the matching override sets table, then the parameter table where:
- fw_acl shows source "Company Override" with tooltip "Cord Firewall"
- scheduler shows source "Company Override" with tooltip "Baltech Scheduler"
- io_enable shows source "Company Override" with tooltip "I-22 Power Config"
- Other parameters show their 3-way rule, model default, or schema default sources as before
```

---

## Prompt P8.5 — Update Device Page V2 Preview for Multiple Override Sets

```
Update the Device Page V2 Configuration Preview (Prompt P4.3) and the V2 Configuration section context info to reflect the new flexible override set model.

**Changes to device page context info (P4.1):**

When set to V2 Engine, replace the single "Override Set: Cord Financial RMS Access" line with a list:

- Override Sets (3 active):
  - Cord Firewall (Any + Any + Any) — 3 params
  - Baltech Scheduler (Any + Any + Any) — 2 params
  - I-22 Power Config (I-22 + Any + Any) — 1 param

If no override sets match: "Override Sets: None matching"
If no override sets assigned to company: "Override Sets: None assigned"

**Changes to V2 Configuration Preview (P4.3):**

Same changes as the Resolution Preview (P8.4):

1. Add "Matching Override Sets" info section showing which sets are contributing
2. "Company Override" badges include tooltip showing which specific override set provided the value
3. Expandable detail per row shows the override set name next to the Company Override value
4. Summary stats include "Override sets matched: X of Y"

**Changes to Side-by-Side Comparison (P4.4):**

In the V2 column header, replace "Override Set: Cord Financial RMS Access" with:
- "Override Sets: Cord Firewall, Baltech Scheduler, I-22 Power Config" (or "3 active" with expandable list if space is tight)

**Sample device data updates:**

Update the sample devices (from P4.1) to show the new model:

- Device "ATM-VZW-001" — Legacy, File: VZW_22_01282025.dat, Company: Cord Financial
- Device "ATM-VZW-002" — V2 Engine, Rule: I-22 + Verizon + [ATM, Standard], Override Sets: Cord Firewall (Any+Any+Any), I-22 Power Config (I-22+Any+Any), Company: Cord Financial
- Device "ATM-TMO-003" — V2 Engine, Rule: I-22 + T-Mobile + [ATM], Override Sets: Cord Firewall (Any+Any+Any), Company: Loaded ATMs
- Device "ATM-VZW-004" — V2 Engine, Rule: 4100 + Verizon + [ATM], Override Sets: none, Company: Miele
```

---

## Prompt P8.6 — Update Company Edit Page for Multiple Override Sets

```
Update the Company Edit Page override set assignment section to reflect the new model where a company can have multiple override sets with flexible scoping.

**Replace the current section (which groups override sets by 3-way rule) with:**

A section titled "Configuration Override Sets" showing ALL override sets assigned to this company, grouped by scope breadth:

**Layout:**

Section header: "Configuration Override Sets (X assigned)"

A table/card list showing each assigned override set:

| Override Set Name | Scope | Parameters | Actions |
|---|---|---|---|
| Cord Firewall | Any + Any + Any | 3 (fw_acl, fw_enable, fw_rule_count) | View, Unassign |
| Baltech Scheduler | Any + Any + Any | 2 (scheduler, sched_time) | View, Unassign |
| I-22 Power Config | I-22 + Any + Any | 1 (io_enable) | View, Unassign |

Use the same scope badge styling as the browse page (gray "Any" badges, normal text for specific values).

**"Assign Override Set" button:**
Opens a modal or inline picker showing available override sets that can be assigned to this company without conflicts:

- Shows all existing override sets NOT already assigned to this company
- Each row shows: override set name, scope, parameter count
- Conflict indicator: if assigning would conflict with an existing assignment, show a red warning with tooltip explaining the conflict
- Override sets with conflicts are shown but not selectable (grayed out with conflict reason)
- Multi-select for bulk assignment
- "Assign Selected" button runs conflict detection and assigns

**"Create New Override Set" link:**
Links to the Create Override Set workflow (P8.2) — same as before but now reflects the new scoping model.

**Unassign behavior:**
- "Unassign" button per override set with confirmation: "Remove 'Cord Firewall' from this company? This will affect X devices."
- If the company has sub-companies inheriting this set, show additional warning: "Y sub-companies inherit this override set and will also be affected."

**Sample data for company "Cord Financial":**
- 3 override sets assigned (as shown in the table above)
- 145 devices total
- Available to assign: show 2-3 more override sets, one with a conflict indicator
```

---
---

# Part 9: Resolution Logic Page (April 12, 2026)

These prompts add a "Resolution Logic" page to the Configuration Management section of the prototype. This is an interactive educational/reference page that explains how the configuration engine compiles the final configuration for a device by walking through the 5-level hierarchy. It mirrors the standalone HTML diagram (Resolution_Logic_Diagram.html) but is built into the prototype as a first-class page.

---

## Prompt P9.1 — Add Resolution Logic to Navigation and Landing Page

```
Add a "Resolution Logic" link to the Configuration Management section in the admin navigation menu. Place it as the 5th item after Company Override Sets. It should use an icon that suggests a flowchart, diagram, or logic flow (e.g., a branching/workflow icon or a "git branch" style icon).

On the Configuration Management landing page, add a 5th card-style link for Resolution Logic alongside the existing four cards (Grand Schema, Model Configuration, 3-Way Rules, Company Override Sets). The card should show:
- Icon: flowchart/diagram icon
- Name: "Resolution Logic"
- Description: "Visual walkthrough of how the 5-level hierarchy compiles a device's final configuration"
- Stat: "5 hierarchy levels"

Position this card in a way that makes visual sense — if the current 4 cards are in a 2x2 grid, either make it a 3+2 layout or a full-width featured card below the grid (since this is a reference/educational page rather than a CRUD section, a slightly different treatment is appropriate).
```

---

## Prompt P9.2 — Resolution Logic Page: Layout, Header, and Legend

```
Build the Resolution Logic page. This is an interactive reference page — not a CRUD screen — so style it distinctly from the data management pages.

**Page header:**
- Use a dark background header (dark navy/charcoal gradient, e.g., linear-gradient from #1a1a2e to #16213e) with white text — this sets it apart visually from the standard admin pages.
- Heading: "Configuration Engine — Resolution Logic"
- Subtitle (lighter text, e.g., #a0aec0): "How the system compiles the final configuration for a device by walking through 5 hierarchy levels. Click any step to see details."
- Center-align the header text.

**Color-coded legend bar:**
Below the header, add a horizontal legend bar on a white background with a bottom border. Display the following items in a centered, wrapping flex row with 16px gaps:

Each legend item is a small colored square (14px, 3px border-radius) followed by the label text (13px, 500 weight, #4a5568 color):

- Schema Default — square color: #90a4ae (gray)
- Model Default — square color: #42a5f5 (blue)
- 3-Way Rule — square color: #66bb6a (green)
- Company Override — square color: #ffa726 (orange)
- Device Override — square color: #ab47bc (purple)
- Decision — square color: #fff9c4 with 2px solid #f9a825 border (yellow)
- Final Result — square color: #e8f5e9 with 2px solid #2e7d32 border (green outline)
- Error — square color: #ffebee with 2px solid #c62828 border (red outline)

**Footer instruction line:**
Below the main content area (where the diagram will go), add a centered instruction line on a white background with a top border: "Click any step in the diagram to see detailed information about that stage of the resolution process." (13px, #718096 color)

**Responsive behavior:**
- Header padding reduces on mobile.
- Legend bar items wrap naturally on smaller screens.
- Main content area should allow horizontal scrolling if needed on narrow screens.

For now, leave the main content area between the legend bar and the footer instruction as an empty centered container — the diagram will be added in the next prompt.
```

---

## Prompt P9.3 — Resolution Logic Page: Interactive Flowchart Diagram

```
Add the interactive resolution flowchart to the main content area of the Resolution Logic page (built in P9.2). This is a vertical flowchart that walks through the parameter resolution process step by step.

**Diagram implementation:**
Use Mermaid.js (include via CDN: https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js) to render the flowchart. If your prototype framework doesn't support Mermaid, build an equivalent flowchart using CSS/HTML boxes connected by lines/arrows, or use a React flowchart library — the key requirement is that every node is clickable.

**Flowchart structure (top-down vertical flow):**

Nodes and connections:

START["① Start Resolution — For each parameter in the device's model parameter list"]
  → SCHEMA["② Check Schema Default — Grand Schema default value"]
    → HAS_SCHEMA{③ Schema default exists?}
      -- Yes → SET_SCHEMA["④ Use Schema Default"]
      -- No → NO_DEFAULT["⑤ No value yet — Parameter starts as null"]
    SET_SCHEMA → MODEL
    NO_DEFAULT → MODEL
  MODEL["⑥ Check Model Default — Model-level default for this parameter"]
    → HAS_MODEL{⑦ Model default exists?}
      -- Yes → SET_MODEL["⑧ Override → Model Default"]
      -- No → RULE
    SET_MODEL → RULE
  RULE["⑨ Check 3-Way Rule — Model + Carrier + Service Plan value"]
    → HAS_RULE{⑩ 3-Way Rule value exists?}
      -- Yes → SET_RULE["⑪ Override → 3-Way Rule"]
      -- No → COMPANY
    SET_RULE → COMPANY
  COMPANY["⑫ Check Company Override — Override set assigned to device's company?"]
    → HAS_COMPANY{⑬ Company override value exists?}
      -- Yes → SET_COMPANY["⑭ Override → Company Override"]
      -- No → DEVICE
    SET_COMPANY → DEVICE
  DEVICE["⑮ Check Device Override — Device-level parameter override"]
    → HAS_DEVICE{⑯ Device override exists?}
      -- Yes → SET_DEVICE["⑰ Override → Device Override"]
      -- No → FINAL
    SET_DEVICE → FINAL
  FINAL["⑱ Final Compiled Value — Parameter resolution complete"]
    → NULL_CHECK{Value is null?}
      -- No → VALID["✓ Value Resolved — Move to next parameter"]
      -- Yes → REQUIRED{⑲ Parameter required?}
        -- Yes → MISSING["⑳ CONFIG ERROR — Required parameter has no value"]
        -- No → VALID

**Node color coding (match the legend):**

- START: fill #e8eaf6, stroke #5c6bc0 (indigo — entry point)
- SCHEMA, SET_SCHEMA: fill #eceff1, stroke #90a4ae (gray — schema level)
- NO_DEFAULT: fill #fafafa, stroke #bdbdbd (light gray — null state)
- MODEL, SET_MODEL: fill #e3f2fd, stroke #42a5f5 (blue — model level)
- RULE, SET_RULE: fill #e8f5e9, stroke #66bb6a (green — 3-way rule level)
- COMPANY, SET_COMPANY: fill #fff3e0, stroke #ffa726 (orange — company level)
- DEVICE, SET_DEVICE: fill #f3e5f5, stroke #ab47bc (purple — device level)
- FINAL, VALID: fill #e8f5e9, stroke #2e7d32 (green result)
- MISSING: fill #ffebee, stroke #c62828 (red error)
- All decision nodes (HAS_SCHEMA, HAS_MODEL, HAS_RULE, HAS_COMPANY, HAS_DEVICE, REQUIRED, NULL_CHECK): fill #fff9c4, stroke #f9a825 (yellow)

**All nodes must be clickable.** On click, they should trigger a flyout panel (built in the next prompt). Use 2px stroke width on all nodes. Use "basis" curve style for connections. Center the diagram on the page with padding (40px top/bottom, 24px sides). Cap the diagram max-width at ~1000px.

**Node hover behavior:** On hover, apply a slight brightness reduction (e.g., filter: brightness(0.92)) to indicate interactivity.
```

---

## Prompt P9.4 — Resolution Logic Page: Flyout Detail Panel

```
Add a slide-out flyout panel to the Resolution Logic page. When any node in the flowchart diagram is clicked, a detail panel slides in from the right side of the screen with rich information about that step.

**Flyout panel structure:**

- Width: 440px on desktop, 100% on mobile (screens ≤ 600px)
- Position: fixed, right side, full viewport height
- Background: white
- Shadow: -4px 0 24px rgba(0,0,0,0.15)
- Slide-in animation: translateX(100%) → translateX(0), 300ms cubic-bezier(0.4, 0, 0.2, 1)
- Z-index: above the diagram (200)

**Backdrop:**
A semi-transparent overlay (rgba(0,0,0,0.3)) behind the flyout covering the full viewport. Clicking the backdrop closes the flyout. Pressing Escape also closes it. Fade in: opacity 0→1, 250ms ease.

**Flyout layout:**

Header area (with bottom border):
- Title (h2, 20px, 700 weight, #1a1a2e) — the step name (e.g., "② Check Schema Default")
- Below the title: a color-coded badge (11px, 700 weight, uppercase, pill-shaped with 12px border-radius and 3px/10px padding). Badge colors match the node's layer.
- Close button (×) — top right, 28px font size, #718096 color, hover #1a1a2e

Body area (scrollable, flex: 1):
- Padding: 24px
- Font: 14px, line-height 1.7, color #2d3748
- Section headers (h3): 14px, 700 weight, #1a1a2e, uppercase, 0.5px letter-spacing, 16px top margin
- Bullet lists: 20px padding-left, 4px bottom margin per item
- Code spans: #edf2f7 background, 2px/6px padding, 3px border-radius, 13px font, #e53e3e color
- Example boxes: #f7fafc background, 1px solid #e2e8f0 border, 6px border-radius, 14px padding, monospace font, 13px, pre-wrap white-space
- Source badges (inline): 11px, 600 weight, 2px/8px padding, 10px border-radius. Colors:
  - Schema: bg #eceff1, color #546e7a
  - Model: bg #e3f2fd, color #1565c0
  - 3-Way Rule: bg #e8f5e9, color #2e7d32
  - Company Override: bg #fff3e0, color #e65100
  - Device Override: bg #f3e5f5, color #7b1fa2
  - Error: bg #ffebee, color #c62828
  - Result: bg #e8f5e9, color #1b5e20

**Flyout content for each node:**

When a node is clicked, populate the flyout with the following content (match node ID to content):

---

**① Start Resolution** — Badge: "Entry Point" (bg #e8eaf6, color #1a237e)

[h3] What Happens
The resolution engine iterates through every parameter in the device's model parameter list and resolves each one individually by walking down the 5-level hierarchy.

[h3] Inputs Required
- The device record — identifies the model, carrier, service plan, and company
- The model's selected parameter list — determines which of the ~697 schema parameters are relevant for this device

[h3] How It Works
For each parameter, the system starts with no value and checks each hierarchy level in order. The first level to provide a value becomes the baseline; each subsequent level can override it. The last level to set a value wins.

[example box]
For each parameter in device.model.selectedParameters:
  → Walk levels 1-5
  → Record final value + source layer
  → Flag errors if required and null

---

**② Check Schema Default** — Badge: "Level 1 — Schema" (schema colors)

[h3] What Happens
Look up the parameter in the Grand Schema and check if it has a default value defined.

[h3] The Grand Schema
The schema is the single source of truth for all ~697 configuration parameters. Each parameter entry includes metadata (name, type, category, validation) and an optional default_value.

[h3] Key Points
- Schema defaults apply system-wide as the baseline for all devices, all models, all carriers
- Not every parameter has a schema default — some are intentionally null, meaning they must be set at a lower level
- This level replaces the concept of a "global configuration layer" from V1

[example box]
wan0_proto → schema default: "dhcp"
wl0_ssid   → schema default: "WATM-Default"
fw_table_1 → schema default: "" (empty string)
cell_apn   → schema default: null (no default)

---

**③ Schema Default Exists?** — Badge: "Decision" (bg #fff9c4, color #f57f17)

[h3] Decision
Does the Grand Schema have a non-null default_value for this parameter?

[h3] Outcomes
- Yes → Use the schema default as the starting value. Most parameters (~400+) will have a schema default.
- No → The parameter starts with no value. It must be set at a lower level or it will remain null.

---

**④ Use Schema Default** — Badge: "Level 1 — Schema" (schema colors)

[h3] What Happens
The parameter's current resolved value is set to the schema default. The source is recorded as [Schema Default badge].

[h3] In the Preview
Parameters at this level appear with a gray badge in the configuration preview. This is the most common source — the bulk of a device's configuration comes from schema defaults.

---

**⑤ No Value Yet** — Badge: "Null" (bg #fafafa, color #757575, 1px solid #bdbdbd border)

[h3] What Happens
The parameter has no schema default. Its value is currently null. The system continues checking lower levels to see if any of them provide a value.

[h3] Why Some Parameters Have No Default
- Parameters like cell_apn are carrier-specific — there's no universal default
- Parameters like wl0_password are security-sensitive — no default makes sense
- Custom parameters that are always set per-model or per-rule

---

**⑥ Check Model Default** — Badge: "Level 2 — Model" (model colors)

[h3] What Happens
Look up the parameter in the Model Configuration for this device's model and check if a model-level default is set.

[h3] Why Model Defaults Exist
Some parameters are model-specific but constant across all carrier/service plan combinations. Setting them here avoids repeating the same value in every 3-Way Rule.

[example box]
I-22 model defaults:
  io_enable = 1      (always on for I-22)
  io_port_count = 4  (hardware constant)
  mon_heartbeat = 30 (model-specific interval)

[h3] Key Points
- Only parameters in the model's selected parameter list can have model defaults
- Model defaults override schema defaults
- Not every selected parameter needs a model default — only set values that should be constant for this model

---

**⑦ Model Default Exists?** — Badge: "Decision" (decision colors)

[h3] Decision
Does the Model Configuration for this device's model have a default value set for this parameter?

[h3] Outcomes
- Yes → Override the current value with the model default. Typically ~50 parameters per model have model defaults.
- No → Keep the current value (schema default or null) and continue to the next level.

---

**⑧ Override → Model Default** — Badge: "Level 2 — Model" (model colors)

[h3] What Happens
The parameter's resolved value is overridden with the model default. The source is updated to [Model Default badge].

This value will be used unless a 3-Way Rule, Company Override, or Device Override sets a different value at a lower level.

---

**⑨ Check 3-Way Rule** — Badge: "Level 3 — 3-Way Rule" (rule colors)

[h3] What Happens
Look up the 3-Way Rule that matches this device's Model + Carrier + Service Plan combination and check if it has a value for this parameter.

[h3] How the Rule Is Found
The system matches the device's model, carrier, and service plan against existing 3-Way Rules. A rule can cover multiple service plans (e.g., I-22 + Verizon + [ATM, Standard]).

[example box]
Device: ATM-VZW-001
  Model: I-22
  Carrier: Verizon
  Service Plan: ATM
  → Matches rule: I-22 + Verizon + [ATM, Standard]

[h3] Key Points
- This is where carrier-specific and service-plan-specific values are set
- Only parameters in the model's selected parameter list appear in 3-Way Rules
- Typically ~40 parameters are configured per 3-Way Rule

---

**⑩ 3-Way Rule Value Exists?** — Badge: "Decision" (decision colors)

[h3] Decision
Does the matched 3-Way Rule have a value set for this parameter?

[h3] Outcomes
- Yes → Override the current value with the 3-Way Rule value. This means the parameter is carrier/service-plan specific.
- No → Keep the current value (model default, schema default, or null) and continue.

---

**⑪ Override → 3-Way Rule** — Badge: "Level 3 — 3-Way Rule" (rule colors)

[h3] What Happens
The parameter's resolved value is overridden with the 3-Way Rule value. The source is updated to [3-Way Rule badge].

[example box]
wan0_proto:
  Schema Default: "dhcp"
  Model Default:  —
  3-Way Rule:     "static"  ← overrides schema default
  → Current value: "static" (source: 3-Way Rule)

---

**⑫ Check Company Override** — Badge: "Level 4 — Company Override" (company colors)

[h3] What Happens
Find all override sets assigned to the device's company (or inherited from its parent company) whose scope matches this device, and check if any of them have a value for this parameter.

[h3] Override Set Matching
Each override set defines its own scope using three selectors — Model, Carrier, and Service Plan — each of which can be a specific value or "Any". The system finds all sets where:
1. Override set model = device's model OR "Any"
2. Override set carrier = device's carrier OR "Any"
3. Override set service plan includes device's service plan OR "Any"

[example box]
Device: ATM-VZW-001 (I-22, Verizon, ATM)
  Company: Cord Financial

  Matching override sets:
    "Cord Firewall"     — Any + Any + Any    → fw_acl, fw_enable
    "I-22 Power Config" — I-22 + Any + Any   → io_enable
  Non-matching:
    "TMO DNS Config"    — Any + T-Mobile + Any  (wrong carrier)

[h3] Key Points
- Multiple override sets can match one device, but conflict detection at assignment time guarantees no two sets provide the same parameter for overlapping scopes
- For any given parameter, at most one override set will have a value — there is no ambiguity
- Override sets are reusable — the same set can be assigned to multiple companies
- Sub-companies inherit from parent by default but can be unlinked

---

**⑬ Company Override Value Exists?** — Badge: "Decision" (decision colors)

[h3] Decision
Do any of the matching override sets (those whose scope matches this device's model, carrier, and service plan) have a value for this parameter?

[h3] Outcomes
- Yes → Exactly one matching override set has a value for this parameter (guaranteed by conflict detection). Override the current value with it.
- No → No matching override set provides this parameter. Keep the current value and continue to the device level. This is the case for most parameters — company overrides are selective.

---

**⑭ Override → Company Override** — Badge: "Level 4 — Company Override" (company colors)

[h3] What Happens
The parameter's resolved value is overridden with the value from the matching override set. The source is updated to [Company Override badge]. Because conflict detection prevents parameter overlap across override sets for the same device scope, this value is unambiguous — exactly one set provides it.

[example box]
fw_acl:
  Schema Default:    null
  3-Way Rule:        "default_rules"
  Company Override:  "cord_rules"  ← from "Cord Firewall" (Any+Any+Any)
  → Current value: "cord_rules" (source: Company Override)

---

**⑮ Check Device Override** — Badge: "Level 5 — Device Override" (device colors)

[h3] What Happens
Check if the individual device has an override value for this parameter. Device overrides are the final and highest-priority level.

[h3] Key Points
- Only parameters with available_at_device = true in the schema can be overridden here
- Limited scope — primarily customer-facing settings (Wi-Fi SSID, password, cellular settings)
- Customers can modify these themselves through the portal
- Most parameters (~690 of 697) are NOT available at this level

---

**⑯ Device Override Exists?** — Badge: "Decision" (decision colors)

[h3] Decision
Does the device have an override value for this parameter?

[h3] Outcomes
- Yes → Override the current value with the device-specific value. This is the final override — nothing can override it. Typically ~5 parameters per device.
- No → Keep the current value from whichever level last set it. Proceed to final compilation.

---

**⑰ Override → Device Override** — Badge: "Level 5 — Device Override" (device colors)

[h3] What Happens
The parameter's resolved value is overridden with the device-specific value. The source is updated to [Device Override badge]. This is the highest priority — no other level can override it.

[example box]
wl0_ssid:
  Schema Default:    "WATM-Default"
  3-Way Rule:        "WATM-ATM"
  Company Override:  "CordFinancial"
  Device Override:   "CordATM-FL01"  ← device-specific
  → Final value: "CordATM-FL01" (source: Device Override)

---

**⑱ Final Compiled Value** — Badge: "Resolution Complete" (result colors)

[h3] What Happens
All 5 hierarchy levels have been checked. The parameter now has its final resolved value (or null if no level provided a value).

[h3] What's Recorded
- Final value — the compiled result to push to the device
- Source layer — which level provided the value (for the preview badge color)
- Full resolution chain — every layer's value for audit/debugging

[h3] Typical Distribution (per device)

[example box]
~400 parameters → Schema Default  (gray)
~50  parameters → Model Default   (blue)
~40  parameters → 3-Way Rule      (green)
~8   parameters → Company Override (orange)
~5   parameters → Device Override  (purple)
~2   parameters → Missing/Error   (red)

---

**Value is null?** — Badge: "Decision" (decision colors)

[h3] Decision
After checking all 5 levels, is the parameter's final value still null (no level provided a value)?

[h3] Outcomes
- No (has a value) → Resolution is complete. The parameter is valid.
- Yes (still null) → Check if this parameter is marked as required in the schema. A null value may or may not be an error.

---

**⑲ Parameter Required?** — Badge: "Decision" (decision colors)

[h3] Decision
Is this parameter marked as is_required = true in the Grand Schema?

[h3] Outcomes
- Yes → This is a configuration error. A required parameter has no value at any level. Flagged as [MISSING badge] in the preview.
- No → A null value is acceptable for optional parameters. The parameter is simply not set — this is valid.

---

**⑳ CONFIG ERROR** — Badge: "Error" (error colors)

[h3] What Happens
A required parameter has no value after checking all 5 hierarchy levels. This is flagged as a configuration error.

[h3] How It's Displayed
In the configuration preview, this parameter shows a red [MISSING badge]. It's included in the "Missing (required)" count in the summary stats.

[h3] When This Happens
- A new required parameter was added to the schema but hasn't been configured in any 3-Way Rule yet
- A model was configured for a new carrier/service plan combination but required values weren't filled in
- A parameter was made required after existing rules were created

[h3] Resolution
An admin needs to set a value at any level: schema default, model default, 3-Way Rule, company override, or device override. The system warns but does not block saving — the error can be caught during review.

---

**✓ Value Resolved** — Badge: "Complete" (result colors)

[h3] What Happens
This parameter's resolution is complete. The system moves to the next parameter in the model's selected parameter list and repeats the entire 5-level resolution process.

[h3] After All Parameters Are Resolved
The complete compiled configuration is assembled — all ~697 parameters with their final values and source layers. This is what gets:
- Displayed in the Configuration Preview (with color-coded source badges)
- Exported as a key-value file (for comparison with legacy .DAT files)
- Pushed to the device as its active configuration
```

---
---

# Part 10: Config Push, Delivery & Verification (April 13, 2026)

These prompts add config push and delivery visibility to the prototype. This covers two main capabilities:

1. **On-demand push** — admins can manually select devices and trigger an immediate config push from the portal, without waiting for the next device check-in.
2. **Push status tracking and reporting** — per-device push status records showing whether a config push succeeded or failed, and whether the device verified the config via hostname match on its next check-in. Includes both device-level and fleet-level visibility.

**Context:** The delivery model is push-based — the portal always pushes configs to devices via RouterApi. There are two triggers: (a) check-in triggered push (existing — device checks in, hostname mismatch detected, push queued automatically), and (b) on-demand manual push (new — admin triggers immediate push from the portal). Verification is hostname-based — after a push, the system waits for the device's next check-in to confirm the hostname updated, which means the config was applied.

Run these prompts AFTER all previous parts are working.

---

## Prompt P10.1 — Add Config Push to Admin Navigation and Landing Page

```
Add "Config Push & Status" as a new item in the Configuration Management admin navigation menu. Place it as the 6th item after Resolution Logic. Use an icon that suggests deployment or sending (e.g., a rocket, upload, or paper plane icon).

On the Configuration Management landing page, add a 6th card-style link. The card should show:
- Icon: deployment/push icon
- Name: "Config Push & Status"
- Description: "Push configurations to devices and monitor deployment status across your fleet"
- Stat: "1,520 devices tracked" (hardcoded placeholder)

Position this card in the grid alongside the existing cards. This is an operational dashboard, not a configuration builder — the card design should still match the others but the description signals it's about delivery and monitoring rather than building configs.
```

---

## Prompt P10.2 — Config Push Dashboard: Fleet Overview

```
Build the Config Push & Status dashboard page. This is the primary operational view for monitoring config deployment across the fleet.

**Page header:**
- Heading: "Config Push & Status"
- Subtitle: "Monitor configuration deployment status and push configurations to devices on demand."

**Summary stats bar (top of page):**

A row of stat cards showing fleet-wide push status counts. Each card shows a count, a label, and a percentage of total. Use color coding to make status immediately scannable:

- Verified (green) — 1,247 — 82% — "Push succeeded, hostname confirmed on check-in"
- Pushed, Pending (blue) — 142 — 9% — "Push succeeded, awaiting next check-in for hostname verification"
- Failed (red) — 47 — 3% — "Push failed — device unreachable, API disabled, or upload error"
- Mismatch (orange/amber) — 31 — 2% — "Push succeeded but hostname didn't update on check-in"
- Retry Queued (gray) — 53 — 4% — "Previous push failed, will retry on next check-in"

Total: 1,520 devices

**Action buttons bar (below stats):**

- "Push to Selected" — disabled by default, enabled when devices are selected in the table below. Primary/blue button.
- "Retry All Failed" — secondary button. Tooltip: "Queue a retry push for all devices with Failed status."
- "Retry All Mismatched" — secondary button. Tooltip: "Queue a retry push for all devices where hostname didn't match."

**Device status table:**

A paginated, filterable table showing per-device push status. Columns:

| Column | Description |
|--------|-------------|
| Checkbox | Multi-select for bulk actions (push, retry) |
| Device Name | Device identifier (e.g., "ATM-VZW-001"). Clickable — links to device page. |
| Company | Company the device belongs to |
| Model | Device model (e.g., I-22, 4100) |
| Carrier | Verizon, T-Mobile, AT&T |
| Config Source | "Legacy" or "V2 Engine" badge |
| Push Status | Color-coded status badge: Verified (green), Pushed (blue), Failed (red), Mismatch (orange), Retry Queued (gray), No Push (light gray for devices never pushed) |
| Last Push | Timestamp of last push attempt (e.g., "Apr 13, 2026 2:34 PM") |
| Push Trigger | How the last push was triggered: "Check-in" / "On-demand" / "Retry" |
| Push Result | "Success" or failure reason: "Unreachable" / "API Disabled" / "Upload Error" |
| Hostname Status | "Verified" (with timestamp) / "Pending" / "Mismatch" / "—" (no push yet) |

**Filters:**

- Status filter dropdown: All / Verified / Pushed (Pending) / Failed / Mismatch / Retry Queued / No Push
- Config source filter: All / Legacy / V2 Engine
- Company filter: dropdown with search
- Model filter: dropdown
- Carrier filter: dropdown
- Search bar: search by device name

**Pagination:** 25 per page with page controls.

**Sample data — populate at least 15-20 devices showing a mix of statuses:**

- 8 devices: Verified — various companies, models, carriers, push timestamps from past week, hostname verified
- 3 devices: Pushed, Pending — pushed today, awaiting check-in
- 2 devices: Failed — one "Unreachable", one "API Disabled"
- 2 devices: Mismatch — push succeeded but hostname didn't update
- 2 devices: Retry Queued — previous failure, queued for retry
- 3 devices: No Push — V2 Engine but never pushed yet (newly configured)

Use realistic device names (ATM-VZW-001 through ATM-VZW-020), a mix of companies (Cord Financial, Miele, Baltech, National ATM Services), models (I-22, 4100, 4500), and carriers (Verizon, T-Mobile, AT&T).
```

---

## Prompt P10.3 — On-Demand Push: Selection and Confirmation Flow

```
Implement the on-demand push flow from the Config Push & Status dashboard. This allows an admin to select specific devices and trigger an immediate config push.

**Selection behavior:**

- Checkboxes in the first column of the device table allow multi-select
- A "Select all on this page" checkbox in the header row
- When any devices are selected, the "Push to Selected" button enables and shows a count: "Push to Selected (5)"
- Selected row count shown above table: "5 of 1,520 devices selected"

**Push to Selected button click → Confirmation modal:**

When the admin clicks "Push to Selected", show a confirmation modal before executing:

**Modal: "Push Configuration to X Devices"**

Header: "Push Configuration to 5 Devices"

Body — summary of what will happen:
- "You are about to push the current compiled configuration to the following devices:"
- A compact scrollable list (max height ~200px) showing selected devices:

  | Device | Company | Model | Carrier | Current Status |
  |--------|---------|-------|---------|---------------|
  | ATM-VZW-001 | Cord Financial | I-22 | Verizon | Verified |
  | ATM-VZW-005 | Miele | I-22 | Verizon | Failed |
  | ATM-TMO-008 | Baltech | 4100 | T-Mobile | No Push |
  | ATM-VZW-012 | Cord Financial | I-22 | Verizon | Mismatch |
  | ATM-ATT-015 | National ATM | 4500 | AT&T | Retry Queued |

- Below the list, show a warning if any devices are on Legacy config: "Note: 1 device is using Legacy configuration. Push will use the legacy .DAT file, not the V2 engine."
- If all devices are V2: "All selected devices use the V2 configuration engine."

Buttons:
- "Push Now" — primary/blue button. Triggers the push (see below).
- "Cancel" — secondary button. Closes modal, returns to dashboard.

**After "Push Now" is clicked:**

1. The modal shows a brief progress state: "Pushing configuration to 5 devices..." with a spinner.
2. After a simulated delay (1-2 seconds), show results in the same modal:

   **Modal: "Push Results"**

   | Device | Result |
   |--------|--------|
   | ATM-VZW-001 | Success — hostname pending |
   | ATM-VZW-005 | Success — hostname pending |
   | ATM-TMO-008 | Failed: Unreachable |
   | ATM-VZW-012 | Success — hostname pending |
   | ATM-ATT-015 | Failed: API Disabled |

   Summary: "3 of 5 pushes succeeded. 2 failed and have been queued for retry on next check-in."

   Button: "Close" — closes modal and refreshes the dashboard table to show updated statuses.

3. Back on the dashboard, the pushed devices now show:
   - Successfully pushed devices: status changes to "Pushed" (blue badge), push trigger shows "On-demand", push result shows "Success", hostname status shows "Pending"
   - Failed devices: status changes to "Retry Queued" (gray badge), push result shows the failure reason
```

---

## Prompt P10.4 — Device Page: Push Status Section

```
Add a "Configuration Push Status" section to the device detail page. This shows the push history and current status for this specific device. Place it below the existing V2 Configuration Preview section (or below the configuration source selector area).

**Section: "Configuration Push Status"**

**Current status card (prominent, at top of section):**

A card showing the device's current push state:

- Status badge (large, color-coded): "Verified" (green) / "Pushed, Pending" (blue) / "Failed" (red) / "Mismatch" (orange) / "Retry Queued" (gray) / "No Push" (light gray)
- Config version: "v2.3 — compiled Apr 13, 2026"
- Expected hostname: "VZW22_04132026"
- Last push: "Apr 13, 2026 2:34 PM via On-demand"
- Push result: "Success"
- Hostname verification: "Verified — Apr 13, 2026 3:12 PM" or "Pending — awaiting next check-in" or "Mismatch — device reported VZW22_03272026"

**Action button:**
- "Push Configuration Now" button — triggers an on-demand push for this single device. Same confirmation pattern as the dashboard but simplified for one device:
  - Confirm: "Push the current configuration to ATM-VZW-002?"
  - On confirm: show spinner, then result (Success or Failed with reason)
  - On success: status updates to "Pushed, Pending" and push history gets a new entry

**Push history table (below the current status card):**

A chronological list of all push attempts for this device, most recent first:

| Timestamp | Config Version | Trigger | Push Result | Hostname Status | Verified At |
|-----------|---------------|---------|-------------|----------------|-------------|
| Apr 13, 2026 2:34 PM | v2.3 | On-demand | Success | Verified | Apr 13, 2026 3:12 PM |
| Apr 10, 2026 8:15 AM | v2.2 | Check-in | Success | Verified | Apr 10, 2026 9:01 AM |
| Apr 7, 2026 11:42 PM | v2.1 | Check-in | Failed: Unreachable | — | — |
| Apr 7, 2026 6:30 PM | v2.1 | On-demand | Failed: Unreachable | — | — |
| Mar 28, 2026 3:15 PM | v2.0 | Check-in | Success | Verified | Mar 28, 2026 4:02 PM |

Show a maximum of 10 entries with a "Show all" link to expand.

**Sample data for different devices:**

- Device ATM-VZW-002 (V2 Engine): Show as Verified, with 5 push history entries (mix of on-demand and check-in triggers, one failure in the history)
- Device ATM-TMO-008 (V2 Engine): Show as Failed, last push attempt was unreachable, 2 entries in history
- Device ATM-VZW-001 (Legacy): Show as Verified with a note: "This device uses Legacy configuration. Push status tracks legacy .DAT file delivery."
```

---

## Prompt P10.5 — Config Push Dashboard: Push by Config Change View

```
Add a second view mode to the Config Push & Status dashboard. The default view (from P10.2) is device-centric — showing every device and its status. This new view is config-change-centric — grouped by config change events, showing which changes have been fully rolled out and which still have pending/failed devices.

**View toggle:**
Add a toggle at the top of the dashboard (below the header, above the stats):
- "By Device" (default, current view from P10.2)
- "By Config Change" (new view)

The summary stats bar stays the same in both views.

**"By Config Change" view:**

A list of recent configuration changes, each shown as an expandable card. Most recent first.

Each card shows:

**Card header (always visible):**
- Config change description: "3-Way Rule updated: I-22 + Verizon + [ATM, Standard]" or "Override Set updated: Cord Firewall" or "Model Defaults updated: I-22"
- Changed by: "Aksana R." with timestamp: "Apr 13, 2026 1:15 PM"
- Affected devices count: "Affects 342 devices"
- Rollout progress bar: a horizontal bar showing verified (green) / pushed (blue) / failed (red) / mismatch (orange) / retry (gray) proportions
- Rollout summary: "312 verified, 18 pending, 8 failed, 4 mismatch"

**Card expanded (click to expand):**
- A table of affected devices (same columns as the device-centric view but filtered to only devices affected by this config change)
- "Push to All Remaining" button — pushes to all devices that haven't been verified yet (pending + failed + mismatch)
- "Push to Selected" — same multi-select behavior as the device view

**Sample data — 4-5 config change cards:**

1. "3-Way Rule updated: I-22 + Verizon + [ATM, Standard]" — Apr 13, 2026 — 342 devices — 91% verified, 5% pending, 3% failed, 1% mismatch
2. "Override Set updated: Cord Firewall" — Apr 12, 2026 — 145 devices — 100% verified (fully rolled out)
3. "Model Defaults updated: I-22" — Apr 10, 2026 — 520 devices — 97% verified, 1% pending, 2% failed
4. "3-Way Rule created: 4100 + T-Mobile + [ATM]" — Apr 8, 2026 — 28 devices — 75% verified, 25% no push (new rule, devices awaiting first push)
5. "Override Set updated: ATM Data Limit" — Apr 5, 2026 — 890 devices — 99% verified, 1% retry queued
```

---

## Prompt P10.6 — Device Page: Update V2 Context Info with Push Status Badge

```
Update the device page V2 Configuration section (from P4.1 / P8.5) to include a push status badge in the context info area. This gives the admin immediate visibility into whether this device's config is current without scrolling to the push status section.

**Changes to the V2 Engine context info block:**

Add a "Push Status" line to the existing context info, placed after the Override Sets line:

Current context info (from P8.5):
- Model: I-22
- Carrier: Verizon
- Service Plan: ATM, Standard
- 3-Way Rule: I-22 + Verizon + [ATM, Standard]
- Override Sets (3 active): Cord Firewall, Baltech Scheduler, I-22 Power Config
- Config Source: V2 Engine

Add:
- Push Status: [color-coded badge] with detail text

Examples:
- Push Status: [Verified ✓] (green badge) — "Pushed Apr 13, hostname confirmed Apr 13 3:12 PM"
- Push Status: [Pending] (blue badge) — "Pushed Apr 13 2:34 PM, awaiting check-in"
- Push Status: [Failed] (red badge) — "Last push Apr 13 failed: Unreachable. Retry queued."
- Push Status: [Mismatch ⚠] (orange badge) — "Push succeeded but hostname didn't match on check-in. Expected: VZW22_04132026, Reported: VZW22_03272026"
- Push Status: [No Push] (gray badge) — "Configuration has not been pushed to this device yet"

The badge should be clickable — clicking it scrolls to or opens the Configuration Push Status section (from P10.4) for full details.

**For Legacy devices:**
- Push Status: [Verified ✓] (green badge) — "Legacy .DAT pushed Mar 28, hostname confirmed"
- Or [No Status] (gray badge) — "Legacy configuration — push tracking not available" (for devices that haven't been pushed since tracking was added)

**Update sample devices to show the badge:**
- ATM-VZW-002: Verified (green)
- ATM-TMO-003: Pending (blue)
- ATM-TMO-008: Failed (red)
- ATM-VZW-004: No Push (gray)
```

---
---

# Part 11: Versioning and Approval Workflow (April 13, 2026)

These prompts add a draft/publish/approval workflow to 3-Way Rules and Company Override Sets. Changes to these entities are saved as drafts, reviewed by a second admin, and only go live after approval. This protects against accidental or unreviewed changes propagating to hundreds or thousands of devices.

**Context:** 3-Way Rules and Override Sets have the widest blast radius in the configuration system — a single change can affect every device for a model + carrier + service plan combination, or every company assigned to an override set. The versioning layer adds a safety gate: changes are saved as a draft, previewed with a diff against the current published version, and approved by a different admin before becoming the new published version. Schema defaults, model defaults, and device overrides are NOT versioned (lower blast radius).

Run these prompts AFTER all previous parts are working.

---

## Prompt P11.1 — 3-Way Rule Editor: Draft Save and Status Indicator

```
Update the 3-Way Rule editor page to support a draft/publish workflow. Instead of changes going live immediately on save, they are saved as a draft that must be approved by a different admin.

**Changes to the save behavior:**

Replace the current "Save" button with draft-aware buttons:

- When NO draft exists (viewing the published version):
  - The editor is read-only by default. Parameter values are shown but not editable.
  - "Edit" button in the header/toolbar — clicking it enters edit mode, which creates a new draft.
  - "Preview Configuration" button — same as before, shows the published config.

- When in edit mode (creating/editing a draft):
  - Parameter values become editable (same editor as before).
  - "Save Draft" button (primary) — saves the current changes as a draft. The published version remains unchanged. Success toast: "Draft saved. Pending approval."
  - "Discard Draft" button (secondary/destructive) — discards all draft changes and returns to the published version. Confirmation dialog: "Discard all draft changes? The published version will remain unchanged."
  - "Preview Draft" button — opens the same configuration preview but using draft values instead of published values.
  - "Compare Draft vs. Published" button — opens the diff view (see P11.3).

- When a draft already exists (returning to a rule that has a pending draft):
  - Show a prominent banner at the top of the editor:
    ┌──────────────────────────────────────────────────────────────────┐
    │ ⚠ Draft pending approval                                        │
    │ Draft created by Adam Curcie on Apr 13, 2026 at 2:15 PM         │
    │ 5 parameters changed · Affects 342 devices                      │
    │                                                                  │
    │ [Review Draft]  [Edit Draft]  [Discard Draft]                    │
    └──────────────────────────────────────────────────────────────────┘
  - "Review Draft" — opens the diff/approval view (P11.3)
  - "Edit Draft" — enters edit mode with the draft values loaded (only available to the draft creator or other non-approver admins who want to modify it before approval)
  - "Discard Draft" — discards the draft (with confirmation)
  - Below the banner, the published version is displayed read-only so the admin can see what's currently live.

**Status indicator in the header:**

Add a version status badge next to the rule title:
- "Published" (green badge) — no pending draft, this is the current live version
- "Draft Pending" (amber/yellow badge) — a draft exists and is awaiting approval
- "New (Unsaved)" — when creating a brand new 3-way rule that has never been published

**One draft at a time constraint:**
- Only one draft can exist per 3-way rule at any time.
- If a draft already exists, the "Edit" button opens the existing draft rather than creating a new one.
- If another admin's draft is pending, show a notice: "A draft by [Name] is pending approval. You can review it, edit it, or discard it before creating a new draft."

**New 3-Way Rule creation:**
- When creating a brand new 3-way rule (from the "Add 3-Way Rule" workflow), the first save creates the rule as a draft — it does NOT go live immediately.
- The draft must be approved before the rule becomes published and starts affecting devices.
- Show a notice during creation: "This new rule will be saved as a draft and require approval before it goes live."
```

---

## Prompt P11.2 — Company Override Set Editor: Draft Save and Status Indicator

```
Apply the same draft/publish workflow to the Company Override Set editor page. The pattern is identical to the 3-Way Rule editor (P11.1) — changes are saved as drafts, must be approved, and the published version stays live until approval.

**Changes to the save behavior:**

Same button structure as P11.1:
- Read-only published view with "Edit" button when no draft exists
- "Save Draft", "Discard Draft", "Preview Draft", "Compare Draft vs. Published" when editing
- Draft pending banner when a draft exists

**Draft pending banner (same pattern, override-set-specific content):**

  ┌──────────────────────────────────────────────────────────────────┐
  │ ⚠ Draft pending approval                                        │
  │ Draft created by Devon D'Andrea on Apr 13, 2026 at 3:45 PM      │
  │ 3 parameters changed · 2 companies added · Affects 180 devices   │
  │                                                                  │
  │ [Review Draft]  [Edit Draft]  [Discard Draft]                    │
  └──────────────────────────────────────────────────────────────────┘

**Important — what gets versioned in an override set draft:**
- Parameter override value changes (same as 3-way rules)
- Company assignment changes (adding/removing companies, distributor inheritance changes)
- Both parameter changes AND assignment changes are part of the same draft — they are reviewed and approved together as one unit

This means if an admin changes two parameter values AND assigns three new companies, the entire set of changes is one draft. The approver sees all changes together.

**Status indicator:** Same as P11.1 — "Published" (green), "Draft Pending" (amber), "New (Unsaved)" for new override sets.

**New override set creation:** Same as 3-way rules — first save creates a draft, must be approved before going live.

**Scope changes:** Changes to the override set's scope selectors (Model, Carrier, Service Plan — from the flexible scoping in Part 8) are also included in the draft. Changing scope could change which devices are affected, so it requires approval.
```

---

## Prompt P11.3 — Draft Review and Comparison View

```
Build the Draft Review view — a side-by-side comparison of the draft changes against the current published version. This view is used by both 3-Way Rules and Company Override Sets (same component, different data). It is the primary view for the approver to understand what changed before approving.

**Access:**
- "Review Draft" button on the draft pending banner (P11.1, P11.2)
- "Compare Draft vs. Published" button when editing a draft

**Layout:**
Opens as a full-screen modal or dedicated page. Two-column comparison layout.

**Header area:**

- Entity name: "3-Way Rule: I-22 + Verizon + [ATM, Standard]" or "Override Set: Cord Firewall"
- Draft info: "Draft created by Adam Curcie · Apr 13, 2026 at 2:15 PM"
- Impact line: "This draft affects 342 devices across 12 companies"
- Version info: "Comparing: Draft v4 (pending) vs. Published v3 (current)"

**Parameter changes section:**

A table showing only parameters that differ between draft and published:

| Parameter Name | Published Value | Draft Value | Change Type |
|---------------|----------------|-------------|-------------|
| fw_table_1 | DENY ALL | ALLOW 10.0.1.0/24 | Modified |
| fw_rule_count | 10 | 15 | Modified |
| dns_secondary | 8.8.4.4 | 1.1.1.1 | Modified |
| cell_band_lock | — | B2,B4,B66 | Added |
| wl0_channel | 6 | — | Removed |

- "Modified" = value changed (show old → new)
- "Added" = parameter had no value in published, has value in draft
- "Removed" = parameter had a value in published, cleared in draft

Color coding:
- Modified rows: light yellow background
- Added rows: light green background
- Removed rows: light red background

Summary above table: "5 parameters changed: 3 modified, 1 added, 1 removed"

**Company assignment changes section (Override Sets only):**

If the draft includes company assignment changes, show a second section:

| Company | Change | Detail |
|---------|--------|--------|
| ABC Corp | Added | 18 devices will receive this override set |
| XYZ Inc | Added | 5 devices will receive this override set |
| Miele Sub-Co 4 | Removed | Will fall back to 3-way rule values |

Summary: "3 company assignment changes: 2 added, 1 removed"

**Affected devices summary:**

A collapsible section showing which devices are affected by this draft:
- Total affected devices: 342
- By company breakdown:
  - Cord Financial: 145 devices
  - Loaded ATMs: 89 devices
  - ABC Corp: 18 devices (newly added)
  - ... (show top 5, "and X more companies" link to expand)

**Approval section (bottom of the view):**

If the current admin is NOT the draft creator (eligible to approve):
  ┌──────────────────────────────────────────────────────────────────┐
  │ Approval                                                         │
  │                                                                  │
  │ Draft created by: Adam Curcie                                    │
  │ Your action:                                                     │
  │                                                                  │
  │ [Approve & Publish]  [Request Changes]                           │
  │                                                                  │
  │ Approving will make this draft the new published version.        │
  │ 342 devices will receive the updated configuration.              │
  └──────────────────────────────────────────────────────────────────┘

- "Approve & Publish" (primary/green button) — confirmation dialog: "Approve and publish this draft? 342 devices will receive the updated configuration on their next check-in or via on-demand push." On confirm: draft becomes published, toast: "Draft approved and published. 342 devices will receive the updated configuration."
- "Request Changes" (secondary button) — opens a text input for the approver to leave a note. The draft remains pending but gets a "Changes Requested" status. Toast: "Feedback sent to Adam Curcie."

If the current admin IS the draft creator (cannot self-approve):
  ┌──────────────────────────────────────────────────────────────────┐
  │ Approval                                                         │
  │                                                                  │
  │ You created this draft. A different admin must approve it.       │
  │ Status: Awaiting approval                                        │
  │                                                                  │
  │ [Edit Draft]  [Discard Draft]                                    │
  └──────────────────────────────────────────────────────────────────┘

**Sample data for a 3-Way Rule draft review:**
- Rule: I-22 + Verizon + [ATM, Standard]
- Draft by Adam Curcie, Apr 13, 2026
- 5 parameter changes (fw_table_1, fw_rule_count, dns_secondary modified; cell_band_lock added; wl0_channel removed)
- 342 devices affected
- Logged-in user: Aksana Rahouski (can approve)

**Sample data for an Override Set draft review:**
- Override Set: Cord Firewall (scope: Any + Any + Any)
- Draft by Devon D'Andrea, Apr 13, 2026
- 3 parameter changes + 2 companies added + 1 company removed
- 180 devices affected
- Logged-in user: Adam Curcie (can approve)
```

---

## Prompt P11.4 — Browse Pages: Draft Status Column and Indicators

```
Update the 3-Way Rules browse page and the Company Override Sets browse page to show draft status for each entity. Admins need to see at a glance which rules/sets have pending drafts awaiting approval.

**Changes to the 3-Way Rules browse page:**

Add a "Status" column after the existing columns:

| Model | Carrier | Service Plans | Parameters | Override Sets | Status | Last Modified |
|-------|---------|---------------|------------|---------------|--------|---------------|
| I-22 | Verizon | ATM, Standard | 45 | 3 | Published ✓ | Apr 12, 2026 |
| I-22 | T-Mobile | ATM | 42 | 1 | Draft Pending ⚠ | Apr 13, 2026 |
| I-22 | AT&T | Standard, Enterprise | 38 | 0 | Published ✓ | Apr 10, 2026 |
| 4100 | Verizon | ATM, IoT Basic | 35 | 2 | Draft Pending ⚠ | Apr 13, 2026 |

Status column values:
- "Published" (green badge with checkmark) — no pending draft
- "Draft Pending" (amber badge with warning icon) — draft awaiting approval. On hover or click, show tooltip: "Draft by Adam Curcie · Apr 13 · 5 params changed · 342 devices"
- "Changes Requested" (orange badge) — draft was reviewed and sent back for changes

Add a Status filter dropdown: All / Published / Draft Pending / Changes Requested

**Changes to the Company Override Sets browse page:**

Same pattern — add a "Status" column:

| Override Set Name | Model | Carrier | Service Plan | Params | Companies | Status | Modified |
|-------------------|-------|---------|--------------|--------|-----------|--------|----------|
| Cord Firewall | Any | Any | Any | 3 | 12 | Published ✓ | Apr 12 |
| Baltech Scheduler | Any | Any | Any | 2 | 1 | Draft Pending ⚠ | Apr 13 |
| ATM Data Limit | Any | Any | ATM | 4 | 25 | Changes Requested | Apr 11 |

Same status badges, tooltip on hover, and filter dropdown.

**Row click behavior update:**
- Clicking a row with "Published" status opens the editor in read-only mode (as per P11.1/P11.2)
- Clicking a row with "Draft Pending" opens the editor with the draft pending banner visible
- Clicking a row with "Changes Requested" opens the editor with the draft loaded and the reviewer's feedback shown

**Sample data:**
Update existing sample data so 2-3 entities on each browse page show draft/review statuses. The rest remain Published.
```

---

## Prompt P11.5 — Version History Tab on 3-Way Rule and Override Set Pages

```
Add a "Version History" tab to the 3-Way Rule editor page and the Company Override Set editor page. This shows the full publication history of the entity — every version that was published, who created it, who approved it, and what changed.

**Placement:**
Add a "Version History" tab alongside the existing tabs/sections on the editor page (next to the parameter editor and change history).

**Version History table:**

| Version | Status | Created By | Approved By | Published At | Changes |
|---------|--------|------------|-------------|--------------|---------|
| v4 | Draft Pending | Adam Curcie | — | — | 5 params changed |
| v3 | Published (current) | Devon D'Andrea | Adam Curcie | Apr 12, 2026 3:15 PM | 3 params modified |
| v2 | Archived | Adam Curcie | Aksana Rahouski | Apr 5, 2026 11:20 AM | Service plan added, 8 params |
| v1 | Archived | Aksana Rahouski | Adam Curcie | Mar 28, 2026 2:00 PM | Initial creation |

**Column details:**
- Version: incrementing number (v1, v2, v3...)
- Status:
  - "Draft Pending" (amber) — for the current draft awaiting approval (at most one)
  - "Changes Requested" (orange) — draft was reviewed and sent back
  - "Published (current)" (green) — the version currently live
  - "Archived" (gray) — a previously published version that was superseded
- Created By: the admin who made the changes (with initials badge)
- Approved By: the admin who approved the draft (with initials badge). Shows "—" for pending drafts.
- Published At: when the version went live. Shows "—" for pending drafts.
- Changes: summary of what changed in this version

**Expandable detail per version:**
Click any version row to expand and see:
- Full list of parameter changes (same format as the draft review view — parameter name, old value, new value, change type)
- Company assignment changes (for override sets)
- Reviewer feedback notes (if any "Changes Requested" were sent)
- Timestamp details: draft created, review requested, feedback sent, approved, published

**Actions on versions:**
- On the Draft Pending row: "Review" button (goes to P11.3 review view), "Edit" button, "Discard" button
- On the Published (current) row: "View" button (shows the published config in read-only mode)
- On Archived rows: "View" button (shows that historical version in read-only mode for reference)

**Sample data for I-22 + Verizon + [ATM, Standard] 3-Way Rule:**
- v4: Draft Pending, Adam Curcie, —, —, "5 params changed (fw_table_1, fw_rule_count, dns_secondary modified; cell_band_lock added; wl0_channel removed)"
- v3: Published (current), Devon D'Andrea, approved by Adam Curcie, Apr 12 2026, "3 params modified (wan0_gateway, lan0_netmask, dns_primary)"
- v2: Archived, Adam Curcie, approved by Aksana Rahouski, Apr 5 2026, "Added service plan Standard. Updated 8 parameter values."
- v1: Archived, Aksana Rahouski, approved by Adam Curcie, Mar 28 2026, "Initial creation. 45 parameters configured."

**Sample data for Cord Firewall override set:**
- v3: Draft Pending, Devon D'Andrea, —, —, "3 params changed, 2 companies added, 1 removed"
- v2: Published (current), Adam Curcie, approved by Devon D'Andrea, Apr 10 2026, "Updated fw_acl rule. Assigned 3 new companies."
- v1: Archived, Aksana Rahouski, approved by Adam Curcie, Mar 30 2026, "Initial creation. 3 parameters, assigned to 8 companies."
```

---

## Prompt P11.6 — Update Change Log Entries for Draft/Approval Events

```
Update the change history on the 3-Way Rule editor, Override Set editor, and the Configuration Management landing page activity feed to include draft and approval lifecycle events. These are new event types in addition to the existing parameter change events.

**New change log event types:**

For 3-Way Rules:
- "Created draft (v4): 5 parameters changed" — when a draft is saved
- "Updated draft (v4): modified dns_secondary, added cell_band_lock" — when an existing draft is edited
- "Requested changes on draft (v4): 'Please use 1.0.0.1 for dns_secondary instead'" — when an approver requests changes
- "Approved and published draft (v4): 5 parameters changed. Affects 342 devices." — when approved
- "Discarded draft (v4)" — when a draft is discarded

For Override Sets:
- "Created draft (v3): 3 parameters changed, 2 companies added" — when a draft is saved
- "Updated draft (v3): removed company Miele Sub-Co 4" — when an existing draft is edited
- "Requested changes on draft (v3): 'Confirm with Devon before adding ABC Corp'" — when changes requested
- "Approved and published draft (v3): 3 params changed, 2 companies added, 1 removed. Affects 180 devices." — when approved
- "Discarded draft (v3)" — when a draft is discarded

**Display format for approval events:**

Approval-related events should show TWO users — the creator and the approver:
- "Adam Curcie created draft (v4): 5 parameters changed"
- "Aksana Rahouski approved and published draft (v4) — created by Adam Curcie. Affects 342 devices."

This makes the two-person approval chain visible in the activity log.

**Update the landing page activity feed (P5.2) sample data:**

Add these entries interleaved with existing entries at the top:

- Apr 13, 2026 3:45 PM — Devon D'Andrea — [Override Set] Cord Firewall — "Created draft (v3): 3 params changed, 2 companies added"
- Apr 13, 2026 2:15 PM — Adam Curcie — [3-Way Rule] I-22 + Verizon + [ATM, Standard] — "Created draft (v4): 5 params changed"
- Apr 12, 2026 3:15 PM — Adam Curcie — [3-Way Rule] I-22 + Verizon + [ATM, Standard] — "Approved and published draft (v3) — created by Devon D'Andrea. Affects 342 devices."
- Apr 12, 2026 10:00 AM — Devon D'Andrea — [Override Set] Cord Firewall — "Approved and published draft (v2) — created by Adam Curcie. Affects 145 devices."

**Update the per-entity change history (P5.1) sample data:**

For the I-22 + Verizon + [ATM, Standard] 3-Way Rule, add these entries at the top of its change history:
- Apr 13, 2026 2:15 PM — Adam Curcie — "Created draft (v4): modified fw_table_1, fw_rule_count, dns_secondary; added cell_band_lock; removed wl0_channel"
- Apr 12, 2026 3:15 PM — Adam Curcie — "Approved and published draft (v3) — created by Devon D'Andrea"
- Apr 12, 2026 9:30 AM — Devon D'Andrea — "Created draft (v3): modified wan0_gateway, lan0_netmask, dns_primary"

For the Cord Firewall override set, add:
- Apr 13, 2026 3:45 PM — Devon D'Andrea — "Created draft (v3): modified fw_acl, fw_enable; added fw_log_level; added ABC Corp, XYZ Inc; removed Miele Sub-Co 4"
- Apr 12, 2026 10:00 AM — Devon D'Andrea — "Approved and published draft (v2) — created by Adam Curcie"
```

---

## Prompt P11.7 — Landing Page: Pending Approvals Summary Card

```
Add a "Pending Approvals" summary card to the Configuration Management landing page. This gives admins immediate visibility into drafts that need their attention.

**Placement:**
Add a prominent card/banner above the existing four section cards (Grand Schema, Model Configuration, etc.) but below the page heading. This is the first thing an admin sees when entering the Configuration Management area.

**Card design:**

When there are pending drafts:

┌──────────────────────────────────────────────────────────────────┐
│ 📋 Pending Approvals                                      3 drafts │
│                                                                  │
│ ┌────────────────────────────────────────────────────────────┐   │
│ │ 3-Way Rule: I-22 + Verizon + [ATM, Standard]              │   │
│ │ Draft by Adam Curcie · Apr 13 · 5 params · 342 devices    │   │
│ │                                          [Review Draft →]  │   │
│ ├────────────────────────────────────────────────────────────┤   │
│ │ Override Set: Cord Firewall                                │   │
│ │ Draft by Devon D'Andrea · Apr 13 · 3 params, 2 companies  │   │
│ │ · 180 devices                            [Review Draft →]  │   │
│ ├────────────────────────────────────────────────────────────┤   │
│ │ Override Set: ATM Data Limit  ⚠ Changes Requested          │   │
│ │ Draft by Adam Curcie · Apr 11 · Feedback from Devon        │   │
│ │                                          [View Feedback →]  │   │
│ └────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘

Each row is clickable — links to the draft review view (P11.3) for that entity.

For drafts the current user created: the action reads "[View Draft →]" (they can't approve their own).
For drafts by OTHER users: the action reads "[Review Draft →]" (they can approve).
For drafts with "Changes Requested" status: show the status badge and "[View Feedback →]".

**When there are no pending drafts:**

┌──────────────────────────────────────────────────────────────────┐
│ ✓ No pending approvals                                           │
│ All configuration changes are published and up to date.          │
└──────────────────────────────────────────────────────────────────┘

This card can be a subtle, muted style when empty — it's informational, not a call to action.

**Sample data:**
Show 3 pending drafts as described above. Logged-in user is Aksana Rahouski, so:
- Adam Curcie's drafts show "[Review Draft →]" (Aksana can approve)
- Devon D'Andrea's draft shows "[Review Draft →]" (Aksana can approve)
- The "Changes Requested" draft shows the feedback note and "[View Feedback →]"
```
