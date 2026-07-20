# WATM-1806: Twilio Account Investigation

**Ticket:** [WATM-1806](https://orases.atlassian.net/browse/WATM-1806)
**Date:** 2026-07-10
**Status:** Draft
**Goal:** Reactivate APW's Twilio account for SMS-based 2FA and device notifications in APC portal

---

## Login & Account Status Checklist

Use the shared credentials from 1Password (APW vault) to log in.

### 1. Account Status
- [ ] Can you log in at all? (account may be fully deactivated vs. suspended)
- [ ] What is the account status? (Active / Suspended / Closed)
- [ ] Is there an outstanding balance or payment issue?
- [ ] Is this a paid account or a trial account? (Trial accounts cannot register for A2P 10DLC)
- [ ] Check the account SID — note it for reference
- [ ] Check what Twilio products are currently enabled

### 2. Phone Number Status
- [ ] Is the original phone number still associated with the account?
- [ ] If the number was released/lost, note it — a new number may need to be purchased
- [ ] Check the phone number's current status (Active / Suspended / Released)
- [ ] Is there an emergency address associated with the phone number?
- [ ] Check if a regulatory compliance bundle exists for the number

### 3. A2P 10DLC Registration Status
- [ ] Navigate to: Messaging > Compliance > US A2P Brand Registrations
- [ ] Is there an existing Brand registration? What status? (Pending / Approved / Failed / None)
- [ ] Is there an existing Campaign registration? What status?
- [ ] Check if a Messaging Service exists and what phone numbers are assigned to it

### 4. Trust Hub / Business Profile
- [ ] Navigate to: Console > Trust Hub
- [ ] Is there a Business Profile set up?
- [ ] What information is already filled in? What's missing?
- [ ] Is there a Customer Profile for regulatory compliance?

### 5. Billing & Payment
- [ ] Check current balance
- [ ] Is there a valid payment method on file?
- [ ] Check for any past-due invoices
- [ ] Note the current billing plan/tier

### 6. Usage History
- [ ] Check last activity date — when was the account last actively sending SMS?
- [ ] Any error logs or failed message attempts?

---

## What APW Uses Twilio For

Per Devon's comments on the ticket:
1. **2FA on login** — One-time passcodes sent via SMS when users log into APC portal
2. **Device notifications** — SMS alerts for device events (offline, threshold alerts, etc.)

These map to Twilio A2P 10DLC campaign use cases:
- **"2FA / Authentication"** — standard use case, typically gets fastest approval
- **"Account Notifications"** — standard use case for device alerts

---

## A2P 10DLC Registration — What's Required

### Background
Since 2023-2024, all US carriers (AT&T, T-Mobile, Verizon) require A2P 10DLC registration for business SMS sent from 10-digit long code numbers. As of Feb 1, 2025, **100% of unregistered 10DLC traffic is blocked**. This is almost certainly why APW's SMS stopped working.

### Registration Steps (3-part process)

**Step 1: Business Profile / Brand Registration**
- Legal business name (must match IRS records exactly)
- EIN (Employer Identification Number) — must be validated
- Business address
- Business phone number
- Business website URL (must be live and accessible)
- Business type (e.g., Private Company)
- Industry
- Contact person name and email
- **Cost:** $4 (Low-Volume Standard) or $44 (Standard)

**Step 2: Campaign Registration**
- Select campaign use case type (e.g., "2FA", "Account Notifications")
- Campaign description (40-4096 chars) — must clearly state: who sends, who receives, why
- Sample messages (at least 2)
- Opt-in description — how users consent to receive SMS
- Opt-out keywords (STOP, CANCEL, etc.)
- Help keywords (HELP, INFO)
- Privacy Policy URL (REQUIRED as of June 30, 2026)
- Terms & Conditions URL (REQUIRED as of June 30, 2026)
- Message flow description
- **Cost:** $15 one-time vetting + $1.50-$10/month per campaign

**Step 3: Associate Phone Number with Messaging Service**
- Create a Messaging Service (or use existing)
- Link the registered Campaign to the Messaging Service
- Add the phone number(s) to the Messaging Service

### Approval Timeline
- Simple use case (2FA, notifications): **5-10 business days**
- Marketing/high volume: 2-3 weeks
- Complex use cases: 3-4 weeks

### Information Devon/Adam Will Need to Provide
- [ ] APW's EIN (Employer Identification Number)
- [ ] Legal business name as registered with IRS
- [ ] Business mailing address
- [ ] APW website URL (must be live and public)
- [ ] Privacy Policy URL that mentions SMS messaging
- [ ] Terms of Service URL that mentions SMS messaging
- [ ] Contact person authorized to register

---

## Common Rejection Reasons to Avoid

1. **EIN mismatch** — Business name must match IRS records exactly. Newly issued EINs take 30-90 days to propagate.
2. **Website issues** — URL must be live, accessible, not password-protected, and match the business name.
3. **Privacy Policy missing SMS language** — Must explicitly state opt-in data is not shared/sold, mention program name, message frequency, and "Message and data rates may apply."
4. **Campaign description mismatch** — Description must match the declared use case. Don't mix marketing language with 2FA registration.
5. **Missing opt-in mechanism** — Must describe how users consent to receive messages.
6. **Link shorteners** — Bitly/TinyURL in sample messages trigger automatic rejection.
7. **Missing Privacy/ToS URLs** — As of June 30, 2026, these are hard-required fields.

---

## Possible Scenarios & Next Steps

### Scenario A: Account is Suspended (Balance Issue)
- Add funds / update payment method
- Wait 5-10 minutes for reactivation
- Then proceed with A2P 10DLC registration

### Scenario B: Account is Suspended (Compliance/Fraud)
- Contact Twilio Support directly
- May need to provide RCA report
- May need to settle negative balance
- Could take longer to resolve

### Scenario C: Account is Active but SMS is Blocked
- Most likely: unregistered for A2P 10DLC
- Complete the 3-step registration process above
- Expect 5-10 business day approval for 2FA use case

### Scenario D: Account is Fully Closed / Phone Number Lost (MOST LIKELY)
- **After 90 days of suspension, Twilio reclaims phone numbers.** Since APW's account has been inactive ~2+ years, their original number(s) are almost certainly released.
- It may be simpler to **create a new Twilio account** rather than fight to reactivate the old one
- Purchase a new phone number (or port existing number if they still have it with another carrier)
- Complete A2P 10DLC registration from scratch
- Update APC portal code with new Twilio credentials (Account SID, Auth Token, phone number)

---

## Cost Estimate

| Item | Cost |
|------|------|
| Low-Volume Standard Brand registration | $4 one-time |
| Campaign vetting fee (per campaign) | $15 one-time |
| Campaign monthly fee | $1.50 - $10/month |
| Twilio phone number | ~$1.15/month |
| SMS per-message cost | ~$0.0079/segment outbound |

If APW needs higher throughput, Standard Brand registration is $44 one-time instead of $4. Low-Volume Standard is likely sufficient for 2FA + device alerts.

---

## Notes After Investigation

_Fill in after logging into the Twilio account:_

**Account Status:**
**Phone Number Status:**
**A2P Registration Status:**
**Balance:**
**Key Findings:**
**Recommended Next Steps:**
