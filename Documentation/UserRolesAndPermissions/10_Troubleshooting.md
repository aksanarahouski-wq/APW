# Troubleshooting Guide

Common permission issues and how to resolve them in the WATM platform.

## Table of Contents
- [Permission Denied (403)](#permission-denied-403)
- [User Cannot Access Resource](#user-cannot-access-resource)
- [Subcustomer Issues](#subcustomer-issues)
- [Company Hierarchy Problems](#company-hierarchy-problems)
- [API Authentication Issues](#api-authentication-issues)
- [Role Assignment Problems](#role-assignment-problems)
- [Debugging Tools](#debugging-tools)

## Permission Denied (403)

### Symptom
User receives "Permission Denied" or 403 Forbidden error when trying to access a page or perform an action.

### Common Causes & Solutions

#### 1. User Not Authenticated

**Check**:
```php
// In controller or debug
debug($this->Authentication->getIdentity());
```

**Solution**:
- Ensure user is logged in
- Check session is not expired
- Verify authentication middleware is running

#### 2. Incorrect Role Assignment

**Check**:
```php
$user = $this->Authentication->getIdentity();
debug($user->additional_data['role_ids'] ?? 'NO ROLE');
```

**Expected Values**:
- `'1'` - Admin (system)
- `'3'` - Super Admin (customer)
- `'4'` - Admin (customer)

**Solution**:
```sql
-- Fix user role in database
UPDATE o_users
SET additional_data = JSON_SET(additional_data, '$.role_ids', '3')
WHERE id = [user_id];
```

#### 3. Company Not Active

**Check**:
```php
$companyUser = $this->getCompanyUser();
debug($companyUser->company->company_status->status_key);
```

**Expected**: `'active'`

**Solution**:
```sql
-- Activate company
UPDATE companies
SET company_status_id = (SELECT id FROM company_statuses WHERE status_key = 'active')
WHERE id = [company_id];
```

#### 4. Missing Account Type Capability

**Check**:
```php
$company = $companyUser->company;
debug([
    'can_upcharge' => $company->account_type->can_upcharge,
    'can_assign' => $company->account_type->can_assign,
    'account_type' => $company->account_type->title
]);
```

**Solution**: Change company's account type
```sql
-- Change to Distributor (can_upcharge and can_assign)
UPDATE companies
SET account_type_id = 1
WHERE id = [company_id];

-- Change to Customer (can_assign only)
UPDATE companies
SET account_type_id = 2
WHERE id = [company_id];
```

#### 5. Permission Not Configured

**Check**: Look in `config/permissions.php` for matching rule

**Solution**: Add permission rule
```php
// In config/permissions.php
[
    'prefix' => 'Admin',
    'plugin' => 'YourPlugin',
    'controller' => 'YourController',
    'action' => 'yourAction',
    'role' => '*',
    'allowed' => new \App\Auth\Rules\WatmRules(),
],
```

#### 6. Custom Rule Returning False

**Check**: Add debug logging to custom rule
```php
// In WatmRules.php
public function orasesYourRule($user, $role, $request, $route_params = []): bool
{
    Log::debug('orasesYourRule called', [
        'user_id' => $user->id,
        'role' => $role,
        'controller' => $request->getParam('controller'),
        'action' => $request->getParam('action'),
    ]);

    // Your rule logic
    $result = /* ... */;

    Log::debug('orasesYourRule result', ['result' => $result]);

    return $result;
}
```

**Check logs**: `logs/debug.log`

## User Cannot Access Resource

### Symptom
User can see list page but gets "Permission Denied" when trying to view/edit specific resource.

### Diagnosis

**Check resource ownership**:
```php
// In controller
$device = $this->Devices->get($id);
debug('Device company_id: ' . $device->company_id);

$accessibleIds = $this->getAccessibleCompanyIds();
debug('Accessible company IDs: ' . implode(', ', $accessibleIds));

if (!in_array($device->company_id, $accessibleIds)) {
    debug('MISMATCH: Device does not belong to accessible companies');
}
```

### Solutions

#### 1. Resource Belongs to Different Company

**Scenario**: User in Company A trying to access Company B resource

**Fix**: Either:
- Move resource to user's company
```sql
UPDATE devices
SET company_id = [user_company_id]
WHERE id = [device_id];
```

- Or create parent-child relationship if appropriate
```sql
-- Make Company B a child of Company A
UPDATE companies
SET parent_company_id = [company_a_id]
WHERE id = [company_b_id];
```

#### 2. Account Type Cannot Access Sub-Companies

**Scenario**: Customer trying to access child company resource, but `can_assign = 0`

**Check**:
```php
debug($company->account_type->can_assign);  // Should be 1
```

**Fix**: Upgrade account type
```sql
UPDATE companies
SET account_type_id = (SELECT id FROM account_types WHERE can_assign = 1)
WHERE id = [company_id];
```

#### 3. Authorization Rule Not Checking Hierarchy

**Scenario**: Custom rule only checks exact company match, not children

**Fix**: Use helper method in rule
```php
// In WatmRules.php
return $this->entityBelongsToCompanyOrSubCompany($entity, $company);
```

## Subcustomer Issues

### Symptom
Subcustomer user cannot view pricing or manage payment methods.

### Diagnosis

#### 1. Cannot View Pricing

**Check**:
```php
$company = $companyUser->company;
debug([
    'account_type' => $company->account_type->title,
    'allow_self_billing' => $company->allow_self_billing
]);
```

**Expected for pricing access**:
- `account_type->title !== 'Subcustomer'` OR
- `allow_self_billing === true`

**Fix**:
```sql
-- Enable self-billing for subcustomer
UPDATE companies
SET allow_self_billing = 1
WHERE id = [subcustomer_company_id];
```

#### 2. Cannot Add Payment Methods

**Check**:
```php
// orasesHasOwnPaymentMethod() checks this
$hasParent = !empty($company->parent_company_id);
$allowSelfBilling = $company->allow_self_billing;

debug([
    'has_parent' => $hasParent,
    'allow_self_billing' => $allowSelfBilling,
    'can_have_own_payment' => !$hasParent || $allowSelfBilling
]);
```

**Fix**: Enable self-billing (see above)

#### 3. Cannot Create Sub-Companies

**Scenario**: Subcustomers cannot create sub-companies by design

**Expected**: `can_assign = 0` for Subcustomer account type

**Fix**: If they need this capability, upgrade to Customer account type
```sql
UPDATE companies
SET account_type_id = (SELECT id FROM account_types WHERE title = 'Customer')
WHERE id = [company_id];
```

## Company Hierarchy Problems

### Symptom
Parent company cannot access child company resources, or hierarchy queries return unexpected results.

### Diagnosis

#### 1. Parent-Child Relationship Not Set

**Check**:
```sql
SELECT id, title, parent_company_id
FROM companies
WHERE id IN ([parent_id], [child_id]);
```

**Fix**:
```sql
UPDATE companies
SET parent_company_id = [parent_id]
WHERE id = [child_id];
```

#### 2. Circular Reference

**Check**:
```sql
-- This should return empty for valid hierarchy
SELECT c1.id as company_id, c1.parent_company_id, c2.parent_company_id as grandparent_id
FROM companies c1
JOIN companies c2 ON c1.parent_company_id = c2.id
WHERE c2.parent_company_id = c1.id;
```

**Fix**: Break circular reference
```sql
UPDATE companies
SET parent_company_id = NULL
WHERE id = [problematic_company_id];
```

#### 3. Tree Behavior Not Working

**Check CakePHP tree behavior**:
```php
$companiesTable = TableRegistry::getTableLocator()->get('Companies.Companies');

// Test tree find
$children = $companiesTable
    ->find('children', ['for' => $parentId])
    ->all();

debug('Children count: ' . $children->count());
foreach ($children as $child) {
    debug('Child: ' . $child->title . ' (ID: ' . $child->id . ')');
}
```

**Fix**: Rebuild tree structure
```bash
bin/cake companies rebuild_tree
```

Or manually:
```php
$companiesTable->recover();
```

## API Authentication Issues

### Symptom
API requests return 401 Unauthorized or "API key required" error.

### Diagnosis

#### 1. Missing X-API-Key Header

**Check Request**:
```bash
# cURL example
curl -H "X-API-Key: your-api-key-here" \
     https://watm.example.com/api/v1/devices
```

**Solution**: Include header in all API requests

#### 2. Invalid API Key

**Check Database**:
```sql
SELECT id, api_key, company_id, is_active, expires
FROM company_api_keys
WHERE api_key = '[your_api_key]';
```

**Expected**:
- Record exists
- `is_active = 1`
- `expires` is NULL or future date

**Fix**:
```sql
-- Activate API key
UPDATE company_api_keys
SET is_active = 1
WHERE api_key = '[your_api_key]';

-- Extend expiration
UPDATE company_api_keys
SET expires = DATE_ADD(NOW(), INTERVAL 1 YEAR)
WHERE api_key = '[your_api_key]';
```

#### 3. API Key Expired

**Check**:
```sql
SELECT api_key, expires, expires < NOW() as is_expired
FROM company_api_keys
WHERE api_key = '[your_api_key]';
```

**Fix**: Update expiration (see above)

#### 4. API Gateway Secret Mismatch

**Check**: Look for API gateway secret validation in middleware

**Fix**: Ensure correct secret in request or disable gateway check in `app_local.php`
```php
// In config/app_local.php
'ApiGateway' => [
    'enforce' => false,  // Disable gateway check
    // or
    'secret' => 'correct-secret-value'
],
```

## Role Assignment Problems

### Symptom
User has incorrect permissions or cannot access expected features.

### Diagnosis

#### 1. Role Stored Incorrectly

**Check**:
```sql
SELECT id, username, email, additional_data
FROM o_users
WHERE id = [user_id];
```

**Expected Format**:
```json
{
  "role_ids": "3"
}
```

**Common Mistakes**:
- Number instead of string: `"role_ids": 3` ❌
- Array instead of string: `"role_ids": [3]` ❌
- Missing quotes: `"role_ids": 3` ❌

**Fix**:
```sql
-- Correct format
UPDATE o_users
SET additional_data = JSON_SET(additional_data, '$.role_ids', '3')
WHERE id = [user_id];
```

#### 2. Role Type Mismatch

**Check**:
```sql
SELECT r.id, r.title, r.role_type
FROM o_users u
JOIN JSON_TABLE(
    u.additional_data,
    '$' COLUMNS (role_ids VARCHAR(10) PATH '$.role_ids')
) jt
JOIN o_users_roles r ON r.id = jt.role_ids
WHERE u.id = [user_id];
```

**Expected**:
- System admins: `role_type = 'admin'`
- Company users: `role_type = 'customer'`

**Fix**: Assign correct role
```sql
-- Get correct role ID
SELECT id, title, role_type FROM o_users_roles;

-- Assign correct role
UPDATE o_users
SET additional_data = JSON_SET(additional_data, '$.role_ids', '[correct_role_id]')
WHERE id = [user_id];
```

#### 3. User Not Linked to Company

**Check**:
```sql
SELECT o_user_id, company_id
FROM company_users
WHERE o_user_id = [user_id];
```

**Expected**: One record

**Fix**: Link user to company
```sql
INSERT INTO company_users (o_user_id, company_id, created, modified)
VALUES ([user_id], [company_id], NOW(), NOW());
```

## Debugging Tools

### Enable Debug Mode

**File**: `config/app_local.php`

```php
return [
    'debug' => true,  // Enable debug mode
    'Error' => [
        'errorLevel' => E_ALL,
    ],
    'Log' => [
        'debug' => [
            'className' => FileLog::class,
            'path' => LOGS,
            'file' => 'debug',
            'levels' => ['notice', 'info', 'debug'],
        ],
    ],
];
```

### Debug Permission Check

**Add to Controller**:
```php
public function yourAction($id)
{
    // Debug current user
    $user = $this->Authentication->getIdentity();
    debug([
        'user_id' => $user->id,
        'email' => $user->email,
        'is_superuser' => $user->is_superuser,
        'role_ids' => $user->additional_data['role_ids'] ?? null
    ]);

    // Debug company
    $companyUser = $this->getCompanyUser();
    debug([
        'company_id' => $companyUser->company->id,
        'company_title' => $companyUser->company->title,
        'account_type' => $companyUser->company->account_type->title,
        'can_upcharge' => $companyUser->company->account_type->can_upcharge,
        'can_assign' => $companyUser->company->account_type->can_assign,
        'status' => $companyUser->company->company_status->status_key
    ]);

    // Debug accessible companies
    $accessibleIds = $this->getAccessibleCompanyIds();
    debug('Accessible company IDs: ' . implode(', ', $accessibleIds));

    // Debug permission check
    $isAuthorized = $this->isAuthorized([
        'prefix' => 'Admin',
        'plugin' => 'YourPlugin',
        'controller' => 'YourController',
        'action' => 'yourAction',
    ]);
    debug('Is authorized: ' . ($isAuthorized ? 'YES' : 'NO'));

    // Continue with action...
}
```

### Check Permission Configuration

**Command Line**:
```bash
cd watm/watm
php -l config/permissions.php  # Check syntax

# Search for specific permission
grep -A 5 "controller' => 'Devices'" config/permissions.php
```

### View Authorization Logs

**Enable Logging in WatmRules.php**:
```php
public function orasesDefaultAllow($user, $role, $request, $route_params = [])
{
    Log::debug('orasesDefaultAllow', [
        'user_id' => $user->id ?? 'unknown',
        'role' => $role,
        'is_superuser' => $user->is_superuser ?? false,
        'controller' => $request->getParam('controller'),
        'action' => $request->getParam('action'),
        'plugin' => $request->getParam('plugin'),
    ]);

    // ... rest of method
}
```

**View Logs**:
```bash
tail -f logs/debug.log
```

### Database Inspection Queries

**Check User Setup**:
```sql
-- Complete user information
SELECT
    u.id,
    u.username,
    u.email,
    u.is_superuser,
    u.additional_data->'$.role_ids' as role_id,
    r.title as role_title,
    r.role_type,
    cu.company_id,
    c.title as company_title,
    at.title as account_type,
    at.can_upcharge,
    at.can_assign,
    cs.status_key as company_status
FROM o_users u
LEFT JOIN company_users cu ON u.id = cu.o_user_id
LEFT JOIN companies c ON cu.company_id = c.id
LEFT JOIN account_types at ON c.account_type_id = at.id
LEFT JOIN company_statuses cs ON c.company_status_id = cs.id
LEFT JOIN o_users_roles r ON r.id = CAST(u.additional_data->'$.role_ids' AS UNSIGNED)
WHERE u.id = [user_id];
```

**Check Company Hierarchy**:
```sql
-- Company tree
WITH RECURSIVE company_tree AS (
    -- Root company
    SELECT id, title, parent_company_id, 0 as level
    FROM companies
    WHERE id = [root_company_id]

    UNION ALL

    -- Children
    SELECT c.id, c.title, c.parent_company_id, ct.level + 1
    FROM companies c
    JOIN company_tree ct ON c.parent_company_id = ct.id
)
SELECT * FROM company_tree ORDER BY level, title;
```

**Check API Keys**:
```sql
SELECT
    ak.id,
    ak.api_key,
    ak.company_id,
    c.title as company_title,
    ak.is_active,
    ak.expires,
    ak.expires < NOW() as is_expired
FROM company_api_keys ak
JOIN companies c ON ak.company_id = c.id
WHERE ak.company_id = [company_id];
```

## Common Error Messages

| Error Message | Likely Cause | Solution |
|--------------|--------------|----------|
| "Permission Denied" | Authorization failed | Check role, company status, account type |
| "User not associated with a company" | Missing company_users record | Link user to company |
| "You do not have permission to view this resource" | Resource belongs to different company | Check resource ownership |
| "API key required" | Missing X-API-Key header | Include header in request |
| "Invalid API key" | API key not in database or inactive | Check/activate API key |
| "API key expired" | Expiration date passed | Update expiration date |
| "Company is inactive" | Company status not 'active' | Activate company |

## Getting Help

If issues persist after troubleshooting:

1. **Check Logs**:
   - `logs/debug.log`
   - `logs/error.log`

2. **Enable Verbose Logging**:
   - Add `Log::debug()` calls to authorization rules
   - Enable SQL query logging

3. **Test with Superuser**:
   - Try action as superuser to rule out authorization issues
   - If works as superuser, problem is permission-related

4. **Compare Working vs Non-Working**:
   - Find similar working scenario
   - Compare user, role, company setups

5. **Review Documentation**:
   - [01_Architecture_Overview.md](01_Architecture_Overview.md)
   - [02_Permission_Configuration.md](02_Permission_Configuration.md)
   - [03_Authorization_Rules.md](03_Authorization_Rules.md)
   - [07_Implementation_Guide.md](07_Implementation_Guide.md)

6. **Database Consistency Check**:
   ```sql
   -- Check for users without companies (should be empty for customer users)
   SELECT u.id, u.username, u.email
   FROM o_users u
   LEFT JOIN company_users cu ON u.id = cu.o_user_id
   WHERE cu.company_id IS NULL
   AND u.additional_data->'$.role_ids' IN ('3', '4');

   -- Check for companies without account types
   SELECT c.id, c.title
   FROM companies c
   WHERE c.account_type_id IS NULL;

   -- Check for invalid role assignments
   SELECT u.id, u.username, u.additional_data->'$.role_ids' as role_id
   FROM o_users u
   WHERE u.additional_data->'$.role_ids' IS NOT NULL
   AND NOT EXISTS (
       SELECT 1 FROM o_users_roles r
       WHERE r.id = CAST(u.additional_data->'$.role_ids' AS UNSIGNED)
   );
   ```
