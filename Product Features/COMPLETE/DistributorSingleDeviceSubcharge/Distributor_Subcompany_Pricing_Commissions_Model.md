# WATM Distributor to Subcompany Pricing and Commissions Model

## Overview

The system implements a **multi-tier reseller/distributor model** where parent companies (distributors) can set custom pricing for their subcompanies and automatically earn commissions based on price differences.

---

## 1. Company Hierarchy Structure

**Location:** `plugins/Companies/src/Model/Table/CompaniesTable.php:122-124`

Companies use CakePHP's **TreeBehavior** for hierarchical relationships:
- `parent_company_id` - Links to parent distributor
- `lft` / `rght` - Tree traversal fields
- Account type with `can_upcharge = 1` flag identifies distributors

```php
$this->belongsTo('ParentCompanies', [
    'foreignKey' => 'parent_company_id',
    'className' => 'Companies.Companies',
]);
```

---

## 2. Pricing Model: Three-Layer System

**Location:** `plugins/Devices/src/Model/Table/CompanyServicePlanTiersTable.php:84-190`

### Layer 1: Base Service Plan Tier Price
- Global pricing defined in `service_plan_tiers` table
- Example: 1GB data tier = $19.99/month

### Layer 2: Distributor Price Adjustment
- Stored in `company_service_plan_tiers.price_adjustment`
- Can be positive (markup) or negative (discount)
- Decimal field (16,2) precision

### Layer 3: Subcompany Price Adjustment
- Additional markup/discount on top of distributor pricing
- Same table structure

### Price Calculation Formula:
```
Final Price = Base Tier Price + Parent Adjustment(s) + Company Adjustment
```

### Validation Rules:
```php
// Maximum: $100 above base price
$value <= 100.0 + $basePrice

// Minimum: Cannot go below base price
$value >= -$basePrice
```

**Example Scenario:**
```
Base Price: $19.99
Distributor upcharge: +$5.00
Subcompany upcharge: +$2.50
─────────────────────────────────
Final price to subcompany: $27.49
Distributor earnings: $7.50 (accumulated upcharges)
```

---

## 3. Invoice Generation & Price Resolution

**Location:** `plugins/Billing/src/Model/Table/InvoicesTable.php:210-530`

The `generate()` method walks the company hierarchy to calculate pricing:

```php
// Get parent's custom pricing
$parentCustomPlans = $companyServicePlans->match([
    'service_plan_id' => $hsp->service_plan_id,
    'company_id' => $company->parent_company_id,  // Distributor
]);

// Get subcompany's custom pricing
$customPlans = $companyServicePlans->match([
    'service_plan_id' => $hsp->service_plan_id,
    'company_id' => $company->id,  // Subcompany
]);

// Calculate adjustments
$priceAdjustment = $matchingParentCustomTiers->sumOf('price_adjustment');
$upCharge = $matchingCustomTiers->sumOf('price_adjustment');

// Final price
$price = $tier->tier_price + $priceAdjustment + $upCharge;
```

### Invoice Detail Creation:
```php
$invoice['invoice_details'][] = [
    'service_plan_tier_id' => $tier->id,
    'price' => $price,  // What subcompany pays
    'service_plan_upcharge' => $upCharge,  // Subcompany's markup
    'dual_sim_upcharge' => $dualSimUpcharge,  // Additional dual SIM fees
];
```

---

## 4. Commission Calculation & Payouts

**Location:** `plugins/Billing/src/Model/Table/InvoicesTable.php:445-458`

During invoice generation, distributor earnings are automatically calculated:

```php
if (!empty($totalUpcharges)) {
    foreach ($totalUpcharges as $paymentId => $totalUpcharge) {
        if ($totalUpcharge > 0) {
            $invoice['company_payouts'][] = [
                'company_id' => $company->parent_company_id,  // Distributor
                'company_payment_method_id' => $paymentId,
                'amount' => $totalUpcharge,  // Total earnings
                'is_pending' => 1,  // Awaiting NACHA processing
            ];
        }
    }
}
```

### Commission Components:
1. **Service Plan Upcharges** - Difference between distributor and subcompany pricing
2. **Dual SIM Upcharges** - Additional per-device fees
   ```php
   $dualSimUpcharge = $company->dual_sim_upcharge - $parentCompany->dual_sim_upcharge;
   ```

---

## 5. Earnings Reports

**Location:** `plugins/Companies/src/Model/Table/CompanyEarningsReportsTable.php:248-302`

Distributors can view detailed earnings reports:

```php
// Calculate earnings per device
$servicePlanBasePrice = $invoiceDetail->price - $invoiceDetail->service_plan_upcharge;
$earnings = $invoiceDetail->dual_sim_upcharge + $invoiceDetail->service_plan_upcharge;

$worksheetsData[] = [
    'Company' => $invoice->company->title,
    'Device' => $device->location_name,
    'Base Price' => number_format($servicePlanBasePrice, 2),
    'Charged Price' => number_format($servicePlanPrice, 2),
    'Earnings' => number_format($earnings, 2),  // Distributor commission
];
```

---

## 6. Key Database Tables

| Table | Purpose |
|-------|---------|
| `companies` | Company hierarchy with `parent_company_id`, `dual_sim_upcharge` |
| `account_types` | Identifies which companies `can_upcharge` |
| `company_service_plans` | Links companies to service plans |
| `company_service_plan_tiers` | Stores `price_adjustment` per tier |
| `invoices` | Bills sent to subcompanies |
| `invoice_details` | Line items with `price`, `service_plan_upcharge` breakdown |
| `company_payouts` | Tracks distributor earnings per invoice |

---

## 7. Complete Workflow Example

### Setup:
- **Distributor**: "Acme Corp" (account_type: Distributor)
- **Subcompany**: "Acme East" (parent_company_id = Acme Corp)

### Pricing Configuration:
```
Base Service Plan Tier: 1GB Data = $19.99
Acme Corp adjustment: +$3.00
Acme East adjustment: +$2.00
```

### Monthly Billing Cycle:
1. Device at Acme East uses 1GB data
2. Invoice generated:
   - Base price: $19.99
   - Distributor adjustment: +$3.00
   - Subcompany adjustment: +$2.00
   - **Total charged to Acme East**: $24.99

3. Invoice detail created:
   ```php
   price: $24.99
   service_plan_upcharge: $5.00  // Total markup from base
   ```

4. Company payout created:
   ```php
   company_id: Acme Corp (distributor)
   amount: $5.00  // Commission earned
   is_pending: 1
   ```

5. **Result**:
   - Acme East pays $24.99
   - Acme Corp earns $5.00 commission
   - WATM receives $19.99 base revenue

---

## 8. Key Implementation Files

### Pricing & Service Plans
- `plugins/Devices/src/Model/Table/CompanyServicePlanTiersTable.php` - Price adjustment logic and validation
- `plugins/Devices/src/Controller/Admin/CompanyServicePlansController.php` - Service plan configuration UI

### Billing & Invoicing
- `plugins/Billing/src/Model/Table/InvoicesTable.php` - Invoice generation with hierarchical pricing calculation
- `plugins/Billing/src/Model/Table/InvoiceDetailsTable.php` - Invoice line items

### Company Management
- `plugins/Companies/src/Model/Table/CompaniesTable.php` - Company hierarchy and relationships
- `plugins/Companies/src/Model/Entity/Company.php` - Company entity with `allow_upcharging`, `dual_sim_upcharge`

### Payouts & Commissions
- `plugins/Companies/src/Model/Table/CompanyPayoutsTable.php` - Payout tracking (pending, earmarked, suspended)
- `plugins/Companies/src/Model/Table/CompanyEarningsReportsTable.php` - Earnings report generation
- `plugins/Companies/src/Controller/Admin/CommissionsController.php` - Commission management UI

### Database Migrations
- `20240425135818_AddCanUpchargeToAccountTypes.php` - Account type flag for distributors
- Various migrations in `plugins/Companies/config/Migrations/` for company payout fields

---

## 9. Business Logic Summary

### Distributor Capabilities:
1. Set custom pricing for service plan tiers (markup or discount)
2. Set dual SIM upcharge per device
3. Apply default pricing to all subcompanies or allow individual customization
4. View earnings reports showing all commissions from subcompanies
5. Receive automatic payouts via NACHA/ACH files

### Subcompany Behavior:
1. Inherit parent distributor pricing by default
2. Can override with custom pricing (if `allow_upcharging = 1`)
3. Pay total price (base + distributor markup + own markup)
4. Parent earns commission on the full markup chain

### Multi-Level Hierarchy:
The system supports multiple distributor levels:
```
WATM Base Pricing ($19.99)
  └─ Master Distributor (+$3.00) = $22.99
      └─ Regional Distributor (+$2.00) = $24.99
          └─ Local Reseller (+$1.50) = $26.49
              └─ End Customer pays $26.49
```

Each level earns their respective markup:
- Local Reseller earns: $1.50
- Regional Distributor earns: $2.00
- Master Distributor earns: $3.00

---

## 10. Important Configuration Fields

### Company Entity (`companies` table):
```php
allow_self_billing: boolean       // Can subcompany manage own billing?
allow_upcharging: boolean         // Can company set custom pricing?
dual_sim_upcharge: decimal(16,2)  // Per-device dual SIM fee
parent_company_id: integer|null   // Link to parent distributor
account_type_id: integer          // Links to account_types
```

### Account Types (`account_types` table):
```php
id: integer
title: string                     // e.g., "Distributor", "Reseller"
can_upcharge: boolean             // Permission to set custom pricing
```

### Company Service Plan Tiers (`company_service_plan_tiers` table):
```php
id: integer
company_service_plan_id: integer
service_plan_tier_id: integer
price_adjustment: decimal(16,2)   // Markup/discount amount
```

### Invoice Details (`invoice_details` table):
```php
id: integer
invoice_id: integer
price: decimal(16,2)              // Final charged price
service_plan_upcharge: decimal(16,2)  // Total markup from base
dual_sim_upcharge: decimal(16,2)  // Dual SIM fee difference
```

### Company Payouts (`company_payouts` table):
```php
id: integer
company_id: integer               // Distributor earning the payout
invoice_id: integer               // Related subcompany invoice
amount: decimal(16,2)             // Commission amount
is_pending: boolean               // Awaiting NACHA processing
is_earmarked: boolean             // Reserved for specific purpose
is_suspended: boolean             // Payment on hold
```

---

## 11. How Price Adjustments Are Managed (NOT Hardcoded)

### Answer: Web-Based UI with Bulk Editing

Price adjustments are **NOT hardcoded** - they're managed through a web interface that allows distributors to set pricing dynamically.

**URL:** `/admin/devices/company-service-plans/edit/{id}`

**Files:**
- Controller: `plugins/Devices/src/Controller/Admin/CompanyServicePlansController.php`
- Template: `plugins/Devices/templates/Admin/CompanyServicePlans/edit.php`
- JavaScript: `plugins/Devices/webroot/js/price-adjustment.js`

---

### Management Interface Features

#### 1. Service Plan Selection
Distributors select which service plan to customize for a specific company:

```php
// Controller loads service plan with all tiers
$companyServicePlan = $this->CompanyServicePlans->find()
    ->contain([
        'Companies',
        'CompanyServicePlanTiers',
        'ServicePlans.LastApprovedHistoricalServicePlans.HistoricalServicePlanTiers',
    ])
    ->where(['CompanyServicePlans.id' => $id])
    ->firstOrFail();
```

#### 2. Base Price Calculation
The system automatically calculates the base price for each tier by walking up the company hierarchy:

```php
// From CompanyServicePlansController.php:240-286
$basePricesForPlan = [];

// Start with service plan base tier prices
foreach ($servicePlanTiers as $mainPlanTier) {
    $basePricesForPlan[$mainPlanTier->id] = $mainPlanTier->tier_price;
}

// If company has parent, add parent's adjustments to base
if (isset($company->parent_company_id)) {
    $adminPlanTiers = $this->CompanyServicePlans->find()->where([
        'company_id' => $company->parent_company_id,
        'service_plan_id' => $companyServicePlan->service_plan_id,
        'is_default' => 0
    ])->contain(['CompanyServicePlanTiers'])->first();

    // Add parent adjustments to base
    foreach ($adminPlanTiers->company_service_plan_tiers as $adminPlanTier) {
        $basePricesForPlan[$adminPlanTier->service_plan_tier_id] +=
            $adminPlanTier->price_adjustment;
    }
}
```

#### 3. Individual Tier Pricing
For each data tier (e.g., 1GB, 5GB, 10GB), the UI displays:
- **Usage Limit** - e.g., "1 GB" (read-only)
- **Base Price** - Calculated from parent chain (read-only)
- **Adjusted Price** - Editable field where distributor enters final price

```html
<!-- From edit.php template -->
<div class="form-row">
    <!-- Usage limit (read-only) -->
    <div class="col">
        <input value="1 GB" disabled />
    </div>

    <!-- Base price from parent (read-only) -->
    <div class="col">
        <input value="$19.99" readonly />
    </div>

    <!-- Adjusted price (editable) -->
    <div class="col">
        <input name="company_service_plan_tiers[0].price"
               value="$24.99"
               required />
    </div>
</div>
```

#### 4. Bulk Price Adjustment Widget

**Key Feature:** Distributors can adjust ALL tiers at once using percentage or fixed amount adjustments.

**UI Controls:**
```html
<div id="price-widget">
    <!-- 1. Select all checkbox -->
    <input type="checkbox" id="select-all" />

    <!-- 2. Increase or Decrease -->
    <select id="discount-or-upcharge">
        <option value="u">Increase</option>
        <option value="d">Decrease</option>
    </select>

    <!-- 3. Adjustment amount -->
    <input type="text" id="adjustment" placeholder="5.00" />

    <!-- 4. Amount or Percentage -->
    <select id="amount-or-percent">
        <option value="a">Amount</option>
        <option value="p">Percentage</option>
    </select>

    <!-- 5. Apply button -->
    <button id="apply-btn">Apply</button>
</div>
```

**JavaScript Logic** (`price-adjustment.js:14-36`):
```javascript
$('#apply-btn').click(function () {
    let isUpcharge = $('#discount-or-upcharge').val() === 'u',
        isDiscount = !isUpcharge,
        adjustment = parseFloat($('#adjustment').val()),
        isAmount = $('#amount-or-percent').val() === 'a',
        isPercentage = !isAmount;

    // Loop through selected tiers
    $priceTiers.find('.form-row').each(function () {
        let isSelected = $(this).find('input[type=checkbox]').prop('checked');

        if (isSelected && !isNaN(adjustment)) {
            let priceInput = $(this).find('input[id$="-price"]'),
                newPrice = parseFloat(priceInput.val());

            if (isPercentage) {
                // Apply percentage adjustment
                newPrice += (isDiscount ?
                    (newPrice * adjustment / 100 * -1) :
                    (newPrice * adjustment / 100));
            } else {
                // Apply fixed amount adjustment
                newPrice += (isDiscount ?
                    (adjustment * -1) :
                    adjustment);
            }

            // Update the price field (rounded to 2 decimals)
            priceInput.val((Math.round(newPrice * 100) / 100).toFixed(2));
        }
    });
});
```

---

### Bulk Adjustment Examples

#### Example 1: Add $2.00 to all tiers
1. Check "Select All"
2. Select "Increase"
3. Enter "2.00"
4. Select "Amount"
5. Click "Apply"

**Result:**
```
1GB tier:  $19.99 → $21.99
5GB tier:  $29.99 → $31.99
10GB tier: $39.99 → $41.99
```

#### Example 2: Increase all tiers by 10%
1. Check "Select All"
2. Select "Increase"
3. Enter "10"
4. Select "Percentage"
5. Click "Apply"

**Result:**
```
1GB tier:  $19.99 → $21.99 (19.99 + 10% = 21.99)
5GB tier:  $29.99 → $32.99 (29.99 + 10% = 32.99)
10GB tier: $39.99 → $43.99 (39.99 + 10% = 43.99)
```

#### Example 3: Selective adjustments
1. Check only "1GB" and "5GB" tiers
2. Select "Decrease"
3. Enter "1.50"
4. Select "Amount"
5. Click "Apply"

**Result:**
```
1GB tier:  $19.99 → $18.49
5GB tier:  $29.99 → $28.49
10GB tier: $39.99 → $39.99 (unchanged)
```

---

### How Adjustments Are Saved

#### Backend Processing (`CompanyServicePlansController.php:296-321`):

```php
if ($this->request->is(['patch', 'post', 'put'])) {
    // Patch entity with form data including nested CompanyServicePlanTiers
    $companyServicePlan = $this->CompanyServicePlans->patchEntity(
        $companyServicePlan,
        $this->getRequest()->getData(),
        ['associated' => ['CompanyServicePlanTiers']]
    );

    if ($this->CompanyServicePlans->save($companyServicePlan)) {
        $this->Flash->success('Service Plans Pricing customized.');
        return $this->redirect(['action' => 'index']);
    } else {
        $this->Flash->error('Could not save. Please try again.');
    }
}
```

#### Data Flow:
1. **User enters prices** in UI (e.g., $24.99 for 1GB tier)
2. **Form submits** adjusted prices for each tier
3. **Controller calculates** `price_adjustment` = (Adjusted Price - Base Price)
   ```
   $24.99 - $19.99 = $5.00 price_adjustment
   ```
4. **Table saves** `price_adjustment` to `company_service_plan_tiers` table
5. **Validation runs** to ensure:
   - Adjustment doesn't exceed $100 above base
   - Final price doesn't go below base price

#### Saved Database Record:
```php
[
    'company_service_plan_id' => 123,
    'service_plan_tier_id' => 456,
    'price_adjustment' => 5.00,  // Stored as adjustment, not final price!
    'created' => '2024-10-27 12:00:00',
    'modified' => '2024-10-27 12:00:00',
]
```

---

### Key Takeaways

✅ **NOT Hardcoded** - Prices are managed through web UI
✅ **Dynamic Calculation** - Base prices calculated from parent hierarchy
✅ **Bulk Editing** - Apply changes to multiple tiers simultaneously
✅ **Percentage or Fixed** - Flexible adjustment options
✅ **Validation** - Backend enforces pricing rules
✅ **Database Storage** - Adjustments saved as differential values

The system provides a user-friendly interface that allows distributors to quickly set pricing for their subcompanies without any code changes or manual database edits.

---

## 12. Minimum Charge Per Payment Method ($7.95)

### Overview

The system enforces a **$7.95 minimum charge per payment method** (not per device or per company). This ensures that each payment method on an invoice generates at least $7.95 in charges.

**Location:** `plugins/Billing/src/Model/Entity/Invoice.php:539-560`

---

### How the Minimum Charge Works

#### Rule Application:
```
IF total charges for a payment method < $7.95
AND company.is_surcharge_excluded = false
THEN adjust one line item to $7.95 minimum
```

#### Implementation Logic:

```php
// From Invoice.php:539-560
if (!$this->company->is_surcharge_excluded) {
    $minimumPrice = 7.95;

    foreach ($totalByPaymentMethod as $paymentMethodId => $paymentMethodTotal) {
        if ($paymentMethodTotal < $minimumPrice) {
            // Update the price of 1 of the items for that payment id
            foreach ($lineItems as $index => $lineItem) {
                if ($lineItem['payment_method_id'] == $paymentMethodId &&
                    !$lineItem['delayed_billing']) {

                    // If billing method is Credit Card, apply 3% processing fee
                    if ($lineItem['cc_processing_fee']) {
                        $ccProcessingFee = round($minimumPrice * .03, 2);
                        $lineItems[$index]['price'] = $minimumPrice + $ccProcessingFee;
                        // Total: $7.95 + $0.24 = $8.19
                    } else {
                        $lineItems[$index]['price'] = $minimumPrice;
                    }

                    $lineItems[$index]['minimum_price_fee_applied'] = true;
                    break; // Only adjust ONE line item
                }
            }
        }
    }
}
```

---

### Key Characteristics

#### 1. Per Payment Method, NOT Per Device
- **Incorrect**: "Companies with 1 device pay $7.95"
- **Correct**: "Each payment method must generate at least $7.95 in charges"

#### 2. Examples:

**Example A: Company with 1 device, 1 payment method**
```
Device charges: $3.50
Total for Payment Method 1: $3.50 (below minimum)
→ Adjusted to: $7.95
Final invoice: $7.95
```

**Example B: Company with 3 devices, 1 payment method**
```
Device 1: $2.00
Device 2: $2.50
Device 3: $2.00
Total for Payment Method 1: $6.50 (below minimum)
→ One device charge adjusted to: $7.95 - $4.50 = $3.45 added
Final invoice: $7.95
```

**Example C: Company with 2 devices, 2 payment methods**
```
Payment Method 1:
  Device 1: $3.00 (below minimum)
  → Adjusted to: $7.95

Payment Method 2:
  Device 2: $4.25 (below minimum)
  → Adjusted to: $7.95

Final invoice: $7.95 + $7.95 = $15.90
```

**Example D: Company with 5 devices, 1 payment method**
```
5 devices × $5.00 each = $25.00 (above minimum)
→ No adjustment needed
Final invoice: $25.00
```

---

### Credit Card Processing Fee Impact

When the minimum charge is applied to a **credit card** payment method:

```php
$minimumPrice = 7.95;
$ccProcessingFee = round($minimumPrice * 0.03, 2); // $0.24
$finalCharge = $minimumPrice + $ccProcessingFee;   // $8.19
```

**Result:**
- Base minimum: $7.95
- Processing fee: $0.24 (3%)
- Total charged: **$8.19**

---

### Exclusion Rule: `is_surcharge_excluded`

Companies can be marked as **exempt** from the minimum charge:

**Field:** `companies.is_surcharge_excluded` (boolean)

**UI Label:** "Exclude Device Surcharge" (checkbox in company edit form)

**Location:** `plugins/Companies/templates/Admin/Companies/edit.php:186-187`

**When enabled:**
```php
if (!$this->company->is_surcharge_excluded) {
    // Apply minimum charge logic
}
// If is_surcharge_excluded = true, minimum charge is SKIPPED
```

**Use Cases:**
- Special contract agreements
- Wholesale/bulk customers
- Testing/development companies
- Distributor parent companies

---

### Which Line Item Gets Adjusted?

The system adjusts **the first eligible line item** for that payment method:

```php
foreach ($lineItems as $index => $lineItem) {
    if ($lineItem['payment_method_id'] == $paymentMethodId &&
        !$lineItem['delayed_billing']) {

        // This line item gets adjusted
        $lineItems[$index]['price'] = $minimumPrice;
        $lineItems[$index]['minimum_price_fee_applied'] = true;
        break; // Stop after first match
    }
}
```

**Criteria:**
1. Must belong to the payment method needing adjustment
2. Must NOT have `delayed_billing = true`
3. First match wins

---

### Invoice Display

On the generated invoice, the adjusted line item shows:
- Original charge description (e.g., "Service Plan - 1GB Tier")
- Adjusted price: $7.95 (or $8.19 with CC fee)
- Flag: `minimum_price_fee_applied = true`

**Invoice Detail Fields:**
```php
[
    'payment_method_id' => 123,
    'price' => 7.95,
    'minimum_price_fee_applied' => true,
    'cc_processing_fee' => false,
    'delayed_billing' => false,
]
```

---

### Interaction with Credits

Credits are applied **AFTER** the minimum charge adjustment:

**Processing Order:**
1. Calculate all line item charges
2. Group by payment method
3. **Apply minimum charge** ($7.95) if needed
4. Apply available credits to reduce charges
5. Calculate final amount due

**Example:**
```
Device charge: $3.00
→ Adjusted to minimum: $7.95
→ Credit available: $5.00
→ Credit applied: -$5.00
→ Final amount due: $2.95
```

---

### Documentation References

**Detailed Billing Rules:**
- See: `/watm/claude/apw_concepts/billing_caveats.md:5-13`
- Section: "Minimum Charges → Per Payment Method Minimum"

**Implementation:**
- Entity: `plugins/Billing/src/Model/Entity/Invoice.php:539-560`
- Company Field: `plugins/Companies/src/Model/Entity/Company.php:48`
- UI Control: `plugins/Companies/templates/Admin/Companies/edit.php:186-187`

---

### Summary

✅ **Per Payment Method** - Not per device or per company
✅ **$7.95 Minimum** - Ensures each payment method generates minimum revenue
✅ **Credit Card Fee** - 3% applied to adjusted minimum ($8.19 total)
✅ **Exclusion Available** - `is_surcharge_excluded` flag bypasses rule
✅ **Single Line Item** - Only one item per payment method gets adjusted
✅ **After Credits** - Credits applied after minimum adjustment

This minimum charge rule ensures that processing costs are covered even for low-usage customers, while maintaining flexibility through the exclusion flag for special business arrangements.

---

This model creates a flexible multi-tier reseller system where distributors automatically earn commissions based on pricing differences, with full transparency through detailed invoicing and earnings reports.
