# Additional Cellular Backup Service Plan Requirements - Discovery

**Date:** July 20, 2026
**Duration:** ~73 minutes
**Attendees:**
- Aksana Rahouski (Orases, Senior PM/BA -- Organizer)
- Noah Bratzel (Orases, Developer)
- Richard Sacco (Orases, Developer)
- Aaron Diefes (Orases, Developer)
- Adam Curcie (APW, Technical Lead)
- Devon D'Andrea (APW, Operations)

**Recording:** https://tldv.io/app/meetings/6a5e45fe6a784400139ec50b

---

## Meeting Purpose

This discovery session focused on understanding APW's requirements for a new "Flexible Failover" cellular backup service plan offering. The goal was to walk through how customers would transition devices from Pay Per Use (PPU) plans to Fixed Wireless Access (FWA) plans when enabling cellular backup, including the multi-step technical process, carrier-specific differences, and the new pooled plan monitoring needs. Adam and Devon presented the business vision and technical workflow for the first time to the development team.

---

## Key Topics Discussed

### 1. Current Cellular Backup Limitations
- Today, cellular backup configuration has no impact on the device's carrier-side service plan -- the device stays on whatever PPU tier it is already on.
- For customers with low-usage backup needs (3, 7, or 10 GB monthly on tiers 1-3), the current PPU model works fine.
- Problem: For sustained outages or high-data-usage locations, PPU costs become extremely expensive (e.g., over $1,000/month in overage scenarios).

### 2. New "Flexible Failover" FWA Service Plan
- APW wants to create a new FWA-type service plan called "Flexible Failover" specifically for cellular backup use cases.
- **Two-tier pricing model:** A base rate (~$25) covering up to ~2 GB of data, and an unlimited tier (~$90-100) that automatically kicks in if usage exceeds the threshold.
- At the end of each billing cycle (the 11th), the device resets back to the base/low-data plan automatically.
- This provides predictable pricing for customers: "If you don't have an outage, it's $20. If you do, it's $100." No surprises.
- The plan would be restricted to cellular backup use cases, not offered for primary internet devices (though APW could manually assign it if needed).

### 3. PPU-to-FWA Transition Process (Verizon)
- A multi-step, time-sensitive process requiring the device to be online:
  1. Customer selects cellular backup and chooses between staying on PPU or switching to Flexible Failover.
  2. Customer configures cellular backup settings (existing configuration options remain unchanged).
  3. Customer acknowledges the time requirement (15-30 minutes).
  4. Portal pings device to confirm it is online.
  5. Portal sends new configuration to device; waits for acknowledgment.
  6. Device reboots.
  7. Portal deactivates the ICCID on the PPU account via ThingSpace API (~5 min).
  8. Portal activates the ICCID on the FWA account with the specified service plan (~5 min).
  9. Device reconnects (~5 min after activation).
  10. Device check-in confirms success.
  11. Customer receives email/text notification of completion.
- If the device has a secondary/backup SIM, that SIM will also be deactivated.
- The device needs to be in a "frozen" or locked status during transition to prevent duplicate actions.

### 4. Carrier-Specific Differences
- **AT&T:** Does not offer unlimited plans. Customers with AT&T-only SIMs cannot use Flexible Failover but should still see a disclaimer/notification that the option exists (with instructions to contact APW for alternatives).
- **Verizon:** Full support for PPU-to-FWA transition. FWA plans are on a separate Verizon account. This is the MVP carrier.
- **T-Mobile:** Has unlimited plans but they are NOT flexible -- once a SIM is placed on an unlimited T-Mobile plan, it cannot be moved off. This makes T-Mobile impractical for the Flexible Failover concept. T-Mobile has extensive pooled plans (1 GB through 300 GB) that could be leveraged differently in the future.
- **Dual-SIM devices:** If a device has a Verizon SIM and wants Flexible Failover, the secondary SIM must be deactivated; the device becomes Verizon-only.

### 5. Mid-Cycle Transition Billing Problem
- When a device transitions from PPU to FWA mid-billing-cycle, any data usage already consumed on the PPU account does NOT transfer to the FWA account.
- Verizon invoices the two accounts separately -- APW still owes for PPU usage that cycle.
- The portal needs to show the customer their current PPU usage and require them to consent/acknowledge that those charges will appear on their final PPU invoice.

### 6. Bidirectional Transitions (FWA back to PPU)
- Customers should be able to move back from FWA to PPU.
- However, it makes no financial sense to do this mid-cycle because the customer already owes $90 for the FWA plan that month.
- The portal should recommend/enforce that downgrades happen at billing cycle boundaries.
- If a customer deactivates a SIM on an FWA plan, they still pay the $90 for that month -- this must be clearly communicated.

### 7. Pooled Plan Monitoring (Future / MVP Foundation)
- FWA plans on Verizon are pooled -- all devices on a given pool plan share the total data allotment.
- **MVP:** Monitor individual device usage against the ~2 GB threshold to determine when to bump to unlimited. Full pool-level monitoring is not required yet.
- **Future:** Build pool monitoring to track total usage across all devices vs. total pool allotment. Then break pools into per-customer "micro pools." Eventually optimize pool assignments for profitability.
- This same mechanism will later extend to T-Mobile's pooled plans.

### 8. Business Context -- MSP Sales Channel
- APW's target customer for this feature is a company whose core business is hardware sales to MSPs (Managed Service Providers).
- MSPs will resell cellular backup to their business clients.
- Pricing must be dead simple: "$20 or $100" -- no complicated explanations needed.

---

## Decisions Made

- **MVP scope is Verizon-only.** T-Mobile and AT&T cellular backup enhancements are deferred.
- **New FWA service plan ("Flexible Failover") will be created** with two tiers: base rate (~$25 for up to 2 GB) and unlimited (~$90-100). Resets to base tier each billing cycle.
- **Flexible Failover plan will be restricted to cellular backup use cases** (not offered as a general FWA plan option to all customers).
- **T-Mobile unlimited plans will NOT be supported for Flexible Failover** because T-Mobile locks SIMs to unlimited permanently.
- **Dual-SIM devices choosing Flexible Failover must deactivate the secondary SIM** and operate as Verizon-only.
- **AT&T devices will see an informational disclaimer** about Flexible Failover availability (contact APW), but cannot self-serve the transition.
- **Device must be placed in a locked/frozen status** during the PPU-to-FWA transition process to prevent concurrent modifications.
- **Asynchronous notification model:** Customer does NOT wait on screen for 30 minutes. They get an email/text when the transition completes.
- **Mid-cycle PPU usage must be disclosed and acknowledged** by the customer before executing the transition.
- **The four-week timeline Adam mentioned is NOT a hard deadline.** More like 6-8 weeks, ideally before September trade show season.

---

## Action Items

| Action Item | Owner | Due/Priority |
|---|---|---|
| Create a sample Flexible Failover FWA business plan in beta portal (tiers, pricing) for use in prototypes and requirements | Adam Curcie / Devon D'Andrea | Before next working session |
| Think through the requirements, identify gaps, and prepare follow-up questions | Aksana Rahouski | By next day's sync (July 21) |
| Confirm whether activation API calls to Verizon require/support specifying a service plan name | Richard Sacco | Next sync |
| Clarify exact timeline/priority for MVP delivery (soft deadline acknowledged as 6-8 weeks, before September trade shows) | Adam Curcie | Next sync |
| Schedule dedicated follow-up session if needed after initial requirements review | Aksana Rahouski | TBD |

---

## Technical Details

### PPU-to-FWA Transition Sequence (Verizon)
1. Portal pings device to confirm online status (prerequisite).
2. Portal sends new configuration file to device.
3. Device acknowledges config receipt.
4. Device reboots.
5. Portal calls ThingSpace API to **deactivate** ICCID on PPU account (~5 min callback).
6. Portal calls ThingSpace API to **activate** ICCID on FWA account with specified service plan name (~5 min callback).
7. Device reconnects and checks in (~5 min).
8. Portal sends email/text confirmation to customer.
- **Total estimated time:** 15-30 minutes.
- **Secondary SIM:** If present, also deactivated during this process.
- **Service plan name is required** in the activation API call (minimum required field).

### Billing Cycle Reset Logic
- On the billing cycle reset date (the 11th of each month), devices on the Flexible Failover plan are automatically moved back to the base/low-data tier.
- The portal monitors per-device usage: if a device exceeds the ~2 GB threshold during a cycle, it gets bumped to the unlimited tier ($90-100).

### Carrier API Considerations
- Verizon PPU and FWA are **separate accounts** -- deactivation and activation are separate API operations.
- The portal currently does not specify service plans during activation; this will need to be added.
- ThingSpace callback reliability is a factor -- "worst case" delays possible on bad days.

---

## Open Questions / Follow-ups

1. **Exact pricing for the Flexible Failover tiers** -- APW to create sample plan in beta. Base rate TBD (~$20-25), unlimited rate TBD (~$90-100). Verizon's cost to APW for the 2 GB pool plan is ~$20.
2. **Exact usage threshold for tier bump** -- 1 GB or 2 GB? To be confirmed with the service plan details.
3. **How does the portal determine the right service plan name for the FWA activation API call?** Richard noted the current PPU activation uses a static/default plan name; FWA will need the correct plan specified.
4. **What happens if the transition process fails at any step?** Rollback strategy not yet defined. Adam noted that individual steps are not device-bricking, but failure handling needs design.
5. **Bidirectional transition timing enforcement** -- Should the portal block FWA-to-PPU downgrades mid-cycle, or just warn/inform?
6. **T-Mobile pooled plan support** -- Deferred, but the pool monitoring mechanism built for Verizon MVP should be designed with T-Mobile extensibility in mind.
7. **AT&T disclaimer wording** -- Exact messaging for AT&T-only devices seeing the Flexible Failover option needs to be determined.
8. **Mid-cycle PPU usage billing** -- Exact mechanism for rolling PPU charges into the final invoice or presenting them to the customer needs further design.
9. **What does the customer-facing UI flow look like?** Adam described a multi-step wizard (service plan choice, configuration, acknowledgment, execution). Needs formal UX design.
10. **Will the "frozen" device status during transition be a new status, or reuse an existing one?** Technical design needed.
