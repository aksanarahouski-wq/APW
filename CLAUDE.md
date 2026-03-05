# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WATM (Wireless Access and Telemetry Management) is an IoT device management and billing platform consisting of two main components:

- **CakePHP 4.x Application** (`watm/watm/`) - Multi-tenant web application for device management, billing, and reporting
- **Node.js Check-ins Service** (`watm/checkins/`) - UDP server that receives device telemetry and forwards to AWS SQS queues

The system manages IoT devices from multiple manufacturers, tracks usage, generates billing cycles, processes payments via ACH/NACHA files, and provides white-label multi-tenant capabilities.

## Essential Commands

### CakePHP Application (run from `watm/watm/` directory)

```bash
# Dependencies
composer install

# Database
bin/cake migrations migrate -c migrations  # Run migrations (uses migrations user)
bin/cake handle_plugin_migrations          # Run all plugin migrations
bin/cake migrations seed --seed MasterSeed # Seed database

# Code Quality
composer cs-check    # Check PSR-12 compliance
composer cs-fix      # Auto-fix code style issues
composer stan        # Run PHPStan static analysis
composer test        # Run PHPUnit tests
vendor/bin/phpunit tests/TestCase/Specific/TestFile.php  # Run single test

# Queue Workers
bin/cake worker -p update-configs -c update-configs -r 295
bin/cake worker -p default-worker --max-runtime 300

# Development with DDEV
cd watm/
ddev start           # Start local environment
ddev ssh             # SSH into container
```

### Node.js Check-ins Service (run from `watm/checkins/` directory)

```bash
npm install
npm run dev          # Development with nodemon auto-restart
npm run server-setup # Install PM2 globally
npm run start        # Start with PM2 (production)
```

## Architecture

### Plugin-Based Architecture

The CakePHP application uses a plugin architecture where most business logic lives in plugins rather than the core app. Key plugins:

**Business Logic Plugins:**
- `Devices` - Device CRUD, configuration management, status tracking, alarms
- `Companies` - Multi-tenant company management, hierarchical relationships, power schedules
- `Billing` - Billing cycles, invoices, NACHA/ACH file generation, payment methods
- `Notifications` - Alert system for device events
- `Dashboard` - Reporting and analytics
- `Admin` - Administrative interface and user management
- `UserManagement` - User roles and RBAC permissions
- `SystemManagement` - System configuration and maintenance

**Integration Plugins:**
- `AdobeSign` - Digital signatures
- `TwilioApi` - SMS communications
- `GoogleApis` - Google services
- `WoocommerceApi` - E-commerce integration

**UI/Framework Plugins:**
- `BackendTheme` - Admin UI theme with CKEditor and custom plugins
- `ExcelWriter` - Excel export functionality

### Orases Custom Packages

The project depends on proprietary Orases packages from `packages.orases.com`:
- `orases/users` - Authentication foundation
- `orases/sites` - Multi-site/tenant management
- `orases/files` - File upload and S3 storage
- `orases/logs` - Audit logging (`o_logs` table)
- `orases/imports` - Data import utilities
- `orases/theme-limitless` - Limitless admin theme

### Device Check-ins Architecture

The Node.js service (`watm/checkins/index.js`) handles device telemetry:
1. Listens on UDP port 8002 for device check-ins and alarm messages
2. Batches messages (configurable, default 25 per batch)
3. Sends batches to AWS SQS queues (`process-checkins` and `process-device-alarms`)
4. CakePHP queue workers process messages from SQS
5. Sends CloudWatch metrics for monitoring (packet counts, batch sizes, message sizes)
6. Auto-detects EC2 instance metadata for hostname/instance ID
7. Supports LocalStack for local development

Uses EC2 instance roles for AWS authentication (no hardcoded credentials).

### Multi-Tenant Architecture

- Company-based data isolation with parent-child relationships
- White-label support via `orases/sites` package
- Role-Based Access Control (RBAC) using CakeDC/Auth
- Custom permission rules in `App\Auth\Rules\WatmRules`
- Permissions defined in `watm/config/permissions.php`

### Database Architecture

- MySQL 8.0 with UTF-8mb4 charset
- Two database users:
  - `watm` - Limited SELECT/INSERT/UPDATE/DELETE for application
  - `watm_migrations` - Full privileges for migrations
- Extensive migration history in `watm/config/Migrations/`
- Audit logging via `o_logs` table (Orases package)
- Database type encryption: `OneWayCryptedType` (hashed) and `TwoWayCryptedType` (reversible)

### Queue System

Uses CakePHP Queue plugin with Redis backend and AWS SQS for device data:
- `process-checkins` - Device check-in processing
- `process-device-alarms` - Device alarm processing
- `update-configs` - Device configuration updates
- `process-invoices` - Billing/invoice processing
- `send-notifications` - Notification sending
- `export-devices` - Device data export
- `default-worker` - General tasks including mailer

Workers run via cron jobs using `bin/cake worker` command.

## Development Environment

### DDEV Setup

The project uses DDEV for local development (configured in `watm/.ddev/`):
- PHP 8.1 with Apache-FPM
- MySQL 8.0
- Node.js 16
- Redis for queues
- Separate containers for check-ins service

### Database Setup

```sql
CREATE DATABASE watm_db DEFAULT CHARSET=utf8mb4 DEFAULT COLLATE=utf8mb4_general_ci;
CREATE USER watm@'localhost' IDENTIFIED BY 'watm_db';
GRANT SELECT, INSERT, DELETE, UPDATE ON watm_db.* TO watm@'localhost';
CREATE USER watm_migrations@'localhost' IDENTIFIED BY 'watm_db';
GRANT ALL PRIVILEGES ON watm_db.* TO watm_migrations@'localhost';
```

### Required Dependencies

- PHP 8.1+ with extensions: dom, json, zip
- Node.js v16.20.0 (for check-ins service)
- WKHTMLTOPDF (PDF generation)
- AWS CLI (data archival)
- MySQL 8.0
- Redis

### Configuration Files

**CakePHP:**
- `watm/config/app_local.php` - Local environment config (use `app_local.example.php` as template)
- `watm/config/sites.php` - Multi-site/white-label configuration
- `watm/config/permissions.php` - RBAC permission definitions
- `watm/config/theme_settings.php` - Admin theme settings

**Node.js:**
- `checkins/.env` - Environment variables (use `.env-example` as template)
  - `AWS_REGION`, `SQS_CHECKINS_QUEUE_URL`, `SQS_ALARMS_QUEUE_URL`
  - `USE_LOCAL_STACK` for local development
  - `ENVIRONMENT` for CloudWatch dimension tagging

## Key Business Logic

### Device Management Flow
1. Devices belong to companies (multi-tenant isolation)
2. Each device has manufacturer-specific configuration via `config_groups`
3. Devices check in via UDP, data flows: UDP → SQS → Queue Worker → Database
4. Status changes trigger alerts via Notifications plugin
5. Remote power cycling and configuration updates supported

### Billing System Flow
1. Monthly billing cycles created automatically
2. Device usage tracked (data usage, connection time)
3. Invoice generation aggregates usage by company and service plan
4. NACHA/ACH files generated for automated bank transfers
5. Distributor credit and payout system for resellers
6. PDF invoice generation using WKHTMLTOPDF

### Data Archival
Automated archival to AWS Glacier for:
- Device status logs (>3 months old)
- Device checkins (>3 months old)
- Provider data usage (>2 months old)

Configuration in `watm/config/ArchiveData/`

## Coding Standards

### CakePHP (PHP)
- PSR-12 coding standard
- 4-space indentation (not tabs)
- CamelCase for classes, camelCase for methods/variables
- Use CakePHP conventions: Table classes in `Model/Table/`, Entity classes in `Model/Entity/`
- Include JavaScript files at template top: `$this->Html->script('file.js', ['block' => 'extra-scripts'])`

### Node.js (JavaScript)
- JavaScript Standard Style
- 2-space indentation
- kebab-case for filenames
- camelCase for variables/functions, PascalCase for classes
- Prefer `const`/`let` over `var`
- Node.js v16.20.0 for compatibility

## Repository Structure

```
.
├── CLAUDE.md                    # This file
├── CONVENTIONS.md               # Detailed coding conventions
├── watm/                        # Main project directory
│   ├── .ddev/                   # DDEV local development config
│   ├── checkins/                # Node.js UDP server
│   │   ├── index.js            # Main UDP server implementation
│   │   ├── package.json
│   │   └── .env-example
│   ├── watm/                    # CakePHP application root
│   │   ├── bin/cake            # CakePHP CLI
│   │   ├── config/             # Configuration files
│   │   │   ├── app_local.example.php
│   │   │   ├── permissions.php
│   │   │   ├── sites.php
│   │   │   └── Migrations/     # Database migrations (390+ files)
│   │   ├── plugins/            # Application plugins (19 plugins)
│   │   │   ├── Devices/
│   │   │   ├── Companies/
│   │   │   ├── Billing/
│   │   │   └── [...]
│   │   ├── src/                # App-level source code
│   │   │   ├── Application.php
│   │   │   ├── Controller/
│   │   │   ├── Model/
│   │   │   ├── Command/        # CLI commands
│   │   │   ├── Job/            # Queue job classes
│   │   │   └── Auth/           # Custom auth rules
│   │   ├── templates/          # App-level view templates
│   │   ├── tests/              # PHPUnit tests
│   │   ├── webroot/            # Public web files
│   │   └── composer.json
│   └── CLAUDE.md               # Plugin-specific guidance
└── Configurations/             # External configuration files
    └── Files/                  # Various config files
```

### Plugin Structure (each plugin follows this pattern)
```
plugins/PluginName/
├── src/
│   ├── Controller/
│   │   └── Admin/              # Admin-prefixed controllers
│   ├── Model/
│   │   ├── Table/              # Database interaction
│   │   └── Entity/             # Data objects
│   ├── Job/                    # Queue jobs
│   └── Command/                # CLI commands
├── templates/
│   └── Admin/                  # Admin view templates
└── webroot/                    # Plugin assets (JS/CSS)
```

## Important Notes

- **Working Directory**: Always run CakePHP commands from `watm/watm/` directory
- **Indentation**: CakePHP uses 4 spaces, Node.js uses 2 spaces
- **Plugins**: Autoloaded via PSR-4 in `composer.json` - no manual loading required
- **Migrations**: Use migrations connection (separate user) for schema changes
- **Queue Workers**: Run as separate processes via cron, not web requests
- **Device Check-ins**: UDP server must run independently of web application

## Testing

### CakePHP Tests
```bash
cd watm/watm/
composer test                                              # Run all tests
vendor/bin/phpunit tests/TestCase/Specific/TestClass.php  # Run single test
```

Test files follow CakePHP conventions in `watm/tests/TestCase/`

### Node.js Tests
Currently no test suite configured for check-ins service.

## Additional Documentation

- `CONVENTIONS.md` - Detailed coding standards and conventions
- `BillingCyclesTemplateTestCases.md` - Comprehensive test cases for billing cycle templates
- See `watm/CLAUDE.md` for additional plugin-specific details
