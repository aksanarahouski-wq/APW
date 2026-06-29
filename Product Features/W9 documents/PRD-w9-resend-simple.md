# Product Requirements Document - Simple Feature
## Resend W9 Documents to Distributors

**Document Version:** 1.0
**Date:** 2026-04-07
**Author:** Aksana / Orases
**Status:** Draft
**Related Tickets:** [WATM-1501](https://orases.atlassian.net/browse/WATM-1501), [WATM-1491](https://orases.atlassian.net/browse/WATM-1491) (duplicate — consolidate)
**Parent Epic:** [WATM-2020](https://orases.atlassian.net/browse/WATM-2020) — W9 Document Enhancements
**Document Owner:** Aksana
**Last Updated:** 07.04.2026

---

## Table of Contents

1. [Overview](#overview)
2. [Goals and Success Criteria](#goals-and-success-criteria)
3. [Users](#users)
4. [Scope](#scope)
5. [Functional Requirements and Business Rules](#functional-requirements-and-business-rules)
6. [Technical Requirements](#technical-requirements)
7. [Testing Requirements](#testing-requirements)
8. [Dependencies and Risks](#dependencies-and-risks)
9. [Open Questions](#open-questions)
10. [References](#references)
11. [Notes](#notes)

---

## Overview

This feature adds the ability for WATM administrators to request a fresh W9 signature from individual distributors by sending a new Adobe Sign envelope. Currently, the only option is a "Send Reminder" that nudges distributors to complete an existing unsigned agreement. The new "Resend" action creates a brand new envelope with fresh documents (W9 + 1099 Electronic Consent + Distributor Agreement) for a new signature — enabling the client to collect up-to-date W9 forms annually or whenever needed.

### Key Features
- **"Resend" Button**: New per-distributor action on the company view page that sends a fresh Adobe Sign envelope for a new signature
- **"Send Reminder" Preserved**: Existing reminder functionality remains available for nudging distributors who haven't signed an outstanding agreement
- **Agreement Replacement**: When a new W9 is signed, the previous agreement is replaced — only the most recent is retained
- **Tax ID Extraction**: Tax ID is extracted from the newly signed W9 and saved to the company record

### Business Impact
- Enables the client to collect current, up-to-date W9 forms from distributors annually (the IRS W9 form may change by tax year)
- Provides admin self-service — no developer involvement needed to request fresh signatures
- Supports WATM-1981 (Tax ID on commissions report) by capturing Tax ID from newly signed W9s going forward

---

## Goals and Success Criteria

### Primary Goals
1. **Enable fresh W9 collection per distributor**: Admins can request a new W9 signature from any distributor at any time via the admin panel
2. **Preserve existing reminder functionality**: The "Send Reminder" action for unsigned agreements continues to work alongside the new "Resend" action
3. **Keep only the latest agreement on file**: When a distributor signs a new W9, the previous agreement is replaced

### Success Criteria

- Admin can see both "Send Reminder" and "Resend" buttons on a distributor's company page
- Clicking "Resend" creates a new Adobe Sign envelope with all 3 documents and sends it to the distributor
- When the new W9 is signed, the old agreement is replaced and the Tax ID is extracted and saved
- "Send Reminder" continues to function as before for existing unsigned agreements
- No bulk send and no automation — manual per-distributor action only

---

## Users

### Primary Users

**WATM Administrator (APW Team)**
- **Role**: Admin panel user responsible for managing distributor companies and compliance documentation
- **Need**: Ability to request a fresh W9 signature from individual distributors without developer involvement
- **Pain Point**: Currently can only send a reminder on an existing unsigned agreement. There is no way to request a brand new W9 signature — the admin cannot collect updated W9 forms when the IRS form changes or when distributor information changes.
- **Benefit**: Self-service W9 re-collection on demand, per distributor, with a clear distinction between "remind to sign" and "request new signature"

---

## Scope

### In Scope

**Admin UI**
- "Resend" button on the distributor company view page (sends new envelope)
- "Send Reminder" button preserved (existing reminder functionality)
- Confirmation dialog on "Resend" to prevent accidental sends
- Success message after sending

**Adobe Sign Integration**
- New envelope creation reusing the existing 3-document bundle (W9 + 1099 Consent + Distributor Agreement)
- Agreement replacement when new envelope is signed (old agreement replaced)
- Tax ID extraction from newly signed W9 form data

**Data**
- Update `distributor_agreement_id` on company record when new agreement is signed
- Persist extracted Tax ID to company's `tax_id` field (per WATM-1981)

### Out of Scope
- Bulk "Send to All Distributors" action (client confirmed per-distributor is sufficient at 33 distributors)
- Automated yearly trigger (manual only for now)
- W9 history or archive of previous signed agreements (only latest retained)
- Changes to the initial distributor setup envelope flow (stays as-is)
- Tax ID backfill for distributors who signed W9s before this enhancement (per WATM-1981 — no backfill)
- Changes to the Adobe Sign reminder behavior (existing functionality preserved as-is)

---

## Functional Requirements and Business Rules

### FR-1: Resend — New Adobe Sign Envelope

**FR-1.1: Resend Action**
- The system shall provide a "Resend" button on the distributor company view page in the admin panel
- When clicked, the system shall display a confirmation dialog stating: "This will send a new envelope to [distributor name] for a fresh signature. This counts as a new Adobe Sign transaction. Continue?"
- If confirmed, the system shall create a new Adobe Sign envelope containing all 3 documents: W9, 1099 Electronic Consent, and Distributor Agreement
- The envelope shall be sent to the distributor's contact email address
- After sending, the system shall display a success message confirming the envelope was sent

**Business Rules:**
- The "Resend" button is always available on distributor company pages, regardless of whether a previous agreement exists or its status
- Each resend creates a new Adobe Sign transaction (counts against the 500-transaction plan; current usage is well below this limit)
- The 3-document bundle is identical to the initial distributor setup envelope — same library document IDs, same flow

**Interaction & UI Details:**
- The "Resend" button is visually distinct from the "Send Reminder" button so the admin understands the difference
- Confirmation dialog prevents accidental sends
- Success/error feedback displayed inline on the company view page

**FR-1.2: Send Reminder (Existing — Preserved)**
- The existing "Send Reminder" button shall continue to function as-is
- It sends an Adobe Sign reminder to complete an existing unsigned agreement (free, no new envelope)
- The "Send Reminder" button is only available when the distributor has an existing unsigned agreement

**Business Rules:**
- Reminders do not create a new envelope or transaction
- Reminders are only applicable when there is an outstanding unsigned agreement

### FR-2: Agreement Replacement

**FR-2.1: Replace on New Signature**
- When the distributor signs the new W9 envelope, the system shall update the company's `distributor_agreement_id` to reference the new agreement
- The previous agreement is not retained — only the most recent signed agreement is kept on file

**FR-2.2: Signed Status Indicator**
- The signed status indicator on the company view page shall reflect the status of the latest agreement (e.g., "Pending Signature", "Signed")
- When a new envelope is sent via "Resend", the status shall update to reflect the pending new agreement

**Business Rules:**
- Only one agreement is active per distributor at any time
- The system replaces the old `distributor_agreement_id` — no history table or archive

### FR-3: Tax ID Extraction

**FR-3.1: Extract and Persist Tax ID**
- When a new W9 is signed, the system shall extract the Tax ID (SSN or EIN) from the Adobe Sign form data
- The extracted Tax ID shall be persisted to the company's `tax_id` field (encrypted via `TwoWayCryptedType`)
- This replaces any previously stored Tax ID with the value from the latest W9

**Business Rules:**
- Tax ID extraction follows the existing logic in `CompanyTaxDocumentsTable.php` (SSN fields: SSN1-SSN3, EIN fields: EIN1-EIN2)
- If the W9 form data does not contain a valid Tax ID, the `tax_id` field is not updated
- This requirement depends on WATM-1981 (Tax ID storage column) being implemented first or concurrently

### FR-4: No Bulk Send

**FR-4.1: Per-Distributor Only**
- There is no bulk "Send to All Distributors" action
- Admins must navigate to each distributor's company page and click "Resend" individually
- With 33 current distributor profiles, this is manageable without bulk functionality

### Key User Flows

- **Happy Path — Resend W9**: Admin navigates to distributor company page → clicks "Resend" → confirms in dialog → envelope sent → distributor receives email → signs documents → agreement replaced on company record → Tax ID extracted and saved → signed status updates
- **Reminder Path**: Admin navigates to distributor company page → sees unsigned agreement → clicks "Send Reminder" → reminder sent → distributor receives nudge email (existing flow, unchanged)
- **Error Path**: Admin clicks "Resend" → confirms → Adobe Sign API error → error message displayed → admin can retry

---

## Technical Requirements

### Integration

**Adobe Sign API**
- Reuse existing `sendDistributorEnvelope()` method in `CompaniesTable.php` to create new envelopes with the same 3-document bundle
- Reuse existing `createEnvelope()` in `AdobeSignApi.php`
- Existing webhook flow (`AgreementWebhooksController.php`) handles agreement completion — extend to replace `distributor_agreement_id` and extract Tax ID

### Security & Compliance
- Tax ID stored encrypted via existing `TwoWayCryptedType` (per WATM-1981)
- Admin-only access enforced by existing RBAC on `CompaniesController` actions
- Audit logging via existing `o_logs` integration for resend actions

### Files Impacted

| File | Change |
|------|--------|
| `plugins/Companies/src/Controller/Admin/CompaniesController.php` | New action: `resendDistributorAgreement()` — creates new Adobe Sign envelope; existing reminder action preserved |
| `plugins/Companies/src/Model/Table/CompaniesTable.php` | Reuse `sendDistributorEnvelope()` for resend (same 3-doc bundle) |
| `plugins/Companies/templates/Admin/Companies/view.php` | Add "Resend" button alongside existing "Send Reminder" button |
| `plugins/AdobeSign/src/Controller/Api/AgreementWebhooksController.php` | Handle new agreement replacing old `distributor_agreement_id` |
| `plugins/Companies/src/Model/Table/CompanyTaxDocumentsTable.php` | Extract and persist Tax ID when new W9 is signed |

---

## Testing Requirements

### Test Plan Overview
**Testing Phases:**
1. Integration Testing (Development team)
2. User Acceptance Testing (UAT) (APW team / Devon D'Andrea)
3. Regression Testing (QA team)

### Key Test Scenarios

**IT-1: Resend Creates New Envelope**
1. Navigate to a distributor's company view page
2. Click "Resend"
3. Confirm in the dialog
4. **Verify:** A new Adobe Sign envelope is created and sent to the distributor's email
5. **Verify:** Success message is displayed

**IT-2: Agreement Replacement on Signature**
1. Send a new envelope to a distributor via "Resend"
2. Complete the signing process (simulate or use test account)
3. **Verify:** The company's `distributor_agreement_id` is updated to the new agreement
4. **Verify:** The signed status indicator on the company page shows "Signed"

**IT-3: Tax ID Extraction**
1. Send a new envelope and complete signing with a test Tax ID
2. **Verify:** The company's `tax_id` field is populated with the extracted Tax ID
3. **Verify:** If the distributor had a previous Tax ID, it is replaced with the new value

**IT-4: Send Reminder Still Works**
1. Create a scenario where a distributor has an unsigned agreement
2. Click "Send Reminder"
3. **Verify:** An Adobe Sign reminder is sent (not a new envelope)
4. **Verify:** No new transaction is created

**IT-5: Confirmation Dialog Prevents Accidental Sends**
1. Click "Resend"
2. Click "Cancel" in the confirmation dialog
3. **Verify:** No envelope is sent

**IT-6: Button Availability**
1. Navigate to a distributor with a signed agreement
2. **Verify:** "Resend" button is available; "Send Reminder" button is not available (no unsigned agreement)
3. Navigate to a distributor with an unsigned agreement
4. **Verify:** Both "Resend" and "Send Reminder" buttons are available

**UAT-1: End-to-End W9 Re-Collection**
- **Persona:** WATM Administrator (APW Team)
- **Scenario:** Admin needs to collect a fresh W9 from a specific distributor at the start of the year
- **Steps:** Navigate to distributor's company page → click "Resend" → confirm → distributor receives and signs → verify agreement updated and Tax ID captured
- **Success Criteria:** New signed W9 on file, Tax ID extracted, old agreement replaced

**RT-1: Initial Distributor Setup Regression**
- **Verify:** Setting a company's account type to "Distributor" still triggers the automatic initial envelope send (existing behavior)
- **Verify:** The initial setup still sends all 3 documents as one envelope
- **Verify:** "Send Reminder" still works for unsigned initial agreements

### Testing Notes
- Adobe Sign test/sandbox environment required for integration testing
- Confirm Adobe Sign account is active and renewed before testing
- Test with both SSN and EIN format Tax IDs to verify extraction logic

---

## Dependencies and Risks

### Dependencies

**Must Exist Before Development:**
1. **WATM-1981 (Tax ID storage)** - The encrypted `tax_id` column on the `companies` table must exist for Tax ID extraction to persist | **Mitigation:** Develop WATM-1981 first or concurrently
2. **Adobe Sign account renewal** - Devon mentioned they're up for renewal; account must be active | **Mitigation:** Confirm renewal status before development begins

**Integrates With:**
- Adobe Sign API (`createEnvelope`, `setAgreementReminder`, agreement webhooks)
- Existing distributor setup flow in `CompaniesTable::sendDistributorEnvelope()`
- WATM-1981 Tax ID extraction and storage

### Risks

**LOW RISK: Adobe Sign Account Renewal**
- **Description:** The client's Adobe Sign account is up for renewal. If not renewed, envelope creation will fail.
- **Impact:** Medium — feature is entirely dependent on Adobe Sign
- **Probability:** Low — client is aware and in renewal process
- **Mitigation:** Confirm account status before development; feature is lower priority so timeline is flexible

**LOW RISK: Reminder Behavior Uncertainty**
- **Description:** There was discussion about whether Adobe Sign "reminders" actually resend documents or just send a notification. The exact behavior needs verification during development.
- **Impact:** Low — reminder is existing functionality, not changing
- **Probability:** Medium — Richard and Devon had differing recollections
- **Mitigation:** Verify reminder behavior in Adobe Sign sandbox during development; document findings for future reference

---

## Open Questions

No open questions. All critical decisions were resolved via Confluence comments and meeting discussion.

---

## References

### Related Features
- [WATM-1981](https://orases.atlassian.net/browse/WATM-1981) — Annual Commissions Report: Add Tax ID and Update Addresses (Tax ID storage dependency)
- [WATM-1491](https://orases.atlassian.net/browse/WATM-1491) — Add ability to resend W9 document to distributors (duplicate — consolidate into WATM-1501)
- [WATM-2020](https://orases.atlassian.net/browse/WATM-2020) — W9 Document Enhancements (parent epic)

### Supporting Documentation
- [WATM-1501 Technical Analysis](./Analysis.md) — Architecture review, current W9 flow, and Adobe Sign integration details
- [WATM-1501 Client Message](./Client_Message.md) — Client communication with questions
- [W9 Document Enhancements — Confluence Page](https://orases.atlassian.net/wiki/spaces/WATM/pages/3095560194/W9+document+enhancements) — Client Q&A with inline comments

---

## Notes

### Evidence Sources
- Client inline comments on Confluence page (Devon D'Andrea, April 6, 2026)
- Meeting transcript (Meeting1.md) — W9 discussion timestamps 10:35–25:54
- Technical analysis performed 2026-04-04 (Analysis.md)
- Jira ticket WATM-1501 (Jon Mack, creator) and WATM-1491 (duplicate)

### Key Decisions
- **All 3 documents in resend envelope**: Confluence comment said W9 + 1099 consent only, but meeting discussion concluded all 3 (W9 + 1099 Consent + Distributor Agreement) since it's one transaction regardless and reuses the existing flow. **Meeting used as source of truth.** (source: Devon D'Andrea, meeting ~16:44)
- **Keep both reminder and resend**: Existing "Send Reminder" preserved alongside new "Resend" button. Reminder pokes for existing unsigned agreement; Resend creates fresh envelope. (source: user decision 2026-04-07)
- **Per-distributor only, no bulk**: Client confirmed bulk is not needed at 33 distributors. (source: Confluence comment + meeting ~14:45)
- **Manual trigger, no automation**: Admin clicks a button; no automated yearly process. (source: Confluence comment + meeting ~25:27)
- **Only retain latest agreement**: No W9 history; new agreement replaces old. (source: Confluence comment)
- **Lower priority**: Devon stated this should be put on hold; groomed but not scheduled. (source: meeting ~01:02:05)

---

END OF DOCUMENT
