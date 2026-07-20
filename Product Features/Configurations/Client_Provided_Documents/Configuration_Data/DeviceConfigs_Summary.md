# DeviceConfigs - Device.csv Summary

## Purpose
Device-specific configuration overrides that apply to individual devices. This is the highest priority configuration layer, allowing per-device customization of any inherited settings from Global, Model, Carrier, Service Plan, or Customer layers.

## Key Configuration Areas

### 1. Port Mode Configuration (line 1)
- `port_mode=0` - LAN port mode configuration
  - **0**: 2xLAN (both ports function as LAN)
  - **1**: WAN+LAN (one port as WAN, one as LAN)
- Critical for devices requiring cellular backup via physical WAN port

### 2. Status Reporting Configuration (line 3)
- `rmon_advance_cfg=_wan1_imei,_wan1_iccid,_wan1_sinr`
- Devices with Cellular Backup enabled may need to report:
  - WAN address
  - IMEI (device identifier)
  - ICCID (SIM card identifier)
  - SINR (Signal-to-Interference-plus-Noise Ratio)
- Additional information for advanced monitoring

### 3. SIM Card Binding (lines 6-9)
- `sim1_binding_iccid=` - Bind SIM1 to specific ICCID (marked with `!!` indicating importance)
- `sim1_card_operator=2` - SIM1 operator profile (from PROFILE TABLE)
- `sim2_binding_iccid=` - Bind SIM2 to specific ICCID (marked with `!!`)
- `sim2_card_operator=1` - SIM2 operator profile (from PROFILE TABLE)
- Allows locking device to specific SIM cards

### 4. SMS Control System (lines 10-18)
All parameters marked as "SMS" category:
- `sms_acl=` - SMS access control list
- `sms_apn=APN` - APN command via SMS
- `sms_ctrl_cmd=` - Control commands
- `sms_enable=1` - Enable SMS control
- `sms_network_provider=NET` - Network provider command
- `sms_rb=REB` - Reboot command
- `sms_sq=STA` - Status query command
- `sms_strict=0` - Strict SMS validation
- `sms_switch_main_sim=SIM` - Switch SIM command

### 5. Cellular Backup Configuration (lines 20-28)
All marked as "Cellular Back up" and "Dynamic":
- `wan_backup_link=wan1` - Backup interface (cellular)
- `wan_backup_mode=0` - Backup mode (0=auto, 1=manual)
- `wan_backup_retry=3` - Retry attempts
- `wan_backup_rule=0` - Failover rule
- `wan_backup_time=3600` - Time before considering backup (1 hour)
- `wan_linkbackup_enable=0` - Enable link backup feature
- `wan_main_link=wan0` - Primary interface (physical WAN)
- `wan_main_retry=3` - Main link retry attempts

### 6. Physical WAN Interface (WAN0) Configuration (lines 30-39)
Settings for physical WAN port (when port_mode=1):
- `wan0_default_route=1` - Use as default route
- `wan0_gateway=192.168.1.1` - Gateway address
- `wan0_icmp_host=` - ICMP monitoring host
- `wan0_icmp_interval=30` - Check every 30 seconds
- `wan0_icmp_retries=3` - Retry count
- `wan0_icmp_timeout=20` - Timeout in seconds
- `wan0_iface=eth2.2` - Physical interface
- `wan0_ip=192.168.1.29` - IP address
- `wan0_lan_mode=lan` - LAN mode setting
- `wan0_proto=disabled` - Protocol (disabled by default)

### 7. WiFi Configuration (lines 42-69)
All marked as "Wifi" and "Dynamic":

**Basic WiFi Settings**:
- `wl0_enable=0` - WiFi disabled by default
- `wl0_ssid=inhand` - Default SSID
- `wl0_channel=11` - WiFi channel
- `wl0_mode=9` - WiFi mode
- `wl0_ap=1` - Access point mode
- `wl0_bridge=0` - Bridge mode
- `wl0_bw=0` - Bandwidth
- `wl0_ssid_brdcast=1` - Broadcast SSID
- `wl0_iface=none` - Interface assignment

**WiFi Security**:
- `wl0_auth=0` - Authentication type
- `wl0_encrypt=0` - Encryption type
- `wl0_wep_key=12345` - WEP key
- `wl0_wpa_encrypt=2` - WPA encryption
- `wl0_wpa_psk=abcdefgh` - WPA pre-shared key
- `wl0_gkey_cycle=0` - Group key rotation

**RADIUS Settings** (for enterprise WiFi):
- `wl0_radius_ip=192.168.2.2` - RADIUS server IP
- `wl0_radius_port=1812` - RADIUS port
- `wl0_radius_key=123456` - RADIUS shared secret
- `wl0_radius_idle_timeout=0` - Idle timeout
- `wl0_radius_session_timeout=0` - Session timeout

**WDS (Wireless Distribution System)**:
- `wl0_wds_enable=0` - WDS disabled
- `wl0_wds_ssid=` - WDS SSID
- `wl0_wds_bssid=` - WDS BSSID
- `wl0_wds_auth=0` - WDS authentication
- `wl0_wds_encrypt=0` - WDS encryption
- `wl0_wds_wep_key=12345` - WDS WEP key
- `wl0_wds_wpa_encrypt=2` - WDS WPA encryption
- `wl0_wds_wpa_psk=abcdefgh` - WDS WPA PSK

### 8. WiFi in STA (Station) Mode Configuration (lines 73-109)
When WiFi is used as WAN connection (WAN2):

**Interface Settings**:
- `wan2_iface=ra0` - WiFi interface
- `wan2_proto=none` - Protocol disabled by default
- `wan2_ip=192.168.3.29` - IP address
- `wan2_gateway=192.168.3.1` - Gateway
- `wan2_netmask=255.255.255.0` - Subnet mask
- `wan2_mac=` - MAC address override
- `wan2_mtu=1500` - MTU size
- `wan2_mtu_enable=0` - MTU override disabled

**Connection Monitoring**:
- `wan2_default_route=1` - Use as default route
- `wan2_icmp_host=` - ICMP monitoring host
- `wan2_icmp_interval=30` - Check interval
- `wan2_icmp_retries=3` - Retry count
- `wan2_icmp_timeout=20` - Timeout

**PPPoE Settings** (if needed):
- `wan2_ppp_mode=0` - PPP mode
- `wan2_ppp_username=` - Username
- `wan2_ppp_passwd=` - Password
- `wan2_ppp_service=` - Service name
- `wan2_ppp_ac=` - Access concentrator
- `wan2_ppp_check_interval=55` - PPP keepalive
- `wan2_ppp_check_retries=10` - PPP retries
- `wan2_ppp_redial_interval=30` - Redial interval
- And many more PPPoE parameters...

**Scheduling & Triggers**:
- `wan2_schedule=` - Connection schedule
- `wan2_trig_data=1` - Data-triggered connection
- `wan2_shared=1` - Shared connection
- `wan2_debug=0` - Debug mode

## Key Insights

1. **Override Authority**: Device-level configs override all other layers (Global, Model, Carrier, Service Plan, Customer)
2. **Cellular Backup**: Comprehensive support for automatic failover to cellular when primary WAN fails
3. **Flexible WAN Options**: Supports physical WAN port, cellular (WAN1), and WiFi (WAN2) as connectivity options
4. **SMS Remote Management**: Full SMS command system for remote device control
5. **SIM Locking**: Can bind device to specific SIM cards via ICCID
6. **WiFi Versatility**: Can function as AP, bridge, or WDS; also supports enterprise RADIUS authentication
7. **Dynamic Configuration**: Most parameters are dynamic and can be customized per device

## Configuration Parameters Summary
- **Total Parameters**: ~70
- **Port Mode**: 1
- **SIM Management**: 4
- **SMS Control**: 9
- **Cellular Backup**: 8
- **WAN0 (Physical Port)**: 10
- **WiFi AP/Client**: 28
- **WAN2 (WiFi as WAN)**: 37

## Common Use Cases

### Use Case 1: Cellular-Only Device
- `port_mode=0` (2xLAN)
- `wan0_proto=disabled`
- `wl0_enable=0`
- Primary connectivity via cellular (WAN1)

### Use Case 2: Wired WAN with Cellular Backup
- `port_mode=1` (WAN+LAN)
- `wan0_proto=dhcp` or `static`
- `wan_linkbackup_enable=1`
- `wan_main_link=wan0`
- `wan_backup_link=wan1`
- Cellular activates if WAN0 fails

### Use Case 3: WiFi Client with Cellular Backup
- `wl0_enable=1` (WiFi in STA mode)
- `wan2_proto=dhcp`
- `wan_linkbackup_enable=1`
- `wan_backup_link=wan1`

### Use Case 4: WiFi Access Point
- `wl0_enable=1`
- `wl0_ap=1`
- `wl0_ssid=custom_ssid`
- `wl0_wpa_psk=secure_password`
- Provides WiFi for connected ATM or devices

## Classification
- **Layer**: Device-specific (highest priority)
- **Type**: Dynamic (varies per device)
- **Override Capability**: Overrides all other layers

## Implementation Notes

1. **SIM Binding**: The `!!` markers on SIM binding parameters indicate critical importance for device-SIM association
2. **Profile Table Dependencies**: SIM operator values reference external profile table
3. **ICMP Monitoring**: Should be configured per device if monitoring specific upstream hosts
4. **Default State**: Most advanced features (WiFi, cellular backup, SMS) are disabled by default and must be explicitly enabled
5. **SMS Commands**: SMS command strings (APN, REB, STA, NET, SIM) are customizable keywords for remote control

---

**Source File**: `/Users/aksana/Documents/Projects/WATM/Configurations/01_Client_Provided_Documents/Configuration_Data/DeviceConfigs - Device.csv`
**Date Generated**: 2026-02-04
