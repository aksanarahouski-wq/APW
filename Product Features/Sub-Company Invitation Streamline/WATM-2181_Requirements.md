# WATM-2181: Streamline Sub-Company Invitation — Payment Method Flag

**Status:** Draft
**Priority:** High
**Jira:** [WATM-2181](https://orases.atlassian.net/browse/WATM-2181)
**Date:** June 28, 2026
**Source:** Client request (Devon D'Andrea) — discussed in APW Check-in, June 23, 2026

---

## Overview

When an APW admin or distributor invites a sub-company, they should be able to specify at invite time whether that sub-company is allowed to create their own payment method. Today this requires a manual post-registration step: someone must go into the company detail page and check the "Allow Payment Method Creation" flag (`allow_self_billing`) after the sub-company has already registered. This creates friction, delays payment setup, and requires follow-up with the sub-company to go back and add their payment method.

This enhancement moves that decision to the invitation itself so the flag is automatically applied when the sub-company registers. The sub-company can then add their payment method on their own time without waiting for someone to manually enable access.

---

## Current Behavior

1. Admin or distributor sends a sub-company invitation (email, account type, parent company)
2. Invitee receives email, clicks link, fills out company profile + user account
3. Company is created with `allow_self_billing = false` (default)
4. **Manual step required:** Someone (APW admin or distributor) must go to the company detail page and toggle "Allow Payment Method Creation" to Yes
5. **Another manual step:** The sub-company must then log in and add their payment method — they couldn't do it before because the flag wasn't set

**Result:** Multi-step process with back-and-forth coordination just to get a sub-company set up to pay.

---

## Scope

### In Scope

1. **Add "Allow Payment Method Creation" option to the invitation form** — when sending a sub-company invitation, the sender can choose whether the new company should be allowed to manage their own payment method
2. **Auto-set `allow_self_billing` flag on registration** — when a sub-company registers via an invitation that has this option enabled, the company's `allow_self_billing` flag is automatically set to `true`
3. **Support both invitation types** — invitations with payment method allowed, and invitations without (current default behavior preserved)

### Out of Scope

- **Device reassignment automation** — automatically transferring devices from a parent/distributor's placeholder payment method to the new sub-company's payment method. Agreed to be an edge case (e.g., the Great Lakes/Primero scenario). Will be handled manually as-is.
- **Retroactive billing optimization** — streamlining the process of charging sub-companies for past months when they finally add a payment method. Deferred per Devon: "Let's just table that for now."
- **Changes to the existing `allow_self_billing` toggle on the company edit page** — the manual toggle remains available for companies not created via invitation, or for changing the setting after the fact.

---

## Functional Requirements

### FR-1: Invitation Form — Payment Method Option

When an admin or distributor creates a sub-company invitation:

- The invitation form must include an "Allow Payment Method Creation" checkbox (or toggle)
- This option should only be visible/applicable when the invitation is for a **Subcustomer** account type (i.e., when `parent_company_id` will be set)
- Default value: **unchecked** (preserves current behavior — sub-companies do not get payment method access by default)
- The selected value must be stored on the invitation record

### FR-2: Automatic Flag Propagation on Company Creation

When a company is created from an accepted invitation:

- If the invitation had "Allow Payment Method Creation" enabled, set the new company's `allow_self_billing` field to `true`
- If the invitation did not have this option enabled, `allow_self_billing` remains `false` (default)
- This must happen automatically — no manual intervention required

### FR-3: Existing Behavior Preserved

- Invitations without the flag (or for non-Subcustomer types) should behave exactly as they do today
- The manual "Allow Payment Method Creation" toggle on the company edit page continues to function independently
- Resending an invitation should preserve the original payment method setting

---

## Current Implementation (Technical Context)

### Key Files

| Layer | File | Purpose |
|-------|------|---------|
| **Invite Form** | `plugins/Companies/templates/Admin/CompanyInvites/invite.php` | Admin form to send invitations |
| **Invite Controller** | `plugins/Companies/src/Controller/Admin/CompanyInvitesController.php` | Create, resend, list invitations |
| **Invite Model** | `plugins/Companies/src/Model/Table/CompanyInvitesTable.php` | Invitation business logic |
| **Invite Entity** | `plugins/Companies/src/Model/Entity/CompanyInvite.php` | Invitation data structure |
| **Accept Controller** | `plugins/Companies/src/Controller/CompaniesController.php` | `invitation()` method handles registration |
| **Accept Template** | `plugins/Companies/templates/Companies/invitation.php` | Public registration form |
| **Company Entity** | `plugins/Companies/src/Model/Entity/Company.php` | Has `allow_self_billing` property |
| **Company Table** | `plugins/Companies/src/Model/Table/CompaniesTable.php` | Auto-sets `allow_self_billing` for non-Subcustomer types |
| **Auth Rules** | `src/Auth/Rules/WatmRules.php` | Gates payment method access via `allow_self_billing` |
| **Company View** | `plugins/Companies/templates/Admin/Companies/view.php` | Shows Payment Methods section only if `allow_self_billing` or no parent |
| **Company Edit** | `plugins/Companies/templates/Admin/Companies/edit.php` | Manual `allow_self_billing` toggle |

### The `allow_self_billing` Flag

- **Column:** `companies.allow_self_billing` (boolean, default `false`)
- **Added in:** Migration `20240419203003_AddAllowSelfBillingToCompanies`
- **UI Label:** "Allow Payment Method Creation"
- **Effect:** Controls whether a sub-company can see/manage payment methods
  - `WatmRules` line 357: Returns `allow_self_billing` for Subcustomers (gates permission)
  - `WatmRules` line 631: Allows billing access if no parent OR `allow_self_billing` is true
  - Company view line 448: Payment Methods section only rendered if no parent or `allow_self_billing`
- **Auto-set logic:** In `CompaniesTable::beforeSave()` (line 690), if account type changes to non-Subcustomer, `allow_self_billing` is auto-set to `true`

### Current Invitation Acceptance Flow

In `CompaniesController::invitation()` (~line 190):

```php
$requirePaymentInfo = true;
if (!empty($companyInvitation->parent_company_id)) {
    $requirePaymentInfo = false;  // Sub-companies always skip
}
```

The registration form itself does not need to change — sub-companies still do not enter payment info during registration. The key change is that after the company is created, the `allow_self_billing` flag must be set based on the invitation's setting, so the sub-company can add their payment method later without manual intervention.

### `company_invites` Table (Current Schema)

| Column | Type | Description |
|--------|------|-------------|
| `id` | INT, unsigned, PK | |
| `token` | VARCHAR(255) | UUID for invitation link |
| `email` | VARCHAR(255) | Invitee email |
| `account_type_id` | INT, unsigned | FK to account_types |
| `parent_company_id` | INT, unsigned, nullable | FK to companies (parent) |
| `company_id` | INT, unsigned, nullable | FK to companies (created on accept) |
| `accepted_at` | DATETIME, nullable | When accepted |
| `status` | ENUM('accepted','pending') | Invitation status |
| `sent_by_user_id` | CHAR(36), nullable | FK to o_users |
| `latest_send_history_id` | INT, nullable | FK to history table |
| `created` | DATETIME | |
| `modified` | DATETIME | |

**Missing:** No column to store the payment method permission choice. A new column (e.g., `allow_self_billing`, boolean, default `false`) needs to be added.

---

## Invitation Flow — Current vs. Enhanced

### Current Flow

```
Admin/Distributor                    Sub-Company                     System
      |                                   |                            |
      |--- Send invitation -------------->|                            |
      |    (email, account type)          |                            |
      |                                   |--- Click link, register -->|
      |                                   |    (no payment form)       |
      |                                   |                            |--- Company created
      |                                   |                            |    allow_self_billing = false
      |                                   |                            |
      |--- Manually toggle flag --------->|                            |--- allow_self_billing = true
      |    (company edit page)            |                            |
      |                                   |                            |
      |--- Contact sub-company ---------->|--- Log back in ---------->|
      |    "go add your payment method"   |    Add payment method      |
      |                                   |                            |
```

### Enhanced Flow

```
Admin/Distributor                    Sub-Company                     System
      |                                   |                            |
      |--- Send invitation -------------->|                            |
      |    (email, account type,          |                            |
      |     allow payment method: YES)    |                            |
      |                                   |--- Click link, register -->|
      |                                   |    (company profile only)  |
      |                                   |                            |--- Company created
      |                                   |                            |    allow_self_billing = true
      |                                   |                            |    (auto-set from invitation)
      |                                   |                            |
      |                                   |--- Log in --------------->|
      |                                   |    Payment Methods visible |
      |                                   |    Add payment method      |
      |                                   |                            |
      |              No manual flag-setting needed                     |
```

---

## Acceptance Criteria / Test Cases

### AC-1: Invitation Form Shows Payment Method Option

**Given** an admin or distributor is on the "Send Invitation" form
**When** the account type is set to Subcustomer
**Then** an "Allow Payment Method Creation" checkbox is visible and defaults to unchecked

**Given** an admin is on the "Send Invitation" form
**When** the account type is set to Distributor or Customer (top-level)
**Then** the "Allow Payment Method Creation" checkbox is not shown (not applicable)

### AC-2: Invitation Saved with Flag

**Given** an admin sends a sub-company invitation with "Allow Payment Method Creation" checked
**When** the invitation is created
**Then** the invitation record stores the payment method flag as `true`

**Given** an admin sends a sub-company invitation without checking "Allow Payment Method Creation"
**When** the invitation is created
**Then** the invitation record stores the payment method flag as `false`

### AC-3: Company Created with Correct Flag

**Given** a sub-company invitation had "Allow Payment Method Creation" enabled
**When** the invitee completes registration
**Then** the new company's `allow_self_billing` is set to `true`
**And** the company detail page shows "Allow Payment Method Creation: Yes"

**Given** a sub-company invitation did not have "Allow Payment Method Creation" enabled
**When** the invitee completes registration
**Then** the new company's `allow_self_billing` remains `false` (current behavior)

### AC-4: Payment Method Access After Registration

**Given** a sub-company was created from an invitation with payment method allowed
**When** the sub-company admin logs into the portal
**Then** they can see the Payment Methods section on their company page
**And** they can add, edit, or remove payment methods

**Given** a sub-company was created from an invitation without payment method allowed
**When** the sub-company admin logs into the portal
**Then** they cannot see the Payment Methods section (current behavior)

### AC-5: Resend Preserves Setting

**Given** an invitation was sent with "Allow Payment Method Creation" enabled
**When** the invitation is resent
**Then** the payment method flag remains `true` on the resent invitation

### AC-6: Existing Invitations Unaffected

**Given** invitations that were created before this enhancement (no payment method flag stored)
**When** those invitations are accepted
**Then** behavior is identical to current — no payment method form, `allow_self_billing` defaults to `false`

### AC-7: Top-Level Company Invitations Unaffected

**Given** an invitation for a Distributor or Customer (no parent company)
**When** the invitee registers
**Then** the payment method form is shown as it always has been (no change to existing behavior)
