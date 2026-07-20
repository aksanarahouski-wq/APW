# Research and Analysis

**Purpose:** Evidence-based analysis supporting architectural design decisions
**Status:** Analysis Complete - All Key Decisions Validated
**Last Updated:** March 1, 2026

---

## Overview

This folder contains comprehensive research and analysis of production configuration files, parameter dependencies, and rule combination patterns. All analysis documents provide evidence supporting the validated hierarchical configuration architecture.

---

## Folder Structure

### Configuration_Analysis/
**Purpose:** System-wide configuration analysis and pattern identification

**Key Document:**
- **Configuration_Parameter_Analysis.md** - Analysis of 600+ configuration parameters across all production config files

### Parameter_Analysis/
**Purpose:** Detailed analysis of parameter behavior across different layers

**Key Documents:**
- **Multi_Layer_Parameters_Analysis.md** - Identifies parameters that can be set at multiple layers with real-world examples (firewall rules, MQTT settings, data thresholds)
- **Config_Parameters_By_Layer_Analysis_full.md** - Complete parameter-by-layer breakdown showing which parameters belong at which layers

**Key Findings:**
- Firewall rules (fw_acl) need Service Plan → Customer → Device layers
- MQTT/Device Manager settings need Model → Carrier conditional logic
- Traffic thresholds need Model + Carrier + Service Plan conditional logic
- Validates need for both layer-based inheritance AND conditional rules framework

### Dependencies/
**Purpose:** Complete dependency analysis for configuration parameters

**Key Document:**
- **Config_Dependencies_Complete_Analysis.md** - Maps parameter dependencies and relationships

### Rule_Combinations/
**Purpose:** Mathematical and real-world analysis of all possible conditional rule combinations

**Key Documents:**

#### ✅ Two_Way_Rule_Combinations_Analysis.md
**Status:** VALIDATED
**Conclusion:** Support ALL 6 two-factor combinations

**Evidence:**
- `mqtt_enable` requires Model + Carrier (VZW i-22 enables Device Manager, ATT i-22 disables it)
- `traffic_day_threshold` requires Carrier + Service Plan (VZW ATM = 3584MB, ATT ATM varies by model)
- `traffic_day_threshold` also requires Model + Service Plan (i-22 ATM = 350MB, 4500 ATM = 5GB)
- Customer exceptions require Customer + other factors

**Result:** 6 Two-Way Rule types implemented with sub-priority ordering

#### ✅ Three_Way_Rule_Design_Validation.md
**Status:** VALIDATED - FINAL DESIGN
**Conclusion:** Support ONLY Model + Carrier + Service Plan (no other three-way combinations)

**Mathematical Analysis:**
- 4 factors → C(4,3) = 4 possible three-way combinations
- Evaluated: M+C+SP, M+C+Customer, M+SP+Customer, C+SP+Customer

**Real-World Evidence:**
- ATM plan firewall baselines are identical across all Model+Carrier combinations
- Customer exceptions are service-plan-specific (CORD on ATM, not on Tier1)
- Customer exceptions are Model+Carrier independent (same servers regardless)
- Pattern: Service Plan + Customer (TWO factors, not three)

**Architectural Decision:**
- Three-Way Rules define service plan BASELINES for all customers
- Four-Way Rules handle customer-specific EXCEPTIONS
- Other three-way combinations already covered by Two-Way Rules or Four-Way Rules

**Complexity Analysis:**
- Adding 3 more three-way types = 4x complexity
- No real-world benefit identified
- Would require complex sub-priority ordering
- Admin confusion about when to use which type

**Result:** ONE Three-Way Rule type (Model+Carrier+ServicePlan) confirmed optimal

#### ✅ Four_Way_Rule_Combinations_Analysis.md
**Status:** VALIDATED
**Conclusion:** Support Model + Carrier + Service Plan + Customer

**Mathematical Fact:** Only 1 possible four-way combination exists

**Use Cases:**
- Customer firewall exceptions (CORD needs additional ACL entries for RMS servers)
- Customer data limit overrides (VIP customer gets upgraded allowance)
- Customer-specific exceptions to service plan policies

**When to Use:**
- ✅ Customer-specific exception
- ✅ Applies to specific service plan
- ✅ Model matters
- ✅ Carrier matters

**When NOT to Use:**
- If model-independent → Use Two-Way Rule (Carrier+Customer or Plan+Customer)
- If carrier-independent → Use Two-Way Rule (Model+Customer or Plan+Customer)
- If plan-independent (rare) → Use Company Layer Override

**Priority:** Level 2 (just below Device Override, above Company Override)

**Frequency:** Relatively rare (estimated 10-20 rules)

**Result:** Four-Way Rules implemented for customer exceptions to Three-Way baselines

#### README.md
Summary of all rule combination analysis and design decisions

---

## Key Findings Summary

### Finding 1: Multi-Layer Parameters Are Common
**Evidence:** Multi_Layer_Parameters_Analysis.md

Production configs show extensive multi-layer usage:
- Firewall rules have Service Plan baseline + Customer exceptions + Device overrides
- MQTT settings vary by Model + Carrier
- Traffic thresholds vary by Model + Carrier + Service Plan
- Infrastructure settings (DNS, NTP) can be overridden at Company and Device layers

**Implication:** Both layer inheritance AND conditional rules are needed

### Finding 2: Different Parameters Need Different Rule Types
**Evidence:** Two_Way_Rule_Combinations_Analysis.md

Not all parameters follow the same pattern:
- MQTT: Model + Carrier
- Traffic limits: Carrier + Plan, Model + Plan, or Model + Carrier + Plan
- Customer exceptions: Plan + Customer or full Four-Way

**Implication:** Need flexible rule framework, not one-size-fits-all

### Finding 3: Three-Way Rules Are Only for Service Plan Baselines
**Evidence:** Three_Way_Rule_Design_Validation.md

Real-world analysis shows:
- Service plan baselines apply to ALL customers (Three-Way)
- Customer exceptions are specific (Four-Way)
- Other three-way patterns don't exist in production configs

**Implication:** Only ONE three-way type needed (Model+Carrier+ServicePlan)

### Finding 4: Customer Exceptions Are Model+Carrier Independent
**Evidence:** Three_Way_Rule_Design_Validation.md, Multi_Layer_Parameters_Analysis.md

Customer-specific configurations (CORD RMS servers, custom infrastructure):
- Same values across all models (i-22, 4500)
- Same values across all carriers (VZW, ATT)
- Only vary by Service Plan + Customer

**Implication:** Most customer exceptions can use Two-Way Rules (Plan+Customer), not Four-Way

### Finding 5: Most Parameters Use Simple Patterns
**Evidence:** All analysis documents

Pattern distribution:
- Global infrastructure (DNS, NTP): Global Layer (no rules needed)
- Model capabilities (I/O, alarms): Model Layer (no rules needed)
- Carrier settings (APN): Carrier Layer (no rules needed)
- Plan features: Service Plan Layer or Three-Way if varies by model/carrier
- Customer infrastructure: Company Layer Override (no rules needed)
- Complex dependencies: Conditional Rules (Two-Way, Three-Way, Four-Way)

**Implication:** System provides right level of complexity for each parameter type

---

## Validated Architecture

### Layer-Based Configuration (Single Factor)
1. Global Layer - System-wide defaults
2. Model Layer - Model-specific settings
3. Carrier Layer - Carrier-specific settings
4. Service Plan Layer - Plan features and policies
5. Company Layer - Customer portfolio-wide settings
6. Device Layer - Individual device overrides

### Conditional Rules Framework

**Two-Way Rules (6 types):**
1. Model + Carrier - Carrier features per model
2. Carrier + Service Plan - Carrier data policies by plan
3. Model + Service Plan - Model pricing tier by plan
4. Carrier + Customer - Customer carrier agreements
5. Model + Customer - Customer model configurations
6. Service Plan + Customer - Customer plan exceptions

**Three-Way Rules (1 type):**
1. Model + Carrier + Service Plan - Service plan baselines for ALL customers

**Four-Way Rules (1 type):**
1. Model + Carrier + Service Plan + Customer - Customer-specific exceptions to baselines

### 11-Level Priority Hierarchy

1. Device Override (highest)
2. Four-Way Rule
3. Company Override
4. Three-Way Rule
5. Service Plan Layer
6. Two-Way Rule (6 sub-priorities: C+SP, M+SP, M+C, C+Cu, M+Cu, SP+Cu)
7. Carrier Layer
8. Model Layer
9. Global Layer
10. Schema Default
11. Required Validation (lowest)

---

## Evidence Quality

### Production Data Analysis
✅ Real production configuration files analyzed
✅ Actual customer configs reviewed (CORD, Altech, Baum, etc.)
✅ Multiple carriers, models, and service plans examined
✅ 600+ parameters categorized and analyzed

### Mathematical Rigor
✅ All possible combinations evaluated (C(4,2)=6, C(4,3)=4, C(4,4)=1)
✅ Complexity trade-offs quantified (4x complexity increase for minimal benefit)
✅ Priority ordering mathematically sound

### Business Logic Validation
✅ Use cases mapped to rule types
✅ Customer behavior patterns analyzed
✅ Service plan policies documented
✅ Edge cases identified and solutions provided

### Architectural Soundness
✅ Database schema implications considered
✅ Resolution algorithm complexity analyzed
✅ Admin usability evaluated
✅ Maintainability assessed

---

## Design Decisions Status

| Decision | Status | Evidence Document | Confidence |
|----------|--------|-------------------|------------|
| 6 Layer Hierarchy | ✅ VALIDATED | Multi_Layer_Parameters_Analysis.md | HIGH |
| 6 Two-Way Rule Types | ✅ VALIDATED | Two_Way_Rule_Combinations_Analysis.md | HIGH |
| 1 Three-Way Rule Type | ✅ VALIDATED | Three_Way_Rule_Design_Validation.md | HIGH |
| 1 Four-Way Rule Type | ✅ VALIDATED | Four_Way_Rule_Combinations_Analysis.md | HIGH |
| 11-Level Priority | ✅ VALIDATED | All analysis documents | HIGH |

---

## Next Steps

**Research Phase:** ✅ COMPLETE

**Ready for:**
- Technical design (database schema, API design)
- Implementation planning (phased rollout)
- Development (Phase 1 MVP)

---

**Analysis Status:** COMPLETE - All Design Decisions Validated
**Last Updated:** March 1, 2026
