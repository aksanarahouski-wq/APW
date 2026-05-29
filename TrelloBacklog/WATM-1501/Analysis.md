# WATM-1501: Resend W9 Documents Yearly — Technical Analysis

**Ticket:** WATM-1501 — Add the ability to resend W9 documents yearly
**Date:** 2026-04-04
**Status:** Draft
**Assignee:** aaron.diefes
**Creator:** Jon Mack
**Parent:** WATM-2020: W9 document enhancements
**Related:** WATM-1491: Add ability to resend W9 document to distributors

## Client Request

> WATM will need the ability to resend W9 documents at the beginning of each calendar year. They should be able to send to all distributors and individual distributors.
>
> **Research:** We need to figure out if resending the W9 doc counts as an envelope.

## Related Ticket: WATM-1491

WATM-1491 is nearly identical in scope:

> WATM will need the ability to resend W9 documents to distributor account types. The W9 document should be sent via Adobe Sign. They should be able to select an individual distributor or all distributors.

**Important comments on WATM-1491:**
- **Kevin Long (2024-11-12):** "Is resending to all distributors a valuable option at this time, given the small number of distributors? Also, not sure of the use case for needing to REsend to all."
- **Jon Mack (2024-11-12):** "This story was created when we discussed all of the initial items that WATM wanted for commissions. This is part of the Commissions P2 initiative, and it hasn't been prioritized at all. I think they might want this at some point, but I doubt we would work on this for a long time."

**Recommendation:** WATM-1501 and WATM-1491 should likely be consolidated into a single ticket. They describe the same feature.

## Current W9/Adobe Sign Architecture

### How W9s Are Sent Today

1. **Automatic on distributor creation:** When a company's `account_type_id` is changed to "Distributor", `CompaniesTable::beforeSave()` automatically calls `sendDistributorEnvelope()` (lines 696-717)
2. **Manual send/resend per company:** `CompaniesController::sendDistributorAgreement($companyId)` (lines 297-366) — button on the company view page that says "Send" or "Resend" depending on whether an agreement already exists

### What Gets Sent

A single Adobe Sign envelope containing **3 documents** (`CompaniesTable.php` lines 910-915):
- `1099Consent` — Electronic consent checkbox
- `DistributorAgreement` — Main distributor agreement
- `W9` — W9 tax form

All three are bundled as one envelope. The W9 is **not sent independently** — it's part of the full distributor agreement package.

### Resend vs. New Send

The current `sendDistributorAgreement()` action handles both cases (lines 313-345):
- **If agreement exists:** Calls `AdobeSignApi::setAgreementReminder()` — sends a reminder to complete the existing agreement (no new envelope)
- **If no agreement:** Calls `sendDistributorEnvelope()` — creates a brand new envelope (counts as a new transaction)

### No Bulk Send Capability

Today, agreements are sent **one at a time** only. There is no batch/bulk send interface. The admin must go to each company's view page and click Send/Resend individually.

### Adobe Sign Cost Implications — RESOLVED

The ticket asks: *"We need to figure out if resending the W9 doc counts as an envelope."*

**Answer: Yes, resending a W9 for a fresh signature counts as a new envelope.**

From the code:
- **Reminder** (`setAgreementReminder`): Sends a reminder for an existing unsigned agreement. This does **not** count as a new envelope/transaction. However, a reminder only nudges the signer to complete an already-open agreement — it does not collect a new signature.
- **New envelope** (`sendDistributorEnvelope`): Creates a new Adobe Sign agreement. This **does** count as a new envelope/transaction.

For the yearly W9 use case, the intent is to collect a **freshly signed W9** each year. That requires a new envelope each time — meaning **one new Adobe Sign transaction per distributor per year**. With 33 current distributors, that's 33 envelopes annually.

## What Needs to Be Built

### For Individual Resend (Partially Exists)

The "Resend" button already exists on the company view page. However:
- It sends a **reminder** on the existing agreement, not a new W9
- For a yearly fresh W9, we'd need a "Request New W9" action that creates a **new** envelope with just the W9 (not the full distributor agreement bundle)
- Need to handle the old `distributor_agreement_id` — archive it? Replace it? Track history?

### For Bulk Resend (New Feature)

- New admin UI: likely on the Commissions or Companies index page
- "Send W9 to All Distributors" button with confirmation
- Queue-based processing: create a job that iterates through all distributor-type companies and sends envelopes
- Progress tracking / status reporting
- Error handling for individual failures in the batch

### Key Technical Decisions

| Decision | Options | Notes |
|----------|---------|-------|
| Send W9 only, or full envelope? | W9-only vs. all 3 documents | Yearly renewal probably only needs W9, not the full distributor agreement + 1099 consent again |
| New envelope or reminder? | New envelope (fresh signature) vs. reminder (complete existing) | Fresh W9 each year = new envelope = Adobe Sign cost per distributor |
| Agreement history | Replace `distributor_agreement_id` vs. track history | If we replace, we lose the old signed W9. May need a history table or keep old agreements linked. |
| Bulk processing | Synchronous vs. queued job | Must be queued — bulk Adobe Sign API calls would timeout in a web request |

## Key Files

| File | Purpose |
|------|---------|
| `plugins/Companies/src/Model/Table/CompaniesTable.php` | `sendDistributorEnvelope()` (lines 905-965), `beforeSave()` auto-send (lines 696-717) |
| `plugins/Companies/src/Controller/Admin/CompaniesController.php` | `sendDistributorAgreement()` (lines 297-366) |
| `plugins/AdobeSign/src/Api/AdobeSignApi.php` | `setAgreementReminder()` (lines 117-139), `createEnvelope()` |
| `plugins/AdobeSign/src/Command/PollAgreementsCommand.php` | Agreement completion polling and PDF download |
| `plugins/AdobeSign/src/Controller/Api/AgreementWebhooksController.php` | Webhook handler for status changes |
| `plugins/Companies/src/Model/Table/CompanyTaxDocumentsTable.php` | Tax ID extraction from signed W9 (lines 193-200) |
| `plugins/Companies/src/Controller/Admin/CommissionsController.php` | Annual report, tax document generation |
| `config/app_local.php` | Adobe Sign library document IDs (W9, DistributorAgreement, 1099Consent) |

## Open Questions for Client

1. **WATM-1491 overlap:** This ticket and WATM-1491 describe the same feature. Should we consolidate them into one?
2. **W9 only or full package?** Today the W9 is sent as part of a 3-document bundle (W9 + Distributor Agreement + 1099 Consent). For yearly renewal, do you need just the W9 re-signed, or the full package again?
3. **Adobe Sign costs:** Sending a fresh W9 each year creates a new Adobe Sign envelope per distributor. Have you confirmed this is acceptable with your Adobe Sign plan? (Reminders on existing unsigned agreements are free, but a new signature request is a new transaction.)
4. **Agreement history:** When a distributor signs a new yearly W9, should we keep the previous year's signed W9 on file, or is only the most recent needed?
5. **Bulk vs. individual:** There are currently 33 distributor profiles in the system. Is bulk send worth the added development effort and Adobe Sign cost (33 envelopes at once) at this scale, or would per-distributor "Request New W9" buttons suffice?
6. **Timing/trigger:** Should this be a manual action ("Send W9 to all distributors" button), or an automated annual process (e.g., auto-send every January 1)?

## Relationship to WATM-2020 Epic

This is the third ticket under the W9 document enhancements epic. The three tickets are interrelated:

| Ticket | Summary | Dependency |
|--------|---------|------------|
| WATM-1981 | Add Tax ID and split address on commissions report | Needs Tax ID stored in DB (currently only in Adobe Sign) |
| WATM-1501 | Resend W9 documents yearly | Core W9 re-collection flow |
| WATM-1491 | Resend W9 to individual/all distributors | Duplicate of WATM-1501 |

If we build WATM-1501 with a new envelope flow, we should also extract and persist the Tax ID when the new W9 is signed — that would solve WATM-1981's Tax ID requirement at the same time.
