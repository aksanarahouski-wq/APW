# WATM-1501: Client Message Draft

**To:** Laura Perry / Jon Mack / APW Team
**Re:** WATM-1501 — Add the ability to resend W9 documents yearly (and WATM-1491)

---

Hi Laura / Jon,

We've reviewed WATM-1501 (resend W9 documents yearly) and noticed that WATM-1491 (resend W9 to distributors) describes the same feature. We'd suggest consolidating these into one ticket to keep things clean. Let us know if you agree.

Here's what we found from the technical review:

**What exists today:**
- When a company is set up as a distributor, the system automatically sends a 3-document Adobe Sign envelope: W9 + Distributor Agreement + 1099 Electronic Consent.
- There's a "Resend" button on individual company pages, but it sends a **reminder** to complete the existing agreement — it doesn't request a fresh signature.
- There is no bulk send option. Each distributor must be handled individually.

**What needs to be built:**
- A way to request a **new W9 signature** (not just a reminder on the old one) — either per distributor or in bulk.
- This would create a new Adobe Sign envelope each time, which counts as a transaction on your Adobe Sign plan.

**A few questions before we scope this:**

1. **W9 only, or the full package?** Today the W9 is bundled with the Distributor Agreement and 1099 Consent form. For the yearly renewal, do you just need a fresh W9, or do all three documents need to be re-signed?

2. **Adobe Sign envelope costs:** Each new W9 request counts as a new envelope/transaction in Adobe Sign. Is that acceptable, or would you prefer we explore alternatives (e.g., only sending to distributors whose information has changed)?

3. **Individual vs. bulk send:** We currently have 33 distributor profiles in the system. Given that relatively small number, do you want us to build a bulk "Send to All Distributors" action, or would enhancing the existing per-distributor "Request New W9" button be sufficient? Building bulk adds development time and Adobe Sign cost (33 envelopes at once), so we want to make sure it's worth it at this scale.

4. **Manual or automatic?** Should this be a button an admin clicks at the start of each year, or would you like it automated (e.g., system sends W9 requests to all distributors every January)?

5. **Previous W9 history:** When a distributor signs a new W9, should we keep the old signed W9 on file, or only retain the most recent?

6. **Connection to WATM-1981 (Tax ID on commissions report):** If we're collecting fresh W9s annually, we can also extract and store the Tax ID at that point — which would solve the Tax ID requirement for the commissions report. Should we plan these together?

Let us know your thoughts and we'll get this scoped.

Thanks,
Aksana
