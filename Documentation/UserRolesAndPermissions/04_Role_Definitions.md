# Role Definitions and Management

This document covers user roles, role types, account types, and role management in the WATM platform.

## Table of Contents
- [Role Types](#role-types)
- [System Admin Roles](#system-admin-roles)
- [Customer Roles](#customer-roles)
- [Account Types](#account-types)
- [Role Assignment](#role-assignment)
- [Role-Permission Mapping](#role-permission-mapping)
- [Role Hierarchy](#role-hierarchy)

## Role Types

The WATM platform uses two distinct role types:

### Role Type: 'admin'

**Purpose**: System-level administrators

**Database**: `o_users_roles.role_type = 'admin'`

**Characteristics**:
- Full platform access across all companies
- Can manage all users, companies, and data
- Bypass most permission checks
- Seed file: `config/Seeds/WATMRolesSeed.php`

**Roles**:
- Admin

### Role Type: 'customer'

**Purpose**: Company-level users (tenants)

**Database**: `o_users_roles.role_type = 'customer'`

**Characteristics**:
- Access limited to their company and sub-companies
- Subject to account type restrictions
- Company-based data isolation enforced
- Seed file: `config/Seeds/CompanyUserRolesSeed.php`

**Roles**:
- Super Admin (customer-level)
- Admin (customer-level)

## System Admin Roles

### Admin Role

**Role Type**: `admin`

**Title**: `Admin`

**Database Record**:
```sql
INSERT INTO o_users_roles (title, role_type) VALUES ('Admin', 'admin');
```

**Seed File**: `config/Seeds/WATMRolesSeed.php`

```php
if (!$rolesTable->exists(['title' => 'Admin'])) {
    $data[] = [
        'title' => 'Admin',
        'role_type' => 'admin'
    ];
}
```

**Access Level**:
- Full system access
- Can manage all companies and their users
- Can access all features regardless of company account type
- Bypasses most authorization checks

**Typical Use Cases**:
- Platform administrators
- Support team members
- System maintenance personnel

**Permission Behavior**:
Most authorization rules return `true` immediately for admin role:

```php
if ($role === 'admin') {
    return true;
}
```

## Customer Roles

Customer roles are company-specific and apply to users within a tenant company.

### Super Admin (Customer)

**Role Type**: `customer`

**Title**: `Super Admin`

**Database Record**:
```sql
INSERT INTO o_users_roles (title, role_type) VALUES ('Super Admin', 'customer');
```

**Seed File**: `config/Seeds/CompanyUserRolesSeed.php`

```php
if (!$rolesTable->exists(['title' => 'Super Admin', 'role_type' => 'customer'])) {
    $data[] = [
        'title' => 'Super Admin',
        'role_type' => 'customer'
    ];
}
```

**Access Level**:
- Full access within their company and sub-companies
- Can manage company users
- Can configure company settings
- Subject to account type restrictions

**Typical Use Cases**:
- Company owner/primary contact
- Company administrators

### Admin (Customer)

**Role Type**: `customer`

**Title**: `Admin`

**Database Record**:
```sql
INSERT INTO o_users_roles (title, role_type) VALUES ('Admin', 'customer');
```

**Seed File**: `config/Seeds/CompanyUserRolesSeed.php`

```php
if (!$rolesTable->exists(['title' => 'Admin', 'role_type' => 'customer'])) {
    $data[] = [
        'title' => 'Admin',
        'role_type' => 'customer'
    ];
}
```

**Access Level**:
- Subset of Super Admin permissions
- Limited administrative access
- Specific permissions assigned via role-permission mapping

**Typical Use Cases**:
- Department managers
- Limited administrative users

**Note**: The distinction between Super Admin and Admin (customer) is defined by the specific permissions assigned to each role in the `o_users_roles_o_users_permissions` table.

## Account Types

Account types define company capabilities that affect what features users can access.

### Database Table: `account_types`

**Migration**: `config/Migrations/20240403125244_CreateAccountTypes.php`

**Schema**:
```sql
CREATE TABLE account_types (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    sort_order INT UNSIGNED NOT NULL DEFAULT 99,
    can_upcharge TINYINT(1) NOT NULL DEFAULT 0,  -- Added 20240425135818
    can_assign TINYINT(1) NOT NULL DEFAULT 0,    -- Added 20240503170156
    INDEX (sort_order)
);
```

### Account Type: Distributor

**Title**: `Distributor`

**Sort Order**: `10` (highest priority)

**Capabilities**:
- `can_upcharge = 1` - Can mark up pricing for sub-companies
- `can_assign = 1` - Can create sub-companies

**Features Enabled**:
- View and edit upcharge settings
- Create customer and subcustomer accounts
- Manage sub-company pricing
- Distributor credits and payouts
- Service plan management

**Typical Companies**:
- Resellers
- Managed service providers
- Partners with sub-clients

**Authorization Methods**:
- `orasesIsPremiumCustomer()` → `true`
- `orasesAllowUpcharging()` → `true` (if `company.allow_upcharging = 1`)
- `orasesCanAssign()` → `true`

### Account Type: Customer

**Title**: `Customer`

**Sort Order**: `20`

**Capabilities**:
- `can_upcharge = 0` - Cannot mark up pricing
- `can_assign = 1` - Can create sub-companies

**Features Enabled**:
- View own pricing (no markup)
- Create subcustomer accounts
- Manage own devices and billing
- Cannot see/edit upcharge settings

**Typical Companies**:
- Direct customers with divisions
- Companies with multiple locations

**Authorization Methods**:
- `orasesIsPremiumCustomer()` → `false`
- `orasesCanAssign()` → `true`
- `orasesIsCustomerNotSub()` → `true`

### Account Type: Subcustomer

**Title**: `Subcustomer`

**Sort Order**: `30` (lowest priority)

**Capabilities**:
- `can_upcharge = 0` - Cannot mark up pricing
- `can_assign = 0` - Cannot create sub-companies

**Features Enabled**:
- Limited feature access
- Pricing visibility controlled by `company.allow_self_billing`
- Uses parent company's payment methods (unless `allow_self_billing`)
- Cannot manage sub-companies

**Typical Companies**:
- Department or division of parent company
- Franchise location
- Branch office

**Authorization Methods**:
- `orasesIsPremiumCustomer()` → `false`
- `orasesCanAssign()` → `false`
- `orasesIsCustomerNotSub()` → `false`
- `orasesCanViewPricing()` → depends on `allow_self_billing`

### Account Type Comparison

| Feature | Distributor | Customer | Subcustomer |
|---------|-------------|----------|-------------|
| Can upcharge | ✓ | ✗ | ✗ |
| Can create sub-companies | ✓ | ✓ | ✗ |
| View all pricing | ✓ | ✓ | Conditional* |
| Edit upcharge settings | ✓ | ✗ | ✗ |
| Own payment methods | ✓ | ✓ | Conditional* |
| Manage service plans | ✓ | ✗ | ✗ |
| Distributor credits | ✓ | ✗ | ✗ |

*Conditional on `company.allow_self_billing` flag

## Role Assignment

### User Role Storage

Roles are stored in the `o_users` table in the `additional_data` JSON column:

**Field**: `o_users.additional_data['role_ids']`

**Format**: JSON string containing role ID(s)

**Example**:
```json
{
  "role_ids": "3",
  "other_settings": "..."
}
```

**Database Type**: JSON (MySQL 8.0+) or TEXT with JSON validation

### Assigning Roles

#### Method 1: Direct Database

```sql
-- Assign Super Admin (customer) role (ID 3) to user
UPDATE o_users
SET additional_data = JSON_SET(additional_data, '$.role_ids', '3')
WHERE id = 123;
```

#### Method 2: CakePHP Code

```php
$usersTable = TableRegistry::getTableLocator()->get('UserManagement.OUsers');
$user = $usersTable->get($userId);

// Assign role
$additionalData = $user->additional_data ?? [];
$additionalData['role_ids'] = '3';  // Role ID as string
$user->additional_data = $additionalData;

$usersTable->save($user);
```

#### Method 3: User Admin Interface

Users can be assigned roles through the admin interface:
- Navigate to: Admin → User Management → Users → Edit User
- Select role from dropdown
- Save changes

**Template**: `plugins/UserManagement/templates/Admin/Profile/edit.php`

### Role Lookup

Roles are searched using SQL LIKE query:

**OUsersTable Search Filter**:
```php
'Search.Callback',
[
    'callback' => function (\Cake\ORM\Query $query, array $args, \Search\Model\Filter\Base $filter) {
        if (!empty($args['user_role'])) {
            $query->where(
                [
                    'OUsers.additional_data LIKE "%\"role_ids\":\"' . intval($args['user_role']) . '\"%"',
                    'OUsers.is_superuser' => 0
                ]
            );
        } elseif (($args['user_role'] ?? null) === '0') {
            $query->where(['OUsers.is_superuser' => 1]);
        }
    }
]
```

**File**: `plugins/UserManagement/src/Model/Table/OUsersTable.php:45-61`

### Superuser Flag

In addition to roles, users can have the superuser flag set:

**Field**: `o_users.is_superuser`

**Type**: TINYINT(1) / BOOLEAN

**Behavior**:
- `is_superuser = 1` → Bypasses **all** permission checks
- Returns `true` for any authorization check
- Used for emergency access and system maintenance

**Usage**:
```php
if ($user->is_superuser) {
    return true;  // Grant all permissions
}
```

**Typical Use**: Platform owners, senior support staff

## Role-Permission Mapping

### Database Table: `o_users_roles_o_users_permissions`

**Purpose**: Links roles to specific permissions

**Schema**:
```sql
CREATE TABLE o_users_roles_o_users_permissions (
    o_users_role_id INT UNSIGNED NOT NULL,
    o_users_permission_id INT UNSIGNED NOT NULL,
    run_method VARCHAR(255) DEFAULT NULL,
    PRIMARY KEY (o_users_role_id, o_users_permission_id),
    FOREIGN KEY (o_users_role_id) REFERENCES o_users_roles(id),
    FOREIGN KEY (o_users_permission_id) REFERENCES o_users_permissions(id)
);
```

**Columns**:
- `o_users_role_id` - Role ID from `o_users_roles`
- `o_users_permission_id` - Permission ID from `o_users_permissions`
- `run_method` - Optional custom authorization method to call

### Permissions Table: `o_users_permissions`

**Purpose**: Define available permissions

**Schema**:
```sql
CREATE TABLE o_users_permissions (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    keyword VARCHAR(255) NOT NULL UNIQUE,
    description TEXT,
    run_method VARCHAR(255) DEFAULT NULL
);
```

**Example Permissions**:
- Keyword: `companies_browse` - View company list
- Keyword: `company_view` - View company details
- Keyword: `company_edit` - Edit company information
- Keyword: `device_browse` - View device list
- Keyword: `device_edit` - Edit devices
- Keyword: `invoice_browse` - View invoices
- Keyword: `invoice_edit` - Edit invoices
- Keyword: `faq_edit` - Edit FAQs
- Keyword: `faq_delete` - Delete FAQs

### Permission Assignment Example

```sql
-- Get role IDs
SELECT id, title FROM o_users_roles WHERE role_type = 'customer';
-- +----+-------------+
-- | id | title       |
-- +----+-------------+
-- |  3 | Super Admin |
-- |  4 | Admin       |
-- +----+-------------+

-- Get permission IDs
SELECT id, keyword FROM o_users_permissions WHERE keyword LIKE 'device%';
-- +----+---------------+
-- | id | keyword       |
-- +----+---------------+
-- | 10 | device_browse |
-- | 11 | device_edit   |
-- | 12 | device_delete |
-- +----+---------------+

-- Assign permissions to Super Admin role (ID 3)
INSERT INTO o_users_roles_o_users_permissions (o_users_role_id, o_users_permission_id)
VALUES
    (3, 10),  -- device_browse
    (3, 11),  -- device_edit
    (3, 12);  -- device_delete

-- Assign limited permissions to Admin role (ID 4)
INSERT INTO o_users_roles_o_users_permissions (o_users_role_id, o_users_permission_id)
VALUES
    (4, 10),  -- device_browse
    (4, 11);  -- device_edit (no delete)
```

### Using run_method

The `run_method` field specifies a custom authorization method:

**Example**:
```sql
INSERT INTO o_users_roles_o_users_permissions (o_users_role_id, o_users_permission_id, run_method)
VALUES (3, 15, 'orasesIsPremiumCustomer');
```

This will call `WatmRules::orasesIsPremiumCustomer()` when checking permission ID 15 for role ID 3.

## Role Hierarchy

### Company Hierarchy

```
┌─────────────────────────────────────────────────────────┐
│                   Platform Level                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              Admin (role_type=admin)            │   │
│  │  - Full system access                           │   │
│  │  - Manages all companies                        │   │
│  │  - Bypasses all permission checks               │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                   Company Level                          │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │    Distributor (account_type, can_upcharge=1)   │  │
│  │  ┌────────────────────────────────────────────┐ │  │
│  │  │ Super Admin, Admin (role_type=customer)    │ │  │
│  │  │  - Full company access                     │ │  │
│  │  │  - Manage sub-companies                    │ │  │
│  │  │  - Edit upcharge settings                  │ │  │
│  │  └────────────────────────────────────────────┘ │  │
│  │                         │                        │  │
│  │           ┌─────────────┴─────────────┐          │  │
│  │           ▼                           ▼          │  │
│  │  ┌─────────────────┐         ┌─────────────────┐│  │
│  │  │  Sub-Company 1  │         │  Sub-Company 2  ││  │
│  │  │  (Customer or   │         │  (Customer or   ││  │
│  │  │   Subcustomer)  │         │   Subcustomer)  ││  │
│  │  └─────────────────┘         └─────────────────┘│  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │    Customer (account_type, can_assign=1)        │  │
│  │  ┌────────────────────────────────────────────┐ │  │
│  │  │ Super Admin, Admin (role_type=customer)    │ │  │
│  │  │  - Full company access                     │ │  │
│  │  │  - Manage sub-companies                    │ │  │
│  │  │  - No upcharge settings                    │ │  │
│  │  └────────────────────────────────────────────┘ │  │
│  │                         │                        │  │
│  │                         ▼                        │  │
│  │              ┌─────────────────────┐             │  │
│  │              │  Sub-Company        │             │  │
│  │              │  (Subcustomer only) │             │  │
│  │              └─────────────────────┘             │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │    Subcustomer (account_type, can_assign=0)     │  │
│  │  ┌────────────────────────────────────────────┐ │  │
│  │  │ Super Admin, Admin (role_type=customer)    │ │  │
│  │  │  - Limited company access                  │ │  │
│  │  │  - Cannot create sub-companies             │ │  │
│  │  │  - Conditional pricing visibility          │ │  │
│  │  └────────────────────────────────────────────┘ │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Access Patterns by Role and Account Type

#### System Admin + Any Account Type
- Access: All companies, all features
- Restrictions: None
- Authorization: Bypasses all checks

#### Customer Role + Distributor Account Type
- Access: Own company + sub-companies
- Features: Full (upcharging, sub-company creation, pricing management)
- Restrictions: None within hierarchy

#### Customer Role + Customer Account Type
- Access: Own company + sub-companies
- Features: Most features (no upcharging)
- Restrictions: Cannot edit upcharge settings

#### Customer Role + Subcustomer Account Type
- Access: Own company only
- Features: Limited (conditional pricing, no sub-companies)
- Restrictions: Many features unavailable, uses parent's payment methods

### Permission Check Order

When a customer user attempts an action:

1. **Check Superuser Flag**
   ```php
   if ($user->is_superuser) return true;
   ```

2. **Check Role Type**
   ```php
   if (!in_array($role, ['admin', 'customer'])) return false;
   ```

3. **Admin Bypass**
   ```php
   if ($role === 'admin') return true;
   ```

4. **Get Company and Account Type**
   ```php
   $companyUser = $companyUsersTable->find()
       ->contain('Companies.AccountTypes')
       ->where(['o_user_id' => $user->id])
       ->first();
   ```

5. **Check Account Type Capability**
   ```php
   if (!$company->account_type->can_upcharge) return false;
   ```

6. **Check Resource Ownership**
   ```php
   return $this->entityBelongsToCompanyOrSubCompany($entity, $company);
   ```

## Role Management Best Practices

### 1. Role Assignment
- Assign roles via `o_users.additional_data['role_ids']`
- Store role ID as string in JSON
- Use role type 'admin' sparingly
- Default customer users to 'customer' role type

### 2. Superuser Usage
- Use `is_superuser` for emergency access only
- Document all superuser accounts
- Regular audit of superuser accounts
- Consider separate accounts for normal operations

### 3. Account Type Selection
- **Distributor**: Only for resellers/MSPs with sub-clients
- **Customer**: Direct customers with potential divisions
- **Subcustomer**: Departments, franchises, branches

### 4. Permission Granularity
- Use role-permission mapping for fine-grained control
- Define permissions with clear, descriptive keywords
- Document custom `run_method` implementations
- Test permission changes thoroughly

### 5. Company Hierarchy
- Keep hierarchy depth reasonable (2-3 levels max)
- Document parent-child relationships
- Set `allow_self_billing` appropriately for subcustomers
- Assign primary_user_id for each company

## Next Steps

- See [05_Database_Schema.md](05_Database_Schema.md) for database structure details
- See [06_Multi_Tenant_Architecture.md](06_Multi_Tenant_Architecture.md) for multi-tenancy implementation
- See [07_Implementation_Guide.md](07_Implementation_Guide.md) for practical development guidance
