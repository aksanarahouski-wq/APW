# WATM-1981: Annual Commissions Report — Technical Analysis

**Ticket:** WATM-1981 — Annual Commissions Report: Add Tax ID and Update Addresses
**Date:** 2026-04-03
**Status:** Draft
**Assignee:** aaron.diefes
**Creator:** Laura Perry
**Parent:** WATM-2020: W9 document enhancements

## Client Request

> Update the Annual Commissions report:
> - Add the distributor's Tax ID as the first column of the report
> - Separate the address into individual columns for Address, City, State, Zip

## Current Implementation

### Annual Commissions Report

**Controller:** `plugins/Companies/src/Controller/Admin/CommissionsController.php`
- `annualReport()` (lines 395-410) — HTML view
- `exportAnnualReport()` (lines 501-523) — CSV export
- `getPayoutsForYear()` (lines 529-575) — data query

**HTML View** (`plugins/Companies/templates/Admin/Commissions/annual_report.php`):

| Column | Source |
|--------|--------|
| Company | `$company->title` |
| Unpaid Earnings | `$company->total_payout` |
| Paid Out Earnings | `$company->total_paid` |
| Consented to Electronic Tax Documents | `$company->has_electronically_consented` |

**CSV Export** (different columns from HTML view):

| Column | Source |
|--------|--------|
| Company | `title` |
| Earnings | `total_paid` |
| Address | `readable_address` (virtual property — single concatenated string) |

**Note:** The HTML view and CSV export show **different column sets**. The HTML view does not include address at all. The CSV includes address but not unpaid earnings or consent status.

### Address Storage (Separate Columns Already Exist)

The `companies` table already stores addresses in separate columns:

| Column | Type | Example |
|--------|------|---------|
| `address_1` | varchar(255) | 123 Main Street |
| `address_2` | varchar(255) | Suite 100 |
| `city` | varchar(255) | Denver |
| `state_id` | int (FK to `states`) | → "CO" via `states.abbreviation` |
| `zip_code` | varchar(255) | 80202 |

The `readable_address` virtual property on the Company entity (lines 272-302) concatenates these into a single comma-separated string. The CSV export uses this concatenated form today.

**Splitting the address into separate columns is trivial** — the data is already stored separately. We just need to change the CSV `$extract` array from `readable_address` to the individual fields.

### Tax ID Storage (Problem Area)

**Tax ID is NOT stored in the database.** Here's how it currently works:

1. Distributors submit W9 forms via **Adobe Sign**
2. The signed W9 is stored as a PDF in `adobe_agreements` (linked to company via `companies.distributor_agreement_id`)
3. When generating 1099 tax documents, the system calls the **Adobe Sign API** to extract form data from the signed W9
4. Tax ID (SSN or EIN) is extracted at that point and written directly to the 1099 PDF
5. **The Tax ID is never persisted to the database**

Extraction logic in `CompanyTaxDocumentsTable.php` (lines 193-200):
```php
$w9Information = $agreementFormData['form_data'][0];
if (!empty($w9Information['SSN1'])) {
    $recipientTin = implode('-', [$w9Information['SSN1'], $w9Information['SSN2'], $w9Information['SSN3']]);
} else {
    $recipientTin = implode('-', [$w9Information['EIN1'], $w9Information['EIN2']]);
}
```

### To Add Tax ID to the Report, We Have Two Paths:

#### Path A: Fetch from Adobe Sign API at Report Time

- Call Adobe Sign API for each distributor's W9 agreement when generating the report
- Extract the Tax ID from the form data
- Display in the report without persisting

**Pros:** No schema change, Tax ID stays in Adobe Sign as source of truth
**Cons:** Slow (API call per distributor), dependent on Adobe Sign availability, rate limits, API costs. Not practical for a report that could have dozens of distributors.

#### Path B: Store Tax ID in the Database

- Add a `tax_id` column to the `companies` table (encrypted via `TwoWayCryptedType`)
- Populate it when the W9 is processed/signed (extend the existing Adobe Sign webhook flow)
- Backfill existing distributors from their signed W9 data
- Query directly in the report

**Pros:** Fast report generation, no API dependency at report time, reusable for other features
**Cons:** Sensitive data now stored locally (needs encryption), migration + backfill effort, must keep in sync if W9 is re-submitted

## Feasibility Summary

| Request | Feasible? | Effort | Notes |
|---------|-----------|--------|-------|
| Split address into separate columns | Yes | Low | Data already stored separately. Change CSV extract config. |
| Add Tax ID column | Conditionally | Medium | Tax ID is not currently stored in DB. Requires either API calls at report time (slow) or a new encrypted DB column + backfill. |

## Key Files

| File | Purpose |
|------|---------|
| `plugins/Companies/src/Controller/Admin/CommissionsController.php` | Report controller (annualReport, exportAnnualReport) |
| `plugins/Companies/templates/Admin/Commissions/annual_report.php` | HTML report template |
| `plugins/Companies/src/Model/Entity/Company.php` | Company entity (address fields, readable_address virtual property) |
| `plugins/Companies/src/Model/Table/CompanyTaxDocumentsTable.php` | Tax ID extraction logic (lines 193-200) |
| `plugins/Companies/src/Service/TenNinetyNinePdfService.php` | 1099 PDF generation |
| `plugins/Companies/src/Job/GenerateTaxDocumentsJob.php` | Tax document generation job |
| `plugins/AdobeSign/src/Api/AdobeSignApi.php` | Adobe Sign API (getAgreementFormData) |

## Open Questions for Client

1. **Which report format are you referring to?** The HTML view and CSV export currently show different columns. Should Tax ID and split address appear on both, or just the CSV export?
2. **Tax ID source:** Tax IDs are currently only available through signed W9 forms in Adobe Sign. Should we add a Tax ID field to the company record in the system so it's readily available for reports? This would require storing it (encrypted) in our database.
3. **What about distributors without a W9 on file?** Should the Tax ID column show blank, or should we flag those companies as missing W9 data?
4. **Address Line 2:** Should we include Address Line 2 as a separate column, or only Address, City, State, Zip (4 columns)?
5. **Does this change relate to WATM-2020 (W9 document enhancements)?** If W9 enhancements will include persisting Tax ID to the company record, we should coordinate these two tickets to avoid double work.
