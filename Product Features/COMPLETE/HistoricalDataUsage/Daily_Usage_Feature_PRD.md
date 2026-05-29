# Daily Usage Feature - Product Requirements Document

## Overview
Add a new "Daily Usage" tab to the device management interface that displays historical daily data usage for the past 30 days, providing more granular visibility into device data consumption patterns.

## Feature Summary
- Add new "Daily Usage" tab under the statistics card on the Manage Device page
- Display 30-day historical data usage with one data point per day
- Rename existing "Data Usage" tab to "Monthly Usage"
- Restrict feature to specific service plan tiers

## Requirements

### Functional Requirements

#### 1. New Daily Usage Tab
- Add "Daily Usage" tab to the statistics card section on Manage Device page
- Display data usage for the last 30 days (30-day lookback period)
- Show one data point for each day in the range
- Data should be presented in a clear, readable format (table or chart)

#### 2. Service Plan Filtering
The Daily Usage tab should **only** be visible/accessible for devices on the following service plans:
- Tier 1
- Tier 2
- Tier 3
- Super Tier

Devices on other service plans should not see this tab.

#### 3. Tab Renaming
- Rename the current "Data Usage" tab to "Monthly Usage"
- Maintain all existing functionality of this tab

### Data Requirements
- Data source: Device check-in data from existing telemetry system
- Data granularity: Daily aggregation (one value per day)
- Date range: Last 30 days from current date
- Data metrics: Data usage (likely in MB/GB)

### UI/UX Requirements
- Tab should follow existing statistics card tab design patterns
- Display should be consistent with other usage reporting in the system
- Consider using charts/graphs for visual representation of daily trends
- Include date labels for each data point
- Show units of measurement clearly (MB, GB, etc.)

## Acceptance Criteria
- [ ] "Daily Usage" tab appears on Manage Device page for eligible service plans
- [ ] Tab displays exactly 30 days of historical data
- [ ] Each day has a distinct data point/value
- [ ] Tab is NOT visible for devices on non-eligible service plans
- [ ] "Data Usage" tab has been renamed to "Monthly Usage"
- [ ] "Monthly Usage" tab maintains all existing functionality
- [ ] Data displays accurately match device telemetry records
- [ ] UI is consistent with existing statistics card design

## Out of Scope
- Historical data beyond 30 days
- Export functionality for daily usage data (unless already exists for monthly)
- Custom date range selection
- Real-time/live data updates

## Questions/Clarifications Needed
- [ ] What is the exact data source for daily usage? (table/column names)
- [ ] How should missing days (no data) be displayed?
- [ ] Should data be displayed as a chart, table, or both?
- [ ] What timezone should be used for "daily" aggregation?
- [ ] Should totals/averages be displayed alongside daily values?
- [ ] Are there any performance requirements (e.g., max load time)?

## Original Request
From: Devon

> We would like to add "Daily Usage" tab under the statistics card on the manage device page, if possible. We would like it to be a 30-day look back, if possible. This would only apply to Tier 1, 2, 3, or Super Tier. We would also change the name of the current "data usage" tab to "Monthly Usage".
