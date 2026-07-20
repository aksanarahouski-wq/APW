# ModelConfigs - Model.csv Summary

## Purpose
Device model-specific hardware and feature configurations. This layer defines capabilities and settings that vary based on the physical device model (I-22, I-52, IR611, IR615, I4100, I4500, etc.), including I/O support, alarm systems, and manufacturer-specific features.

## Key Configuration Areas

### 1. Alarm System Configuration (lines 5-14)

**I-22/I-52 Models** (lines 5-8):
```
alarm_input_options=fault-service,memory-low,port0-wan-link-up/down,
    port1-4-link-up/down,dialup-up/down,traffic-alarm,traffic-discon,
    switch-sim-card,switch-backup-link,fault-sim-card,fault-signal-quality,

alarm_input=0,0,1,1,0,0,0,0,0,0,0,

alarm_output_options=cli,out-dm,out-rmon,

alarm_output=0,0,1,
```

**Alarm Input Events**:
1. fault-service - Service failures (disabled: 0)
2. memory-low - Low memory conditions (disabled: 0)
3. port0-wan-link-up/down - WAN port status changes (enabled: 1)
4. port1-4-link-up/down - LAN port status changes (enabled: 1)
5. dialup-up/down - Cellular connection status (disabled: 0)
6. traffic-alarm - Traffic limit warnings (disabled: 0)
7. traffic-discon - Traffic-based disconnections (disabled: 0)
8. switch-sim-card - SIM card switching events (disabled: 0)
9. switch-backup-link - Link failover events (disabled: 0)
10. fault-sim-card - SIM card failures (disabled: 0)
11. fault-signal-quality - Poor signal quality (disabled: 0)

**Alarm Output Methods**:
1. cli - Command line interface output (disabled: 0)
2. out-dm - Device manager output (disabled: 0)
3. out-rmon - RMON (Remote Monitoring) output (enabled: 1)

**Configuration**: Only port link status changes trigger alarms, reported via RMON

**Generic Model Config** (lines 11-14):
- Same alarm options available for all models
- Consistent alarm framework across device types

### 2. Digital I/O Configuration (lines 16-19)
**I-22 and I-52 Models ONLY** (IR611/IR615 do NOT support)

```
digitalio_config=1,0,0;1,0,0;
io_chip=0
io_triggered_report=0
iodigital_input_data=0c
```

**Features**:
- `digitalio_config` - Digital I/O pin configuration (input/output modes)
- `io_chip=0` - I/O chip selection
- `io_triggered_report=0` - Automatic reporting on I/O state changes (disabled)
- `iodigital_input_data=0c` - Current input data state (hex value)

**Note**: "I/O ONLY Device - I-22 and I-52 || Absent on IR611/IR615 Configs"

### 3. LAN Port Configuration (lines 21-28)
**May vary between IR611/615 and I22/52 models**

```
lan_port1=1                          # Port 1 enabled
lan_port2=1                          # Port 2 enabled
lan0_default_route=1                 # LAN can be default route
lan0_gateway=                        # No gateway (LAN interface)
lan0_iface=eth2.1                    # VLAN interface
lan0_ip=192.168.1.90                 # IP address
lan0_last_ip=192.168.2.1             # Previous IP (history)
lan0_last_netmask=255.255.255.0      # Previous netmask
```

**Customization Note**: "Modifications Available for Customer / Device"
- Port enable/disable can be customized
- IP addressing can be changed per deployment
- Different models may have different default interfaces

### 4. Link Backup Feature (lines 30-31)
**I-22/I-52 Models ONLY** (IR611/IR615 do NOT support)

```
linkbackup_enable=0      # Link backup disabled by default
linkbackup_hot_mode=1    # Hot standby mode (immediate failover)
```

**Note**: "Only available for I22/I52"
- Provides automatic WAN failover
- Hot mode = zero-downtime failover
- Dynamic - can be enabled per deployment

### 5. MQTT Device Manager (lines 33-45)
**Enabled for Verizon/T-Mobile I-22 and I-52 models**
**NOT enabled for AT&T or IR611/IR615**

```
mqtt_atm_id=                                    # ATM identifier
mqtt_center=iot.inhandnetworks.com              # MQTT broker
mqtt_device_info=                               # Device information payload
mqtt_enable=1                                   # MQTT enabled
mqtt_experience_confirmed=1                     # User confirmed setup
mqtt_experience_mode=disable                    # Experience mode disabled
mqtt_keepalive=60                               # 60-second keepalive
mqtt_lbs_interval=24                            # Location reporting: 24 hours
mqtt_series_interval=24                         # Series data: 24 hours
mqtt_sniffer_enable=1                           # Packet sniffer enabled
mqtt_sniffer_filter_enable=0                    # Sniffer filter disabled
mqtt_tls=1                                      # TLS encryption enabled
mqtt_username=service@wirelessatmstore.com      # MQTT username
```

**Purpose**: Cloud-based device management and monitoring
**Broker**: InHand Networks IoT platform
**Security**: TLS-encrypted connections
**Reporting**: 24-hour intervals for location and telemetry
**Carrier-Specific**: Enabled for Verizon/T-Mobile, disabled for AT&T

### 6. Status Reporting Configuration (line 47)
**Model-dependent advanced reporting**

```
rmon_advance_cfg=_wan1_imei,_wan1_iccid,_wan1_sinr
```

**Reported Metrics**:
- `_wan1_imei` - International Mobile Equipment Identity
- `_wan1_iccid` - Integrated Circuit Card Identifier (SIM)
- `_wan1_sinr` - Signal-to-Interference-plus-Noise Ratio

**Note**: "I-22 may need IO Digital and Other models may not"
- I-22 devices may report additional I/O state
- Other models report cellular metrics only

### 7. SMS Commands (lines 49-57)
**I-22/I-52 Models ONLY** (IR611/IR615 do NOT support)

```
sms_acl=                 # SMS access control list (Static)
sms_apn=APN              # Command to query/set APN (Static)
sms_ctrl_cmd=            # Control command set (Static)
sms_enable=1             # SMS control enabled (Static)
sms_network_provider=NET # Query network provider command (Static)
sms_rb=REB               # Reboot command (Static)
sms_sq=STA               # Status query command (Static)
sms_strict=0             # Strict validation disabled (Static)
sms_switch_main_sim=SIM  # Switch SIM command (Static)
```

**Features**:
- Remote control via SMS messages
- Command keywords: APN, REB, STA, NET, SIM
- Static configuration (same across all devices)
- Security: ACL controls which numbers can send commands

**Note**: "I-22/I52 Only" - Not available on IR611/IR615

## Model Comparison Matrix

| Feature | I-22/I-52 | IR611/IR615 | Notes |
|---------|-----------|-------------|-------|
| Alarm System | ✓ Full | ✓ Full | All models |
| Digital I/O | ✓ Yes | ✗ No | I-22/52 only |
| SMS Commands | ✓ Yes | ✗ No | I-22/52 only |
| Link Backup | ✓ Yes | ✗ No | I-22/52 only |
| MQTT Manager | ✓ Conditional | ✗ No | VZW/TMO only, not AT&T |
| Status Reporting | ✓ Enhanced | ✓ Basic | I-22 adds I/O data |
| LAN Ports | ✓ 2 ports | ✓ Varies | Configuration differs |

## MQTT Device Manager - Carrier Matrix

| Model | AT&T | Verizon | T-Mobile |
|-------|------|---------|----------|
| I-22 | ✗ No | ✓ Yes | ✓ Yes |
| I-52 | ✗ No | ✓ Yes | ✓ Yes |
| IR611 | ✗ No | ✗ No | ✗ No |
| IR615 | ✗ No | ✗ No | ✗ No |

**Why AT&T excluded**: Likely carrier-specific restrictions or network architecture differences

## Key Insights

1. **Model Stratification**: Clear feature differentiation between I-series (I-22, I-52) and IR-series (IR611, IR615)
2. **I/O Capability**: Only I-22/I-52 support digital I/O for sensor integration
3. **SMS Management**: Remote SMS control only on I-22/I-52 models
4. **Carrier Dependencies**: MQTT device manager availability depends on both model AND carrier
5. **Alarm Consistency**: All models support same alarm framework, but default configurations may differ
6. **Link Redundancy**: Advanced link backup only on higher-end I-22/I-52 models

## Configuration Parameters Summary
- **Total Parameters**: ~30
- **Alarm System**: 4 parameters
- **Digital I/O**: 4 parameters (I-22/52 only)
- **LAN Configuration**: 8 parameters
- **Link Backup**: 2 parameters (I-22/52 only)
- **MQTT Manager**: 13 parameters (conditional)
- **Status Reporting**: 1 parameter
- **SMS Commands**: 9 parameters (I-22/52 only)

## Implementation Considerations

### Model Detection
System must:
1. Detect device model during provisioning
2. Enable/disable features based on model capabilities
3. Validate configurations against model support
4. Prevent assignment of unsupported features

### Feature Flags
Implement feature flags based on model:
```
Model I-22/I-52:
  - has_digital_io = true
  - has_sms_control = true
  - has_link_backup = true
  - mqtt_eligible = true (if VZW/TMO)

Model IR611/IR615:
  - has_digital_io = false
  - has_sms_control = false
  - has_link_backup = false
  - mqtt_eligible = false
```

### Carrier-Specific Logic
For MQTT Device Manager:
```
if model in ['I-22', 'I-52'] AND carrier in ['Verizon', 'T-Mobile']:
    mqtt_enable = 1
    mqtt_center = 'iot.inhandnetworks.com'
else:
    mqtt_enable = 0
```

### Configuration Validation
Before applying configuration:
1. Check if model supports requested features
2. Validate I/O configurations for I-22/I-52 only
3. Verify MQTT settings against model+carrier matrix
4. Reject SMS command configs for IR models

## Use Case Scenarios

### I-22 ATM with Sensors
- Digital I/O for door sensors, temperature monitoring
- SMS commands for remote reboot/diagnostics
- Link backup for high availability
- MQTT for cloud monitoring (VZW/TMO)
- Enhanced status reporting with I/O state

### IR611 Basic Connectivity
- Basic alarm system (link status only)
- No I/O integration needed
- No SMS control (use web/API instead)
- No cloud management platform
- Standard status reporting

### I-52 Multi-WAN Deployment
- Link backup between cellular and wired WAN
- SMS fallback for out-of-band management
- I/O for external equipment status
- MQTT for centralized fleet management
- Hot-mode failover for zero downtime

## Classification
- **Layer**: Model-specific (hardware capabilities)
- **Type**: Static (determined by device model)
- **Override Capability**: Cannot be overridden (hardware limitations)
- **Influences**: Available features for higher layers

## Hardware Limitations

**Cannot Override**:
- Digital I/O presence (hardware dependent)
- SMS modem capability (hardware dependent)
- Alarm output options (firmware/hardware dependent)
- Physical port count (hardware fixed)

**Can Customize**:
- Alarm enable/disable states
- MQTT connection parameters
- Status reporting content
- SMS command keywords
- Link backup policies

---

**Source File**: `/Users/aksana/Documents/Projects/WATM/Configurations/01_Client_Provided_Documents/Configuration_Data/ModelConfigs - Model.csv`
**Date Generated**: 2026-02-04
