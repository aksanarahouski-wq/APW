# WATM-1981: Annual Commissions Report — Add Tax ID and Split Address

**Ticket:** WATM-1981
**Date:** 2026-04-07
**Status:** Requirements In Progress
**Requested By:** Laura Perry / APW Team
**Parent Epic:** WATM-2020 (W9 Document Enhancements)

---

## Decision Summary

The client selected **Option B: Store Tax ID in the database** (encrypted). Tax ID will be persisted to the company record when the W9 is signed, making it instantly available for reports without Adobe Sign API calls at report time.

### Client Answers to Clarifying Questions

| Question | Answer |
|----------|--------|
| Should Tax ID and split address appear on the CSV export only, or also on the HTML report page? | **Both** — should also show on the report page |
| For distributors without a W9 — blank or flag? | **Blank** — just start collecting going forward, leave empty for companies without a Tax ID |
| Include Address Line 2 as its own column? | **Yes** — separate column, 5 address columns total (Address 1, Address 2, City, State, Zip) |
| Coordinate with WATM-2020 (W9 enhancements)? | **Not yet answered** |

---

## Outstanding Questions

These need answers before development begins:

~~**Q-1: Address Line 2** — Resolved: Yes, include Address Line 2 as its own column (5 address columns total).~~

**Q-2: WATM-2020 Coordination**
Since this ticket is a child of the W9 enhancements epic (WATM-2020), should the Tax ID database storage work be coordinated with other W9 changes? If WATM-2020 already plans to persist Tax ID, we should avoid duplicating that work.

---

## Requirements

### 1. Tax ID Storage (New Encrypted Column)

**1.1** Add a `tax_id` column to the `companies` table, encrypted using the existing `TwoWayCryptedType` (reversible encryption, since we need to display the value).

**1.2** When a distributor signs a W9 via Adobe Sign, the system shall extract the Tax ID (SSN or EIN) from the signed W9 form data and persist it to the company's `tax_id` field.

**1.3** If a distributor re-submits a W9, the `tax_id` field shall be updated with the new value from the latest signed W9.

**1.4** No backfill of existing W9 data. Distributors who signed a W9 before this enhancement will not have a Tax ID populated. Tax ID collection begins with new W9 submissions going forward only.

### 2. Annual Commissions Report — CSV Export

**2.1** Add **Tax ID** as the first column of the CSV export.

**2.2** Replace the single `Address` column (currently `readable_address` — a concatenated string) with separate columns:
- Address 1 (address_1)
- Address 2 (address_2)
- City
- State (state abbreviation)
- Zip

**2.3** For distributors **without a Tax ID** (no W9 on file or Tax ID not yet collected), the Tax ID column shall be **blank**.

**2.4** Updated CSV column order:
| # | Column | Source |
|---|--------|--------|
| 1 | Tax ID | `company.tax_id` (decrypted) or blank |
| 2 | Company | `company.title` |
| 3 | Earnings | `company.total_paid` |
| 4 | Address 1 | `company.address_1` |
| 5 | Address 2 | `company.address_2` |
| 6 | City | `company.city` |
| 7 | State | `states.abbreviation` |
| 8 | Zip | `company.zip_code` |

### 3. Annual Commissions Report — HTML Report Page

**3.1** Add **Tax ID** as the first column of the HTML report table.

**3.2** Add an **Address** column displaying the full formatted address (single column, not split). Use the existing `readable_address` virtual property which concatenates address_1, address_2, city, state, and zip.

**3.3** For distributors without a Tax ID, the Tax ID column shall be **blank** (same behavior as CSV).

**3.4** Updated HTML report columns:
| # | Column | Source |
|---|--------|--------|
| 1 | Tax ID | `company.tax_id` (decrypted) or blank |
| 2 | Company | `company.title` |
| 3 | Address | `company.readable_address` (full formatted address) |
| 4 | Unpaid Earnings | `company.total_payout` |
| 5 | Paid Out Earnings | `company.total_paid` |
| 6 | Consented to Electronic Tax Documents | `company.has_electronically_consented` |

### 4. Data Query Updates

**4.1** The `getPayoutsForYear()` method must be updated to include the `tax_id` field and the individual address fields (`address_1`, `address_2`, `city`, `state_id` with state association, `zip_code`) in its query.

**4.2** The state abbreviation must be resolved via the `states` table join (already available in the company model associations).

---

## Out of Scope

- Tax ID collection or entry via admin UI (Tax ID is populated only from signed W9 forms)
- Changes to the 1099 generation process (it already extracts Tax ID from Adobe Sign independently)
- Changes to other reports beyond the Annual Commissions report
- Primary status or any device-related changes

---

## Files Impacted

| File | Change |
|------|--------|
| `config/Migrations/` | New migration: add `tax_id` column to `companies` table |
| `plugins/Companies/src/Model/Entity/Company.php` | Add `tax_id` to hidden fields (encryption handling) |
| `plugins/Companies/src/Model/Table/CompaniesTable.php` | Register `tax_id` as `TwoWayCryptedType` |
| `plugins/Companies/src/Controller/Admin/CommissionsController.php` | Update `annualReport()` and `exportAnnualReport()` to include Tax ID and split address; update `getPayoutsForYear()` query |
| `plugins/Companies/templates/Admin/Commissions/annual_report.php` | Add Tax ID column and full Address column |
| `plugins/Companies/src/Model/Table/CompanyTaxDocumentsTable.php` | After W9 signing, persist extracted Tax ID to company record |
| `plugins/Companies/src/Job/GenerateTaxDocumentsJob.php` | Potentially update to also save Tax ID during 1099 generation |

---

## Acceptance Criteria

1. The `companies` table has a new `tax_id` column, encrypted with `TwoWayCryptedType`
2. When a distributor signs a W9 via Adobe Sign, the Tax ID is automatically extracted and saved to the company record
3. Existing distributors (W9 signed before this enhancement) will have a blank Tax ID — no backfill
4. The Annual Commissions CSV export shows Tax ID as the first column and address split into separate columns (Address 1, Address 2, City, State, Zip)
5. The Annual Commissions HTML report page shows Tax ID as the first column and address as a single full-address column
6. Distributors without a Tax ID show a blank Tax ID column on both CSV and HTML report
7. Tax ID values are decrypted for display but stored encrypted at rest
