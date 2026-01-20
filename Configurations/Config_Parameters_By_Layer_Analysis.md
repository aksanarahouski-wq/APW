# Config Parameters By Layer Analysis

**Document Version:** 1.0
**Date:** 2026-01-02
**Source:** Client-provided CSV files from ConfigsClientAnalisys directory

## Overview

This document provides a comprehensive analysis of all configuration parameters organized by hierarchical layer. The configuration system uses a 6-layer cascade model where more specific layers override more general layers.

**Layer Hierarchy:**
1. Global (most general)
2. Model
3. Carrier
4. Service Plan
5. Company/Customer
6. Device (most specific)

**Total Unique Parameters Identified:** ~400+ parameters across all layers

---

## 1. Global Layer

**Parameter Count:** ~250-300 parameters
**Scope:** Base configuration applied to all devices unless overridden
**Customer Access:** No (admin-only)

### Categories

#### 1.1 Authentication & Access Control
- `adm_user` - Admin username (matrixatm1)
- `adm_passwd` - Admin password (encrypted with $AES$)
- `adm_users` - Additional admin users
- `login_timeout` - Login timeout (500)

#### 1.2 DNS Configuration
- `dns_proxy_disable` - DNS proxy disable flag (0)
- `dns_static` - Static DNS servers (8.8.8.8;1.1.1.1)
- `dns_addget` - DNS add/get flag (1)
- `dnsmasq_custom` - Custom DNS masq configuration
- `dnsrelay_enable` - DNS relay enable (1)
- `dnsrelay_static` - Static DNS relay
- `dnsrelay` - DNS relay flag (1)
- `domainname` - Domain name (inhand-router.com)

#### 1.3 Time & NTP
- `ntp_server` - NTP servers (10.4.6.30;time.nist.gov;time.google.com)
- `ntp_updates` - NTP update interval (24 hours)
- `ntp_tdod` - Time/Date on Demand (1)
- `ntp_kiss` - NTP kiss-of-death
- `tm_dst` - Daylight Saving Time (1)
- `tm_sel` - Timezone selection (EST5EDT,M3.2.0/2,M11.1.0/2)
- `tm_tz` - Timezone (EST5EDT,M3.2.0/2,M11.1.0/2)

#### 1.4 System Identification
- `hostname` - Device hostname (DC_22_ATM_05082025) - **Dynamic, can be customized**
- `description` - System description (www.inhandnetworks.com)
- `router_name` - Router name (Router)
- `oem_name` - OEM name
- `language` - System language (English)

#### 1.5 HTTP/HTTPS Services
- `http_api_enable` - HTTP API enable (1)
- `http_api_local` - HTTP API local access (1)
- `http_api_remote` - HTTP API remote access (1)
- `http_api_port` - HTTP API port (4444)
- `http_api_src` - HTTP API source restriction
- `http_enable` - HTTP enable (1)
- `http_local` - HTTP local access (1)
- `http_remote` - HTTP remote access (1)
- `http_port` - HTTP port (80)
- `http_src` - HTTP source restriction
- `https_enable` - HTTPS enable (0)
- `https_local` - HTTPS local access (1)
- `https_remote` - HTTPS remote access (1)
- `https_port` - HTTPS port (443)
- `https_src` - HTTPS source restriction

#### 1.6 SSH Service
- `sshd_enable` - SSH enable (0)
- `sshd_local` - SSH local access (1)
- `sshd_remote` - SSH remote access (0)
- `sshd_port` - SSH port (22)
- `sshd_src` - SSH source restriction
- `sshd_pass` - SSH password authentication (0)
- `sshd_authkeys` - SSH authorized keys
- `sshd_hostkey` - SSH host key
- `sshd_hostkey2` - SSH host key 2

#### 1.7 Telnet Service
- `telnet_enable` - Telnet enable (0)
- `telnet_local` - Telnet local access (1)
- `telnet_remote` - Telnet remote access (1)
- `telnet_port` - Telnet port (50023)
- `telnet_src` - Telnet source restriction

#### 1.8 SNMP Service
- `snmpd_enable` - SNMP enable (0)
- `snmpd_port` - SNMP port (161)
- `snmpd_version` - SNMP version
- `snmpd_comlist` - SNMP community list
- `snmpd_grouplist` - SNMP group list
- `snmpd_userlist` - SNMP user list
- `snmpd_syscontact` - SNMP system contact
- `snmpd_syslocation` - SNMP system location
- `snmptrap_server_ip` - SNMP trap server IP
- `snmptrap_signal_level` - SNMP trap signal level (10)

#### 1.9 Firewall - Basic Settings
- `fw_anti_dos` - Anti-DoS protection (1)
- `fw_block_activex` - Block ActiveX (0)
- `fw_block_applet` - Block Java applets (0)
- `fw_block_cookie` - Block cookies (0)
- `fw_block_ident` - Block ident (0)
- `fw_block_loopback` - Block loopback (0)
- `fw_block_multicast` - Block multicast (1)
- `fw_block_proxy` - Block proxy (0)
- `fw_block_wan` - Block WAN access (0)
- `fw_clear_conntrack` - Clear connection tracking (1)
- `fw_log_limit` - Firewall log limit (60)
- `fw_mac_rules` - MAC-based firewall rules
- `fw_mac_rules_saved` - Saved MAC rules
- `fw_portmap` - Port mapping rules
- `fw_router_vip` - Router VIP
- `fw_strict` - Strict firewall mode (0)
- `fw_vip_range` - VIP range
- `fw_vip` - Virtual IP settings
- `multicast_pass` - Allow multicast pass-through (1)

#### 1.10 DHCP Server
- `dhcpd_enable` - DHCP server enable (1) - **May be customer/device configurable**
- `dhcpd_start` - DHCP start IP (192.168.1.100) - **May be customer/device configurable**
- `dhcpd_end` - DHCP end IP (192.168.1.254) - **May be customer/device configurable**
- `dhcpd_lease` - DHCP lease time (60) - **May be model-specific**
- `dhcpd_ifname` - DHCP interface name (lan0)
- `dhcpd_option_domain` - DHCP domain option (0)
- `dhcpd_wins` - DHCP WINS server (0.0.0.0)
- `dhcpd_static` - DHCP static leases - **May be customer/device configurable**
- `dhcpd_slt` - DHCP SLT

#### 1.11 LAN Configuration
- `lan0_mac` - LAN0 MAC address (inherited)
- `lan0_mdix` - LAN0 MDIX mode (0)
- `lan0_mip` - LAN0 MIP
- `lan0_mode` - LAN0 mode (0)
- `lan0_mtu_enable` - LAN0 MTU enable (0)
- `lan0_mtu` - LAN0 MTU (1500)
- `lan0_name` - LAN0 name (lan0)
- `lan0_netmask` - LAN0 netmask (255.255.255.0)
- `lan0_proto` - LAN0 protocol (static)
- `lan0_server` - LAN0 server (0.0.0.0)

#### 1.12 WAN1 (Cellular) - Default Settings
- `wan1_band_config` - Cellular band configuration (ALL;ALL;)
- `wan1_bridge_mode` - Bridge mode (0)
- `wan1_debug` - WAN1 debug mode (0)
- `wan1_debug_modem` - Modem debug mode (0)
- `wan1_default_route` - Default route flag (1)
- `wan1_iface` - Interface device (/dev/ttyUSB3)
- `wan1_ims` - IMS enable (0)
- `wan1_ip` - WAN1 IP (0.0.0.0)
- `wan1_mip` - WAN1 MIP
- `wan1_mtu_enable` - MTU enable (0)
- `wan1_mtu` - MTU size (1500)
- `wan1_name` - WAN1 name
- `wan1_proto` - Protocol (dialup)
- `wan1_shared` - Shared mode (1)
- `wan1_trig_call` - Trigger on call (1)
- `wan1_trig_data` - Trigger on data (1)
- `wan1_trig_sms` - Trigger on SMS (0)
- `wan1_uptime` - Uptime reporting (37)

#### 1.13 WAN1 PPP Settings
- `wan1_ppp_am` - PPP authentication mode (0)
- `wan1_ppp_authen` - PPP authentication (0)
- `wan1_ppp_callno` - PPP call number (*99#)
- `wan1_ppp_check_interval` - PPP check interval (55)
- `wan1_ppp_check_retries` - PPP check retries (6)
- `wan1_ppp_debug` - PPP debug (1)
- `wan1_ppp_freq_select` - Frequency selection (0)
- `wan1_ppp_idletime` - PPP idle time (0)
- `wan1_ppp_init` - PPP init string (AT)
- `wan1_ppp_ip` - PPP IP
- `wan1_ppp_iph_comp` - IP header compression (1)
- `wan1_ppp_mode` - PPP mode (0)
- `wan1_ppp_modem_mode` - Modem mode (0)
- `wan1_ppp_modem` - Modem settings
- `wan1_ppp_mru` - PPP MRU (1500)
- `wan1_ppp_network_select` - Network selection (0)
- `wan1_ppp_network` - Network type (FDD-LTE)
- `wan1_ppp_options` - PPP options (nomppe nomppc nodeflate nobsdcomp novj novjccomp noccp)
- `wan1_ppp_passwd` - PPP password
- `wan1_ppp_peer` - PPP peer IP (1.1.1.3)
- `wan1_ppp_peerdns` - Use peer DNS (1)
- `wan1_ppp_pincode` - SIM PIN code
- `wan1_ppp_provider` - Provider ID (2)
- `wan1_ppp_redial_interval` - Redial interval (30)
- `wan1_ppp_static` - Static IP mode (0)
- `wan1_ppp_timeout` - PPP timeout (120)
- `wan1_ppp_txql` - TX queue length (64)
- `wan1_ppp_username` - PPP username

#### 1.14 WAN1 ICMP Detection
- `wan1_icmp_detect_enable` - ICMP detection enable (1)
- `wan1_icmp_host` - Primary ICMP host (10.4.6.30) - **Must be configurable**
- `wan1_icmp_backup_host` - Backup ICMP host (10.4.6.30) - **Must be configurable**
- `wan1_icmp_main_host` - Main ICMP host (10.4.6.30) - **Must be configurable**
- `wan1_icmp_interval` - ICMP check interval (3600)
- `wan1_icmp_lost_packets` - Lost packets threshold (100)
- `wan1_icmp_mode` - ICMP mode (1)
- `wan1_icmp_retries` - ICMP retries (5)
- `wan1_icmp_timeout` - ICMP timeout (20)

#### 1.15 WAN0 (Physical Port) - Multi-Port Devices
- `wan0_debug` - WAN0 debug (0)
- `wan0_mac` - WAN0 MAC address (inherited)
- `wan0_mip` - WAN0 MIP
- `wan0_mtu_enable` - MTU enable (0)
- `wan0_mtu` - MTU (1500)
- `wan0_netmask` - Netmask (255.255.255.0)
- `wan0_ppp_ac` - PPPoE access concentrator
- `wan0_ppp_am` - PPP auth mode (0)
- `wan0_ppp_check_interval` - Check interval (55)
- `wan0_ppp_check_retries` - Check retries (10)
- `wan0_ppp_hostuniq` - PPPoE host unique
- `wan0_ppp_idletime` - Idle time (0)
- `wan0_ppp_ip` - PPP IP
- `wan0_ppp_iph_comp` - IP header compression (0)
- `wan0_ppp_mode` - PPP mode (0)
- `wan0_ppp_mru` - MRU (1500)
- `wan0_ppp_options` - PPP options
- `wan0_ppp_passwd` - PPP password
- `wan0_ppp_peer` - PPP peer
- `wan0_ppp_peerdns` - Use peer DNS (1)
- `wan0_ppp_redial_interval` - Redial interval (30)
- `wan0_ppp_service` - PPPoE service name
- `wan0_ppp_static` - Static IP (0)
- `wan0_ppp_txql` - TX queue length (3)
- `wan0_ppp_username` - PPP username
- `wan0_shared` - Shared mode (1)
- `wan0_trig_data` - Trigger on data (1)

#### 1.16 WAN2 (WiFi STA Mode)
- `wan2_debug` - WAN2 debug (0)
- `wan2_default_route` - Default route (1)
- `wan2_iface` - Interface (ra0)
- `wan2_mac` - MAC address
- `wan2_mip` - MIP
- `wan2_mtu_enable` - MTU enable (0)
- `wan2_mtu` - MTU (1500)
- `wan2_netmask` - Netmask (255.255.255.0)
- `wan2_proto` - Protocol (none)
- `wan2_shared` - Shared mode (1)
- `wan2_trig_data` - Trigger on data (1)
- Multiple `wan2_ppp_*` parameters for PPPoE mode

#### 1.17 QoS (Quality of Service)
- `qos_enable` - QoS enable (0)
- `qos_ack` - QoS ACK (1)
- `qos_default` - Default QoS class (0)
- `qos_iface` - QoS interface (wan1)
- `qos_ibw` - Inbound bandwidth (100000)
- `qos_obw` - Outbound bandwidth (100000)
- `qos_icmp` - ICMP priority (1)
- `qos_method` - QoS method (1)
- `qos_inuse` - QoS in use (1)
- `qos_burst0` - Burst 0 (64)
- `qos_burst1` - Burst 1 (64)
- `qos_irates` - Ingress rates
- `qos_orates` - Egress rates
- `qos_reset` - QoS reset (1)
- `qoslimit_enable` - QoS limit enable (0)
- `qoslimit_rule` - QoS limit rules
- `down_bandwidth` - Download bandwidth (1000)
- `up_bandwidth` - Upload bandwidth (1000)

#### 1.18 VPN - IPSec
- `ipsec_compress` - IPSec compression (1)
- `ipsec_debug` - IPSec debug (0)
- `ipsec_dynnattport` - Dynamic NAT-T port (0)
- `ipsec_force_natt` - Force NAT-T (0)
- `ipsec_natt_enable` - NAT-T enable (1)
- `ipsec_natt_interval` - NAT-T keepalive interval (60)
- `ipsec_nocrsend` - No CR send (0)
- `ipsec_policies` - IPSec policies
- `ipsec_stack` - IPSec stack (netkey)
- `ipsec_tunnel_action` - Tunnel action
- `ipsec_tunnel_name` - Tunnel name
- `ipsec_tunnels` - IPSec tunnels
- `ipsec_uniqueids` - Unique IDs (1)
- `ike_policies` - IKE policies

#### 1.19 VPN - L2TP Client
- `l2tpc_tunnel_action` - L2TP client tunnel action
- `l2tpc_tunnel_name` - L2TP client tunnel name
- `l2tpc_tunnels` - L2TP client tunnels
- `l2tp_debug` - L2TP debug (0)

#### 1.20 VPN - L2TP Server
- `l2tps_enable` - L2TP server enable (0)
- `l2tps_debug` - L2TP server debug (0)
- `l2tps_expert` - L2TP server expert mode
- `l2tps_interval` - L2TP server interval (60)
- `l2tps_localip` - L2TP server local IP
- `l2tps_mppe` - L2TP server MPPE (0)
- `l2tps_passwd` - L2TP server password
- `l2tps_remoteip` - L2TP server remote IP
- `l2tps_remotemask` - L2TP server remote mask
- `l2tps_remotenet` - L2TP server remote network
- `l2tps_retry` - L2TP server retry (5)
- `l2tps_username` - L2TP server username

#### 1.21 VPN - PPTP Client
- `pptpc_tunnel_action` - PPTP client tunnel action
- `pptpc_tunnel_name` - PPTP client tunnel name
- `pptpc_tunnels` - PPTP client tunnels

#### 1.22 VPN - PPTP Server
- `pptps_enable` - PPTP server enable (0)
- `pptps_debug` - PPTP server debug (0)
- `pptps_expert` - PPTP server expert mode
- `pptps_interval` - PPTP server interval (60)
- `pptps_localip` - PPTP server local IP
- `pptps_mppe` - PPTP server MPPE (0)
- `pptps_passwd` - PPTP server password
- `pptps_remoteip` - PPTP server remote IP
- `pptps_remotemask` - PPTP server remote mask
- `pptps_remotenet` - PPTP server remote network
- `pptps_retry` - PPTP server retry (5)
- `pptps_username` - PPTP server username

#### 1.23 VPN - OpenVPN
- `openvpn_c2c_enable` - OpenVPN client-to-client (0)
- `openvpn_dns1` - OpenVPN DNS 1 (1)
- `openvpn_dns2` - OpenVPN DNS 2
- `openvpn_dns3` - OpenVPN DNS 3
- `openvpn_dns4` - OpenVPN DNS 4
- `openvpn_ovpn1` - OpenVPN config 1 (complete .ovpn file with certificates)
- `openvpn_secret1` - OpenVPN secret 1
- `openvpn_secret2` - OpenVPN secret 2
- `openvpn_secret3` - OpenVPN secret 3
- `openvpn_secret4` - OpenVPN secret 4
- `openvpn_tunnel_action` - OpenVPN tunnel action (edit)
- `openvpn_tunnel_name` - OpenVPN tunnel name (OpenVPN_T_1)
- `openvpn_tunnels` - OpenVPN tunnel definitions
- `openvpn_user_list` - OpenVPN user list

#### 1.24 VPN - GRE Tunnels
- `gre_tunnels` - GRE tunnel definitions

#### 1.25 VPN - Certificate Management
- `cert_ca` - CA certificate
- `cert_crl` - Certificate revocation list
- `cert_key` - Certificate key
- `cert_private` - Private certificate
- `cert_public` - Public certificate
- `scep_enable` - SCEP enable (0)
- `scep_challenge` - SCEP challenge
- `scep_common_name` - SCEP common name
- `scep_domain` - SCEP domain
- `scep_fqdn` - SCEP FQDN
- `scep_key_len` - SCEP key length (1024)
- `scep_poll_interval` - SCEP poll interval (60)
- `scep_poll_retries` - SCEP poll retries (3)
- `scep_poll_timeout` - SCEP poll timeout (3600)
- `scep_req` - SCEP request
- `scep_serialno` - SCEP serial number
- `scep_status` - SCEP status
- `scep_transid` - SCEP transaction ID
- `scep_unaddr` - SCEP unit address
- `scep_unit1` - SCEP unit 1
- `scep_unit2` - SCEP unit 2
- `scep_url` - SCEP URL

#### 1.26 DMZ Configuration
- `dmz_enable` - DMZ enable (0)
- `dmz_iface` - DMZ interface
- `dmz_ip` - DMZ IP
- `dmz_src` - DMZ source
- `dmz_tunnel` - DMZ tunnel
- `dmz0_iface` - DMZ0 interface (none)
- `dmz0_ip` - DMZ0 IP (192.168.3.1)
- `dmz0_mac` - DMZ0 MAC
- `dmz0_mip` - DMZ0 MIP
- `dmz0_mtu_enable` - DMZ0 MTU enable (0)
- `dmz0_mtu` - DMZ0 MTU (1500)
- `dmz0_name` - DMZ0 name (dmz0)
- `dmz0_netmask` - DMZ0 netmask (255.255.255.0)
- `dmz0_proto` - DMZ0 protocol (none)

#### 1.27 DDNS (Dynamic DNS)
- `ddnsx0` - DDNS config 0
- `ddnsx0_cache` - DDNS cache 0
- `ddnsx1` - DDNS config 1
- `ddnsx1_cache` - DDNS cache 1
- `ddnsx2` - DDNS config 2
- `ddnsx2_cache` - DDNS cache 2

#### 1.28 Serial/DTU Configuration
- `com0_config` - COM0 configuration (115200 8N1)
- `com0_hw_flow` - COM0 hardware flow control (0)
- `com0_sw_flow` - COM0 software flow control (0)
- `com1_config` - COM1 configuration (115200 8N1)
- `com1_hw_flow` - COM1 hardware flow control (0)
- `com1_sw_flow` - COM1 software flow control (0)
- `com4_config` - COM4 configuration (19200 8N1)
- `com4_hw_flow` - COM4 hardware flow control (0)
- `com4_sw_flow` - COM4 software flow control (0)
- `console_enable` - Console enable (0)
- `console_description` - Console description
- `console_iface` - Console interface (/dev/ttyS1)

#### 1.29 DTU (Data Terminal Unit)
- `dtu_enable` - DTU enable (0)
- `dtu_buf_size` - DTU buffer size (10240)
- `dtu_conn_idle` - Connection idle timeout (30)
- `dtu_conn_interval` - Connection interval (5)
- `dtu_conn_retry_max` - Max retry time (180)
- `dtu_conn_retry_min` - Min retry time (15)
- `dtu_conn_retry_num` - Retry count (5)
- `dtu_connection_mode` - Connection mode (0)
- `dtu_crlf` - CRLF mode (0)
- `dtu_downstr` - Down string
- `dtu_frameval` - Frame value (100)
- `dtu_hbnum` - Heartbeat number (5)
- `dtu_hbval` - Heartbeat value (60)
- `dtu_id_trap_intv` - ID trap interval (0)
- `dtu_id` - DTU ID
- `dtu_iface` - DTU interface (/dev/ttyS1)
- `dtu_ma8_enable` - MA8 enable (0)
- `dtu_ma8_hb_context` - MA8 heartbeat context
- `dtu_ma8_hb_format` - MA8 heartbeat format (0)
- `dtu_ma8_hb_used` - MA8 heartbeat used (0)
- `dtu_mserver_policy` - Multi-server policy (0)
- `dtu_port` - DTU port (502)
- `dtu_protocol` - DTU protocol (0)
- `dtu_server_mode` - Server mode (0)
- `dtu_server` - DTU server
- `dtu_src` - DTU source
- `dtu_str_format` - String format (1)
- `dtu_tcp_mode` - TCP mode (0)
- `dtu_timeout` - DTU timeout (120)
- `dtu_uartframe_num` - UART frame number (4)
- `dtu_upstr` - Up string
- `dtu1_enable` - DTU1 enable
- `dtu1_iface` - DTU1 interface (/dev/ttyS0)

#### 1.30 Status Reporting (RMON)
- `rmon_enable` - RMON enable (1)
- `rmon_cur_softver` - Current software version reporting (1)
- `rmon_geolocation` - Geolocation reporting (0)
- `rmon_hostname` - Hostname reporting (1)
- `rmon_httpapi_enable` - HTTP API reporting (1)
- `rmon_io_value` - I/O value reporting (0)
- `rmon_ipaddr` - IP address reporting (1)
- `rmon_ipaddr2` - IP address 2 reporting (1)
- `rmon_log_enable` - Log enable (1)
- `rmon_passwd` - RMON password (test)
- `rmon_protocol` - RMON protocol (2)
- `rmon_rep_interval` - Report interval (120 seconds)
- `rmon_report_args` - Report arguments (1)
- `rmon_serial_num` - Serial number reporting (1)
- `rmon_server_domain` - RMON server domain (apcommand.com)
- `rmon_server_port` - RMON server port (8002)
- `rmon_signal_strength` - Signal strength reporting (1)
- `rmon_terminal_id` - Terminal ID reporting (1)
- `rmon_timestamp` - Timestamp reporting (1)
- `rmon_uptime` - Uptime reporting (1)
- `rmon_user` - RMON user (Test_4)

#### 1.31 Scheduler/Cron Jobs
- `cron_rb_enable` - Cron reboot enable (0)
- `cron_rb_days` - Cron reboot days (0)
- `cron_rb_time` - Cron reboot time (225)
- `schedule_list` - Schedule list

#### 1.32 Logging
- `log_console` - Console logging (0)
- `log_crond` - Cron daemon logging (1)
- `log_level` - Log level (7)
- `log_mark` - Log mark (0)
- `log_remote` - Remote logging (0)
- `log_remoteip` - Remote log IP
- `log_remoteport` - Remote log port (514)

#### 1.33 Cellular Configuration
- `max_modem_reset` - Max modem reset attempts (120)
- `max_ppp_redial` - Max PPP redial attempts (10)
- `gsm_wcdma_band_config` - GSM/WCDMA band config (ALL)
- `auto_ping_enable` - Auto ping enable (0)

#### 1.34 Hardware & System
- `hw_reset_disable` - Hardware reset disable (0) - **Possible value: 1**
- `advanced` - Advanced mode (1)
- `debug_keepfiles` - Keep debug files (1)
- `engineer_mode` - Engineer mode (0)
- `env_path` - Environment path
- `port_mode` - Port mode (0)

#### 1.35 Smart ATM/SSL Proxy
- `ssl_proxy_enable` - SSL proxy enable (1)
- `ssl_proxy_processor_host` - Processor host
- `ssl_proxy_processor_port` - Processor port
- `ssl_proxy_tcp_server_addr` - TCP server address
- `ssl_proxy_tcp_server_port` - TCP server port
- `ssl_server` - SSL server configuration (extensive port mapping table)

#### 1.36 VRRP (Virtual Router Redundancy Protocol)
- `vrrpd0_enable` - VRRP0 enable (0)
- `vrrpd0_iface` - VRRP0 interface (lan0)
- `vrrpd0_vrid` - VRRP0 virtual router ID (1)
- `vrrpd0_prio` - VRRP0 priority (20)
- `vrrpd0_vip` - VRRP0 virtual IP
- `vrrpd0_adv_interval` - VRRP0 advertisement interval (60)
- `vrrpd0_auth` - VRRP0 authentication (none)
- `vrrpd0_mon` - VRRP0 monitoring (0)
- `vrrpd0_vmac` - VRRP0 virtual MAC (0)
- Multiple `vrrpd1_*` parameters for second VRRP instance

#### 1.37 Device Manager (OVDP/InHand)
- `ovdp_enable` - OVDP enable (0)
- `ovdp_center` - OVDP center (g.inhandnetworks.com)
- `ovdp_center_port` - OVDP port (20003)
- `ovdp_device_id` - Device ID (302003017)
- `ovdp_entity_conf` - Entity configuration
- `ovdp_hb_interval` - Heartbeat interval (120)
- `ovdp_hb_retries` - Heartbeat retries (3)
- `ovdp_hb_timeout` - Heartbeat timeout (3600)
- `ovdp_login_retries` - Login retries (3)
- `ovdp_mode` - OVDP mode (0)
- `ovdp_rx_timeout` - RX timeout (30)
- `ovdp_sms_interval` - SMS interval (24)
- `ovdp_trust_list` - Trust list
- `ovdp_tx_retries` - TX retries (3)
- `ovdp_vendor_id` - Vendor ID (0003)

#### 1.38 GPS
- `gps_enable` - GPS enable (0)

#### 1.39 OSPF
- `ospf_enable` - OSPF enable (0)

#### 1.40 Misc Network Settings
- `ct_max` - Connection tracking max (2048)
- `ct_tcp_timeout` - TCP connection timeout
- `ct_udp_timeout` - UDP connection timeout
- `nf_ttl` - NetFilter TTL (0)
- `rstats_exclude` - Statistics exclude
- `ip_passthrough_mac` - IP passthrough MAC

#### 1.41 Security Settings
- `pam_shield_interval` - PAM shield interval (600)
- `pam_shield_retention` - PAM shield retention (3600)
- `pam_shield_retry` - PAM shield retry (20)

#### 1.42 Portal
- `portal_enable` - Portal enable

#### 1.43 State Grid (China-specific)
- `stategrid_enable` - State grid enable

#### 1.44 Dial While Connected
- `dialwh_enable` - Dial while connected enable (0)
- `dialwh_port` - Dial while port (6001)
- `dialwh_proto` - Dial while protocol (TCP)
- `dialwh_conn_cmd` - Dial while connect command (CONNECT)
- `dialwh_conn_timeout` - Dial while connect timeout (60)
- `dialwh_disconn_cmd` - Dial while disconnect command (DOWN)
- `dialwh_disconn_timeout` - Dial while disconnect timeout (30)

#### 1.45 System Configuration
- `sysconf_timestamp` - System config timestamp
- `t_cafree` - CA free parameter (1234)

---

## 2. Model Layer

**Parameter Count:** ~40-50 parameters
**Scope:** Model-specific configurations (I-22, I-52, Origin, IR611, IR615)
**Customer Access:** No (admin-only)

### Categories

#### 2.1 Alarm System
**Availability:** I-22, I-52, Origin models
- `alarm_input_options` - Alarm input options list (fault-service, memory-low, port0-wan-link-up/down, port1-4-link-up/down, dialup-up/down, traffic-alarm, traffic-discon, switch-sim-card, switch-backup-link, fault-sim-card, fault-signal-quality)
- `alarm_input` - Alarm input enable flags (0,0,1,1,0,0,0,0,0,0,0)
- `alarm_output_options` - Alarm output options (cli, out-dm, out-rmon)
- `alarm_output` - Alarm output enable flags (0,0,1)
- `alarm_clear` - Alarm clear flag (0)
- `alarm_confirm` - Alarm confirm flag (0)

#### 2.2 Digital I/O
**Availability:** I-22, I-52 only (NOT on IR611/IR615)
- `digitalio_config` - Digital I/O configuration (1,0,0;1,0,0;)
- `io_chip` - I/O chip selection (0)
- `io_triggered_report` - I/O triggered reporting (0)
- `iodigital_input_data` - Digital input data (0c)

#### 2.3 LAN Port Configuration
**Note:** May differ between IR611/IR615 and I-22/I-52
- `lan_port1` - LAN port 1 enable (1)
- `lan_port2` - LAN port 2 enable (1)
- `lan0_default_route` - LAN0 default route (1)
- `lan0_gateway` - LAN0 gateway
- `lan0_iface` - LAN0 interface (eth2.1)
- `lan0_ip` - LAN0 IP address (192.168.1.90)
- `lan0_last_ip` - LAN0 last IP (192.168.2.1)
- `lan0_last_netmask` - LAN0 last netmask (255.255.255.0)

#### 2.4 Link Backup
**Availability:** IR611/IR615 only (NOT on I-22/I-52)
- `linkbackup_enable` - Link backup enable (0)
- `linkbackup_hot_mode` - Link backup hot mode (1)

#### 2.5 MQTT Device Manager
**Availability:** Verizon/T-Mobile I-22 and I-52 only (NOT AT&T, NOT IR611/IR615)
- `mqtt_enable` - MQTT enable (1)
- `mqtt_center` - MQTT center (iot.inhandnetworks.com)
- `mqtt_atm_id` - ATM ID
- `mqtt_device_info` - Device info
- `mqtt_experience_confirmed` - Experience confirmed (1)
- `mqtt_experience_mode` - Experience mode (disable)
- `mqtt_keepalive` - MQTT keepalive (60)
- `mqtt_lbs_interval` - LBS interval (24)
- `mqtt_series_interval` - Series interval (24)
- `mqtt_sniffer_enable` - Sniffer enable (1)
- `mqtt_sniffer_filter_enable` - Sniffer filter enable (0)
- `mqtt_tls` - MQTT TLS (1)
- `mqtt_username` - MQTT username (service@wirelessatmstore.com)

#### 2.6 Status Report Advanced Config
**Availability:** I-22 may need I/O digital data; other models may not
- `rmon_advance_cfg` - Advanced RMON config (_wan1_imei,_wan1_iccid,_wan1_sinr)

#### 2.7 SMS Controls
**Availability:** I-22/I-52 only
- `sms_enable` - SMS enable (1)
- `sms_acl` - SMS ACL
- `sms_apn` - SMS APN command (APN)
- `sms_ctrl_cmd` - SMS control commands
- `sms_network_provider` - SMS network provider command (NET)
- `sms_rb` - SMS reboot command (REB)
- `sms_sq` - SMS status query command (STA)
- `sms_strict` - SMS strict mode (0)
- `sms_switch_main_sim` - SMS switch SIM command (SIM)

#### 2.8 Serial Port Configuration
**Model-specific differences**
- `com0_config` - COM0 config (115200 8N1)
- `com1_config` - COM1 config (115200 8N1)
- `com4_config` - COM4 config (19200 8N1)

---

## 3. Carrier Layer

**Parameter Count:** ~50+ parameters
**Scope:** Carrier-specific configurations (Verizon, AT&T, T-Mobile)
**Customer Access:** No (admin-only)

### Categories

#### 3.1 Dual SIM Configuration
**Two variants: Non-Dual SIM vs Dual SIM enabled**

**Non-Dual SIM Devices:**
- `dual_sim_enable` - Dual SIM enable (0)
- `dual_sim_main` - Main SIM slot (0)
- `dual_sim_csq_retry` - CSQ retry (4)
- `dual_sim_max_retry` - Max retry (5)
- `dual_sim_min_conn_time` - Min connection time (0)
- `dual_sim_min_csq` - Min CSQ threshold (4)

**Dual SIM Enabled Devices:**
- `dual_sim_enable` - Dual SIM enable (1)
- `dual_sim_main` - Main SIM slot (0) - **"Flip between Carriers by SIM1, SIM2"**
- `dual_sim_csq_retry` - CSQ retry (4)
- `dual_sim_max_retry` - Max retry (5)
- `dual_sim_min_conn_time` - Min connection time (120)
- `dual_sim_min_csq` - Min CSQ threshold (4)

#### 3.2 Backup SIM Policy
- `backup_sim_policy_enable` - Backup SIM policy enable (0)
- `backup_sim_policy_revert_day` - Revert to main SIM after days (1)
- `backup_sim_policy_using_time` - Backup SIM using time (0)

#### 3.3 APN Configuration
**Carrier-specific APNs:**
- **Verizon:** `wan1_ppp_apn=Matrxatm.gw12.vzwentp`
- **AT&T:** `wan1_ppp_apn=matrix.com.attz`
- **T-Mobile:** `wan1_ppp_apn=simpl.cc.static`

**SIM2 APNs:**
- `wan1_ppp_sim2_apn` - SIM2 APN (carrier-specific)
- `wan1_ppp_sim2_authen` - SIM2 authentication (0)
- `wan1_ppp_sim2_callno` - SIM2 call number (*99#)
- `wan1_ppp_sim2_network` - SIM2 network type
- `wan1_ppp_sim2_passwd` - SIM2 password
- `wan1_ppp_sim2_pincode` - SIM2 PIN code
- `wan1_ppp_sim2_provider` - SIM2 provider ID (1)
- `wan1_ppp_sim2_username` - SIM2 username

#### 3.4 Redial Configuration
- `redial_dual_apn_enable` - Dual APN redial enable (0)
- `redial_ppp_profiles` - PPP profile table (1:matrix.com.attz:*99#:0::;2:Matrxatm.gw12.vzwentp:*99#:0::;)
- `redial_reboot` - Reboot on redial failure (1)
- `redial_sim1_profile` - SIM1 profile ID
- `redial_sim1_profile2` - SIM1 secondary profile ID
- `redial_sim2_profile` - SIM2 profile ID
- `redial_sim2_profile2` - SIM2 secondary profile ID

#### 3.5 SIM Card Operator
**Profile table values** (from redial_ppp_profiles)
- `sim1_card_operator` - SIM1 operator ID (2) - **PROFILE TABLE VALUE**
- `sim2_card_operator` - SIM2 operator ID (1) - **PROFILE TABLE VALUE**

#### 3.6 WAN1 Settings (Carrier-specific)
- `wan1_ppp_apn` - Primary APN (carrier-specific)
- `wan1_ppp_redial_interval` - Redial interval (30) - **DYNAMIC**
- `wan1_icmp_host` - ICMP host (10.4.6.30) - **Must be configurable**
- `wan1_icmp_backup_host` - Backup ICMP host (10.4.6.30) - **Must be configurable**
- `wan1_icmp_main_host` - Main ICMP host (10.4.6.30) - **Must be configurable**
- `wan1_band_config` - Band configuration (ALL;ALL;)

#### 3.7 Cellular Band Configuration
- `lte_band_config` - LTE band config (ALL)

---

## 4. Service Plan Layer

**Parameter Count:** ~20+ parameters
**Scope:** Service plan-specific configurations (ATM vs Non-ATM plans)
**Customer Access:** No (admin-only, but affects what customer gets)

### Categories

#### 4.1 Firewall ACL Rules
**Two variants: Non-ATM vs ATM service plans**

**Non-ATM Service Plan:**
- `fw_acl` - Simple allow-all rule: `1<4<0.0.0.0/0<<0.0.0.0/0<<1<1<>`

**ATM Service Plan:**
- `fw_acl` - Extensive whitelist with 25+ entries including:
  - Amazon Resource (34.199.0.0/16)
  - CDS (208.35.209.1, 216.58.156.98, 198.28.31.98, etc.)
  - Switch Commerce (multiple IPs)
  - PAI (206.71.17.21/32)
  - Digital Network (20.88.238.228/32, 20.221.234.232)
  - PAI RMS (216.26.158.20, 216.26.158.19)
  - 1st ISO (69.21.165.134, 209.103.211.74)
  - EFX (64.88.167.58) - **Block flag**
  - LibertyX (34.199.247.183)
  - Planet Payment DCC (208.78.47.0/24)
  - DNS servers (8.8.8.8, 8.8.4.4, 1.1.1.1)
  - AWS Genmega (3.5.0.0/16, 52.192.0.0/12)
  - Final block-all rule: `1<4<0.0.0.0/0<<0.0.0.0/0<<2<1<Block>`

#### 4.2 Firewall NAT Rules
**Certain customers have designated NAT rules**
- `fw_nat` - NAT port forwarding rules (customer-specific)

#### 4.3 Content Filtering
- `fw_web` - URL filtering rules:
  - libertyx.com
  - atm1.switchcommerce.net
  - atm2.switchcommerce.net
  - atmssl.dnsatm.com (DNS)
  - atmtls.dnsatm.com (DNS)
  - gm.rmsfm.com (PAI RMS)
  - mv.rmsfm.com (PAI RMS)

**Note:** URL filtering differences between models:
- **I-22:** URLs go into `fw_acl`
- **I4100/I-45:** URLs go into `fw_web` (content filtering)
- Both need to be managed in the same "place" for customer configuration segmentation

#### 4.4 Traffic Management
**Different service plans have different data thresholds**
- `traffic_enable` - Traffic management enable (1)
- `traffic_day_action` - Daily action (1)
- `traffic_day_threshold` - Daily threshold (3584) - **Different per service plan**
- `traffic_day_unit` - Daily unit (1)
- `traffic_month_action` - Monthly action (1)
- `traffic_month_alarm` - Monthly alarm (0)
- `traffic_month_discon` - Monthly disconnect (0)
- `traffic_month_start_day` - Month start day (1)
- `traffic_month_threshold` - Monthly threshold (0) - **Different per service plan**
- `traffic_month_unit` - Monthly unit (1)
- `traffic_month2_action` - Secondary monthly action (0)
- `traffic_month2_threshold` - Secondary monthly threshold (0)
- `traffic_month2_unit` - Secondary monthly unit (1)
- `traffic_custom_enable` - Custom traffic rules enable (0)
- `traffic_custom_rule` - Custom traffic rules
- `traffic_custom_actions` - Custom actions (0,0,0,0)
- `traffic_exceed_report` - Exceed report (0)
- `traffic_sms_up` - SMS notification on traffic up

---

## 5. Company/Customer Layer

**Parameter Count:** ~10-20 parameters
**Scope:** Company/customer-specific configurations
**Customer Access:** Potentially yes (via self-service portal)

### Categories

#### 5.1 Firewall ACL Rules (Customer-Specific)
**Certain customers contain additional ACL entries for specific customer servers**
- `fw_acl` - Extended firewall rules beyond service plan defaults with customer-specific IPs:
  - CORD RMS (13.67.184.126)
  - DEPLOYER RMS (208.92.212.170)
  - Custom DNS servers (198.224.183.135)
  - CORD RMS Old (64.183.178.180)
  - ICMP & NTP (10.4.6.30)
  - Additional internal IPs (10.4.0.31, 10.4.0.32)
  - Additional CDS/SC ranges (216.58.156.0/24, 76.77.157.0/24, 216.117.40.0/24, 64.88.167.0/24)

#### 5.2 Firewall NAT Rules (Customer-Specific)
**Certain customers have designated NAT rules for traffic manipulation optimization**
- `fw_nat` - Customer-specific NAT rules:
  - ATM RMS forwarding
  - ALTECH forwarding
  - Example: `1<1<1<0.0.0.0/0<<0.0.0.0/0<18458<<10.4.0.32<18458<1<>`
  - Example: `1<1<1<0.0.0.0/0<<0.0.0.0/0<9999<<10.4.0.31<9999<1<>`

#### 5.3 DHCP Settings (Possibly Customer-Configurable)
- `dhcpd_enable` - DHCP enable
- `dhcpd_start` - DHCP start IP
- `dhcpd_end` - DHCP end IP
- `dhcpd_static` - Static DHCP leases

#### 5.4 LAN Settings (Possibly Customer-Configurable)
- Selected `lan0_*` parameters may be customer-configurable

---

## 6. Device Layer

**Parameter Count:** ~100+ parameters
**Scope:** Device-specific configurations
**Customer Access:** Potentially yes for some parameters (via self-service portal)

### Categories

#### 6.1 Port Mode
- `port_mode` - LAN port mode (0: 2xLAN; 1: WAN+LAN)

#### 6.2 SIM Card Binding
- `sim1_binding_iccid` - SIM1 ICCID binding - **!!**
- `sim2_binding_iccid` - SIM2 ICCID binding - **!!**
- `sim1_card_operator` - SIM1 operator (2) - **PROFILE TABLE VALUE**
- `sim2_card_operator` - SIM2 operator (1) - **PROFILE TABLE VALUE**

#### 6.3 SMS Configuration (Device-Specific)
- `sms_enable` - SMS enable (1)
- `sms_acl` - SMS ACL
- `sms_apn` - SMS APN command (APN)
- `sms_ctrl_cmd` - SMS control commands
- `sms_network_provider` - SMS network provider command (NET)
- `sms_rb` - SMS reboot command (REB)
- `sms_sq` - SMS status query command (STA)
- `sms_strict` - SMS strict mode (0)
- `sms_switch_main_sim` - SMS switch SIM command (SIM)

#### 6.4 Cellular Backup Configuration
- `wan_linkbackup_enable` - WAN link backup enable (0)
- `wan_backup_mode` - Backup mode (0)
- `wan_backup_link` - Backup link (wan1)
- `wan_main_link` - Main link (wan0)
- `wan_backup_retry` - Backup retry (3)
- `wan_main_retry` - Main retry (3)
- `wan_backup_rule` - Backup rule (0)
- `wan_backup_time` - Backup time (3600)

#### 6.5 WAN0 Configuration (Physical Port - Cellular Backup)
**These settings are only for Physical WAN interface**
- `wan0_proto` - Protocol (disabled)
- `wan0_default_route` - Default route (1)
- `wan0_gateway` - Gateway (192.168.1.1)
- `wan0_ip` - IP address (192.168.1.29)
- `wan0_iface` - Interface (eth2.2)
- `wan0_lan_mode` - LAN mode (lan)
- `wan0_icmp_host` - ICMP host
- `wan0_icmp_interval` - ICMP interval (30)
- `wan0_icmp_retries` - ICMP retries (3)
- `wan0_icmp_timeout` - ICMP timeout (20)

#### 6.6 WiFi Configuration
**Extensive WiFi settings (30+ parameters)**
- `wl0_enable` - WiFi enable (0)
- `wl0_ap` - AP mode (1)
- `wl0_ssid` - SSID (inhand)
- `wl0_ssid_brdcast` - SSID broadcast (1)
- `wl0_channel` - Channel (11)
- `wl0_mode` - WiFi mode (9)
- `wl0_bw` - Bandwidth (0)
- `wl0_bridge` - Bridge mode (0)
- `wl0_iface` - Interface (none)
- `wl0_auth` - Authentication (0)
- `wl0_encrypt` - Encryption (0)
- `wl0_wep_key` - WEP key (12345)
- `wl0_wpa_encrypt` - WPA encryption (2)
- `wl0_wpa_psk` - WPA PSK (abcdefgh)
- `wl0_gkey_cycle` - Group key cycle (0)
- `wl0_radius_ip` - RADIUS IP (192.168.2.2)
- `wl0_radius_port` - RADIUS port (1812)
- `wl0_radius_key` - RADIUS key (123456)
- `wl0_radius_idle_timeout` - RADIUS idle timeout (0)
- `wl0_radius_session_timeout` - RADIUS session timeout (0)
- `wl0_wds_enable` - WDS enable (0)
- `wl0_wds_bssid` - WDS BSSID
- `wl0_wds_ssid` - WDS SSID
- `wl0_wds_auth` - WDS authentication (0)
- `wl0_wds_encrypt` - WDS encryption (0)
- `wl0_wds_wep_key` - WDS WEP key (12345)
- `wl0_wds_wpa_encrypt` - WDS WPA encryption (2)
- `wl0_wds_wpa_psk` - WDS WPA PSK (abcdefgh)

#### 6.7 WiFi as WAN2 (STA Mode)
**When WiFi is configured as WAN backup (30+ parameters)**
- `wan2_proto` - Protocol (none)
- `wan2_iface` - Interface (ra0)
- `wan2_gateway` - Gateway (192.168.3.1)
- `wan2_ip` - IP address (192.168.3.29)
- `wan2_default_route` - Default route (1)
- `wan2_debug` - Debug mode (0)
- `wan2_icmp_host` - ICMP host
- `wan2_icmp_interval` - ICMP interval (30)
- `wan2_icmp_retries` - ICMP retries (3)
- `wan2_icmp_timeout` - ICMP timeout (20)
- `wan2_mac` - MAC address
- `wan2_mip` - MIP
- `wan2_mtu_enable` - MTU enable (0)
- `wan2_mtu` - MTU (1500)
- `wan2_netmask` - Netmask (255.255.255.0)
- `wan2_shared` - Shared mode (1)
- `wan2_trig_data` - Trigger on data (1)
- Multiple `wan2_ppp_*` parameters for PPPoE

#### 6.8 Static Routes
- `routes_static` - Static routes (device-specific)
- `routes_static_saved` - Saved static routes

#### 6.9 Redial Configuration (Device-Specific)
- `redial_dual_apn_enable` - Dual APN redial enable (0)
- `redial_ppp_profiles` - PPP profiles (device/carrier)
- `redial_reboot` - Reboot on failure (1)
- `redial_sim1_profile` - SIM1 profile
- `redial_sim1_profile2` - SIM1 profile 2
- `redial_sim2_profile` - SIM2 profile
- `redial_sim2_profile2` - SIM2 profile 2

#### 6.10 Advanced Status Reporting
**Devices with Cellular Backup Enabled may need to report WAN address or other additional information**
- `rmon_advance_cfg` - Advanced RMON config (_wan1_imei,_wan1_iccid,_wan1_sinr)

#### 6.11 Digital I/O (Device-Specific Values)
- `digitalio_config` - Digital I/O configuration (device-specific pin states)

#### 6.12 Traffic Management (Device-Specific Overrides)
- `traffic_enable` - Traffic enable (device override)
- `traffic_day_threshold` - Daily threshold (device override)
- `traffic_month_threshold` - Monthly threshold (device override)
- All other `traffic_*` parameters can be device-specific

---

## Parameter Categories Summary

### By Function

1. **Network Connectivity** (~80 parameters)
   - WAN1 (Cellular): 40+ parameters
   - WAN0 (Physical Port): 25+ parameters
   - WAN2 (WiFi STA): 30+ parameters
   - LAN: 15+ parameters

2. **Security** (~50 parameters)
   - Firewall: 20+ parameters
   - VPN (IPSec, L2TP, PPTP, OpenVPN, GRE): 60+ parameters (Global layer)
   - SSL/TLS: 5+ parameters
   - Authentication: 10+ parameters

3. **Services** (~40 parameters)
   - HTTP/HTTPS: 10 parameters
   - SSH: 8 parameters
   - Telnet: 6 parameters
   - SNMP: 10 parameters
   - DNS: 8 parameters

4. **Cellular-Specific** (~40 parameters)
   - Dual SIM: 10+ parameters
   - APN: 10+ parameters
   - Redial: 8 parameters
   - Band configuration: 3 parameters

5. **WiFi** (~30 parameters)
   - AP mode: 20+ parameters
   - WDS: 8 parameters

6. **Monitoring & Management** (~30 parameters)
   - RMON: 20+ parameters
   - Device Manager (MQTT): 13 parameters
   - Alarms: 5 parameters
   - Logging: 8 parameters

7. **Traffic Management** (~15 parameters)
   - Daily limits: 5 parameters
   - Monthly limits: 8 parameters
   - Custom rules: 4 parameters

8. **Hardware-Specific** (~20 parameters)
   - Digital I/O: 4 parameters
   - Serial ports: 10+ parameters
   - DTU: 30+ parameters

9. **System** (~30 parameters)
   - Time/NTP: 6 parameters
   - Hostname: 3 parameters
   - Scheduler: 4 parameters
   - System configuration: 10+ parameters

### By Static vs Dynamic

**Static Parameters** (~100-150)
- Never change once set for a model/carrier/service plan
- Examples: VPN disabled, specific model features, carrier APNs

**Dynamic Parameters** (~250-300)
- Can be overridden at lower layers
- Examples: Hostname, IP addresses, firewall rules, traffic thresholds

### By Customer-Configurable Flag

**Admin-Only** (~300-350 parameters)
- Global defaults
- Model-specific features
- Carrier-specific settings
- Most security settings
- VPN configurations

**Potentially Customer-Configurable** (~50-100 parameters)
- Firewall ACL (whitelist additions)
- Traffic management thresholds
- DHCP settings
- LAN IP configuration
- WiFi settings
- Hostname
- Some WAN settings (for devices with physical WAN port)

---

## Key Insights for Config System Design

### 1. Layer Distribution
- **Global layer** contains the vast majority of parameters (~60-70%)
- **Model layer** is relatively small but critical for hardware differences (~10%)
- **Carrier layer** focuses on cellular connectivity (~12%)
- **Service Plan layer** focuses on security and usage limits (~5%)
- **Company/Customer layer** is minimal but important for customization (~3%)
- **Device layer** can override most parameters but typically only a subset (~10%)

### 2. Inheritance Patterns
- Most devices inherit 90%+ of their config from upper layers
- Device-specific configs typically limited to:
  - SIM bindings
  - IP addresses
  - Hostnames
  - Custom firewall rules
  - WiFi settings (if applicable)
  - Cellular backup settings (if applicable)

### 3. Critical Parameters for Customer Self-Service
Customers should be able to configure:
- Firewall whitelist additions (specific IPs/domains)
- Traffic management thresholds
- Hostname
- DHCP pool settings
- WiFi SSID and password (if WiFi-enabled model)
- Device-specific IP addressing (for devices with physical WAN)

### 4. Model-Specific Feature Flags
Key differentiators between models:
- **I-22/I-52:** Digital I/O, Alarms, SMS controls, MQTT device manager
- **IR611/IR615:** Link backup, no I/O, no SMS controls
- **Carrier dependency:** MQTT device manager (Verizon/TMO only, not AT&T)

### 5. Validation Requirements
- Carrier APNs must match carrier layer
- Service plan firewall rules must not be customer-editable
- SIM bindings must match physical SIM ICCID
- Traffic thresholds must not exceed service plan maximums
- Model-specific features must not be enabled on incompatible models

### 6. Config Groups/Categories
For UI organization:
1. Network (WAN, LAN, DHCP, DNS)
2. Security (Firewall, VPN)
3. Cellular (Dual SIM, APN, Signal)
4. WiFi (AP settings, STA mode)
5. Monitoring (RMON, Alarms, Device Manager)
6. Traffic Management (Limits, Actions)
7. System (Time, Hostname, Services)
8. Hardware (I/O, Serial ports)

---

## Next Steps for PRD

1. **Create Config Key Schema** with all ~400 parameters
   - Define data types (string, integer, boolean, enum, IP address, etc.)
   - Define layer availability matrix
   - Define required vs optional
   - Define customer-configurable flag
   - Define static vs dynamic flag
   - Group by category

2. **Define Validation Rules**
   - Cross-layer validation (e.g., device can't exceed service plan limits)
   - Model compatibility validation
   - Carrier compatibility validation
   - IP address format validation
   - Dependency validation (e.g., WiFi settings only valid if WiFi enabled)

3. **Design UI Components**
   - Reusable layer editor component
   - Category-based navigation
   - Effective config view with inheritance visualization
   - Customer self-service filtered view

4. **Plan Migration Strategy**
   - Map existing 200 config files to new hierarchy
   - Identify overlaps and conflicts
   - Create migration scripts
   - Define rollback procedures

---

**End of Analysis**
