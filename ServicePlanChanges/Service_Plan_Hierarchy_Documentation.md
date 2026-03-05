# Service Plan Hierarchy Documentation

**Document Date:** 2025-11-10
**System:** WATM (Wireless Access and Telemetry Management)

## Overview

The WATM system uses a three-tier service plan hierarchy that enables distributors to purchase services at discounted rates and resell to their sub-customers at higher prices, earning commission on the difference.

---

## Three-Tier Hierarchy

```
┌─────────────────────────────┐
│   Base Service Plan         │  ← Global pricing set by WATM
│   (e.g., ATM: $4.95)        │
└──────────────┬──────────────┘
               │
               │ Discount applied
               ↓
┌─────────────────────────────┐
│ Distributor Custom Service  │  ← Discounted rate for distributor
│ Plan (e.g., $3.75)          │     (Lower than base)
└──────────────┬──────────────┘
               │
               │ Markup applied
               ↓
┌─────────────────────────────┐
│ Sub-Customer Service Plan   │  ← Price charged to end customer
│ (e.g., $6.99)               │     (Higher than distributor cost)
└─────────────────────────────┘
```

---

## 1. Base Service Plan

### Definition
The foundational service plan with global pricing set by WATM for all customers.

### Characteristics
- **Global pricing**: Single price applies to all standard customers
- **No discounts**: Represents standard retail pricing
- **Parent to custom plans**: All custom service plans are derived from base plans

### Current Base Service Plans
- **ATM Plan**: $4.95 (current global price for most devices)
- **Tier 1, 2, 3 Plans**: Usage-based pricing tiers
- Other service plans as defined in system

### Example
```
Service Plan: ATM
Base Price: $4.95
Applies to: All standard customers without custom pricing
```

---

## 2. Distributor Custom Service Plan

### Definition
A discounted version of the base service plan, created specifically for distributors who resell services to their own customers.

### Purpose
- Provide **wholesale pricing** to distributors
- Enable distributors to be **competitive** in their markets
- Create **profit margin** for distributor business model

### How Discounts Are Applied
Distributors receive custom pricing based on:
- **Volume commitments**
- **Business relationship with WATM**
- **Negotiated discount rates**

### Characteristics
- **Lower than base price**: Always discounted from the base service plan
- **Distributor-specific**: Each distributor can have different discount levels
- **Inherited from base**: Tied to parent base service plan via service plan ID
- **Cost to distributor**: This is what the distributor pays WATM

### Examples from Production

#### Example 1: Tyler's Custom ATM Plan
```
Base Service Plan: ATM
Base Price: $4.95

Tyler's Custom Service Plan: ATM (Custom)
Tyler's Price: $3.75
Discount: $1.20 (24% off base price)
```

#### Example 2: Chord Base Custom ATM Plan
```
Base Service Plan: ATM
Base Price: $4.95

Chord Base Custom Service Plan: ATM (Custom)
Chord Base Price: $3.50
Discount: $1.45 (29% off base price)
```

### How Distributors Create Custom Plans
1. Distributor logs into WATM portal
2. Navigates to **Manage Customer Service Plans**
3. Creates custom service plan based on their negotiated discount
4. System displays their **discounted base price** (e.g., $3.75)
5. Distributor sees this as their **cost** for providing service

---

## 3. Sub-Customer Service Plan

### Definition
The pricing that a distributor charges to their own customers (sub-customers) for services.

### Purpose
- Generate **revenue** for the distributor
- Create **profit margin** between distributor's cost and sub-customer's price
- Allow distributors to set **competitive market pricing**

### Characteristics
- **Higher than distributor cost**: Must be greater than what distributor pays
- **Set by distributor**: Distributor has full control over sub-customer pricing
- **Flexible**: Can vary by sub-customer based on distributor's business strategy
- **Revenue to distributor**: This is what the sub-customer pays the distributor

### How Sub-Customer Pricing Works

#### Scenario 1: Standard Markup
```
Tyler's Cost (Distributor Custom Plan): $3.75
Tyler charges sub-customer: $6.99
Tyler's profit per device: $3.24

Calculation:
$6.99 (sub-customer pays) - $3.75 (Tyler pays WATM) = $3.24 profit
```

#### Scenario 2: Higher Markup
```
Tyler's Cost (Distributor Custom Plan): $3.75
Tyler charges sub-customer: $9.00
Tyler's profit per device: $5.25

Calculation:
$9.00 (sub-customer pays) - $3.75 (Tyler pays WATM) = $5.25 profit
```

#### Scenario 3: Minimal Markup (Pass-through pricing)
```
Tyler's Cost (Distributor Custom Plan): $3.75
Tyler charges sub-customer: $5.99
Tyler's profit per device: $2.24

Calculation:
$5.99 (sub-customer pays) - $3.75 (Tyler pays WATM) = $2.24 profit
```

---

## Commission/Profit Calculation Model

### Basic Formula
```
Distributor Profit = Sub-Customer Price - Distributor Cost
```

### Where:
- **Sub-Customer Price**: Amount the distributor charges their customer
- **Distributor Cost**: Distributor's custom service plan price (discounted from base)

### Expected Behavior
Distributors set sub-customer pricing with the assumption that their commission is calculated based on their known discount rate.

### Example Calculation
```
Base Service Plan: $4.95
Distributor Cost: $3.75 (discount of $1.20)
Sub-Customer Price: $7.00

Expected Commission: $7.00 - $3.75 = $3.25
```

---

## The $4.25 Minimum Charge Issue

### Problem
The WATM system has a **$4.25 minimum charge** due to ACH (Automated Clearing House) processing costs. This affects distributors with heavy discounts.

### Impact on Commission Calculations

#### When Distributor Cost < $4.25
If a distributor's custom price is below $4.25, but the sub-customer only has **one device**, the system charges the distributor $4.25 instead of their custom price.

#### Example: Commission Calculation Error
```
Tyler's Custom Price: $3.75
Sub-Customer Price: $7.00
Expected Commission: $7.00 - $3.75 = $3.25

ACTUAL Charge to Tyler: $4.25 (minimum for single device)
ACTUAL Commission: $7.00 - $4.25 = $2.75

Loss: $0.50 per device
```

### When This Applies
- Sub-customer has **only one device** on a payment method
- Distributor's custom price is **below $4.25**
- Payment is processed via **ACH/bank account**

### Why Distributors Aren't Aware
- The $4.25 minimum is **baked into the system silently**
- No messaging informs distributors about this exception
- Distributors assume their custom price always applies
- Results in **unexpected lower commissions**

---

## Parent-Child Relationship: Base to Custom Plans

### How Custom Plans Are Created

#### 1. Technical Relationship
```
Base Service Plan
  ├── ID: 123
  ├── Name: "ATM"
  ├── Base Price: $4.95
  │
  ├─► Custom Service Plan (Tyler)
  │     ├── ID: 456
  │     ├── Parent ID: 123
  │     ├── Custom Price: $3.75
  │
  ├─► Custom Service Plan (Chord Base)
  │     ├── ID: 457
  │     ├── Parent ID: 123
  │     ├── Custom Price: $3.50
  │
  └─► Custom Service Plan (Other Distributor)
        ├── ID: 458
        ├── Parent ID: 123
        ├── Custom Price: $4.25
```

### Key Technical Points

#### Parent ID Reference
- Custom service plans store a **parent service plan ID**
- This links the custom plan to the base plan
- Allows system to understand the relationship and inheritance

#### Price Adjustment
- Custom plans store a **price adjustment** or **discount amount**
- Can be stored as absolute price ($3.75) or percentage discount (24% off)
- System applies this adjustment to calculate final price

#### Inheritance of Attributes
- Custom plans inherit most attributes from base plan:
  - Service plan name
  - Device group compatibility
  - Usage limits and tiers
  - Additional features and restrictions

---

## Critical Bug: Service Plan ID Changes

### The Problem
**When a base service plan is edited, the system creates a new service plan ID, orphaning all custom service plans.**

### Bug Behavior
```
BEFORE EDIT:
Base Service Plan ID: 123
  ├─► Custom Plan (Tyler) - Parent ID: 123
  ├─► Custom Plan (Chord Base) - Parent ID: 123
  └─► Custom Plan (Other) - Parent ID: 123

AFTER EDITING BASE PLAN:
Base Service Plan ID: 124 (NEW ID!)
  └─► (No children - starts fresh)

Orphaned Custom Plans:
  ├─► Custom Plan (Tyler) - Parent ID: 123 ← BROKEN LINK
  ├─► Custom Plan (Chord Base) - Parent ID: 123 ← BROKEN LINK
  └─► Custom Plan (Other) - Parent ID: 123 ← BROKEN LINK
```

### Why This Happens
- Related to the **approval system** for service plan changes
- When a plan is edited and saved, a new version is created
- New version gets a new ID
- Old ID is deprecated
- Child custom plans still reference the old ID

### Impact
- **All custom service plans break**
- Distributors lose their custom pricing
- System may fall back to base pricing (charging distributors more)
- Requires **manual database patching** to fix parent ID references
- Hours of work to repair relationships

### Previous Incidents
#### Tier 3 Micro-Tier Addition
```
Action: Added micro-tiers to Tier 3 service plan
Result: All custom Tier 3 plans orphaned
Fix: Manual database updates to repair parent IDs
Time: Hours of manual work
```

### Current Workaround
- **Avoid editing base service plans**
- WATM has only edited service plans 1-2 times in 2.5-3 years
- When edits are necessary, manual database patching required
- Not sustainable for upcoming carrier/model pricing changes

### Required Fix
Before implementing carrier/model pricing changes:
1. Service plan edits must **NOT create new IDs**
2. Parent-child relationships must be **preserved**
3. Custom plans must remain linked to base plans
4. Updates to base plans should **cascade** to custom plans appropriately

---

## Creating and Managing Custom Service Plans

### From Distributor Perspective

#### Step 1: Access Custom Service Plans
1. Log into WATM portal as distributor
2. Navigate to **Service Plans** or **Manage Customer Service Plans**
3. View available base service plans

#### Step 2: Create Custom Plan
1. Select base service plan (e.g., ATM)
2. System displays base price (e.g., $4.95)
3. System applies distributor's discount
4. Displays custom price (e.g., $3.75)
5. Distributor sees this price as their **cost**

#### Step 3: Assign to Sub-Customers
1. Create or edit sub-customer
2. Select service plan for sub-customer
3. Set sub-customer's **billable price** (e.g., $6.99)
4. System calculates expected commission

### From WATM Admin Perspective

#### Granting Distributor Discounts
1. Negotiate pricing with distributor
2. Create custom service plan for distributor account
3. Set discount rate or custom price
4. Distributor can now see and use custom pricing

#### Managing Base Service Plans
1. Edit base service plans from admin interface
2. **CAUTION**: Currently breaks custom plans (bug)
3. Changes affect all customers using that base plan
4. Custom plans may need manual adjustment

---

## Use Cases and Examples

### Use Case 1: Standard Distributor Operation

#### Setup
- Distributor: Tyler
- Base ATM Plan: $4.95
- Tyler's Custom ATM Plan: $3.75
- Sub-Customer: Test Company

#### Monthly Billing Flow
```
1. Test Company has 10 devices on ATM plan
2. WATM bills Tyler: 10 × $3.75 = $37.50
3. Tyler bills Test Company: 10 × $6.99 = $69.90
4. Tyler's profit: $69.90 - $37.50 = $32.40
```

### Use Case 2: Single Device Minimum Charge Impact

#### Setup
- Distributor: Tyler
- Tyler's Custom ATM Plan: $3.75
- Sub-Customer: Single Device Customer
- Sub-Customer has 1 device

#### Monthly Billing Flow
```
1. Single Device Customer has 1 device on ATM plan
2. Tyler charges customer: 1 × $6.99 = $6.99
3. Tyler expects to pay WATM: 1 × $3.75 = $3.75
4. Tyler expects profit: $6.99 - $3.75 = $3.24

ACTUAL BILLING:
1. WATM charges Tyler: 1 × $4.25 = $4.25 (minimum)
2. Tyler's actual profit: $6.99 - $4.25 = $2.74
3. Unexpected loss: $0.50 per device
```

#### Problem
Tyler doesn't know about the $4.25 minimum and may not adjust sub-customer pricing accordingly.

### Use Case 3: Multiple Sub-Customers with Different Pricing

#### Setup
- Distributor: Tyler
- Tyler's Custom ATM Plan: $3.75

#### Sub-Customer Pricing Strategy
```
Sub-Customer A: Premium customer
  - Devices: 50
  - Price per device: $5.99
  - Tyler's revenue: 50 × $5.99 = $299.50
  - Tyler's cost: 50 × $3.75 = $187.50
  - Tyler's profit: $112.00

Sub-Customer B: Volume customer
  - Devices: 200
  - Price per device: $4.99 (volume discount)
  - Tyler's revenue: 200 × $4.99 = $998.00
  - Tyler's cost: 200 × $3.75 = $750.00
  - Tyler's profit: $248.00

Sub-Customer C: Small customer
  - Devices: 5
  - Price per device: $7.99
  - Tyler's revenue: 5 × $7.99 = $39.95
  - Tyler's cost: 5 × $3.75 = $18.75
  - Tyler's profit: $21.20
```

### Use Case 4: Carrier/Model Pricing (Future State)

#### Setup
- Base ATM Plan with carrier/model pricing:
  - Verizon i-22: $4.95
  - AT&T i-22: $4.95
  - T-Mobile i-22: $3.50

#### Tyler's Custom Plan (After Implementation)
```
Tyler's Custom ATM Plan:
  - Verizon i-22: $3.75 (discount from $4.95)
  - AT&T i-22: $3.75 (discount from $4.95)
  - T-Mobile i-22: $2.50 (discount from $3.50)

Tyler's Sub-Customer Pricing:
  - Verizon i-22: $6.99
  - AT&T i-22: $6.99
  - T-Mobile i-22: $5.99
```

#### Monthly Billing (Mixed Devices)
```
Sub-Customer has:
  - 10 Verizon i-22 devices
  - 5 AT&T i-22 devices
  - 3 T-Mobile i-22 devices

Tyler's Cost:
  - Verizon: 10 × $3.75 = $37.50
  - AT&T: 5 × $3.75 = $18.75
  - T-Mobile: 3 × $2.50 = $7.50
  - Total: $63.75

Tyler's Revenue:
  - Verizon: 10 × $6.99 = $69.90
  - AT&T: 5 × $6.99 = $34.95
  - T-Mobile: 3 × $5.99 = $17.97
  - Total: $122.82

Tyler's Profit: $122.82 - $63.75 = $59.07
```

---

## UI/Portal Pages

### For Distributors

#### Manage Customer Service Plans Page
**Purpose**: View and manage custom service plan pricing

**Displays:**
- Base service plan name (e.g., "ATM")
- Distributor's custom price (e.g., $3.75)
- Discount compared to base price
- List of sub-customers using this plan

**Actions:**
- View custom pricing details
- Assign service plans to sub-customers
- Adjust sub-customer billable prices

**Current Issue:**
- No messaging about $4.25 minimum charge
- Distributors don't see exceptions to their custom pricing
- Need to add informational message about single-device minimum

#### Create Sub-Customer Page
**Purpose**: Set up new sub-customer accounts

**Current Fields:**
- Company information
- Contact details
- Service plan selection
- Billing information

**Suggested Enhancement:**
- Add field: "Single device charge override"
- Add message: "Minimum charge for single device is $4.25"
- Allow distributor to set custom pricing for single-device scenario

### For WATM Admins

#### Service Plans Management Page
**Purpose**: Create and manage base service plans

**Displays:**
- All base service plans
- Pricing tiers and attributes
- Usage limits and conditions
- Device group compatibility

**Actions:**
- Create new service plans
- Edit existing plans (CAUTION: bug)
- View custom plans derived from base

**Critical Warning:**
- Editing a base service plan creates new ID
- Orphans all custom service plans
- Requires manual database fixes

#### Create Service Plan Page
**Purpose**: Define new base service plan

**Current Fields:**
- Service plan name
- Device group names
- Usage limits
- Price tiers

**Future Enhancement (Carrier/Model Pricing):**
- Carrier selection checkboxes
- Model selection
- Price per carrier/model combination
- Optional granular pricing (backwards compatible)

---

## Business Logic and Rules

### Rule 1: Minimum Charge
```
IF sub_customer.device_count == 1 AND payment_method == "bank_account"
THEN minimum_charge = $4.25
ELSE minimum_charge = distributor_custom_price
```

### Rule 2: Distributor Subcharge ($7.95)
```
IF company.is_distributor == true
AND company.sub_customers.count >= 1
THEN apply_subcharge = $7.95
```

### Rule 3: Commission Calculation
```
commission = sub_customer_price - MAX(distributor_cost, minimum_charge)
```

### Rule 4: Service Plan Inheritance
```
custom_plan.base_price = base_plan.base_price
custom_plan.attributes = base_plan.attributes
custom_plan.price = base_plan.price - discount_amount
```

---

## Data Model

### Base Service Plan Table
```sql
service_plans
  - id (Primary Key)
  - name (e.g., "ATM")
  - base_price (e.g., 4.95)
  - device_group_id
  - usage_limit
  - created_at
  - updated_at
  - status (active/inactive)
```

### Custom Service Plan Table
```sql
custom_service_plans
  - id (Primary Key)
  - parent_service_plan_id (Foreign Key → service_plans.id)
  - company_id (Foreign Key → companies.id)
  - custom_price (e.g., 3.75)
  - discount_amount (e.g., 1.20)
  - discount_percentage (e.g., 0.24)
  - created_at
  - updated_at
  - status (active/inactive)
```

### Service Plan Price Tiers Table (Future)
```sql
service_plan_price_tiers
  - id (Primary Key)
  - service_plan_id (Foreign Key)
  - carrier_id (Foreign Key → carriers.id)
  - device_model_id (Foreign Key → device_models.id)
  - price
  - effective_date
  - end_date
```

---

## Key Takeaways

### For Distributors
1. Custom service plans provide **wholesale pricing**
2. Markup sub-customer prices to create **profit margin**
3. Watch for **$4.25 minimum** on single-device customers
4. Commission is the difference between what you charge and what you pay

### For WATM Admins
1. **Do not edit base service plans** (creates orphaned custom plans)
2. Carrier/model pricing requires **backwards compatibility**
3. Custom plans must mirror base plan structure
4. Fix the service plan ID bug before major changes

### For Developers
1. Preserve **parent-child relationships** when editing service plans
2. Implement **optional carrier/model pricing** (backwards compatible)
3. Add **messaging** about minimum charges and exceptions
4. Ensure billing logic applies correct pricing per device/carrier/model

---

## Future Enhancements

### Phase 1: Fix Critical Bug
- Resolve service plan ID orphaning issue
- Ensure edits don't create new IDs
- Test extensively with custom plans

### Phase 2: Add Messaging
- Inform distributors about $4.25 minimum
- Add warnings on custom service plan pages
- Display expected vs. actual commission calculations

### Phase 3: Carrier/Model Pricing
- Implement optional carrier/model attributes
- Add granular pricing per combination
- Migrate existing custom plans
- Update UI for multi-dimensional pricing

### Phase 4: Dynamic Pricing Rules
- Allow distributors to set rules for single-device pricing
- Automated markup calculations
- Commission reporting and analytics
- Alerts for pricing below minimums

---

## References

- Meeting Notes: `Service_Plan_change_meeting.md`
- Requirements: `ATM_Service_Plan_Requirements.md`
- Billing Test Cases: `BillingCyclesTemplateTestCases.md` (referenced in CLAUDE.md)
- WATM Documentation: `/Users/aksana/Documents/Projects/WATM/CLAUDE.md`
