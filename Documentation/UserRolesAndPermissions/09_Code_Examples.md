# Code Examples

Real-world code examples from the WATM codebase demonstrating permission usage.

## Table of Contents
- [Controller Examples](#controller-examples)
- [Template Examples](#template-examples)
- [Model Examples](#model-examples)
- [Custom Rule Examples](#custom-rule-examples)
- [Helper Method Examples](#helper-method-examples)

## Controller Examples

### Example 1: Device List with Company Filtering

**File**: `plugins/Devices/src/Controller/Admin/DevicesController.php`

```php
public function index()
{
    // Get current user and their company
    $user = $this->Authentication->getIdentity();

    $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
    $companyUser = $companyUsersTable->find()
        ->contain(['Companies.AccountTypes'])
        ->where(['o_user_id' => $user->id])
        ->first();

    if (!$companyUser) {
        $this->Flash->error('User not associated with a company.');
        return $this->redirect(['controller' => 'Dashboard', 'action' => 'index']);
    }

    $company = $companyUser->company;

    // Build list of accessible company IDs
    $accessibleCompanyIds = [$company->id];

    // If can_assign, include child companies
    if ($company->account_type->can_assign) {
        $childIds = $this->Devices->Companies
            ->find('children', [
                'for' => $company->id,
                'threads' => 'full'
            ])
            ->select(['Companies.id'])
            ->all()
            ->extract('id')
            ->toArray();

        $accessibleCompanyIds = array_merge($accessibleCompanyIds, $childIds);
    }

    // Build query with company filter
    $query = $this->Devices->find()
        ->where(['Devices.company_id IN' => $accessibleCompanyIds])
        ->contain([
            'Companies',
            'DeviceStatuses',
            'ConfigGroups',
            'Carriers'
        ])
        ->order(['Devices.created' => 'DESC']);

    // Add search filters from query string
    if ($this->request->getQuery('serial_number')) {
        $query->where([
            'Devices.serial_number LIKE' => '%' . $this->request->getQuery('serial_number') . '%'
        ]);
    }

    if ($this->request->getQuery('company_id')) {
        $query->where(['Devices.company_id' => $this->request->getQuery('company_id')]);
    }

    if ($this->request->getQuery('status_id')) {
        $query->where(['Devices.device_status_id' => $this->request->getQuery('status_id')]);
    }

    // Paginate results
    $devices = $this->paginate($query);

    // Get companies for filter dropdown
    $companies = $this->Devices->Companies->find('list')
        ->where(['Companies.id IN' => $accessibleCompanyIds])
        ->order(['Companies.title' => 'ASC'])
        ->all();

    // Get statuses for filter dropdown
    $deviceStatuses = $this->Devices->DeviceStatuses->find('list')
        ->order(['DeviceStatuses.title' => 'ASC'])
        ->all();

    $this->set(compact('devices', 'companies', 'deviceStatuses'));
}
```

### Example 2: Device Edit with Ownership Validation

**File**: `plugins/Devices/src/Controller/Admin/DevicesController.php`

```php
public function edit($id = null)
{
    // Get device
    $device = $this->Devices->get($id, [
        'contain' => ['Companies', 'ConfigGroups', 'DeviceStatuses']
    ]);

    // Get user's accessible company IDs
    $accessibleCompanyIds = $this->getAccessibleCompanyIds();

    // Validate device belongs to accessible companies
    if (!in_array($device->company_id, $accessibleCompanyIds)) {
        $this->Flash->error('You do not have permission to edit this device.');
        $this->logEvent('Unauthorized device edit attempt', 'security', 'warning', [
            'device_id' => $device->id,
            'device_company_id' => $device->company_id,
            'user_id' => $this->Authentication->getIdentity()->id
        ]);
        return $this->redirect(['action' => 'index']);
    }

    if ($this->request->is(['patch', 'post', 'put'])) {
        $device = $this->Devices->patchEntity($device, $this->request->getData());

        // Validate company_id hasn't changed to unauthorized company
        if ($device->isDirty('company_id')) {
            if (!in_array($device->company_id, $accessibleCompanyIds)) {
                $this->Flash->error('Invalid company selected.');
                $device->setError('company_id', 'You do not have access to the selected company.');
            }
        }

        if (!$device->getErrors() && $this->Devices->save($device)) {
            $this->Flash->success('Device has been updated.');
            return $this->redirect(['action' => 'view', $device->id]);
        }

        $this->Flash->error('Unable to update the device. Please check the form for errors.');
    }

    // Get dropdown options (filtered by accessible companies)
    $companies = $this->Devices->Companies->find('list')
        ->where(['Companies.id IN' => $accessibleCompanyIds])
        ->order(['Companies.title' => 'ASC'])
        ->all();

    $configGroups = $this->Devices->ConfigGroups->find('list')
        ->order(['ConfigGroups.title' => 'ASC'])
        ->all();

    $deviceStatuses = $this->Devices->DeviceStatuses->find('list')
        ->order(['DeviceStatuses.title' => 'ASC'])
        ->all();

    $this->set(compact('device', 'companies', 'configGroups', 'deviceStatuses'));
}

/**
 * Helper method to get accessible company IDs for current user
 *
 * @return array
 */
protected function getAccessibleCompanyIds(): array
{
    $user = $this->Authentication->getIdentity();

    $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
    $companyUser = $companyUsersTable->find()
        ->contain(['Companies.AccountTypes'])
        ->where(['o_user_id' => $user->id])
        ->first();

    if (!$companyUser) {
        return [];
    }

    $company = $companyUser->company;
    $ids = [$company->id];

    if ($company->account_type->can_assign) {
        $childIds = $this->Devices->Companies
            ->find('children', [
                'for' => $company->id,
                'threads' => 'full'
            ])
            ->select(['Companies.id'])
            ->all()
            ->extract('id')
            ->toArray();

        $ids = array_merge($ids, $childIds);
    }

    return $ids;
}
```

### Example 3: Company View with Custom Rule

**File**: `plugins/Companies/src/Controller/Admin/CompaniesController.php`

```php
public function view($id = null)
{
    // Authorization checked by WATMCustomerRules in permissions.php
    // Customer users can view:
    // - Their own company
    // - Their parent company (if subcustomer)
    // - Their child companies (if can_assign)

    $company = $this->Companies->get($id, [
        'contain' => [
            'AccountTypes',
            'CompanyStatuses',
            'ParentCompanies',
            'ChildCompanies',
            'PrimaryUsers',
            'CompanyUsers.OUsers',
            'Devices',
            'Invoices' => [
                'sort' => ['Invoices.created' => 'DESC'],
                'limit' => 10
            ]
        ]
    ]);

    // Check if user can edit this company
    $canEdit = $this->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'Companies',
        'controller' => 'Companies',
        'action' => 'edit',
    ]);

    // Check if user can manage users for this company
    $canManageUsers = $this->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'UserManagement',
        'controller' => 'OUsers',
        'action' => 'add',
    ]);

    // Check if user can view pricing
    $canViewPricing = $this->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'Billing',
        'controller' => 'Invoices',
        'action' => 'view',
    ]);

    // Check if user can edit upcharge settings
    $canEditUpcharge = $this->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'Companies',
        'controller' => 'CompanyServicePlans',
        'action' => 'edit',
    ]);

    $this->set(compact(
        'company',
        'canEdit',
        'canManageUsers',
        'canViewPricing',
        'canEditUpcharge'
    ));
}
```

### Example 4: Invoice Download with File Access Control

**File**: `plugins/Billing/src/Controller/Admin/InvoicesController.php`

```php
public function downloadFile($id = null)
{
    // Permission granted in permissions.php, but controller enforces ownership

    $invoice = $this->Invoices->get($id, [
        'contain' => ['Companies', 'OFiles']
    ]);

    // Get user's accessible company IDs
    $accessibleCompanyIds = $this->getAccessibleCompanyIds();

    // Validate invoice belongs to accessible companies
    if (!in_array($invoice->company_id, $accessibleCompanyIds)) {
        $this->Flash->error('You do not have permission to download this invoice.');
        $this->logEvent('Unauthorized invoice download attempt', 'security', 'warning', [
            'invoice_id' => $invoice->id,
            'invoice_company_id' => $invoice->company_id,
            'user_id' => $this->Authentication->getIdentity()->id
        ]);
        return $this->redirect(['action' => 'index']);
    }

    // Check if invoice has file
    if (!$invoice->o_file_id || !$invoice->o_file) {
        $this->Flash->error('Invoice file not found.');
        return $this->redirect(['action' => 'view', $invoice->id]);
    }

    // Build file path
    $filePath = WWW_ROOT . 'files' . DS . $invoice->o_file->path;

    if (!file_exists($filePath)) {
        $this->Flash->error('Invoice file not found on server.');
        return $this->redirect(['action' => 'view', $invoice->id]);
    }

    // Set response with file download
    $response = $this->response
        ->withFile($filePath, [
            'download' => true,
            'name' => 'invoice_' . $invoice->invoice_number . '.pdf'
        ])
        ->withType('application/pdf');

    // Log download
    $this->logEvent('Invoice downloaded', 'billing', 'info', [
        'invoice_id' => $invoice->id,
        'company_id' => $invoice->company_id,
        'user_id' => $this->Authentication->getIdentity()->id
    ]);

    return $response;
}
```

## Template Examples

### Example 1: Device List with Action Buttons

**File**: `plugins/Devices/templates/Admin/Devices/index.php`

```php
<div class="card">
    <div class="card-header">
        <h5 class="card-title">Devices</h5>
        <div class="card-actions">
            <?php if ($this->IsAuthorized->isAuthorized([
                'prefix' => 'Admin',
                'plugin' => 'Devices',
                'controller' => 'Devices',
                'action' => 'add'
            ])): ?>
                <?= $this->Html->link(
                    '<i class="icon-plus"></i> Add Device',
                    ['action' => 'add'],
                    ['class' => 'btn btn-primary', 'escape' => false]
                ) ?>
            <?php endif; ?>

            <?php if ($this->IsAuthorized->isAuthorized([
                'prefix' => 'Admin',
                'plugin' => 'Devices',
                'controller' => 'Devices',
                'action' => 'export'
            ])): ?>
                <?= $this->Html->link(
                    '<i class="icon-download"></i> Export',
                    ['action' => 'export', '?' => $this->request->getQuery()],
                    ['class' => 'btn btn-success', 'escape' => false]
                ) ?>
            <?php endif; ?>
        </div>
    </div>

    <div class="table-responsive">
        <table class="table table-striped">
            <thead>
                <tr>
                    <th><?= $this->Paginator->sort('serial_number', 'Serial Number') ?></th>
                    <th><?= $this->Paginator->sort('Companies.title', 'Company') ?></th>
                    <th><?= $this->Paginator->sort('DeviceStatuses.title', 'Status') ?></th>
                    <th><?= $this->Paginator->sort('created', 'Created') ?></th>
                    <th class="actions">Actions</th>
                </tr>
            </thead>
            <tbody>
                <?php foreach ($devices as $device): ?>
                    <tr>
                        <td><?= h($device->serial_number) ?></td>
                        <td><?= h($device->company->title) ?></td>
                        <td>
                            <span class="badge badge-<?= $device->device_status->status_class ?>">
                                <?= h($device->device_status->title) ?>
                            </span>
                        </td>
                        <td><?= $device->created->format('Y-m-d H:i') ?></td>
                        <td class="actions">
                            <?php if ($this->IsAuthorized->isAuthorized([
                                'prefix' => 'Admin',
                                'plugin' => 'Devices',
                                'controller' => 'Devices',
                                'action' => 'view'
                            ])): ?>
                                <?= $this->Html->link(
                                    '<i class="icon-eye"></i>',
                                    ['action' => 'view', $device->id],
                                    ['class' => 'btn btn-sm btn-info', 'escape' => false, 'title' => 'View']
                                ) ?>
                            <?php endif; ?>

                            <?php if ($this->IsAuthorized->isAuthorized([
                                'prefix' => 'Admin',
                                'plugin' => 'Devices',
                                'controller' => 'Devices',
                                'action' => 'edit'
                            ])): ?>
                                <?= $this->Html->link(
                                    '<i class="icon-pencil"></i>',
                                    ['action' => 'edit', $device->id],
                                    ['class' => 'btn btn-sm btn-primary', 'escape' => false, 'title' => 'Edit']
                                ) ?>
                            <?php endif; ?>

                            <?php if ($this->IsAuthorized->isAuthorized([
                                'prefix' => 'Admin',
                                'plugin' => 'Devices',
                                'controller' => 'Devices',
                                'action' => 'delete'
                            ])): ?>
                                <?= $this->Form->postLink(
                                    '<i class="icon-trash"></i>',
                                    ['action' => 'delete', $device->id],
                                    [
                                        'confirm' => 'Are you sure you want to delete device ' . $device->serial_number . '?',
                                        'class' => 'btn btn-sm btn-danger',
                                        'escape' => false,
                                        'title' => 'Delete'
                                    ]
                                ) ?>
                            <?php endif; ?>
                        </td>
                    </tr>
                <?php endforeach; ?>
            </tbody>
        </table>
    </div>

    <div class="card-footer">
        <?= $this->element('pagination') ?>
    </div>
</div>
```

### Example 2: Company View with Conditional Sections

**File**: `plugins/Companies/templates/Admin/Companies/view.php`

```php
<div class="row">
    <div class="col-md-8">
        <div class="card">
            <div class="card-header">
                <h5 class="card-title"><?= h($company->title) ?></h5>
                <div class="card-actions">
                    <?php if ($canEdit): ?>
                        <?= $this->Html->link(
                            '<i class="icon-pencil"></i> Edit',
                            ['action' => 'edit', $company->id],
                            ['class' => 'btn btn-primary', 'escape' => false]
                        ) ?>
                    <?php endif; ?>
                </div>
            </div>

            <div class="card-body">
                <dl class="row">
                    <dt class="col-sm-3">Company Name:</dt>
                    <dd class="col-sm-9"><?= h($company->title) ?></dd>

                    <dt class="col-sm-3">Account Type:</dt>
                    <dd class="col-sm-9">
                        <span class="badge badge-info">
                            <?= h($company->account_type->title) ?>
                        </span>
                    </dd>

                    <dt class="col-sm-3">Status:</dt>
                    <dd class="col-sm-9">
                        <span class="badge badge-<?= $company->company_status->status_class ?>">
                            <?= h($company->company_status->title) ?>
                        </span>
                    </dd>

                    <?php if ($company->parent_company_id): ?>
                        <dt class="col-sm-3">Parent Company:</dt>
                        <dd class="col-sm-9">
                            <?= $this->Html->link(
                                h($company->parent_company->title),
                                ['action' => 'view', $company->parent_company->id]
                            ) ?>
                        </dd>
                    <?php endif; ?>

                    <?php if ($company->primary_user_id): ?>
                        <dt class="col-sm-3">Primary Contact:</dt>
                        <dd class="col-sm-9">
                            <?= h($company->primary_user->first_name . ' ' . $company->primary_user->last_name) ?>
                            (<?= h($company->primary_user->email) ?>)
                        </dd>
                    <?php endif; ?>
                </dl>
            </div>
        </div>

        <!-- Pricing Section (Only for users who can view pricing) -->
        <?php if ($canViewPricing): ?>
            <div class="card mt-3">
                <div class="card-header">
                    <h5 class="card-title">Pricing Information</h5>
                    <?php if ($canEditUpcharge): ?>
                        <div class="card-actions">
                            <?= $this->Html->link(
                                '<i class="icon-cog"></i> Manage Pricing',
                                [
                                    'plugin' => 'Devices',
                                    'controller' => 'CompanyServicePlans',
                                    'action' => 'index',
                                    $company->id
                                ],
                                ['class' => 'btn btn-sm btn-primary', 'escape' => false]
                            ) ?>
                        </div>
                    <?php endif; ?>
                </div>
                <div class="card-body">
                    <!-- Pricing details here -->
                </div>
            </div>
        <?php else: ?>
            <div class="alert alert-info mt-3">
                <i class="icon-info"></i>
                Contact your account administrator for pricing information.
            </div>
        <?php endif; ?>

        <!-- Child Companies -->
        <?php if (!empty($company->child_companies)): ?>
            <div class="card mt-3">
                <div class="card-header">
                    <h5 class="card-title">Sub-Companies</h5>
                </div>
                <div class="table-responsive">
                    <table class="table">
                        <thead>
                            <tr>
                                <th>Company Name</th>
                                <th>Account Type</th>
                                <th>Status</th>
                                <th class="actions">Actions</th>
                            </tr>
                        </thead>
                        <tbody>
                            <?php foreach ($company->child_companies as $child): ?>
                                <tr>
                                    <td><?= h($child->title) ?></td>
                                    <td><?= h($child->account_type->title) ?></td>
                                    <td>
                                        <span class="badge badge-<?= $child->company_status->status_class ?>">
                                            <?= h($child->company_status->title) ?>
                                        </span>
                                    </td>
                                    <td class="actions">
                                        <?= $this->Html->link(
                                            'View',
                                            ['action' => 'view', $child->id],
                                            ['class' => 'btn btn-sm btn-info']
                                        ) ?>
                                    </td>
                                </tr>
                            <?php endforeach; ?>
                        </tbody>
                    </table>
                </div>
            </div>
        <?php endif; ?>
    </div>

    <div class="col-md-4">
        <!-- Users Section -->
        <?php if ($canManageUsers): ?>
            <div class="card">
                <div class="card-header">
                    <h5 class="card-title">Users</h5>
                    <div class="card-actions">
                        <?= $this->Html->link(
                            '<i class="icon-plus"></i> Add User',
                            [
                                'plugin' => 'UserManagement',
                                'controller' => 'OUsers',
                                'action' => 'add',
                                $company->id
                            ],
                            ['class' => 'btn btn-sm btn-primary', 'escape' => false]
                        ) ?>
                    </div>
                </div>
                <div class="list-group list-group-flush">
                    <?php foreach ($company->company_users as $companyUser): ?>
                        <div class="list-group-item">
                            <div class="d-flex justify-content-between">
                                <strong>
                                    <?= h($companyUser->o_user->first_name . ' ' . $companyUser->o_user->last_name) ?>
                                </strong>
                                <?php if ($company->primary_user_id === $companyUser->o_user_id): ?>
                                    <span class="badge badge-primary">Primary</span>
                                <?php endif; ?>
                            </div>
                            <small class="text-muted"><?= h($companyUser->o_user->email) ?></small>
                        </div>
                    <?php endforeach; ?>
                </div>
            </div>
        <?php endif; ?>
    </div>
</div>
```

## Model Examples

### Example 1: OUsersTable with Role Search

**File**: `plugins/UserManagement/src/Model/Table/OUsersTable.php`

```php
public function initialize(array $config): void
{
    parent::initialize($config);

    $this->hasOne('CompanyUsers', [
        'className' => 'UserManagement.CompanyUsers'
    ]);

    $this->addBehavior('Search.Search');

    // Search by name
    $this->searchManager()
        ->like('name', [
            'fields' => ['first_name', 'last_name'],
            'before' => true,
            'after' => true,
        ])
        // Search by role
        ->add('user_role', 'Search.Callback', [
            'callback' => function (\Cake\ORM\Query $query, array $args, \Search\Model\Filter\Base $filter) {
                if (!empty($args['user_role'])) {
                    // Search for specific role in additional_data JSON
                    $query->where([
                        'OUsers.additional_data LIKE "%\"role_ids\":\"' . intval($args['user_role']) . '\"%"',
                        'OUsers.is_superuser' => 0
                    ]);
                } elseif (($args['user_role'] ?? null) === '0') {
                    // Search for superusers
                    $query->where(['OUsers.is_superuser' => 1]);
                }
            }
        ]);
}

/**
 * Find active, non-deleted users
 *
 * @param \Cake\ORM\Query $query
 * @param array $options
 * @return \Cake\ORM\Query
 */
public function findActiveAndNotDeleted(Query $query, array $options): Query
{
    return $query->where([
        'OUsers.is_active' => 1,
        'OUsers.deleted IS' => null
    ]);
}

/**
 * Find customer users for a specific company
 *
 * @param \Cake\ORM\Query $query
 * @param array $options Must contain 'company_id'
 * @return \Cake\ORM\Query
 */
public function findCustomerUsersForCompany(Query $query, array $options): Query
{
    $companyId = $options['company_id'];

    return $query
        ->matching('CompanyUsers', function ($q) use ($companyId) {
            return $q->where(['CompanyUsers.company_id' => $companyId]);
        })
        ->where(['OUsers.deleted IS' => null]);
}

/**
 * Find system admin users (superusers)
 *
 * @param \Cake\ORM\Query $query
 * @param array $options
 * @return \Cake\ORM\Query
 */
public function findSuperUsers(Query $query, array $options): Query
{
    return $query->where(['OUsers.is_superuser' => 1]);
}
```

### Example 2: After Save Audit Logging

**File**: `plugins/UserManagement/src/Model/Table/OUsersTable.php`

```php
public function afterSave(Event $event, EntityInterface $entity, ArrayObject $options)
{
    parent::afterSave($event, $entity, $options);

    if ($entity->isNew()) {
        // User created
        $this->logEvent('created user', 'user', 'created', [
            'user_id' => $entity->id,
            'username' => $entity->username,
            'email' => $entity->email,
            'role_ids' => $entity->additional_data['role_ids'] ?? null
        ]);
    } elseif (!empty($entity->deleted)) {
        // User soft-deleted
        $this->logEvent('deleted user', 'user', 'deleted', [
            'user_id' => $entity->id,
            'username' => $entity->username,
            'email' => $entity->email
        ]);
    } elseif ($entity->isDirty('password')) {
        // Password changed
        $this->logEvent('changed password', 'user', 'password_changed', [
            'user_id' => $entity->id,
            'changed_by' => $this->getCurrentUserId()
        ]);
    } elseif ($entity->isDirty('additional_data')) {
        // Profile or role updated
        $changes = [];

        if (isset($entity->additional_data['role_ids']) &&
            $entity->getOriginal('additional_data')['role_ids'] !== $entity->additional_data['role_ids']) {
            $changes['role_changed'] = [
                'from' => $entity->getOriginal('additional_data')['role_ids'] ?? null,
                'to' => $entity->additional_data['role_ids']
            ];
        }

        if (!empty($changes)) {
            $this->logEvent('updated user', 'user', 'updated', [
                'user_id' => $entity->id,
                'changes' => $changes
            ]);
        }
    }

    return true;
}

protected function getCurrentUserId()
{
    // Get from session or authentication service
    $session = Router::getRequest()->getSession();
    return $session->read('Auth.id');
}
```

## Custom Rule Examples

See [03_Authorization_Rules.md](03_Authorization_Rules.md) for comprehensive custom rule examples.

## Helper Method Examples

See [07_Implementation_Guide.md](07_Implementation_Guide.md) for helper method examples.
