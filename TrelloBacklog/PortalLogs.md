# Portal Logs Documentation (admin/logging)

## Overview

The WATM portal exposes an **Audit Logging** system at the path `/admin/logging` that tracks user actions, system events, device operations, and API interactions across the entire platform. This logging system is implemented through the **Logging plugin** which extends the Orases `o_logs` package.

## Purpose

The logging system provides:
- **Audit Trail**: Complete history of who did what, when, and from where
- **Security Monitoring**: Track user actions including authentication, authorization changes, and sensitive operations
- **Device Tracking**: Monitor device lifecycle events (creation, modification, assignment, status changes)
- **API Error Tracking**: Log errors from carrier API integrations (Verizon, AT&T, T-Mobile)
- **Troubleshooting**: Track system events for debugging and support purposes
- **Compliance**: Maintain records for regulatory and business compliance requirements

## Log Entry Structure

Each log entry captures:

| Field | Description |
|-------|-------------|
| **Log Type** | Category of the log entry (see categories below) |
| **Message** | Descriptive message about the event (supports template interpolation) |
| **User** | User who performed the action (first name, last name) |
| **Company** | Associated company (multi-tenant context) |
| **Device** | Associated device (if applicable, shows serial number) |
| **IP Address** | Client IP address where action originated |
| **URL** | Request path where action occurred |
| **Created Date** | Timestamp in America/New_York timezone |
| **Plugin** | Source plugin that generated the log |

## Log Categories

The system uses predefined categories to classify log entries. Categories have an `is_customer_viewable` flag that controls whether customer role users can see those logs.

### User Management Logs
- `USER CREATION` - New user account created
- `USER DELETE` - User account deleted
- `USER PROFILE EDIT` - User profile information modified
- `PASSWORD CHANGES` - User password changed

### Device Management Logs
- `DEVICE CREATION` - New device added to system
- `DEVICE MODIFICATION` - Device configuration or details changed
- `DEVICE ASSIGNMENT` - Device assigned to/reassigned between companies
- `DEVICE ADMIN NOTES UPDATE` - Administrative notes added/modified on device
- `DEVICE PAGE VIEW` - Device detail page viewed (tracking sensitive access)

### Configuration Management Logs
- `CONFIGURATION CREATION` - New device configuration created
- `CONFIGURATION MODIFICATION` - Device configuration modified

### Service Plan Logs
- `SERVICE PLAN CREATION` - New service plan created
- `SERVICE PLAN MODIFICATION` - Service plan details changed

### Billing Logs
- `BILLING METHOD CHANGE` - Payment method changed
- `PAYMENT METHOD MODIFICATION` - Payment method details modified

### Company Management Logs
- `COMPANY PROFILE EDIT` - Company information modified
- `COMPANY NOTES UPDATE` - Company administrative notes updated

### API Integration Logs
- `VERIZON API DEVICE ERROR` - Error occurred during Verizon API operation
- `ATT API DEVICE ERROR` - Error occurred during AT&T API operation
- `TMOBILE API DEVICE ERROR` - Error occurred during T-Mobile API operation

### API Key Management Logs
- `API KEY CREATION` - New API key generated
- `API KEY MODIFICATION` - API key settings changed
- `API KEY DELETION` - API key deleted
- `API KEY ACTIVATED` - API key activated
- `API KEY REVOKED` - API key revoked
- `API KEY AWS SYNC` - API key synced to AWS API Gateway
- `API KEY AWS SYNC FAILURE` - Failed to sync API key to AWS
- `API USAGE PLAN DELETION` - API usage plan deleted

### Notification Logs
- `NOTIFICATION CREATION` - New notification created
- `NOTIFICATION EDITED` - Notification modified
- `NOTIFICATION APPROVED` - Notification approved
- `NOTIFICATION DENIED` - Notification denied

### Support Logs
- `RECEIVED SUPPORT CALL` - Support call logged in system

## Access Control

### Customer Users (Role: 'customer')
- **Filtered View**: Only see logs where `is_customer_viewable = true`
- **Company Scope**: Restricted to their own company and sub-companies
- Cannot see administrative actions, API errors, or sensitive operations

### Administrative Users
- **Full Access**: See all log categories
- **Company Scope**: Can filter by company, or view all if permissions allow
- See API errors, system events, and all user actions

## Filter Capabilities

Users can filter logs by:
1. **Log Type** - Select from dropdown of available categories (filtered by customer_viewable for customer users)
2. **Message** - Full-text search in log message
3. **User** - Search by user's first/last name (supports single or two-name search)
4. **Company** - Search by company name
5. **Device** - Search by device serial number or manufacturer serial number
6. **IP Address** - Search by client IP address

Results are paginated using simple pagination (next/previous) and ordered by created date descending (newest first).

## Technical Implementation

### Database Tables
- **o_logs** - Main log entries table (from Orases package)
  - Fields: id, message, ip_address, url, plugin, site_id, o_log_category_id, o_user_id, company_id, device_id, created, modified
- **o_log_categories** - Log category definitions
  - Fields: id, title, is_customer_viewable, created, modified

### Key Files
- **Controller**: `/watm/watm/plugins/Logging/src/Controller/Admin/LoggingController.php`
  - `index()` - Browse logs with filtering
  - `view($id)` - View individual log detail
- **Model**: `/watm/watm/plugins/Logging/src/Model/Table/OLogsTable.php`
  - Extends `Orases\Logs\Model\Table\LogsTable`
  - `logEvent()` method for creating log entries
  - Search behavior configured for filtering
- **View**: `/watm/watm/plugins/Logging/templates/Admin/Logging/index.php`
  - Filter form with all search fields
  - Table display with log entries
- **Trait**: `/watm/watm/plugins/Logging/src/Traits/LogEventTrait.php`
  - Provides `logEvent()` method for use in other controllers/models

### Usage Pattern

Throughout the codebase, logging is triggered using:

```php
$this->logEvent(
    'User {user.first_name} {user.last_name} created device {device.serial_number}',
    $identity,
    [
        'category' => 'DEVICE CREATION',
        'company_id' => $device->company_id,
        'device_id' => $device->id
    ]
);
```

The `logEvent()` method:
1. Interpolates template variables from the identity object
2. Captures request context (IP, URL, plugin)
3. Validates that the category exists
4. Creates and saves the log entity
5. Silently fails if identity is missing or category is invalid (no exception thrown)

## Data Retention

The log entries are stored indefinitely in the database. There is no automatic archival or purging configured for audit logs (unlike device check-ins and status logs which are archived after 3 months).

## Security Considerations

- Logs are **company-scoped** to prevent cross-tenant data leakage
- Customer users have **restricted visibility** (only customer-viewable categories)
- Logs capture **IP addresses** for security analysis
- **Sensitive operations** (password changes, payment methods, API keys) are logged
- Device page views are logged to track access to potentially sensitive device information
- API errors are logged to track integration issues but are not shown to customers

## Use Cases

1. **Audit & Compliance**: Track who modified what and when for regulatory requirements
2. **Security Investigation**: Identify unauthorized access attempts or suspicious activity patterns
3. **Troubleshooting**: Diagnose issues by reviewing user actions and API errors
4. **Customer Support**: Review customer's activity when investigating support tickets
5. **Device Lifecycle**: Track complete history of device changes and assignments
6. **API Monitoring**: Monitor carrier API integration health and error patterns
7. **User Activity Monitoring**: Track user login patterns and access to sensitive areas

## Limitations

- No automated log retention/archival policy
- No alerting based on log patterns
- No log aggregation or analytics dashboard
- Cannot export logs in bulk
- No log correlation between related events
- Limited search capabilities (no date range filter, no regex support)
- Customer users cannot see technical errors or administrative actions affecting them
