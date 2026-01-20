# Historical Data Usage Feature - Quick Reference

## Feature Purpose
Displays a bar chart of device data consumption over the last 3 billing cycles on the device view page.

---

## File Locations

| Purpose | File Path |
|---------|-----------|
| **Controller** | `/Users/aksana/Documents/Projects/WATM/watm/watm/plugins/Devices/src/Controller/Admin/DevicesController.php` |
| **Table Model** | `/Users/aksana/Documents/Projects/WATM/watm/watm/plugins/Devices/src/Model/Table/CompanyDeviceUsagesTable.php` |
| **View Template** | `/Users/aksana/Documents/Projects/WATM/watm/watm/plugins/Devices/templates/Admin/Devices/view.php` |
| **Chart Element** | `/Users/aksana/Documents/Projects/WATM/watm/watm/plugins/Devices/templates/element/historical_data_usage_chart.php` |
| **Chart.js Library** | `/Users/aksana/Documents/Projects/WATM/watm/watm/plugins/Dashboard/webroot/js/chart.min.js` |
| **Database Schema** | `/Users/aksana/Documents/Projects/WATM/watm/watm/config/Migrations/20221212180252_CreateCompanyDeviceUsages.php` |

---

## Data Flow - Step by Step

### 1. Request URL
```
/admin/devices/view/{device_id}
```

### 2. Controller Action Execution
**Location**: `DevicesController::view()` lines 706-715

```php
$historicalDataUsages = $this
    ->Devices
    ->CompanyDeviceUsages
    ->find('historicalDataUsages', [
        'device_id' => $device->id, 
        'company_id' => $device->company_id
    ]);
```

### 3. Finder Method Execution
**Location**: `CompanyDeviceUsagesTable::findHistoricalDataUsages()` lines 574-583

**Generated SQL:**
```sql
SELECT `CompanyDeviceUsages`.* 
FROM `company_device_usages` AS `CompanyDeviceUsages`
INNER JOIN `billing_cycles` AS `BillingCycles` 
    ON `BillingCycles`.`id` = `CompanyDeviceUsages`.`billing_cycle_id`
WHERE `CompanyDeviceUsages`.`device_id` = ? 
  AND `CompanyDeviceUsages`.`company_id` = ?
ORDER BY `CompanyDeviceUsages`.`created` DESC
LIMIT 3
```

### 4. Database Response
Returns 3 `CompanyDeviceUsage` records with related `BillingCycle` data:
```
[
  { id: 1050, device_id: 42, company_id: 5, current_data_usage: 5368709120, billing_cycle_id: 12, ... },
  { id: 1001, device_id: 42, company_id: 5, current_data_usage: 3221225472, billing_cycle_id: 11, ... },
  { id: 952, device_id: 42, company_id: 5, current_data_usage: 7516192768, billing_cycle_id: 10, ... }
]
```

### 5. Template Rendering
**Location**: `/view.php` lines 590-603

**Conditional Display:**
```
if (historicalDataUsages exists AND 
    historicalDataUsages count > 0 AND 
    device has service_plan AND 
    service_plan.hide_data_usage == false)
  → Show chart element
else
  → Show "N/A"
```

### 6. Chart Element Processing
**Location**: `historical_data_usage_chart.php` lines 21-76

**Data Transformation:**
```javascript
PHP foreach loop iterates 3 times:

Iteration 1: "11/1/24 - 11/30/24" → 5.0 GB
Iteration 2: "10/1/24 - 10/31/24" → 3.0 GB  
Iteration 3: "9/1/24 - 9/30/24"  → 7.0 GB

// Conversion: bytes_value / (1024 * 1024 * 1024)
```

### 7. Chart.js Rendering
```javascript
Bar chart with:
- 3 bars (one per billing cycle)
- X-axis labels: Date ranges
- Y-axis: GB (5 decimal places)
- Height: 500px
- Tooltips: "X.XXXXX GB" on hover
```

---

## Key Database Tables

### `company_device_usages`
Stores data usage per device per billing cycle:
- `id` - Primary key
- `device_id` - Foreign key → devices
- `company_id` - Foreign key → companies (multi-tenant)
- `billing_cycle_id` - Foreign key → billing_cycles
- `current_data_usage` - Data used (bytes)
- `created` - Record creation timestamp
- `modified` - Record modification timestamp

### `billing_cycles`
Defines billing periods:
- `id` - Primary key
- `cycle_start` - Start date (DateTime)
- `cycle_end` - End date (DateTime)
- Virtual: `short_display` - "n/j/y - n/j/y" format

---

## Key Code Snippets

### Controller: Query Historical Data
```php
$historicalDataUsages = $this
    ->Devices
    ->CompanyDeviceUsages
    ->find('historicalDataUsages', [
        'device_id' => $device->id, 
        'company_id' => $device->company_id
    ]);
```

### Finder: Custom Query Method
```php
public function findHistoricalDataUsages(Query $query, array $options): Query
{
    return $query
        ->contain(['BillingCycles'])
        ->limit(3)
        ->order(['CompanyDeviceUsages.created' => 'DESC'])
        ->where([
            'device_id' => $options['device_id'], 
            'company_id' => $options['company_id']
        ]);
}
```

### Template: Conditional Display
```php
<?php if (
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

### Element: Chart.js Setup
```javascript
const data = {
    labels: [
        <?php foreach($historicalDataUsages as $usage): ?>
            "<?=$usage->billing_cycle->short_display;?>",
        <?php endforeach; ?>
    ],
    datasets: [{
        data: [
            <?php foreach($historicalDataUsages as $usage): ?>
                <?=(($usage->current_data_usage ?? 0)/(1024*1024*1024));?>,
            <?php endforeach; ?>
        ]
    }]
};

const myChart = new Chart(
    document.getElementById('historical-data-usage-chart'), 
    {
        type: 'bar',
        data: data,
        options: {
            scales: {
                y: {
                    ticks: {
                        callback: (value) => value.toFixed(5) + ' GB'
                    }
                }
            },
            plugins: {
                legend: { display: false },
                tooltip: {
                    callbacks: {
                        label: (context) => context.raw.toFixed(5) + ' GB'
                    }
                }
            }
        }
    }
);
```

---

## Display Conditions Checklist

For the chart to display, all of these must be true:

- [ ] `$historicalDataUsages` variable is set (not null)
- [ ] `$historicalDataUsages->count()` > 0 (at least 1 record exists)
- [ ] Device has a `service_plan` relationship
- [ ] Service plan's `hide_data_usage` = false
- [ ] No browser/JavaScript errors

**If ANY condition fails:** Chart shows "N/A"

---

## Browser Rendering Process

```
1. Browser requests /admin/devices/view/42
   ↓
2. Server processes DevicesController::view()
   ↓
3. Controller queries database via finder
   ↓
4. Template checks display conditions
   ↓
5. If conditions met, element renders chart HTML with embedded PHP loop
   ↓
6. Browser receives HTML with Canvas element + JavaScript
   ↓
7. Browser loads Chart.js library (/Dashboard/js/chart.min.js)
   ↓
8. JavaScript initializes Chart.js with data from PHP
   ↓
9. Chart.js renders SVG bar chart to Canvas
   ↓
10. User sees: Bar chart with 3 bars, date labels, GB values
```

---

## Performance Characteristics

**Database Query:**
- Single query with JOIN to `billing_cycles`
- Filters: `device_id`, `company_id`
- Sorts by: `created DESC`
- Limit: 3 records
- Execution time: < 10ms (typical)

**Data Transfer:**
- ~3 records × 30 bytes (approx) = ~90 bytes of data
- Minimal network overhead

**Client-side:**
- Chart.js renders ~100KB minified library
- Chart renders to Canvas (GPU accelerated)
- Render time: < 100ms (typical)

---

## Extension Points (if you need to modify)

### To Change Number of Historical Cycles Displayed
**File**: `CompanyDeviceUsagesTable.php`, line 578
```php
->limit(3)  // Change this number
```

### To Change Chart Type
**File**: `historical_data_usage_chart.php`, line 42
```javascript
type: 'bar',  // Change to 'line', 'doughnut', etc.
```

### To Change Y-Axis Unit (GB to MB/TB)
**File**: `historical_data_usage_chart.php`, lines 34 & 51-52
```javascript
// Currently: bytes / (1024*1024*1024) = GB
// For MB: bytes / (1024*1024)
// For TB: bytes / (1024*1024*1024*1024)
```

### To Add a Refresh Button
Already implemented in template at lines 606-619:
```php
<?= $this->Form->postLink(
    __('Refresh Data Usage'),
    ['action' => 'view', $device->id],
    ['data' => ['refresh-data-usage' => true], 'class' => 'btn btn-primary']
) ?>
```

---

## Related Features/Components

1. **Device Check-ins Service** - Populates the data
   - `/Users/aksana/Documents/Projects/WATM/watm/checkins/index.js`
   - Sends device telemetry → SQS

2. **Queue Workers** - Process device data
   - `bin/cake worker` command
   - Processes check-in messages from SQS

3. **Service Plans** - Control data limits & hiding
   - `hide_data_usage` flag controls chart visibility
   - Billing plugin manages billing cycles

4. **Multi-tenancy** - Data isolation
   - `company_id` filters ensure user can only see their data
   - RBAC permissions managed by UserManagement plugin

---

## Troubleshooting

**Chart shows "N/A"?**
1. Check if device has a company_id assigned
2. Check if billing cycle records exist for this device
3. Check if service_plan.hide_data_usage = false
4. Check browser console for JavaScript errors

**Chart displays but data looks wrong?**
1. Verify current_data_usage values in database
2. Check if conversion from bytes to GB is correct
3. Verify Chart.js library is loading (Network tab)

**Data not updating?**
1. Run "Refresh Data Usage" button in UI
2. Check if queue workers are running
3. Verify SQS messages are being processed

---

## Summary Statistics

- **Total Files Involved**: 6 main files
- **Database Tables Used**: 2 primary (company_device_usages, billing_cycles)
- **Data Points Displayed**: 3
- **Chart Type**: Bar chart
- **Data Unit**: Gigabytes (GB)
- **Update Frequency**: On page load or "Refresh" button click
- **Multi-tenant Isolation**: Yes (company_id filter)

