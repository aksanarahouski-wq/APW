# Company Invitations: Visibility & Management
## Proposal for APCommand Enhancement

**Date:** March 3, 2026
**Prepared for:** WATM Client

---

## Executive Summary

**Current Situation:**
When you send a company invitation through APCommand, the system stores the information and sends the email successfully. However, there is no way to view invitations you've sent or check their status, creating a "blind spot" in your customer onboarding process.

**Proposed Solution:**
Implement a Company Invitations dashboard in two phases to provide complete visibility into who has been invited and whether they've created accounts.

**Recommendation:**
Build Phase 1 and Phase 2 together (low effort). Phase 3 can be added later if needed.

---

## The Problem

**What's Missing:**
- ❌ No way to see who has been invited
- ❌ Cannot tell if an invitation was accepted or is still pending
- ❌ No ability to track how long invitations have been outstanding
- ❌ Cannot identify which invitations led to company registrations

---

## Proposed Solution

### Phase 1: Invitation Visibility
**Effort: Small**

A simple "Company Invitations" page showing all sent invitations.

**What You'll See:**
```
Company Invitations                                [+ Send New Invite]

Search: [__________________]  [Search]

┌──────────────────────────────────────────────────────────────┐
│ Email Address     │ Account Type │ Sent Date    │ Sent By    │
├──────────────────────────────────────────────────────────────┤
│ john@customer.com │ Customer     │ Feb 15, 2026 │ Admin User │
│ jane@dist.com     │ Distributor  │ Feb 10, 2026 │ Admin User │
│ bob@subclient.com │ Subcustomer  │ Feb 8, 2026  │ Customer   │
└──────────────────────────────────────────────────────────────┘
```

**Features:**
- View all invitations in one place
- Search by email address
- Sort by any column

---

### Phase 2: Status Tracking (Recommended)
**Effort: Small**

**What's Missing Today:**
Currently, there's no way to answer the critical question: *"What happened to that invitation?"*
- Did the recipient create a company account?
- Is the invitation still pending?
- Has it been sitting too long without response?
- Which company was created from this invitation?

Phase 2 adds status tracking to answer these questions and close the loop on your invitation process.

**What You'll See (Additional):**
```
Company Invitations                                [+ Send New Invite]

Summary:  Total: 47  |  Pending: 12  |  Accepted: 32 (68%)  |  Expired: 3

Filters:  [Status ▼]  [Account Type ▼]  [Date Range ▼]

┌──────────────────────────────────────────────────────────────────┐
│ Email           │ Type      │ Status   │ Sent Date │ Action     │
├──────────────────────────────────────────────────────────────────┤
│ john@cust.com   │ Customer  │ Pending  │ Feb 15    │ Follow up  │
│ jane@dist.com   │ Dist.     │ Accepted │ Feb 10    │ View Co.   │
│ old@inactive.com│ Customer  │ Expired  │ Jan 1     │ -          │
└──────────────────────────────────────────────────────────────────┘
```

**Additional Features:**
- **Status Column:** Pending, Accepted, or Expired
- **Summary Dashboard:** See acceptance rates and key metrics
- **Filters:** By status, account type, or date range
- **Company Link:** Click to view accepted company accounts

**Why This Matters:**
Status tracking answers the critical question: *"What happened to my invitations?"* This enables you to prioritize follow-ups, measure success, and connect invitations to actual customers.

---

### Phase 3: Advanced Management (Future)
**Effort: Medium - Recommend waiting**

Additional features that could be added later based on usage:
- Resend invitation
- Revoke/cancel invitation
- Bulk upload via CSV
- Export to Excel
- Detailed analytics dashboard

**Why Wait:**
Phase 1+2 solves your immediate need. After 2-3 months of usage, you'll know which Phase 3 features would actually add value.

---

## Recommendation: Build Phase 1 + 2 Together

**Why Together:**
- Building separately creates rework (more expensive)
- Phase 1 alone only gives partial visibility
- Status tracking (Phase 2) makes the feature truly useful
- Complete solution in one delivery

**Effort Comparison:**
- Phase 1 only: **Small**
- Phase 1 + 2 together: **Small** *(most efficient)*
- Building Phase 2 later: **Small + rework overhead** *(less efficient)*

---

## Key Decisions Needed

### 1. Scope Confirmation
Do you want Phase 1 + 2 (recommended), or Phase 1 only?
**Our recommendation:** Phase 1 + 2 together

### 2. Invitation Expiration
After how many days should an unused invitation show as "Expired"?
**Options:** 7, 14, 30, 60 days, or never
**Our recommendation:** 30 days

### 3. Access Permissions
Who should view the invitation list?
**Options:**
- A) Only administrators
- B) Administrators + Customer Super Admins (for their subcustomer invites)
- C) Anyone who can send invitations

**Our recommendation:** Option B (mirrors current permissions)

### 4. Expired Invitation Behavior
Should expired invitation links stop working?
**Options:**
- A) Link still works, shows "Expired" in list (tracking only)
- B) Link stops working, must send new invitation

**Our recommendation:** Option A

---

## Success Criteria

After implementation, you will be able to:
1. ✅ View complete list of all invitations sent
2. ✅ Identify which invitations are pending, accepted, or expired
3. ✅ Track which invitations resulted in company accounts
4. ✅ Search and filter invitations quickly
5. ✅ Measure invitation acceptance rate
6. ✅ Navigate from invitation to company record
7. ✅ Identify invitations needing follow-up

---

## Comparison: What You Get

| Feature | Current | Phase 1 | Phase 1+2 | Phase 3 |
|---------|---------|---------|-----------|---------|
| Send invitations | ✅ | ✅ | ✅ | ✅ |
| View invitation list | ❌ | ✅ | ✅ | ✅ |
| Search invitations | ❌ | ✅ | ✅ | ✅ |
| See status | ❌ | ❌ | ✅ | ✅ |
| Track conversions | ❌ | ❌ | ✅ | ✅ |
| Filter by status | ❌ | ❌ | ✅ | ✅ |
| Summary metrics | ❌ | ❌ | ✅ | ✅ |
| Link to company | ❌ | ❌ | ✅ | ✅ |
| Resend/revoke | ❌ | ❌ | ❌ | ✅ |
| Bulk upload | ❌ | ❌ | ❌ | ✅ |
| Export to Excel | ❌ | ❌ | ❌ | ✅ |

---

## Important Notes

**Low Risk:**
- ✅ All data already exists in the system
- ✅ No impact on existing invitation sending
- ✅ No changes to recipient experience

**Items to Note:**
- Historical invitations (sent before this feature) will show as "Pending" by default
- Status tracking relies on matching invitation email to registration email (works in 95%+ of cases)
- Customer Super Admins will only see invitations they sent

---

## Summary

**The Problem:** No visibility into company invitations you've sent.

**The Solution:** Company Invitations dashboard with complete visibility and status tracking.

**Our Recommendation:**
- ✅ **Build Phase 1 + 2 together** (Small effort)
- ⏸️ **Wait on Phase 3** (evaluate after 2-3 months)

**Your Next Step:** Confirm you'd like to proceed with Phase 1+2 and answer the 4 key decisions above.

---

*Questions? Let us know and we'll clarify before moving forward.*
