# 11-Level Priority Hierarchy Test Cases
## Configuration Resolution Sample Scenarios

**Document Version:** 1.0
**Date:** February 23, 2026
**Author:** Aksana Rahouski / Orases Team
**Status:** Test Scenarios
**Parent Document:** [Config Management System PRD](./Config_Management_System_PRD.md)

---

## Purpose

This document provides comprehensive test case samples demonstrating the 11-level priority hierarchy with sub-priorities for the Hierarchical Configuration Management System. Each scenario shows when a specific level "wins" the config resolution battle.

---

## Test Device Profile

**Common Test Device:**
- **Device ID:** 12345
- **Company:** ACME Corp (company_id: 100)
- **Model:** i-22 (model_id: 1)
- **Carrier:** Verizon Wireless (carrier_id: VZW)
- **Service Plan:** ATM (service_plan_id: 2)

---

## 11-Level Priority Hierarchy Reference

When resolving a parameter value, the system checks these sources in order and uses the **first value found**:

1. **Device Override** *(Highest Priority)*
2. **Four-Way Rule** (Model + Carrier + Service Plan + Company)
3. **Company Override**
4. **Three-Way Rule** (Model + Carrier + Service Plan)
5. **Service Plan Layer**
6. **Two-Way Rules** (6 sub-priorities):
   - 6.1: Carrier + Service Plan
   - 6.2: Model + Service Plan
   - 6.3: Model + Carrier
   - 6.4: Carrier + Customer
   - 6.5: Model + Customer
   - 6.6: Service Plan + Customer
7. **Carrier Layer**
8. **Model Layer**
9. **Global Layer**
10. **Schema Default**
11. **Required Validation** *(Lowest Priority)*

---

## Test Case Scenarios

### Scenario 1: Device Override Wins (Level 1)
**Parameter:** `lan_ip`
**Use Case:** Individual device needs custom IP address for site-specific networking requirements

#### Configuration Setup:
```
Level 1 (Device Override):        192.168.50.100 ✅ WINS
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       192.168.1.10
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           192.168.0.1
Level 6.1 (Carrier+Plan):         Not set
Level 6.2 (Model+Plan):           Not set
Level 6.3 (Model+Carrier):        Not set
Level 6.4 (Carrier+Customer):     Not set
Level 6.5 (Model+Customer):       Not set
Level 6.6 (Plan+Customer):        Not set
Level 7 (Carrier):                192.168.10.1
Level 8 (Model):                  192.168.20.1
Level 9 (Global):                 192.168.100.1
Level 10 (Schema Default):        192.168.0.1
```

#### Resolution Result:
- **Resolved Value:** `192.168.50.100`
- **Source Attribution:** "Device Override (Level 1)"
- **Explanation:** Device-specific override takes absolute precedence over all other sources. Used for site-specific requirements that cannot be generalized.

#### Business Context:
Site technician configured this specific device to integrate with customer's existing network infrastructure (192.168.50.x subnet). This override ensures the device works in their environment regardless of company-wide or service plan defaults.

---

### Scenario 2: Four-Way Rule Wins (Level 2)
**Parameter:** `fw_acl` (firewall access control list)
**Use Case:** CORD company needs custom firewall rules that override the standard ATM service plan configuration

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          [50 custom CORD rules] ✅ WINS
    Rule: IF model=i-22 AND carrier=VZW AND plan=ATM AND company=CORD
    Value: fw_acl=[CORD custom ACL with 50 whitelist entries]
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         [40 standard ATM rules]
    Rule: IF model=i-22 AND carrier=VZW AND plan=ATM
    Value: fw_acl=[Standard ATM ACL with 40 whitelist entries]
Level 5 (Service Plan):           [30 basic rules]
Level 6.1-6.6 (Two-Way Rules):    Not applicable
Level 7 (Carrier):                [20 VZW rules]
Level 8 (Model):                  [10 i-22 rules]
Level 9 (Global):                 [5 default rules]
Level 10 (Schema Default):        []
```

#### Resolution Result:
- **Resolved Value:** `[50 custom CORD rules]`
- **Source Attribution:** "Four-Way Rule: i-22 + VZW + ATM + CORD (Level 2)"
- **Explanation:** Customer-specific exception to service plan policy. CORD negotiated custom security requirements that override the standard ATM service plan baseline.

#### Business Context:
CORD company (a defense contractor) requires stricter firewall policies than the standard ATM plan provides. The Four-Way Rule creates a customer-specific exception to the Three-Way Rule baseline that applies to all other ATM customers.

---

### Scenario 3: Company Override Wins (Level 3)
**Parameter:** `dns_primary`
**Use Case:** Company-wide custom DNS server for all devices

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       10.50.1.5 ✅ WINS
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           8.8.8.8
Level 6.1 (Carrier+Plan):         Not set
Level 6.2 (Model+Plan):           Not set
Level 6.3 (Model+Carrier):        Not set
Level 6.4 (Carrier+Customer):     Not set
Level 6.5 (Model+Customer):       Not set
Level 6.6 (Plan+Customer):        Not set
Level 7 (Carrier):                1.1.1.1
Level 8 (Model):                  Not set
Level 9 (Global):                 8.8.8.8
Level 10 (Schema Default):        8.8.8.8
```

#### Resolution Result:
- **Resolved Value:** `10.50.1.5`
- **Source Attribution:** "Company Override - ACME Corp (Level 3)"
- **Explanation:** ACME Corp operates their own internal DNS servers for all their devices. Company-level setting applies to all 500 devices in their fleet.

#### Business Context:
Enterprise customer with sophisticated IT infrastructure. They require all devices to use their internal DNS servers (10.50.1.5) for network resolution, overriding APW's default Google DNS (8.8.8.8). This applies to ALL 500 ACME devices regardless of model, carrier, or service plan.

---

### Scenario 4: Three-Way Rule Wins (Level 4)
**Parameter:** `traffic_day_threshold`
**Use Case:** Service plan data policy that varies by model and carrier

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         350MB ✅ WINS
    Rule: IF model=i-22 AND carrier=VZW AND plan=ATM
    Value: 350MB daily data threshold
Level 5 (Service Plan):           5GB
Level 6.1 (Carrier+Plan):         Not set
Level 6.2 (Model+Plan):           Not set
Level 6.3 (Model+Carrier):        Not set
Level 6.4 (Carrier+Customer):     Not set
Level 6.5 (Model+Customer):       Not set
Level 6.6 (Plan+Customer):        Not set
Level 7 (Carrier):                10GB
Level 8 (Model):                  1GB
Level 9 (Global):                 500MB
Level 10 (Schema Default):        100MB
```

#### Resolution Result:
- **Resolved Value:** `350MB`
- **Source Attribution:** "Three-Way Rule: i-22 + VZW + ATM (Level 4)"
- **Explanation:** ATM service plan has different data policies based on model and carrier combination. This baseline applies to ALL customers with i-22 devices on Verizon's ATM plan.

#### Business Context:
The ATM service plan is designed for low-bandwidth ATM transactions. When deployed on i-22 devices with Verizon carrier, the daily data threshold is 350MB (sufficient for ATM operations). However, if deployed on 4500 devices (different model), the threshold would be different per the Three-Way Rule for that combination.

---

### Scenario 5: Service Plan Layer Wins (Level 5)
**Parameter:** `reboot_schedule_enable`
**Use Case:** Service plan feature enablement

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           true ✅ WINS
Level 6.1 (Carrier+Plan):         Not set
Level 6.2 (Model+Plan):           Not set
Level 6.3 (Model+Carrier):        Not set
Level 6.4 (Carrier+Customer):     Not set
Level 6.5 (Model+Customer):       Not set
Level 6.6 (Plan+Customer):        Not set
Level 7 (Carrier):                Not set
Level 8 (Model):                  Not set
Level 9 (Global):                 false
Level 10 (Schema Default):        false
```

#### Resolution Result:
- **Resolved Value:** `true`
- **Source Attribution:** "Service Plan Layer - ATM (Level 5)"
- **Explanation:** ATM service plan includes scheduled reboot feature as part of the plan. Feature is enabled at plan level, overriding global default (disabled).

#### Business Context:
Premium ATM service plan includes advanced features like scheduled reboots for maintenance windows. Lower-tier plans (Tier 1, Tier 2) don't have this feature, so their devices would fall through to Global Layer (false). This differentiates service tiers.

---

### Scenario 6.1: Two-Way Rule Wins - Carrier + Service Plan (Level 6.1)
**Parameter:** `traffic_day_threshold`
**Use Case:** Carrier data policy varies by service plan tier

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           5GB
Level 6.1 (Carrier+Plan):         3584MB (3.5GB) ✅ WINS
    Rule: IF carrier=VZW AND plan=ATM
    Value: 3584MB
Level 6.2 (Model+Plan):           Not set
Level 6.3 (Model+Carrier):        Not set
Level 6.4 (Carrier+Customer):     Not set
Level 6.5 (Model+Customer):       Not set
Level 6.6 (Plan+Customer):        Not set
Level 7 (Carrier):                10GB
Level 8 (Model):                  Not set
Level 9 (Global):                 500MB
Level 10 (Schema Default):        100MB
```

#### Resolution Result:
- **Resolved Value:** `3584MB`
- **Source Attribution:** "Two-Way Rule: VZW + ATM (Level 6.1 - Carrier+Plan)"
- **Explanation:** Verizon has negotiated specific data allowances for the ATM service plan tier. This carrier-specific plan policy overrides the generic service plan setting.

#### Business Context:
Different carriers have different pricing and data policies. Verizon's ATM plan includes 3.5GB daily allowance, while AT&T's ATM plan might only include 350MB. The Two-Way Rule (Carrier+Plan) captures these carrier-specific plan variations.

---

### Scenario 6.2: Two-Way Rule Wins - Model + Service Plan (Level 6.2)
**Parameter:** `traffic_day_threshold`
**Use Case:** Model pricing tier affects service plan limits

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           5GB
Level 6.1 (Carrier+Plan):         Not set
Level 6.2 (Model+Plan):           350MB ✅ WINS
    Rule: IF model=i-22 AND plan=ATM
    Value: 350MB
Level 6.3 (Model+Carrier):        Not set
Level 6.4 (Carrier+Customer):     Not set
Level 6.5 (Model+Customer):       Not set
Level 6.6 (Plan+Customer):        Not set
Level 7 (Carrier):                Not set
Level 8 (Model):                  1GB
Level 9 (Global):                 500MB
Level 10 (Schema Default):        100MB
```

#### Resolution Result:
- **Resolved Value:** `350MB`
- **Source Attribution:** "Two-Way Rule: i-22 + ATM (Level 6.2 - Model+Plan)"
- **Explanation:** i-22 devices (lower-tier hardware) on ATM plan get restrictive data limits (350MB), while 4500 devices (higher-tier) on same plan get 5GB. Model pricing tier affects plan features.

#### Business Context:
Service plan features vary by hardware tier. Entry-level i-22 devices on ATM plan get basic data allowances (350MB sufficient for ATM transactions), while premium 4500 devices on ATM plan get higher allowances (5GB for additional features). This enables tiered pricing strategy.

---

### Scenario 6.3: Two-Way Rule Wins - Model + Carrier (Level 6.3)
**Parameter:** `mqtt_enable` (Device Manager feature)
**Use Case:** Carrier-specific features per device model

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           Not set
Level 6.1 (Carrier+Plan):         Not set
Level 6.2 (Model+Plan):           Not set
Level 6.3 (Model+Carrier):        1 (enabled) ✅ WINS
    Rule: IF model=i-22 AND carrier=VZW
    Value: mqtt_enable=1
Level 6.4 (Carrier+Customer):     Not set
Level 6.5 (Model+Customer):       Not set
Level 6.6 (Plan+Customer):        Not set
Level 7 (Carrier):                Not set
Level 8 (Model):                  0 (disabled)
Level 9 (Global):                 0 (disabled)
Level 10 (Schema Default):        0 (disabled)
```

#### Resolution Result:
- **Resolved Value:** `1` (enabled)
- **Source Attribution:** "Two-Way Rule: i-22 + VZW (Level 6.3 - Model+Carrier)"
- **Explanation:** Device Manager (MQTT) is only enabled for i-22 devices on Verizon network. AT&T i-22 devices and all 4500 devices have it disabled.

#### Business Context:
Verizon has Device Manager infrastructure that works with i-22 devices. This Two-Way Rule enables MQTT for the specific Model+Carrier combination that supports it, while keeping it disabled for unsupported combinations (AT&T i-22, or any carrier with 4500 models).

---

### Scenario 6.4: Two-Way Rule Wins - Carrier + Customer (Level 6.4)
**Parameter:** `traffic_day_threshold`
**Use Case:** Customer-specific carrier agreement

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           5GB
Level 6.1 (Carrier+Plan):         Not set
Level 6.2 (Model+Plan):           Not set
Level 6.3 (Model+Carrier):        Not set
Level 6.4 (Carrier+Customer):     10GB ✅ WINS
    Rule: IF carrier=VZW AND customer=ACME
    Value: 10GB
Level 6.5 (Model+Customer):       Not set
Level 6.6 (Plan+Customer):        Not set
Level 7 (Carrier):                3GB
Level 8 (Model):                  Not set
Level 9 (Global):                 500MB
Level 10 (Schema Default):        100MB
```

#### Resolution Result:
- **Resolved Value:** `10GB`
- **Source Attribution:** "Two-Way Rule: VZW + ACME Corp (Level 6.4 - Carrier+Customer)"
- **Explanation:** ACME Corp negotiated special data allowances with Verizon for their fleet. This applies to all ACME devices on Verizon, regardless of model or service plan.

#### Business Context:
Large enterprise customer (ACME Corp) with 500 devices negotiated bulk data rates directly with Verizon. The Carrier+Customer Two-Way Rule ensures all ACME devices on Verizon network get the negotiated 10GB daily allowance, overriding standard plan limits. Other customers on Verizon and ACME devices on other carriers don't get this benefit.

---

### Scenario 6.5: Two-Way Rule Wins - Model + Customer (Level 6.5)
**Parameter:** `digitalio_config`
**Use Case:** Customer configuration varies by deployed model

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           Not set
Level 6.1 (Carrier+Plan):         Not set
Level 6.2 (Model+Plan):           Not set
Level 6.3 (Model+Carrier):        Not set
Level 6.4 (Carrier+Customer):     Not set
Level 6.5 (Model+Customer):       [Miele custom I/O settings] ✅ WINS
    Rule: IF model=i-22 AND customer=Miele
    Value: digitalio_config=[specific I/O configuration for Miele appliances]
Level 6.6 (Plan+Customer):        Not set
Level 7 (Carrier):                Not set
Level 8 (Model):                  [default i-22 I/O settings]
Level 9 (Global):                 []
Level 10 (Schema Default):        []
```

#### Resolution Result:
- **Resolved Value:** `[Miele custom I/O settings]`
- **Source Attribution:** "Two-Way Rule: i-22 + Miele (Level 6.5 - Model+Customer)"
- **Explanation:** Miele (appliance manufacturer) has custom I/O requirements for their i-22 devices that differ from standard i-22 configuration. Applies to all Miele i-22 devices.

#### Business Context:
Miele uses i-22 devices in their commercial appliances with custom digital I/O configurations for sensor integration. The Model+Customer Two-Way Rule ensures all Miele i-22 devices get the correct I/O setup, while other customers' i-22 devices use standard configuration. If Miele also deployed 4500 devices, those would use different I/O settings (separate rule).

---

### Scenario 6.6: Two-Way Rule Wins - Service Plan + Customer (Level 6.6)
**Parameter:** `traffic_day_threshold`
**Use Case:** Customer exception to plan baseline (when Model+Carrier don't matter)

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           350MB
Level 6.1 (Carrier+Plan):         Not set
Level 6.2 (Model+Plan):           Not set
Level 6.3 (Model+Carrier):        Not set
Level 6.4 (Carrier+Customer):     Not set
Level 6.5 (Model+Customer):       Not set
Level 6.6 (Plan+Customer):        1000MB (1GB) ✅ WINS
    Rule: IF plan=ATM AND customer=Premium_Retail_Chain
    Value: 1000MB
Level 7 (Carrier):                Not set
Level 8 (Model):                  Not set
Level 9 (Global):                 500MB
Level 10 (Schema Default):        100MB
```

#### Resolution Result:
- **Resolved Value:** `1000MB`
- **Source Attribution:** "Two-Way Rule: ATM + Premium_Retail_Chain (Level 6.6 - Plan+Customer)"
- **Explanation:** Premium customer negotiated upgraded data allowance on ATM plan. Applies to all their devices on ATM plan regardless of model or carrier.

#### Business Context:
Premium_Retail_Chain is a valued customer who negotiated better terms on the ATM service plan. They get 1GB daily allowance instead of the standard 350MB, applied uniformly across all their ATM devices (any model, any carrier). This is simpler than Four-Way Rules when the exception applies across all model+carrier combinations.

---

### Scenario 7: Carrier Layer Wins (Level 7)
**Parameter:** `apn` (Access Point Name)
**Use Case:** Carrier-specific network configuration

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           Not set
Level 6.1-6.6 (Two-Way Rules):    Not set
Level 7 (Carrier):                "vzwinternet" ✅ WINS
Level 8 (Model):                  Not set
Level 9 (Global):                 "internet"
Level 10 (Schema Default):        "internet"
```

#### Resolution Result:
- **Resolved Value:** `"vzwinternet"`
- **Source Attribution:** "Carrier Layer - Verizon Wireless (Level 7)"
- **Explanation:** Verizon Wireless requires specific APN settings for cellular connectivity. All devices on Verizon network use "vzwinternet" regardless of model, plan, or customer.

#### Business Context:
Carrier network configuration is fundamental to cellular connectivity. Verizon devices must use "vzwinternet" APN, AT&T devices use "broadband", T-Mobile uses "fast.t-mobile.com". The Carrier Layer ensures all devices on a specific carrier get the correct network settings.

---

### Scenario 8: Model Layer Wins (Level 8)
**Parameter:** `alarm_io_config`
**Use Case:** Hardware-specific feature configuration

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           Not set
Level 6.1-6.6 (Two-Way Rules):    Not set
Level 7 (Carrier):                Not set
Level 8 (Model):                  [i-22 specific I/O config] ✅ WINS
Level 9 (Global):                 []
Level 10 (Schema Default):        []
```

#### Resolution Result:
- **Resolved Value:** `[i-22 specific I/O configuration]`
- **Source Attribution:** "Model Layer - i-22 (Level 8)"
- **Explanation:** i-22 device model has specific alarm I/O capabilities (8 digital inputs, 4 relay outputs) that require model-specific configuration. This hardware feature is model-dependent.

#### Business Context:
Different device models have different hardware capabilities. i-22 devices have extensive alarm I/O ports for industrial monitoring, while 4100 devices have minimal I/O. The Model Layer captures hardware-specific default configurations that apply to all devices of that model type, regardless of carrier, plan, or customer.

---

### Scenario 9: Global Layer Wins (Level 9)
**Parameter:** `time_server_primary`
**Use Case:** System-wide infrastructure setting

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           Not set
Level 6.1-6.6 (Two-Way Rules):    Not set
Level 7 (Carrier):                Not set
Level 8 (Model):                  Not set
Level 9 (Global):                 "time.nist.gov" ✅ WINS
Level 10 (Schema Default):        "pool.ntp.org"
```

#### Resolution Result:
- **Resolved Value:** `"time.nist.gov"`
- **Source Attribution:** "Global Layer (Level 9)"
- **Explanation:** APW selected NIST time servers as the standard for all devices platform-wide. No model, carrier, plan, or customer has overridden this infrastructure setting.

#### Business Context:
Infrastructure parameters like time servers, default DNS servers, and system-wide security settings are typically set at Global Layer. Unless there's a specific business reason to override (like ACME Corp's custom DNS), all 100,000+ devices inherit the global default. This ensures consistency and simplifies maintenance (change once at Global Layer, applies everywhere).

---

### Scenario 10: Schema Default Wins (Level 10)
**Parameter:** `factory_reset_enable`
**Use Case:** Security parameter with hardcoded default

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           Not set
Level 6.1-6.6 (Two-Way Rules):    Not set
Level 7 (Carrier):                Not set
Level 8 (Model):                  Not set
Level 9 (Global):                 Not set
Level 10 (Schema Default):        false ✅ WINS
```

#### Resolution Result:
- **Resolved Value:** `false`
- **Source Attribution:** "Schema Default (Level 10)"
- **Explanation:** Factory reset is disabled by default for security (prevents unauthorized device repurposing). No layer has overridden this schema default, so it applies.

#### Business Context:
Schema defaults are hardcoded safety values defined when parameters are created. They act as ultimate fallbacks when no layer provides a value. For security-sensitive parameters like factory_reset_enable, the schema default (false) ensures devices are protected even if admins forget to set the Global Layer value.

---

### Scenario 11: Required Validation Fails (Level 11)
**Parameter:** `vpn_server_address` (marked as REQUIRED)
**Use Case:** Required parameter has no value at any level - config generation fails

#### Configuration Setup:
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           Not set
Level 6.1-6.6 (Two-Way Rules):    Not set
Level 7 (Carrier):                Not set
Level 8 (Model):                  Not set
Level 9 (Global):                 Not set
Level 10 (Schema Default):        Not set
```

#### Resolution Result:
- **Resolved Value:** `ERROR - Cannot generate config`
- **Error Message:** "Required parameter 'vpn_server_address' has no value at any level. Config generation blocked."
- **Explanation:** Required validation (Level 11) prevents generating invalid configurations. If a required parameter has no value after checking all 10 sources, the system refuses to generate the config file.

#### Business Context:
This is a safety mechanism. If a parameter is marked "required" in the schema (e.g., VPN server address for secure connectivity), the system will not generate a device configuration without it. This forces admins to set the Global Layer value at minimum, preventing devices from being deployed with incomplete/invalid configurations that would cause operational failures.

**Admin Action Required:**
Administrator must set `vpn_server_address` at Global Layer (or higher specific layer) before any device can receive a valid configuration.

---

## Complex Multi-Level Scenario

### Scenario 12: Full Hierarchy Walk-Through
**Parameter:** `traffic_day_threshold`
**Device Profile:**
- Device ID: 67890
- Company: TechCorp (company_id: 250)
- Model: 4500 (model_id: 3)
- Carrier: AT&T (carrier_id: ATT)
- Service Plan: Super Tier (service_plan_id: 5)

#### Configuration Setup (Full Stack):
```
Level 1 (Device Override):        Not set → continue
Level 2 (Four-Way Rule):          Not set → continue
    No rule for: 4500 + ATT + SuperTier + TechCorp
Level 3 (Company Override):       Not set → continue
Level 4 (Three-Way Rule):         25GB → continue? NO, check first...
    Rule: IF model=4500 AND carrier=ATT AND plan=SuperTier
    Value: 25GB
    ✅ MATCH FOUND - STOP HERE
Level 5 (Service Plan):           35GB (not checked)
Level 6.1 (Carrier+Plan):         15GB (not checked)
Level 6.2 (Model+Plan):           20GB (not checked)
Level 6.3 (Model+Carrier):        Not set (not checked)
Level 6.4 (Carrier+Customer):     Not set (not checked)
Level 6.5 (Model+Customer):       Not set (not checked)
Level 6.6 (Plan+Customer):        Not set (not checked)
Level 7 (Carrier):                10GB (not checked)
Level 8 (Model):                  30GB (not checked)
Level 9 (Global):                 500MB (not checked)
Level 10 (Schema Default):        100MB (not checked)
```

#### Resolution Result:
- **Resolved Value:** `25GB`
- **Source Attribution:** "Three-Way Rule: 4500 + ATT + SuperTier (Level 4)"
- **Explanation:** The Three-Way Rule defines the service plan baseline for this specific model+carrier+plan combination. Even though Service Plan Layer (Level 5) has 35GB and Model Layer (Level 8) has 30GB, the Three-Way Rule at Level 4 takes precedence because it was found first in the priority order.

#### Why Other Levels Didn't Win:
- **Level 1 (Device):** No device-specific override configured
- **Level 2 (Four-Way):** No customer-specific exception rule for TechCorp
- **Level 3 (Company):** TechCorp didn't set company-wide data threshold override
- **Level 4 (Three-Way):** ✅ **FOUND HERE** - Stops resolution
- **Levels 5-10:** Not evaluated (first-match-wins principle)

#### Business Context:
Super Tier service plan on 4500 devices with AT&T carrier has a defined baseline of 25GB daily data (sufficient for high-bandwidth applications). This is lower than the generic Service Plan setting (35GB at Level 5) because it's specifically tuned for the 4500+ATT combination's network performance characteristics. The Three-Way Rule provides precision tuning that overrides both generic plan settings and model defaults.

---

## Sub-Priority Resolution Example

### Scenario 13: Two-Way Rules Sub-Priority Competition
**Parameter:** `traffic_day_threshold`
**Device Profile:** Same as main test device (i-22 + VZW + ATM + ACME)

#### Configuration Setup (Multiple Two-Way Rules Set):
```
Level 1 (Device Override):        Not set
Level 2 (Four-Way Rule):          Not set
Level 3 (Company Override):       Not set
Level 4 (Three-Way Rule):         Not set
Level 5 (Service Plan):           5GB
Level 6 (Two-Way Rules):          Multiple rules exist!
    Level 6.1 (Carrier+Plan):     3584MB → VZW + ATM ✅ WINS (checked first)
    Level 6.2 (Model+Plan):       350MB  → i-22 + ATM (not checked - lower priority)
    Level 6.3 (Model+Carrier):    1GB    → i-22 + VZW (not checked - lower priority)
    Level 6.4 (Carrier+Customer): 10GB   → VZW + ACME (not checked - lower priority)
    Level 6.5 (Model+Customer):   2GB    → i-22 + ACME (not checked - lower priority)
    Level 6.6 (Plan+Customer):    1GB    → ATM + ACME (not checked - lower priority)
Level 7 (Carrier):                10GB (not checked)
Level 8 (Model):                  1GB (not checked)
Level 9 (Global):                 500MB (not checked)
Level 10 (Schema Default):        100MB (not checked)
```

#### Resolution Result:
- **Resolved Value:** `3584MB`
- **Source Attribution:** "Two-Way Rule: VZW + ATM (Level 6.1 - Carrier+Plan)"
- **Explanation:** When multiple Two-Way Rules match the device's attributes, the sub-priority order determines the winner. Carrier+Plan (6.1) has highest priority among Two-Way Rules and is checked first.

#### Why This Sub-Priority Order?
1. **6.1 Carrier+Plan** - Carrier policies by service tier (most specific business rule)
2. **6.2 Model+Plan** - Hardware tier pricing by plan (business logic)
3. **6.3 Model+Carrier** - Technical compatibility (original design)
4. **6.4 Carrier+Customer** - Customer carrier agreements (business relationships)
5. **6.5 Model+Customer** - Customer hardware configs (customization)
6. **6.6 Plan+Customer** - Customer plan exceptions (last resort override)

#### Business Context:
This scenario shows why sub-priorities matter. The device technically matches all 6 two-way rules:
- It's a VZW device on ATM plan (6.1) ✅
- It's an i-22 device on ATM plan (6.2) ✅
- It's an i-22 device on VZW (6.3) ✅
- It's ACME's device on VZW (6.4) ✅
- It's ACME's i-22 device (6.5) ✅
- It's ACME's device on ATM plan (6.6) ✅

Without sub-priority ordering, the system wouldn't know which rule to use. The sub-priority order ensures carrier-specific plan policies (6.1) take precedence over other factors, reflecting the business reality that carrier agreements heavily influence service plan features.

---

## Testing Recommendations

### Test Coverage Requirements

**1. Single-Level Tests (11 tests)**
- One test per level where only that level has a value
- Verify correct source attribution in output
- Verify inheritance skips empty levels correctly

**2. Multi-Level Competition Tests (55+ tests)**
- Test every adjacent level pair (1 vs 2, 2 vs 3, ..., 10 vs 11)
- Test non-adjacent pairs (1 vs 5, 3 vs 9, etc.)
- Test complex scenarios with multiple levels set
- Verify higher priority always wins

**3. Two-Way Rules Sub-Priority Tests (21 tests)**
- Test each sub-priority alone (6 tests)
- Test all pairs competing (6.1 vs 6.2, 6.1 vs 6.3, ...) - 15 tests
- Verify sub-priority order is enforced

**4. NULL Override Tests (6 tests)**
- Test explicit NULL at each layer (Device, Company, Service Plan, Carrier, Model, Global)
- Verify NULL prevents inheritance from lower-priority layers
- Verify "Not Set" (no row) vs NULL (row with NULL value) behavior difference

**5. Required Parameter Tests (3 tests)**
- Required parameter with value at Global → success
- Required parameter with no value anywhere → error
- Required parameter with NULL at all layers → error

**6. Conditional Rules Edge Cases (10+ tests)**
- Device matches multiple Two-Way rule types simultaneously
- Device matches Two-Way and Three-Way rules simultaneously (Three-Way wins)
- Device matches Three-Way and Four-Way rules simultaneously (Four-Way wins)
- Device attributes don't match any rule conditions (falls through to layers)
- Rule with NULL value (explicit disable)

**7. Real-World Integration Tests (10+ tests)**
- Full device profile with realistic layer stack
- Service plan change triggers config version update
- Company override applies to all company devices
- Migration scenario (old system → new system)
- Customer self-service restriction enforcement

### Expected Total Tests: ~116+ test cases

---

## Validation Checklist

For each test case, verify:

- ✅ **Correct value resolved** - Expected value matches actual value
- ✅ **Correct source attribution** - System reports correct winning level
- ✅ **First-match-wins enforced** - Lower priority levels not evaluated after match
- ✅ **NULL handling correct** - Explicit NULL vs "not set" behavior
- ✅ **Required validation enforced** - Config generation fails when required param missing
- ✅ **Sub-priority order correct** - Two-Way Rules checked in correct order (6.1 → 6.6)
- ✅ **Conditional rules logic** - Device attributes correctly matched to rule conditions
- ✅ **Audit trail captured** - Config resolution logged with source attribution
- ✅ **Performance acceptable** - Resolution completes in <100ms for single device
- ✅ **Error messages clear** - Validation failures provide actionable feedback

---

## Notes for Testers

1. **Device Attributes Matter**: Ensure test devices have correct model_id, carrier_id, service_plan_id, and company_id for conditional rules to match properly.

2. **Database State**: Tests should start with known database state (seed data). Use fixtures or factories to create consistent test data.

3. **Isolation**: Each test should be independent. Don't rely on side effects from previous tests.

4. **Negative Tests**: Include tests for invalid states (e.g., missing required parameters, invalid rule conditions, malformed values).

5. **Performance Tests**: Include benchmarks for config resolution with various hierarchy depths and rule complexity.

6. **Integration Tests**: Test actual device check-in flow end-to-end (check-in → version compare → config generation → config delivery).

7. **Customer Safety**: Test that customer users CANNOT access admin-only layers or restricted parameters.

8. **Migration Safety**: Test parallel operation (old system and new system running simultaneously without interference).

---

## Appendix: Quick Reference Matrix

| Level | Priority | Rule Type | Factors | Use Case |
|-------|----------|-----------|---------|----------|
| 1 | Highest | Device Override | Device ID | Site-specific customization |
| 2 | Very High | Four-Way Rule | Model+Carrier+Plan+Customer | Customer exception to plan baseline |
| 3 | High | Company Override | Company ID | Portfolio-wide settings |
| 4 | Medium-High | Three-Way Rule | Model+Carrier+Plan | Service plan baseline for all customers |
| 5 | Medium | Service Plan Layer | Plan ID | Service tier differentiation |
| 6.1 | Medium-Low | Two-Way Rule | Carrier+Plan | Carrier policy by plan tier |
| 6.2 | Medium-Low | Two-Way Rule | Model+Plan | Model pricing tier by plan |
| 6.3 | Medium-Low | Two-Way Rule | Model+Carrier | Carrier features per model |
| 6.4 | Medium-Low | Two-Way Rule | Carrier+Customer | Customer carrier agreements |
| 6.5 | Medium-Low | Two-Way Rule | Model+Customer | Customer model configurations |
| 6.6 | Medium-Low | Two-Way Rule | Plan+Customer | Customer plan exceptions |
| 7 | Low | Carrier Layer | Carrier ID | Carrier network settings |
| 8 | Very Low | Model Layer | Model ID | Hardware-specific defaults |
| 9 | Minimal | Global Layer | System-wide | Infrastructure defaults |
| 10 | Fallback | Schema Default | Hardcoded | Safety fallback values |
| 11 | Lowest | Required Validation | Error | Prevents invalid configs |

---

**END OF DOCUMENT**
