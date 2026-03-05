# Three-Way Rule Design Validation

**Date:** March 1, 2026
**Status:** VALIDATED - Current Design is Optimal
**Decision:** Support ONLY Model + Carrier + Service Plan (no other three-way combinations)

---

## Executive Summary

**Validated Decision: Three-Way Rules should support ONLY Model + Carrier + Service Plan combination.**

After comprehensive analysis of all possible three-way combinations, the current design is optimal. Other three-way combinations (Model+Carrier+Customer, Model+ServicePlan+Customer, Carrier+ServicePlan+Customer) are either already covered by existing architecture or not needed based on real-world usage patterns.

---

## Mathematical Analysis

### All Possible Three-Way Combinations

From 4 factors (Model, Carrier, Service Plan, Customer), there are C(4,3) = **4 possible three-way combinations**:

1. ✅ **Model + Carrier + Service Plan** (CURRENT DESIGN - VALIDATED)
2. ❌ **Model + Carrier + Customer** (NOT NEEDED)
3. ❌ **Model + Service Plan + Customer** (NOT NEEDED)
4. ❌ **Carrier + Service Plan + Customer** (NOT NEEDED)

---

## Why Model + Carrier + Service Plan is the ONLY Three-Way Type Needed

### Purpose of Three-Way Rules

**Design Principle:** Three-Way Rules define **service plan baselines** that apply to ALL customers with matching Model+Carrier+ServicePlan combination.

**Business Logic:**
- "All i-22 + ATT + ATM devices get 40 firewall whitelist rules"
- Service plan features that vary by both model AND carrier
- Applies to EVERY customer on that plan (not customer-specific)

**Customer Exceptions:** Use Four-Way Rules (adds Customer factor)

---

## Why Other Three-Way Combinations Are Not Needed

### Model + Carrier + Customer (NOT NEEDED)

**Hypothetical Use Case:** Customer config for Model+Carrier combo, regardless of service plan

**Why Not Needed:**
1. **Customers typically have ONE service plan** - configuration doesn't span multiple plans
2. **Service plan defines feature set** - customer customizations are within plan context
3. **Already covered by existing design:**
   - If customer has one plan: Use **Four-Way Rule** (Model+Carrier+Plan+Customer)
   - If truly applies to all plans (rare): Use **Two-Way Rule** (Model+Carrier) + **Company Layer Override**
   - Alternative: Use **Model+Customer Two-Way Rule** if carrier doesn't matter

**Real-World Evidence:**
- Customer firewall exceptions (CORD, DEPLOYER) are explicitly tied to "ATM Service Plan"
- No evidence of customer configs that span multiple service plans
- Customer exceptions are plan-specific by nature

---

### Model + Service Plan + Customer (NOT NEEDED)

**Hypothetical Use Case:** Customer config for Model+Plan combo, regardless of carrier

**Why Not Needed:**
1. **Most customer exceptions are carrier-independent** - already handled by simpler Two-Way Rules
2. **Real-world analysis shows simpler patterns:**

**Example: CORD Firewall Exception**
```
CORD's RMS server (13.67.184.126):
- Same server for i-22 and 4500 devices (model-independent)
- Same server on VZW and ATT carriers (carrier-independent)
- Only varies by: Service Plan (ATM) and Customer (CORD)

Pattern: Service Plan + Customer (TWO factors, not three)
```

3. **Already covered by existing design:**
   - If carrier doesn't matter: Use **ServicePlan+Customer Two-Way Rule** (simpler and more accurate)
   - If carrier matters: Use **Four-Way Rule**
   - If model doesn't matter either: Use **Company Layer Override**

---

### Carrier + Service Plan + Customer (NOT NEEDED)

**Hypothetical Use Case:** Customer config for Carrier+Plan combo, regardless of model

**Why Not Needed:**
1. **Model usually matters for pricing tiers** - different models have different service levels
2. **Real-world patterns:**
   - Data limits vary by model (i-22 entry-level vs 4500 premium)
   - VIP customer on i-22 gets upgraded i-22 limit (not same as 4500 limit)
   - Customer exceptions are typically model-specific

3. **Already covered by existing design:**
   - If model-independent (rare): Use **Carrier+Customer** or **Plan+Customer Two-Way Rule**
   - If model-independent infrastructure: Use **Company Layer Override**
   - If model matters: Use **Four-Way Rule**

---

## Real-World Evidence

### Finding 1: ATM Plan Baselines are Model+Carrier Independent

**Config File Analysis:**
```
ATT + i-22 + ATM:  fw_acl=[40 whitelist rules] (IDENTICAL)
VZW + i-22 + ATM:  fw_acl=[40 whitelist rules] (IDENTICAL)
ATT + 4500 + ATM:  fw_acl=[40 whitelist rules] (IDENTICAL)
VZW + 4500 + ATM:  fw_acl=[40 whitelist rules] (IDENTICAL)
```

**Implication:**
- The "Three-Way Rule" for fw_acl could actually be **Service Plan Layer** value
- Carrier doesn't change it, Model doesn't change it
- BUT: Keeping Three-Way Rule design provides flexibility for future requirements where model or carrier DO matter

### Finding 2: Customer Exceptions are Service-Plan-Specific

**Evidence from Production Configs:**
```
CustomerConfigs - Customer.csv:
"ATM Service Plan, Certain customers contain additional ACL entries for specific customer servers"

Examples:
- CORD RMS: 13.67.184.126
- DEPLOYER RMS: 208.92.212.170
```

**Implication:**
- Customer firewall exceptions are tied to the ATM service plan
- They don't apply to Tier1 or other plans
- This validates the Four-Way Rule design (includes Service Plan factor)

### Finding 3: Customer Exceptions are Model+Carrier Independent

**Evidence:**
```
CORD's RMS server IP: 13.67.184.126
- Same server for i-22 devices
- Same server for 4500 devices
- Same server on VZW carrier
- Same server on ATT carrier

Only varies by:
- Service Plan (ATM specific)
- Customer (CORD specific)
```

**Implication:**
- Customer exceptions could be handled by **Service Plan + Customer Two-Way Rule**
- Four-Way Rule provides more granular control if needed in future
- Most customer exceptions follow simpler patterns than full four-way

### Finding 4: Most Parameters Use Simple Patterns

**Pattern Distribution:**

| Parameter Type | Best Rule Type |
|----------------|----------------|
| Global infrastructure (DNS, NTP) | Global Layer (no rule) |
| Customer infrastructure (servers) | Company Layer Override (no rule) |
| Model capabilities (I/O, alarms) | Model Layer (no rule) |
| Carrier settings (APN) | Carrier Layer (no rule) |
| Plan features (firewall baseline) | Service Plan Layer (or Three-Way if varies) |
| Carrier data policies | Carrier + Plan (Two-Way) |
| Model pricing tiers | Model + Plan (Two-Way) |
| Carrier-specific model features | Model + Carrier (Two-Way) |
| Customer plan exceptions | Plan + Customer (Two-Way) |
| Complex service plan policies | Model + Carrier + Plan (Three-Way) |
| Customer exceptions to plan policies | Model + Carrier + Plan + Customer (Four-Way) |

**Implication:**
- Most configurations use simple patterns (Layers or Two-Way Rules)
- Three-Way Rules are for complex service plan policies
- Four-Way Rules are for customer-specific exceptions to those policies
- No need for other three-way combinations

---

## Business Logic Analysis

### When Would Model+Carrier+Customer Be Needed?

**Scenario:** Customer config applies to Model+Carrier combo, regardless of plan

**Real-World Example:** "CORD's i-22 VZW devices need setting X, on ANY plan"

**Why This is Unlikely:**
1. Customers typically have ONE service plan
2. Service plan defines the feature set and policies
3. Customer customizations are within the plan's context
4. Different plans have different policies - exception to one doesn't apply to others

**If This Scenario Arises:**
- Option A: Create Four-Way Rules for each plan (most accurate)
- Option B: Use Model+Carrier Two-Way Rule + Company Layer (if truly plan-independent)
- Option C: Re-evaluate whether customer should be on multiple plans

**Verdict:** Edge case, already covered by existing design

### When Would Model+Plan+Customer Be Needed?

**Scenario:** Customer config for Model+Plan combo, regardless of carrier

**Real-World Example:** "CORD's i-22 ATM devices need firewall exception, whether VZW or ATT"

**Analysis:**
- CORD's firewall exception: Add CORD RMS server (13.67.184.126)
- This server is same regardless of carrier
- Is it model-specific?
  - fw_acl syntax is same for i-22 and 4500
  - CORD RMS server IP is same for all models
  - **Model doesn't matter!**

**Better Solution:**
- Use **Service Plan + Customer Two-Way Rule** (simpler and more accurate)

**If Model Actually Matters:**
- Use Four-Way Rule with same value for all carriers

**Verdict:** Edge case, better handled by Two-Way Rule or Four-Way Rule

### When Would Carrier+Plan+Customer Be Needed?

**Scenario:** Customer config for Carrier+Plan combo, regardless of model

**Real-World Example:** "CORD on VZW ATM plan gets 10GB data limit, all models"

**Analysis:**
- Data limits typically vary by model (pricing tiers)
- i-22 is entry-level, 4500 is premium
- Would a customer exception apply to all models?
  - Probably NO - VIP customer on i-22 gets upgraded i-22 limit
  - VIP customer on 4500 gets upgraded 4500 limit
  - Limits are model-specific

**If Truly Model-Independent:**
- Use **Carrier + Customer Two-Way Rule** or **Plan + Customer Two-Way Rule**
- Or **Company Layer Override** if completely independent

**If Model Matters:**
- Use **Four-Way Rule**

**Verdict:** Edge case, already covered

---

## Architectural Considerations

### Complexity vs. Flexibility Trade-off

**Adding 3 More Three-Way Combinations Would:**
- ❌ Increase system complexity 4x
- ❌ Make rule selection harder for admins
- ❌ Require complex sub-priority ordering (which wins: M+C+Cu vs M+SP+Cu?)
- ❌ More database indexes required
- ❌ Increased documentation burden
- ❌ Confusion about when to use which type

**Benefits:**
- ✅ Slightly more granular control (minimal)
- ✅ Potentially fewer rules needed (questionable)

**Trade-off Analysis:**
- Current design: 1 three-way type + 4-way for exceptions
- Expanded design: 4 three-way types + 4-way for exceptions
- **Result:** 4x complexity for minimal benefit

### Resolution Algorithm Impact

**Current Algorithm (11-Level Priority Hierarchy):**
```
1. Device Override
2. Four-Way Rule (Model+Carrier+Plan+Customer)
3. Company Override
4. Three-Way Rule (Model+Carrier+Plan)
5. Service Plan Layer
6. Two-Way Rule (6 types with sub-priorities)
   6.1: Carrier + Service Plan
   6.2: Model + Service Plan
   6.3: Model + Carrier
   6.4: Carrier + Customer
   6.5: Model + Customer
   6.6: Service Plan + Customer
7. Carrier Layer
8. Model Layer
9. Global Layer
10. Schema Default
11. Required Validation
```

**If We Add 3 More Three-Way Types:**
```
1. Device Override
2. Four-Way Rule
3. Company Override
4. Three-Way Rule (4 types - requires sub-priority ordering):
   4.1: Model + Carrier + Plan (current)
   4.2: Model + Carrier + Customer
   4.3: Model + Plan + Customer
   4.4: Carrier + Plan + Customer
5. Service Plan Layer
6. Two-Way Rule (6 types)
... (7-11 same)
```

**Problems:**
- How to order sub-priorities? Not obvious which should win
- Model+Carrier+Customer vs Model+Plan+Customer - which is higher priority?
- Confusion for admins - when to use which type?
- Increased cognitive load for understanding resolution logic

**Verdict:** Added complexity without clear benefit

### Database Schema Impact

**Current Schema (Single Table for All Rules):**
```sql
CREATE TABLE conditional_rules (
  id INT PRIMARY KEY,
  config_key_id INT NOT NULL,
  model_id INT NULL,
  carrier_id INT NULL,
  service_plan_id INT NULL,
  company_id INT NULL,
  value TEXT NOT NULL,
  priority INT NOT NULL,        -- 2, 4, or 6 (Four/Three/Two-Way)
  sub_priority INT NULL,         -- For Two-Way: 1-6

  CHECK (
    -- Four-Way: all 4 non-NULL
    (model_id IS NOT NULL AND carrier_id IS NOT NULL
     AND service_plan_id IS NOT NULL AND company_id IS NOT NULL AND priority = 2)
    OR
    -- Three-Way: exactly 3 non-NULL (currently only M+C+SP)
    (model_id IS NOT NULL AND carrier_id IS NOT NULL
     AND service_plan_id IS NOT NULL AND company_id IS NULL AND priority = 4)
    OR
    -- Two-Way: exactly 2 non-NULL (6 combinations)
    (priority = 6 AND sub_priority BETWEEN 1 AND 6)
  )
);
```

**If We Add More Three-Way Types:**
- CHECK constraint becomes significantly more complex
- Need to track which three-way type (M+C+SP vs M+C+Cu vs M+SP+Cu vs C+SP+Cu)
- Sub-priority ordering needed at Level 4
- More indexes required for query optimization
- More complex admin interface for rule creation

**Verdict:** Significant schema complexity increase for minimal benefit

---

## Design Principle

**Use the SIMPLEST rule type that accurately models the dependency:**

```
Parameter depends on:
- 1 factor  → Use appropriate Layer (Global, Model, Carrier, Plan, Company, Device)
- 2 factors → Use Two-Way Rule (choose from 6 combinations)
- 3 factors → Use Three-Way Rule (Model + Carrier + Plan ONLY)
- 4 factors → Use Four-Way Rule (Model + Carrier + Plan + Customer)
```

**Why Three-Way is ONLY Model+Carrier+Plan:**
- Models: "Service plan features that vary by model AND carrier"
- Applies to: ALL customers on that plan (baseline/default)
- Customer exceptions: Use Four-Way Rule (adds Customer factor)

**Why Other Three-Way Combinations Aren't Needed:**
- **M+C+Customer**: Customer has one plan; use Four-Way Rule instead
- **M+Plan+Customer**: Carrier usually matters; use Four-Way or Two-Way (Plan+Customer)
- **C+Plan+Customer**: Model usually matters; use Four-Way or Two-Way (Plan+Customer)

---

## Validated Design

### Current Architecture (OPTIMAL)

**Rule Types:**
- ✅ **6 Layer Types:** Global → Model → Carrier → Plan → Company → Device
- ✅ **6 Two-Way Rule Combinations:** All combinations of 2 factors (expanded in v1.5)
  1. Model + Carrier
  2. Carrier + Service Plan
  3. Model + Service Plan
  4. Carrier + Customer
  5. Model + Customer
  6. Service Plan + Customer
- ✅ **1 Three-Way Rule Type:** Model + Carrier + Service Plan
- ✅ **1 Four-Way Rule Type:** Model + Carrier + Service Plan + Customer

**11-Level Priority Hierarchy:** Provides complete coverage of all identified real-world scenarios

### Coverage Analysis

**All Real-World Scenarios Covered:**
1. ✅ Production config analysis shows no patterns requiring other three-way combinations
2. ✅ All customer exceptions are either two-factor (Two-Way Rule) or four-factor (Four-Way Rule)
3. ✅ Three-Way Rules are for service plan BASELINES that apply to all customers
4. ✅ Four-Way Rules are for customer-specific EXCEPTIONS to those baselines

### Benefits of Current Design

1. **Simplicity:** Clear rule selection logic
2. **Clarity:** Obvious priority ordering
3. **Evidence-based:** All patterns found in production configs are supported
4. **Flexibility:** Two-Way Rules (6 types) cover simpler patterns
5. **Accuracy:** Rule type matches business logic (baselines vs exceptions)
6. **Maintainability:** Admins can easily understand when to use which rule type
7. **Performance:** Optimized resolution algorithm without complex sub-priorities

---

## Recommendation

**VALIDATED: Keep current design - do NOT add additional three-way combinations.**

### Rationale Summary

1. **Complete Coverage:** All real-world scenarios are covered by existing design
2. **Simplicity:** Adding 3 more three-way types adds 4x complexity
3. **Evidence:** Config file analysis shows no need for other combinations
4. **Flexibility:** Two-Way Rules (6 types) cover simpler patterns
5. **Accuracy:** Most "three-way" patterns are actually two-way or four-way
6. **Business Logic:** Three-Way models plan baselines; Four-Way models customer exceptions
7. **Maintainability:** Clear decision tree for rule type selection

### Risk Assessment

**Risk of Adding More Three-Way Types:**
- Increased complexity in rule selection
- Confusion about priority ordering
- More database indexes and schema complexity
- Higher cognitive load for admins
- Minimal real-world benefit

**Risk of NOT Adding More Three-Way Types:**
- None identified - all use cases covered by existing design

---

## Conclusion

The current architecture with **one three-way rule type** (Model + Carrier + Service Plan) is **optimal and validated** based on:
- Mathematical analysis of all possible combinations
- Real-world production configuration analysis
- Business logic review
- Architectural complexity considerations
- Coverage validation

**No changes needed to three-way rule design.**

---

**Document Status:** FINAL - Design Validated
**Last Updated:** March 1, 2026
**Approval:** Architecture decision confirmed based on comprehensive evidence
