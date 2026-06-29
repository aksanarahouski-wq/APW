# WATM Portal — Roles & Permissions Overview

**Date:** 2026-04-06
**Purpose:** Summary of current roles, what each role can do, and where there may be gaps to discuss.

---

## Current Roles

The WATM portal has **4 user roles**:

| Role | Who Uses It | Description |
|------|------------|-------------|
| **WATM Super Admin** | Orases dev team and APW admin staff | Full, unrestricted access to every feature in the portal. This is the primary admin role used today. |
| **WATM Admin** | WATM admins (APW staff) | Admin-level access but subject to the permission rules below. Currently not actively used — all admin accounts have Super Admin access. |
| **Company Super Admin** | Distributor-level customers | Full customer portal access for their company and sub-companies. Can manage employees, payment methods, notifications, white labeling, and assign devices. |
| **Company Admin** | Standard customer users | Limited customer portal access. Can view most things but cannot edit employees, payment methods, notifications, or assign devices. |

---

## Company Account Types

In addition to user roles, each company has an **account type** that controls what features are available at the company level:

| Account Type | Description | Key Capabilities |
|-------------|-------------|-----------------|
| **Distributor** | Parent company that manages sub-companies | Can assign devices to sub-companies, set pricing upcharges, manage white labeling, view commissions and earnings |
| **Customer** | Standard company | Can assign devices, manage their own settings. No upcharge or white label capabilities. |
| **Subcustomer** | Child company under a Distributor or Customer | Most limited — cannot assign devices or set upcharges. Access is scoped to their own company data only. |

A user's effective access is a combination of their **role** (what actions they can perform) and their **company's account type** (what features are available to that company).

---

## Permission Matrix

### How to Read This Table

- **Yes** = Has access
- **View Only** = Can browse and view but cannot create, edit, or delete
- **No** = No access to this feature
- Company Super Admin and Company Admin permissions are further filtered by their company's account type (see above)

### Companies & Employees

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Browse & View Companies | Yes | Yes | Yes (own + sub-companies) | Yes (own + sub-companies) |
| Edit Company | Yes | Yes | Yes (own + sub-companies) | No |
| Merge Companies | Yes | Yes | No | No |
| Deactivate Company | Yes | No | Yes | No |
| Edit Protected Fields (pricing, carrier settings) | Yes | No | No | No |
| Edit Dual SIM Upcharge | Yes | No | Distributors only | No |
| Allow Self Billing | Yes | No | Distributors only | No |
| Invite New Companies | Yes | No | Distributors/Customers only | No |
| Browse & View Employees | Yes | Yes | Yes (own company) | Yes (own company) |
| Edit / Delete Employees | Yes | No | Yes (own company) | No |

### Devices

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Browse & View Devices | Yes | Yes | Yes (own company) | Yes (own company) |
| Edit Devices | Yes | Yes | Yes (own company) | No |
| Ping Devices | Yes | Yes | Yes (own company) | Yes (own company) |
| Assign Devices | Yes | No | Distributors/Customers only | No |
| Import Devices | Yes | No | No | No |
| Bulk Actions | Yes | Yes | Yes | No |
| Export Devices | Yes | Yes | Yes | Yes |
| Export Device Check-ins | Yes | Yes | Yes (own company) | Yes (own company) |
| Export Device Alerts | Yes | Yes | Yes | Yes |

### Service Plans

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Browse & View Service Plans | Yes | Yes | No | No |
| Edit / Delete / Approve Service Plans | Yes | No | No | No |
| Customize Service Plan (upcharge) | Yes | No | Distributors only | No |
| Edit / Delete Custom Service Plan | Yes | No | Distributors only | No |

### Device Configuration

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Browse & View Config Groups | Yes | Yes | No | No |
| Edit / Delete Config Groups | Yes | No | No | No |
| Browse & View Configs | Yes | Yes | No | No |
| Edit / Delete / Approve Configs | Yes | No | No | No |

### Billing & Invoices

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Manage Billing Cycles | Yes | No | No | No |
| Close Billing Cycle | Yes | No | No | No |
| Browse & View Invoices | Yes | Yes | Yes (own company) | Yes (own company) |
| Preview Invoice PDF | Yes | Yes | Yes (own company) | Yes (own company) |
| Generate / Regenerate Invoices | Yes | No | No | No |
| Send Invoice Emails | Yes | No | No | No |
| Edit Invoice Exclusions | Yes | No | No | No |
| View Pricing Info | Yes | Yes | Yes (Distributors + self-billing Subcustomers) | Yes (Distributors + self-billing Subcustomers) |
| Generate / Download Billing Report | Yes | Yes | No | No |

### Payment Methods

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Browse & View Payment Methods (Global) | Yes | Yes | No | No |
| Payment Methods Report | Yes | Yes | No | No |
| Browse & View Company Payment Methods | Yes | Yes | Yes (own company) | Yes (own company) |
| Edit / Assign / Delete Company Payment Methods | Yes | No | Yes (own company) | No |
| Export Payment Methods | Yes | No | Distributors only | No |
| Credit Payment Method | Yes | No | No | No |
| Deactivate Devices by Payment Method | Yes | No | No | No |

### Commissions & Earnings

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Browse Commissions (admin view) | Yes | Yes | No | No |
| Commission Preferences | Yes | No | Distributors only | No |
| Company Payouts | Yes | No | Distributors only | No |
| Browse Earnings Reports | Yes | No | Yes | Yes |

### Company Notifications

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Browse & View Notifications | Yes | Yes | Yes | Yes |
| Edit / Delete Notifications | Yes | No | Yes | No |
| View Notification Logs | Yes | Yes | Yes | Yes |
| Browse & View Sub-company Global Notifications | Yes | No | Yes | Yes |
| Edit Sub-company Global Notifications | Yes | No | Yes | Yes |

### System Notifications (Global)

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Browse & View System Notifications | Yes | Yes | No | No |
| Edit / Delete / Approve System Notifications | Yes | No | No | No |
| View Notification Logs | Yes | Yes | Yes | Yes |
| Export Notifications | Yes | Yes | Yes | Yes |

### RMAs

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Browse & View RMAs | Yes | Yes | No | No |
| Edit / Export / Delete RMAs | Yes | No | No | No |

### API Keys

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Browse & View API Keys | Yes | Yes | Yes | Yes |
| Add / Edit / Delete API Keys | Yes | Yes | Yes | No |
| Revoke / Activate API Keys | Yes | Yes | Yes | No |
| View API Usage | Yes | Yes | Yes | Yes |

### White Labeling

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| View White Label Settings | Yes | Yes | Distributors only | No |
| Edit White Label Settings | Yes | No | Distributors only | No |
| Manage White Label Manufacturers & Models | Yes | No | Distributors only | No |
| Invoice Preview (White Label) | Yes | No | Distributors only | No |

### Power Management

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Browse Power Schedules | Yes | No | Yes (own company) | Yes (own company) |
| Edit / Delete Power Schedules | Yes | No | Yes (own company) | Yes (own company) |

### Other

| Capability | WATM Super Admin | WATM Admin | Company Super Admin | Company Admin |
|-----------|:---:|:---:|:---:|:---:|
| Admin User Management (Browse & View) | Yes | Yes | No | No |
| Admin User Management (Edit / Delete) | Yes | No | No | No |
| Browse & View Logs | Yes | Yes | Yes | No |
| Tax Documents | Yes | No | Distributors only (primary user) | No |
| Company Credits (Browse) | Yes | Yes | Yes | Yes |
| Company Credits (Edit / Delete) | Yes | No | No | No |
| FAQs (Browse & View) | Yes | Yes | Yes | Yes |
| FAQs (Edit / Delete) | Yes | No | No | No |
| Check-in Failures (Browse & View) | Yes | Yes | No | No |
| Support Tickets | Yes | Yes | Yes | Yes |
| Edit Profile | Yes | Yes | Yes | Yes |

---

## Gaps & Discussion Points

### 1. No Read-Only Customer Role

The most restricted customer role (Company Admin) still has some edit capabilities (power schedules, sub-company notifications) and export access. There is no true "viewer" role for customer portal users who just need to monitor device status, view invoices, or check alerts without being able to change anything.

**Question:** Is there a need for a read-only customer role? For example, a field technician who needs to see device status and signal strength but shouldn't be able to edit devices or manage billing.

### 2. No Granular Admin Roles

All WATM admin staff share the same role. There is no way to give an admin user access to billing features without also giving them access to device configuration, RMAs, and everything else. It's all-or-nothing.

**Question:** Would it be useful to have specialized admin roles? For example:
- **Admin Viewer** — Read-only access across all sections. For stakeholders who need visibility without edit access.
- **Billing Admin** — Can manage billing cycles, invoices, and payment methods, but not device configurations or service plans.
- **Config Admin** — Can manage device configurations and service plans, but not billing or company management.

### 3. WATM Admin Role Not Actively Used

The WATM Admin role (non-super-admin) exists but hasn't been actively used. All admin accounts currently have full Super Admin access. If we want to start restricting admin access, this role needs to be reviewed and updated first.

### 4. Company Admin Has Limited Write Access

The Company Admin role can view most things but cannot:
- Edit devices, employees, or payment methods
- Manage notifications
- Assign devices
- Perform bulk actions

**Question:** Is this the right level of restriction, or are there capabilities Company Admins should have that they currently don't?

### 5. Account Type Controls Are Company-Wide

Features like device assignment, upcharging, and white labeling are controlled at the company level (Distributor vs. Customer vs. Subcustomer), not per-user. This means all users within a Distributor company have access to distributor-level features (based on their role), with no way to restrict individual users from specific distributor capabilities.

**Question:** Is per-user control within a company account type needed, or is the current company-wide approach sufficient?

---

## Potential New Roles to Consider

If new roles are needed, here are options to discuss:

| Potential Role | Type | Who Would Use It | What They Could Do |
|---------------|------|-----------------|-------------------|
| **Admin Viewer** | Admin | Client stakeholders, auditors | View everything across all sections. Cannot edit, delete, or create anything. |
| **Billing Admin** | Admin | Finance/billing staff | Full access to billing cycles, invoices, payment methods, and reports. View-only for devices and companies. |
| **Config Admin** | Admin | Technical/device management staff | Full access to device configurations, config groups, and service plans. View-only for billing and companies. |
| **Customer Viewer** | Customer | Field technicians, monitoring staff | View devices, invoices, alerts, and company info. Cannot edit anything. |
| **Customer Technician** | Customer | Field technicians who manage devices | Can view and edit devices, ping, export check-ins. No access to billing, employees, or company settings. |

These are starting points for discussion — the exact capabilities of each role would be defined based on your operational needs.
