# Quick Reference Cheat Sheet

Quick reference for common tasks with the WATM roles and permissions system.

## Permission Checks

### In Controller

```php
// Check permission
if ($this->isAuthorized([
    'prefix' => 'Admin',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => 'edit',
])) {
    // User is authorized
}
```

### In Template

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

## User & Company Data

### Get Current User

```php
$user = $this->Authentication->getIdentity();
$userId = $user->id;
$email = $user->email;
$isSuperuser = $user->is_superuser;
```

### Get User's Company

```php
$companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
$companyUser = $companyUsersTable->find()
    ->contain(['Companies.AccountTypes', 'Companies.CompanyStatuses'])
    ->where(['o_user_id' => $user->id])
    ->first();

$company = $companyUser->company;
$accountType = $company->account_type;
```

### Get User's Role

```php
$roleId = $user->additional_data['role_ids'] ?? null;

// Role IDs:
// '1' = Admin (system)
// '3' = Super Admin (customer)
// '4' = Admin (customer)
```

### Check Account Type

```php
$accountType = $company->account_type;

if ($accountType->can_upcharge) {
    // Distributor
}

if ($accountType->can_assign) {
    // Can create sub-companies (Distributor or Customer)
}

if ($accountType->title === 'Subcustomer') {
    // Subcustomer
}
```

### Check Company Status

```php
$isActive = $company->company_status->status_key === 'active';
```

### Check Primary User

```php
$isPrimary = $company->primary_user_id === $user->id;
```

## Multi-Tenant Filtering

### Get Accessible Company IDs

```php
$accessibleCompanyIds = [$company->id];

if ($company->account_type->can_assign) {
    $companiesTable = TableRegistry::getTableLocator()->get('Companies.Companies');
    $childIds = $companiesTable
        ->find('children', ['for' => $company->id, 'threads' => 'full'])
        ->select(['Companies.id'])
        ->all()
        ->extract('id')
        ->toArray();

    $accessibleCompanyIds = array_merge($accessibleCompanyIds, $childIds);
}
```

### Filter Query by Company

```php
$devices = $this->Devices->find()
    ->where(['Devices.company_id IN' => $accessibleCompanyIds])
    ->all();
```

### Validate Resource Ownership

```php
$device = $this->Devices->get($id);

if (!in_array($device->company_id, $accessibleCompanyIds)) {
    throw new ForbiddenException('Access denied');
}
```

## Common Authorization Rules

### Superuser Check

```php
if ($user->is_superuser) {
    return true;  // Bypass all checks
}
```

### Admin Role Check

```php
if ($role === 'admin') {
    return true;  // System admin bypass
}

if ($role !== 'customer') {
    return false;  // Only customer and admin allowed
}
```

### Premium Customer (Distributor)

```php
// In WatmRules
public function orasesIsPremiumCustomer($user, $role, $request, $route_params = []): bool
{
    if ($role === 'admin') return true;
    if ($role !== 'customer') return false;

    // Get company and check can_upcharge
    $company = $this->getCompany($user);
    return $company->account_type->can_upcharge;
}
```

### Allow Upcharging

```php
// Requires BOTH capability AND setting
return $company->account_type->can_upcharge && $company->allow_upcharging;
```

### Is NOT Subcustomer

```php
return $company->account_type->title !== 'Subcustomer';
```

### Can View Pricing

```php
// Premium customers always see pricing
if ($company->account_type->title !== 'Subcustomer') {
    return true;
}

// Subcustomers only if allow_self_billing
return $company->allow_self_billing;
```

### Is Active Customer

```php
return $company->company_status->status_key === 'active';
```

### Is Child Company

```php
$passedCompanyId = $request->getParam('pass.0');

$childIds = $companiesTable
    ->find('children', ['for' => $userCompanyId, 'threads' => 'full'])
    ->select(['id'])
    ->all()
    ->extract('id')
    ->toArray();

return in_array($passedCompanyId, $childIds);
```

## Adding Permissions

### 1. Add to permissions.php

```php
[
    'prefix' => 'Admin',
    'plugin' => 'MyPlugin',
    'controller' => 'MyController',
    'action' => 'myAction',
    'role' => '*',
    'allowed' => new \App\Auth\Rules\WatmRules(),
],
```

### 2. Add Custom Rule (if needed)

```php
// In WatmRules.php
public function orasesMyCustomCheck($user, $role, $request, $route_params = []): bool
{
    if ($role === 'admin') return true;
    if ($role !== 'customer') return false;

    // Custom logic here
    return $someCondition;
}
```

### 3. Create Controller Action

```php
public function myAction()
{
    // Authorization already checked by middleware

    // Your logic here
}
```

### 4. Add UI Element

```php
<?php if ($this->IsAuthorized->isAuthorized([
    'prefix' => 'Admin',
    'plugin' => 'MyPlugin',
    'controller' => 'MyController',
    'action' => 'myAction'
])): ?>
    <?= $this->Html->link('My Action', ['action' => 'myAction']) ?>
<?php endif; ?>
```

## Database Queries

### Find User by ID

```php
$usersTable = TableRegistry::getTableLocator()->get('UserManagement.OUsers');
$user = $usersTable->get($userId);
```

### Find User with Company

```php
$companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
$companyUser = $companyUsersTable->find()
    ->contain(['Companies.AccountTypes', 'OUsers'])
    ->where(['o_user_id' => $userId])
    ->first();

$user = $companyUser->o_user;
$company = $companyUser->company;
```

### Find Company with Children

```php
$companiesTable = TableRegistry::getTableLocator()->get('Companies.Companies');

// Get all children
$children = $companiesTable
    ->find('children', ['for' => $companyId, 'threads' => 'full'])
    ->all();

// Get direct children only
$directChildren = $companiesTable
    ->find('children', ['for' => $companyId, 'direct' => true])
    ->all();
```

### Search Users by Role

```php
$usersTable = TableRegistry::getTableLocator()->get('UserManagement.OUsers');

// Find users with specific role
$users = $usersTable->find()
    ->where(['OUsers.additional_data LIKE' => '%"role_ids":"3"%'])
    ->all();

// Find superusers
$superusers = $usersTable->find()
    ->where(['OUsers.is_superuser' => 1])
    ->all();
```

## Role & Account Type Values

### Role Types
- `'admin'` - System administrator
- `'customer'` - Company user

### Standard Roles

| ID | Title | Role Type | Description |
|----|-------|-----------|-------------|
| 1 | Admin | admin | System administrator |
| 3 | Super Admin | customer | Company super admin |
| 4 | Admin | customer | Company admin (limited) |

### Account Types

| ID | Title | Can Upcharge | Can Assign | Sort Order |
|----|-------|--------------|------------|------------|
| 1 | Distributor | 1 | 1 | 10 |
| 2 | Customer | 0 | 1 | 20 |
| 3 | Subcustomer | 0 | 0 | 30 |

### Company Statuses

| Status Key | Description |
|-----------|-------------|
| `active` | Company is operational |
| `inactive` | Company is suspended |
| `pending` | Company not yet activated |

## Common Patterns

### Controller Helper Methods

```php
// In AppController
protected function getCompanyUser($user = null)
{
    if ($user === null) {
        $user = $this->Authentication->getIdentity();
    }

    $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
    return $companyUsersTable->find()
        ->contain(['Companies.AccountTypes', 'Companies.CompanyStatuses'])
        ->where(['o_user_id' => $user->id])
        ->first();
}

protected function getAccessibleCompanyIds($company = null)
{
    if ($company === null) {
        $companyUser = $this->getCompanyUser();
        $company = $companyUser->company;
    }

    $ids = [$company->id];

    if ($company->account_type->can_assign) {
        $companiesTable = TableRegistry::getTableLocator()->get('Companies.Companies');
        $childIds = $companiesTable
            ->find('children', ['for' => $company->id, 'threads' => 'full'])
            ->select(['id'])
            ->all()
            ->extract('id')
            ->toArray();

        $ids = array_merge($ids, $childIds);
    }

    return $ids;
}
```

### Index Action Pattern

```php
public function index()
{
    $query = $this->MyModel->find()
        ->where(['MyModel.company_id IN' => $this->getAccessibleCompanyIds()])
        ->contain([...]);

    $this->set('entities', $this->paginate($query));
}
```

### Edit Action Pattern

```php
public function edit($id = null)
{
    $entity = $this->MyModel->get($id);

    // Validate ownership
    if (!in_array($entity->company_id, $this->getAccessibleCompanyIds())) {
        throw new ForbiddenException('Access denied');
    }

    if ($this->request->is(['patch', 'post', 'put'])) {
        $entity = $this->MyModel->patchEntity($entity, $this->request->getData());

        if ($this->MyModel->save($entity)) {
            $this->Flash->success('Updated successfully.');
            return $this->redirect(['action' => 'view', $entity->id]);
        }

        $this->Flash->error('Unable to update.');
    }

    $this->set(compact('entity'));
}
```

## Troubleshooting

### Permission Denied (403)

**Check**:
1. User is authenticated
2. User has correct role (`additional_data['role_ids']`)
3. Company is active (`company_status.status_key === 'active'`)
4. Account type has required capability (`can_upcharge`, `can_assign`)
5. Resource belongs to accessible company

**Debug**:
```php
// In controller
debug($this->Authentication->getIdentity());
debug($this->getCompanyUser());
debug($this->getAccessibleCompanyIds());
debug($this->isAuthorized([...]));
```

### Cannot Access Sub-Company Resources

**Check**:
1. Account type has `can_assign = 1`
2. Company is actually a parent (has children)
3. Resource's `company_id` matches child company
4. Authorization rule uses `entityBelongsToCompanyOrSubCompany()`

### Subcustomer Cannot See Pricing

**Check**:
1. `company.allow_self_billing` flag is set
2. Authorization rule uses `orasesCanViewPricing()`
3. Template checks permission correctly

### User Cannot Edit Own Company

**Check**:
1. Rule uses `orasesIsCustomerSelf()` OR pattern
2. Permission configuration has correct route
3. `company_id` is passed as first URL parameter

## File Locations

### Configuration
- `watm/watm/config/permissions.php` - Permission rules
- `watm/watm/config/app_local.php` - Local environment config

### Authorization
- `watm/watm/src/Auth/Rules/WatmRules.php` - Main auth rules
- `plugins/UserManagement/src/Auth/Rbac/Rules/WATMCustomerRules.php` - Customer rules
- `plugins/UserManagement/src/Auth/Rbac/Rules/RmaRules.php` - RMA rules

### Models
- `plugins/UserManagement/src/Model/Table/OUsersTable.php` - Users
- `plugins/UserManagement/src/Model/Table/CompanyUsersTable.php` - User-company link
- `plugins/Companies/src/Model/Table/CompaniesTable.php` - Companies

### Middleware
- `plugins/UserManagement/src/Middleware/UserConfigurationMiddleware.php` - Auth config
- `plugins/UserAuthorization/src/Middleware/AuthorizationAwareMiddleware.php` - Auth prep
- `plugins/Companies/src/Middleware/ApiAuthenticationMiddleware.php` - API auth

### Seeds
- `watm/watm/config/Seeds/WATMRolesSeed.php` - Admin role
- `watm/watm/config/Seeds/CompanyUserRolesSeed.php` - Customer roles

## Useful Commands

### Database Seeds

```bash
# Run all seeds
bin/cake migrations seed --seed MasterSeed

# Run specific seed
bin/cake migrations seed --seed WATMRolesSeed
```

### Check Permissions Config

```bash
# Check syntax
php -l config/permissions.php
```

### Clear Cache

```bash
# Clear application cache
bin/cake cache clear_all
```

## Links to Detailed Documentation

- [01_Architecture_Overview.md](01_Architecture_Overview.md) - System architecture
- [02_Permission_Configuration.md](02_Permission_Configuration.md) - All permission rules
- [03_Authorization_Rules.md](03_Authorization_Rules.md) - All auth rule methods
- [04_Role_Definitions.md](04_Role_Definitions.md) - Roles and account types
- [05_Database_Schema.md](05_Database_Schema.md) - Database structure
- [06_Multi_Tenant_Architecture.md](06_Multi_Tenant_Architecture.md) - Multi-tenancy
- [07_Implementation_Guide.md](07_Implementation_Guide.md) - Developer guide
