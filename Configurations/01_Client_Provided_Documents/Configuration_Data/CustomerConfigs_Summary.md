# CustomerConfigs - Customer.csv Summary

## Purpose
Customer-specific security and network configurations. This layer allows individual customers to have customized firewall rules, NAT configurations, and URL filtering tailored to their specific business requirements.

## Key Configuration Areas

### 1. Firewall ACL Rules (line 3)
Extensive whitelist configuration for ATM Service Plan customers with specific allowed destinations.

**Format**: `1<4<source_ip<<dest_ip<<protocol<enable<description`

**Allowed Services and IPs**:

**Cloud & AWS Resources**:
- `34.199.0.0/16` - Amazon Resource
- `3.5.0.0/16` - AWS Genmega
- `52.192.0.0/12` - AWS Genmega2

**Payment Processor - CDS (Columbus Data Services)**:
- `208.35.209.1` - CDS
- `216.58.156.98` - CDS
- `198.28.31.98` - CDS
- `216.58.156.80` - CDS
- `63.175.24.215` - CDS

**Payment Processor - Switch Commerce**:
- `204.8.249.126` - Switch Commerce
- `67.23.48.67` - Switch Commerce
- `207.108.146.108` - Switch Commerce
- `208.224.248.160` - Switch Commerce
- `67.23.53.167` - Switch Commerce
- `64.211.210.167` - Switch Commerce
- `216.58.156.0/24` - SC subnet
- `76.77.157.0/24` - SC subnet
- `216.117.40.0/24` - SC subnet
- `64.88.167.0/24` - SC subnet

**PAI (Payment Alliance International)**:
- `206.71.17.21/32` - PAI

**Digital Network**:
- `20.88.238.228/32` - Digital Network
- `20.221.234.232` - Digital Network

**RMS Systems**:
- `13.67.184.126` - CORD RMS
- `208.92.212.170` - DEPLOYER RMS
- `64.183.178.180` - Cord RMS Old

**1st ISO**:
- `69.21.165.134` - 1st ISO
- `209.103.211.74` - 1st ISO

**Other Services**:
- `64.88.167.58` - EFX (disabled by default - protocol=0)
- `34.199.247.183` - LibertyX
- `198.224.183.135` - DNS
- `10.4.6.30` - ICMP & NTP (internal)

**DNS Servers**:
- `8.8.8.8` - Google DNS
- `8.8.4.4` - Google DNS
- `1.1.1.1` - Cloudflare DNS

**Internal/Development**:
- `10.4.0.31` - Internal server
- `10.4.0.32` - Internal server

**Final Rule**:
- `0.0.0.0/0 << 0.0.0.0/0 << 2 < 1 < Block` - **Block all other traffic**

**Important Notes**:
- Certain customers contain additional ACL entries for specific customer servers
- ATM Service Plan customers have restrictive whitelisting
- Category: FW Rules, Dynamic

### 2. NAT Rules (line 5)
Network Address Translation for specific port forwarding scenarios.

**Format**: `enable<protocol<type<source<<dest<port<<forward_ip<port<nat<description`

**Configured Rules**:
- `1<1<1<0.0.0.0/0<<0.0.0.0/0<18458<<10.4.0.32<18458<1<>` - Forward port 18458 to 10.4.0.32
- `1<1<1<0.0.0.0/0<<0.0.0.0/0<9999<<10.4.0.31<9999<1<>` - Forward port 9999 to 10.4.0.31

**Purpose**:
- Certain customers have designated NAT rules for traffic manipulation optimization
- Category: FW RULES, Dynamic

### 3. URL Filtering Notes (lines 8-9)
Important architectural notes about URL management:

**Device-Specific URL Handling**:
- **I-22 Devices**: URLs go into `fw_acl` (firewall ACL)
- **I4100/I4500 Devices**: URLs go into Content Filtering

**Management Requirement**:
> "both need to be managed in the same 'place' for customer configuration segmentation"

This indicates a need for unified URL management across different device models despite different implementation mechanisms.

## Key Insights

1. **Whitelist Security Model**: Customers use a strict whitelist approach - only explicitly allowed IPs can be reached
2. **Payment Industry Focus**: Heavy emphasis on payment processors, ATM networks, and financial services
3. **Default Deny**: The final rule blocks all traffic not explicitly whitelisted (security best practice)
4. **Custom Forwarding**: NAT rules enable specific port forwarding for customer-specific backend systems
5. **Model-Specific Implementation**: URL filtering works differently on I-22 vs I4100/I4500 but needs unified management

## Security Architecture

**Defense in Depth**:
1. Only specific payment processor IPs allowed
2. DNS limited to trusted servers (Google, Cloudflare, internal)
3. Cloud access restricted to known AWS ranges
4. All other traffic blocked by default
5. Internal monitoring via dedicated ICMP/NTP host

**ATM-Specific Security**:
- Prevents ATM devices from accessing arbitrary internet resources
- Limits attack surface for compromised devices
- Ensures all transactions route through approved processors
- Blocks potential data exfiltration paths

## Configuration Parameters Summary
- **ACL Rules**: 35+ allowed destinations
- **NAT Rules**: 2 port forwarding rules
- **Default Action**: Block all other traffic
- **Source Filtering**: 0.0.0.0/0 (any source) in customer config, but typically restricted to 192.168.1.0/24 in service plan

## Common Customer Customizations

### Custom Processor Addition
When customer uses specific payment processor not in standard list:
1. Add processor IP(s) to `fw_acl`
2. Include descriptive name
3. Set protocol (4=allow, 2=deny, 0=disabled)
4. Enable flag (1=enabled)

### RMS/Backend Integration
For customers with remote management systems:
1. Add NAT rule for inbound port forwarding
2. Add firewall rule to allow outbound to RMS IP
3. Document customer-specific configuration

### Development/Testing Access
Some customers have internal development environments:
- `10.4.0.31`, `10.4.0.32` entries
- Port forwarding for development access
- Should be removed for production deployments

## URL Management Requirements

Based on line 8-9 notes, the system needs:
1. **Unified Interface**: Single management UI for URL filtering regardless of device model
2. **Automatic Translation**: Convert URL rules to appropriate format:
   - I-22: Add to `fw_acl`
   - I4100/I4500: Add to Content Filtering
3. **Customer Segmentation**: Each customer's URL rules apply only to their devices
4. **Inheritance**: URLs can be defined at Service Plan level and inherited by customers

## Classification
- **Layer**: Customer-specific
- **Type**: Dynamic (varies per customer)
- **Override Capability**: Overrides Global, Model, Carrier, and Service Plan firewall settings
- **Can be overridden by**: Device-level configurations

## Implementation Notes

1. **ACL Format**: Uses `<` delimiter for structured data (protocol, action, description)
2. **Protocol Codes**: 4=IP allow, 2=IP deny, 1=TCP, 0=disabled
3. **Optimization Note**: NAT rules are described as "traffic manipulation optimization"
4. **Block Rule**: Final catch-all block rule is critical - must always be last in ACL list
5. **Customer Isolation**: Each customer should have isolated ACL namespace to prevent conflicts

## Compliance & Audit Considerations

- **PCI DSS**: Whitelist approach supports PCI requirement for segmented networks
- **Audit Trail**: Changes to customer ACLs should be logged
- **Documentation**: Each allowed IP should have business justification (description field)
- **Review Cycle**: Customer ACLs should be reviewed periodically to remove obsolete entries

---

**Source File**: `/Users/aksana/Documents/Projects/WATM/Configurations/01_Client_Provided_Documents/Configuration_Data/CustomerConfigs - Customer.csv`
**Date Generated**: 2026-02-04
