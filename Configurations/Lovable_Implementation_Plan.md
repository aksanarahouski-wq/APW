# WATM Configuration Management System - Lovable Implementation Plan

**Date:** January 12, 2026
**Target Platform:** Lovable.dev (AI-powered web app builder)
**PRD Reference:** Config_Management_System_PRD.md v1.5
**Technology Stack:** React, TypeScript, Tailwind CSS, Supabase (via Lovable)

---

## 📋 Table of Contents

1. [Overview & Architecture](#overview--architecture)
2. [Data Model Design](#data-model-design)
3. [Implementation Phases](#implementation-phases)
4. [Phase 1: Foundation & Schema Management](#phase-1-foundation--schema-management)
5. [Phase 2: Layer Configuration System](#phase-2-layer-configuration-system)
6. [Phase 3: Conditional Rules Management](#phase-3-conditional-rules-management)
7. [Phase 4: Device Effective Config View](#phase-4-device-effective-config-view)
8. [Phase 5: Validation & Safety Systems](#phase-5-validation--safety-systems)
9. [Phase 6: Change Detection & Versioning](#phase-6-change-detection--versioning)
10. [Design System Guidelines](#design-system-guidelines)
11. [Testing Strategy](#testing-strategy)

---

## Overview & Architecture

### System Goals
Build a hierarchical configuration management system for IoT devices with:
- 600+ configuration parameters managed via schema
- 6-layer inheritance hierarchy (Global → Model → Carrier → Service Plan → Company → Device)
- Multi-factor conditional rules (2-way, 3-way, 4-way)
- Full audit trail and version control
- Admin and customer user interfaces

### Key Architectural Decisions

**Frontend:** React + TypeScript + Tailwind CSS (Lovable default stack)
**Backend:** Supabase (PostgreSQL + Row Level Security)
**State Management:** React Query (for server state) + Zustand (for UI state)
**Routing:** React Router
**Forms:** React Hook Form + Zod validation
**UI Components:** shadcn/ui (Lovable-compatible)
**Data Visualization:** Recharts (for coverage matrices, stats)

### Out of Scope for Initial Build
- FR-3: Config Resolution Engine (design considered, implementation deferred)
- Integration with actual IoT device UDP server
- NACHA/ACH billing integration
- Production deployment infrastructure

---

## Data Model Design

### Core Tables (Supabase Schema)

#### 1. `config_keys` (Config Key Schema)
```sql
CREATE TABLE config_keys (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  key_name TEXT UNIQUE NOT NULL,
  display_name TEXT NOT NULL,
  description TEXT,
  data_type TEXT NOT NULL, -- 'string', 'integer', 'boolean', 'ip_address', 'time', 'enum'
  category TEXT NOT NULL,
  subcategory TEXT,
  default_value TEXT,
  example_value TEXT,

  -- Layer availability
  available_at_global BOOLEAN DEFAULT true,
  available_at_model BOOLEAN DEFAULT false,
  available_at_carrier BOOLEAN DEFAULT false,
  available_at_service_plan BOOLEAN DEFAULT false,
  available_at_company BOOLEAN DEFAULT false,
  available_at_device BOOLEAN DEFAULT false,

  -- Flags
  is_required BOOLEAN DEFAULT false,
  is_customer_configurable BOOLEAN DEFAULT false,
  has_two_way_rules BOOLEAN DEFAULT false,
  has_three_way_rules BOOLEAN DEFAULT false,
  has_four_way_rules BOOLEAN DEFAULT false,

  -- Validation rules (JSON)
  validation_rules JSONB DEFAULT '{}',

  -- Metadata
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  created_by UUID REFERENCES auth.users(id),

  -- Indexes
  CONSTRAINT valid_data_type CHECK (data_type IN ('string', 'integer', 'boolean', 'ip_address', 'time', 'enum'))
);

CREATE INDEX idx_config_keys_category ON config_keys(category);
CREATE INDEX idx_config_keys_flags ON config_keys(is_required, is_customer_configurable);
CREATE INDEX idx_config_keys_conditional ON config_keys(has_two_way_rules, has_three_way_rules, has_four_way_rules);
```

#### 2. `models` (Device Models)
```sql
CREATE TABLE models (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  model_code TEXT UNIQUE NOT NULL, -- 'i-22', 'i-52', '4100', '4500', etc.
  display_name TEXT NOT NULL,
  manufacturer TEXT,
  description TEXT,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

#### 3. `carriers` (Cellular Carriers)
```sql
CREATE TABLE carriers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  carrier_code TEXT UNIQUE NOT NULL, -- 'VZW', 'ATT', 'TMO', etc.
  display_name TEXT NOT NULL,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

#### 4. `service_plans` (Service Plan Tiers)
```sql
CREATE TABLE service_plans (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  plan_code TEXT UNIQUE NOT NULL, -- 'ATM', 'TIER1', 'TIER2', etc.
  display_name TEXT NOT NULL,
  description TEXT,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

#### 5. `companies` (Customers)
```sql
CREATE TABLE companies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_name TEXT NOT NULL,
  company_code TEXT UNIQUE NOT NULL,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

#### 6. `devices` (IoT Devices)
```sql
CREATE TABLE devices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  device_code TEXT UNIQUE NOT NULL,
  device_name TEXT NOT NULL,
  model_id UUID REFERENCES models(id) NOT NULL,
  carrier_id UUID REFERENCES carriers(id) NOT NULL,
  service_plan_id UUID REFERENCES service_plans(id) NOT NULL,
  company_id UUID REFERENCES companies(id) NOT NULL,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_devices_company ON devices(company_id);
CREATE INDEX idx_devices_model ON devices(model_id);
CREATE INDEX idx_devices_carrier ON devices(carrier_id);
CREATE INDEX idx_devices_plan ON devices(service_plan_id);
CREATE INDEX idx_devices_lookup ON devices(model_id, carrier_id, service_plan_id, company_id);
```

#### 7. `config_values` (Multi-Layer Configuration Values)
```sql
CREATE TABLE config_values (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  config_key_id UUID REFERENCES config_keys(id) NOT NULL,

  -- Layer type
  layer_type TEXT NOT NULL, -- 'global', 'model', 'carrier', 'service_plan', 'company', 'device'

  -- Layer references (nullable - only one should be set based on layer_type)
  model_id UUID REFERENCES models(id),
  carrier_id UUID REFERENCES carriers(id),
  service_plan_id UUID REFERENCES service_plans(id),
  company_id UUID REFERENCES companies(id),
  device_id UUID REFERENCES devices(id),

  -- Value
  value TEXT, -- NULL means explicitly cleared
  is_null BOOLEAN DEFAULT false, -- Explicit NULL (cleared) vs not set

  -- Metadata
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  created_by UUID REFERENCES auth.users(id),
  updated_by UUID REFERENCES auth.users(id),

  -- Constraints
  CONSTRAINT valid_layer_type CHECK (layer_type IN ('global', 'model', 'carrier', 'service_plan', 'company', 'device')),
  CONSTRAINT unique_config_layer UNIQUE (config_key_id, layer_type, model_id, carrier_id, service_plan_id, company_id, device_id)
);

-- Indexes for layer queries
CREATE INDEX idx_config_values_global ON config_values(config_key_id) WHERE layer_type = 'global';
CREATE INDEX idx_config_values_model ON config_values(config_key_id, model_id) WHERE layer_type = 'model';
CREATE INDEX idx_config_values_carrier ON config_values(config_key_id, carrier_id) WHERE layer_type = 'carrier';
CREATE INDEX idx_config_values_plan ON config_values(config_key_id, service_plan_id) WHERE layer_type = 'service_plan';
CREATE INDEX idx_config_values_company ON config_values(config_key_id, company_id) WHERE layer_type = 'company';
CREATE INDEX idx_config_values_device ON config_values(config_key_id, device_id) WHERE layer_type = 'device';
```

#### 8. `conditional_rules` (Multi-Factor Rules)
```sql
CREATE TABLE conditional_rules (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  config_key_id UUID REFERENCES config_keys(id) NOT NULL,

  -- Rule type
  rule_type TEXT NOT NULL, -- 'two_way', 'three_way', 'four_way'

  -- Two-way rule combination type (only for two_way rules)
  two_way_combination TEXT, -- 'model_carrier', 'carrier_plan', 'model_plan', 'carrier_customer', 'model_customer', 'plan_customer'

  -- Factor references (nullable based on rule type)
  model_id UUID REFERENCES models(id),
  carrier_id UUID REFERENCES carriers(id),
  service_plan_id UUID REFERENCES service_plans(id),
  company_id UUID REFERENCES companies(id),

  -- Value
  value TEXT,
  is_null BOOLEAN DEFAULT false,

  -- Priority (auto-assigned based on rule type and combination)
  priority_level INTEGER NOT NULL, -- 2 (four-way), 4 (three-way), 6 (two-way)
  sub_priority INTEGER, -- 1-6 for two-way rules

  -- Metadata
  description TEXT,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  created_by UUID REFERENCES auth.users(id),

  -- Constraints
  CONSTRAINT valid_rule_type CHECK (rule_type IN ('two_way', 'three_way', 'four_way')),
  CONSTRAINT valid_two_way_combo CHECK (
    (rule_type != 'two_way') OR
    (two_way_combination IN ('model_carrier', 'carrier_plan', 'model_plan', 'carrier_customer', 'model_customer', 'plan_customer'))
  ),
  CONSTRAINT unique_rule UNIQUE (config_key_id, model_id, carrier_id, service_plan_id, company_id)
);

-- Indexes for rule queries (6 two-way combinations)
CREATE INDEX idx_rules_two_way_mc ON conditional_rules(config_key_id, model_id, carrier_id)
  WHERE rule_type = 'two_way' AND carrier_id IS NOT NULL AND model_id IS NOT NULL AND service_plan_id IS NULL AND company_id IS NULL;
CREATE INDEX idx_rules_two_way_cp ON conditional_rules(config_key_id, carrier_id, service_plan_id)
  WHERE rule_type = 'two_way' AND carrier_id IS NOT NULL AND service_plan_id IS NOT NULL AND model_id IS NULL AND company_id IS NULL;
CREATE INDEX idx_rules_two_way_mp ON conditional_rules(config_key_id, model_id, service_plan_id)
  WHERE rule_type = 'two_way' AND model_id IS NOT NULL AND service_plan_id IS NOT NULL AND carrier_id IS NULL AND company_id IS NULL;
CREATE INDEX idx_rules_two_way_cc ON conditional_rules(config_key_id, carrier_id, company_id)
  WHERE rule_type = 'two_way' AND carrier_id IS NOT NULL AND company_id IS NOT NULL AND model_id IS NULL AND service_plan_id IS NULL;
CREATE INDEX idx_rules_two_way_mc_cust ON conditional_rules(config_key_id, model_id, company_id)
  WHERE rule_type = 'two_way' AND model_id IS NOT NULL AND company_id IS NOT NULL AND carrier_id IS NULL AND service_plan_id IS NULL;
CREATE INDEX idx_rules_two_way_pc ON conditional_rules(config_key_id, service_plan_id, company_id)
  WHERE rule_type = 'two_way' AND service_plan_id IS NOT NULL AND company_id IS NOT NULL AND model_id IS NULL AND carrier_id IS NULL;

-- Indexes for three-way and four-way rules
CREATE INDEX idx_rules_three_way ON conditional_rules(config_key_id, model_id, carrier_id, service_plan_id)
  WHERE rule_type = 'three_way';
CREATE INDEX idx_rules_four_way ON conditional_rules(config_key_id, model_id, carrier_id, service_plan_id, company_id)
  WHERE rule_type = 'four_way';
```

#### 9. `config_history` (Change Tracking & Versioning)
```sql
CREATE TABLE config_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- What changed
  entity_type TEXT NOT NULL, -- 'config_value', 'conditional_rule'
  entity_id UUID NOT NULL,
  config_key_id UUID REFERENCES config_keys(id) NOT NULL,

  -- Change details
  change_type TEXT NOT NULL, -- 'created', 'updated', 'deleted'
  old_value TEXT,
  new_value TEXT,

  -- Context
  layer_type TEXT,
  rule_type TEXT,
  affected_device_count INTEGER,

  -- Metadata
  changed_at TIMESTAMPTZ DEFAULT now(),
  changed_by UUID REFERENCES auth.users(id),
  change_reason TEXT,

  CONSTRAINT valid_entity_type CHECK (entity_type IN ('config_value', 'conditional_rule')),
  CONSTRAINT valid_change_type CHECK (change_type IN ('created', 'updated', 'deleted'))
);

CREATE INDEX idx_history_key ON config_history(config_key_id);
CREATE INDEX idx_history_entity ON config_history(entity_type, entity_id);
CREATE INDEX idx_history_date ON config_history(changed_at DESC);
CREATE INDEX idx_history_user ON config_history(changed_by);
```

#### 10. `users` (Admin/Customer Users)
```sql
-- Extends Supabase auth.users with roles
CREATE TABLE user_profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  role TEXT NOT NULL, -- 'admin', 'customer'
  company_id UUID REFERENCES companies(id), -- NULL for admin, required for customer
  full_name TEXT,
  created_at TIMESTAMPTZ DEFAULT now(),

  CONSTRAINT valid_role CHECK (role IN ('admin', 'customer'))
);

CREATE INDEX idx_user_profiles_company ON user_profiles(company_id) WHERE role = 'customer';
```

### Row Level Security (RLS) Policies

```sql
-- Enable RLS on all tables
ALTER TABLE config_keys ENABLE ROW LEVEL SECURITY;
ALTER TABLE config_values ENABLE ROW LEVEL SECURITY;
ALTER TABLE conditional_rules ENABLE ROW LEVEL SECURITY;
ALTER TABLE devices ENABLE ROW LEVEL SECURITY;
ALTER TABLE companies ENABLE ROW LEVEL SECURITY;

-- Admin: Full access
CREATE POLICY "Admins have full access" ON config_keys FOR ALL
  USING (EXISTS (SELECT 1 FROM user_profiles WHERE id = auth.uid() AND role = 'admin'));

CREATE POLICY "Admins have full access" ON config_values FOR ALL
  USING (EXISTS (SELECT 1 FROM user_profiles WHERE id = auth.uid() AND role = 'admin'));

-- Customer: Read-only access to own company data
CREATE POLICY "Customers see own company devices" ON devices FOR SELECT
  USING (company_id IN (SELECT company_id FROM user_profiles WHERE id = auth.uid() AND role = 'customer'));

CREATE POLICY "Customers see customer-configurable keys" ON config_keys FOR SELECT
  USING (is_customer_configurable = true);

-- Customer: Can update own company and device configs (customer-configurable keys only)
CREATE POLICY "Customers can update own company configs" ON config_values FOR ALL
  USING (
    layer_type = 'company' AND
    company_id IN (SELECT company_id FROM user_profiles WHERE id = auth.uid() AND role = 'customer') AND
    config_key_id IN (SELECT id FROM config_keys WHERE is_customer_configurable = true)
  );

CREATE POLICY "Customers can update own device configs" ON config_values FOR ALL
  USING (
    layer_type = 'device' AND
    device_id IN (SELECT id FROM devices WHERE company_id IN (SELECT company_id FROM user_profiles WHERE id = auth.uid() AND role = 'customer')) AND
    config_key_id IN (SELECT id FROM config_keys WHERE is_customer_configurable = true)
  );
```

---

## Implementation Phases

### Phase 1: Foundation & Schema Management (FR-1)
**Duration:** 1-2 weeks
**Focus:** Config key schema CRUD, data model setup, basic UI framework

### Phase 2: Layer Configuration System (FR-2)
**Duration:** 2-3 weeks
**Focus:** 6-layer configuration interfaces, inheritance display, contextual rules access

### Phase 3: Conditional Rules Management (FR-3a)
**Duration:** 2-3 weeks
**Focus:** Two-way, three-way, four-way rule creation, coverage matrices, rule validation

### Phase 4: Device Effective Config View (FR-4)
**Duration:** 1 week
**Focus:** Config resolution preview, source attribution display

### Phase 5: Validation & Safety Systems (FR-5)
**Duration:** 1 week
**Focus:** Input validation, preview before save, confirmation dialogs

### Phase 6: Change Detection & Versioning (FR-6)
**Duration:** 1 week
**Focus:** Audit logs, change history, rollback capability

**Total Estimated Duration:** 8-10 weeks

---

## Phase 1: Foundation & Schema Management

### Goals
- Set up data model in Supabase
- Build config key schema CRUD interface
- Create authentication system (admin/customer roles)
- Establish design system and reusable components

### Lovable Prompts

#### Prompt 1.1: Project Setup & Data Model
```
Create a new React TypeScript project with the following setup:

PROJECT NAME: WATM Config Management System

TECHNOLOGY STACK:
- Frontend: React 18 + TypeScript + Vite
- UI: Tailwind CSS + shadcn/ui components
- Backend: Supabase (PostgreSQL)
- State: React Query + Zustand
- Forms: React Hook Form + Zod validation
- Routing: React Router v6

DATABASE SCHEMA (Supabase):
Create the following tables with PostgreSQL:

1. config_keys table:
   - id: UUID primary key
   - key_name: TEXT unique, required
   - display_name: TEXT required
   - description: TEXT
   - data_type: ENUM ('string', 'integer', 'boolean', 'ip_address', 'time', 'enum')
   - category: TEXT required
   - subcategory: TEXT
   - default_value: TEXT
   - example_value: TEXT
   - available_at_global: BOOLEAN default true
   - available_at_model: BOOLEAN default false
   - available_at_carrier: BOOLEAN default false
   - available_at_service_plan: BOOLEAN default false
   - available_at_company: BOOLEAN default false
   - available_at_device: BOOLEAN default false
   - is_required: BOOLEAN default false
   - is_customer_configurable: BOOLEAN default false
   - has_two_way_rules: BOOLEAN default false
   - has_three_way_rules: BOOLEAN default false
   - has_four_way_rules: BOOLEAN default false
   - validation_rules: JSONB default '{}'
   - created_at: TIMESTAMPTZ
   - updated_at: TIMESTAMPTZ
   - created_by: UUID references auth.users

2. models table (device models):
   - id: UUID primary key
   - model_code: TEXT unique (e.g., 'i-22', '4100', '4500')
   - display_name: TEXT
   - manufacturer: TEXT
   - is_active: BOOLEAN
   - created_at: TIMESTAMPTZ

3. carriers table:
   - id: UUID primary key
   - carrier_code: TEXT unique (e.g., 'VZW', 'ATT', 'TMO')
   - display_name: TEXT
   - is_active: BOOLEAN
   - created_at: TIMESTAMPTZ

4. service_plans table:
   - id: UUID primary key
   - plan_code: TEXT unique (e.g., 'ATM', 'TIER1')
   - display_name: TEXT
   - description: TEXT
   - is_active: BOOLEAN
   - created_at: TIMESTAMPTZ

5. companies table:
   - id: UUID primary key
   - company_name: TEXT
   - company_code: TEXT unique
   - is_active: BOOLEAN
   - created_at: TIMESTAMPTZ

DESIGN SYSTEM:
- Color scheme: Primary purple (#667eea), secondary indigo (#764ba2)
- Use gradient headers: linear-gradient(135deg, #667eea 0%, #764ba2 100%)
- Card-based layouts with subtle shadows
- Clean, modern aesthetic similar to Tailwind UI

Set up Supabase client configuration and create type definitions for all database tables.
```

#### Prompt 1.2: Authentication & User Roles
```
Implement authentication system using Supabase Auth:

USER ROLES:
1. Admin: Full access to all features
2. Customer: Limited access to own company data only

CREATE:
1. Login page with email/password authentication
2. user_profiles table extending auth.users:
   - id: UUID (references auth.users)
   - role: ENUM ('admin', 'customer')
   - company_id: UUID (references companies, NULL for admin)
   - full_name: TEXT

3. Row Level Security (RLS) policies:
   - Admins: Full access to all tables
   - Customers: Read-only access to config_keys where is_customer_configurable = true
   - Customers: Can only see devices from their company

4. Auth context provider with:
   - Current user data
   - User role
   - Company ID (for customers)
   - isAdmin helper function
   - isCustomer helper function

5. Protected route wrapper component that checks authentication

DESIGN:
- Modern login page with gradient background
- Clean form with email and password inputs
- "Remember me" checkbox
- Loading states during authentication
- Error message display for failed login
```

#### Prompt 1.3: Main Layout & Navigation
```
Create the main application layout with navigation:

LAYOUT STRUCTURE:
1. Top Navigation Bar:
   - Left: Logo "WATM Config Management"
   - Center: Main navigation items (different for admin vs customer)
   - Right: User menu dropdown (user name, role badge, logout)

2. Admin Navigation Items:
   - Schema Management (icon: database)
   - Layer Configurations (icon: layers)
   - Conditional Rules (icon: git-branch)
   - Devices (icon: cpu)
   - Audit Log (icon: history)

3. Customer Navigation Items:
   - My Company Config (icon: building)
   - My Devices (icon: cpu)
   - Effective Configs (icon: eye)

4. Sidebar (optional, collapsible):
   - Quick stats dashboard
   - Recent changes
   - System alerts

DESIGN:
- Sticky top navigation with shadow on scroll
- Active navigation item highlighted with primary color
- Role badge next to user name (Admin in purple, Customer in blue)
- Smooth transitions on navigation changes
- Responsive mobile menu (hamburger on small screens)

ROUTING:
Set up React Router with routes for:
- /login
- / or /dashboard (home/dashboard)
- /schema (admin only)
- /layers/global (admin only)
- /layers/model/:id (admin only)
- /layers/carrier/:id (admin only)
- /layers/service-plan/:id (admin only)
- /layers/company/:id (admin/customer)
- /layers/device/:id (admin/customer)
- /rules (admin only)
- /devices (admin/customer)
- /audit-log (admin only)
```

#### Prompt 1.4: Dashboard & 11-Level Priority Overview
```
Create the Dashboard home page with 11-level priority hierarchy explanation:

PAGE: / or /dashboard

LAYOUT:

1. Welcome Banner:
   - "Welcome to WATM Configuration Management"
   - User name and role
   - Quick stats: "Managing 600+ parameters across 10,000 devices"

2. 11-Level Priority Hierarchy Panel (expandable):

   Title: "📊 Configuration Resolution: 11-Level Priority Hierarchy"

   Info icon with tooltip: "Understanding how device configurations are determined"

   Expandable panel content:

   ```
   How Configuration Values are Resolved:

   When a device needs a configuration value, the system checks in this order (first match wins):

   LEVEL 1: Device Override 🟢
   ├─ Admin-set value for this specific device
   ├─ Priority: HIGHEST
   └─ Use case: Emergency fixes, device-specific troubleshooting

   LEVEL 2: Four-Way Rule (Customer Exception) 🔴
   ├─ Model + Carrier + Service Plan + Customer
   ├─ Example: i-22 + VZW + ATM + CORD → custom firewall rules
   └─ Use case: Customer-specific requirements that override plan baselines

   LEVEL 3: Company Override 🟣
   ├─ Customer's portfolio-wide setting
   ├─ Applies to all devices in company (regardless of model/carrier/plan)
   └─ Use case: Company-wide DNS servers, NTP servers, reboot schedules

   LEVEL 4: Three-Way Rule (Service Plan Baseline) 🟣
   ├─ Model + Carrier + Service Plan
   ├─ Example: i-22 + ATT + ATM → 350MB daily threshold
   └─ Use case: Service plan baselines that apply to ALL customers on that plan

   LEVEL 5: Service Plan Layer 🔵
   ├─ Service plan tier settings (ATM, TIER1, TIER2)
   ├─ Applies to all devices with this plan
   └─ Use case: Plan-specific thresholds, features

   LEVEL 6: Two-Way Rules (6 Sub-Priorities) 🔵
   ├─ ANY TWO of four factors (Model, Carrier, Service Plan, Customer)
   ├─ Sub-priority order (first match wins):
   │   6.1: Carrier + Service Plan (carrier data policies by plan tier)
   │   6.2: Model + Service Plan (model pricing tier by plan)
   │   6.3: Model + Carrier (carrier-specific features per model)
   │   6.4: Carrier + Customer (customer carrier agreements)
   │   6.5: Model + Customer (customer model configurations)
   │   6.6: Service Plan + Customer (customer plan exceptions)
   └─ Use case: Multi-factor configurations that don't require all 3 or 4 factors

   LEVEL 7: Carrier Layer 🔵
   ├─ Carrier-specific settings (Verizon, AT&T, T-Mobile)
   ├─ Example: VZW → gateway address
   └─ Use case: Carrier-specific network settings

   LEVEL 8: Model Layer 🔵
   ├─ Device model defaults (i-22, 4100, 4500)
   ├─ Example: i-22 → has 8 I/O pins
   └─ Use case: Model-specific hardware capabilities

   LEVEL 9: Global Layer ⚪
   ├─ System-wide defaults for all devices
   ├─ Example: Default DNS 8.8.8.8
   └─ Use case: Organization-wide standards

   LEVEL 10: Schema Default ⚪
   ├─ Parameter's default value from schema definition
   └─ Use case: Fallback when nothing else is set

   LEVEL 11: Required Validation ❌
   ├─ If required parameter is still not set: ERROR
   └─ Prevents device configuration without required values
   ```

   Visual diagram:
   ```
   [Device Override] ◄─── HIGHEST PRIORITY
         ↓
   [Four-Way Rule]
         ↓
   [Company Override]
         ↓
   [Three-Way Rule]
         ↓
   [Service Plan Layer]
         ↓
   [Two-Way Rules: 6.1 → 6.2 → 6.3 → 6.4 → 6.5 → 6.6]
         ↓
   [Carrier Layer]
         ↓
   [Model Layer]
         ↓
   [Global Layer]
         ↓
   [Schema Default]
         ↓
   [Required Validation] ◄─── LOWEST PRIORITY
   ```

   Color Legend:
   - 🟢 Green: Device-specific overrides
   - 🔴 Red: Four-Way Rules (customer exceptions)
   - 🟣 Purple: Company & Three-Way Rules
   - 🔵 Blue: Two-Way Rules & Single-factor layers
   - ⚪ Gray: Global defaults
   - ❌ Red X: Validation error

3. Quick Actions Cards:
   - Manage Schema (link to /schema)
   - Configure Layers (dropdown to select layer)
   - Manage Rules (link to /rules)
   - View Audit Log (link to /audit-log)

4. Recent Activity Feed:
   - Last 5 configuration changes
   - User, timestamp, action
   - Link to view details

5. System Statistics:
   - Total Parameters: 600+
   - Active Devices: 10,000
   - Companies: 50
   - Total Rules: 72 (25 Two-Way + 35 Three-Way + 12 Four-Way)

INTERACTIVE FEATURES:

- Click any level in hierarchy to see example
- "View Real Example" button → opens device config with full resolution path
- "Why This Order?" tooltip explaining priority rationale
- Toggle between simplified and detailed view

DESIGN:
- Clean, modern dashboard
- Priority hierarchy panel prominent but collapsible
- Visual hierarchy with colors and icons
- Card-based layout
- Responsive grid
```

#### Prompt 1.5: Config Key Schema List View
```
Create the Config Key Schema Management list view (FR-1):

PAGE: /schema

FEATURES:
1. Toolbar with:
   - Search box (filter by key name, description)
   - Category filter dropdown
   - Data type filter dropdown
   - Flags filter (show only Required, Customer Configurable, Has Rules)
   - "New Key" button (primary purple)
   - "Import Keys" button (secondary)
   - "Export Schema" button (secondary)

2. Statistics Cards (top of page):
   - Total Parameters: 600+ (large number)
   - Required Keys: 45 (with red icon)
   - Customer Configurable: 10 (with blue icon)
   - With Conditional Rules: 25 (with purple icon)

3. Key List Table:
   Columns:
   - Key Name (bold, monospace font)
   - Display Name
   - Data Type (badge: blue for string, green for integer, etc.)
   - Category
   - Flags (small badges):
     - Red asterisk if required
     - Blue "C" if customer configurable
     - Purple "2W", "3W", "4W" if has conditional rules
   - Layer Availability (6 checkmarks/X marks for Global/Model/Carrier/Plan/Company/Device)
   - Actions (Edit, Duplicate, Delete icons)

4. Table Features:
   - Sortable columns
   - Pagination (50 per page)
   - Row hover effect
   - Click row to view details

5. Empty State:
   - If no keys: "No configuration keys found. Click 'New Key' to create your first parameter."
   - With illustration

DESIGN:
- Clean table with alternating row colors
- Sticky header when scrolling
- Loading skeleton while fetching data
- Use shadcn/ui Table component
- Responsive: Stack cards on mobile instead of table
```

#### Prompt 1.6: Config Key Detail View
```
Create the Config Key Detail view panel (opens when clicking a row):

COMPONENT: Sliding panel from right side (shadcn/ui Sheet component)

SECTIONS:
1. Header:
   - Display Name (large text)
   - Key Name (monospace, gray, smaller)
   - Close button (top right)

2. Basic Information Section:
   - Data Type: <badge>
   - Category: <text> → Subcategory: <text>
   - Customer Configurable: <Yes/No badge>
   - Required: <Yes/No badge>

3. Description Section:
   - Full description text
   - Default Value: <code>
   - Example Value: <code>

4. Layer Availability Matrix:
   Visual grid showing 6 layers:
   - Global ✓ (available)
   - Model ✗ (not available)
   - Carrier ✓ (available)
   - Service Plan ✓ (available)
   - Company ✓ (available)
   - Device ✓ (available)
   Use green checkmarks and gray X marks

5. Conditional Rules Flags Section:
   If any flags are true, show:
   - ⚠️ This parameter uses Multi-Factor Conditional Rules
   - [If has_two_way_rules] 🔵 Two-Way Rules (Priority Level 6)
     - Explanation: "Value depends on ANY TWO of four factors"
     - List 6 combinations: Model+Carrier, Carrier+Plan, Model+Plan, etc.
   - [If has_three_way_rules] 🟣 Three-Way Rules (Priority Level 4)
     - Explanation: "Service plan baselines for all customers"
   - [If has_four_way_rules] 🔴 Four-Way Rules (Priority Level 2)
     - Explanation: "Customer-specific exceptions"

6. Validation Rules Section:
   Display validation_rules JSON in readable format:
   - For string: regex pattern, min/max length
   - For integer: min/max value
   - For enum: list of allowed values
   - For IP: IPv4 format (auto)

7. Action Buttons (bottom):
   - Edit Key (primary button)
   - Duplicate Key (secondary)
   - Delete Key (danger, with confirmation)

DESIGN:
- Width: 600px panel
- Smooth slide-in animation
- Scrollable content
- Sectioned layout with dividers
- Color-coded badges matching rule types
```

#### Prompt 1.7: Config Key Create/Edit Form
```
Create the Config Key create/edit form modal:

COMPONENT: Dialog modal (shadcn/ui Dialog component)

FORM FIELDS:

1. Basic Information Tab:
   - Key Name: text input (lowercase, no spaces, validation: /^[a-z0-9_]+$/)
   - Display Name: text input (required)
   - Description: textarea (optional)
   - Data Type: select dropdown (string, integer, boolean, ip_address, time, enum)
   - Category: text input with suggestions (Network, Security, Alarm I/O, etc.)
   - Subcategory: text input (optional)

2. Default Values Tab:
   - Default Value: input (type changes based on data_type)
   - Example Value: input (type changes based on data_type)

3. Layer Availability Tab:
   Visual checkboxes for 6 layers:
   - [x] Global (always checked, disabled)
   - [ ] Model
   - [ ] Carrier
   - [ ] Service Plan
   - [ ] Company
   - [ ] Device
   Validation: At least one must be checked

4. Flags Tab:
   - [ ] Required (if checked, Global must be available)
   - [ ] Customer Configurable
   - [ ] Has Two-Way Rules (show tooltip explaining 6 combinations)
   - [ ] Has Three-Way Rules (show tooltip: service plan baselines)
   - [ ] Has Four-Way Rules (show tooltip: customer exceptions)

5. Validation Rules Tab:
   Dynamic form based on data_type:

   If string:
   - Regex Pattern: text input
   - Min Length: number input
   - Max Length: number input

   If integer:
   - Min Value: number input
   - Max Value: number input
   - [ ] Must be positive

   If enum:
   - Allowed Values: tag input (comma-separated)

   If ip_address or time or boolean:
   - Show: "Validation is automatic for this type"

VALIDATION:
- Use Zod schema for form validation
- Show inline errors on blur
- Disable submit button until form is valid
- Show validation summary at top if errors exist

ACTIONS:
- Cancel button (secondary)
- Save button (primary, disabled if invalid)
- For edit: Show "Last updated: <date> by <user>"

DESIGN:
- Modal width: 800px
- Tabbed interface for organizing sections
- Use React Hook Form for form state
- Loading state on submit
- Success toast message after save
- Auto-close modal after successful save
```

#### Prompt 1.8: Seed Data & Sample Keys
```
Create seed data script to populate the database with sample configuration keys:

SAMPLE KEYS TO CREATE (12 examples):

1. hostname:
   - display_name: "Device Hostname"
   - data_type: string
   - category: Network
   - available at: Global, Model, Company, Device
   - is_required: true
   - description: "Unique identifier for device on network"

2. lan0_ip:
   - display_name: "LAN IP Address"
   - data_type: ip_address
   - category: Network
   - available at: Global, Model, Device
   - description: "IPv4 address for LAN interface"

3. dns_primary:
   - display_name: "Primary DNS Server"
   - data_type: ip_address
   - category: Network
   - available at: Global, Company
   - default_value: "8.8.8.8"

4. mqtt_enable:
   - display_name: "MQTT Connection Enable"
   - data_type: boolean
   - category: Device Features
   - available at: Global, Model, Carrier
   - has_two_way_rules: true
   - description: "Enable MQTT connection (varies by model+carrier)"

5. traffic_day_threshold:
   - display_name: "Daily Traffic Threshold"
   - data_type: integer
   - category: Data Management
   - available at: Global, Service Plan
   - has_two_way_rules: true
   - has_three_way_rules: true
   - validation_rules: {"min_value": 0, "max_value": 10000}
   - description: "Daily data limit in MB"

6. fw_acl:
   - display_name: "Firewall Access Control List"
   - data_type: string
   - category: Security
   - available at: Global, Service Plan
   - has_three_way_rules: true
   - has_four_way_rules: true
   - description: "Firewall rules (varies by plan + customer exceptions)"

7. reboot_time:
   - display_name: "Scheduled Reboot Time"
   - data_type: time
   - category: Scheduling
   - available at: Global, Company, Device
   - is_customer_configurable: true
   - default_value: "03:00"

8. wan1_proto:
   - display_name: "WAN Protocol"
   - data_type: enum
   - category: Network
   - available at: Global, Carrier
   - validation_rules: {"allowed_values": ["dialup", "static", "dhcp"]}

9. ntp_server:
   - display_name: "NTP Server"
   - data_type: string
   - category: Network
   - available at: Global, Company
   - has_four_way_rules: true
   - is_customer_configurable: true
   - default_value: "pool.ntp.org"

10. alarm_io_1:
    - display_name: "Alarm I/O Pin 1"
    - data_type: integer
    - category: Alarm I/O
    - available at: Global, Model
    - validation_rules: {"min_value": 0, "max_value": 1}

11. advanced:
    - display_name: "Advanced Mode Enable"
    - data_type: boolean
    - category: Device Features
    - available at: Model, Carrier
    - has_two_way_rules: true
    - description: "Enable advanced features (model+carrier specific)"

12. lan_subnet:
    - display_name: "LAN Subnet Mask"
    - data_type: ip_address
    - category: Network
    - available at: Global, Device
    - default_value: "255.255.255.0"

Also create sample data for:
- 3 models: i-22, 4100, 4500
- 3 carriers: VZW (Verizon), ATT (AT&T), TMO (T-Mobile)
- 3 service plans: ATM, TIER1, TIER2
- 2 companies: CORD Company, Miele Corporation
- 5 devices (mix of models/carriers/plans/companies)

Create Supabase Edge Function or SQL migration script to insert this data.
```

---

## Phase 2: Layer Configuration System

### Goals
- Build 6 layer configuration interfaces
- Implement inheritance display (Global only for single-factor layers)
- Add conditional rules indicators
- Create contextual rules access buttons
- Build reusable layer editor components

### Lovable Prompts

#### Prompt 2.1: Global Layer Configuration Interface
```
Create the Global Layer configuration interface:

PAGE: /layers/global

LAYOUT:
1. Page Header:
   - Title: "Global Configuration"
   - Subtitle: "System-wide defaults for all 600+ parameters"
   - Icon: Globe icon
   - Stats: "Affects X devices across Y companies"

2. Priority Context Panel (collapsible info box):
   ```
   📊 Priority Level 9 of 11

   Global Layer provides system-wide defaults for all devices.

   Priority Hierarchy:
   1. Device Override (highest)
   2. Four-Way Rule
   3. Company Override
   4. Three-Way Rule
   5. Service Plan Layer
   6. Two-Way Rules (6 sub-priorities)
   7. Carrier Layer
   8. Model Layer
   → 9. Global Layer ◄ YOU ARE HERE
   10. Schema Default
   11. Required Validation (lowest)

   ℹ️ Values set here are overridden by any higher-priority configuration.
   Devices fall back to Global when no other layer or rule provides a value.

   [View Full Priority Explanation]
   ```

3. Toolbar:
   - Search box (filter keys)
   - Category filter dropdown
   - Toggle: "Show only set values" / "Show all keys"
   - "Save Changes" button (primary, sticky at bottom)
   - "Export Global Config" button

3. Parameter Groups (Collapsible Sections):
   Organize keys by category:
   - Network Configuration (expanded by default)
   - Security & Firewall
   - Alarm I/O
   - Scheduling & Timing
   - Device Features
   - Data Management

4. Parameter Display (for each key):
   Layout:
   - Left: Key label and description
     - Display Name (bold)
     - key_name (gray, small, monospace)
     - Description (gray, smaller)
   - Right: Value editor
     - Input field (type based on data_type)
     - Required indicator (red asterisk if is_required)
     - Current value or "Not set"

   If has conditional rules flags:
   - Show: ⚠️ "Uses Conditional Rules" badge
   - Tooltip: "This parameter has [2-way/3-way/4-way] rules defined"

VALUE EDITORS (based on data_type):
- string: text input
- integer: number input with validation
- boolean: toggle switch (styled)
- ip_address: 4-part IP input (xxx.xxx.xxx.xxx)
- time: time picker (HH:MM dropdown)
- enum: select dropdown with allowed values

VALIDATION:
- Inline validation on blur
- Show validation errors below input
- Required fields cannot be empty
- Respect validation_rules from schema

SAVE BEHAVIOR:
- Track modified values (show badge count)
- Preview affected devices before save
- Confirmation dialog: "Save changes to Global config? This will affect X devices."
- Loading state during save
- Success toast: "Global configuration saved. X devices will receive updates."

DESIGN:
- Use card-based layout for parameter groups
- Clean, organized appearance
- Visual grouping with section headers
- Collapsible sections to reduce clutter
- Sticky save button bar at bottom
```

#### Prompt 2.2: Model Layer Configuration Interface
```
Create the Model Layer configuration interface:

PAGE: /layers/model/:modelId

LAYOUT:
1. Page Header:
   - Breadcrumb: "Layer Configurations > Model"
   - Model selector dropdown (select different model)
   - Current model: "i-22" (large text with icon)
   - Stats: "X devices of this model will be affected"

2. Priority Context Panel (collapsible):
   ```
   📊 Priority Level 8 of 11

   Model Layer provides defaults for all devices of this model type.

   Priority Hierarchy:
   1. Device Override (highest)
   2. Four-Way Rule
   3. Company Override
   4. Three-Way Rule
   5. Service Plan Layer
   6. Two-Way Rules (including 6.3: Model+Carrier, 6.2: Model+Plan, 6.5: Model+Customer)
   7. Carrier Layer
   → 8. Model Layer ◄ YOU ARE HERE
   9. Global Layer
   10. Schema Default
   11. Required Validation (lowest)

   ℹ️ Values set here apply to all devices of this model, but can be overridden
   by higher-priority configurations. Inherits from Global Layer (Level 9).

   💡 Tip: For model+carrier or model+plan combinations, use Two-Way Rules (Level 6).

   [View Full Priority Explanation]
   ```

3. Parameter Display:
   For each key where available_at_model = true:

   Layout (3-column):
   - Left: Parameter info (same as Global layer)
   - Center: Inherited Value Display
     - Show: <value> with label "Global"
     - Styled with green background, grayed out, not editable
     - If no Global value: "Not set"
   - Right: Override Value Editor
     - Input field for override
     - Buttons: "Override" (if not overridden), "Clear Override" (if overridden)
     - Visual indicator:
       - Green checkmark icon: Inherited (no override)
       - Blue edit icon: Overridden
       - Gray X icon: Explicitly cleared (NULL)

CONDITIONAL RULES INDICATOR:
- If key has has_two_way_rules/has_three_way_rules/has_four_way_rules = true:
  - Show: ⚠️ "Uses Conditional Rules" badge
  - Link: "View Rules" (opens filtered Conditional Rules page)
  - Tooltip: "This parameter may depend on Model+Carrier, Model+Plan, or other combinations"

CONTEXTUAL RULES ACCESS:
- Button at top: "View Conditional Rules involving this Model"
- Opens /rules page filtered to show:
  - Two-Way Rules: where model_id = this model
  - Three-Way Rules: where model_id = this model
  - Four-Way Rules: where model_id = this model
- Badge count: "5 rules involving this model"

NULL HANDLING:
- "Clear Override" button sets is_null = true (explicitly cleared)
- Explicitly cleared values show: "-" with gray X icon and tooltip "Explicitly cleared"
- Empty override (not set) falls back to Global value

SAVE BEHAVIOR:
- Preview: "This will update X devices of model i-22"
- Confirmation if > 10 devices affected
- Track which keys were modified

DESIGN:
- 3-column layout clearly shows: Parameter | Inherited | Override
- Color-coding: Green = inherited, Blue = overridden, Gray = cleared
- Clean visual separation between columns
```

#### Prompt 2.3: Carrier Layer Configuration Interface
```
Create the Carrier Layer configuration interface:

PAGE: /layers/carrier/:carrierId

LAYOUT: Similar to Model layer

KEY DIFFERENCES:

1. Header:
   - Carrier selector dropdown
   - Current carrier: "Verizon (VZW)"
   - Stats: "X devices on this carrier"

2. Priority Context Panel (collapsible):
   ```
   📊 Priority Level 7 of 11

   Carrier Layer provides defaults for all devices on this carrier network.

   Priority Hierarchy:
   1. Device Override (highest)
   2. Four-Way Rule
   3. Company Override
   4. Three-Way Rule
   5. Service Plan Layer
   6. Two-Way Rules (including 6.3: Model+Carrier, 6.1: Carrier+Plan, 6.4: Carrier+Customer)
   → 7. Carrier Layer ◄ YOU ARE HERE
   8. Model Layer
   9. Global Layer
   10. Schema Default
   11. Required Validation (lowest)

   ℹ️ Values set here apply to all devices on this carrier, but can be overridden
   by higher-priority configurations. Inherits from Global Layer (Level 9).

   ⚠️ Note: Cannot show "inherited from Model" because there are multiple models.
   For carrier+model combinations, use Two-Way Rules (Level 6.3).

   💡 Tip: For carrier+plan combinations, use Two-Way Rules (Level 6.1).

   [View Full Priority Explanation]
   ```

3. Inheritance Display:
   - Show: <value> with label "Global" (green background)
   - Note below header:
     "⚠️ Note: Cannot show 'inherited from Model' because there are multiple models (i-22, 4500, Origin, etc.). To set values for specific Carrier+Model combinations, use Two-Way Rules."
   - This note should be in an info box with light blue background

3. Conditional Rules Indicator:
   - Same as Model layer
   - Emphasize: "Use Conditional Rules for multi-factor combinations"

4. Contextual Rules Access:
   Button: "Manage Conditional Rules for Verizon"
   Shows rules where carrier_id = this carrier:
   - Two-Way Rules:
     - Model + Carrier
     - Carrier + Service Plan
     - Carrier + Customer
   - Three-Way Rules: Model + Carrier + Service Plan
   - Four-Way Rules: Model + Carrier + Service Plan + Customer

   Badge count: "8 rules involving Verizon"

5. Save Preview:
   - Show device breakdown: "500 i-22 devices, 300 4100 devices, 200 4500 devices"
   - Breakdown by model shows affected device count per model

DESIGN:
- Same 3-column layout as Model layer
- Info box explaining inheritance limitation (prominent, not dismissible)
- Conditional rules button more prominent (since it's the primary way to set model-specific values)
```

#### Prompt 2.4: Service Plan Layer Configuration Interface
```
Create the Service Plan Layer configuration interface:

PAGE: /layers/service-plan/:servicePlanId

LAYOUT: Similar to Model and Carrier layers

KEY DIFFERENCES:

1. Header:
   - Service Plan selector dropdown
   - Current plan: "ATM - Unlimited Transactions"
   - Stats: "X devices across Y companies"

2. Priority Context Panel (collapsible):
   ```
   📊 Priority Level 5 of 11

   Service Plan Layer provides defaults for all devices on this service plan.

   Priority Hierarchy:
   1. Device Override (highest)
   2. Four-Way Rule
   3. Company Override
   4. Three-Way Rule (Model+Carrier+Plan baselines)
   → 5. Service Plan Layer ◄ YOU ARE HERE
   6. Two-Way Rules (including 6.1: Carrier+Plan, 6.2: Model+Plan, 6.6: Plan+Customer)
   7. Carrier Layer
   8. Model Layer
   9. Global Layer
   10. Schema Default
   11. Required Validation (lowest)

   ℹ️ Values set here apply to all devices on this plan, but can be overridden
   by higher-priority configurations. Inherits from Global Layer (Level 9).

   ⚠️ Note: Cannot show "inherited from Model/Carrier" because service plans
   are used across different model+carrier combinations.

   💡 Tips:
   - For plan+carrier combinations: Use Two-Way Rules (Level 6.1)
   - For plan+model combinations: Use Two-Way Rules (Level 6.2)
   - For plan baselines by model+carrier: Use Three-Way Rules (Level 4)

   [View Full Priority Explanation]
   ```

3. Inheritance Display:
   - Show: <value> with label "Global"
   - Note below header:
     "⚠️ Note: Cannot show 'inherited from Model/Carrier' because service plans are used across different model+carrier combinations. To set values for specific combinations:
     - ServicePlan + Model → Two-Way Rule
     - ServicePlan + Carrier → Two-Way Rule
     - ServicePlan + Model + Carrier → Three-Way Rule"
   - Info box with light blue background

3. Contextual Rules Access:
   Button: "Manage Conditional Rules for ATM Plan"
   Shows rules where service_plan_id = this plan:
   - Two-Way Rules:
     - Carrier + Service Plan
     - Model + Service Plan
     - Service Plan + Customer
   - Three-Way Rules: Model + Carrier + Service Plan (service plan baselines)
   - Four-Way Rules: Model + Carrier + Service Plan + Customer (customer exceptions)

   Badge count: "12 rules involving ATM plan"

4. Save Preview:
   - Show: "5 companies with 1,200 total devices"
   - Breakdown by company with device counts

DESIGN:
- Same 3-column layout
- Prominent info box explaining why multi-factor combinations need rules
- Examples in info box showing the three types of combinations
```

#### Prompt 2.5: Company Layer Configuration Interface (Admin)
```
Create the Company Layer configuration interface (Admin view):

PAGE: /layers/company/:companyId

LAYOUT: Similar to previous layers

KEY DIFFERENCES:

1. Header:
   - Company selector (search/dropdown)
   - Current company: "CORD Company"
   - Stats: "X devices in this company"
   - Role badge: "Admin View"

2. Priority Context Panel (collapsible):
   ```
   📊 Priority Level 3 of 11

   Company Layer provides portfolio-wide settings for all devices in this company.

   Priority Hierarchy:
   1. Device Override (highest)
   2. Four-Way Rule (Customer-specific exceptions)
   → 3. Company Override ◄ YOU ARE HERE
   4. Three-Way Rule (Service plan baselines)
   5. Service Plan Layer
   6. Two-Way Rules (including 6.4: Carrier+Customer, 6.5: Model+Customer, 6.6: Plan+Customer)
   7. Carrier Layer
   8. Model Layer
   9. Global Layer
   10. Schema Default
   11. Required Validation (lowest)

   ℹ️ Values set here apply to ALL devices in this company (regardless of
   model/carrier/plan), but can be overridden by higher-priority configurations.
   Inherits from Global Layer (Level 9).

   ⚠️ Note: Cannot show "inherited from Model/Carrier/ServicePlan" because
   companies have devices with multiple different combinations.

   Example: CORD has 200 i-22+VZW+ATM, 150 4500+ATT+ATM, 100 i-22+VZW+Tier1 devices.

   💡 Tips:
   - For customer+carrier: Use Two-Way Rules (Level 6.4)
   - For customer+model: Use Two-Way Rules (Level 6.5)
   - For customer exceptions to plan baselines: Use Four-Way Rules (Level 2)

   [View Full Priority Explanation]
   ```

3. Inheritance Display:
   - Show: <value> with label "Global"
   - Note below header:
     "⚠️ Note: Cannot show 'inherited from Model/Carrier/ServicePlan' because companies have devices with multiple different combinations.

     Example: CORD has:
     - 200 i-22 + VZW + ATM devices
     - 150 4500 + ATT + ATM devices
     - 100 i-22 + VZW + Tier1 devices

     To set customer-specific values:
     - Customer + Model → Two-Way Rule
     - Customer + Carrier → Two-Way Rule
     - Customer + ServicePlan → Two-Way Rule
     - Customer + Model + Carrier + ServicePlan → Four-Way Rule"
   - Info box with example data

3. Contextual Rules Access:
   Button: "Manage Conditional Rules for CORD Company"
   Shows rules where company_id = this company:
   - Two-Way Rules:
     - Model + Customer
     - Carrier + Customer
     - Service Plan + Customer
   - Four-Way Rules: Model + Carrier + Service Plan + Customer (customer exceptions)

   Badge count: "5 customer-specific rules"

   Note: "Four-Way Rules require corresponding Three-Way Rule baseline"

4. Save Preview:
   - "This will update 500 devices in CORD Company"
   - Confirmation required before saving
   - Warning if changes will override conditional rules

DESIGN:
- Same 3-column layout
- Detailed info box with concrete example
- Prominent contextual rules button
- Clear indication this is Admin view (can edit all keys)
```

#### Prompt 2.6: Company Layer Configuration Interface (Customer)
```
Create the Company Layer configuration interface (Customer view):

PAGE: /layers/company (customer can only see own company)

LAYOUT: Simplified customer-friendly interface

KEY DIFFERENCES FROM ADMIN VIEW:

1. Header:
   - No company selector (locked to own company)
   - Company name: "Your Company Configuration"
   - Subtitle: "Portfolio-wide settings for all your devices"
   - Stats: "This will apply to 500 of your devices"
   - Role badge: "Customer View"

2. Priority Context Panel (simplified, collapsible):
   ```
   📊 How Your Configurations Work

   Settings you configure here apply to ALL your devices.

   Configuration Priority:
   1. Individual Device Settings (highest priority)
      - Settings for a specific device override all other settings

   → 2. Your Company Settings ◄ YOU ARE HERE
      - Portfolio-wide settings for all your devices

   3. System Defaults
      - Standard defaults provided by your service plan and system settings

   ℹ️ Changes here affect all 500 devices in your fleet.
   Individual device settings will override these company-wide settings.

   [Learn More About Configuration Priority]
   ```

3. Parameter Display:
   - Show ONLY keys where is_customer_configurable = true
   - All other keys completely hidden (not shown as read-only)
   - Simplified 2-column layout: Parameter | Value

3. Inheritance Display:
   - Show inherited value with simplified label: "System Default"
   - Don't show layer source details
   - For parameters with conditional rules:
     - Show: "⚠️ Managed via Conditional Rules"
     - Tooltip: "This value is managed by system administrators based on your service plan and device configuration"
     - NOT EDITABLE by customer

4. User-Friendly Input Controls:
   - IP Address: 4 separate octet inputs (xxx . xxx . xxx . xxx)
   - Time: Time picker with hour/minute dropdowns
   - Boolean: Toggle switch with labels (Enable/Disable)
   - Enum: Dropdown with friendly labels
   - String: Text input with placeholder and validation

5. Validation:
   - Inline validation with friendly error messages
   - Red border + error message below input
   - Cannot enter invalid values
   - Preview shows: "✓ Valid" or "⚠️ Requires correction"

6. Save Behavior:
   - "Apply to All Devices" button (large, primary)
   - Preview: "This will update 500 of your devices"
   - Confirmation dialog: "Are you sure you want to apply these changes to all 500 devices in your fleet?"
   - Success message: "Configuration applied successfully. Your devices will update on next check-in."

7. No Conditional Rules Access:
   - Customers cannot create or edit conditional rules
   - Parameters with rules show read-only indicator

DESIGN:
- Simplified, consumer-friendly interface
- Large, easy-to-use input controls
- Clear validation feedback
- No technical jargon
- Prominent save button
- Helpful tooltips and guidance text
```

#### Prompt 2.7: Device Layer Configuration Interface (Admin)
```
Create the Device Layer configuration interface (Admin view):

PAGE: /layers/device/:deviceId

LAYOUT: Most detailed view with full 11-level resolution display

KEY FEATURES:

1. Header:
   - Device selector (search by device code/name)
   - Current device: "ATM_STORE_001"
   - Device details:
     - Model: i-22
     - Carrier: Verizon (VZW)
     - Service Plan: ATM
     - Company: CORD Company
   - Badge: "Admin View"

2. Priority Context Panel (collapsible):
   ```
   📊 Priority Level 1 of 11 - Full Resolution View

   Device Layer shows the FINAL RESOLVED VALUE for each parameter with complete
   source attribution across all 11 priority levels.

   Complete Priority Hierarchy:
   → 1. Device Override ◄ YOU ARE HERE (HIGHEST PRIORITY)
   2. Four-Way Rule (i-22 + VZW + ATM + CORD)
   3. Company Override (CORD)
   4. Three-Way Rule (i-22 + VZW + ATM)
   5. Service Plan Layer (ATM)
   6. Two-Way Rules:
      6.1: Carrier + Plan (VZW + ATM)
      6.2: Model + Plan (i-22 + ATM)
      6.3: Model + Carrier (i-22 + VZW)
      6.4: Carrier + Customer (VZW + CORD)
      6.5: Model + Customer (i-22 + CORD)
      6.6: Plan + Customer (ATM + CORD)
   7. Carrier Layer (VZW)
   8. Model Layer (i-22)
   9. Global Layer
   10. Schema Default
   11. Required Validation (lowest)

   ✓ Why full resolution works here:
   This device has specific Model (i-22), Carrier (VZW), ServicePlan (ATM),
   and Company (CORD), so the complete 11-level resolution algorithm can execute
   and show exactly where each value comes from.

   ℹ️ Device Overrides (Level 1) have HIGHEST priority and override everything else.

   [View Full Priority Explanation]
   ```

3. Parameter Display (3-column + source details):
   - Left: Parameter info
   - Center: Current Value with Full Source Attribution
   - Right: Override editor

3. SOURCE ATTRIBUTION:
   For each parameter, show where the current value comes from:

   Example displays:
   - "8.8.8.8" (Global)
   - "192.168.1.90" (Model Layer: i-22)
   - "Matrxatm.gw12.vzwentp" (Carrier Layer: VZW)
   - "1" (Two-Way Rule: Model=i-22 + Carrier=VZW)
   - "3584MB" (Two-Way Rule: Carrier=VZW + Plan=ATM)
   - "[40 rules]" (Three-Way Rule: Model=i-22 + Carrier=VZW + Plan=ATM)
   - "[50 custom rules]" (Four-Way Rule: Customer Exception for CORD)
   - "02:00" (Company Layer: CORD)
   - "192.168.1.100" (Device Override)

   Source Display Format:
   - Value in bold
   - Source in parentheses, color-coded:
     - Gray: Global, Schema Default
     - Blue: Layer (Model/Carrier/ServicePlan/Company)
     - Purple: Conditional Rules (Two-Way/Three-Way/Four-Way)
     - Green: Device Override

4. Why This Works:
   Info box at top:
   "✓ This device has specific Model (i-22), Carrier (VZW), ServicePlan (ATM), and Company (CORD), so the complete 11-level resolution algorithm can execute and show exactly where each value comes from."

5. Visual Indicators:
   - Color-code by source type
   - Icon next to each source:
     - 🌐 Global
     - 📱 Model Layer
     - 📡 Carrier Layer
     - 📋 Service Plan Layer
     - 🏢 Company Layer
     - 🔵 Two-Way Rule
     - 🟣 Three-Way Rule
     - 🔴 Four-Way Rule
     - 💻 Device Override

6. Override Controls:
   - Input field for new value
   - "Override" button
   - "Clear Override" button
   - Tooltip: "Device overrides have highest priority (Level 1)"

7. Save Behavior:
   - Apply to single device only
   - Confirmation: "Apply config changes to Device ATM_STORE_001?"
   - Success: "Device configuration updated. Changes will apply on next check-in."

DESIGN:
- Rich, detailed interface
- Clear visual hierarchy showing resolution path
- Color-coding helps understand source types
- Icons make it visually scannable
- Info box explains why full resolution is possible
```

#### Prompt 2.8: Device Layer Configuration Interface (Customer)
```
Create the Device Layer configuration interface (Customer view):

PAGE: /layers/device/:deviceId (customer can only see own devices)

LAYOUT: Simplified customer view with friendly labels

KEY FEATURES:

1. Header:
   - Device selector (filtered to own company)
   - Device name: "ATM_STORE_001"
   - Device info (simplified):
     - Location: [if available]
     - Status: Online/Offline
     - Last Check-in: <timestamp>
   - Badge: "Customer View"

2. Priority Context Panel (simplified, collapsible):
   ```
   📊 How Device Configuration Works

   You're viewing settings for a specific device in your fleet.

   Configuration Priority:
   → 1. This Device's Settings ◄ YOU ARE HERE (HIGHEST PRIORITY)
      - Custom settings for this specific device
      - Overrides all other settings

   2. Your Company Settings
      - Portfolio-wide settings you configured for all devices

   3. System Defaults
      - Service plan defaults and system-wide settings

   ℹ️ Current Value Shows:
   Each parameter displays its current value and where it comes from:
   - "This Device" = Custom setting for this device
   - "Your Company Settings" = Company-wide configuration
   - "Service Plan Settings" = Defaults from your service plan
   - "System Default" = Standard system defaults

   💡 Changes here affect ONLY this device.

   [Learn More About Configuration Priority]
   ```

3. Parameter Display:
   - Show ONLY keys where is_customer_configurable = true AND available_at_device = true
   - 2-column layout: Parameter | Current Value

3. Source Attribution (Simplified):
   Instead of technical layer names, show friendly labels:
   - "System Default" - Global or schema default
   - "Your Company Settings" - Company Layer value
   - "Service Plan Settings" - Service Plan Layer or Three-Way Rule baseline
   - "Custom Configuration" - Two-Way Rule or Four-Way Rule (customer-specific)
   - "This Device" - Device Override

   Color-coding (subtle):
   - System Default: Gray
   - Your Company Settings: Blue
   - Custom Configuration: Purple
   - This Device: Green

4. Parameter Display Example:
   ```
   Primary DNS Server
   dns_primary
   Current value: 8.8.8.8 (System Default)
   [input field] [Update button]
   ```

5. User-Friendly Controls:
   - Same as Company Layer customer view
   - IP inputs, time pickers, toggles, dropdowns
   - Inline validation with friendly messages

6. Parameters with Conditional Rules:
   - Show: Current value with "(Custom Configuration)" label
   - Tooltip: "This setting is customized for your devices"
   - May or may not be editable depending on is_customer_configurable flag

7. Save Behavior:
   - "Update This Device" button (primary)
   - Confirmation: "Apply these changes to Device ATM_STORE_001?"
   - Success: "Configuration updated. Device will receive changes on next check-in (typically within 15 minutes)."

DESIGN:
- Clean, simple interface
- Friendly language (no technical jargon)
- Clear indication of current value and source
- Easy-to-use input controls
- Helpful guidance text
```

#### Prompt 2.9: Reusable Layer Editor Components
```
Create reusable React components for layer configuration interfaces:

COMPONENTS TO BUILD:

1. ConfigKeyRow Component:
   Props:
   - configKey: ConfigKey (from database)
   - inheritedValue: string | null
   - inheritedSource: string (e.g., "Global", "Model Layer (i-22)")
   - currentValue: string | null
   - isOverridden: boolean
   - isNull: boolean (explicitly cleared)
   - onValueChange: (value: string) => void
   - onOverride: () => void
   - onClearOverride: () => void
   - isCustomerView: boolean
   - isEditable: boolean

   Renders:
   - Parameter info (left column)
   - Inherited value display (center column, if applicable)
   - Value editor (right column)
   - Visual indicators (icons, badges)
   - Conditional rules indicator (if has_*_rules flags)

2. ValueEditor Component:
   Props:
   - dataType: 'string' | 'integer' | 'boolean' | 'ip_address' | 'time' | 'enum'
   - value: string
   - onChange: (value: string) => void
   - validationRules: ValidationRules
   - disabled: boolean

   Renders appropriate input based on dataType:
   - StringInput
   - NumberInput (with min/max)
   - ToggleSwitch (for boolean)
   - IPAddressInput (4-part input)
   - TimePicker (HH:MM dropdowns)
   - EnumSelect (dropdown)

3. IPAddressInput Component:
   4 separate number inputs for octets
   Validation: 0-255 per octet
   Format: xxx . xxx . xxx . xxx

4. TimePicker Component:
   Hour dropdown (00-23)
   Minute dropdown (00-59)
   Format: HH:MM

5. InheritedValueDisplay Component:
   Props:
   - value: string | null
   - source: string
   - sourceType: 'global' | 'layer' | 'rule' | 'override'

   Renders:
   - Value in grayed-out box
   - Source label with color-coding
   - Icon based on sourceType

6. ConditionalRulesIndicator Component:
   Props:
   - configKey: ConfigKey
   - onViewRules: () => void

   Renders:
   - ⚠️ "Uses Conditional Rules" badge
   - Tooltip explaining which rule types
   - Link to view/manage rules

7. SourceAttributionBadge Component:
   Props:
   - source: string
   - sourceType: 'global' | 'model' | 'carrier' | 'service_plan' | 'company' | 'two_way_rule' | 'three_way_rule' | 'four_way_rule' | 'device_override'

   Renders:
   - Icon + label
   - Color-coded background
   - Tooltip with full explanation

8. LayerConfigToolbar Component:
   Props:
   - layerType: 'global' | 'model' | 'carrier' | 'service_plan' | 'company' | 'device'
   - selectedEntityId: string | null
   - onEntityChange: (id: string) => void
   - onSave: () => void
   - modifiedCount: number
   - affectedDeviceCount: number

   Renders:
   - Search box
   - Entity selector (if applicable)
   - Category filter
   - Save button with badge count
   - Stats display

DESIGN PATTERNS:
- All components use Tailwind CSS for styling
- Consistent color scheme (purple/indigo theme)
- Responsive design
- Accessible (ARIA labels, keyboard navigation)
- Loading states
- Error states
```

---

## Phase 3: Conditional Rules Management

### Goals
- Build rule creation wizard for 2-way, 3-way, 4-way rules
- Display rule coverage matrices
- Show rules list with filtering
- Implement rule validation and conflict detection

### Lovable Prompts

#### Prompt 3.1: Conditional Rules Overview Page
```
Create the Conditional Rules Management overview page:

PAGE: /rules

LAYOUT:

1. Page Header:
   - Title: "Conditional Rules Management"
   - Subtitle: "Multi-Factor configuration rules for complex scenarios"
   - Info icon with tooltip: Explains conditional rules framework

2. Statistics Dashboard (top cards):
   - Two-Way Rules: 25 (blue card, icon: 🔵)
   - Three-Way Rules: 35 (purple card, icon: 🟣)
   - Four-Way Rules: 12 (red card, icon: 🔴)
   - Total Devices Affected: 85,000+ (gray card)

3. Rule Type Tabs:
   - Two-Way Rules (blue tab)
   - Three-Way Rules (purple tab)
   - Four-Way Rules (red tab)

4. Two-Way Rules Tab Content:

   Header:
   - Title: "Two-Way Rules"
   - Description: "Define values based on ANY TWO of the four factors (Model, Carrier, Service Plan, Customer). Six combinations supported with sub-priority ordering (6.1-6.6)."

   Info Panel (expandable):
   "Supported Combinations:
   6.1: Carrier + Service Plan | 6.2: Model + Service Plan | 6.3: Model + Carrier
   6.4: Carrier + Customer | 6.5: Model + Customer | 6.6: Service Plan + Customer"

   Filters:
   - Parameter filter (dropdown)
   - Combination type filter (dropdown: all/model+carrier/carrier+plan/etc.)
   - Model filter (multi-select)
   - Carrier filter (multi-select)
   - Service Plan filter (multi-select)
   - Customer filter (multi-select)

   Rules Table:
   Columns:
   - Parameter (key_name)
   - Combination Type (badge: "Model+Carrier", "Carrier+Plan", etc.)
   - Factors (e.g., "i-22 + VZW")
   - Value
   - Sub-Priority (6.1-6.6)
   - Affected Devices (count)
   - Actions (Edit, Delete, Duplicate)

   Empty state:
   "No two-way rules defined. Click 'Create Rule' to add your first rule."

5. Three-Way Rules Tab Content:

   Header:
   - Title: "Three-Way Rules (Service Plan Baselines)"
   - Description: "Define service plan baselines that apply to ALL customers with matching Model + Carrier + Service Plan combination."

   Filters:
   - Parameter filter
   - Model filter
   - Carrier filter
   - Service Plan filter

   Rules Table:
   Columns:
   - Parameter
   - Model
   - Carrier
   - Service Plan
   - Value
   - Priority (Level 4)
   - Affected Devices
   - Customer Exceptions (count of four-way rules overriding this)
   - Actions

6. Four-Way Rules Tab Content:

   Header:
   - Title: "Four-Way Rules (Customer Exceptions)"
   - Description: "Customer-specific exceptions to Three-Way Rule baselines. Highest priority after device overrides."

   Warning banner:
   "⚠️ Four-Way Rules override Three-Way baselines. Ensure corresponding baseline exists before creating customer exceptions."

   Filters:
   - Parameter filter
   - Model filter
   - Carrier filter
   - Service Plan filter
   - Customer filter

   Rules Table:
   Columns:
   - Parameter
   - Model + Carrier + Plan (compact display)
   - Customer
   - Value
   - Overrides Baseline (link to three-way rule)
   - Priority (Level 2)
   - Affected Devices
   - Actions

7. Action Buttons:
   - "Create Rule" button (primary, top right)
   - "Import Rules" button (secondary)
   - "Export Rules" button (secondary)

DESIGN:
- Tabbed interface with color-coded tabs
- Clean table layout
- Filterable/sortable
- Visual distinction between rule types
- Badges for combination types
- Click row to view details
```

#### Prompt 3.2: Rule Creation Wizard (Two-Way Rules)
```
Create the Two-Way Rule creation wizard:

COMPONENT: Multi-step modal dialog

STEP 1: Select Parameter
- Dropdown showing config keys where has_two_way_rules = true
- Filter by category
- Search box
- Show parameter details when selected:
  - Display name
  - Data type
  - Current rule count
  - Existing combinations
- "Next" button (disabled until parameter selected)

STEP 2: Choose Factor Combination
- Title: "Which two factors determine this value?"
- Radio button selection (6 options):

  ○ Model + Carrier
    Carrier-specific features per model
    Example: i-22 + VZW → mqtt_enable=1

  ○ Carrier + Service Plan
    Carrier data policies by plan tier
    Example: VZW + ATM → traffic_day_threshold=3584MB

  ○ Model + Service Plan
    Model pricing tier by plan
    Example: i-22 + ATM → traffic_day_threshold=350MB

  ○ Carrier + Customer
    Customer carrier agreements
    Example: VZW + VIP_Corp → traffic_day_threshold=10GB

  ○ Model + Customer
    Customer model configurations
    Example: i-22 + Miele → digitalio_config=[custom]

  ○ Service Plan + Customer
    Customer plan exceptions
    Example: ATM + Premium → traffic_day_threshold=1000MB

- Each option shows:
  - Radio button
  - Title (bold)
  - Use case description
  - Example
- "Next" button (disabled until selection made)

STEP 3: Select Factor Values
Dynamic form based on combination selected:

If Model + Carrier:
- Model dropdown (all active models)
- Carrier dropdown (all active carriers)

If Carrier + Service Plan:
- Carrier dropdown
- Service Plan dropdown

If Model + Service Plan:
- Model dropdown
- Service Plan dropdown

If Carrier + Customer:
- Carrier dropdown
- Customer dropdown (search/select)

If Model + Customer:
- Model dropdown
- Customer dropdown

If Service Plan + Customer:
- Service Plan dropdown
- Customer dropdown

Show existing rules for selected combination:
"⚠️ A rule already exists for i-22 + VZW (value: 1). Continue to override?"

"Next" button (disabled until both factors selected)

STEP 4: Set Value
- Value editor (appropriate input based on data_type)
- Validation feedback (inline)
- Option: "Set to NULL (explicitly disable)"
- Show sub-priority assignment:
  "This rule will be assigned sub-priority 6.3 (Model+Carrier) in the 11-level hierarchy."

STEP 5: Preview Affected Devices
- Query and show device count
- Display: "This rule will affect 150 devices"
- Device breakdown:
  - By company (if applicable)
  - By location (if available)
- Warning if >100 devices affected
- Optional description field:
  "Add a note explaining why this rule exists (optional)"

STEP 6: Confirm & Save
- Summary of rule:
  ```
  Parameter: mqtt_enable
  Combination: Model + Carrier
  Factors: i-22 + VZW
  Value: 1
  Sub-Priority: 6.3
  Affected Devices: 150
  ```
- Checkbox: "I understand this will override standard layer inheritance for 150 devices"
- "Create Rule" button (primary)
- "Back" button (review steps)
- "Cancel" button

AFTER SAVE:
- Success message: "Rule created successfully. 150 devices will receive new configuration."
- Close modal
- Refresh rules list
- Highlight newly created rule

DESIGN:
- Wizard steps indicator at top (1 of 6)
- Back/Next navigation
- Form validation at each step
- Loading states
- Error handling
```

#### Prompt 3.3: Rule Creation Wizard (Three-Way Rules)
```
Create the Three-Way Rule creation wizard:

COMPONENT: Multi-step modal dialog (similar structure to Two-Way)

STEP 1: Select Parameter
- Same as Two-Way, but filter: has_three_way_rules = true

STEP 2: Info Screen
- Title: "Three-Way Rules: Service Plan Baselines"
- Explanation:
  "Three-Way Rules define service plan baselines that apply to ALL customers with matching Model + Carrier + Service Plan combination.

  Example: All devices with Model i-22 + Carrier ATT + Plan ATM get traffic_threshold=350MB

  This creates a baseline that:
  - Applies to ALL customers (not customer-specific)
  - Can be overridden by Four-Way Rules (customer exceptions)
  - Overrides Two-Way Rules and simple layer inheritance"

- Checkbox: "I understand this creates a baseline for all customers"
- "Next" button

STEP 3: Select Factors
- Model dropdown
- Carrier dropdown
- Service Plan dropdown

Check if rule exists:
"⚠️ A baseline rule already exists for i-22 + ATT + ATM (value: 350MB)"

Show count of existing customer exceptions:
"ℹ️ 3 customers have Four-Way Rule exceptions to this baseline"

STEP 4: Set Baseline Value
- Value editor
- Label: "Baseline Value (applies to all customers)"
- Validation

STEP 5: Preview Coverage
- Show affected device count
- Breakdown by customer:
  ```
  Company A: 500 devices
  Company B: 300 devices
  Company C: 200 devices (has exception rule)
  Company D: 150 devices
  ```
- Customers with exceptions marked with icon
- Total: "1,150 devices across 4 companies"

STEP 6: Confirm & Save
- Summary showing:
  - Parameter
  - Model + Carrier + Service Plan
  - Baseline value
  - Priority Level: 4
  - Affected devices
  - Customer exceptions count
- Warning: "Four-Way Rules will override this baseline for specific customers"
- "Create Baseline Rule" button

DESIGN:
- Same wizard structure
- Purple theme (matching Three-Way Rules color)
- Emphasis on "baseline for all customers"
```

#### Prompt 3.4: Rule Creation Wizard (Four-Way Rules)
```
Create the Four-Way Rule creation wizard (Customer Exceptions):

COMPONENT: Multi-step modal dialog

STEP 1: Select Parameter
- Filter: has_four_way_rules = true

STEP 2: Info Screen
- Title: "Four-Way Rules: Customer-Specific Exceptions"
- Explanation:
  "Four-Way Rules create customer-specific exceptions to Three-Way Rule baselines.

  These rules:
  - Have highest priority (Level 2, only Device Override is higher)
  - Override Three-Way baseline for specific customer only
  - Require corresponding Three-Way baseline to exist

  Example: CORD company needs custom firewall rules that override the standard ATM plan baseline."

- Warning box: "⚠️ Create Three-Way Rule baseline first, then add Four-Way exceptions as needed."
- "Next" button

STEP 3: Select Factors
- Model dropdown
- Carrier dropdown
- Service Plan dropdown
- Customer dropdown (required)

After selecting Model + Carrier + Plan:
Check if Three-Way baseline exists:

If exists:
"✓ Baseline rule found: value = 350MB (applies to all customers)"

If not exists:
"⚠️ No baseline rule found for i-22 + ATT + ATM
Recommendation: Create Three-Way Rule baseline first
[Create Baseline] button (opens Three-Way wizard)"

STEP 4: Set Exception Value
- Label: "Customer-Specific Value"
- Show baseline value for comparison:
  ```
  Baseline Value (all customers): 350MB
  Exception Value (CORD only): [input] 1000MB
  ```
- Validation
- Option: "Set to NULL (disable for this customer)"

STEP 5: Preview Impact
- Show: "This exception will affect 500 devices in CORD Company"
- Device breakdown by model/carrier if mixed
- Show what's being overridden:
  ```
  Current baseline: 350MB (from Three-Way Rule)
  New value for CORD: 1000MB
  Change: +650MB (increased limit)
  ```

STEP 6: Confirm & Save
- Summary:
  ```
  Parameter: traffic_day_threshold
  Factors: Model i-22 + Carrier ATT + Plan ATM + Customer CORD
  Baseline Value: 350MB (all customers)
  Exception Value: 1000MB (CORD only)
  Priority Level: 2
  Affected Devices: 500 (CORD Company only)
  Reason: [optional description]
  ```
- Checkbox: "I understand this creates a customer-specific exception to the baseline"
- "Create Exception Rule" button

DESIGN:
- Red theme (matching Four-Way Rules color)
- Clear indication it's an exception
- Always show baseline value for context
- Warning if no baseline exists
```

#### Prompt 3.5: Rule Coverage Matrix View
```
Create the Rule Coverage Matrix visualization:

COMPONENT: Can be shown in rules overview or as standalone view

FOR TWO-WAY RULES:
Different matrix for each combination type:

1. Model + Carrier Matrix:
   - Rows: Models (i-22, 4100, 4500, etc.)
   - Columns: Carriers (VZW, ATT, TMO)
   - Cells:
     - Green: Rule exists
     - Gray: No rule (falls back to layers)
     - Hover shows: Rule value and affected devices
   - Click cell: View/edit rule

2. Carrier + Service Plan Matrix:
   - Rows: Carriers
   - Columns: Service Plans (ATM, TIER1, TIER2, etc.)
   - Same cell behavior

3. Model + Service Plan Matrix:
   - Rows: Models
   - Columns: Service Plans

4. Customer-Based Matrices (Carrier+Customer, Model+Customer, Plan+Customer):
   - Special view: List format instead of matrix (too many customers)
   - Table showing:
     - Customer name
     - Factor (Carrier/Model/Plan)
     - Rules count
     - Click to expand rules

FOR THREE-WAY RULES:
3D visualization (simplified as layered 2D):

Parameter selector at top (choose which parameter to visualize)

For selected parameter (e.g., fw_acl):
- Service Plan tabs (ATM, TIER1, TIER2)
- Within each tab: Model × Carrier matrix
- Cells:
  - Green: Baseline rule exists (all customers)
  - Red: Baseline exists + customer exceptions
  - Gray: No rule
  - Hover shows: Baseline value, # of exceptions
  - Click cell: View baseline rule details

Stats below matrix:
"8 baseline rules defined, 3 customer exceptions"

FOR FOUR-WAY RULES:
Customer Exception View:

- Group by customer
- For each customer:
  - Company name (expandable)
  - Exception count badge
  - Expanded view shows:
    - Model + Carrier + Plan combination
    - Baseline value
    - Exception value
    - Affected devices

Example:
```
CORD Company (3 exceptions)
▼ i-22 + ATT + ATM
  Baseline: 350MB → Exception: 1000MB
  Affects: 200 devices

▼ 4500 + ATT + ATM
  Baseline: 5GB → Exception: 10GB
  Affects: 150 devices
```

DESIGN:
- Interactive matrix (clickable cells)
- Color-coded (green = has rule, gray = no rule, red = has exceptions)
- Hover tooltips with details
- Loading skeleton while fetching data
- Export matrix as image/PDF button
```

#### Prompt 3.6: Rule Validation & Conflict Detection
```
Create rule validation and conflict detection system:

VALIDATION RULES:

1. Two-Way Rule Validation:
   - Both factors must be selected
   - Factors must exist in database (valid model, carrier, plan, customer)
   - Value must match parameter's data type
   - Value must pass parameter's validation_rules
   - Check for duplicate rule (same factors)
   - Warning if rule will never match (impossible combination)

2. Three-Way Rule Validation:
   - All three factors required (model, carrier, service plan)
   - Factors must exist
   - Value validation (same as above)
   - Check for duplicate baseline rule
   - Show count of devices that will be affected
   - Warning if no devices match this combination

3. Four-Way Rule Validation:
   - All four factors required
   - Check if corresponding Three-Way baseline exists:
     - If not: Show warning + offer to create baseline first
     - If yes: Show baseline value for comparison
   - Value validation
   - Check for duplicate exception
   - Verify customer has devices with this Model+Carrier+Plan combination
   - Warning if customer has 0 devices matching

CONFLICT DETECTION:

1. Rule Conflicts:
   - Same factors, different values → Error: "Conflicting rule exists"
   - Sub-priority conflicts (for two-way rules) → Auto-resolve based on combination type

2. Layer Override Conflicts:
   - If layer value exists at Company or Device layer for affected devices:
     - Warning: "X devices have Company/Device overrides that will take precedence over this rule"
     - Show list of affected devices

3. Rule Precedence Preview:
   When creating rule, show precedence chain:
   ```
   For Device #12345 (i-22 + VZW + ATM + CORD):

   Resolution order for traffic_day_threshold:
   1. Device Override: Not set
   2. Four-Way Rule (CORD exception): 1000MB ← This rule
   3. Company Layer (CORD): Not set
   4. Three-Way Rule (baseline): 350MB
   5. Service Plan Layer: Not set
   6. Two-Way Rule (6.1): Not set
   7. Two-Way Rule (6.2): Not set
   8. Two-Way Rule (6.3): Not set
   ...

   Result: Device will use 1000MB (your new rule)
   ```

VALIDATION UI:

1. Inline Validation:
   - Real-time validation as user types
   - Red border + error message below input
   - Green checkmark when valid

2. Validation Summary Panel:
   - Shows all validation errors
   - Shows all warnings
   - Blocks save if errors exist
   - Allows save with warnings (user must acknowledge)

3. Conflict Resolution Dialog:
   If conflict detected:
   - Show conflicting rule details
   - Options:
     - Cancel (don't create new rule)
     - Override (replace existing rule)
     - Modify (go back and change factors/value)

4. Affected Devices Preview:
   - Query database to count affected devices
   - Show breakdown
   - Warning if >100 devices
   - Error if 0 devices (rule will never apply)

API ENDPOINTS NEEDED:

- POST /api/rules/validate
  - Input: rule data
  - Output: validation errors, warnings, affected device count

- POST /api/rules/check-conflicts
  - Input: rule data
  - Output: conflicting rules, precedence preview

- GET /api/rules/:id/affected-devices
  - Output: list of devices affected by rule

DESIGN:
- Validation messages clear and actionable
- Use icons: ❌ Error, ⚠️ Warning, ✓ Valid
- Color-coding: Red = error, Orange = warning, Green = valid
- Expandable sections for detailed explanations
```

---

## Phase 4: Device Effective Config View

### Goals
- Build device config viewer showing final resolved values
- Display source attribution for each parameter
- Provide read-only view for troubleshooting
- Color-code by source type

### Lovable Prompts

#### Prompt 4.1: Device Effective Config View
```
Create the Device Effective Config View:

PAGE: /devices/:deviceId/config

PURPOSE: Read-only view showing final resolved configuration for device with complete source attribution

LAYOUT:

1. Header:
   - Title: "Effective Configuration"
   - Device info card:
     - Device Code: ATM_STORE_001
     - Device Name
     - Model: i-22
     - Carrier: Verizon (VZW)
     - Service Plan: ATM
     - Company: CORD Company
     - Status: Online/Offline
     - Last Check-in: <timestamp>

2. Actions:
   - "Export Config" button (download as JSON, YAML, or .dat file)
   - "Edit Device Config" button (opens device layer editor)
   - "View History" button (opens change history)

3. View Options:
   - Toggle: "Show all parameters" / "Show only set values"
   - Filter by category dropdown
   - Search box (filter parameters)
   - "Group by source" toggle

4. Configuration Display:

   FOR EACH PARAMETER:

   Display format:
   ```
   dns_primary
   Primary DNS Server
   Value: 8.8.8.8
   Source: Global
   [🌐 Global Layer]
   ```

   ```
   mqtt_enable
   MQTT Connection Enable
   Value: 1 (Enabled)
   Source: Two-Way Rule (Model=i-22 + Carrier=VZW)
   [🔵 Two-Way Rule - Priority Level 6.3]
   ```

   ```
   fw_acl
   Firewall Access Control List
   Value: [50 custom rules]
   Source: Four-Way Rule (Customer Exception for CORD)
   [🔴 Four-Way Rule - Priority Level 2]
   Overrides baseline: [40 rules] (Three-Way Rule)
   ```

   Layout:
   - Parameter name (bold, monospace)
   - Display name (gray, smaller)
   - Current value (large, prominent)
   - Source badge (color-coded)
   - Source icon + description
   - If overriding another rule: Show what it overrides

5. Source Color Coding:
   - Gray: Global Layer
   - Light Blue: Model Layer
   - Blue: Carrier Layer
   - Blue: Service Plan Layer
   - Indigo: Company Layer
   - Purple (light): Two-Way Rule
   - Purple (medium): Three-Way Rule
   - Purple (dark): Four-Way Rule
   - Green: Company-Specific Device Override
   - Green (bright): Device Override
   - Black: Schema Default
   - Red: Required Validation (error state)

6. Resolution Path Visualization:
   Expandable section showing full 11-level resolution walk:

   "Resolution Path for traffic_day_threshold:"
   ```
   Level 1: Device Override → Not set
   Level 2: Four-Way Rule → 1000MB ✓ MATCH (used)
   Level 3: Company Override → Not set
   Level 4: Three-Way Rule → 350MB (overridden by Level 2)
   Level 5: Service Plan Layer → Not set
   Level 6.1: Two-Way (Carrier+Plan) → Not set
   Level 6.2: Two-Way (Model+Plan) → Not set
   Level 6.3: Two-Way (Model+Carrier) → Not set
   Level 6.4: Two-Way (Carrier+Customer) → Not set
   Level 6.5: Two-Way (Model+Customer) → Not set
   Level 6.6: Two-Way (Plan+Customer) → Not set
   Level 7: Carrier Layer → Not set
   Level 8: Model Layer → Not set
   Level 9: Global Layer → Not set
   Level 10: Schema Default → 500MB
   Level 11: Required Validation → Pass

   Final Value: 1000MB (from Level 2: Four-Way Rule)
   ```

7. Grouping Options:
   If "Group by source" is enabled, group parameters:

   - Device Overrides (5 parameters)
   - Four-Way Rules (3 parameters)
   - Company Layer (8 parameters)
   - Three-Way Rules (12 parameters)
   - Service Plan Layer (2 parameters)
   - Two-Way Rules (7 parameters)
   - Carrier Layer (5 parameters)
   - Model Layer (10 parameters)
   - Global Layer (548 parameters)
   - Schema Defaults (0 parameters)

8. Empty/Null Value Display:
   - If value is NULL: Show "—" (em dash) with tooltip "Explicitly disabled"
   - If not set anywhere: Show "Not Set" in gray
   - If required but not set: Show "⚠️ ERROR: Required value missing" in red

ACCESS CONTROL:
- Admin: Can see all 600+ parameters
- Customer: Can only see parameters where is_customer_configurable = true OR device is in their company

DESIGN:
- Clean, card-based layout
- Parameter cards with clear visual hierarchy
- Color-coded source badges immediately visible
- Expandable resolution path (collapsed by default, expand on click)
- Read-only (no editing from this view)
- Export button prominent
- Responsive: Stack cards on mobile
```

#### Prompt 4.2: Config Resolution Preview Component
```
Create reusable Config Resolution Preview component:

COMPONENT: ResolutionPreviewPanel

PURPOSE: Shows 11-level resolution walk for a single parameter

PROPS:
- deviceId: string
- configKeyId: string
- deviceAttributes: { model, carrier, servicePlan, company }

DISPLAY:

Title: "Resolution Path for <parameter>"

For each level (1-11), show:
- Level number
- Level name
- Query performed (simplified)
- Result: "Not set" or value
- Status:
  - ✓ MATCH (this level provided the value)
  - ✗ Not set
  - ⊘ Overridden (value exists but overridden by higher priority)

Color coding:
- Green background: MATCH (used)
- Yellow background: Overridden
- Gray: Not set

Example rendering:
```
Level 1: Device Override
Query: WHERE device_id = 12345 AND config_key_id = 45
Result: Not set ✗

Level 2: Four-Way Rule (Customer Exception)
Query: WHERE model=i-22 AND carrier=VZW AND plan=ATM AND company=CORD
Result: 1000MB ✓ MATCH
[GREEN BACKGROUND]

Level 3: Company Override
Query: WHERE company_id = 5 AND config_key_id = 45
Result: Not set ✗

Level 4: Three-Way Rule (Service Plan Baseline)
Query: WHERE model=i-22 AND carrier=VZW AND plan=ATM
Result: 350MB ⊘ Overridden by Level 2
[YELLOW BACKGROUND]

... (continue for all 11 levels)
```

Final Result Box:
```
Final Resolved Value: 1000MB
Source: Four-Way Rule (Priority Level 2)
Reason: Customer-specific exception for CORD Company
```

INTERACTIVE FEATURES:
- Click on a level to see full SQL query
- Hover shows tooltip with explanation
- Link to edit value at each level (if user has permission)

API:
- GET /api/devices/:deviceId/config/:keyId/resolution
  Returns: Full resolution path data for this parameter

DESIGN:
- Vertical timeline layout
- Visual connector lines between levels
- Clear hierarchy
- Expandable SQL details
- Color-coded status indicators
```

---

## Phase 5: Validation & Safety Systems

### Goals
- Implement input validation system
- Build preview-before-save functionality
- Add confirmation dialogs for critical actions
- Create validation feedback UI
- Implement safety checks for large-scale changes

### Lovable Prompts

#### Prompt 5.1: Input Validation System
```
Create comprehensive input validation system:

VALIDATION ARCHITECTURE:

1. Schema-Based Validation (Zod schemas):

```typescript
// Base validation schema factory
function createValidationSchema(configKey: ConfigKey) {
  switch (configKey.data_type) {
    case 'string':
      return z.string()
        .min(validationRules.min_length || 0)
        .max(validationRules.max_length || 255)
        .regex(new RegExp(validationRules.regex || '.*'));

    case 'integer':
      return z.number()
        .int()
        .min(validationRules.min_value || 0)
        .max(validationRules.max_value || Number.MAX_SAFE_INTEGER);

    case 'boolean':
      return z.boolean();

    case 'ip_address':
      return z.string().regex(/^(\d{1,3}\.){3}\d{1,3}$/);

    case 'time':
      return z.string().regex(/^([01]\d|2[0-3]):([0-5]\d)$/);

    case 'enum':
      return z.enum(validationRules.allowed_values);
  }
}
```

2. Real-Time Validation:
   - Validate on blur (not on every keystroke)
   - Show validation state: ✓ Valid, ⚠️ Invalid
   - Display error message below input
   - Prevent form submission if invalid

3. Custom Validation Rules:
   For specific parameters:
   - IP Address: Validate not in reserved ranges (127.x, 0.x, 255.255.255.255)
   - Hostname: RFC 1123 compliance
   - Port: 1-65535 range
   - URL: Valid URL format
   - Email: Valid email format

4. Cross-Field Validation:
   - If multiple related fields, validate together
   - Example: Start time must be before end time
   - Example: Primary DNS cannot equal secondary DNS

VALIDATION UI COMPONENTS:

1. ValidationMessage Component:
```tsx
interface ValidationMessageProps {
  type: 'error' | 'warning' | 'info';
  message: string;
  field: string;
}

Displays:
- ❌ Error: Red background, must fix before save
- ⚠️ Warning: Yellow background, can save with acknowledgment
- ℹ️ Info: Blue background, helpful information
```

2. ValidationSummaryPanel Component:
   Displayed at top of form if errors exist:
   ```
   ⚠️ Please fix the following errors before saving:

   • hostname: Must be lowercase alphanumeric
   • lan0_ip: Invalid IP address format
   • traffic_day_threshold: Value exceeds maximum (10000 MB)
   ```
   - List all validation errors
   - Click error to scroll to field
   - Show count: "3 errors"

3. FieldValidationIndicator Component:
   Icon next to input field:
   - ✓ Green checkmark: Valid
   - ❌ Red X: Invalid
   - ⏳ Loading spinner: Validating (for async validation)
   - Empty: Not yet validated

ASYNC VALIDATION:

For complex validations requiring database queries:
- Check if hostname already exists
- Check if IP is already in use
- Verify rule combination doesn't exist
- Query affected device count

Debounce: Wait 500ms after user stops typing before validating

API Endpoints:
- POST /api/validate/hostname
- POST /api/validate/ip-address
- POST /api/validate/rule-combination

DESIGN:
- Inline validation messages below inputs
- Non-intrusive (don't block typing)
- Clear, actionable error messages
- Color-coded: Red = error, Yellow = warning, Green = valid
```

#### Prompt 5.2: Preview Before Save System
```
Create preview-before-save functionality for all config changes:

PREVIEW MODAL COMPONENT:

Triggered when user clicks "Save" button

DISPLAY SECTIONS:

1. Header:
   - Title: "Preview Configuration Changes"
   - Subtitle: "Review changes before applying"

2. Changes Summary:
   ```
   You are about to update:
   - Layer Type: Global Configuration
   - Modified Parameters: 5
   - Affected Devices: 8,500
   - Affected Companies: 45
   ```

3. Modified Values Table:
   Columns:
   - Parameter Name
   - Current Value
   - New Value
   - Change (visual diff)

   Row example:
   ```
   dns_primary
   8.8.8.8 → 1.1.1.1
   [visual: strikethrough 8.8.8.8, green 1.1.1.1]
   ```

4. Affected Devices Breakdown:
   If > 10 devices, show:
   - Total devices: 8,500
   - Breakdown by company (top 10)
   - "View full list" expandable

   If <= 10 devices, show full list:
   - Device code
   - Device name
   - Company
   - Current values for modified parameters

5. Impact Analysis:
   Automatically detect potential issues:

   ⚠️ Warnings:
   - "Changing DNS server will disrupt connectivity during update"
   - "500 devices will receive new configuration during peak hours"
   - "This overrides 12 conditional rules"

   ℹ️ Information:
   - "Devices will receive updates on next check-in (typically 5-15 minutes)"
   - "Change will be logged in audit trail"

6. Safety Checks:
   Required validations before allowing save:

   ✓ All values are valid
   ✓ No required parameters are empty
   ✓ No conflicting rules detected
   ⚠️ 2 devices currently offline (will update when they reconnect)

7. Confirmation:
   - Checkbox: "I have reviewed these changes and understand the impact"
   - Optional: Reason for change (text area)
   - "Cancel" button (secondary)
   - "Confirm & Save" button (primary, disabled until checkbox checked)

SPECIAL CASES:

High-Impact Changes:
If > 1,000 devices affected:
- Additional warning banner in red
- Require admin password re-entry
- Mandatory reason for change
- Confirmation: Type "CONFIRM" to proceed

Critical Parameters:
For parameters in "Security" or "Network" categories:
- Show additional warning
- Recommend scheduling during maintenance window
- Option to schedule change for later (future feature)

DESIGN:
- Full-screen modal (overlay)
- Clear visual hierarchy
- Color-coded change indicators:
  - Green: Addition
  - Red: Deletion
  - Yellow: Modification
- Scrollable content
- Sticky action buttons at bottom
```

#### Prompt 5.3: Confirmation Dialogs System
```
Create standardized confirmation dialog system:

DIALOG TYPES:

1. Simple Confirmation:
```typescript
interface ConfirmDialogProps {
  title: string;
  message: string;
  confirmText: string;
  cancelText: string;
  severity: 'info' | 'warning' | 'danger';
  onConfirm: () => void;
  onCancel: () => void;
}
```

Example usage:
```tsx
<ConfirmDialog
  title="Delete Configuration Key"
  message="Are you sure you want to delete 'mqtt_enable'? This will remove the key from the schema and all layer values."
  confirmText="Delete"
  cancelText="Cancel"
  severity="danger"
  onConfirm={handleDelete}
/>
```

2. Confirmation with Type-to-Confirm:
For destructive actions, require typing confirmation word:
```
Title: Delete Company Configuration
Message: This will delete all configuration values for CORD Company (500 devices).
This action cannot be undone.

Type "DELETE" to confirm:
[input field]

[Cancel] [Delete] (disabled until "DELETE" is typed)
```

3. Confirmation with Device Count:
Show affected device count prominently:
```
Title: Update Global Configuration
Message: You are about to update 5 parameters in the Global layer.
This will affect 8,500 devices across 45 companies.

Parameters to update:
• dns_primary
• ntp_server
• reboot_time
• fw_acl
• traffic_day_threshold

Continue?

[Cancel] [Update 8,500 Devices]
```

4. Confirmation with Preview:
Show before/after comparison:
```
Title: Override Layer Value
Current Value: 350MB (inherited from Global)
New Value: 1000MB (override at Company layer)

This will affect 500 devices in CORD Company.

[Cancel] [Apply Override]
```

CONFIRMATION REQUIRED FOR:

1. Delete operations:
   - Delete config key (schema)
   - Delete layer value (clear override)
   - Delete conditional rule
   - Delete device

2. Bulk updates:
   - Update > 100 devices
   - Update Global layer
   - Update Company layer (all devices in company)

3. Destructive changes:
   - Clear required parameter
   - Override security parameters
   - Disable critical features

4. Rule operations:
   - Delete rule affecting > 50 devices
   - Create Four-Way rule without baseline
   - Override Three-Way baseline

DESIGN:
- Modal dialog (centered)
- Icon matching severity:
  - Info: Blue ℹ️
  - Warning: Yellow ⚠️
  - Danger: Red 🛑
- Clear, concise message
- Primary action button color-coded:
  - Info: Blue
  - Warning: Orange
  - Danger: Red
- Always show cancel button (easy to dismiss)
```

#### Prompt 5.4: Rollback & Undo System
```
Create undo/rollback functionality for configuration changes:

UNDO SYSTEM:

1. Undo Buffer:
   Store last 10 actions in memory:
   - Action type (create, update, delete)
   - Entity type (config_value, conditional_rule)
   - Entity ID
   - Old state
   - New state
   - Timestamp
   - User

2. Undo Button:
   - Displayed after successful save
   - Toast notification: "Configuration saved. [Undo]"
   - Undo button available for 30 seconds
   - Click to revert changes

3. Undo Confirmation:
   ```
   Title: Undo Configuration Change
   Message: This will revert the following changes:

   • dns_primary: 1.1.1.1 → 8.8.8.8 (reverted)
   • ntp_server: time.google.com → pool.ntp.org (reverted)

   Affected devices: 8,500

   Continue?

   [Cancel] [Undo Changes]
   ```

ROLLBACK SYSTEM:

1. View Change History:
   Page: /audit-log or /history

   Display recent changes:
   - Timestamp
   - User
   - Action
   - Layer/Rule type
   - Parameters modified
   - Devices affected
   - "Rollback" button

2. Rollback Dialog:
   ```
   Title: Rollback to Previous State

   Current State (2026-01-12 14:35):
   • dns_primary: 1.1.1.1
   • ntp_server: time.google.com

   Previous State (2026-01-12 14:20):
   • dns_primary: 8.8.8.8
   • ntp_server: pool.ntp.org

   Rolling back will restore the previous state and affect 8,500 devices.

   Reason for rollback: [optional text area]

   [Cancel] [Rollback to Previous State]
   ```

3. Rollback Limitations:
   - Can rollback if:
     - Change was within last 24 hours
     - User has admin role
     - No conflicting changes made since

   - Cannot rollback if:
     - Other changes made to same parameters
     - Devices have been deleted
     - Schema has been modified

SAFETY FEATURES:

1. Version Comparison:
   Before rollback, compare current state with rollback target:
   - If different (someone else modified): Show warning
   - Offer to view diff and resolve conflicts

2. Staged Rollback:
   For high-impact rollbacks (> 1,000 devices):
   - Option to rollback in stages:
     - 10% of devices first
     - Monitor for 1 hour
     - Rollback remaining 90%
   - Automatic pause if error rate > 5%

3. Rollback History:
   Track all rollbacks in audit log:
   - Who initiated rollback
   - What was rolled back
   - Reason
   - Result (success/failure)

DESIGN:
- Undo button in toast notification (temporary)
- Rollback button in history view (persistent)
- Clear visual diff showing before/after
- Color-coded: Red = removal, Green = addition, Yellow = modification
```

---

## Phase 6: Change Detection & Versioning

### Goals
- Build comprehensive audit log system
- Track all configuration changes
- Display change history with diff view
- Implement version comparison
- Create automated change reports

### Lovable Prompts

#### Prompt 6.1: Audit Log System
```
Create comprehensive audit log system:

DATABASE:
Uses existing config_history table (see data model)

PAGE: /audit-log

LAYOUT:

1. Header:
   - Title: "Configuration Audit Log"
   - Subtitle: "Complete history of all configuration changes"

2. Filters (top toolbar):
   - Date range picker (default: last 7 days)
   - User filter (multi-select)
   - Entity type filter: Config Values / Conditional Rules / Schema Changes
   - Layer type filter: Global / Model / Carrier / etc.
   - Parameter filter (search/select)
   - Change type filter: Created / Updated / Deleted

3. Quick Stats:
   - Total Changes (last 7 days): 342
   - Users Active: 8
   - Devices Affected: 5,200
   - Most Modified Parameter: traffic_day_threshold (45 changes)

4. Timeline View (default):

   Grouped by date, sorted newest first:

   ```
   Today (12 changes)
   ├── 14:35 - John Doe updated Global Configuration
   │   Modified: dns_primary, ntp_server
   │   Affected: 8,500 devices
   │   [View Details] [Rollback]
   │
   ├── 11:20 - Jane Smith created Two-Way Rule
   │   Parameter: mqtt_enable
   │   Rule: Model i-22 + Carrier VZW
   │   Affected: 150 devices
   │   [View Details] [Edit Rule]
   │
   └── 09:15 - System Auto-Update
       Applied scheduled changes to Service Plan ATM
       Modified: traffic_day_threshold
       Affected: 2,000 devices
       [View Details]

   Yesterday (23 changes)
   ├── ...
   ```

5. Table View (alternative):
   Columns:
   - Timestamp
   - User (with avatar)
   - Action (badge: Created/Updated/Deleted)
   - Entity Type
   - Parameter(s)
   - Layer/Rule Type
   - Old Value → New Value
   - Devices Affected
   - Actions (View, Rollback)

6. Detail View (expandable):
   Click row to expand details:

   ```
   Change ID: abc-123-def
   Timestamp: 2026-01-12 14:35:22
   User: John Doe (john@company.com)
   Action: Updated

   Layer: Global Configuration

   Changes Made:
   ┌─────────────┬─────────────┬─────────────┐
   │ Parameter   │ Old Value   │ New Value   │
   ├─────────────┼─────────────┼─────────────┤
   │ dns_primary │ 8.8.8.8     │ 1.1.1.1     │
   │ ntp_server  │ pool.ntp.org│ time.google │
   └─────────────┴─────────────┴─────────────┘

   Affected Devices: 8,500 across 45 companies

   Reason: Migrating to Cloudflare DNS for better performance

   [View Affected Devices] [Rollback This Change] [Export Details]
   ```

SEARCH & FILTERING:

1. Full-text search:
   - Search by parameter name
   - Search by user name
   - Search by change reason
   - Search by device code

2. Advanced filters:
   - Multiple parameters
   - Date range
   - Change magnitude (filter by impact: >1000 devices)
   - Change source: Manual / API / Scheduled / System

EXPORT OPTIONS:

- Export as CSV
- Export as PDF report
- Export as JSON (API format)
- Date range selector for export

REAL-TIME UPDATES:

- New changes appear at top (auto-refresh every 30s)
- Toast notification when new change detected
- "New changes available" banner (click to refresh)

ACCESS CONTROL:

- Admin: View all changes
- Customer: View only changes to own company's devices

DESIGN:
- Timeline visualization with connecting lines
- Color-coded action badges (green=created, blue=updated, red=deleted)
- User avatars
- Expandable details
- Clear visual hierarchy
```

#### Prompt 6.2: Change Diff Viewer
```
Create visual diff viewer for configuration changes:

COMPONENT: ConfigDiffViewer

PROPS:
- oldValue: string | object
- newValue: string | object
- dataType: string
- format: 'inline' | 'side-by-side'

RENDERING:

1. Simple Values (string, integer, boolean, IP, time):

   Inline format:
   ```
   dns_primary
   [red background] 8.8.8.8 [strikethrough]
   [green background] 1.1.1.1
   ```

   Side-by-side format:
   ```
   Before                After
   ┌──────────────┐      ┌──────────────┐
   │ 8.8.8.8      │  →   │ 1.1.1.1      │
   └──────────────┘      └──────────────┘
   ```

2. Complex Values (JSON, arrays, firewall rules):

   For JSON objects, use line-by-line diff:
   ```
   {
   -  "primary": "8.8.8.8",
   +  "primary": "1.1.1.1",
      "secondary": "8.8.4.4",
   -  "timeout": 5000
   +  "timeout": 3000
   }
   ```

   Color coding:
   - Red background (-): Removed line
   - Green background (+): Added line
   - White: Unchanged line

3. Multi-Parameter Changes:

   Table view:
   ```
   ┌──────────────────┬──────────────┬──────────────┐
   │ Parameter        │ Before       │ After        │
   ├──────────────────┼──────────────┼──────────────┤
   │ dns_primary      │ 8.8.8.8      │ 1.1.1.1      │
   │ ntp_server       │ pool.ntp.org │ time.google  │
   │ reboot_time      │ 03:00        │ 04:00        │
   └──────────────────┴──────────────┴──────────────┘
   ```

   Each row clickable to expand details

4. Visual Diff for Special Types:

   IP Address:
   ```
   192.168.1.100
   [red: 192.168].[green: 10].1.100
   (changed from 192.168.1.100 to 10.1.100)
   ```

   Time:
   ```
   03:00 → 04:00
   (+1 hour)
   ```

   Boolean:
   ```
   [toggle switch: OFF → ON]
   mqtt_enable: Disabled → Enabled
   ```

5. Firewall Rules Diff (complex):

   Line-by-line comparison with syntax highlighting:
   ```
   Rule 1:  ALLOW 192.168.1.0/24 → 0.0.0.0/0 [unchanged]
   - Rule 2:  DENY 10.0.0.0/8 → 0.0.0.0/0 [removed]
   + Rule 2:  DENY 172.16.0.0/12 → 0.0.0.0/0 [added]
   Rule 3:  ALLOW * → 8.8.8.8 [unchanged]
   ```

INTERACTION:

- Hover over changed value to see tooltip with details
- Click to expand full value (if truncated)
- "Copy old value" button
- "Copy new value" button
- Toggle between inline and side-by-side view

DESIGN:
- Red/green color-coding (colorblind-safe: use icons too)
- Monospace font for technical values
- Clear visual separation between before/after
- Syntax highlighting for JSON/code
- Responsive: Stack side-by-side on mobile
```

#### Prompt 6.3: Version Comparison Tool
```
Create version comparison tool for configuration snapshots:

PAGE: /compare

PURPOSE: Compare configuration states across time or between entities

LAYOUT:

1. Comparison Mode Selector:

   Three modes:
   - Time Comparison: Compare same entity at different times
   - Entity Comparison: Compare two entities at same time
   - Baseline Comparison: Compare entity against baseline (Global/Plan)

2. TIME COMPARISON MODE:

   Selectors:
   - Entity type: Global / Model / Carrier / Service Plan / Company / Device
   - Entity: [dropdown based on type]
   - Version A: [date/time picker or "Select from history"]
   - Version B: [date/time picker or "Current"]

   Display:
   ```
   Comparing: Global Configuration

   Version A: 2026-01-10 09:00 (2 days ago)
   Version B: 2026-01-12 14:35 (Current)

   Changes: 12 parameters modified

   Modified Parameters:
   ┌─────────────────┬──────────────┬──────────────┬──────────┐
   │ Parameter       │ 2 days ago   │ Current      │ Change   │
   ├─────────────────┼──────────────┼──────────────┼──────────┤
   │ dns_primary     │ 8.8.8.8      │ 1.1.1.1      │ Modified │
   │ ntp_server      │ pool.ntp.org │ time.google  │ Modified │
   │ mqtt_enable     │ (not set)    │ 1            │ Added    │
   │ old_param       │ value123     │ (not set)    │ Removed  │
   └─────────────────┴──────────────┴──────────────┴──────────┘

   [Export Comparison] [Restore Version A]
   ```

3. ENTITY COMPARISON MODE:

   Selectors:
   - Entity type: Model / Carrier / Service Plan / Company / Device
   - Entity A: [dropdown]
   - Entity B: [dropdown]

   Display:
   ```
   Comparing: Model Configurations

   Entity A: i-22
   Entity B: 4500

   Differences: 45 parameters differ

   ┌─────────────────┬──────────────┬──────────────┬──────────┐
   │ Parameter       │ i-22         │ 4500         │ Diff     │
   ├─────────────────┼──────────────┼──────────────┼──────────┤
   │ mqtt_enable     │ 1            │ 0            │ ≠        │
   │ alarm_io_1      │ 1            │ 0            │ ≠        │
   │ advanced        │ 1            │ 1            │ =        │
   └─────────────────┴──────────────┴──────────────┴──────────┘

   Filters:
   [x] Show only differences
   [ ] Show identical values

   [Export Comparison] [Copy from i-22 to 4500]
   ```

4. BASELINE COMPARISON MODE:

   Compare device/company against baseline:

   Selectors:
   - Device: [search/select device]
   - Baseline: Global / Service Plan / Expected (from rules)

   Display:
   ```
   Comparing: Device ATM_STORE_001

   Device: ATM_STORE_001 (i-22 + VZW + ATM + CORD)
   Baseline: Service Plan ATM

   Configuration Drift: 8 parameters differ from baseline

   ┌─────────────────┬──────────────┬──────────────┬──────────┐
   │ Parameter       │ Device       │ Baseline     │ Source   │
   ├─────────────────┼──────────────┼──────────────┼──────────┤
   │ dns_primary     │ 192.168.1.1  │ 8.8.8.8      │ Override │
   │ reboot_time     │ 02:00        │ 03:00        │ Company  │
   │ mqtt_enable     │ 1            │ 0            │ 2W Rule  │
   └─────────────────┴──────────────┴──────────────┴──────────┘

   Analysis:
   • 3 Device Overrides
   • 2 Company Layer values
   • 2 Conditional Rules
   • 1 Carrier Layer value

   Drift Score: 13% (8 of 600 parameters differ)

   [Reset to Baseline] [Export Drift Report]
   ```

VISUALIZATION:

1. Side-by-side comparison table
2. Visual diff highlighting
3. Statistics summary:
   - Total parameters
   - Identical: X (%)
   - Modified: Y (%)
   - Added: Z
   - Removed: W

4. Drift score visualization (progress bar):
   ```
   Configuration Drift: ▓▓▓▓▓▓▓▓▓░░░░░░░░░░░ 13%
   ```

EXPORT OPTIONS:

- Export as CSV
- Export as PDF report
- Export as JSON
- Generate compliance report

DESIGN:
- Split-screen layout
- Color-coded differences
- Collapsible sections for categories
- Search/filter within comparison
- Sortable columns
```

#### Prompt 6.4: Automated Change Reports
```
Create automated change report generation system:

REPORT TYPES:

1. Daily Change Summary:

   Auto-generated daily report:

   ```
   WATM Configuration Management
   Daily Change Summary - January 12, 2026

   OVERVIEW
   • Total Changes: 45
   • Users Active: 6
   • Devices Affected: 3,200
   • Companies Impacted: 12

   TOP CHANGES
   1. Global Configuration Update
      User: John Doe
      Time: 14:35
      Impact: 8,500 devices
      Changes: dns_primary, ntp_server

   2. Service Plan ATM Update
      User: Jane Smith
      Time: 11:20
      Impact: 2,000 devices
      Changes: traffic_day_threshold

   [... top 10 changes ...]

   STATISTICS
   • Most Modified Parameter: traffic_day_threshold (12 changes)
   • Most Active User: John Doe (18 changes)
   • Peak Activity Time: 14:00-15:00 (15 changes)

   ALERTS
   ⚠️ 3 high-impact changes (>1000 devices affected)
   ⚠️ 2 changes to security parameters

   VIEW FULL DETAILS: [link to audit log]
   ```

2. Weekly Change Report:

   More detailed report sent every Monday:

   ```
   Weekly Configuration Change Report
   Week of January 6-12, 2026

   EXECUTIVE SUMMARY
   • 342 total changes across all layers
   • 5,200 unique devices affected
   • 98.5% of changes applied successfully
   • 3 changes rolled back due to errors

   CHANGE BREAKDOWN BY LAYER
   • Global: 12 changes (8,500 devices each)
   • Model: 45 changes (150-800 devices each)
   • Carrier: 23 changes (300-1200 devices each)
   • Service Plan: 18 changes (500-2000 devices each)
   • Company: 156 changes (50-500 devices each)
   • Device: 88 changes (1 device each)

   CONDITIONAL RULES
   • 15 Two-Way Rules created
   • 8 Three-Way Rules modified
   • 2 Four-Way Rules deleted

   TOP PARAMETERS MODIFIED
   1. traffic_day_threshold: 45 changes
   2. reboot_time: 34 changes
   3. dns_primary: 23 changes
   4. fw_acl: 18 changes
   5. mqtt_enable: 15 changes

   USER ACTIVITY
   • John Doe: 89 changes
   • Jane Smith: 67 changes
   • Bob Johnson: 45 changes
   [... all users ...]

   COMPLIANCE & SECURITY
   ✓ All changes logged in audit trail
   ✓ No unauthorized access attempts
   ⚠️ 5 changes made to security parameters (review recommended)

   RECOMMENDATIONS
   • Consider creating Two-Way Rule for mqtt_enable (modified 15 times)
   • Review traffic_day_threshold changes with network team

   DETAILED CHANGE LOG: [attachment: changes.csv]
   ```

3. On-Demand Custom Reports:

   User-configurable report builder:

   Form:
   - Report Name: [text input]
   - Date Range: [date picker]
   - Filters:
     - [ ] Users: [multi-select]
     - [ ] Parameters: [multi-select]
     - [ ] Layers: [multi-select]
     - [ ] Companies: [multi-select]
   - Grouping: By User / By Parameter / By Date / By Layer
   - Format: PDF / CSV / Excel / JSON
   - Include:
     - [x] Change summary
     - [x] Detailed change log
     - [x] Statistics & charts
     - [ ] Device breakdown
     - [ ] Compliance notes

   [Generate Report] button

REPORT DELIVERY:

1. Email delivery:
   - Daily summary: 7:00 AM to all admins
   - Weekly report: Monday 8:00 AM to managers
   - Custom reports: On-demand download or email

2. In-app notifications:
   - Toast: "Daily report generated. View Report"
   - Reports page: List of all generated reports

3. Scheduled exports:
   - Auto-export to S3/storage
   - Retention policy: Keep reports for 1 year

REPORT STORAGE:

Page: /reports

Display:
- List of generated reports
- Filters: Date, Type, User
- Download button
- Re-generate button
- Delete button (admin only)

DESIGN:
- Professional report template
- Company logo/branding
- Charts and visualizations
- Executive summary at top
- Detailed data in appendix
- Page numbers, table of contents
```

---

## Design System Guidelines

### Color Palette

**Primary Colors:**
```
Purple: #667eea (primary actions, highlights)
Indigo: #764ba2 (secondary accents)
Gradient: linear-gradient(135deg, #667eea 0%, #764ba2 100%)
```

**Semantic Colors:**
```
Success: #10b981 (green)
Warning: #f59e0b (yellow/orange)
Error: #ef4444 (red)
Info: #3b82f6 (blue)
```

**Source Attribution Colors:**
```
Global Layer: #6b7280 (gray)
Model Layer: #60a5fa (light blue)
Carrier Layer: #3b82f6 (blue)
Service Plan Layer: #6366f1 (indigo)
Company Layer: #8b5cf6 (purple)
Two-Way Rule: #a78bfa (light purple)
Three-Way Rule: #7c3aed (medium purple)
Four-Way Rule: #6d28d9 (dark purple)
Device Override: #10b981 (green)
```

**Neutral Colors:**
```
Gray 50: #f9fafb (backgrounds)
Gray 100: #f3f4f6 (subtle backgrounds)
Gray 200: #e5e7eb (borders)
Gray 300: #d1d5db (disabled text)
Gray 500: #6b7280 (secondary text)
Gray 700: #374151 (body text)
Gray 900: #111827 (headings)
```

### Typography

**Font Family:**
```
Body: Inter, system-ui, sans-serif
Monospace: 'JetBrains Mono', 'Fira Code', monospace
```

**Font Sizes:**
```
xs: 0.75rem (12px)   - Small labels
sm: 0.875rem (14px)  - Body text
base: 1rem (16px)    - Default
lg: 1.125rem (18px)  - Subheadings
xl: 1.25rem (20px)   - Section titles
2xl: 1.5rem (24px)   - Page titles
3xl: 1.875rem (30px) - Hero text
```

**Font Weights:**
```
normal: 400
medium: 500
semibold: 600
bold: 700
```

### Component Styling

**Buttons:**
```tsx
// Primary
className="bg-gradient-to-r from-purple-600 to-indigo-600 text-white px-4 py-2 rounded-lg hover:shadow-lg transition-all"

// Secondary
className="bg-gray-100 text-gray-700 px-4 py-2 rounded-lg hover:bg-gray-200 transition-colors"

// Danger
className="bg-red-600 text-white px-4 py-2 rounded-lg hover:bg-red-700 transition-colors"
```

**Cards:**
```tsx
className="bg-white rounded-lg shadow-sm border border-gray-200 p-6 hover:shadow-md transition-shadow"
```

**Input Fields:**
```tsx
className="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-purple-500 focus:border-transparent"
```

**Badges:**
```tsx
// Status badge
className="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-purple-100 text-purple-800"
```

### Spacing System

Use Tailwind's spacing scale (4px base unit):
```
1: 0.25rem (4px)
2: 0.5rem (8px)
3: 0.75rem (12px)
4: 1rem (16px)
6: 1.5rem (24px)
8: 2rem (32px)
12: 3rem (48px)
```

### Responsive Breakpoints

```
sm: 640px   - Mobile landscape
md: 768px   - Tablet
lg: 1024px  - Desktop
xl: 1280px  - Large desktop
2xl: 1536px - Extra large
```

### Icons

Use Lucide React icons:
```tsx
import { Database, Layers, GitBranch, Cpu, History } from 'lucide-react';
```

Icon sizes:
- Small: 16px
- Medium: 20px (default)
- Large: 24px
- Extra large: 32px

### Animations

**Transitions:**
```
transition-colors duration-200
transition-all duration-300
transition-shadow duration-200
```

**Hover Effects:**
- Buttons: Shadow lift + subtle scale
- Cards: Shadow increase
- Links: Color change + underline

---

## Testing Strategy

### Unit Testing

**Framework:** Vitest + React Testing Library

**Test Coverage Goals:**
- Utility functions: 100%
- Components: 80%+
- API integration: 90%+

**Key Areas:**
1. Validation logic
2. Resolution algorithm (11-level priority)
3. Form submission handlers
4. Data transformations

**Example Tests:**
```typescript
describe('ConfigKey Validation', () => {
  it('validates IP address format', () => {
    expect(validateIP('192.168.1.1')).toBe(true);
    expect(validateIP('999.999.999.999')).toBe(false);
  });

  it('validates integer range', () => {
    const rule = { min_value: 0, max_value: 10000 };
    expect(validateInteger(5000, rule)).toBe(true);
    expect(validateInteger(15000, rule)).toBe(false);
  });
});
```

### Integration Testing

**Framework:** Playwright

**Test Scenarios:**

1. Complete User Flows:
   - Login → Navigate to Schema → Create config key → Save
   - Create Two-Way Rule → Select factors → Set value → Save
   - Edit Global config → Preview changes → Confirm → Verify affected devices

2. Multi-Layer Configuration:
   - Set Global value → Set Model override → Verify device sees override
   - Create Three-Way Rule → Create Four-Way exception → Verify priority

3. Validation & Error Handling:
   - Submit invalid IP address → See error message
   - Try to save without required fields → See validation summary
   - Create duplicate rule → See conflict warning

### End-to-End Testing

**Test Critical Paths:**

1. Config Resolution:
   - Create values at multiple layers
   - Verify device shows correct resolved value
   - Check source attribution is accurate

2. Rule Creation:
   - Create Two-Way Rule
   - Verify affected device count
   - Check preview shows correct devices
   - Confirm rule applied after save

3. Change Tracking:
   - Make configuration change
   - Verify audit log entry created
   - Check change history shows diff
   - Test rollback functionality

### Performance Testing

**Metrics to Track:**
- Page load time: < 2s
- Table rendering (600 rows): < 500ms
- Search/filter response: < 300ms
- Save operation: < 1s
- Device count query: < 2s

**Load Testing:**
- Simulate 100 concurrent users
- Test bulk updates (>1000 devices)
- Measure database query performance

### Accessibility Testing

**Standards:** WCAG 2.1 AA compliance

**Key Tests:**
- Keyboard navigation (Tab, Enter, Escape)
- Screen reader compatibility (ARIA labels)
- Color contrast ratios (4.5:1 minimum)
- Focus indicators visible
- Form error announcements

**Tools:**
- axe DevTools
- NVDA/JAWS screen readers
- Keyboard-only navigation testing

### Browser Testing

**Supported Browsers:**
- Chrome (latest 2 versions)
- Firefox (latest 2 versions)
- Safari (latest 2 versions)
- Edge (latest 2 versions)

**Mobile Testing:**
- iOS Safari
- Android Chrome
- Responsive design (320px - 2560px)

---

## Implementation Checklist

### Phase 1: Foundation ✓
- [ ] Supabase project setup
- [ ] Database schema created
- [ ] Authentication system
- [ ] Main navigation
- [ ] Dashboard with 11-level priority explanation
- [ ] Config key schema CRUD
- [ ] Seed data loaded

### Phase 2: Layer Configuration ✓
- [ ] Global layer interface
- [ ] Model layer interface
- [ ] Carrier layer interface
- [ ] Service Plan layer interface
- [ ] Company layer (Admin)
- [ ] Company layer (Customer)
- [ ] Device layer (Admin)
- [ ] Device layer (Customer)
- [ ] Reusable components

### Phase 3: Conditional Rules ✓
- [ ] Rules overview page
- [ ] Two-Way Rule wizard
- [ ] Three-Way Rule wizard
- [ ] Four-Way Rule wizard
- [ ] Coverage matrix visualization
- [ ] Rule validation system

### Phase 4: Device Config View ✓
- [ ] Effective config viewer
- [ ] Source attribution display
- [ ] Resolution path visualization
- [ ] Export functionality

### Phase 5: Validation & Safety ✓
- [ ] Input validation system
- [ ] Preview before save
- [ ] Confirmation dialogs
- [ ] Undo/rollback system

### Phase 6: Change Tracking ✓
- [ ] Audit log page
- [ ] Change diff viewer
- [ ] Version comparison tool
- [ ] Automated reports

### Testing & Polish
- [ ] Unit tests (80%+ coverage)
- [ ] Integration tests
- [ ] E2E tests (critical paths)
- [ ] Accessibility audit
- [ ] Performance optimization
- [ ] Browser testing
- [ ] Mobile responsive testing
- [ ] User acceptance testing

---

## Deployment Notes

### Supabase Configuration

1. **Database Setup:**
   - Run all table creation scripts
   - Configure RLS policies
   - Set up indexes
   - Load seed data

2. **Storage:**
   - Create bucket for exports
   - Set up CORS policies
   - Configure file size limits

3. **Functions:**
   - Deploy Edge Functions (if needed)
   - Set up cron jobs for reports
   - Configure webhooks

### Environment Variables

```env
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
VITE_APP_NAME=WATM Config Management
VITE_API_BASE_URL=https://api.yourapp.com
```

### Production Checklist

- [ ] Enable database backups (daily)
- [ ] Set up monitoring (Sentry, LogRocket)
- [ ] Configure CDN for assets
- [ ] Enable SSL/HTTPS
- [ ] Set up error tracking
- [ ] Configure rate limiting
- [ ] Test disaster recovery
- [ ] Document operational procedures

---

## Estimated Timeline

- **Phase 1:** 1-2 weeks
- **Phase 2:** 2-3 weeks
- **Phase 3:** 2-3 weeks
- **Phase 4:** 1 week
- **Phase 5:** 1 week
- **Phase 6:** 1 week
- **Testing & Polish:** 1-2 weeks

**Total:** 8-12 weeks for complete implementation

---

## Success Metrics

### Technical Metrics
- Page load time < 2 seconds
- API response time < 500ms (p95)
- Database query time < 100ms (p95)
- Zero data loss incidents
- 99.9% uptime

### User Experience Metrics
- User can create config key in < 2 minutes
- User can create rule in < 3 minutes
- Device config view loads in < 1 second
- Search returns results in < 300ms
- No critical bugs in production

### Business Metrics
- 600+ configuration parameters managed
- 10,000+ devices configured
- 50+ companies supported
- <5% error rate on config updates
- 100% audit trail completeness

---

## Enhancement Prompts: Add Priority Context Panels to Existing Layer Pages

### Background
The layer configuration pages (Prompts 2.1-2.8) have already been built. These enhancement prompts add Priority Context Panels to help users understand where each layer fits in the 11-level priority hierarchy.

---

### Enhancement Prompt E1: Add Priority Panel to Global Layer
```
Add Priority Context Panel to existing Global Layer page (/layers/global):

LOCATION: Insert immediately after Page Header, before Toolbar section

COMPONENT: Collapsible info box (can use shadcn/ui Collapsible or Accordion)

CONTENT:
```tsx
<Collapsible defaultOpen={false}>
  <CollapsibleTrigger className="flex items-center gap-2 p-4 bg-blue-50 border border-blue-200 rounded-lg w-full hover:bg-blue-100 transition-colors">
    <Info className="w-5 h-5 text-blue-600" />
    <span className="font-semibold text-blue-900">📊 Priority Level 9 of 11</span>
    <ChevronDown className="w-4 h-4 ml-auto text-blue-600" />
  </CollapsibleTrigger>

  <CollapsibleContent className="p-4 bg-white border border-blue-200 border-t-0 rounded-b-lg">
    <div className="space-y-3">
      <p className="text-sm text-gray-700">
        <strong>Global Layer</strong> provides system-wide defaults for all devices.
      </p>

      <div className="bg-gray-50 p-3 rounded-md">
        <p className="text-xs font-semibold text-gray-600 mb-2">Priority Hierarchy:</p>
        <ul className="text-xs text-gray-600 space-y-1 font-mono">
          <li>1. Device Override (highest)</li>
          <li>2. Four-Way Rule</li>
          <li>3. Company Override</li>
          <li>4. Three-Way Rule</li>
          <li>5. Service Plan Layer</li>
          <li>6. Two-Way Rules (6 sub-priorities)</li>
          <li>7. Carrier Layer</li>
          <li>8. Model Layer</li>
          <li className="text-blue-600 font-bold">→ 9. Global Layer ◄ YOU ARE HERE</li>
          <li>10. Schema Default</li>
          <li>11. Required Validation (lowest)</li>
        </ul>
      </div>

      <div className="flex items-start gap-2 p-2 bg-blue-50 rounded">
        <Info className="w-4 h-4 text-blue-600 mt-0.5 flex-shrink-0" />
        <p className="text-xs text-gray-700">
          Values set here are overridden by any higher-priority configuration.
          Devices fall back to Global when no other layer or rule provides a value.
        </p>
      </div>

      <Link to="/dashboard#priority-hierarchy" className="text-xs text-blue-600 hover:text-blue-800 underline">
        View Full Priority Explanation
      </Link>
    </div>
  </CollapsibleContent>
</Collapsible>
```

STYLING:
- Collapsed: Blue background with info icon
- Expanded: White content area with hierarchy list
- Monospace font for hierarchy levels
- "YOU ARE HERE" in bold blue
- Responsive: Collapses to full width on mobile

STATE MANAGEMENT:
- Default collapsed (users can expand if needed)
- Remember state in localStorage: `priority-panel-global-expanded`
```

---

### Enhancement Prompt E2: Add Priority Panel to Model Layer
```
Add Priority Context Panel to existing Model Layer page (/layers/model/:modelId):

LOCATION: Insert after Page Header (after model selector and stats), before Parameter Display

CONTENT:
```tsx
<Collapsible defaultOpen={false}>
  <CollapsibleTrigger className="flex items-center gap-2 p-4 bg-blue-50 border border-blue-200 rounded-lg w-full hover:bg-blue-100 transition-colors">
    <Info className="w-5 h-5 text-blue-600" />
    <span className="font-semibold text-blue-900">📊 Priority Level 8 of 11</span>
    <ChevronDown className="w-4 h-4 ml-auto text-blue-600" />
  </CollapsibleTrigger>

  <CollapsibleContent className="p-4 bg-white border border-blue-200 border-t-0 rounded-b-lg">
    <div className="space-y-3">
      <p className="text-sm text-gray-700">
        <strong>Model Layer</strong> provides defaults for all devices of this model type.
      </p>

      <div className="bg-gray-50 p-3 rounded-md">
        <p className="text-xs font-semibold text-gray-600 mb-2">Priority Hierarchy:</p>
        <ul className="text-xs text-gray-600 space-y-1 font-mono">
          <li>1. Device Override (highest)</li>
          <li>2. Four-Way Rule</li>
          <li>3. Company Override</li>
          <li>4. Three-Way Rule</li>
          <li>5. Service Plan Layer</li>
          <li>6. Two-Way Rules (including 6.3: Model+Carrier, 6.2: Model+Plan, 6.5: Model+Customer)</li>
          <li>7. Carrier Layer</li>
          <li className="text-blue-600 font-bold">→ 8. Model Layer ◄ YOU ARE HERE</li>
          <li>9. Global Layer</li>
          <li>10. Schema Default</li>
          <li>11. Required Validation (lowest)</li>
        </ul>
      </div>

      <div className="flex items-start gap-2 p-2 bg-blue-50 rounded">
        <Info className="w-4 h-4 text-blue-600 mt-0.5 flex-shrink-0" />
        <p className="text-xs text-gray-700">
          Values set here apply to all devices of this model, but can be overridden
          by higher-priority configurations. Inherits from Global Layer (Level 9).
        </p>
      </div>

      <div className="flex items-start gap-2 p-2 bg-purple-50 rounded">
        <Lightbulb className="w-4 h-4 text-purple-600 mt-0.5 flex-shrink-0" />
        <p className="text-xs text-gray-700">
          <strong>Tip:</strong> For model+carrier or model+plan combinations, use Two-Way Rules (Level 6).
        </p>
      </div>

      <Link to="/dashboard#priority-hierarchy" className="text-xs text-blue-600 hover:text-blue-800 underline">
        View Full Priority Explanation
      </Link>
    </div>
  </CollapsibleContent>
</Collapsible>
```
```

---

### Enhancement Prompt E3: Add Priority Panel to Carrier Layer
```
Add Priority Context Panel to existing Carrier Layer page (/layers/carrier/:carrierId):

LOCATION: Insert after Page Header, before existing inheritance note

CONTENT:
```tsx
<Collapsible defaultOpen={false}>
  <CollapsibleTrigger className="flex items-center gap-2 p-4 bg-blue-50 border border-blue-200 rounded-lg w-full hover:bg-blue-100 transition-colors">
    <Info className="w-5 h-5 text-blue-600" />
    <span className="font-semibold text-blue-900">📊 Priority Level 7 of 11</span>
    <ChevronDown className="w-4 h-4 ml-auto text-blue-600" />
  </CollapsibleTrigger>

  <CollapsibleContent className="p-4 bg-white border border-blue-200 border-t-0 rounded-b-lg">
    <div className="space-y-3">
      <p className="text-sm text-gray-700">
        <strong>Carrier Layer</strong> provides defaults for all devices on this carrier network.
      </p>

      <div className="bg-gray-50 p-3 rounded-md">
        <p className="text-xs font-semibold text-gray-600 mb-2">Priority Hierarchy:</p>
        <ul className="text-xs text-gray-600 space-y-1 font-mono">
          <li>1. Device Override (highest)</li>
          <li>2. Four-Way Rule</li>
          <li>3. Company Override</li>
          <li>4. Three-Way Rule</li>
          <li>5. Service Plan Layer</li>
          <li>6. Two-Way Rules (including 6.3: Model+Carrier, 6.1: Carrier+Plan, 6.4: Carrier+Customer)</li>
          <li className="text-blue-600 font-bold">→ 7. Carrier Layer ◄ YOU ARE HERE</li>
          <li>8. Model Layer</li>
          <li>9. Global Layer</li>
          <li>10. Schema Default</li>
          <li>11. Required Validation (lowest)</li>
        </ul>
      </div>

      <div className="flex items-start gap-2 p-2 bg-blue-50 rounded">
        <Info className="w-4 h-4 text-blue-600 mt-0.5 flex-shrink-0" />
        <p className="text-xs text-gray-700">
          Values set here apply to all devices on this carrier, but can be overridden
          by higher-priority configurations. Inherits from Global Layer (Level 9).
        </p>
      </div>

      <div className="flex items-start gap-2 p-2 bg-yellow-50 rounded border border-yellow-200">
        <AlertCircle className="w-4 h-4 text-yellow-600 mt-0.5 flex-shrink-0" />
        <p className="text-xs text-gray-700">
          <strong>Note:</strong> Cannot show "inherited from Model" because there are multiple models.
          For carrier+model combinations, use Two-Way Rules (Level 6.3).
        </p>
      </div>

      <div className="flex items-start gap-2 p-2 bg-purple-50 rounded">
        <Lightbulb className="w-4 h-4 text-purple-600 mt-0.5 flex-shrink-0" />
        <p className="text-xs text-gray-700">
          <strong>Tip:</strong> For carrier+plan combinations, use Two-Way Rules (Level 6.1).
        </p>
      </div>

      <Link to="/dashboard#priority-hierarchy" className="text-xs text-blue-600 hover:text-blue-800 underline">
        View Full Priority Explanation
      </Link>
    </div>
  </CollapsibleContent>
</Collapsible>
```
```

---

### Enhancement Prompt E4: Add Priority Panel to Service Plan Layer
```
Add Priority Context Panel to existing Service Plan Layer page (/layers/service-plan/:servicePlanId):

LOCATION: Insert after Page Header, before existing inheritance note

CONTENT:
```tsx
<Collapsible defaultOpen={false}>
  <CollapsibleTrigger className="flex items-center gap-2 p-4 bg-blue-50 border border-blue-200 rounded-lg w-full hover:bg-blue-100 transition-colors">
    <Info className="w-5 h-5 text-blue-600" />
    <span className="font-semibold text-blue-900">📊 Priority Level 5 of 11</span>
    <ChevronDown className="w-4 h-4 ml-auto text-blue-600" />
  </CollapsibleTrigger>

  <CollapsibleContent className="p-4 bg-white border border-blue-200 border-t-0 rounded-b-lg">
    <div className="space-y-3">
      <p className="text-sm text-gray-700">
        <strong>Service Plan Layer</strong> provides defaults for all devices on this service plan.
      </p>

      <div className="bg-gray-50 p-3 rounded-md">
        <p className="text-xs font-semibold text-gray-600 mb-2">Priority Hierarchy:</p>
        <ul className="text-xs text-gray-600 space-y-1 font-mono">
          <li>1. Device Override (highest)</li>
          <li>2. Four-Way Rule</li>
          <li>3. Company Override</li>
          <li>4. Three-Way Rule (Model+Carrier+Plan baselines)</li>
          <li className="text-blue-600 font-bold">→ 5. Service Plan Layer ◄ YOU ARE HERE</li>
          <li>6. Two-Way Rules (including 6.1: Carrier+Plan, 6.2: Model+Plan, 6.6: Plan+Customer)</li>
          <li>7. Carrier Layer</li>
          <li>8. Model Layer</li>
          <li>9. Global Layer</li>
          <li>10. Schema Default</li>
          <li>11. Required Validation (lowest)</li>
        </ul>
      </div>

      <div className="flex items-start gap-2 p-2 bg-blue-50 rounded">
        <Info className="w-4 h-4 text-blue-600 mt-0.5 flex-shrink-0" />
        <p className="text-xs text-gray-700">
          Values set here apply to all devices on this plan, but can be overridden
          by higher-priority configurations. Inherits from Global Layer (Level 9).
        </p>
      </div>

      <div className="flex items-start gap-2 p-2 bg-yellow-50 rounded border border-yellow-200">
        <AlertCircle className="w-4 h-4 text-yellow-600 mt-0.5 flex-shrink-0" />
        <p className="text-xs text-gray-700">
          <strong>Note:</strong> Cannot show "inherited from Model/Carrier" because service plans
          are used across different model+carrier combinations.
        </p>
      </div>

      <div className="bg-purple-50 p-3 rounded border border-purple-200">
        <p className="text-xs font-semibold text-purple-800 mb-2">💡 Tips:</p>
        <ul className="text-xs text-gray-700 space-y-1">
          <li>• For plan+carrier combinations: Use Two-Way Rules (Level 6.1)</li>
          <li>• For plan+model combinations: Use Two-Way Rules (Level 6.2)</li>
          <li>• For plan baselines by model+carrier: Use Three-Way Rules (Level 4)</li>
        </ul>
      </div>

      <Link to="/dashboard#priority-hierarchy" className="text-xs text-blue-600 hover:text-blue-800 underline">
        View Full Priority Explanation
      </Link>
    </div>
  </CollapsibleContent>
</Collapsible>
```
```

---

### Enhancement Prompt E5: Add Priority Panel to Company Layer (Admin)
```
Add Priority Context Panel to existing Company Layer Admin page (/layers/company/:companyId):

LOCATION: Insert after Page Header, before existing inheritance note

CONTENT:
```tsx
<Collapsible defaultOpen={false}>
  <CollapsibleTrigger className="flex items-center gap-2 p-4 bg-blue-50 border border-blue-200 rounded-lg w-full hover:bg-blue-100 transition-colors">
    <Info className="w-5 h-5 text-blue-600" />
    <span className="font-semibold text-blue-900">📊 Priority Level 3 of 11</span>
    <ChevronDown className="w-4 h-4 ml-auto text-blue-600" />
  </CollapsibleTrigger>

  <CollapsibleContent className="p-4 bg-white border border-blue-200 border-t-0 rounded-b-lg">
    <div className="space-y-3">
      <p className="text-sm text-gray-700">
        <strong>Company Layer</strong> provides portfolio-wide settings for all devices in this company.
      </p>

      <div className="bg-gray-50 p-3 rounded-md">
        <p className="text-xs font-semibold text-gray-600 mb-2">Priority Hierarchy:</p>
        <ul className="text-xs text-gray-600 space-y-1 font-mono">
          <li>1. Device Override (highest)</li>
          <li>2. Four-Way Rule (Customer-specific exceptions)</li>
          <li className="text-blue-600 font-bold">→ 3. Company Override ◄ YOU ARE HERE</li>
          <li>4. Three-Way Rule (Service plan baselines)</li>
          <li>5. Service Plan Layer</li>
          <li>6. Two-Way Rules (including 6.4: Carrier+Customer, 6.5: Model+Customer, 6.6: Plan+Customer)</li>
          <li>7. Carrier Layer</li>
          <li>8. Model Layer</li>
          <li>9. Global Layer</li>
          <li>10. Schema Default</li>
          <li>11. Required Validation (lowest)</li>
        </ul>
      </div>

      <div className="flex items-start gap-2 p-2 bg-blue-50 rounded">
        <Info className="w-4 h-4 text-blue-600 mt-0.5 flex-shrink-0" />
        <p className="text-xs text-gray-700">
          Values set here apply to <strong>ALL devices</strong> in this company (regardless of
          model/carrier/plan), but can be overridden by higher-priority configurations.
          Inherits from Global Layer (Level 9).
        </p>
      </div>

      <div className="flex items-start gap-2 p-2 bg-yellow-50 rounded border border-yellow-200">
        <AlertCircle className="w-4 h-4 text-yellow-600 mt-0.5 flex-shrink-0" />
        <div className="text-xs text-gray-700">
          <p className="font-semibold mb-1">Note: Cannot show "inherited from Model/Carrier/ServicePlan"</p>
          <p>Companies have devices with multiple different combinations.</p>
          <p className="mt-2 text-gray-600 italic">
            Example: CORD has 200 i-22+VZW+ATM, 150 4500+ATT+ATM, 100 i-22+VZW+Tier1 devices.
          </p>
        </div>
      </div>

      <div className="bg-purple-50 p-3 rounded border border-purple-200">
        <p className="text-xs font-semibold text-purple-800 mb-2">💡 Tips:</p>
        <ul className="text-xs text-gray-700 space-y-1">
          <li>• For customer+carrier: Use Two-Way Rules (Level 6.4)</li>
          <li>• For customer+model: Use Two-Way Rules (Level 6.5)</li>
          <li>• For customer exceptions to plan baselines: Use Four-Way Rules (Level 2)</li>
        </ul>
      </div>

      <Link to="/dashboard#priority-hierarchy" className="text-xs text-blue-600 hover:text-blue-800 underline">
        View Full Priority Explanation
      </Link>
    </div>
  </CollapsibleContent>
</Collapsible>
```
```

---

### Enhancement Prompt E6: Add Priority Panel to Company Layer (Customer)
```
Add simplified Priority Context Panel to existing Company Layer Customer page (/layers/company):

LOCATION: Insert after Page Header (after company name and stats), before Parameter Display

NOTE: Customer view uses simplified, non-technical language

CONTENT:
```tsx
<Collapsible defaultOpen={false}>
  <CollapsibleTrigger className="flex items-center gap-2 p-4 bg-blue-50 border border-blue-200 rounded-lg w-full hover:bg-blue-100 transition-colors">
    <Info className="w-5 h-5 text-blue-600" />
    <span className="font-semibold text-blue-900">📊 How Your Configurations Work</span>
    <ChevronDown className="w-4 h-4 ml-auto text-blue-600" />
  </CollapsibleTrigger>

  <CollapsibleContent className="p-4 bg-white border border-blue-200 border-t-0 rounded-b-lg">
    <div className="space-y-3">
      <p className="text-sm text-gray-700">
        Settings you configure here apply to <strong>ALL your devices</strong>.
      </p>

      <div className="bg-gray-50 p-3 rounded-md">
        <p className="text-xs font-semibold text-gray-600 mb-2">Configuration Priority:</p>
        <div className="space-y-2 text-xs text-gray-700">
          <div className="flex items-start gap-2">
            <span className="text-gray-400">1.</span>
            <div>
              <p className="font-semibold">Individual Device Settings</p>
              <p className="text-gray-600">Settings for a specific device override all other settings</p>
            </div>
          </div>

          <div className="flex items-start gap-2 p-2 bg-blue-100 rounded">
            <span className="text-blue-600 font-bold">→</span>
            <div>
              <p className="font-semibold text-blue-900">Your Company Settings ◄ YOU ARE HERE</p>
              <p className="text-gray-700">Portfolio-wide settings for all your devices</p>
            </div>
          </div>

          <div className="flex items-start gap-2">
            <span className="text-gray-400">3.</span>
            <div>
              <p className="font-semibold">System Defaults</p>
              <p className="text-gray-600">Standard defaults provided by your service plan and system settings</p>
            </div>
          </div>
        </div>
      </div>

      <div className="flex items-start gap-2 p-2 bg-blue-50 rounded">
        <Info className="w-4 h-4 text-blue-600 mt-0.5 flex-shrink-0" />
        <p className="text-xs text-gray-700">
          Changes here affect all {deviceCount} devices in your fleet.
          Individual device settings will override these company-wide settings.
        </p>
      </div>

      <Link to="/dashboard#priority-hierarchy" className="text-xs text-blue-600 hover:text-blue-800 underline">
        Learn More About Configuration Priority
      </Link>
    </div>
  </CollapsibleContent>
</Collapsible>
```

VARIABLES:
- Replace {deviceCount} with actual device count from state/props
```

---

### Enhancement Prompt E7: Add Priority Panel to Device Layer (Admin)
```
Add comprehensive Priority Context Panel to existing Device Layer Admin page (/layers/device/:deviceId):

LOCATION: Insert after Device Details section, before Parameter Display

CONTENT:
```tsx
<Collapsible defaultOpen={true}>
  <CollapsibleTrigger className="flex items-center gap-2 p-4 bg-green-50 border border-green-200 rounded-lg w-full hover:bg-green-100 transition-colors">
    <Info className="w-5 h-5 text-green-600" />
    <span className="font-semibold text-green-900">📊 Priority Level 1 of 11 - Full Resolution View</span>
    <ChevronDown className="w-4 h-4 ml-auto text-green-600" />
  </CollapsibleTrigger>

  <CollapsibleContent className="p-4 bg-white border border-green-200 border-t-0 rounded-b-lg">
    <div className="space-y-3">
      <p className="text-sm text-gray-700">
        <strong>Device Layer</strong> shows the <strong>FINAL RESOLVED VALUE</strong> for each parameter
        with complete source attribution across all 11 priority levels.
      </p>

      <div className="bg-gray-50 p-3 rounded-md">
        <p className="text-xs font-semibold text-gray-600 mb-2">Complete Priority Hierarchy:</p>
        <ul className="text-xs text-gray-600 space-y-1 font-mono">
          <li className="text-green-600 font-bold">→ 1. Device Override ◄ YOU ARE HERE (HIGHEST PRIORITY)</li>
          <li>2. Four-Way Rule ({device.model} + {device.carrier} + {device.plan} + {device.company})</li>
          <li>3. Company Override ({device.company})</li>
          <li>4. Three-Way Rule ({device.model} + {device.carrier} + {device.plan})</li>
          <li>5. Service Plan Layer ({device.plan})</li>
          <li className="pl-4">6. Two-Way Rules:</li>
          <li className="pl-8">6.1: Carrier + Plan ({device.carrier} + {device.plan})</li>
          <li className="pl-8">6.2: Model + Plan ({device.model} + {device.plan})</li>
          <li className="pl-8">6.3: Model + Carrier ({device.model} + {device.carrier})</li>
          <li className="pl-8">6.4: Carrier + Customer ({device.carrier} + {device.company})</li>
          <li className="pl-8">6.5: Model + Customer ({device.model} + {device.company})</li>
          <li className="pl-8">6.6: Plan + Customer ({device.plan} + {device.company})</li>
          <li>7. Carrier Layer ({device.carrier})</li>
          <li>8. Model Layer ({device.model})</li>
          <li>9. Global Layer</li>
          <li>10. Schema Default</li>
          <li>11. Required Validation (lowest)</li>
        </ul>
      </div>

      <div className="flex items-start gap-2 p-2 bg-green-50 rounded border border-green-200">
        <CheckCircle className="w-4 h-4 text-green-600 mt-0.5 flex-shrink-0" />
        <div className="text-xs text-gray-700">
          <p className="font-semibold mb-1">✓ Why full resolution works here:</p>
          <p>
            This device has specific Model ({device.model}), Carrier ({device.carrier}),
            ServicePlan ({device.plan}), and Company ({device.company}), so the complete
            11-level resolution algorithm can execute and show exactly where each value comes from.
          </p>
        </div>
      </div>

      <div className="flex items-start gap-2 p-2 bg-blue-50 rounded">
        <Info className="w-4 h-4 text-blue-600 mt-0.5 flex-shrink-0" />
        <p className="text-xs text-gray-700">
          <strong>Device Overrides</strong> (Level 1) have HIGHEST priority and override everything else.
        </p>
      </div>

      <Link to="/dashboard#priority-hierarchy" className="text-xs text-blue-600 hover:text-blue-800 underline">
        View Full Priority Explanation
      </Link>
    </div>
  </CollapsibleContent>
</Collapsible>
```

VARIABLES:
- Replace {device.model}, {device.carrier}, {device.plan}, {device.company} with actual device attributes
- Default to OPEN (defaultOpen={true}) since this is the most detailed view

NOTE: This panel shows device-specific factor values in the hierarchy to make it concrete
```

---

### Enhancement Prompt E8: Add Priority Panel to Device Layer (Customer)
```
Add simplified Priority Context Panel to existing Device Layer Customer page (/layers/device/:deviceId):

LOCATION: Insert after Device Info section, before Parameter Display

NOTE: Customer view uses simple, friendly language

CONTENT:
```tsx
<Collapsible defaultOpen={false}>
  <CollapsibleTrigger className="flex items-center gap-2 p-4 bg-green-50 border border-green-200 rounded-lg w-full hover:bg-green-100 transition-colors">
    <Info className="w-5 h-5 text-green-600" />
    <span className="font-semibold text-green-900">📊 How Device Configuration Works</span>
    <ChevronDown className="w-4 h-4 ml-auto text-green-600" />
  </CollapsibleTrigger>

  <CollapsibleContent className="p-4 bg-white border border-green-200 border-t-0 rounded-b-lg">
    <div className="space-y-3">
      <p className="text-sm text-gray-700">
        You're viewing settings for a specific device in your fleet.
      </p>

      <div className="bg-gray-50 p-3 rounded-md">
        <p className="text-xs font-semibold text-gray-600 mb-2">Configuration Priority:</p>
        <div className="space-y-2 text-xs text-gray-700">
          <div className="flex items-start gap-2 p-2 bg-green-100 rounded">
            <span className="text-green-600 font-bold">→</span>
            <div>
              <p className="font-semibold text-green-900">This Device's Settings ◄ YOU ARE HERE</p>
              <p className="text-gray-700">Custom settings for this specific device</p>
              <p className="text-gray-600 text-xs mt-1">Overrides all other settings</p>
            </div>
          </div>

          <div className="flex items-start gap-2">
            <span className="text-gray-400">2.</span>
            <div>
              <p className="font-semibold">Your Company Settings</p>
              <p className="text-gray-600">Portfolio-wide settings you configured for all devices</p>
            </div>
          </div>

          <div className="flex items-start gap-2">
            <span className="text-gray-400">3.</span>
            <div>
              <p className="font-semibold">System Defaults</p>
              <p className="text-gray-600">Service plan defaults and system-wide settings</p>
            </div>
          </div>
        </div>
      </div>

      <div className="bg-blue-50 p-3 rounded border border-blue-200">
        <p className="text-xs font-semibold text-blue-800 mb-2">ℹ️ Current Value Shows:</p>
        <p className="text-xs text-gray-700 mb-2">
          Each parameter displays its current value and where it comes from:
        </p>
        <ul className="text-xs text-gray-700 space-y-1">
          <li>• <strong>"This Device"</strong> = Custom setting for this device</li>
          <li>• <strong>"Your Company Settings"</strong> = Company-wide configuration</li>
          <li>• <strong>"Service Plan Settings"</strong> = Defaults from your service plan</li>
          <li>• <strong>"System Default"</strong> = Standard system defaults</li>
        </ul>
      </div>

      <div className="flex items-start gap-2 p-2 bg-green-50 rounded">
        <Lightbulb className="w-4 h-4 text-green-600 mt-0.5 flex-shrink-0" />
        <p className="text-xs text-gray-700">
          <strong>💡 Changes here affect ONLY this device.</strong>
        </p>
      </div>

      <Link to="/dashboard#priority-hierarchy" className="text-xs text-blue-600 hover:text-blue-800 underline">
        Learn More About Configuration Priority
      </Link>
    </div>
  </CollapsibleContent>
</Collapsible>
```
```

---

## Enhancement Checklist

### Priority Context Panels
- [ ] E1: Global Layer priority panel
- [ ] E2: Model Layer priority panel
- [ ] E3: Carrier Layer priority panel
- [ ] E4: Service Plan Layer priority panel
- [ ] E5: Company Layer (Admin) priority panel
- [ ] E6: Company Layer (Customer) priority panel (simplified)
- [ ] E7: Device Layer (Admin) priority panel (full resolution)
- [ ] E8: Device Layer (Customer) priority panel (simplified)

### Icons Needed
Import from lucide-react:
```tsx
import {
  Info,
  ChevronDown,
  AlertCircle,
  Lightbulb,
  CheckCircle
} from 'lucide-react';
```

### Common Styling
All panels use:
- Collapsible component (default collapsed except Device Admin)
- Blue color scheme for info (except Device uses green)
- Monospace font for hierarchy lists
- Responsive design
- localStorage for remembering expanded state

---

**END OF LOVABLE IMPLEMENTATION PLAN**