# Two-Way Rule Combinations Analysis

**Date:** January 12, 2026
**Question:** Do we need to support ALL two-factor combinations, or just Model + Carrier?
**Source:** Configuration files in `Files/model_carrier_serviceplan/`, `Files/carrier_VZW_diffmodels/`, `Files/model_I22_diffCarriers/`

---

## Executive Summary

**Answer: YES - We need to support MULTIPLE two-factor combinations beyond just Model + Carrier.**

Analysis reveals that different parameters require different two-factor combinations:
- ✅ **Model + Carrier** (confirmed - mqtt_enable, advanced, dual_sim_main)
- ✅ **Carrier + Service Plan** (NEW - traffic_day_threshold for I-22 ATM vs I-22 Tier1 on same carrier)
- ✅ **Model + Service Plan** (NEW - traffic_day_threshold varies by model even on same carrier+plan)
- ⚠️ **Model + Customer** (likely needed - seen in fw_nat customer configs)
- ⚠️ **Carrier + Customer** (possibly needed - carrier-specific customer agreements)
- ⚠️ **Service Plan + Customer** (likely needed - custom pricing/limits per customer)

---

## Confirmed Two-Factor Combinations

### 1. Model + Carrier (Original Design)

**Parameters Affected:**
- `mqtt_enable` - Device Manager availability
- `mqtt_*` (all related) - Device Manager configuration
- `advanced` - Advanced features toggle
- `dual_sim_main` - Which SIM is primary
- `wan1_ppp_apn` - Carrier APN
- `alarm_input_options` - Model hardware capabilities
- `alarm_output_options` - Model hardware capabilities

**Real-World Examples:**

#### mqtt_enable (Device Manager)
```
i-22 + VZW → mqtt_enable=1 (enabled)
i-22 + ATT → mqtt_enable=0 (disabled)
4500 + VZW → mqtt_enable=0 (not supported)
4500 + ATT → mqtt_enable=0 (not supported)
```

**Rule Logic:**
```
IF model = "i-22" AND carrier = "VZW" THEN mqtt_enable = 1
IF model = "i-22" AND carrier = "TMO" THEN mqtt_enable = 1
IF model = "i-22" AND carrier = "ATT" THEN mqtt_enable = 0
IF model = "4500" AND carrier = * THEN mqtt_enable = 0
IF model = "IR611" AND carrier = * THEN mqtt_enable = 0
```

#### advanced (Advanced Features)
```
i-22 + VZW → advanced=1 (enabled)
i-22 + ATT → advanced=0 (disabled)
4500 + VZW → advanced=0 (disabled)
```

**Rule Logic:**
```
IF model = "i-22" AND carrier = "VZW" THEN advanced = 1
ELSE advanced = 0
```

#### dual_sim_main (Primary SIM Selection)
```
i-22 + ATT → dual_sim_main=1 (SIM2 primary - ATT SIM)
i-22 + VZW → dual_sim_main=0 (SIM1 primary - VZW SIM)
```

**Business Context:** This is based on carrier agreements - which carrier's SIM should be primary for cost/performance optimization.

---

### 2. Carrier + Service Plan (NEW FINDING)

**Parameters Affected:**
- `traffic_day_threshold` - Daily data limit (varies by Carrier + Plan combination)
- `traffic_month_threshold` - Monthly data limit

**Real-World Examples:**

#### traffic_day_threshold (Daily Data Limit)

**ATT Carrier:**
```
ATT + Tier1 → 5GB daily
ATT + ATM   → 350MB daily (I-22) / 5GB daily (4500)
```

**VZW Carrier:**
```
VZW + Tier1 → 3584MB daily (3.5GB)
VZW + ATM   → 3584MB daily (3.5GB)
```

**Key Observation:** VZW has SAME limit regardless of plan, ATT has DIFFERENT limits by plan.

**Rule Logic (Simplified):**
```
IF carrier = "ATT" AND service_plan = "Tier1" THEN traffic_day_threshold = 5GB
IF carrier = "ATT" AND service_plan = "ATM" THEN traffic_day_threshold = 350MB (for I-22) or 5GB (for 4500)
IF carrier = "VZW" AND service_plan = "Tier1" THEN traffic_day_threshold = 3584MB
IF carrier = "VZW" AND service_plan = "ATM" THEN traffic_day_threshold = 3584MB
```

**Business Context:**
- ATT ATM plan has very restrictive data for I-22 devices (350MB) - possibly business/pricing logic
- VZW has consistent limits across plans
- This is a **carrier pricing policy + service plan tier** interaction

---

### 3. Model + Service Plan (NEW FINDING)

**Parameters Affected:**
- `traffic_day_threshold` - Model affects data allowance even on same carrier+plan
- `fw_acl` - Firewall rules format may vary by model

**Real-World Examples:**

#### traffic_day_threshold by Model

**Same Carrier (ATT) + Same Plan (ATM):**
```
ATT + ATM + i-22  → 350MB daily
ATT + ATM + 4500  → 5GB daily
```

**Same Carrier (VZW) + Same Plan (Tier1):**
```
VZW + Tier1 + i-22  → 3584MB daily
VZW + Tier1 + 4500  → 5GB daily
```

**Rule Logic:**
```
IF model = "i-22" AND service_plan = "ATM" THEN use_restrictive_limits = true
IF model = "4500" AND service_plan = "ATM" THEN use_standard_limits = true
```

**Business Context:**
- I-22 devices on ATM plan get more restrictive data limits
- 4500 devices get more generous limits
- Possibly related to:
  - Model hardware capabilities
  - Business tier (I-22 = entry level, 4500 = premium)
  - Pricing structure

---

## Traffic Threshold Matrix (Full Picture)

| Model | Carrier | Service Plan | Daily Threshold | Notes |
|-------|---------|--------------|-----------------|-------|
| i-22  | ATT     | Tier1        | 5 GB            | Standard |
| i-22  | ATT     | ATM          | **350 MB**      | **VERY restrictive** |
| i-22  | VZW     | Tier1        | 3584 MB (3.5GB) | VZW standard |
| i-22  | VZW     | ATM          | 3584 MB (3.5GB) | Same as Tier1 |
| 4500  | ATT     | Tier1        | 5 GB            | Standard |
| 4500  | ATT     | ATM          | 5 GB            | Same as Tier1 |
| 4500  | VZW     | Tier1        | 5 GB            | Higher than i-22 |
| 4500  | VZW     | ATM          | 5 GB            | Same as Tier1 |

**Observations:**
1. **ATT + i-22 + ATM** is uniquely restrictive (350MB)
2. **VZW has carrier-level consistency** - same limit regardless of plan
3. **4500 gets higher limits than i-22** on VZW
4. **Model matters on VZW** - 4500 gets 5GB, i-22 gets 3.5GB

**This requires THREE factors: Model + Carrier + Service Plan**

---

## Likely Two-Factor Combinations (Not Yet Confirmed)

### 4. Model + Customer (Likely)

**Expected Parameters:**
- `fw_nat` - Customer-specific NAT rules may vary by model capabilities
- `digitalio_config` - I/O configuration (only I-22/I-52 have I/O, customer-specific setup)
- `lan0_ip` - Customer network scheme may vary by model

**Business Context:**
- Customer "Miele" may need I/O configured for I-22 devices but not for IR611 devices
- Customer network architectures may differ based on deployed model

**Rule Logic Example:**
```
IF model = "i-22" AND customer = "Miele" THEN digitalio_config = "1,0,0;1,0,0;"
IF model = "IR611" AND customer = "Miele" THEN digitalio_config = NULL (not supported)
```

---

### 5. Carrier + Customer (Possibly)

**Expected Parameters:**
- `wan1_ppp_redial_interval` - Custom carrier agreements per customer
- `traffic_day_threshold` - VIP customer gets higher limits on specific carrier
- `dual_sim_*` - Customer-specific carrier failover policies

**Business Context:**
- Large customer may negotiate special data rates with VZW
- Customer may have SLA with specific carrier requiring custom settings

**Rule Logic Example:**
```
IF carrier = "VZW" AND customer = "BigRetailChain" THEN traffic_day_threshold = 10GB (VIP)
IF carrier = "VZW" AND customer = "StandardCustomer" THEN traffic_day_threshold = 3584MB
```

---

### 6. Service Plan + Customer (Likely)

**Expected Parameters:**
- `traffic_day_threshold` - Customer-specific data allowance within plan
- `traffic_month_threshold` - Custom billing limits
- `fw_acl` - Customer exceptions to plan baseline

**Business Context:**
- Customer "CORD" on ATM plan needs custom firewall rules (already confirmed as Four-Way Rule)
- BUT, there may be simpler cases where Customer + Plan matters WITHOUT Model+Carrier

**Rule Logic Example:**
```
IF service_plan = "ATM" AND customer = "VIP_Customer" THEN traffic_day_threshold = 1000MB (upgraded)
IF service_plan = "ATM" AND customer = "Standard_Customer" THEN traffic_day_threshold = 350MB
```

**Note:** This might be better handled as Four-Way Rule (Model + Carrier + Plan + Customer) but could also be simplified as two-factor if Model+Carrier don't matter.

---

## Recommendation: Support 6 Two-Factor Combinations

Based on analysis, we should support **ALL** two-factor combinations of the 4 entities (Model, Carrier, Service Plan, Customer):

### Required Two-Factor Rule Types:

| # | Combination | Confirmed? | Example Parameters | Use Case |
|---|-------------|------------|-------------------|----------|
| 1 | **Model + Carrier** | ✅ Yes | mqtt_enable, advanced, dual_sim_main | Carrier features per model |
| 2 | **Model + Service Plan** | ✅ Yes | traffic_day_threshold (model-based pricing) | Model tier affects plan limits |
| 3 | **Model + Customer** | ⚠️ Likely | digitalio_config, fw_nat | Customer config varies by model |
| 4 | **Carrier + Service Plan** | ✅ Yes | traffic_day_threshold (carrier policy) | Carrier data policies by plan |
| 5 | **Carrier + Customer** | ⚠️ Possibly | traffic_day_threshold (VIP limits) | Customer SLAs with carriers |
| 6 | **Service Plan + Customer** | ⚠️ Likely | traffic_*, fw_acl | Customer exceptions to plan |

---

## Architecture Implications

### Current PRD Design:
- Two-Way Rules: **Model + Carrier only**
- Three-Way Rules: Model + Carrier + Service Plan
- Four-Way Rules: Model + Carrier + Service Plan + Customer

### Proposed Enhanced Design:

#### Option A: Expand Two-Way Rules to All Combinations
```
Two-Way Rules (6 types):
- Model + Carrier
- Model + Service Plan
- Model + Customer
- Carrier + Service Plan
- Carrier + Customer
- Service Plan + Customer

Three-Way Rules (4 types):
- Model + Carrier + Service Plan
- Model + Carrier + Customer
- Model + Service Plan + Customer
- Carrier + Service Plan + Customer

Four-Way Rules (1 type):
- Model + Carrier + Service Plan + Customer
```

**Priority Levels:** 11 + 6 + 4 = **21 levels total**

---

#### Option B: Generalized N-Way Rules (Recommended)

Instead of fixed Two-Way/Three-Way/Four-Way, use **Conditional Rules** with flexible factor combinations:

**Rule Definition:**
```json
{
  "parameter": "traffic_day_threshold",
  "conditions": {
    "carrier": "ATT",
    "service_plan": "ATM"
  },
  "value": "350MB",
  "priority": 40
}
```

**Priority Hierarchy (Enhanced 15-Level):**
```
1.  Device Override
2.  Four-Way Rule (Model + Carrier + Plan + Customer)
3.  Company Layer Override
4.  Three-Way Rule: Model + Carrier + Plan
5.  Three-Way Rule: Model + Carrier + Customer
6.  Three-Way Rule: Model + Plan + Customer
7.  Three-Way Rule: Carrier + Plan + Customer
8.  Service Plan Layer
9.  Two-Way Rule: Carrier + Service Plan
10. Two-Way Rule: Model + Service Plan
11. Two-Way Rule: Model + Customer
12. Two-Way Rule: Carrier + Customer
13. Two-Way Rule: Service Plan + Customer
14. Two-Way Rule: Model + Carrier
15. Carrier Layer
16. Model Layer
17. Global Layer
18. Schema Default
19. Required Validation
```

---

#### Option C: Simplified - Keep 11 Levels, Group Two-Way Rules

Keep the existing 11-level hierarchy but **expand the definition of Two-Way Rules** to include all 6 combinations at the same priority level:

**Priority Hierarchy (Original 11-Level):**
```
1.  Device Override
2.  Four-Way Rule (Model + Carrier + Plan + Customer)
3.  Company Layer Override
4.  Three-Way Rule (any 3 factors)
5.  Service Plan Layer
6.  Two-Way Rule (any 2 factors) ← EXPANDED
7.  Carrier Layer
8.  Model Layer
9.  Global Layer
10. Schema Default
11. Required Validation
```

**At Level 6 (Two-Way Rules), check in this order:**
```
6a. Carrier + Service Plan
6b. Model + Service Plan
6c. Model + Carrier
6d. Carrier + Customer
6e. Model + Customer
6f. Service Plan + Customer
```

**Advantage:**
- Keeps 11-level simplicity
- Supports all combinations
- Sub-prioritization within Level 6

**Disadvantage:**
- Less granular control between two-factor combinations

---

## Recommended Implementation

### **Option C: Enhanced 11-Level with Expanded Two-Way Rules**

**Why:**
1. ✅ Maintains 11-level simplicity for client understanding
2. ✅ Supports all required two-factor combinations
3. ✅ Sub-prioritization handles edge cases
4. ✅ Backward compatible with existing PRD
5. ✅ Easy to explain: "Two-Way means any 2 factors"

**Enhanced Level 6 Definition:**

```markdown
### Level 6: Two-Way Conditional Rules (Multi-Factor)

**Purpose:** Rules based on any TWO of the four factors (Model, Carrier, Service Plan, Customer)

**Sub-Priority Order:**
1. **Carrier + Service Plan** - Carrier data policies by plan tier
2. **Model + Service Plan** - Model-based pricing/features by plan
3. **Model + Carrier** - Carrier features per model (original design)
4. **Carrier + Customer** - Customer-specific carrier agreements
5. **Model + Customer** - Customer config varies by deployed model
6. **Service Plan + Customer** - Customer exceptions to plan baseline (alternative to Four-Way)

**Resolution:** Check rules in sub-priority order. First matching rule wins.
```

---

## Database Schema Updates

### Current Schema:
```sql
CREATE TABLE conditional_rules (
  id INT PRIMARY KEY,
  parameter_id INT,
  model_id INT NULL,           -- Required for Two-Way (M+C)
  carrier_id INT NULL,         -- Required for Two-Way (M+C)
  service_plan_id INT NULL,    -- Three-Way adds this
  company_id INT NULL,         -- Four-Way adds this
  value TEXT,
  priority INT,                -- 2, 4, or 6 (Four/Three/Two-Way)
  created DATETIME
);
```

### Updated Schema (No Change Needed!):
```sql
CREATE TABLE conditional_rules (
  id INT PRIMARY KEY,
  parameter_id INT,
  model_id INT NULL,           -- Optional - any 2+ of these
  carrier_id INT NULL,         -- Optional - any 2+ of these
  service_plan_id INT NULL,    -- Optional - any 2+ of these
  company_id INT NULL,         -- Optional - any 2+ of these
  value TEXT,
  priority INT,                -- 2, 4, or 6
  sub_priority INT,            -- NEW: 1-6 for Two-Way sub-ordering
  created DATETIME,

  -- Validation constraint
  CHECK (
    (model_id IS NOT NULL AND carrier_id IS NOT NULL AND service_plan_id IS NOT NULL AND company_id IS NOT NULL) OR  -- Four-Way
    (model_id IS NOT NULL AND carrier_id IS NOT NULL AND service_plan_id IS NOT NULL) OR  -- Three-Way
    (model_id IS NOT NULL AND carrier_id IS NOT NULL) OR  -- Two-Way M+C
    (model_id IS NOT NULL AND service_plan_id IS NOT NULL) OR  -- Two-Way M+SP
    (model_id IS NOT NULL AND company_id IS NOT NULL) OR  -- Two-Way M+Cu
    (carrier_id IS NOT NULL AND service_plan_id IS NOT NULL) OR  -- Two-Way C+SP
    (carrier_id IS NOT NULL AND company_id IS NOT NULL) OR  -- Two-Way C+Cu
    (service_plan_id IS NOT NULL AND company_id IS NOT NULL)  -- Two-Way SP+Cu
  )
);
```

**Sub-Priority Mapping:**
```
1 = Carrier + Service Plan
2 = Model + Service Plan
3 = Model + Carrier
4 = Carrier + Customer
5 = Model + Customer
6 = Service Plan + Customer
```

---

## Resolution Algorithm Update

### Enhanced Two-Way Rule Matching:

```php
function resolveTwoWayRule($parameter, $device) {
    // Check Carrier + Service Plan (sub-priority 1)
    $rule = findRule($parameter, [
        'carrier' => $device->carrier_id,
        'service_plan' => $device->service_plan_id
    ]);
    if ($rule) return $rule->value;

    // Check Model + Service Plan (sub-priority 2)
    $rule = findRule($parameter, [
        'model' => $device->model_id,
        'service_plan' => $device->service_plan_id
    ]);
    if ($rule) return $rule->value;

    // Check Model + Carrier (sub-priority 3 - original design)
    $rule = findRule($parameter, [
        'model' => $device->model_id,
        'carrier' => $device->carrier_id
    ]);
    if ($rule) return $rule->value;

    // Check Carrier + Customer (sub-priority 4)
    $rule = findRule($parameter, [
        'carrier' => $device->carrier_id,
        'customer' => $device->company_id
    ]);
    if ($rule) return $rule->value;

    // Check Model + Customer (sub-priority 5)
    $rule = findRule($parameter, [
        'model' => $device->model_id,
        'customer' => $device->company_id
    ]);
    if ($rule) return $rule->value;

    // Check Service Plan + Customer (sub-priority 6)
    $rule = findRule($parameter, [
        'service_plan' => $device->service_plan_id,
        'customer' => $device->company_id
    ]);
    if ($rule) return $rule->value;

    // No Two-Way Rule found
    return null;
}
```

---

## UI Updates Required

### Conditional Rules Management UI (FR-3a)

**Current Design:**
- Three tabs: Two-Way, Three-Way, Four-Way
- Two-Way Rules: Model + Carrier only

**Updated Design:**
- Keep three tabs: Two-Way, Three-Way, Four-Way
- **Two-Way Rules Tab:** Add factor combination selector

**Two-Way Rule Creation Form:**
```
┌─────────────────────────────────────────┐
│ Create Two-Way Conditional Rule        │
├─────────────────────────────────────────┤
│ Parameter: [traffic_day_threshold ▼]   │
│                                         │
│ Factor Combination:                     │
│ ○ Carrier + Service Plan               │
│ ○ Model + Service Plan                 │
│ ○ Model + Carrier                      │
│ ○ Carrier + Customer                   │
│ ○ Model + Customer                     │
│ ○ Service Plan + Customer              │
│                                         │
│ Factor 1: [VZW ▼]                      │
│ Factor 2: [ATM Plan ▼]                 │
│                                         │
│ Value: [3584MB]                        │
│                                         │
│ [Create Rule] [Cancel]                 │
└─────────────────────────────────────────┘
```

**Two-Way Rules List View:**
```
┌────────────────────────────────────────────────────────┐
│ Two-Way Conditional Rules                              │
├────────┬───────────────┬─────────────┬────────┬────────┤
│ Param  │ Combination   │ Conditions  │ Value  │ Action │
├────────┼───────────────┼─────────────┼────────┼────────┤
│ mqtt_  │ Model +       │ i-22 + VZW  │ 1      │ Edit   │
│ enable │ Carrier       │             │        │        │
├────────┼───────────────┼─────────────┼────────┼────────┤
│ traffi │ Carrier +     │ VZW + ATM   │ 3584MB │ Edit   │
│ c_day  │ Service Plan  │             │        │        │
├────────┼───────────────┼─────────────┼────────┼────────┤
│ traffi │ Model +       │ i-22 + ATM  │ 350MB  │ Edit   │
│ c_day  │ Service Plan  │             │        │        │
└────────┴───────────────┴─────────────┴────────┴────────┘
```

---

## Migration Considerations

### Existing Rules:
- Current system may have implicit two-factor patterns (Model + Carrier)
- Need to identify and migrate to explicit conditional rules

### Migration Script:
```sql
-- Identify parameters that vary by Carrier + Service Plan
SELECT
    cv1.config_schema_id,
    cv1.carrier_id,
    cv1.service_plan_id,
    cv1.value,
    COUNT(DISTINCT cv2.value) as value_variations
FROM config_values cv1
JOIN config_values cv2 ON cv1.config_schema_id = cv2.config_schema_id
WHERE cv1.layer_type = 'carrier'
  AND cv2.layer_type = 'service_plan'
GROUP BY cv1.config_schema_id, cv1.carrier_id, cv1.service_plan_id
HAVING value_variations > 1;

-- Convert to conditional rules
INSERT INTO conditional_rules (parameter_id, carrier_id, service_plan_id, value, priority, sub_priority)
SELECT
    config_schema_id,
    carrier_id,
    service_plan_id,
    value,
    6,  -- Two-Way priority level
    1   -- Carrier + Service Plan sub-priority
FROM config_values
WHERE layer_type = 'carrier_service_plan_combo';  -- Hypothetical
```

---

## Testing Requirements

### Test Cases for Two-Way Rules:

1. **Carrier + Service Plan:**
   ```
   GIVEN device with VZW carrier and ATM plan
   WHEN resolving traffic_day_threshold
   THEN should return 3584MB (not 350MB for ATT+ATM)
   ```

2. **Model + Service Plan:**
   ```
   GIVEN device with i-22 model and ATM plan on ATT carrier
   WHEN resolving traffic_day_threshold
   THEN should return 350MB (not 5GB for 4500+ATM+ATT)
   ```

3. **Priority Order:**
   ```
   GIVEN rules exist for:
     - Carrier + Service Plan (VZW + ATM → 3584MB)
     - Model + Carrier (i-22 + VZW → 4000MB)
   WHEN device is i-22 on VZW with ATM plan
   THEN Carrier + Service Plan should win (3584MB)
   BECAUSE sub-priority 1 beats sub-priority 3
   ```

4. **Three-Way Overrides Two-Way:**
   ```
   GIVEN rules exist for:
     - Three-Way (i-22 + VZW + ATM → 2000MB)
     - Two-Way (VZW + ATM → 3584MB)
   WHEN device is i-22 on VZW with ATM plan
   THEN Three-Way rule should win (2000MB)
   BECAUSE Level 4 beats Level 6
   ```

---

## Documentation Updates Required

### PRD Updates:

1. **FR-3a: Conditional Rules Management**
   - Update "Two-Way Rules" section to include all 6 combinations
   - Add sub-priority ordering explanation
   - Update rule creation UI wireframes

2. **11-Level Priority Hierarchy**
   - Expand Level 6 definition
   - Add sub-priority order
   - Update resolution algorithm pseudocode

3. **Examples**
   - Add traffic_day_threshold example showing Carrier + Service Plan
   - Add Model + Service Plan example

### Prototype Updates:

1. **conditional-rules-demo.html**
   - Update Two-Way Rules tab to show all 6 combinations
   - Add combination selector in rule creation
   - Update example rules to include new combinations

2. **README.md**
   - Update statistics: "Two-Way Rules: 6 types, ~40-60 rules total"
   - Add explanation of two-factor combinations

---

## Summary & Recommendations

### Key Findings:
1. ✅ **Model + Carrier** two-way rules are confirmed (mqtt_enable, advanced)
2. ✅ **Carrier + Service Plan** two-way rules are confirmed (traffic thresholds)
3. ✅ **Model + Service Plan** two-way rules are confirmed (traffic thresholds by model tier)
4. ⚠️ **Other combinations likely needed** but not yet confirmed in data

### Recommended Architecture:

**Option C: Enhanced 11-Level with Expanded Two-Way Rules**

```
Level 6: Two-Way Conditional Rules (any 2 factors)
  Sub-priority:
    6.1 Carrier + Service Plan
    6.2 Model + Service Plan
    6.3 Model + Carrier
    6.4 Carrier + Customer
    6.5 Model + Customer
    6.6 Service Plan + Customer
```

### Implementation Steps:

1. ✅ Update PRD FR-3a section (expand Two-Way Rules definition)
2. ✅ Add `sub_priority` column to `conditional_rules` table
3. ✅ Update resolution algorithm to check all 6 two-factor combinations
4. ✅ Update conditional-rules-demo.html prototype
5. ✅ Update UI wireframes for rule creation
6. ⬜ Implement migration script to identify existing patterns
7. ⬜ Create test cases for all combinations
8. ⬜ Update user documentation

### Estimated Rule Counts:

| Rule Type | Combinations | Est. Count |
|-----------|--------------|------------|
| Two-Way   | 6 types      | 40-60 rules |
| Three-Way | 4 types      | 30-40 rules |
| Four-Way  | 1 type       | 10-20 rules |
| **Total** | **11 types** | **80-120 rules** |

---

## Conclusion

**YES - We need to support multiple two-factor combinations beyond just Model + Carrier.**

The data clearly shows that parameters like `traffic_day_threshold` require:
- **Carrier + Service Plan** rules (VZW policies vs ATT policies)
- **Model + Service Plan** rules (I-22 tier vs 4500 tier pricing)
- **Model + Carrier** rules (mqtt_enable, advanced features)

The Enhanced 11-Level architecture with expanded Two-Way Rules (Option C) provides the flexibility needed while maintaining the simplicity of the original design.

