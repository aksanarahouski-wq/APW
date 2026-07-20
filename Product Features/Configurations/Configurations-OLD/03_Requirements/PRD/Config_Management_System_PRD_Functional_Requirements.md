# Functional Requirements
## Hierarchical Configuration Management System

**Document Version:** 1.5
**Date:** January 12, 2026
**Last Updated:** FR-1 Updated - Two-Way Rules definition expanded to include all 6 two-factor combinations with examples and sub-priority ordering
**Author:** Aksana Rahouski / Orases Team
**Status:** Draft for Review
**Parent Document:** [Config Management System PRD](./Config_Management_System_PRD.md)

---

This document contains the detailed functional requirements for the Hierarchical Configuration Management System. For the full context, executive summary, and other sections, please refer to the [main PRD document](./Config_Management_System_PRD.md).

---

## Functional Requirements

### FR-1: Config Key Schema Management

**FR-1.1: Config Key CRUD Operations**
- System must provide admin interface to Create, Read, Update, Delete config keys
- Each key must have: unique name (string, required), data type (enum, required), description (text, optional)
- Supported data types: String, Integer, Boolean, IP Address, Time (HH:MM format), Enum (dropdown with predefined values)
- Key names must be unique across system (case-insensitive validation)
- Deleting a key must require confirmation and check if key is used in any layer configs
- If key has values set in any layer, deletion must be blocked with warning: "Key is in use in X layers"

**FR-1.1a: Multi-Factor Conditional Rules Framework**
- System must support generic conditional rules framework for parameters that require multi-factor logic
- **Rule Types Supported:**
  - **Two-Way Rules**: Conditions on ANY TWO of the four factors (Model, Carrier, Service Plan, Customer)
    - **6 Supported Combinations:**
      1. Model + Carrier (carrier-specific features per model)
      2. Carrier + Service Plan (carrier data policies by plan tier)
      3. Model + Service Plan (model pricing tier by plan)
      4. Carrier + Customer (customer carrier agreements)
      5. Model + Customer (customer model configurations)
      6. Service Plan + Customer (customer plan exceptions)
    - **Sub-Priority Ordering**: System checks combinations in order 1-6 above (first match wins)
  - **Three-Way Rules**: Conditions on Model AND Carrier AND Service Plan (service plan baselines)
  - **Four-Way Rules**: Conditions on Model AND Carrier AND Service Plan AND Customer (customer-specific exceptions)
  - **Extensibility**: Architecture supports adding additional factors if needed in the future
- **Configuration Flags per Key:**
  - `has_two_way_rules` (boolean): Parameter uses any 2-factor conditional logic (6 combinations supported)
  - `has_three_way_rules` (boolean): Parameter uses Model + Carrier + Service Plan conditional logic
  - `has_four_way_rules` (boolean): Parameter uses Model + Carrier + Service Plan + Customer conditional logic (customer exceptions)
  - Parameters can have multiple flags enabled (higher-factor rules take precedence in the priority hierarchy)
- **11-Level Priority Hierarchy Integration:**
  - Conditional rules are integrated into the config resolution engine at specific priority levels:
    - **Level 2**: Four-Way Rules (customer-specific exceptions) - checked after Device Override
    - **Level 4**: Three-Way Rules (service plan baselines) - checked after Company Override
    - **Level 6**: Two-Way Rules (any 2-factor combination with 6 sub-priorities 6.1-6.6) - checked after Service Plan Layer
  - Device Override (Level 1) and Company Override (Level 3) can still override any rule values
  - See FR-3.1 for complete 11-level priority hierarchy details with sub-priority ordering
- **Rule Definition Interface:**
  - Admin selects parameter and rule type (2-way, 3-way, or 4-way)
  - **For two-way rules**:
    - **Step 1**: Select which 2-factor combination (6 options: Model+Carrier, Carrier+Plan, Model+Plan, Carrier+Customer, Model+Customer, Plan+Customer)
    - **Step 2**: Select factor values based on combination chosen (dropdowns populated accordingly)
    - **Step 3**: Set value
    - **System automatically assigns sub-priority** (6.1-6.6) based on combination type selected
  - For three-way rules: Select model, carrier, and service plan, set value (baseline for all customers)
  - For four-way rules: Select model, carrier, service plan, and specific customer, set value (customer exceptions only)
  - System validates all selected factors exist in database
  - Priority level is determined by rule type (2-way=Level 6, 3-way=Level 4, 4-way=Level 2); sub-priorities auto-assigned for 2-way rules
  - Add optional description/notes for rule business logic
- **Rule Examples:**
  - Two-way (Model+Carrier): `IF model=i-22 AND carrier=VZW THEN mqtt_enable=1`
  - Two-way (Carrier+Plan): `IF carrier=VZW AND service_plan=ATM THEN traffic_day_threshold=3584MB`
  - Two-way (Model+Plan): `IF model=i-22 AND service_plan=ATM THEN traffic_day_threshold=350MB`
  - Two-way (Carrier+Customer): `IF carrier=VZW AND customer=VIP_Corp THEN traffic_day_threshold=10GB`
  - Two-way (Model+Customer): `IF model=i-22 AND customer=Miele THEN digitalio_config=[custom]`
  - Two-way (Plan+Customer): `IF service_plan=ATM AND customer=Premium THEN traffic_day_threshold=1000MB`
  - Three-way: `IF model=4500 AND carrier=ATT AND service_plan=ATM THEN fw_acl=[40 whitelist rules]`
  - Four-way: `IF model=i-22 AND carrier=ATT AND service_plan=ATM AND customer=CORD THEN fw_acl=[50 custom rules]`
  - Multiple rules: Parameter can have rules of different types for different combinations
- **Rule Type Selection Guidelines:**
  - Use **Two-Way Rules** when value depends on ANY TWO factors:
    - **Model+Carrier**: Carrier-specific features per model (e.g., i-22 VZW devices enable MQTT)
    - **Carrier+ServicePlan**: Carrier data policies by plan tier (e.g., VZW ATM plan gets 3584MB daily limit)
    - **Model+ServicePlan**: Model pricing tier by plan (e.g., i-22 devices on ATM get restrictive limit)
    - **Carrier+Customer**: Customer carrier agreements (e.g., VIP Corp negotiated higher VZW data allowance)
    - **Model+Customer**: Customer model configs (e.g., Miele's i-22 devices need specific I/O settings)
    - **ServicePlan+Customer**: Customer plan exceptions (e.g., Premium customer gets upgraded ATM allowance)
  - Use **Three-Way Rules** to define service plan baselines that apply to all customers (e.g., all ATM plan devices get certain features)
  - Use **Four-Way Rules** ONLY for customer-specific exceptions to Three-Way Rule baselines (e.g., CORD needs custom firewall rules)
  - **Important**: Create Three-Way Rule baseline first, then add Four-Way Rules for customer exceptions as needed
- **Rule Validation:**
  - Cannot create conflicting rules (same factors, different values)
  - Cannot create incomplete rules (all required factors must be specified)
  - Warn if rule will never match (factors don't exist or combination impossible)
  - For Four-Way Rules: Warn if no corresponding Three-Way Rule baseline exists
  - Show coverage report: "Parameter has X two-way rules, Y three-way rules, Z customer exceptions (four-way)"
- **Rule Behavior:**
  - Rules are evaluated during config resolution based on device attributes (model, carrier, service plan, company)
  - When rule conditions match, rule value overrides standard layer inheritance
  - If no rule matches, fall back to standard layer resolution (Service Plan → Carrier → Model → Global)
  - Device Override (Level 1) and Company Override (Level 3) can still override any rule-determined values
  - NULL rules explicitly disable a parameter for specific combinations

**FR-1.2: Layer Availability Configuration**
- For each key, admin must specify which layers can set this key
- Layer checkboxes: Global, Model, Carrier, Service Plan, Company, Device
- At least one layer must be selected (validation error if none)
- If key is marked "Required", Global layer must be selected (validation enforced)
- Changes to layer availability must show impact: "This will affect X existing configs"

**FR-1.3: Key Flags & Properties**
- **Required Flag**: Boolean (yes/no)
  - If yes, effective config must have value for this key (validation on render)
  - If yes, Global layer must be in available layers
  - Default: No
- **Customer Configurable Flag**: Boolean (yes/no)
  - If yes, customers can set this key at Company/Device layers (if those layers are available)
  - If no, key is completely hidden from customer UI
  - Default: No
- **Conditional Rules Flags**: Indicate which multi-factor conditional rule types apply to this parameter
  - **has_two_way_rules** (boolean): Parameter requires any 2-factor conditional logic (6 combinations supported)
    - If yes, admin can create Two-Way Rules for this parameter (evaluated at Priority Level 6 with sub-priorities 6.1-6.6)
    - **Supported combinations**: Model+Carrier, Carrier+ServicePlan, Model+ServicePlan, Carrier+Customer, Model+Customer, ServicePlan+Customer
    - Example parameters: `mqtt_enable` (Model+Carrier), `traffic_day_threshold` (Carrier+Plan or Model+Plan), `digitalio_config` (Model+Customer)
    - Default: No
  - **has_three_way_rules** (boolean): Parameter requires Model + Carrier + Service Plan conditional logic
    - If yes, admin can create Three-Way Rules for this parameter (evaluated at Priority Level 4)
    - Used for service plan baselines that apply to all customers
    - Example parameters: `fw_acl`, `traffic_day_threshold`, `alarm_output_options` (vary by plan)
    - Default: No
  - **has_four_way_rules** (boolean): Parameter requires Model + Carrier + Service Plan + Company conditional logic
    - If yes, admin can create Four-Way Rules for this parameter (evaluated at Priority Level 2)
    - Used ONLY for customer-specific exceptions to Three-Way Rule baselines
    - Example parameters: `fw_acl`, `ntp_server`, `traffic_day_threshold` (customer-specific overrides)
    - Default: No
  - **Multiple Flags**: Parameters can have multiple conditional rule flags enabled
    - Higher-factor rules take precedence: Four-Way (Level 2) > Three-Way (Level 4) > Two-Way (Level 6)
    - Common pattern: has_three_way_rules=true AND has_four_way_rules=true (baseline + customer exceptions)
  - **Integration with 11-Level Priority Hierarchy**: Conditional rules are checked at specific levels during config resolution (see FR-3.1)
- **Default Value**: Optional fallback value used if key is not set anywhere in hierarchy
  - Data type must match key's data type
  - Used at Priority Level 10 during config resolution (after all layers and rules checked)
  - Can be NULL (no default)
  - Schema default is checked only if no layer, rule, or override provides a value

**FR-1.4: Validation Rules Configuration**
- Admin must be able to specify validation rules per key:
  - **String**: Regex pattern, min/max length
  - **Integer**: Min/max value, must be positive/negative
  - **IP Address**: IPv4 format validation (auto-applied)
  - **Time**: 24-hour format HH:MM (auto-applied)
  - **Enum**: List of allowed values (comma-separated)
  - **Boolean**: true/false or 0/1 (auto-applied)
- Validation rules must be enforced when setting key values at any layer
- Invalid values must show inline error message with validation requirement

**FR-1.5: Config Key Schema Import**
- System must provide import tool to migrate existing 600+ keys from current config files
- Import process:
  1. Admin uploads sample config files (or system reads from existing file storage)
  2. System analyzes files and extracts unique keys
  3. For each key, system infers: data type (best guess), common values
  4. System presents list to admin for review
  5. Admin assigns for each key: layer availability, required flag, customer configurable flag, validation rules
  6. System creates key schema records in database
- Import tool must detect conflicts (keys with different data types across files)
- Import tool must allow bulk editing (select multiple keys, set layer availability for all)
- Import must be idempotent (can run multiple times without duplicating keys)

**FR-1.6: Key List & Search**
- Admin interface must show list of all config keys
- List must support:
  - Search by key name (partial match)
  - Filter by: layer availability, required flag, customer configurable flag, data type
  - Sort by: name (A-Z, Z-A), recently modified, data type
  - Pagination (50 keys per page default)
- List must show key summary: name, data type, layers, flags (icons for required/customer configurable)
- Clicking key opens detail view/edit form

---

### FR-2: Layer Configuration System

**FR-2.1: Global Layer Configuration**
- System must provide interface for admin to configure Global layer
- Interface must show all config keys (600+) from schema
- For each key, show:
  - Key name and description
  - Data type
  - Current value (editable)
  - Required indicator (red asterisk if required)
  - Default value from schema (if set)
- Keys must be organized in logical groups (TBD with APW: DNS/Network, Security, Alarm I/O, Scheduling, etc.)
- Collapsible sections for each group
- Search/filter within keys
- Validation on value change:
  - Must match data type
  - Must pass validation rules
  - Required keys cannot be left empty (NULL not allowed for required keys)
- "Save Changes" button at top and bottom
- On save, validate all required keys have values
- If validation fails, show error summary at top: "3 required keys are missing values"
- On successful save, show success message: "Global configuration saved. X devices will receive updates."
- Calculate affected devices count on save (all devices in system)

**FR-2.2: Model Layer Configuration**
- System must provide interface per model (i-22, i-52, 4100, Origin, etc.)
- Model selection dropdown or list page
- For selected model, show only keys available at Model layer (per schema)
- For each key, show:
  - Key name and description
  - Inherited value from Global Layer only (displayed in grayed-out box with label "Global")
  - Override value (editable input)
  - "Clear Override" button (sets to NULL, explicitly clear)
  - **For parameters with conditional rules** (`has_two_way_rules`, `has_three_way_rules`, `has_four_way_rules` flags enabled):
    - Show indicator: "⚠️ Uses Conditional Rules"
    - Link to Conditional Rules Management (FR-3a) filtered to show rules involving this model
    - **Why**: This parameter's value may depend on multi-factor combinations (e.g., Model+Carrier, Model+ServicePlan, etc.)
    - **Note**: Conditional rules are managed in the Conditional Rules interface, not here
- If override is not set (empty), config resolution will use Global value (or applicable conditional rule)
- If override is set to NULL (explicitly cleared), config resolution uses NULL even if Global has value or rules exist
- Visual indicator:
  - Inherited (no override): green checkmark icon
  - Overridden: blue edit icon
  - Explicitly cleared: gray X icon
- Validation on value change (data type and validation rules)
- **Contextual Access to Conditional Rules:**
  - Button/link: "View Conditional Rules involving [Model Name]"
  - Opens Conditional Rules Management (FR-3a) filtered to show all rules (Two-Way, Three-Way, Four-Way) WHERE model = this model
  - Read-only overview showing how this model is used in various rules
  - Can click through to edit rules in centralized Rules Management
  - **Purpose**: Visibility into how this model participates in conditional logic across different carriers, plans, and customers
  - **Note**: Rules are not created/edited here; this is a read-only filtered view for awareness
- On save, calculate affected devices: "X devices of model i-22 will receive updates"
- Confirmation if affecting >10 devices

**FR-2.3: Carrier Layer Configuration**
- System must provide interface per carrier (Verizon, T-Mobile, AT&T, etc.)
- Carrier selection dropdown or list page
- Show only keys available at Carrier layer
- Display inherited values from Global Layer only (displayed in grayed-out box with label "Global")
  - **Note**: Cannot show "inherited from Model" because there are multiple models (i-22, 4500, Origin, etc.). Each model may have different values.
  - **Why**: Carrier layer is used WITH multiple models simultaneously (many-to-many relationship, not hierarchical)
  - **To set values for specific Carrier+Model combinations**: Use Two-Way Rules (Model+Carrier) in Conditional Rules interface
- For each key, show:
  - Key name and description
  - Inherited value from Global (if no override set)
  - Override value (editable input)
  - "Clear Override" button (sets to NULL)
  - **For parameters with conditional rules** (`has_two_way_rules`, `has_three_way_rules`, `has_four_way_rules` flags enabled):
    - Show indicator: "⚠️ Uses Conditional Rules"
    - Link to Conditional Rules Management filtered to show rules involving this carrier
    - **Why**: This parameter's value may depend on multi-factor combinations involving this carrier
- Allow overrides and NULL clearing (same pattern as Model layer)
- Visual indicators: inherited (green), overridden (blue), cleared (gray)
- Validation on value change (data type and validation rules)
- **Contextual Access to Conditional Rules:**
  - Button/link: "Manage Conditional Rules for [Carrier Name]"
  - Opens Conditional Rules Management (FR-3a) filtered to show rules WHERE carrier = this carrier
  - Shows all rule types involving this carrier:
    - **Two-Way Rules**: Model+Carrier, Carrier+ServicePlan, Carrier+Customer
    - **Three-Way Rules**: Model+Carrier+ServicePlan
    - **Four-Way Rules**: Model+Carrier+ServicePlan+Customer
  - Shows existing rules (e.g., "Model i-22 + Carrier VZW → mqtt_enable=1", "Carrier VZW + Plan ATM → traffic_day_threshold=3584MB")
  - Allows creating new rules involving this carrier
  - **Note**: Rules are managed centrally in FR-3a; this is a contextual entry point for discovery and convenience
- On save, calculate affected devices: "X devices on Verizon will receive updates"
- If change affects devices across multiple models, show breakdown: "500 i-22 devices, 300 4100 devices"

**FR-2.4: Service Plan Layer Configuration**
- System must provide interface per service plan
- Service plan selection dropdown or list page
- Show only keys available at Service Plan layer
- Display inherited values from Global Layer only (displayed in grayed-out box with label "Global")
  - **Note**: Cannot show "inherited from Model/Carrier" because there are multiple models and carriers. Service plans are used across different model+carrier combinations.
  - **Why**: Service Plan layer is used WITH multiple models and carriers simultaneously (many-to-many relationship, not hierarchical)
  - **To set values for specific combinations**:
    - ServicePlan + Model → Two-Way Rule (Model+ServicePlan)
    - ServicePlan + Carrier → Two-Way Rule (Carrier+ServicePlan)
    - ServicePlan + Model + Carrier → Three-Way Rule (service plan baselines)
- For each key, show:
  - Key name and description
  - Inherited value from Global (if no override set)
  - Override value (editable input)
  - "Clear Override" button (sets to NULL)
  - **For parameters with conditional rules** (`has_two_way_rules`, `has_three_way_rules`, `has_four_way_rules` flags enabled):
    - Show indicator: "⚠️ Uses Conditional Rules"
    - Link to Conditional Rules Management filtered to show rules involving this service plan
    - **Why**: This parameter's value may depend on multi-factor combinations involving this service plan
- Allow overrides and NULL clearing
- Visual indicators: inherited (green), overridden (blue), cleared (gray)
- Validation on value change (data type and validation rules)
- **Contextual Access to Conditional Rules:**
  - Button/link: "Manage Conditional Rules for [Service Plan Name]"
  - Opens Conditional Rules Management (FR-3a) filtered to show rules WHERE service_plan = this plan
  - Shows all rule types involving this service plan:
    - **Two-Way Rules**: Carrier+ServicePlan, Model+ServicePlan, ServicePlan+Customer
    - **Three-Way Rules**: Model+Carrier+ServicePlan (service plan baselines)
    - **Four-Way Rules**: Model+Carrier+ServicePlan+Customer (customer exceptions to baselines)
  - Shows existing rules (e.g., "Carrier VZW + Plan ATM → traffic_day_threshold=3584MB", "Model i-22 + Carrier ATT + Plan ATM → fw_acl=[40 rules]")
  - Allows creating new rules involving this service plan
  - **Purpose**: Service plan policies and features that vary by multi-factor combinations
  - **Note**: Rules are managed centrally in FR-3a; this is a contextual entry point for discovery and convenience
- On save, calculate affected devices by service plan
- Preview affected companies: "5 companies with 1,200 total devices"

**FR-2.5: Company Layer Configuration (Admin)**
- System must provide interface per company
- Company selection (search by name, dropdown, or list page)
- Show only keys available at Company layer
- Display inherited values from Global Layer only (displayed in grayed-out box with label "Global")
  - **Note**: Cannot show "inherited from Model/Carrier/ServicePlan" because companies have devices with multiple different combinations
  - **Why**: A company may have devices with different Models (i-22, 4500), different Carriers (VZW, ATT), and different Service Plans (ATM, Tier1)
  - **Example**: Company CORD has: 200 i-22+VZW+ATM devices, 150 4500+ATT+ATM devices, 100 i-22+VZW+Tier1 devices
  - **To set customer-specific values for combinations**:
    - Customer + Model → Two-Way Rule (Model+Customer)
    - Customer + Carrier → Two-Way Rule (Carrier+Customer)
    - Customer + ServicePlan → Two-Way Rule (ServicePlan+Customer)
    - Customer + Model + Carrier + ServicePlan → Four-Way Rule (customer-specific exceptions)
- For each key, show:
  - Key name and description
  - Inherited value from Global (if no override set)
  - Override value (editable input)
  - "Clear Override" button (sets to NULL)
  - **For parameters with conditional rules** (`has_two_way_rules`, `has_three_way_rules`, `has_four_way_rules` flags enabled):
    - Show indicator: "⚠️ Uses Conditional Rules"
    - Link to Conditional Rules Management filtered to show rules involving this customer
    - **Why**: This parameter's value may depend on multi-factor combinations involving this customer
- Allow overrides and NULL clearing
- Visual indicators: inherited (green), overridden (blue), cleared (gray)
- Validation on value change (data type and validation rules)
- **Contextual Access to Conditional Rules:**
  - Button/link: "Manage Conditional Rules for [Company Name]"
  - Opens Conditional Rules Management (FR-3a) filtered to show rules WHERE company = this company
  - Shows all rule types involving this customer:
    - **Two-Way Rules**: Model+Customer, Carrier+Customer, ServicePlan+Customer
    - **Four-Way Rules**: Model+Carrier+ServicePlan+Customer (customer-specific exceptions)
  - Shows existing rules (e.g., "Model i-22 + Customer CORD → digitalio_config=[custom]", "Model i-22 + Carrier ATT + Plan ATM + Customer CORD → fw_acl=[50 custom rules]")
  - Allows creating new rules involving this customer
  - **Purpose**: Customer-specific configurations and exceptions to service plan baselines
  - **Workflow**: For Four-Way Rules, system warns if no corresponding Three-Way Rule baseline exists; recommends creating baseline first
  - **Note**: Rules are managed centrally in FR-3a; this is a contextual entry point for customer-specific configuration management
- On save, apply to all devices in company (portfolio-wide)
- Preview: "This will update 500 devices in Company ABC Corp"
- Confirmation required before saving

**FR-2.6: Company Layer Configuration (Customer)**
- Customer must see own company config page only (cannot select other companies)
- Show only keys where "Customer Configurable" = Yes AND "Available at Company Layer" = Yes
- All other keys completely hidden (not shown as read-only)
- Display inherited values (customers see values but not which layer they come from - simplify)
- UI must be highly user-friendly:
  - **IP Address inputs**: Separate fields per octet with validation (e.g., 192.168.1.1 as four inputs)
  - **Time inputs**: Time picker (dropdown for hour, dropdown for minute) or time picker widget
  - **Boolean**: Toggle switch or radio buttons (Yes/No)
  - **Enum**: Dropdown with allowed values only
  - **String**: Text input with validation feedback
- Inline validation: red border and error message if invalid
- "Apply to All Devices" button
- Preview: "This will update 500 of your devices"
- Confirmation dialog: "Are you sure you want to apply these changes to all 500 devices in your fleet?"
- Success message after save: "Configuration applied successfully. Your devices will update on next check-in."

**FR-2.7: Device Layer Configuration (Admin)**
- System must provide interface for any device (search by device ID, name, or browse)
- Show only keys available at Device layer
- Display inherited values with full source attribution (complete 11-level resolution)
  - **Why this works**: Each device has specific Model, Carrier, ServicePlan, and Company values, so the full conditional rules resolution algorithm can execute
  - **Example device**: Device #12345 has Model=i-22, Carrier=VZW, ServicePlan=ATM, Company=CORD
  - **Possible sources shown:**
    - "Global" - System-wide default
    - "Model Layer (i-22)" - Model-specific value
    - "Carrier Layer (VZW)" - Carrier-specific value
    - "Service Plan Layer (ATM)" - Service plan-specific value
    - "Company Layer (CORD)" - Customer portfolio-wide setting
    - "Two-Way Rule (Model=i-22 + Carrier=VZW)" - Model+Carrier conditional rule
    - "Two-Way Rule (Carrier=VZW + Plan=ATM)" - Carrier+ServicePlan conditional rule
    - "Two-Way Rule (Model=i-22 + Plan=ATM)" - Model+ServicePlan conditional rule
    - "Three-Way Rule (Model=i-22 + Carrier=VZW + Plan=ATM)" - Service plan baseline
    - "Four-Way Rule (Model=i-22 + Carrier=VZW + Plan=ATM + Customer=CORD)" - Customer exception
    - "Company-Specific Device Override" - Customer set this device specifically
    - "Device Override" - Admin set this device specifically
  - **Source attribution format**: "Parameter: value (Source)"
  - Example: "mqtt_enable: 1 (Two-Way Rule: Model=i-22 + Carrier=VZW)"
  - Example: "fw_acl: [50 rules] (Four-Way Rule: Customer Exception for CORD)"
  - Example: "lan_ip: 192.168.1.100 (Device Override)"
- For each key, show:
  - Key name and description
  - Inherited value with source (if no override set)
  - Override value (editable input)
  - "Clear Override" button (sets to NULL)
- Allow overrides and NULL clearing (same pattern as other layers)
- Visual indicators: color-code by source type (Global=gray, Layer=blue, Rule=purple, Override=green)
- Validation on value change (data type and validation rules)
- On save, apply to single device only
- Confirmation: "Apply config changes to Device 12345?"
- Success message: "Device 12345 configuration updated. Changes will apply on next check-in."

**FR-2.8: Device Layer Configuration (Customer)**
- Customer must see own devices only (filtered by company)
- Device selection/list page showing customer's devices
- Show only keys where "Customer Configurable" = Yes AND "Available at Device Layer" = Yes
- Display inherited values with source attribution (full 11-level resolution for this specific device)
  - **Why this works**: Each device has specific Model, Carrier, ServicePlan, and Company, so resolution algorithm executes completely
  - **Sources shown** (simplified labels for customer view):
    - "System Default" - Global or schema default
    - "Your Company Settings" - Company Layer value
    - "Service Plan Settings" - Service Plan Layer or Three-Way Rule baseline
    - "Custom Configuration" - Two-Way Rule or Four-Way Rule (customer-specific)
    - "Device Setting" - Device Override
  - **Source attribution format**: "Parameter: value (Source)"
  - Example: "dns_primary: 8.8.8.8 (System Default)"
  - Example: "reboot_time: 02:00 (Your Company Settings)"
  - Example: "traffic_threshold: 1000MB (Custom Configuration)"
- For each key, show:
  - Key name and user-friendly description
  - Current value with source
  - Override input (if customer can modify)
- Same user-friendly UI as Company layer (IP inputs, time pickers, toggles, etc.)
- Visual indicators: inherited (green), overridden (blue)
- Validation on value change (data type and validation rules)
- On save, apply to selected single device
- Confirmation: "Apply these changes to Device 12345?"
- Success message with instruction: "Configuration updated. Device will receive changes on next check-in (typically within 15 minutes)."

**FR-2.9: Reusable Layer Editor Components**
- System must implement reusable UI components for layer editors
- Components:
  - **Key List Component**: Displays keys with grouping, search, filter
  - **Value Editor Component**: Renders appropriate input based on data type (text, IP, time, toggle, dropdown)
  - **Inherited Value Display**: Shows value with source attribution label
  - **Override Controls**: "Override" button, "Clear Override" button, current override value
  - **Validation Feedback**: Inline error messages, field highlighting
  - **Preview Panel**: Shows affected devices count and breakdown
  - **Save Bar**: Sticky bar at bottom with Save/Cancel buttons, validation summary
- Components must be configuration-driven (layer type, available keys, user permissions)
- Reduces development time since all 6 layers use same components with different data

---

### FR-3: Config Resolution Engine

**FR-3.1: Effective Config Calculation with 11-Level Priority Hierarchy**
- System must calculate effective config for any device on-demand
- Input: Device ID
- Process:
  1. Fetch device attributes: Model, Carrier, Service Plan, Company
  2. Fetch config key schema (all 600+ keys)
  3. **For each key, resolve value using 11-level priority hierarchy:**

  **Level 1: Device Override** *(Highest Priority)*
  - If value exists at Device layer (including NULL) → use it, stop resolution
  - Source attribution: "Device Layer - Device #{id}"
  - Use case: Site-specific configurations, one-off overrides, troubleshooting

  **Level 2: Four-Way Rule (Customer-Specific Exception)**
  - If key has `has_four_way_rules = true`:
    - Query for matching rule WHERE model=device.model AND carrier=device.carrier AND service_plan=device.service_plan AND company=device.company
    - If matching rule found → use rule value, stop resolution
    - If multiple rules match (shouldn't happen), use highest priority (lowest priority number)
    - If no matching rule → continue to next level
  - Source attribution: "Four-Way Rule - [Company Name] Exception"
  - Use case: Customer-specific exceptions to service plan policies

  **Level 3: Company Override**
  - If value exists at Company layer (including NULL) → use it, stop resolution
  - Source attribution: "Company Layer - [Company Name]"
  - Use case: Customer portfolio-wide settings, operational standards

  **Level 4: Three-Way Rule (Service Plan Baseline)**
  - If key has `has_three_way_rules = true`:
    - Query for matching rule WHERE model=device.model AND carrier=device.carrier AND service_plan=device.service_plan
    - If matching rule found → use rule value, stop resolution
    - If multiple rules match (shouldn't happen), use highest priority (lowest priority number)
    - If no matching rule → continue to next level
  - Source attribution: "Three-Way Rule - [Model]+[Carrier]+[ServicePlan]"
  - Use case: Service plan features and policies that apply to all customers

  **Level 5: Service Plan Layer**
  - If value exists at Service Plan layer (including NULL) → use it, stop resolution
  - Source attribution: "Service Plan Layer - [Plan Name]"
  - Use case: Service tier differentiation

  **Level 6: Two-Way Rule (Any 2-Factor Combination)**
  - If key has `has_two_way_rules = true`:
    - Query for matching rules based on device attributes
    - **Sub-Priority Order** (check combinations in this sequence, first match wins):

    **6.1: Carrier + Service Plan Rule**
    - Query: WHERE carrier=device.carrier AND service_plan=device.service_plan (model_id IS NULL, company_id IS NULL)
    - If matching rule found → use rule value, stop resolution
    - Source attribution: "Two-Way Rule (Carrier+Plan) - [Carrier]+[ServicePlan]"
    - Use case: Carrier data policies vary by service plan tier
    - **Real-World Example:**
      ```
      Device: i-22, VZW, ATM Plan, CORD Company
      Rule: IF carrier=VZW AND service_plan=ATM THEN traffic_day_threshold=3584MB
      Result: Uses 3584MB (VZW's ATM plan policy applies regardless of model)

      Different carrier, same plan:
      Device: i-22, ATT, ATM Plan, CORD Company
      Rule: IF carrier=ATT AND service_plan=ATM THEN traffic_day_threshold=350MB
      Result: Uses 350MB (AT&T's ATM plan has different data policy)
      ```

    **6.2: Model + Service Plan Rule**
    - Query: WHERE model=device.model AND service_plan=device.service_plan (carrier_id IS NULL, company_id IS NULL)
    - If matching rule found → use rule value, stop resolution
    - Source attribution: "Two-Way Rule (Model+Plan) - [Model]+[ServicePlan]"
    - Use case: Model pricing tier affects service plan features
    - **Real-World Example:**
      ```
      Device: i-22, ATT, ATM Plan, Customer A
      Rule: IF model=i-22 AND service_plan=ATM THEN traffic_day_threshold=350MB
      Result: Uses 350MB (i-22 is entry-level tier with restrictive limits)

      Higher model tier, same plan:
      Device: 4500, ATT, ATM Plan, Customer A
      Rule: IF model=4500 AND service_plan=ATM THEN traffic_day_threshold=5GB
      Result: Uses 5GB (4500 is premium tier with generous limits)
      ```

    **6.3: Model + Carrier Rule**
    - Query: WHERE model=device.model AND carrier=device.carrier (service_plan_id IS NULL, company_id IS NULL)
    - If matching rule found → use rule value, stop resolution
    - Source attribution: "Two-Way Rule (Model+Carrier) - [Model]+[Carrier]"
    - Use case: Carrier-specific features per device model
    - **Real-World Example:**
      ```
      Device: i-22, VZW, ATM Plan, Customer A
      Rule: IF model=i-22 AND carrier=VZW THEN mqtt_enable=1
      Result: mqtt_enable=1 (Device Manager enabled for VZW i-22 devices)

      Same model, different carrier:
      Device: i-22, ATT, ATM Plan, Customer A
      Rule: IF model=i-22 AND carrier=ATT THEN mqtt_enable=0
      Result: mqtt_enable=0 (AT&T i-22 devices don't get Device Manager)

      Different model, same carrier:
      Device: 4500, VZW, ATM Plan, Customer A
      No rule (4500 doesn't support Device Manager hardware)
      Result: Continue to next level (falls back to layer inheritance)
      ```

    **6.4: Carrier + Customer Rule**
    - Query: WHERE carrier=device.carrier AND company=device.company (model_id IS NULL, service_plan_id IS NULL)
    - If matching rule found → use rule value, stop resolution
    - Source attribution: "Two-Way Rule (Carrier+Customer) - [Carrier]+[Company]"
    - Use case: Customer-specific carrier agreements (VIP data allowances, custom SLAs)
    - **Real-World Example:**
      ```
      Device: i-22, VZW, ATM Plan, VIP_Corp
      Rule: IF carrier=VZW AND customer=VIP_Corp THEN traffic_day_threshold=10GB
      Result: Uses 10GB (VIP_Corp negotiated higher VZW data allowance)

      Standard customer, same carrier:
      Device: i-22, VZW, ATM Plan, Standard_Customer
      No rule for Standard_Customer
      Result: Continue checking (will find Carrier+Plan rule at 6.1)
      ```

    **6.5: Model + Customer Rule**
    - Query: WHERE model=device.model AND company=device.company (carrier_id IS NULL, service_plan_id IS NULL)
    - If matching rule found → use rule value, stop resolution
    - Source attribution: "Two-Way Rule (Model+Customer) - [Model]+[Company]"
    - Use case: Customer configuration varies by deployed model (I/O settings, networking)
    - **Real-World Example:**
      ```
      Device: i-22, VZW, ATM Plan, Miele
      Rule: IF model=i-22 AND customer=Miele THEN digitalio_config="1,0,0;1,0,0;"
      Result: Uses custom I/O config (Miele's i-22 devices need specific I/O setup)

      Same customer, different model (no I/O hardware):
      Device: IR611, VZW, ATM Plan, Miele
      No rule (IR611 doesn't have I/O hardware)
      Result: Continue to next level (parameter may not apply)
      ```

    **6.6: Service Plan + Customer Rule**
    - Query: WHERE service_plan=device.service_plan AND company=device.company (model_id IS NULL, carrier_id IS NULL)
    - If matching rule found → use rule value, stop resolution
    - Source attribution: "Two-Way Rule (Plan+Customer) - [ServicePlan]+[Company]"
    - Use case: Customer exceptions to plan baseline (when Model+Carrier don't matter)
    - **Real-World Example:**
      ```
      Device: i-22, VZW, ATM Plan, Premium_Customer
      Rule: IF service_plan=ATM AND customer=Premium_Customer THEN traffic_day_threshold=1000MB
      Result: Uses 1000MB (upgraded from standard ATM limit)

      Standard customer, same plan:
      Device: i-22, VZW, ATM Plan, Standard_Customer
      No rule for Standard_Customer
      Result: Continue checking (will find Carrier+Plan or Model+Plan rule)

      Note: This is simpler than Four-Way Rule when Model+Carrier don't affect the exception
      ```

    - If no matching rule found in any sub-priority → continue to next level (Level 7: Carrier Layer)
  - **Multiple Rules Scenario**: If multiple rules at same sub-priority match (shouldn't happen with proper validation), use highest priority (lowest priority number)
  - **Use case**: Parameters where value depends on TWO (and only two) device attributes simultaneously

  **Level 7: Carrier Layer**
  - If value exists at Carrier layer (including NULL) → use it, stop resolution
  - Source attribution: "Carrier Layer - [Carrier Name]"
  - Use case: Carrier network settings (APN, etc.)

  **Level 8: Model Layer**
  - If value exists at Model layer (including NULL) → use it, stop resolution
  - Source attribution: "Model Layer - [Model Name]"
  - Use case: Hardware-specific configurations

  **Level 9: Global Layer**
  - If value exists at Global layer (including NULL) → use it, stop resolution
  - Source attribution: "Global Layer"
  - Use case: System-wide defaults (DNS, time servers, security settings)

  **Level 10: Schema Default**
  - If no value found in any layer/rule, use Default Value from key schema (if set)
  - Source attribution: "Schema Default"
  - Use case: Hardcoded fallback values

  **Level 11: Required Key Validation** *(Lowest Priority)*
  - If still no value and key is Required → ERROR, cannot generate config
  - Log error: "Device {id} config invalid: required key '{name}' has no value"
  - Mark device status as "Config Error"
  - Source attribution: "ERROR - Missing Required Key"

  4. Output: Map of all keys with resolved values, including source attribution
- **Complete 11-Level Priority Order with Sub-Priorities:**
  ```
  1. Device Override (highest)
  2. Four-Way Rule (Model + Carrier + Service Plan + Company)
  3. Company Override
  4. Three-Way Rule (Model + Carrier + Service Plan)
  5. Service Plan Layer
  6. Two-Way Rule (Any 2 Factors - checked in sub-priority order):
     6.1 Carrier + Service Plan
     6.2 Model + Service Plan
     6.3 Model + Carrier
     6.4 Carrier + Customer
     6.5 Model + Customer
     6.6 Service Plan + Customer
  7. Carrier Layer
  8. Model Layer
  9. Global Layer
  10. Schema Default Value
  11. Required Validation (error if missing)
  ```

  **Example Resolution Walkthrough for `traffic_day_threshold` parameter:**
  ```
  Device: i-22 Model, VZW Carrier, ATM Service Plan, CORD Company

  Level 1 (Device Override): No value set → continue
  Level 2 (Four-Way Rule): Check i-22+VZW+ATM+CORD → No rule → continue
  Level 3 (Company Override): Check CORD company layer → No value → continue
  Level 4 (Three-Way Rule): Check i-22+VZW+ATM → No rule → continue
  Level 5 (Service Plan Layer): Check ATM plan → No value → continue
  Level 6 (Two-Way Rule):
    6.1: Check VZW+ATM → FOUND! Rule: traffic_day_threshold=3584MB
    → STOP, use 3584MB

  Source: Two-Way Rule (Carrier+Plan) - VZW+ATM
  Result: traffic_day_threshold = 3584MB
  ```

  **Another Example - Multiple Rules Scenario for same device:**
  ```
  Device: i-22 Model, VZW Carrier, ATM Service Plan, CORD Company
  Parameter: traffic_day_threshold

  Available Rules:
  - Carrier+Plan: VZW+ATM → 3584MB (sub-priority 6.1)
  - Model+Plan: i-22+ATM → 350MB (sub-priority 6.2)
  - Model+Carrier: i-22+VZW → 4000MB (sub-priority 6.3)

  Resolution Process at Level 6:
    6.1: Check VZW+ATM → FOUND 3584MB → USE THIS (first match wins)
    (6.2 and 6.3 are not checked because 6.1 matched)

  Result: 3584MB from Carrier+Plan rule
  Reason: VZW's carrier policy for ATM plan is checked first
  ```
- **NULL Handling:**
  - If layer has explicit NULL (row exists with NULL value) → that's the resolved value (don't continue walking)
  - If layer has no row for key (not set) → continue walking to next priority level
  - Rules can also specify NULL value to explicitly disable parameter for specific combinations
- **Rule Matching Logic:**
  - **Two-way rule**: Must match EXACTLY TWO device attributes (any combination):
    - Carrier + Service Plan: WHERE carrier=device.carrier AND service_plan=device.service_plan (model_id IS NULL, company_id IS NULL)
    - Model + Service Plan: WHERE model=device.model AND service_plan=device.service_plan (carrier_id IS NULL, company_id IS NULL)
    - Model + Carrier: WHERE model=device.model AND carrier=device.carrier (service_plan_id IS NULL, company_id IS NULL)
    - Carrier + Customer: WHERE carrier=device.carrier AND company=device.company (model_id IS NULL, service_plan_id IS NULL)
    - Model + Customer: WHERE model=device.model AND company=device.company (carrier_id IS NULL, service_plan_id IS NULL)
    - Service Plan + Customer: WHERE service_plan=device.service_plan AND company=device.company (model_id IS NULL, carrier_id IS NULL)
  - **Three-way rule**: Must match device.model_id AND device.carrier_id AND device.service_plan_id exactly (company_id IS NULL)
  - **Four-way rule**: Must match device.model_id AND device.carrier_id AND device.service_plan_id AND device.company_id exactly
  - **Important**: Partial matches do not count (all specified factors must match, unspecified factors must be NULL)
  - **Rule precedence by priority level**: Four-Way (Level 2) > Three-Way (Level 4) > Two-Way Level 6 (with sub-priorities 6.1-6.6)
  - Multiple rule types can exist for same parameter; higher-level rules checked first per priority hierarchy
  - **Within Level 6**: Check sub-priorities sequentially (6.1 → 6.2 → 6.3 → 6.4 → 6.5 → 6.6); first match wins
- **Performance Requirements:**
  - Resolution must complete in <2 seconds per device (including all conditional rule lookups)
  - Use database indexes on:
    - **Two-Way Rules** (6 indexes for different combinations):
      - (carrier_id, service_plan_id) WHERE model_id IS NULL AND company_id IS NULL
      - (model_id, service_plan_id) WHERE carrier_id IS NULL AND company_id IS NULL
      - (model_id, carrier_id) WHERE service_plan_id IS NULL AND company_id IS NULL
      - (carrier_id, company_id) WHERE model_id IS NULL AND service_plan_id IS NULL
      - (model_id, company_id) WHERE carrier_id IS NULL AND service_plan_id IS NULL
      - (service_plan_id, company_id) WHERE model_id IS NULL AND carrier_id IS NULL
    - (model_id, carrier_id, service_plan_id) for Three-Way Rules
    - (model_id, carrier_id, service_plan_id, company_id) for Four-Way Rules
  - Consider caching rule lookups for parameters with rules (affects <15% of parameters)
  - Batch resolution for 1000 devices must complete in <30 seconds

**FR-3.2: Config Versioning**
- System must generate unique version identifier for effective config
- Version format options (TBD with engineering):
  - **Option A**: Hash of all resolved key-value pairs (deterministic, same config = same hash)
  - **Option B**: Timestamp-based (when config was last modified at any layer affecting this device)
  - **Option C**: Composite: "globalV.modelV.carrierV.planV.companyV.deviceV" (tracks each layer version)
- Recommended: Option A (hash-based) for simplicity and determinism
- Version must be stored with device: `expected_version` column
- Device reports `current_version` when checking in
- If `current_version != expected_version`, device needs config update

**FR-3.3: Required Key Validation**
- During config resolution, system must validate all required keys have values
- If required key is NULL after walking all layers (including default), system must:
  - Log error: "Device 12345 config is invalid: required key 'dns_primary' has no value"
  - Mark device status as "Config Error"
  - Alert admin via dashboard: "3 devices have invalid configs due to missing required keys"
  - Do not generate config file (fail safe)
- Admin must fix by setting required key value at appropriate layer

**FR-3.4: Config File Generation**
- System must format effective config into file format expected by devices
- Output format: Key-value pairs in InHand firmware format (TBD with engineering)
- All 600+ keys must be included in output (even if NULL or not applicable to model)
- NULL values must be represented appropriately (e.g., empty string, "0", or omitted depending on key - TBD per key)
- File must include metadata: version identifier, generation timestamp
- Generated file must be identical to current file format so devices can parse it

**FR-3.5: Batch Config Generation**
- System must support batch generation of configs for multiple devices
- Use case: Generate configs for all devices in company, or all devices on new system
- Must be performant: 1000 devices in <30 seconds
- Batch generation may be async (background job) if >500 devices
- Progress tracking for batch jobs

---

### FR-3a: Conditional Rules Management

**FR-3a.1: Conditional Rule Creation**
- System must provide admin interface to create conditional rules for any parameter
- **Rule Creation Workflow:**
  1. Select config key from list (filter by: has conditional rules, available keys)
  2. Choose rule type: Two-Way (any 2 factors) OR Three-Way (Model + Carrier + Service Plan) OR Four-Way (Model + Carrier + Service Plan + Company)
  3. **For Two-Way Rules:**
     - **Step 1: Select Factor Combination** (which two factors determine the value?)
       - ○ Model + Carrier (carrier-specific features per model)
       - ○ Carrier + Service Plan (carrier data policies by plan)
       - ○ Model + Service Plan (model pricing tier by plan)
       - ○ Carrier + Customer (customer carrier agreements)
       - ○ Model + Customer (customer model configurations)
       - ○ Service Plan + Customer (customer plan exceptions)
     - **Step 2: Select Factor Values** (dropdowns shown based on combination selected):
       - **If Model + Carrier selected:**
         - Select Model from dropdown (show all active models: i-22, i-52, 4500, IR611, etc.)
         - Select Carrier from dropdown (show all active carriers: VZW, ATT, TMO, DC, etc.)
         - Example: Model=i-22, Carrier=VZW → `mqtt_enable=1`
       - **If Carrier + Service Plan selected:**
         - Select Carrier from dropdown
         - Select Service Plan from dropdown (show all active plans: ATM, Tier1, Tier2, etc.)
         - Example: Carrier=VZW, ServicePlan=ATM → `traffic_day_threshold=3584MB`
       - **If Model + Service Plan selected:**
         - Select Model from dropdown
         - Select Service Plan from dropdown
         - Example: Model=i-22, ServicePlan=ATM → `traffic_day_threshold=350MB`
       - **If Carrier + Customer selected:**
         - Select Carrier from dropdown
         - Select Company from dropdown (show all active companies)
         - Example: Carrier=VZW, Company=VIP_Corp → `traffic_day_threshold=10GB`
       - **If Model + Customer selected:**
         - Select Model from dropdown
         - Select Company from dropdown
         - Example: Model=i-22, Company=Miele → `digitalio_config=[custom I/O]`
       - **If Service Plan + Customer selected:**
         - Select Service Plan from dropdown
         - Select Company from dropdown
         - Example: ServicePlan=ATM, Company=Premium_Customer → `traffic_day_threshold=1000MB`
     - **Step 3: Enter Value**
       - Enter value for this combination (validate against key's data type)
       - Show preview: "When device has [Factor1]=[Value1] AND [Factor2]=[Value2], parameter will be set to [entered value]"
     - **Step 4: Set Priority**
       - Sub-priority is automatically assigned based on combination type:
         - 1 = Carrier + Service Plan (highest within Two-Way)
         - 2 = Model + Service Plan
         - 3 = Model + Carrier
         - 4 = Carrier + Customer
         - 5 = Model + Customer
         - 6 = Service Plan + Customer (lowest within Two-Way)
       - Admin can manually override if needed (Advanced option)
     - **Step 5: Add Description**
       - Add optional description/business justification
       - Recommended format: "Why this combination needs this value"
  4. **For Three-Way Rules:**
     - Select Model from dropdown
     - Select Carrier from dropdown
     - Select Service Plan from dropdown
     - Enter value for this combination (validate against key's data type)
     - Set priority (default: 100)
     - Add optional description/business justification
  5. **For Four-Way Rules:**
     - Select Model from dropdown
     - Select Carrier from dropdown
     - Select Service Plan from dropdown
     - **Select Company from dropdown (REQUIRED):**
       - Four-Way Rules are for customer-specific exceptions only
       - Must select a specific company; creates exception rule applying ONLY to that customer
       - **Note**: For general baselines that apply to all customers, use Three-Way Rules instead
     - Enter value for this combination (validate against key's data type)
     - Set priority (default: 100)
     - Add optional description/business justification
     - System checks if corresponding Three-Way Rule baseline exists and warns if missing
  6. Preview affected devices: "This customer exception will affect X devices at [Company Name]"
  7. Save rule (with validation checks)
- **Validation on Save:**
  - Check for conflicting rules (same factors, different values)
  - Verify all selected factors exist in database
  - Validate value matches key's data type and validation rules
  - Ensure priority is unique for this key (or warn if duplicate)
  - Check if combination is possible (e.g., Model 22 doesn't work with Carrier X)
  - **Four-Way Rule Validation:**
    - Warn if no corresponding Three-Way Rule baseline exists for this Model+Carrier+ServicePlan combination
    - Cannot create duplicate rules (same Model+Carrier+Plan+Company combination)
    - Customer exception takes precedence over Company layer value (warn admin)
- **Success/Error Feedback:**
  - Success (2-way/3-way): "Rule created. Will affect X devices on next config regeneration."
  - Success (4-way): "Customer-specific exception created for [Company Name]. Will affect X devices."
  - Error: "Conflict detected: Rule already exists for Model 22 + VZW with value '0'"
  - Warning: "No devices currently match this combination"
  - Warning (4-way): "No Three-Way Rule baseline exists for this Model+Carrier+ServicePlan combination. Consider creating the baseline rule first."

**FR-3a.2: Conditional Rule Listing & Search**
- System must provide interface to view all conditional rules
- **List View Features:**
  - Filter by: Rule type (2-way, 3-way, 4-way), Parameter name, Model, Carrier, Service Plan, Company
  - **Four-Way Specific Filters:**
    - Filter by specific company name
    - "Show all customer exceptions" (default view for Four-Way Rules)
  - Sort by: Parameter name, Priority, Created date, Affected devices count, Company name
  - Show columns: Parameter, Rule Type, Conditions (Model/Carrier/Plan/Company), Value, Priority, Affected Devices, Last Modified
  - **Four-Way Display:**
    - All Four-Way Rules show the specific company name (customer exceptions only)
    - Company column shows: "[Company Name] ⚠️ Exception"
    - Hierarchical view option: Group by Model+Carrier+ServicePlan showing Three-Way baseline rule (if exists) with Four-Way customer exceptions listed below
  - Pagination (50 rules per page default)
  - Bulk actions: Delete selected, Change priority, Clone rule for different company
- **Rule Detail View:**
  - Show full rule conditions and value
  - Show affected devices count with link to device list
  - Show created by user and timestamp
  - Show last modified by user and timestamp
  - Show description/business justification
  - Show source attribution: which layer would apply if rule didn't exist
  - Actions: Edit, Delete, Duplicate, View Conflicts

**FR-3a.3: Conditional Rule Editing**
- Admin must be able to edit existing conditional rules
- **Editable Fields:**
  - Value (must validate against key's data type)
  - Priority (must be numeric)
  - Description/business justification
- **Non-Editable Fields:**
  - Rule factors (Model, Carrier, Service Plan) - cannot change after creation
  - Parameter name - cannot change after creation
  - Reason: Changing factors would affect different devices, better to delete and recreate
- **Edit Workflow:**
  1. Select rule to edit
  2. Update editable fields
  3. Preview changes: "This will affect X devices. Previous value was 'Y', new value is 'Z'"
  4. Confirm change
  5. System updates expected_version for affected devices
- **Validation on Edit:**
  - Same validation as creation
  - Check if new value conflicts with other rules

**FR-3a.4: Conditional Rule Deletion**
- Admin must be able to delete conditional rules
- **Delete Workflow:**
  1. Select rule(s) to delete
  2. Show confirmation: "Delete rule for Model 22 + VZW? This affects X devices."
  3. Show fallback behavior: "These devices will fall back to [Layer] value or [default]"
  4. Confirm deletion
  5. System updates expected_version for affected devices
- **Bulk Delete:**
  - Support deleting multiple rules at once
  - Show combined impact: "Deleting 5 rules affecting X total devices"
- **Cascade Considerations:**
  - Deleting rule does NOT delete parameter values at other layers
  - Devices fall back to standard layer resolution (Carrier → Model → Global)
  - Audit log records deletion with user and reason

**FR-3a.5: Rule Coverage & Conflict Detection**
- System must show rule coverage for parameters with conditional rules
- **Coverage Report:**
  - For parameters flagged with conditional rules, show matrix of possible combinations
  - **Two-way**: 6 combination types possible (coverage report shows relevant type based on rules created):
    - Model+Carrier: M models × N carriers = M*N combinations
    - Carrier+ServicePlan: N carriers × P plans = N*P combinations
    - Model+ServicePlan: M models × P plans = M*P combinations
    - Carrier+Customer: N carriers × C customers = N*C combinations
    - Model+Customer: M models × C customers = M*C combinations
    - ServicePlan+Customer: P plans × C customers = P*C combinations
  - **Three-way**: M models × N carriers × P plans = M*N*P combinations
  - **Four-way**: M models × N carriers × P plans × (1 baseline + C customer exceptions)
  - Highlight which combinations have rules (green) vs fall back to layers (gray)
  - Show coverage percentage: "traffic_day_threshold has rules for 12 of 20 Carrier+Plan combinations (60%)"
  - **Four-Way Coverage:**
    - "fw_acl has 8 baseline rules + 3 customer exceptions"
    - Show hierarchical coverage: "Model 22 + ATT + ATM: Baseline (all) + 1 exception (CORD)"
    - List companies with exceptions: "Customer exceptions: ALTECH, BAUM, CORD"
- **Conflict Detection:**
  - Detect if multiple rules could match same device
  - Example conflict: Two rules with same Model+Carrier but different Service Plans (shouldn't happen)
  - Show warning: "Conflict detected: 2 rules match same conditions"
  - Recommend resolution: Use priority to disambiguate or delete conflicting rule
  - **Four-Way Conflicts:**
    - Detect customer with both Company layer value AND customer-specific rule (rule wins)
    - Warning: "CORD has Company layer value AND customer-specific rule. Rule takes precedence."
- **Gap Analysis:**
  - Show combinations with no rule and no layer value
  - Warning: "Model 4500 + Carrier DC + Plan tier1 has no value (rule or layer)"
  - Recommend: Add rule or set value at appropriate layer
  - **Four-Way Gaps:**
    - Warning: "Four-Way customer exception exists without Three-Way Rule baseline"
    - Recommend: "Create Three-Way Rule baseline for Model 22 + ATT + ATM first"

**FR-3a.6: Rule Priority Management**
- For parameters with multiple rules, system must handle priority
- **Priority Rules:**
  - Lower priority number = higher priority (1 is highest, 100 is default)
  - If multiple rules match same device (shouldn't happen), use highest priority rule
  - Priority is unique per parameter per rule type (can have same priority across different parameters)
- **Priority Interface:**
  - Show rules for parameter sorted by priority
  - Drag-and-drop to reorder priorities
  - Bulk update priorities: "Set all to 100"
  - Warning if multiple rules have same priority for same parameter
- **Use Cases:**
  - Priority mostly irrelevant if rules are mutually exclusive (different model/carrier combos)
  - Priority matters if rules overlap (e.g., wildcard support in future)
  - Currently, with exact matching, priority is tie-breaker for conflicts only

**FR-3a.7: Rule Matrix Visualization**
- System must provide visual matrix interface for rule management
- **Two-Way Rule Matrix:**
  - Rows: Models, Columns: Carriers
  - Cell value: Rule value for that combination (or "inherit" if no rule)
  - Click cell to edit rule value
  - Color coding: Green (rule exists), Gray (inherits from layer), Red (no value anywhere)
  - Show affected devices count per cell
- **Three-Way Rule Matrix:**
  - Tabs for each Service Plan
  - Each tab shows Model × Carrier matrix
  - Quick comparison across service plans
  - Bulk operations: "Copy all rules from tier1 to ATM plan"
- **Four-Way Rule Matrix:**
  - **Primary View**: Tabs for each Service Plan, showing Model × Carrier matrix (baseline rules only)
  - **Customer Exception View**:
    - Toggle button: "Show Customer Exceptions"
    - When enabled, cells with customer exceptions show indicator: "⚠️ 3 exceptions"
    - Click cell to expand and see list of customer-specific rules
  - **Hierarchical Table View** (alternative to matrix):
    - Rows organized by Model → Carrier → Service Plan
    - Each row shows baseline value with expandable section for customer exceptions
    - Indented list of customer exceptions under each baseline
    - Visual indicator: Baseline (bold) vs Exception (indented with company name)
  - **Customer-Specific View:**
    - Filter: "Show rules for company: [Dropdown]"
    - Highlights cells where selected company has exceptions
    - Shows both baseline value and company-specific override side-by-side
  - **Bulk Operations:**
    - "Create customer exception from baseline" - Clone baseline for specific company
    - "Apply to multiple customers" - Create same exception for multiple companies
    - "Remove all exceptions for company" - Delete all customer-specific rules for one company
- **Matrix Export:**
  - Export to CSV for documentation
  - Include: Parameter, Model, Carrier, Plan, Company (NULL=Baseline or Company Name), Value, Affected Devices
  - Separate sheets for baselines and exceptions
  - Use for stakeholder reviews and audits

**FR-3a.8: Rule Business Logic Documentation**
- System must support documenting business logic for conditional rules
- **Documentation Fields:**
  - Description: Short summary of why rule exists
  - Business Justification: Why this specific value for this combination
  - Related Ticket/Request: Link to ticket that requested this rule
  - Owner: Team/person responsible for maintaining this rule
  - Review Date: When rule should be reviewed for continued relevance
- **Documentation Use Cases:**
  - Audit trail for compliance
  - Knowledge transfer when team members change
  - Understanding why configurations differ across combinations
  - Identifying rules that may be outdated or no longer needed

---

### FR-4: Device Effective Config View

**FR-4.1: Effective Config Display**
- System must provide UI to view final effective config for any device
- Accessible from device detail page or direct link
- Display all 600+ keys with resolved values
- Keys organized in logical groups (same grouping as Global layer config)
- Each key shows:
  - Key name
  - Resolved value
  - Source attribution: "(Global)" or "(Model: i-22)" or "(Company)" or "(Device Override)"
  - Visual indicator by source (color coding or icon)
- Example:
  ```
  dns_primary: 8.8.8.8 (Global) [green]
  alarm_io_1: 1 (Model: i-22) [light blue]
  reboot_schedule_enabled: true (Company) [orange]
  lan_ip: 10.1.50.100 (Device Override) [blue]
  reboot_schedule_time: [Not Set] (NULL) [gray]
  ```

**FR-4.2: Source Attribution Detail**
- Clicking on source attribution label must show detail popup:
  - Layer: Company
  - Value set by: user@example.com
  - Value set at: 2026-01-15 14:32:05
  - Previous value: [Not Set]
- Provides full transparency for troubleshooting

**FR-4.3: Filter & Search**
- User must be able to filter/search keys within effective config view
- Filter by: Source layer (show only Global keys, only Device overrides, etc.)
- Filter by: Key name (partial match search)
- Filter by: Value status (Inherited, Overridden, Not Set/NULL)
- Search must update display in real-time

**FR-4.4: Export Effective Config**
- User must be able to export effective config for troubleshooting
- Export formats: JSON, CSV, or device file format
- JSON format must include source attribution for each key
- Device file format must match what device receives (for comparison)
- Export must be logged in audit trail

**FR-4.5: Access Control for Effective Config View**
- Admin/Operations: Can view all keys with full source attribution
- Customer: Can view only customer-visible keys (customer-configurable keys + potentially others flagged as "viewable")
  - May see values but not always source attribution (simplified view)
  - Cannot see system keys (DNS, time servers, firmware settings, etc.)
- Customer view must be explicitly simplified to avoid confusion

---

### FR-5: Validation & Safety Systems

**FR-5.1: Data Type Validation**
- System must validate all values match key's data type before saving
- **String**: Any text allowed unless regex pattern specified
- **Integer**: Must be numeric, must be within min/max range if specified
- **Boolean**: Must be true/false or 1/0 or on/off (normalized to boolean)
- **IP Address**: Must be valid IPv4 format (xxx.xxx.xxx.xxx where xxx = 0-255)
- **Time**: Must be valid 24-hour format HH:MM (HH = 00-23, MM = 00-59)
- **Enum**: Must be one of allowed values (dropdown prevents invalid values)
- Validation must occur on client-side (immediate feedback) and server-side (security)
- Invalid values must prevent save with clear error message

**FR-5.2: Custom Validation Rules**
- System must enforce custom validation rules defined in key schema
- **Regex validation**: If key has regex pattern, value must match
- **Range validation**: If key has min/max, value must be in range
- **IP range validation**: For LAN IP config, customer must only use allowed ranges:
  - 192.168.0.0 - 192.168.255.255
  - 10.0.0.0 - 10.255.255.255
  - 172.16.0.0 - 172.31.255.255 (if allowed)
  - Block: 0.0.0.0, 127.x.x.x, 224.x.x.x and higher (reserved ranges)
- Validation failure must show specific message: "LAN IP must be in private address range (192.168.x.x or 10.x.x.x)"

**FR-5.3: Permission Validation**
- System must enforce access control on all config operations
- **Admin**:
  - Can access all layers, all companies, all devices
  - Can modify all keys at all layers
  - Can access key schema management
  - Can access migration tools
- **Operations**:
  - Can view all layers (read-only)
  - Can view all companies/devices
  - Can modify Device layer only
  - Cannot access key schema management
  - Cannot access migration tools
- **Customer**:
  - Can view/modify only own Company and Devices
  - Can modify only keys marked "Customer Configurable"
  - Cannot access other layers (completely hidden)
  - Cannot access key schema
  - Cannot access migration tools
  - Cannot download raw config files
- Unauthorized access attempts must be logged and blocked with 403 Forbidden

**FR-5.4: Preview & Confirmation**
- System must show preview before applying config changes
- Preview must include:
  - Number of devices affected
  - List of affected device IDs (first 100 shown, "and X more" if over 100)
  - Breakdown by model/carrier if applicable
  - Summary of changes: "3 keys modified, 1 key cleared"
- Confirmation required for changes affecting:
  - >10 devices: Standard confirmation dialog
  - >1000 devices: Double confirmation ("Type 'CONFIRM' to proceed")
  - Global layer: Always require confirmation
- User must be able to cancel at confirmation stage

**FR-5.5: Inline Validation Feedback**
- System must provide immediate feedback as user enters values
- Red border on input field if invalid
- Error message below field: "Must be valid IPv4 address" or "Must be between 0 and 23"
- Green checkmark icon if valid
- "Save" button must be disabled while any validation errors exist
- Validation summary at top of page: "2 errors must be fixed before saving"

---

### FR-6: Change Detection & Versioning

**FR-6.1: Change Detection**
- System must detect when any layer config changes
- On save of any layer config (Global, Model, Carrier, Service Plan, Company, Device):
  1. Calculate which devices are affected
  2. For each affected device, recalculate effective config
  3. Generate new version identifier
  4. Update device.expected_version in database
  5. Log change in audit trail
- Change detection must be synchronous (complete before save returns to user)

**FR-6.2: Affected Device Calculation**
- System must identify affected devices based on which layer changed:
  - **Global**: All devices in system
  - **Model**: All devices of that model (filter devices where device.model_id = changed_model_id)
  - **Carrier**: All devices on that carrier (filter where device.carrier_id = changed_carrier_id)
  - **Service Plan**: All devices on that plan (filter where device.service_plan_id = changed_plan_id)
  - **Company**: All devices in that company (filter where device.company_id = changed_company_id)
  - **Device**: Only that specific device (device_id = changed_device_id)
- Calculation must be efficient (database query with indexes)
- For Global changes affecting 100K+ devices, may need background job (TBD with performance testing)

**FR-6.3: Version Identifier Generation**
- System must generate unique version identifier per device
- Generation method: Hash of effective config key-value pairs (recommended)
  - Example: SHA256 hash of JSON serialization of config
  - Deterministic: Same config always produces same hash
  - Collision probability: negligible with SHA256
- Alternative: Timestamp-based UUID (less deterministic but simpler)
- Version identifier must be stored: device.expected_version
- Device reports version on check-in: device.current_version

**FR-6.4: Integration with Device Check-In**
- When device checks in (existing UDP check-in process):
  1. Device sends current_version (version of config currently applied)
  2. System compares current_version to expected_version
  3. If different: device needs config update
     - System generates effective config (or retrieves cached config)
     - System sends config to device via existing mechanism
     - Device applies config, reboots if necessary
     - Device sends new check-in with updated current_version
  4. If same: no action needed
- Integration must not break existing check-in process
- Must support both old system devices (use hostname validation) and new system devices (use version comparison)

**FR-6.5: Config Push Triggers**
- Config push must be triggered by:
  - **Check-in version mismatch** (primary mechanism)
  - **Manual push**: Admin can manually trigger config push to specific devices
  - **Service plan change**: If device service plan changes, recalculate config and push
  - **Wi-Fi update**: If customer updates Wi-Fi settings (existing feature), trigger config push
  - **Carrier change**: If device carrier changes, recalculate config and push
- All triggers must update expected_version and initiate push on next check-in

---

### FR-7: Migration & Parallel Operation

**FR-7.1: Device Config System Flag**
- System must add flag to device record: `config_system` enum ('old', 'new')
- Default for all existing devices: 'old'
- Flag determines which system generates config for device
- Flag must be visible and editable in device admin interface

**FR-7.2: Device Migration Selection**
- System must provide admin interface to select devices for migration
- Selection methods:
  - **By Device ID**: Enter comma-separated list or upload CSV (device_id,device_id,...)
  - **By Device ID Range**: Enter start and end ID (e.g., 1-1000)
  - **By Search/Filter**: Use device list filters (model, carrier, company, status), select all results
- Selection interface must show preview: "23 devices selected for migration"
- Confirm before migrating: "Migrate these 23 devices to new config system?"

**FR-7.3: Migration Process**
- On migration confirmation:
  1. For each selected device:
     - Verify device is on 'old' system (if already 'new', skip with warning)
     - Calculate effective config using new system
     - Generate version identifier
     - Update device.config_system = 'new'
     - Update device.expected_version = calculated_version
     - Log migration event in audit trail
  2. Show success summary: "23 devices migrated successfully"
- Migration must be atomic per device (if error, rollback that device only, continue with others)
- Migration status must be logged: migrated_at timestamp, migrated_by user

**FR-7.4: Parallel Config Generation**
- System must support both old and new config systems running simultaneously
- Old system:
  - Continues to generate configs from existing config files
  - Uses hostname validation for version control
  - Applies to devices where config_system = 'old'
- New system:
  - Generates configs from layer hierarchy
  - Uses version identifier for version control
  - Applies to devices where config_system = 'new'
- No interference between systems (separate code paths)
- Check-in process must route to correct system based on device flag

**FR-7.5: Rollback Capability**
- Admin must be able to rollback device to old config system
- Rollback process:
  1. Select device(s) to rollback (same selection methods as migration)
  2. Confirm rollback: "Rollback 5 devices to old config system?"
  3. For each device:
     - Update device.config_system = 'old'
     - Clear device.expected_version (or set to NULL)
     - Device will use old config on next check-in
     - Log rollback event in audit trail
  4. Show success summary
- Rollback must not delete layer configs (data preserved if device re-migrates later)
- Use case: If new system has issues, quickly rollback affected devices

**FR-7.6: Migration Status Dashboard**
- System must provide dashboard showing migration progress
- Display:
  - Total devices: 100,000
  - On old system: 99,000 (99%)
  - On new system: 1,000 (1%)
  - Migration progress chart (over time)
  - Recent migrations: list of last 50 migrations with timestamp
  - Devices with config errors: X devices cannot generate valid config (alert)
- Admin must be able to view list of devices on each system
- Export list of device IDs for external analysis

---

### FR-8: Customer Self-Service

**FR-8.1: Customer-Configurable Keys**
- Customer self-service UI must show ONLY keys where:
  - Key.customer_configurable = true
  - AND Key is available at Company or Device layer
- All other keys must be completely hidden (not shown as read-only)
- Customer must never see system keys (DNS, time servers, firmware settings, security settings, etc.)

**FR-8.2: Company-Level Portfolio Settings**
- Customer must be able to configure company-level settings for their entire fleet
- Show only company-level customer-configurable keys
- For each key, show:
  - Key name (friendly label, not technical name if possible)
  - Description/help text
  - Current value (editable)
  - Inherited value (if not overridden, shown as grayed text "System default: ...")
- "Apply to All Devices" button (primary action)
- On click:
  - Validate all inputs
  - Show confirmation: "Apply these settings to all 500 of your devices?"
  - If confirmed, save to Company layer
  - Show success: "Settings applied. Your devices will update on next check-in."

**FR-8.3: Device-Level Settings**
- Customer must be able to configure individual device settings
- Device selection: Browse customer's devices, search by device name/ID
- Show only device-level customer-configurable keys
- For each key, show:
  - Key name and description
  - Current value (editable)
  - Inherited value from company or system (shown as "Currently: ..." if different)
- "Apply to This Device" button
- On click:
  - Validate inputs
  - Show confirmation: "Apply these settings to Device 12345?"
  - If confirmed, save to Device layer
  - Show success: "Settings applied to Device 12345."

**FR-8.4: UI-Driven Safe Configuration**
- Customer UI must use safe input controls (no free-form text where risky):
  - **Boolean (e.g., Enable Reboot Schedule)**: Toggle switch (On/Off) or radio buttons (Yes/No)
  - **Time (e.g., Reboot Time)**: Time picker with hour and minute dropdowns (00-23, 00-59)
  - **IP Address (e.g., LAN IP)**: Four separate input fields for octets (0-255 each)
  - **Enum (e.g., Reboot Frequency)**: Dropdown with predefined options (Daily, Weekly, Monthly)
  - **Integer in Range (e.g., DHCP Pool Size)**: Number input with min/max, increment/decrement buttons
  - **String (e.g., Device Name)**: Text input with validation (alphanumeric, max length)
- All inputs must have inline validation with immediate feedback
- Invalid inputs must prevent form submission
- Help text/tooltips must explain each setting

**FR-8.5: Preview & Confirmation**
- Before applying any customer config changes, show preview:
  - Company-level: "This will affect 500 devices. Changes will apply on next device check-in (typically within 15 minutes to 1 hour)."
  - Device-level: "This will affect Device 12345. Changes will apply on next check-in."
- Confirmation dialog must be clear and non-technical
- After save, show success message with next steps:
  - "Configuration updated successfully!"
  - "Your devices will receive these settings on their next check-in."
  - "Most devices check in every 15 minutes."
  - "You can view device status on the Devices page."

**FR-8.6: Customer Cannot Break Devices**
- System must enforce safety constraints per Adam's requirement: "they will take themselves out of business and blame us"
- Safety measures:
  - Cannot set invalid values (validation prevents it)
  - Cannot access system-critical parameters (DNS, firmware settings, etc.)
  - Cannot download or view raw config files
  - Cannot see config layers or resolution logic (abstracted away)
  - Cannot configure device into non-functional state
  - LAN IP validation prevents IP conflicts and reserved ranges
  - Reboot schedule validation prevents invalid times
- If customer misconfigures (e.g., wrong LAN IP), worst case: they contact support, admin fixes at Device layer or rolls back to company default
- Admin must be able to override or clear any customer settings

---

### FR-9: Access Control & Permissions

**FR-9.1: Role-Based Access Control**
- System must enforce role-based permissions:
  - **System Admin**: Full access (all layers, all companies, all devices, schema management, migration tools)
  - **Operations**: Read all, write Device layer only (for support)
  - **Customer Admin**: Read/write own Company and Devices only, customer-configurable keys only
  - **Sales/BD** (future): Read-only demo environment
- Roles must be assigned via existing user management system
- Permission checks must occur on all config operations (read and write)

**FR-9.2: Data Isolation for Customers**
- Customers must only see own company and devices
- System must filter all queries by company_id for customer users
- Customers must never see other customers' data (even read-only)
- API endpoints (future) must enforce same filtering
- URL manipulation (changing company_id in URL) must not bypass security

**FR-9.3: Layer Access Restrictions**
- Customers cannot access or view:
  - Global layer
  - Model layer
  - Carrier layer
  - Service Plan layer
  - Other companies' Company layer
  - Other companies' Device layer
- Customers can access:
  - Own Company layer (customer-configurable keys only)
  - Own Devices' Device layer (customer-configurable keys only)
- Attempts to access restricted layers must return 403 Forbidden

**FR-9.4: Key Access Restrictions**
- For customers, system must filter keys by customer_configurable flag
- If key.customer_configurable = false:
  - Customer cannot see key (not in UI at all)
  - Customer cannot set value (API blocks if future)
  - Customer cannot read value (even read-only)
- Only show keys customer can actually configure

**FR-9.5: Audit Trail for Customer Actions**
- All customer config changes must be logged in audit trail:
  - User (customer email)
  - Company
  - Device (if device-level change)
  - Key name
  - Old value
  - New value
  - Timestamp
  - IP address
- Audit trail must be accessible to admins for troubleshooting and compliance
- Customers may view their own audit trail (optional, TBD)

---

### FR-10: Data Model & Storage

**FR-10.1: Config Key Schema Table**
- Table: `config_keys`
- Columns:
  - `id` (primary key)
  - `name` (string, unique, indexed)
  - `data_type` (enum: string, integer, boolean, ip_address, time, enum)
  - `description` (text, nullable)
  - `validation_rules` (JSON, nullable) - stores regex, min/max, enum values, etc.
  - `available_at_global` (boolean, default true)
  - `available_at_model` (boolean, default false)
  - `available_at_carrier` (boolean, default false)
  - `available_at_service_plan` (boolean, default false)
  - `available_at_company` (boolean, default false)
  - `available_at_device` (boolean, default false)
  - `is_required` (boolean, default false)
  - `is_customer_configurable` (boolean, default false)
  - `has_two_way_rules` (boolean, default false) - NEW: parameter uses Model + Carrier conditional rules
  - `has_three_way_rules` (boolean, default false) - NEW: parameter uses Model + Carrier + Service Plan conditional rules
  - `has_four_way_rules` (boolean, default false) - NEW: parameter uses Model + Carrier + Service Plan + Company conditional rules (customer exceptions)
  - `default_value` (string, nullable) - stored as string, cast to data_type on use
  - `created_at`, `updated_at` (timestamps)
- Indexes: `name` (unique), `is_required`, `is_customer_configurable`, `has_two_way_rules`, `has_three_way_rules`, `has_four_way_rules`
- Note: Parameters can have multiple conditional rule flags enabled (e.g., both `has_three_way_rules` and `has_four_way_rules` = true for baseline + customer exceptions)
- Precedence: Four-Way Rules (Level 2) > Three-Way Rules (Level 4) > Two-Way Rules (Level 6) in the 11-level priority hierarchy

**FR-10.2: Layer Config Value Tables**
- Separate tables for each layer:
  - `config_global` - Stores Global layer values
  - `config_model` - Stores Model layer values (per model_id)
  - `config_carrier` - Stores Carrier layer values (per carrier_id)
  - `config_service_plan` - Stores Service Plan layer values (per plan_id)
  - `config_company` - Stores Company layer values (per company_id)
  - `config_device` - Stores Device layer values (per device_id)

**FR-10.3: Layer Table Schema (Pattern for all layer tables)**
- Example for `config_model`:
- Columns:
  - `id` (primary key)
  - `model_id` (foreign key to models table, indexed) - NULL for Global layer table
  - `config_key_id` (foreign key to config_keys table, indexed)
  - `value` (text, nullable) - NULL means "explicitly cleared", no row means "not set"
  - `set_by_user_id` (foreign key to users table, nullable) - who set this value
  - `set_at` (timestamp) - when value was set
  - `updated_at` (timestamp) - when value was last updated
- Composite unique index: (model_id, config_key_id) - one value per key per model
- Note: For Global layer table, model_id column doesn't exist (or is always NULL)

**FR-10.4: "Not Set" vs "Explicitly Cleared" Logic**
- **No row in table** = "not set" → inherit from higher layer during config resolution
- **Row exists with NULL value** = "explicitly cleared" → use NULL, do not inherit
- This distinction allows customers/admins to clear inherited values
- Example:
  - Global: reboot_schedule_time = "02:00"
  - Company: (no row) → inherits "02:00"
  - Device A: (no row) → inherits "02:00" (from Global via Company)
  - Device B: (row with value = NULL) → explicitly cleared → NULL (does not inherit)
  - Device C: (row with value = "05:00") → overridden → "05:00"

**FR-10.5: Device Table Updates**
- Add columns to existing `devices` table:
  - `config_system` (enum: 'old', 'new', default 'old') - which config system device uses
  - `expected_version` (string, nullable) - version identifier device should have
  - `current_version` (string, nullable) - version identifier device reports
  - `config_error` (boolean, default false) - flag if device config is invalid (missing required keys)
  - `config_error_message` (text, nullable) - details of config error
  - `config_migrated_at` (timestamp, nullable) - when device was migrated to new system
  - `config_migrated_by` (foreign key to users, nullable) - who migrated device
- Indexes: `config_system`, `expected_version`, `current_version`, `config_error`

**FR-10.6: Two-Way Conditional Rules Table**
- Table: `config_conditional_rules_2way` - NEW
- Purpose: Store conditional value rules for parameters that depend on Model AND Carrier
- Columns:
  - `id` (primary key)
  - `config_key_id` (foreign key to config_keys, indexed, NOT NULL)
  - `model_id` (foreign key to models table, indexed, NOT NULL)
  - `carrier_id` (foreign key to carriers table, indexed, NOT NULL)
  - `value` (text, nullable) - the value to use when conditions match; NULL = explicitly disable parameter
  - `priority` (integer, default 100, NOT NULL) - for tie-breaking if multiple rules match (lower = higher priority)
  - `description` (text, nullable) - business justification for this rule
  - `created_by_user_id` (foreign key to users, NOT NULL)
  - `created_at` (timestamp, default CURRENT_TIMESTAMP)
  - `updated_at` (timestamp, default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)
  - `updated_by_user_id` (foreign key to users, nullable)
- Constraints:
  - UNIQUE KEY `uk_key_model_carrier` (config_key_id, model_id, carrier_id) - prevents duplicate rules for same combination
  - FOREIGN KEY constraints on all _id fields with CASCADE options
- Indexes:
  - Composite index: (model_id, carrier_id) - for fast rule matching during config resolution
  - Index: (config_key_id) - for finding all rules for a parameter
  - Index: (created_at) - for audit queries
- Example rows:
  ```sql
  -- advanced=1 for Model 22 + VZW
  config_key_id=5, model_id=22, carrier_id=3 (VZW), value='1', priority=100

  -- console_enable=1 for Model 22 + DC
  config_key_id=8, model_id=22, carrier_id=2 (DC), value='1', priority=100

  -- mqtt_enable=1 for Model 22 + VZW
  config_key_id=15, model_id=22, carrier_id=3 (VZW), value='1', priority=100
  ```

**FR-10.6a: Three-Way Conditional Rules Table**
- Table: `config_conditional_rules_3way` - NEW
- Purpose: Store conditional value rules for parameters that depend on Model AND Carrier AND Service Plan
- Columns:
  - `id` (primary key)
  - `config_key_id` (foreign key to config_keys, indexed, NOT NULL)
  - `model_id` (foreign key to models table, indexed, NOT NULL)
  - `carrier_id` (foreign key to carriers table, indexed, NOT NULL)
  - `service_plan_id` (foreign key to service_plans table, indexed, NOT NULL)
  - `value` (text, nullable) - the value to use when all conditions match; NULL = explicitly disable
  - `priority` (integer, default 100, NOT NULL) - for tie-breaking (lower = higher priority)
  - `description` (text, nullable) - business justification
  - `created_by_user_id` (foreign key to users, NOT NULL)
  - `created_at` (timestamp, default CURRENT_TIMESTAMP)
  - `updated_at` (timestamp, default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)
  - `updated_by_user_id` (foreign key to users, nullable)
- Constraints:
  - UNIQUE KEY `uk_key_model_carrier_plan` (config_key_id, model_id, carrier_id, service_plan_id)
  - FOREIGN KEY constraints on all _id fields
- Indexes:
  - Composite index: (model_id, carrier_id, service_plan_id) - for fast rule matching
  - Index: (config_key_id) - for finding all rules for a parameter
  - Index: (created_at) - for audit queries
- Example rows:
  ```sql
  -- fw_acl whitelist for any model/carrier with ATM service plan
  config_key_id=10, model_id=22, carrier_id=1 (ATT), service_plan_id=2 (ATM), value='[40 whitelist rules]', priority=100
  config_key_id=10, model_id=22, carrier_id=3 (VZW), service_plan_id=2 (ATM), value='[40 whitelist rules]', priority=100

  -- advanced=1 for ATT + Model 4500 + ATM only
  config_key_id=5, model_id=4500, carrier_id=1 (ATT), service_plan_id=2 (ATM), value='1', priority=100

  -- qos_iface=wan1 for Model 4500 + ATM (both carriers)
  config_key_id=20, model_id=4500, carrier_id=1 (ATT), service_plan_id=2 (ATM), value='wan1', priority=100
  config_key_id=20, model_id=4500, carrier_id=3 (VZW), service_plan_id=2 (ATM), value='wan1', priority=100
  ```

**FR-10.6b: Four-Way Conditional Rules Table**
- Table: `config_conditional_rules_4way` - NEW
- Purpose: Store customer-specific exception rules for parameters that depend on Model AND Carrier AND Service Plan AND Company
- **Key Design**: Customer-specific exceptions that override Three-Way Rule baselines; `company_id` is nullable for technical flexibility but primary use is customer exceptions
- Columns:
  - `id` (primary key)
  - `config_key_id` (foreign key to config_keys, indexed, NOT NULL)
  - `model_id` (foreign key to models table, indexed, NOT NULL)
  - `carrier_id` (foreign key to carriers table, indexed, NOT NULL)
  - `service_plan_id` (foreign key to service_plans table, indexed, NOT NULL)
  - `company_id` (foreign key to companies table, indexed, **NULLABLE**)
    - **Primary Use**: Customer-specific exception rules (company_id=SET)
    - **Note**: company_id can be NULL for technical/implementation flexibility, but application logic treats NULL as equivalent to Three-Way Rules (use Three-Way Rules table for baselines)
  - `value` (text, nullable) - the value to use when all conditions match; NULL = explicitly disable
  - `priority` (integer, default 100, NOT NULL) - for tie-breaking (lower = higher priority)
  - `description` (text, nullable) - business justification for this rule or customer exception
  - `created_by_user_id` (foreign key to users, NOT NULL)
  - `created_at` (timestamp, default CURRENT_TIMESTAMP)
  - `updated_at` (timestamp, default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)
  - `updated_by_user_id` (foreign key to users, nullable)
- Constraints:
  - UNIQUE KEY `uk_4way_rule` (config_key_id, model_id, carrier_id, service_plan_id, company_id)
    - Ensures one rule per unique combination
    - Supports multiple customer exceptions (different company_id values) for same Model+Carrier+ServicePlan
  - FOREIGN KEY constraints on all _id fields
- Indexes:
  - Composite index: (model_id, carrier_id, service_plan_id, company_id) - for fast rule matching during config resolution
  - Index: (config_key_id) - for finding all rules for a parameter
  - Index: (company_id) - for finding all customer-specific exceptions for a company
  - Index: (created_at) - for audit queries
- Resolution Priority:
  - Customer-specific exceptions (company_id=SET) evaluated at Priority Level 2 (after Device Override, before Company Layer)
  - Overrides Three-Way Rule baselines (Priority Level 4)
- Use Cases:
  - Firewall rules with customer-specific security requirements
  - Data thresholds with customer-specific overrides
  - Advanced features enabled for specific customers
  - Infrastructure services (NTP, DNS) with customer-specific configurations

**FR-10.7: Audit Trail Table**
- Table: `config_audit_log`
- Columns:
  - `id` (primary key)
  - `layer` (enum: global, model, carrier, service_plan, company, device, conditional_rule_2way, conditional_rule_3way, conditional_rule_4way) - NEW: added conditional rule types including 4-way
  - `layer_entity_id` (integer, nullable) - model_id, company_id, device_id, rule_id, etc. (NULL for global)
  - `config_key_id` (foreign key to config_keys)
  - `rule_conditions` (JSON, nullable) - NEW: stores conditions for rule changes (e.g., {"model_id": 22, "carrier_id": 3})
  - `old_value` (text, nullable)
  - `new_value` (text, nullable)
  - `action` (enum: create, update, delete) - NEW: type of change
  - `changed_by_user_id` (foreign key to users)
  - `changed_at` (timestamp)
  - `ip_address` (string)
  - `reason` (text, nullable) - optional reason for change
- Indexes: `changed_at`, `layer`, `config_key_id`, `changed_by_user_id`, `layer_entity_id`, `action`
- Retention: 2 years minimum
- Special logging for conditional rules:
  - When rule created: log action=create, new_value, rule_conditions
  - When rule updated: log action=update, old_value, new_value, rule_conditions
  - When rule deleted: log action=delete, old_value, rule_conditions
  - Include affected devices count in reason field

**FR-10.8: Query Performance**
- Config resolution queries must be optimized:
  - All foreign keys indexed
  - Composite indexes on (entity_id, config_key_id) for fast lookups
  - Composite indexes on (model_id, carrier_id) for conditional rule lookups
  - Consider caching effective configs (pre-generated and stored)
  - Consider caching conditional rule lookups for parameters with rules (affects subset of total parameters)
  - Consider read replicas for heavy read traffic
- Config resolution must complete in <2 seconds per device (including conditional rule evaluation)
- Batch operations (1000 devices) should complete in <30 seconds

---

