# WATM-1501: Resend W9 Documents to Distributors
## Feature Summary

**Date:** April 7, 2026
**Ticket:** WATM-1501
**Parent Epic:** WATM-2020 — W9 Document Enhancements
**Status:** Requirements Finalized
**Priority:** Lower — groomed and ready, not yet scheduled

---

## What We're Building

We're adding a **"Resend"** button on each distributor's company page that lets an admin request a brand new W9 signature. This sends a fresh Adobe Sign envelope with all three documents (W9, 1099 Electronic Consent, and Distributor Agreement) for the distributor to sign again.

The existing **"Send Reminder"** button stays as-is — it still nudges distributors who haven't signed an outstanding agreement.

**Why this matters:** The IRS W9 form may change from year to year. This feature lets you collect current, up-to-date W9s from any distributor whenever you need to — no developer involvement required.

---

## How It Works

### Requesting a New W9

1. **Navigate** to the distributor's company page in the admin panel
2. Click **"Resend"**
3. A confirmation dialog appears: *"This will send a new envelope to [distributor name] for a fresh signature. Continue?"*
4. **Confirm** — a new Adobe Sign envelope is sent to the distributor's email
5. The distributor signs the documents
6. The new signed agreement **replaces** the previous one on file
7. The distributor's **Tax ID** is automatically extracted from the signed W9 and saved to their company record

### Sending a Reminder (Unchanged)

- If a distributor has an outstanding unsigned agreement, you'll still see a **"Send Reminder"** button
- This sends a nudge to complete the existing agreement — it does not create a new envelope
- This works exactly as it does today

### What Stays the Same

- **Initial distributor setup** is unchanged — when a company is first set up as a distributor, the system still automatically sends the 3-document envelope
- **Email notifications** — you'll still get notified when a distributor signs
- **Signed status** on the company page reflects the latest agreement

---

## Decisions Made

| Topic | Decision |
|-------|----------|
| What documents are sent on resend? | All 3 — W9, 1099 Electronic Consent, and Distributor Agreement (same as initial setup, one envelope transaction) |
| Individual or bulk send? | Per-distributor only — navigate to each distributor's page and click "Resend" |
| Manual or automatic? | Manual — admin clicks the button when needed, no automated yearly trigger |
| Keep previous W9 on file? | No — only the most recent signed agreement is retained |
| Keep the existing "Send Reminder" button? | Yes — both "Send Reminder" and "Resend" will be available |
| Adobe Sign cost impact? | Each resend is one new transaction. At 33 distributors and a 500-transaction plan, this is well within your limits |
| What happens to Tax ID? | Automatically extracted from the new W9 and saved to the company record (supports the Tax ID on commissions report work in WATM-1981) |

---

## What to Expect

- **After deployment**, you'll see two buttons on each distributor's company page:
  - **"Send Reminder"** — same as today, for unsigned agreements
  - **"Resend"** — new, sends a fresh envelope for a new signature
- **When a distributor signs** the new documents, their agreement and Tax ID are automatically updated
- **Previous agreements are not kept** — only the latest is on file
- **No bulk option** — you'll go to each distributor individually (manageable at 33 distributors)
- **No automation** — you control when and who gets a resend

---

## Scope Boundaries

**Included:**
- "Resend" button per distributor (new envelope with fresh documents)
- "Send Reminder" preserved (existing behavior)
- Agreement replacement on new signature
- Tax ID extraction from newly signed W9

**Not included at this time:**
- Bulk "Send to All Distributors" action
- Automated yearly W9 requests
- W9 history or archive of previous signed documents
- Tax ID backfill for distributors who signed before this feature

---

## Related Work

- **WATM-1981** — Adding Tax ID and address columns to the Annual Commissions Report. The Tax ID storage built for that ticket is reused here when extracting Tax ID from newly signed W9s.
- **WATM-1491** — Duplicate of this ticket. Will be consolidated into WATM-1501.
