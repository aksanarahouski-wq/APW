# Permission Configuration Reference

This document provides a complete reference for the WATM permission configuration file.

**File Location**: `/watm/watm/config/permissions.php`

## Table of Contents
- [Configuration Structure](#configuration-structure)
- [Rule Properties](#rule-properties)
- [Public Access Rules](#public-access-rules)
- [API Endpoint Rules](#api-endpoint-rules)
- [Authenticated User Rules](#authenticated-user-rules)
- [Role-Specific Rules](#role-specific-rules)
- [Custom Rule References](#custom-rule-references)
- [Catch-All Rule](#catch-all-rule)

## Configuration Structure

The permissions are configured as an array under the key `'CakeDC/Auth.permissions'`:

```php
return [
    'CakeDC/Auth.permissions' => [
        // Array of permission rules...
    ]
];
```

**Evaluation Order**: Rules are evaluated **top to bottom**, and the **first matching rule** wins.

**Matching Logic**: A rule matches when all specified properties match the current request.

## Rule Properties

Each permission rule is an associative array with the following properties:

| Property | Type | Description | Examples |
|----------|------|-------------|----------|
| `prefix` | string\|false | Route prefix | `'Admin'`, `'Api'`, `false`, `'Api/V1'` |
| `plugin` | string | Plugin name | `'Devices'`, `'Companies'`, `'Billing'` |
| `controller` | string | Controller name | `'Devices'`, `'Companies'`, `'UserBehaviors'` |
| `action` | string\|array | Action name(s) | `'index'`, `'*'`, `['edit', 'delete']` |
| `role` | string | Required role | `'admin'`, `'customer'`, `'*'` (any authenticated) |
| `allowed` | bool\|object | Permission grant | `true`, `false`, `new RuleClass()` |
| `bypassAuth` | bool | Skip authentication | `true`, `false` |

### Property Wildcards

- `'*'` - Matches any value
- `['action1', 'action2']` - Matches any value in array
- Omitted property - Matches any value

### Permission Grant Values

**`bypassAuth: true`**
- Allows access without authentication
- No user login required
- Use for public pages, API callbacks, etc.

**`allowed: true`**
- Grants permission (user must be authenticated)
- No additional checks performed
- Use when role alone determines access

**`allowed: false`**
- Explicitly denies permission
- Typically not used (rely on catch-all instead)

**`allowed: new RuleClass()`**
- Evaluates permission dynamically
- Calls rule class method for decision
- Use for complex business logic

**`role: '*'`**
- Any authenticated user
- Works with `allowed: true` for universal access

## Public Access Rules

Rules that allow access without authentication (`bypassAuth: true`):

### User Authentication Actions

```php
[
    'prefix' => 'Admin',
    'plugin' => 'Admin',
    'controller' => 'UserBehaviors',
    'action' => [
        'login',
        'tokenToSession',
        'logout',
        'requestResetPassword',
        'resetPassword',
        'changePassword',
        'validateEmail',
        'verify',
        'resendCode',
    ],
    'bypassAuth' => true,
],
```

**Purpose**: Login, logout, password reset, email verification

**Access**: Public (no authentication required)

### Verizon API Callbacks

```php
[
    'prefix' => 'Api',
    'plugin' => 'Devices',
    'controller' => 'VerizonCallbacks',
    'action' => ['index', 'deviceUsage'],
    'bypassAuth' => true,
],
```

**Purpose**: Verizon service webhook callbacks

**Access**: Public (external system callback)

**Security Note**: Controller should validate callback authenticity

### Devices API

```php
[
    'prefix' => 'Api',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => '*',
    'bypassAuth' => true,
],
```

**Purpose**: Device check-in and telemetry submission

**Access**: Public (devices authenticate via device ID)

**Security Note**: Controller validates device identity

### API v1 Devices

```php
[
    'prefix' => 'Api/V1',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => '*',
    'bypassAuth' => true,
],
```

**Purpose**: REST API for device management

**Access**: Public (uses API key authentication)

**Security Note**: `ApiAuthenticationMiddleware` validates `X-API-Key` header

### WooCommerce Integration

```php
[
    'prefix' => 'Api',
    'plugin' => 'WoocommerceApi',
    'controller' => 'WoocommerceIntegrations',
    'action' => 'index',
    'bypassAuth' => true,
],
```

**Purpose**: WooCommerce webhook endpoint

**Access**: Public (external e-commerce system callback)

### Adobe Sign Verification

```php
[
    'prefix' => false,
    'plugin' => 'AdobeSign',
    'controller' => 'AdobeSign',
    'action' => 'index',
    'bypassAuth' => true,
],
```

**Purpose**: Adobe Sign webhook verification

**Access**: Public (Adobe Sign callback)

### Company Creation (WooCommerce)

```php
[
    'prefix' => false,
    'plugin' => 'Companies',
    'controller' => 'Companies',
    'action' => 'index',
    'bypassAuth' => true,
],
```

**Purpose**: Company creation via WooCommerce integration

**Access**: Public (called by WooCommerce webhook)

### Company Invitation

```php
[
    'prefix' => false,
    'plugin' => 'Companies',
    'controller' => 'Companies',
    'action' => 'invitation',
    'bypassAuth' => true,
],
```

**Purpose**: New user registration via invitation link

**Access**: Public (requires valid invitation token in URL)

### VPN Access List

```php
[
    'prefix' => false,
    'plugin' => 'Companies',
    'controller' => 'Companies',
    'action' => 'accessList',
    'bypassAuth' => true,
],
```

**Purpose**: VPN configuration retrieval

**Access**: Public (VPN server accesses this)

### Debug Kit

```php
[
    'role' => '*',
    'plugin' => 'DebugKit',
    'controller' => '*',
    'action' => '*',
    'bypassAuth' => true,
],
```

**Purpose**: CakePHP DebugKit development tool

**Access**: Public in development (should be disabled in production)

## Authenticated User Rules

Rules that require authentication but allow any role (`role: '*', allowed: true`):

### Dashboard

```php
[
    'prefix' => 'Admin',
    'plugin' => 'Dashboard',
    'controller' => 'Dashboard',
    'action' => 'index',
    'role' => '*',
    'allowed' => true,
    'bypassAuth' => false
],
```

**Purpose**: Main dashboard page

**Access**: All authenticated users

**File**: `plugins/Dashboard/src/Controller/Admin/DashboardController.php:index()`

### User Filter Preferences

```php
[
    'prefix' => 'Admin',
    'plugin' => 'UserManagement',
    'controller' => 'OUserPreferredFilters',
    'action' => 'edit',
    'role' => '*',
    'allowed' => true,
    'bypassAuth' => false
],
```

**Purpose**: User's saved filter preferences

**Access**: All authenticated users (editing own preferences)

**File**: `plugins/UserManagement/src/Controller/Admin/OUserPreferredFiltersController.php:edit()`

### Device Location Update

```php
[
    'prefix' => 'Admin',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => 'getLocation',
    'role' => '*',
    'allowed' => true,
    'bypassAuth' => false
],
```

**Purpose**: AJAX endpoint to fetch device location

**Access**: All authenticated users

**Note**: Controller enforces device belongs to user's company

### Device Password Update

```php
[
    'prefix' => 'Admin',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => 'updatePassword',
    'role' => '*',
    'allowed' => true,
    'bypassAuth' => false
],
```

**Purpose**: Update device access password

**Access**: All authenticated users

**Note**: Controller enforces device ownership

### Companies API

```php
[
    'prefix' => 'Api',
    'plugin' => 'Companies',
    'controller' => 'Companies',
    'action' => '*',
    'allowed' => true,
    'bypassAuth' => false
],
```

**Purpose**: Internal API for company data

**Access**: All authenticated users

**Note**: Returns data filtered by user's company access

### File Upload/Download

```php
[
    'plugin' => 'Orases/Files',
    'controller' => 'OFiles',
    'action' => ['upload', 'downloadFile', 'thumbnail', 'viewFile'],
    'allowed' => true,
],
```

**Purpose**: File upload and download functionality

**Access**: All authenticated users

**Note**: Files are associated with specific records and company

**Security**: Controller validates user has access to the parent record

### Invoice Files

```php
[
    'prefix' => 'Admin',
    'plugin' => 'Billing',
    'controller' => 'Invoices',
    'action' => ['downloadFile', 'viewFile'],
    'allowed' => true,
],
```

**Purpose**: Invoice PDF viewing and downloading

**Access**: All authenticated users

**Security**: Controller enforces invoice belongs to user's company

**Note**: Permission granted here, but **restriction of downloadable files is enforced in the controller action**

### Tax Documents

```php
[
    'prefix' => 'Admin',
    'plugin' => 'Companies',
    'controller' => 'CompanyTaxDocuments',
    'action' => ['viewTaxDocument', 'downloadTaxDocument'],
    'allowed' => true,
],
```

**Purpose**: Company tax document access

**Access**: All authenticated users

**Security**: Controller enforces document belongs to user's company

**Note**: Permission granted here, but **restriction of downloadable files is enforced in the controller action**

### API Documentation

```php
[
    'prefix' => 'Admin',
    'plugin' => 'Companies',
    'controller' => 'ApiDocumentation',
    'action' => '*',
    'role' => '*',
    'allowed' => true,
],
```

**Purpose**: API documentation viewer

**Access**: All authenticated users

**File**: `plugins/Companies/src/Controller/Admin/ApiDocumentationController.php`

### Power Schedule Management

```php
[
    'prefix' => 'Admin',
    'plugin' => 'Companies',
    'controller' => 'CompanyPowerSchedules',
    'action' => [
        'getDevicesData',
        'getSchedules',
        'assignSchedule',
        'getDeviceLogs',
        'emergencyPowerOverride',
        'removePowerOverride',
        'bulkAssignSchedule',
        'bulkEmergencyPowerOverrides',
        'bulkRemovePowerOverrides',
    ],
    'allowed' => true,
],
```

**Purpose**: Device power schedule configuration API endpoints

**Access**: All authenticated users

**Security**: Controller filters devices by user's company

**Note**: AJAX endpoints for power schedule management UI

## Role-Specific Rules

Rules that restrict access to specific roles:

### My Company Page (Customer Role)

```php
[
    'prefix' => 'Admin',
    'plugin' => 'Companies',
    'controller' => 'Companies',
    'action' => 'myCompany',
    'role' => 'customer',
    'allowed' => true,
],
```

**Purpose**: Customer-facing company profile page

**Access**: Customer role only

**Note**: Shows read-only view of user's own company

**File**: `plugins/Companies/src/Controller/Admin/CompaniesController.php:myCompany()`

## Custom Rule References

Rules that use custom authorization logic:

### Company View (Customer Rule)

```php
[
    'prefix' => 'Admin',
    'plugin' => 'Companies',
    'controller' => 'Companies',
    'action' => 'view',
    'role' => 'customer',
    'bypassAuth' => false,
    'allowed' => new \UserManagement\Auth\Rbac\Rules\WATMCustomerRules()
],
```

**Purpose**: View company details

**Access**: Customer role only

**Custom Logic** (`WATMCustomerRules::orasesDefaultAllow()`):
- User can view their own company
- User can view child companies (if parent)
- User can view parent company

**File**: `plugins/UserManagement/src/Auth/Rbac/Rules/WATMCustomerRules.php:11-30`

**Logic**:
```php
// They can view:
// 1. Their own company
// 2. Their parent company (if subcustomer)
// 3. Their child companies (if parent)

$companyId = $request->getParam('pass.0');
$companyEntity = $companiesTable->find()->where(['Companies.id' => $companyId])->first();

// Check if user is associated with this company or its parent
$companyIds = [$companyId];
if ($companyEntity->hasValue('parent_company_id')) {
    $companyIds[] = $companyEntity->parent_company_id;
}

return $companyUsersTable->exists([
    'CompanyUsers.company_id IN' => $companyIds,
    'CompanyUsers.o_user_id' => $user->id
]);
```

### RMA Editing (RMA Rules)

```php
[
    'prefix' => 'Admin',
    'plugin' => 'Devices',
    'controller' => 'Rmas',
    'action' => 'edit',
    'role' => '*',
    'bypassAuth' => false,
    'allowed' => new \UserManagement\Auth\Rbac\Rules\RmaRules()
],
```

**Purpose**: Edit RMA (Return Merchandise Authorization)

**Access**: All authenticated users

**Custom Logic** (`RmaRules::orasesDefaultAllow()`):
- **Superusers**: Can edit RMAs with status "RMA In Progress"
- **Regular users**: Cannot edit RMAs (returns false)

**File**: `plugins/UserManagement/src/Auth/Rbac/Rules/RmaRules.php:12-39`

**Logic**:
```php
if ($user->is_superuser) {
    if (empty($passedParam)) {
        return true;  // Creating new RMA
    } else {
        // Editing existing RMA - check status
        $rmaEntity = $rmasTable->find()
            ->where(['Rmas.id' => $passedParam[0]])
            ->matching('RmaStatuses', function (Query $query) {
                return $query->where(['RmaStatuses.title' => 'RMA In Progress']);
            })
            ->first();

        if (isset($rmaEntity)) {
            return true;  // RMA is in editable status
        }
    }
}

return false;  // All other cases denied
```

## Catch-All Rule

The last rule in the configuration handles all unmatched requests:

```php
[
    'role' => '*',
    'controller' => '*',
    'plugin' => '*',
    'prefix' => '*',
    'action' => '*',
    'allowed' => new \App\Auth\Rules\WatmRules()
]
```

**Purpose**: Handle all routes not explicitly configured above

**Access**: Evaluated by `WatmRules` class

**Evaluation**: Calls `WatmRules::orasesDefaultAllow()` which:
1. Returns `true` if user is superuser
2. Returns `false` if role is not 'customer' or 'admin'
3. Delegates to parent class otherwise

**File**: `src/Auth/Rules/WatmRules.php:24-35`

**Default Logic**:
```php
if ($user->is_superuser) {
    return true;  // Superusers bypass all checks
}

if (!in_array($role, ['customer', 'admin'])) {
    return false;  // Only 'customer' and 'admin' roles allowed
}

return parent::orasesDefaultAllow($user, $role, $request, $route_params);
```

**Additional Checks**: The WatmRules class contains 25+ methods for specific business logic (see [03_Authorization_Rules.md](03_Authorization_Rules.md))

## Permission Debugging

### Check Current Permissions

In a controller:
```php
use CakeDC\Auth\Traits\IsAuthorizedTrait;

public function someAction() {
    $isAuthorized = $this->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'Devices',
        'controller' => 'Devices',
        'action' => 'edit',
    ]);

    debug($isAuthorized);  // true or false
}
```

### Log Permission Checks

Enable debug logging in `config/app_local.php`:

```php
'Log' => [
    'debug' => [
        'className' => FileLog::class,
        'path' => LOGS,
        'file' => 'debug',
        'levels' => ['notice', 'info', 'debug'],
    ],
],
```

Add logging to custom rules:
```php
public function orasesDefaultAllow($user, $role, $request, $route_params = [])
{
    Log::debug('Permission check', [
        'user_id' => $user->id,
        'role' => $role,
        'controller' => $request->getParam('controller'),
        'action' => $request->getParam('action'),
    ]);

    // ... permission logic
}
```

## Common Patterns

### Pattern 1: Public API Endpoint
```php
[
    'prefix' => 'Api',
    'plugin' => 'MyPlugin',
    'controller' => 'MyController',
    'action' => 'webhook',
    'bypassAuth' => true,
]
```

### Pattern 2: All Authenticated Users
```php
[
    'prefix' => 'Admin',
    'plugin' => 'MyPlugin',
    'controller' => 'MyController',
    'action' => 'index',
    'role' => '*',
    'allowed' => true,
]
```

### Pattern 3: Role-Specific Access
```php
[
    'prefix' => 'Admin',
    'plugin' => 'MyPlugin',
    'controller' => 'MyController',
    'action' => 'adminOnly',
    'role' => 'admin',
    'allowed' => true,
]
```

### Pattern 4: Custom Business Logic
```php
[
    'prefix' => 'Admin',
    'plugin' => 'MyPlugin',
    'controller' => 'MyController',
    'action' => 'complexCheck',
    'role' => '*',
    'allowed' => new \App\Auth\Rules\WatmRules()
]
```

Then in `WatmRules.php`:
```php
public function orasesMyCustomCheck($user, $role, $request, $route_params = [])
{
    // Custom authorization logic here
    return $someCondition;
}
```

## Best Practices

1. **Order Matters**: Place more specific rules before general rules
2. **Security First**: Default to deny, explicitly grant permissions
3. **Role Restriction**: Limit to 'admin' and 'customer' roles only
4. **Multi-tenant**: Always check resource belongs to user's company
5. **API Keys**: Use `bypassAuth: true` + `ApiAuthenticationMiddleware` for API routes
6. **Controller Validation**: Permission grants access, but controller should validate ownership
7. **File Access**: Even with `allowed: true`, enforce ownership in controller action
8. **Document**: Comment complex rules explaining the business logic

## Next Steps

- See [03_Authorization_Rules.md](03_Authorization_Rules.md) for detailed custom rule methods
- See [07_Implementation_Guide.md](07_Implementation_Guide.md) for adding new permissions
