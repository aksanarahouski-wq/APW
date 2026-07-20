# Verizon I-22 Company Override Analysis

**Date:** July 19, 2026
**Base Rule:** `VZW_22_3wayrule.dat` (661 parameters)
**Companies Analyzed:** Altech, Cord, Baum

Analysis compares each company's full config file against the base 3-way rule to identify which parameters are actually overridden. Device-specific parameters (hostname, MAC addresses, device IDs, serial numbers) are excluded.

---

## Override Counts

| Company | Total Overridden Params | Changed Value | Added | Removed |
|---------|------------------------|---------------|-------|---------|
| **Altech** | 35 | 20 | 2 | 13 |
| **Cord** | 15 | 8 | 2 | 5 |
| **Baum** | 31 | 21 | 3 | 6 |

---

## Parameter Overlap Between Companies

### All 3 companies override (4 params)
- `advanced` (1 -> 0)
- `fw_acl` (custom firewall rules — different values per company)
- `http_api_description` (added)
- `http_api_src` (added)

### Altech + Cord only (9 params)
- `dhcpd_lease`, `portal_enable`, `rmon_tls_sni`
- SMS group: `sms_apn`, `sms_enable`, `sms_network_provider`, `sms_rb`, `sms_sq`, `sms_switch_main_sim`

### Altech + Baum only (13 params)
- MQTT group: `mqtt_center`, `mqtt_enable`, `mqtt_keepalive`, `mqtt_tls`, `mqtt_username`
- `fw_nat`, `iodigital_input_data`, `ntp_server`, `rmon_advance_cfg`
- `ssl_server`, `traffic_day_threshold`, `traffic_day_unit`, `wan1_icmp_interval`

### Cord + Baum only (0 params)
No parameters are shared exclusively between Cord and Baum.

### Unique to Altech (9 params)
- `alarm_output_options`, `backup_sim_policy_enable`, `backup_sim_policy_revert_day`, `backup_sim_policy_using_time`
- `dhcpd_end`, `dhcpd_start`, `dns_static`, `rmon_io_value`, `wan1_icmp_detect_enable`

### Unique to Cord (2 params)
- `wl0_radius_ip`, `wl0_radius_key`

### Unique to Baum (14 params)
- `alarm_input`, `alarm_output`, `console_enable`, `cron_advanced`, `cron_rb_enable`, `cron_rb_time`
- `dtu1_enable`, `gsm_wcdma_band_config`, `io_chip`, `lan0_iface`, `lte_band_config`
- `openvpn_ovpn1`, `redial_reboot`, `rmon_user`

---

## Key Findings

### 1. Overrides are not small
Altech touches 35 parameters, Baum touches 31. This is not "3-5 parameters per company" — company configs can modify significant portions of the base rule.

### 2. Same parameters, different values
All three companies override `fw_acl`, but each has completely different firewall rules (Cord has named ACLs for RMS servers, Baum has a full allow/block list with a default-deny rule). Override sets for firewall rules cannot be shared across these companies — the parameter key overlaps but the values are unique.

### 3. Natural parameter clusters
Parameters change together in logical groups:
- **MQTT group:** `mqtt_enable`, `mqtt_center`, `mqtt_username`, `mqtt_keepalive`, `mqtt_tls` (5 params)
- **SMS group:** `sms_enable`, `sms_apn`, `sms_network_provider`, `sms_rb`, `sms_sq`, `sms_switch_main_sim` (6 params)
- **Traffic thresholds:** `traffic_day_threshold`, `traffic_day_unit` (2 params)
- **Firewall:** `fw_acl`, `fw_nat` (2 params, but very large values)

### 4. The overlap problem is real
If two companies override the same parameter key with different values, they cannot share an override set for that parameter. Shareable override sets are limited to cases where multiple companies need the exact same values (e.g., all companies disabling MQTT the same way: `mqtt_enable=0`, `mqtt_center=`, `mqtt_username=`).

### 5. Some overrides are really removals
Several "overrides" are parameters present in the base that are absent from the company file (e.g., Altech removes `backup_sim_policy_*`, Cord removes `sms_apn`). The config engine needs to handle "parameter removal" as a valid override action, not just value changes.

---

## Implications for Override Design

- The two-layer model (base rule + company overrides) is validated by this data. The base holds ~660 shared params; companies override 15-35 on top.
- Override sets that CAN be shared are ones with identical values (e.g., "disable MQTT" = same 5 params with same values for Altech and Baum).
- Override sets that CANNOT be shared are ones where the parameter is common but values differ (e.g., `fw_acl` — every company has unique firewall rules).
- Conflict detection must check parameter key overlap, not value equality. Two override sets assigned to the same company cannot both set `fw_acl`, even if the values happen to be the same.

---

## Detailed Override Values

### Altech — 35 overridden parameters

| Parameter | Base Value | Altech Value | Notes |
|-----------|-----------|--------------|-------|
| `advanced` | 1 | 0 | |
| `alarm_output_options` | cli,out-dm,out-rmon, | cli,out-dm, | Removed rmon output |
| `backup_sim_policy_enable` | 0 | *(removed)* | |
| `backup_sim_policy_revert_day` | 1 | *(removed)* | |
| `backup_sim_policy_using_time` | 0 | *(removed)* | |
| `dhcpd_end` | 192.168.1.254 | 192.168.1.103 | Narrower DHCP range |
| `dhcpd_lease` | 60 | 600 | 10x longer lease |
| `dhcpd_start` | 192.168.1.100 | 192.168.1.101 | |
| `dns_static` | 8.8.8.8;1.1.1.1 | 8.8.8.8;198.224.183.135 | Custom secondary DNS |
| `fw_acl` | *(default allow all)* | *(minimal ACL)* | Simplified firewall |
| `fw_nat` | *(empty)* | *(port forwarding rules)* | 6 NAT rules for ports 80/443 |
| `http_api_description` | *(absent)* | *(empty)* | Added |
| `http_api_src` | *(absent)* | *(empty)* | Added |
| `iodigital_input_data` | fc | *(removed)* | |
| `mqtt_center` | iot.inhandnetworks.com | *(empty)* | MQTT disabled |
| `mqtt_enable` | 1 | 0 | MQTT disabled |
| `mqtt_keepalive` | 60 | *(removed)* | |
| `mqtt_tls` | 1 | *(removed)* | |
| `mqtt_username` | service@wirelessatmstore.com | *(empty)* | MQTT disabled |
| `ntp_server` | 10.4.6.30;time.nist.gov;time.google.com | 10.4.6.30;74.118.247.208;time.google.com | Custom NTP |
| `portal_enable` | *(empty)* | *(removed)* | |
| `rmon_advance_cfg` | _wan1_imei,_wan1_iccid,_wan1_sinr | _wan1_imei,_wan1_iccid,_traffic_daily_value,_traffic_monthly_value,_traffic_monthly2_value | Traffic monitoring instead of signal |
| `rmon_io_value` | 0 | *(removed)* | |
| `rmon_tls_sni` | *(empty)* | *(removed)* | |
| `sms_apn` | APN | *(removed)* | SMS disabled |
| `sms_enable` | 1 | 0 | SMS disabled |
| `sms_network_provider` | NET | *(removed)* | |
| `sms_rb` | REB | *(empty)* | |
| `sms_sq` | STA | *(empty)* | |
| `sms_switch_main_sim` | SIM | *(removed)* | |
| `ssl_server` | *(24 tunnel entries)* | *(13 tunnel entries)* | Different SSL tunnels |
| `traffic_day_threshold` | 3584 | 3 | Different threshold + unit |
| `traffic_day_unit` | 1 | 2 | |
| `wan1_icmp_detect_enable` | 0 | *(removed)* | |
| `wan1_icmp_interval` | 3600 | 6000 | Longer ICMP interval |

### Cord — 15 overridden parameters

| Parameter | Base Value | Cord Value | Notes |
|-----------|-----------|------------|-------|
| `advanced` | 1 | 0 | |
| `dhcpd_lease` | 60 | 600 | 10x longer lease |
| `fw_acl` | *(default allow all)* | *(25+ named ACL entries)* | Extensive firewall: Amazon, CDS, Switch Commerce, PAI, Digital Network, CORD RMS, 1st ISO, LibertyX, DNS, Loaded ATMs RMS, Cord Puloon RMS |
| `http_api_description` | *(absent)* | *(empty)* | Added |
| `http_api_src` | *(absent)* | *(empty)* | Added |
| `portal_enable` | *(empty)* | *(removed)* | |
| `rmon_tls_sni` | *(empty)* | *(removed)* | |
| `sms_apn` | APN | *(removed)* | SMS disabled |
| `sms_enable` | 1 | 0 | SMS disabled |
| `sms_network_provider` | NET | *(removed)* | |
| `sms_rb` | REB | *(empty)* | |
| `sms_sq` | STA | *(empty)* | |
| `sms_switch_main_sim` | SIM | *(removed)* | |
| `wl0_radius_ip` | *(empty)* | 192.168.2.2 | RADIUS auth enabled |
| `wl0_radius_key` | *(empty)* | 123456 | RADIUS key |

### Baum — 31 overridden parameters

| Parameter | Base Value | Baum Value | Notes |
|-----------|-----------|------------|-------|
| `advanced` | 1 | 0 | |
| `alarm_input` | *(empty)* | 0,0,1,1,0,0,0,0,0,0,0, | Custom alarm inputs |
| `alarm_output` | *(empty)* | 0,0,1, | Custom alarm outputs |
| `console_enable` | 1 | 0 | Console disabled |
| `cron_advanced` | *(absent)* | 0 | Added |
| `cron_rb_enable` | 0 | 1 | Scheduled reboot enabled |
| `cron_rb_time` | 225 | 240 | Reboot at 4:00 AM |
| `dtu1_enable` | *(empty)* | *(removed)* | |
| `fw_acl` | *(default allow all)* | *(30+ named ACL entries + default deny)* | Extensive firewall with default-deny block rule at end |
| `fw_nat` | *(empty)* | *(3 NAT rules)* | Port forwarding for 3 devices |
| `gsm_wcdma_band_config` | ALL | *(removed)* | |
| `http_api_description` | *(absent)* | *(empty)* | Added |
| `http_api_src` | *(absent)* | *(empty)* | Added |
| `io_chip` | 1 | 0 | I/O chip disabled |
| `iodigital_input_data` | fc | 00 | I/O inputs cleared |
| `lan0_iface` | eth2.1 | lan0 | Different LAN interface |
| `lte_band_config` | ALL | *(removed)* | |
| `mqtt_center` | iot.inhandnetworks.com | *(empty)* | MQTT disabled |
| `mqtt_enable` | 1 | 0 | MQTT disabled |
| `mqtt_keepalive` | 60 | *(removed)* | |
| `mqtt_tls` | 1 | *(removed)* | |
| `mqtt_username` | service@wirelessatmstore.com | *(empty)* | MQTT disabled |
| `ntp_server` | 10.4.6.30;time.nist.gov;time.google.com | time.nist.gov;10.4.6.30;time.google.com | Reordered NTP servers |
| `openvpn_ovpn1` | *(VPN config with AES-256-CBC, port 1194)* | *(VPN config with AES-128-CBC, port 1198)* | Different cipher and port |
| `redial_reboot` | 1 | *(removed)* | |
| `rmon_advance_cfg` | _wan1_imei,_wan1_iccid,_wan1_sinr | _wan1_imei,_wan1_iccid,_wan1_sinr, | Trailing comma (minor) |
| `rmon_user` | Test_4 | test | |
| `ssl_server` | *(24 tunnel entries)* | *(24 tunnel entries)* | Same count, different endpoints |
| `traffic_day_threshold` | 3584 | 5 | Different threshold + unit |
| `traffic_day_unit` | 1 | 2 | |
| `wan1_icmp_interval` | 3600 | 6000 | Longer ICMP interval |
