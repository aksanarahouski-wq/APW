# WATM User Roles and Permissions System Documentation

This folder contains comprehensive documentation of the WATM (Wireless Access and Telemetry Management) platform's user roles and permissions system.

## Documentation Overview

### Core Documentation

1. **[01_Architecture_Overview.md](01_Architecture_Overview.md)**
   High-level architecture of the authentication and authorization system, including middleware stack, plugin structure, and system flow.

2. **[02_Permission_Configuration.md](02_Permission_Configuration.md)**
   Complete reference for the permissions configuration file (`config/permissions.php`), including all permission rules and their purposes.

3. **[03_Authorization_Rules.md](03_Authorization_Rules.md)**
   Detailed documentation of all custom authorization rule methods in `WatmRules` and related rule classes.

4. **[04_Role_Definitions.md](04_Role_Definitions.md)**
   User role types, role hierarchy, account types, and role management.

5. **[05_Database_Schema.md](05_Database_Schema.md)**
   Database tables, relationships, and data structures related to users, roles, and permissions.

6. **[06_Multi_Tenant_Architecture.md](06_Multi_Tenant_Architecture.md)**
   How company-based data isolation works, including parent-child company relationships and resource access patterns.

7. **[07_Implementation_Guide.md](07_Implementation_Guide.md)**
   Practical guide for developers: how to check permissions in controllers, templates, and when adding new features.

8. **[08_Quick_Reference.md](08_Quick_Reference.md)**
   Quick reference cheat sheet for common permission checks and authorization patterns.

### Additional Resources

9. **[09_Code_Examples.md](09_Code_Examples.md)**
   Real-world code examples from the codebase demonstrating permission usage.

10. **[10_Troubleshooting.md](10_Troubleshooting.md)**
    Common permission issues and how to resolve them.

## Key Concepts

### Role Types
- **Admin Roles** (`role_type = 'admin'`) - System administrators with full platform access
- **Customer Roles** (`role_type = 'customer'`) - Company-level users with tenant-specific access

### Account Types
Companies are categorized by account type, which determines capabilities:
- **Distributor** - Can upcharge, can create sub-companies
- **Customer** - Cannot upcharge, can create sub-companies
- **Subcustomer** - Cannot upcharge, cannot create sub-companies

### Permission Layers
1. **Configuration-based** - Static permissions in `config/permissions.php`
2. **Rule-based** - Dynamic permissions evaluated by custom rule methods
3. **Multi-tenant** - Company-based data isolation enforced automatically
4. **Resource-level** - Per-record authorization checks

## Quick Start

### For Developers

**Check if user can perform an action:**
```php
// In a controller
use CakeDC\Auth\Traits\IsAuthorizedTrait;

if ($this->isAuthorized([
    'prefix' => 'Admin',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => 'edit',
])) {
    // User is authorized
}
```

**In a template:**
```php
<?php if ($this->IsAuthorized->isAuthorized([
    'prefix' => 'Admin',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => 'delete'
])): ?>
    <!-- Show delete button -->
<?php endif; ?>
```

### For System Administrators

**User Role Assignment:**
- Roles are stored in `o_users.additional_data['role_ids']` as JSON
- System admin roles: Use role with `role_type = 'admin'`
- Customer admin roles: Use roles with `role_type = 'customer'`

**Company Account Types:**
- Set via `companies.account_type_id` foreign key
- Determines `can_upcharge` and `can_assign` capabilities
- Affects what resources users can access

## System Components

### Authentication
- **CakePHP Authentication Plugin** - Handles login/logout
- **Session, Token, Form authenticators** - Multiple auth methods
- **Two-Factor Authentication** - Optional 2FA support

### Authorization
- **CakeDC/Auth** - Role-based access control framework
- **Custom Rules** - Business-specific authorization logic
- **RbacPolicy** - Permission evaluation engine
- **Multi-tenant Isolation** - Automatic company-based filtering

### Key Files
```
watm/watm/
├── config/
│   ├── permissions.php              # Main permission configuration
│   └── Seeds/
│       ├── WATMRolesSeed.php        # System admin roles
│       └── CompanyUserRolesSeed.php # Customer roles
├── src/
│   └── Auth/Rules/
│       └── WatmRules.php            # Primary authorization rules
└── plugins/
    ├── UserManagement/
    │   ├── src/
    │   │   ├── Model/Table/
    │   │   │   ├── OUsersTable.php
    │   │   │   └── CompanyUsersTable.php
    │   │   ├── Auth/Rbac/Rules/
    │   │   │   ├── WATMCustomerRules.php
    │   │   │   └── RmaRules.php
    │   │   └── Middleware/
    │   │       └── UserConfigurationMiddleware.php
    └── UserAuthorization/
        └── src/
            ├── Middleware/
            │   └── AuthorizationAwareMiddleware.php
            └── Loader/
                └── AuthorizationServiceLoader.php
```

## Database Tables

### Core Tables
- `o_users` - User accounts
- `o_users_roles` - Role definitions
- `o_users_permissions` - Permission definitions
- `o_users_roles_o_users_permissions` - Role-permission mapping
- `company_users` - User-company association
- `companies` - Company/tenant records
- `account_types` - Company capability definitions

### Audit & Logging
- `o_logs` - Complete audit trail of user actions

## Important Notes

- **Superuser Bypass**: Users with `is_superuser = 1` bypass all permission checks
- **Role Restriction**: Only 'admin' and 'customer' roles are permitted system-wide
- **Company Status**: Inactive companies restrict user access automatically
- **API Authentication**: Uses API keys instead of standard user sessions
- **Multi-tenancy**: All customer data is automatically filtered by company membership

## Getting Help

For specific topics, refer to the individual documentation files listed above. Each file contains detailed information with code examples and explanations.

---

**Last Updated**: 2025-11-10
**WATM Version**: CakePHP 4.x
**Framework**: CakeDC/Auth with custom RBAC rules
