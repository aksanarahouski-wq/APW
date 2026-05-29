# GlobalConfigs - Global.csv Summary

## Purpose
Base default configurations inherited by all devices. This is the foundation layer containing ~481 parameters that define system-wide defaults for administrative access, network services, security, VPN, serial communications, and more.

## Key Configuration Areas

### 1. Administrative Access (lines 2-5, 159-176, 386-393, 406-411)

**Login Credentials**:
- `adm_user=matrixatm1` - Default admin username
- `adm_passwd=$AES$2B1A2AA21A3C2A792AD50857C21AE55C` - Encrypted password
- `adm_users=` - Additional admin users (empty)
- `advanced=1` - Advanced features enabled

**HTTP/HTTPS Web Interface**:
- `http_enable=1` - HTTP enabled
- `http_port=80` - Standard HTTP port
- `http_local=1` - Allow local access
- `http_remote=1` - Allow remote access
- `http_src=` - Source IP restrictions (none)
- `https_enable=0` - HTTPS disabled by default
- `https_port=443` - Standard HTTPS port

**HTTP API**:
- `http_api_enable=1` - API enabled
- `http_api_port=4444` - Custom API port
- `http_api_local=1` - Local API access
- `http_api_remote=1` - Remote API access

**SSH Access**:
- `sshd_enable=0` - SSH disabled by default
- `sshd_port=22` - Standard SSH port
- `sshd_local=1` - Local access allowed
- `sshd_remote=0` - Remote access disabled
- `sshd_pass=0` - Password authentication disabled
- `sshd_authkeys=` - Authorized keys (empty)
- Includes RSA and DSA host keys

**Telnet Access**:
- `telnet_enable=0` - Telnet disabled by default
- `telnet_port=50023` - Non-standard port
- `telnet_local=1` - Local access
- `telnet_remote=1` - Remote access

### 2. Network Services

#### DHCP Server (lines 47-55)
```
dhcpd_enable=1                    # DHCP server enabled
dhcpd_start=192.168.1.100         # Pool start (customizable: Altech/Miele)
dhcpd_end=192.168.1.254           # Pool end (customizable: Altech/Miele)
dhcpd_ifname=lan0                 # Interface
dhcpd_lease=60                    # Lease time: 60 minutes
dhcpd_static=                     # Static DHCP reservations (MAC-IP binding)
dhcpd_wins=0.0.0.0               # WINS server (disabled)
dhcpd_option_domain=0            # Domain option disabled
```
**Note**: "Customer or Device Override Possible" - Altech/Miele customers use custom ranges
**Static Reservations**: "Device level override possible - IP MAC LOCK" (Miele)

#### DNS Configuration (lines 81-88)
```
dns_addget=1                              # Get DNS from provider
dns_proxy_disable=0                       # DNS proxy enabled
dns_static=8.8.8.8,1.1.1.1               # Google & Cloudflare DNS
dnsmasq_custom=                           # Custom DNS settings
dnsrelay_enable=1                         # DNS relay enabled
dnsrelay_static=                          # Static DNS entries
dnsrelay=1                                # DNS relay active
domainname=inhand-router.com              # Default domain
```

#### NTP (Time Synchronization) (lines 249-252)
```
ntp_server=10.4.6.30;time.nist.gov;time.google.com    # NTP servers (MUST be configurable)
ntp_tdod=1                                             # Time-of-day enabled
ntp_updates=24                                         # Update every 24 hours
```
**Critical**: "Must be configurable" - Internal NTP server (10.4.6.30) needs to be deployment-specific

#### Time Zone (lines 412-414, 498-500)
```
tm_dst=1                                  # Daylight Saving Time enabled
tm_sel=EST5EDT,M3.2.0/2,M11.1.0/2        # Eastern Time
tm_tz=EST5EDT,M3.2.0/2,M11.1.0/2         # Time zone string
```

### 3. Firewall & Security (lines 129-153, 159-201)

#### Basic Firewall Settings
```
fw_anti_dos=1                # Anti-DoS protection enabled
fw_block_activex=0           # Allow ActiveX
fw_block_applet=0            # Allow Java applets
fw_block_cookie=0            # Allow cookies
fw_block_ident=0             # Allow ident
fw_block_loopback=0          # Allow loopback
fw_block_multicast=1         # Block multicast
fw_block_proxy=0             # Allow proxy
fw_block_wan=0               # Allow WAN access
fw_clear_conntrack=1         # Clear connection tracking
fw_log_limit=60              # Log limit per minute
```

#### Advanced Firewall Features
```
fw_nat=                      # NAT rules (Dynamic: Customer/Device Override/Service Plan)
fw_acl=                      # Access control lists (varies by service plan)
fw_web=                      # Content filtering (Service Plan/Customer/Model specific)
fw_portmap=                  # Port mapping rules
fw_vip=                      # Virtual IP addresses
fw_dmz_enable=0              # DMZ disabled
fw_strict=0                  # Strict filtering disabled
multicast_pass=1             # Allow multicast pass-through
```

### 4. WAN Interfaces

#### WAN1 (Cellular Interface) (lines 587-655)
Comprehensive cellular configuration (~68 parameters):

**Basic Settings**:
```
wan1_proto=dialup            # Protocol: PPP dial-up
wan1_iface=/dev/ttyUSB3      # Serial interface
wan1_band_config=ALL;ALL;    # All cellular bands
wan1_default_route=1         # Use as default route
wan1_mtu=1500                # MTU size
```

**PPP Configuration**:
```
wan1_ppp_apn=Matrxatm.gw12.vzwentp    # Verizon APN (carrier-specific)
wan1_ppp_callno=*99#                   # Dial number
wan1_ppp_authen=0                      # No authentication
wan1_ppp_username=                     # No username
wan1_ppp_passwd=                       # No password
wan1_ppp_peerdns=1                     # Use carrier DNS
wan1_ppp_check_interval=55             # Keepalive: 55 seconds
wan1_ppp_check_retries=6               # 6 retry attempts
wan1_ppp_redial_interval=30            # Redial after 30 seconds
wan1_ppp_timeout=120                   # Connection timeout: 2 minutes
wan1_ppp_network=FDD-LTE               # Network type
wan1_ppp_options=nomppe nomppc nodeflate nobsdcomp novj novjccomp noccp  # Disable compression
```

**ICMP Monitoring**:
```
wan1_icmp_host=10.4.6.30          # Primary monitoring host (MUST be configurable)
wan1_icmp_backup_host=10.4.6.30   # Backup host
wan1_icmp_main_host=10.4.6.30     # Main host
wan1_icmp_interval=3600           # Check every hour
wan1_icmp_retries=5               # 5 retry attempts
wan1_icmp_timeout=20              # 20-second timeout
wan1_icmp_detect_enable=1         # Detection enabled
```

**SIM2 Configuration**:
```
wan1_ppp_sim2_apn=matrix.com.attz    # AT&T APN for SIM2
wan1_ppp_sim2_provider=1             # Provider profile
```

#### WAN0 (Physical WAN Port) (lines 437-465, 549-586)
```
wan0_proto=disabled          # Disabled by default
wan0_iface=eth2.2           # VLAN interface
wan0_ip=192.168.1.29        # Static IP (optional)
wan0_gateway=192.168.1.1    # Gateway
wan0_netmask=255.255.255.0  # Netmask
wan0_mtu=1500               # MTU size
wan0_icmp_host=             # ICMP monitoring
wan0_icmp_interval=30       # Check every 30 seconds
```

#### WAN2 (WiFi as WAN) (lines 656-692)
```
wan2_proto=none             # Disabled by default
wan2_iface=ra0              # WiFi interface
wan2_ip=192.168.3.29        # IP address
wan2_gateway=192.168.3.1    # Gateway
```
Supports PPPoE if needed (full PPP parameter set included)

#### WAN3 (Dual APN) (lines 693-705)
```
wan3_proto=disabled         # Disabled by default
wan3_ppp_apn=cmnet          # China Mobile APN
wan3_ppp_sim2_apn=matrix.com.attz  # AT&T for SIM2
```

### 5. LAN Interface (lines 213-223, 263-280)
```
lan0_proto=static           # Static IP configuration
lan0_ip=192.168.1.90        # LAN IP (varies by model/device)
lan0_netmask=255.255.255.0  # Subnet mask
lan0_iface=eth2.1           # VLAN interface
lan0_gateway=               # No gateway (LAN side)
lan0_mtu=1500               # MTU size
lan0_mac=                   # MAC address (inherited)
lan_port1=1                 # Port 1 enabled (model/device specific)
lan_port2=1                 # Port 2 enabled (model/device specific)
```

### 6. WiFi Configuration (lines 706-733)
```
wl0_enable=0                # WiFi disabled by default
wl0_ssid=inhand             # Default SSID
wl0_channel=11              # WiFi channel
wl0_mode=9                  # WiFi mode
wl0_ap=1                    # Access Point mode
wl0_auth=0                  # Authentication: open
wl0_encrypt=0               # Encryption: none
wl0_wpa_encrypt=2           # WPA encryption: AES
wl0_wpa_psk=abcdefgh        # WPA password
wl0_ssid_brdcast=1          # Broadcast SSID
wl0_bridge=0                # Bridge mode disabled
```

**RADIUS Support** (Enterprise WiFi):
```
wl0_radius_ip=192.168.2.2
wl0_radius_port=1812
wl0_radius_key=123456
```

**WDS (Wireless Distribution System)**:
```
wl0_wds_enable=0            # WDS disabled
wl0_wds_ssid=
wl0_wds_bssid=
```

### 7. VPN Support

#### IPSec VPN (lines 182-193, 233-245)
```
ipsec_tunnels=              # Tunnel definitions (empty)
ipsec_policies=             # Security policies
ipsec_stack=netkey          # Kernel stack
ipsec_natt_enable=1         # NAT traversal enabled
ipsec_natt_interval=60      # Keepalive: 60 seconds
ipsec_compress=1            # Compression enabled
ipsec_uniqueids=1           # Unique IDs required
ipsec_debug=0               # Debug disabled
ike_policies=               # IKE policies
```

#### L2TP VPN (lines 196-211, 247-262)
```
l2tpc_tunnels=              # Client tunnels (empty)
l2tps_enable=0              # Server disabled
l2tps_localip=              # Local IP pool
l2tps_remoteip=             # Remote IP pool
l2tps_remotenet=            # Remote network
l2tps_interval=60           # Keepalive
l2tps_retry=5               # Retry attempts
l2tps_mppe=0                # MPPE encryption disabled
```

#### PPTP VPN (lines 290-304, 360-374)
```
pptpc_tunnels=              # Client tunnels (empty)
pptps_enable=0              # Server disabled
pptps_localip=              # Local IP pool
pptps_remoteip=             # Remote IP pool
pptps_mppe=0                # MPPE disabled
```

#### OpenVPN (lines 255-267, 323-336)
```
openvpn_tunnels=            # Tunnel configurations
openvpn_c2c_enable=0        # Client-to-client disabled
openvpn_dns1=1              # Use DNS server 1
openvpn_ovpn1=              # OpenVPN config file
openvpn_secret1=            # Shared secret
```
**Note**: Full OpenVPN config with certificate embedded in FullConfigs.csv

#### GRE Tunnels (line 204)
```
gre_tunnels=                # GRE tunnel definitions (empty)
```

### 8. Serial Port / DTU Configuration (lines 16-24, 90-138)
Serial-to-IP bridge functionality:

```
com0_config=115200 8N1      # Serial port 0: 115200 baud, 8 data bits, no parity, 1 stop bit
com0_hw_flow=0              # Hardware flow control disabled
com0_sw_flow=0              # Software flow control disabled
com1_config=115200 8N1      # Serial port 1
com4_config=19200 8N1       # Serial port 4
```

**DTU (Data Transfer Unit)** - Serial-to-TCP/IP gateway:
```
dtu_enable=0                # DTU disabled by default
dtu_port=502                # Modbus TCP port
dtu_protocol=0              # Protocol type
dtu_iface=/dev/ttyS1        # Serial interface
dtu_buf_size=10240          # Buffer: 10 KB
dtu_timeout=120             # Timeout: 2 minutes
dtu_conn_idle=30            # Idle timeout: 30 seconds
```
**Note**: "These parameters are for serial port configuration which we do not utilize."

### 9. Smart ATM SSL Proxy (lines 394-400, 481-487)
```
ssl_proxy_enable=1          # SSL proxy enabled
ssl_server=1:1:1:7000:atm.columbusdata.net:6965:0:0:0;... [26+ entries]
```

**SSL Server Table** - Maps local ports to remote SSL services:
Format: `enable:ssl:proxy:local_port:remote_host:remote_port:auth:cert:verify;`

**Major entries (partial list)**:
- Port 7000 → atm.columbusdata.net:6965 (CDS)
- Port 7003 → 208.224.248.160:1440 (Switch Commerce)
- Port 561 → atmssl.dnsatm.com:8002
- Port 8444 → sslgb.1stiso.com:8444 (1st ISO)
- Port 447 → EMV.SIBISYSTEMS.Com:9056
- Port 7002 → pos.tnsi.com:5162 (TNSI)
- Many more payment processors...

**Note**: "Must be configurable for GLOBAL ADMIN"

### 10. QoS (Quality of Service) (lines 306-324, 376-394, 416, 520)
```
qos_enable=0                # QoS disabled by default
qos_iface=wan1              # Apply to cellular interface
qos_method=1                # HTB (Hierarchical Token Bucket)
qos_obw=100000              # Outbound bandwidth: 100 Mbps
qos_ibw=100000              # Inbound bandwidth: 100 Mbps
qos_ack=1                   # Prioritize ACK packets
qos_icmp=1                  # Prioritize ICMP
qos_default=0               # Default class
up_bandwidth=1000           # Upload: 1 Mbps
down_bandwidth=1000         # Download: 1 Mbps
```

### 11. Logging (lines 228-235, 284-291)
```
log_level=7                 # Debug level (highest)
log_console=0               # Console logging disabled
log_crond=1                 # Cron logging enabled
log_mark=0                  # Log marking disabled
log_remote=0                # Remote logging disabled
log_remoteip=               # Syslog server IP
log_remoteport=514          # Syslog port
login_timeout=500           # Login timeout: 500 seconds
```

### 12. Scheduler (lines 30-32, 44-46, 372)
```
cron_rb_enable=0            # Scheduled reboot disabled (device override possible)
cron_rb_time=225            # Reboot time: 02:25 AM
cron_rb_days=0              # Days of week (0=never)
schedule_list=              # Schedule definitions
```

### 13. Status Reporting (RMON) (lines 326-347, 403-424)
```
rmon_enable=1                       # Reporting enabled
rmon_protocol=2                     # Protocol: UDP
rmon_server_domain=apcommand.com    # Reporting server
rmon_server_port=8002               # Reporting port
rmon_rep_interval=120               # Report every 2 minutes
rmon_user=Test_4                    # Username
rmon_passwd=test                    # Password
rmon_hostname=1                     # Include hostname
rmon_ipaddr=1                       # Include IP address
rmon_signal_strength=1              # Include signal strength
rmon_serial_num=1                   # Include serial number
rmon_uptime=1                       # Include uptime
rmon_timestamp=1                    # Include timestamp
rmon_advance_cfg=_wan1_imei,_wan1_iccid,_wan1_sinr  # Advanced metrics (model/device specific)
```

### 14. SNMP (lines 374-383, 461-470)
```
snmpd_enable=0              # SNMP disabled by default
snmpd_port=161              # Standard SNMP port
snmpd_syscontact=           # System contact
snmpd_syslocation=          # System location
snmpd_version=              # SNMP version
snmpd_comlist=              # Community strings
snmpd_userlist=             # SNMPv3 users
snmptrap_server_ip=         # Trap destination
snmptrap_signal_level=10    # Trap on signal < 10
```

### 15. Link Backup (lines 225-226, 282-283, 541-548)
```
linkbackup_enable=0         # Link backup disabled (device override possible)
linkbackup_hot_mode=1       # Hot standby mode

wan_linkbackup_enable=0     # WAN link backup disabled
wan_main_link=wan0          # Primary: Physical WAN
wan_backup_link=wan1        # Backup: Cellular
wan_backup_mode=0           # Auto failover
wan_backup_retry=3          # Retry attempts
wan_backup_time=3600        # Failover after 1 hour
wan_main_retry=3            # Main link retry
```

### 16. DMZ (Demilitarized Zone) (lines 65-78, 81-94)
```
dmz_enable=0                # DMZ disabled
dmz0_proto=none             # No protocol
dmz0_ip=192.168.3.1         # DMZ subnet
dmz0_netmask=255.255.255.0  # Subnet mask
```

### 17. DDNS (Dynamic DNS) (lines 38-43, 52-57)
```
ddnsx0=                     # DDNS provider 1
ddnsx0_cache=               # Cached IP
ddnsx1=                     # DDNS provider 2
ddnsx2=                     # DDNS provider 3
```

### 18. Miscellaneous Settings

**Certificates (VPN)** (lines 10-14, 23-27):
```
cert_ca=                    # CA certificate
cert_public=                # Public certificate
cert_private=               # Private key
cert_key=                   # Key file
cert_crl=                   # Certificate Revocation List
```

**Hardware Reset** (lines 177, 226):
```
hw_reset_disable=0          # Hardware reset enabled (possible customer/model override)
```

**Connection Tracking** (lines 34-36, 48-50):
```
ct_max=2048                 # Max tracked connections
ct_tcp_timeout=             # TCP timeout
ct_udp_timeout=             # UDP timeout
```

**Cellular Parameters** (lines 156, 237, 292, 299-300):
```
gsm_wcdma_band_config=ALL   # 2G/3G bands
lte_band_config=ALL         # 4G LTE bands
max_modem_reset=120         # Max modem reset interval
max_ppp_redial=10           # Max PPP redial attempts
```

**Hostname** (line 158, 207):
```
hostname=DC_22_ATM_05082025 # Device hostname (DYNAMIC - must be unique per device)
```

**Router Name** (lines 349, 425):
```
router_name=Router          # Display name
```

**Language** (lines 223, 281):
```
language=English            # UI language
```

**Static Routes** (lines 350-351, 426-427):
```
routes_static_saved=        # Saved routes
routes_static=              # Active static routes (device level override)
```

**Traffic Statistics** (line 352, 428):
```
rstats_exclude=             # Exclude from statistics
```

## Configuration Parameters Summary
- **Total Parameters**: ~481
- **Administrative**: 25+
- **Network Services**: 40+
- **Firewall**: 20+
- **WAN Interfaces**: 120+
- **LAN/WiFi**: 40+
- **VPN**: 80+
- **Serial/DTU**: 40+
- **Smart ATM SSL**: 10+
- **QoS**: 20+
- **Logging**: 10+
- **Other**: 100+

## Critical Configurable Parameters

**Must be customizable per deployment**:
1. **NTP Server** (`ntp_server`) - Currently 10.4.6.30
2. **ICMP Monitoring Hosts** (`wan1_icmp_*`) - Currently 10.4.6.30
3. **Hostname** (`hostname`) - Must be unique per device
4. **DHCP Range** (`dhcpd_start`, `dhcpd_end`) - Customer-specific
5. **SSL Server Table** (`ssl_server`) - Must be admin-configurable
6. **APNs** (`wan1_ppp_apn`, `wan1_ppp_sim2_apn`) - Carrier-specific

## Classification
- **Layer**: Global (base defaults for all devices)
- **Type**: Static (system-wide defaults)
- **Override Capability**: Can be overridden by all higher layers (Model, Carrier, Service Plan, Customer, Device)

## Implementation Notes

1. **Inheritance Foundation**: All devices start with these defaults, then apply higher-layer overrides
2. **Security Defaults**: Most remote access disabled by default (SSH, Telnet, HTTPS)
3. **VPN Ready**: All major VPN types supported but disabled by default
4. **Cellular First**: Default WAN is cellular (wan1), others disabled
5. **Monitoring Built-in**: RMON reporting enabled by default
6. **Feature Rich**: Supports advanced features (QoS, VRRP, DDNS) but disabled by default

---

**Source File**: `/Users/aksana/Documents/Projects/WATM/Configurations/01_Client_Provided_Documents/Configuration_Data/GlobalConfigs - Global.csv`
**Date Generated**: 2026-02-04
