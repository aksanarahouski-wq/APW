# Three-Way Rule Combinations Analysis

**Date:** January 12, 2026
**Question:** Do we need to support additional 3-way combinations beyond Model+Carrier+ServicePlan?
**Source:** Configuration files analysis and business logic review

---

## Executive Summary

**Answer: NO - Additional three-way combinations are NOT needed.**

After comprehensive analysis of production config files and business logic, the current **Model + Carrier + Service Plan** three-way rule design is sufficient. Other possible 3-factor combinations are either:
1. **Already covered** by expanded Two-Way Rules (6 combinations)
2. **Better handled** by Four-Way Rules (customer exceptions)
3. **Not needed** based on actual usage patterns

---

## Possible Three-Way Combinations (Mathematical)

From 4 factors (Model, Carrier, Service Plan, Customer), there are C(4,3) = **4 possible three-way combinations**:

1. ✅ **Model + Carrier + Service Plan** (current design)
2. ❓ **Model + Carrier + Customer**
3. ❓ **Model + Service Plan + Customer**
4. ❓ **Carrier + Service Plan + Customer**

Let's analyze each combination to determine if it's needed.

---

## Combination 1: Model + Carrier + Service Plan (Current Design)

**Status:** ✅ **Already Supported**

**Use Case:** Service plan features that vary by both model AND carrier

**Business Logic:**
- Defines BASELINE configuration for a specific Model+Carrier+ServicePlan combination
- Applies to ALL customers on that plan with that model+carrier
- Example: "All i-22 + ATT + ATM devices get 40 firewall whitelist rules"

**Real-World Evidence:**
```
ATT + i-22 + ATM → fw_acl=[40 baseline rules]
VZW + i-22 + ATM → fw_acl=[40 baseline rules] (same list!)
ATT + 4500 + ATM → fw_acl=[40 baseline rules] (same list!)
```

**Key Finding:** In practice, many Three-Way "Rules" have the same value across different model+carrier combinations. The firewall baseline for ATM plan is **model-independent and carrier-independent**.

**Why Keep It Anyway:**
- Flexibility for future requirements
- Some parameters DO vary by model+carrier (e.g., traffic_day_threshold showed model AND carrier matter)
- Provides consistent framework even if some rules have same values

**Verdict:** ✅ **Keep as primary Three-Way Rule type**

---

## Combination 2: Model + Carrier + Customer

**Status:** ❌ **NOT Needed**

**Hypothetical Use Case:** Customer-specific configuration for a model+carrier combo, regardless of service plan

**Example:** "CORD's i-22 VZW devices get setting X, on ANY service plan"

**Analysis:**

### Question 1: Do customers have multiple service plans?
**Research:** Typically NO
- Most customers are on a single service plan (e.g., ATM plan)
- Customers rarely have devices on multiple plans simultaneously
- If they do, configurations are usually plan-specific

### Question 2: Would a customer configuration apply across all plans?
**Research:** Unlikely
- Service plans define feature sets and policies
- Customer customizations are usually within the context of their plan
- Different plans have different firewall policies, data limits, feature sets
- Customer exceptions to "ATM plan policy" don't apply to "Tier1 plan policy"

### Question 3: Can existing design handle this?
**Answer:** YES, multiple ways:

**If customer config applies to ONE plan:**
- Use **Four-Way Rule**: Model + Carrier + Plan + Customer ✅ (current design)

**If customer config truly applies to ALL plans** (rare):
- Option A: Use **Model + Carrier** (Two-Way Rule) + **Company Layer Override** ✅ (already supported)
- Option B: Create multiple **Four-Way Rules**, one per plan ✅ (already supported)
- Option C: Use **Model + Customer** (Two-Way Rule) if carrier doesn't matter ✅ (already supported)

**Real-World Evidence:**
- Customer firewall exceptions (CORD, DEPLOYER) are explicitly tied to "ATM Service Plan"
- No evidence of customer configs that span multiple service plans

**Verdict:** ❌ **NOT Needed** - Use Four-Way Rules for customer exceptions

---

## Combination 3: Model + Service Plan + Customer

**Status:** ❌ **NOT Needed** (covered by Two-Way Rules or Four-Way Rules)

**Hypothetical Use Case:** Customer-specific configuration for a model+plan combo, regardless of carrier

**Example:** "CORD's i-22 devices on ATM plan get setting X, whether VZW or ATT"

**Analysis:**

### Question 1: Are there carrier-independent customer customizations?
**Research:** Some examples:
- ✅ Customer-specific servers (fw_acl additions) - same regardless of carrier
- ✅ Custom DHCP ranges - network topology is carrier-independent
- ✅ Custom NTP servers - infrastructure is carrier-independent
- ✅ Custom scheduling (reboot times) - operational policy is carrier-independent

### Question 2: Do these also depend on model?
**Research:**

**Firewall Rules (fw_acl):**
- CORD needs to whitelist: 13.67.184.126 (CORD RMS server)
- Does this depend on model? **Let's check:**
  ```
  ATT + i-22 + ATM: fw_acl=[40 baseline rules] (same list)
  VZW + i-22 + ATM: fw_acl=[40 baseline rules] (same list)
  ATT + 4500 + ATM: fw_acl=[40 baseline rules] (same list)
  ```
- Finding: fw_acl syntax and content are **model-independent** AND **carrier-independent**
- CORD's additional servers would be added to same baseline, regardless of model or carrier

**DHCP Settings:**
- Custom dhcpd_start/end for Miele, Altech
- Does this depend on model? **NO** - network topology is device-independent
- Does this depend on carrier? **NO** - LAN configuration is unrelated to WAN carrier

**NTP Servers:**
- Custom time.customer.com
- Model-independent? **YES**
- Carrier-independent? **YES**

### Question 3: Can existing design handle Model+Plan+Customer?

**Answer:** YES, already covered:

**If it's model-independent (most common):**
- Use **Service Plan + Customer** (Two-Way Rule) ✅ (v1.5 expansion)
- This is simpler and more accurate

**If it truly needs model but not carrier (rare):**
- Use **Model + Service Plan** (Two-Way Rule) ✅ (v1.5 expansion) for baseline
- Plus **Model + Customer** (Two-Way Rule) ✅ (v1.5 expansion) for customer override
- Or use **Four-Way Rule** with same value for all carriers

**If it needs model AND carrier:**
- Use **Four-Way Rule**: Model + Carrier + Plan + Customer ✅ (current design)

**Real-World Evidence:**
```
Customer: CORD
Service Plan: ATM
Firewall Exception: Add 13.67.184.126 (CORD RMS)
Applies to:
  - i-22 + ATT + ATM + CORD → Add CORD RMS
  - i-22 + VZW + ATM + CORD → Add CORD RMS
  - 4500 + ATT + ATM + CORD → Add CORD RMS
  - 4500 + VZW + ATM + CORD → Add CORD RMS

Pattern: Service Plan + Customer (TWO factors, not three)
```

**Verdict:** ❌ **NOT Needed**
- Use **Service Plan + Customer** (Two-Way Rule) for carrier-independent exceptions
- Use **Four-Way Rule** if carrier or model actually matters

---

## Combination 4: Carrier + Service Plan + Customer

**Status:** ❌ **NOT Needed**

**Hypothetical Use Case:** Customer-specific configuration for a carrier+plan combo, regardless of model

**Example:** "CORD on VZW ATM plan gets X data limit, applies to i-22, 4500, any model"

**Analysis:**

### Question 1: Are there model-independent customer customizations?
**Research:**
- Custom data limits? **NO** - different models have different pricing tiers (i-22 vs 4500)
- Custom VPN settings? **Possibly** - customer VPN endpoint might be model-independent
- Custom scheduling? **YES** - reboot times are model-independent
- Custom infrastructure? **YES** - NTP, DNS servers are model-independent

### Question 2: Do carrier+plan settings vary by customer?
**Research:**

**Data Limits:**
- Base: Carrier + Service Plan determines limit (Two-Way Rule already supported)
  - VZW + ATM → 3584MB
  - ATT + ATM → varies by model (350MB for i-22, 5GB for 4500)
- Customer override: VIP customer gets higher limit
  - If model-independent: Use **Carrier + Customer** (Two-Way) ✅
  - If model matters: Use **Four-Way Rule** ✅

**Network Settings:**
- VPN endpoint: vpn.customer.com
- Does this vary by carrier? **NO** - customer infrastructure is same
- Does this vary by model? **NO**
- Does this vary by plan? **Possibly** - different plans might have different VPN policies
- **Pattern: Service Plan + Customer** (Two-Way Rule) ✅ or **Company Layer** ✅

### Question 3: Can existing design handle Carrier+Plan+Customer?

**Answer:** YES, already covered:

**If it's truly model-independent (most cases):**
- Use **Carrier + Service Plan + Customer** → Wait, this is what we're analyzing!
- Actually, for most parameters that vary by Carrier+Plan, model ALSO matters
- Example: traffic_day_threshold varies by Carrier, Plan, AND Model (as we saw in Two-Way analysis)

**If model doesn't matter:**
- Use **Carrier + Plan** (Two-Way Rule) ✅ for baseline
- Plus **Carrier + Customer** (Two-Way Rule) ✅ or **Plan + Customer** (Two-Way Rule) ✅ for override
- Or use **Company Layer Override** ✅

**If model matters:**
- Use **Four-Way Rule** ✅

**Real-World Evidence:**
- Traffic limits: Model matters (i-22 vs 4500 different tiers)
- Firewall rules: Carrier doesn't matter (same servers for VZW and ATT)
- Most customer customizations are either:
  - Completely infrastructure-independent (Company Layer)
  - Or tied to specific Model+Carrier+Plan combination (Four-Way Rule)

**Verdict:** ❌ **NOT Needed**
- Use **Two-Way Rules** (Carrier+Plan, Carrier+Customer, Plan+Customer)
- Use **Four-Way Rule** if all factors matter

---

## Key Findings from Config File Analysis

### Finding 1: ATM Plan Baselines are Model+Carrier Independent

**Evidence:**
```bash
ATT + i-22 + ATM:  fw_acl=[40 whitelist rules] (identical list)
VZW + i-22 + ATM:  fw_acl=[40 whitelist rules] (identical list)
ATT + 4500 + ATM:  fw_acl=[40 whitelist rules] (identical list)
VZW + 4500 + ATM:  fw_acl=[40 whitelist rules] (identical list)
```

**Implication:**
- The "Three-Way Rule" for fw_acl could actually be **Service Plan Layer** value
- Carrier doesn't change it, Model doesn't change it
- BUT: Keeping Three-Way Rule design provides flexibility for future requirements

### Finding 2: Customer Exceptions are Service-Plan-Specific

**Evidence:**
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
- Customer exceptions could be handled by **Service Plan + Customer** (Two-Way Rule)
- But Four-Way Rule provides more granular control if needed

### Finding 4: Most Parameters Have Simple Patterns

**Pattern Distribution:**
```
Parameter Type                          | Best Rule Type
----------------------------------------|------------------------------------------
Global infrastructure (DNS, NTP)        | Global Layer (no rule)
Customer infrastructure (servers)       | Company Layer Override (no rule)
Model capabilities (I/O, alarms)        | Model Layer (no rule)
Carrier settings (APN)                  | Carrier Layer (no rule)
Plan features (firewall baseline)       | Service Plan Layer (or Three-Way if varies)
Carrier data policies                   | Carrier + Plan (Two-Way)
Model pricing tiers                     | Model + Plan (Two-Way)
Carrier-specific model features         | Model + Carrier (Two-Way)
Customer plan exceptions                | Plan + Customer (Two-Way)
Complex service plan policies           | Model + Carrier + Plan (Three-Way)
Customer exceptions to plan policies    | Model + Carrier + Plan + Customer (Four-Way)
```

**Implication:**
- Most configurations use simple patterns (Layers or Two-Way Rules)
- Three-Way Rules are for complex service plan policies
- Four-Way Rules are for customer-specific exceptions to those policies
- No need for other three-way combinations

---

## Business Logic Analysis

### When Would Model+Carrier+Customer Be Needed?

**Scenario:** Customer config applies to a specific model+carrier combo, regardless of plan

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

**Scenario:** Customer config for a specific model+plan combo, regardless of carrier

**Real-World Example:** "CORD's i-22 ATM devices need firewall exception, whether VZW or ATT"

**Analysis:**
- CORD's firewall exception: Add CORD RMS server (13.67.184.126)
- This server is same regardless of carrier
- But is it model-specific?
  - fw_acl syntax is same for i-22 and 4500
  - CORD RMS server IP is same for all models
  - **Model doesn't matter!**

**Better Solution:**
- Use **Service Plan + Customer** (Two-Way Rule)
- Simpler and more accurate

**If Model Actually Matters:**
- Use Four-Way Rule with same value for all carriers
- Or create separate rules per carrier (more explicit)

**Verdict:** Edge case, better handled by Two-Way Rule or Four-Way Rule

### When Would Carrier+Plan+Customer Be Needed?

**Scenario:** Customer config for a specific carrier+plan combo, regardless of model

**Real-World Example:** "CORD on VZW ATM plan gets 10GB data limit, all models"

**Analysis:**
- Data limits typically vary by model (pricing tiers)
- i-22 is entry-level, 4500 is premium
- Would a customer exception apply to all models?
  - Probably NO - VIP customer on i-22 gets upgraded i-22 limit
  - VIP customer on 4500 gets upgraded 4500 limit
  - Limits are model-specific

**If Truly Model-Independent:**
- Use **Carrier + Customer** (Two-Way Rule) or **Plan + Customer** (Two-Way Rule)
- Or **Company Layer Override** if completely independent

**If Model Matters:**
- Use **Four-Way Rule**

**Verdict:** Edge case, already covered

---

## Architectural Considerations

### Complexity vs. Flexibility

**Adding More Three-Way Combinations Would:**
- ❌ Increase system complexity
- ❌ Make rule selection harder for admins
- ❌ Increase resolution algorithm complexity
- ❌ Require more database indexes
- ❌ Add more documentation burden

**Benefits:**
- ✅ Slightly more granular control
- ✅ Potentially fewer rules needed (maybe)

**Trade-off Analysis:**
- Current design: 1 three-way type + 4-way for exceptions
- Expanded design: 4 three-way types + 4-way for exceptions
- **Result:** 4x complexity for minimal benefit

### Resolution Algorithm Impact

**Current Algorithm (11-Level with expanded Two-Way):**
```
1. Device Override
2. Four-Way Rule
3. Company Override
4. Three-Way Rule (1 type: Model+Carrier+Plan)
5. Service Plan Layer
6. Two-Way Rule (6 types, sub-priorities)
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
4. Three-Way Rule (4 types, sub-priorities needed):
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
- **Verdict:** Added complexity without clear benefit

### Database Schema Impact

**Current Schema (Single Table for All Rules):**
```sql
CREATE TABLE conditional_rules (
  model_id INT NULL,
  carrier_id INT NULL,
  service_plan_id INT NULL,
  company_id INT NULL,
  value TEXT,
  priority INT,        -- 2, 4, or 6 (Four/Three/Two-Way)
  sub_priority INT,    -- For Two-Way: 1-6
  ...
  CHECK (
    -- Four-Way: all 4 non-NULL
    -- Three-Way: exactly 3 non-NULL (currently only M+C+SP)
    -- Two-Way: exactly 2 non-NULL (6 combinations)
  )
);
```

**If We Add More Three-Way Types:**
- CHECK constraint becomes more complex
- Need to track which three-way type (M+C+SP vs M+C+Cu vs M+SP+Cu vs C+SP+Cu)
- Sub-priority ordering needed
- More indexes required

**Verdict:** Significant schema complexity increase

---

## Recommendation: KEEP CURRENT DESIGN

### Summary

**NO - Do NOT add additional three-way combinations.**

The current design with:
- ✅ **Model + Carrier + Service Plan** (Three-Way Rule)
- ✅ **Model + Carrier + Service Plan + Customer** (Four-Way Rule)
- ✅ **6 Two-Way Rule combinations** (v1.5 expansion)
- ✅ **Layer inheritance** (6 layers)

...is **sufficient and optimal** for all identified use cases.

### Rationale

1. **Coverage:** All real-world scenarios are covered by existing design
2. **Simplicity:** Adding 3 more three-way types adds 4x complexity
3. **Evidence:** Config file analysis shows no need for other combinations
4. **Flexibility:** Two-Way Rules (6 types) cover simpler patterns
5. **Accuracy:** Most "three-way" patterns are actually two-way or four-way

### Design Principle

**Use the SIMPLEST rule type that accurately models the dependency:**

```
Parameter depends on:
- 1 factor  → Use appropriate Layer (Global, Model, Carrier, Plan, Company, Device)
- 2 factors → Use Two-Way Rule (choose from 6 combinations)
- 3 factors → Use Three-Way Rule (Model + Carrier + Plan)
- 4 factors → Use Four-Way Rule (Model + Carrier + Plan + Customer)
```

**Why Three-Way is ONLY Model+Carrier+Plan:**
- This models: "Service plan features that vary by model AND carrier"
- Applies to: ALL customers on that plan
- Examples: Plan-specific firewall baseline, plan-specific feature sets
- Customer exceptions: Use Four-Way Rule (adds Customer factor)

**Why Other Three-Way Combinations Aren't Needed:**
- **M+C+Customer**: Customer has one plan; use Four-Way Rule
- **M+Plan+Customer**: Carrier usually matters; use Four-Way or Two-Way (Plan+Customer)
- **C+Plan+Customer**: Model usually matters; use Four-Way or Two-Way (Plan+Customer)

---

## Real-World Use Case Resolution

### Use Case 1: Customer Firewall Exception

**Requirement:** CORD needs custom firewall rules (add CORD RMS servers)

**Current Design Solution:**
```
Step 1: Create Three-Way Rule baseline
  IF model=i-22 AND carrier=ATT AND plan=ATM THEN fw_acl=[40 baseline rules]
  IF model=i-22 AND carrier=VZW AND plan=ATM THEN fw_acl=[40 baseline rules]
  (repeat for all combinations)

Step 2: Create Four-Way Rule exception
  IF model=i-22 AND carrier=ATT AND plan=ATM AND customer=CORD
    THEN fw_acl=[40 baseline + CORD RMS server]
```

**Alternative with M+Plan+Customer Three-Way:**
```
Step 1: Create Model+Plan baseline (if carrier doesn't matter)
  IF model=i-22 AND plan=ATM THEN fw_acl=[40 baseline rules]

Step 2: Create Model+Plan+Customer exception
  IF model=i-22 AND plan=ATM AND customer=CORD
    THEN fw_acl=[40 baseline + CORD RMS]
```

**Analysis:**
- Alternative is simpler (fewer baseline rules needed)
- BUT: Loss of flexibility if carrier matters in future
- ALSO: Can achieve same with Two-Way Rule:
  ```
  Baseline: plan=ATM → fw_acl=[40 rules] (Service Plan Layer)
  Exception: plan=ATM AND customer=CORD → fw_acl=[40 + CORD] (Two-Way Rule)
  ```

**Verdict:** Current design (Four-Way) or Two-Way (Plan+Customer) both work. No need for new three-way type.

### Use Case 2: Customer Data Limit Override

**Requirement:** VIP customer gets higher data allowance

**Current Design Solution:**
```
Baseline: Carrier+Plan Two-Way Rule
  IF carrier=VZW AND plan=ATM THEN traffic_day_threshold=3584MB

Override: Carrier+Customer Two-Way Rule
  IF carrier=VZW AND customer=VIP_Corp THEN traffic_day_threshold=10GB
```

**Alternative with C+Plan+Customer Three-Way:**
```
IF carrier=VZW AND plan=ATM AND customer=VIP_Corp
  THEN traffic_day_threshold=10GB
```

**Analysis:**
- Alternative is more explicit
- BUT: Model usually matters for data limits (pricing tiers)
- Better solution: Use Four-Way Rule if model matters
- Or use existing Two-Way Rule (Carrier+Customer)

**Verdict:** Existing design is sufficient

### Use Case 3: Customer Scheduling

**Requirement:** Customer wants all devices to reboot at 3 AM

**Current Design Solution:**
```
Company Layer Override:
  customer=CORD → cron_rb_enable=1, cron_rb_time=180
```

**Analysis:**
- This is model-independent, carrier-independent, plan-independent
- Company Layer is the correct place for this
- No rule needed at all!

**Verdict:** Layer inheritance handles this perfectly

---

## Conclusion

**Final Recommendation: NO additional three-way combinations needed.**

The current architecture with:
- 6 Layer types (Global → Model → Carrier → Plan → Company → Device)
- 6 Two-Way Rule combinations (expanded in v1.5)
- 1 Three-Way Rule type (Model + Carrier + Plan)
- 1 Four-Way Rule type (Model + Carrier + Plan + Customer)

...provides complete coverage of all identified real-world scenarios while maintaining architectural simplicity and clarity.

**Evidence:**
- ✅ Production config analysis shows no patterns requiring other three-way combinations
- ✅ All customer exceptions are either two-factor (Two-Way Rule) or four-factor (Four-Way Rule)
- ✅ Three-Way Rules are for service plan BASELINES that apply to all customers
- ✅ Four-Way Rules are for customer-specific EXCEPTIONS to those baselines

**Risk of Adding More Three-Way Types:**
- Increased complexity in rule selection
- Confusion about priority ordering
- More database indexes
- Limited real-world benefit

**Verdict:** **KEEP CURRENT DESIGN** - it's optimal for the identified requirements.

