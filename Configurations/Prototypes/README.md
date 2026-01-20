# WATM Config Management System - Rapid Prototypes

This directory contains interactive HTML prototypes demonstrating the key concepts of the WATM Configuration Management System. These prototypes can be opened directly in a web browser without requiring a backend server.

## 📁 Prototype Files

### 1. Schema Management Demo
**File:** `schema-management-demo.html`

**Purpose:** Demonstrates the Config Key Schema Management system - the foundation for managing all 694 configuration parameters.

**Key Features:**
- ✅ Browse all configuration keys with search and filtering
- ✅ View detailed schema for each parameter (data type, validation rules, layer availability)
- ✅ CRUD operations for managing configuration keys
- ✅ Layer availability matrix showing which of 6 layers can set each parameter
- ✅ Configuration flags (Required, Customer Configurable, etc.)
- ✅ Validation rules display
- ✅ Export/import functionality preview

**What to Show Client:**
1. **Parameter Browser**: 12 sample parameters with different data types
2. **Detail Panel**: Click any parameter to see complete schema definition
3. **Layer Matrix**: Visual representation of which layers can configure each parameter
4. **Statistics**: Dashboard showing 694 total parameters with categorization
5. **New Key Modal**: Click "New Key" button to see CRUD interface

**Demo Script:**
```
1. "Here's our schema management interface - the foundation of the system"
2. Click on "hostname" parameter → Show complete schema details
3. Point out layer availability matrix: "Notice hostname can be set at Global, Model, Company, and Device layers"
4. Click "New Key" → "This is how admins define new configuration parameters"
5. Show filters: "We can filter by category, data type, or flags"
```

---

### 2. Layer-Based Configuration Demo
**File:** `layer-config-demo.html`

**Purpose:** Demonstrates the 6-layer hierarchical configuration system with automatic inheritance.

**Key Features:**
- ✅ Visual representation of all 6 layers (Global → Model → Carrier → Service Plan → Company → Device)
- ✅ Configuration inheritance display (values cascade from parent layers)
- ✅ Three distinct configuration states:
  - **Inherited** (yellow background): Value from parent layer
  - **Overridden** (blue background): Value set at current layer
  - **Explicitly NULL** (red background): Cleared by administrator
- ✅ Source badges showing which layer provides each value
- ✅ Inheritance chain visualization
- ✅ Resolution path preview showing how system walks layers

**What to Show Client:**
1. **Layer Selector**: 6 cards representing the hierarchy
2. **Configuration States**: Parameters showing different inheritance states
3. **Inheritance Indicators**: Labels showing "From Global", "Set at Device", etc.
4. **Resolution Preview**: Bottom panel showing how system resolves values
5. **Legend**: Color-coded explanation of configuration states

**Demo Script:**
```
1. "This shows configuration for a specific device (ATM_STORE_001)"
2. "Notice how some values are inherited (yellow) from parent layers"
3. Point to "hostname": "This is overridden at the Device layer (blue)"
4. Point to "lan0_ip": "This inherits from the Model layer (I-22)"
5. Point to "mqtt_enable": "This has been explicitly cleared (red - NULL)"
6. Scroll to bottom: "The resolution path shows how the system walks through layers"
```

---

### 3. Conditional Rules & Priority Hierarchy Demo
**File:** `conditional-rules-demo.html` ⭐ **NEW**

**Purpose:** Demonstrates the Multi-Factor Conditional Rules Framework and 11-Level Priority Hierarchy - the intelligent system that determines which configuration value wins when multiple sources could provide a value.

**Key Features:**
- ✅ Visual 11-Level Priority Hierarchy with interactive explanations
- ✅ Three rule types (Two-Way, Three-Way, Four-Way) with real examples
- ✅ Device config resolution walkthrough showing the decision-making process
- ✅ Rule creation wizard preview
- ✅ Coverage matrix showing which Model+Carrier+ServicePlan combinations have rules
- ✅ Statistics dashboard (25 Two-Way rules, 35 Three-Way rules, 12 Four-Way rules)
- ✅ Customer exception handling demonstration

**What to Show Client:**
1. **11-Level Hierarchy**: Visual priority ladder from Device Override (highest) to Required Validation (lowest)
2. **Rule Types Tabs**: Switch between Two-Way, Three-Way, and Four-Way rules
3. **Priority Levels**: Click any level to see detailed explanation
4. **Resolution Example**: See how fw_acl parameter is resolved for a CORD device
5. **Coverage Matrix**: Visual grid showing rule coverage across combinations

**Demo Script:**
```
1. "This shows our intelligent config resolution engine with 11 priority levels"
2. Click Level 2: "Four-Way Rules are customer exceptions - highest priority after device overrides"
3. Click Level 4: "Three-Way Rules define service plan baselines for all customers"
4. Switch to "Three-Way Rules" tab: "Here's how we set the ATM plan baseline"
5. Switch to "Four-Way Rules" tab: "And here's how CORD overrides it with custom rules"
6. Scroll to Resolution Example: "Watch how the system walks through levels to find the value"
7. Show Coverage Matrix: "Green cells have baselines, red cells have customer exceptions"
```

---

### 4. Import & Migration Tool Demo
**File:** `import-migration-demo.html`

**Purpose:** Demonstrates the 3-step process for importing existing configuration files into the schema database.

**Key Features:**
- ✅ 3-step wizard: Scan Files → Review Schema → Confirm Import
- ✅ File upload and directory scanning
- ✅ Auto-detection of data types from value patterns
- ✅ Confidence scoring for detected parameters
- ✅ Preview table with 8 sample parameters showing:
  - Auto-detected vs. manually reviewed parameters
  - Confidence levels (High 95%+, Medium 70-95%, Low <70%)
  - Usage percentages
- ✅ Import summary with category breakdown
- ✅ Progress animation for import completion

**What to Show Client:**
1. **Step 1 - Scan Files**: Upload interface and directory selection
2. **Step 2 - Review**: Preview table showing auto-detected parameters
3. **Confidence Scoring**: Visual bars showing detection confidence
4. **Step 3 - Confirm**: Summary statistics and import options
5. **Progress Animation**: Click "Complete Import" to see animated progress

**Demo Script:**
```
1. "This is how we migrate your existing 694 parameters into the schema"
2. Step 1: "You point it at your configuration files directory"
3. Click "Start Scan" → Move to Step 2
4. "The system automatically detects data types with 89% confidence"
5. "Parameters with IP patterns become 'ip_address' type"
6. "Boolean values (0/1) become 'boolean' type"
7. "You can review and adjust any low-confidence detections"
8. Step 3: "Here's the final summary before import"
9. Click "Complete Import" → Show progress animation
```

---

## ⭐ Updates to Existing Demos (Version 2.0 - Jan 11, 2026)

### Schema Management Demo - UPDATED (v1.5 - Jan 12, 2026)
**New Features Added:**
- ✅ **Conditional Rules Flags**: All parameters now show three new flags:
  - `has_two_way_rules` - Parameter uses any 2-factor conditional logic (6 combinations: Model+Carrier, Carrier+Plan, Model+Plan, Carrier+Customer, Model+Customer, Plan+Customer)
  - `has_three_way_rules` - Parameter uses Model + Carrier + Service Plan conditional logic
  - `has_four_way_rules` - Parameter uses Model + Carrier + Service Plan + Company conditional logic
- ✅ **Example Parameters with Rules**:
  - `fw_acl`: Has both Three-Way and Four-Way rules (service plan baselines + customer exceptions)
  - `mqtt_enable`: Has Two-Way rules (model+carrier variations)
  - `ntp_server`: Has Four-Way rules (customer-specific infrastructure)
- ✅ Flags display in Configuration Flags section with checkmarks/X marks
- ✅ Ready to demonstrate the complete conditional rules framework

### Layer Configuration Demo - UPDATED (v1.5 - Jan 12, 2026)
**New Features Added:**
- ✅ **Corrected Inheritance Display Logic**:
  - **Single-factor layers** (Model, Carrier, Service Plan, Company): Display inherited values from Global Layer ONLY
  - **Device Layer**: Shows full 11-level resolution with source attribution (can show "From Model", "Two-Way Rule", etc.)
  - **Why**: Single-factor layers have many-to-many relationships, not hierarchical
- ✅ **Contextual Rules Access Buttons**:
  - **Carrier Layer**: "🔵 Two-Way Rules" button - access all 6 two-factor combinations involving this carrier
  - **Service Plan Layer**: "🟣 Three-Way Rules" button - manage ATM plan baselines
  - **Company Layer**: "🔴 Customer Exceptions" button - manage Four-Way rules for Matrix ATM Store
- ✅ Interactive alerts showing:
  - Example rules for each layer
  - Priority levels in the 11-level hierarchy
  - Statistics (affected devices, rule counts)
  - Production workflow descriptions
- ✅ Color-coded buttons matching rule types (Blue/Purple/Red)

**Demo Improvement:**
These buttons demonstrate how admins can access conditional rules management directly from the layer they're working in, providing contextual discovery and natural workflow integration.

### Conditional Rules Demo - UPDATED (v1.5 - Jan 12, 2026)
**New Features Added:**
- ✅ **Expanded Two-Way Rules (Level 6)**:
  - Updated to show 6 sub-priorities (6.1-6.6) instead of just Model+Carrier
  - Priority hierarchy visual now shows: "Two-Way Rule (Any 2-Factor Combo)"
  - Sub-priorities: Carrier+Plan → Model+Plan → Model+Carrier → Carrier+Customer → Model+Customer → Plan+Customer
- ✅ **Enhanced Level Details**:
  - Clicking Level 6 now shows comprehensive explanation of all 6 two-factor combinations with examples
  - Shows sub-priority ordering and first-match-wins logic
- ✅ **Updated Rule Creation Wizard**:
  - Step 2 now shows factor combination selector (6 options)
  - System automatically assigns sub-priority (6.1-6.6) based on selected combination
- ✅ **Two-Way Rules Tab**:
  - Added informational panel listing all 6 supported combinations
  - Updated description to reflect "ANY TWO of the four factors"

---

## 🚀 How to Use These Prototypes

### Opening the Prototypes
1. Navigate to `/Users/aksana/Documents/Projects/WATM/Configurations/Prototypes/`
2. Double-click any `.html` file to open in your default browser
3. Or right-click → Open With → Choose your preferred browser

### Recommended Browsers
- ✅ Google Chrome (Recommended)
- ✅ Safari
- ✅ Firefox
- ✅ Microsoft Edge

### No Installation Required
These are standalone HTML files with embedded CSS and JavaScript. No web server, database, or backend required.

---

## 🎯 Client Presentation Strategy

### Recommended Order:
1. **Start with Schema Management Demo** (5-7 minutes)
   - Establishes the foundation
   - Shows the 600+ parameters with conditional rules flags
   - Demonstrates CRUD operations
   - **NEW**: Point out parameters with conditional rules (fw_acl, mqtt_enable, ntp_server)

2. **Move to Conditional Rules & Priority Hierarchy Demo** (7-10 minutes) ⭐ **NEW**
   - Shows the intelligent decision-making system
   - Explains 11-level priority hierarchy
   - Demonstrates Two-Way, Three-Way, and Four-Way rules
   - Shows device resolution walkthrough
   - Coverage matrix visualization

3. **Show Layer-Based Configuration Demo** (5-7 minutes)
   - Shows how the schema and rules work together in practice
   - Demonstrates inheritance concept
   - Explains the 6-layer hierarchy
   - **NEW**: Click contextual rules access buttons (Carrier, Service Plan, Company layers)

4. **Finish with Import & Migration Tool Demo** (3-5 minutes)
   - Shows how they get from current state to new system
   - Demonstrates automation (89% auto-detected)
   - Builds confidence in migration path

**Total Presentation Time:** 20-30 minutes

### Key Messages to Emphasize:

1. **600+ Parameters → Fully Managed**
   - "Every single configuration parameter gets a formal definition"
   - "Type safety, validation rules, access control built-in"
   - **NEW**: "Conditional rules flags show which parameters need multi-factor logic"

2. **11-Level Priority Hierarchy → Intelligent Resolution**
   - "System automatically resolves conflicts by checking 11 sources in priority order"
   - "Device overrides always win (Level 1)"
   - "Customer exceptions (Four-Way Rules) beat service plan baselines (Three-Way Rules)"
   - "Conditional rules integrate seamlessly with layer inheritance"

3. **Conditional Rules Framework → Ultimate Flexibility**
   - "Two-Way Rules: 6 two-factor combinations with sub-priorities (e.g., Carrier+Plan: VZW ATM → 3584MB; Model+Carrier: i-22 VZW → mqtt enabled)"
   - "Three-Way Rules: Service plan baselines for all customers (e.g., ATM plan → 40 firewall rules)"
   - "Four-Way Rules: Customer exceptions to baselines (e.g., CORD → 50 custom firewall rules)"
   - "Balance between consistency (baselines) and flexibility (exceptions)"

4. **6-Layer Hierarchy → Massive Efficiency**
   - "Set once at Global layer → applies to all devices"
   - "Override only when needed at specific layers"
   - "No more copy-paste of 600+ parameters per device"

5. **Customer Self-Service → Reduced Support Load**
   - "Customer Configurable flag enables safe self-service"
   - "~90 parameters customers can safely modify"
   - "All other parameters protected by access control"

6. **Migration Path → Low Risk**
   - "89% auto-detection means minimal manual work"
   - "Import tool validates everything before committing"
   - "Can review and adjust all parameters before import"

---

## 📊 Statistics to Highlight

From the prototypes, emphasize these numbers:

| Metric | Value | What It Means |
|--------|-------|---------------|
| **Total Parameters** | 600+ | Every configuration option formally defined |
| **Layers** | 6 | Global → Model → Carrier → Service Plan → Company → Device |
| **Priority Levels** | 11 | Intelligent hierarchy for conflict resolution |
| **Two-Way Rules** | 25 | Model + Carrier conditional combinations |
| **Three-Way Rules** | 35 | Service plan baselines (apply to all customers) |
| **Four-Way Rules** | 12 | Customer-specific exceptions to baselines |
| **Devices Affected by Rules** | 85,000+ | Scale of the conditional rules system |
| **Auto-Detection Rate** | 89% | Minimal manual work during migration |
| **Customer Configurable** | ~90 params | Safe self-service parameters |

---

## 🎨 Visual Elements to Point Out

### Color Coding in Prototypes:

**Schema Management Demo:**
- 🔵 Blue badges: Data types
- 🟢 Green badges: Customer Configurable
- 🟡 Yellow badges: Required fields
- **NEW**: Checkmarks (✓) and X marks (✗) for conditional rules flags

**Layer Configuration Demo:**
- 🟡 Yellow background: Inherited from parent layer
- 🔵 Blue background: Overridden at current layer
- 🔴 Red background: Explicitly NULL (cleared)
- **NEW**: 🔵 Blue button: Two-Way Rules access
- **NEW**: 🟣 Purple button: Three-Way Rules access
- **NEW**: 🔴 Red button: Four-Way Rules (Customer Exceptions) access

**Conditional Rules Demo:** ⭐ **NEW**
- 🟢 Green levels: Highest priority (Device Override)
- 🔵 Blue levels: Conditional rules (Levels 2, 4, 6)
- ⚪ Gray levels: Standard layers (Levels 3, 5, 7-9)
- ⚫ Black levels: Defaults and validation (Levels 10-11)
- 🔵 Blue cards: Two-Way Rules
- 🟣 Purple cards: Three-Way Rules (Service Plan Baselines)
- 🔴 Red cards: Four-Way Rules (Customer Exceptions)
- 🟢 Green matrix cells: Has baseline rule
- 🔴 Red matrix cells: Has customer exceptions

**Import Tool Demo:**
- 🟢 Green confidence bars: High (95%+)
- 🟡 Yellow confidence bars: Medium (70-95%)
- 🔴 Red confidence bars: Low (<70%)

---

## 💡 Expected Client Questions & Answers

### Q: "Can we customize which parameters customers can modify?"
**A:** Yes! Point to the "Customer Configurable" flag in Schema Management Demo. Each parameter has a boolean flag controlling customer access.

### Q: "What happens if we add a new parameter later?"
**A:** Show the "New Key" modal in Schema Management Demo. Admins can add new parameters anytime through the CRUD interface.

### Q: "How do we know which layer a value came from?"
**A:** Show the Layer Configuration Demo. Each parameter displays its source layer and full inheritance chain.

### Q: "What if the auto-detection gets something wrong?"
**A:** Show Step 2 of Import Tool. All auto-detected parameters can be reviewed and manually adjusted before import.

### Q: "Can we export configurations for backup?"
**A:** Yes! Point to "Export" buttons in both Schema Management and Layer Configuration demos.

---

## 🔄 Next Steps After Demo

If client approves the concept:

1. **Phase 1 - Schema Foundation (2-3 weeks)**
   - Implement schema management CRUD
   - Build import/migration tool
   - Import all 694 parameters

2. **Phase 2 - Layer System (3-4 weeks)**
   - Build 6-layer configuration system
   - Implement inheritance engine
   - Create layer configuration UI

3. **Phase 3 - Customer Portal (2-3 weeks)**
   - Customer-facing configuration interface
   - Apply customer_configurable flags
   - Build validation and error handling

4. **Phase 4 - Migration (1-2 weeks)**
   - Migrate existing device configurations
   - Testing and validation
   - Staff training

**Total Estimated Timeline:** 8-12 weeks for complete system

---

## 📝 Prototype Limitations

These are rapid prototypes for concept demonstration. They do NOT include:

- ❌ Real database connections
- ❌ API integrations
- ❌ User authentication
- ❌ Data persistence (changes are not saved)
- ❌ Actual file processing
- ❌ Complete validation logic
- ❌ All 694 parameters (only 8-12 samples shown)

These will be implemented in the production system.

---

## 🛠️ Technical Implementation Notes

For the development team:

### Technology Stack (Proposed):
- **Backend:** CakePHP 4.x (existing WATM stack)
- **Database:** MySQL 8.0 with new `config_schema` and `config_values` tables
- **Frontend:** React or Vue.js for dynamic UI
- **API:** RESTful JSON API
- **Queue:** CakePHP Queue for bulk operations
- **Validation:** CakePHP Validation + Custom validators

### Database Schema (Simplified):
```sql
-- Schema definition table
CREATE TABLE config_schema (
  id INT PRIMARY KEY AUTO_INCREMENT,
  key_name VARCHAR(255) UNIQUE NOT NULL,
  display_name VARCHAR(255) NOT NULL,
  data_type VARCHAR(50) NOT NULL,
  category VARCHAR(100),
  validation_rules JSON,
  layer_availability JSON,
  flags JSON,
  ui_hints JSON,
  created DATETIME,
  modified DATETIME
);

-- Configuration values table (multi-layer)
CREATE TABLE config_values (
  id INT PRIMARY KEY AUTO_INCREMENT,
  config_schema_id INT NOT NULL,
  layer_type ENUM('global','model','carrier','service_plan','company','device'),
  layer_id INT, -- Foreign key to respective table
  value TEXT,
  is_null BOOLEAN DEFAULT FALSE,
  created DATETIME,
  modified DATETIME,
  FOREIGN KEY (config_schema_id) REFERENCES config_schema(id),
  UNIQUE KEY unique_layer_key (config_schema_id, layer_type, layer_id)
);
```

---

## 📧 Contact & Feedback

For questions about these prototypes:
- Review the conversation summary for full context
- Reference `/Users/aksana/Documents/Projects/WATM/Configurations/Config_Parameters_By_Layer_Analysis_full.md` for complete parameter analysis
- See `/Users/aksana/Documents/Projects/WATM/Configurations/Documentation/Configuration_Parameter_Analysis.md` for usage statistics

---

## 📅 Version History

**Version 1.5** - 2026-01-12
- **Two-Way Rules Expansion**: Updated to support 6 two-factor combinations (not just Model+Carrier)
- **Sub-Priorities Added**: Level 6 now has 6 sub-priorities (6.1-6.6) with ordering
- **Inheritance Logic Correction**: Single-factor layers now correctly show "inherited from Global only"
- **Schema Demo**: Added comprehensive Conditional Rules Information section
- **Layer Demo**: Updated showTwoWayRules() to list all 6 combinations
- **Conditional Rules Demo**: Expanded Level 6 details, rule creation wizard, and tab descriptions
- Aligned all prototypes with PRD v1.5

**Version 2.0** - 2026-01-11
- Added Conditional Rules Framework support
- Added has_two_way_rules, has_three_way_rules, has_four_way_rules flags
- Added contextual rules access to layer configs
- Updated statistics and badges
- Aligned with comprehensive PRD

**Version 1.0** - 2026-01-02
- Initial rapid prototypes created
- Schema Management Demo
- Layer-Based Configuration Demo
- Import & Migration Tool Demo
- Complete documentation

---

## ✅ Demo Checklist

Before presenting to client:

- [ ] Test all three prototypes in target browser
- [ ] Review demo script for each prototype
- [ ] Prepare answers to expected questions
- [ ] Have backup plan if technology fails (screenshots, PDF export)
- [ ] Print this README for reference during demo
- [ ] Prepare timeline and budget estimates
- [ ] Have next steps ready to discuss

**Good luck with your client demo! 🚀**
