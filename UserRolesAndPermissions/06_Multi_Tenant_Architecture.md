# Multi-Tenant Architecture

This document explains how the WATM platform implements multi-tenancy with company-based data isolation.

## Table of Contents
- [Overview](#overview)
- [Company-Based Isolation](#company-based-isolation)
- [Hierarchical Companies](#hierarchical-companies)
- [Data Filtering Patterns](#data-filtering-patterns)
- [Resource Ownership Validation](#resource-ownership-validation)
- [API Multi-Tenancy](#api-multi-tenancy)

## Overview

### Multi-Tenancy Model

WATM uses a **shared database, single application** multi-tenancy model where:
- All companies share the same database
- Data is logically isolated by `company_id` foreign key
- Authorization enforces company-based access control
- Each user belongs to exactly one company

### Key Principles

1. **User-Company Binding**: One user = one company (via `company_users` table)
2. **Hierarchical Tenancy**: Companies can have parent-child relationships
3. **Authorization-Based**: Access control enforced by custom rules, not database constraints
4. **Audit Logging**: All cross-company access attempts logged

## Company-Based Isolation

### User-Company Link

**Table**: `company_users`

**Relationship**: One-to-One (user to company)

```sql
CREATE TABLE company_users (
    o_user_id INT UNSIGNED PRIMARY KEY,  -- One user, one company
    company_id INT UNSIGNED NOT NULL,
    FOREIGN KEY (o_user_id) REFERENCES o_users(id),
    FOREIGN KEY (company_id) REFERENCES companies(id)
);
```

**Enforcement**:
```php
// Get user's company
$companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
$companyUser = $companyUsersTable->find()
    ->contain('Companies.AccountTypes')
    ->where(['o_user_id' => $user->id])
    ->first();

$company = $companyUser->company;
```

**Implications**:
- User cannot switch companies without admin intervention
- No direct many-to-many user-company relationship
- Company deletion requires removing all user associations first

### Resource Association

Most resources are associated with a company via `company_id`:

**Devices**:
```sql
CREATE TABLE devices (
    id INT UNSIGNED PRIMARY KEY,
    company_id INT UNSIGNED NOT NULL,
    -- other fields
    FOREIGN KEY (company_id) REFERENCES companies(id)
);
```

**Billing/Invoices**:
```sql
CREATE TABLE invoices (
    id INT UNSIGNED PRIMARY KEY,
    company_id INT UNSIGNED NOT NULL,
    -- other fields
    FOREIGN KEY (company_id) REFERENCES companies(id)
);
```

**Payment Methods**:
```sql
CREATE TABLE company_payment_methods (
    id INT UNSIGNED PRIMARY KEY,
    company_id INT UNSIGNED NOT NULL,
    -- other fields
    FOREIGN KEY (company_id) REFERENCES companies(id)
);
```

### Automatic Filtering

When a customer user accesses resources, filtering by company is automatic:

**Example Controller Action**:
```php
public function index()
{
    // Get current user's company
    $companyId = $this->request->getAttribute('identity')->getCompanyId();

    // Filter devices by company
    $devices = $this->Devices->find()
        ->where(['Devices.company_id' => $companyId])
        ->all();

    $this->set(compact('devices'));
}
```

**Authorization Check**:
```php
// In WatmRules::orasesMustBelongToCompany()
$deviceId = $request->getParam('pass.0');
$deviceEntity = $devicesTable->find()->where(['id' => $deviceId])->first();

// Check device belongs to user's company
if ($deviceEntity->company_id !== $company->id) {
    return false;  // Unauthorized
}

return true;  // Authorized
```

## Hierarchical Companies

### Parent-Child Relationships

**Database Field**: `companies.parent_company_id`

```sql
CREATE TABLE companies (
    id INT UNSIGNED PRIMARY KEY,
    parent_company_id INT UNSIGNED DEFAULT NULL,
    -- other fields
    FOREIGN KEY (parent_company_id) REFERENCES companies(id)
);
```

**Example Hierarchy**:
```
Company A (Distributor)
├── Company B (Customer)
│   ├── Company C (Subcustomer)
│   └── Company D (Subcustomer)
└── Company E (Customer)
```

### Tree Behavior

CakePHP's Tree behavior enables hierarchical queries:

**Find All Children**:
```php
$childCompanies = $companiesTable
    ->find('children', [
        'for' => $parentCompanyId,
        'threads' => 'full'
    ])
    ->all();
```

**Find Path to Root**:
```php
$path = $companiesTable
    ->find('path', ['for' => $companyId])
    ->all();
```

### Access Rules by Hierarchy

#### Distributor (can_assign = 1, can_upcharge = 1)

**Can Access**:
- Own company resources
- Direct children companies
- All descendant companies (grandchildren, etc.)

**Code**:
```php
// Get all accessible company IDs
$accessibleCompanyIds = [$company->id];  // Own company

if ($company->account_type->can_assign) {
    // Add all children
    $childIds = $companiesTable
        ->find('children', ['for' => $company->id])
        ->select(['Companies.id'])
        ->all()
        ->extract('id')
        ->toArray();

    $accessibleCompanyIds = array_merge($accessibleCompanyIds, $childIds);
}

// Filter resources
$devices = $devicesTable->find()
    ->where(['Devices.company_id IN' => $accessibleCompanyIds])
    ->all();
```

#### Customer (can_assign = 1, can_upcharge = 0)

**Can Access**:
- Own company resources
- Direct children companies (subcustomers)

**Restrictions**:
- Cannot view/edit upcharge settings
- Cannot manage service plan pricing

#### Subcustomer (can_assign = 0, can_upcharge = 0)

**Can Access**:
- Own company resources only
- Parent company (view-only, via WATMCustomerRules)

**Restrictions**:
- Cannot create sub-companies
- Cannot access sibling companies
- Pricing visibility controlled by `allow_self_billing`

### Hierarchy Validation Methods

#### entityBelongsToCompanyOrSubCompany

**Purpose**: Check if entity belongs to user's company or sub-companies

**Code**:
```php
private function entityBelongsToCompanyOrSubCompany($entity, $company)
{
    // Direct ownership
    if ($entity->company_id === $company->id) {
        return true;
    }

    // Check child companies if can_assign
    if ($company->account_type->can_assign) {
        $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
        $childCompanies = $companyUsersTable
            ->Companies
            ->find('children', [
                'for' => $company->id,
                'threads' => 'full',
            ])
            ->select(['Companies.id'])
            ->all()
            ->extract('id')
            ->toArray();

        if (in_array($entity->company_id, $childCompanies)) {
            return true;
        }
    }

    return false;
}
```

**File**: `src/Auth/Rules/WatmRules.php:168-196`

**Usage**:
```php
// In authorization rule
$deviceId = $request->getParam('pass.0');
$deviceEntity = $devicesTable->find()->where(['id' => $deviceId])->first();

if (!$this->entityBelongsToCompanyOrSubCompany($deviceEntity, $company)) {
    return false;  // Device doesn't belong to accessible companies
}

return true;
```

#### orasesIsChildCustomer

**Purpose**: Check if company in URL is a child of user's company

**Code**:
```php
public function orasesIsChildCustomer($user, $role, $request, $route_params = []): bool
{
    $isChild = false;
    $passedParams = $request->getParam('pass');

    if (!empty($passedParams) && $role == 'customer') {
        $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
        $companyUser = $companyUsersTable->find()
            ->where(['o_user_id' => $user->get('id')])
            ->first();

        $companyIds = $companyUsersTable->Companies
            ->find('children', [
                'for' => $companyUser->company_id,
                'threads' => 'full',
            ])
            ->where(['Companies.id IS' => $passedParams[0]])
            ->select(['Companies.id'])
            ->all()
            ->extract('id')
            ->toArray();

        if (in_array($passedParams[0], $companyIds)) {
            $isChild = true;
        }
    }

    return $isChild;
}
```

**File**: `src/Auth/Rules/WatmRules.php:459-486`

## Data Filtering Patterns

### Pattern 1: Controller-Level Filtering

**Scenario**: List all devices for current user

**Implementation**:
```php
public function index()
{
    // Get current user
    $user = $this->Authentication->getIdentity();

    // Get user's company
    $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
    $companyUser = $companyUsersTable->find()
        ->contain('Companies.AccountTypes')
        ->where(['o_user_id' => $user->id])
        ->first();

    $company = $companyUser->company;

    // Build accessible company IDs
    $accessibleCompanyIds = [$company->id];

    if ($company->account_type->can_assign) {
        $childIds = $this->Companies
            ->find('children', ['for' => $company->id])
            ->select(['Companies.id'])
            ->all()
            ->extract('id')
            ->toArray();

        $accessibleCompanyIds = array_merge($accessibleCompanyIds, $childIds);
    }

    // Filter devices
    $devices = $this->Devices->find()
        ->where(['Devices.company_id IN' => $accessibleCompanyIds])
        ->contain(['Companies', 'DeviceStatuses'])
        ->all();

    $this->set(compact('devices'));
}
```

### Pattern 2: Authorization-Level Filtering

**Scenario**: Editing a specific device

**Permission Configuration** (`config/permissions.php`):
```php
[
    'prefix' => 'Admin',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => 'edit',
    'role' => '*',
    'allowed' => new \App\Auth\Rules\WatmRules()
]
```

**Authorization Rule** (`WatmRules::orasesMustBelongToCompany()`):
```php
if ($controller === 'Devices' and $plugin === 'Devices') {
    $devicesTable = TableRegistry::getTableLocator()->get('Devices.Devices');
    $deviceId = $request->getParam('pass.0');

    if ($deviceId === null) {
        return true;  // No specific device, proceed
    }

    $deviceEntity = $devicesTable->find()->where(['id' => $deviceId])->first();

    if (!isset($deviceEntity)) {
        return true;  // Device doesn't exist, let it 404
    }

    return $this->entityBelongsToCompanyOrSubCompany($deviceEntity, $company);
}
```

**Controller Action**:
```php
public function edit($id = null)
{
    // Authorization already checked device ownership
    $device = $this->Devices->get($id);

    if ($this->request->is(['patch', 'post', 'put'])) {
        $device = $this->Devices->patchEntity($device, $this->request->getData());

        if ($this->Devices->save($device)) {
            $this->Flash->success('Device updated.');
            return $this->redirect(['action' => 'view', $id]);
        }

        $this->Flash->error('Unable to update device.');
    }

    $this->set(compact('device'));
}
```

### Pattern 3: Table-Level Filtering (Custom Finder)

**Scenario**: Reusable device filtering

**Table Method** (`DevicesTable.php`):
```php
public function findForCompany(Query $query, array $options)
{
    $companyId = $options['company_id'];
    $includeChildren = $options['include_children'] ?? false;

    $companyIds = [$companyId];

    if ($includeChildren) {
        $companiesTable = TableRegistry::getTableLocator()->get('Companies.Companies');
        $childIds = $companiesTable
            ->find('children', ['for' => $companyId])
            ->select(['Companies.id'])
            ->all()
            ->extract('id')
            ->toArray();

        $companyIds = array_merge($companyIds, $childIds);
    }

    return $query->where(['Devices.company_id IN' => $companyIds]);
}
```

**Usage**:
```php
// In controller
$devices = $this->Devices->find('forCompany', [
    'company_id' => $company->id,
    'include_children' => $company->account_type->can_assign
])->all();
```

### Pattern 4: Query Modification Behavior

**Scenario**: Automatic company filtering for all queries

**Behavior** (`CompanyFilterBehavior.php`):
```php
namespace App\Model\Behavior;

use Cake\ORM\Behavior;
use Cake\ORM\Query;

class CompanyFilterBehavior extends Behavior
{
    public function beforeFind(EventInterface $event, Query $query, ArrayObject $options, $primary)
    {
        // Skip if admin user or filter disabled
        if ($options['skipCompanyFilter'] ?? false) {
            return;
        }

        // Get current user's company ID
        $companyId = $this->getCurrentCompanyId();

        if ($companyId) {
            $query->where([$this->_table->aliasField('company_id') => $companyId]);
        }
    }

    protected function getCurrentCompanyId()
    {
        // Get from session, request, or identity
        // Implementation depends on how you store current company context
    }
}
```

**Table Configuration**:
```php
class DevicesTable extends Table
{
    public function initialize(array $config): void
    {
        parent::initialize($config);
        $this->addBehavior('CompanyFilter');
    }
}
```

**Usage**:
```php
// Automatically filtered by company
$devices = $this->Devices->find()->all();

// Skip filtering (admin use)
$allDevices = $this->Devices->find('all', ['skipCompanyFilter' => true])->all();
```

## Resource Ownership Validation

### Devices

**Ownership Check**:
```php
// WatmRules::orasesMustBelongToCompany()
if ($controller === 'Devices' and $plugin === 'Devices') {
    $deviceEntity = $devicesTable->find()->where(['id' => $deviceId])->first();
    return $this->entityBelongsToCompanyOrSubCompany($deviceEntity, $company);
}
```

### Payment Methods

**Ownership Check**:
```php
if ($controller === 'CompanyPaymentMethods') {
    $companyId = $request->getParam('pass.0');
    $companyEntity = $companiesTable->find()->where(['id' => $companyId])->first();
    return $this->companyBelongsToUsersCompanyOrSubCompany($companyEntity, $companyUser);
}
```

### Company Users (Employees)

**Ownership Check**:
```php
if ($controller === 'CompanyUsers') {
    $companyId = $request->getParam('pass.0');
    $companyEntity = $companiesTable->find()->where(['id' => $companyId])->first();
    return $this->companyBelongsToUsersCompanyOrSubCompany($companyEntity, $companyUser);
}
```

### Invoices

**Ownership Check**:
```php
// In InvoicesController
public function view($id = null)
{
    $invoice = $this->Invoices->get($id, [
        'contain' => ['Companies']
    ]);

    // Check invoice belongs to user's company
    $companyUser = $this->getCompanyUser();
    $company = $companyUser->company;

    $accessibleCompanyIds = $this->getAccessibleCompanyIds($company);

    if (!in_array($invoice->company_id, $accessibleCompanyIds)) {
        $this->Flash->error('You do not have permission to view this invoice.');
        return $this->redirect(['action' => 'index']);
    }

    $this->set(compact('invoice'));
}
```

### API Keys

**Special Handling**: API keys use key ID instead of company ID in URL

**Ownership Check**:
```php
// WatmRules::orasesMustBelongToCompany()
if ($controller === 'CompanyApiKeys') {
    $apiKeyIdActions = ['view', 'edit', 'awsUsage', 'syncToAws'];

    if (in_array($action, $apiKeyIdActions)) {
        $apiKeyId = $request->getParam('pass.0');
        $apiKeyEntity = $apiKeysTable->find()
            ->where(['CompanyApiKeys.id' => $apiKeyId])
            ->first();

        return $this->entityBelongsToCompanyOrSubCompany($apiKeyEntity, $company);
    }
}
```

## API Multi-Tenancy

### API Key Authentication

**Middleware**: `Companies\Middleware\ApiAuthenticationMiddleware`

**Process**:
1. Extract `X-API-Key` header
2. Validate API key in database
3. Load associated company
4. Set request attributes with company context

**Code**:
```php
// ApiAuthenticationMiddleware::process()
$apiKey = $request->getHeaderLine('X-API-Key');

if (empty($apiKey)) {
    throw new UnauthorizedException('API key required');
}

// Validate API key
$apiKeysTable = TableRegistry::getTableLocator()->get('Companies.CompanyApiKeys');
$apiKeyEntity = $apiKeysTable->validateApiKey($apiKey);

if (!$apiKeyEntity) {
    throw new UnauthorizedException('Invalid API key');
}

// Check expiration
if ($apiKeyEntity->expires && $apiKeyEntity->expires < new DateTime()) {
    throw new UnauthorizedException('API key expired');
}

// Check active status
if (!$apiKeyEntity->is_active) {
    throw new UnauthorizedException('API key inactive');
}

// Load company
$company = $apiKeyEntity->company;

// Set request attributes
$request = $request
    ->withAttribute('apiKey', $apiKeyEntity)
    ->withAttribute('company', $company)
    ->withAttribute('companyId', $company->id);

return $handler->handle($request);
```

**File**: `plugins/Companies/src/Middleware/ApiAuthenticationMiddleware.php`

### API Company Context

**Controller Access**:
```php
public function index()
{
    // Get company from API key
    $company = $this->request->getAttribute('company');
    $companyId = $this->request->getAttribute('companyId');

    // Filter devices by API key's company
    $devices = $this->Devices->find()
        ->where(['Devices.company_id' => $companyId])
        ->all();

    $this->set([
        'devices' => $devices,
        '_serialize' => ['devices']
    ]);
}
```

### API Permissions

**Configuration** (`config/permissions.php`):
```php
[
    'prefix' => 'Api/V1',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => '*',
    'bypassAuth' => true,  // Uses API key instead of session
]
```

**Company Filtering**: Automatic via middleware-set company context

## Multi-Tenancy Best Practices

### 1. Always Validate Ownership

Even with authorization, validate resource ownership in controller:
```php
public function edit($id)
{
    $device = $this->Devices->get($id);
    $accessibleIds = $this->getAccessibleCompanyIds();

    if (!in_array($device->company_id, $accessibleIds)) {
        throw new ForbiddenException('Access denied');
    }

    // Continue with edit
}
```

### 2. Use Helper Methods

Create reusable methods for common checks:
```php
protected function getAccessibleCompanyIds()
{
    $companyUser = $this->getCompanyUser();
    $company = $companyUser->company;

    $ids = [$company->id];

    if ($company->account_type->can_assign) {
        $childIds = $this->Companies
            ->find('children', ['for' => $company->id])
            ->select(['id'])
            ->all()
            ->extract('id')
            ->toArray();

        $ids = array_merge($ids, $childIds);
    }

    return $ids;
}
```

### 3. Filter Lists Automatically

Always filter list views by company:
```php
public function index()
{
    $query = $this->Devices->find()
        ->where(['company_id IN' => $this->getAccessibleCompanyIds()]);

    $this->set('devices', $this->paginate($query));
}
```

### 4. Test Cross-Company Access

Write tests that verify users cannot access other companies' data:
```php
public function testCannotAccessOtherCompanyDevice()
{
    $this->loginAsCustomerUser(1);  // Company A user

    // Try to access Company B device
    $this->get('/admin/devices/devices/edit/999');

    $this->assertResponseCode(403);
}
```

### 5. Log Suspicious Activity

Log failed cross-company access attempts:
```php
if (!$this->isAuthorized()) {
    $this->logEvent('Unauthorized cross-company access attempt', 'security', 'warning', [
        'user_id' => $user->id,
        'user_company_id' => $userCompany->id,
        'attempted_resource_id' => $deviceId,
        'resource_company_id' => $device->company_id
    ]);

    throw new ForbiddenException();
}
```

### 6. Consider Soft Deletes

Use soft deletes (`deleted` timestamp) instead of hard deletes:
```php
public function delete($id)
{
    $device = $this->Devices->get($id);
    $device->deleted = new DateTime();

    if ($this->Devices->save($device)) {
        $this->Flash->success('Device deleted.');
    }

    return $this->redirect(['action' => 'index']);
}
```

### 7. Document Hierarchy Limits

Set practical limits on company hierarchy depth:
```php
public function add()
{
    $company = $this->Companies->newEmptyEntity();

    if ($this->request->is('post')) {
        $data = $this->request->getData();

        // Check hierarchy depth
        if ($this->getHierarchyDepth($data['parent_company_id']) >= 3) {
            $this->Flash->error('Maximum hierarchy depth reached (3 levels).');
            return;
        }

        // Continue with save
    }
}
```

## Next Steps

- See [07_Implementation_Guide.md](07_Implementation_Guide.md) for practical development examples
- See [05_Database_Schema.md](05_Database_Schema.md) for database structure details
