# ServicePlanConfigs - Service Plan.csv Summary

## Purpose
Service plan tier-specific configurations that define different levels of service, security restrictions, and data usage policies. Service plans are assigned to devices and determine their network access policies and traffic management rules.

## Service Plan Types

### 1. Non-ATM Service Plan (line 4)
**Basic, Open Network Access**

**Firewall ACL**:
- `fw_acl=1<4<0.0.0.0/0<<0.0.0.0/0<<1<1<>`
- **Meaning**: Allow all traffic from any source to any destination
- **Security**: Minimal restrictions - open internet access
- Category: FW Rules, Dynamic

**Use Case**: Devices that need unrestricted internet access, non-payment processing applications

---

### 2. ATM Service Plan (line 5)
**Restrictive, Payment Industry Focused**

**Firewall ACL** - Comprehensive whitelist with LAN subnet restriction:
- **Source Restriction**: `192.168.1.0/24` - All rules scoped to local subnet
- **Strategy**: Whitelist approved payment processors and services only
- **Final Rule**: Block all other traffic

**Allowed Destinations**:

**Cloud & AWS**:
- `34.199.0.0/16` - Amazon Resource
- `3.5.0.0/16` - AWS Genmega
- `52.192.0.0/12` - AWS Genmega2

**Payment Processors - CDS**:
- `208.35.209.1` - CDS
- `216.58.156.98` - CDS
- `198.28.31.98` - CDS
- `216.58.156.80` - CDS
- `63.175.24.215` - CDS

**Payment Processors - Switch Commerce**:
- `204.8.249.126`
- `67.23.48.67`
- `207.108.146.108`
- `208.224.248.160`
- `67.23.53.167`
- `64.211.210.167`

**Payment Alliance International (PAI)**:
- `206.71.17.21/32` - PAI
- `216.26.158.20` - PAI MV RMS
- `216.26.158.19` - PAI GM RMS

**Digital Network**:
- `20.88.238.228/32`
- `20.221.234.232`

**1st ISO**:
- `69.21.165.134`
- `209.103.211.74`

**Other Services**:
- `64.88.167.58` - EFX (disabled - protocol=0)
- `34.199.247.183` - LibertyX
- `208.78.47.0/24` - Planet Payment DCC

**DNS Servers**:
- `8.8.8.8` - Google DNS
- `8.8.4.4` - Google DNS
- `1.1.1.1` - Cloudflare DNS

**Final Rule**: `0.0.0.0/0 << 0.0.0.0/0 << 2 < 1 < Block` - Block all other traffic

Category: FW Rules, Dynamic, ATM Service Plan

---

## Traffic Management Configuration (lines 8-25)

All parameters marked as "Traffic Management" and "Dynamic":

### Traffic Custom Rules (lines 8-10)
- `traffic_custom_actions=0,0,0,0` - Custom action definitions (disabled)
- `traffic_custom_enable=0` - Custom traffic rules disabled
- `traffic_custom_rule=` - No custom rules defined

### Daily Data Limits (lines 11-13)
- `traffic_day_action=1` - Action when daily limit exceeded (1=alert, 2=disconnect)
- `traffic_day_threshold=3584` - **Daily data limit: 3584 KB (3.5 MB)**
- `traffic_day_unit=1` - Unit: 1=KB, 2=MB, 3=GB

### Traffic Monitoring (lines 14-15)
- `traffic_enable=1` - **Traffic monitoring enabled**
- `traffic_exceed_report=0` - Don't report when limits exceeded

### Monthly Data Limits - Primary (lines 16-21)
- `traffic_month_action=1` - Action when monthly limit exceeded
- `traffic_month_alarm=0` - No alarm generation
- `traffic_month_discon=0` - Don't disconnect when exceeded
- `traffic_month_start_day=1` - Billing cycle starts on day 1 of month
- `traffic_month_threshold=0` - **No monthly limit set (0=unlimited)**
- `traffic_month_unit=1` - Unit: KB

### Monthly Data Limits - Secondary (lines 22-24)
- `traffic_month2_action=0` - Secondary threshold disabled
- `traffic_month2_threshold=0` - No secondary threshold
- `traffic_month2_unit=1` - Unit: KB

### SMS Notification (line 25)
- `traffic_sms_up=` - SMS notification on traffic events (not configured)

## Key Insights

### Service Plan Differentiation

**Non-ATM Plan**:
- Open firewall (all traffic allowed)
- Suitable for general IoT devices
- No payment industry restrictions
- Lower security requirements

**ATM Service Plan**:
- Restrictive whitelist firewall
- Payment processor focus
- Source IP limited to LAN subnet (192.168.1.0/24)
- PCI compliance oriented
- Higher security requirements

### Traffic Management Features

1. **Daily Monitoring**: 3.5 MB daily threshold with alert action
2. **Monthly Flexibility**: No monthly limit by default (can be customized)
3. **Graduated Actions**: Can alert, limit, or disconnect based on thresholds
4. **Billing Cycle Awareness**: Monthly tracking starts on day 1
5. **Multi-Threshold Support**: Primary and secondary monthly thresholds available
6. **SMS Integration**: Can notify via SMS when limits exceeded

### Data Usage Control Strategy

**Current Default Configuration**:
- Daily limit: 3.5 MB (restrictive - prevents excessive usage)
- Monthly limit: Unlimited (flexibility for legitimate traffic)
- Action: Alert only (non-disruptive)
- No automatic disconnection (customer service oriented)

**Customization Capability**:
> "Different Service plans can be designated different daily or monthly data aggregates"

This allows creating tiered service plans:
- **Basic**: Low daily/monthly limits
- **Standard**: Medium limits (current default)
- **Premium**: High or unlimited limits
- **Enterprise**: Unlimited with monitoring only

## Configuration Parameters Summary
- **Firewall Rules**: 2 variants (open vs. restrictive)
- **Traffic Management**: 18 parameters
- **Daily Limit**: 3.5 MB default
- **Monthly Limit**: Unlimited default
- **Actions**: Alert, disconnect, report

## Common Service Plan Scenarios

### Scenario 1: Standard ATM Service Plan
```
fw_acl = [Restrictive whitelist - 30+ allowed IPs]
traffic_enable = 1
traffic_day_threshold = 3584 KB (3.5 MB)
traffic_month_threshold = 0 (unlimited)
traffic_day_action = 1 (alert)
```
- Secure payment processing
- Reasonable daily limit prevents abuse
- Monthly flexibility for legitimate traffic
- Alerts trigger investigation

### Scenario 2: Premium ATM Service Plan
```
fw_acl = [Restrictive whitelist]
traffic_day_threshold = 10240 KB (10 MB)
traffic_month_threshold = 102400 KB (100 MB)
traffic_*_action = 1 (alert only)
```
- Higher volume payment processing
- More generous limits
- Still monitored and alerted

### Scenario 3: Budget ATM Service Plan
```
fw_acl = [Restrictive whitelist]
traffic_day_threshold = 1024 KB (1 MB)
traffic_month_threshold = 10240 KB (10 MB)
traffic_month_discon = 1 (disconnect)
```
- Lower cost option
- Strict limits
- Automatic enforcement via disconnection

### Scenario 4: IoT General Purpose (Non-ATM)
```
fw_acl = [Allow all]
traffic_enable = 0
```
- Open access
- No traffic management
- Suitable for non-sensitive applications

## Implementation Considerations

### Firewall Management
1. **Default Deny**: ATM plan ends with explicit block rule
2. **Source Scoping**: ATM rules scoped to 192.168.1.0/24 subnet
3. **Protocol Specific**: Rules specify protocol (IP, TCP, etc.)
4. **Extensible**: New IPs can be added to whitelist as needed

### Traffic Enforcement
1. **Real-time Monitoring**: Must track daily and monthly usage
2. **Action Engine**: Support alert, throttle, disconnect actions
3. **Reporting**: Generate reports on traffic events
4. **Reset Schedule**: Daily counter resets at midnight, monthly on start day
5. **Customer Notification**: SMS integration for limit warnings

### Service Plan Assignment
1. **Device Association**: Each device assigned to one service plan
2. **Bulk Operations**: Change multiple devices to new plan
3. **Grace Period**: Consider grace period before disconnection
4. **Override**: Customer or device-level overrides possible

## Classification
- **Layer**: Service Plan (applies to groups of devices)
- **Type**: Dynamic (multiple plan variants)
- **Override Capability**: Overrides Global, Model, Carrier layers
- **Can be overridden by**: Customer and Device layers

## Compliance & Business Logic

### PCI DSS Alignment (ATM Plan)
- Segmented network (source IP restriction)
- Whitelist approach (principle of least privilege)
- Monitored access (traffic logging)
- Blocked unauthorized destinations

### Revenue Management
- Traffic limits support tiered pricing
- Overage detection enables billing
- SMS notifications improve customer experience
- Automatic enforcement reduces support costs

### SLA Support
- Different plans for different service levels
- Measurable limits enable SLA compliance
- Monitoring enables performance tracking

---

**Source File**: `/Users/aksana/Documents/Projects/WATM/Configurations/01_Client_Provided_Documents/Configuration_Data/ServicePlanConfigs - Service Plan.csv`
**Date Generated**: 2026-02-04
