# Client Provided Documents

This directory contains original configuration files and data provided by clients and extracted from the existing WATM system.

## Contents

### Configuration_Files/
Raw device configuration files (.dat format) from various carriers, models, and service plans:
- **ATT configurations** - AT&T carrier-specific device configs for various models (I-22, CR202, 4100, 4500)
- **VZW configurations** - Verizon Wireless carrier-specific device configs
- **DC configurations** - Direct Connect configurations
- **Subdirectories:**
  - `carrier_VZW_diffmodels/` - Verizon configs across different device models
  - `model_carrier_serviceplan/` - Configurations organized by model, carrier, and service plan combinations
  - `model_I22_diffCarriers/` - I-22 model configurations across different carriers

### Configuration_Data/
CSV exports of configuration data extracted from the system, organized by configuration layer:
- `CarrierConfigs - Carrier.csv` - Carrier-level configuration parameters
- `CustomerConfigs - Customer.csv` - Customer/company-specific configurations
- `DeviceConfigs - Device.csv` - Individual device-level configurations
- `GlobalConfigs - Global.csv` - System-wide global default configurations
- `ModelConfigs - Model.csv` - Device model-specific configurations
- `ServicePlanConfigs - Service Plan.csv` - Service plan-level configurations
- `FullConfigs.csv` - Combined view of all configuration data

### Verizon_I22_Standard/
Standard and custom Verizon I-22 configuration files:
- `ATT_22_11252025.dat` - AT&T I-22 standard configuration
- `VZW_22_01272025.dat` - Verizon I-22 standard configuration
- `ATT_Custom/` - Custom AT&T I-22 configurations for specific clients
- `VZW_Custom/` - Custom Verizon I-22 configurations for specific clients

## File Naming Convention

Configuration files follow the naming pattern:
```
{CARRIER}_{MODEL}_{VARIANT}_{DATE}.dat
```

Examples:
- `VZW_22_ATM_05072025.dat` - Verizon, I-22 model, ATM variant, dated May 7, 2025
- `ATT_4100_05242023.DAT` - AT&T, 4100 model, standard, dated May 24, 2023

## Purpose

These files serve as:
1. **Reference data** for understanding current configuration patterns
2. **Test data** for validation and comparison
3. **Historical record** of configuration evolution
4. **Input data** for analysis and requirements gathering

## Related Documentation

- Analysis of these files can be found in `02_Research_and_Analysis/`
- Requirements derived from these files are in `03_Requirements/`
