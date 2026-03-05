# Database Schema Reference

This document details the database tables and relationships for the WATM user roles and permissions system.

## Table of Contents
- [Users and Roles](#users-and-roles)
- [Company-User Relationship](#company-user-relationship)
- [Companies and Account Types](#companies-and-account-types)
- [Permissions](#permissions)
- [Audit Logging](#audit-logging)
- [Entity Relationship Diagram](#entity-relationship-diagram)

## Users and Roles

### o_users

**Purpose**: User accounts

**Source**: Orases Users package

**Key Fields**:
```sql
CREATE TABLE o_users (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) UNIQUE,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    first_name VARCHAR(255),
    last_name VARCHAR(255),
    is_superuser TINYINT(1) NOT NULL DEFAULT 0,
    is_active TINYINT(1) NOT NULL DEFAULT 1,
    role VARCHAR(255) DEFAULT 'customer',          -- Legacy, use additional_data
    additional_data JSON,                          -- Contains role_ids and other settings
    created DATETIME,
    modified DATETIME,
    deleted DATETIME DEFAULT NULL                  -- Soft delete
);
```

**Indexes**:
- PRIMARY KEY (`id`)
- UNIQUE KEY (`username`)
- UNIQUE KEY (`email`)
- INDEX (`is_active`)
- INDEX (`deleted`)

**Important JSON Fields** (`additional_data`):
- `role_ids` - Role ID(s) as string (e.g., "3")
- `can_access_company_documents` - Boolean for special document access
- Other custom settings

**Relationships**:
- `hasOne` → `company_users` (via `o_user_id`)
- `belongsTo` → `o_users_roles` (via `additional_data['role_ids']`, non-standard)

**File**: `plugins/UserManagement/src/Model/Table/OUsersTable.php`

**Custom Finders**:
- `findActiveAndNotDeleted()` - Active, non-deleted users
- `findCustomerUsersForCompany($companyId)` - Customer users for specific company
- `findSuperUsers()` - System admin users (`role_type = 'admin'`)

**Search Filters**:
```php
// Search by name
->like('name', ['fields' => ['first_name', 'last_name']])

// Search by role
->add('user_role', 'Search.Callback', [
    'callback' => function ($query, $args) {
        if (!empty($args['user_role'])) {
            $query->where([
                'OUsers.additional_data LIKE "%\"role_ids\":\"' . intval($args['user_role']) . '\"%"',
                'OUsers.is_superuser' => 0
            ]);
        } elseif (($args['user_role'] ?? null) === '0') {
            $query->where(['OUsers.is_superuser' => 1]);
        }
    }
])
```

### o_users_roles

**Purpose**: Role definitions

**Source**: Orases Users package

**Schema**:
```sql
CREATE TABLE o_users_roles (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    role_type VARCHAR(50) NOT NULL DEFAULT 'admin',  -- 'admin' or 'customer'
    run_method VARCHAR(255),                          -- Optional custom authorization method
    created DATETIME,
    modified DATETIME
);
```

**Migration**: `config/Migrations/20220902151522_AddTypeToOUsersRoles.php`

**Indexes**:
- PRIMARY KEY (`id`)
- INDEX (`role_type`)

**Standard Roles**:

| ID | Title | Role Type | Description |
|----|-------|-----------|-------------|
| 1 | Admin | admin | System administrators |
| 3 | Super Admin | customer | Company administrators (full access) |
| 4 | Admin | customer | Company administrators (limited access) |

**Seed Files**:
- `config/Seeds/WATMRolesSeed.php` - Admin role
- `config/Seeds/CompanyUserRolesSeed.php` - Super Admin, Admin (customer)

**Relationships**:
- `belongsToMany` → `o_users_permissions` (via `o_users_roles_o_users_permissions`)
- `hasMany` → `o_users` (via `additional_data['role_ids']`, non-standard)

### o_users_permissions

**Purpose**: Permission definitions

**Source**: Orases Users package

**Schema**:
```sql
CREATE TABLE o_users_permissions (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    keyword VARCHAR(255) UNIQUE NOT NULL,
    description TEXT,
    run_method VARCHAR(255),                   -- Optional custom authorization method
    created DATETIME,
    modified DATETIME
);
```

**Indexes**:
- PRIMARY KEY (`id`)
- UNIQUE KEY (`keyword`)

**Example Permissions**:

| ID | Keyword | Title | Run Method |
|----|---------|-------|------------|
| 1 | companies_browse | Browse Companies | NULL |
| 2 | company_view | View Company | orasesDefaultAllow |
| 3 | company_edit | Edit Company | orasesIsPremiumCustomer |
| 10 | device_browse | Browse Devices | NULL |
| 11 | device_edit | Edit Device | orasesMustBelongToCompany |
| 20 | invoice_browse | Browse Invoices | NULL |
| 21 | invoice_edit | Edit Invoice | orasesCanViewPricing |

**Relationships**:
- `belongsToMany` → `o_users_roles` (via `o_users_roles_o_users_permissions`)

### o_users_roles_o_users_permissions

**Purpose**: Maps roles to permissions

**Source**: Orases Users package

**Schema**:
```sql
CREATE TABLE o_users_roles_o_users_permissions (
    o_users_role_id INT UNSIGNED NOT NULL,
    o_users_permission_id INT UNSIGNED NOT NULL,
    run_method VARCHAR(255),                    -- Override permission's run_method
    PRIMARY KEY (o_users_role_id, o_users_permission_id),
    FOREIGN KEY (o_users_role_id) REFERENCES o_users_roles(id) ON DELETE CASCADE,
    FOREIGN KEY (o_users_permission_id) REFERENCES o_users_permissions(id) ON DELETE CASCADE
);
```

**Indexes**:
- PRIMARY KEY (`o_users_role_id`, `o_users_permission_id`)
- FOREIGN KEY (`o_users_role_id`)
- FOREIGN KEY (`o_users_permission_id`)

**Run Method Priority**:
1. If junction table has `run_method`, use that
2. Else if permission has `run_method`, use that
3. Else grant permission by default

**Example Data**:
```sql
-- Super Admin (customer) role gets all permissions
INSERT INTO o_users_roles_o_users_permissions (o_users_role_id, o_users_permission_id, run_method)
VALUES
    (3, 1, NULL),                           -- Browse companies
    (3, 2, 'orasesDefaultAllow'),          -- View company
    (3, 3, 'orasesIsPremiumCustomer'),     -- Edit company (if premium)
    (3, 10, NULL),                          -- Browse devices
    (3, 11, 'orasesMustBelongToCompany');  -- Edit device (if belongs to company)

-- Admin (customer) role gets limited permissions
INSERT INTO o_users_roles_o_users_permissions (o_users_role_id, o_users_permission_id, run_method)
VALUES
    (4, 1, NULL),                           -- Browse companies
    (4, 2, 'orasesDefaultAllow'),          -- View company
    (4, 10, NULL);                          -- Browse devices
    -- Note: No company edit, device edit permissions
```

## Company-User Relationship

### company_users

**Purpose**: Links users to companies (one-to-one)

**Source**: UserManagement plugin

**Schema**:
```sql
CREATE TABLE company_users (
    o_user_id INT UNSIGNED PRIMARY KEY,          -- Also serves as primary key
    company_id INT UNSIGNED NOT NULL,
    created DATETIME,
    modified DATETIME,
    FOREIGN KEY (o_user_id) REFERENCES o_users(id) ON DELETE CASCADE,
    FOREIGN KEY (company_id) REFERENCES companies(id) ON DELETE RESTRICT
);
```

**Migration**: `config/Migrations/20220920181417_CreateCompanyUsers.php`

**Indexes**:
- PRIMARY KEY (`o_user_id`)
- FOREIGN KEY (`o_user_id`)
- FOREIGN KEY (`company_id`)
- INDEX (`company_id`)

**Important Notes**:
- **One user = One company** (enforced by PK on `o_user_id`)
- User cannot belong to multiple companies
- Deleting user cascades to company_users
- Cannot delete company if users still linked (RESTRICT)

**Relationships**:
- `belongsTo` → `companies` (via `company_id`)
- `belongsTo` → `o_users` (via `o_user_id`)
- Alias: `belongsTo` → `o_users` as `Employees`

**File**: `plugins/UserManagement/src/Model/Table/CompanyUsersTable.php`

**Usage Example**:
```php
$companyUser = $companyUsersTable->find()
    ->contain(['Companies.AccountTypes', 'OUsers'])
    ->where(['o_user_id' => $userId])
    ->first();

$company = $companyUser->company;
$accountType = $company->account_type;
$user = $companyUser->o_user;
```

## Companies and Account Types

### companies

**Purpose**: Company/tenant records

**Source**: Companies plugin

**Key Fields**:
```sql
CREATE TABLE companies (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    account_type_id INT UNSIGNED NOT NULL,
    parent_company_id INT UNSIGNED DEFAULT NULL,    -- Hierarchical relationship
    primary_user_id INT UNSIGNED DEFAULT NULL,      -- Primary contact
    company_status_id INT UNSIGNED NOT NULL,
    allow_upcharging TINYINT(1) NOT NULL DEFAULT 0, -- Distributor setting
    allow_self_billing TINYINT(1) NOT NULL DEFAULT 0, -- Subcustomer setting
    has_electronically_consented TINYINT(1) NOT NULL DEFAULT 0,
    additional_data JSON,
    created DATETIME,
    modified DATETIME,
    deleted DATETIME DEFAULT NULL,
    FOREIGN KEY (account_type_id) REFERENCES account_types(id),
    FOREIGN KEY (parent_company_id) REFERENCES companies(id) ON DELETE SET NULL,
    FOREIGN KEY (primary_user_id) REFERENCES o_users(id) ON DELETE SET NULL,
    FOREIGN KEY (company_status_id) REFERENCES company_statuses(id)
);
```

**Migration**: `config/Migrations/20240403131020_AddAccountTypeIdToCompanies.php`

**Indexes**:
- PRIMARY KEY (`id`)
- FOREIGN KEY (`account_type_id`)
- FOREIGN KEY (`parent_company_id`)
- FOREIGN KEY (`primary_user_id`)
- FOREIGN KEY (`company_status_id`)
- INDEX (`deleted`)

**Tree Behavior**: Uses CakePHP Tree behavior for hierarchical queries

**Relationships**:
- `belongsTo` → `account_types` (via `account_type_id`)
- `belongsTo` → `companies` as `ParentCompanies` (via `parent_company_id`)
- `hasMany` → `companies` as `ChildCompanies` (via `parent_company_id`)
- `belongsTo` → `o_users` as `PrimaryUsers` (via `primary_user_id`)
- `hasMany` → `company_users` (via `company_id`)
- `belongsTo` → `company_statuses` (via `company_status_id`)

**Important Flags**:
- `allow_upcharging` - Enables upcharge features for distributors
- `allow_self_billing` - Allows subcustomers to see pricing and have own payment methods
- `has_electronically_consented` - Required for document access

**Hierarchical Queries**:
```php
// Find all child companies
$childCompanies = $companiesTable
    ->find('children', [
        'for' => $parentCompanyId,
        'threads' => 'full'
    ])
    ->select(['Companies.id', 'Companies.title'])
    ->all();

// Find path to root
$path = $companiesTable
    ->find('path', ['for' => $companyId])
    ->all();
```

### account_types

**Purpose**: Define company capability types

**Source**: Core application

**Schema**:
```sql
CREATE TABLE account_types (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    sort_order INT UNSIGNED NOT NULL DEFAULT 99,
    can_upcharge TINYINT(1) NOT NULL DEFAULT 0,  -- Can mark up pricing
    can_assign TINYINT(1) NOT NULL DEFAULT 0,    -- Can create sub-companies
    created DATETIME,
    modified DATETIME
);
```

**Migration Files**:
- `config/Migrations/20240403125244_CreateAccountTypes.php` - Create table
- `config/Migrations/20240425135818_AddCanUpchargeToAccountTypes.php` - Add can_upcharge
- `config/Migrations/20240503170156_AddCanAssignToAccountTypes.php` - Add can_assign

**Indexes**:
- PRIMARY KEY (`id`)
- INDEX (`sort_order`)

**Standard Account Types**:

| ID | Title | Sort Order | Can Upcharge | Can Assign |
|----|-------|------------|--------------|------------|
| 1 | Distributor | 10 | 1 | 1 |
| 2 | Customer | 20 | 0 | 1 |
| 3 | Subcustomer | 30 | 0 | 0 |

**Relationships**:
- `hasMany` → `companies` (via `account_type_id`)

**Sort Order Usage**: Lower numbers = higher priority/more capabilities

### company_statuses

**Purpose**: Company status definitions

**Source**: Companies plugin

**Key Fields**:
```sql
CREATE TABLE company_statuses (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    status_key VARCHAR(50) UNIQUE NOT NULL,  -- 'active', 'inactive', 'pending'
    sort_order INT UNSIGNED NOT NULL DEFAULT 99,
    created DATETIME,
    modified DATETIME
);
```

**Standard Statuses**:

| ID | Title | Status Key | Description |
|----|-------|------------|-------------|
| 1 | Active | active | Company is operational |
| 2 | Inactive | inactive | Company is suspended |
| 3 | Pending | pending | Company not yet activated |

**Relationships**:
- `hasMany` → `companies` (via `company_status_id`)

**Authorization Usage**:
```php
// Check if company is active
if ($company->company_status->status_key === 'active') {
    return true;
}
```

## Permissions

The permission system uses several configuration mechanisms:

### Static Configuration

**File**: `config/permissions.php`

Not stored in database - evaluated at runtime by authorization middleware.

### Dynamic Permissions (Database-Driven)

**Tables Used**:
1. `o_users_permissions` - Define permissions
2. `o_users_roles` - Define roles
3. `o_users_roles_o_users_permissions` - Map roles to permissions

**How It Works**:
1. Permission defined in `o_users_permissions`
2. Permission assigned to role in junction table
3. User assigned role via `o_users.additional_data['role_ids']`
4. Authorization check:
   - Get user's role from `additional_data`
   - Lookup role-permission mapping
   - If `run_method` specified, call that method
   - Otherwise grant permission

## Audit Logging

### o_logs

**Purpose**: Audit trail of all user actions

**Source**: Orases Logs package

**Key Fields**:
```sql
CREATE TABLE o_logs (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    o_user_id INT UNSIGNED,
    category VARCHAR(255),                    -- e.g., 'UserManagement', 'Devices'
    action VARCHAR(255),                      -- e.g., 'created', 'updated', 'deleted'
    message TEXT,
    metadata JSON,                            -- Additional structured data
    ip_address VARCHAR(45),
    user_agent TEXT,
    created DATETIME,
    FOREIGN KEY (o_user_id) REFERENCES o_users(id) ON DELETE SET NULL
);
```

**Indexes**:
- PRIMARY KEY (`id`)
- INDEX (`o_user_id`)
- INDEX (`category`)
- INDEX (`action`)
- INDEX (`created`)

**Usage Example** (`OUsersTable::afterSave()`):
```php
if ($entity->isNew()) {
    // User created
    $this->logEvent('created user', 'user', 'created', [
        'user_id' => $entity->id,
        'username' => $entity->username,
        'email' => $entity->email
    ]);
} elseif ($entity->isDirty('password')) {
    // Password changed
    $this->logEvent('changed password', 'user', 'password_changed', [
        'user_id' => $entity->id
    ]);
} elseif (!empty($entity->deleted)) {
    // User soft-deleted
    $this->logEvent('deleted user', 'user', 'deleted', [
        'user_id' => $entity->id,
        'username' => $entity->username
    ]);
}
```

**File**: `plugins/UserManagement/src/Model/Table/OUsersTable.php:275-329`

**Searchable**: Yes, via admin interface

**Retention**: Permanent (no automatic purging)

## Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              USERS & ROLES                               │
└─────────────────────────────────────────────────────────────────────────┘

    ┌──────────────────┐
    │    o_users       │
    ├──────────────────┤
    │ PK id            │
    │    username      │
    │    email         │
    │    password      │
    │    first_name    │
    │    last_name     │
    │    is_superuser  │
    │    additional_data│◄──── Contains: {"role_ids": "3"}
    │    deleted       │
    └────────┬─────────┘
             │
             │ hasOne
             ▼
    ┌──────────────────┐
    │  company_users   │
    ├──────────────────┤
    │ PK,FK o_user_id  │───┐
    │ FK company_id    │   │
    └────────┬─────────┘   │
             │              │
             │ belongsTo    │
             ▼              │
    ┌──────────────────┐   │
    │    companies     │   │
    ├──────────────────┤   │
    │ PK id            │◄──┘
    │ FK account_type_id│───┐
    │ FK parent_company_id│─┼───┐
    │ FK primary_user_id│◄──┼───┼── (circular reference to o_users)
    │ FK company_status_id│ │   │
    │    title         │   │   │
    │    allow_upcharging│ │   │
    │    allow_self_billing│ │  │
    │    deleted       │   │   │
    └──────────────────┘   │   │
             ▲              │   │
             │ belongsTo    │   │
             │              │   │
    ┌────────┴────────┐    │   │
    │  account_types  │◄───┘   │
    ├─────────────────┤        │
    │ PK id           │        │
    │    title        │        │
    │    sort_order   │        │
    │    can_upcharge │        │
    │    can_assign   │        │
    └─────────────────┘        │
                               │
             ┌─────────────────┘
             │ self-reference (parent_company_id)
             │
    ┌────────▼────────┐
    │    companies    │
    │   (hierarchy)   │
    └─────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                           ROLES & PERMISSIONS                            │
└─────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────────┐
    │  o_users_roles      │
    ├─────────────────────┤
    │ PK id               │
    │    title            │
    │    role_type        │◄──── 'admin' or 'customer'
    │    run_method       │
    └──────────┬──────────┘
               │
               │ belongsToMany
               │
    ┌──────────▼───────────────────────────────┐
    │ o_users_roles_o_users_permissions        │
    ├──────────────────────────────────────────┤
    │ PK,FK o_users_role_id                    │
    │ PK,FK o_users_permission_id              │
    │       run_method (override)              │
    └──────────┬───────────────────────────────┘
               │
               │ belongsToMany
               │
    ┌──────────▼──────────┐
    │ o_users_permissions │
    ├─────────────────────┤
    │ PK id               │
    │    title            │
    │    keyword          │
    │    run_method       │
    └─────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                              AUDIT LOGGING                               │
└─────────────────────────────────────────────────────────────────────────┘

    ┌──────────────────┐
    │    o_logs        │
    ├──────────────────┤
    │ PK id            │
    │ FK o_user_id     │───► references o_users(id)
    │    category      │
    │    action        │
    │    message       │
    │    metadata      │◄──── JSON structured data
    │    ip_address    │
    │    user_agent    │
    │    created       │
    └──────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                            COMPANY STATUS                                │
└─────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────────┐
    │  company_statuses   │
    ├─────────────────────┤
    │ PK id               │
    │    title            │
    │    status_key       │◄──── 'active', 'inactive', 'pending'
    │    sort_order       │
    └──────────┬──────────┘
               │
               │ hasMany
               ▼
         (companies.company_status_id)
```

## Database Queries Reference

### Get User with Company and Account Type

```php
$companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
$companyUser = $companyUsersTable->find()
    ->contain([
        'Companies.AccountTypes',
        'Companies.CompanyStatuses',
        'OUsers'
    ])
    ->where(['o_user_id' => $userId])
    ->first();

$user = $companyUser->o_user;
$company = $companyUser->company;
$accountType = $company->account_type;
$canUpcharge = $accountType->can_upcharge;
$isActive = $company->company_status->status_key === 'active';
```

### Get All Child Companies

```php
$companiesTable = TableRegistry::getTableLocator()->get('Companies.Companies');
$childCompanies = $companiesTable
    ->find('children', [
        'for' => $parentCompanyId,
        'threads' => 'full'
    ])
    ->select(['Companies.id', 'Companies.title'])
    ->all()
    ->extract('id')
    ->toArray();
```

### Check User Role

```php
$usersTable = TableRegistry::getTableLocator()->get('UserManagement.OUsers');
$user = $usersTable->get($userId);

$roleId = $user->additional_data['role_ids'] ?? null;

if ($roleId === '3') {
    // Super Admin (customer)
} elseif ($roleId === '4') {
    // Admin (customer)
} elseif ($roleId === '1') {
    // Admin (system)
}
```

### Search Users by Role

```php
$usersTable = TableRegistry::getTableLocator()->get('UserManagement.OUsers');
$users = $usersTable->find()
    ->where([
        'OUsers.additional_data LIKE' => '%"role_ids":"3"%',
        'OUsers.is_superuser' => 0
    ])
    ->all();
```

### Get Role Permissions

```php
$rolesTable = TableRegistry::getTableLocator()->get('Orases/Users.OUsersRoles');
$role = $rolesTable->find()
    ->contain(['OUsersPermissions'])
    ->where(['OUsersRoles.id' => $roleId])
    ->first();

foreach ($role->o_users_permissions as $permission) {
    echo $permission->keyword . ': ' . $permission->title . "\n";
}
```

### Audit Log Query

```php
$logsTable = TableRegistry::getTableLocator()->get('Orases/Logs.OLogs');
$logs = $logsTable->find()
    ->where([
        'o_user_id' => $userId,
        'category' => 'user',
        'created >=' => new DateTime('-30 days')
    ])
    ->order(['created' => 'DESC'])
    ->all();
```

## Next Steps

- See [06_Multi_Tenant_Architecture.md](06_Multi_Tenant_Architecture.md) for how multi-tenancy is implemented
- See [07_Implementation_Guide.md](07_Implementation_Guide.md) for working with these tables in code
