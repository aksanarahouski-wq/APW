# Product Requirements Document (PRD)
## Hierarchical Configuration Management System

**Document Version:** 1.5
**Date:** January 12, 2026
**Last Updated:** FR-1 Updated - Two-Way Rules definition expanded to include all 6 two-factor combinations with examples and sub-priority ordering
**Author:** Aksana Rahouski / Orases Team
**Status:** Draft for Review
**Related Tickets:** TBD
**Document Owner:** Aksana Rahouski

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Background and Problem Statement](#background-and-problem-statement)
3. [Goals and Objectives](#goals-and-objectives)
4. [Target Users](#target-users)
5. [User Stories](#user-stories)
6. [Scope](#scope)
7. [Functional Requirements](#functional-requirements)
8. [Technical Design](#technical-design)
9. [Testing Requirements](#testing-requirements)
10. [Dependencies and Risks](#dependencies-and-risks)
11. [Implementation Plan](#implementation-plan)
12. [Success Metrics](#success-metrics)
13. [Open Questions](#open-questions)
14. [Document History](#document-history)

---

## Executive Summary

The WATM platform currently manages device configurations for 100,000+ IoT devices using a flat file-based system with approximately 200 individual configuration files. This approach has become unmaintainable and prevents APW from efficiently responding to infrastructure changes, security requirements, and customer customization needs. A single parameter change (such as updating DNS servers) requires manually editing all 200 config files, taking hours and introducing significant error risk.

This PRD defines requirements for a **Hierarchical Configuration Management System** that transforms the current config repository into an intelligent configuration engine. The new system will enable configuration inheritance across multiple layers (Global → Model → Carrier → Service Plan → Company → Device), allowing administrators to set default values at high levels while supporting specific overrides at lower levels. Configuration changes will cascade automatically to affected devices, reducing maintenance time from hours to minutes while eliminating manual errors.

### Key Features
- **Six-Layer Configuration Hierarchy**: Hierarchical configuration model (Global → Model → Carrier → Service Plan → Company → Device) with inheritance and override capabilities at each layer
- **Conditional Rules Framework**: Multi-factor conditional logic for parameter values with support for:
  - **Two-Way Rules**: Any 2-factor combinations (6 types):
    - Model + Carrier (carrier features per model)
    - Carrier + Service Plan (carrier data policies by plan)
    - Model + Service Plan (model pricing tier by plan)
    - Carrier + Customer (customer carrier agreements)
    - Model + Customer (customer model configurations)
    - Service Plan + Customer (customer plan exceptions)
  - **Three-Way Rules**: Model + Carrier + Service Plan combinations (service plan baselines for all customers)
  - **Four-Way Rules**: Model + Carrier + Service Plan + Company combinations (customer-specific exceptions to Three-Way Rule baselines)
- **11-Level Priority Hierarchy with Sub-Priorities**: Intelligent config resolution engine that checks 11 sources in priority order:
  1. Device Override
  2. Four-Way Rule
  3. Company Override
  4. Three-Way Rule
  5. Service Plan Layer
  6. Two-Way Rule (6 sub-priorities: Carrier+Plan → Model+Plan → Model+Carrier → Carrier+Customer → Model+Customer → Plan+Customer)
  7. Carrier Layer
  8. Model Layer
  9. Global Layer
  10. Schema Default
  11. Required Validation
- **Automated Version Control**: System-managed config versioning and device synchronization
- **Layer-Specific Parameter Control**: Define which parameters are available at each layer (Global, Model, Carrier, Service Plan, Company, Device) with schema-enforced validation. Parameters can be configured as single-layer (e.g., Global only) or multi-layer (available at multiple levels with override capability based on hierarchy)
- **Customer Self-Service**: Safe, UI-driven configuration for end customers with strict validation
- **Gradual Migration**: Parallel operation of old and new systems with controlled device migration
- **Audit Trail**: Complete visibility into configuration changes (who, what, when, why)

### Business Impact
- **Operational Efficiency**: Reduce config management time by 95% (hours → minutes for global changes)
- **Risk Reduction**: Eliminate manual hostname management errors that cause device update loops
- **Business Agility**: Respond to infrastructure and security requirements in minutes vs. days
- **Scalability**: Support platform growth to 200,000+ devices without architectural changes
- **Competitive Advantage**: Offer flexible customization that competitors cannot match

---

## Background and Problem Statement

### Current State

The WATM platform manages device configurations using a file-based system where each configuration is stored as a separate file. Device-to-config mapping occurs automatically based on device attributes (model, carrier, cellular backup status, service plan, and company). Currently there are:

- **~200 total configuration files** (167 custom configs + base configs)
- **100,000+ active devices** across all customers
- **Manual file editing** required for all configuration changes
- **Manual hostname/timestamp management** for version control
- **No inheritance model** - each file contains complete parameter set
- **Limited customer self-service** - only Wi-Fi configuration currently available

**Configuration Parameters:**
- Each config file contains 600+ parameters (key-value pairs)
- Parameters include: DNS servers, time servers, network settings, security settings, alarm I/O, scheduling, LAN IP configuration, VPN settings, and firmware-specific settings
- Many parameters are identical across most/all configs (e.g., DNS, time servers)
- Some parameters are model-specific (e.g., i-22 alarm I/O doesn't apply to 4100 devices)
- Some parameters are carrier-specific (e.g., Verizon vs T-Mobile APN settings)

### Problems

**Problem 1: Unmaintainable Scale**
- Changing a single parameter that applies to all devices requires editing up to 200 config files manually
- Example: Migrating from Google DNS (8.8.8.8) to private DNS requires updating all config files
- **Who is affected**: System administrators, DevOps team
- **Business impact**: Hours of manual work for simple changes, prevents rapid response to infrastructure needs, high operational cost

**Problem 2: Manual Error-Prone Version Control**
- Admins manually type timestamps into hostname fields to track config versions
- Hostname typos cause devices to enter infinite update loops (device gets config → reboots → gets config again)
- No automated way to identify which devices are affected by a config change
- **Who is affected**: System administrators, customers (when their devices are affected)
- **Business impact**: Service disruptions, customer churn risk, time spent troubleshooting loops

**Problem 3: No Configuration Hierarchy**
- Cannot set global defaults that cascade to all devices
- Cannot easily identify which of 200 configs need updating when requirements change
- No clear separation between global, model-specific, carrier-specific, and customer-specific parameters
- **Who is affected**: System administrators
- **Business impact**: Difficulty finding affected configs, high risk of missing files during updates, inconsistent configurations

**Problem 4: Cannot Respond to Business Requirements**
- **Security**: Cannot quickly disable factory reset across all devices to prevent unauthorized repurposing
- **Infrastructure**: Cannot test DNS changes on small device set before full deployment
- **Customer Service**: Cannot enable portfolio-wide settings for customers with many devices
- **Who is affected**: Business stakeholders, customers, system administrators
- **Business impact**: Missed revenue opportunities, competitive disadvantage, customer dissatisfaction

**Problem 5: Model-Specific Parameter Confusion**
- Different device models support different parameters, but system doesn't enforce this
- No validation prevents applying invalid parameters to wrong models
- Documentation of which parameters apply to which models exists only in tribal knowledge
- **Who is affected**: System administrators, operations team
- **Business impact**: Configuration errors, difficult troubleshooting, steep learning curve for new staff

**Problem 6: High-Risk Deployments**
- No ability to test config changes on small device sets before full deployment
- All-or-nothing config updates pose significant risk with 100,000 devices
- No easy rollback if config changes cause issues
- **Who is affected**: All stakeholders, especially customers
- **Business impact**: High deployment risk, potential for large-scale service disruptions, fear of making necessary changes

### Impact if Not Addressed

**Operational Costs:**
- Continued high labor costs for config management (hours per week)
- Incident response costs when config errors occur

**Business Limitations:**
- Cannot quickly respond to security threats or compliance requirements
- Cannot offer competitive configuration flexibility to customers
- Difficulty scaling beyond current device count (approaching architectural limits)

**Risk Exposure:**
- Continued risk of manual errors causing widespread device outages
- Infrastructure changes (DNS, time servers) remain high-risk operations
- No audit trail for compliance and troubleshooting

**Quantified Impact:**
- System admin time: 5-10 hours/week on config file maintenance
- Customer dissatisfaction: Config-related complaints appear in monthly reviews
- Competitive disadvantage: Lost deals where config flexibility was requirement

---

## Goals and Objectives

### Primary Goals

1. **Operational Efficiency**: Enable single-point configuration changes that cascade to appropriate device sets without manual file editing
   - **Target**: Reduce global config change time from hours/days to <10 minutes
   - **Benefit**: 95%+ time savings, minimal error risk, rapid response capability

2. **Hierarchical Configuration Model**: Implement cascading configuration across six layers with inheritance and override capabilities
   - **Layers**: Global → Model → Carrier → Service Plan → Company → Device
   - **Benefit**: Set defaults once at high level, override at specific levels as needed
   - **Pattern**: More general configurations cascade down, more specific configurations override

3. **Risk Reduction**: Minimize configuration errors and enable controlled rollout of changes
   - **Automated versioning**: Eliminate manual hostname/timestamp management
   - **Gradual migration**: Test on 5-10 devices, expand gradually to full fleet
   - **Rollback capability**: Revert changes if issues occur
   - **Benefit**: Near-zero config-related incidents, high confidence deployments

4. **Business Agility**: Enable rapid response to infrastructure, security, and business requirements
   - **Security changes**: Implement global security parameters in minutes (e.g., disable factory reset)
   - **Infrastructure migrations**: Change DNS, time servers with minimal effort and risk
   - **Customer customization**: Support complex requirements without config file sprawl
   - **Benefit**: Competitive advantage, faster time-to-market for config-based features

### Success Criteria

_Measurable criteria that define when this project is successful_

**Phase 1 (MVP):**
- ✅ Config Key Schema Management built and tested
- ✅ Full schema imported (600+ configuration parameters)
- ✅ Hierarchical config engine built and tested for all 6 layers (Global → Model → Carrier → Service Plan → Company → Device)
- ✅ Conditional Rules Framework implemented (Multi-factor conditional logic for 2-way, 3-way, and 4-way rules)
- ✅ Four-Way Rules support customer-specific exceptions to service plan policies defined by Three-Way Rules
- ✅ Both config management systems operational in parallel (OLD and NEW)
- ✅ 1,000 devices successfully migrated from old to new system
- ✅ Zero critical incidents related to new config system

**Phase 2 (Customer Self-Service - Device Level):**
- ✅ Customer-facing device configuration UI with validation added
- ✅ Customers can edit device-level configs (not only Admins)
- ✅ Customer access restricted to device-level only (no company-wide configuration access)
- ✅ 50,000 devices migrated to new system

**Phase 2+ (Future Enhancement - Company-Wide Customer Access):**
- ⚠️ Customer-facing company-wide configuration UI (portfolio-wide settings)
- ⚠️ Customers can apply settings across all their devices at once

**Phase 3 (Complete Migration):**
- ✅ All devices migrated to new config system
- ✅ OLD config management system retired
- ✅ Code cleanup completed

### Non-Goals (Out of Scope for Phase 1 & 2)

_What this project will NOT include_

- ❌ **Customer company-wide configuration access (Phase 1 & 2)**: Customers can only configure individual devices; portfolio-wide settings (Company Layer access for customers) deferred to future phase
- ❌ **Customer config template creation**: Customers can modify allowed parameters but cannot create new configuration structures or define new parameters
- ❌ **Real-time config push**: Continue using check-in based delivery model; not implementing WebSocket/push notification system
- ❌ **Config templates marketplace**: Not building shareable config templates between companies
- ❌ **Advanced analytics**: Config usage analytics, trend analysis, recommendations deferred to future enhancements
- ❌ **Automated config optimization**: System will not automatically suggest or apply config optimizations

---

## Target Users

### Primary Users

**1. System Administrator (APW Admin)**
- **Role**: Platform administrators (Adam Curcie, John Gates, DevOps team) with full system access and deep technical knowledge
- **Current Pain Point**: Managing 600+ config parameters across multiple files manually; cannot respond quickly to global changes (DNS, security); manual hostname management causes errors
- **Need**:
  - Efficiently manage global configurations affecting all 100,000+ devices
  - Define model-specific parameters and carrier-specific settings
  - Respond to security/infrastructure requirements in minutes
  - Safe gradual rollout and migration tools
  - Complete audit trail for compliance
- **Benefit**: 95% time reduction for config changes (hours → minutes); near-zero config errors; ability to implement security measures instantly; confidence in deployments
- **Access Level**: Full administrative access to all config layers
- **Key Use Cases**:
  - Update global DNS across all devices in <10 minutes
  - Disable factory reset on all devices for security
  - Test new carrier settings on 10 devices before full rollout
  - Troubleshoot device config with source attribution view

**2. End Customer Administrator (APW Client)**
- **Role**: APW customers who manage their own device fleets (10-10,000 devices); fleet managers and enterprise IT admins with intermediate technical skills
- **Need**:
  - Configure individual device settings that affect their operations (Wi-Fi, scheduling, LAN IPs)
  - Customize individual devices when needed
  - **Must be prevented from breaking their devices** (per meeting: "they will take themselves out of business and blame us")
- **Benefit**: Self-service device configuration; same-day changes vs. multi-day wait; reduced dependency on APW support
- **Access Level**: **Phase 1: Device-level configuration only** - Limited write access to own devices only, customer-configurable parameters only, UI-driven with strict validation, **cannot download raw config files**, **cannot modify company-wide settings**
- **Allowed Parameters**: Reboot scheduling, LAN IP (within ranges), Wi-Fi networks, power management schedules (device-level only)
- **Restricted Parameters**: DNS, time servers, firmware settings, carrier configs, factory reset control, **company-wide configurations**
- **Key Use Cases**:
  - Configure device LAN IP with validation (dropdowns, not free-form)
  - Manage Wi-Fi networks for individual devices (current pattern to extend)
  - Set device-specific reboot schedules
- **Future Enhancement (Phase 2+)**: Portfolio-wide settings to apply configurations across entire device fleet

### User Access Summary

| Capability | System Admin | Customer Admin (Phase 1) |
|------------|--------------|----------------|
| **Edit Global/Model/Carrier Configs** | ✅ Full | ❌ No |
| **View System Configs** | ✅ All | ❌ No |
| **Edit Company Configs** | ✅ All companies | ❌ **Phase 1: No** (⚠️ Phase 2+) |
| **Edit Device Configs** | ✅ All devices | ✅ Own only (allowed params) |
| **Download Raw Config Files** | ✅ Yes | ❌ **NEVER** |

## Scope

### In Scope

**All components below are included in Phase 1 (MVP) unless explicitly noted otherwise.**

#### System Infrastructure

**✅ Config Key Schema Management**
- CRUD interface for defining configuration keys (600+ parameters)
- Key properties: name, data type, description, validation rules
- Layer availability matrix (which layers can set this key)
- Flags: Required (yes/no), Customer Configurable (yes/no)
- **Conditional dependency tracking**: Flag parameters that require multi-factor conditional rules logic
  - `has_two_way_rules` flag: Parameter requires any 2-factor combination logic (6 types):
    - Model + Carrier (carrier features per model)
    - Carrier + Service Plan (carrier data policies by plan)
    - Model + Service Plan (model pricing tier by plan)
    - Carrier + Customer (customer carrier agreements)
    - Model + Customer (customer model configurations)
    - Service Plan + Customer (customer plan exceptions)
  - `has_three_way_rules` flag: Parameter requires Model + Carrier + Service Plan combination logic (3-way)
  - `has_four_way_rules` flag: Parameter requires Model + Carrier + Service Plan + Company combination logic (4-way)
  - Support for parameters with multiple rule types (higher-factor rules take precedence)
- Schema import tool to migrate existing 600+ keys from current config files
- Key search, filter, and list views (with conditional rules indicators)
- Export/import schema for backup and documentation

**✅ Conditional Rules Framework**
- Generic system for defining multi-factor conditional logic for parameter values that override standard layer inheritance when specific device attribute combinations match
- **Purpose**: Handle complex business logic where parameter values depend on multiple device characteristics simultaneously (e.g., firewall rules differ based on Model + Carrier + Service Plan combination)

**Rule Types:**
- **Two-Way Rules**: Define values based on ANY TWO of the four factors (Model, Carrier, Service Plan, Customer)
  - **Supported Combinations:**
    1. **Model + Carrier** - Carrier-specific features per device model
       - Example: `IF model=i-22 AND carrier=VZW THEN mqtt_enable=1` (Device Manager enabled for VZW i-22 devices only)
       - Example: `IF model=i-22 AND carrier=VZW THEN advanced=1` (Advanced mode enabled for VZW i-22 devices)
    2. **Carrier + Service Plan** - Carrier data policies by service plan tier
       - Example: `IF carrier=VZW AND service_plan=ATM THEN traffic_day_threshold=3584MB` (VZW ATM plan gets 3.5GB daily limit)
       - Example: `IF carrier=ATT AND service_plan=Tier1 THEN traffic_day_threshold=5GB` (AT&T Tier1 gets 5GB daily)
    3. **Model + Service Plan** - Model-based pricing/features by service plan
       - Example: `IF model=i-22 AND service_plan=ATM THEN traffic_day_threshold=350MB` (i-22 devices on ATM plan have restrictive limit)
       - Example: `IF model=4500 AND service_plan=ATM THEN traffic_day_threshold=5GB` (4500 devices on ATM plan get higher limit)
    4. **Carrier + Customer** - Customer-specific carrier agreements
       - Example: `IF carrier=VZW AND customer=VIP_Corp THEN traffic_day_threshold=10GB` (VIP customer negotiated higher VZW data allowance)
    5. **Model + Customer** - Customer configuration varies by deployed model
       - Example: `IF model=i-22 AND customer=Miele THEN digitalio_config=[I/O settings]` (Miele's i-22 devices need specific I/O config)
    6. **Service Plan + Customer** - Customer exceptions to plan baseline (alternative to Four-Way)
       - Example: `IF service_plan=ATM AND customer=Premium_Customer THEN traffic_day_threshold=1000MB` (upgraded allowance)
  - **Sub-Priority Order**: When multiple two-factor combinations could apply, system checks in this order:
    1. Carrier + Service Plan (carrier policy by plan)
    2. Model + Service Plan (model tier by plan)
    3. Model + Carrier (original design)
    4. Carrier + Customer (customer carrier agreements)
    5. Model + Customer (customer model configs)
    6. Service Plan + Customer (customer plan exceptions)
  - **Use case**: Any scenario where parameter value depends on TWO (and only two) device attributes simultaneously
- **Three-Way Rules**: Define values based on Model AND Carrier AND Service Plan combinations
  - Example: `IF model=4500 AND carrier=ATT AND service_plan=ATM THEN fw_acl=[40 whitelist rules]`
  - Applies to ALL customers with matching Model+Carrier+ServicePlan
  - Use case: Service plan features that vary by model and carrier
- **Four-Way Rules**: Define customer-specific exceptions based on Model AND Carrier AND Service Plan AND Company combinations
  - **Purpose**: Enable customer-specific exceptions to service plan policies defined by Three-Way Rules
  - Rule applies ONLY to that specific customer; overrides the Three-Way Rule baseline
  - Example: `IF model=i-22 AND carrier=ATT AND service_plan=ATM AND company=CORD THEN fw_acl=[custom rules]`
    - CORD company gets custom firewall rules that override the standard ATM service plan rules for all CORD devices
  - Resolves at higher priority (Level 2) than Three-Way Rules (Level 4)
  - Use case: Customer-specific exceptions to service plan policies (custom firewall rules, modified data thresholds, customer-specific advanced features)
  - **Note**: The general baseline for all customers is handled by Three-Way Rules; Four-Way Rules are only used for customer exceptions

**How Rules Work:**
- Rules are evaluated during config resolution based on device attributes (model_id, carrier_id, service_plan_id, company_id)
- When rule conditions match, rule value overrides standard layer inheritance
- Multiple rule types can exist for same parameter (e.g., two-way, three-way, and four-way rules)
- Higher-factor customer-specific rules take precedence: Four-Way (customer exception) wins over Three-Way, Three-Way wins over Two-Way
- Rules integrate into 11-level priority hierarchy (see Config Resolution Engine below)

**Rule Type Selection Guide:**

**Use Two-Way Rules** when value depends on ANY TWO factors (choose appropriate combination):
- **Model + Carrier**: Carrier-specific features per device model
  - Example: VZW i-22 devices enable Device Manager (`mqtt_enable=1`), but AT&T i-22 devices don't (`mqtt_enable=0`)
- **Carrier + Service Plan**: Carrier data policies vary by service plan
  - Example: VZW ATM plan gets 3584MB daily, but AT&T ATM plan only gets 350MB daily (different carrier policies)
- **Model + Service Plan**: Model pricing tier affects service plan limits
  - Example: i-22 devices on ATM plan get 350MB daily, but 4500 devices on ATM plan get 5GB daily (model tier pricing)
- **Carrier + Customer**: Customer-specific carrier agreements (VIP data allowances, custom SLAs)
- **Model + Customer**: Customer configuration varies by deployed model (I/O settings, custom networking)
- **Service Plan + Customer**: Simple customer exceptions to plan baseline (when Model+Carrier don't matter)

**Use Three-Way Rules** when value depends on Model + Carrier + Service Plan:
- This defines the baseline/default for ALL customers with that Model+Carrier+ServicePlan combination
- Example: All Model i-22 + ATT + ATM devices get 40 firewall whitelist rules (applies to every customer on ATM plan)
- Use when service plan features vary by both model AND carrier

**Use Four-Way Rules** only when specific customers need exceptions to Three-Way Rule baselines:
- First, create the Three-Way Rule for the general baseline that applies to all customers
- Then, create Four-Way Rules only for customers that need different values
- Example: Three-Way Rule sets `fw_acl=[40 rules]` for all ATM customers, but CORD needs `fw_acl=[50 rules]`, so create a Four-Way Rule for CORD
- Four-Way Rules override Three-Way baselines for specific companies only

**Decision Tree:**
```
Does value depend on customer-specific exception to a service plan policy?
  YES → Use Four-Way Rule (Model + Carrier + Plan + Customer)
  NO  ↓

Does value depend on THREE factors (Model + Carrier + Plan)?
  YES → Use Three-Way Rule
  NO  ↓

Does value depend on TWO factors?
  YES → Use Two-Way Rule (choose appropriate combination from 6 types)
  NO  ↓

Use Layer-based inheritance (set value at appropriate layer: Global, Model, Carrier, Service Plan, Company, Device)
```

**Admin Interface:**
- Create/edit/delete conditional rules for any parameter
- Define conditions: Select factors (Model, Carrier, Service Plan, Company) from dropdowns
- Set value when conditions match device attributes
- Rule validation (no conflicts, all factors exist, no duplicate conditions)
- Visual matrix interface for viewing and managing rules across combinations
- Hierarchical view for four-way rules (baseline rules with nested customer exceptions)
- Rule coverage reporting and gap analysis ("fw_acl has 8 baseline rules + 3 customer exceptions")
- Customer exception tracking and reporting
- Business logic documentation for each rule

**✅ Six-Layer Configuration Hierarchy**
- Hierarchical configuration model where configs flow from general (Global) to specific (Device)
- More specific layers inherit values from more general layers
- More specific layers can override inherited values
- Final "effective config" for a device is resolved by walking through all layers in priority order

**The Six Layers (All operational in Phase 1):**

1. **Global Layer** - System-wide master configuration
   - Contains default values for all parameters
   - Applies to all devices unless overridden at lower layers
   - Example: DNS servers = 8.8.8.8, time server = time.nist.gov
   - Use case: Infrastructure settings that should be consistent across entire fleet

2. **Model Layer** - Device model-specific configurations
   - Configurations specific to hardware models (i-22, i-52, 4100, Origin, etc.)
   - Example: Model i-22 has specific alarm I/O settings, Model 4100 has modem-specific parameters
   - Use case: Hardware capabilities, model-specific features, default settings per model

3. **Carrier Layer** - Cellular carrier-specific configurations
   - Configurations specific to carriers (Verizon, T-Mobile, AT&T, etc.)
   - Example: Verizon devices use specific APN settings, AT&T has different network parameters
   - Use case: Carrier network settings, carrier-specific features

4. **Service Plan Layer** - Service plan/tier-specific configurations
   - Configurations tied to APW service offerings (ATM plan, tier1 plan, etc.)
   - Example: ATM plan has higher data thresholds (350MB vs 5MB), extensive firewall whitelists
   - Use case: Feature differentiation, service level policies, data allowances

5. **Company Layer** - Customer portfolio-wide configurations
   - Configurations that apply to all devices belonging to a specific customer
   - Example: ACME Corp sets all their devices to reboot at 3 AM, custom DNS = 10.1.1.1
   - Use case: Customer-specific settings, branding, operational preferences, network integration

6. **Device Layer** - Individual device overrides
   - Configurations for a single specific device
   - Example: Device #12345 has custom LAN IP = 192.168.50.100, specific Wi-Fi networks
   - Use case: Site-specific settings, one-off customizations, troubleshooting overrides

**Inheritance & Override Model:**
- Device inherits from Company, which inherits from Service Plan, which inherits from Carrier, etc.
- Example inheritance chain: Device → Company → Service Plan → Carrier → Model → Global
- Example resolution: DNS not set at Device → **SET at Company = 10.1.1.1** → STOP, use 10.1.1.1
- If DNS not set at any layer: Fall back to Global Layer value (8.8.8.8)

**✅ Config Resolution Engine**
- **Core Function**: Intelligent algorithm to resolve final "effective config" for any device by determining which configuration value to use when multiple sources could provide a value for the same parameter
- **Resolution Method**: Check multiple sources in priority order; first source with a value wins
- **Output**: Complete configuration file with source attribution for troubleshooting

**11-Level Priority Hierarchy** (Highest to Lowest Priority):

When resolving a parameter value for a device, the system checks these sources in order and uses the first value found:

**Level 1: Device Override** *(Highest Priority)*
- Individual device-specific value set at Device Layer
- Example: Device #12345 has custom LAN IP = 192.168.50.100
- Use case: Site-specific configurations, one-off overrides, troubleshooting

**Level 2: Four-Way Rule (Customer-Specific)**
- Conditional rule: Model + Carrier + Service Plan + Specific Company
- Example: CORD company needs custom firewall rules → `fw_acl=[CORD custom rules]` only for CORD's devices on Model i-22 + ATT + ATM
- Use case: Customer exceptions to service plan policies

**Level 3: Company Override**
- Customer portfolio-wide setting from Company Layer
- Example: ACME Corp sets DNS = 10.1.1.1 for all their devices
- Use case: Customer operational standards, network integration

**Level 4: Three-Way Rule**
- Conditional rule: Model + Carrier + Service Plan
- Applies to ALL customers with matching Model+Carrier+ServicePlan combination
- Example: All devices with Model i-22 + ATT + ATM get `traffic_threshold=350MB` (service plan policy baseline)
- Use case: Service plan features, model+carrier+plan specific configurations that apply to all customers

**Level 5: Service Plan Layer**
- Service plan default value
- Example: ATM plan has `data_limit=unlimited`
- Use case: Service tier differentiation

**Level 6: Two-Way Rule**
- Conditional rule: Model + Carrier
- Example: Model i-22 + VZW always has `advanced=1` enabled
- Use case: Carrier-specific features per model

**Level 7: Carrier Layer**
- Carrier default value
- Example: Verizon has `apn=vzwinternet`
- Use case: Carrier network settings

**Level 8: Model Layer**
- Model default value
- Example: Model i-22 has `alarm_io_config=[specific settings]`
- Use case: Hardware-specific configurations

**Level 9: Global Layer**
- System-wide default value
- Example: `dns_primary=8.8.8.8` (applies to all devices unless overridden)
- Use case: Infrastructure defaults

**Level 10: Schema Default**
- Hardcoded default value defined in parameter schema
- Example: `reboot_enable=false` (schema default)
- Use case: Fallback when no layer provides a value

**Level 11: Required Validation** *(Lowest Priority)*
- If parameter is required and no value found at any level → ERROR, cannot generate config
- System prevents generating invalid configurations

**Resolution Example Scenario:**

For Device #12345 with attributes:
- Company: ACME Corp
- Model: i-22
- Carrier: ATT
- Service Plan: ATM

Resolving parameter `dns_static`:

```
Level 1: Device override for #12345? → No value set, continue
Level 2: Four-way customer rule (i-22+ATT+ATM+ACME)? → No rule found, continue
Level 3: Company override for ACME? → YES! Value = 10.1.1.1 ✅ STOP HERE
(Levels 4-11 not checked because Level 3 provided value)

Result: dns_static = 10.1.1.1
Source Attribution: "Company Layer - ACME Corp"
```

If Company Layer didn't have a value, resolution would continue:
```
Level 3: Company override? → No value, continue
Level 4: Three-way rule (i-22+ATT+ATM)? → No rule, continue
Level 5: Service Plan ATM? → No value, continue
Level 6: Two-way rule (i-22+ATT)? → No rule, continue
Level 7: Carrier ATT? → No value, continue
Level 8: Model i-22? → No value, continue
Level 9: Global Layer? → YES! Value = 8.8.8.8 ✅ STOP HERE

Result: dns_static = 8.8.8.8
Source Attribution: "Global Layer"
```

**Key Capabilities:**
- **Conditional Rules Integration**: Rules (Levels 2, 4, 6) evaluated based on device attributes; override layer inheritance when conditions match
- **NULL Override Handling**: Explicitly cleared values (row with NULL) prevent inheritance from higher layers
- **Inheritance Resolution**: "Not set" (no row) means inherit from next level up
- **Source Attribution**: Every resolved value tracks its source for troubleshooting ("DNS: 8.8.8.8 (Level 9: Global)")
- **Required Key Validation**: System enforces that required parameters have values before generating config
- **Config File Generation**: Produces device-expected format with all resolved values
- **Version Identifier**: Generates version hash/timestamp for change detection
- **Device Sync Integration**: Integrated with existing device check-in process

**✅ Data Model & Storage**
- Database schema for config keys, layer values, and device mappings
- "No row = not set" vs "row with NULL = explicitly cleared" distinction
- Efficient querying for config resolution
- Audit trail storage (all changes tracked)
- Version history and rollback data

---

#### Layer Configuration Interfaces

 Single-factor layers have many-to-many relationships:
  Global Layer ────┐
  Model Layer ─────┼──────┐
  Carrier Layer ───┼──────┼──────┐
  Service Plan ────┼──────┼──────┼──────┐
  Company Layer ───┼──────┼──────┼──────┼──────┐
                   └──────┴──────┴──────┴──────┴───→ DEVICE

**✅ Global Layer Configuration (Admin Only)**
- Interface to set values for all 600+ config keys
- Visual indication of required vs optional keys
- Validation on save (required keys must have values)
- Preview affected devices count
- Bulk value update capability
- Export/import global config for backup


**✅ Model Layer Configuration (Admin Only)**
- Configuration interface per model (i-22, i-52, 4100, Origin, etc.)
- Show only keys available at Model layer (per schema)
- Display inherited values from Global Layer only (grayed out with source label: "Global")
- For parameters with conditional rules (`has_two_way_rules`, `has_three_way_rules`, etc.): Show indicator "⚠️ Uses Conditional Rules" with link to Conditional Rules Management filtered view
- Allow overrides for available keys
- Support NULL (explicitly clear inherited value)
- Preview: "X devices of this model will be affected"
- (Copy config from one model to another (template feature))

**✅ Carrier Layer Configuration (Admin Only)**
- Configuration interface per carrier (Verizon, T-Mobile, AT&T, etc.)
- Show only keys available at Carrier layer
- Display inherited values from Global Layer only (grayed out with source label: "Global")
- For parameters with conditional rules: Show indicator "⚠️ Uses Conditional Rules" with link to Conditional Rules Management filtered view
  - **Note**: Cannot show "inherited from Model" because there are multiple models. To set values for specific Carrier+Model combinations, use Two-Way Rules (Conditional Rules interface)
- Allow overrides and NULL clearing
- Contextual access to Two-Way Rules involving this carrier (Model+Carrier, Carrier+ServicePlan, Carrier+Customer)
- Preview affected devices count

**✅ Service Plan Layer Configuration (Admin Only)**
- Configuration interface per service plan
- Show only keys available at Service Plan layer
- Display inherited values from Global Layer only (grayed out with source label: "Global")
- For parameters with conditional rules: Show indicator "⚠️ Uses Conditional Rules" with link to Conditional Rules Management filtered view
  - **Note**: Cannot show "inherited from Model/Carrier" because there are multiple models and carriers. To set values for specific combinations:
    - ServicePlan + Model → Two-Way Rule
    - ServicePlan + Carrier → Two-Way Rule
    - ServicePlan + Model + Carrier → Three-Way Rule
- Allow overrides and NULL clearing
- Contextual access to Conditional Rules involving this service plan (Two-Way: Carrier+Plan, Model+Plan, Plan+Customer; Three-Way: Model+Carrier+Plan)
- Preview affected devices count

**Service Plans:**
- ATM - Unlimited Transactions
- Tier 1 (Up to 3GB)
- Tier 2 (Up to 7GB)
- Tier 3 (Up to 10GB)
- SUPER TIER (Up to 35GB)
- Rundash
- Business Internet

**✅ Company Layer Configuration**
- **Admin View**: Full access to all keys available at Company layer
- **Customer View (Phase 2+)**: Only customer-configurable keys - **NOT AVAILABLE IN PHASE 1**
- Display inherited values from Global Layer only (grayed out with source label: "Global")
- For parameters with conditional rules: Show indicator "⚠️ Uses Conditional Rules" with link to Conditional Rules Management filtered view
  - **Note**: Cannot show "inherited from Model/Carrier/ServicePlan" because companies have devices with multiple different combinations. To set customer-specific values:
    - Customer + Model → Two-Way Rule (Model+Customer)
    - Customer + Carrier → Two-Way Rule (Carrier+Customer)
    - Customer + ServicePlan → Two-Way Rule (ServicePlan+Customer)
    - Customer + Model + Carrier + ServicePlan → Four-Way Rule (customer-specific exceptions)
- Visual indicators: inherited (green), overridden (blue), cleared (gray)
- Allow overrides and NULL clearing
- Contextual access to Conditional Rules for this company (Two-Way: Model+Customer, Carrier+Customer, Plan+Customer; Four-Way: Model+Carrier+Plan+Customer)
- Confirmation dialog with preview before applying changes
- Validation feedback (inline and on save)

**✅ Device Layer Configuration**
- **Admin View**: Full access to all keys available at Device layer
- **Customer View**: Only customer-configurable keys for own devices
- Display inherited values with full source attribution (complete 11-level resolution)
  - **Why this works**: Each device has specific Model, Carrier, ServicePlan, and Company, so the full conditional rules resolution algorithm can execute
  - Example sources: "Global", "Model Layer (i-22)", "Two-Way Rule (Model=i-22 + Carrier=VZW)", "Three-Way Rule (Model=i-22 + Carrier=VZW + Plan=ATM)", "Company Layer (CORD)", etc.
- Allow overrides and NULL clearing
- Apply changes to single device
- Confirmation dialog before applying
- Validation feedback

**Reusable Component Pattern:**
All layer editors share common UI components with configuration-driven differences:
- Key list display (filtered by layer availability)
- Value editor (based on data type: text, IP, integer, boolean, dropdown, time picker)
- Inherited value display (source attribution label)
- Override/clear controls
- Validation feedback
- Save/cancel actions

---

#### User Interfaces

**✅ Device Effective Config View**
- Show final rendered config for any device (all 600+ keys)
- Source attribution for each key showing complete 11-level resolution result:
  - "dns_primary: 8.8.8.8 (Global)"
  - "mqtt_enable: 1 (Two-Way Rule: Model=i-22 + Carrier=VZW)"
  - "fw_acl: [whitelist] (Three-Way Rule: Model=4500 + Carrier=ATT + Plan=ATM)"
  - "lan_ip: 10.1.50.100 (Device Override)"
- Visual indicators by source layer (color coding)
- Highlight overridden values vs inherited values
- Show NULL values as "Not Set" or "Disabled" with source
- Filter/search within config keys
- Export effective config (for troubleshooting)
- Access control:
  - Admin/Operations: see all keys
  - Customer: see only customer-visible keys (customer-configurable + read-only allowed keys)
- Read-only view (cannot edit here, must go to layer editor)

**✅ Customer Self-Service Portal (Phase 1: Device-Level Only)**
- Simplified interface for customer administrators
- **Phase 1 Access**: Own Devices only (Device-level configuration)
- **Phase 2+ Access**: Own Company + Own Devices (Company-wide + Device-level configuration)

**Phase 1 Capabilities:**
- **Device Config Page**:
  - Individual device configuration only
  - Show only customer-configurable keys available at Device layer
  - Display inherited values with source attribution (full 11-level resolution for this specific device)
  - Shows sources like: "Global", "Company Layer", "Device Override", etc.
  - For parameters with conditional rules: Shows "Two-Way Rule", "Three-Way Rule", etc. as source
  - UI-driven controls (dropdowns for enums, validated text inputs, time pickers, IP address inputs)
  - Preview and confirmation
- **Safety Features**:
  - Cannot enter invalid values (strict validation)
  - Cannot access keys marked "not customer configurable"
  - Cannot download raw config files
  - Cannot see system layers (Global, Model, Carrier, Service Plan)
  - **Cannot access Company Layer configurations in Phase 1**
  - Preview before apply with affected device count
  - Confirmation dialogs for all changes

**Phase 2+ Enhancements (Out of Scope for Phase 1):**
- **Company Config Page** (portfolio-wide settings):
  - Portfolio-wide settings for all customer devices
  - Show only customer-configurable keys available at Company layer
  - Display inherited values from Global Layer only (system baseline)
  - For parameters with conditional rules: Show "⚠️ Managed via Conditional Rules" (customer cannot modify these in self-service portal)
  - UI-driven controls (dropdowns for enums, validated text inputs, time pickers, IP address inputs)
  - Preview: "This will apply to 500 devices"
  - Confirmation dialog with summary before applying
  - Success/error feedback

**✅ Admin Configuration Dashboard**
- Overview of all layers
- Quick navigation to each layer config interface
- System health indicators (devices with invalid configs, required keys missing, etc.)
- Recent changes audit log
- Migration status (X devices on new system, Y on old system)

---

#### Validation & Safety Systems

**✅ Multi-Level Validation**
- **Schema Validation**:
  - Key must have unique name
  - Must specify data type
  - Must specify at least one available layer
  - Required keys must be available at Global layer
  - Validation rules must be valid (regex, ranges, formats)
- **Layer Config Validation**:
  - Can only set keys available at this layer (per schema)
  - Value must match specified data type
  - Value must pass validation rules (format, range, allowed values)
  - IP addresses must be valid IPv4 format
  - Time values must be valid 24-hour format
  - Integer values must be within specified ranges
- **Required Key Validation**:
  - When rendering effective config, all required keys must have values
  - If required key is NULL after walking all layers → error, cannot generate config
  - Admin warned at Global layer if required keys are not set
- **Permission Validation**:
  - Customer can only access own Company/Devices
  - Customer can only modify customer-configurable keys
  - Operations cannot modify Global/Model/Carrier layers
  - All layer access checked on read and write
- **Preview & Confirmation**:
  - Show affected device count before applying changes
  - Require confirmation for changes affecting >10 devices
  - Require double confirmation for changes affecting >1000 devices

**✅ Audit Trail & Logging**
- Log all config changes (key, old value, new value, layer, user, timestamp, reason)
- Log effective config generation events
- Log validation failures
- Log customer access and modifications
- Searchable audit log interface
- Export audit logs for compliance

---

#### Change Detection & Versioning

**✅ Automated Config Versioning**
- Generate unique version identifier when any layer changes
- Version format: timestamp-based or hash-based (TBD with engineering)
- Track expected_version per device (what config device should have)
- Track current_version per device (what config device reports)
- Automatic version calculation when layer configs change

**✅ Affected Device Identification**
- When Global layer changes: all devices affected
- When Model layer changes: only devices of that model affected
- When Carrier layer changes: only devices on that carrier affected
- When Service Plan layer changes: only devices on that plan affected
- When Company layer changes: only devices in that company affected
- When Device layer changes: only that specific device affected
- Update expected_version for all affected devices

**✅ Config Push Integration**
- Integration with existing device check-in process
- On device check-in, compare current_version vs expected_version
- If mismatch: device needs new config
- Generate effective config on-demand or pre-generate and cache (TBD)
- Push config to device via existing mechanism
- Device updates current_version after successful config application
- Support existing triggers: service plan change, Wi-Fi update, manual push

**✅ Change Impact Preview**
- Before saving layer config changes, show preview:
  - Number of devices affected
  - List of affected device IDs (expandable, first 100 shown)
  - Breakdown by model/carrier/company if Global/Model/Carrier change
- "Dry run" mode: calculate impact without committing changes

---

#### Migration & Parallel Operation

**✅ Device Migration System**
- Flag per device: "config_system" = "old" | "new"
- Admin interface to select devices for migration:
  - By device ID (manual selection or CSV upload)
  - By device ID range (e.g., devices 1-1000)
  - Start small (5-10 devices), gradually expand
- Migration status dashboard:
  - Total devices: 100,000
  - On old system: 99,000
  - On new system: 1,000
  - Migration progress chart
- Rollback capability: switch device back to old system if issues

**✅ Parallel Config Generation**
- Old system continues to generate configs for flagged devices
- New system generates configs for migrated devices
- No interference between systems
- Both systems write to separate columns or tables if needed
- Ability to compare old vs new config output for validation

**✅ Config System Import/Migration**
- One-time import of 600+ keys from existing config files into schema
- Import tool workflow:
  1. Analyze existing config files, extract unique keys
  2. For each key, infer: data type, common values, variance across files
  3. Present to admin for review/classification
  4. Admin assigns: layer availability, required flag, customer configurable flag
  5. Import creates key schema records
- Import existing config file values into Global layer (baseline)
- Map model-specific variations to Model layer
- Map customer-specific variations to Company/Device layers
- Validation: ensure all current devices can resolve valid configs after import

---

#### Integration & Compatibility

**✅ Integration with Existing Systems**
- Device check-in service (UDP server)
- Device status tracking
- Service plan management (triggers config changes)
- Company management (new companies get default configs)
- Device provisioning (new devices get effective configs)
- Wi-Fi management (existing feature continues to work, values stored in Device layer)

**✅ Backward Compatibility**
- Devices on old system continue to receive configs from old files
- Check-in process supports both old and new config delivery
- Hostname validation remains for old system devices
- No disruption to existing device operations during migration

**✅ Config File Format**
- Final rendered config must match format expected by devices (InHand firmware)
- Key-value pairs in specific format (TBD with engineering, based on current file format)
- All 600+ keys included in output (even if NULL or not applicable to model)
- Device firmware ignores keys it doesn't understand

---

### Out of Scope (Future Enhancements)

**❌ Phase 1 Exclusions:**

**Customer Config Template Creation**
- Customers cannot define new config keys or parameters
- Customers cannot create templates to share with other companies
- Future: Template marketplace or preset configs

**Real-Time Config Push**
- Not implementing WebSocket or push notification system
- Continue using check-in based delivery model (device checks in, system responds)
- Config push happens when device checks in (passive model)
- Future: Active push via cellular/network connection

**Advanced Config Analytics**
- Not building analytics on config usage, trends, or optimization suggestions
- No reporting on "most common overrides" or "unused keys"
- No machine learning for config recommendations
- Future: Analytics dashboard showing config patterns and optimization opportunities

**Configuration API (Phase 1)**
- REST API for config management deferred to Phase 2
- Phase 1: UI-driven only
- Future: API for external integrations, automation, scripting

**Config History & Rollback (Advanced)**
- Phase 1: Audit trail of changes (what changed, when, by whom)
- No "rollback to config as of DATE" feature at device level
- No "compare config versions" at layer level
- Future: Full versioned history with rollback to any point in time

**Config Validation Testing**
- No built-in "test config on device" or "simulate config before applying"
- Admins rely on preview and gradual migration for safety
- Future: Sandbox/test mode where configs can be tested on virtual devices

**Advanced Scheduling**
- No "schedule config change for future date/time"
- Changes applied immediately when saved
- Future: Scheduled config deployments

---


---

## Functional Requirements

See [Functional Requirements Document](./Config_Management_System_PRD_Functional_Requirements.md)

---

## Technical Design

_To be completed in next iteration_

---

## Technical Design

_To be completed in next iteration_

---

## Testing Requirements

_To be completed in next iteration_

---

## Dependencies and Risks

_To be completed in next iteration_

---

## Implementation Plan

_To be completed in next iteration_

---

## Success Metrics

_To be completed in next iteration_

---

## Open Questions

_To be completed in next iteration_

---

## Next Steps

1. **Review and Approve Executive Summary, Background, Goals, and Target Users** - Stakeholders - by TBD
2. **Define User Stories and Acceptance Criteria** - Product Team - by TBD
3. **Define Scope (In/Out)** - Product + Engineering - by TBD
4. **Draft Functional Requirements** - Product Team - by TBD
5. **Technical Design Review** - Engineering Team - by TBD

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-02 | Aksana Rahouski | Initial PRD created - Executive Summary, Background, Goals, Target Users populated from Meeting 2 discussion |
| 1.1 | 2026-01-02 | Aksana Rahouski | Added comprehensive Scope (In/Out) and Functional Requirements (FR-1 through FR-10) sections |
| 1.2 | 2026-01-03 | Aksana Rahouski | Added Model+Carrier conditional dependency requirements based on config file analysis. Updated FR-1.1a, FR-1.3, FR-3.1, FR-10.1, FR-10.6-10.8. Added new table config_model_carrier_rules. Multiple parameters identified requiring dual-condition logic. |
| 1.3 | 2026-01-03 | Aksana Rahouski | **MAJOR UPDATE**: Replaced specific Model+Carrier logic with generic Conditional Rules Framework. Added support for Two-Way (Model+Carrier) AND Three-Way (Model+Carrier+ServicePlan) conditional rules. Added comprehensive FR-3a section for Conditional Rules Management (8 sub-requirements). Updated FR-1.1a, FR-3.1 with generic multi-factor logic. Added two new tables: config_conditional_rules_2way and config_conditional_rules_3way. Updated audit logging. Framework now supports extensible multi-factor conditional logic for any parameter. Based on complete analysis of two-way and three-way parameter dependencies. |
| 1.4 | 2026-01-05 | Aksana Rahouski | **MAJOR UPDATE**: Added Four-Way Conditional Rules support to framework. Updated Conditional Rules Framework to support Model+Carrier+ServicePlan+Company combinations with nullable Company field enabling both general baselines (all customers) and customer-specific exceptions. Updated Config Resolution Engine to 11-level priority hierarchy (from 10). Added FR-10.6b for new config_conditional_rules_4way table. Updated FR-3a.1, FR-3a.2, FR-3a.5, FR-3a.7 with four-way rule creation, listing, coverage, and matrix visualization interfaces. Updated audit logging to include conditional_rule_4way layer type. Added hierarchical rule management UI patterns. Based on customer config analysis identifying parameters requiring customer-specific exceptions to service plan policies. |
| 1.5 | 2026-01-12 | Aksana Rahouski | **MAJOR UPDATE**: Expanded Two-Way Rules to support ALL 6 two-factor combinations (not just Model+Carrier). Analysis of actual device configs revealed need for: Carrier+ServicePlan (carrier data policies by plan), Model+ServicePlan (model pricing tier by plan), Carrier+Customer (customer carrier agreements), Model+Customer (customer model configs), ServicePlan+Customer (customer plan exceptions). Added sub-priority ordering within Level 6 (6.1-6.6). Updated Executive Summary, Scope section, FR-3a.1 (rule creation workflow), FR-3.1 (11-Level Priority Hierarchy with detailed examples). Added comprehensive real-world examples for each sub-priority. Updated Rule Type Selection Guide with decision tree. Updated database indexes for all 6 two-way combinations. Based on analysis of production config files showing traffic_day_threshold, mqtt_enable, and other parameters requiring multiple two-factor patterns beyond Model+Carrier. |

---

**Document Status:** Draft for Review
**Next Review Date:** TBD
**Approvals Required:**
- [ ] Product Manager (Aksana Rahouski)
- [ ] Engineering Lead
- [ ] QA Lead
- [ ] Business Stakeholders (APW - Adam Curcie)
- [ ] Orases Leadership

---

## Appendix

### A. Key Definitions

**Configuration Layers (Hierarchy):**
1. **Global**: Parameters that apply to all devices (e.g., DNS servers, time servers, security settings)
2. **Model**: Parameters specific to device models (e.g., i-22 alarm I/O settings, 4100 modem settings)
3. **Carrier**: Parameters specific to cellular carriers (e.g., Verizon APN, T-Mobile settings)
4. **Service Plan**: Parameters tied to APW service offerings (e.g., data limits, features)
5. **Company**: Customer portfolio-wide settings (e.g., reboot schedules, naming conventions)
6. **Device**: Individual device overrides (e.g., specific LAN IP, custom Wi-Fi networks)

**Cascade/Inheritance Model:**
- Configurations flow from general (Global) to specific (Device)
- More specific levels override values from more general levels
- Device receives final "effective config" by resolving all layers
- Example: DNS set at Global (8.8.8.8) → Company overrides for one customer (10.1.1.1) → That company's devices use 10.1.1.1, all others use 8.8.8.8

**Static vs Dynamic Parameters:**
- **Static**: Values that never change (e.g., VPN settings APW doesn't use = always disabled)
- **Dynamic**: Values that can be overridden at various levels (e.g., DNS, LAN IP, scheduling)

**Source Attribution:**
- The ability to see where each config parameter value comes from
- Example: "DNS: 8.8.8.8 (Global)" or "LAN IP: 10.1.50.1 (Device Override)"
- Critical for troubleshooting and understanding effective config

**Model+Carrier Conditional Dependencies:**
- Configuration parameters whose values depend on BOTH the device model AND carrier combination
- Analysis identified multiple parameters requiring dual-condition logic (actual count will be determined during implementation)
- Examples:
  - `advanced` parameter: Only enabled (=1) for Model 22 + VZW combination; disabled (=0) for all others
  - `console_enable`: Enabled for Model 22 with DC or VZW carriers; disabled for Model 4500 with VZW
  - `mqtt_enable`: Enabled for Model 22 with DC or VZW; disabled for ATT and all Model 4500
  - `dns_static`: Different DNS servers based on model+carrier combination (VZW+Model22 uses 8.8.8.8;1.1.1.1 but VZW+Model4500 uses 8.8.8.8;8.8.4.4)
- Implementation: System uses conditional rules that override standard layer inheritance when model+carrier combination matches
- Priority: Device Override > Company Override > Model+Carrier Rule > Service Plan > Standard Layer Resolution
- See [Configuration Parameter Dependencies Analysis](/Users/aksana/Documents/Projects/WATM/Config_Parameter_Dependencies_Analysis.md) for complete list and implementation details

---

END OF DOCUMENT
