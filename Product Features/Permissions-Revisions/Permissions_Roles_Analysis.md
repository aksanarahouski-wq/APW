# WATM Permissions & Roles - Comprehensive Analysis

**Date:** 2026-04-05
**Purpose:** Holistic review of all roles, permissions, and authorization rules in the WATM portal to support adding restrictions to the "Admin" role and/or introducing additional roles.

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Role Definitions](#2-role-definitions)
3. [Account Types (Company Classification)](#3-account-types)
4. [Permission Inventory](#4-permission-inventory)
5. [Role-Permission Matrix](#5-role-permission-matrix)
6. [Route-Level Authorization Rules (permissions.php)](#6-route-level-authorization-rules)
7. [Custom Authorization Rule Methods (WatmRules)](#7-custom-authorization-rule-methods)
8. [Superuser Bypass](#8-superuser-bypass)
9. [Observations & Gaps](#9-observations--gaps)
10. [Source Files Reference](#10-source-files-reference)

---

## 1. System Overview

WATM uses a **hybrid RBAC system** built on three layers:

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Role Assignment** | `o_users.role` field (`admin` or `customer`) | Broad role category on the user record |
| **Granular Permissions** | CakeDC/Auth + `o_users_roles` / `o_users_permissions` / junction table | Database-driven permission matrix per role |
| **Custom Business Rules** | `WatmRules`, `WATMCustomerRules`, `RmaRules`, `OrasesOnlyRules` | Runtime authorization checks (company hierarchy, account type, etc.) |

### Database Tables

| Table | Purpose |
|-------|---------|
| `o_users` | User accounts; `role` = 'admin' or 'customer'; `is_superuser` flag |
| `o_users_roles` | Role definitions with `title` and `role_type` |
| `o_users_permissions` | Individual permission definitions (keyword, route mapping, run_method) |
| `o_users_roles_o_users_permissions` | Junction table mapping roles to permissions |
| `company_users` | Links users to companies (multi-tenant isolation) |
| `account_types` | Company classifications controlling feature flags (can_assign, can_upcharge) |

### Authorization Flow

```
Request --> CakeDC/Auth middleware
  --> Check permissions.php rules (top to bottom, first match wins)
    --> bypassAuth? --> Allow without login
    --> role match? --> Check 'allowed' value
    --> allowed = Rule object? --> Execute custom rule method
    --> Catch-all --> WatmRules.orasesDefaultAllow()
      --> Check is_superuser (bypass all)
      --> Check role is 'admin' or 'customer'
      --> Check DB permission exists for user's role
      --> If run_method set, execute additional check
```

---

## 2. Role Definitions

### The 4 Effective User Roles

The WATM system has **4 effective user roles**, though the database only stores 3 role records. The 4th distinction (Global Super Admin vs. Global Admin) is implemented via the `is_superuser` flag on the user record rather than as a separate database role.

| # | Role (Functional) | Database Implementation | Description |
|---|---|---|---|
| 1 | **Global WATM Super Admin** | `role=admin` + `is_superuser=1` | Orases dev team and APW admin staff with unrestricted access. Bypasses ALL permission checks. |
| 2 | **Global Admin** | `role=admin` + `is_superuser=0` | WATM admins (APW staff) subject to the permission matrix. Limited to 55 seeded permissions. |
| 3 | **Company Super Admin** | `role=customer`, role title "Super Admin" | Premium customer portal user (distributor-level). 63 seeded permissions. |
| 4 | **Company Admin** | `role=customer`, role title "Admin" | Standard customer portal user with limited access. 37 seeded permissions. |

### How the 4 Roles Map to the Database

```
o_users_roles table (3 records):
  ┌──────────────────────────────────────────────────┐
  │ title="Admin"       │ role_type="admin"           │ ── Used by roles #1 and #2
  │ title="Super Admin" │ role_type="customer"         │ ── Used by role #3
  │ title="Admin"       │ role_type="customer"         │ ── Used by role #4
  └──────────────────────────────────────────────────┘

o_users table:
  ┌──────────────────────────────────────────────────┐
  │ role="admin"    │ is_superuser=1  │ ── Role #1: Global WATM Super Admin
  │ role="admin"    │ is_superuser=0  │ ── Role #2: Global Admin
  │ role="customer" │ is_superuser=0  │ ── Role #3 or #4 (determined by o_users_roles assignment)
  └──────────────────────────────────────────────────┘
```

> **Note:** There are two roles titled "Admin" in `o_users_roles` - one with `role_type=admin` (WATM staff) and one with `role_type=customer` (customer portal). They are distinct database records differentiated by `role_type`.

### Critical Nuance: The Superuser Gap

The distinction between Role #1 (Global Super Admin) and Role #2 (Global Admin) has a **significant practical impact**:

- **Role #1 (is_superuser=1):** Bypasses the entire permission matrix. Has access to everything regardless of what permissions are seeded.
- **Role #2 (is_superuser=0):** Is subject to the 55 permissions assigned to the "Admin" role in the seed data. However, **~30 critical permissions are NOT seeded for this role**, including:
  - Billing cycle management (browse, close, generate, regenerate)
  - Invoice operations (generate, regenerate, send email, edit exclusions)
  - Admin user management (edit, delete)
  - Device imports and deactivation
  - Service plan management (edit, delete, approve)
  - All config management (edit, delete, approve for both configs and config groups)
  - RMA management (edit, export, delete)
  - Notification management (edit, delete, approve)
  - FAQ management (edit, delete)

**This means:** If a non-superuser Global Admin account exists today, they would be locked out of most write operations for billing, configs, service plans, RMAs, and system notifications. The system has historically relied on the superuser flag to grant full access, making the Admin permission seed incomplete.

**Before adding restrictions or new admin-level roles, the Admin role's seeded permissions must first be audited and corrected to establish a proper baseline.**

### Removed Roles

| Role | Removed By | Date |
|------|-----------|------|
| Call Center Admin | Migration `20230327144040_RemoveCallCenterAdminRole.php` | 2023-03-27 |

---

## 3. Account Types

Account types are a **separate axis of authorization** applied to companies (not users directly). They control feature flags that custom rule methods check at runtime.

| Account Type | `can_assign` | `can_upcharge` | Description |
|-------------|-------------|---------------|-------------|
| **Distributor** | Yes | Yes | Parent company; can assign devices to sub-companies, set upcharges |
| **Customer** | Yes | No | Standard company; can assign devices but no upcharge capability |
| **Subcustomer** | No | No | Child company; cannot assign devices or set upcharges |

These are checked by `WatmRules` methods like `orasesIsPremiumCustomer()`, `orasesCanAssign()`, `orasesIsCustomerNotSub()`, etc.

---

## 4. Permission Inventory

### All Permissions by Group (92 total from seed data)

#### Companies Group (27 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `companies_browse` | Browse Companies | Companies/Companies::index | IsCustomerNotSub |
| 2 | `company_view` | View Company | Companies/Companies::view | IsCustomerNotSub |
| 3 | `company_edit` | Edit Company | Companies/Companies::edit | IsCustomerNotSubOrSelf |
| 4 | `company_merge` | Merge Companies | Companies/Companies::merge | _(none - admin only)_ |
| 5 | `company_deactivate` | Deactivate Company | Companies/Companies::deactivate | _(none)_ |
| 6 | `company_edit_protected_fields` | Edit Company Protected Fields | Companies/Companies::edit_protected_fields | IsPremiumCustomer |
| 7 | `company_edit_dual_sim_upcharge` | Edit Dual Sim Upcharge | Companies/Companies::editDualSimUpcharge | IsPremiumCustomerNotSelf |
| 8 | `company_allow_self_billing` | Edit Allow Self Billing | Companies/Companies::allowSelfBilling | IsPremiumCustomerNotSelf |
| 9 | `company_view_self_billing` | View Allow Self Billing | Companies/Companies::viewSelfBilling | IsPremiumCustomer |
| 10 | `company_payouts` | Company Payouts | Companies/Commissions::companyPayouts | IsPremiumCustomer |
| 11 | `company_invites` | Company Invites | Companies/CompanyInvites::invite | IsParentCustomer |
| 12 | `company_employees_browse` | Browse Company Employees | Companies/CompanyUsers::index | MustBelongToCompany |
| 13 | `company_employees_view` | View Company Employees | Companies/CompanyUsers::view | MustBelongToCompany |
| 14 | `company_employees_edit` | Edit Company Employees | Companies/CompanyUsers::edit | MustBelongToCompany |
| 15 | `company_employees_delete` | Delete Company Employees | Companies/CompanyUsers::delete | MustBelongToCompany |
| 16 | `company_payment_methods_browse` | Browse Company Payment Methods | Companies/CompanyPaymentMethods::index | MustBelongToCompany |
| 17 | `company_payment_methods_view` | View Company Payment Method | Companies/CompanyPaymentMethods::view | MustBelongToCompany |
| 18 | `company_payment_methods_edit` | Edit Company Payment Method | Companies/CompanyPaymentMethods::edit | MustBelongToCompany |
| 19 | `company_payment_methods_assign` | Assign Company Payment Method | Companies/CompanyPaymentMethods::assign | MustBelongToCompany |
| 20 | `company_payment_methods_delete` | Delete Company Payment Method | Companies/CompanyPaymentMethods::delete | MustBelongToCompany |
| 21 | `company_payment_methods_export` | Export Payment Methods | Companies/CompanyPaymentMethods::export | IsPremiumCustomer |
| 22 | `company_payment_methods_credit` | Credit Company Payment Method | Companies/CompanyPaymentMethods::credit | _(none - admin only)_ |
| 23 | `company_credits_browse` | Browse Company Credits | Companies/CompanyCredits::index | MustBelongToCompany |
| 24 | `company_credits_edit` | Edit Company Credit | Companies/CompanyCredits::edit | _(none - admin only)_ |
| 25 | `company_credits_delete` | Delete Company Credit | Companies/CompanyCredits::delete | _(none - admin only)_ |
| 26 | `company_tax_documents_browse` | View Company Tax Documents | Companies/CompanyTaxDocuments::index | IsPrimaryUserOfDistributor |
| 27 | `company_power_management_browse` | Browse Company Power Management | Companies/CompanyPowerSchedules::index | MustBelongToCompany |
| 28 | `company_power_management_edit` | Edit Company Power Management | Companies/CompanyPowerSchedules::edit | MustBelongToCompany |
| 29 | `company_power_management_delete` | Delete Company Power Management | Companies/CompanyPowerSchedules::delete | MustBelongToCompany |

#### API Keys Group (9 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `company_api_keys_browse` | Browse Company API Keys | Companies/CompanyApiKeys::index | _(none)_ |
| 2 | `company_api_keys_view` | View Company API Key | Companies/CompanyApiKeys::view | _(none)_ |
| 3 | `company_api_keys_add` | Add Company API Key | Companies/CompanyApiKeys::add | _(none)_ |
| 4 | `company_api_keys_edit` | Edit Company API Key | Companies/CompanyApiKeys::edit | _(none)_ |
| 5 | `company_api_keys_delete` | Delete Company API Key | Companies/CompanyApiKeys::delete | _(none)_ |
| 6 | `company_api_keys_revoke` | Revoke Company API Key | Companies/CompanyApiKeys::revoke | _(none)_ |
| 7 | `company_api_keys_activate` | Activate Company API Key | Companies/CompanyApiKeys::activate | _(none)_ |
| 8 | `company_api_keys_aws_usage` | AWS Usage for API Key | Companies/CompanyApiKeys::awsUsage | _(none)_ |
| 9 | `company_api_keys_company_usage` | Company Usage for API Key | Companies/CompanyApiKeys::companyUsage | _(none)_ |

#### Commissions Group (3 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `commissions_browse` | Browse Commissions | Companies/Commissions::commissions | _(none - admin only)_ |
| 2 | `commissions_preferences` | Commissions Preferences | Companies/Commissions::preferences | IsPremiumCustomer |
| 3 | `browse_company_earnings_reports` | Browse Company Earnings Reports | Companies/CompanyEarningsReports::index | IsPremiumCustomer |

#### Notifications Group (8 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `company_notifications_browse` | Browse Company Notifications | Notifications/CompanyNotifications::index | _(none)_ |
| 2 | `company_notification_view` | View Company Notification | Notifications/CompanyNotifications::view | _(none)_ |
| 3 | `company_notification_edit` | Edit Company Notifications | Notifications/CompanyNotifications::edit | _(none)_ |
| 4 | `company_notification_delete` | Delete Company Notifications | Notifications/CompanyNotifications::delete | _(none)_ |
| 5 | `company_notification_log_view` | View Logged Company Notification | Notifications/CompanyNotifications::viewNotificationLog | _(none)_ |
| 6 | `subcompany_global_notification_browse` | Browse Subcompany Global Notifications | Notifications/SubcompanyGlobalNotifications::index | _(none)_ |
| 7 | `subcompany_global_notification_view` | View Subcompany Global Notifications | Notifications/SubcompanyGlobalNotifications::view | _(none)_ |
| 8 | `subcompany_global_notification_edit` | Edit Subcompany Global Notifications | Notifications/SubcompanyGlobalNotifications::edit | _(none)_ |

#### Billing Group (10 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `billing_cycles_browse` | Manage Billing Cycles | Billing/BillingCycles::index | _(none - admin only)_ |
| 2 | `billing_cycle_close` | Close Billing Cycle | Billing/Invoices::closeBillingCycle | _(none - admin only)_ |
| 3 | `invoices_browse` | Browse Invoices | Billing/Invoices::index | HasOwnPaymentMethod |
| 4 | `invoices_view` | View Invoices | Billing/Invoices::view | HasOwnPaymentMethod |
| 5 | `invoices_preview` | Preview Invoices | Billing/Invoices::viewFile | HasOwnPaymentMethod |
| 6 | `invoices_generate` | Generate Invoices | Billing/Invoices::generate | _(none - admin only)_ |
| 7 | `invoices_regenerate` | Regenerate Invoices | Billing/Invoices::regenerate | _(none - admin only)_ |
| 8 | `invoices_send_email` | Send Invoice Email | Billing/Invoices::sendInvoiceEmailsForActiveBillingCycle | _(none - admin only)_ |
| 9 | `invoices_edit_exclusions` | Edit Invoice Exclusions | Billing/Invoices::saveExclusion | _(none - admin only)_ |
| 10 | `view_pricing_info` | View Pricing Info | Billing/BillingCycles::viewPricingInfo | canViewPricing |
| 11 | `billing_generate_report` | Generate Billing Report | Billing/BillingCycles::generateBillingReport | _(none - admin only)_ |
| 12 | `billing_download_report` | Download Billing Report | Billing/BillingCycles::downloadBillingReport | _(none - admin only)_ |

#### Payment Methods Group (2 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `payment_methods_browse` | Browse Payment Methods | Companies/PaymentMethods::index | _(none - admin only)_ |
| 2 | `payment_methods_report` | Payment Methods Report | Companies/PaymentMethods::report | _(none - admin only)_ |

#### Administrators Group (3 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `admins_browse` | Browse Admin Users | Admin/Admins::index | _(none - admin only)_ |
| 2 | `admin_view` | View Admin User | Admin/Admins::view | _(none - admin only)_ |
| 3 | `admin_edit` | Edit Admin User | Admin/Admins::edit | _(none - admin only)_ |
| 4 | `admin_delete` | Delete Admin User | Admin/Admins::delete | _(none - admin only)_ |

#### Devices Group (14 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `devices_browse` | Browse Devices | Devices/Devices::index | _(none)_ |
| 2 | `devices_view` | View Devices | Devices/Devices::view | MustBelongToCompany |
| 3 | `devices_edit` | Edit Devices | Devices/Devices::edit | MustBelongToCompany |
| 4 | `devices_ping` | Ping Devices | Devices/Devices::ping | MustBelongToCompany |
| 5 | `devices_assign` | Assign Devices | Devices/Devices::assign | CanAssign |
| 6 | `devices_import` | Import Devices | Devices/Devices::import | _(none - admin only)_ |
| 7 | `devices_bulk_actions` | Devices Bulk Actions | Devices/Devices::bulkActions | _(none)_ |
| 8 | `devices_export` | Export Devices | Devices/Devices::export | _(none)_ |
| 9 | `devices_export_download` | Download Devices Export | Devices/Devices::downloadExport | _(none)_ |
| 10 | `device_checkins_export` | Export Device Checkins | Devices/Devices::exportCheckins | MustBelongToCompany |
| 11 | `devices_alerts_export` | Export Device Alerts | Devices/Devices::exportRecentAlerts | _(none)_ |
| 12 | `devices_deactivate_payment_method` | Devices Deactivate | Companies/PaymentMethods::deactivateDevices | _(none - admin only)_ |

#### RMAs Group (5 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `rmas_browse` | Browse RMAs | Devices/Rmas::index | _(none)_ |
| 2 | `rma_view` | View RMA | Devices/Rmas::view | _(none)_ |
| 3 | `rma_edit` | Edit RMA | Devices/Rmas::edit | _(none)_ |
| 4 | `rma_export` | Export RMAs | Devices/Rmas::export | _(none)_ |
| 5 | `rma_delete` | Delete RMA | Devices/Rmas::delete | _(none)_ |

#### Service Plans Group (6 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `service_plan_browse` | Browse Service Plans | Devices/ServicePlans::index | _(none)_ |
| 2 | `service_plan_view` | View Service Plan | Devices/ServicePlans::view | _(none)_ |
| 3 | `service_plan_edit` | Edit Service Plan | Devices/ServicePlans::edit | _(none - admin only)_ |
| 4 | `service_plan_delete` | Delete Service Plan | Devices/ServicePlans::delete | _(none - admin only)_ |
| 5 | `service_plan_approve` | Approve Service Plan | Devices/ServicePlans::approve | _(none - admin only)_ |
| 6 | `customize_service_plan` | Customize Service Plan | Devices/CompanyServicePlans::customize | IsDistributorAndCanUpcharge |
| 7 | `edit_custom_service_plan` | Edit Custom Service Plan | Devices/CompanyServicePlans::edit | IsDistributorAndCanUpcharge |
| 8 | `delete_custom_service_plan` | Delete Custom Service Plan | Devices/CompanyServicePlans::delete | IsDistributorAndCanUpcharge |

#### Device Configuration Group (7 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `config_groups_browse` | Browse Config Groups | Devices/ConfigGroups::index | _(none - admin only)_ |
| 2 | `config_group_view` | View Config Group | Devices/ConfigGroups::view | _(none - admin only)_ |
| 3 | `config_group_edit` | Edit Config Group | Devices/ConfigGroups::edit | _(none - admin only)_ |
| 4 | `config_group_delete` | Delete Config Group | Devices/ConfigGroups::delete | _(none - admin only)_ |
| 5 | `configs_browse` | Browse Configs | Devices/Configs::index | _(none - admin only)_ |
| 6 | `config_view` | View Config | Devices/Configs::view | _(none - admin only)_ |
| 7 | `config_edit` | Edit Config | Devices/Configs::edit | _(none - admin only)_ |
| 8 | `config_delete` | Delete Config | Devices/Configs::delete | _(none - admin only)_ |
| 9 | `config_approve` | Approve Config | Devices/Configs::approve | _(none - admin only)_ |

#### Check-in Failures Group (2 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `checkin_fail_browse` | Browse Check-in Failures | Devices/CheckinFailures::index | _(none - admin only)_ |
| 2 | `checkin_fail_view` | View Check-in Failure | Devices/CheckinFailures::view | _(none - admin only)_ |

#### White Label Group (7 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `white_label_view` | View White Labels | Companies/WhiteLabels::view | IsPremiumCustomer |
| 2 | `white_label_edit` | Edit White Labels | Companies/WhiteLabels::edit | IsPremiumCustomer |
| 3 | `white_label_invoice_preview` | White Labels Invoice Preview | Companies/WhiteLabels::invoicePreview | IsPremiumCustomer |
| 4 | `white_label_manufacturers_edit` | Edit White Labels Manufacturers | Companies/WhiteLabelManufacturers::edit | IsPremiumCustomer |
| 5 | `white_label_manufacturers_delete` | Delete White Labels Manufacturers | Companies/WhiteLabelManufacturers::delete | IsPremiumCustomer |
| 6 | `white_label_models_edit` | Edit White Labels Models | Companies/WhiteLabelModels::edit | IsPremiumCustomer |
| 7 | `white_label_models_delete` | Delete White Labels Models | Companies/WhiteLabelModels::delete | IsPremiumCustomer |

#### System & Logging Group (4 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `logs_browse` | Browse Logs | _(Orases/Logs)_ | _(none)_ |
| 2 | `log_view` | View Log | _(Orases/Logs)_ | _(none)_ |
| 3 | `notifications_export` | Export Notifications | _(Notifications)_ | _(none)_ |
| 4 | `devices_alerts_export` | Export Device Alerts | _(Devices)_ | _(none)_ |

#### Notifications Management Group (5 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `notification_browse` | Browse Notifications | Notifications/Notifications::index | _(none - admin only)_ |
| 2 | `notification_view` | View Notification | Notifications/Notifications::view | _(none - admin only)_ |
| 3 | `notification_edit` | Edit Notification | Notifications/Notifications::edit | _(none - admin only)_ |
| 4 | `notification_delete` | Delete Notification | Notifications/Notifications::delete | _(none - admin only)_ |
| 5 | `notification_log_view` | View Notification Log | Notifications/Notifications::viewLog | _(none)_ |
| 6 | `notification_approve` | Approve Notification | Notifications/Notifications::approve | _(none - admin only)_ |

#### FAQ Group (4 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `faq_browse` | Browse FAQs | _(FAQ controller)_ | _(none)_ |
| 2 | `faq_view` | View FAQ | _(FAQ controller)_ | _(none)_ |
| 3 | `faq_edit` | Edit FAQ | _(FAQ controller)_ | _(none - admin only)_ |
| 4 | `faq_delete` | Delete FAQ | _(FAQ controller)_ | _(none - admin only)_ |

#### Other (2 permissions)

| # | Keyword | Title | Route | Run Method |
|---|---------|-------|-------|------------|
| 1 | `support_ticket_edit` | Support Ticket | _(support)_ | _(none)_ |
| 2 | `profile_edit` | Edit Profile | _(user profile)_ | _(none)_ |

---

## 5. Role-Permission Matrix

### Legend
- **ALL** = Access granted via superuser bypass (not subject to permission checks)
- **Y** = Has permission (via seed data)
- **-** = Does NOT have permission
- Permissions marked with a `run_method` have additional runtime checks even when granted

> **Important:** The "Global Super Admin" column shows ALL for every permission because `is_superuser=1` bypasses the entire matrix. The "Global Admin" column reflects what a non-superuser admin account would actually have based on seed data - gaps here represent permissions that are **missing** and would need to be added before creating non-superuser admin accounts.

| Permission Keyword | Global Super Admin | Global Admin | Company Super Admin | Company Admin |
|---|:---:|:---:|:---:|:---:|
| **COMPANIES** | | | | |
| `companies_browse` | ALL | Y | Y | Y |
| `company_view` | ALL | Y | Y | Y |
| `company_view_self_billing` | ALL | Y | Y | Y |
| `company_edit` | ALL | Y | Y | - |
| `company_merge` | ALL | Y | - | - |
| `company_deactivate` | ALL | - | Y | - |
| `company_edit_protected_fields` | ALL | - | - | - |
| `company_edit_dual_sim_upcharge` | ALL | - | Y | - |
| `company_allow_self_billing` | ALL | - | Y | - |
| `company_payouts` | ALL | - | Y | - |
| `company_invites` | ALL | - | Y | - |
| **EMPLOYEES** | | | | |
| `company_employees_browse` | ALL | Y | Y | Y |
| `company_employees_view` | ALL | Y | Y | Y |
| `company_employees_edit` | ALL | - | Y | - |
| `company_employees_delete` | ALL | - | Y | - |
| **PAYMENT METHODS (Company)** | | | | |
| `company_payment_methods_browse` | ALL | Y | Y | Y |
| `company_payment_methods_view` | ALL | Y | Y | Y |
| `company_payment_methods_edit` | ALL | - | Y | - |
| `company_payment_methods_assign` | ALL | - | Y | - |
| `company_payment_methods_delete` | ALL | - | Y | - |
| `company_payment_methods_export` | ALL | - | Y | - |
| `company_payment_methods_credit` | ALL | - | - | - |
| **PAYMENT METHODS (Global)** | | | | |
| `payment_methods_browse` | ALL | Y | - | - |
| `payment_methods_report` | ALL | Y | - | - |
| **API KEYS** | | | | |
| `company_api_keys_browse` | ALL | Y | Y | Y |
| `company_api_keys_view` | ALL | Y | Y | Y |
| `company_api_keys_add` | ALL | Y | Y | - |
| `company_api_keys_edit` | ALL | Y | Y | - |
| `company_api_keys_delete` | ALL | Y | Y | - |
| `company_api_keys_revoke` | ALL | Y | Y | - |
| `company_api_keys_activate` | ALL | Y | Y | - |
| `company_api_keys_aws_usage` | ALL | Y | Y | Y |
| `company_api_keys_company_usage` | ALL | Y | Y | Y |
| **COMMISSIONS** | | | | |
| `commissions_browse` | ALL | Y | - | - |
| `commissions_preferences` | ALL | - | Y | - |
| `browse_company_earnings_reports` | ALL | - | Y | Y |
| **NOTIFICATIONS (Company)** | | | | |
| `company_notifications_browse` | ALL | Y | Y | Y |
| `company_notification_view` | ALL | Y | Y | Y |
| `company_notification_edit` | ALL | - | Y | - |
| `company_notification_delete` | ALL | - | Y | - |
| `company_notification_log_view` | ALL | Y | Y | Y |
| `subcompany_global_notification_browse` | ALL | - | Y | Y |
| `subcompany_global_notification_view` | ALL | - | Y | Y |
| `subcompany_global_notification_edit` | ALL | - | Y | Y |
| **NOTIFICATIONS (System-level)** | | | | |
| `notification_browse` | ALL | Y | - | - |
| `notification_view` | ALL | Y | - | - |
| `notification_log_view` | ALL | Y | Y | Y |
| `notifications_export` | ALL | Y | Y | Y |
| `devices_alerts_export` | ALL | Y | Y | Y |
| **POWER MANAGEMENT** | | | | |
| `company_power_management_browse` | ALL | - | Y | Y |
| `company_power_management_edit` | ALL | - | Y | Y |
| `company_power_management_delete` | ALL | - | Y | Y |
| **CREDITS** | | | | |
| `company_credits_browse` | ALL | Y | Y | Y |
| `company_credits_edit` | ALL | - | - | - |
| `company_credits_delete` | ALL | - | - | - |
| **TAX DOCUMENTS** | | | | |
| `company_tax_documents_browse` | ALL | - | Y | - |
| **WHITE LABEL** | | | | |
| `white_label_view` | ALL | Y | Y | - |
| `white_label_edit` | ALL | - | Y | - |
| `white_label_invoice_preview` | ALL | - | Y | - |
| `white_label_manufacturers_edit` | ALL | - | Y | - |
| `white_label_manufacturers_delete` | ALL | - | Y | - |
| `white_label_models_edit` | ALL | - | Y | - |
| `white_label_models_delete` | ALL | - | Y | - |
| **BILLING** | | | | |
| `billing_cycles_browse` | ALL | **MISSING** | - | - |
| `billing_cycle_close` | ALL | **MISSING** | - | - |
| `invoices_browse` | ALL | Y | Y | Y |
| `invoices_view` | ALL | Y | Y | Y |
| `invoices_preview` | ALL | Y | Y | Y |
| `invoices_generate` | ALL | **MISSING** | - | - |
| `invoices_regenerate` | ALL | **MISSING** | - | - |
| `invoices_send_email` | ALL | **MISSING** | - | - |
| `invoices_edit_exclusions` | ALL | **MISSING** | - | - |
| `view_pricing_info` | ALL | Y | Y | Y |
| `billing_generate_report` | ALL | Y | - | - |
| `billing_download_report` | ALL | Y | - | - |
| **ADMINISTRATORS** | | | | |
| `admins_browse` | ALL | Y | - | - |
| `admin_view` | ALL | Y | - | - |
| `admin_edit` | ALL | **MISSING** | - | - |
| `admin_delete` | ALL | **MISSING** | - | - |
| **DEVICES** | | | | |
| `devices_browse` | ALL | Y | Y | Y |
| `devices_view` | ALL | Y | Y | Y |
| `devices_edit` | ALL | Y | Y | - |
| `devices_ping` | ALL | Y | Y | Y |
| `devices_assign` | ALL | - | Y | - |
| `devices_import` | ALL | **MISSING** | - | - |
| `devices_bulk_actions` | ALL | Y | Y | - |
| `devices_export` | ALL | Y | Y | Y |
| `devices_export_download` | ALL | Y | Y | Y |
| `device_checkins_export` | ALL | Y | Y | Y |
| `devices_deactivate_payment_method` | ALL | **MISSING** | - | - |
| **RMAs** | | | | |
| `rmas_browse` | ALL | Y | - | - |
| `rma_view` | ALL | Y | - | - |
| `rma_edit` | ALL | **MISSING** | - | - |
| `rma_export` | ALL | **MISSING** | - | - |
| `rma_delete` | ALL | **MISSING** | - | - |
| **SERVICE PLANS** | | | | |
| `service_plan_browse` | ALL | Y | - | - |
| `service_plan_view` | ALL | Y | - | - |
| `service_plan_edit` | ALL | **MISSING** | - | - |
| `service_plan_delete` | ALL | **MISSING** | - | - |
| `service_plan_approve` | ALL | **MISSING** | - | - |
| `customize_service_plan` | ALL | - | Y | - |
| `edit_custom_service_plan` | ALL | - | Y | - |
| `delete_custom_service_plan` | ALL | - | Y | - |
| **CONFIG MANAGEMENT** | | | | |
| `config_groups_browse` | ALL | Y | - | - |
| `config_group_view` | ALL | Y | - | - |
| `config_group_edit` | ALL | **MISSING** | - | - |
| `config_group_delete` | ALL | **MISSING** | - | - |
| `configs_browse` | ALL | Y | - | - |
| `config_view` | ALL | Y | - | - |
| `config_edit` | ALL | **MISSING** | - | - |
| `config_delete` | ALL | **MISSING** | - | - |
| `config_approve` | ALL | **MISSING** | - | - |
| **CHECK-IN FAILURES** | | | | |
| `checkin_fail_browse` | ALL | Y | - | - |
| `checkin_fail_view` | ALL | Y | - | - |
| **FAQ** | | | | |
| `faq_browse` | ALL | Y | - | - |
| `faq_view` | ALL | Y | Y | Y |
| `faq_edit` | ALL | **MISSING** | - | - |
| `faq_delete` | ALL | **MISSING** | - | - |
| **LOGS** | | | | |
| `logs_browse` | ALL | Y | Y | - |
| `log_view` | ALL | Y | Y | - |
| **OTHER** | | | | |
| `support_ticket_edit` | ALL | Y | Y | Y |
| `profile_edit` | ALL | Y | Y | Y |

### Permission Count Summary

| Role | Seeded Permissions | Missing Permissions | Effective Access | Total Available |
|------|:---:|:---:|:---:|:---:|
| **Global Super Admin** (is_superuser=1) | 55 | N/A | **ALL (92+)** | 92+ |
| **Global Admin** (is_superuser=0) | 55 | **~30** | 55 | 92+ |
| **Company Super Admin** (customer) | 63 | 0 | 63 | 92+ |
| **Company Admin** (customer) | 37 | 0 | 37 | 92+ |

### Global Admin Missing Permissions (30 total)

These permissions exist in the database but are **NOT seeded** for the Global Admin role. They are only accessible via superuser bypass. If a non-superuser Global Admin account is created, these features would be inaccessible:

| Category | Missing Permissions | Impact |
|----------|---|---|
| **Billing Operations** | `billing_cycles_browse`, `billing_cycle_close`, `invoices_generate`, `invoices_regenerate`, `invoices_send_email`, `invoices_edit_exclusions` | Cannot manage billing cycles or generate/send invoices |
| **Admin User Management** | `admin_edit`, `admin_delete` | Can browse/view admins but cannot edit or delete them |
| **Device Operations** | `devices_import`, `devices_deactivate_payment_method` | Cannot import devices or deactivate payment methods |
| **RMA Management** | `rma_edit`, `rma_export`, `rma_delete` | Can browse/view RMAs but cannot manage them |
| **Service Plans** | `service_plan_edit`, `service_plan_delete`, `service_plan_approve` | Can browse/view plans but cannot manage them |
| **Config Management** | `config_group_edit`, `config_group_delete`, `config_edit`, `config_delete`, `config_approve` | Can browse/view configs but cannot manage them |
| **Notification Templates** | `notification_edit`, `notification_delete`, `notification_approve` | Can browse/view notification templates but cannot manage them |
| **FAQ Management** | `faq_edit`, `faq_delete` | Can browse/view FAQs but cannot edit/delete them |
| **Company Features** | `company_deactivate`, `company_edit_protected_fields`, `company_payment_methods_credit`, `company_credits_edit`, `company_credits_delete`, `company_tax_documents_browse` | Various company management gaps |
| **Customer-Specific** | `company_power_management_*`, `subcompany_global_notification_*`, `company_notification_edit/delete`, `company_payment_methods_edit/assign/delete/export`, `company_employees_edit/delete`, various customer features | Features designed for customer roles (may be intentionally excluded from admin) |

> **Recommendation:** Before implementing any role restrictions or new roles, a decision is needed on whether the "missing" permissions for Global Admin are (a) intentional omissions because superuser was always the plan, or (b) oversights that need to be corrected. This determines the baseline for any role restructuring work.

---

## 6. Route-Level Authorization Rules

The `config/permissions.php` file defines route-level rules evaluated **top to bottom** (first match wins):

### Bypass Auth Routes (No Login Required)
| Route | Purpose |
|-------|---------|
| Admin/UserBehaviors (login, logout, reset, etc.) | Authentication flows |
| Api/Devices/VerizonCallbacks | Verizon webhook callbacks |
| Api/Devices/Devices | Device API endpoints |
| Api/V1/Devices/Devices | REST API v1 (uses API key auth) |
| Api/WoocommerceApi/WoocommerceIntegrations | WooCommerce integration |
| AdobeSign/AdobeSign | AdobeSign verification |
| Companies/Companies::index (no prefix) | WooCommerce customer creation |
| Companies/Companies::invitation (no prefix) | Customer invitation flow |
| Companies/Companies::accessList (no prefix) | VPN access list |
| DebugKit | Debug toolbar |

### Allow All Authenticated Users
| Route | Purpose |
|-------|---------|
| Dashboard/Dashboard::index | Main dashboard |
| UserManagement/OUserPreferredFilters::edit | Filter preferences |
| Devices/Devices::getLocation | Device location lookup |
| Devices/Devices::updatePassword | Device password update |
| Devices/Devices::dailyUsage | Daily usage data |
| Api/Devices/DeviceModels::* | Device model dropdown data |
| Api/Companies/Companies::* | Companies API |
| Orases/Files/OFiles (upload, download, etc.) | File operations |
| Billing/Invoices (downloadFile, viewFile) | Invoice file access |
| Companies/CompanyTaxDocuments (view, download) | Tax document access |
| Companies/ApiDocumentation::* | API documentation |
| Companies/CompanyPowerSchedules (various actions) | Power schedule operations |

### Role-Specific Route Rules
| Route | Role | Rule |
|-------|------|------|
| Companies/Companies::myCompany | `customer` | Allow (customer portal landing) |
| Companies/Companies::view | `customer` | `WATMCustomerRules` (limits to own/sub companies) |
| Devices/Rmas::edit | `*` | `RmaRules` (status-based restriction) |
| Devices/DeviceManufacturers CRUD | `admin` | Allow (admin only) |
| Devices/DeviceModels CRUD | `admin` | Allow (admin only) |

### Catch-All
All remaining routes fall through to `WatmRules()` which checks database permissions + run_methods.

---

## 7. Custom Authorization Rule Methods

### WatmRules (`src/Auth/Rules/WatmRules.php` - 638 lines)

| Method | Description | Used By |
|--------|-------------|---------|
| `orasesDefaultAllow` | Base check: superuser bypass, role must be admin/customer, checks DB permissions + run_method | Catch-all rule |
| `orasesMustBelongToCompany` | Resource must belong to user's company or child companies | `MustBelongToCompany` run_method |
| `orasesIsPremiumCustomer` | Company has `can_upcharge` flag | `IsPremiumCustomer` run_method |
| `orasesAllowUpcharging` | Company allows upcharging AND account type permits it | _(internal)_ |
| `orasesIsCustomerNotSub` | Account type is NOT "Subcustomer" | `IsCustomerNotSub` run_method |
| `orasesCanViewPricing` | Premium customer OR self-billing subcustomer | `canViewPricing` run_method |
| `orasesIsPremiumCustomerOrSelf` | Premium customer OR editing own company | `IsPremiumCustomerOrSelf` run_method |
| `orasesIsCustomerNotSubOrSelf` | Not subcustomer OR editing own company | `IsCustomerNotSubOrSelf` run_method |
| `orasesIsCustomerSelf` | User is editing their own company | _(internal)_ |
| `orasesCanAssign` | Account type has `can_assign` flag | `CanAssign` run_method |
| `orasesIsCustomerActive` | Company status is "active" | `IsCustomerActive` run_method |
| `orasesIsChildCustomer` | Target company is child of user's company | `IsChildCustomer` run_method |
| `orasesIsCustomerActiveAndChild` | Active AND child company | _(internal)_ |
| `orasesIsPremiumCustomerNotSelf` | Premium, can upcharge, NOT editing self | `IsPremiumCustomerNotSelf` run_method |
| `orasesIsPremiumCustomerActiveAndChild` | Premium, active, child company, upcharging allowed | _(internal)_ |
| `orasesIsDistributorAndCanUpcharge` | Distributor with upcharge capability | `IsDistributorAndCanUpcharge` run_method |
| `orasesIsParentCustomer` | Active company with `can_assign` | `IsParentCustomer` run_method |
| `orasesIsPrimaryUserOfDistributor` | Primary user of premium company OR employee with doc access | `IsPrimaryUserOfDistributor` run_method |
| `orasesIsPrimary` | User is `primary_user_id` of their company | _(internal)_ |
| `orasesHasOwnPaymentMethod` | Company has own payment method (not inherited from parent) | `HasOwnPaymentMethod` run_method |

### Other Rule Classes

| Class | File | Purpose |
|-------|------|---------|
| `WATMCustomerRules` | `plugins/UserManagement/src/Auth/Rbac/Rules/WATMCustomerRules.php` | Customer company view: restricts to own company + sub-companies |
| `RmaRules` | `plugins/UserManagement/src/Auth/Rbac/Rules/RmaRules.php` | RMA edit: superusers can only edit RMAs with "RMA In Progress" status |
| `OrasesOnlyRules` | `plugins/UserManagement/src/Auth/Rbac/Rules/OrasesOnlyRules.php` | Restricts to Orases staff (email contains "orases") |

---

## 8. Superuser Bypass & the Global Super Admin vs. Global Admin Distinction

The `is_superuser` flag on `o_users` is what creates the distinction between the **Global Super Admin** (Role #1) and the **Global Admin** (Role #2). This flag is checked first in `WatmRules.orasesDefaultAllow()` - if set, the entire permission matrix is skipped.

### How the Superuser Flag Works

```
WatmRules::orasesDefaultAllow()
  1. if ($user->is_superuser === true) --> return true   // BYPASS EVERYTHING
  2. if ($user->role !== 'admin' && $user->role !== 'customer') --> return false
  3. Check DB permission exists for user's assigned role
  4. If permission has run_method, execute additional business rule check
```

### Current State

- `AdminsSeed.php` creates initial admin accounts **with** `is_superuser=1` (Global Super Admin)
- The Admin panel allows creating admin users where the superuser flag **can be toggled**
- If an admin user is created **without** `is_superuser=1`, they become a Global Admin (Role #2) and are subject to the incomplete permission seed (see Section 5)

### Why This Matters for the Permissions Review

The client's request to "add restrictions to the Admin role" fundamentally depends on how the superuser flag is handled going forward:

| Approach | Description | Effort |
|----------|-------------|--------|
| **A. Restrict within superuser** | Keep all admins as superusers but add UI-level restrictions or feature flags | Low effort, but doesn't enforce at the authorization layer |
| **B. Remove superuser from some admins** | Create non-superuser admin accounts and rely on the permission matrix | Requires fixing the ~30 missing permissions first, then selectively granting/revoking |
| **C. Create new admin-level roles** | Add new role records (e.g., "Admin Viewer", "Billing Admin") under `role_type=admin` | Most flexible, requires new seed data and potentially new WatmRules methods |
| **D. Hybrid approach** | Keep Global Super Admin as superuser for Orases staff, create restricted Global Admin roles for client staff | Recommended - preserves Orases full access while enabling client-specific restrictions |

---

## 9. Observations & Gaps

### Key Findings

#### 1. The Superuser Gap is the Biggest Risk

The Global Admin role (Role #2) has **~30 missing permissions** in the seed data. This was never a problem because all admin accounts have historically been created as superusers (Role #1). However, this means:

- There is **no tested, working non-superuser admin experience** today
- Before any admin role restrictions can be implemented, the Global Admin permission baseline must be established and verified
- Any work to restrict admin access must start with fixing the seed data

#### 2. Global Admin Has Fewer Seeded Permissions Than Company Super Admin

| Role | Seeded Count |
|------|:---:|
| Global Admin (Role #2) | 55 |
| Company Super Admin (Role #3) | 63 |

This inversion exists because the Global Admin was never meant to operate without the superuser flag. The Company Super Admin, by contrast, was designed to work strictly from its seeded permissions.

#### 3. Permissions Not Assigned to ANY Role

These 30 permissions exist in the database but are **only accessible via the superuser bypass**. They are not seeded for any of the 3 database roles:

| Category | Unassigned Permissions |
|----------|----------------------|
| Billing | `billing_cycles_browse`, `billing_cycle_close`, `invoices_generate`, `invoices_regenerate`, `invoices_send_email`, `invoices_edit_exclusions` |
| Admin Users | `admin_edit`, `admin_delete` |
| Devices | `devices_import`, `devices_deactivate_payment_method` |
| RMAs | `rma_edit`, `rma_export`, `rma_delete` |
| Service Plans | `service_plan_edit`, `service_plan_delete`, `service_plan_approve` |
| Configs | `config_group_edit`, `config_group_delete`, `config_edit`, `config_delete`, `config_approve` |
| Notifications | `notification_edit`, `notification_delete`, `notification_approve` |
| FAQ | `faq_edit`, `faq_delete` |
| Company | `company_edit_protected_fields`, `company_payment_methods_credit`, `company_credits_edit`, `company_credits_delete` |

#### 4. Seed File Anomaly

`company_power_management_browse` is assigned to Customer Admin (`$rolesDictionaryCustomer['Admin']`) at line 112-114 of the seed file, but it's positioned within the WATM Admin section of the code. This appears intentional (Customer Admins should see power schedules) but the placement is confusing.

#### 5. WATM-1464: Removed Customer Permissions

Two permissions were intentionally removed from Company Super Admin (commented out in seed):
- `invoices_regenerate`
- `invoices_send_email`

These remain as superuser-only operations.

#### 6. No Read-Only Customer Role

The Company Admin (Role #4) is the most restricted customer role but still has export capabilities and power management edit access. There is no true "viewer" role for customer portal users.

#### 7. No Granular Admin Roles

All WATM admin staff share one role record. The only differentiation is the superuser flag (all-or-nothing). There is no middle ground for admin users who should have some but not all administrative capabilities.

### Recommended Next Steps

#### Step 1: Establish Global Admin Baseline (Prerequisites)

Before adding restrictions or new roles, decide:
- Which of the 30 "missing" permissions should the Global Admin role actually have?
- Are some intentionally excluded (e.g., customer-specific features like power management)?
- Create a migration to seed the correct permissions for the Global Admin role

#### Step 2: Decide on Role Architecture

| Approach | New Admin Roles | New Customer Roles | Effort |
|----------|---|---|---|
| **Minimal** | Keep 1 admin role, fix seed, use superuser flag for Orases-only access | No changes | Low |
| **Moderate** | Add "Admin Viewer" (read-only) | Add "Customer Viewer" (read-only) | Medium |
| **Comprehensive** | Add "Admin Viewer", "Billing Admin", "Config Admin" | Add "Customer Viewer", "Customer Technician" | High |

#### Step 3: Potential New Roles to Consider

| Potential Role | Type | Scope | Use Case |
|---|---|---|---|
| **Admin Viewer** | admin | Read-only across all sections | Client stakeholders who need visibility without edit access |
| **Billing Admin** | admin | Billing cycles, invoices, payment methods, reports | Separation of billing duties from device management |
| **Config Admin** | admin | Device configs, config groups, service plans | Separation of device config management |
| **Customer Viewer** | customer | Read-only company portal (devices, invoices, notifications) | Employee who needs to monitor device status |
| **Customer Technician** | customer | Devices (view, ping, edit, export) only | Field technician managing devices without billing/company access |

#### Step 4: Implementation Requirements

For each new role:
1. Add role record to `o_users_roles` via migration
2. Define permission mappings in `o_users_roles_o_users_permissions` via migration
3. Update seed files for fresh installs
4. Update Admin panel UI to allow assigning the new roles
5. Test all permission boundaries with the new role
6. Update any `WatmRules` methods if new business logic is needed

---

## 10. Source Files Reference

| File | Purpose |
|------|---------|
| `watm/config/permissions.php` | Route-level permission rules |
| `watm/config/Seeds/WATMRolesSeed.php` | Admin role creation |
| `watm/config/Seeds/CompanyUserRolesSeed.php` | Customer role creation |
| `watm/config/Seeds/OUsersPermissionsSeed.php` | All permission definitions |
| `watm/config/Seeds/OUsersRolesOUsersPermissionsSeed.php` | Role-to-permission mappings |
| `watm/src/Auth/Rules/WatmRules.php` | Core authorization logic (638 lines) |
| `watm/plugins/UserManagement/src/Auth/Rbac/Rules/WATMCustomerRules.php` | Customer company view rules |
| `watm/plugins/UserManagement/src/Auth/Rbac/Rules/RmaRules.php` | RMA-specific rules |
| `watm/plugins/UserManagement/src/Auth/Rbac/Rules/OrasesOnlyRules.php` | Orases-staff-only rules |
| `watm/plugins/UserAuthorization/src/Plugin.php` | Authorization middleware registration |
| `watm/config/Migrations/20230327144040_RemoveCallCenterAdminRole.php` | Removed Call Center Admin |
| `watm/config/Migrations/20220902151522_AddTypeToOUsersRoles.php` | Added role_type to roles |
