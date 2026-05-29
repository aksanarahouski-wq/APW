# Configuration Management - Data Structure Requirements

**Date:** December 9, 2025
**Version:** 1.0
**Status:** Draft for Discovery Session

---

## Table of Contents

1. [Configuration Entities & Relationships](#1-configuration-entities--relationships)
2. [Configuration Hierarchy & Inheritance](#2-configuration-hierarchy--inheritance)
3. [Proposed Data Model](#3-proposed-data-model)
4. [Parameter Categorization by Level](#4-parameter-categorization-by-level)
5. [Validation Rules](#5-validation-rules)
6. [Override Rules & Conflict Resolution](#6-override-rules--conflict-resolution)
7. [Implementation Considerations](#7-implementation-considerations)

---

## 1. Configuration Entities & Relationships

### Configuration Entities

Based on the current system and meeting discussions, configurations exist at the following entity levels:

```
┌─────────────────────────────────────────────────────────────┐
│                    GLOBAL CONFIGURATION                      │
│  694 total parameters (320 commonly used, 302 never used)   │
└─────────────────────────────────────────────────────────────┘
                            ▼
        ┌───────────────────┴───────────────────┐
        ▼                                       ▼
┌──────────────────┐                  ┌──────────────────┐
│  MANUFACTURER    │                  │  CARRIER         │
│  Configuration   │                  │  Configuration   │
│  (InHand, etc.)  │                  │  (VZW, AT&T...)  │
└──────────────────┘                  └──────────────────┘
        ▼                                       ▼
┌──────────────────┐                  ┌──────────────────┐
│  MODEL           │                  │  SERVICE PLAN    │
│  Configuration   │                  │  Configuration   │
│  (IR915L, etc.)  │                  │  (Plan-specific) │
└──────────────────┘                  └──────────────────┘
        │                                       │
        └───────────────────┬───────────────────┘
                            ▼
                  ┌──────────────────┐
                  │  COMPANY/CLIENT  │
                  │  Configuration   │
                  │  (Custom Configs)│
                  └──────────────────┘
                            ▼
                  ┌──────────────────┐
                  │  DEVICE          │
                  │  Configuration   │
                  │  (Device-specific)│
                  └──────────────────┘
```

### Entity Definitions

#### 1.1 Global Configuration
**Description:** Base configuration containing all possible parameters with default values

**Characteristics:**
- Contains all 694 parameters from InHand router specification
- Defines system-wide defaults
- Read-only for most users (admin-only modifications)
- Version controlled
- Single source of truth for parameter definitions

**Who Manages:** System administrators only

**Example Use Case:**
- Default DNS servers (8.8.8.8, 1.1.1.1)
- Default DHCP lease time (60)
- Default NTP server (time.nist.gov)

---

#### 1.2 Manufacturer Configuration
**Description:** Manufacturer-specific parameter defaults and constraints

**Characteristics:**
- Associated with manufacturer entity (e.g., InHand Networks)
- Defines hardware-specific parameters
- Sets hardware capability constraints
- Rarely changes

**Who Manages:** System administrators

**Example Parameters:**
- Hardware-specific settings
- Firmware version requirements
- Supported feature sets

**Current Gap:** Not explicitly implemented in current system

---

#### 1.3 Model Configuration
**Description:** Device model-specific configurations

**Characteristics:**
- Associated with specific device models (e.g., IR915L, IR302, IR615)
- Inherits from Manufacturer config
- Defines model-specific capabilities and defaults
- Hardware interface definitions (number of LAN ports, cellular modems, etc.)

**Who Manages:** System administrators, Sales/Support (with proper permissions)

**Example Parameters:**
```
Model: IR915L
- wan1_iface: /dev/ttyUSB3
- io_chip: 1
- lan_port1, lan_port2: enabled by default
- lan_port3, lan_port4: availability depends on model variant
- cellular_modem: present
- wifi_capability: model-dependent
```

**Override Scope:** ~50-75 parameters out of 694 total

---

#### 1.4 Carrier Configuration
**Description:** Cellular carrier-specific configurations

**Characteristics:**
- Associated with carriers (Verizon, AT&T, T-Mobile, etc.)
- Defines carrier-specific APN settings, network parameters
- May include carrier-mandated security settings

**Who Manages:** System administrators, Network operations team

**Example Parameters:**
```
Carrier: Verizon
- wan1_ppp_apn: Matrxatm.gw12.vzwentp
- wan1_ppp_authen: chap (or other auth method)
- dns_static: Carrier-preferred DNS servers
- mtu_settings: Carrier-optimized values

Carrier: AT&T
- wan1_ppp_apn: [AT&T APN]
- [carrier-specific parameters]
```

**Override Scope:** ~20-30 parameters

---

#### 1.5 Service Plan Configuration
**Description:** Service plan-specific configurations and policies

**Characteristics:**
- Associated with billing service plans
- May define data limits, QoS policies, feature enablement
- Connected to billing system

**Who Manages:** Sales, Billing administrators

**Example Parameters:**
```
Service Plan: ATM Premium
- traffic_day_threshold: 3584 (MB)
- qos_settings: priority traffic rules
- ssl_proxy_enable: 1 (required for payment processing)
- feature_flags: OpenVPN enabled, MQTT enabled

Service Plan: ATM Basic
- traffic_day_threshold: 1024
- qos_settings: standard
- ssl_proxy_enable: 1
- feature_flags: limited
```

**Override Scope:** ~25-40 parameters

**Current Gap:** Service plans exist but not directly tied to configs currently

---

#### 1.6 Company Configuration
**Description:** Customer/company-specific custom configurations

**Characteristics:**
- Associated with company entity in multi-tenant system
- Contains customer-requested custom parameters
- **NOT customer-manageable** - managed by sales/support on behalf of customer
- Can apply to all devices within a company or subset of devices

**Who Manages:** Sales team, Support team, Account managers

**Example Parameters:**
```
Company: Altec Enterprises
- Custom translation rules (for printer communication)
- fw_nat: Custom port forwarding rules
- fw_web: Custom web filtering rules
- dhcpd_start/end: Custom IP ranges
- Custom DHCP reservations

Company: LibertyX
- fw_web: 1<libertyx.com<1<1<> (web filter rules)
- Custom firewall ACL rules
- Custom SSL tunnel configurations
```

**Override Scope:** Variable, typically 10-50 parameters

**Key Point from Meeting:**
> "Customers do NOT configure these themselves. Sales/support staff take customer requirements and build these configurations for them."

---

#### 1.7 Device Configuration
**Description:** Individual device-specific configurations

**Characteristics:**
- Associated with single device
- Customer-manageable for limited parameters
- Contains device-unique values (MAC addresses, serial numbers, hostname)
- Highest priority in override hierarchy

**Who Manages:**
- Admin users: all device parameters
- Customers: limited subset (WiFi, basic firewall)

**Customer-Manageable Parameters (10-15 parameters):**
```
- wl0_enable: WiFi on/off
- wl0_ssid: WiFi network name
- wl0_passwd: WiFi password
- wl0_encrypt: WiFi encryption type
- fw_acl: Basic firewall rules (simplified interface)
- hostname: Device name (optional)
```

**Admin-Only Device Parameters:**
```
- lan0_ip: LAN IP address
- lan0_mac: MAC address (read-only, device-assigned)
- wan0_ip, wan0_mac: WAN configuration
- serialnum: Device serial number
- All cellular/PPP settings
- Advanced firewall rules
- SSL tunnel configurations
```

**Override Scope:** All parameters can be overridden at device level by admins

---

### Entity Relationship Summary

| Entity | Can Have Configs? | Can Override? | Managed By | Typical Parameter Count |
|--------|------------------|---------------|------------|------------------------|
| **Global** | ✅ Yes | ❌ No (base) | System Admin | 694 (all) |
| **Manufacturer** | ✅ Yes | ✅ Yes | System Admin | 50-75 |
| **Model** | ✅ Yes | ✅ Yes | System Admin, Sales | 50-75 |
| **Carrier** | ✅ Yes | ✅ Yes | System Admin, NetOps | 20-30 |
| **Service Plan** | ✅ Yes | ✅ Yes | Sales, Billing Admin | 25-40 |
| **Company** | ✅ Yes | ✅ Yes | Sales, Support | 10-50 (variable) |
| **Device** | ✅ Yes | ✅ Yes | Admin, Customer (limited) | 10-15 (customer), All (admin) |

---

## 2. Configuration Hierarchy & Inheritance

### Override Priority (Low to High)

```
Priority 1 (Lowest):  Global Configuration
Priority 2:           Manufacturer Configuration
Priority 3:           Model Configuration
Priority 4:           Carrier Configuration
Priority 5:           Service Plan Configuration
Priority 6:           Company Configuration
Priority 7 (Highest): Device Configuration
```

**Rule:** Higher priority levels override lower priority levels

**Example:**
```
Global:       dns_static = 8.8.8.8;1.1.1.1
Carrier:      dns_static = [Carrier DNS servers]  ← Overrides Global
Company:      dns_static = [Company DNS servers]  ← Overrides Carrier
Device:       dns_static = [Custom DNS]           ← Overrides Company

Final Value = Device DNS (if set), else Company DNS (if set), else Carrier DNS, else Global DNS
```

### Inheritance Model

#### Direct Inheritance
Parameters are inherited from lower-priority levels unless explicitly overridden

```
Example: hostname parameter

Global:    hostname = "InHandRouter"
Model:     [not set - inherits Global]
Company:   [not set - inherits Global]
Device A:  hostname = "DC_22_06012023"  ← Override
Device B:  [not set - inherits Global]

Result:
- Device A hostname = "DC_22_06012023" (overridden)
- Device B hostname = "InHandRouter" (inherited from Global)
```

#### Partial Override
Some complex parameters can be partially overridden (e.g., firewall rules)

```
Example: fw_acl (firewall rules)

Global:    fw_acl = [Basic security rules]
Company:   fw_acl = [Company rules] + [inherited Global rules]
Device:    fw_acl = [Device-specific rules] + [inherited Company + Global rules]

Result: Merged rule set with device-specific rules applied first
```

### Inheritance Behavior by Parameter Type

| Parameter Type | Inheritance Behavior | Example |
|----------------|---------------------|---------|
| **Simple Value** | Complete override | `hostname`, `lan0_ip` |
| **Boolean Flag** | Complete override | `wl0_enable`, `ssl_proxy_enable` |
| **List/Array** | Merge or Replace (configurable) | `fw_acl`, `fw_nat` |
| **Complex Object** | Merge or Replace (configurable) | `ssl_server`, `openvpn_tunnels` |

---

## 3. Proposed Data Model

### 3.1 Database Schema

#### Table: `config_templates`
Stores configuration templates at various levels

```sql
CREATE TABLE config_templates (
    id INT PRIMARY KEY AUTO_INCREMENT,
    template_type ENUM('global', 'manufacturer', 'model', 'carrier', 'service_plan', 'company', 'device') NOT NULL,
    entity_id INT NULL,  -- Foreign key to respective entity table (null for global)
    template_name VARCHAR(255) NOT NULL,
    description TEXT,
    config_data JSON NOT NULL,  -- All parameters stored as JSON
    version INT DEFAULT 1,
    is_active BOOLEAN DEFAULT TRUE,
    created_by INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_by INT,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_template_type (template_type),
    INDEX idx_entity_id (entity_id),
    INDEX idx_active (is_active),

    FOREIGN KEY (created_by) REFERENCES users(id),
    FOREIGN KEY (updated_by) REFERENCES users(id)
);
```

#### Table: `config_parameters`
Metadata about each configuration parameter

```sql
CREATE TABLE config_parameters (
    id INT PRIMARY KEY AUTO_INCREMENT,
    parameter_name VARCHAR(255) UNIQUE NOT NULL,
    display_name VARCHAR(255),
    description TEXT,
    category VARCHAR(100),  -- 'Administration', 'Network', 'Firewall', etc.
    data_type ENUM('string', 'integer', 'boolean', 'ip_address', 'json', 'encrypted') NOT NULL,
    default_value TEXT,
    is_required BOOLEAN DEFAULT FALSE,
    is_encrypted BOOLEAN DEFAULT FALSE,
    validation_regex VARCHAR(500),
    min_value DECIMAL(10,2),
    max_value DECIMAL(10,2),
    allowed_at_levels JSON,  -- Which levels can override this parameter
    usage_frequency ENUM('always_empty', 'mostly_empty', 'sometimes_used', 'commonly_used') NOT NULL,
    ui_priority ENUM('exclude', 'expert', 'advanced', 'essential') NOT NULL,
    help_text TEXT,
    related_parameters JSON,  -- Parameters that should be shown/hidden based on this value

    INDEX idx_category (category),
    INDEX idx_usage (usage_frequency),
    INDEX idx_priority (ui_priority)
);
```

#### Table: `config_overrides`
Tracks parameter overrides at each level

```sql
CREATE TABLE config_overrides (
    id INT PRIMARY KEY AUTO_INCREMENT,
    template_id INT NOT NULL,
    parameter_name VARCHAR(255) NOT NULL,
    parameter_value TEXT,
    override_reason TEXT,
    override_type ENUM('complete', 'merge', 'append') DEFAULT 'complete',
    created_by INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_by INT,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (template_id) REFERENCES config_templates(id) ON DELETE CASCADE,
    FOREIGN KEY (parameter_name) REFERENCES config_parameters(parameter_name),
    FOREIGN KEY (created_by) REFERENCES users(id),
    FOREIGN KEY (updated_by) REFERENCES users(id),

    UNIQUE KEY unique_template_parameter (template_id, parameter_name),
    INDEX idx_parameter (parameter_name)
);
```

#### Table: `device_configs` (or extend existing `devices` table)
Links devices to their configuration hierarchy

```sql
CREATE TABLE device_configs (
    id INT PRIMARY KEY AUTO_INCREMENT,
    device_id INT NOT NULL,
    global_template_id INT NOT NULL,
    manufacturer_template_id INT,
    model_template_id INT,
    carrier_template_id INT,
    service_plan_template_id INT,
    company_template_id INT,
    device_template_id INT,  -- Device-specific overrides
    compiled_config JSON,  -- Cached resolved configuration
    compiled_at TIMESTAMP,
    needs_recompile BOOLEAN DEFAULT FALSE,

    FOREIGN KEY (device_id) REFERENCES devices(id) ON DELETE CASCADE,
    FOREIGN KEY (global_template_id) REFERENCES config_templates(id),
    FOREIGN KEY (manufacturer_template_id) REFERENCES config_templates(id),
    FOREIGN KEY (model_template_id) REFERENCES config_templates(id),
    FOREIGN KEY (carrier_template_id) REFERENCES config_templates(id),
    FOREIGN KEY (service_plan_template_id) REFERENCES config_templates(id),
    FOREIGN KEY (company_template_id) REFERENCES config_templates(id),
    FOREIGN KEY (device_template_id) REFERENCES config_templates(id),

    UNIQUE KEY unique_device (device_id),
    INDEX idx_needs_recompile (needs_recompile)
);
```

#### Table: `config_audit_log`
Audit trail of all configuration changes

```sql
CREATE TABLE config_audit_log (
    id INT PRIMARY KEY AUTO_INCREMENT,
    template_id INT NOT NULL,
    parameter_name VARCHAR(255),
    old_value TEXT,
    new_value TEXT,
    change_type ENUM('create', 'update', 'delete') NOT NULL,
    changed_by INT NOT NULL,
    change_reason TEXT,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (template_id) REFERENCES config_templates(id),
    FOREIGN KEY (changed_by) REFERENCES users(id),

    INDEX idx_template (template_id),
    INDEX idx_changed_at (changed_at),
    INDEX idx_changed_by (changed_by)
);
```

---

### 3.2 JSON Configuration Structure

Each configuration template stores parameters as JSON:

```json
{
    "version": "1.0",
    "parameters": {
        "adm_user": {
            "value": "matrixatm1",
            "source": "global",
            "override_allowed": false
        },
        "adm_passwd": {
            "value": "$AES$2B1A2AA2...",
            "source": "company",
            "override_allowed": true
        },
        "hostname": {
            "value": "DC_22_06012023",
            "source": "device",
            "override_allowed": true
        },
        "lan0_ip": {
            "value": "192.168.1.90",
            "source": "company",
            "override_allowed": true
        },
        "wan1_ppp_apn": {
            "value": "Matrxatm.gw12.vzwentp",
            "source": "carrier",
            "override_allowed": true
        },
        "fw_acl": {
            "value": "1<4<0.0.0.0/0<<0.0.0.0/0<...",
            "source": "company",
            "override_allowed": true,
            "merge_strategy": "append"
        }
    },
    "metadata": {
        "created_at": "2025-12-09T10:00:00Z",
        "created_by": "admin@watm.com",
        "last_modified": "2025-12-09T15:30:00Z",
        "last_modified_by": "support@watm.com"
    }
}
```

---

### 3.3 Configuration Resolution Algorithm

When a device needs its configuration, the system resolves it through this algorithm:

```javascript
function resolveDeviceConfiguration(deviceId) {
    // 1. Load all template layers
    const global = loadTemplate('global');
    const manufacturer = loadTemplate('manufacturer', device.manufacturer_id);
    const model = loadTemplate('model', device.model_id);
    const carrier = loadTemplate('carrier', device.carrier_id);
    const servicePlan = loadTemplate('service_plan', device.service_plan_id);
    const company = loadTemplate('company', device.company_id);
    const device = loadTemplate('device', deviceId);

    // 2. Merge configurations in priority order (lowest to highest)
    let resolvedConfig = {};

    mergeConfig(resolvedConfig, global);          // Priority 1
    mergeConfig(resolvedConfig, manufacturer);    // Priority 2
    mergeConfig(resolvedConfig, model);           // Priority 3
    mergeConfig(resolvedConfig, carrier);         // Priority 4
    mergeConfig(resolvedConfig, servicePlan);     // Priority 5
    mergeConfig(resolvedConfig, company);         // Priority 6
    mergeConfig(resolvedConfig, device);          // Priority 7 (highest)

    // 3. Apply validation rules
    validateConfiguration(resolvedConfig);

    // 4. Generate .dat file format
    const datFile = generateDatFile(resolvedConfig);

    // 5. Cache resolved configuration
    cacheResolvedConfig(deviceId, resolvedConfig, datFile);

    return { resolvedConfig, datFile };
}

function mergeConfig(target, source) {
    for (const [param, config] of Object.entries(source.parameters)) {
        if (!target[param]) {
            // Parameter not set at higher priority levels - inherit
            target[param] = config;
        } else if (config.merge_strategy === 'append') {
            // Special case: append values (e.g., firewall rules)
            target[param].value = appendValues(target[param].value, config.value);
            target[param].source = source.template_type;
        } else if (config.merge_strategy === 'merge') {
            // Special case: merge objects
            target[param].value = mergeObjects(target[param].value, config.value);
            target[param].source = source.template_type;
        }
        // else: parameter already set at higher priority - skip (override)
    }
}
```

---

## 4. Parameter Categorization by Level

Based on the 694 total parameters analyzed, here's how they map to configuration levels:

### 4.1 Global Level Parameters (ALL 694 parameters)

**All parameters** exist at global level as base configuration. Commonly used ones (320 parameters) include:

**Essential Categories (Priority 1 - 320 parameters):**

<details>
<summary><strong>Administration (10 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `adm_user` | Company, Device | ❌ No | Admin username |
| `adm_passwd` | Company, Device | ❌ No | Admin password (encrypted) |
| `advanced` | Model, Device | ❌ No | Advanced mode flag |
| `console_enable` | Model, Device | ❌ No | Console access |
| `hostname` | Company, Device | ⚠️ Limited | Device identifier |

</details>

<details>
<summary><strong>Network - WAN Interfaces (40+ parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `wan0_proto` | Carrier, Company | ❌ No | WAN0 protocol (none, dhcp, static) |
| `wan0_ip` | Company, Device | ❌ No | WAN0 IP address |
| `wan0_netmask` | Company, Device | ❌ No | WAN0 subnet mask |
| `wan0_gateway` | Company, Device | ❌ No | WAN0 gateway |
| `wan0_mac` | Device | ❌ No | WAN0 MAC (read-only) |
| `wan1_proto` | Carrier, Model | ❌ No | WAN1 protocol (dialup for cellular) |
| `wan1_iface` | Model | ❌ No | WAN1 interface (/dev/ttyUSB3) |
| `wan1_ppp_apn` | Carrier, Service Plan | ❌ No | Cellular APN |
| `wan1_ppp_authen` | Carrier | ❌ No | PPP authentication method |
| `wan1_icmp_interval` | Carrier, Company | ❌ No | ICMP keepalive interval |

</details>

<details>
<summary><strong>Network - LAN Configuration (15 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `lan0_ip` | Company, Device | ❌ No | LAN IP address |
| `lan0_netmask` | Company, Device | ❌ No | LAN subnet mask |
| `lan0_mac` | Device | ❌ No | LAN MAC (read-only) |
| `lan_port1` | Model, Device | ❌ No | LAN port 1 enable |
| `lan_port2` | Model, Device | ❌ No | LAN port 2 enable |
| `lan_port3` | Model, Device | ❌ No | LAN port 3 enable (model-dependent) |
| `lan_port4` | Model, Device | ❌ No | LAN port 4 enable (model-dependent) |

</details>

<details>
<summary><strong>DHCP Server (10 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `dhcpd_enable` | Company, Device | ❌ No | DHCP server on/off |
| `dhcpd_start` | Company, Device | ❌ No | DHCP pool start IP |
| `dhcpd_end` | Company, Device | ❌ No | DHCP pool end IP |
| `dhcpd_lease` | Company, Device | ❌ No | DHCP lease time (minutes) |

</details>

<details>
<summary><strong>DNS Configuration (5 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `dns_static` | Carrier, Company | ❌ No | Static DNS servers (8.8.8.8;1.1.1.1) |
| `dnsrelay_enable` | Company, Device | ❌ No | DNS relay enable |

</details>

<details>
<summary><strong>Firewall (25+ parameters) - CRITICAL</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `fw_acl` | Company, Device | ⚠️ Limited | Firewall ACL rules (100% usage!) |
| `fw_nat` | Company, Device | ❌ No | NAT/Port forwarding (35% usage) |
| `fw_web` | Company, Device | ❌ No | Web filtering rules (35% usage) |
| `fw_anti_dos` | Global, Model | ❌ No | Anti-DoS protection |
| `fw_block_multicast` | Global, Model | ❌ No | Block multicast traffic |

**Format Examples:**
```
fw_acl: 1<4<0.0.0.0/0<<0.0.0.0/0<tcp<53<53<Accept<Policy<0<0<...
fw_nat: 1<1<1<0.0.0.0/0<192.168.1.100<tcp<8080<80<NAT Rule 1<...
fw_web: 1<libertyx.com<1<1<>
```

</details>

<details>
<summary><strong>Remote Monitoring - RMON (15 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `rmon_enable` | Service Plan, Company | ❌ No | RMON on/off (96% usage!) |
| `rmon_server_domain` | Global | ❌ No | RMON server (apcommand.com) |
| `rmon_server_port` | Global | ❌ No | RMON port (8002) |
| `rmon_user` | Company, Device | ❌ No | RMON username |
| `rmon_ipaddr2` | Company, Device | ❌ No | Secondary RMON server |

</details>

<details>
<summary><strong>SSL Tunnels (10 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `ssl_proxy_enable` | Service Plan, Company | ❌ No | SSL proxy on/off (88% usage!) |
| `ssl_server` | Service Plan, Company | ❌ No | SSL tunnel configuration |
| `ssl_proxy_processor_host` | Service Plan | ❌ No | Processor host |
| `ssl_proxy_processor_port` | Service Plan | ❌ No | Processor port |

**Payment Industry Requirement** - 88% of configs have SSL tunnels enabled

</details>

<details>
<summary><strong>Link Backup/Failover (10 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `linkbackup_enable` | Service Plan, Company | ❌ No | Failover enable |
| `wan_main_link` | Carrier, Company | ❌ No | Primary WAN (wan0/wan1) |
| `wan_backup_link` | Carrier, Company | ❌ No | Backup WAN |

</details>

<details>
<summary><strong>MQTT (10 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `mqtt_enable` | Service Plan, Company | ❌ No | MQTT on/off |
| `mqtt_center` | Global | ❌ No | MQTT broker (iot.inhandnetworks.com) |
| `mqtt_keepalive` | Global, Company | ❌ No | Keepalive interval |
| `mqtt_username` | Company | ❌ No | MQTT auth username |
| `mqtt_tls` | Global, Company | ❌ No | Use TLS |

</details>

<details>
<summary><strong>OpenVPN Tunnels (15 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `openvpn_tunnels` | Service Plan, Company | ❌ No | OpenVPN tunnel config (62% usage) |
| `openvpn_tunnel_name` | Service Plan, Company | ❌ No | Tunnel name |
| `openvpn_tunnel_action` | Service Plan, Company | ❌ No | Tunnel action (edit) |
| `openvpn_ovpn1` | Service Plan, Company | ❌ No | OpenVPN config file |
| `openvpn_dns1` | Service Plan, Company | ❌ No | OpenVPN DNS |

</details>

<details>
<summary><strong>WiFi Configuration (25+ parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `wl0_enable` | Company, Device | ✅ Yes | WiFi on/off |
| `wl0_ssid` | Company, Device | ✅ Yes | WiFi network name |
| `wl0_passwd` | Company, Device | ✅ Yes | WiFi password |
| `wl0_encrypt` | Company, Device | ✅ Yes | Encryption type (WPA2) |
| `wl0_auth` | Device | ⚠️ Limited | Auth mode (rarely used) |
| `wl0_radius_ip` | Company | ❌ No | RADIUS server (enterprise WiFi) |
| `wl0_radius_key` | Company | ❌ No | RADIUS shared secret |

**Customer-Manageable:** First 4 parameters only
**Admin-Only:** RADIUS, advanced WiFi settings

</details>

<details>
<summary><strong>Traffic Monitoring (15 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `traffic_enable` | Service Plan | ❌ No | Traffic monitoring on/off |
| `traffic_day_threshold` | Service Plan, Company | ❌ No | Daily data threshold (MB) |
| `traffic_custom_actions` | Service Plan, Company | ❌ No | Custom actions on threshold |

</details>

<details>
<summary><strong>Dual SIM Configuration (10 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `dual_sim_enable` | Model, Service Plan | ❌ No | Dual SIM on/off |
| `dual_sim_main` | Carrier, Company | ❌ No | Primary SIM (1 or 2) |
| `dual_sim_min_csq` | Carrier | ❌ No | Signal quality threshold (expert) |
| `dual_sim_min_conn_time` | Carrier | ❌ No | Min connection time (expert) |

</details>

<details>
<summary><strong>Time/NTP (5 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `ntp_server` | Global, Company | ❌ No | NTP servers (time.nist.gov) |

</details>

<details>
<summary><strong>QoS (10 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `qos_enable` | Service Plan, Company | ❌ No | QoS on/off (usually disabled) |
| `qos_iface` | Service Plan, Company | ❌ No | QoS interface |

</details>

<details>
<summary><strong>SSH/Telnet (10 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `sshd_enable` | Global, Model | ❌ No | SSH access (usually disabled) |
| `telnet_enable` | Global, Model | ❌ No | Telnet access |

</details>

<details>
<summary><strong>SMS Commands (10 parameters)</strong></summary>

| Parameter | Override At | Customer Access | Notes |
|-----------|-------------|-----------------|-------|
| `sms_enable` | Service Plan | ❌ No | SMS commands on/off |
| `sms_rb` | Global | ❌ No | SMS reboot command (REB) |
| `sms_sq` | Global | ❌ No | SMS status query (STA) |
| `sms_switch_main_sim` | Global | ❌ No | SMS SIM switch (SIM) |

</details>

</details>

---

### 4.2 Model-Level Parameters (50-75 parameters)

Parameters that vary by device model:

| Category | Parameters | Example Values | Notes |
|----------|-----------|----------------|-------|
| **Hardware Interfaces** | `wan1_iface`, `lan_port1-4`, `io_chip` | `/dev/ttyUSB3`, `1` | Hardware-specific |
| **Cellular Capabilities** | `wan1_ppp_modem`, `wan1_bridge_mode` | Model-dependent | Modem type, features |
| **WiFi Capabilities** | `wl0_enable`, WiFi params | Model-dependent | Some models lack WiFi |
| **I/O Configuration** | `digitalio_config`, `io_chip` | `1,0,0;1,0,0;` | Digital I/O availability |
| **Port Availability** | `lan_port3`, `lan_port4` | `0/1` | Some models have 2 ports, others 4 |

---

### 4.3 Carrier-Level Parameters (20-30 parameters)

Parameters specific to cellular carriers:

| Category | Parameters | Verizon Example | AT&T Example | T-Mobile Example |
|----------|-----------|----------------|-------------|------------------|
| **APN Settings** | `wan1_ppp_apn` | `Matrxatm.gw12.vzwentp` | `[AT&T APN]` | `[T-Mobile APN]` |
| **Authentication** | `wan1_ppp_authen` | `chap` | Carrier-specific | Carrier-specific |
| **DNS Servers** | `dns_static` | Carrier-preferred | Carrier-preferred | Carrier-preferred |
| **Network Selection** | `wan1_ppp_operator` | `auto` | `auto` | `auto` |
| **Frequency Bands** | `gsm_wcdma_band_config`, `lte_band_config` | Carrier-optimized | Carrier-optimized | Carrier-optimized |

---

### 4.4 Service Plan-Level Parameters (25-40 parameters)

Parameters tied to billing/service plans:

| Category | Parameters | Basic Plan | Premium Plan | Enterprise Plan |
|----------|-----------|------------|-------------|----------------|
| **Data Limits** | `traffic_day_threshold` | 1024 MB | 3584 MB | Unlimited |
| **Features** | `ssl_proxy_enable` | ✅ Enabled | ✅ Enabled | ✅ Enabled |
| **VPN** | `openvpn_tunnels` | ❌ Disabled | ✅ Enabled | ✅ Enabled |
| **QoS** | `qos_enable`, `qos_settings` | Standard | Priority | Custom |
| **Monitoring** | `rmon_enable` | ✅ Enabled | ✅ Enabled | ✅ Enhanced |
| **Backup Link** | `linkbackup_enable` | ❌ Disabled | ✅ Enabled | ✅ Enabled |

---

### 4.5 Company-Level Parameters (10-50 parameters, variable)

Company-specific custom configurations:

| Parameter Category | Example Company | Parameters | Values |
|-------------------|----------------|------------|--------|
| **Custom NAT Rules** | Altec Enterprises | `fw_nat` | Printer translation rules |
| **Web Filtering** | LibertyX | `fw_web` | `1<libertyx.com<1<1<>` |
| **Custom Firewall** | Various | `fw_acl` | Company-specific ACL rules |
| **Custom DHCP** | Various | `dhcpd_start/end` | Custom IP ranges |
| **Custom DNS** | Enterprise clients | `dns_static` | Internal DNS servers |
| **Branding** | White-label clients | `oem_name`, `model_name` | Custom branding (encrypted) |

**Important:** Highly variable - some companies have 10 custom parameters, others have 50+

---

### 4.6 Device-Level Parameters

#### Customer-Manageable (10-15 parameters)
```
WiFi Settings:
- wl0_enable (on/off)
- wl0_ssid (network name)
- wl0_passwd (password)
- wl0_encrypt (encryption type)

Basic Firewall:
- fw_acl (simplified rules through UI)

Device Identity:
- hostname (device name)
```

#### Admin-Only (All 694 parameters)
Admins can override ANY parameter at device level for troubleshooting or special cases

---

### 4.7 Parameters by Usage Frequency

Based on analysis of 26 configuration files:

| Usage Category | Count | UI Priority | Configuration Levels |
|---------------|-------|-------------|---------------------|
| **Commonly Used** (>80% files) | 320 | Essential (Priority 1) | All levels |
| **Sometimes Used** (20-80% files) | 41 | Advanced (Priority 2) | Company, Device mainly |
| **Mostly Empty** (<20% files) | 31 | Expert Mode (Priority 3) | Device, Company (edge cases) |
| **Always Empty** (0% usage) | 302 | Exclude from UI | N/A - not used |

**Development Impact:**
- Focus on **320 commonly-used parameters** for 95%+ coverage
- Exclude **302 never-used parameters** (44% complexity reduction)

---

## 5. Validation Rules

### 5.1 Parameter-Level Validation

Each parameter has validation rules defined in `config_parameters` table:

| Validation Type | Parameters | Rule Example | Error Message |
|----------------|-----------|--------------|---------------|
| **Required** | `lan0_ip`, `wan1_ppp_apn` | `is_required=true` | "LAN IP address is required" |
| **Format - IP Address** | `lan0_ip`, `wan0_ip`, `dns_static` | `validation_regex=^(\d{1,3}\.){3}\d{1,3}$` | "Invalid IP address format" |
| **Format - MAC Address** | `lan0_mac`, `wan0_mac` | `validation_regex=^([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}$` | "Invalid MAC address format" |
| **Range - Port Numbers** | `rmon_server_port`, `http_api_port` | `min_value=1, max_value=65535` | "Port must be between 1-65535" |
| **Range - Integer** | `dhcpd_lease`, `wan1_icmp_interval` | `min_value=1` | "Value must be positive" |
| **Enum - Boolean** | `wl0_enable`, `ssl_proxy_enable` | `data_type=boolean` | "Value must be 0 or 1" |
| **Enum - Protocol** | `wan0_proto` | `allowed_values=['none','dhcp','static','dialup']` | "Invalid protocol" |
| **Length - String** | `hostname`, `wl0_ssid` | `max_length=255` | "Hostname too long" |
| **Encrypted** | `adm_passwd`, `wl0_passwd` | `is_encrypted=true` | Auto-encrypt on save |

---

### 5.2 Cross-Parameter Validation

Some parameters depend on others:

| Rule Type | Condition | Validation | Example |
|-----------|-----------|-----------|---------|
| **Conditional Required** | If `dhcpd_enable=1` | Then `dhcpd_start`, `dhcpd_end`, `dhcpd_lease` required | DHCP enabled requires pool config |
| **Conditional Required** | If `wl0_enable=1` | Then `wl0_ssid`, `wl0_passwd` required | WiFi enabled requires SSID/password |
| **Range Validation** | `dhcpd_start` < `dhcpd_end` | IP range must be valid | Start IP must be before end IP |
| **Subnet Validation** | `dhcpd_start`, `dhcpd_end` must be in `lan0_ip` subnet | DHCP pool must match LAN subnet | Prevent misconfiguration |
| **Mutual Exclusion** | If `wan0_proto=none` | Then `linkbackup_enable` cannot use `wan0` | Can't use disabled WAN as backup |
| **Dependency** | If `ssl_proxy_enable=1` | Then `ssl_server` required | SSL proxy requires server config |

---

### 5.3 Level-Specific Validation

| Level | Validation Rules | Example |
|-------|-----------------|---------|
| **Global** | Must contain all 694 parameters with defaults | System validates on startup |
| **Model** | Can only override parameters applicable to hardware | Cannot set carrier APN at model level |
| **Carrier** | Can only override network/cellular parameters | Cannot set WiFi SSID at carrier level |
| **Service Plan** | Can only override feature flags and limits | Cannot set device hostname at plan level |
| **Company** | Cannot override read-only device parameters | Cannot set MAC address at company level |
| **Device** | Customers can only modify allowed parameters | Customers cannot modify cellular APN |

---

### 5.4 Read-Only Parameters

Some parameters are read-only (device-assigned):

| Parameter | Level | Why Read-Only | Who Can Modify |
|-----------|-------|---------------|----------------|
| `lan0_mac` | Device | Hardware-assigned | Nobody (device sets) |
| `wan0_mac` | Device | Hardware-assigned | Nobody (device sets) |
| `serialnum` | Device | Factory-assigned | Nobody (device sets) |
| `hardware_ready` | Device | Hardware status | System only |

---

### 5.5 Security Validation

| Rule | Parameters | Validation |
|------|-----------|-----------|
| **Password Strength** | `adm_passwd`, `wl0_passwd` | Min 8 chars, complexity requirements |
| **Encryption Required** | `adm_passwd`, `wl0_passwd` | Auto-encrypt using `$AES$` prefix |
| **Access Control** | Admin-only parameters | Role-based checks before allow edit |
| **Firewall Rule Validation** | `fw_acl`, `fw_nat` | Validate rule syntax, prevent injection |
| **IP Spoofing Prevention** | `lan0_ip`, `wan0_ip` | Prevent private IPs on WAN (where applicable) |

---

## 6. Override Rules & Conflict Resolution

### 6.1 Override Permission Matrix

| Parameter Category | Global | Manufacturer | Model | Carrier | Service Plan | Company | Device (Admin) | Device (Customer) |
|-------------------|--------|--------------|-------|---------|-------------|---------|---------------|-------------------|
| **Administration** | ✅ Set | ❌ | ❌ | ❌ | ❌ | ✅ Override | ✅ Override | ❌ |
| **Hardware Settings** | ✅ Set | ✅ Override | ✅ Override | ❌ | ❌ | ❌ | ✅ Override | ❌ |
| **Cellular/APN** | ✅ Set | ❌ | ✅ Override | ✅ Override | ✅ Override | ❌ | ✅ Override | ❌ |
| **LAN Configuration** | ✅ Set | ❌ | ❌ | ❌ | ❌ | ✅ Override | ✅ Override | ❌ |
| **DHCP Settings** | ✅ Set | ❌ | ❌ | ❌ | ❌ | ✅ Override | ✅ Override | ❌ |
| **DNS Settings** | ✅ Set | ❌ | ❌ | ✅ Override | ❌ | ✅ Override | ✅ Override | ❌ |
| **Firewall Rules** | ✅ Set | ❌ | ❌ | ❌ | ❌ | ✅ Override | ✅ Override | ⚠️ Limited |
| **WiFi Settings** | ✅ Set | ❌ | ✅ Override | ❌ | ❌ | ✅ Override | ✅ Override | ✅ Yes (basic) |
| **SSL Tunnels** | ✅ Set | ❌ | ❌ | ❌ | ✅ Override | ✅ Override | ✅ Override | ❌ |
| **Monitoring (RMON)** | ✅ Set | ❌ | ❌ | ❌ | ✅ Override | ✅ Override | ✅ Override | ❌ |
| **Feature Flags** | ✅ Set | ❌ | ❌ | ❌ | ✅ Override | ✅ Override | ✅ Override | ❌ |

**Legend:**
- ✅ Can set/override
- ❌ Cannot set/override
- ⚠️ Limited/restricted access

---

### 6.2 Conflict Resolution Rules

#### Rule 1: Higher Priority Wins (Default Behavior)
```
Global:    lan0_ip = 192.168.1.90
Company:   lan0_ip = 192.168.2.90
Device:    [not set]

Result:    Device gets 192.168.2.90 (Company override)
```

#### Rule 2: Complete Override (Most Parameters)
```
Global:    hostname = "InHandRouter"
Device:    hostname = "DC_22_06012023"

Result:    Device hostname = "DC_22_06012023" (completely replaces global)
```

#### Rule 3: Merge Strategy (Firewall Rules, Lists)
```
Global:    fw_acl = [Basic security rules]
Company:   fw_acl = [Company-specific rules]
Device:    fw_acl = [Device-specific rules]

Result:    fw_acl = [Device rules] + [Company rules] + [Global rules]
           (Merged with device rules applied first)
```

#### Rule 4: Append Strategy (Some Arrays)
```
Global:    dns_static = "8.8.8.8;1.1.1.1"
Carrier:   dns_static = "10.0.0.1;10.0.0.2"

Result:    dns_static = "10.0.0.1;10.0.0.2;8.8.8.8;1.1.1.1"
           (Carrier DNS first, fallback to global)

           OR (depending on requirements):
           dns_static = "10.0.0.1;10.0.0.2"
           (Complete override - to be decided)
```

#### Rule 5: Validation-Based Conflict Prevention
```
Company:   fw_acl = [Rules allowing port 80]
Device:    fw_acl = [Rules blocking all ports]

Conflict:  Device rules would block company requirements

Resolution Options:
1. Warn user: "Device rules conflict with company policy"
2. Prevent save: "Cannot block ports required by company policy"
3. Merge intelligently: Device rules + Company mandatory rules
```

---

### 6.3 Mandatory vs. Optional Overrides

Some parameters at higher levels can be marked as "mandatory" (cannot be overridden):

| Scenario | Parameter | Level | Mandatory? | Reason |
|----------|-----------|-------|-----------|--------|
| **Security Policy** | `ssl_proxy_enable` | Service Plan | ✅ Yes | Payment industry requirement |
| **Monitoring** | `rmon_enable` | Service Plan | ✅ Yes | Business requirement |
| **Compliance** | Certain firewall rules | Company | ✅ Yes | Company security policy |
| **Hardware** | `wan1_iface` | Model | ✅ Yes | Hardware-specific, cannot change |
| **Customizable** | `hostname` | Any level | ❌ No | Can be overridden at any level |
| **Customizable** | `dhcpd_lease` | Any level | ❌ No | Can be overridden |

**Implementation:**
```json
{
    "ssl_proxy_enable": {
        "value": 1,
        "source": "service_plan",
        "override_allowed": false,
        "mandatory": true,
        "reason": "Required by payment industry compliance"
    }
}
```

---

## 7. Implementation Considerations

### 7.1 Migration from Current System

#### Current System
- Configuration files stored as `.dat` files
- File-based structure (key=value pairs)
- Located in various directories
- Manual file management

#### Migration Strategy

**Phase 1: Import Existing Configs**
1. Parse all existing `.dat` files
2. Import into `config_templates` table
3. Categorize by template type (global, company, device, etc.)
4. Maintain backward compatibility - keep generating `.dat` files

**Phase 2: Dual System Operation**
1. New configs created through UI → stored in database → export to `.dat`
2. Old configs remain as files → can be imported to database
3. System reads from database first, falls back to files
4. Gradual migration of configs from files to database

**Phase 3: Full Database Operation**
1. All configs in database
2. `.dat` files generated on-demand for device consumption
3. Files used only for device delivery, not as source of truth

---

### 7.2 Performance Optimization

#### Configuration Caching

```sql
-- Compiled configuration cached in device_configs table
compiled_config JSON
compiled_at TIMESTAMP
needs_recompile BOOLEAN
```

**Cache Invalidation Rules:**
- When any template in hierarchy is updated, set `needs_recompile = TRUE`
- Recompile on-demand or via background job
- Cache expires after 24 hours (configurable)

#### Indexing Strategy
```sql
INDEX idx_template_type (template_type)
INDEX idx_entity_id (entity_id)
INDEX idx_needs_recompile (needs_recompile)
INDEX idx_parameter (parameter_name)
```

#### Query Optimization
- Load only necessary templates for device
- Use JSON functions for parameter extraction
- Pre-compile commonly accessed configs

---

### 7.3 Backward Compatibility

#### Generate .dat Files from Database

```php
function generateDatFile($deviceId) {
    // 1. Resolve full configuration from database
    $config = resolveDeviceConfiguration($deviceId);

    // 2. Convert to .dat format
    $datContent = "";
    foreach ($config['parameters'] as $param => $data) {
        if (!empty($data['value'])) {
            $datContent .= "$param={$data['value']}\n";
        }
    }

    // 3. Save to file or return as string
    return $datContent;
}
```

#### Support Old .dat File Imports

```php
function importDatFile($filePath, $templateType, $entityId) {
    // Parse .dat file
    $params = parseDatFile($filePath);

    // Create config template
    $template = createConfigTemplate($templateType, $entityId);

    // Add parameters
    foreach ($params as $name => $value) {
        addConfigOverride($template->id, $name, $value);
    }

    return $template;
}
```

---

### 7.4 Security Considerations

#### Role-Based Access Control (RBAC)

| Role | Global | Manufacturer | Model | Carrier | Service Plan | Company | Device |
|------|--------|-------------|-------|---------|-------------|---------|--------|
| **Super Admin** | ✅ Full | ✅ Full | ✅ Full | ✅ Full | ✅ Full | ✅ Full | ✅ Full |
| **System Admin** | ✅ View | ✅ Full | ✅ Full | ✅ Full | ❌ | ✅ Full | ✅ Full |
| **Sales/Support** | ❌ | ❌ | ❌ | ❌ | ✅ View | ✅ Full | ✅ Full |
| **Network Ops** | ❌ | ❌ | ✅ View | ✅ Full | ❌ | ❌ | ✅ View |
| **Customer** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ⚠️ Limited |

#### Audit Logging
- Log ALL configuration changes to `config_audit_log`
- Track: who, what, when, old value, new value, reason
- Immutable audit log (no deletions)
- Retention: minimum 7 years (compliance requirement)

#### Encryption
- Passwords encrypted using `$AES$` prefix
- Store encryption keys securely (not in database)
- Use CakePHP's encryption utilities

---

### 7.5 UI/UX Considerations

#### Progressive Disclosure
```
┌─────────────────────────────────────────┐
│ Device Configuration - IR915L           │
├─────────────────────────────────────────┤
│ ✅ Essential Settings (always visible)  │
│   - Hostname                            │
│   - LAN IP Address                      │
│   - DHCP Settings                       │
│                                         │
│ 🔧 Advanced Settings (collapsible)      │
│   ▼ Show Advanced Settings              │
│                                         │
│ 🔬 Expert Mode (toggle required)        │
│   □ Enable Expert Mode                  │
└─────────────────────────────────────────┘
```

#### Configuration Inheritance Display
```
┌────────────────────────────────────────────┐
│ Parameter: dns_static                      │
├────────────────────────────────────────────┤
│ Current Value: 10.0.0.1;10.0.0.2          │
│ Source: Company Configuration              │
│                                            │
│ Inheritance Chain:                         │
│ ❌ Global:    8.8.8.8;1.1.1.1             │
│ ❌ Carrier:   [not set]                    │
│ ✅ Company:   10.0.0.1;10.0.0.2 ← Active  │
│ ❌ Device:    [not set]                    │
│                                            │
│ [Override at Device Level]                 │
└────────────────────────────────────────────┘
```

---

### 7.6 Testing Strategy

#### Unit Tests
- Parameter validation rules
- Configuration merge algorithm
- Override priority logic
- .dat file generation

#### Integration Tests
- Full config resolution for test devices
- Import/export .dat files
- Multi-level override scenarios
- Conflict resolution

#### End-to-End Tests
- Create config at each level
- Verify device receives correct resolved config
- Test customer vs admin access restrictions
- Test migration from old system

---

## Summary & Next Steps

### Key Decisions Needed (Discovery Session)

1. **Merge vs. Override Strategy**
   - Which parameters should merge (firewall rules)?
   - Which should completely override (simple values)?
   - Define merge strategy for each complex parameter

2. **Mandatory Override Rules**
   - Which parameters cannot be overridden (security/compliance)?
   - At what levels can overrides be marked mandatory?

3. **Service Plan Integration**
   - How tightly integrated with billing system?
   - Can service plans be changed without config migration?

4. **Customer Access Scope**
   - Final list of customer-manageable parameters
   - UI for customer config management (simplified vs full)

5. **Migration Timeline**
   - Phased approach acceptable?
   - How long can old and new systems run in parallel?
   - Migration testing strategy

### Documentation Deliverables

After discovery session, create:
- [ ] Detailed parameter-level mapping (all 694 parameters)
- [ ] UI wireframes for config management screens
- [ ] API specifications for config CRUD operations
- [ ] Migration plan with rollback strategy
- [ ] Test plan and acceptance criteria

---

**End of Document**
