# Configuration Comparison: Altech Custom vs Standard Base

## Overview

**Base Configuration:** VZW_22_01272025.dat
**Custom Configuration:** Vzw_22_Altech_05152024.dat
**Comparison Date:** October 15, 2025

## Summary Statistics

| Metric | Count |
|--------|-------|
| Base parameters | 658 |
| Custom parameters | 647 |
| Parameters added | 3 |
| Parameters removed | 14 |
| Parameters modified | 25 |
| Parameters unchanged | 619 |

**Total Differences:** 42 parameters (6.4% of base configuration)

## Key Findings

### Critical Changes

1. **Network Configuration Changes**
   - DHCP pool drastically reduced (from 154 IPs to 3 IPs)
   - Port forwarding rules added (NAT configuration)
   - DNS server changed

2. **Feature Disabling**
   - Advanced mode disabled
   - MQTT disabled
   - SMS functionality disabled
   - Backup SIM policy removed

3. **Device Identity**
   - Different MAC addresses
   - Different hostname
   - Different device ID

## Detailed Analysis

### 1. ADDED PARAMETERS (3)

These parameters exist in custom but not in base:

```
http_api_description=
http_api_src=
serialnum=RF3022221125553
```

**Analysis:**
- `serialnum`: Specific device serial number for Altech customer
- `http_api_*`: HTTP API configuration (both empty, possibly reserved for future use)

### 2. REMOVED PARAMETERS (14)

These parameters exist in base but removed in custom:

#### Backup SIM Policy (3 parameters)
```
backup_sim_policy_enable=0
backup_sim_policy_using_time=0
backup_sim_policy_revert_day=1
```
**Impact:** Dual SIM failover functionality removed

#### MQTT Related (2 parameters)
```
mqtt_keepalive=60
mqtt_tls=1
```
**Impact:** MQTT connection settings removed (MQTT disabled anyway)

#### SMS Related (3 parameters)
```
sms_apn=APN
sms_network_provider=NET
sms_switch_main_sim=SIM
```
**Impact:** SMS configuration templates removed

#### Monitoring/Remote Management (3 parameters)
```
portal_enable=
rmon_io_value=0
rmon_tls_sni=
```
**Impact:** Remote monitoring features simplified

#### Other (3 parameters)
```
iodigital_input_data=fc
model_name=$AES$B1E080FCB23738E4D10401164A95D0C7
wan1_icmp_detect_enable=0
```
**Impact:** Digital I/O data and model encryption removed

### 3. MODIFIED PARAMETERS (25)

#### Network Configuration Changes

##### DHCP Server - CRITICAL CHANGE
```
dhcpd_start:   192.168.1.100 → 192.168.1.101
dhcpd_end:     192.168.1.254 → 192.168.1.103
dhcpd_lease:   60 minutes → 600 minutes (10 hours)
```
**Impact:**
- **MAJOR**: DHCP pool reduced from 154 IPs to just 3 IPs (.101, .102, .103)
- Lease time increased 10x (reduces DHCP traffic)
- Suggests only 3 specific devices need DHCP

##### DNS Configuration
```
dns_static: 8.8.8.8;1.1.1.1 → 8.8.8.8;198.224.183.135
```
**Impact:**
- Removed Cloudflare DNS (1.1.1.1)
- Added custom DNS server (198.224.183.135)
- Likely internal/ISP DNS server

##### MAC Addresses - Device-Specific
```
lan0_mac: 00:18:05:2D:86:7F → 00:18:05:21:0D:F9
wan0_mac: 00:18:05:2D:85:43 → 00:18:05:21:0D:CF
```
**Impact:** Different physical device hardware

##### Hostname
```
hostname: VZW_22_01272025 → Vzw_22_Altech_05152024
```
**Impact:** Customer-specific identifier

#### Firewall & Security Changes

##### Firewall ACL - Modified Rule
```
Base:   fw_acl=1<4<0.0.0.0/0<<0.0.0.0/0<<1<1<>
Custom: fw_acl=1<4<0.0.0.0/0<<<<1<1<>
```
**Analysis:**
- Base: Full wildcard rule (source and dest 0.0.0.0/0)
- Custom: Removed source and dest IP specifications (empty fields)
- Action remains: Allow (1)
- Enabled: Yes (1)

##### NAT/Port Forwarding - MAJOR ADDITION
```
Base:   fw_nat=
Custom: fw_nat=1<1<3<0.0.0.0/0<<0.0.0.0/0<10101<<192.168.1.101<80<1<>
              1<1<3<0.0.0.0/0<<0.0.0.0/0<10102<<192.168.1.102<80<1<>
              1<1<3<0.0.0.0/0<<0.0.0.0/0<10103<<192.168.1.103<80<1<>
              1<1<3<0.0.0.0/0<<0.0.0.0/0<10141<<192.168.1.101<443<1<>
              1<1<3<0.0.0.0/0<<0.0.0.0/0<10142<<192.168.1.102<443<1<>
              1<1<3<0.0.0.0/0<<0.0.0.0/0<10143<<192.168.1.103<443<1<>
```

**Detailed NAT Rules:**

| External Port | Internal IP | Internal Port | Protocol | Purpose |
|---------------|-------------|---------------|----------|---------|
| 10101 | 192.168.1.101 | 80 | TCP | HTTP to Device 1 |
| 10102 | 192.168.1.102 | 80 | TCP | HTTP to Device 2 |
| 10103 | 192.168.1.103 | 80 | TCP | HTTP to Device 3 |
| 10141 | 192.168.1.101 | 443 | TCP | HTTPS to Device 1 |
| 10142 | 192.168.1.102 | 443 | TCP | HTTPS to Device 2 |
| 10143 | 192.168.1.103 | 443 | TCP | HTTPS to Device 3 |

**Impact:**
- Exposes 3 internal devices for external access
- Each device accessible on unique external ports
- Matches DHCP pool (3 devices at .101, .102, .103)
- Both HTTP and HTTPS forwarded per device

#### SSL Tunnel Configuration - Simplified

**Base:** 25 SSL tunnel definitions
**Custom:** 13 SSL tunnel definitions

**Removed tunnels:**
```
208.224.248.160:1440 → Changed to atm1.switchcommerce.net:1440
192.168.1.100:18456
192.168.1.123:80
EMV.SIBISYSTEMS.Com:9056
pos.tnsi.com:5162
umcstunnellenexa.dyndns.org:49154
EFTDEBITATM.FNFIS.COM:443
umcstunnellenexa.dyndns.org:2097
umcstunnellenexa.dyndns.org:48154
pos.tnsi.com:5550
192.168.1.100:1000
192.168.1.100:5500
192.168.1.100:5900
192.168.1.111:22
```

**Added tunnel:**
```
192.168.1.102:80 (Local web server on device 2)
```

**Impact:**
- Removed 12 payment processing tunnels
- Removed internal service tunnels
- Simplified to core ATM connectivity
- Added tunnel for local device management

#### MQTT Settings - Disabled

```
mqtt_enable:   1 → 0
mqtt_center:   iot.inhandnetworks.com → (empty)
mqtt_username: service@wirelessatmstore.com → (empty)
```
**Impact:** InHand IoT platform integration disabled

#### SMS Settings - Disabled

```
sms_enable: 1 → 0
sms_rb:     REB → (empty)
sms_sq:     STA → (empty)
```
**Impact:** SMS remote control commands disabled

#### System Settings

```
advanced: 1 → 0
```
**Impact:** Advanced features/UI disabled for this customer

#### Alarms

```
alarm_output_options: cli,out-dm,out-rmon, → cli,out-dm,
```
**Impact:** Remote monitoring (rmon) alarms removed

#### NTP Servers

```
Base:   10.4.6.30;time.nist.gov;time.google.com
Custom: 10.4.6.30;74.118.247.208;time.google.com
```
**Impact:**
- Removed NIST NTP server
- Added custom NTP server (74.118.247.208)
- Kept internal NTP (10.4.6.30) and Google

#### Device Management

```
ovdp_device_id: 302274247 → 302125553
oem_name:       $AES$AB5416F1268EBBD2E785E5AB90490B4B → (empty)
```
**Impact:**
- Different device registration ID
- OEM branding removed

#### WAN Monitoring

```
wan1_icmp_interval: 3600 seconds (1 hour) → 6000 seconds (1.67 hours)
```
**Impact:** Less frequent WAN connectivity checks (conserves data)

#### Traffic Monitoring

```
traffic_day_threshold: 3584 → 3
traffic_day_unit:      1 (MB) → 2 (GB)
```
**Analysis:**
- Base: 3584 MB = 3.5 GB daily threshold
- Custom: 3 GB daily threshold
- **Result:** Slightly lower threshold (3 GB vs 3.5 GB)

```
rmon_advance_cfg: _wan1_imei,_wan1_iccid,_wan1_sinr
                → _wan1_imei,_wan1_iccid,_traffic_daily_value,_traffic_monthly_value,_traffic_monthly2_value
```
**Impact:**
- Removed: Signal quality metric (_wan1_sinr)
- Added: Traffic usage metrics (daily and monthly tracking)
- Focus shifted from signal to data consumption

## Use Case Analysis

Based on the configuration changes, this custom configuration is designed for:

### Target Deployment
- **ATM/Payment Terminal Network**
- **3 Fixed Devices** (ATMs or terminals)
- **Remote Management** enabled via port forwarding
- **Simplified Feature Set** for reliability

### Network Architecture
```
Internet (WAN)
    ↓
Router (Altech Configuration)
    ↓
LAN (192.168.1.0/24)
    ├─ 192.168.1.101 - ATM/Terminal 1 (HTTP:10101, HTTPS:10141)
    ├─ 192.168.1.102 - ATM/Terminal 2 (HTTP:10102, HTTPS:10142)
    └─ 192.168.1.103 - ATM/Terminal 3 (HTTP:10103, HTTPS:10143)
```

### Key Design Decisions

1. **Minimal DHCP Pool**
   - Only 3 IPs allocated
   - Suggests static/reserved assignments
   - Reduces DHCP overhead

2. **Port Forwarding Strategy**
   - Each device gets unique external ports
   - Both HTTP and HTTPS exposed
   - Enables remote management/monitoring

3. **Simplified Connectivity**
   - Reduced SSL tunnels (13 vs 25)
   - Focused on core ATM processing
   - Removed redundant payment gateways

4. **Disabled Features**
   - No SMS commands
   - No MQTT telemetry
   - No advanced UI
   - Focus on stability over features

5. **Traffic Monitoring**
   - Track data consumption (not signal)
   - 3GB daily limit
   - Monthly tracking enabled

## Security Considerations

### Positive Changes
- Reduced attack surface (fewer tunnels)
- Simplified configuration (less complexity)
- Disabled unnecessary features

### Concerns
- **6 ports exposed** to internet (HTTP/HTTPS for 3 devices)
- No apparent additional firewall restrictions
- Port forwarding to internal devices could be exploited
- Consider: VPN instead of direct port forwarding

### Recommendations
1. Add firewall rules to restrict source IPs for port forwarding
2. Consider VPN for remote access instead of port forwarding
3. Implement strong authentication on forwarded HTTP/HTTPS services
4. Enable HTTPS-only (disable HTTP forwarding)
5. Monitor traffic to forwarded ports for anomalies

## Compliance Impact

### Payment Card Industry (PCI DSS)
If these are payment devices:
- **Requirement 1.3**: Exposed ports should be documented and justified
- **Requirement 2.2**: Disabled unnecessary services ✓
- **Requirement 8.2**: Ensure strong auth on exposed web services
- **Requirement 10.2**: Log access to forwarded ports

## Migration Checklist

If deploying this configuration to a new device:

- [ ] Update `serialnum` to device serial
- [ ] Update `hostname` to customer/location identifier
- [ ] Update `lan0_mac` and `wan0_mac` to device MACs
- [ ] Update `ovdp_device_id` for device registration
- [ ] Verify DHCP pool matches actual devices
- [ ] Confirm port forwarding rules match device IPs
- [ ] Test SSL tunnel connectivity
- [ ] Verify DNS resolution
- [ ] Validate NTP synchronization
- [ ] Test traffic threshold alerts
- [ ] Document exposed ports for security audit

## Conclusion

The Altech custom configuration represents a **purpose-built, simplified deployment** for a 3-device ATM/payment terminal network. Key characteristics:

- **Minimalist approach**: Removed 14 parameters, disabled several features
- **Device-specific**: Tight DHCP pool, specific port forwarding
- **Remote access enabled**: 6 external ports for device management
- **Stability-focused**: Fewer features, longer monitoring intervals
- **Traffic-conscious**: Data usage tracking prioritized

This configuration would be ideal for a fixed-location deployment with known devices requiring remote management capabilities, but may require additional security hardening before production deployment in PCI-compliant environments.
