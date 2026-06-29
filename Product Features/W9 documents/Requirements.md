# WATM-1501: Resend W9 Documents Yearly — Requirements

**Ticket:** WATM-1501
**Date:** 2026-04-07
**Status:** Requirements In Progress
**Requested By:** Jon Mack / Laura Perry / APW Team
**Parent Epic:** WATM-2020 (W9 Document Enhancements)
**Related:** WATM-1491 (duplicate — consolidate into this ticket)

---

## Decision Summary

The client needs the ability to request a fresh W9 signature from distributors on a per-distributor basis via a manual admin action. This creates a new Adobe Sign envelope (not a reminder on the existing one). Only the most recent signed W9 is retained.

### Client Answers — Confluence Comments (April 6, 2026)

| Question | Answer |
|----------|--------|
| W9 only, or full 3-document package? | **All 3 documents** (W9 + 1099 Consent + Distributor Agreement) — per meeting; it's one envelope transaction regardless, simpler to reuse existing flow |
| Adobe Sign envelope costs acceptable? | Deflected — "how would we know when a distributor's information has changed?" (cost not a concern; they pay for 500 transactions and are well below that) |
| Individual vs. bulk send? | **Per-distributor is fine** |
| Manual or automatic? | **Manual button** — "depends on whether we are going to send to everyone every year or not" |
| Keep previous W9 history? | **Only retain most recent** |

### Additional Context from Meeting (Meeting1.md)

- **Envelope contents — resolved via meeting:** Confluence comment said W9 + 1099 consent only, but during the meeting Devon concluded "I would just keep it all three if that's gonna be easier" since it's one envelope/transaction regardless. **Using meeting as source of truth: all 3 documents.**
- **Adobe Sign cost is not a concern:** Richard confirmed they pay for 500 transactions and are well below that limit. 33 additional envelopes per year is negligible.
- **Reason for yearly re-request:** Richard noted the W9 form itself may change based on tax year — the client wants the current up-to-date version on file.
- **Lower priority:** Devon explicitly said "the W-9 tax ID stuff, just put it on hold" and "don't even think about that stuff" when discussing prioritization. This is groomed but not scheduled.
- **Reminder vs. new envelope:** Current "Resend" button sends an Adobe Sign reminder (free, no new envelope). The new feature needs to actually send a **new envelope** for a fresh signature. Whether to keep the reminder functionality alongside the new "Request New W9" action is an open question.
- **Signed status visibility:** Devon confirmed they get email notifications when a W9 is signed, and the signed status is displayed on the company page in the admin panel.
- **WATM-1491 is a duplicate:** Both the analysis and meeting confirmed these should be consolidated into one ticket.

---

## Resolved Questions

~~**Q-1: Envelope Contents — 2 or 3 documents?** — Resolved: All 3 documents (W9 + 1099 Consent + Distributor Agreement). Per meeting discussion, it's one envelope transaction regardless of doc count, and reusing the existing flow is simpler.~~

~~**Q-2: Keep Reminder Functionality?** — Resolved: Keep both. The existing "Send Reminder" button stays (poke to sign existing unsigned agreement). A new "Resend" button is added that sends a fresh envelope with new documents for a new signature.~~

---

## Requirements

### 1. New "Resend" Action — Fresh Envelope (Per-Distributor)

**1.1** Add a **"Resend"** button on the distributor company view page in the admin panel. This button sends a **new Adobe Sign envelope** with fresh documents for a new signature.

**1.2** When clicked, the system shall create a new Adobe Sign envelope containing all 3 documents: **W9 + 1099 Electronic Consent + Distributor Agreement** (same bundle as the initial distributor setup).

**1.3** The new envelope is sent to the distributor's contact email for signature, following the same Adobe Sign flow as the initial distributor setup.

**1.4** This is a **manual action** — an admin must navigate to the distributor's company page and click the button. There is no automated yearly trigger.

**1.5** This action is available regardless of whether the distributor has a previously signed agreement. It always creates a new envelope.

### 2. Agreement Replacement (Latest Only)

**2.1** When the distributor signs the new W9 envelope, the system shall replace the existing `distributor_agreement_id` on the company record with the new agreement.

**2.2** The previous signed agreement is **not retained** — only the most recent signed W9/agreement is kept on file.

**2.3** The signed status indicator on the company page shall reflect the status of the latest agreement (pending signature, signed, etc.).

### 3. Tax ID Extraction on New W9

**3.1** When a new W9 is signed, the system shall extract the Tax ID (SSN or EIN) from the signed form data and persist it to the company's `tax_id` field (per WATM-1981 requirements).

**3.2** This replaces any previously stored Tax ID with the value from the latest W9.

### 4. No Bulk Send

**4.1** There is no bulk "Send to All Distributors" action. Admins send W9 requests one distributor at a time.

**4.2** With 33 current distributor profiles, the admin navigates to each company's page and clicks "Request New W9" individually.

### 5. UI Updates

**5.1** The distributor company view page shall display two distinct actions:
- **"Send Reminder"** — existing functionality; sends a reminder to complete an existing unsigned agreement (free, no new envelope)
- **"Resend"** — new functionality; creates a new Adobe Sign envelope with fresh documents for a new signature (counts as a new transaction)

**5.2** The two buttons should be clearly distinguishable so the admin understands the difference between reminding (poke to sign existing) and resending (fresh envelope, new signature).

**5.3** The "Resend" button shall display a confirmation dialog before sending (e.g., "This will send a new envelope to [distributor name] for a fresh signature. This counts as a new Adobe Sign transaction. Continue?").

**5.4** The "Send Reminder" button is only available when the distributor has an existing unsigned agreement. The "Resend" button is always available.

**5.5** After sending, display a success message with the distributor name and a note that the envelope has been sent for signature.

---

## Out of Scope

- Bulk send to all distributors (client confirmed per-distributor is fine)
- Automated yearly trigger (manual only for now)
- W9 history / archive of previous signed W9s (only latest retained)
- Changes to the initial distributor setup envelope flow (that stays as-is with all 3 docs)
- Tax ID backfill for existing distributors (per WATM-1981 — no backfill)

---

## Dependencies

| Dependency | Status | Notes |
|------------|--------|-------|
| WATM-1981 (Tax ID storage) | Requirements In Progress | Tax ID column on companies table must exist for requirement 3.1. These tickets should be coordinated. |
| Adobe Sign API | Available | Existing `createEnvelope()` and webhook flow can be reused/adapted |
| Adobe Sign account renewal | Pending | Devon mentioned they're up for renewal. Ensure account is active before development. |

---

## Files Impacted

| File | Change |
|------|--------|
| `plugins/Companies/src/Controller/Admin/CompaniesController.php` | New action: `resendDistributorAgreement()` — creates new Adobe Sign envelope; keep existing reminder action |
| `plugins/Companies/src/Model/Table/CompaniesTable.php` | Reuse `sendDistributorEnvelope()` for resend (same 3-doc bundle) |
| `plugins/Companies/templates/Admin/Companies/view.php` | Add "Resend" button alongside existing "Send Reminder" button |
| `plugins/AdobeSign/src/Controller/Api/AgreementWebhooksController.php` | Handle new agreement replacing old `distributor_agreement_id` |
| `plugins/Companies/src/Model/Table/CompanyTaxDocumentsTable.php` | Extract and persist Tax ID when new W9 is signed |
| `config/app_local.php` | No changes needed — reuses existing library document IDs for all 3 docs |

---

## Acceptance Criteria

1. Admin can navigate to a distributor's company page and see both a "Send Reminder" button and a "Resend" button
2. Clicking "Resend" (with confirmation) sends a new Adobe Sign envelope with all 3 documents to the distributor for fresh signatures
3. Clicking "Send Reminder" sends a reminder for an existing unsigned agreement (existing behavior preserved)
4. When the distributor signs the new W9, the old agreement is replaced with the new one on the company record
5. Only the most recent signed agreement is retained — no history
6. Tax ID is extracted from the newly signed W9 and saved to the company record
7. The signed status indicator on the company page reflects the latest agreement status
8. No bulk send — per-distributor only
9. No automation — manual admin action only
