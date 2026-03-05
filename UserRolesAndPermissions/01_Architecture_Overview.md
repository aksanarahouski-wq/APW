# Architecture Overview

This document provides a high-level overview of the WATM platform's authentication and authorization architecture.

## Table of Contents
- [System Architecture](#system-architecture)
- [Request Flow](#request-flow)
- [Plugin Architecture](#plugin-architecture)
- [Middleware Stack](#middleware-stack)
- [Authentication Components](#authentication-components)
- [Authorization Components](#authorization-components)

## System Architecture

The WATM platform uses a layered security architecture:

```
┌─────────────────────────────────────────────────────────────┐
│                      HTTP Request                            │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                  MIDDLEWARE LAYER                            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  1. SitesMiddleware (Multi-tenant)                    │  │
│  │  2. ErrorHandlerMiddleware                            │  │
│  │  3. CorsMiddleware (API CORS)                         │  │
│  │  4. AssetMiddleware                                   │  │
│  │  5. RoutingMiddleware                                 │  │
│  │  6. BodyParserMiddleware                              │  │
│  │  7. CsrfProtectionMiddleware (conditional)           │  │
│  └───────────────────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              AUTHENTICATION LAYER                            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  UserConfigurationMiddleware                          │  │
│  │    - Configures authenticators (Session, Token, Form) │  │
│  │    - Configures identifiers (Password, Token)         │  │
│  │    - Sets login/logout redirects                      │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  API Authentication (for /api routes)                 │  │
│  │    - ApiAuthenticationMiddleware (X-API-Key header)   │  │
│  │    - Validates API key against company_api_keys       │  │
│  └───────────────────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              AUTHORIZATION LAYER                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  1. AuthorizationAwareMiddleware                      │  │
│  │     - Prepares unauthorized redirect                  │  │
│  │     - Configures service loader                       │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  2. AuthorizationMiddleware (CakePHP Core)            │  │
│  │     - Evaluates permissions via RbacPolicy            │  │
│  │     - Uses MapResolver, RbacPolicy, OrmResolver       │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  3. RequestAuthorizationMiddleware                    │  │
│  │     - Enforces authorization decision                 │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  4. TwoFactorMiddleware (Optional)                    │  │
│  │     - 2FA verification                                │  │
│  └───────────────────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                  CONTROLLER LAYER                            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  AppController::beforeFilter()                        │  │
│  │    - Theme selection                                  │  │
│  │    - Host validation                                  │  │
│  │    - Token-to-session conversion                      │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Controller Action Execution                          │  │
│  │    - May call isAuthorized() for additional checks    │  │
│  │    - Business logic execution                         │  │
│  └───────────────────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                     VIEW LAYER                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Templates may use IsAuthorized helper                │  │
│  │    - Conditionally show/hide UI elements              │  │
│  │    - Render based on user permissions                 │  │
│  └───────────────────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
                  HTTP Response
```

## Request Flow

### Standard Web Request Flow

1. **Request Arrives** → Enters middleware pipeline
2. **SitesMiddleware** → Determines which tenant/site is being accessed
3. **RoutingMiddleware** → Matches URL to controller/action
4. **Authentication** → Identifies user (Session/Token/Form)
5. **Authorization Check** → Evaluates permissions using:
   - `config/permissions.php` configuration
   - Custom rule methods (WatmRules, etc.)
   - Role-based access control
6. **Controller Execution** → Action runs if authorized
7. **Template Rendering** → View with conditional UI based on permissions
8. **Response** → Returned to client

### API Request Flow

1. **Request Arrives** → Enters middleware pipeline
2. **ApiAuthenticationMiddleware** → Validates `X-API-Key` header
3. **API Key Validation** → Checks against `company_api_keys` table:
   - Key exists and is valid
   - Key is not expired
   - Key is active
4. **Company Context Set** → Request attributes populated:
   - `apiKey` - The API key entity
   - `company` - The company entity
   - `companyId` - The company ID
5. **Permission Bypass** → API routes typically use `bypassAuth: true`
6. **Controller Execution** → API action executes with company context
7. **JSON Response** → Returned to client

## Plugin Architecture

The WATM platform uses a plugin-based architecture where most functionality lives in plugins:

### Authentication & Authorization Plugins

#### Orases/Users
**Location**: Vendor package from `packages.orases.com`

**Purpose**: Foundation authentication package

**Key Components**:
- `OUsersTable` - Base user model
- `OrasesRules` - Base authorization rule class
- Authentication configuration utilities

**Provides**:
- User account management foundation
- Base authentication setup
- Base authorization rules to extend

#### UserManagement
**Location**: `/plugins/UserManagement/`

**Purpose**: WATM-specific user and role management

**Key Components**:
- `OUsersTable` (extends Orases) - Custom user model with WATM-specific finders
- `CompanyUsersTable` - Links users to companies
- `WATMCustomerRules` - Custom customer authorization rules
- `RmaRules` - RMA-specific authorization rules
- `UserConfigurationMiddleware` - Authentication configuration
- Controllers: Profile, OUserPreferredFilters

**Relationships**:
```
OUsers (one) ←→ (one) CompanyUser ←→ (many) Company
```

#### UserAuthorization
**Location**: `/plugins/UserAuthorization/`

**Purpose**: Authorization middleware and policy configuration

**Key Components**:
- `AuthorizationAwareMiddleware` - Prepares authorization environment
- `AuthorizationServiceLoader` - Configures authorization service with policies
- `Plugin.php` - Implements AuthorizationServiceProviderInterface

**Policy Stack**:
1. **MapResolver** - Maps ServerRequest to CollectionPolicy
2. **RbacPolicy** - Role-based access control evaluation
3. **OrmResolver** - ORM-based policy resolution

### Application-Level Authorization

#### App\Auth\Rules\WatmRules
**Location**: `/watm/watm/src/Auth/Rules/WatmRules.php`

**Purpose**: Primary authorization rule class for WATM platform

**Extends**: `Orases\Users\Auth\Rbac\Rules\OrasesRules`

**Contains**: 25+ custom authorization methods (see [03_Authorization_Rules.md](03_Authorization_Rules.md))

**Usage**: Referenced as catch-all in `config/permissions.php`:
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

## Middleware Stack

### Application Middleware (src/Application.php)

Configured in `Application::middleware()` method:

```php
public function middleware(MiddlewareQueue $middlewares): MiddlewareQueue
{
    $middlewares
        // Multi-tenant/white-label support
        ->add(new SitesMiddleware())

        // Error handling
        ->add(new ErrorHandlerMiddleware(Configure::read('Error')))

        // CORS for API requests
        ->add(new CorsMiddleware())

        // Static assets
        ->add(new AssetMiddleware([
            'cacheTime' => Configure::read('Asset.cacheTime'),
        ]))

        // Routing
        ->add(new RoutingMiddleware($this))

        // Request body parsing
        ->add(new BodyParserMiddleware())

        // CSRF protection (conditional)
        ->add(new CsrfProtectionMiddleware([
            'httponly' => true,
        ]));

    return $middlewares;
}
```

### Authorization Middleware (UserAuthorization Plugin)

Configured in `UserAuthorization\Plugin::middleware()`:

```php
public function middleware(MiddlewareQueue $middlewares): MiddlewareQueue
{
    $middlewares
        // Prepare authorization environment
        ->add(AuthorizationAwareMiddleware::class)

        // Core authorization evaluation
        ->add(AuthorizationMiddleware::class)

        // Enforce authorization decisions
        ->add(RequestAuthorizationMiddleware::class)

        // Two-factor authentication (optional)
        ->add(TwoFactorMiddleware::class);

    return $middlewares;
}
```

### API Middleware (Companies Plugin)

For `/api` routes, additional middleware is applied:

```php
// In routing configuration or controller
->add(ApiAuthenticationMiddleware::class)  // Validates X-API-Key
->add(ApiAuthorizationMiddleware::class)   // API-specific authorization
```

## Authentication Components

### Authenticators

Configured in `UserManagement\Middleware\UserConfigurationMiddleware`:

#### 1. Token Authenticator
**Purpose**: API and automated authentication via token

**Configuration**:
- Checks `token` query parameter OR
- Checks `Authorization` header
- Validates against user token field

**Use Case**: API integrations, email links with embedded tokens

#### 2. Session Authenticator
**Purpose**: Standard web session-based authentication

**Configuration**:
- Session key: `Auth`
- Identifies user from session data

**Use Case**: Normal logged-in web users

#### 3. Form Authenticator
**Purpose**: Login form submission handling

**Configuration**:
- Login URL: `/admin/admin/user-behaviors/login`
- Username/password fields

**Use Case**: User login page submissions

### Identifiers

#### 1. Password Identifier
**Purpose**: Username/email + password verification

**Configuration**:
- Fields: `['username', 'email']` for username lookup
- Field: `password` for password verification
- Uses CakePHP password hashing

#### 2. Token Identifier
**Purpose**: Token-based identification

**Configuration**:
- Token field name
- Token validation logic

## Authorization Components

### Permission Configuration

**File**: `config/permissions.php`

**Structure**: Array of permission rules

**Rule Format**:
```php
[
    'prefix' => 'Admin|Api|false',      // Route prefix
    'plugin' => 'PluginName',           // Plugin name
    'controller' => 'ControllerName',   // Controller name
    'action' => 'actionName|*|array',   // Action(s)
    'role' => 'admin|customer|*',       // Required role
    'allowed' => true|false|RuleClass,  // Permission grant/deny/evaluate
    'bypassAuth' => true|false,         // Skip authentication entirely
]
```

**Evaluation Order**: Top to bottom, first match wins

**Catch-All Rule**: Last rule uses `WatmRules()` for all unmatched routes

### Custom Rule Classes

Rule classes implement authorization logic:

**Base Class**: `Orases\Users\Auth\Rbac\Rules\OrasesRules`

**Method Signature**:
```php
public function methodName(
    OUser|uuid $user,           // User object or ID
    string $role,               // User's role ('admin'|'customer')
    ServerRequestInterface|array $request,  // Current request
    array $route_params = []    // Optional route parameters
): bool
```

**Return Values**:
- `true` - Grant permission
- `false` - Deny permission
- Other - Continue to next rule

**Key Rule Classes**:
1. `App\Auth\Rules\WatmRules` - Primary application rules
2. `UserManagement\Auth\Rbac\Rules\WATMCustomerRules` - Customer view restrictions
3. `UserManagement\Auth\Rbac\Rules\RmaRules` - RMA editing restrictions

### RbacPolicy

**Provided By**: CakeDC/Auth

**Purpose**: Evaluates permissions using configuration and custom rules

**Process**:
1. Receives authorization request for route
2. Looks up matching permission rule
3. If rule has `allowed` value:
   - `true` → Grant
   - `false` → Deny
   - Object (rule class) → Call rule method
4. Rule method evaluates and returns boolean
5. Result applied to request

### Multi-Tenant Isolation

**Mechanism**: Company-based data filtering

**Implementation**:
- User linked to company via `CompanyUser` record
- Authorization rules check resource belongs to user's company
- Supports parent-child company hierarchy

**Key Methods**:
- `entityBelongsToCompanyOrSubCompany()` - Check entity ownership
- `companyBelongsToUsersCompanyOrSubCompany()` - Check company relationship
- `orasesIsChildCustomer()` - Verify company hierarchy

## Configuration Files

### config/permissions.php
Main permission configuration with all route-level permissions

### config/app_local.php
Local environment configuration (authentication settings, database, etc.)

### config/sites.php
Multi-site/white-label configuration

### plugins/UserManagement/config/
Plugin-specific configuration

### plugins/UserAuthorization/config/
Authorization service configuration

## Key Classes Reference

### Controllers
- `App\Controller\AppController` - Base application controller
- `Admin\Admin\UserBehaviorsController` - Login/logout/password management

### Models
- `UserManagement\Model\Table\OUsersTable` - User model
- `UserManagement\Model\Table\CompanyUsersTable` - User-company link
- `Companies\Model\Table\CompaniesTable` - Company/tenant model
- `Companies\Model\Table\AccountTypesTable` - Company capability types

### Middleware
- `UserManagement\Middleware\UserConfigurationMiddleware` - Auth config
- `UserAuthorization\Middleware\AuthorizationAwareMiddleware` - Auth prep
- `Companies\Middleware\ApiAuthenticationMiddleware` - API key auth

### Rules
- `App\Auth\Rules\WatmRules` - Primary authorization rules
- `UserManagement\Auth\Rbac\Rules\WATMCustomerRules` - Customer rules
- `UserManagement\Auth\Rbac\Rules\RmaRules` - RMA rules

## Next Steps

- See [02_Permission_Configuration.md](02_Permission_Configuration.md) for detailed permission configuration reference
- See [03_Authorization_Rules.md](03_Authorization_Rules.md) for all custom rule methods
- See [07_Implementation_Guide.md](07_Implementation_Guide.md) for practical development guidance
