# T-Mobile Business Account Discovery - Summary

**Date:** May 4, 2026, 3:00 PM ET
**Duration:** ~43 minutes
**Recording:** [tl;dv Link](https://tldv.io/app/meetings/69f8ecb88155de00141b6344)

## Attendees
- **Aksana Rahouski** (Orases) - Organizer
- **Aaron Diefes** (Orases)
- **Noah Bratzel** (Orases)
- **Adam Curcie** (APW / Allpoint Wireless)
- **Devon D'Andrea** (APW / Allpoint Wireless)

---

## Topics Covered

### Verizon FWA - Remaining Items
1. **Fix "unlimited" display logic** — Only display "unlimited" for tier values >= 100 gigabytes. Must match on both limit value AND unit (100 megabytes should NOT show as unlimited). Only applies to FWA-type service plans; traditional PPU plans should still display their actual limits.
2. **Scope unlimited display to FWA plans only** — Traditional plans (e.g., "ATM Unlimited") should still show their actual limits, not "unlimited."
3. **Remove "Service Plan Usage Limit" field from FWA service plans** — This field is not applicable to FWA plan types and should not appear.

### T-Mobile Business Account Overview
- T-Mobile FWA is the same account type as regular T-Mobile SIMs -- no separate API endpoint needed
- Portal cannot differentiate between FWA and non-FWA devices via API; same differentiator applies
- T-Mobile SIM cards cannot be set to FWA plan via API or manual process currently
- All T-Mobile SIM cards use a single rate plan; differentiation managed via device groups or tags
- T-Mobile FWA plan costs $75/month for unlimited data with soft one-terabyte maximum
- T-Mobile reserves right to penalize network abuse exceeding one terabyte monthly usage
- SIM card plan changes require email request to T-Mobile; they confirm via email response

### T-Mobile Portal Implementation
- Add rate plan type flag to T-Mobile devices to specify PPU or FWA designation
- "Rate plan type" terminology more accurate than "provider account type" for T-Mobile
- PPU should be the default rate plan type when no selection is made
- T-Mobile device status changes use same API endpoint regardless of rate plan type
- Portal should allow changing rate plan type without restrictions for administrative clarity
- Rate plan type changes do not affect billing or device performance
- Only super admin access level can change T-Mobile rate plan type

### T-Mobile Service Plan Design
- T-Mobile FWA devices can use the same service plan structure as Verizon FWA devices
- Pricing tiers allow carrier-specific pricing modifications for T-Mobile and Verizon
- T-Mobile FWA devices restricted to FWA service plans only; cannot use pay-per-use plans
- Pay-per-use T-Mobile devices restricted to pay-per-use service plans only
- Verizon and T-Mobile share the same FWA service plan with different carrier-specific pricing

### T-Mobile Device Import and Assignment
- Update device import process to allow selecting PPU or FWA rate plan type for T-Mobile
- Device assignment and service plan assignment rules identical for T-Mobile and Verizon
- Validate that FWA T-Mobile devices can only be assigned FWA service plans

### Future Considerations
- Investigate whether T-Mobile can initiate callback to portal when SIM plan changes
- Potential future automation of T-Mobile FWA plan assignment via API if capability becomes available
- Verizon FWA plan automation theoretically possible but not currently planned

---

## Action Items
- [ ] **Noah:** Fix unlimited display — only show "unlimited" for values >= 100 GB (match on both limit AND unit); only on FWA plans
- [ ] **Noah:** Scope unlimited display to FWA plans only — traditional plans keep showing actual limits
- [ ] **Noah:** Remove "Service Plan Usage Limit" field from FWA service plan type
- [ ] **Team:** Create new T-Mobile business account Jira ticket with requirements
