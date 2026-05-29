# WATM-1981: Client Message Draft

**To:** Laura Perry / APW Team
**Re:** WATM-1981 — Annual Commissions Report: Add Tax ID and Update Addresses

---

Hi Laura,

We've reviewed the request to add Tax ID and separate address columns to the Annual Commissions report. Here's what we found:

**Address — No Problem**
The addresses are already stored as separate fields in the system (Address, City, State, Zip). The current CSV export combines them into one column for display. Splitting them into individual columns is straightforward.

**Tax ID — Needs Discussion**
Currently, Tax IDs are not stored in our database. They live inside the signed W9 forms in Adobe Sign, and we only pull them when generating 1099 documents. To include Tax ID on the report, we'd need to either:

- **Option A:** Call Adobe Sign's API for each distributor every time the report runs. This would work but could be slow and unreliable if there are many distributors or Adobe Sign is having issues.
- **Option B (Recommended):** Add a Tax ID field to the company record in our system, populated when the W9 is signed. This makes the data instantly available for the report and any future features that need it. The field would be stored encrypted for security.

We'd recommend Option B since it aligns with the W9 document enhancements work (WATM-2020) and gives you reliable, fast access to Tax IDs across the system.

**A few questions before we scope this:**

1. Currently the HTML report page and the CSV export show different columns. We'll be adding Tax ID and the split address columns to the **CSV export**. Would you also like Tax ID and address columns added to the **HTML report page** in the admin panel?
2. Should we include **Address Line 2** as its own column, or just Address, City, State, Zip?
3. For distributors who **haven't submitted a W9** yet — should the Tax ID column be blank, or would you like us to flag them?
4. Since this is part of the W9 enhancements epic (WATM-2020), should we coordinate this with the other W9 work to avoid overlap?

Let us know your thoughts and we'll get this scoped.

Thanks,
Aksana
