# Device Configuration Application System

## Overview

The WATM system applies configurations to devices through a multi-layered architecture. Here's how it works:

## Configuration Hierarchy

Configurations follow a priority order:

1. **Custom Company Configuration** (highest priority) - Company-specific config file
2. **Parent Company Configuration** (if `apply_to_children` is enabled) - Inherited from parent company
3. **Base Configuration** (default) - Standard configuration from ConfigGroup

Source: `DevicesTable.php:1671-1768`

## How Configurations Are Applied

### Step 1: Configuration Detection on Device Check-in

When a device checks in via UDP:
- Device sends check-in data to Node.js UDP server
- Data flows: UDP Server → AWS SQS → CakePHP Queue Worker
- Worker compares device's current `hostname` with expected `host_name` from configuration
- If they **don't match**, a configuration update job is queued

Source: `DeviceCheckinsTable.php:169-185`

### Step 2: Configuration Update Job Execution

The `DeviceConfigurationUpdateJob` (`Devices/src/Job/DeviceConfigurationUpdateJob.php`) handles the actual update:

#### 1. Fetch Configuration (lines 43-76)
   - Retrieves device with approved configuration
   - Gets main configuration using hierarchy (custom → parent → base)
   - Checks if hostname matches (unless `ignore_hostname` is true)

#### 2. Download & Customize Config File (lines 91-118)
   - Downloads configuration file from S3 storage
   - Applies **custom configurations** from `device.custom_configurations` JSON field
   - Custom configs override base config values using regex replacement
   - Example: `fw_acl=value` in custom configs replaces same key in base config

#### 3. Upload to Device (lines 120-134)
   - Creates `RouterApi` instance with device credentials (IP, username, password)
   - Uploads modified config file to device via HTTP API (port 4444)
   - Uses `RouterApi::uploadConfig()` which POSTs to `/upload.cgi?type=config`

#### 4. Reboot & Log (lines 136-151)
   - If upload succeeds, device is rebooted to apply changes
   - Creates `DeviceConfigurationLog` entry with success/failure status
   - Logs stored in `device_configuration_logs` table

## Configuration Components

### ConfigGroups
- Each `DeviceModel` has one `ConfigGroup` (1:1 relationship)
- Contains multiple `Configuration` versions
- Provides manufacturer/model-specific base settings

Source: `ConfigGroupsTable.php:44-132`

### Configurations
- Base configuration files stored in S3 (via Orases Files package)
- Have `status` field (e.g., "approved")
- Contain `host_name` that should match device's actual hostname
- File path accessed via `configuration.o_file.path`

### CustomCompanyConfigurations
- Override base configurations per company
- Can be set to `apply_to_children` for multi-tenant inheritance
- Take precedence over base configurations

### Device.custom_configurations
- JSON field on Device entity storing key-value pairs
- Applied as overrides during config file processing
- Methods: `appendToCustomConfiguration()`, `removeFromCustomConfiguration()`
- Example use: Firewall rules (`fw_acl`), IP settings, etc.

Source: `Device.php:377-408`

## Manual Configuration Update

Admins can manually trigger config updates via:
```bash
bin/cake worker -p update-configs -c update-configs -r 295
```

Queue workers listen for `update-configs` topic and process `DeviceConfigurationUpdateJob` instances.

## Router API Communication

The `RouterApi` class (`Devices/src/Util/RouterApi.php`) handles device communication:

- **Port:** 4444 (HTTP)
- **Authentication:** HTTP Basic Auth with device credentials
- **Key Endpoints:**
  - `upload.cgi` - Upload configuration file
  - `download.cgi` - Download current config
  - `apply.cgi` - Apply settings
  - `getinfo.cgi` - Query device parameters

### Upload Process (lines 318-324)
```php
public function uploadConfig($filePath = TMP . 'config.dat')
{
    $request = $this->client->post($this->endPoints['upload'] . '?type=config', [
        'filename' => fopen($filePath, 'r'),
    ]);
    return $request->getStringBody(); // Returns "SUCCESS" or error
}
```

## Key Features

- **Automatic Updates:** Triggered on device check-in if hostname mismatch detected
- **Custom Overrides:** Per-device settings stored in JSON field
- **Multi-tenant Support:** Parent companies can push configs to child companies
- **Audit Trail:** All config pushes logged in `device_configuration_logs`
- **Graceful Handling:** Device rebooted regardless of success/failure (lines 139-145)

## Flow Summary

```
Device Check-in (UDP)
  → Hostname Comparison
  → Queue Job (if mismatch)
  → Fetch Config (hierarchy: custom → parent → base)
  → Download from S3
  → Apply custom_configurations overrides
  → Upload to device (RouterApi)
  → Reboot device
  → Log result
```

## Configuration Priority Logic

The `getDeviceConfiguration()` method in `DevicesTable.php` implements the hierarchy:

```php
// 1. Check for company-specific custom configuration
if (isset($deviceEntity->custom_company_configuration)) {
    return $deviceEntity->custom_company_configuration;
}

// 2. Check for parent company configuration (if apply_to_children is enabled)
if (!empty($deviceEntity->company->parent_company_id)) {
    $parentCompanyConfiguration = // ... fetch parent config with apply_to_children=1
    if (isset($parentCompanyConfiguration)) {
        return $parentCompanyConfiguration;
    }
}

// 3. Fall back to base configuration
if (!empty($deviceEntity->configuration)) {
    return $deviceEntity->configuration;
}

return false;
```

## Custom Configuration Override Process

When applying device-specific custom configurations:

1. Base configuration file is downloaded from S3
2. `custom_configurations` JSON is decoded into key-value pairs
3. For each custom config key-value:
   - Searches for matching line in base config using regex: `/^($configKey=.*)$\s+?/m`
   - If found: replaces the line with new value
   - If not found: appends new line to config file
4. Modified config string is written to temporary file
5. Temporary file is uploaded to device

Source: `DeviceConfigurationUpdateJob.php:100-110`

## Error Handling

- Configuration fetch failures return `Processor::ACK` (acknowledged but skipped)
- S3 download failures create failure log and return `Processor::REJECT` (will retry)
- Upload failures create failure log, reboot device anyway, return `Processor::REJECT`
- All failures logged to `device_configuration_logs` table with status and message

## Related Database Tables

- `config_groups` - Configuration groups per device model
- `configurations` - Configuration versions with files and approval status
- `custom_company_configurations` - Company-specific config overrides
- `devices` - Stores `custom_configurations` JSON field
- `device_configuration_logs` - Audit trail of all config push attempts
- `o_files` - File storage metadata (Orases package)

This system ensures devices always run the correct configuration while supporting company-specific customizations and hierarchical inheritance.
