# WATM Admin Dashboard Improvement Ideas

**Date:** November 19, 2025
**Project:** WATM - Wireless Access and Telemetry Management
**Purpose:** Comprehensive dashboard enhancement recommendations

---

## Table of Contents

1. [Current State Analysis](#current-state-analysis)
2. [Your Requested Features](#your-requested-features)
3. [Additional Critical Metrics for Admins](#additional-critical-metrics-for-admins)
4. [Advanced Analytics & Visualizations](#advanced-analytics--visualizations)
5. [Technical Implementation Recommendations](#technical-implementation-recommendations)
6. [Priority Roadmap](#priority-roadmap)

---

## Current State Analysis

### What We Have Now ✅

The Dashboard plugin currently provides:
- Device status widgets (all devices, connection status, status breakdown)
- Device uptime metrics (7-day, 30-day tracking)
- Data usage per service plan
- Unique check-ins (last 2 hours)
- Devices per carrier breakdown
- Google Maps with device locations
- Recent alerts (last 7 days)
- Recent notifications (last 7 days)

### What's Missing or Needs Improvement ❌

- **No customer/company overview** - Can't see customer counts, growth, or hierarchies
- **Limited trend analysis** - Most metrics are point-in-time, not time-series
- **No billing integration** - Revenue, invoices, and payment status not visible
- **Basic visualizations** - Only doughnut charts, no bar/line/area charts
- **No drill-down capabilities** - Can't click widgets to see details
- **Average uptime disabled** - Currently returns "N/A" due to technical issues
- **No carrier performance comparison** - Have data but not visualized
- **Limited alert analytics** - Just a list, no distribution or trends

---

## Your Requested Features

### 1. My Customers 👥

#### Customer Overview Widget
**What to Show:**
- **Total Customers Count** - Badge showing active companies
- **Active vs Inactive** - Pie chart breakdown
- **Account Types** - Distribution (Distributor, Customer, Dealer, etc.)
- **Customer Status** - Active, Suspended, Deactivated counts
- **Parent-Child Relationships** - How many parent companies vs child accounts

**Additional Metrics:**
- New customers this month/quarter/year
- Customers by state/region (US map)
- Top 10 customers by device count
- Top 10 customers by revenue
- Customer churn rate (customers lost vs gained)
- Customer lifetime value (CLV) estimates

**Drill-Down:**
- Click customer count → Full customer list with filters
- Click customer name → Company detail page
- Click status → Filtered list of customers in that status

**Database Tables:**
- `companies` table (company_status_id, account_type_id, parent_company_id, created)
- Join with `devices` for device counts
- Join with `invoices` for revenue data

---

### 2. Devices by Companies 🏢📱

#### Company Device Distribution Widget
**What to Show:**
- **Device Count per Company** - Sortable table
  - Company Name
  - Total Devices
  - Active Devices
  - Inactive Devices
  - % Utilization
- **Top 10 Companies by Device Count** - Horizontal bar chart
- **Device Distribution** - Histogram showing how many companies have X devices
  - Example: 50 companies with 1-10 devices, 30 with 11-50, 20 with 51-100, etc.

**Advanced Views:**
- **Hierarchical View** - Parent company with child company rollups
- **Growth Trend** - Line chart showing device additions per company over time
- **Comparison View** - Compare device counts across companies side-by-side

**Filters:**
- By account type (Distributor, Customer, etc.)
- By company status
- By date range (device additions)
- By state/region
- By device manufacturer

**Database Implementation:**
```sql
SELECT
  c.id, c.title,
  COUNT(d.id) as total_devices,
  SUM(CASE WHEN ds.title = 'Active' THEN 1 ELSE 0 END) as active_devices,
  SUM(CASE WHEN das.title = 'Offline' THEN 1 ELSE 0 END) as offline_devices
FROM companies c
LEFT JOIN devices d ON c.id = d.company_id
LEFT JOIN device_statuses ds ON d.device_status_id = ds.id
LEFT JOIN device_addl_statuses das ON d.device_addl_status_id = das.id
GROUP BY c.id, c.title
ORDER BY total_devices DESC
```

---

### 3. Status Dashboard 📊

#### Comprehensive Device Status Widget
**What to Show:**
- **Primary Status Distribution** (Enhanced)
  - Active, Deactivated, Admin Suspension, Data Suspension, Customer Suspension
  - Stacked bar chart showing trends over time (last 30 days)
- **Additional Status** (New)
  - Offline vs Online counts
  - How long devices have been in each status
- **Status Transitions** (New)
  - Flow diagram showing status changes this month
  - Most common status transitions

#### Connection Status Widget (Enhanced)
**What to Show:**
- **Real-Time Connection Status**
  - Connected (checked in within last 2 hours)
  - Recently Disconnected (2-24 hours)
  - Disconnected (24+ hours)
  - Never Connected
- **Connection Quality**
  - Signal strength distribution (Excellent, Good, Fair, Poor)
  - SINR percentage averages by device model
- **Connection Trend** - Line chart showing connection status over last 30 days

**Alerts:**
- Highlight devices newly offline (status changed in last 24 hours)
- Show count of devices offline > 7 days (needs attention)
- Devices with poor signal strength (< 20%)

**Database Tables:**
- `devices` (device_status_id, device_addl_status_id, last_ping_time)
- `device_status_logs` (from_check_in, signal_strength, sinr_percentage)
- `device_statuses`, `device_addl_statuses`

---

### 4. Devices per Carrier 📡

#### Carrier Distribution Widget (Enhanced)
**Current:** Simple count breakdown
**Enhanced Features:**

**Visual Displays:**
- **Pie Chart** - AT&T vs Verizon vs T-Mobile vs Dual Carrier
- **Bar Chart** - Carrier comparison with active vs inactive breakdown
- **Trend Chart** - Carrier distribution changes over time (device activations by carrier)

**Advanced Metrics:**
- **Dual SIM Analysis**
  - How many dual-sim devices are active
  - Primary vs backup carrier usage
  - Cellular backup devices vs WiFi-capable
- **Carrier Performance Comparison**
  - Average signal strength by carrier (RSSI)
  - Average data usage by carrier
  - Connection reliability by carrier (check-in frequency)
- **Carrier Costs** (if billing data available)
  - Total data usage per carrier
  - Cost per GB by carrier
  - Projected overage charges

**Detailed Tables:**
- Devices per carrier per company
- Service plans per carrier
- SIM inventory (total SIMs vs assigned SIMs)

**Database Implementation:**
```sql
-- Basic carrier distribution
SELECT
  CASE
    WHEN d.is_dual_sim = 1 THEN 'Dual Carrier'
    WHEN d.att_sim_number IS NOT NULL THEN 'AT&T'
    WHEN d.verizon_sim_number IS NOT NULL THEN 'Verizon'
    WHEN d.tmo_sim_number IS NOT NULL THEN 'T-Mobile'
    ELSE 'Unknown'
  END as carrier,
  COUNT(*) as device_count,
  AVG(dsl.signal_strength) as avg_signal_strength
FROM devices d
LEFT JOIN device_status_logs dsl ON d.id = dsl.device_id AND dsl.from_check_in = 1
WHERE d.device_status_id = 1 -- Active
GROUP BY carrier
```

**New Widget Ideas:**
- **SIM Inventory Dashboard**
  - Total SIMs purchased
  - SIMs assigned to devices
  - Available SIM pool
  - SIMs pending activation
- **Carrier Health Score**
  - Composite score based on signal strength, uptime, check-in reliability

---

### 5. Check-ins Status ✔️

#### Check-in Analytics Dashboard
**Current:** Just shows unique check-ins in last 2 hours
**Enhanced Features:**

**Real-Time Metrics:**
- **Check-ins in Last Hour** - Live counter with auto-refresh
- **Check-ins in Last 24 Hours** - With hourly breakdown
- **Expected vs Actual Check-ins** - Based on device count and check-in frequency
- **Check-in Success Rate** - Percentage of devices checking in on schedule

**Visual Displays:**
- **Timeline Chart** - Check-ins per hour for last 24 hours (line chart)
- **Heatmap** - Check-in activity by hour and day of week
- **Distribution Chart** - How often devices check in (every 5 min, 15 min, 1 hour, etc.)

**Problem Detection:**
- **Devices with Failed Check-ins** - Haven't checked in when expected
- **Irregular Check-in Patterns** - Devices with erratic timing
- **Check-in Frequency Changes** - Devices that used to check in frequently but stopped
- **First Check-in Today** - Devices that just came online

**Advanced Tables:**
- Last check-in time per device (sortable)
- Check-in reliability score per device (% on-time)
- Devices grouped by last check-in:
  - Within 1 hour
  - 1-6 hours ago
  - 6-24 hours ago
  - 1-7 days ago
  - 7+ days ago (critical)

**Integration with Alerts:**
- Auto-create alerts for devices that miss 3+ consecutive check-ins
- Show check-in related alerts on dashboard

**Database Tables:**
- `device_checkins` (created timestamp, device_id)
- `devices` (last_ping_time)
- Group by time ranges using MySQL date functions

**Database Implementation:**
```sql
-- Check-in activity by hour (last 24 hours)
SELECT
  DATE_FORMAT(created, '%Y-%m-%d %H:00:00') as hour,
  COUNT(DISTINCT device_id) as unique_devices,
  COUNT(*) as total_checkins
FROM device_checkins
WHERE created >= DATE_SUB(NOW(), INTERVAL 24 HOUR)
GROUP BY hour
ORDER BY hour

-- Devices by last check-in age
SELECT
  CASE
    WHEN last_ping_time >= DATE_SUB(NOW(), INTERVAL 1 HOUR) THEN 'Within 1 hour'
    WHEN last_ping_time >= DATE_SUB(NOW(), INTERVAL 6 HOUR) THEN '1-6 hours'
    WHEN last_ping_time >= DATE_SUB(NOW(), INTERVAL 24 HOUR) THEN '6-24 hours'
    WHEN last_ping_time >= DATE_SUB(NOW(), INTERVAL 7 DAY) THEN '1-7 days'
    WHEN last_ping_time IS NOT NULL THEN '7+ days (CRITICAL)'
    ELSE 'Never'
  END as check_in_age,
  COUNT(*) as device_count
FROM devices
WHERE device_status_id = 1 -- Active only
GROUP BY check_in_age
```

---

### 6. Device Locations Heat Map 🗺️🔥

#### Enhanced Location Visualization
**Current:** Google Maps with colored markers
**Enhanced Features:**

**Heat Map Implementation:**
- **Density Heat Map** - Show device concentration geographically
  - Red/hot = high concentration
  - Blue/cold = low concentration
- **Status Heat Map** - Color-code by connection status
  - Green = all devices online
  - Yellow = some devices offline
  - Red = most devices offline
- **Alert Heat Map** - Show regions with high alert frequency

**Google Maps Enhancements:**
- **Clustering** - Group nearby devices into clusters with device count
- **Custom Markers** - Different icons for device types/manufacturers
- **Info Windows** - Rich tooltips showing:
  - Device serial number
  - Connection status
  - Last check-in time
  - Signal strength
  - Active alerts
- **Filtering** - Toggle layers (by status, manufacturer, carrier, alerts)

**Geographic Analytics:**
- **Devices by State/Country** - Bar chart showing distribution
- **Devices by City** - Top 20 cities with most devices
- **Geographic Comparison** - Performance metrics by region
  - Average uptime by state
  - Average signal strength by region
  - Alert frequency by location

**New Visualizations:**
- **US Map Choropleth** - Color-coded states by device count
- **Regional Performance Dashboard** - Multi-metric view by region
- **Location-Based Alerts** - Map showing recent alert locations

**Technical Implementation:**
- Use **Google Maps JavaScript API** with Heat Map Layer
- Add **MarkerClusterer** library for clustering
- Implement **real-time updates** (WebSocket or polling)
- Add **drawing tools** to select regions for filtering

**Database Enhancement:**
- Geocode devices missing lat/long (use address fields)
- Add `region` field for grouping (Northwest, Southwest, etc.)
- Create materialized view for faster location queries

**Code Example:**
```javascript
// Heat map layer
var heatmapData = [
  {location: new google.maps.LatLng(37.782, -122.447), weight: 5},
  // ... more points
];
var heatmap = new google.maps.visualization.HeatmapLayer({
  data: heatmapData,
  radius: 20,
  opacity: 0.6
});
heatmap.setMap(map);

// Marker clustering
var markerCluster = new MarkerClusterer(map, markers, {
  imagePath: 'https://developers.google.com/maps/documentation/javascript/examples/markerclusterer/m',
  maxZoom: 15
});
```

---

### 7. Device Uptimes ⏱️

#### Uptime Analytics Dashboard
**Current Issues:**
- Average uptime calculation disabled (returns "N/A")
- Only showing InHand devices
- Limited to 7-day and 30-day buckets

**Enhanced Features:**

**Primary Metrics:**
- **Average Uptime (All Devices)** - Fix the calculation
  - Overall average
  - By device manufacturer
  - By device model
  - By carrier
  - By company
- **Uptime Distribution** - Histogram
  - How many devices have 0-1 day, 1-7 days, 7-30 days, 30+ days uptime
- **Uptime Percentage** - % of time devices are online
  - Last 24 hours, 7 days, 30 days, 90 days
- **Target vs Actual** - Compare against SLA targets (e.g., 99.9% uptime)

**Visual Displays:**
- **Uptime Trend Chart** - Line chart showing average uptime over time
- **Uptime by Device** - Bar chart showing top/bottom performers
- **Uptime Heatmap** - Calendar view showing daily uptime percentages
- **SLA Compliance** - Gauge charts showing % meeting uptime SLAs

**Problem Detection:**
- **Devices with Low Uptime** - Below threshold (e.g., < 95%)
- **Uptime Degradation** - Devices with decreasing uptime trends
- **Frequent Restarts** - Devices restarting more than X times per day
- **Never Online** - Devices that have never connected since activation

**Detailed Tables:**
- Per-device uptime report
  - Device serial number
  - Current uptime
  - Average uptime (30 days)
  - Max uptime achieved
  - Restart count
  - Last restart date
- Company uptime summary
  - Company name
  - Average device uptime
  - % devices meeting SLA
  - Worst performing device

**New Calculations:**
- **MTBF** (Mean Time Between Failures) - Average time device stays online
- **MTTR** (Mean Time To Repair) - Average time device stays offline
- **Availability** - (Total Time - Downtime) / Total Time × 100%

**Database Implementation:**
```sql
-- Fix average uptime calculation
SELECT
  d.id, d.serial_number,
  AVG(dsl.uptime) as avg_uptime,
  MAX(dsl.uptime) as max_uptime,
  COUNT(CASE WHEN dsl.uptime = 0 THEN 1 END) as restart_count
FROM devices d
LEFT JOIN device_status_logs dsl ON d.id = dsl.device_id
WHERE dsl.created >= DATE_SUB(NOW(), INTERVAL 30 DAY)
  AND dsl.from_check_in = 1
  AND d.device_manufacturer_id IS NOT NULL
GROUP BY d.id, d.serial_number

-- Uptime distribution buckets
SELECT
  CASE
    WHEN avg_uptime < 86400 THEN '0-1 day'
    WHEN avg_uptime < 604800 THEN '1-7 days'
    WHEN avg_uptime < 2592000 THEN '7-30 days'
    ELSE '30+ days'
  END as uptime_bucket,
  COUNT(*) as device_count
FROM (
  SELECT device_id, AVG(uptime) as avg_uptime
  FROM device_status_logs
  WHERE from_check_in = 1 AND created >= DATE_SUB(NOW(), INTERVAL 30 DAY)
  GROUP BY device_id
) as uptime_data
GROUP BY uptime_bucket
```

**Widget Priority:**
1. Fix existing uptime calculation (HIGH)
2. Add uptime distribution chart (HIGH)
3. Add uptime trend over time (MEDIUM)
4. Add SLA compliance metrics (MEDIUM)
5. Add MTBF/MTTR calculations (LOW)

---

### 8. Notifications by Date 📧📅

#### Notification Analytics Dashboard
**Current:** Just shows last 7 days in a table
**Enhanced Features:**

**Time-Series Analysis:**
- **Notifications Per Day** - Line chart for last 30 days
- **Notifications Per Hour** - For high-frequency monitoring
- **Day of Week Pattern** - Bar chart showing busiest days
- **Month-over-Month Comparison** - Trend analysis

**Notification Breakdown:**
- **By Type** - Pie chart
  - Company Notifications
  - Global Notifications
  - System Notifications
- **By Notification Name** - Which notifications fire most often
- **By Company** - Which customers receive most notifications
- **By Device** - Which devices trigger most notifications

**Delivery Metrics:**
- **Sent vs Failed** - Success rate
- **Delivery Methods** - Email, SMS, Dashboard
- **Response Time** - Time from event to notification sent
- **Acknowledgement Rate** - How many notifications are read/acknowledged

**Visual Displays:**
- **Notification Timeline** - Scrollable timeline with filterable events
- **Notification Heatmap** - Intensity by hour and day
- **Notification Funnel** - Created → Queued → Sent → Delivered → Read

**Filters:**
- Date range picker (custom ranges)
- Notification type
- Company
- Notification name
- Delivery status (sent, failed, pending)

**Top Insights:**
- **Most Active Notification** - Which notification fires most
- **Peak Notification Time** - When most notifications sent
- **Companies with Most Notifications** - Top 10 list
- **Notification Trends** - Increasing or decreasing

**Database Tables:**
- `notification_logs` (created, subject, status)
- `company_notification_logs` (created, company_id, notification_id)
- Join with `companies` and `devices` for context

**Database Implementation:**
```sql
-- Notifications per day (last 30 days)
SELECT
  DATE(created) as notification_date,
  COUNT(*) as notification_count,
  'Global' as type
FROM notification_logs
WHERE created >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY notification_date

UNION ALL

SELECT
  DATE(created) as notification_date,
  COUNT(*) as notification_count,
  'Company' as type
FROM company_notification_logs
WHERE created >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY notification_date
ORDER BY notification_date

-- Top notification types
SELECT
  notification_name,
  COUNT(*) as count,
  DATE_FORMAT(MAX(created), '%Y-%m-%d %H:%i:%s') as last_sent
FROM (
  SELECT subject as notification_name, created FROM notification_logs
  UNION ALL
  SELECT notification_name, created FROM company_notification_logs
) as all_notifications
WHERE created >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY notification_name
ORDER BY count DESC
LIMIT 20
```

---

### 9. Alerts Dashboard 🚨

#### Alert Analytics & Management
**Current:** Shows last 7 days of alerts in table
**Enhanced Features:**

**Real-Time Metrics:**
- **Active Alerts** - Count of unresolved alerts (big badge)
- **New Alerts Today** - Count with comparison to yesterday
- **Alert Trend** - Increasing, stable, or decreasing
- **Critical Alerts** - High priority alerts needing immediate attention

**Alert Type Analysis:**
- **Alert Type Distribution** - Pie chart showing most common alerts
  - Examples: Connection Lost, Low Signal, High Data Usage, Device Offline, etc.
- **Alert Frequency by Type** - Bar chart showing alert counts per type
- **Alert Type Trends** - Line chart showing alert types over time

**Device Alert Analysis:**
- **Devices with Most Alerts** - Top 10 problematic devices
- **Alert-Free Devices** - Devices with no alerts (good performers)
- **Repeat Offenders** - Devices with recurring same alert

**Company Alert Analysis:**
- **Companies with Most Alerts** - Which customers have issues
- **Alerts per Company** - Average and distribution
- **Company Alert Trends** - Improving or worsening

**Time Analysis:**
- **Alerts by Hour** - Identify problem time periods
- **Alerts by Day of Week** - Pattern detection
- **Alert Duration** - How long alerts stay active
- **Time to Resolution** - Average time to resolve alerts

**Visual Displays:**
- **Alert Timeline** - Chronological view with color-coding by severity
- **Alert Heatmap** - By device/time showing alert intensity
- **Alert Flow Diagram** - Sankey diagram showing alert → device → company
- **Geographic Alert Map** - Where alerts are occurring

**Alert Management Features:**
- **Acknowledge Alerts** - Mark as seen/being handled
- **Assign Alerts** - To team members for resolution
- **Alert Notes** - Add comments/resolution steps
- **Alert Status** - New, Acknowledged, In Progress, Resolved
- **Auto-Resolve** - Alerts that resolve themselves (e.g., device back online)

**Advanced Insights:**
- **Alert Patterns** - ML-based prediction of which devices will alert
- **Cascade Detection** - Multiple alerts from same root cause
- **Alert Fatigue Score** - Are we alerting too much?
- **False Positive Rate** - Alerts that auto-resolve quickly

**Database Tables:**
- `device_alerts` (device_id, device_alert_type_id, created)
- `device_alert_types` (title, description, severity)
- Join with `devices` and `companies`

**Database Implementation:**
```sql
-- Alert type distribution (last 30 days)
SELECT
  dat.title as alert_type,
  COUNT(da.id) as alert_count,
  COUNT(DISTINCT da.device_id) as affected_devices,
  COUNT(DISTINCT d.company_id) as affected_companies
FROM device_alerts da
JOIN device_alert_types dat ON da.device_alert_type_id = dat.id
JOIN devices d ON da.device_id = d.id
WHERE da.created >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY dat.id, dat.title
ORDER BY alert_count DESC

-- Devices with most alerts
SELECT
  d.id, d.serial_number, c.title as company_name,
  COUNT(da.id) as alert_count,
  GROUP_CONCAT(DISTINCT dat.title SEPARATOR ', ') as alert_types
FROM device_alerts da
JOIN devices d ON da.device_id = d.id
JOIN companies c ON d.company_id = c.id
JOIN device_alert_types dat ON da.device_alert_type_id = dat.id
WHERE da.created >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY d.id, d.serial_number, c.title
ORDER BY alert_count DESC
LIMIT 10

-- Alerts by hour of day (pattern detection)
SELECT
  HOUR(created) as hour,
  COUNT(*) as alert_count
FROM device_alerts
WHERE created >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY hour
ORDER BY hour
```

---

## Additional Critical Metrics for Admins

### 10. Billing & Revenue Dashboard 💰

**Why Admins Need This:**
- Financial health visibility
- Revenue forecasting
- Payment issue detection
- Billing cycle progress tracking

**Key Metrics:**
- **Monthly Recurring Revenue (MRR)** - Total predictable revenue
- **Annual Recurring Revenue (ARR)** - MRR × 12
- **Revenue by Company** - Top revenue-generating customers
- **Revenue by Service Plan** - Which plans are most profitable
- **Billing Cycle Status** - Current cycle progress (% complete)
- **Outstanding Invoices** - Total amount owed
  - Overdue invoices (30, 60, 90+ days)
  - By company
- **Payment Success Rate** - % successful payments
- **Failed Payments** - Count and total amount
- **Revenue Trend** - Month-over-month growth
- **Average Revenue Per User (ARPU)** - Total revenue / customer count
- **Customer Lifetime Value (CLV)** - Predicted long-term revenue

**Visual Displays:**
- **Revenue Trend Chart** - Line chart showing last 12 months
- **Revenue Breakdown** - Stacked bar chart by service plan
- **Invoice Status** - Pie chart (Paid, Pending, Overdue)
- **Payment Method Distribution** - ACH, Credit Card, etc.

**Database Tables:**
- `billing_cycles`
- `invoices` (total, status, due_date)
- `invoice_details`
- `company_payment_methods`
- `company_device_usages`

---

### 11. Data Usage Analytics 📊

**Why Admins Need This:**
- Identify overuse and potential overage charges
- Plan capacity and carrier contracts
- Detect anomalies (potential device issues)
- Optimize service plans

**Key Metrics:**
- **Total Data Usage** - Sum across all devices this month
- **Average Data Usage per Device** - By model, manufacturer, service plan
- **Data Usage Trend** - Last 12 months
- **Projected Usage** - Based on current cycle
- **Overage Risk** - Devices approaching/exceeding limits
- **Underutilization** - Devices using much less than plan allows
- **Data Usage by Carrier** - AT&T, Verizon, T-Mobile comparison
- **Top Data Consumers** - Devices using most data
- **Zero Usage Devices** - Devices not transmitting data

**Visual Displays:**
- **Usage vs Plan Allocation** - Gauge charts showing % used
- **Usage Trend** - Line chart over time
- **Usage Distribution** - Histogram showing device usage ranges
- **Carrier Comparison** - Bar chart comparing carrier usage

**Advanced Analytics:**
- **Anomaly Detection** - Devices with unusual usage spikes
- **Forecast Accuracy** - Predicted vs actual usage
- **Cost Per GB** - By carrier and service plan
- **Usage Efficiency** - Data usage relative to device uptime

**Database Tables:**
- `company_device_usages` (current_data_usage)
- `provider_data_usages` (data_usage, provider)
- `service_plans`
- `historical_service_plans`

---

### 12. System Health & Performance 🏥

**Why Admins Need This:**
- Proactive issue detection
- System reliability monitoring
- Identify infrastructure problems
- Plan scaling and maintenance

**Key Metrics:**
- **System Uptime** - Overall system availability
- **API Response Times** - Average latency
- **Queue Health** - Queue depths and processing times
  - process-checkins queue
  - process-device-alarms queue
  - Other worker queues
- **Database Performance** - Query times, slow queries
- **UDP Server Health** (Node.js check-ins service)
  - Packets received per minute
  - Packet processing time
  - Queue backlog
  - Error rate
- **AWS SQS Metrics**
  - Messages sent/received
  - Message age
  - Dead letter queue counts
- **CloudWatch Metrics** - From check-ins service
- **Redis Status** - Queue backend health

**Visual Displays:**
- **System Status Dashboard** - Green/yellow/red indicators
- **Response Time Chart** - Line chart showing API latency
- **Queue Depth Chart** - Monitor queue backlogs
- **Error Rate Chart** - System errors over time

**Alerts:**
- API response time > threshold
- Queue depth > threshold
- High error rate detected
- UDP server not responding

**Database/System Monitoring:**
- Track slow queries from `mysql.slow_log`
- Monitor Redis memory usage
- Track CakePHP Queue job failures

---

### 13. User Activity & Audit Log 📋

**Why Admins Need This:**
- Security monitoring
- Compliance requirements
- Troubleshooting user issues
- Usage analytics

**Key Metrics:**
- **Active Users** - Logged in today/this week/this month
- **User Login Frequency** - Average logins per user
- **Most Active Users** - By action count
- **Failed Login Attempts** - Security monitoring
- **Recent Admin Actions** - Audit trail
  - Device activations/deactivations
  - Configuration changes
  - User additions/deletions
  - Permission changes
- **User Sessions** - Currently logged in users
- **API Usage** - If API access is available

**Visual Displays:**
- **User Activity Heatmap** - By hour and day
- **Action Type Distribution** - Pie chart of action types
- **Login Timeline** - Successful vs failed logins

**Database Tables:**
- `o_logs` (Orases audit log table)
- `users` table
- Session storage (Redis/database)

---

### 14. Device Inventory & Lifecycle 📦

**Why Admins Need This:**
- Inventory management
- Warranty tracking
- Procurement planning
- Device age analysis

**Key Metrics:**
- **Total Device Inventory** - All devices regardless of status
- **Available vs Deployed** - Devices in stock vs assigned to customers
- **Devices by Manufacturer** - InHand, Systech, etc.
- **Devices by Model** - Distribution of models
- **Device Age Distribution** - How old are devices
  - 0-6 months, 6-12 months, 1-2 years, 2+ years
- **Warranty Status**
  - Under warranty
  - Warranty expiring soon (< 90 days)
  - Out of warranty
- **Device Lifecycle Stage**
  - New (< 30 days)
  - Active (30 days - 2 years)
  - Aging (2-4 years)
  - End of Life (4+ years)
- **Replacement Schedule** - Devices due for replacement

**Visual Displays:**
- **Inventory Dashboard** - Cards showing counts by category
- **Age Distribution** - Histogram
- **Manufacturer/Model Matrix** - Heatmap showing distribution

**Procurement Planning:**
- **Projected Inventory Needs** - Based on growth trends
- **RMA/Failure Rate** - Devices needing replacement
- **Device Turnover Rate** - How fast inventory deploys

**Database Tables:**
- `devices` (created, warranty_start_date, warranty_end_date, device_manufacturer_id, device_model_id)
- `device_manufacturers`
- `device_models`

---

### 15. Service Plan Analytics 📋

**Why Admins Need This:**
- Plan optimization
- Pricing strategy
- Customer segmentation
- Revenue optimization

**Key Metrics:**
- **Devices per Service Plan** - Distribution
- **Revenue per Service Plan** - Which plans generate most revenue
- **Average Data Usage per Plan** - Actual vs allocated
- **Plan Utilization** - % of plan limits being used
- **Plan Changes** - Upgrades vs downgrades
- **Most Popular Plans** - By device count
- **Plan Profitability** - Revenue vs cost (if cost data available)

**Visual Displays:**
- **Plan Distribution** - Pie chart
- **Plan Comparison** - Side-by-side comparison table
- **Plan Trends** - Line chart showing plan adoption over time

**Optimization Insights:**
- **Underutilized Plans** - Customers paying for more than they need
- **Overutilized Plans** - Customers exceeding limits (upsell opportunity)
- **Plan Recommendations** - Suggest better plans for customers

**Database Tables:**
- `service_plans`
- `historical_service_plans`
- `company_device_usages` (historical_service_plan_id)
- `company_service_plans` (custom pricing)

---

### 16. Carrier Performance & Reliability 📡

**Why Admins Need This:**
- Carrier contract negotiations
- Issue escalation to carriers
- Identify carrier-specific problems
- Optimize carrier selection

**Key Metrics:**
- **Uptime by Carrier** - AT&T vs Verizon vs T-Mobile
- **Signal Strength by Carrier** - Average RSSI
- **Data Speed by Carrier** - If measurable
- **Latency by Carrier** - Check-in response times
- **Alert Frequency by Carrier** - Which carrier has most issues
- **Dual-SIM Failover Events** - How often backup carrier used
- **Geographic Coverage** - Carrier performance by region

**Visual Displays:**
- **Carrier Comparison Dashboard** - Multi-metric comparison
- **Signal Strength Map** - Heat map by carrier
- **Carrier Reliability Score** - Composite metric

**Cost Analysis:**
- **Cost per GB by Carrier**
- **Total Spend by Carrier**
- **ROI by Carrier** - Performance vs cost

**Database Tables:**
- `devices` (att_sim_number, verizon_sim_number, tmo_sim_number, is_dual_sim)
- `device_status_logs` (signal_strength, sinr)
- `provider_data_usages` (provider, data_usage)

---

### 17. Multi-Tenant Analytics 🏢

**Why Admins Need This:**
- Understand tenant usage patterns
- Identify expansion opportunities
- Resource allocation
- White-label performance

**Key Metrics:**
- **Tenants/Sites** - Total count (via orases/sites)
- **Devices per Tenant** - Distribution
- **Revenue per Tenant** - If applicable
- **Active Users per Tenant**
- **Tenant Growth Rate** - New tenants over time
- **Tenant Churn** - Lost tenants

**Parent-Child Relationships:**
- **Distributor Hierarchy** - Parent companies with child accounts
- **Rollup Metrics** - Aggregate child account metrics to parent
- **Distributor Performance** - Credit, payouts, device counts

**Database Tables:**
- `companies` (o_site_id, parent_company_id)
- `o_sites` (from Orases Sites package)

---

### 18. Security & Compliance Dashboard 🔒

**Why Admins Need This:**
- Security posture visibility
- Compliance reporting (GDPR, SOC 2, etc.)
- Threat detection
- Access control monitoring

**Key Metrics:**
- **Failed Login Attempts** - Potential brute force attacks
- **Suspicious Activity** - Unusual access patterns
- **User Permission Changes** - Audit trail
- **Data Access Logs** - Who accessed what
- **Encryption Status** - OneWayCryptedType/TwoWayCryptedType usage
- **Vulnerability Alerts** - If security scanning in place
- **Session Management** - Active sessions, session duration

**Compliance Reports:**
- **Data Retention Compliance** - Are old records archived properly?
- **Access Control Compliance** - Proper RBAC implementation
- **Audit Log Completeness** - All actions logged

**Database Tables:**
- `o_logs` (audit logging)
- `users`, `user_roles`
- Session data

---

## Advanced Analytics & Visualizations

### 19. Predictive Analytics 🔮

**Machine Learning Opportunities:**
- **Predictive Maintenance** - Identify devices likely to fail
- **Data Usage Forecasting** - Predict overage risks
- **Alert Prediction** - Which devices will alert next
- **Churn Prediction** - Which customers might leave
- **Revenue Forecasting** - Predict future revenue

**Implementation:**
- Use historical data to train models
- Integration with Python/R for ML
- Show confidence intervals in predictions

---

### 20. Custom Dashboard Builder 🎨

**Allow Admins to Customize:**
- **Widget Selection** - Choose which widgets to display
- **Layout Customization** - Drag-and-drop positioning
- **Date Range Presets** - Last 24 hours, 7 days, 30 days, custom
- **Filtering** - Global filters (company, manufacturer, carrier)
- **Saved Views** - Save custom dashboard configurations
- **Role-Based Dashboards** - Different dashboards for different roles
  - Super Admin Dashboard
  - Distributor Dashboard
  - Customer Dashboard
  - Technical Support Dashboard

---

### 21. Comparative Analytics 📊

**Benchmarking:**
- **Company vs Company** - Compare two companies side-by-side
- **Time Period Comparison** - This month vs last month, YoY
- **Plan vs Plan** - Service plan performance comparison
- **Carrier vs Carrier** - Side-by-side carrier metrics
- **Industry Benchmarks** - Compare to industry averages (if data available)

**Visual Displays:**
- **Comparison Tables** - Side-by-side columns
- **Overlaid Charts** - Multiple lines on same chart
- **Difference Charts** - Show delta between periods

---

### 22. Real-Time Dashboard 🔴

**Live Data Updates:**
- **Auto-Refresh** - Update metrics without page reload
- **WebSocket Integration** - Real-time push updates
- **Live Counters** - Check-ins, alerts animating in real-time
- **Activity Feed** - Stream of recent events
  - Device check-ins
  - Alerts triggered
  - Status changes
  - User logins

**Implementation:**
- WebSocket or Server-Sent Events (SSE)
- Redis Pub/Sub for event broadcasting
- Background job to push updates

---

### 23. Export & Reporting 📄

**Export Capabilities:**
- **Dashboard Export** - PDF snapshot of dashboard
- **Excel Export** - All widget data to Excel
- **CSV Export** - Raw data for analysis
- **Scheduled Reports** - Email dashboard summary daily/weekly
- **Custom Report Builder** - Create ad-hoc reports

**Report Templates:**
- Executive Summary (high-level KPIs)
- Technical Operations Report (system health)
- Customer Report (per-company metrics)
- Financial Report (billing and revenue)
- Compliance Report (audit trails)

---

### 24. Mobile-Optimized Dashboard 📱

**Responsive Design:**
- **Mobile-First Widgets** - Key metrics optimized for small screens
- **Simplified Charts** - Mobile-friendly visualizations
- **Swipeable Widgets** - Navigate between widgets
- **Push Notifications** - Critical alerts to mobile

**Mobile App Considerations:**
- Progressive Web App (PWA) support
- Native app integration (if exists)

---

## Technical Implementation Recommendations

### Architecture & Performance

#### 1. Caching Strategy
**Problem:** Dashboard queries can be expensive
**Solution:**
- **Redis Caching** - Cache expensive queries for 5-15 minutes
- **Materialized Views** - Pre-calculate complex aggregations
- **Query Optimization** - Add indexes, optimize joins
- **Lazy Loading** - Load widgets as user scrolls

**Implementation:**
```php
// Example: Cache dashboard stats
$cacheKey = 'dashboard_device_stats_' . $companyId;
$stats = Cache::remember($cacheKey, 300, function() use ($companyId) {
    return $this->Devices->find('dashboardStats', [
        'company_id' => $companyId
    ])->first();
});
```

---

#### 2. Asynchronous Loading
**Problem:** Loading all widgets at once slows page load
**Solution:**
- **AJAX Loading** - Load widgets after page renders
- **Progressive Enhancement** - Show skeleton screens while loading
- **Prioritization** - Load critical widgets first

**Implementation:**
```javascript
// Load widget asynchronously
$(document).ready(function() {
    $('.widget').each(function() {
        var $widget = $(this);
        var url = $widget.data('ajax-url');
        $.get(url, function(html) {
            $widget.html(html);
        });
    });
});
```

---

#### 3. Database Optimization
**Indexes to Add:**
```sql
-- Improve check-in queries
CREATE INDEX idx_checkins_created ON device_checkins(created, device_id);
CREATE INDEX idx_checkins_device_created ON device_checkins(device_id, created);

-- Improve alert queries
CREATE INDEX idx_alerts_created ON device_alerts(created, device_id);
CREATE INDEX idx_alerts_type_created ON device_alerts(device_alert_type_id, created);

-- Improve status log queries
CREATE INDEX idx_status_logs_device_created ON device_status_logs(device_id, created, from_check_in);

-- Improve company queries
CREATE INDEX idx_companies_status ON companies(company_status_id);
CREATE INDEX idx_devices_company_status ON devices(company_id, device_status_id);
```

**Materialized Views:**
```sql
-- Pre-calculate device counts per company (refresh hourly)
CREATE TABLE dashboard_company_device_counts (
    company_id INT,
    total_devices INT,
    active_devices INT,
    offline_devices INT,
    last_updated DATETIME,
    PRIMARY KEY (company_id)
);

-- Refresh via cron job
-- bin/cake refresh_dashboard_cache
```

---

#### 4. Chart Library Recommendations
**Current:** Chart.js (doughnut charts only)
**Upgrade Options:**
- **Chart.js 4.x** - Continue with latest version (line, bar, area, radar charts)
- **ApexCharts** - Modern, interactive charts with great mobile support
- **Highcharts** - Enterprise-grade (requires license)
- **D3.js** - Maximum customization (steeper learning curve)
- **Google Charts** - Free, comprehensive, but less customizable

**Recommendation:** **ApexCharts** or **Chart.js 4.x**
- Both are free and open-source
- Excellent documentation
- Responsive and interactive
- Wide variety of chart types
- Easy to integrate with CakePHP

**Example ApexCharts Integration:**
```javascript
var options = {
    series: [{
        name: 'Check-ins',
        data: [30, 40, 35, 50, 49, 60, 70]
    }],
    chart: {
        type: 'line',
        height: 350
    },
    xaxis: {
        categories: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    }
};
var chart = new ApexCharts(document.querySelector("#chart"), options);
chart.render();
```

---

#### 5. Real-Time Updates
**WebSocket Implementation:**
```javascript
// Client-side (dashboard.js)
const ws = new WebSocket('ws://localhost:3000');

ws.onmessage = function(event) {
    const data = JSON.parse(event.data);

    if (data.type === 'device_checkin') {
        updateCheckinCounter(data.count);
    }

    if (data.type === 'alert') {
        showAlertNotification(data.alert);
    }
};

// Server-side (Node.js WebSocket server)
// Broadcast check-in events from UDP server
wss.clients.forEach(client => {
    client.send(JSON.stringify({
        type: 'device_checkin',
        count: checkinCount
    }));
});
```

**Alternative: Server-Sent Events (SSE)**
```php
// CakePHP controller
public function stream() {
    $this->autoRender = false;
    header('Content-Type: text/event-stream');
    header('Cache-Control: no-cache');

    while (true) {
        $data = $this->getDashboardUpdates();
        echo "data: " . json_encode($data) . "\n\n";
        flush();
        sleep(5);
    }
}
```

---

### User Experience

#### 6. Dashboard Layout Options
**Grid System:**
- Use CSS Grid or Bootstrap Grid
- Responsive breakpoints (mobile, tablet, desktop)
- Widget sizes: small (1/4 width), medium (1/2 width), large (full width)

**Layout Examples:**
```
Desktop Layout:
+----------------+----------------+
| Customer Count | Device Status  |
+----------------+----------------+
| Devices by Co  | Carrier Dist   |
+----------------+----------------+
| Check-ins Timeline (Full Width) |
+---------------------------------+
| Location Map (Full Width)       |
+---------------------------------+

Mobile Layout:
+--------------------+
| Customer Count     |
+--------------------+
| Device Status      |
+--------------------+
| Devices by Co      |
+--------------------+
```

---

#### 7. Filtering & Drill-Down
**Global Filters:**
- Company dropdown (multi-select)
- Date range picker
- Device manufacturer
- Carrier
- Status

**Drill-Down Flow:**
- Click widget → Detailed view
- Click metric → Filtered list
- Breadcrumb navigation to return

**Example:**
```
Dashboard > Device Status Widget > "Offline Devices" > Filtered Device List
```

---

#### 8. User Preferences
**Save Settings:**
- Preferred widgets
- Widget order
- Default date range
- Default filters
- Refresh interval

**Database Table:**
```sql
CREATE TABLE user_dashboard_preferences (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    widget_id VARCHAR(50) NOT NULL,
    position INT,
    visible BOOLEAN DEFAULT TRUE,
    settings JSON,
    created DATETIME,
    modified DATETIME,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

## Priority Roadmap

### Phase 1: High Priority (Immediate Impact) 🔴
**Timeline:** 2-4 weeks

1. **Fix Existing Issues**
   - Fix average uptime calculation (currently returns "N/A")
   - Optimize slow dashboard queries
   - Add database indexes for performance

2. **Enhance Existing Widgets**
   - Add carrier comparison bar chart
   - Add devices by company table with sorting
   - Enhance check-in widget with 24-hour timeline

3. **Critical New Widgets**
   - Customer overview widget (total, active, suspended)
   - Active alerts widget (count with link to details)
   - Billing dashboard (MRR, outstanding invoices)

**Impact:** Immediate visibility into customers, devices, and critical alerts

---

### Phase 2: Medium Priority (Core Features) 🟡
**Timeline:** 4-8 weeks

1. **Advanced Visualizations**
   - Heat map for device locations
   - Device uptime distribution chart
   - Notification timeline chart
   - Alert type distribution chart

2. **Time-Series Analysis**
   - Check-in trend over 30 days
   - Alert frequency trend
   - Data usage trend
   - Device growth trend

3. **Performance Optimization**
   - Implement Redis caching
   - Asynchronous widget loading
   - Create materialized views

**Impact:** Better trend analysis and improved performance

---

### Phase 3: Advanced Features 🟢
**Timeline:** 8-12 weeks

1. **Real-Time Dashboard**
   - WebSocket or SSE for live updates
   - Real-time check-in counter
   - Live alert feed

2. **Custom Dashboard Builder**
   - Drag-and-drop widget positioning
   - Show/hide widgets
   - Save custom layouts
   - Role-based default dashboards

3. **Advanced Analytics**
   - Comparative analytics (month-over-month)
   - Predictive maintenance insights
   - Geographic performance analysis

**Impact:** Highly customizable and actionable insights

---

### Phase 4: Enterprise Features 🟣
**Timeline:** 12+ weeks

1. **Machine Learning Integration**
   - Predictive device failure
   - Data usage forecasting
   - Alert prediction
   - Churn prediction

2. **Advanced Reporting**
   - Custom report builder
   - Scheduled email reports
   - PDF/Excel export
   - White-label reports for customers

3. **Mobile Optimization**
   - Mobile-responsive widgets
   - Progressive Web App (PWA)
   - Push notifications
   - Mobile-specific dashboard

**Impact:** Enterprise-grade analytics and mobile access

---

## Implementation Notes

### Technology Stack Recommendations

**Frontend:**
- **Chart Library:** ApexCharts or Chart.js 4.x
- **UI Framework:** Continue with Limitless theme (BackendTheme plugin)
- **JavaScript Framework:** Vanilla JS or jQuery (already in use)
- **Grid System:** CSS Grid or Bootstrap Grid

**Backend:**
- **Caching:** Redis (already in use for queues)
- **Optimization:** Database indexes, materialized views
- **Real-Time:** WebSocket (Node.js) or SSE (CakePHP)

**Database:**
- **Indexes:** Add indexes for dashboard queries
- **Materialized Views:** For expensive aggregations
- **Archive Strategy:** Continue with AWS Glacier archival

---

### Testing Strategy

**Performance Testing:**
- Load test with 10,000+ devices
- Measure query execution times
- Test cache effectiveness
- Monitor memory usage

**User Testing:**
- Get feedback from admin users
- A/B test different layouts
- Track widget usage analytics

**Browser Testing:**
- Chrome, Firefox, Safari, Edge
- Mobile browsers (iOS Safari, Chrome Mobile)
- Test on different screen sizes

---

### Documentation Requirements

**Admin Guide:**
- How to use each widget
- How to interpret metrics
- How to customize dashboard
- Troubleshooting guide

**Developer Guide:**
- How to add new widgets
- Database query optimization tips
- Caching strategy
- Widget development template

---

## Conclusion

This comprehensive plan provides a roadmap for transforming the WATM Admin Dashboard from a basic status view into a powerful analytics and management platform. By implementing these features in phases, you'll provide administrators with the visibility and insights needed to:

- **Manage customers** effectively
- **Monitor device health** proactively
- **Optimize carrier performance**
- **Track financial metrics**
- **Respond to alerts** quickly
- **Make data-driven decisions**

The key is to start with high-impact, quick wins (Phase 1) and progressively add more sophisticated features based on user feedback and business priorities.

**Next Steps:**
1. Review and prioritize features with stakeholders
2. Create detailed technical specifications for Phase 1
3. Set up development environment with necessary libraries
4. Begin implementation starting with performance fixes
5. Iterate based on user feedback

Good luck with your dashboard enhancement project! 🚀
