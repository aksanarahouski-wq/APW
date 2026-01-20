# Configuration Parameter Dependencies: Complete Analysis
## Model + Carrier + Service Plan Conditional Logic

**Date:** January 3, 2026 (Updated: January 5, 2026)
**Analysis Scope:**
- **Two-Way Analysis:** Model + Carrier (4 combinations: 2 models × 2 carriers)
- **Three-Way Analysis:** Model + Carrier + Service Plan (8 combinations: 2 models × 2 carriers × 2 plans)
- **Four-Way Analysis:** Model + Carrier + Service Plan + Company (Customer-specific overrides)
- **Configuration Files Analyzed:** 18 total (11 base + 7 customer configs)
- **Common Parameters:** 622-624

---

## Executive Summary

### Overall Dependency Statistics

| Dependency Type | Parameters | Percentage | Notes |
|----------------|------------|------------|-------|
| **Consistent across all combinations** | 586 | 94.2% | Same value for all model/carrier/plan combinations |
| **Two-Way (Model + Carrier)** | 22 | 3.5% | Value depends on specific model AND carrier combo |
| **Three-Way (Model + Carrier + Service Plan)** | 19 | 3.1% | Value depends on all three factors |
| **Four-Way (Model + Carrier + Service Plan + Company)** | 7 | 1.1% | **NEW**: Customer-specific overrides of service plan policies |
| **Carrier-Only** | 8 | 1.3% | Same across models, varies by carrier |
| **Model-Only** | 8 | 1.3% | Same across carriers, varies by model |
| **Service-Plan driven** | 8 | 1.3% | Primarily driven by service plan tier |

### Key Findings

1. **Model + Carrier dependencies are real** - 22 parameters require dual-condition logic
2. **Service Plan is a first-class factor** - 19 parameters need three-way logic (Model + Carrier + Service Plan)
3. **Service Plan dominates security/features** - ATM service plan gets extensive firewall rules, web filtering, higher data limits
4. **VZW Model 22 = High-performance** - Gets 3.5GB data allowance, QoS always enabled, advanced features
5. **Customer overrides of service plan policies** - 7 parameters with three-way rules are also overridden at customer level, requiring four-way conditional logic
6. **Most parameters are consistent** - 94%+ of parameters work the same across all combinations

---

## Part 1: Two-Way Dependencies (Model + Carrier)

### Analysis Scope
- **Carriers:** ATT, DC, VZW
- **Models:** 22, 4500
- **Files:** 4 configurations covering major combinations

### Two-Way Dependent Parameters (22 total)

#### **Critical Business Logic Parameters**

##### 1. **advanced** (Advanced Mode Enable)

| Model 22 + ATT | Model 22 + DC | Model 22 + VZW | Model 4500 + VZW |
|----------------|---------------|----------------|------------------|
| 0 (disabled)   | 0 (disabled)  | **1 (enabled)** | 0 (disabled)    |

```sql
IF (model=22 AND carrier=VZW) THEN advanced=1 ELSE advanced=0
```

**Note:** Later analysis reveals this is actually three-way dependent when service plan is added.

##### 2. **alarm_output_options** (Alarm Output Configuration)

| Model 22 + ATT | Model 22 + DC | Model 22 + VZW | Model 4500 + VZW |
|----------------|---------------|----------------|------------------|
| cli,out-dm,out-rmon | cli,out-dm | cli,out-dm,out-rmon | cli,out-dm |

```sql
IF (model=22 AND carrier IN [ATT, VZW]) THEN alarm_output_options='cli,out-dm,out-rmon,'
ELSE IF (model=22 AND carrier=DC) THEN alarm_output_options='cli,out-dm,'
ELSE alarm_output_options='cli,out-dm,'
```

##### 3. **console_enable** (Console Access)

| Model 22 + ATT | Model 22 + DC | Model 22 + VZW | Model 4500 + VZW |
|----------------|---------------|----------------|------------------|
| 0 (disabled)   | 1 (enabled)   | 1 (enabled)    | 0 (disabled)     |

```sql
IF (model=22 AND carrier IN [DC, VZW]) THEN console_enable=1 ELSE console_enable=0
```

**Note:** Three-way analysis shows this is more complex with service plan variations.

##### 4. **dns_static** (Static DNS Servers)

| Model 22 + ATT | Model 22 + DC | Model 22 + VZW | Model 4500 + VZW |
|----------------|---------------|----------------|------------------|
| 8.8.8.8;1.1.1.1 | 8.8.8.8;8.8.4.4 | 8.8.8.8;1.1.1.1 | 8.8.8.8;8.8.4.4 |

**Observation:** Same carrier (VZW) but different DNS servers based on model.

##### 5. **fw_acl** (Firewall ACL Rules)

| Model 22 + ATT | Model 22 + DC | Model 22 + VZW | Model 4500 + VZW |
|----------------|---------------|----------------|------------------|
| Minimal rules  | Minimal rules | Additional network segment | Minimal rules |

**Note:** Three-way analysis reveals this is primarily service-plan driven (ATM vs tier1).

##### 6. **mqtt_enable** (MQTT Service Enable)

| Model 22 + ATT | Model 22 + DC | Model 22 + VZW | Model 4500 + VZW |
|----------------|---------------|----------------|------------------|
| 0 (disabled)   | 1 (enabled)   | 1 (enabled)    | 0 (disabled)     |

```sql
IF (model=22 AND carrier IN [DC, VZW]) THEN mqtt_enable=1 ELSE mqtt_enable=0
```

##### 7. **ntp_server** (NTP Time Servers)

| Model 22 + ATT | Model 22 + DC | Model 22 + VZW | Model 4500 + VZW |
|----------------|---------------|----------------|------------------|
| 10.4.6.30;time.nist.gov;time.google.com | 10.4.6.30;time.google.com;time.nist.gov | 10.4.6.30;time.nist.gov;time.google.com | time.nist.gov;74.118.247.208;time.google.com |

**Pattern:** Model 22 prioritizes internal NTP (10.4.6.30), Model 4500 varies by service plan.

##### 8. **qos_iface** (QoS Interface)

| Model 22 + ATT | Model 22 + DC | Model 22 + VZW | Model 4500 + VZW |
|----------------|---------------|----------------|------------------|
| (empty)        | wan1          | wan1           | (empty)          |

```sql
IF (model=22 AND carrier IN [DC, VZW]) THEN qos_iface='wan1' ELSE qos_iface=''
```

**Note:** Three-way analysis shows Model 4500 + ATM service plan also enables QoS.

##### 9. **sms_enable** (SMS Control Enable)

| Model 22 + ATT | Model 22 + DC | Model 22 + VZW | Model 4500 + VZW |
|----------------|---------------|----------------|------------------|
| 1 (enabled)    | 0 (disabled)  | 1 (enabled)    | 0 (disabled)     |

```sql
IF (model=22 AND carrier IN [ATT, VZW]) THEN sms_enable=1 ELSE sms_enable=0
```

##### 10-11. **sms_rb** and **sms_sq** (SMS Commands)
Related to sms_enable - set to "REB" and "STA" when SMS is enabled.

##### 12. **ssl_server** (SSL Tunnel Endpoints)
- VZW + Model 22: Uses `208.224.248.160:1440` for port 7003
- Other combinations: Use `atm1.switchcommerce.net:1440`

##### 13-14. **traffic_day_threshold** and **traffic_day_unit**

| Model 22 + ATT | Model 22 + DC | Model 22 + VZW | Model 4500 + VZW |
|----------------|---------------|----------------|------------------|
| 5 (unit=2)     | 5 (unit=2)    | 3584 (unit=1, **3.5GB**) | 5 (unit=2) |

**Critical Finding:** VZW Model 22 gets 700x higher data threshold (3584MB vs 5MB).
**Note:** Three-way analysis shows ATT+22+ATM gets 350MB (service plan factor).

##### 15. **wan1_icmp_interval** (ICMP Keepalive Interval)

| Model 22 + ATT | Model 22 + DC | Model 22 + VZW | Model 4500 + VZW |
|----------------|---------------|----------------|------------------|
| 3000ms         | 3600ms        | 3600ms         | 3000ms           |

##### 16-17. **wan1_mtu** and **wan3_mtu** (WAN MTU Size)

| Model 22 + ATT | Model 22 + DC | Model 22 + VZW | Model 4500 + VZW |
|----------------|---------------|----------------|------------------|
| 1500           | 1428          | 1500           | 1428             |

```sql
IF ((model=22 AND carrier=DC) OR (model=4500 AND carrier=VZW)) THEN mtu=1428 ELSE mtu=1500
```

##### 18-22. **Device-Specific Parameters**
- hostname (unique, includes model/carrier in name)
- lan0_mac, wan0_mac (hardware MAC addresses)
- ovdp_device_id (unique device identifier)
- oem_name (encrypted OEM name)

### Carrier-Only Dependencies (8 parameters)

**1. Dual SIM Configuration**
- DC: Dual SIM enabled with quality thresholds (`dual_sim_enable=1`, `dual_sim_min_csq=5`, `dual_sim_min_conn_time=180`)
- ATT/VZW: Disabled

**2. Alarm Input/Output**
- ATT: Specific alarm configs (`alarm_input=0,0,1,1,0,0,0,0,0,0,0,`)
- DC/VZW: Empty

**3. WAN PPP Configuration**
- ATT: `wan1_ppp_apn=matrix.com.attz`, `wan1_ppp_provider=1`
- DC/VZW: `wan1_ppp_apn=Matrxatm.gw12.vzwentp`, `wan1_ppp_provider=2`

### Model-Only Dependencies (8 parameters)

**1. DHCP Configuration**
- Model 22: `dhcpd_start=192.168.1.100`, `dhcpd_lease=60` minutes
- Model 4500: `dhcpd_start=192.168.1.91`, `dhcpd_lease=600` minutes (10 hours)

**2. MQTT Configuration**
- Model 22: Configured (`mqtt_center=iot.inhandnetworks.com`)
- Model 4500: Empty (no MQTT support at hardware level)

**3. Serial Interface**
- Model 22: `wan1_iface=/dev/ttyUSB3`
- Model 4500: `wan1_iface=/dev/ttyUSB0`

**4. ICMP Detection Host** (Updated by three-way analysis)
- Model 22: `wan1_icmp_host=10.4.6.30` (internal)
- Model 4500: Varies by service plan

**5. Alarm Input Options**
- Model 22: Includes `switch-sim-card` option
- Model 4500: Does not include `switch-sim-card`

---

## Part 2: Three-Way Dependencies (Model + Carrier + Service Plan)

### Analysis Scope
- **Carriers:** ATT, VZW
- **Models:** 22, 4500
- **Service Plans:** tier1 (basic), ATM (premium)
- **Files:** 8 configurations covering all combinations

### Key Finding: Service Plan is a First-Class Configuration Factor

The Service Plan layer significantly affects:
- **Security features** (firewall rules, web filtering)
- **Data thresholds** (traffic monitoring limits)
- **Premium features** (QoS, advanced mode, monitoring)
- **Network configuration** (interface naming, VLAN tagging)

### Three-Way Dependent Parameters (19 total)

#### Category 1: Service Plan Features (5 parameters)

##### **fw_acl** (Firewall ACL Rules) ⭐ CRITICAL

| Service Plan | Configuration |
|--------------|---------------|
| **ATM** (all models/carriers) | **~40 whitelist entries**: Payment processors (Switch Commerce, PAI, 1st ISO, EFX), AWS resources, DNS servers, specific domains (libertyx.com, atmssl.dnsatm.com), ends with "Block all others" |
| **tier1** - Most combinations | Minimal: `1<4<0.0.0.0/0<<<<1<1<>` (no whitelist) |
| **tier1** - VZW+22 only | Additional network: `1<4<0.0.0.0/0<<0.0.0.0/0<<1<1<>` |

**Business Logic:**
```sql
IF (service_plan=ATM) THEN
    fw_acl = '~40 whitelist rules for ATM operations'
ELSE IF (model=22 AND carrier=VZW AND service_plan=tier1) THEN
    fw_acl = 'minimal + additional network segment'
ELSE
    fw_acl = 'minimal rules only'
```

**Critical Insight:** ATM service plan = Premium ATM management tier with strict security whitelist, identical across all model/carrier combinations.

##### **fw_web** (Web Filtering Rules)

|           | **tier1** | **ATM** |
|-----------|-----------|---------|
| **Model 22** | Empty | Empty |
| **Model 4500** | Empty | **5 domain rules** (libertyx.com, atmssl.dnsatm.com, RMS domains) |

```sql
IF (model=4500 AND service_plan=ATM) THEN fw_web='5 ATM domain rules' ELSE fw_web=''
```

**Observation:** Web filtering ONLY for Model 4500 + ATM service plan, not applied to Model 22 even with ATM plan.

##### **qos_iface** (QoS Interface Enable) ⭐ COMPLEX

|           | **ATT + tier1** | **ATT + ATM** | **VZW + tier1** | **VZW + ATM** |
|-----------|-----------------|---------------|-----------------|---------------|
| **Model 22**   | ❌ Disabled | ❌ Disabled | ✅ wan1 | ✅ wan1 |
| **Model 4500** | ❌ Disabled | ✅ wan1 | ❌ Disabled | ✅ wan1 |

```sql
IF (carrier=VZW AND model=22) THEN qos_iface='wan1'  -- Both service plans
ELSE IF (model=4500 AND service_plan=ATM) THEN qos_iface='wan1'  -- Both carriers
ELSE qos_iface=''
```

##### **traffic_day_threshold** + **traffic_day_unit** ⭐ CRITICAL

|           | **ATT + tier1** | **ATT + ATM** | **VZW + tier1** | **VZW + ATM** |
|-----------|-----------------|---------------|-----------------|---------------|
| **Model 22**   | 5 MB | **350 MB** | **3584 MB (3.5 GB)** | **3584 MB (3.5 GB)** |
| **Model 4500** | 5 MB | 5 MB | 5 MB | 5 MB |

```sql
IF (carrier=ATT AND model=22 AND service_plan=ATM) THEN
    threshold=350, unit=1  -- 350 MB/day
ELSE IF (carrier=VZW AND model=22) THEN
    threshold=3584, unit=1  -- 3584 MB/day (3.5 GB) for BOTH service plans
ELSE
    threshold=5, unit=2  -- Default low threshold
```

**Critical Insight:** VZW Model 22 gets 10x more data than ATT+22+ATM (3584MB vs 350MB).

#### Category 2: System Configuration (2 parameters)

##### **advanced** (Advanced Mode Enable) ⭐ UPDATED

|           | **ATT + tier1** | **ATT + ATM** | **VZW + tier1** | **VZW + ATM** |
|-----------|-----------------|---------------|-----------------|---------------|
| **Model 22**   | ❌ Disabled | ❌ Disabled | ✅ Enabled | ✅ Enabled |
| **Model 4500** | ❌ Disabled | ✅ Enabled | ❌ Disabled | ❌ Disabled |

```sql
IF (carrier=ATT AND model=4500 AND service_plan=ATM) THEN advanced=1
ELSE IF (carrier=VZW AND model=22) THEN advanced=1  -- Both service plans
ELSE advanced=0
```

**Key Update:** Two-way analysis showed VZW+22=1. Three-way reveals ATT+4500 also gets it, but ONLY with ATM service plan.

##### **console_enable** (Console Access) ⭐ COMPLEX

|           | **ATT + tier1** | **ATT + ATM** | **VZW + tier1** | **VZW + ATM** |
|-----------|-----------------|---------------|-----------------|---------------|
| **Model 22**   | ❌ Disabled | ❌ Disabled | ✅ Enabled | ❌ Disabled |
| **Model 4500** | ✅ Enabled | ✅ Enabled | ❌ Disabled | ❌ Disabled |

```sql
IF (carrier=ATT AND model=4500) THEN console_enable=1  -- Both service plans
ELSE IF (carrier=VZW AND model=22 AND service_plan=tier1) THEN console_enable=1
ELSE console_enable=0
```

**Observation:** VZW+22 console is disabled for ATM plan but enabled for tier1 (inverse of typical pattern).

#### Category 3: Network Interfaces (3 parameters)

##### **lan0_iface** (LAN Interface Name)

|           | **tier1** | **ATM** |
|-----------|-----------|---------|
| **Model 22** | eth2.1 | **lan0** |
| **Model 4500** | eth2.1 | eth2.1 |

```sql
IF (model=22 AND service_plan=ATM) THEN lan0_iface='lan0' ELSE lan0_iface='eth2.1'
```

##### **wan0_iface** (WAN Interface)

|           | **ATT + tier1** | **ATT + ATM** | **VZW + tier1** | **VZW + ATM** |
|-----------|-----------------|---------------|-----------------|---------------|
| **Model 22**   | eth2.2 | eth2.2 | eth2.2 | eth2.2 |
| **Model 4500** | eth2.2 | **eth2.4015** | eth2.2 | eth2.2 |

```sql
IF (carrier=ATT AND model=4500 AND service_plan=ATM) THEN wan0_iface='eth2.4015'
ELSE wan0_iface='eth2.2'
```

**Observation:** ATT+4500+ATM uses VLAN tagging (4015) on WAN interface.

##### **wan1_icmp_host** (ICMP Keepalive Target)

|           | **tier1** | **ATM** |
|-----------|-----------|---------|
| **Model 22** (all carriers) | 10.4.6.30 (internal) | 10.4.6.30 (internal) |
| **Model 4500** (both carriers) | **74.118.247.208** (external) | 10.4.6.30 (internal) |

```sql
IF (model=22) THEN wan1_icmp_host='10.4.6.30'  -- Always internal
ELSE IF (model=4500 AND service_plan=tier1) THEN wan1_icmp_host='74.118.247.208'  -- External
ELSE IF (model=4500 AND service_plan=ATM) THEN wan1_icmp_host='10.4.6.30'  -- Internal
```

#### Category 4: Monitoring & Alarms (2 parameters)

##### **alarm_input_options**

| Combination | Configuration |
|-------------|---------------|
| Model 22 (all) | Includes: `switch-sim-card`, `switch-backup-link` |
| Model 4500 + tier1 | Missing `switch-sim-card`, has `switch-backup-link` |
| VZW + Model 4500 + ATM | Has `traffic-month-alarm`, `traffic-month-discon` (monthly vs daily) |
| ATT + Model 4500 + ATM | No switch options |

**Pattern:** Model 4500 + ATM gets monthly traffic alarms instead of daily.

##### **alarm_output_options**

|           | **ATT + tier1** | **ATT + ATM** | **VZW + tier1** | **VZW + ATM** |
|-----------|-----------------|---------------|-----------------|---------------|
| **Model 22**   | cli,out-dm,out-rmon | cli,out-dm,out-rmon | cli,out-dm,out-rmon | cli,out-dm,out-rmon |
| **Model 4500** | cli | cli | cli,out-dm | cli |

**Pattern:** Model 22 always has full RMON output. Model 4500 outputs vary by carrier+plan.

#### Category 5: Infrastructure Services (1 parameter)

##### **ntp_server** (Time Servers)

|           | **tier1** | **ATM** |
|-----------|-----------|---------|
| **Model 22** (all carriers) | 10.4.6.30;time.nist.gov;time.google.com | 10.4.6.30;time.nist.gov;time.google.com |
| **Model 4500** (both carriers) | time.nist.gov;**74.118.247.208**;time.google.com | time.nist.gov;**10.4.6.30**;time.google.com |

**Pattern:**
- Model 22: Internal NTP (10.4.6.30) first, always
- Model 4500 + tier1: External IP (74.118.247.208) second
- Model 4500 + ATM: Internal NTP (10.4.6.30) second

#### Device-Specific Parameters (6)
These don't represent business logic - just unique identifiers:
- hostname (unique per device)
- lan0_mac, wan0_mac (hardware MAC addresses)
- ovdp_device_id (unique device ID)
- alarm_input, alarm_output (device-specific configurations)

---

## Critical Business Rules

### ATM Service Plan = Premium Tier

ATM service plan consistently receives premium features:
- ✅ **Extensive firewall whitelist** (~40 entries for payment processors, AWS, DNS)
- ✅ **Web filtering** (Model 4500 only)
- ✅ **Higher data thresholds** (ATT+22: 350MB vs 5MB baseline)
- ✅ **QoS enabled** (Model 4500 gets QoS with ATM plan)
- ✅ **Advanced mode** (ATT+4500 gets advanced mode with ATM plan)
- ✅ **Internal network resources** (ICMP host, NTP priority to internal servers)
- ✅ **Different interface naming** (Model 22 uses 'lan0' instead of 'eth2.1')
- ✅ **VLAN tagging** (ATT+4500 uses eth2.4015 with VLAN)

### tier1 Service Plan = Basic Connectivity

tier1 service plan is bare-bones connectivity:
- ❌ Minimal firewall rules (no whitelist)
- ❌ No web filtering
- ❌ Lower data thresholds (5MB default)
- ⚠️ Mixed QoS (VZW+22 only)
- ⚠️ Mixed advanced mode (VZW+22 only)
- ✅ External monitoring targets (uses external IP for ICMP)

### VZW Model 22 = High-Performance Configuration

VZW + Model 22 devices (BOTH service plans) are configured for high-bandwidth operations:
- **3584MB (3.5GB) daily data allowance** (10x more than ATT+22+ATM)
- **QoS always enabled** (regardless of service plan)
- **Advanced mode always enabled** (regardless of service plan)
- **Observation:** Suggests VZW Model 22 is used for high-bandwidth ATM operations

### ATT Model 4500 + ATM = Enterprise Configuration

ATT + Model 4500 + ATM gets unique enterprise features:
- VLAN tagging on WAN interface (eth2.4015)
- Advanced mode enabled
- Web filtering enabled
- QoS enabled
- Internal network priority (ICMP, NTP)

---

## Part 3: Four-Way Dependencies (Model + Carrier + Service Plan + Company)

### Analysis Scope
- **Carriers:** ATT, VZW
- **Models:** Model 22
- **Service Plans:** tier1, ATM (inferred from base configs)
- **Customer Files Analyzed:** 7 customer-specific configs
  - **ATT Customers:** ALTECH, BAUM, CORD, SplitFirst, VanceRMS
  - **VZW Customers:** ALTECH, CORD, VanceRMS
- **Analysis Date:** January 5, 2026

### Key Finding: Customer Overrides of Three-Way Conditional Parameters

**Critical Discovery:** 7 parameters that have three-way conditional rules (Model + Carrier + Service Plan) are ALSO being overridden at the customer level. This indicates that **Four-Way Conditional Rules are required** to support customer-specific exceptions to service plan policies.

### Four-Way Dependent Parameters (7 total)

#### **Category 1: Firewall & Security (3 parameters)**

##### 1. **fw_acl** (Firewall ACL Rules) ⚠️ CRITICAL

**Three-Way Baseline Pattern:**
- ATM service plan: ~40 whitelist entries (payment processors, AWS, DNS servers)
- tier1 service plan: Minimal/no whitelist entries

**Customer Overrides Observed:**
| Customer | Carrier | Override Reason |
|----------|---------|-----------------|
| **CORD** | ATT | Custom firewall rules beyond ATM baseline |
| **ALTECH** | VZW | Modified whitelist for specific integrations |
| **CORD** | VZW | Custom security requirements |

**Business Impact:** Service plans define baseline security policies, but some customers need additional or modified firewall rules for their specific use cases (payment gateways, cloud services, APIs).

**Four-Way Rule Example:**
```sql
-- Priority 1: Customer-specific rule (highest)
IF (model=22 AND carrier=ATT AND service_plan=ATM AND company='CORD')
  THEN fw_acl='[CORD custom ACL with extra whitelists]'

-- Priority 2: General service plan rule
ELSE IF (model=22 AND carrier=ATT AND service_plan=ATM AND company IS NULL)
  THEN fw_acl='[Default ATM 40 entries]'

-- Priority 3: tier1 baseline
ELSE IF (model=22 AND carrier=ATT AND service_plan=tier1 AND company IS NULL)
  THEN fw_acl='[Minimal ACL]'
```

##### 2. **fw_web** (Web Filtering Rules)

**Three-Way Baseline Pattern:**
- Only configured for Model 4500 + ATM service plan (both carriers)
- Model 22 typically does NOT have web filtering

**Customer Overrides Observed:**
| Customer | Carrier | Override Reason |
|----------|---------|-----------------|
| **CORD** | ATT | Enabling web filtering on Model 22 (non-standard) |

**Business Impact:** CORD customer requires web filtering capability on Model 22 devices, which is not part of the standard Model 22 + ATM configuration.

##### 3. **advanced** (Advanced Mode Enable)

**Three-Way Baseline Pattern:**
- ATT + Model 4500 + ATM: Enabled
- VZW + Model 22 (both service plans): Enabled
- All other combinations: Disabled

**Customer Overrides Observed:**
| Customer | Carrier | Override Reason |
|----------|---------|-----------------|
| **ALTECH** | VZW | Custom advanced mode configuration |
| **CORD** | VZW | Modified advanced settings |

**Business Impact:** Customers occasionally need to modify advanced mode settings beyond the standard carrier+model+plan configuration.

#### **Category 2: Network Monitoring & Thresholds (2 parameters)**

##### 4. **traffic_day_threshold** (Daily Data Threshold)

**Three-Way Baseline Pattern:**
- ATT + Model 22 + ATM: 350MB
- VZW + Model 22 (both plans): 3584MB (3.5GB)
- All others: 5MB (conservative default)

**Customer Overrides Observed:**
| Customer | Carrier | Override Reason |
|----------|---------|-----------------|
| **ALTECH** | VZW | Custom data threshold for their operations |

**Business Impact:** Service plans set baseline data allowances, but customers may need custom thresholds based on their operational requirements or cost management.

##### 5. **traffic_day_unit** (Data Threshold Unit)

**Three-Way Baseline Pattern:**
- ATT + Model 22 + ATM: unit=1
- VZW + Model 22 (both plans): unit=1
- Others: unit=2

**Customer Overrides Observed:**
| Customer | Carrier | Override Reason |
|----------|---------|-----------------|
| **ALTECH** | VZW | Custom unit setting related to data threshold |

**Business Impact:** Coupled with traffic_day_threshold; customers needing custom thresholds also customize the unit.

#### **Category 3: Infrastructure Services (2 parameters)**

##### 6. **ntp_server** (NTP Time Servers)

**Three-Way Baseline Pattern:**
- Model 4500 + tier1: External priority (time.nist.gov first, then internal 10.4.6.30)
- Model 4500 + ATM: Internal priority (10.4.6.30 first, then external)
- Model 22: Varies by carrier and service plan

**Customer Overrides Observed:**
| Customer | Carrier | Override Reason |
|----------|---------|-----------------|
| **ALTECH** | ATT | Custom NTP server order (time.nist.gov, 10.4.6.30, time.google.com) |
| **BAUM** | ATT | Modified NTP configuration |
| **CORD** | ATT | Custom time server preferences |

**Business Impact:** Some customers require specific NTP server configurations for compliance, internal infrastructure, or network design reasons.

##### 7. **alarm_output_options** (Alarm Output Destinations)

**Three-Way Baseline Pattern:**
- Model 22 + ATM/tier1 (both carriers): Full options (cli, out-dm, out-rmon)
- Model 4500 + tier1: Varies by carrier
- Model 4500 + ATM: Minimal options (cli only)

**Customer Overrides Observed:**
| Customer | Carrier | Override Reason |
|----------|---------|-----------------|
| **ALTECH** | ATT | Reduced alarm outputs (removed out-rmon) |
| **ALTECH** | VZW | Modified alarm output destinations |

**Business Impact:** Customers may want to reduce alarm verbosity or route alarms to different destinations based on their monitoring infrastructure.

---

### Customer Override Patterns Summary

#### Most Active Customers (by override count)
1. **ALTECH** (ATT & VZW): 40+ parameter overrides - Heavily customized deployment
2. **BAUM** (ATT): 25+ parameter overrides - Custom networking and scheduling
3. **CORD** (ATT & VZW): 18-20 parameter overrides - Firewall and security customizations

#### Common Customer-Level Overrides (NOT requiring four-way rules)

The following parameters are commonly overridden at the customer level but do NOT require four-way conditional rules because they have no three-way baseline logic:

**Network Configuration (~10 parameters):**
- `dhcpd_start`, `dhcpd_end`, `dhcpd_lease` - LAN IP ranges
- `fw_nat` - Port forwarding rules
- `ssl_server` - Payment processor connections
- `dns_static` - DNS servers

**Operational Preferences (~15 parameters):**
- `cron_rb_enable`, `cron_rb_time`, `cron_list` - Reboot schedules
- `sms_enable`, `sms_rb`, `sms_sq` - SMS notifications
- `mqtt_center`, `mqtt_username`, `mqtt_keepalive` - MQTT integration
- `rmon_user` - Monitoring username

**Device-Specific (~8 parameters):**
- `hostname` - Device hostname
- `ovdp_device_id` - Device ID
- `model_name`, `oem_name` - Encrypted identifiers

**Recommendation:** These ~35 parameters should remain at Company/Device layers ONLY. They are inherently customer-specific and should NOT use conditional rules framework.

---

### Four-Way Rule Definition

**Rule Structure:** Model + Carrier + Service Plan + Company (nullable)

**Rule Resolution Logic:**
1. **When `company_id IS NULL`**: Rule applies to ALL customers with that Model + Carrier + ServicePlan combination (general baseline)
2. **When `company_id IS SET`**: Rule applies ONLY to that specific customer (customer-specific exception to baseline)

**Priority Hierarchy:**
```
Priority 1 (Highest): Four-Way Rule with company_id SET (customer exception)
Priority 2:           Company layer value (customer portfolio-wide)
Priority 3:           Four-Way Rule with company_id NULL (general baseline) = Three-Way Rule
Priority 4:           Service Plan layer
Priority 5:           Two-Way Rule (Model + Carrier)
Priority 6-10:        Standard layer hierarchy (Carrier → Model → Global → Schema default)
```

**Example Resolution for `fw_acl` parameter:**

```
Device: company_id=123 (CORD), model_id=22, carrier_id=ATT, service_plan_id=ATM

Step 1: Check four-way rule with company
  → FOUND: (model=22, carrier=ATT, plan=ATM, company=CORD) → [CORD custom ACL]
  → RETURN custom ACL

Alternative path if no company-specific rule:
Step 1: No four-way rule with company → Continue
Step 2: Check company layer → Not set → Continue
Step 3: Check four-way rule without company (three-way baseline)
  → FOUND: (model=22, carrier=ATT, plan=ATM, company=NULL) → [ATM default 40 entries]
  → RETURN default ACL
```

---

### Implementation Impact

#### Database Schema Addition

New table required: `config_conditional_rules_4way`

```sql
CREATE TABLE config_conditional_rules_4way (
    id INT PRIMARY KEY AUTO_INCREMENT,
    config_key_id INT NOT NULL,
    model_id INT NOT NULL,
    carrier_id INT NOT NULL,
    service_plan_id INT NOT NULL,
    company_id INT NULL,  -- NULL = general rule, SET = customer-specific
    value TEXT,
    priority INT NOT NULL DEFAULT 100,
    description TEXT,
    created_by_user_id INT,
    updated_by_user_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (config_key_id) REFERENCES config_keys(id),
    FOREIGN KEY (model_id) REFERENCES models(id),
    FOREIGN KEY (carrier_id) REFERENCES carriers(id),
    FOREIGN KEY (service_plan_id) REFERENCES service_plans(id),
    FOREIGN KEY (company_id) REFERENCES companies(id),

    UNIQUE KEY uk_4way_rule (config_key_id, model_id, carrier_id, service_plan_id, company_id),
    INDEX idx_model_carrier_plan_company (model_id, carrier_id, service_plan_id, company_id)
);
```

#### Config Keys Schema Update

```sql
ALTER TABLE config_keys
ADD COLUMN has_four_way_dependency BOOLEAN DEFAULT FALSE;
```

Flag these 7 parameters: `fw_acl`, `fw_web`, `advanced`, `ntp_server`, `alarm_output_options`, `traffic_day_threshold`, `traffic_day_unit`

---

### Business Logic Summary

| Parameter | Three-Way Baseline | Customer Exception Use Case | Four-Way Required? |
|-----------|-------------------|----------------------------|-------------------|
| **fw_acl** | ATM=40 rules, tier1=minimal | Custom security requirements | ✅ YES |
| **fw_web** | Model 4500+ATM only | Enable on Model 22 for specific customer | ✅ YES |
| **advanced** | Specific combos enabled | Customer-specific advanced settings | ✅ YES |
| **traffic_day_threshold** | Plan-based allowances | Custom data limits per customer | ✅ YES |
| **traffic_day_unit** | Plan-based units | Coupled with threshold | ✅ YES |
| **ntp_server** | Plan-based priority | Custom NTP for compliance/infrastructure | ✅ YES |
| **alarm_output_options** | Model+Plan-based | Reduce verbosity per customer | ✅ YES |

---

## Implementation Requirements

### 1. Database Schema

#### Config Keys Table (Updated)
```sql
ALTER TABLE config_keys
ADD COLUMN has_model_carrier_dependency BOOLEAN DEFAULT FALSE,
ADD COLUMN has_three_way_dependency BOOLEAN DEFAULT FALSE,
ADD COLUMN has_four_way_dependency BOOLEAN DEFAULT FALSE,
ADD INDEX idx_two_way (has_model_carrier_dependency),
ADD INDEX idx_three_way (has_three_way_dependency),
ADD INDEX idx_four_way (has_four_way_dependency);
```

#### Two-Way Conditional Rules Table
```sql
CREATE TABLE config_model_carrier_rules (
    id INT PRIMARY KEY AUTO_INCREMENT,
    config_key_id INT NOT NULL,  -- FK to config_keys
    model_id INT NOT NULL,        -- FK to models
    carrier_id INT NOT NULL,      -- FK to carriers
    value TEXT,
    priority INT DEFAULT 100,
    created_by_user_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_key_model_carrier (config_key_id, model_id, carrier_id),
    INDEX idx_model_carrier (model_id, carrier_id),
    INDEX idx_config_key (config_key_id)
);
```

#### Three-Way Conditional Rules Table
```sql
CREATE TABLE config_model_carrier_plan_rules (
    id INT PRIMARY KEY AUTO_INCREMENT,
    config_key_id INT NOT NULL,     -- FK to config_keys
    model_id INT NOT NULL,           -- FK to models
    carrier_id INT NOT NULL,         -- FK to carriers
    service_plan_id INT NOT NULL,    -- FK to service_plans
    value TEXT,
    priority INT DEFAULT 100,
    created_by_user_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_key_model_carrier_plan (config_key_id, model_id, carrier_id, service_plan_id),
    INDEX idx_model_carrier_plan (model_id, carrier_id, service_plan_id),
    INDEX idx_config_key (config_key_id)
);
```

#### Four-Way Conditional Rules Table
```sql
CREATE TABLE config_conditional_rules_4way (
    id INT PRIMARY KEY AUTO_INCREMENT,
    config_key_id INT NOT NULL,      -- FK to config_keys
    model_id INT NOT NULL,            -- FK to models
    carrier_id INT NOT NULL,          -- FK to carriers
    service_plan_id INT NOT NULL,     -- FK to service_plans
    company_id INT NULL,              -- FK to companies (NULL = general rule, SET = customer-specific)
    value TEXT,
    priority INT DEFAULT 100,
    description TEXT,                 -- Business logic explanation for this rule
    created_by_user_id INT,
    updated_by_user_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (config_key_id) REFERENCES config_keys(id),
    FOREIGN KEY (model_id) REFERENCES models(id),
    FOREIGN KEY (carrier_id) REFERENCES carriers(id),
    FOREIGN KEY (service_plan_id) REFERENCES service_plans(id),
    FOREIGN KEY (company_id) REFERENCES companies(id),
    UNIQUE KEY uk_4way_rule (config_key_id, model_id, carrier_id, service_plan_id, company_id),
    INDEX idx_model_carrier_plan_company (model_id, carrier_id, service_plan_id, company_id),
    INDEX idx_config_key (config_key_id),
    INDEX idx_company (company_id)
);
```

**Key Design Decisions:**
- `company_id` is NULLABLE to support both general rules (NULL) and customer-specific exceptions (SET)
- When `company_id IS NULL`: Rule applies to ALL customers with matching Model+Carrier+ServicePlan
- When `company_id IS SET`: Rule applies ONLY to that specific customer (overrides general rule)
- Unique constraint includes company_id to prevent duplicate rules
- Priority field allows fine-grained control when multiple rules could match

### 2. Configuration Resolution Algorithm

```python
def resolve_config_value(device, config_key):
    """
    Resolve configuration value with full priority hierarchy including four-way rules
    """
    # Priority 1: Device Override
    if has_device_override(device.id, config_key.id):
        return get_device_value(device.id, config_key.id)

    # Priority 2: Four-Way Rule with Company (customer-specific exception)
    if config_key.has_four_way_dependency:
        rule = find_four_way_rule(
            config_key.id,
            device.model_id,
            device.carrier_id,
            device.service_plan_id,
            device.company_id  # Specific company
        )
        if rule:
            return rule.value

    # Priority 3: Company Override (portfolio-wide setting)
    if has_company_override(device.company_id, config_key.id):
        return get_company_value(device.company_id, config_key.id)

    # Priority 4: Four-Way Rule without Company (general baseline) = Three-Way Rule
    if config_key.has_four_way_dependency or config_key.has_three_way_dependency:
        rule = find_four_way_rule(
            config_key.id,
            device.model_id,
            device.carrier_id,
            device.service_plan_id,
            company_id=None  # General rule for all customers
        )
        if rule:
            return rule.value

    # Priority 5: Service Plan Layer
    if has_service_plan_value(device.service_plan_id, config_key.id):
        return get_service_plan_value(device.service_plan_id, config_key.id)

    # Priority 6: Two-Way Rule (Model + Carrier)
    if config_key.has_model_carrier_dependency:
        rule = find_two_way_rule(
            config_key.id,
            device.model_id,
            device.carrier_id
        )
        if rule:
            return rule.value

    # Priority 7-10: Standard Layer Resolution
    # Carrier Layer
    if has_carrier_value(device.carrier_id, config_key.id):
        return get_carrier_value(device.carrier_id, config_key.id)

    # Model Layer
    if has_model_value(device.model_id, config_key.id):
        return get_model_value(device.model_id, config_key.id)

    # Global Layer
    if has_global_value(config_key.id):
        return get_global_value(config_key.id)

    # Priority 11: Default Value
    if config_key.default_value is not None:
        return config_key.default_value

    # Priority 12: Required Key Validation
    if config_key.is_required:
        raise ConfigError(f"Required key {config_key.name} has no value for device {device.id}")

    return None
```

### 3. Configuration Resolution Priority

```
1. Device Override (highest priority)
2. Four-Way Rule with Company (Model + Carrier + Service Plan + Specific Company)
3. Company Override (portfolio-wide setting)
4. Four-Way Rule without Company / Three-Way Rule (Model + Carrier + Service Plan + NULL)
5. Service Plan Layer
6. Two-Way Rule (Model + Carrier)
7. Carrier Layer
8. Model Layer
9. Global Layer
10. Default Value from Schema
11. Required Key Validation (error if missing)
```

**Key Changes:**
- Four-way rules split into two priority levels: customer-specific (Priority 2) and general baseline (Priority 4)
- Company layer moved to Priority 3 (between customer-specific rules and general rules)
- Total priority levels increased from 10 to 11

### 4. Admin UI Requirements

#### Two-Way Rule Management Interface
- Select config key
- Define Model + Carrier combination
- Set value for that combination
- View matrix of all model/carrier combinations
- Validate no conflicting rules

#### Three-Way Rule Management Interface
- Select config key
- Define Model + Carrier + Service Plan combination (3D matrix)
- Set value for that specific combination
- View comparison table (like matrices above)
- Validate no conflicting rules
- Show which combinations have rules vs inherit from layers

#### Four-Way Rule Management Interface
- Select config key (only keys with has_four_way_dependency flag)
- Define Model + Carrier + Service Plan + Company combination
- **Company field is nullable:**
  - **NULL (default)**: General baseline rule for all customers
  - **Select specific company**: Customer-specific exception to baseline
- Set value for that combination
- View hierarchical table:
  - Show general baseline rules (company=NULL)
  - Show customer-specific exceptions (company=SET) nested under baseline
- Visual indicator: "3 customer exceptions to this baseline rule"
- Validate no conflicting rules
- **Priority visualization**: Show which rule wins for each company
- **Coverage report**: "fw_acl has baseline + 3 customer exceptions"

**Key UI Features:**
- Toggle view: "Show only general rules" vs "Show all (including customer exceptions)"
- Bulk create: "Apply this baseline to Model 22 + ATT + ATM (all customers)"
- Exception builder: "Create exception for CORD customer on existing baseline"
- Conflict warnings: "CORD already has portfolio-wide Company layer value - four-way rule will override it"

#### Service Plan Configuration Layer
- Full layer editor (same capabilities as Model/Carrier layers)
- Show inherited values from Model/Carrier/Global
- Allow overrides specific to service plan
- Preview affected devices by service plan

### 5. Validation Rules

**Conflict Detection:**
- Cannot have both two-way and three-way rules for same key/model/carrier
- Three-way rule takes precedence over two-way
- Four-way rule with company_id=NULL is equivalent to three-way rule (same priority)
- Warn admin if adding three-way rule when two-way exists
- Warn admin if customer has both Company layer value AND four-way customer-specific rule (four-way wins)

**Completeness Checking:**
- For parameters flagged as requiring conditional logic, validate all necessary combinations have rules
- Show coverage report: "qos_iface has rules for 6/8 combinations (75% coverage)"
- For four-way rules: "fw_acl has 8 baseline rules + 3 customer exceptions"

**Four-Way Rule Validation:**
- Cannot create duplicate rules (same config_key + model + carrier + plan + company combination)
- Customer-specific rule (company_id SET) requires matching baseline rule (company_id NULL)
- Warn if creating customer exception without baseline: "No baseline rule exists for Model 22 + ATT + ATM"

---

## Complete Parameter Lists

### Two-Way Dependent Parameters (22)

**Meaningful Business Logic (17):**
1. advanced (updated to three-way)
2. alarm_output_options (also three-way)
3. console_enable (updated to three-way)
4. dns_static
5. fw_acl (updated to three-way)
6. mqtt_enable
7. ntp_server (updated to three-way)
8. qos_iface (updated to three-way)
9. sms_enable
10. sms_rb
11. sms_sq
12. ssl_server
13. traffic_day_threshold (updated to three-way)
14. traffic_day_unit (updated to three-way)
15. wan1_icmp_interval
16. wan1_mtu
17. wan3_mtu

**Device Identifiers (5):**
18. hostname
19. lan0_mac
20. wan0_mac
21. ovdp_device_id
22. oem_name

### Three-Way Dependent Parameters (19)

**Meaningful Business Logic (13):**
1. advanced ⭐ (System Config)
2. alarm_input_options (Monitoring)
3. alarm_output_options (Monitoring)
4. console_enable ⭐ (System Config)
5. fw_acl ⭐ CRITICAL (Security)
6. fw_web (Security)
7. lan0_iface (Network)
8. ntp_server (Infrastructure)
9. qos_iface ⭐ COMPLEX (Service Plan Features)
10. traffic_day_threshold ⭐ CRITICAL (Service Plan Features)
11. traffic_day_unit (Service Plan Features)
12. wan0_iface (Network)
13. wan1_icmp_host (Network)

**Device Identifiers (6):**
14. alarm_input
15. alarm_output
16. hostname
17. lan0_mac
18. ovdp_device_id
19. wan0_mac

### Parameters with BOTH Two-Way and Three-Way Dependencies

Some parameters appear in both lists because they have different requirements:
- **Two-way baseline:** Works for basic model/carrier combinations
- **Three-way refinement:** Service plan adds additional complexity

**Examples:**
- `advanced`: Two-way shows VZW+22=1. Three-way reveals ATT+4500+ATM also=1
- `qos_iface`: Two-way shows Model22+DC/VZW=wan1. Three-way shows Model4500+ATM also=wan1
- `traffic_day_threshold`: Two-way shows VZW+22=3584. Three-way shows ATT+22+ATM=350

**Implementation:** Use three-way rules when available, fall back to two-way, then standard layers.

---

## Summary Statistics

### Coverage Analysis

| Metric | Count | Percentage |
|--------|-------|------------|
| **Total common parameters** | 622-624 | 100% |
| **Consistent (no conditions)** | 586 | 94.2% |
| **Require conditional logic** | 36 | 5.8% |
| └─ Two-way (Model + Carrier) | 22 | 3.5% |
| └─ Three-way (Model + Carrier + Plan) | 19 | 3.1% |
| └─ Four-way (Model + Carrier + Plan + Company) | 7 | 1.1% |
| └─ Carrier-only | 8 | 1.3% |
| └─ Model-only | 8 | 1.3% |

**Note:** 7 four-way parameters are a subset of the 19 three-way parameters (they need BOTH general baseline AND customer exceptions)

### Dependency Complexity

| Dependency Level | Parameters | Implementation Complexity |
|-----------------|------------|---------------------------|
| **No dependency** | 586 | Simple (single value) |
| **Single factor** | 16 | Medium (if-then based on one attribute) |
| **Two factors** | 22 | Complex (matrix lookup model×carrier) |
| **Three factors** | 19 | Very Complex (3D matrix model×carrier×plan) |
| **Four factors** | 7 | Most Complex (4D matrix with nullable company dimension) |

**Key Insights:**
- 94% of parameters are straightforward (no conditional logic)
- Only 6% require conditional logic
- Four-way rules enable customer-specific exceptions to service plan policies
- Four-way parameters support BOTH general baseline (company=NULL) AND customer exceptions (company=SET)

---

## Migration Path for Existing Configs

### Phase 1: Implement Two-Way Support
1. Add `has_model_carrier_dependency` flag to config_keys
2. Create `config_model_carrier_rules` table
3. Import existing 22 two-way parameters as rules
4. Update resolution engine to check two-way rules
5. Test with subset of devices

### Phase 2: Add Service Plan Layer
1. Add Service Plan as full configuration layer
2. Add service-plan specific overrides (non-conditional)
3. Test service plan layer with standard resolution

### Phase 3: Implement Three-Way Support
1. Add `has_three_way_dependency` flag to config_keys
2. Create `config_model_carrier_plan_rules` table
3. Import existing 19 three-way parameters as rules
4. Update resolution engine to check three-way rules first
5. Migrate devices to three-way logic incrementally

### Phase 4: Implement Four-Way Support
1. Add `has_four_way_dependency` flag to config_keys
2. Create `config_conditional_rules_4way` table with nullable company_id
3. Import existing customer-specific config overrides as four-way rules
4. Update resolution engine to check four-way rules (customer-specific first, then general)
5. Migrate 7 parameters to four-way logic
6. Test with customers that have known exceptions (ALTECH, BAUM, CORD)

### Phase 5: Admin UI Enhancement
1. Build two-way rule management interface
2. Build three-way rule management interface
3. Build four-way rule management interface (with nullable company field)
4. Add service plan layer configuration UI
5. Add visual comparison matrices for rule validation
6. Add customer exception builder tool

---

## Appendix A: Parameter Quick Reference

### Parameters Requiring Conditional Rules

| Parameter | Type | Primary Driver | Customer Exceptions | Notes |
|-----------|------|----------------|-------------------|-------|
| advanced | 3-way + 4-way ⚠️ | Model+Carrier+Plan | ALTECH, CORD | VZW+22 always, ATT+4500+ATM only |
| alarm_input_options | 3-way | Model+Plan | - | Monthly vs daily alarms |
| alarm_output_options | Both + 4-way ⚠️ | Model+Carrier | ALTECH (both carriers) | RMON on Model 22 only |
| console_enable | 3-way | Model+Carrier+Plan | - | Complex inverse pattern |
| dns_static | 2-way | Model+Carrier | - | DNS server selection |
| fw_acl | 3-way + 4-way ⭐⚠️ | **Service Plan** | CORD, ALTECH | ATM=40 rules, tier1=minimal |
| fw_web | 3-way + 4-way ⚠️ | Model+Plan | CORD | Model 4500+ATM only |
| lan0_iface | 3-way | Model+Plan | - | Model 22+ATM='lan0' |
| mqtt_enable | 2-way | Model+Carrier | - | Model 22+DC/VZW only |
| ntp_server | 3-way + 4-way ⚠️ | Model+Plan | ALTECH, BAUM, CORD | Server priority differs |
| qos_iface | 3-way ⭐ | Model+Carrier+Plan | - | VZW+22 or Model4500+ATM |
| sms_enable | 2-way | Model+Carrier | - | Model 22+ATT/VZW only |
| ssl_server | 2-way | Model+Carrier | - | Different endpoints |
| traffic_day_threshold | 3-way + 4-way ⭐⚠️ | Model+Carrier+Plan | ALTECH | VZW+22=3.5GB, ATT+22+ATM=350MB |
| traffic_day_unit | 3-way + 4-way ⚠️ | Model+Carrier+Plan | ALTECH | Related to threshold |
| wan0_iface | 3-way | Model+Carrier+Plan | - | ATT+4500+ATM VLAN tagging |
| wan1_icmp_host | 3-way | Model+Plan | - | Internal vs external |
| wan1_icmp_interval | 2-way | Model+Carrier | - | Timing variance |
| wan1_mtu | 2-way | Model+Carrier | - | 1428 vs 1500 bytes |

**Legend:**
- ⭐ = Critical parameter with significant business impact
- ⚠️ = Requires four-way rule support (customer-specific exceptions exist)
- **Customer Exceptions column**: Lists customers with known overrides requiring four-way rules

---

## Appendix B: Testing Matrix

For complete validation, test all combinations:

### Two-Way Testing Matrix (Model × Carrier)
- Model 22 + ATT
- Model 22 + DC
- Model 22 + VZW
- Model 4500 + VZW
- Model 4500 + DC
- Model 4500 + ATT

### Three-Way Testing Matrix (Model × Carrier × Service Plan)
- Model 22 + ATT + tier1
- Model 22 + ATT + ATM
- Model 22 + VZW + tier1
- Model 22 + VZW + ATM
- Model 4500 + ATT + tier1
- Model 4500 + ATT + ATM
- Model 4500 + VZW + tier1
- Model 4500 + VZW + ATM

**Total test cases required:** 8 combinations × 19 three-way parameters = 152 test cases

### Four-Way Testing Matrix (Model × Carrier × Service Plan × Company)

**General Baseline Rules (company_id = NULL):**
- Test same 8 combinations as three-way matrix
- Verify baseline values apply to all customers

**Customer-Specific Exception Rules (company_id = SET):**

Test with known customer exceptions:

| Parameter | Baseline Combo | Customer | Expected Behavior |
|-----------|---------------|----------|-------------------|
| **fw_acl** | Model 22 + ATT + ATM + NULL | CORD | Customer exception overrides baseline |
| **fw_acl** | Model 22 + VZW + ATM + NULL | ALTECH | Customer exception overrides baseline |
| **fw_acl** | Model 22 + VZW + ATM + NULL | CORD | Customer exception overrides baseline |
| **fw_web** | Model 22 + ATT + ATM + NULL | CORD | Customer exception adds web filtering |
| **advanced** | Model 22 + VZW + tier1 + NULL | ALTECH | Customer exception modifies advanced mode |
| **advanced** | Model 22 + VZW + ATM + NULL | CORD | Customer exception modifies advanced mode |
| **ntp_server** | Model 22 + ATT + ATM + NULL | ALTECH | Customer custom NTP order |
| **ntp_server** | Model 22 + ATT + ATM + NULL | BAUM | Customer custom NTP order |
| **ntp_server** | Model 22 + ATT + ATM + NULL | CORD | Customer custom NTP order |
| **alarm_output_options** | Model 22 + ATT + ATM + NULL | ALTECH | Customer reduced alarm outputs |
| **alarm_output_options** | Model 22 + VZW + ATM + NULL | ALTECH | Customer reduced alarm outputs |
| **traffic_day_threshold** | Model 22 + VZW + tier1 + NULL | ALTECH | Customer custom data threshold |
| **traffic_day_unit** | Model 22 + VZW + tier1 + NULL | ALTECH | Customer custom unit setting |

**Priority Resolution Testing:**
For each four-way parameter, test:
1. Device with no exception → Uses general baseline (company_id=NULL)
2. Device with customer exception → Uses customer-specific rule (company_id=SET)
3. Device with Company layer value → Four-way rule wins over Company layer
4. Device with no rules → Falls back to Service Plan → Carrier → Model → Global layers

**Total four-way test cases:** 7 parameters × (1 baseline + 3 avg exceptions per param) = ~28 test cases

**Grand total test cases:** 152 (three-way) + 28 (four-way) = **180 test cases**

---

END OF DOCUMENT
