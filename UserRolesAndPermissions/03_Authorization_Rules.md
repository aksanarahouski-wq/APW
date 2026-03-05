# Authorization Rules Reference

This document provides complete reference for all custom authorization rule methods in the WATM platform.

**Primary File**: `/watm/watm/src/Auth/Rules/WatmRules.php`

## Table of Contents
- [Overview](#overview)
- [Base Rules](#base-rules)
- [Multi-Tenant Rules](#multi-tenant-rules)
- [Account Type Rules](#account-type-rules)
- [Company Hierarchy Rules](#company-hierarchy-rules)
- [Premium/Distributor Rules](#premiumdistributor-rules)
- [Special Permission Rules](#special-permission-rules)
- [Helper Methods](#helper-methods)
- [Custom Rule Classes](#custom-rule-classes)

## Overview

### Rule Method Signature

All rule methods follow this signature:

```php
public function methodName(
    OUser|uuid $user,                    // User object or user ID
    string $role,                        // User's role ('admin' or 'customer')
    ServerRequestInterface|array $request,  // Current HTTP request
    array $route_params = []             // Optional manual route parameters
): bool
```

### Return Values

- `true` - Grant permission (user is authorized)
- `false` - Deny permission (user is not authorized)
- Other value - Continue to next permission rule

### Common Pattern

Most rules follow this pattern:

```php
public function orasesMethodName($user, $role, $request, $route_params = []): bool
{
    // 1. Admin users bypass the check
    if ($role === 'admin') {
        return true;
    }

    // 2. Only customer role can proceed
    if ($role !== 'customer') {
        return false;
    }

    // 3. Get user's company
    $companyUser = $companyUsersTable->find()
        ->contain('Companies.AccountTypes')
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser)) {
        return false;
    }

    // 4. Check specific business logic
    $company = $companyUser->get('company');
    return $someCondition;
}
```

## Base Rules

### orasesDefaultAllow

**File**: `src/Auth/Rules/WatmRules.php:24-35`

**Purpose**: Default catch-all authorization logic

**Logic**:
1. Superusers bypass all checks → return `true`
2. Only 'customer' and 'admin' roles allowed
3. Delegate to parent class for additional checks

**Code**:
```php
public function orasesDefaultAllow($user, $role, $request, $route_params = [])
{
    if ($user->is_superuser) {
        return true;
    }

    if (!in_array($role, ['customer', 'admin'])) {
        return false;
    }

    return parent::orasesDefaultAllow($user, $role, $request, $route_params);
}
```

**Usage**: Automatically called for catch-all permission rule

**Returns**:
- `true` - Superusers
- `false` - Roles other than 'customer' or 'admin'
- Parent result - For valid roles

## Multi-Tenant Rules

### orasesMustBelongToCompany

**File**: `src/Auth/Rules/WatmRules.php:48-163`

**Purpose**: Enforce multi-tenant data isolation - ensure resources belong to user's company or sub-companies

**Applies To**:
- Devices
- Company Payment Methods
- Company Credits
- Company Users (Employees)
- Company Power Schedules
- Company API Keys

**Logic**:
1. Admin role bypasses check → return `true`
2. Get user's company and account type
3. Extract resource ID from request
4. Verify resource belongs to user's company or sub-company (if `can_assign`)

**Key Code Sections**:

#### Device Ownership Check
```php
if ($controller === 'Devices' and $plugin === 'Devices') {
    $devicesTable = TableRegistry::getTableLocator()->get('Devices.Devices');
    $deviceId = $request->getParam('pass.0');

    if ($deviceId === null) {
        return true;  // No specific device, let it 404
    }

    $deviceEntity = $devicesTable->find()->where(['id' => $deviceId])->first();

    if (!isset($deviceEntity)) {
        return true;  // Device doesn't exist, let it 404
    }

    return $this->entityBelongsToCompanyOrSubCompany($deviceEntity, $company);
}
```

#### Company Resources Check
```php
if ($plugin === 'Companies' and (
    $controller === 'CompanyPaymentMethods' or
    $controller === 'CompanyCredits' or
    $controller === 'CompanyUsers' or
    $controller === 'CompanyPowerSchedules' or
    $controller === 'CompanyApiKeys'
)) {
    $companiesTable = TableRegistry::getTableLocator()->get('Companies.Companies');
    $companyId = $request->getParam('pass.0');

    if ($companyId === null) {
        return true;  // No specific company, proceed
    }

    $companyEntity = $companiesTable->find()->where(['id' => $companyId])->first();

    if (!isset($companyEntity)) {
        return true;  // Company doesn't exist, let it 404
    }

    return $this->companyBelongsToUsersCompanyOrSubCompany($companyEntity, $companyUser);
}
```

#### API Key Special Handling
```php
// Special handling for CompanyApiKeys controller
if ($controller === 'CompanyApiKeys') {
    // Actions that use API key ID instead of company ID
    $apiKeyIdActions = ['view', 'edit', 'awsUsage', 'syncToAws'];

    if (in_array($action, $apiKeyIdActions)) {
        $apiKeyId = $request->getParam('pass.0');

        if ($apiKeyId === null) {
            return true;
        }

        // Get the API key and check if its company belongs to user
        $apiKeysTable = TableRegistry::getTableLocator()->get('Companies.CompanyApiKeys');
        $apiKeyEntity = $apiKeysTable->find()
            ->where(['CompanyApiKeys.id' => $apiKeyId])
            ->first();

        if (!isset($apiKeyEntity)) {
            return true; // let it 404
        }

        return $this->entityBelongsToCompanyOrSubCompany($apiKeyEntity, $company);
    }
}
```

**Returns**:
- `true` - Admin role, or resource belongs to user's company/sub-companies
- `false` - Customer role and resource doesn't belong to their company

## Account Type Rules

### orasesIsPremiumCustomer

**File**: `src/Auth/Rules/WatmRules.php:237-273`

**Purpose**: Check if user's company has `can_upcharge` capability (Distributor account type)

**Logic**:
1. Admin role → return `true`
2. Get user's company account type
3. Check `account_type.can_upcharge` flag
4. If company ID in URL, verify it belongs to user's hierarchy

**Code**:
```php
public function orasesIsPremiumCustomer($user, string $role, $request, $route_params = []): bool
{
    if ($role === 'admin') {
        return true;
    }

    if ($role !== 'customer') {
        return false;
    }

    // Get user's company
    $companyUser = $companyUsersTable->find()
        ->contain('Companies.AccountTypes')
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser) || empty($company) || !$company->has('account_type')) {
        return false;
    }

    // Check if company ID was passed (viewing/editing another company)
    $pass = $request->getParam('pass.0');
    if (!empty($pass)) {
        $companyEntity = $companiesTable->find()->where(['id' => $pass])->first();

        if (empty($companyEntity)) {
            return $company->account_type->can_upcharge;
        }

        // Check belongs to hierarchy AND can upcharge
        return $company->account_type->can_upcharge and
               $this->companyBelongsToUsersCompanyOrSubCompany($companyEntity, $companyUser);
    }

    return $company->account_type->can_upcharge;
}
```

**Account Types with `can_upcharge`**:
- Distributor (`can_upcharge = 1`)

**Account Types without `can_upcharge`**:
- Customer (`can_upcharge = 0`)
- Subcustomer (`can_upcharge = 0`)

**Returns**:
- `true` - Admin role, or company has `can_upcharge` capability
- `false` - Customer role and company doesn't have capability

### orasesAllowUpcharging

**File**: `src/Auth/Rules/WatmRules.php:284-307`

**Purpose**: Check if company can AND is allowed to upcharge (both capability and setting)

**Difference from `orasesIsPremiumCustomer`**: Requires **both**:
1. Account type has `can_upcharge = 1` capability
2. Company setting `allow_upcharging = 1`

**Code**:
```php
public function orasesAllowUpcharging($user, string $role, $request, $route_params = []): bool
{
    if ($role === 'admin') {
        return true;
    }

    if ($role !== 'customer') {
        return false;
    }

    $companyUser = $companyUsersTable->find()
        ->contain('Companies.AccountTypes')
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser) || empty($company) || !$company->has('account_type')) {
        return false;
    }

    return $company->account_type->can_upcharge && $company->allow_upcharging;
}
```

**Returns**:
- `true` - Admin, or (account type can upcharge AND company setting allows it)
- `false` - Otherwise

### orasesIsCustomerNotSub

**File**: `src/Auth/Rules/WatmRules.php:318-340`

**Purpose**: Check if company account type is NOT 'Subcustomer'

**Use Case**: Features restricted from subcustomers

**Code**:
```php
public function orasesIsCustomerNotSub($user, string $role, $request, $route_params = []): bool
{
    if ($role === 'admin') {
        return true;
    }

    if ($role !== 'customer') {
        return false;
    }

    $companyUser = $companyUsersTable->find()
        ->contain('Companies.AccountTypes')
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser) || empty($company) || !$company->has('account_type')) {
        return false;
    }

    return $company->account_type->title !== 'Subcustomer';
}
```

**Account Types**:
- **Permitted**: Distributor, Customer
- **Denied**: Subcustomer

**Returns**:
- `true` - Admin, or account type is not 'Subcustomer'
- `false` - Subcustomer account type

### orasesCanViewPricing

**File**: `src/Auth/Rules/WatmRules.php:342-367`

**Purpose**: Determine if user can see pricing information

**Logic**:
1. Premium customers (Distributor, Customer) → `true`
2. Subcustomers → check `company.allow_self_billing` flag

**Code**:
```php
public function orasesCanViewPricing($user, $role, $request, $route_params = [])
{
    if ($role === 'admin') {
        return true;
    }

    if ($role !== 'customer') {
        return false;
    }

    $companyUser = $companyUsersTable->find()
        ->contain('Companies.AccountTypes')
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser) || empty($company) || !$company->has('account_type')) {
        return false;
    }

    // Non-subcustomers always see pricing
    if ($company->account_type->title !== 'Subcustomer') {
        return true;
    }

    // Subcustomers only see pricing if allow_self_billing is enabled
    return $company->allow_self_billing;
}
```

**Returns**:
- `true` - Admin, Distributor, Customer, or Subcustomer with `allow_self_billing`
- `false` - Subcustomer without `allow_self_billing`

### orasesCanAssign

**File**: `src/Auth/Rules/WatmRules.php:418-436`

**Purpose**: Check if company can create sub-companies

**Logic**: Check `account_type.can_assign` flag

**Code**:
```php
public function orasesCanAssign($user, $role, $request, $route_params = []): bool
{
    if ($role == 'customer') {
        $companyUser = $companyUsersTable->find()
            ->where(['o_user_id' => $user->get('id')])
            ->contain(['Companies.AccountTypes'])
            ->first();

        if (!isset($companyUser)) {
            return false;
        }

        return $companyUser->company->account_type->can_assign;
    }

    return true;  // Admin can always assign
}
```

**Account Types with `can_assign`**:
- Distributor (`can_assign = 1`)
- Customer (`can_assign = 1`)

**Account Types without `can_assign`**:
- Subcustomer (`can_assign = 0`)

**Returns**:
- `true` - Admin, or account type has `can_assign` capability
- `false` - Account type doesn't have capability

## Company Hierarchy Rules

### orasesIsChildCustomer

**File**: `src/Auth/Rules/WatmRules.php:459-486`

**Purpose**: Check if company in URL is a child of user's company

**Use Case**: Parent companies managing their sub-companies

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

        // Find all children of user's company
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

**Returns**:
- `true` - Company in URL is a child of user's company
- `false` - Company is not a child, or no company specified

### orasesIsCustomerSelf

**File**: `src/Auth/Rules/WatmRules.php:401-416`

**Purpose**: Check if user is accessing their own company

**Use Case**: Allow users to edit their own company even if other restrictions apply

**Code**:
```php
public function orasesIsCustomerSelf($user, $role, $request, $route_params = []): bool
{
    $isSelf = false;
    $passedParams = $request->getParam('pass');

    if (!empty($passedParams) && $role == 'customer') {
        $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
        $companyUser = $companyUsersTable->find()
            ->where(['o_user_id' => $user->get('id')])
            ->first();

        if ($companyUser->company_id == $passedParams[0]) {
            $isSelf = true;
        }
    }

    return $isSelf;
}
```

**Returns**:
- `true` - Company in URL matches user's company
- `false` - Different company or no company specified

### orasesIsParentCustomer

**File**: `src/Auth/Rules/WatmRules.php:546-550`

**Purpose**: Check if company is active AND can create sub-companies

**Composite Check**: Combines active status + can_assign capability

**Code**:
```php
public function orasesIsParentCustomer($user, $role, $request, $route_params = []): bool
{
    return $this->orasesIsCustomerActive($user, $role, $request, $route_params) &&
           $this->orasesCanAssign($user, $role, $request, $route_params);
}
```

**Returns**:
- `true` - Company is active AND has `can_assign` capability
- `false` - Otherwise

## Premium/Distributor Rules

### orasesIsDistributorAndCanUpcharge

**File**: `src/Auth/Rules/WatmRules.php:509-544`

**Purpose**: Check distributor permissions for company service plans

**Special Feature**: Validates company service plan ownership for distributors

**Code**:
```php
public function orasesIsDistributorAndCanUpcharge($user, $role, $request, $route_params = []): bool
{
    if ($role === 'admin') {
        return true;
    }

    if ($role !== 'customer') {
        return false;
    }

    $companyUser = $companyUsersTable->find()
        ->contain('Companies.AccountTypes')
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser) || empty($company) || !$company->has('account_type')) {
        return false;
    }

    // Check if company service plan ID was passed
    $pass = $request->getParam('pass.0');
    if (!empty($pass)) {
        $companyServicePlansTable = TableRegistry::getTableLocator()->get('Devices.CompanyServicePlans');
        $companyServicePlanEntity = $companyServicePlansTable->find()
            ->where(['CompanyServicePlans.id' => $pass])
            ->contain(['Companies.AccountTypes'])
            ->first();

        if (empty($companyServicePlanEntity)) {
            return $company->account_type->can_upcharge;
        }

        // Check belongs to hierarchy AND can upcharge
        return $company->account_type->can_upcharge &&
               $this->companyBelongsToUsersCompanyOrSubCompany($companyServicePlanEntity->company, $companyUser);
    }

    return $company->account_type->can_upcharge;
}
```

**Returns**:
- `true` - Admin, or distributor managing service plan in their hierarchy
- `false` - Not a distributor or service plan doesn't belong

### orasesIsPremiumCustomerOrSelf

**File**: `src/Auth/Rules/WatmRules.php:378-383`

**Purpose**: Check if premium customer OR editing own company

**Use Case**: Allow premium features OR self-editing

**Code**:
```php
public function orasesIsPremiumCustomerOrSelf($user, string $role, $request, $route_params = []): bool
{
    $isPremium = $this->orasesIsPremiumCustomer($user, $role, $request, $route_params);
    $isSelf = $this->orasesIsCustomerSelf($user, $role, $request, $route_params);
    return $isPremium || $isSelf;
}
```

**Returns**:
- `true` - Premium customer OR accessing own company
- `false` - Otherwise

### orasesIsCustomerNotSubOrSelf

**File**: `src/Auth/Rules/WatmRules.php:394-399`

**Purpose**: Check if NOT subcustomer OR editing own company

**Use Case**: Feature restrictions for subcustomers, except self-editing

**Code**:
```php
public function orasesIsCustomerNotSubOrSelf($user, string $role, $request, $route_params = []): bool
{
    $isCustomerNotSub = $this->orasesIsCustomerNotSub($user, $role, $request, $route_params);
    $isSelf = $this->orasesIsCustomerSelf($user, $role, $request, $route_params);
    return $isCustomerNotSub || $isSelf;
}
```

**Returns**:
- `true` - Not a subcustomer OR accessing own company
- `false` - Subcustomer accessing another company

### orasesIsPremiumCustomerNotSelf

**File**: `src/Auth/Rules/WatmRules.php:494-499`

**Purpose**: Premium customer accessing another company (not self)

**Composite Check**: Premium + Allow Upcharging + Not Self

**Code**:
```php
public function orasesIsPremiumCustomerNotSelf($user, $role, $request, $route_params = []): bool
{
    return $this->orasesIsPremiumCustomer($user, $role, $request, $route_params) &&
           $this->orasesAllowUpcharging($user, $role, $request, $route_params) &&
           !$this->orasesIsCustomerSelf($user, $role, $request, $route_params);
}
```

**Returns**:
- `true` - Premium customer with upcharging accessing different company
- `false` - Otherwise

### orasesIsPremiumCustomerActiveAndChild

**File**: `src/Auth/Rules/WatmRules.php:501-507`

**Purpose**: Premium customer managing active child company

**Composite Check**: Premium + Active + Child + Allow Upcharging

**Code**:
```php
public function orasesIsPremiumCustomerActiveAndChild($user, $role, $request, $route_params = []): bool
{
    return $this->orasesIsPremiumCustomer($user, $role, $request, $route_params) &&
           $this->orasesIsCustomerActive($user, $role, $request, $route_params) &&
           $this->orasesIsChildCustomer($user, $role, $request, $route_params) &&
           $this->orasesAllowUpcharging($user, $role, $request, $route_params);
}
```

**Returns**:
- `true` - All conditions met
- `false` - Any condition fails

## Special Permission Rules

### orasesIsCustomerActive

**File**: `src/Auth/Rules/WatmRules.php:438-457`

**Purpose**: Check if user's company status is 'active'

**Logic**: Verify `company_status.status_key === 'active'`

**Code**:
```php
public function orasesIsCustomerActive($user, $role, $request, $route_params = []): bool
{
    $isActive = false;

    if ($role === 'customer') {
        $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
        $companyUser = $companyUsersTable->find()
            ->contain('Companies.CompanyStatuses')
            ->where(['o_user_id' => $user->get('id')])
            ->first();

        if ($companyUser->has('company') &&
            $companyUser->company->has('company_status') &&
            $companyUser->company->company_status->get('status_key') === 'active'
        ) {
            $isActive = true;
        }
    }

    return $isActive;
}
```

**Company Statuses**:
- `active` - Company is operational
- `inactive` - Company is suspended
- `pending` - Company is not yet activated

**Returns**:
- `true` - Customer role and company status is 'active'
- `false` - Otherwise

### orasesIsPrimary

**File**: `src/Auth/Rules/WatmRules.php:585-603`

**Purpose**: Check if user is the primary user of their company

**Logic**: Verify `company.primary_user_id === user.id`

**Code**:
```php
public function orasesIsPrimary($user, string $role, $request, $route_params = []): bool
{
    if ($role === 'admin') {
        return true;  // Admins are always considered "primary"
    }

    $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
    $companyUser = $companyUsersTable->find()
        ->contain('Companies')
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser)) {
        return true;  // No company user, allow by default
    }

    $company = $companyUser->get('company');

    return $company->primary_user_id === $user->id;
}
```

**Use Case**: Special permissions for company primary contact

**Returns**:
- `true` - Admin, no company, or user is primary
- `false` - User is not the primary user

### orasesIsPrimaryUserOfDistributor

**File**: `src/Auth/Rules/WatmRules.php:552-558`

**Purpose**: Primary user of active distributor OR employee with document access

**Composite Check**: (Active + Premium + Primary) OR (Employee with special flag)

**Code**:
```php
public function orasesIsPrimaryUserOfDistributor($user, $role, $request, $route_params = []): bool
{
    return ($this->orasesIsCustomerActive($user, $role, $request, $route_params) &&
            $this->orasesIsPremiumCustomer($user, $role, $request, $route_params) &&
            $this->orasesIsPrimary($user, $role, $request, $route_params))
           || $this->orasesIsEmployeeOfDistributorWithAccessToCompanyDocuments($user);
}
```

**Private Helper**:
```php
private function orasesIsEmployeeOfDistributorWithAccessToCompanyDocuments($user): bool
{
    $companyUser = $companyUsersTable->find()
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    $companyEntity = $companiesTable->find()->where(
        ['Companies.id' => $companyUser->company_id]
    )->first();

    if (!$companyEntity->has_electronically_consented) {
        return false;
    }

    if (!empty($user->additional_data) and
        isset($user->additional_data['can_access_company_documents'])) {
        return $user->additional_data['can_access_company_documents'];
    }

    return false;
}
```

**Use Case**: Access to sensitive company documents (e.g., signed agreements)

**Returns**:
- `true` - Primary user of active distributor, or employee with document permission
- `false` - Otherwise

### orasesHasOwnPaymentMethod

**File**: `src/Auth/Rules/WatmRules.php:614-642`

**Purpose**: Check if company can have its own payment methods

**Logic**:
- Top-level companies (no parent) → `true`
- Sub-companies with `allow_self_billing` → `true`
- Sub-companies without `allow_self_billing` → `false` (use parent's payment method)

**Code**:
```php
public function orasesHasOwnPaymentMethod($user, string $role, $request, $route_params = []): bool
{
    if ($role === 'admin') {
        return true;
    }

    if ($role !== 'customer') {
        return true;
    }

    $companyUser = $companyUsersTable->find()
        ->contain('Companies')
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser)) {
        return true;
    }

    $company = $companyUser->get('company');

    // Company has own payment method if:
    // 1. No parent company, OR
    // 2. Has parent but allow_self_billing is enabled
    if (
        !isset($company->parent_company_id) ||
        empty($company->parent_company_id) ||
        $company->allow_self_billing
    ) {
        return true;
    }

    return false;
}
```

**Returns**:
- `true` - Admin, no parent company, or `allow_self_billing` enabled
- `false` - Sub-company without `allow_self_billing`

## Helper Methods

### entityBelongsToCompanyOrSubCompany (private)

**File**: `src/Auth/Rules/WatmRules.php:168-196`

**Purpose**: Check if entity belongs to user's company or sub-company

**Parameters**:
- `$entity` - Entity with `company_id` property
- `$company` - User's company entity

**Logic**:
1. Check if entity's `company_id` matches user's company
2. If account type has `can_assign`, check child companies
3. Return `true` if entity belongs to company or any child

**Code**:
```php
private function entityBelongsToCompanyOrSubCompany($entity, $company) {
    // Direct ownership
    if ($entity->company_id === $company->id) {
        return true;
    }

    // Check child companies if can_assign
    if ($company->account_type->can_assign) {
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

**Returns**:
- `true` - Entity belongs to company or sub-company
- `false` - Entity doesn't belong

### companyBelongsToUsersCompanyOrSubCompany (private)

**File**: `src/Auth/Rules/WatmRules.php:198-226`

**Purpose**: Check if company belongs to user's company hierarchy

**Parameters**:
- `$entity` - Company entity to check
- `$companyUser` - User's company user record

**Logic**: Same as `entityBelongsToCompanyOrSubCompany` but for company entities

**Code**:
```php
private function companyBelongsToUsersCompanyOrSubCompany($entity, $companyUser) {
    // Direct match
    if ($companyUser->company_id === $entity->id) {
        return true;
    }

    // Check child companies if can_assign
    if ($companyUser->company->account_type->can_assign) {
        $childCompanies = $companyUsersTable
            ->Companies
            ->find('children', [
                'for' => $companyUser->company->id,
                'threads' => 'full',
            ])
            ->select(['Companies.id'])
            ->all()
            ->extract('id')
            ->toArray();

        if (in_array($entity->id, $childCompanies)) {
            return true;
        }
    }

    return false;
}
```

**Returns**:
- `true` - Company belongs to user's hierarchy
- `false` - Company doesn't belong

## Custom Rule Classes

### WATMCustomerRules

**File**: `plugins/UserManagement/src/Auth/Rbac/Rules/WATMCustomerRules.php:11-30`

**Purpose**: Special authorization for customer company viewing

**Method**: `orasesDefaultAllow()`

**Logic**:
- Customer can view their own company
- Customer can view their parent company (if subcustomer)
- Admin users delegate to parent class

**Code**:
```php
public function orasesDefaultAllow($user, $role, $request, $route_params = [])
{
    if ($role === 'customer') {
        $companyUsersTable = TableRegistry::getTableLocator()->get('UserManagement.CompanyUsers');
        $companiesTable = TableRegistry::getTableLocator()->get('Companies.Companies');

        // Company they are attempting to view
        $companyId = $request->getParam('pass.0');
        $companyEntity = $companiesTable->find()->where(['Companies.id' => $companyId])->first();

        // Build list of allowed companies: company itself + parent if exists
        $companyIds = [$companyId];
        if ($companyEntity->hasValue('parent_company_id')) {
            $companyIds[] = $companyEntity->parent_company_id;
        }

        // Check if user belongs to one of these companies
        return $companyUsersTable->exists([
            'CompanyUsers.company_id IN' => $companyIds,
            'CompanyUsers.o_user_id' => $user->id
        ]);
    }

    return parent::orasesDefaultAllow($user, $role, $request, $route_params);
}
```

**Usage**: `config/permissions.php` line 205 for `Companies/Companies/view` action

### RmaRules

**File**: `plugins/UserManagement/src/Auth/Rbac/Rules/RmaRules.php:12-39`

**Purpose**: Control RMA editing based on status

**Method**: `orasesDefaultAllow()`

**Logic**:
- **Superusers**: Can edit RMAs with status "RMA In Progress"
- **Regular users**: Cannot edit RMAs

**Code**:
```php
public function orasesDefaultAllow($user, $role, $request, $route_params = [])
{
    $parameters = $request->getAttribute('params');
    $passedParam = $parameters['pass'];

    if ($user->is_superuser) {
        if (empty($passedParam)) {
            return true;  // Creating new RMA
        } else {
            // Check if RMA status allows editing
            $rmasTable = TableRegistry::getTableLocator()->get('Devices.Rmas');
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
}
```

**RMA Statuses**:
- **Editable**: "RMA In Progress"
- **Not Editable**: All other statuses (completed, cancelled, etc.)

**Usage**: `config/permissions.php` line 215 for `Devices/Rmas/edit` action

## Usage Examples

### Example 1: Check Premium Customer in Controller

```php
use CakeDC\Auth\Traits\IsAuthorizedTrait;

class CompaniesController extends AppController
{
    use IsAuthorizedTrait;

    public function upchargeSettings($id)
    {
        // Check if user is premium customer
        $canUpcharge = $this->isAuthorized([
            'prefix' => 'Admin',
            'plugin' => 'Companies',
            'controller' => 'Companies',
            'action' => 'upchargeSettings',
        ]);

        if (!$canUpcharge) {
            $this->Flash->error('Only distributors can access upcharge settings.');
            return $this->redirect(['action' => 'index']);
        }

        // Continue with action...
    }
}
```

### Example 2: Conditional UI in Template

```php
<?php
// Check if user can view pricing
$canViewPricing = $this->IsAuthorized->isAuthorized([
    'prefix' => 'Admin',
    'plugin' => 'Billing',
    'controller' => 'Invoices',
    'action' => 'index',
]);
?>

<?php if ($canViewPricing): ?>
    <div class="pricing-section">
        <h3>Pricing Details</h3>
        <!-- Pricing info here -->
    </div>
<?php endif; ?>
```

### Example 3: Add New Custom Rule

**Step 1: Add method to WatmRules.php**

```php
public function orasesCanExportData($user, $role, $request, $route_params = []): bool
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
        ->contain('Companies')
        ->where(['o_user_id' => $user->get('id')])
        ->first();

    if (empty($companyUser)) {
        return false;
    }

    // Check if company has export permission in additional_data
    $company = $companyUser->get('company');
    return isset($company->additional_data['can_export_data']) &&
           $company->additional_data['can_export_data'] === true;
}
```

**Step 2: Add permission rule to config/permissions.php**

```php
[
    'prefix' => 'Admin',
    'plugin' => 'Devices',
    'controller' => 'Devices',
    'action' => 'export',
    'role' => '*',
    'allowed' => new \App\Auth\Rules\WatmRules(),
    // Will call orasesCanExportData() method
],
```

**Step 3: Specify run_method if needed**

If the method name doesn't follow the `orases*` pattern, specify it explicitly in database:

```sql
INSERT INTO o_users_roles_o_users_permissions (o_users_role_id, o_users_permission_id, run_method)
VALUES (1, 123, 'orasesCanExportData');
```

## Next Steps

- See [02_Permission_Configuration.md](02_Permission_Configuration.md) for how rules are referenced in permissions
- See [07_Implementation_Guide.md](07_Implementation_Guide.md) for practical usage in development
