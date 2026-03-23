# 11-Layer Priority Resolution: Complete Walkthrough

**Date:** March 12, 2026
**Purpose:** Demonstrate how the 11-layer configuration hierarchy resolves a final parameter value across six real-world scenarios
**Source:** Production configuration data, validated rule analysis, and client-provided CSV datasets

### How to Read This Document

This document walks through **six real scenarios** showing how a device configuration gets built, parameter by parameter. Each scenario demonstrates a different layer of the hierarchy "winning" — from the simplest case (Global Layer, where every device gets the same value) to the most specific (Device Override, where one individual device gets a unique value).

**If you're short on time**, read these sections:
1. "How Configuration Works Today vs. the New System" — the 30-second overview
2. "Why Not Just Three Layers?" — addresses the core question about rule complexity
3. "End-to-End: What a Device's Config Preview Looks Like" — shows the final output
4. Scenarios 2 (mqtt_enable) and 4 (CORD firewall) — the most realistic examples

**If you want the full picture**, read straight through — each scenario builds on the previous one.

---

## How Configuration Works Today vs. the New System

### Today: One .DAT File Per Combination

Today, APW manually creates configuration files — one per combination of model, carrier, and service plan. Each file is named something like **"T-Mobile origin ATM"** and contains every parameter and its value as key-value pairs. When a device checks in, the system finds the matching file based on the device's model, carrier, and service plan, and pushes that file to the device.

The problem: with ~200+ config files, changing a single global value (like a DNS server) means editing every file. Adding a new customer exception means duplicating and editing existing files. There's no inheritance, no cascading, and no audit trail for why a value is what it is.

### New System: Layered Resolution

Instead of storing a flat file per combination, the new system stores values at the **layer where they belong**. A parameter like `lan0_netmask` that is always `255.255.255.0` for every device gets stored once at the Global Layer. A parameter like `mqtt_enable` that depends on model and carrier gets stored as a Two-Way Rule. A firewall exception for CORD gets stored as a Four-Way Rule.

When a device needs its config, the system **resolves** each parameter by walking the hierarchy from most-specific to least-specific. The first layer that has a value wins.

**The result is the same .DAT file the device expects** — the device doesn't know or care about layers. But instead of maintaining 200+ hand-edited files, APW maintains values at the layer where the decision actually lives.

### A Device Must Have All Context Before Config Can Be Built

A critical requirement: **a device must have model, carrier, and service plan assigned before the system will generate any configuration.** A device sitting on a shelf with only a model and carrier (no service plan) gets no config file at all — exactly how it works today. The hierarchy only activates when all required context is present.

---

## The 11-Layer Priority Hierarchy

```
Priority 1  (HIGHEST)  │ Device Override           │ Individual device setting
Priority 2             │ Four-Way Rule             │ Model + Carrier + Plan + Customer exception
Priority 3             │ Company Override           │ Customer portfolio-wide setting
Priority 4             │ Three-Way Rule             │ Model + Carrier + Plan baseline
Priority 5             │ Service Plan Layer         │ Service tier default
Priority 6             │ Two-Way Rule (6 sub-types) │ Any two-factor combination
         6.1           │   Carrier + Service Plan   │
         6.2           │   Model + Service Plan     │
         6.3           │   Model + Carrier          │
         6.4           │   Carrier + Customer       │
         6.5           │   Model + Customer         │
         6.6           │   Service Plan + Customer  │
Priority 7             │ Carrier Layer              │ Carrier-specific default
Priority 8             │ Model Layer                │ Model-specific default
Priority 9             │ Global Layer               │ System-wide default
Priority 10            │ Schema Default             │ Hardcoded parameter fallback
Priority 11 (LOWEST)   │ Required Validation        │ Error if still missing
```

**Resolution Rule:** Walk from Priority 1 down to Priority 11. The **first match wins**.

---

## "Why Not Just Three Layers?" — Understanding Rules vs. Layers

A common question: if a device always needs model, carrier, and service plan before it gets a config, why do we need Two-Way rules? Why not just define everything at the Three-Way (Model + Carrier + Plan) level?

The answer is about **how many rules you have to maintain.**

### Example: `mqtt_enable` (Device Manager)

Device Manager depends on **model** and **carrier** — it works on i-22 + VZW but not on i-22 + ATT. It does NOT depend on the service plan. Whether the device is on ATM or Tier1, the Device Manager answer is the same.

**Without Two-Way rules**, you'd need to define `mqtt_enable` for every three-way combination:

```
i-22 + VZW + ATM   → mqtt_enable = 1
i-22 + VZW + Tier1 → mqtt_enable = 1    ← duplicate, same answer
i-22 + ATT + ATM   → mqtt_enable = 0
i-22 + ATT + Tier1 → mqtt_enable = 0    ← duplicate, same answer
i-22 + TMO + ATM   → mqtt_enable = 1
i-22 + TMO + Tier1 → mqtt_enable = 1    ← duplicate, same answer
```

That's 6 rules to express something that only needs 3:

**With a Two-Way rule (Model + Carrier):**

```
i-22 + VZW → mqtt_enable = 1    (applies to ALL service plans)
i-22 + ATT → mqtt_enable = 0    (applies to ALL service plans)
i-22 + TMO → mqtt_enable = 1    (applies to ALL service plans)
```

The Two-Way rule says: "For this model on this carrier, every service plan gets this value." It's not about bypassing the three-factor requirement for the device — the device still must have all three. It's about **where the decision lives** so you don't duplicate rules that don't depend on the third factor.

### Think of Rules as Exceptions

The layers (Global, Model, Carrier, Service Plan, Company, Device) are where you set values that depend on **one** factor. Rules are where you handle exceptions that depend on **combinations**:

- **Two-Way Rule:** "For this specific model AND carrier, override the value" — saves you from duplicating across every service plan
- **Three-Way Rule:** "For this model AND carrier AND plan, set this baseline" — the standard combination
- **Four-Way Rule:** "For this model AND carrier AND plan AND customer, make an exception" — customer-specific override to the three-way baseline

Each rule level is an **exception** to the level below it. Four-Way overrides Three-Way. Three-Way overrides Two-Way. Two-Way overrides individual layers.

---

## Device Context for All Scenarios

Unless otherwise noted, all scenarios resolve for this device:

| Attribute      | Value                    |
|----------------|--------------------------|
| **Device**     | W48284 (serial number)   |
| **Model**      | InHand i-22              |
| **Carrier**    | Verizon (VZW)            |
| **Service Plan** | ATM Plan               |
| **Company**    | CORD                     |

---

## Scenario 1: Global Layer Wins — `ntp_server`

**Parameter:** `ntp_server` (NTP time synchronization server list)
**Question:** Where does this device get its NTP servers from?

### Layer-by-Layer Resolution

| Priority | Layer | Match? | Value | Reason |
|----------|-------|--------|-------|--------|
| 1 | Device Override | — | *not set* | No device-specific NTP override |
| 2 | Four-Way Rule | — | *not set* | No CORD-specific NTP exception |
| 3 | Company Override | — | *not set* | CORD has no custom NTP servers |
| 4 | Three-Way Rule | — | *not set* | NTP is not plan-dependent |
| 5 | Service Plan Layer | — | *not set* | ATM plan does not customize NTP |
| 6 | Two-Way Rule | — | *not set* | No two-factor NTP rules exist |
| 7 | Carrier Layer | — | *not set* | No carrier-specific NTP |
| 8 | Model Layer | — | *not set* | No model-specific NTP |
| **9** | **Global Layer** | **MATCH** | **`10.4.6.30;time.nist.gov;time.google.com`** | **System-wide NTP servers** |

### Final Value

```
ntp_server = 10.4.6.30;time.nist.gov;time.google.com
Source: Global Layer (Priority 9)
```

### Why This is Global

NTP time servers are infrastructure parameters — all 100,000+ devices in the fleet use the same time servers regardless of model, carrier, plan, or customer. The internal NTP server (`10.4.6.30`) is the APW infrastructure endpoint, with `time.nist.gov` and `time.google.com` as public fallbacks.

**Business Impact:** If APW changes their internal NTP server, one update to the Global Layer propagates to every device in the fleet — taking minutes instead of editing 200+ config files.

---

## Scenario 2: Model + Carrier (Two-Way Rule) Wins — `mqtt_enable`

**Parameter:** `mqtt_enable` (InHand Device Manager toggle)
**Question:** Is the Device Manager feature enabled for this device?

### Layer-by-Layer Resolution

| Priority | Layer | Match? | Value | Reason |
|----------|-------|--------|-------|--------|
| 1 | Device Override | — | *not set* | No per-device MQTT override |
| 2 | Four-Way Rule | — | *not set* | No customer exception for MQTT |
| 3 | Company Override | — | *not set* | CORD has no company-wide MQTT override |
| 4 | Three-Way Rule | — | *not set* | MQTT is not plan-dependent |
| 5 | Service Plan Layer | — | *not set* | ATM plan does not set MQTT |
| **6.3** | **Two-Way: Model + Carrier** | **MATCH** | **`1` (enabled)** | **i-22 + VZW → enabled** |

### Final Value

```
mqtt_enable = 1 (enabled)
Source: Two-Way Rule — Model + Carrier (Priority 6.3)
```

### The Model + Carrier Matrix

| Model + Carrier | mqtt_enable | Reason |
|-----------------|-------------|--------|
| **i-22 + VZW** | **1 (enabled)** | VZW supports Device Manager on i-22 |
| i-22 + TMO | 1 (enabled) | TMO supports Device Manager on i-22 |
| i-22 + ATT | 0 (disabled) | ATT does not support Device Manager |
| 4500 + any | 0 (disabled) | Model 4500 does not support Device Manager |
| IR611 + any | *(absent)* | Parameter not applicable to this model |

### Why This is a Two-Way Rule

Device Manager availability depends on **two independent factors**:

1. **Model capability** — Only i-22 and i-52 models support MQTT Device Manager hardware; IR611/IR615/4500 do not
2. **Carrier agreement** — Even on supported models, AT&T has not enabled Device Manager in their network agreement; VZW and TMO have

Neither factor alone determines the value. A carrier layer would set `mqtt_enable=1` for VZW but that would incorrectly enable it on 4500 devices. A model layer would set it for i-22 but incorrectly enable it on ATT.

**Related parameters also governed by this rule:**
`mqtt_center`, `mqtt_keepalive`, `mqtt_lbs_interval`, `mqtt_series_interval`, `mqtt_sniffer_enable`, `mqtt_tls`, `mqtt_username`, `advanced`, `console_enable`, `qos_iface`

---

## Scenario 3: Three-Way Rule Wins — `fw_acl` (baseline)

**Parameter:** `fw_acl` (Firewall Access Control List)
**Device context change:** Company = **Standard Corp** (a typical customer, not CORD)
**Question:** What firewall rules does a standard ATM-plan device get?

### Layer-by-Layer Resolution

| Priority | Layer | Match? | Value | Reason |
|----------|-------|--------|-------|--------|
| 1 | Device Override | — | *not set* | No per-device firewall override |
| 2 | Four-Way Rule | — | *not set* | Standard Corp has no firewall exception |
| 3 | Company Override | — | *not set* | Standard Corp has no company-wide firewall |
| **4** | **Three-Way Rule** | **MATCH** | **40 whitelist rules + Block** | **i-22 + VZW + ATM baseline** |

### Final Value

```
fw_acl = [40 whitelist rules for payment processors, DNS, AWS] + Block-all
Source: Three-Way Rule — Model + Carrier + Service Plan (Priority 4)

Whitelist includes:
  34.199.0.0/16    — Amazon Resource
  208.35.209.1     — CDS
  204.8.249.126    — Switch Commerce
  208.224.248.160  — Switch Commerce
  206.71.17.21/32  — PAI
  20.88.238.228/32 — Digital Network
  8.8.8.8          — DNS
  8.8.4.4          — DNS
  1.1.1.1          — DNS
  3.5.0.0/16       — AWS Genmega
  ... (30+ more entries)
  0.0.0.0/0        — Block all other traffic
```

### Why This is a Three-Way Rule

The firewall ACL for ATM-plan devices is the **single most complex configuration parameter** in the system. It determines which servers ATM devices can communicate with, blocking everything else as a security measure.

The three factors matter because:
1. **Service Plan** — ATM plan requires strict firewall; Tier1 plan allows all traffic (`1<4<0.0.0.0/0<<0.0.0.0/0<<1<1<>`)
2. **Model** — i-22 uses `fw_acl` for URL filtering; 4500 uses a separate content filtering mechanism
3. **Carrier** — Baseline rules are currently identical across carriers, but the architecture supports carrier-specific variations

### Contrast with Non-ATM Plan

A Tier1 device on the same model+carrier gets a completely different firewall:

```
Tier1 Plan:   fw_acl = 1<4<0.0.0.0/0<<0.0.0.0/0<<1<1<>     (allow ALL traffic)
ATM Plan:     fw_acl = [40 whitelist rules] + Block            (strict whitelist)
```

This is the defining difference between ATM and Tier1 service plans — ATM devices are locked down to specific payment processor servers.

---

## Scenario 4: Four-Way Rule Wins — `fw_acl` (customer exception)

**Parameter:** `fw_acl` (Firewall Access Control List)
**Device context:** Company = **CORD** (back to original device context)
**Question:** What firewall rules does CORD's ATM device get?

### Layer-by-Layer Resolution

| Priority | Layer | Match? | Value | Reason |
|----------|-------|--------|-------|--------|
| 1 | Device Override | — | *not set* | No per-device firewall override |
| **2** | **Four-Way Rule** | **MATCH** | **40 baseline + CORD servers** | **i-22 + VZW + ATM + CORD** |

*Resolution stops at Priority 2 — does not reach the Three-Way baseline at Priority 4.*

### Final Value

```
fw_acl = [40 baseline whitelist rules] + [CORD-specific additions] + Block-all
Source: Four-Way Rule — Model + Carrier + Plan + Customer (Priority 2)

Additional entries for CORD:
  13.67.184.126    — CORD RMS (Remote Management Server)
  64.183.178.180   — CORD RMS Old (legacy server)
  10.4.0.31        — Internal server
  10.4.0.32        — Internal server

Total: ~50 whitelist rules (40 baseline + ~10 CORD-specific)
```

### Why This is a Four-Way Rule

CORD's firewall exception passes the Four-Way Test — all four factors matter:

1. **Customer-Specific:** Only CORD needs access to `13.67.184.126` (their RMS server)
2. **Plan-Dependent:** Exception only applies to ATM plan (Tier1 devices allow all traffic anyway)
3. **Model-Dependent:** fw_acl format is model-specific (i-22 uses ACL, 4500 uses content filtering)
4. **Carrier-Dependent:** While CORD's additions are the same for VZW and ATT today, the architecture supports carrier-specific variations if carrier agreements change

### The Baseline → Exception Pattern

This is the core design pattern of the hierarchy:

```
Three-Way Rule (Priority 4):    i-22 + VZW + ATM        → 40 rules  (ALL customers)
Four-Way Rule  (Priority 2):    i-22 + VZW + ATM + CORD → 50 rules  (CORD exception)
```

Every ATM customer gets the 40-rule baseline. Only CORD (and a few others like DEPLOYER with `208.92.212.170`) get additional entries.

### Other Known Four-Way Customer Exceptions

| Customer | Additional Server | Purpose |
|----------|-------------------|---------|
| CORD | 13.67.184.126 | CORD RMS |
| DEPLOYER | 208.92.212.170 | DEPLOYER RMS |
| 1st ISO | 69.21.165.134, 209.103.211.74 | 1st ISO servers |

---

## Scenario 5: Company Override Wins — `dhcpd_start` / `dhcpd_end`

**Parameter:** `dhcpd_start` and `dhcpd_end` (DHCP address range)
**Device context:** Company = **Altech**
**Question:** What DHCP range does this Altech device use for its LAN?

### Layer-by-Layer Resolution

| Priority | Layer | Match? | Value | Reason |
|----------|-------|--------|-------|--------|
| 1 | Device Override | — | *not set* | No device-specific DHCP override |
| 2 | Four-Way Rule | — | *not set* | DHCP is not plan-dependent |
| **3** | **Company Override** | **MATCH** | **`192.168.2.100` / `192.168.2.200`** | **Altech network architecture** |

### Final Value

```
dhcpd_start = 192.168.2.100
dhcpd_end   = 192.168.2.200
Source: Company Override (Priority 3)
```

### What Would Have Happened Without the Company Override?

If Altech had no company-level DHCP settings, resolution would continue down:

| Priority | Layer | Value |
|----------|-------|-------|
| 4 | Three-Way Rule | *not set* (DHCP is not plan-dependent) |
| 5 | Service Plan | *not set* |
| 6 | Two-Way Rule | *not set* |
| 7 | Carrier Layer | *not set* |
| 8 | Model Layer | *not set* |
| **9** | **Global Layer** | **`192.168.1.100` / `192.168.1.254`** |

The Global default (`192.168.1.x`) would apply, but Altech's internal network uses a different subnet.

### Why This is a Company Override (Not a Rule)

DHCP settings are **customer infrastructure** — they depend solely on the customer's network architecture, not on model, carrier, or service plan:

- Altech uses `192.168.2.x` regardless of whether the device is an i-22 or 4500
- Altech uses `192.168.2.x` regardless of whether the carrier is VZW or ATT
- Altech uses `192.168.2.x` regardless of whether the plan is ATM or Tier1

**Decision tree:** Does it vary by model? No. By carrier? No. By plan? No. Only by customer? → **Company Override.**

### Other Known Company Overrides

| Customer | Parameter | Value | Default |
|----------|-----------|-------|---------|
| Altech | dhcpd_start | 192.168.2.100 | 192.168.1.100 |
| Altech | dhcpd_end | 192.168.2.200 | 192.168.1.254 |
| Miele | lan0_ip | 10.142.12.1 | 192.168.1.90 |
| Miele | dhcpd_start | 10.142.12.100 | 192.168.1.100 |
| Miele | dhcpd_end | 10.142.12.200 | 192.168.1.254 |
| CORD | fw_nat | *(port forwarding rules)* | *(empty)* |

---

## Scenario 6: Device Override Wins — `cron_rb_time`

**Parameter:** `cron_rb_time` (Scheduled reboot time, in minutes past midnight)
**Device context:** Device W48284, CORD company
**Question:** When does this specific device reboot?

### Layer-by-Layer Resolution

| Priority | Layer | Match? | Value | Reason |
|----------|-------|--------|-------|--------|
| **1** | **Device Override** | **MATCH** | **`330` (5:30 AM)** | **Technician set for off-hours** |

*Resolution stops immediately at Priority 1.*

### Final Value

```
cron_rb_time = 330 (5:30 AM local time)
Source: Device Override (Priority 1)
```

### What Would Have Happened Without the Device Override?

| Priority | Layer | Value | Reason |
|----------|-------|-------|--------|
| 2 | Four-Way Rule | *not set* | Reboot time is not plan-specific |
| 3 | Company Override | *not set* | CORD has no company-wide reboot time |
| 4-6 | Rules | *not set* | Reboot time is not rule-driven |
| 7-8 | Carrier/Model | *not set* | Not carrier/model-dependent |
| **9** | **Global Layer** | **`225` (3:45 AM)** | **System-wide default reboot** |

The system default of 3:45 AM would apply, but this specific device services a location that needs to be operational during early morning hours, so a technician overrode it to 5:30 AM.

### Why Device Override Exists

Device overrides are the **most specific** layer — they apply to exactly one device. Use cases:

- **Individual device has unique site requirements** (this device's ATM location opens at 4 AM)
- **Troubleshooting:** Temporarily override a config to test something on one device
- **Special hardware:** Device has a non-standard setup that differs from its model defaults
- **Customer request:** Customer wants a specific device to behave differently from the rest of their fleet

### Related Parameters with Device Override Capability

| Parameter | Global Default | Override Example |
|-----------|---------------|------------------|
| cron_rb_time | 225 (3:45 AM) | 330 (5:30 AM) for early-open ATM |
| cron_rb_enable | 0 (disabled) | 1 (enabled) for specific device |
| cron_rb_days | 0 (daily) | 1 (specific days) |
| lan0_ip | 192.168.1.90 | 192.168.1.50 for special LAN setup |
| fw_acl | *(inherits from above)* | *(device-specific firewall for testing)* |

---

## Full Resolution Comparison

This table shows the same device (i-22, VZW, ATM, CORD, W48284) resolving all six parameters:

| Parameter | Winner | Priority | Value | Why Not Simpler? |
|-----------|--------|----------|-------|------------------|
| `ntp_server` | Global Layer | 9 | `10.4.6.30;time.nist.gov;time.google.com` | Same for all 100K+ devices |
| `mqtt_enable` | Two-Way: Model+Carrier | 6.3 | `1` (enabled) | Depends on both model AND carrier |
| `fw_acl` (Standard Corp) | Three-Way Rule | 4 | 40 whitelist rules + Block | Depends on model + carrier + plan |
| `fw_acl` (CORD) | Four-Way Rule | 2 | 50 rules (40 + CORD servers) | Customer exception to plan baseline |
| `dhcpd_start` (Altech) | Company Override | 3 | `192.168.2.100` | Customer infrastructure, not plan-dependent |
| `cron_rb_time` | Device Override | 1 | `330` (5:30 AM) | This one specific device needs it |

---

## End-to-End: What a Device's Config Preview Looks Like

This is what the system produces for device **W48284** (i-22, VZW, ATM, CORD). Think of this as the equivalent of clicking on a device in the portal and seeing its configuration — except now every parameter shows **where** it came from and **why**.

```
┌─────────────────────────────────────────────────────────────────────┐
│  Device W48284 — Config Preview                                     │
│  Model: i-22  │  Carrier: VZW  │  Plan: ATM  │  Company: CORD      │
├────────────────────┬──────────────────────────────────┬──────────────┤
│  Parameter         │  Final Value                     │  Source      │
├────────────────────┼──────────────────────────────────┼──────────────┤
│  lan0_netmask      │  255.255.255.0                   │  Global      │
│  lan0_ip           │  192.168.1.90                    │  Global      │
│  ntp_server        │  10.4.6.30;time.nist.gov;...     │  Global      │
│  dns_static        │  8.8.8.8;1.1.1.1                 │  Global      │
│  cron_rb_enable    │  0                               │  Global      │
│  dhcpd_start       │  192.168.1.100                   │  Global      │
│  dhcpd_end         │  192.168.1.254                   │  Global      │
│  ...               │  (580+ more global params)       │  Global      │
│────────────────────┼──────────────────────────────────┼──────────────│
│  digitalio_config  │  1                               │  Model       │
│  alarm_di_option   │  ABCD                            │  Model       │
│  ...               │  (6 more model params)           │  Model       │
│────────────────────┼──────────────────────────────────┼──────────────│
│  wan1_iccid_apn    │  Matrxatm.gw12.vzwentp           │  Carrier     │
│  dual_sim_enable   │  1                               │  Carrier     │
│  ...               │  (6 more carrier params)         │  Carrier     │
│────────────────────┼──────────────────────────────────┼──────────────│
│  mqtt_enable       │  1 (enabled)                     │  2-Way Rule  │
│  mqtt_center       │  (InHand server URL)             │  2-Way Rule  │
│  ...               │  (20 more two-way params)        │  2-Way Rule  │
│────────────────────┼──────────────────────────────────┼──────────────│
│  traffic_day_thres │  3584                            │  Svc Plan    │
│  ...               │  (7 more service plan params)    │  Svc Plan    │
│────────────────────┼──────────────────────────────────┼──────────────│
│  fw_acl            │  [40 rules + CORD RMS servers]   │  4-Way Rule  │
│  ...               │  (6 more four-way params)        │  4-Way Rule  │
│────────────────────┼──────────────────────────────────┼──────────────│
│  cron_rb_time      │  330 (5:30 AM)                   │  Device      │
└────────────────────┴──────────────────────────────────┴──────────────┘
```

**This is the same .DAT file the device has always received** — the device doesn't know about layers. But now APW can see exactly where every value came from. If something looks wrong, they know which layer to fix. If they need to change a global DNS server, they change it once and every device that doesn't have an override picks it up automatically.

### What This Looks Like for a Different Device

Now compare device **W48284** (CORD) with a hypothetical **Miele** device on the same model/carrier/plan:

| Parameter | CORD Device (W48284) | Miele Device | Why Different? |
|-----------|---------------------|--------------|----------------|
| `lan0_ip` | 192.168.1.90 (Global) | 10.142.12.1 (Company Override) | Miele uses different LAN |
| `dhcpd_start` | 192.168.1.100 (Global) | 10.142.12.100 (Company Override) | Miele's network range |
| `fw_acl` | 40 rules + CORD RMS (4-Way) | 40 rules (3-Way baseline) | Miele has no firewall exception |
| `mqtt_enable` | 1 (2-Way Rule) | 1 (2-Way Rule) | Same — both are i-22 + VZW |
| `ntp_server` | 10.4.6.30;... (Global) | 10.4.6.30;... (Global) | Same — infrastructure param |
| `cron_rb_time` | 330 (Device Override) | 225 (Global default) | No device override for Miele |

Same model, same carrier, same plan — but different companies produce different configs. The hierarchy handles this without duplicating files.

---

## How the Hierarchy Prevents Mistakes

### Scenario: DNS Server Migration

APW is building an on-site DNS server and needs to migrate from `8.8.8.8;8.8.4.4` to their new internal DNS across the fleet.

**Old system:** Edit all 200+ config files manually. Find every file that has `dns_static`, update the value. Time: 5-10 hours. Risk: typos, missed files, inconsistencies.

**New system:** One change to the Global Layer:

```
Global Layer: dns_static = 10.4.6.50;1.1.1.1
```

This automatically propagates to all 100,000+ devices on their next config update. Any customer with a Company Override for DNS is NOT affected because Priority 3 beats Priority 9 — their override still wins.

### Scenario: New Customer Firewall Exception

Customer "NewCo" needs access to their private server `203.0.113.50` on ATM plan.

**Old system:** Find every .DAT file for NewCo's model/carrier/plan combination. Copy the existing file. Add the new IP to the firewall rules. Hope you didn't break the formatting.

**New system:**
- **Step 1:** Three-Way baseline already exists (40 rules for i-22 + VZW + ATM). No changes needed.
- **Step 2:** Create one Four-Way Rule:

```
Model=i-22, Carrier=VZW, Plan=ATM, Customer=NewCo
→ fw_acl = [40 baseline rules] + 203.0.113.50 + Block
```

No other customers are affected. No existing files are modified. If the baseline rules change (say a payment processor changes their IP), the Three-Way Rule is updated once and NewCo's Four-Way Rule inherits the new baseline.

### Scenario: Different Parameter Keys Per Model (APN Example)

This is a real complexity of the current system: the AT&T APN value is the same (`matrix.com.attz`), but the **parameter name** is different depending on the model:

| Model | Parameter Key | Value |
|-------|--------------|-------|
| i-22 | `wan1_iccid_apn` | matrix.com.attz |
| 4500 | `cellular_apn` | matrix.com.attz |

In the old system, this means the i-22 ATT file and the 4500 ATT file have different keys for the same concept. In the new system, each model's schema defines which parameter keys apply to it. When you set the carrier layer for ATT, the system knows which APN parameter to populate based on the model — you don't have to remember the key names yourself.

---

## Layer Statistics from Production Data

Based on analysis of 622-624 configuration parameters:

| Dependency Type | Parameters | % | Resolution Layer |
|-----------------|-----------|---|------------------|
| Consistent across all | 586 | 94.2% | Global Layer (Priority 9) |
| Model-only | 8 | 1.3% | Model Layer (Priority 8) |
| Carrier-only | 8 | 1.3% | Carrier Layer (Priority 7) |
| Two-Way (Model+Carrier) | 22 | 3.5% | Two-Way Rule (Priority 6) |
| Three-Way (M+C+SP) | 19 | 3.1% | Three-Way Rule (Priority 4) |
| Service-Plan driven | 8 | 1.3% | Service Plan Layer (Priority 5) |
| Customer-specific exceptions | 7 | 1.1% | Four-Way Rule (Priority 2) |

**Key insight:** 94.2% of parameters resolve at the Global Layer. The hierarchy exists for the remaining 5.8% where context matters — but those parameters (firewall rules, MQTT, traffic thresholds, carrier settings) are the ones that define service quality and security.

---

## Common Questions

### "Can we simplify this? 11 layers feels like a lot."

The 11 layers exist in the hierarchy definition, but in practice, **94.2% of parameters only use one layer (Global)**. Most parameters are like `lan0_netmask` — always `255.255.255.0`, no exceptions ever. Those resolve at Global and never touch the other layers.

The complexity lives in the remaining 5.8% — and those are the parameters that cause the most pain today: firewall rules, carrier-specific settings, customer exceptions. The hierarchy doesn't add complexity to those; it **organizes complexity that already exists** in the 200+ .DAT files.

### "Can't we just define carrier settings per model and skip the Two-Way rules?"

Yes — and that's essentially what a Two-Way (Model + Carrier) rule is. It's carrier settings that are specific to a model. The "Two-Way Rule" name just formalizes the pattern so the system can resolve it predictably. Whether you think of it as "carrier settings broken out by model" or "a Model + Carrier two-way rule," the end result is the same: for i-22 on ATT, `mqtt_enable = 0`; for i-22 on VZW, `mqtt_enable = 1`.

### "Why does Company Override (Priority 3) outrank Three-Way Rule (Priority 4)?"

Because a customer's infrastructure requirements override standard baselines. Example: the Three-Way Rule says `dhcpd_start = 192.168.1.100` for all i-22 + VZW + ATM devices. But Miele's network requires `10.142.12.x`. Miele's Company Override at Priority 3 wins over the Three-Way baseline at Priority 4 — meaning Miele's devices get their custom LAN regardless of which model/carrier/plan combination they're on.

### "Does this need to be 100% complete on day one?"

The hierarchy design needs to be complete, but the **rollout** can be gradual. The plan is to run both systems in parallel — legacy .DAT files alongside the new configuration engine. One device, one customer at a time can be migrated over. The system can generate a preview config and compare it against the existing .DAT file to verify they match before switching a device over.

### "What happens if I set the same parameter at two different layers?"

The higher-priority layer always wins. If you set `lan0_ip` at both Global (Priority 9) and Company Override (Priority 3), the Company Override wins for that customer's devices. All other customers still get the Global value. This is the entire point of the hierarchy — you don't have to worry about conflicts because the priority order is deterministic.

---

## Appendix A: Parameter Quick Reference

### Parameters Used in This Document

| Parameter | Description | Global Default | Data Type |
|-----------|-------------|---------------|-----------|
| `ntp_server` | NTP time server list | `10.4.6.30;time.nist.gov;time.google.com` | Semicolon-separated IPs |
| `mqtt_enable` | Device Manager toggle | `0` (disabled) | Boolean (0/1) |
| `fw_acl` | Firewall whitelist rules | *(empty — allow all)* | ACL rule string |
| `dhcpd_start` | DHCP range start | `192.168.1.100` | IP address |
| `dhcpd_end` | DHCP range end | `192.168.1.254` | IP address |
| `cron_rb_time` | Scheduled reboot (min past midnight) | `225` (3:45 AM) | Integer (0-1440) |
| `dns_static` | Static DNS servers | `8.8.8.8;1.1.1.1` | Semicolon-separated IPs |
| `traffic_day_threshold` | Daily data limit (MB) | `3584` (3.5 GB) | Integer |

### Models Referenced

| Model | Device Manager | I/O Ports | Firewall Method |
|-------|----------------|-----------|-----------------|
| i-22 | Supported (VZW/TMO only) | Yes | fw_acl (ACL rules) |
| i-52 | Supported (VZW/TMO only) | Yes | fw_acl (ACL rules) |
| 4500 | Not supported | No | Content filtering |
| IR611 | Not supported | No | Content filtering |

### Carriers Referenced

| Carrier | Code | Device Manager | Primary APN |
|---------|------|----------------|-------------|
| Verizon | VZW | Enabled | Matrxatm.gw12.vzwentp |
| AT&T | ATT | Disabled | matrix.com.attz |
| T-Mobile | TMO | Enabled | *(varies)* |
| Data Connect | DC | Enabled | *(varies)* |

### Service Plans Referenced

| Plan | Firewall | Data Limit (i-22/VZW) | Use Case |
|------|----------|----------------------|----------|
| ATM | Strict whitelist (40 rules) | 3,584 MB/day (3.5 GB) | ATM/payment devices |
| Tier1 | Allow all traffic | 5,120 MB/day (5 GB) | General IoT devices |

---

**Document Status:** Complete
**Evidence Sources:**
- Production configuration files (26+ analyzed)
- Client-provided CSV data (GlobalConfigs, ModelConfigs, CarrierConfigs, ServicePlanConfigs, CustomerConfigs, DeviceConfigs)
- Validated rule combination analysis (Two-Way, Three-Way, Four-Way)
- Config Dependencies Complete Analysis (622-624 parameters)
- Multi-Layer Parameters Analysis (13 categories)
