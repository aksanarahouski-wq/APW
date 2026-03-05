# Device Configuration Grouping and Comparison

**Issue:** How to identify devices with identical configurations in dynamic resolution system
**Date:** March 1, 2026
**Status:** Solution Proposed - Awaiting Review
**Priority:** HIGH - Critical for admin usability and migration

---

## Problem Statement

### Context

In the old file-based configuration system, administrators could easily tell whether devices shared the same configuration by looking at the filename:

**Old System:**
- Device A → `VZW_22_ATM_10142025.dat`
- Device B → `VZW_22_ATM_10142025.dat`
- **Visual proof they're identical:** Same filename

**New System:**
- Device A → Resolved dynamically from (Model=i-22, Carrier=VZW, Plan=ATM, Company=ACME, Device#123)
- Device B → Resolved dynamically from (Model=i-22, Carrier=VZW, Plan=ATM, Company=ACME, Device#456)
- **Question:** Are their effective configs identical?

### Why This Matters

**Troubleshooting:**
- "Do all my ATM devices have the same firewall rules?"
- "Which devices were affected by my global DNS change?"

**Validation:**
- "Did my global DNS change actually apply to all devices?"
- "Are there any devices with unexpected config differences?"

**Migration:**
- "Which devices have identical configs so I can migrate them together?"
- "Show me all devices using the 'standard ATM configuration'"

**Customer Support:**
- "Which customer devices have custom configurations?"
- "Are all CORD devices using the same firewall exceptions?"

**Bulk Operations:**
- "Update all devices that share this configuration profile"
- "Test config changes on a subset of identical devices first"

---

## Solution Approaches Analyzed

### Approach 1: Configuration Profile Hash ⭐ RECOMMENDED

**Concept:** Generate a hash/fingerprint of the device attributes that determine config resolution.

#### Implementation

**Step 1: Define "Configuration Profile"**

A device's configuration profile is determined by the attributes that affect config resolution:
```
Configuration Profile = (Model, Carrier, Service Plan, Company, Has Device Overrides?)
```

**Step 2: Generate Profile Hash**

```php
function getConfigurationProfileHash($device) {
    $profile = [
        'model_id' => $device->model_id,
        'carrier_id' => $device->carrier_id,
        'service_plan_id' => $device->service_plan_id,
        'company_id' => $device->company_id,
        'has_device_overrides' => $device->hasDeviceLevelOverrides(), // boolean
        'has_conditional_rules' => $device->matchesAnyConditionalRules() // boolean
    ];

    return hash('sha256', json_encode($profile));
}
```

**Step 3: Database Schema**

```sql
ALTER TABLE devices ADD COLUMN config_profile_hash VARCHAR(64);
CREATE INDEX idx_devices_config_profile_hash ON devices(config_profile_hash);
```

Update hash whenever device attributes change or when config layers change.

**Step 4: Grouping Queries**

```sql
-- Find all devices with identical config profiles
SELECT config_profile_hash, COUNT(*) as device_count
FROM devices
GROUP BY config_profile_hash
ORDER BY device_count DESC;

-- Find all devices sharing same profile as Device #123
SELECT d2.*
FROM devices d1
JOIN devices d2 ON d1.config_profile_hash = d2.config_profile_hash
WHERE d1.id = 123;

-- Count devices per profile
SELECT
    config_profile_hash,
    COUNT(*) as device_count,
    MIN(id) as example_device_id
FROM devices
GROUP BY config_profile_hash
HAVING COUNT(*) > 1
ORDER BY device_count DESC;
```

#### Pros
- ✅ Fast queries (indexed hash column)
- ✅ Works for all devices
- ✅ Automatically groups devices with identical resolution logic
- ✅ Handles device overrides (different hash if device has custom values)
- ✅ Can show "5,234 devices use this profile"
- ✅ Low storage overhead (single VARCHAR(64) column)
- ✅ Fast to recalculate (only when device attributes change)

#### Cons
- ❌ Hash doesn't tell you WHAT the config is, just that devices are grouped
- ❌ Needs to be recalculated when layer configs change
- ❌ Doesn't handle "similar but not identical" configs well

---

### Approach 2: Configuration Template/Archetype Pattern

**Concept:** Create explicit configuration templates that represent common device patterns.

#### Implementation

**Step 1: Define Configuration Templates**

```sql
CREATE TABLE config_templates (
    id INT PRIMARY KEY,
    name VARCHAR(255),              -- "Standard VZW i-22 ATM"
    description TEXT,                -- "Standard configuration for i-22 devices on VZW carrier with ATM service plan"
    model_id INT,
    carrier_id INT,
    service_plan_id INT,
    company_id INT NULL,            -- NULL = applies to all companies
    created_at TIMESTAMP
);
```

**Step 2: Auto-Detect or Manually Define Templates**

System can:
1. **Auto-detect common patterns:**
   ```sql
   -- Find common device attribute combinations
   SELECT model_id, carrier_id, service_plan_id, company_id, COUNT(*) as count
   FROM devices
   WHERE company_id IS NOT NULL  -- Customer-specific template
   GROUP BY model_id, carrier_id, service_plan_id, company_id
   HAVING COUNT(*) > 10  -- Only create template if 10+ devices share it
   ORDER BY count DESC;
   ```

2. **Admins manually create templates:**
   - Name: "Standard VZW i-22 ATM"
   - Applies to: Model=i-22, Carrier=VZW, Plan=ATM, Company=Any

**Step 3: Link Devices to Templates**

```sql
ALTER TABLE devices ADD COLUMN config_template_id INT;

-- Auto-assign devices to templates based on matching attributes
UPDATE devices d
SET config_template_id = t.id
FROM config_templates t
WHERE d.model_id = t.model_id
  AND d.carrier_id = t.carrier_id
  AND d.service_plan_id = t.service_plan_id
  AND (t.company_id IS NULL OR d.company_id = t.company_id)
  AND NOT EXISTS (
      SELECT 1 FROM device_config_overrides
      WHERE device_id = d.id
  );  -- Only assign if no custom device values
```

**Step 4: Admin UI Shows Templates**

```
Configuration Templates:
┌────────────────────────────────────────────────────────────┐
│ Template: Standard VZW i-22 ATM                           │
│ Devices: 4,523                                            │
│ Model: i-22 | Carrier: VZW | Plan: ATM | Company: Any    │
│ [View Effective Config] [Edit Global/Model/Carrier/Plan] │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Template: CORD Custom (ATM)                               │
│ Devices: 156                                              │
│ Model: i-22 | Carrier: VZW | Plan: ATM | Company: CORD   │
│ Uses Four-Way Rule for custom firewall exceptions        │
│ [View Effective Config] [Edit Company Override]          │
└────────────────────────────────────────────────────────────┘
```

#### Pros
- ✅ Human-readable names ("Standard VZW i-22 ATM")
- ✅ Can be used for bulk operations ("Update all devices in this template")
- ✅ Mirrors old system's mental model (filename → template name)
- ✅ Can track template usage over time
- ✅ Templates can have descriptions/documentation
- ✅ Admins can organize/categorize configurations

#### Cons
- ❌ Requires template creation/maintenance
- ❌ Devices with overrides don't fit templates cleanly
- ❌ Templates can become stale if device attributes change
- ❌ Extra database table to maintain

---

### Approach 3: Virtual Configuration File Name

**Concept:** Generate a virtual filename that mimics the old system's naming convention.

#### Implementation

```php
function getVirtualConfigFileName($device) {
    $parts = [];

    // Carrier code
    $parts[] = $device->carrier->code; // "VZW"

    // Model code
    $parts[] = $device->model->code; // "22"

    // Service plan code
    if ($device->service_plan) {
        $parts[] = $device->service_plan->code; // "ATM"
    }

    // Company suffix (if company overrides exist)
    if ($device->company->hasConfigOverrides()) {
        $parts[] = $device->company->code; // "CORD"
    }

    // Device override indicator
    if ($device->hasDeviceLevelOverrides()) {
        $parts[] = "DEVICE" . $device->id;
    }

    // Version hash (changes when config layers change)
    $parts[] = substr($device->config_version_hash, 0, 8);

    return implode('_', $parts) . '.dat';
}
```

**Examples:**
- `VZW_22_ATM_a3f9c2b1.dat` - Standard VZW i-22 ATM config
- `VZW_22_ATM_CORD_a3f9c2b1.dat` - CORD company with custom overrides
- `VZW_22_ATM_DEVICE12345_b7e4f8a2.dat` - Device #12345 with device-level overrides

**Display in Admin UI:**
```
Device List:
┌──────────┬─────────────────────────────────────────────┬──────────┐
│ Device # │ Virtual Config File                         │ Status   │
├──────────┼─────────────────────────────────────────────┼──────────┤
│ 12345    │ VZW_22_ATM_a3f9c2b1.dat                    │ Up to date│
│ 12346    │ VZW_22_ATM_a3f9c2b1.dat                    │ Up to date│
│ 12347    │ VZW_22_ATM_CORD_b5f8e3c2.dat               │ Up to date│
│ 12348    │ VZW_22_ATM_DEVICE12348_c9a2f4e1.dat        │ Up to date│
└──────────┴─────────────────────────────────────────────┴──────────┘

Group by config: 2,345 devices use VZW_22_ATM_a3f9c2b1.dat
```

#### Pros
- ✅ Familiar to admins (looks like old system)
- ✅ Easy to see device groupings at a glance
- ✅ Can filter/search by virtual filename
- ✅ Version hash shows when configs differ
- ✅ Smooth transition from old system

#### Cons
- ❌ Not a real file (might be confusing)
- ❌ Filename becomes complex with many overrides
- ❌ Doesn't reflect conditional rules clearly
- ❌ Need to store virtual filename or regenerate each time

---

### Approach 4: Effective Config Hash + Diff Detection

**Concept:** Hash the actual resolved configuration values, not just the resolution inputs.

#### Implementation

**Step 1: Generate Effective Config**
```php
$effectiveConfig = resolveConfigForDevice($device); // Returns array of 600+ key-value pairs
```

**Step 2: Hash the Effective Config**
```php
// Sort keys for consistent hashing
ksort($effectiveConfig);

// Generate hash of actual values
$configHash = hash('sha256', json_encode($effectiveConfig));
```

**Step 3: Database Schema**
```sql
ALTER TABLE devices ADD COLUMN effective_config_hash VARCHAR(64);
CREATE INDEX idx_devices_effective_config_hash ON devices(effective_config_hash);
```

**Step 4: Grouping by Effective Config Hash**
```sql
-- Find devices with IDENTICAL configs (100% match)
SELECT effective_config_hash, COUNT(*) as count
FROM devices
GROUP BY effective_config_hash
ORDER BY count DESC;
```

**Step 5: Diff Detection**
```php
// Compare two devices
$device1Config = resolveConfigForDevice($device1);
$device2Config = resolveConfigForDevice($device2);

$diff = array_diff_assoc($device1Config, $device2Config);

if (empty($diff)) {
    echo "Devices have IDENTICAL configurations";
} else {
    echo "Differences found:";
    foreach ($diff as $key => $value) {
        echo "$key: Device1={$device1Config[$key]}, Device2={$device2Config[$key]}";
    }
}
```

#### Pros
- ✅ 100% accurate - based on actual resolved values
- ✅ Catches subtle differences other approaches might miss
- ✅ Can show exact differences between configs
- ✅ Guarantees identical hash = identical config
- ✅ No false positives (same profile but different rules)

#### Cons
- ❌ Expensive to calculate (must resolve all 600+ parameters)
- ❌ Must recalculate whenever ANY layer changes
- ❌ Large storage requirement if storing entire config for comparison
- ❌ Doesn't explain WHY configs are different (just that they are)
- ❌ Performance impact on large-scale recalculations

---

## Recommended Solution: Hybrid Approach ⭐

**Combine Approach 1 (Profile Hash) + Approach 4 (Effective Config Hash)**

### Why Hybrid?

- **Profile Hash** = Fast, lightweight, good enough for 90% of cases
- **Effective Config Hash** = Accurate, validates that configs are truly identical

### Implementation

**Store Two Hashes on Each Device:**

```sql
ALTER TABLE devices
ADD COLUMN config_profile_hash VARCHAR(64),    -- Fast grouping
ADD COLUMN effective_config_hash VARCHAR(64);  -- Accurate validation

CREATE INDEX idx_devices_config_profile_hash ON devices(config_profile_hash);
CREATE INDEX idx_devices_effective_config_hash ON devices(effective_config_hash);
```

**Profile Hash** (Fast, updated when device attributes change):
```php
function updateConfigProfileHash($device) {
    $profileHash = hash('sha256', json_encode([
        'model_id' => $device->model_id,
        'carrier_id' => $device->carrier_id,
        'service_plan_id' => $device->service_plan_id,
        'company_id' => $device->company_id,
        'has_device_overrides' => $device->hasDeviceLevelOverrides()
    ]));

    $device->config_profile_hash = $profileHash;
    $device->save();
}
```

**Effective Config Hash** (Expensive, pre-calculated and cached):
```php
function updateEffectiveConfigHash($device) {
    $effectiveConfig = resolveConfigForDevice($device);
    ksort($effectiveConfig);
    $effectiveHash = hash('sha256', json_encode($effectiveConfig));

    $device->effective_config_hash = $effectiveHash;
    $device->save();
}
```

### When to Recalculate Hashes

#### Profile Hash (Lightweight)
Recalculate when:
- Device model changes
- Device carrier changes
- Device service plan changes
- Device company changes
- Device overrides added/removed

**Performance:** Instant (simple attribute hashing)

#### Effective Config Hash (Expensive)
Recalculate when:
- Profile hash changes
- Global layer changes (affects ALL devices)
- Model layer changes (affects devices of that model)
- Carrier layer changes (affects devices on that carrier)
- Service Plan layer changes (affects devices on that plan)
- Company layer changes (affects devices in that company)
- Device override changes (affects that specific device)
- Conditional rule changes (Two-Way, Three-Way, Four-Way)

**Performance:** Expensive - queue as background job, don't block admin actions

**Optimization Strategy:**
```php
// Queue background job for effective hash recalculation
Queue::push(new RecalculateEffectiveConfigHashJob([
    'device_ids' => $affectedDeviceIds,
    'priority' => 'medium'
]));

// Don't wait for completion - update asynchronously
```

---

## Admin UI Features

### 1. Device List with Grouping

```
Devices (Grouped by Configuration Profile)

┌────────────────────────────────────────────────────────────────┐
│ Profile: VZW i-22 ATM (Standard)                              │
│ Profile Hash: a3f9c2b1...                                     │
│ Devices: 4,523                                                │
│ Effective Config Hash: e8f7a3c2... (4,523 devices identical)  │
│ ✓ All devices in this profile have identical configurations  │
│                                                                │
│ [View Effective Config] [Compare to Other Profile]           │
└────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│ Profile: VZW i-22 ATM (CORD Custom)                          │
│ Profile Hash: b5f8e3c2...                                     │
│ Devices: 156                                                   │
│ Effective Config Hash: f9e4b8d1... (156 devices identical)    │
│ Uses Four-Way Rule: Custom firewall whitelist                │
│ ✓ All devices in this profile have identical configurations  │
│                                                                │
│ [View Effective Config] [Compare to Standard]                │
└────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│ Profile: VZW i-22 ATM (With Device Overrides)                │
│ Profile Hash: c9d5a7e3...                                     │
│ Devices: 23                                                    │
│ ⚠ Multiple Effective Config Hashes (devices differ):         │
│   - d1b5e7f3... (Device #12348 - custom LAN IP)              │
│   - e2c6f8a4... (Device #12501 - custom Wi-Fi)               │
│   - (21 more unique configs)                                  │
│                                                                │
│ [View Individual Configs] [Group Management]                 │
└────────────────────────────────────────────────────────────────┘
```

### 2. Configuration Comparison Tool

```
Compare Configurations

Device A: #12345 (Profile: VZW i-22 ATM Standard)
  Profile Hash: a3f9c2b1...
  Effective Hash: e8f7a3c2...

Device B: #12348 (Profile: VZW i-22 ATM With Override)
  Profile Hash: c9d5a7e3...
  Effective Hash: d1b5e7f3...

Profile Match: ❌ NO (different profiles)
Config Match: ❌ NO (different effective configs)

Differences (2 found):
┌─────────────────┬──────────────────┬──────────────────┬─────────────────┐
│ Parameter       │ Device A Value   │ Device B Value   │ Source          │
├─────────────────┼──────────────────┼──────────────────┼─────────────────┤
│ lan0_ip         │ 192.168.1.1      │ 192.168.50.100   │ Device B:       │
│                 │ (Global Layer)   │ (Device Override)│ Override        │
├─────────────────┼──────────────────┼──────────────────┼─────────────────┤
│ wifi_ssid_1     │ (not set)        │ CorpWiFi         │ Device B:       │
│                 │                  │ (Device Override)│ Override        │
└─────────────────┴──────────────────┴──────────────────┴─────────────────┘

Identical parameters: 598 / 600 (99.7%)
```

### 3. Bulk Operations by Hash

```
Bulk Configuration Update

Select Target Devices:
○ By Profile Hash (fast, approximate grouping)
  → Profile: a3f9c2b1... (4,523 devices)

● By Effective Hash (accurate, exact match)
  → Effective: e8f7a3c2... (4,523 devices)
  ✓ Guarantees all devices have identical configs

○ By Device IDs (manual selection)

Operation: Update DNS Server
  New Value: 10.1.1.1 (was: 8.8.8.8)

This will apply to: 4,523 devices
Estimated time: 15 minutes

[Preview Changes] [Apply]
```

### 4. Profile Analytics Dashboard

```
Configuration Profile Analytics

Total Devices: 100,234
Unique Profiles: 47
Unique Effective Configs: 89 (indicates 42 profiles have device overrides)

Top 10 Profiles by Device Count:
┌──────────┬────────────────────────────┬─────────┬──────────┐
│ Rank     │ Profile Description        │ Devices │ % Total  │
├──────────┼────────────────────────────┼─────────┼──────────┤
│ 1        │ VZW i-22 ATM Standard      │ 45,234  │ 45.1%    │
│ 2        │ ATT i-22 ATM Standard      │ 23,456  │ 23.4%    │
│ 3        │ VZW 4500 ATM Standard      │ 12,345  │ 12.3%    │
│ 4        │ VZW i-22 Tier1 Standard    │ 5,678   │ 5.7%     │
│ 5        │ ATT i-22 CORD Custom       │ 3,456   │ 3.4%     │
│ ...      │ ...                        │ ...     │ ...      │
└──────────┴────────────────────────────┴─────────┴──────────┘

Profiles with Device Overrides: 15 profiles (1,234 devices affected)
Profiles using Conditional Rules: 8 profiles
  - Two-Way Rules: 5 profiles
  - Three-Way Rules: 6 profiles
  - Four-Way Rules: 3 profiles
```

---

## Migration from Old System

### Map Old Filenames to New Profiles

**Create Mapping Table:**
```sql
CREATE TABLE legacy_config_file_mapping (
    filename VARCHAR(255) PRIMARY KEY,
    config_profile_hash VARCHAR(64),
    effective_config_hash VARCHAR(64),
    notes TEXT,
    migrated_at TIMESTAMP
);
```

**Populate During Migration:**
```sql
INSERT INTO legacy_config_file_mapping VALUES
('VZW_22_ATM_10142025.dat', 'a3f9c2b1...', 'e8f7a3c2...', 'Standard VZW i-22 ATM config'),
('VZW_22_ATM_CORD_05222024.dat', 'b5f8e3c2...', 'f9e4b8d1...', 'CORD custom with RMS servers'),
('ATT_22_ATM_09162025.dat', 'd7c3a9f5...', 'a2b8e4f7...', 'Standard ATT i-22 ATM config');
```

**Show Legacy Filename in UI During Transition:**
```
Device #12345
┌────────────────────────────────────────────────────────────────┐
│ Migration Status: ✓ Migrated to New System                   │
│                                                                │
│ Old System:                                                    │
│   Config File: VZW_22_ATM_10142025.dat                        │
│                                                                │
│ New System:                                                    │
│   Profile Hash: a3f9c2b1... (4,523 devices in this profile)  │
│   Effective Hash: e8f7a3c2... (identical to 4,523 devices)   │
│   Virtual Filename: VZW_22_ATM_e8f7a3c2.dat                   │
│                                                                │
│ [View Effective Config] [Compare Old vs New]                 │
└────────────────────────────────────────────────────────────────┘
```

**Validation Report:**
```
Migration Validation Report

Total Devices Migrated: 1,000
Config Matches: 998 (99.8%)
Config Mismatches: 2 (0.2%)

Mismatches Requiring Review:
┌──────────┬─────────────────────┬───────────────────────────────┐
│ Device # │ Old Config File     │ Issue                         │
├──────────┼─────────────────────┼───────────────────────────────┤
│ 12789    │ VZW_22_ATM_...dat  │ DNS value differs (8.8.8.8 vs │
│          │                     │ 10.1.1.1) - needs review      │
├──────────┼─────────────────────┼───────────────────────────────┤
│ 13456    │ ATT_22_ATM_...dat  │ Firewall rules count differs  │
│          │                     │ (40 vs 42 rules)              │
└──────────┴─────────────────────┴───────────────────────────────┘
```

---

## Performance Considerations

### Hash Calculation Performance

**Profile Hash:**
- Calculation time: <1ms per device
- Can be calculated synchronously
- Update on device attribute change (infrequent)

**Effective Config Hash:**
- Calculation time: 50-100ms per device (must resolve 600+ parameters)
- Should be calculated asynchronously
- Update when any layer changes (potentially frequent)

**Optimization Strategy:**

1. **Incremental Updates:**
   ```php
   // When Global Layer changes, queue ALL devices
   Queue::push(new RecalculateEffectiveHashJob([
       'scope' => 'global',
       'priority' => 'high'
   ]));

   // When Model Layer changes, queue only affected model
   Queue::push(new RecalculateEffectiveHashJob([
       'scope' => 'model',
       'model_id' => 22,
       'priority' => 'high'
   ]));
   ```

2. **Batch Processing:**
   ```php
   // Process in batches of 100 devices
   foreach (array_chunk($deviceIds, 100) as $batch) {
       Queue::push(new RecalculateEffectiveHashJob([
           'device_ids' => $batch
       ]));
   }
   ```

3. **Smart Caching:**
   ```php
   // If profile hash hasn't changed, effective hash likely same
   if ($device->config_profile_hash === $previousProfileHash) {
       // Skip expensive effective hash recalculation
       return;
   }
   ```

### Database Query Performance

**Indexed Columns:**
```sql
CREATE INDEX idx_devices_config_profile_hash ON devices(config_profile_hash);
CREATE INDEX idx_devices_effective_config_hash ON devices(effective_config_hash);
CREATE INDEX idx_devices_profile_effective_hash ON devices(config_profile_hash, effective_config_hash);
```

**Query Optimization:**
```sql
-- Fast: Group by profile hash (uses index)
SELECT config_profile_hash, COUNT(*) as count
FROM devices
GROUP BY config_profile_hash;

-- Fast: Find devices with same profile
SELECT * FROM devices
WHERE config_profile_hash = 'a3f9c2b1...'
LIMIT 1000;

-- Slower but accurate: Find devices with same effective config
SELECT * FROM devices
WHERE effective_config_hash = 'e8f7a3c2...'
LIMIT 1000;
```

---

## Recommended Implementation

### Phase 1: Profile Hash (MVP)
**Timeline:** Include in Phase 1 MVP
**Scope:**
- Add `config_profile_hash` column
- Calculate profile hash on device save
- Admin UI shows profile hash grouping
- Bulk operations by profile hash

**Deliverables:**
- Database migration
- Profile hash calculation logic
- Admin UI device list with grouping
- Bulk operation targeting by profile

### Phase 2: Effective Config Hash
**Timeline:** Phase 2
**Scope:**
- Add `effective_config_hash` column
- Background job for hash recalculation
- Comparison tool showing exact differences
- Validation that profile hash matches effective hash

**Deliverables:**
- Effective hash calculation logic
- Queue jobs for async recalculation
- Config comparison UI
- Migration validation report

### Phase 3: Enhanced Features (Optional)
**Timeline:** Phase 2+
**Scope:**
- Configuration templates (Approach 2)
- Virtual filename display (Approach 3)
- Analytics dashboard
- Advanced grouping/filtering

---

## Success Criteria

### Must Have (Phase 1)
- ✅ Admin can see how many devices share a configuration profile
- ✅ Admin can select all devices with same profile for bulk operations
- ✅ Admin can identify devices with device-level overrides
- ✅ Performance acceptable for 100,000 devices (<2 seconds to group)

### Should Have (Phase 2)
- ✅ Admin can verify devices have truly identical effective configs
- ✅ Admin can see exact differences between two device configs
- ✅ System validates profile hash matches effective hash
- ✅ Migration validation confirms old and new configs match

### Nice to Have (Phase 3)
- ✅ Admin can create/name configuration templates
- ✅ Admin can see virtual filenames for migration familiarity
- ✅ Analytics dashboard shows config profile distribution
- ✅ System suggests config optimizations (consolidation opportunities)

---

## Open Questions

1. **Hash Recalculation Timing:**
   - Should effective hash be calculated immediately when layer changes, or queued?
   - What's acceptable delay for hash to be current?
   - **Recommendation:** Queue as background job, show "calculating..." status in UI

2. **Profile Hash Granularity:**
   - Should conditional rules be included in profile hash?
   - Or should they only affect effective hash?
   - **Recommendation:** Include conditional rule indicators in profile hash (has_two_way_rules, has_three_way_rules, has_four_way_rules booleans)

3. **Template Creation:**
   - Should templates be auto-created or admin-created only?
   - What's the threshold for auto-creating templates? (10+ devices? 100+ devices?)
   - **Recommendation:** Start with profile hash only (Phase 1), add templates later if needed (Phase 3)

4. **Virtual Filename Display:**
   - Show always, or only during migration period?
   - Store in database or generate on-the-fly?
   - **Recommendation:** Generate on-the-fly, show during migration, make optional after migration complete

---

## Decision Required

**Stakeholder Input Needed:**

1. **Approach Selection:** Hybrid (Profile + Effective Hash) vs Single approach
   - **Recommendation:** Hybrid approach for best balance

2. **Phase 1 Scope:** Profile Hash only vs Full hybrid in Phase 1
   - **Recommendation:** Profile Hash only in Phase 1, Effective Hash in Phase 2

3. **UI Priority:** What grouping/comparison features are must-have vs nice-to-have?
   - **Recommendation:** Device list grouping and bulk operations are must-have (Phase 1)

4. **Performance Targets:** What's acceptable recalculation time for 100,000 devices?
   - **Recommendation:** <5 minutes for profile hash, <30 minutes for effective hash (async)

---

**Document Status:** Solution Proposed - Awaiting Review
**Next Steps:**
1. Review proposed solution with stakeholders
2. Validate performance assumptions with engineering
3. Prioritize features for Phase 1 vs Phase 2+
4. Create implementation tickets

**Last Updated:** March 1, 2026
