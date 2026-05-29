# Implementation Guide

This guide provides practical examples for developers working with the WATM roles and permissions system.

## Table of Contents
- [Checking Permissions](#checking-permissions)
- [Adding New Permissions](#adding-new-permissions)
- [Creating Custom Authorization Rules](#creating-custom-authorization-rules)
- [Multi-Tenant Data Access](#multi-tenant-data-access)
- [Template Permissions](#template-permissions)
- [Testing Permissions](#testing-permissions)

## Checking Permissions

### In Controllers

#### Method 1: Using IsAuthorizedTrait

**Import Trait**:
```php
use CakeDC\Auth\Traits\IsAuthorizedTrait;

class MyController extends AppController
{
    use IsAuthorizedTrait;

    public function edit($id)
    {
        // Check if user can edit
        if (!$this->isAuthorized([
            'prefix' => 'Admin',
            'plugin' => 'MyPlugin',
            'controller' => 'MyController',
            'action' => 'edit',
        ])) {
            $this->Flash->error('You do not have permission to edit.');
            return $this->redirect(['action' => 'index']);
        }

        // Continue with edit logic
        $entity = $this->MyModel->get($id);
        // ...
    }
}
```

**File**: Any controller extending `AppController`

#### Method 2: beforeFilter Hook

**Use Case**: Restrict entire action before execution

```php
public function beforeFilter(EventInterface $event)
{
    parent::beforeFilter($event);

    // Restrict specific action
    if ($this->request->getParam('action') === 'dangerousAction') {
        if (!$this->isAuthorized([
            'prefix' => 'Admin',
            'plugin' => 'MyPlugin',
            'controller' => 'MyController',
            'action' => 'dangerousAction',
        ])) {
            throw new ForbiddenException('Access denied');
        }
    }
}
```

#### Method 3: Conditional Logic

**Use Case**: Show different data based on permissions

```php
public function view($id)
{
    $entity = $this->MyModel->get($id);

    // Check if user can see sensitive data
    $canViewSensitive = $this->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'MyPlugin',
        'controller' => 'MyController',
        'action' => 'viewSensitive',
    ]);

    if ($canViewSensitive) {
        $entity = $this->MyModel->get($id, [
            'contain' => ['SensitiveData']
        ]);
    }

    $this->set(compact('entity', 'canViewSensitive'));
}
```

### In Templates

#### Method 1: Using IsAuthorized Helper

**Helper**: Automatically available in all templates

```php
<?php
// Check permission for delete action
if ($this->IsAuthorized->isAuthorized([
    'prefix' => 'Admin',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => 'delete'
])): ?>
    <?= $this->Form->postLink(
        'Delete',
        ['action' => 'delete', $device->id],
        ['confirm' => 'Are you sure?', 'class' => 'btn btn-danger']
    ) ?>
<?php endif; ?>
```

#### Method 2: Variable from Controller

**Controller**:
```php
public function view($id)
{
    $device = $this->Devices->get($id);

    $canEdit = $this->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'Devices',
        'controller' => 'Devices',
        'action' => 'edit',
    ]);

    $canDelete = $this->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'Devices',
        'controller' => 'Devices',
        'action' => 'delete',
    ]);

    $this->set(compact('device', 'canEdit', 'canDelete'));
}
```

**Template**:
```php
<div class="actions">
    <?php if ($canEdit): ?>
        <?= $this->Html->link('Edit', ['action' => 'edit', $device->id], ['class' => 'btn btn-primary']) ?>
    <?php endif; ?>

    <?php if ($canDelete): ?>
        <?= $this->Form->postLink('Delete', ['action' => 'delete', $device->id], ['confirm' => 'Are you sure?', 'class' => 'btn btn-danger']) ?>
    <?php endif; ?>
</div>
```

## Adding New Permissions

### Step 1: Define Permission Rule

**File**: `config/permissions.php`

**Example**: Add permission for exporting devices

```php
// In config/permissions.php
return [
    'CakeDC/Auth.permissions' => [
        // ... existing rules ...

        // New permission for device export
        [
            'prefix' => 'Admin',
            'plugin' => 'Devices',
            'controller' => 'Devices',
            'action' => 'export',
            'role' => '*',
            'allowed' => new \App\Auth\Rules\WatmRules(),
        ],

        // Catch-all rule (must be last)
        [
            'role' => '*',
            'controller' => '*',
            'plugin' => '*',
            'prefix' => '*',
            'action' => '*',
            'allowed' => new \App\Auth\Rules\WatmRules()
        ],
    ]
];
```

### Step 2: Create Custom Authorization Method (if needed)

**File**: `src/Auth/Rules/WatmRules.php`

**Add Method**:
```php
/**
 * Check if user can export device data
 *
 * @param OUser|string $user User Object or User ID
 * @param string $role User role
 * @param ServerRequestInterface|array $request Current request
 * @param array $route_params Optional route parameters
 * @return bool
 */
public function orasesCanExportDevices($user, string $role, $request, $route_params = []): bool
{
    // Admin users can always export
    if ($role === 'admin') {
        return true;
    }

    // Only customer role can proceed
    if ($role !== 'customer') {
        return false;
    }

    // Get user's company
    $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
    $companyUser = $companyUsersTable->find()
        ->contain('Companies')
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser)) {
        return false;
    }

    $company = $companyUser->get('company');

    // Check if company has export capability
    // This could be a flag in additional_data or a specific setting
    if (isset($company->additional_data['can_export_data'])) {
        return $company->additional_data['can_export_data'] === true;
    }

    // Default: allow export for all customers
    return true;
}
```

### Step 3: Create Controller Action

**File**: `plugins/Devices/src/Controller/Admin/DevicesController.php`

```php
public function export()
{
    // Authorization is already checked by middleware

    // Get accessible company IDs
    $user = $this->Authentication->getIdentity();
    $companyUser = $this->getCompanyUser($user);
    $company = $companyUser->company;

    $accessibleCompanyIds = $this->getAccessibleCompanyIds($company);

    // Get devices for export
    $devices = $this->Devices->find()
        ->where(['Devices.company_id IN' => $accessibleCompanyIds])
        ->contain(['Companies', 'DeviceStatuses'])
        ->all();

    // Build CSV
    $csv = $this->buildCsv($devices);

    // Send response
    $response = $this->response
        ->withType('text/csv')
        ->withDownload('devices_' . date('Y-m-d') . '.csv')
        ->withStringBody($csv);

    return $response;
}

protected function buildCsv($devices)
{
    $handle = fopen('php://temp', 'r+');

    // Header row
    fputcsv($handle, ['ID', 'Serial Number', 'Company', 'Status', 'Created']);

    // Data rows
    foreach ($devices as $device) {
        fputcsv($handle, [
            $device->id,
            $device->serial_number,
            $device->company->title,
            $device->device_status->title,
            $device->created->format('Y-m-d H:i:s'),
        ]);
    }

    rewind($handle);
    $csv = stream_get_contents($handle);
    fclose($handle);

    return $csv;
}
```

### Step 4: Add UI Element

**File**: `plugins/Devices/templates/Admin/Devices/index.php`

```php
<div class="actions">
    <?php if ($this->IsAuthorized->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'Devices',
        'controller' => 'Devices',
        'action' => 'export'
    ])): ?>
        <?= $this->Html->link(
            'Export to CSV',
            ['action' => 'export'],
            ['class' => 'btn btn-success']
        ) ?>
    <?php endif; ?>
</div>
```

## Creating Custom Authorization Rules

### Simple Rule (Boolean Logic)

**Scenario**: Allow action only for primary users

**Add to WatmRules.php**:
```php
public function orasesIsPrimaryUser($user, string $role, $request, $route_params = []): bool
{
    if ($role === 'admin') {
        return true;  // Admins bypass
    }

    if ($role !== 'customer') {
        return false;
    }

    $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
    $companyUser = $companyUsersTable->find()
        ->contain('Companies')
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser)) {
        return false;
    }

    $company = $companyUser->get('company');

    // Check if user is primary user of company
    return $company->primary_user_id === $user->id;
}
```

**Add to permissions.php**:
```php
[
    'prefix' => 'Admin',
    'plugin' => 'Companies',
    'controller' => 'Companies',
    'action' => 'changePrimaryUser',
    'role' => '*',
    'allowed' => new \App\Auth\Rules\WatmRules(),
    // Will call orasesIsPrimaryUser() method
],
```

### Complex Rule (Multiple Conditions)

**Scenario**: Allow editing service plans only for active distributors with upcharge capability

**Add to WatmRules.php**:
```php
public function orasesCanEditServicePlans($user, string $role, $request, $route_params = []): bool
{
    if ($role === 'admin') {
        return true;
    }

    if ($role !== 'customer') {
        return false;
    }

    $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
    $companyUser = $companyUsersTable->find()
        ->contain(['Companies.AccountTypes', 'Companies.CompanyStatuses'])
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser)) {
        return false;
    }

    $company = $companyUser->get('company');

    // Check multiple conditions
    $isDistributor = $company->account_type->can_upcharge;
    $isActive = $company->company_status->status_key === 'active';
    $canUpcharge = $company->allow_upcharging;

    return $isDistributor && $isActive && $canUpcharge;
}
```

### Rule with Resource Validation

**Scenario**: Allow editing service plan only if it belongs to user's company hierarchy

**Add to WatmRules.php**:
```php
public function orasesCanEditServicePlan($user, string $role, $request, $route_params = []): bool
{
    if ($role === 'admin') {
        return true;
    }

    if ($role !== 'customer') {
        return false;
    }

    // Get user's company
    $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
    $companyUser = $companyUsersTable->find()
        ->contain('Companies.AccountTypes')
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser)) {
        return false;
    }

    $company = $companyUser->get('company');

    // Check if user is distributor
    if (!$company->account_type->can_upcharge) {
        return false;
    }

    // Get service plan ID from URL
    $servicePlanId = $request->getParam('pass.0');

    if ($servicePlanId === null) {
        return true;  // Creating new service plan
    }

    // Load service plan
    $servicePlansTable = TableRegistry::getTableLocator()->get('Devices.CompanyServicePlans');
    $servicePlan = $servicePlansTable->find()
        ->where(['CompanyServicePlans.id' => $servicePlanId])
        ->contain(['Companies'])
        ->first();

    if (!isset($servicePlan)) {
        return true;  // Let it 404
    }

    // Check if service plan's company belongs to user's hierarchy
    return $this->companyBelongsToUsersCompanyOrSubCompany($servicePlan->company, $companyUser);
}
```

## Multi-Tenant Data Access

### Getting User's Company

**Helper Method** (add to AppController):
```php
protected function getCompanyUser($user = null)
{
    if ($user === null) {
        $user = $this->Authentication->getIdentity();
    }

    $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
    $companyUser = $companyUsersTable->find()
        ->contain(['Companies.AccountTypes', 'Companies.CompanyStatuses'])
        ->where(['o_user_id' => $user->id])
        ->first();

    return $companyUser;
}
```

### Getting Accessible Company IDs

**Helper Method** (add to AppController):
```php
protected function getAccessibleCompanyIds($company = null)
{
    if ($company === null) {
        $companyUser = $this->getCompanyUser();
        $company = $companyUser->company;
    }

    $ids = [$company->id];

    // Add child companies if can_assign
    if ($company->account_type->can_assign) {
        $companiesTable = TableRegistry::getTableLocator()->get('Companies.Companies');
        $childIds = $companiesTable
            ->find('children', ['for' => $company->id, 'threads' => 'full'])
            ->select(['Companies.id'])
            ->all()
            ->extract('id')
            ->toArray();

        $ids = array_merge($ids, $childIds);
    }

    return $ids;
}
```

### Filtering Lists by Company

**Controller Action**:
```php
public function index()
{
    // Get accessible company IDs
    $accessibleCompanyIds = $this->getAccessibleCompanyIds();

    // Build query with company filter
    $query = $this->Devices->find()
        ->where(['Devices.company_id IN' => $accessibleCompanyIds])
        ->contain(['Companies', 'DeviceStatuses'])
        ->order(['Devices.created' => 'DESC']);

    // Add search conditions
    if ($this->request->getQuery('serial_number')) {
        $query->where([
            'Devices.serial_number LIKE' => '%' . $this->request->getQuery('serial_number') . '%'
        ]);
    }

    $devices = $this->paginate($query);
    $this->set(compact('devices'));
}
```

### Validating Resource Ownership

**Controller Action**:
```php
public function edit($id = null)
{
    $device = $this->Devices->get($id);

    // Validate device belongs to accessible companies
    $accessibleCompanyIds = $this->getAccessibleCompanyIds();

    if (!in_array($device->company_id, $accessibleCompanyIds)) {
        $this->Flash->error('You do not have permission to edit this device.');
        return $this->redirect(['action' => 'index']);
    }

    if ($this->request->is(['patch', 'post', 'put'])) {
        $device = $this->Devices->patchEntity($device, $this->request->getData());

        // Ensure company_id doesn't change to unauthorized company
        if ($device->isDirty('company_id') &&
            !in_array($device->company_id, $accessibleCompanyIds)) {
            $this->Flash->error('Invalid company selected.');
            return;
        }

        if ($this->Devices->save($device)) {
            $this->Flash->success('Device updated.');
            return $this->redirect(['action' => 'view', $device->id]);
        }

        $this->Flash->error('Unable to update device.');
    }

    // Get list of companies for form dropdown
    $companies = $this->Devices->Companies->find('list')
        ->where(['Companies.id IN' => $accessibleCompanyIds])
        ->order(['Companies.title' => 'ASC'])
        ->all();

    $this->set(compact('device', 'companies'));
}
```

## Template Permissions

### Conditional Buttons

**Example**: Show/hide action buttons based on permissions

```php
<div class="btn-group">
    <?php if ($this->IsAuthorized->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'Devices',
        'controller' => 'Devices',
        'action' => 'edit'
    ])): ?>
        <?= $this->Html->link(
            '<i class="icon-pencil"></i> Edit',
            ['action' => 'edit', $device->id],
            ['class' => 'btn btn-primary', 'escape' => false]
        ) ?>
    <?php endif; ?>

    <?php if ($this->IsAuthorized->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'Devices',
        'controller' => 'Devices',
        'action' => 'delete'
    ])): ?>
        <?= $this->Form->postLink(
            '<i class="icon-trash"></i> Delete',
            ['action' => 'delete', $device->id],
            [
                'confirm' => 'Are you sure you want to delete this device?',
                'class' => 'btn btn-danger',
                'escape' => false
            ]
        ) ?>
    <?php endif; ?>
</div>
```

### Conditional Menu Items

**Example**: Navigation menu with permission checks

```php
<ul class="nav">
    <li>
        <?= $this->Html->link('Dashboard', [
            'prefix' => 'Admin',
            'plugin' => 'Dashboard',
            'controller' => 'Dashboard',
            'action' => 'index'
        ]) ?>
    </li>

    <?php if ($this->IsAuthorized->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'Devices',
        'controller' => 'Devices',
        'action' => 'index'
    ])): ?>
        <li>
            <?= $this->Html->link('Devices', [
                'prefix' => 'Admin',
                'plugin' => 'Devices',
                'controller' => 'Devices',
                'action' => 'index'
            ]) ?>
        </li>
    <?php endif; ?>

    <?php if ($this->IsAuthorized->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'Companies',
        'controller' => 'Companies',
        'action' => 'index'
    ])): ?>
        <li>
            <?= $this->Html->link('Companies', [
                'prefix' => 'Admin',
                'plugin' => 'Companies',
                'controller' => 'Companies',
                'action' => 'index'
            ]) ?>
        </li>
    <?php endif; ?>
</ul>
```

### Conditional Content Sections

**Example**: Show pricing section only if user can view pricing

```php
<?php if ($this->IsAuthorized->isAuthorized([
    'prefix' => 'Admin',
    'plugin' => 'Billing',
    'controller' => 'Invoices',
    'action' => 'index'
])): ?>
    <div class="pricing-section">
        <h3>Pricing Information</h3>
        <table class="table">
            <tr>
                <td>Monthly Fee:</td>
                <td>$<?= h($device->monthly_fee) ?></td>
            </tr>
            <tr>
                <td>Data Overage Rate:</td>
                <td>$<?= h($device->data_overage_rate) ?>/GB</td>
            </tr>
        </table>
    </div>
<?php else: ?>
    <div class="alert alert-info">
        Contact your account administrator for pricing information.
    </div>
<?php endif; ?>
```

## Testing Permissions

### Unit Test Example

**File**: `tests/TestCase/Auth/Rules/WatmRulesTest.php`

```php
namespace App\Test\TestCase\Auth\Rules;

use App\Auth\Rules\WatmRules;
use Cake\ORM\TableRegistry;
use Cake\TestSuite\TestCase;

class WatmRulesTest extends TestCase
{
    protected $fixtures = [
        'app.OUsers',
        'app.Companies',
        'app.CompanyUsers',
        'app.AccountTypes',
    ];

    public function setUp(): void
    {
        parent::setUp();
        $this->WatmRules = new WatmRules();
        $this->OUsers = TableRegistry::getTableLocator()->get('UserManagement.OUsers');
        $this->Companies = TableRegistry::getTableLocator()->get('Companies.Companies');
        $this->CompanyUsers = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
    }

    public function testOrasesIsPremiumCustomer()
    {
        // Get distributor user
        $user = $this->OUsers->get(1);
        $role = 'customer';
        $request = $this->createMockRequest([]);

        // Should return true for distributor
        $result = $this->WatmRules->orasesIsPremiumCustomer($user, $role, $request, []);
        $this->assertTrue($result);
    }

    public function testOrasesIsNotPremiumCustomer()
    {
        // Get subcustomer user
        $user = $this->OUsers->get(3);
        $role = 'customer';
        $request = $this->createMockRequest([]);

        // Should return false for subcustomer
        $result = $this->WatmRules->orasesIsPremiumCustomer($user, $role, $request, []);
        $this->assertFalse($result);
    }

    public function testAdminBypassesCheck()
    {
        // Any user with admin role
        $user = $this->OUsers->get(2);
        $role = 'admin';
        $request = $this->createMockRequest([]);

        // Should return true for admin role
        $result = $this->WatmRules->orasesIsPremiumCustomer($user, $role, $request, []);
        $this->assertTrue($result);
    }

    protected function createMockRequest($params)
    {
        $request = $this->getMockBuilder('Cake\Http\ServerRequest')
            ->getMock();

        $request->method('getParam')
            ->willReturnCallback(function ($key) use ($params) {
                return $params[$key] ?? null;
            });

        return $request;
    }
}
```

### Integration Test Example

**File**: `tests/TestCase/Controller/DevicesControllerTest.php`

```php
namespace App\Test\TestCase\Controller;

use Cake\TestSuite\IntegrationTestTrait;
use Cake\TestSuite\TestCase;

class DevicesControllerTest extends TestCase
{
    use IntegrationTestTrait;

    protected $fixtures = [
        'app.OUsers',
        'app.Companies',
        'app.CompanyUsers',
        'app.Devices',
        'app.AccountTypes',
    ];

    public function setUp(): void
    {
        parent::setUp();
        $this->enableCsrfToken();
        $this->enableSecurityToken();
    }

    public function testCustomerCanViewOwnDevice()
    {
        // Login as Company A customer
        $this->session(['Auth' => ['id' => 1]]);

        // Access Company A device
        $this->get('/admin/devices/devices/view/1');

        $this->assertResponseOk();
        $this->assertResponseContains('Device Details');
    }

    public function testCustomerCannotViewOtherCompanyDevice()
    {
        // Login as Company A customer
        $this->session(['Auth' => ['id' => 1]]);

        // Try to access Company B device
        $this->get('/admin/devices/devices/view/99');

        $this->assertResponseCode(403);
    }

    public function testDistributorCanViewChildCompanyDevice()
    {
        // Login as distributor user
        $this->session(['Auth' => ['id' => 2]]);

        // Access child company device
        $this->get('/admin/devices/devices/view/50');

        $this->assertResponseOk();
    }

    public function testSubcustomerCannotEditDevice()
    {
        // Login as subcustomer user
        $this->session(['Auth' => ['id' => 3]]);

        // Try to edit device
        $this->get('/admin/devices/devices/edit/1');

        $this->assertResponseCode(403);
    }
}
```

## Best Practices

### 1. Use Descriptive Permission Names

**Good**:
```php
$this->isAuthorized([
    'prefix' => 'Admin',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => 'export'
])
```

**Bad**:
```php
$this->isAuthorized([
    'prefix' => 'Admin',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => 'x'
])
```

### 2. Check Permissions Early

**Good**:
```php
public function edit($id)
{
    // Check permission first
    if (!$this->isAuthorized([...])) {
        throw new ForbiddenException();
    }

    // Then do expensive operations
    $device = $this->Devices->get($id, ['contain' => [...]]);
    // ...
}
```

**Bad**:
```php
public function edit($id)
{
    // Expensive operation first
    $device = $this->Devices->get($id, ['contain' => [...]]);

    // Check permission later
    if (!$this->isAuthorized([...])) {
        throw new ForbiddenException();
    }
}
```

### 3. Always Validate Resource Ownership

**Good**:
```php
public function edit($id)
{
    $device = $this->Devices->get($id);

    // Double-check ownership even with authorization
    if (!in_array($device->company_id, $this->getAccessibleCompanyIds())) {
        throw new ForbiddenException();
    }

    // Continue...
}
```

### 4. Log Permission Failures

**Good**:
```php
public function edit($id)
{
    if (!$this->isAuthorized([...])) {
        $this->logEvent('Permission denied for device edit', 'security', 'warning', [
            'device_id' => $id,
            'user_id' => $this->Authentication->getIdentity()->id
        ]);

        throw new ForbiddenException();
    }
}
```

### 5. Test Permission Edge Cases

Always test:
- Admin users
- Different customer roles
- Different account types
- Cross-company access attempts
- Hierarchy permissions

## Next Steps

- See [08_Quick_Reference.md](08_Quick_Reference.md) for a quick reference cheat sheet
- See [03_Authorization_Rules.md](03_Authorization_Rules.md) for all available rule methods
- See [02_Permission_Configuration.md](02_Permission_Configuration.md) for permission configuration reference
