# PRD Version 1.5 Update Summary

**Date:** January 12, 2026
**Document:** Config_Management_System_PRD.md
**Version:** 1.0 → 1.5
**Change Type:** MAJOR UPDATE - Expanded Two-Way Rules Framework

---

## Executive Summary

Based on analysis of actual production configuration files, we discovered that **Two-Way Rules need to support multiple two-factor combinations**, not just Model + Carrier. The PRD has been updated to support **6 two-factor combinations** with a **sub-priority ordering system** at Level 6 of the 11-Level Priority Hierarchy.

This update ensures the system can handle real-world scenarios where:
- VZW ATM plan gets 3584MB daily data, but AT&T ATM plan only gets 350MB (Carrier + Service Plan)
- i-22 devices on ATM plan get 350MB daily, but 4500 devices get 5GB (Model + Service Plan)
- VZW i-22 devices enable Device Manager, but AT&T i-22 devices don't (Model + Carrier)

---

## What Changed

### 1. **Executive Summary (Lines 40-61)**

**Before:**
```
- Two-Way Rules: Model + Carrier combinations
- 11-Level Priority Hierarchy
```

**After:**
```
- Two-Way Rules: Any 2-factor combinations (6 types):
  - Model + Carrier
  - Carrier + Service Plan
  - Model + Service Plan
  - Carrier + Customer
  - Model + Customer
  - Service Plan + Customer
- 11-Level Priority Hierarchy with Sub-Priorities
  (Full expanded list showing sub-priorities 6.1-6.6)
```

---

### 2. **Scope Section - Conditional Rules Framework (Lines 298-382)**

**Major Additions:**

#### Expanded Two-Way Rules Definition:
- Added **6 supported combinations** with examples for each
- Added **sub-priority ordering** (Carrier+Plan → Model+Plan → Model+Carrier → Carrier+Customer → Model+Customer → Plan+Customer)
- Added comprehensive **real-world examples** from production configs

**Key Examples Added:**
```markdown
Carrier + Service Plan:
- VZW + ATM → traffic_day_threshold=3584MB
- ATT + ATM → traffic_day_threshold=350MB
(Different carrier policies for same plan)

Model + Service Plan:
- i-22 + ATM → traffic_day_threshold=350MB
- 4500 + ATM → traffic_day_threshold=5GB
(Model pricing tier affects limits)

Model + Carrier:
- i-22 + VZW → mqtt_enable=1
- i-22 + ATT → mqtt_enable=0
(Carrier features per model)
```

#### New Rule Type Selection Guide:
- Added comprehensive guide for choosing rule type
- Added **decision tree** flowchart
- Clarified when to use Two-Way vs Three-Way vs Four-Way rules

---

### 3. **FR-3a.1: Conditional Rule Creation (Lines 1310-1357)**

**Major Update to Two-Way Rule Creation Workflow:**

#### Added 5-Step Process:
1. **Step 1: Select Factor Combination** (radio buttons for 6 options)
2. **Step 2: Select Factor Values** (dynamic dropdowns based on combination)
3. **Step 3: Enter Value** (with validation)
4. **Step 4: Set Priority** (automatic sub-priority assignment 1-6)
5. **Step 5: Add Description** (business justification)

**Before:**
```
For Two-Way Rules:
- Select Model from dropdown
- Select Carrier from dropdown
- Enter value
```

**After:**
```
For Two-Way Rules:
- Step 1: Choose which 2 factors (6 options)
- Step 2: Select values based on chosen combination
- Step 3: Enter value with preview
- Step 4: Automatic sub-priority (1=Carrier+Plan ... 6=Plan+Customer)
- Step 5: Add business justification
```

**Each combination gets:**
- Specific dropdown fields
- Example values
- Use case explanation

---

### 4. **FR-3.1: 11-Level Priority Hierarchy (Lines 1196-1395)**

**MAJOR EXPANSION of Level 6 with Sub-Priorities:**

**Before (Simple):**
```
Level 6: Two-Way Rule (Model+Carrier)
- Query: WHERE model=device.model AND carrier=device.carrier
- If found → use value
- Source: "Two-Way Rule - [Model]+[Carrier]"
```

**After (Comprehensive with 6 Sub-Priorities):**
```
Level 6: Two-Way Rule (Any 2-Factor Combination)
- Sub-Priority Order (check sequentially, first match wins):

  6.1: Carrier + Service Plan Rule
  - Query: WHERE carrier=X AND service_plan=Y (model_id IS NULL, company_id IS NULL)
  - Real-World Example:
    Device: i-22, VZW, ATM Plan
    Rule: VZW+ATM → traffic_day_threshold=3584MB
    Result: Uses 3584MB (VZW's ATM policy)

  6.2: Model + Service Plan Rule
  - Query: WHERE model=X AND service_plan=Y (carrier_id IS NULL, company_id IS NULL)
  - Real-World Example:
    Device: i-22, ATT, ATM Plan
    Rule: i-22+ATM → traffic_day_threshold=350MB
    Result: Uses 350MB (i-22 entry-level tier)

  6.3: Model + Carrier Rule (original design)
  - Query: WHERE model=X AND carrier=Y (service_plan_id IS NULL, company_id IS NULL)
  - Real-World Example:
    Device: i-22, VZW, ATM Plan
    Rule: i-22+VZW → mqtt_enable=1
    Result: mqtt_enable=1 (Device Manager enabled)

  6.4: Carrier + Customer Rule
  - Use case: VIP customer carrier agreements

  6.5: Model + Customer Rule
  - Use case: Customer-specific model configs (I/O settings)

  6.6: Service Plan + Customer Rule
  - Use case: Customer plan exceptions (simpler than Four-Way)
```

**Added:**
- Full query syntax for each sub-priority
- NULL constraint requirements
- 3 comprehensive real-world examples with full walkthroughs
- Comparison scenarios showing how different devices resolve differently

---

### 5. **Complete 11-Level Priority Order (Lines 1341-1395)**

**Updated Summary:**
```
6. Two-Way Rule (Any 2 Factors - checked in sub-priority order):
   6.1 Carrier + Service Plan
   6.2 Model + Service Plan
   6.3 Model + Carrier
   6.4 Carrier + Customer
   6.5 Model + Customer
   6.6 Service Plan + Customer
```

**Added Two Resolution Examples:**

#### Example 1: Simple Resolution
```
Device: i-22, VZW, ATM Plan, CORD Company
Parameter: traffic_day_threshold

Walk through levels:
Level 1-5: No values found
Level 6:
  6.1: Check VZW+ATM → FOUND 3584MB → STOP

Result: 3584MB from Carrier+Plan rule
```

#### Example 2: Multiple Rules Conflict
```
Device: i-22, VZW, ATM Plan, CORD Company
Parameter: traffic_day_threshold

Available Rules:
- Carrier+Plan: VZW+ATM → 3584MB (sub-priority 6.1)
- Model+Plan: i-22+ATM → 350MB (sub-priority 6.2)
- Model+Carrier: i-22+VZW → 4000MB (sub-priority 6.3)

Resolution Process at Level 6:
  6.1: Check VZW+ATM → FOUND 3584MB → USE THIS
  (6.2 and 6.3 not checked - first match wins)

Result: 3584MB
Reason: Carrier policy checked first (highest sub-priority)
```

---

### 6. **Rule Matching Logic (Lines 1400-1413)**

**Updated to include all 6 combinations:**

**Before:**
```
Two-way rule: Must match device.model_id AND device.carrier_id exactly
```

**After:**
```
Two-way rule: Must match EXACTLY TWO device attributes (any combination):
- Carrier + Service Plan: WHERE carrier=X AND service_plan=Y (model_id IS NULL, company_id IS NULL)
- Model + Service Plan: WHERE model=X AND service_plan=Y (carrier_id IS NULL, company_id IS NULL)
- Model + Carrier: WHERE model=X AND carrier=Y (service_plan_id IS NULL, company_id IS NULL)
- Carrier + Customer: WHERE carrier=X AND company=Y (model_id IS NULL, service_plan_id IS NULL)
- Model + Customer: WHERE model=X AND company=Y (carrier_id IS NULL, service_plan_id IS NULL)
- Service Plan + Customer: WHERE service_plan=X AND company=Y (model_id IS NULL, carrier_id IS NULL)

Important: Partial matches do not count (all specified factors must match, unspecified must be NULL)
Within Level 6: Check sub-priorities sequentially (6.1 → 6.6); first match wins
```

---

### 7. **Performance Requirements (Lines 1414-1427)**

**Updated Database Indexes:**

**Before:**
```
- (model_id, carrier_id) for Two-Way Rules
```

**After:**
```
Two-Way Rules (6 indexes for different combinations):
- (carrier_id, service_plan_id) WHERE model_id IS NULL AND company_id IS NULL
- (model_id, service_plan_id) WHERE carrier_id IS NULL AND company_id IS NULL
- (model_id, carrier_id) WHERE service_plan_id IS NULL AND company_id IS NULL
- (carrier_id, company_id) WHERE model_id IS NULL AND service_plan_id IS NULL
- (model_id, company_id) WHERE carrier_id IS NULL AND service_plan_id IS NULL
- (service_plan_id, company_id) WHERE model_id IS NULL AND carrier_id IS NULL
```

**Rationale:** Each two-factor combination needs its own partial index with NULL constraints for optimal query performance.

---

### 8. **Document Metadata**

**Updated:**
- Version: 1.0 → **1.5**
- Date: January 2, 2026 → **January 12, 2026**
- Added: "Last Updated: Expanded Two-Way Rules to support all 6 two-factor combinations with sub-priority ordering"

**Document History Entry Added:**
```
Version 1.5 (2026-01-12):
MAJOR UPDATE - Expanded Two-Way Rules to support ALL 6 two-factor combinations.
Analysis of actual device configs revealed need for: Carrier+ServicePlan,
Model+ServicePlan, Carrier+Customer, Model+Customer, ServicePlan+Customer.
Added sub-priority ordering (6.1-6.6). Updated Executive Summary, Scope,
FR-3a.1, FR-3.1 with detailed examples. Based on production config analysis.
```

---

## Why These Changes Were Made

### Evidence from Production Configs

**1. traffic_day_threshold Analysis:**

| Model | Carrier | Plan   | Daily Limit | Rule Type Needed       |
|-------|---------|--------|-------------|------------------------|
| i-22  | ATT     | ATM    | **350MB**   | Model+Plan             |
| 4500  | ATT     | ATM    | **5GB**     | Model+Plan             |
| i-22  | VZW     | ATM    | **3584MB**  | Carrier+Plan           |
| i-22  | ATT     | Tier1  | **5GB**     | Carrier+Plan           |

**Key Findings:**
- **Carrier+Plan matters**: VZW ATM (3584MB) ≠ ATT ATM (350MB)
- **Model+Plan matters**: i-22 ATM (350MB) ≠ 4500 ATM (5GB)
- **Both patterns exist**: Can't use just Model+Carrier

**2. mqtt_enable Analysis:**

| Model | Carrier | Value | Rule Type Needed |
|-------|---------|-------|------------------|
| i-22  | VZW     | 1     | Model+Carrier    |
| i-22  | ATT     | 0     | Model+Carrier    |
| 4500  | VZW     | 0     | Model+Carrier    |

**Key Finding:**
- Model+Carrier is essential for this parameter
- Original design was correct but insufficient alone

**3. Future Use Cases Identified:**

| Combination       | Example Use Case                                    |
|-------------------|-----------------------------------------------------|
| Carrier+Customer  | VIP_Corp negotiated 10GB daily limit with VZW       |
| Model+Customer    | Miele's i-22 devices need custom I/O configuration  |
| Plan+Customer     | Premium_Customer upgraded ATM limit (1000MB vs 350MB) |

---

## Technical Implementation Impact

### Database Schema (No Changes Needed!)

**Good News:** The existing `conditional_rules` table already supports all combinations:

```sql
CREATE TABLE conditional_rules (
  model_id INT NULL,
  carrier_id INT NULL,
  service_plan_id INT NULL,
  company_id INT NULL,
  value TEXT,
  priority INT,
  sub_priority INT,  -- NEW: for sub-ordering within Level 6
  ...
);
```

**How it works:**
- Carrier+Plan rule: carrier_id=X, service_plan_id=Y, model_id=NULL, company_id=NULL
- Model+Plan rule: model_id=X, service_plan_id=Y, carrier_id=NULL, company_id=NULL
- Model+Carrier rule: model_id=X, carrier_id=Y, service_plan_id=NULL, company_id=NULL
- etc.

**Validation:** CHECK constraint ensures exactly 2 non-NULL foreign keys.

### Resolution Algorithm Changes

**Before (Simple):**
```php
// Level 6: Check Model+Carrier only
$rule = findRule($param, [
    'model' => $device->model_id,
    'carrier' => $device->carrier_id
]);
```

**After (Sub-Priority):**
```php
// Level 6: Check all 6 combinations in sub-priority order
function resolveTwoWayRule($param, $device) {
    // 6.1: Carrier + Service Plan
    $rule = findRule($param, [
        'carrier' => $device->carrier_id,
        'service_plan' => $device->service_plan_id
    ]);
    if ($rule) return $rule->value;

    // 6.2: Model + Service Plan
    $rule = findRule($param, [
        'model' => $device->model_id,
        'service_plan' => $device->service_plan_id
    ]);
    if ($rule) return $rule->value;

    // 6.3: Model + Carrier (original)
    $rule = findRule($param, [
        'model' => $device->model_id,
        'carrier' => $device->carrier_id
    ]);
    if ($rule) return $rule->value;

    // 6.4-6.6: Continue checking...

    return null; // No two-way rule found
}
```

### UI Changes Required

**Rule Creation Form:**
- Add radio button group for selecting factor combination (6 options)
- Dynamic dropdowns based on selected combination
- Auto-assign sub_priority based on combination type
- Show preview of affected devices

**Rule List View:**
- Show combination type: "Two-Way (Carrier+Plan)"
- Color code by combination type
- Filter by combination type
- Sort by sub-priority

---

## Migration Impact

### Existing Rules

**If you have existing Model+Carrier rules:**
- They continue to work (sub-priority 6.3)
- No database changes needed
- Set sub_priority=3 for existing rules

### New Rules

**Admin can now create:**
- Carrier+Plan rules for data policies
- Model+Plan rules for pricing tiers
- Customer-specific combinations

---

## Testing Requirements

### Unit Tests Needed:

1. **Sub-Priority Ordering:**
   - Test that Carrier+Plan (6.1) beats Model+Carrier (6.3)
   - Test that Model+Plan (6.2) beats Model+Carrier (6.3)

2. **Query Logic:**
   - Test NULL constraints work correctly
   - Test partial matches don't accidentally match

3. **Resolution Examples:**
   - Test: i-22, VZW, ATM → should use VZW+ATM rule (3584MB)
   - Test: i-22, ATT, ATM → should use i-22+ATM rule (350MB)
   - Test: i-22, VZW, Tier1 → should use i-22+VZW rule (mqtt_enable=1)

### Integration Tests:

1. Create rules for all 6 combinations
2. Resolve config for devices matching multiple rules
3. Verify first-match-wins behavior

---

## Documentation Impact

### Files Updated:
- ✅ Config_Management_System_PRD.md (v1.5)
- ✅ Two_Way_Rule_Combinations_Analysis.md (supporting analysis)
- ⬜ Technical Design Document (needs update)
- ⬜ API Documentation (needs update)
- ⬜ User Manual (needs update)
- ⬜ Admin Training Materials (needs update)

### Prototypes Need Updates:
- ⬜ conditional-rules-demo.html - Expand Two-Way Rules tab to show all 6 combinations
- ⬜ README.md - Update statistics and examples

---

## Benefits of This Update

### 1. **Handles Real-World Complexity**
- ✅ Carrier data policies differ by plan (VZW vs ATT ATM limits)
- ✅ Model pricing tiers affect limits (i-22 vs 4500)
- ✅ Customer agreements with carriers (VIP data allowances)

### 2. **Maintains Simplicity**
- ✅ Still 11 levels (didn't add complexity)
- ✅ Sub-priorities are intuitive (carrier policy → model tier → model+carrier)
- ✅ Decision tree helps admins choose correct rule type

### 3. **Flexible for Future**
- ✅ Can add customer-specific rules as needed
- ✅ Framework supports any two-factor combination
- ✅ Sub-priority can be adjusted if business logic changes

### 4. **Database-Efficient**
- ✅ No schema changes required
- ✅ Partial indexes optimize queries
- ✅ Resolution performance <2 seconds per device

---

## Backward Compatibility

### Existing Model+Carrier Rules:
- ✅ Continue to work at sub-priority 6.3
- ✅ No migration required
- ✅ Same query pattern

### Existing Code:
- ⚠️ Resolution algorithm needs update to check sub-priorities
- ✅ Database queries remain similar
- ✅ API endpoints can stay the same (just add sub_priority field)

---

## Next Steps

### Immediate:
1. ✅ PRD updated (v1.5)
2. ⬜ Update Technical Design Document
3. ⬜ Update conditional-rules-demo.html prototype
4. ⬜ Update README.md with new examples

### Before Implementation:
1. ⬜ Review with engineering team
2. ⬜ Finalize sub-priority ordering
3. ⬜ Design UI mockups for new rule creation form
4. ⬜ Write database migration script (add sub_priority column, set defaults)

### During Implementation:
1. ⬜ Implement resolution algorithm updates
2. ⬜ Add 6 partial indexes
3. ⬜ Update admin UI
4. ⬜ Write comprehensive tests

---

## Summary

**Version 1.5 is a critical update** that ensures the Configuration Management System can handle real-world production scenarios identified through actual config file analysis. The expansion from 1 to 6 two-factor combinations provides the flexibility needed while maintaining the elegant 11-level priority hierarchy.

**Key Takeaway:** Two-Way Rules are now "any 2 factors" instead of "Model + Carrier only", with intelligent sub-priority ordering that checks Carrier+Plan first (most common carrier policy use case), then Model+Plan (pricing tiers), then Model+Carrier (original design).

**Analysis Source:** `/Users/aksana/Documents/Projects/WATM/Configurations/Two_Way_Rule_Combinations_Analysis.md`

