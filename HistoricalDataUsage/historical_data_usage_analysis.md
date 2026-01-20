# Single Device Historical Data Usage Feature - Complete Technical Analysis

## Overview
The Single Device Historical Data Usage feature displays a bar chart showing a device's data consumption across the last 3 billing cycles on the device view page (`/admin/devices/view/{device_id}`). This is a read-only visualization that helps users track data usage trends over time.

---

## 1. CONTROLLER LAYER - Data Retrieval & Preparation

### File: `/Users/aksana/Documents/Projects/WATM/watm/watm/plugins/Devices/src/Controller/Admin/DevicesController.php`

**Action**: `view()` (line 544-1057)

#### Key Steps:

1. **Fetch Current Billing Cycle** (lines 546-548)
   ```php
   $billingCycle = $this->Devices->CompanyDeviceUsages->BillingCycles->findOrCreate(
       $this->Devices->CompanyDeviceUsages->BillingCycles->getCurrentConditions()
   );
   ```
   - Gets or creates the current billing cycle
   - Used for context and filtering comparisons

2. **Load Device with Eager Loading** (lines 550-702)
   - Uses `contain()` to eager-load relationships:
     - `DeviceManufacturers` - Device maker info
     - `DeviceStatuses` - Current status
     - `CompanyDeviceUsages` - Usage for current billing cycle
     - `BillingCycles` - Billing context
     - `HistoricalServicePlans` - Service plan pricing tiers
     - `Companies` - Company info
     - `ServicePlans` - Device's selected service plan
     - `States`, `Countries` - Location data

3. **Query Historical Data Usages** (lines 706-715)
   ```php
   $historicalDataUsages = null;
   if (isset($device->company_id)) {
       $historicalDataUsages = $this
           ->Devices
           ->CompanyDeviceUsages
           ->find(
               'historicalDataUsages',
               ['device_id' => $device->id, 'company_id' => $device->company_id]
           );
   }
   ```
   - Uses custom finder `historicalDataUsages` (see below)
   - Only executes if device has an associated company
   - Result is a Query object (lazy-loaded until template renders)

4. **Calculate Signal Strength Stats** (lines 724-747)
   - Uses `DeviceStatusLogs` for daily signal strength averages
   - Converts to percentages for display

5. **Prepare Usage Data for Display** (lines 954-987)
   ```php
   $usage = current($device->company_device_usages);
   $currentDeviceUsage = empty($usage) ? 0 : $usage->current_data_usage;
   
   // Calculate projected usage based on daily average
   $avg = $currentDays === 0 ? $currentDeviceUsage : $currentDeviceUsage / $currentDays;
   $projectedDeviceUsage = ($avg * $remainingDays) + $currentDeviceUsage;
   ```
   - Extracts current cycle usage from eager-loaded data
   - Projects end-of-cycle usage using daily average formula

6. **Set Variables for Template** (lines 1019-1042)
   ```php
   $this->set(
       compact(
           'billingCycle',
           'device',
           'historicalDataUsages',  // ← Passed to template
           'currentDeviceUsage',
           'projectedDeviceUsage',
           // ... other variables
       )
   );
   ```

---

## 2. FINDER METHOD - Custom Query Builder

### File: `/Users/aksana/Documents/Projects/WATM/watm/watm/plugins/Devices/src/Model/Table/CompanyDeviceUsagesTable.php`

**Method**: `findHistoricalDataUsages()` (lines 574-583)

```php
public function findHistoricalDataUsages(Query $query, array $options): Query
{
    $query
        ->contain(['BillingCycles'])
        ->limit(3)
        ->order(['CompanyDeviceUsages.created' => 'DESC'])
        ->where(['device_id' => $options['device_id'], 'company_id' => $options['company_id']]);

    return $query;
}
```

**What it does:**
1. **Limits Results to 3 Records** - Shows last 3 billing cycles
2. **Sorts by Creation Date (Descending)** - Newest first
3. **Includes BillingCycles Relationship** - Brings in cycle dates via eager loading
4. **Filters by Device & Company** - Ensures data isolation and relevance
5. **Returns a Query Object** - Lazy-loaded, executed when template iterates

**Key Parameters:**
- `device_id`: The device being viewed
- `company_id`: Ensures multi-tenant data isolation

---

## 3. DATABASE SCHEMA - Data Storage

### Table: `company_device_usages`

**Key Columns:**
| Column | Type | Purpose |
|--------|------|---------|
| `id` | INT (PK) | Primary key |
| `billing_cycle_id` | INT (FK) | Links to billing cycle |
| `company_id` | INT (FK) | Multi-tenant isolation |
| `device_id` | INT (FK) | Which device |
| `current_data_usage` | INT | Bytes used in this cycle |
| `created` | DATETIME | When record was created |
| `modified` | DATETIME | When record was updated |
| `verizon_buffer`, `att_buffer`, `tmobile_buffer` | INT | Provider-specific usage tracking |
| `transfer_buffer`, `transfer_offset` | INT | Data transfer between devices |

**Foreign Keys:**
- `billing_cycle_id` → `billing_cycles.id`
- `company_id` → `companies.id`
- `device_id` → `devices.id`

**Data Flow:**
1. Device sends check-in via UDP → Node.js service → SQS queue
2. CakePHP queue worker processes message → updates `company_device_usages.current_data_usage`
3. Each billing cycle creates one record per device per company payment method
4. Historical queries retrieve the last 3 billing cycles for a device

### Table: `billing_cycles`

**Virtual Properties:**
- `short_display`: Format like `"1/1/25 - 1/31/25"` (used in chart labels)
- `display`: Format like `"January 1, 2025 - January 31, 2025"`

---

## 4. TEMPLATE LAYER - Display & Visualization

### Main View File: `/Users/aksana/Documents/Projects/WATM/watm/watm/plugins/Devices/templates/Admin/Devices/view.php`

**Chart Section** (lines 590-603):
```php
if (
    isset($historicalDataUsages) &&
    $historicalDataUsages->count() &&
    ($device->has('service_plan') && 
     !$device->service_plan->last_approved_historical_service_plan->hide_data_usage)
): ?>
    <?= $this->element('historical_data_usage_chart'); ?>
<?php else: ?>
    <p class="ml-3">N/A</p>
<?php endif; ?>
```

**Display Conditions:**
1. `$historicalDataUsages` must exist
2. Must have at least one record
3. Service plan must not hide data usage (privacy setting)
4. If conditions fail, shows "N/A"

### Element: `/Users/aksana/Documents/Projects/WATM/watm/watm/plugins/Devices/templates/element/historical_data_usage_chart.php`

**HTML Structure:**
```html
<div class="row mx-3 my-1">
    <div class="p-3 bg-white col-md-12">
        <h2 class="mb-3">Historical Data Usage</h2>
        <div class="chartWrapper" style="position: relative;overflow: auto;width: 100%;">
            <div class="chartContainer" style="position: relative;height: 500px;">
                <canvas id="historical-data-usage-chart"></canvas>
            </div>
        </div>
    </div>
</div>
```

---

## 5. JAVASCRIPT/CLIENT-SIDE - Chart Rendering

### Chart.js Integration

**File Included** (line 18):
```php
<?php $this->Html->script('/Dashboard/js/chart.min', ['block' => 'script']); ?>
```
- Loads Chart.js library from `/Dashboard/js/chart.min.js`
- Chart.js is a popular open-source charting library

**Chart Configuration** (lines 21-76):

```javascript
const historicalDataUsageChart = document.getElementById('historical-data-usage-chart');

// 1. Build Data Structure
const data = {
    labels: [
        <?php foreach($historicalDataUsages as $usage): ?>
            "<?=$usage->billing_cycle->short_display;?>",
        <?php endforeach; ?>
    ],
    datasets: [
        {
            data: [
                <?php foreach($historicalDataUsages as $usage): ?>
                    <?=(($usage->current_data_usage ?? 0)/(1024*1024*1024));?>,
                <?php endforeach; ?>
            ],
        },
    ]
};

// 2. Create Chart Instance
const myChart = new Chart(historicalDataUsageChart, {
    type: 'bar',
    data: data,
    options: {
        layout: { autoPadding: true },
        scales: {
            y: {
                ticks: {
                    callback: function(value) {
                        return value.toFixed(5) + ' GB';
                    }
                }
            }
        },
        plugins: {
            legend: { display: false },
            title: { display: false },
            tooltip: {
                callbacks: {
                    label: function(context) {
                        return context.raw.toFixed(5) + ' GB';
                    }
                }
            },
        }
    },
});
```

**Key Features:**

1. **Data Preparation (PHP → JavaScript)**
   - Iterates through `$historicalDataUsages` query result
   - Extracts billing cycle display dates as labels
   - Converts data usage from bytes to GB (dividing by 1024*1024*1024)

2. **Chart Type**: Bar chart (`type: 'bar'`)

3. **Y-Axis Formatting**
   - Shows values in GB with 5 decimal places
   - Example: `"1.23456 GB"`

4. **Tooltips**
   - Hover shows formatted data: `"1.23456 GB"`

5. **Visual Simplification**
   - No legend (only one dataset)
   - No title (section has h2 header)
   - Auto padding for responsive sizing

---

## 6. DATA FLOW DIAGRAM

```
┌─────────────────────────────────────────────────────────────────┐
│ User visits: /admin/devices/view/{device_id}                    │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
        ┌────────────────────────────────┐
        │ DevicesController::view()       │
        │ (lines 544-1057)               │
        └────────────┬───────────────────┘
                     │
        ┌────────────┴───────────────────────────────────────────┐
        │                                                        │
        ▼                                                        ▼
  ┌──────────────────────────────┐            ┌─────────────────────────────┐
  │ Load Device with Relationships   │            │ Query Historical Data       │
  │ - Companies                  │            │ $historicalDataUsages =     │
  │ - ServicePlans               │            │ CompanyDeviceUsages.find(   │
  │ - CompanyDeviceUsages        │            │   'historicalDataUsages'... │
  │   (current cycle)            │            │ )                            │
  └──────────────────────────────┘            └──────────┬──────────────────┘
        │                                                 │
        │                          ┌──────────────────────┘
        │                          │
        ▼                          ▼
  ┌─────────────────────────────────────────────────────────┐
  │ CompanyDeviceUsagesTable::findHistoricalDataUsages()   │
  │ (lines 574-583)                                        │
  │ - limit(3)                                             │
  │ - order by created DESC                                │
  │ - contain(['BillingCycles'])                           │
  │ - where device_id, company_id                          │
  └──────────────────┬──────────────────────────────────────┘
                     │
                     ▼
            ┌─────────────────────┐
            │ Database Query      │
            │ SELECT * FROM       │
            │ company_device_     │
            │ usages WHERE ...    │
            │ LIMIT 3             │
            │ ORDER BY created... │
            └──────────┬──────────┘
                       │
                       ▼
            ┌─────────────────────┐
            │ ResultSet with:     │
            │ - current_data_     │
            │   usage (bytes)     │
            │ - BillingCycles     │
            │   (3 cycles)        │
            └──────────┬──────────┘
                       │
                       ▼
        ┌──────────────────────────────────┐
        │ $this->set(compact(...)) passes  │
        │ $historicalDataUsages to view    │
        └──────────────┬───────────────────┘
                       │
                       ▼
    ┌───────────────────────────────────────────────┐
    │ Template: view.php + element:                 │
    │ historical_data_usage_chart.php               │
    └──────────────────┬────────────────────────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
        ▼                             ▼
  ┌──────────────┐          ┌──────────────────────┐
  │ HTML Canvas: │          │ JavaScript Data      │
  │ #historical- │          │ - Extract labels from│
  │ data-usage-  │          │   billing_cycle.     │
  │ chart        │          │   short_display      │
  └──────┬───────┘          │ - Convert bytes → GB │
         │                  │ - Build Chart.js obj │
         │                  └──────────┬───────────┘
         │                             │
         └─────────────────┬───────────┘
                           │
                           ▼
            ┌──────────────────────────────┐
            │ Chart.js Renders:            │
            │ - Bar chart                  │
            │ - 3 bars (3 billing cycles)  │
            │ - Y-axis: GB format (5 dec)  │
            │ - Tooltips on hover          │
            │ - Height: 500px              │
            └──────────────────────────────┘
```

---

## 7. EXAMPLE DATA FLOW - CONCRETE SCENARIO

**Scenario**: Viewing device #42 in company #5

**Request URL**: `/admin/devices/view/42`

**Controller Execution**:
1. Calls `CompanyDeviceUsages->find('historicalDataUsages', ['device_id' => 42, 'company_id' => 5])`

**Database Query Generated**:
```sql
SELECT `CompanyDeviceUsages`.*
FROM `company_device_usages` AS `CompanyDeviceUsages`
WHERE 
    device_id = 42 
    AND company_id = 5
ORDER BY `CompanyDeviceUsages`.`created` DESC
LIMIT 3
```

**Sample Query Result** (3 records, newest first):
```
[
  {
    id: 1050,
    billing_cycle_id: 12,      // November 2024
    device_id: 42,
    company_id: 5,
    current_data_usage: 5368709120,  // ~5 GB in bytes
    billing_cycle: {
      id: 12,
      cycle_start: 2024-11-01,
      cycle_end: 2024-11-30,
      short_display: "11/1/24 - 11/30/24"
    }
  },
  {
    id: 1001,
    billing_cycle_id: 11,      // October 2024
    device_id: 42,
    company_id: 5,
    current_data_usage: 3221225472,  // ~3 GB
    billing_cycle: {
      short_display: "10/1/24 - 10/31/24"
    }
  },
  {
    id: 952,
    billing_cycle_id: 10,      // September 2024
    device_id: 42,
    company_id: 5,
    current_data_usage: 7516192768,  // ~7 GB
    billing_cycle: {
      short_display: "9/1/24 - 9/30/24"
    }
  }
]
```

**Template Rendering**:
```javascript
// PHP in template generates:
const data = {
    labels: [
        "11/1/24 - 11/30/24",
        "10/1/24 - 10/31/24",
        "9/1/24 - 9/30/24"
    ],
    datasets: [
        {
            data: [
                5.0,   // 5368709120 / (1024*1024*1024)
                3.0,   // 3221225472 / (1024*1024*1024)
                7.0    // 7516192768 / (1024*1024*1024)
            ]
        }
    ]
};
```

**Final Chart Display**:
```
Historical Data Usage

         7.5 GB ├─────────────────
                │
         5.0 GB ├─ ■         ■
                │ ■ ■       ■
         2.5 GB ├─ ■ ■ ■   ■
                │ ■ ■ ■ ■ ■
         0.0 GB └─ ■ ■ ■ ■ ■
                   11/1 10/1  9/1
```

---

## 8. KEY ASSOCIATIONS & RELATIONSHIPS

### CompanyDeviceUsages Associations (Lines 79-108 in Table file):

```php
$this->belongsTo('BillingCycles');          // Foreign Key: billing_cycle_id
$this->belongsTo('Companies');              // Foreign Key: company_id
$this->belongsTo('Devices');                // Foreign Key: device_id
$this->belongsTo('HistoricalServicePlans'); // Links to service plan pricing
$this->belongsTo('CompanyPaymentMethods');  // Who's being billed
$this->hasMany('ProviderDataUsages');       // Detailed usage by provider
$this->hasMany('InvoiceDetails');           // Links to invoices
```

### Devices Associations (relevant):

```php
$this->hasMany('CompanyDeviceUsages', [
    'foreignKey' => ['device_id', 'company_id'],
    'bindingKey' => ['id', 'company_id']
]);
```

---

## 9. IMPORTANT BUSINESS LOGIC

### Data Usage Calculation (CompanyDeviceUsagesTable::beforeSave, lines 368-406)

```php
$currentUsage = (
    $entity->verizon_buffer + 
    $entity->att_buffer + 
    $entity->tmobile_buffer + 
    $entity->transfer_buffer
) - (
    $entity->transfer_offset + 
    $entity->verizon_offset + 
    $entity->tmobile_offset
);

$entity->current_data_usage = ($currentUsage >= 0) ? $currentUsage : 0;
```

- Aggregates usage from multiple cellular carriers
- Accounts for data transfers between devices
- Ensures non-negative values

### Data Suspension Logic (afterSave, lines 442-465)

When a device exceeds max usage limit for its service plan:
1. Device status automatically changes to "Data Suspension"
2. Event is logged
3. Company is notified

### Privacy Control (Template, line 596)

```php
!$device->service_plan->last_approved_historical_service_plan->hide_data_usage
```

- Service plans can hide data usage for privacy
- Chart only displays if `hide_data_usage = false`

---

## 10. QUERY PARAMETERS & CONDITIONAL DISPLAY

**Chart Display Conditions** (template line 591-596):
1. ✓ `$historicalDataUsages` is set (not null)
2. ✓ `$historicalDataUsages->count() > 0` (at least 1 record)
3. ✓ Device has a service plan
4. ✓ Service plan's `hide_data_usage = false`

**If any condition fails**: Shows "N/A" instead of chart

---

## 11. PERFORMANCE CONSIDERATIONS

**Query Optimization:**
- Only fetches 3 billing cycles (LIMIT 3) - minimal data transfer
- Single query with eager loading via `contain(['BillingCycles'])`
- Compound filter on `device_id` + `company_id` + indexed fields
- Sorted by `created DESC` without JOIN overhead

**Rendering:**
- Chart.js renders client-side (reduces server load)
- Data preparation happens at template render time
- No additional AJAX calls needed

---

## 12. SUMMARY OF COMPONENT INTERACTIONS

| Component | Responsibility |
|-----------|-----------------|
| **Controller** | Orchestrates data fetching, calculations, template context setup |
| **Finder** | Encapsulates query logic for retrieving last 3 billing cycles |
| **Table Model** | Database interaction, relationships, before/after hooks |
| **Template** | Conditional display, visibility checks |
| **Element** | Chart HTML structure, data iteration |
| **JavaScript** | Data transformation (bytes→GB), Chart.js configuration |
| **Chart.js** | Rendering bar chart visualization |

---

## 13. REFRESH FUNCTIONALITY

**Refresh Data Usage Button** (template lines 606-619):
```php
<?= $this->Form->postLink(
    __('Refresh Data Usage'),
    ['action' => 'view', $device->id],
    ['data' => ['refresh-data-usage' => true], 'class' => 'btn btn-primary']
) ?>
```

**Controller Handling** (lines 753-759):
```php
elseif (!empty($this->request->getData('refresh-data-usage'))) {
    try {
        $this->Devices->fetchAndStoreDeviceUsage($device->id, false, false, false);
    } catch (Exception $e) {
        $this->Flash->error($e->getMessage());
    }
    return $this->redirect(['action' => 'view', $device->id, '?' => ['stats-tab' => 'data-usage']]);
}
```

- Manually triggers device usage fetching
- Updates `current_data_usage` in database
- Redirects back to view with tab anchor

---

## 14. MULTI-TENANCY & DATA ISOLATION

**Isolation Points:**

1. **Controller Level**:
   - `$userCompany = $this->getCompany();` (line 704)
   - Validates user has access to this device's company

2. **Query Level**:
   - `where(['device_id' => $device->id, 'company_id' => $device->company_id])`
   - Ensures only relevant company's data is queried

3. **Permission System**:
   - RBAC checks before view action executes
   - Managed by `UserManagement` plugin

---

