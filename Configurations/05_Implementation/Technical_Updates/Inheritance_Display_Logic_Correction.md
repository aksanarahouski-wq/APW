# Multi-Layer Parameter Inheritance Logic - Corrected Model

**Date:** January 12, 2026
**Version:** 1.0
**Related:** Config_Management_System_PRD.md v1.5

---

## Executive Summary

This document clarifies the **correct inheritance display logic** for the Configuration Management System's layer-based interfaces. The key insight: **single-factor layers have many-to-many relationships**, not hierarchical parent-child relationships, which fundamentally affects what "inherited values" can be displayed in each interface.

---

## The Core Problem Identified

### Original PRD Statement (INCORRECT):
- **Carrier Layer**: "Display inherited values from Global/Model (with source)"
- **Service Plan Layer**: "Display inherited values with source attribution"
- **Company Layer**: "Display inherited values from upstream layers (Global/Model/Carrier/Service Plan)"

### Why This Is Conceptually Flawed:

**Question:** When editing Carrier Layer (e.g., VZW), which Model's value should be shown as "inherited from Model"?
- There are multiple models: i-22, 4500, Origin, etc.
- Each model may have different values for the same parameter
- A carrier is used WITH multiple models simultaneously
- **There is no single "Model value" to inherit from**

**Question:** When editing Service Plan Layer (e.g., ATM), which Model? Which Carrier?
- Multiple models exist (i-22, 4500, etc.)
- Multiple carriers exist (VZW, ATT, T-Mobile, etc.)
- **There is no single value to inherit from these layers**

**Question:** When editing Company Layer (e.g., CORD), which Model/Carrier/Plan?
- Companies have devices with many different Model+Carrier+Plan combinations
- **Cannot show "inherited from Model" - there are hundreds of devices with different models**

---

## The Correct Conceptual Model

### Single-Factor Layers Have Many-to-Many Relationships

**Device Attributes (simultaneous, not hierarchical):**
- Model = i-22
- Carrier = VZW
- Service Plan = ATM
- Company = CORD

**Layer Relationships:**
```
Global Layer ────┐
                 │
Model Layer ─────┼──────┐
                 │      │
Carrier Layer ───┼──────┼──────┐
                 │      │      │
Service Plan ────┼──────┼──────┼──────┐
                 │      │      │      │
Company Layer ───┼──────┼──────┼──────┼──────┐
                 │      │      │      │      │
                 └──────┴──────┴──────┴──────┴───→ DEVICE
```

**Key Insight:** Model, Carrier, ServicePlan, and Company layers are **NOT hierarchical**. They are parallel single-factor dimensions that **converge at the device level**.

---

## Corrected Inheritance Display Logic

### Rule 1: Single-Factor Layers → Inherit from Global Only

**Layers:** Model, Carrier, Service Plan, Company

**Can Display:**
- ✅ Inherited values from **Global Layer only** (there's only one Global baseline)
- ✅ Overridden values set at this layer
- ✅ Indicator that parameter uses Conditional Rules (with link to Conditional Rules interface)

**Cannot Display:**
- ❌ "Inherited from Model" (which model? there are many!)
- ❌ "Inherited from Carrier" (which carrier? there are many!)
- ❌ "Inherited from Service Plan" (which plan? there are many!)
- ❌ "Inherited from Company" (which company? there are many!)

**Why:** These layers have many-to-many relationships. To set a value for a specific combination of factors = Conditional Rule.

---

### Rule 2: Device Layer → Full Resolution Possible

**Layer:** Device

**Can Display:**
- ✅ Full 11-level priority hierarchy resolution
- ✅ Complete source attribution for each parameter

**Why This Works:**
Each device has **specific** Model, Carrier, ServicePlan, and Company values. The resolution algorithm can execute:

**Example Device:**
- Model = i-22
- Carrier = VZW
- Service Plan = ATM
- Company = CORD

**Parameter Resolution Examples:**
```
dns_primary: 8.8.8.8
  Source: Global Layer

mqtt_enable: 1
  Source: Two-Way Rule (Model=i-22 + Carrier=VZW)

traffic_day_threshold: 3584MB
  Source: Two-Way Rule (Carrier=VZW + ServicePlan=ATM)

fw_acl: [40 whitelist rules]
  Source: Three-Way Rule (Model=i-22 + Carrier=VZW + ServicePlan=ATM)

lan_ip: 10.1.50.100
  Source: Device Override
```

---

### Rule 3: Where Multi-Factor Values Are Managed

**Interface:** Conditional Rules Management

**This is where you define:**
- **Two-Way Rules** (6 combinations):
  - Model + Carrier → "IF Model=i-22 AND Carrier=VZW THEN mqtt_enable=1"
  - Carrier + Service Plan → "IF Carrier=VZW AND Plan=ATM THEN traffic_day_threshold=3584MB"
  - Model + Service Plan → "IF Model=i-22 AND Plan=ATM THEN traffic_day_threshold=350MB"
  - Carrier + Customer → "IF Carrier=VZW AND Customer=VIP_Corp THEN traffic_day_threshold=10GB"
  - Model + Customer → "IF Model=i-22 AND Customer=Miele THEN digitalio_config=[custom]"
  - Service Plan + Customer → "IF Plan=ATM AND Customer=Premium THEN traffic_day_threshold=1000MB"

- **Three-Way Rules:**
  - Model + Carrier + Service Plan → "IF Model=4500 AND Carrier=ATT AND Plan=ATM THEN fw_acl=[rules]"

- **Four-Way Rules:**
  - Model + Carrier + Service Plan + Customer → "IF Model=i-22 AND Carrier=VZW AND Plan=ATM AND Customer=CORD THEN lan_gw=custom"

**Why:** Setting a value for a specific combination of 2+ factors IS a Conditional Rule, not a simple layer override.

---

## PRD Updates Made (v1.5)

### Updated Sections:

#### 1. Model Layer Configuration (lines 591-599)
**Added:**
- "Display inherited values from Global Layer only"
- Indicator for parameters with conditional rules: "⚠️ Uses Conditional Rules"
- Link to Conditional Rules Management filtered view

#### 2. Carrier Layer Configuration (lines 601-609)
**Changed:** "Display inherited values from Global/Model (with source)"
**To:** "Display inherited values from Global Layer only"

**Added:**
- Indicator for parameters with conditional rules
- **Note explaining why**: "Cannot show 'inherited from Model' because there are multiple models. To set values for specific Carrier+Model combinations, use Two-Way Rules"
- Contextual access to all Two-Way Rules involving this carrier (Model+Carrier, Carrier+ServicePlan, Carrier+Customer)

#### 3. Service Plan Layer Configuration (lines 611-622)
**Changed:** "Display inherited values with source attribution"
**To:** "Display inherited values from Global Layer only"

**Added:**
- Indicator for parameters with conditional rules
- **Note explaining combinations:**
  - ServicePlan + Model → Two-Way Rule
  - ServicePlan + Carrier → Two-Way Rule
  - ServicePlan + Model + Carrier → Three-Way Rule
- Contextual access to all Conditional Rules involving this service plan

#### 4. Company Layer Configuration (lines 633-647)
**Changed:** "Display inherited values from upstream layers (Global/Model/Carrier/Service Plan)"
**To:** "Display inherited values from Global Layer only"

**Added:**
- Indicator for parameters with conditional rules
- **Note explaining combinations:**
  - Customer + Model → Two-Way Rule (Model+Customer)
  - Customer + Carrier → Two-Way Rule (Carrier+Customer)
  - Customer + ServicePlan → Two-Way Rule (ServicePlan+Customer)
  - Customer + Model + Carrier + ServicePlan → Four-Way Rule
- Contextual access to all Conditional Rules for this company

#### 5. Device Layer Configuration (lines 649-658)
**Clarified:** "Display inherited values with full source attribution (complete 11-level resolution)"

**Added explanation:**
- **Why this works**: Each device has specific Model, Carrier, ServicePlan, and Company
- Example sources showing Conditional Rules as sources

#### 6. Device Effective Config View (lines 673-688)
**Enhanced examples** to show Conditional Rules as sources:
```
"mqtt_enable: 1 (Two-Way Rule: Model=i-22 + Carrier=VZW)"
"fw_acl: [whitelist] (Three-Way Rule: Model=4500 + Carrier=ATT + Plan=ATM)"
```

#### 7. Customer Self-Service Portal (lines 690-709)
**Company Config Page:**
- Changed: "Display inherited values from system layers"
- To: "Display inherited values from Global Layer only (system baseline)"
- Added: "For parameters with conditional rules: Show '⚠️ Managed via Conditional Rules' (customer cannot modify)"

**Device Config Page:**
- Changed: "Display inherited values (including company overrides)"
- To: "Display inherited values with source attribution (full 11-level resolution for this specific device)"
- Added examples of sources including Conditional Rules

---

## UI Implementation Guidance

### Single-Factor Layer Interfaces (Model, Carrier, Service Plan, Company)

**Display Logic:**

```
For each parameter:

  IF parameter has value at this layer:
    Show: [Value] (Set at this layer) [BLUE indicator]

  ELSE IF parameter has value at Global layer:
    Show: [Value] (Inherited from Global) [GREEN indicator]

  ELSE:
    Show: [Not Set] [GRAY indicator]

  IF parameter has conditional rules enabled (has_two_way_rules, has_three_way_rules, etc.):
    Show: ⚠️ This parameter uses Conditional Rules
    Link: [View Rules] → Filtered Conditional Rules Management view

  Action buttons: [Override] [Clear Override]
```

**DO NOT attempt to show:**
- "Inherited from Model" when editing Carrier
- "Inherited from Carrier" when editing Service Plan
- "Inherited from Model/Carrier/Plan" when editing Company

**Instead:** Link to Conditional Rules interface where multi-factor combinations are managed.

---

### Device Layer Interface

**Display Logic:**

```
For each parameter:

  Execute full 11-level resolution algorithm for this specific device

  Show: [Value] (Source: [detailed source attribution])

  Possible sources:
    - "Global Layer"
    - "Model Layer (i-22)"
    - "Carrier Layer (VZW)"
    - "Service Plan Layer (ATM)"
    - "Company Layer (CORD)"
    - "Two-Way Rule (Model=i-22 + Carrier=VZW)"
    - "Two-Way Rule (Carrier=VZW + Plan=ATM)"
    - "Three-Way Rule (Model=i-22 + Carrier=VZW + Plan=ATM)"
    - "Four-Way Rule (Model=i-22 + Carrier=VZW + Plan=ATM + Company=CORD)"
    - "Company-Specific Device Override"
    - "Device Override"

  Color code by source type

  Action buttons: [Override] [Clear Override]
```

---

### Device Effective Config View

Same as Device Layer Interface, but **read-only**.

**Purpose:** Troubleshooting and auditing - show exactly where each parameter value came from for this specific device.

---

## Database Query Examples

### Single-Factor Layer Interface Query

**Example: Editing Carrier Layer for VZW**

```sql
-- Get parameter value for Carrier Layer
SELECT value, 'Carrier Layer' as source
FROM config_values
WHERE carrier_id = 1  -- VZW
  AND model_id IS NULL
  AND service_plan_id IS NULL
  AND company_id IS NULL
  AND config_key_id = 123

UNION

-- Get Global value if no Carrier override
SELECT value, 'Global' as source
FROM config_values
WHERE model_id IS NULL
  AND carrier_id IS NULL
  AND service_plan_id IS NULL
  AND company_id IS NULL
  AND config_key_id = 123

LIMIT 1;
```

**DO NOT query for Model-specific values** - there are multiple models!

---

### Device Layer Interface Query

**Example: Device with Model=i-22, Carrier=VZW, Plan=ATM, Company=CORD**

```sql
-- Execute full 11-level resolution
-- (Pseudocode - actual implementation would use stored procedure)

-- Level 10: Device Override
SELECT value, 'Device Override' as source, 10 as priority
FROM config_values
WHERE device_id = 456 AND config_key_id = 123

UNION ALL

-- Level 9: Company-Specific Device Override
SELECT value, 'Company-Specific Device Override' as source, 9 as priority
FROM config_values
WHERE device_id = 456 AND company_id = 5 AND config_key_id = 123

UNION ALL

-- Level 8: Four-Way Rule
SELECT value, 'Four-Way Rule' as source, 8 as priority
FROM config_values
WHERE model_id = 1 AND carrier_id = 1 AND service_plan_id = 2 AND company_id = 5
  AND config_key_id = 123

-- [Continue through all 11 levels...]

ORDER BY priority DESC
LIMIT 1;
```

---

## Testing Scenarios

### Test 1: Single-Factor Layer Interface Should NOT Show Multi-Factor Values

**Test Case:** Editing Carrier Layer for VZW

**Given:**
- Global: `mqtt_enable = 0`
- Model Layer (i-22): `mqtt_enable = NULL`
- Two-Way Rule (Model=i-22 + Carrier=VZW): `mqtt_enable = 1`
- Carrier Layer (VZW): `mqtt_enable = NULL`

**Expected Display in Carrier Layer interface:**
```
mqtt_enable: 0 (Inherited from Global)
⚠️ This parameter uses Conditional Rules [View Rules]
```

**Should NOT show:** "mqtt_enable: 1 (from Two-Way Rule)" because we're editing single-factor Carrier Layer, not a specific Model+Carrier combination.

---

### Test 2: Device Layer Interface Should Show Full Resolution

**Test Case:** Device with Model=i-22, Carrier=VZW, Plan=ATM, Company=CORD

**Given same config as Test 1**

**Expected Display in Device Layer interface:**
```
mqtt_enable: 1 (Two-Way Rule: Model=i-22 + Carrier=VZW)
```

**Why different:** Device has specific Model=i-22 and Carrier=VZW, so the Two-Way Rule applies and overrides Global.

---

### Test 3: Conditional Rules Interface Shows All Combinations

**Test Case:** Viewing Two-Way Rules for `mqtt_enable`

**Expected Display:**
```
Conditional Rules for: mqtt_enable

Two-Way Rules:
  ✓ Model=i-22 + Carrier=VZW → Value: 1
    Applies to: 150 devices

  ✓ Model=i-22 + Carrier=ATT → Value: 0
    Applies to: 75 devices
```

---

## Migration Impact

### Existing Code That May Need Updates:

1. **Layer Editor Components:**
   - Remove logic that attempts to show "inherited from Model" in Carrier editor
   - Remove logic that attempts to show "inherited from Model/Carrier" in Service Plan editor
   - Remove logic that attempts to show "inherited from Model/Carrier/Plan" in Company editor
   - Simplify to show Global inheritance only

2. **Add Conditional Rules Indicators:**
   - Check parameter's `has_two_way_rules`, `has_three_way_rules`, `has_four_way_rules` flags
   - Display "⚠️ Uses Conditional Rules" indicator
   - Provide contextual links to filtered Conditional Rules views

3. **Device Resolution Logic:**
   - Already implemented correctly (full 11-level resolution)
   - Enhance source attribution to show Conditional Rule details (not just "Two-Way Rule" but "Two-Way Rule: Model=i-22 + Carrier=VZW")

---

## Key Takeaways

1. **Single-factor layers inherit from Global ONLY** - they cannot inherit from other single-factor layers due to many-to-many relationships

2. **Setting values for specific factor combinations = Conditional Rules** - this is where Model+Carrier, Carrier+Plan, etc. are managed

3. **Device layer is where full resolution happens** - because each device has specific Model, Carrier, Plan, Company values

4. **UI must guide users to Conditional Rules interface** for multi-factor scenarios, not try to show "inherited from Model" in single-factor editors

5. **The 11-level priority hierarchy is a resolution algorithm**, not a UI display hierarchy - it executes at the device level only

---

## Glossary

**Single-Factor Layer:** Configuration layer that sets values based on ONE device attribute only (Model, Carrier, Service Plan, or Company)

**Conditional Rule:** Multi-factor rule that sets values based on specific combinations of 2+ device attributes

**Resolution Algorithm:** The 11-level priority hierarchy that determines final parameter value for a device

**Source Attribution:** Display showing where a parameter value came from (which layer or rule)

**Many-to-Many Relationship:** A device can have Model=i-22 AND Carrier=VZW simultaneously; these layers don't have parent-child relationships

---

**End of Document**
