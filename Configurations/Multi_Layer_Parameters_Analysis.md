# Multi-Layer Configuration Parameters Analysis

**Date:** January 12, 2026
**Source:** ConfigsClientAnalisys directory
**Purpose:** Identify configuration parameters that can be set at multiple layers in the hierarchy

---

## Executive Summary

Analysis of actual client configurations reveals **significant multi-layer parameter usage**. Many parameters have:
- **Base values** set at Global/Model/Carrier layers
- **Service Plan overrides** for different service tiers
- **Customer exceptions** for specific client needs
- **Device-specific overrides** for individual device requirements

This validates the need for both:
1. **Layer-based inheritance** (FR-2) - Base values cascade down
2. **Conditional Rules Framework** (FR-3a) - Multi-factor conditional logic for complex scenarios

---

## Category 1: Firewall & Security Parameters

### fw_acl (Firewall Access Control Lists)
**Layers:** Service Plan → Customer → Device

**Usage Pattern:**
```
Service Plan Layer (ATM Plan):
- Baseline: 40 whitelist rules for payment processors, DNS, AWS
- Format: 1<4<192.168.1.0/24<<IP<<1<1<Description
- Final rule: Block all other traffic (2<1<Block)

Customer Layer (CORD exception):
- 50 custom whitelist rules
- Adds customer-specific servers:
  - 13.67.184.126 (CORD RMS)
  - 64.183.178.180 (CORD RMS Old)
  - 10.4.0.31, 10.4.0.32 (Internal servers)
  - Additional SC ranges
- Still blocks all other traffic

Non-ATM Plans:
- Single rule: Allow all (1<4<0.0.0.0/0<<0.0.0.0/0<<1<1<)
```

**Conditional Logic Required:**
- **Three-Way Rule**: Model + Carrier + Service Plan = ATM baseline (40 rules)
- **Four-Way Rule**: Model + Carrier + Service Plan + Company = CORD exception (50 rules)

**Layer Availability:**
- ❌ Global: No
- ❌ Model: No
- ❌ Carrier: No
- ✅ Service Plan: Yes (baseline for all customers)
- ✅ Company: Yes (customer exceptions)
- ✅ Device: Yes (rare device-specific overrides)

**Customer Configurable:** ❌ No (admin-only due to security impact)

---

### fw_nat (NAT Rules)
**Layers:** Customer → Device

**Usage Pattern:**
```
Customer Layer (certain customers):
- Port forwarding: 1<1<1<0.0.0.0/0<<0.0.0.0/0<18458<<10.4.0.32<18458<1<
- Traffic manipulation for optimization
- Customer-specific NAT configurations

Device Layer:
- Device-specific port forwarding for special cases
```

**Layer Availability:**
- ❌ Global: No
- ❌ Model: No
- ❌ Carrier: No
- ❌ Service Plan: No
- ✅ Company: Yes (customer-specific NAT)
- ✅ Device: Yes (device-specific NAT)

**Customer Configurable:** ❌ No (requires technical expertise)

---

## Category 2: Device Manager (MQTT) Parameters

### mqtt_enable, mqtt_center, mqtt_* (Device Manager Settings)
**Layers:** Model → Carrier

**Usage Pattern:**
```
Model Layer:
- I-22/I-52: Device Manager available
- IR611/IR615: Device Manager NOT available

Carrier Layer:
- Verizon I-22/I-52: mqtt_enable=1 (enabled)
- T-Mobile I-22/I-52: mqtt_enable=1 (enabled)
- AT&T I-22/I-52: mqtt_enable=0 (disabled)
- IR611/IR615: No MQTT parameters (not applicable)
```

**Conditional Logic Required:**
- **Two-Way Rule**: Model + Carrier
  - i-22 + VZW → mqtt_enable=1
  - i-22 + TMO → mqtt_enable=1
  - i-22 + ATT → mqtt_enable=0 (disabled)
  - IR611 + * → (parameter not applicable)

**Related Parameters:**
- mqtt_center
- mqtt_device_info
- mqtt_keepalive
- mqtt_lbs_interval
- mqtt_series_interval
- mqtt_sniffer_enable
- mqtt_sniffer_filter_enable
- mqtt_tls
- mqtt_username
- mqtt_atm_id
- mqtt_experience_confirmed
- mqtt_experience_mode

**Layer Availability:**
- ❌ Global: No
- ✅ Model: Yes (determines if feature exists)
- ✅ Carrier: Yes (determines if enabled per carrier)
- ❌ Service Plan: No
- ❌ Company: No
- ❌ Device: Possibly (override to disable for specific device)

**Customer Configurable:** ❌ No (infrastructure setting)

---

## Category 3: Network Interface Parameters

### lan0_ip, lan0_gateway, lan0_netmask, lan0_iface
**Layers:** Model → Customer/Device

**Usage Pattern:**
```
Model Layer:
- I-22/I-52: lan0_ip=192.168.1.90, lan0_iface=eth2.1
- IR611/IR615: May use different interface names

Customer Layer:
- Custom LAN subnets for specific customers
- Example: Miele may use 192.168.2.x range

Device Layer:
- Individual device IP assignments
- Special networking requirements
```

**Layer Availability:**
- ✅ Global: Yes (default fallback)
- ✅ Model: Yes (model-specific defaults)
- ❌ Carrier: No
- ❌ Service Plan: No
- ✅ Company: Yes (customer network schemes)
- ✅ Device: Yes (device-specific IPs)

**Customer Configurable:** ⚠️ Possibly (with restrictions)

**Notes:**
- CSV indicates: "Modifications Available for Customer/Device"
- May require validation to prevent conflicts

---

### lan_port1, lan_port2 (LAN Port Enable)
**Layers:** Model → Device

**Usage Pattern:**
```
Model Layer:
- I-22: 4-port switch
- IR615: 5-port switch
- Values may differ by model

Device Layer:
- Enable/disable specific ports
- Customer may need ports disabled for security
```

**Layer Availability:**
- ❌ Global: No
- ✅ Model: Yes (model capabilities)
- ❌ Carrier: No
- ❌ Service Plan: No
- ❌ Company: Possibly
- ✅ Device: Yes (per-device port control)

**Customer Configurable:** ⚠️ Possibly

---

## Category 4: DHCP Server Parameters

### dhcpd_start, dhcpd_end (DHCP IP Range)
**Layers:** Global → Customer → Device

**Usage Pattern:**
```
Global Layer:
- Default: dhcpd_start=192.168.1.100
- Default: dhcpd_end=192.168.1.254

Customer Layer:
- Altech: Custom range
- Miele: Custom range
- Adjusts for customer network architecture

Device Layer:
- Rare: Device-specific range adjustments
```

**Layer Availability:**
- ✅ Global: Yes (default range)
- ❌ Model: No
- ❌ Carrier: No
- ❌ Service Plan: No
- ✅ Company: Yes (customer network schemes)
- ✅ Device: Yes (device-specific adjustments)

**Customer Configurable:** ⚠️ Advanced customers only

**Notes:**
- CSV indicates: "Customer or Device Override Possible"
- Examples: Altech, Miele

---

### dhcpd_static (DHCP Static Reservations)
**Layers:** Device-specific

**Usage Pattern:**
```
Device Layer:
- IP-MAC binding for connected equipment
- Example: Miele ATMs need static IPs for printers/terminals
- Format: MAC:IP pairs
```

**Layer Availability:**
- ❌ Global: No
- ❌ Model: No
- ❌ Carrier: No
- ❌ Service Plan: No
- ⚠️ Company: Possibly (template for all company devices)
- ✅ Device: Yes (primary use case)

**Customer Configurable:** ✅ Yes (with MAC validation)

**Notes:**
- CSV indicates: "Device level override possible - IP MAC LOCK"
- Example customer: Miele

---

### dhcpd_lease (DHCP Lease Time)
**Layers:** Global → Model

**Usage Pattern:**
```
Global Layer:
- Default: 60 minutes

Model Layer:
- May vary by model capabilities
```

**Layer Availability:**
- ✅ Global: Yes (default)
- ⚠️ Model: Possibly (model-specific optimization)
- ❌ Carrier: No
- ❌ Service Plan: No
- ❌ Company: No
- ❌ Device: No

**Customer Configurable:** ❌ No

---

## Category 5: Traffic Management Parameters

### traffic_day_threshold, traffic_month_threshold
**Layers:** Service Plan → Customer

**Usage Pattern:**
```
Service Plan Layer:
- ATM Basic: 350MB daily threshold
- ATM Premium: Higher threshold
- Tier 1: Different limits
- Tier 2: Different limits

Customer Layer:
- Custom data allowances for specific customers
- VIP customers may get higher limits
```

**Layer Availability:**
- ❌ Global: No
- ❌ Model: No
- ❌ Carrier: No
- ✅ Service Plan: Yes (plan defines limits)
- ✅ Company: Yes (customer-specific adjustments)
- ⚠️ Device: Possibly (rare device-specific limits)

**Customer Configurable:** ❌ No (billing-related)

**Related Parameters:**
- traffic_custom_actions
- traffic_custom_enable
- traffic_custom_rule
- traffic_day_action
- traffic_day_unit
- traffic_enable
- traffic_exceed_report
- traffic_month_action
- traffic_month_alarm
- traffic_month_discon
- traffic_month_start_day
- traffic_month_unit
- traffic_month2_action
- traffic_month2_threshold
- traffic_month2_unit
- traffic_sms_up

**Notes:**
- CSV indicates: "Different Service plans can be designated different daily or monthly data aggregates"

---

## Category 6: Carrier/APN Parameters

### wan1_ppp_apn (Access Point Name)
**Layers:** Carrier

**Usage Pattern:**
```
Carrier Layer:
- Verizon: wan1_ppp_apn=Matrxatm.gw12.vzwentp
- AT&T: wan1_ppp_apn=matrix.com.attz
- T-Mobile: wan1_ppp_apn=simpl.cc.static
```

**Conditional Logic Required:**
- **Simple Carrier Lookup** (not conditional rule)
- Carrier → APN mapping table

**Layer Availability:**
- ❌ Global: No
- ❌ Model: No
- ✅ Carrier: Yes (carrier-specific)
- ❌ Service Plan: No
- ❌ Company: No
- ❌ Device: Possibly (override for special SIMs)

**Customer Configurable:** ❌ No (carrier infrastructure)

---

### wan1_ppp_redial_interval (Redial Interval)
**Layers:** Carrier → Dynamic

**Usage Pattern:**
```
Carrier Layer:
- May vary by carrier network behavior
- Dynamic adjustment based on network conditions
```

**Layer Availability:**
- ✅ Global: Yes (default fallback)
- ❌ Model: No
- ✅ Carrier: Yes (carrier-specific optimization)
- ❌ Service Plan: No
- ❌ Company: No
- ❌ Device: No

**Customer Configurable:** ❌ No

---

### sim1_card_operator, sim2_card_operator
**Layers:** Device/Carrier

**Usage Pattern:**
```
Device Layer:
- Links to profile table
- sim1_card_operator=2 (Verizon)
- sim2_card_operator=1 (AT&T)

Carrier Layer:
- Profile table values:
  - 1 = AT&T
  - 2 = Verizon
  - etc.
```

**Layer Availability:**
- ❌ Global: No
- ❌ Model: No
- ✅ Carrier: Yes (defines profile IDs)
- ❌ Service Plan: No
- ❌ Company: No
- ✅ Device: Yes (which SIM in which slot)

**Customer Configurable:** ❌ No (infrastructure)

**Notes:**
- CSV indicates: "PROFILE TABLE VALUE"

---

## Category 7: Dual SIM Parameters

### dual_sim_enable, dual_sim_main, dual_sim_*
**Layers:** Carrier

**Usage Pattern:**
```
Carrier Layer (Dual SIM Enabled Devices):
- dual_sim_enable=1
- dual_sim_main=0 (SIM1 primary)
- dual_sim_min_conn_time=120
- Carrier failover logic

Carrier Layer (Non-Dual SIM Devices):
- dual_sim_enable=0
- dual_sim_min_conn_time=0
- Parameters exist but disabled
```

**Conditional Logic Required:**
- **Two-Way Rule** potentially: Model + Carrier
  - Some carriers may enable dual SIM, others may not
  - Model determines if dual SIM hardware exists

**Layer Availability:**
- ❌ Global: No
- ⚠️ Model: Possibly (hardware capability check)
- ✅ Carrier: Yes (carrier policy on dual SIM)
- ❌ Service Plan: No
- ❌ Company: No
- ❌ Device: Possibly (override to disable)

**Customer Configurable:** ❌ No

**Related Parameters:**
- backup_sim_policy_enable
- backup_sim_policy_revert_day
- backup_sim_policy_using_time
- dual_sim_csq_retry
- dual_sim_max_retry
- dual_sim_min_conn_time
- dual_sim_min_csq

**Notes:**
- CSV shows two configurations: "DUAL SIM ENABLED DEVICES" vs "Non Duel SIM ENABLED DEVICES"

---

## Category 8: Scheduler Parameters

### cron_rb_days, cron_rb_enable, cron_rb_time (Scheduled Reboot)
**Layers:** Global → Device

**Usage Pattern:**
```
Global Layer:
- Default: cron_rb_enable=0 (disabled)
- Default: cron_rb_time=225 (2:25 AM if enabled)
- Default: cron_rb_days=0 (daily if enabled)

Device Layer:
- Enable scheduled reboots for problematic devices
- Custom reboot schedules
- Maintenance windows
```

**Layer Availability:**
- ✅ Global: Yes (default disabled)
- ❌ Model: No
- ❌ Carrier: No
- ❌ Service Plan: No
- ⚠️ Company: Possibly (company-wide reboot policies)
- ✅ Device: Yes (device-specific scheduling)

**Customer Configurable:** ⚠️ Advanced customers (with guidance)

**Notes:**
- CSV indicates: "Device level override possible"

---

## Category 9: WAN/Cellular Parameters

### wan1_icmp_host, wan1_icmp_backup_host, wan1_icmp_main_host
**Layers:** Carrier → Customer

**Usage Pattern:**
```
Default/Carrier Layer:
- wan1_icmp_host=10.4.6.30
- wan1_icmp_backup_host=10.4.6.30
- wan1_icmp_main_host=10.4.6.30
- Used for link health monitoring

Customer Layer:
- Customer may need custom monitoring targets
- Customers with private networks may use internal IPs
```

**Layer Availability:**
- ✅ Global: Yes (default monitoring target)
- ❌ Model: No
- ✅ Carrier: Yes (carrier-specific monitoring endpoints)
- ❌ Service Plan: No
- ✅ Company: Yes (customer infrastructure)
- ⚠️ Device: Possibly (rare device-specific targets)

**Customer Configurable:** ⚠️ Advanced customers

**Notes:**
- CSV indicates: "Must be configurable"

---

## Category 10: I/O Parameters (Model-Specific)

### digitalio_config, io_chip, io_triggered_report
**Layers:** Model → Device

**Usage Pattern:**
```
Model Layer:
- I-22/I-52: Has digital I/O capabilities
  - digitalio_config=1,0,0;1,0,0;
  - io_chip=0
  - io_triggered_report=0
- IR611/IR615: No I/O parameters (absent from config)

Device Layer:
- Configure I/O pins for specific device needs
- Alarm inputs, relay outputs
```

**Layer Availability:**
- ❌ Global: No
- ✅ Model: Yes (determines if I/O exists)
- ❌ Carrier: No
- ❌ Service Plan: No
- ❌ Company: No
- ✅ Device: Yes (device-specific I/O config)

**Customer Configurable:** ⚠️ Possibly (for I/O-capable customers)

**Notes:**
- CSV indicates: "I/O ONLY Device - I-22 and I-52 || Absent on IR611/IR615 Configs"

---

## Category 11: Link Backup Parameters

### linkbackup_enable, linkbackup_hot_mode
**Layers:** Model → Dynamic

**Usage Pattern:**
```
Model Layer:
- IR611/IR615: Link backup available
- I-22/I-52: Link backup NOT available
```

**Layer Availability:**
- ❌ Global: No
- ✅ Model: Yes (model hardware capability)
- ❌ Carrier: No
- ❌ Service Plan: No
- ❌ Company: No
- ⚠️ Device: Possibly (enable for devices with backup)

**Customer Configurable:** ❌ No

**Notes:**
- CSV indicates: "Only available for I22/I52" (note: this seems backwards - should be IR611/IR615)

---

## Category 12: Status Reporting Parameters

### rmon_advance_cfg (Advanced Monitoring Config)
**Layers:** Model → Service Plan

**Usage Pattern:**
```
Model Layer:
- I-22: Includes I/O digital data
  - rmon_advance_cfg=_wan1_imei,_wan1_iccid,_wan1_sinr,_iodigital_input_data
- Other models: May exclude I/O data
  - rmon_advance_cfg=_wan1_imei,_wan1_iccid,_wan1_sinr

Service Plan Layer:
- Different plans may report different metrics
- Premium plans may include additional telemetry
```

**Layer Availability:**
- ❌ Global: No
- ✅ Model: Yes (model capabilities)
- ❌ Carrier: No
- ✅ Service Plan: Yes (plan-based reporting features)
- ❌ Company: No
- ❌ Device: Possibly (custom reporting)

**Customer Configurable:** ❌ No

**Notes:**
- CSV indicates: "I-22 may need IO Digital and Other models may not"

---

## Category 13: Alarm Parameters

### alarm_input, alarm_output, alarm_input_options, alarm_output_options
**Layers:** Model

**Usage Pattern:**
```
Model Layer:
- I-22/I-52/Origin Model:
  - alarm_input_options=fault-service,memory-low,port0-wan-link-up/down,...
  - alarm_input=0,0,1,1,0,0,0,0,0,0,0,
  - alarm_output_options=cli,out-dm,out-rmon,
  - alarm_output=0,0,1,
```

**Layer Availability:**
- ❌ Global: No
- ✅ Model: Yes (model-specific alarm capabilities)
- ❌ Carrier: No
- ❌ Service Plan: No
- ❌ Company: No
- ⚠️ Device: Possibly (device-specific alarm config)

**Customer Configurable:** ❌ No

---

## Multi-Layer Parameter Summary Table

| Parameter | Global | Model | Carrier | Service Plan | Company | Device | Conditional Rule Type |
|-----------|--------|-------|---------|--------------|---------|--------|----------------------|
| **fw_acl** | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ | Three-Way + Four-Way |
| **fw_nat** | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | None |
| **mqtt_enable** | ❌ | ✅ | ✅ | ❌ | ❌ | ⚠️ | Two-Way |
| **mqtt_*** (all) | ❌ | ✅ | ✅ | ❌ | ❌ | ⚠️ | Two-Way |
| **lan0_ip** | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ | None |
| **lan0_gateway** | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ | None |
| **lan_port1/2** | ❌ | ✅ | ❌ | ❌ | ⚠️ | ✅ | None |
| **dhcpd_start** | ✅ | ❌ | ❌ | ❌ | ✅ | ✅ | None |
| **dhcpd_end** | ✅ | ❌ | ❌ | ❌ | ✅ | ✅ | None |
| **dhcpd_static** | ❌ | ❌ | ❌ | ❌ | ⚠️ | ✅ | None |
| **dhcpd_lease** | ✅ | ⚠️ | ❌ | ❌ | ❌ | ❌ | None |
| **traffic_day_threshold** | ❌ | ❌ | ❌ | ✅ | ✅ | ⚠️ | None |
| **traffic_month_threshold** | ❌ | ❌ | ❌ | ✅ | ✅ | ⚠️ | None |
| **traffic_*** (all) | ❌ | ❌ | ❌ | ✅ | ✅ | ⚠️ | None |
| **wan1_ppp_apn** | ❌ | ❌ | ✅ | ❌ | ❌ | ⚠️ | Carrier Lookup |
| **wan1_ppp_redial_interval** | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | None |
| **sim1_card_operator** | ❌ | ❌ | ✅ | ❌ | ❌ | ✅ | None |
| **sim2_card_operator** | ❌ | ❌ | ✅ | ❌ | ❌ | ✅ | None |
| **dual_sim_enable** | ❌ | ⚠️ | ✅ | ❌ | ❌ | ⚠️ | Possibly Two-Way |
| **dual_sim_*** (all) | ❌ | ⚠️ | ✅ | ❌ | ❌ | ⚠️ | Possibly Two-Way |
| **cron_rb_enable** | ✅ | ❌ | ❌ | ❌ | ⚠️ | ✅ | None |
| **cron_rb_time** | ✅ | ❌ | ❌ | ❌ | ⚠️ | ✅ | None |
| **cron_rb_days** | ✅ | ❌ | ❌ | ❌ | ⚠️ | ✅ | None |
| **wan1_icmp_host** | ✅ | ❌ | ✅ | ❌ | ✅ | ⚠️ | None |
| **digitalio_config** | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ | None |
| **io_chip** | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ | None |
| **linkbackup_enable** | ❌ | ✅ | ❌ | ❌ | ❌ | ⚠️ | None |
| **rmon_advance_cfg** | ❌ | ✅ | ❌ | ✅ | ❌ | ⚠️ | None |
| **alarm_input** | ❌ | ✅ | ❌ | ❌ | ❌ | ⚠️ | None |
| **alarm_output** | ❌ | ✅ | ❌ | ❌ | ❌ | ⚠️ | None |

**Legend:**
- ✅ Yes - Commonly used at this layer
- ⚠️ Possibly - May be used in rare cases
- ❌ No - Not used at this layer

---

## Conditional Rules Required

### Two-Way Rules (Model + Carrier)

**Parameters:**
1. **mqtt_enable** - Device Manager availability
   - i-22 + VZW → mqtt_enable=1
   - i-22 + TMO → mqtt_enable=1
   - i-22 + ATT → mqtt_enable=0
   - IR611 + * → (parameter not applicable)

2. **mqtt_*** (all related) - Device Manager configuration
   - Same logic as mqtt_enable

3. **dual_sim_enable** - Dual SIM feature (possibly)
   - May vary by Model + Carrier combination

**Total Estimated:** ~15-25 Two-Way Rules

---

### Three-Way Rules (Model + Carrier + Service Plan)

**Parameters:**
1. **fw_acl** - Service Plan baseline firewall rules
   - i-22 + ATT + ATM → fw_acl=[40 whitelist rules]
   - i-22 + VZW + ATM → fw_acl=[40 whitelist rules]
   - i-22 + * + Non-ATM → fw_acl=[allow all]

2. **traffic_day_threshold** - Service Plan data limits
   - * + * + ATM Basic → traffic_day_threshold=350MB
   - * + * + ATM Premium → traffic_day_threshold=500MB
   - * + * + Tier 1 → traffic_day_threshold=X
   - * + * + Tier 2 → traffic_day_threshold=Y

3. **traffic_month_threshold** - Monthly data caps
   - Same pattern as daily threshold

4. **rmon_advance_cfg** - Reporting configuration
   - May vary by Model + Service Plan

**Total Estimated:** ~30-40 Three-Way Rules

---

### Four-Way Rules (Model + Carrier + Service Plan + Company)

**Parameters:**
1. **fw_acl** - Customer exceptions to ATM baseline
   - i-22 + ATT + ATM + CORD → fw_acl=[50 custom rules]
   - i-22 + ATT + ATM + Deployer → fw_acl=[42 custom rules]
   - Overrides Three-Way Rule baseline

2. **fw_nat** - Customer-specific NAT rules
   - Customer-specific port forwarding
   - Traffic optimization

3. **traffic_day_threshold** - Customer data allowance exceptions
   - VIP customers with higher limits

4. **traffic_month_threshold** - Custom monthly caps
   - Customer-specific billing arrangements

**Total Estimated:** ~10-20 Four-Way Rules

---

## Implementation Recommendations

### 1. Layer Inheritance (FR-2)

**Simple Parameters** - Use layer-based inheritance:
- `lan0_ip` - Global → Model → Company → Device
- `dhcpd_start/end` - Global → Company → Device
- `cron_rb_*` - Global → Company → Device
- `wan1_icmp_host` - Global → Carrier → Company → Device

**Resolution Logic:**
```
1. Check Device Layer
2. If null, check Company Layer
3. If null, check Service Plan Layer
4. If null, check Carrier Layer
5. If null, check Model Layer
6. If null, check Global Layer
7. If null, use Schema Default
8. If null and Required, throw validation error
```

---

### 2. Conditional Rules (FR-3a)

**Complex Parameters** - Use Multi-Factor Conditional Rules:
- `fw_acl` - Three-Way baseline + Four-Way exceptions
- `mqtt_enable` - Two-Way rules (Model + Carrier)
- `traffic_*` - Three-Way baselines + Four-Way exceptions

**Resolution Logic (11-Level Priority):**
```
1. Device Override
2. Four-Way Rule (Customer Exception)
3. Company Layer Override
4. Three-Way Rule (Service Plan Baseline)
5. Service Plan Layer
6. Two-Way Rule (Model + Carrier)
7. Carrier Layer
8. Model Layer
9. Global Layer
10. Schema Default
11. Required Validation
```

---

### 3. Hybrid Approach

**Many parameters use BOTH:**

**Example: fw_acl**
- **Three-Way Rule (Level 4)**: i-22 + ATT + ATM → [40 baseline rules]
- **Four-Way Rule (Level 2)**: i-22 + ATT + ATM + CORD → [50 custom rules]
- **Company Override (Level 3)**: CORD manually sets value → overrides Four-Way Rule
- **Device Override (Level 1)**: ATM_STORE_001 manually sets value → highest priority

**Example: lan0_ip**
- **Model Layer (Level 8)**: i-22 → lan0_ip=192.168.1.90
- **Company Layer (Level 3)**: Miele → lan0_ip=192.168.2.1
- **Device Layer (Level 1)**: ATM_123 → lan0_ip=192.168.2.50

---

## Customer Configurability

### Safe for Customer Self-Service:
- `dhcpd_static` - DHCP reservations (with MAC validation)
- `dhcpd_start/end` - DHCP range (advanced customers, with subnet validation)
- `cron_rb_*` - Scheduled reboots (advanced customers, with guidance)
- `lan0_ip` - LAN IP (advanced customers, with conflict detection)

### Admin-Only (Too Risky):
- `fw_acl` - Firewall rules (security impact)
- `fw_nat` - NAT rules (requires expertise)
- `mqtt_*` - Device Manager (infrastructure)
- `wan1_ppp_apn` - Carrier APN (infrastructure)
- `dual_sim_*` - Dual SIM (carrier policy)
- `traffic_*` - Data limits (billing impact)

---

## Validation Requirements

### Multi-Layer Consistency Checks:

1. **fw_acl:**
   - Validate firewall rule syntax
   - Check for conflicting rules
   - Ensure "Block all" is final rule
   - Validate IP addresses and CIDR notation

2. **dhcpd_start/end:**
   - Ensure start < end
   - Verify range within subnet
   - Check for conflicts with lan0_ip
   - Validate against other devices in company

3. **lan0_ip:**
   - Validate IP format
   - Check subnet compatibility
   - Prevent conflicts with DHCP range
   - Ensure gateway accessibility

4. **mqtt_* parameters:**
   - Validate only for supported models
   - Check carrier compatibility
   - Ensure mqtt_center is reachable

5. **traffic_* parameters:**
   - Validate threshold > 0
   - Check unit consistency
   - Ensure monthly > daily (if both set)

---

## Migration Considerations

### Existing Device Configs:

**Current State:**
- Devices have flat configuration files
- No layer hierarchy
- Values hardcoded per device

**Migration Strategy:**

1. **Identify Base Values:**
   - Find common values across all devices → Global Layer
   - Find model-specific values → Model Layer
   - Find carrier-specific values → Carrier Layer

2. **Identify Conditional Rules:**
   - Analyze fw_acl patterns → Create Three-Way + Four-Way Rules
   - Analyze mqtt_enable patterns → Create Two-Way Rules
   - Analyze traffic_* patterns → Create Three-Way Rules

3. **Identify Exceptions:**
   - Find customer-specific values → Company Layer
   - Find device-specific values → Device Layer

4. **Import Tool:**
   - Auto-detect layer assignment based on usage patterns
   - Flag parameters that appear at multiple layers
   - Suggest conditional rule creation for complex patterns
   - Allow manual review before committing

---

## Statistics Summary

**From ConfigsClientAnalisys:**

| Metric | Count |
|--------|-------|
| **Parameters with Multi-Layer Usage** | ~30-40 |
| **Require Two-Way Rules** | ~15-25 |
| **Require Three-Way Rules** | ~30-40 |
| **Require Four-Way Rules** | ~10-20 |
| **Customer Configurable (Safe)** | ~5-10 |
| **Admin-Only (Multi-Layer)** | ~25-30 |

---

## Conclusion

Analysis confirms:
1. ✅ **Layer inheritance is essential** - Most parameters use it
2. ✅ **Conditional rules are required** - Complex multi-factor logic exists
3. ✅ **Hybrid approach is correct** - Same parameter can use both inheritance AND conditional rules
4. ✅ **11-Level Priority is validated** - Real-world examples demonstrate need for full priority hierarchy
5. ✅ **Customer exceptions are real** - Four-Way Rules have actual use cases (CORD, Deployer, etc.)

**The PRD architecture (FR-2 + FR-3a) accurately reflects actual client configuration requirements.**

---

## Next Steps

1. ✅ Update schema with multi-layer flags for each parameter
2. ✅ Create conditional rules management UI (FR-3a)
3. ✅ Implement 11-Level Priority resolution engine
4. ✅ Build layer configuration UI with contextual rules access (FR-2)
5. ⬜ Create migration tool to extract existing conditional patterns
6. ⬜ Build validation engine for multi-layer consistency
7. ⬜ Implement customer self-service with safe parameter filtering

