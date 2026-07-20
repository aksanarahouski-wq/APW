# CarrierConfigs - Carrier.csv Summary

## Purpose
Defines carrier-specific cellular connectivity configurations for IoT devices, including dual SIM management, APN settings, and PPP connection parameters.

## Key Configuration Areas

### 1. Dual SIM Management
**Non-Dual SIM Devices** (lines 9-14):
- `dual_sim_enable=0` - Feature disabled
- `dual_sim_csq_retry=4` - Signal quality retry attempts
- `dual_sim_max_retry=5` - Maximum connection retries
- `dual_sim_min_conn_time=0` - No minimum connection time
- `dual_sim_min_csq=4` - Minimum signal quality threshold

**Dual SIM Enabled Devices** (lines 16-21):
- `dual_sim_enable=1` - Feature enabled
- `dual_sim_main=0` - Primary SIM selector (0=SIM1, 1=SIM2) - **allows flipping between carriers**
- `dual_sim_min_conn_time=120` - Must maintain connection for 120 seconds before considering it stable
- Same retry and signal quality settings as non-dual

**Backup SIM Policy** (lines 5-7):
- `backup_sim_policy_enable=0` - Disabled by default
- `backup_sim_policy_revert_day=1` - Days before reverting to primary
- `backup_sim_policy_using_time=0` - Time tracking for backup usage

### 2. SIM Card Operator Assignment
- `sim1_card_operator=2` - SIM1 assigned to operator profile #2 (referenced from PROFILE TABLE)
- `sim2_card_operator=1` - SIM2 assigned to operator profile #1
- These values map to carrier-specific configuration profiles

### 3. PPP Redial Profiles (line 25)
Pre-configured profiles for multiple carriers:
```
Profile 1: matrix.com.attz:*99#:0:: (AT&T)
Profile 2: Matrxatm.gw12.vzwentp:*99#:0:: (Verizon)
```
- Format: `profile_num:apn:dial_number:auth:username:password`
- `redial_reboot=1` - Reboot device if connection fails
- Supports dual APN with separate profiles for each SIM

### 4. WAN1 (Cellular Interface) Configuration

**Connection Settings**:
- Protocol: `dialup` (PPP over cellular)
- Interface: `/dev/ttyUSB3`
- MTU: 1500 bytes
- Network: FDD-LTE
- Band config: ALL (all supported bands)

**APN Configuration** (line 60 - **Dynamic/Carrier-specific**):
- **Verizon**: `Matrxatm.gw12.vzwentp`
- **AT&T**: `matrix.com.attz`
- **T-Mobile**: `simpl.cc.static`
- Dial number: `*99#` (standard GSM)
- Authentication: 0 (none required)
- No username/password required

**SIM2 APN** (line 84):
- Default: `matrix.com.attz` (AT&T)
- Provider: 1 (profile lookup)
- Separate configuration for secondary SIM

**PPP Parameters**:
- Check interval: 55 seconds
- Check retries: 6 attempts
- Redial interval: 30 seconds (**Dynamic**)
- Connection timeout: 120 seconds
- Peer DNS: enabled (`wan1_ppp_peerdns=1`)
- TX queue length: 64
- PPP options: `nomppe nomppc nodeflate nobsdcomp novj novjccomp noccp` (disable compression)

**ICMP Monitoring** (lines 43-51):
- Host: `10.4.6.30` (**Must be configurable per deployment**)
- Backup host: `10.4.6.30`
- Main host: `10.4.6.30`
- Interval: 3600 seconds (1 hour)
- Retries: 5 attempts
- Timeout: 20 seconds
- Lost packets threshold: 100
- Mode: 1 (active monitoring)

**Triggers**:
- `wan1_trig_call=1` - Triggered by call
- `wan1_trig_data=1` - Triggered by data
- `wan1_trig_sms=0` - Not triggered by SMS

## Key Insights

1. **Carrier Differentiation**: Primary differentiation is through APN settings - each carrier has a specific APN string
2. **Dual SIM Strategy**: Supports automatic failover between two carriers with configurable connection quality thresholds
3. **Connection Stability**: Dual SIM devices require 120-second stable connection before considering it successful (prevents flapping)
4. **Profile-Based**: Uses profile table lookups for carrier operator assignments
5. **Dynamic Parameters**: APN and redial interval must be configurable per deployment
6. **Critical Configuration Need**: ICMP monitoring hosts (10.4.6.30) must be made configurable as noted in file

## Carrier-Specific APNs

| Carrier | APN | Profile # |
|---------|-----|-----------|
| Verizon | Matrxatm.gw12.vzwentp | 2 |
| AT&T | matrix.com.attz | 1 |
| T-Mobile | simpl.cc.static | - |

## Usage Scenario
This configuration layer allows the system to:
- Support multiple cellular carriers (AT&T, Verizon, T-Mobile)
- Provide automatic carrier failover with dual SIM devices
- Maintain stable connections with appropriate retry and timeout logic
- Monitor connection health via ICMP ping to management infrastructure
- Use carrier-specific APNs and connection parameters

## Configuration Parameters Summary
- **Total Parameters**: ~70
- **Dual SIM Parameters**: 11
- **WAN1 PPP Parameters**: 35+
- **ICMP Monitoring**: 9
- **Redial/Failover**: 7

## Classification
- **Layer**: Carrier-specific
- **Type**: Dynamic (varies by carrier selection)
- **Dependencies**: Requires profile table for operator lookups
- **Override Capability**: Can be overridden at Service Plan, Customer, or Device levels

## Implementation Notes

1. **Profile Table Required**: The system references a separate profile table for operator configurations
2. **Configurable Hosts**: ICMP monitoring hosts must be made configurable (currently hardcoded to 10.4.6.30)
3. **Dual SIM Logic**: When enabled, the system will automatically failover between carriers based on signal quality and connection stability
4. **APN Selection**: APN must be automatically selected based on detected carrier or manually configured
5. **Redial Strategy**: Device will attempt reconnection with exponential backoff, eventually rebooting if all attempts fail

---

**Source File**: `/Users/aksana/Documents/Projects/WATM/Configurations/01_Client_Provided_Documents/Configuration_Data/CarrierConfigs - Carrier.csv`
**Date Generated**: 2026-02-04
