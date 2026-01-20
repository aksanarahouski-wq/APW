# Configuration Parameter Usage Analysis

Analysis of 26 configuration files from `/Configurations/Files`

**Analysis Date:** 2025-10-16

## Summary

- **Total unique parameters:** 694
- **Always Empty (0% usage):** 302 parameters
- **Mostly Empty (<20% usage):** 31 parameters
- **Sometimes Used (20-80% usage):** 41 parameters
- **Commonly Used (>80% usage):** 320 parameters

## Usage Categories

### 1. Always Empty Parameters (302)

These parameters are present in configuration files but **never have values**:

```
adm_users
alarm_clear
alarm_confirm
backup_icmp_host
backup_sim_policy_using_time
cert_ca
cert_crl
cert_key
cert_private
cert_public
com0_hw_flow
com0_sw_flow
com1_hw_flow
com1_sw_flow
com4_hw_flow
com4_sw_flow
console_description
cron_rb_days
ct_tcp_timeout
ct_udp_timeout
ddnsx0
ddnsx0_cache
ddnsx1
ddnsx1_cache
ddnsx2
ddnsx2_cache
ddnsx3
ddnsx3_cache
dhcpd_option_domain
dhcpd_slt
dhcpd_static
dhcpd_wins
dmz0_mac
dmz0_mip
dmz_iface
dmz_ip
dmz_src
dmz_tunnel
dns_proxy_disable
dnsmasq_custom
dnsrelay_static
dtu_connection_mode
dtu_crlf
dtu_downstr
dtu_id
dtu_id_trap_intv
dtu_ma8_hb_context
dtu_ma8_hb_format
dtu_ma8_hb_used
dtu_mserver_policy
dtu_protocol
dtu_server
dtu_server_mode
dtu_src
dtu_tcp_mode
dtu_upstr
dual_sim_csq_retry
engineer_mode
env_path
fw_block_activex
fw_block_applet
fw_block_cookie
fw_block_ident
fw_block_loopback
fw_block_proxy
fw_block_wan
fw_ics
fw_mac_rules
fw_mac_rules_back
fw_mac_rules_saved
fw_portmap
fw_router_vip
fw_strict
fw_sys_acl
fw_vip
fw_vip_range
gre_tunnels
http_api_description
http_api_src
http_description
http_src
https_description
ike_policies
io_triggered_report
ip_passthrough_mac
ipsec_debug
ipsec_dynnattport
ipsec_force_natt
ipsec_nocrsend
ipsec_policies
ipsec_tunnel_action
ipsec_tunnel_name
ipsec_tunnels
l2tp_debug
l2tpc_tunnel_action
l2tpc_tunnel_name
l2tpc_tunnels
l2tps_debug
l2tps_expert
l2tps_localip
l2tps_mppe
l2tps_passwd
l2tps_remoteip
l2tps_remotemask
l2tps_remotenet
l2tps_username
lan0_gateway
lan0_mdix
lan0_mip
lan0_mode
lan0_server
linkbackup_restart_wan
linkmanager_debug
log_console
log_mark
log_remote
log_remoteip
mqtt_atm_id
mqtt_device_info
mqtt_sitename
nf_ttl
ntp_kiss
openvpn_dns2
openvpn_dns3
openvpn_dns4
openvpn_ovpn2
openvpn_ovpn3
openvpn_ovpn4
openvpn_secret1
openvpn_secret2
openvpn_secret3
openvpn_secret4
openvpn_user_list
ovdp_entity_conf
ovdp_trust_list
port_mode
pptpc_tunnel_action
pptpc_tunnel_name
pptpc_tunnels
pptps_debug
pptps_expert
pptps_localip
pptps_mppe
pptps_passwd
pptps_remoteip
pptps_remotemask
pptps_remotenet
pptps_username
qos_default
qos_irates
qos_old_iface
qos_orates
qoslimit_rule
qoslimit_rule_old
redial_sim1_profile
redial_sim1_profile2
redial_sim2_profile
redial_sim2_profile2
rmon_geolocation
rmon_io_value
rmon_tls_sni
routes_static
routes_static_saved
rstats_exclude
scep_challenge
scep_common_name
scep_domain
scep_fqdn
scep_req
scep_serialno
scep_status
scep_transid
scep_unaddr
scep_unit1
scep_unit2
scep_url
schedule_list
sim1_binding_iccid
sim2_binding_iccid
sms_acl
sms_ctrl_cmd
sms_strict
snmpd_comlist
snmpd_grouplist
snmpd_syscontact
snmpd_syslocation
snmpd_userlist
snmpd_version
snmptrap_server_ip
sshd_authkeys
sshd_description
sshd_pass
sshd_remote
sshd_src
ssl_proxy_processor_host
ssl_proxy_processor_port
ssl_proxy_tcp_server_addr
ssl_proxy_tcp_server_port
telnet_description
telnet_src
traffic_custom_rule
traffic_exceed_report
traffic_month2_action
traffic_month2_threshold
traffic_month_alarm
traffic_month_discon
traffic_month_threshold
vrrpd0_mon
vrrpd0_vip
vrrpd0_vmac
vrrpd1_mon
vrrpd1_vip
vrrpd1_vmac
wan0_debug
wan0_icmp_host
wan0_mip
wan0_ppp_ac
wan0_ppp_am
wan0_ppp_hostuniq
wan0_ppp_idletime
wan0_ppp_ip
wan0_ppp_iph_comp
wan0_ppp_mode
wan0_ppp_options
wan0_ppp_passwd
wan0_ppp_peer
wan0_ppp_service
wan0_ppp_static
wan0_ppp_username
wan0_schedule
wan1_bridge_mode
wan1_debug
wan1_debug_modem
wan1_ip
wan1_mip
wan1_name
wan1_ppp_am
wan1_ppp_authen
wan1_ppp_freq_select
wan1_ppp_idletime
wan1_ppp_ip
wan1_ppp_mode
wan1_ppp_modem
wan1_ppp_modem_mode
wan1_ppp_network_select
wan1_ppp_operator_list
wan1_ppp_passwd
wan1_ppp_pincode
wan1_ppp_sim2_authen
wan1_ppp_sim2_network
wan1_ppp_sim2_passwd
wan1_ppp_sim2_pincode
wan1_ppp_sim2_username
wan1_ppp_static
wan1_ppp_username
wan1_renewing
wan1_schedule
wan1_sms_down
wan1_sms_up
wan1_trig_sms
wan2_debug
wan2_icmp_host
wan2_mac
wan2_mip
wan2_ppp_ac
wan2_ppp_am
wan2_ppp_hostuniq
wan2_ppp_idletime
wan2_ppp_ip
wan2_ppp_iph_comp
wan2_ppp_mode
wan2_ppp_options
wan2_ppp_passwd
wan2_ppp_peer
wan2_ppp_service
wan2_ppp_static
wan2_ppp_username
wan2_schedule
wan3_ppp_authen
wan3_ppp_passwd
wan3_ppp_sim2_authen
wan3_ppp_sim2_passwd
wan3_ppp_sim2_provider
wan3_ppp_sim2_username
wan3_ppp_username
wan_backup_mode
wan_backup_rule
wireguard_tunnel_action
wireguard_tunnel_name
wireguard_tunnels
wl0_acl
wl0_acl_list
wl0_bridge
wl0_bw
wl0_encrypt
wl0_gkey_cycle
wl0_radius_idle_timeout
wl0_radius_session_timeout
wl0_wds_auth
wl0_wds_bssid
wl0_wds_encrypt
wl0_wds_ssid
```

### 2. Mostly Empty Parameters (31)

These parameters are **rarely used** (less than 20% of files):

| Parameter | Usage | Sample Values |
|-----------|-------|---------------|
| alarm_input | 5/26 (19%) | 0,0,1,1,0,0,0,0,0,0,0, |
| alarm_output | 5/26 (19%) | 0,0,1, |
| dtu1_enable | 4/26 (15%) |  |
| dual_sim_min_conn_time | 3/26 (12%) | 180 |
| dual_sim_min_csq | 4/26 (15%) | 5 |
| garp_brdcast_cnt | 3/26 (12%) | 5 |
| garp_brdcast_timeout | 3/26 (12%) | 10 |
| garp_enable | 4/26 (15%) | 0 |
| gsm_wcdma_band_config | 4/26 (15%) | ALL |
| hardware_ready | 3/26 (12%) | 1 |
| hw_reset_disable | 4/26 (15%) | 1 |
| ip_passthrough_dhcp_lease | 3/26 (12%) | 2 |
| ip_passthrough_enable | 3/26 (12%) | 0 |
| ip_passthrough_mode | 3/26 (12%) | dhcp-dynamic |
| lan_port3 | 4/26 (15%) | 1 |
| lan_port4 | 4/26 (15%) | 1 |
| lte_band_config | 4/26 (15%) | ALL |
| main_icmp_host | 2/26 (8%) | 8.8.8.8, 1.1.1.1 |
| ovdp_mode | 3/26 (12%) | 2 |
| portal_enable | 4/26 (15%) |  |
| serialnum | 1/26 (4%) | RF3022230147778 |
| smbc_enable | 3/26 (12%) | 0 |
| sysconf_timestamp | 3/26 (12%) | c-1722889882684, c-1754327034002 |
| traffic_sms_up | 3/26 (12%) | online |
| vlan_l2_config | 5/26 (19%) | 1,1111; |
| vlan_l3_config | 5/26 (19%) | 1,0,192.168.1.90,255.255.255.0,1500; |
| vlan_port_mode | 5/26 (19%) | 1,1,0,0,1;2,1,0,0,1;3,1,0,0,1;4,1,0,0,1; |
| wan1_ppp_operator | 3/26 (12%) | auto |
| wl0_auth | 3/26 (12%) | 8, 5 |
| wl0_radius_ip | 4/26 (15%) | 192.168.2.2 |
| wl0_radius_key | 4/26 (15%) | 123456 |

### 3. Sometimes Used Parameters (41)

These parameters are **moderately used** (20-80% of files):

| Parameter | Usage | Sample Values |
|-----------|-------|---------------|
| backup_sim_policy_enable | 7/26 (27%) | 0 |
| backup_sim_policy_revert_day | 7/26 (27%) | 1 |
| cron_rb_time | 18/26 (69%) | 225 |
| digitalio_config | 14/26 (54%) | 1,0,0;1,0,0; |
| dtu1_iface | 16/26 (62%) | /dev/ttyS0 |
| dual_sim_main | 8/26 (31%) | 1 |
| fw_nat | 9/26 (35%) | 1<1<1<0.0.0.0/0<<0.0.0.0/0<2222<<192.168.1.111<22<..., 1<1<3<0.0.0.0/0<<0.0.0.0/0<1440<<67.23.48.67<1440<... |
| fw_web | 9/26 (35%) | 1<libertyx.com<1<1<>1<atm1.switchcommerce.net<1<1<..., 1<libertyx.com<1<1<>1<atmssl.dnsatm.com<1<1<DNS>1<... |
| http_api_enable | 12/26 (46%) | 1, 0 |
| http_api_local | 12/26 (46%) | 1 |
| http_api_port | 13/26 (50%) | 4444 |
| http_api_remote | 12/26 (46%) | 1 |
| io_chip | 10/26 (38%) | 1 |
| iodigital_input_data | 9/26 (35%) | 00, 0c |
| lan_port1 | 18/26 (69%) | 1 |
| lan_port2 | 20/26 (77%) | 1 |
| model_name | 8/26 (31%) | $AES$CC649CBCB1D17FB157D72EA071282666, $AES$B1E080FCB23738E4D10401164A95D0C7 |
| mqtt_center | 8/26 (31%) | iot.inhandnetworks.com |
| mqtt_keepalive | 11/26 (42%) | 60, 30 |
| mqtt_tls | 11/26 (42%) | 1 |
| mqtt_username | 8/26 (31%) | service@wirelessatmstore.com |
| oem_name | 8/26 (31%) | $AES$AB5416F1268EBBD2E785E5AB90490B4B |
| openvpn_dns1 | 18/26 (69%) | 1 |
| openvpn_ovpn1 | 16/26 (62%) | dev tun\0Apersist-tun\0Apersist-key\0Acipher AES-2... |
| openvpn_tunnel_action | 18/26 (69%) | edit |
| openvpn_tunnel_name | 18/26 (69%) | OpenVPN_T_1 |
| openvpn_tunnels | 16/26 (62%) | OpenVPN_T_1,0,2,0,1194,,0,atm,Ifhe@#hIO2ipVIe&,,25... |
| ospf_enable | 18/26 (69%) | 0 |
| qos_iface | 17/26 (65%) | wan1 |
| redial_reboot | 7/26 (27%) | 1 |
| rmon_ipaddr2 | 14/26 (54%) | 1 |
| sms_apn | 7/26 (27%) | APN |
| sms_network_provider | 7/26 (27%) | NET |
| sms_rb | 7/26 (27%) | REB |
| sms_sq | 7/26 (27%) | STA |
| sms_switch_main_sim | 7/26 (27%) | SIM |
| stategrid_enable | 17/26 (65%) |  |
| traffic_custom_actions | 13/26 (50%) | 0,0,0,0 |
| wan1_icmp_detect_enable | 7/26 (27%) | 0 |
| wan1_mtu_enable | 17/26 (65%) | 0 |
| wwan0_default_route | 17/26 (65%) | 1 |

### 4. Commonly Used Parameters (320)

These parameters are **frequently used** (more than 80% of files):

| Parameter | Usage | Sample Values |
|-----------|-------|---------------|
| adm_passwd | 26/26 (100%) | $AES$2B1A2AA21A3C2A792AD50857C21AE55C |
| adm_user | 26/26 (100%) | matrixatm1 |
| advanced | 26/26 (100%) | 1, 0 |
| alarm_input_options | 26/26 (100%) | fault-service,memory-low,port0-wan-link-up/down,po..., fault-service,memory-low,port0-lan-link-up/down,di... |
| alarm_output_options | 26/26 (100%) | cli,, cli,out-dm,out-rmon, |
| auto_ping_enable | 26/26 (100%) | 0 |
| com0_config | 26/26 (100%) | 115200 8N1 |
| com1_config | 26/26 (100%) | 115200 8N1 |
| com4_config | 26/26 (100%) | 19200 8N1 |
| console_enable | 26/26 (100%) | 1, 0 |
| console_iface | 26/26 (100%) | /dev/ttyS1 |
| cron_rb_enable | 26/26 (100%) | 0 |
| ct_max | 26/26 (100%) | 2048 |
| debug_keepfiles | 26/26 (100%) | 1 |
| description | 26/26 (100%) | www.inhandnetworks.com |
| dhcpd_enable | 26/26 (100%) | 1 |
| dhcpd_end | 26/26 (100%) | 192.168.3.254, 192.168.1.254 |
| dhcpd_ifname | 26/26 (100%) | lan0 |
| dhcpd_lease | 26/26 (100%) | 60, 600 |
| dhcpd_start | 26/26 (100%) | 192.168.1.1, 192.168.1.100 |
| dialwh_conn_cmd | 26/26 (100%) | CONNECT |
| dialwh_conn_timeout | 26/26 (100%) | 60 |
| dialwh_disconn_cmd | 26/26 (100%) | DOWN |
| dialwh_disconn_timeout | 26/26 (100%) | 30 |
| dialwh_enable | 26/26 (100%) | 0 |
| dialwh_port | 26/26 (100%) | 6001 |
| dialwh_proto | 26/26 (100%) | TCP |
| dmz0_iface | 26/26 (100%) | none |
| dmz0_ip | 26/26 (100%) | 192.168.3.1 |
| dmz0_mtu | 26/26 (100%) | 1500 |
| dmz0_mtu_enable | 26/26 (100%) | 0 |
| dmz0_name | 26/26 (100%) | dmz0 |
| dmz0_netmask | 26/26 (100%) | 255.255.255.0 |
| dmz0_proto | 26/26 (100%) | none |
| dmz_enable | 26/26 (100%) | 0 |
| dns_addget | 26/26 (100%) | 1 |
| dns_static | 26/26 (100%) | 8.8.8.8;8.8.4.4, 8.8.8.8;1.1.1.1 |
| dnsrelay | 26/26 (100%) | 1 |
| dnsrelay_enable | 26/26 (100%) | 1 |
| domainname | 26/26 (100%) | inhand-router.com |
| down_bandwidth | 26/26 (100%) | 1000 |
| dtu_buf_size | 23/26 (88%) | 10240 |
| dtu_conn_idle | 23/26 (88%) | 30 |
| dtu_conn_interval | 23/26 (88%) | 5 |
| dtu_conn_retry_max | 23/26 (88%) | 180 |
| dtu_conn_retry_min | 23/26 (88%) | 15 |
| dtu_conn_retry_num | 23/26 (88%) | 5 |
| dtu_enable | 23/26 (88%) | 0 |
| dtu_frameval | 23/26 (88%) | 100 |
| dtu_hbnum | 23/26 (88%) | 5 |
| dtu_hbval | 23/26 (88%) | 60 |
| dtu_iface | 26/26 (100%) | /dev/ttyS1 |
| dtu_ma8_enable | 23/26 (88%) | 0 |
| dtu_port | 23/26 (88%) | 502 |
| dtu_str_format | 23/26 (88%) | 1 |
| dtu_timeout | 23/26 (88%) | 120 |
| dtu_uartframe_num | 23/26 (88%) | 4 |
| dual_sim_enable | 26/26 (100%) | 1, 0 |
| dual_sim_max_retry | 26/26 (100%) | 5 |
| fw_acl | 26/26 (100%) | 1<4<0.0.0.0/0<<34.199.0.0/16<<1<1<Amazon Resource>..., 1<4<192.168.1.0/24<<34.199.0.0/16<<1<1<Amazon Reso... |
| fw_anti_dos | 26/26 (100%) | 1 |
| fw_block_multicast | 26/26 (100%) | 1 |
| fw_clear_conntrack | 26/26 (100%) | 1 |
| fw_log_limit | 26/26 (100%) | 60 |
| gps_enable | 23/26 (88%) | 0 |
| hostname | 26/26 (100%) | DC_22_06012023, VZW_4100_ATM_09162025 |
| http_enable | 26/26 (100%) | 1 |
| http_local | 26/26 (100%) | 1 |
| http_port | 26/26 (100%) | 80 |
| http_remote | 26/26 (100%) | 1 |
| https_enable | 26/26 (100%) | 0 |
| https_local | 26/26 (100%) | 1 |
| https_port | 26/26 (100%) | 443 |
| https_remote | 26/26 (100%) | 1 |
| https_src | 23/26 (88%) | 173.15.161.73 |
| ipsec_compress | 26/26 (100%) | 1 |
| ipsec_natt_enable | 26/26 (100%) | 1 |
| ipsec_natt_interval | 26/26 (100%) | 60 |
| ipsec_stack | 26/26 (100%) | netkey |
| ipsec_uniqueids | 26/26 (100%) | 1 |
| l2tps_enable | 23/26 (88%) | 0 |
| l2tps_interval | 23/26 (88%) | 60 |
| l2tps_retry | 23/26 (88%) | 5 |
| lan0_default_route | 26/26 (100%) | 1 |
| lan0_iface | 26/26 (100%) | lan0, eth2.1 |
| lan0_ip | 26/26 (100%) | 192.168.1.90, 192.168.2.1 |
| lan0_last_ip | 26/26 (100%) | 192.168.1.90, 192.168.2.1 |
| lan0_last_netmask | 26/26 (100%) | 255.255.255.0 |
| lan0_mac | 26/26 (100%) | 00:18:05:0F:99:17, 00:18:05:2D:B7:7A |
| lan0_mtu | 26/26 (100%) | 1500 |
| lan0_mtu_enable | 26/26 (100%) | 0 |
| lan0_name | 26/26 (100%) | lan0 |
| lan0_netmask | 26/26 (100%) | 255.255.255.0 |
| lan0_proto | 26/26 (100%) | static |
| language | 26/26 (100%) | English |
| linkbackup_enable | 26/26 (100%) | 1, 0 |
| linkbackup_hot_mode | 26/26 (100%) | failover, 1 |
| log_crond | 26/26 (100%) | 1 |
| log_level | 26/26 (100%) | 7 |
| log_remoteport | 26/26 (100%) | 514 |
| login_timeout | 26/26 (100%) | 500 |
| main_icmp_interval | 26/26 (100%) | 10 |
| main_icmp_retries | 26/26 (100%) | 3 |
| main_icmp_timeout | 26/26 (100%) | 3 |
| main_successive_pkts_rcvd_for_up | 21/26 (81%) | 1 |
| max_modem_reset | 26/26 (100%) | 120, 30 |
| max_ppp_redial | 26/26 (100%) | 10 |
| mqtt_enable | 26/26 (100%) | 1, 0 |
| mqtt_experience_confirmed | 26/26 (100%) | 1 |
| mqtt_experience_mode | 26/26 (100%) | disable, experience |
| mqtt_lbs_interval | 26/26 (100%) | 1, 24 |
| mqtt_series_interval | 26/26 (100%) | 1, 24 |
| mqtt_sniffer_enable | 26/26 (100%) | 1 |
| mqtt_sniffer_filter_enable | 26/26 (100%) | 0 |
| multicast_pass | 26/26 (100%) | 1 |
| ntp_server | 26/26 (100%) | time.nist.gov;time.google.com;10.4.6.30, 74.118.247.208;time.google.com;time.nist.gov |
| ntp_tdod | 21/26 (81%) | 1 |
| ntp_updates | 23/26 (88%) | 24 |
| openvpn_c2c_enable | 26/26 (100%) | 0 |
| ovdp_center | 26/26 (100%) | g.inhandnetworks.com |
| ovdp_center_port | 23/26 (88%) | 20003 |
| ovdp_device_id | 23/26 (88%) | 302276854, 611445393 |
| ovdp_enable | 23/26 (88%) | 0 |
| ovdp_hb_interval | 23/26 (88%) | 120 |
| ovdp_hb_retries | 23/26 (88%) | 3 |
| ovdp_hb_timeout | 23/26 (88%) | 3600 |
| ovdp_login_retries | 23/26 (88%) | 3 |
| ovdp_rx_timeout | 23/26 (88%) | 30 |
| ovdp_sms_interval | 23/26 (88%) | 24 |
| ovdp_tx_retries | 23/26 (88%) | 3 |
| ovdp_vendor_id | 23/26 (88%) | 0003 |
| pam_shield_interval | 26/26 (100%) | 600 |
| pam_shield_retention | 26/26 (100%) | 3600 |
| pam_shield_retry | 26/26 (100%) | 20 |
| pptps_enable | 23/26 (88%) | 0 |
| pptps_interval | 23/26 (88%) | 60 |
| pptps_retry | 23/26 (88%) | 5 |
| qos_ack | 26/26 (100%) | 1 |
| qos_burst0 | 26/26 (100%) | 64 |
| qos_burst1 | 26/26 (100%) | 64 |
| qos_enable | 26/26 (100%) | 0 |
| qos_ibw | 26/26 (100%) | 100000 |
| qos_icmp | 26/26 (100%) | 1 |
| qos_inuse | 26/26 (100%) | 1 |
| qos_method | 26/26 (100%) | 1 |
| qos_obw | 26/26 (100%) | 100000 |
| qos_reset | 26/26 (100%) | 1 |
| qoslimit_enable | 26/26 (100%) | 0 |
| redial_dual_apn_enable | 21/26 (81%) | 0 |
| redial_ppp_profiles | 21/26 (81%) | 1:matrix.com.attz:*99#:0::;2:Matrxatm.gw12.vzwentp..., 1:MATRXATM.GW12.VZWENTP:*99#:0::; |
| rmon_advance_cfg | 25/26 (96%) | _wan1_imei,_wan1_iccid,, _wan1_imei,_wan1_iccid |
| rmon_cur_softver | 25/26 (96%) | 1 |
| rmon_enable | 25/26 (96%) | 1 |
| rmon_hostname | 25/26 (96%) | 1 |
| rmon_httpapi_enable | 25/26 (96%) | 1 |
| rmon_ipaddr | 25/26 (96%) | 1 |
| rmon_log_enable | 25/26 (96%) | 1 |
| rmon_passwd | 25/26 (96%) | test |
| rmon_protocol | 25/26 (96%) | 2 |
| rmon_rep_interval | 25/26 (96%) | 120 |
| rmon_report_args | 25/26 (96%) | 1 |
| rmon_serial_num | 25/26 (96%) | 1 |
| rmon_server_domain | 25/26 (96%) | apcommand.com |
| rmon_server_port | 25/26 (96%) | 8002 |
| rmon_signal_strength | 25/26 (96%) | 1 |
| rmon_terminal_id | 25/26 (96%) | 1 |
| rmon_timestamp | 25/26 (96%) | 1 |
| rmon_uptime | 25/26 (96%) | 1 |
| rmon_user | 25/26 (96%) | Test_4, test |
| router_name | 26/26 (100%) | Router |
| scep_enable | 26/26 (100%) | 0 |
| scep_key_len | 26/26 (100%) | 1024 |
| scep_poll_interval | 26/26 (100%) | 60 |
| scep_poll_retries | 26/26 (100%) | 3 |
| scep_poll_timeout | 26/26 (100%) | 3600 |
| sim1_card_operator | 26/26 (100%) | 2 |
| sim2_card_operator | 21/26 (81%) | 1, 2 |
| sms_enable | 26/26 (100%) | 1, 0 |
| snmpd_enable | 23/26 (88%) | 0 |
| snmpd_port | 23/26 (88%) | 161 |
| snmptrap_signal_level | 23/26 (88%) | 10 |
| sshd_enable | 26/26 (100%) | 0 |
| sshd_hostkey | 26/26 (100%) | AAAAB3NzaC1yc2EAAAADAQABAAAAgwCSf/IA51I6hM8C8QxYzF... |
| sshd_hostkey2 | 26/26 (100%) | AAAAB3NzaC1kc3MAAACBAKQivtUbDu4VxzhuBZXjElN3E5Q44W... |
| sshd_local | 26/26 (100%) | 1 |
| sshd_port | 26/26 (100%) | 22 |
| ssl_proxy_enable | 23/26 (88%) | 1 |
| ssl_server | 23/26 (88%) | 1:1:1:7000:atm.columbusdata.net:6965:0:0:0;2:1:1:7... |
| t_cafree | 26/26 (100%) | 1234 |
| telnet_enable | 26/26 (100%) | 1, 0 |
| telnet_local | 26/26 (100%) | 1 |
| telnet_port | 26/26 (100%) | 50023, 23 |
| telnet_remote | 23/26 (88%) | 1 |
| tm_dst | 24/26 (92%) | 1 |
| tm_sel | 26/26 (100%) | custom, EST5EDT,M3.2.0/2,M11.1.0/2 |
| tm_tz | 26/26 (100%) | EST5, EST5EDT,M3.2.0/2,M11.1.0/2 |
| traffic_custom_enable | 26/26 (100%) | 0 |
| traffic_day_action | 26/26 (100%) | 1 |
| traffic_day_threshold | 23/26 (88%) | 3584, 3 |
| traffic_day_unit | 23/26 (88%) | 1, 2 |
| traffic_enable | 26/26 (100%) | 1, 0 |
| traffic_month2_unit | 25/26 (96%) | 1 |
| traffic_month_action | 23/26 (88%) | 1 |
| traffic_month_start_day | 26/26 (100%) | 1 |
| traffic_month_unit | 26/26 (100%) | 1, 2 |
| up_bandwidth | 26/26 (100%) | 1000 |
| vrrpd0_adv_interval | 23/26 (88%) | 60 |
| vrrpd0_auth | 23/26 (88%) | none |
| vrrpd0_enable | 23/26 (88%) | 0 |
| vrrpd0_iface | 26/26 (100%) | lan0 |
| vrrpd0_prio | 23/26 (88%) | 20 |
| vrrpd0_vrid | 23/26 (88%) | 1 |
| vrrpd1_adv_interval | 23/26 (88%) | 60 |
| vrrpd1_auth | 23/26 (88%) | none |
| vrrpd1_enable | 23/26 (88%) | 0 |
| vrrpd1_iface | 26/26 (100%) | lan0 |
| vrrpd1_prio | 23/26 (88%) | 10 |
| vrrpd1_vrid | 23/26 (88%) | 2 |
| wan0_default_route | 26/26 (100%) | 1 |
| wan0_gateway | 26/26 (100%) | 192.168.1.1 |
| wan0_icmp_interval | 26/26 (100%) | 30 |
| wan0_icmp_retries | 26/26 (100%) | 3 |
| wan0_icmp_timeout | 26/26 (100%) | 20 |
| wan0_iface | 26/26 (100%) | none, eth2.2 |
| wan0_ip | 26/26 (100%) | 192.168.1.29 |
| wan0_lan_mode | 21/26 (81%) | lan, wan |
| wan0_mac | 26/26 (100%) | 00:18:05:2D:BE:19, 00:18:05:1A:28:E4 |
| wan0_mtu | 26/26 (100%) | 1500 |
| wan0_mtu_enable | 26/26 (100%) | 0 |
| wan0_netmask | 26/26 (100%) | 255.255.255.0 |
| wan0_ppp_check_interval | 26/26 (100%) | 55 |
| wan0_ppp_check_retries | 26/26 (100%) | 10 |
| wan0_ppp_mru | 26/26 (100%) | 1500 |
| wan0_ppp_peerdns | 26/26 (100%) | 1 |
| wan0_ppp_redial_interval | 26/26 (100%) | 30 |
| wan0_ppp_txql | 26/26 (100%) | 3 |
| wan0_proto | 26/26 (100%) | none, disabled |
| wan0_shared | 26/26 (100%) | 1 |
| wan0_trig_data | 26/26 (100%) | 1 |
| wan127_ppp_check_interval | 26/26 (100%) | 30 |
| wan1_band_config | 26/26 (100%) | ALL;ALL; |
| wan1_default_route | 26/26 (100%) | 1 |
| wan1_icmp_host | 26/26 (100%) | 74.118.247.208, 10.4.6.30 |
| wan1_icmp_interval | 26/26 (100%) | 3600, 3000 |
| wan1_icmp_mode | 26/26 (100%) | 1 |
| wan1_icmp_retries | 26/26 (100%) | 5 |
| wan1_icmp_timeout | 26/26 (100%) | 20 |
| wan1_iface | 26/26 (100%) | /dev/ttyUSB3, /dev/ttyUSB0 |
| wan1_mtu | 26/26 (100%) | 1430, 1500 |
| wan1_ppp_apn | 26/26 (100%) | Matrxatm.gw12.vzwentp, matrix.com.attz |
| wan1_ppp_callno | 26/26 (100%) | *99#, *99***1# |
| wan1_ppp_check_interval | 26/26 (100%) | 55 |
| wan1_ppp_check_retries | 26/26 (100%) | 3, 6 |
| wan1_ppp_debug | 26/26 (100%) | 1 |
| wan1_ppp_init | 26/26 (100%) | AT |
| wan1_ppp_iph_comp | 26/26 (100%) | 1 |
| wan1_ppp_mru | 26/26 (100%) | 1500 |
| wan1_ppp_network | 26/26 (100%) | TD-LTE, FDD-LTE |
| wan1_ppp_options | 26/26 (100%) | nomppe nomppc nodeflate nobsdcomp novj novjccomp n... |
| wan1_ppp_peer | 26/26 (100%) | 1.1.1.3 |
| wan1_ppp_peerdns | 26/26 (100%) | 1 |
| wan1_ppp_provider | 21/26 (81%) | 1, 2 |
| wan1_ppp_redial_interval | 26/26 (100%) | 30 |
| wan1_ppp_sim2_apn | 21/26 (81%) | matrix.com.attz, MATRXATM.GW12.VZWENTP |
| wan1_ppp_sim2_callno | 26/26 (100%) | *99#, *99***1# |
| wan1_ppp_sim2_provider | 21/26 (81%) | 1, 2 |
| wan1_ppp_timeout | 26/26 (100%) | 120 |
| wan1_ppp_txql | 26/26 (100%) | 64 |
| wan1_proto | 26/26 (100%) | dialup |
| wan1_shared | 26/26 (100%) | 1 |
| wan1_trig_call | 26/26 (100%) | 1 |
| wan1_trig_data | 26/26 (100%) | 1 |
| wan1_uptime | 26/26 (100%) | 37 |
| wan2_default_route | 26/26 (100%) | 1 |
| wan2_gateway | 26/26 (100%) | 192.168.3.1 |
| wan2_icmp_interval | 26/26 (100%) | 30 |
| wan2_icmp_retries | 26/26 (100%) | 3 |
| wan2_icmp_timeout | 26/26 (100%) | 20 |
| wan2_iface | 26/26 (100%) | ra0 |
| wan2_ip | 26/26 (100%) | 192.168.3.29 |
| wan2_mtu | 26/26 (100%) | 1500 |
| wan2_mtu_enable | 26/26 (100%) | 0 |
| wan2_netmask | 26/26 (100%) | 255.255.255.0 |
| wan2_ppp_check_interval | 26/26 (100%) | 55 |
| wan2_ppp_check_retries | 26/26 (100%) | 10 |
| wan2_ppp_mru | 26/26 (100%) | 1500 |
| wan2_ppp_peerdns | 26/26 (100%) | 1 |
| wan2_ppp_redial_interval | 26/26 (100%) | 30 |
| wan2_ppp_txql | 26/26 (100%) | 3 |
| wan2_proto | 26/26 (100%) | none |
| wan2_shared | 26/26 (100%) | 1 |
| wan2_trig_data | 26/26 (100%) | 1 |
| wan3_default_route | 21/26 (81%) | 1 |
| wan3_mtu | 21/26 (81%) | 1430, 1500 |
| wan3_ppp_apn | 21/26 (81%) | matrix.com.attz, cmnet |
| wan3_ppp_provider | 21/26 (81%) | 1, China Mobile (GPRS/EDGE) |
| wan3_ppp_sim2_apn | 21/26 (81%) | matrix.com.attz, MATRXATM.GW12.VZWENTP |
| wan3_proto | 21/26 (81%) | disabled |
| wan3_shared | 21/26 (81%) | 1 |
| wan_backup_link | 26/26 (100%) | wan1 |
| wan_backup_retry | 26/26 (100%) | 3 |
| wan_backup_time | 26/26 (100%) | 3600 |
| wan_linkbackup_enable | 26/26 (100%) | 0 |
| wan_main_link | 26/26 (100%) | wan0 |
| wan_main_retry | 26/26 (100%) | 3 |
| wl0_ap | 26/26 (100%) | 1 |
| wl0_channel | 26/26 (100%) | 11 |
| wl0_enable | 26/26 (100%) | 0 |
| wl0_iface | 26/26 (100%) | ra0, none |
| wl0_mode | 26/26 (100%) | 9 |
| wl0_radius_port | 26/26 (100%) | 1812 |
| wl0_ssid | 25/26 (96%) | textone, CR202-002F96 |
| wl0_ssid_brdcast | 26/26 (100%) | 1 |
| wl0_wds_enable | 26/26 (100%) | 0 |
| wl0_wds_wep_key | 26/26 (100%) | 12345 |
| wl0_wds_wpa_encrypt | 26/26 (100%) | 2 |
| wl0_wds_wpa_psk | 26/26 (100%) | abcdefgh |
| wl0_wep_key | 26/26 (100%) | 12345 |
| wl0_wpa_encrypt | 26/26 (100%) | 4, 3 |
| wl0_wpa_psk | 25/26 (96%) | abcdefgh, tcc54930 |

## Recommendations for Configuration Editor

### Priority 1: Essential Fields (Commonly Used)
Focus UI development on the **320 commonly used parameters** that appear in >80% of configurations.

### Priority 2: Optional Fields (Sometimes Used)
Make the **41 sometimes used parameters** available in an 'Advanced' section.

### Priority 3: Rarely Used Fields
The **31 mostly empty parameters** can be hidden by default or shown only in 'Expert Mode'.

### Exclude from UI
The **302 always empty parameters** should likely be excluded from the UI entirely unless there's a specific future use case.

