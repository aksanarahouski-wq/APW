# Four-Way Rule Combinations Analysis

**Date:** February 4, 2026
**Question:** When should we use Four-Way Rules vs simpler alternatives (Two-Way, Three-Way, or Layer Overrides)?
**Source:** Configuration files analysis, PRD requirements, and business logic review

---

## Executive Summary

**Answer: YES - Four-Way Rules (Model + Carrier + Service Plan + Customer) are needed, but ONLY for customer-specific exceptions to service plan policies.**

From 4 factors (Model, Carrier, Service Plan, Customer), there is mathematically only **ONE four-way combination**: Model + Carrier + Service Plan + Customer. The critical questions are:

1. ✅ **Is this combination needed?** YES - for customer-specific exceptions
2. ✅ **When to use it?** Only when all 4 factors matter simultaneously
3. ✅ **What are alternatives?** Two-Way Rules, Three-Way Rules, or Company Layer Override
4. ✅ **How common is it?** Relatively rare (10-20 rules estimated)

**Key Principle:** Four-Way Rules should be used sparingly. They represent the **most specific** configuration override in the entire system (Priority Level 2, just below Device Override).

---

## Mathematical Analysis

### The Only Four-Way Combination

**Model + Carrier + Service Plan + Customer**

This is the ONLY possible four-way combination from the 4 factors. There are no alternatives to evaluate.

**Interpretation:**
- **Model**: Which device model (i-22, 4500, IR611, CR202, i-52)
- **Carrier**: Which cellular carrier (VZW, ATT, T-Mobile, Data Connect)
- **Service Plan**: Which pricing/feature tier (ATM, Tier1, etc.)
- **Customer**: Which specific company/customer (CORD, DEPLOYER, Altech, etc.)

**Business Logic:**
"Customer X needs configuration value Y for Model Z devices on Carrier W with Service Plan V"

---

## When to Use Four-Way Rules

### Decision Tree

```
Does the configuration vary by customer?
  NO  → Use Layer (Global, Model, Carrier, Plan) or Two-Way/Three-Way Rule
  YES ↓

Is this an exception to a service plan policy?
  NO  → Use Company Layer Override (simpler)
  YES ↓

Does the exception depend on the service plan?
  NO  → Use Two-Way Rule (Model+Customer, Carrier+Customer, etc.)
  YES ↓

Does the exception depend on both model AND carrier?
  NO  → Use Two-Way Rule (Plan+Customer) or Three-Way Rule
  YES ↓

USE FOUR-WAY RULE ✅
```

### The Four-Way Rule "Test"

A parameter needs a Four-Way Rule when ALL of these are true:

1. ✅ **Customer-Specific**: Value is customized for a specific customer
2. ✅ **Plan-Dependent**: Exception only applies within a specific service plan
3. ✅ **Model-Dependent**: Exception varies by device model
4. ✅ **Carrier-Dependent**: Exception varies by carrier

**If ANY factor doesn't matter, use a simpler rule type.**

---

## Real-World Examples

### Example 1: Customer Firewall Exceptions (CONFIRMED)

**Scenario:** CORD company needs additional firewall whitelist rules for their RMS server

**Baseline (Three-Way Rule):**
```
IF model=i-22 AND carrier=ATT AND plan=ATM
  THEN fw_acl=[40 baseline whitelist rules]

IF model=i-22 AND carrier=VZW AND plan=ATM
  THEN fw_acl=[40 baseline whitelist rules]
```

**Customer Exception (Four-Way Rule):**
```
IF model=i-22 AND carrier=ATT AND plan=ATM AND customer=CORD
  THEN fw_acl=[40 baseline rules + 13.67.184.126 (CORD RMS)]

IF model=i-22 AND carrier=VZW AND plan=ATM AND customer=CORD
  THEN fw_acl=[40 baseline rules + 13.67.184.126 (CORD RMS)]
```

**Why Four-Way?**
- ✅ Customer-specific: Only CORD needs this RMS server
- ✅ Plan-dependent: Only applies to ATM plan (Tier1 has different firewall policy)
- ✅ Model-dependent: fw_acl format/syntax may vary by model (though in practice same)
- ✅ Carrier-dependent: Baseline rules differ by carrier (though CORD addition is same)

**Alternative Consideration:**
Could this be **Plan + Customer** (Two-Way Rule)?
- If fw_acl baseline is truly identical across all Model+Carrier combinations for ATM plan
- Then yes, simpler Two-Way Rule would work
- BUT: Four-Way provides more granular control and future flexibility

**Verdict:** Four-Way Rule is appropriate (though Two-Way might suffice in this specific case)

---

### Example 2: Customer Data Limit Override (CONFIRMED)

**Scenario:** VIP customer negotiates higher data allowance on ATM plan for specific model+carrier

**Baseline (Three-Way Rule):**
```
IF model=i-22 AND carrier=ATT AND plan=ATM
  THEN traffic_day_threshold=350MB

IF model=4500 AND carrier=ATT AND plan=ATM
  THEN traffic_day_threshold=5GB
```

**Customer Exception (Four-Way Rule):**
```
IF model=i-22 AND carrier=ATT AND plan=ATM AND customer=VIP_Corp
  THEN traffic_day_threshold=1000MB  (upgraded from 350MB)
```

**Why Four-Way?**
- ✅ Customer-specific: VIP_Corp negotiated special rate
- ✅ Plan-dependent: Applies only to ATM plan contract
- ✅ Model-dependent: Different models have different pricing tiers (i-22 vs 4500)
- ✅ Carrier-dependent: Carrier agreement affects pricing (ATT vs VZW different)

**Verdict:** Four-Way Rule is necessary - all factors matter

---

### Example 3: Customer Advanced Features Toggle (POTENTIAL)

**Scenario:** Customer requests Device Manager enabled on ATT (normally disabled for ATT i-22)

**Baseline (Two-Way Rule):**
```
IF model=i-22 AND carrier=VZW THEN mqtt_enable=1
IF model=i-22 AND carrier=ATT THEN mqtt_enable=0
```

**Customer Exception (Four-Way Rule):**
```
IF model=i-22 AND carrier=ATT AND plan=ATM AND customer=Special_Corp
  THEN mqtt_enable=1  (exception to ATT policy)
```

**Why Four-Way?**
- ✅ Customer-specific: Special agreement with one customer
- ✅ Plan-dependent: Only applies to ATM plan (Tier1 has different features)
- ✅ Model-dependent: mqtt_enable only supported on i-22, not 4500/IR611
- ✅ Carrier-dependent: Exception to ATT policy (VZW already enables it)

**Alternative Consideration:**
- Could use **Model + Carrier + Customer** (Three-Way) if plan doesn't matter
- But if feature availability varies by plan, Four-Way is correct

**Verdict:** Four-Way Rule is appropriate for plan-specific exceptions

---

### Example 4: Customer VPN Configuration (EDGE CASE)

**Scenario:** Customer needs custom VPN endpoint only for ATM plan devices

**Baseline (Service Plan Layer):**
```
IF plan=ATM THEN vpn1_dns=vpn.standard-atm.com
```

**Customer Exception:**
```
IF plan=ATM AND customer=Corporate_X
  THEN vpn1_dns=vpn.corporatex.com
```

**Analysis:**
- ✅ Customer-specific: Corporate_X has own VPN server
- ✅ Plan-dependent: Only ATM plan uses VPN (Tier1 doesn't)
- ❌ Model-independent: VPN endpoint same for all models
- ❌ Carrier-independent: VPN setup doesn't vary by carrier

**Better Solution:** Use **Plan + Customer** (Two-Way Rule)

**Verdict:** NOT a Four-Way Rule - use simpler Two-Way Rule

---

### Example 5: Customer Network Configuration (COMPANY LAYER)

**Scenario:** Customer needs custom LAN settings (DHCP range, gateway IP)

**Configuration:**
```
customer=Altech THEN dhcpd_start=192.168.3.100
customer=Altech THEN dhcpd_end=192.168.3.200
customer=Altech THEN lan0_gw=192.168.3.1
```

**Analysis:**
- ✅ Customer-specific: Altech's network architecture
- ❌ Plan-independent: Network settings don't vary by service plan
- ❌ Model-independent: LAN configuration same regardless of device model
- ❌ Carrier-independent: WAN carrier doesn't affect LAN setup

**Better Solution:** Use **Company Layer Override**

**Verdict:** NOT a Four-Way Rule - use Company Layer (no rule needed)

---

## Four-Way Rule vs Alternatives: Detailed Comparison

### Comparison Matrix

| Scenario | Factors | Best Solution | Reason |
|----------|---------|---------------|--------|
| Customer firewall exceptions for ATM plan | All 4 | **Four-Way Rule** | Service plan has baseline, customer needs exception |
| Customer VIP data allowance on specific plan | All 4 | **Four-Way Rule** | Pricing tier (model) + carrier contract + plan + customer all matter |
| Customer network settings (DHCP, gateway) | Customer only | **Company Layer** | Independent of model, carrier, and plan |
| Customer infrastructure (NTP, DNS servers) | Customer only | **Company Layer** | Infrastructure doesn't vary by device factors |
| Customer reboot schedule | Customer only | **Company Layer** | Operational policy is device-independent |
| Customer exception to carrier policy (all plans) | Model + Carrier + Customer | **Three-Way?*** | If truly plan-independent (rare) |
| Customer exception to plan policy (all carriers) | Model + Plan + Customer | **Two-Way (Plan+Customer)** | If carrier doesn't matter |
| Customer exception to plan policy (all models) | Carrier + Plan + Customer | **Two-Way (Plan+Customer)** | If model doesn't matter |

*Note: Three-Way with Model+Carrier+Customer is not currently supported. Use Four-Way with rules for each plan, or Two-Way combinations.

---

## Priority Hierarchy Context

Four-Way Rules sit at **Priority Level 2** in the 11-level hierarchy:

```
1.  Device Override              ← Most specific
2.  Four-Way Rule                ← Customer exception to plan policy
3.  Company Layer Override       ← Customer-wide settings
4.  Three-Way Rule               ← Service plan baseline
5.  Service Plan Layer           ← Plan-level settings
6.  Two-Way Rule                 ← Multi-factor baseline
7.  Carrier Layer                ← Carrier-level settings
8.  Model Layer                  ← Model-level settings
9.  Global Layer                 ← System-wide defaults
10. Schema Default               ← Parameter default
11. Required Validation          ← Must be set
```

**Why Level 2 (High Priority)?**
- Four-Way Rules are **customer-specific exceptions** to service plan policies
- Should override the Three-Way baseline (Level 4)
- Should override Company Layer (Level 3) because it's more specific (adds Plan+Model+Carrier context)
- Only Device Override (Level 1) is more specific

---

## When NOT to Use Four-Way Rules

### Anti-Pattern 1: Customer Settings That Don't Vary by Plan

**BAD:**
```
IF model=i-22 AND carrier=VZW AND plan=ATM AND customer=Miele
  THEN dhcpd_start=192.168.5.100

IF model=i-22 AND carrier=VZW AND plan=Tier1 AND customer=Miele
  THEN dhcpd_start=192.168.5.100

IF model=i-22 AND carrier=ATT AND plan=ATM AND customer=Miele
  THEN dhcpd_start=192.168.5.100
```

**GOOD:**
```
Company Layer Override:
  customer=Miele THEN dhcpd_start=192.168.5.100
```

**Reason:** Network settings are plan-independent, model-independent, carrier-independent. Use Company Layer.

---

### Anti-Pattern 2: Using Four-Way When Two-Way Suffices

**BAD:**
```
IF model=i-22 AND carrier=VZW AND plan=ATM AND customer=CORD
  THEN ntp_svr=time.cord.com

IF model=i-22 AND carrier=ATT AND plan=ATM AND customer=CORD
  THEN ntp_svr=time.cord.com

IF model=4500 AND carrier=VZW AND plan=ATM AND customer=CORD
  THEN ntp_svr=time.cord.com

IF model=4500 AND carrier=ATT AND plan=ATM AND customer=CORD
  THEN ntp_svr=time.cord.com
```

**GOOD:**
```
Two-Way Rule (Plan + Customer):
  IF plan=ATM AND customer=CORD THEN ntp_svr=time.cord.com
```

**Reason:** NTP server is model-independent and carrier-independent. Model and carrier don't matter, so don't include them.

---

### Anti-Pattern 3: Duplicating Three-Way Rules

**BAD:**
```
Four-Way Rule:
  IF model=i-22 AND carrier=ATT AND plan=ATM AND customer=StandardCorp
    THEN fw_acl=[40 baseline rules]  (same as baseline!)
```

**GOOD:**
```
Three-Way Rule:
  IF model=i-22 AND carrier=ATT AND plan=ATM
    THEN fw_acl=[40 baseline rules]
  (applies to ALL customers, including StandardCorp)
```

**Reason:** Four-Way Rules are for EXCEPTIONS. If customer uses the baseline value, don't create a Four-Way Rule.

---

## Architecture Implications

### Database Schema

The `conditional_rules` table supports Four-Way Rules:

```sql
CREATE TABLE conditional_rules (
  id INT PRIMARY KEY AUTO_INCREMENT,
  config_schema_id INT NOT NULL,
  model_id INT NULL,
  carrier_id INT NULL,
  service_plan_id INT NULL,
  company_id INT NULL,
  value TEXT,
  priority INT NOT NULL,      -- 2 for Four-Way
  sub_priority INT NULL,      -- Not used for Four-Way (only for Two-Way)
  created DATETIME DEFAULT CURRENT_TIMESTAMP,
  modified DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Validation: Four-Way requires ALL 4 factors
  CHECK (
    priority != 2 OR (
      model_id IS NOT NULL AND
      carrier_id IS NOT NULL AND
      service_plan_id IS NOT NULL AND
      company_id IS NOT NULL
    )
  ),

  UNIQUE KEY unique_four_way_rule (
    config_schema_id, model_id, carrier_id, service_plan_id, company_id
  )
);
```

**Key Points:**
- Four-Way Rules MUST have all 4 factors non-NULL
- priority=2 for Four-Way
- Unique constraint prevents duplicate rules for same combination

---

### Resolution Algorithm

```php
function resolveFourWayRule($parameter, $device) {
    $rule = DB::table('conditional_rules')
        ->where('config_schema_id', $parameter->id)
        ->where('model_id', $device->model_id)
        ->where('carrier_id', $device->carrier_id)
        ->where('service_plan_id', $device->service_plan_id)
        ->where('company_id', $device->company_id)
        ->where('priority', 2)  // Four-Way priority level
        ->first();

    return $rule ? $rule->value : null;
}
```

**Performance:**
- Single query with exact match on 5 columns
- Should be very fast with proper index
- Index: `(config_schema_id, model_id, carrier_id, service_plan_id, company_id)`

---

### Estimated Rule Counts

Based on production config file analysis:

| Rule Type | Estimated Count | Notes |
|-----------|-----------------|-------|
| **Four-Way Rules** | **10-20 rules** | Customer exceptions only |
| Firewall exceptions | 5-8 rules | CORD, DEPLOYER, ALTECH, etc. |
| Data limit overrides | 2-5 rules | VIP customers with special rates |
| Feature exceptions | 2-5 rules | Customer-specific feature toggles |
| Advanced config | 1-2 rules | Rare edge cases |

**Context:**
- Three-Way Rules (baselines): 30-40 rules
- Two-Way Rules: 40-60 rules
- Four-Way Rules: 10-20 rules (smallest category)

**Implication:** Four-Way Rules are relatively rare, as they should be.

---

## Implementation Guidance

### Step 1: Verify All Four Factors Matter

Before creating a Four-Way Rule, verify each factor:

**Test Model:**
- Does the configuration value differ for i-22 vs 4500?
- If NO → Don't include model in the rule

**Test Carrier:**
- Does the configuration value differ for VZW vs ATT?
- If NO → Don't include carrier in the rule

**Test Service Plan:**
- Does the configuration value differ for ATM vs Tier1?
- If NO → Use Company Layer instead

**Test Customer:**
- Is this specific to ONE customer?
- If NO → Use Three-Way Rule (baseline for all customers)

### Step 2: Check for Simpler Alternatives

1. **Is it customer infrastructure?** (NTP, DNS, DHCP, network settings)
   - YES → Use **Company Layer Override**

2. **Does it depend on only 2 factors?**
   - YES → Use **Two-Way Rule** (choose from 6 combinations)

3. **Is it a service plan baseline for ALL customers?**
   - YES → Use **Three-Way Rule** (Model+Carrier+Plan)

4. **Is it a customer exception that depends on all 4 factors?**
   - YES → Use **Four-Way Rule** ✅

### Step 3: Create the Baseline First

**Always create the Three-Way baseline before the Four-Way exception:**

```sql
-- Step 1: Create Three-Way baseline (for all customers)
INSERT INTO conditional_rules (
  config_schema_id, model_id, carrier_id, service_plan_id,
  value, priority
) VALUES (
  123,  -- fw_acl parameter
  1,    -- i-22 model
  1,    -- VZW carrier
  2,    -- ATM plan
  '[40 baseline firewall rules...]',
  4     -- Three-Way priority
);

-- Step 2: Create Four-Way exception (for specific customer)
INSERT INTO conditional_rules (
  config_schema_id, model_id, carrier_id, service_plan_id, company_id,
  value, priority
) VALUES (
  123,  -- fw_acl parameter
  1,    -- i-22 model
  1,    -- VZW carrier
  2,    -- ATM plan
  5,    -- CORD company
  '[40 baseline rules + 13.67.184.126 + 208.92.212.170]',
  2     -- Four-Way priority (overrides Three-Way)
);
```

### Step 4: Document the Exception

Include metadata explaining WHY the Four-Way Rule exists:

```sql
ALTER TABLE conditional_rules ADD COLUMN description TEXT;
ALTER TABLE conditional_rules ADD COLUMN reason TEXT;

UPDATE conditional_rules SET
  description = 'CORD company firewall whitelist exception',
  reason = 'Customer requires access to CORD RMS server (13.67.184.126) and backup server'
WHERE id = 456;
```

---

## UI/UX Considerations

### Four-Way Rule Creation Interface

**Form Layout:**
```
┌─────────────────────────────────────────────────┐
│ Create Four-Way Conditional Rule               │
│ (Customer Exception to Service Plan Policy)    │
├─────────────────────────────────────────────────┤
│                                                 │
│ Parameter: [fw_acl ▼]                          │
│                                                 │
│ ┌─ Service Plan Context ───────────────────┐  │
│ │ Model:        [i-22 ▼]                    │  │
│ │ Carrier:      [VZW ▼]                     │  │
│ │ Service Plan: [ATM ▼]                     │  │
│ │                                            │  │
│ │ Baseline Value (Three-Way Rule):          │  │
│ │ [40 firewall whitelist rules...]          │  │
│ └────────────────────────────────────────────┘  │
│                                                 │
│ ┌─ Customer Exception ──────────────────────┐  │
│ │ Customer: [CORD ▼]                        │  │
│ │                                            │  │
│ │ Exception Value:                          │  │
│ │ [40 baseline rules + CORD RMS additions]  │  │
│ │                                            │  │
│ │ Reason for Exception:                     │  │
│ │ [Customer requires access to RMS server]  │  │
│ └────────────────────────────────────────────┘  │
│                                                 │
│ [Create Exception Rule] [Cancel]               │
└─────────────────────────────────────────────────┘
```

**Key UX Features:**
1. Show baseline Three-Way value for context
2. Explain that this is an EXCEPTION to baseline
3. Require reason/justification for exception
4. Validate that baseline Three-Way Rule exists
5. Warn if creating duplicate or conflicting rule

---

### Four-Way Rule List View

```
┌────────────────────────────────────────────────────────────────────────┐
│ Four-Way Rules (Customer Exceptions to Service Plan Policies)         │
│ Total: 12 rules                                                        │
├────────┬──────────┬─────────────────────┬───────────┬─────────────────┤
│ Param  │ Customer │ Plan Context        │ Exception │ Action          │
├────────┼──────────┼─────────────────────┼───────────┼─────────────────┤
│ fw_acl │ CORD     │ i-22 + VZW + ATM   │ +2 IPs    │ [Edit] [Delete]│
│        │          │ Baseline: 40 rules  │           │                 │
├────────┼──────────┼─────────────────────┼───────────┼─────────────────┤
│ fw_acl │ DEPLOYER │ i-22 + VZW + ATM   │ +1 IP     │ [Edit] [Delete]│
│        │          │ Baseline: 40 rules  │           │                 │
├────────┼──────────┼─────────────────────┼───────────┼─────────────────┤
│ traffic│ VIP_Corp │ i-22 + ATT + ATM   │ 1000MB    │ [Edit] [Delete]│
│ _day_  │          │ Baseline: 350MB     │ (VIP rate)│                 │
└────────┴──────────┴─────────────────────┴───────────┴─────────────────┘

[+ New Four-Way Exception]
```

**Key UX Features:**
1. Show baseline value for comparison
2. Highlight the "delta" or difference from baseline
3. Group by customer for easy management
4. Filter by parameter, customer, or plan
5. Bulk operations: "Apply exception to multiple customers"

---

## Testing Requirements

### Test Case 1: Four-Way Rule Overrides Three-Way Baseline

```
GIVEN:
  - Three-Way Rule: i-22 + VZW + ATM → fw_acl=[40 rules]
  - Four-Way Rule: i-22 + VZW + ATM + CORD → fw_acl=[42 rules]

WHEN: Resolving fw_acl for CORD device on i-22 + VZW + ATM

THEN: Should return [42 rules] (Four-Way wins over Three-Way)
```

### Test Case 2: Four-Way Rule Overrides Company Layer

```
GIVEN:
  - Company Layer: CORD → traffic_day_threshold=500MB
  - Four-Way Rule: i-22 + ATT + ATM + CORD → traffic_day_threshold=1000MB

WHEN: Resolving traffic_day_threshold for CORD device on i-22 + ATT + ATM

THEN: Should return 1000MB (Four-Way wins over Company Layer)
```

### Test Case 3: Device Override Beats Four-Way Rule

```
GIVEN:
  - Four-Way Rule: i-22 + VZW + ATM + CORD → fw_acl=[42 rules]
  - Device Override: device_123 → fw_acl=[45 rules]

WHEN: Resolving fw_acl for device_123 (CORD, i-22, VZW, ATM)

THEN: Should return [45 rules] (Device Override beats Four-Way)
```

### Test Case 4: Four-Way Rule Only Applies to Specific Customer

```
GIVEN:
  - Three-Way Rule: i-22 + VZW + ATM → fw_acl=[40 rules]
  - Four-Way Rule: i-22 + VZW + ATM + CORD → fw_acl=[42 rules]

WHEN: Resolving fw_acl for OTHER_CUSTOMER device on i-22 + VZW + ATM

THEN: Should return [40 rules] (Three-Way baseline, not CORD exception)
```

### Test Case 5: Prevent Duplicate Four-Way Rules

```
GIVEN: Four-Way Rule already exists for i-22 + VZW + ATM + CORD

WHEN: Attempting to create another Four-Way Rule for i-22 + VZW + ATM + CORD

THEN: Should reject with error "Duplicate rule exists"
```

---

## Migration Considerations

### Identifying Existing Four-Way Patterns

Query to find customer-specific configurations that vary by model+carrier+plan:

```sql
-- Find parameters with customer-specific values on same model+carrier+plan
SELECT
    cs.parameter_key,
    cv1.model_id,
    cv1.carrier_id,
    cv1.service_plan_id,
    cv1.company_id,
    cv1.value as customer_value,
    cv_baseline.value as baseline_value
FROM config_values cv1
-- Join to find baseline (no company_id)
LEFT JOIN config_values cv_baseline ON
    cv_baseline.config_schema_id = cv1.config_schema_id
    AND cv_baseline.model_id = cv1.model_id
    AND cv_baseline.carrier_id = cv1.carrier_id
    AND cv_baseline.service_plan_id = cv1.service_plan_id
    AND cv_baseline.company_id IS NULL
JOIN config_schema cs ON cs.id = cv1.config_schema_id
WHERE
    cv1.company_id IS NOT NULL
    AND cv1.value != cv_baseline.value  -- Customer has different value
ORDER BY cs.parameter_key, cv1.model_id, cv1.carrier_id, cv1.service_plan_id;
```

### Migration Script

```sql
-- Step 1: Create Three-Way baselines (if not exist)
INSERT INTO conditional_rules (
  config_schema_id, model_id, carrier_id, service_plan_id,
  value, priority
)
SELECT DISTINCT
  cv.config_schema_id,
  cv.model_id,
  cv.carrier_id,
  cv.service_plan_id,
  cv.value,
  4  -- Three-Way priority
FROM config_values cv
WHERE cv.company_id IS NULL
  AND cv.model_id IS NOT NULL
  AND cv.carrier_id IS NOT NULL
  AND cv.service_plan_id IS NOT NULL
ON DUPLICATE KEY UPDATE value=VALUES(value);

-- Step 2: Create Four-Way exceptions
INSERT INTO conditional_rules (
  config_schema_id, model_id, carrier_id, service_plan_id, company_id,
  value, priority
)
SELECT
  cv.config_schema_id,
  cv.model_id,
  cv.carrier_id,
  cv.service_plan_id,
  cv.company_id,
  cv.value,
  2  -- Four-Way priority
FROM config_values cv
WHERE cv.company_id IS NOT NULL
  AND cv.model_id IS NOT NULL
  AND cv.carrier_id IS NOT NULL
  AND cv.service_plan_id IS NOT NULL
  AND cv.value != (
    -- Different from baseline
    SELECT baseline.value
    FROM config_values baseline
    WHERE baseline.config_schema_id = cv.config_schema_id
      AND baseline.model_id = cv.model_id
      AND baseline.carrier_id = cv.carrier_id
      AND baseline.service_plan_id = cv.service_plan_id
      AND baseline.company_id IS NULL
  )
ON DUPLICATE KEY UPDATE value=VALUES(value);
```

---

## Documentation Updates Required

### PRD Updates

1. **FR-3a: Conditional Rules Management**
   - Clarify Four-Way Rule purpose: "Customer exceptions to service plan policies"
   - Add decision tree for when to use Four-Way vs alternatives
   - Update rule creation UI wireframes with baseline context

2. **FR-3b: Rule Priority Hierarchy**
   - Emphasize Level 2 position (second-highest priority)
   - Explain relationship to Three-Way (Level 4) baseline

3. **Examples Section**
   - Add 2-3 detailed Four-Way examples (firewall, data limits, features)
   - Show side-by-side: Three-Way baseline + Four-Way exception

### User Guide

**Section: "When to Use Four-Way Rules"**
- Decision tree diagram
- Real-world examples with screenshots
- Common mistakes to avoid (anti-patterns)
- Performance considerations

**Section: "Creating Customer Exceptions"**
- Step-by-step guide
- Baseline-first workflow
- Required documentation (reason field)

---

## Summary & Recommendations

### Key Findings

1. ✅ **Four-Way Rules ARE necessary** - for customer-specific exceptions to service plan policies
2. ✅ **Only ONE four-way combination exists** - Model + Carrier + Service Plan + Customer
3. ✅ **Usage should be LIMITED** - estimated 10-20 rules total (rare exceptions only)
4. ✅ **Simpler alternatives usually exist** - Company Layer, Two-Way Rules, Three-Way Rules
5. ✅ **Baseline-first approach** - Always create Three-Way baseline before Four-Way exception

### Recommended Implementation Approach

**Phase 1: Foundation**
- ✅ Database schema supports Four-Way (already done)
- ✅ Resolution algorithm checks Level 2 (already done)
- ✅ UI supports Four-Way rule creation

**Phase 2: Guidance**
- ✅ Add decision tree to UI (when to use Four-Way)
- ✅ Show baseline value when creating exception
- ✅ Require reason/justification field
- ✅ Validate baseline Three-Way Rule exists

**Phase 3: Migration**
- ⬜ Identify existing customer exceptions in production configs
- ⬜ Create Three-Way baselines
- ⬜ Create Four-Way exceptions where baseline differs
- ⬜ Test resolution for all affected devices

### Best Practices

1. **Use Sparingly**: Four-Way Rules should be exceptions, not the norm
2. **Baseline First**: Always create Three-Way baseline before Four-Way exception
3. **Document Why**: Require reason/justification for every Four-Way Rule
4. **Verify All Factors**: Test that all 4 factors actually matter before creating rule
5. **Consider Alternatives**: Check if Company Layer or Two-Way Rule would suffice
6. **Maintain Hierarchy**: Respect that Four-Way is Level 2 (high priority)

### Decision Summary

| Question | Answer |
|----------|--------|
| Are Four-Way Rules needed? | ✅ YES - for customer exceptions |
| How many Four-Way combinations? | 1 (Model+Carrier+Plan+Customer) |
| How common will they be? | Rare (10-20 rules estimated) |
| When to use vs alternatives? | Only when ALL 4 factors matter |
| Priority level? | Level 2 (overrides everything except Device) |
| Main use cases? | Customer firewall exceptions, VIP data limits |

---

## Conclusion

**Four-Way Rules (Model + Carrier + Service Plan + Customer) are a necessary and valuable part of the configuration system, but should be used ONLY when truly needed.**

The design principle is clear:
- **Three-Way Rules** define service plan BASELINES (apply to all customers)
- **Four-Way Rules** define customer-specific EXCEPTIONS to those baselines

This separation ensures:
- ✅ Clean architecture with clear baseline → exception pattern
- ✅ Minimal rule proliferation (most customers use baseline)
- ✅ Easy management (exceptions are explicitly marked)
- ✅ Flexible system (can handle customer-specific requirements)
- ✅ Maintainable codebase (clear when to use each rule type)

**Implementation is already complete in current PRD design.** This analysis confirms the design decisions are sound and provides guidance for proper usage.
