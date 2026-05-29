# Company Invite Flow Analysis

## Overview
The Company Invite feature allows administrators and customer super admins to invite new companies to join the WATM platform by sending them a registration link via email.

**Page URL:** `admin/companies/invite`
**Controller:** `Companies\Controller\Admin\CompanyInvitesController::invite()`
**Route:** `/admin/companies/company-invites/invite`

---

## Key Components

### Database Table: `company_invites`
Created by migration: `20240425172444_CreateCompanyInvites.php`

**Schema:**
```sql
company_invites
├── id (int, auto-increment, primary key)
├── token (string, 255) - UUID for unique invite link
├── email (string, 255) - Email address of invitee
├── account_type_id (int) - FK to account_types table
├── parent_company_id (int, nullable) - FK to companies (added in 20240612152555)
├── created (datetime)
└── modified (datetime)
```

### Key Files
- **Controller:** `plugins/Companies/src/Controller/Admin/CompanyInvitesController.php`
- **Registration Controller:** `plugins/Companies/src/Controller/CompaniesController.php`
- **Model:** `plugins/Companies/src/Model/Table/CompanyInvitesTable.php`
- **Mailer:** `plugins/Companies/src/Mailer/CompanyInviteMailer.php`
- **Templates:**
  - Admin form: `plugins/Companies/templates/Admin/CompanyInvites/invite.php`
  - Email (HTML): `plugins/Companies/templates/email/html/invite.php`
  - Email (Text): `plugins/Companies/templates/email/text/invite.php`
  - Registration form: `plugins/Companies/templates/Companies/invitation.php`

---

## The Flow: Step by Step

### Step 1: Admin/Customer Creates Invitation

**Who can send invites:**
- **Admin users** (role = 'admin') - Can invite Distributors or Customers
- **Customer Super Admins** (role = 'customer') - Can only invite Subcustomers

**Process:**
1. User navigates to `/admin/companies/company-invites/invite`
2. Form displays with:
   - **Account Type** dropdown (filtered based on user role)
     - Admins see: Distributor, Customer
     - Customer users see: Subcustomer only
   - **Email** input field
3. User fills out the form and clicks "Save"

**What happens on save:**
```php
// CompanyInvitesController::invite() - lines 28-36
$companyInviteEntity = $this->CompanyInvites->newEntity([
    'token' => Text::uuid(),  // Generate unique UUID token
    'parent_company_id' => $parentCompanyId  // Set if customer user
]);
```

4. Entity is patched with form data (email, account_type_id)
5. Record saved to `company_invites` table
6. **Email automatically sent** via `afterSave` hook in `CompanyInvitesTable`

**Email sending logic:**
```php
// CompanyInvitesTable::afterSave() - lines 71-74
public function afterSave(EventInterface $event, EntityInterface $entity, ArrayObject $options)
{
    $this->getMailer('Companies.CompanyInvite')->send('invite', [$entity]);
}
```

7. Flash message: "Company Invitation sent to: [email]"
8. User redirected back to invite form

### Step 2: Invitee Receives Email

**Email content:**
```
Subject: Welcome to APCommand!

You have been invited to join All Point Command.
Click this link to setup your account: [Account Setup]
```

**Link format:**
```
https://[domain]/companies/companies/invitation?token=[UUID]
```

The link points to the **public-facing** (no prefix) controller action: `CompaniesController::invitation()`

### Step 3: Invitee Clicks Link and Registers

**Registration process** (`CompaniesController::invitation()` - lines 169-271):

1. **Token validation:**
   - Checks for `token` query parameter
   - Looks up invitation in `company_invites` table
   - Throws BadRequestException if token not found

2. **Account type determination:**
   - Retrieves `account_type_id` from invitation
   - Determines if payment info required:
     - **Required:** For Customers and Distributors (no parent company)
     - **Not required:** For Subcustomers (has parent company)

3. **Form pre-population:**
   - Email field pre-filled from invitation
   - Account type set from invitation
   - Parent company ID set if applicable
   - User status set to "Pending Activation"

4. **Form submission:**
   - Collects company information (name, address, phone, etc.)
   - Collects primary user information (first name, last name, email, phone)
   - Collects bank account information (if required)
   - Creates:
     - Company record
     - Primary user (OUser) with role "Super Admin"
     - Payment method (if required, marked as primary)
     - For Distributors: Payment method marked as payout method

5. **Account creation:**
```php
// Lines 246-248
$companyEntity->o_user->username = $companyEntity->o_user->email;
$companyEntity->o_user->password = Text::uuid();  // Random password
$companyEntity->o_user->updateToken(600);  // Token expires in 10 minutes
```

6. **Post-creation:**
   - Company saved with option `['send_agreement_to_primary' => true]`
   - Flash message: "Thank you for registering, check your email for further instructions"
   - User receives email with password reset link
   - Redirect to login page

### Step 4: Invitee Sets Password and Logs In

After registration:
1. User receives email with verification token (10-minute expiration)
2. Clicks link to set password
3. Completes account setup
4. Can now log in and access the platform

---

## User Types and Account Creation

### For Admin Invitations:

| Invited As | Account Type | Parent Company | Payment Required | Can Upcharge |
|------------|--------------|----------------|------------------|--------------|
| Distributor | Distributor | None | Yes (with payout) | Yes |
| Customer | Customer | None | Yes | No |

### For Customer Super Admin Invitations:

| Invited As | Account Type | Parent Company | Payment Required | Can Upcharge |
|------------|--------------|----------------|------------------|--------------|
| Subcustomer | Subcustomer | Inviting company | No | Depends on parent |

---

## Invitation Tracking

### Current Tracking Capabilities

**What is tracked:**
1. **Creation timestamp** - `company_invites.created` records when invite was sent
2. **Last modified** - `company_invites.modified` tracks any updates
3. **Email address** - Who was invited
4. **Account type** - What type of account was offered
5. **Parent company** - For subcustomer invitations

**What is NOT currently tracked:**
1. ❌ **Conversion status** - No field indicates if invite was accepted
2. ❌ **Company ID link** - No foreign key linking invitation to created company
3. ❌ **Acceptance timestamp** - No record of when invite was used
4. ❌ **Expiration** - Tokens never expire (security concern)
5. ❌ **Revocation** - No way to invalidate unused invitations
6. ❌ **Click tracking** - No analytics on email opens or link clicks

### Identifying Conversions (Manual Process)

Currently, to track if an invitation was converted, you would need to:

1. **Query invitations:**
```sql
SELECT * FROM company_invites;
```

2. **Match by email:**
```sql
SELECT ci.*, c.id as company_id, c.created as company_created
FROM company_invites ci
LEFT JOIN o_users ou ON ci.email = ou.email
LEFT JOIN companies c ON ou.id = c.primary_user_id
WHERE c.id IS NOT NULL;
```

3. **Check timing:**
   - Compare `company_invites.created` with `companies.created`
   - If company created after invite sent, likely converted

**Limitations:**
- Email could change after registration
- Multiple invites to same email can't be distinguished
- No way to track which specific invite led to registration if multiple sent

---

## Conversion Rate Analysis (Current Capabilities)

### Possible Queries

**1. Total invitations sent:**
```sql
SELECT COUNT(*) as total_invites FROM company_invites;
```

**2. Invitations by account type:**
```sql
SELECT at.title, COUNT(*) as invite_count
FROM company_invites ci
JOIN account_types at ON ci.account_type_id = at.id
GROUP BY at.title;
```

**3. Estimated conversions (by email matching):**
```sql
SELECT
    ci.email,
    ci.created as invite_sent,
    c.id as company_id,
    c.title as company_name,
    c.created as company_created,
    TIMESTAMPDIFF(DAY, ci.created, c.created) as days_to_convert
FROM company_invites ci
JOIN o_users ou ON ci.email = ou.email
JOIN companies c ON ou.id = c.primary_user_id
WHERE c.created >= ci.created;
```

**4. Unconverted invitations (approximate):**
```sql
SELECT ci.*
FROM company_invites ci
LEFT JOIN o_users ou ON ci.email = ou.email
WHERE ou.id IS NULL;
```

---

## Security and Data Integrity Concerns

### Current Issues:

1. **No expiration on tokens** - Invite links work forever
2. **No revocation mechanism** - Can't cancel an invitation
3. **No usage tracking** - Token can theoretically be reused
4. **No conversion link** - Can't definitively tie invite to registration
5. **Email dependency** - Relies on email matching for tracking

### Recommendations for Improvement:

1. **Add conversion tracking fields:**
```sql
ALTER TABLE company_invites
ADD COLUMN company_id INT(11) UNSIGNED NULL,
ADD COLUMN accepted_at DATETIME NULL,
ADD COLUMN status ENUM('pending', 'accepted', 'expired', 'revoked') DEFAULT 'pending',
ADD COLUMN expires_at DATETIME NULL,
ADD FOREIGN KEY (company_id) REFERENCES companies(id);
```

2. **Update invitation flow to set company_id on registration**
3. **Add expiration logic** (e.g., 7 days)
4. **Add status transitions** (pending → accepted/expired/revoked)
5. **Prevent token reuse** - Check status before allowing registration

---

## Summary

**Current State:**
- ✅ Functional invitation system
- ✅ Role-based invite permissions (admin vs customer)
- ✅ Automatic email sending
- ✅ Differentiated account creation (with/without payment)
- ❌ No conversion tracking
- ❌ No analytics capabilities
- ❌ No token expiration or revocation
- ❌ Manual correlation required to determine conversion rate

**To Track Conversions Properly:**
The system needs to be enhanced to link `company_invites` records to created `companies` records, add status tracking, and implement expiration/revocation mechanisms.
