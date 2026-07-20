# Cellular Backup Service Plans

**Jira:** [WATM-2206](https://orases.atlassian.net/browse/WATM-2206) | **Epic:** WATM-1964 (APW Support 2026) | **Status:** Draft | **Priority:** High
**Confluence:** [WATM-2206 Client Requirements](https://orases.atlassian.net/wiki/spaces/WATM/pages/3466264579)
**Discovery Recording:** [tl;dv - July 20, 2026](https://tldv.io/app/meetings/6a5e45fe6a784400139ec50b)
**Meeting Summary:** [Meetings/Summaries/2026-07-20_Cellular_Backup_Service_Plan_Discovery.md](/Meetings/Summaries/2026-07-20_Cellular_Backup_Service_Plan_Discovery.md)

---

## Summary

Enable customers to switch devices from Pay Per Use (PPU) plans to a new "Flexible Failover" Fixed Wireless Access (FWA) plan when enabling cellular backup. This gives customers predictable pricing for cellular backup: a low base rate (~$25) when there's no outage, and an automatic upgrade to unlimited (~$90-100) during sustained outages. The plan resets to base tier each billing cycle.

**Business driver:** APW's target customer is selling hardware to MSPs who resell cellular backup to business clients. Simple, predictable pricing ("$20 or $100") is critical for the sales channel.

**Client urgency:** Adam wants this completed before September trade show season. Realistic timeline is 6-8 weeks.

---

## What Exists Today

The current cellular backup feature (shipped mid-2025) provides:
- Device model capability flag (`device_models.is_cellular_backup_capable`)
- Enable/disable toggle on device view page
- Primary connection selection (Ethernet-only, WAN Ethernet + WiFi, WiFi + Ethernet)
- WiFi configuration (SSID, password, auth mode, encryption)
- Config push to device (APN, LAN1/WiFi WAN settings)

**What does NOT exist:**
- No service plan type selection in the cellular backup UI
- No SIM account swapping between Verizon PPU and FWA accounts
- No automated plan change (APC) logic based on usage thresholds
- No billing cycle reset logic
- No pooled plan monitoring

---

## What We Learned in Discovery (July 20, 2026)

The original ticket description from Adam described a generic PPU-to-Unlimited workflow. The discovery session revealed significantly more complexity and nuance. Key differences from the original ticket:

### Original Ticket vs. Discovery Reality

| Aspect | Original Ticket (WATM-2206) | Discovery Session |
|---|---|---|
| Plan naming | "PPU vs. Unlimited" | "Flexible Failover" -- a specific FWA plan with two tiers (base + unlimited) |
| Carrier scope | Verizon implied, unclear | **Verizon-only for MVP**. AT&T has no unlimited. T-Mobile locks SIMs permanently. |
| Account structure | Vague "B.I. account" | Two separate Verizon accounts (PPU account + FWA account) with separate API operations |
| Transition process | Simple ping + config push | 8-step process taking 15-30 min: ping, config push, reboot, deactivate PPU ICCID, activate FWA ICCID, reconnect, confirm, notify |
| Offline handling | Different paths for PPU vs Unlimited | **Device MUST be online** -- transition cannot proceed offline because it involves carrier account changes |
| Dual-SIM | Not mentioned | Secondary SIM must be deactivated; device becomes Verizon-only |
| Pricing model | PPU vs. flat Unlimited | Two-tier within FWA: base rate (~$25/2GB) auto-upgrades to unlimited (~$100) on usage, resets each cycle |
| Pooled plans | Not mentioned | FWA plans are pooled across devices; MVP needs per-device monitoring, future needs pool-level monitoring |
| Bidirectional | Not addressed | FWA-to-PPU supported but should be discouraged mid-cycle (customer still pays full FWA rate) |

### Carrier-Specific Constraints

| Carrier | Flexible Failover Support | Reason |
|---|---|---|
| **Verizon** | Yes (MVP) | Supports FWA pooled plans, SIM can move between PPU and FWA accounts |
| **AT&T** | No | Does not offer unlimited plans. Show disclaimer only. |
| **T-Mobile** | No | Unlimited plans lock SIMs permanently -- cannot reset to base tier |

---

## MVP Scope (Verizon Only)

### Core Workflow: PPU-to-FWA Transition

1. Customer enables cellular backup and selects "Flexible Failover" plan (or stays on PPU)
2. Customer configures cellular backup settings (unchanged from today)
3. Customer acknowledges 15-30 minute transition time
4. Portal validates device is online (ping)
5. Portal pushes new configuration; waits for device acknowledgment
6. Device reboots
7. Portal deactivates ICCID on PPU account (ThingSpace API, ~5 min)
8. Portal activates ICCID on FWA account with service plan name (ThingSpace API, ~5 min)
9. Device reconnects and checks in (~5 min)
10. Portal sends email/text confirmation

### Automated Plan Change (APC) Logic

- Monitor per-device data usage against ~2 GB threshold
- Auto-upgrade to unlimited tier when threshold exceeded (Verizon API call)
- Reset all Flexible Failover devices to base tier at billing cycle reset (the 11th)

### Device Status During Transition

- Device enters "frozen" or locked status during the 15-30 min transition
- Prevents concurrent modifications or duplicate transitions
- Status clears on successful completion or failure

### UI Requirements

- Multi-step popup/wizard for service plan selection during cellular backup setup
- Plan choice: "Stay on PPU" or "Switch to Flexible Failover"
- AT&T-only devices: Show disclaimer ("Contact APW for service plan options")
- Mid-cycle usage disclosure: Show current PPU usage, require customer acknowledgment
- Asynchronous completion: Customer does NOT wait on screen; receives email/text notification

### Billing Considerations

- Mid-cycle PPU usage charges remain on PPU account (Verizon invoices separately)
- Customer must acknowledge existing PPU charges before transitioning
- FWA-to-PPU mid-cycle: customer still pays full FWA monthly rate
- Deactivating FWA SIM mid-cycle: customer still pays full monthly rate

---

## Future Phases (Post-MVP)

### Pooled Plan Monitoring
- Track total data usage across all FWA devices vs. total pool allotment
- Per-customer "micro pools" within the larger shared pool
- Pool assignment optimization for cost efficiency

### T-Mobile Pooled Plans
- T-Mobile offers pooled plans from 1 GB to 300 GB
- Different mechanism than Verizon (no unlimited toggle, pool-based instead)
- Pool monitoring infrastructure from MVP should be designed with T-Mobile extensibility

### Bidirectional Transitions
- FWA back to PPU (supported but discouraged mid-cycle)
- Enforce or recommend timing transitions to billing cycle boundaries

---

## Open Questions

| # | Question | Status |
|---|---|---|
| 1 | Exact pricing for Flexible Failover tiers (base rate, unlimited rate)? | Waiting -- Adam/Devon creating sample plan in beta |
| 2 | Exact usage threshold for auto-upgrade: 1 GB or 2 GB? | Waiting -- depends on service plan details |
| 3 | Service plan name for FWA activation API call -- how does portal determine correct value? | Needs Richard to confirm ThingSpace API capabilities |
| 4 | Rollback strategy if transition fails at any step? | Not yet designed |
| 5 | Should portal block FWA-to-PPU downgrades mid-cycle, or just warn? | Not yet decided |
| 6 | AT&T disclaimer wording? | Not yet drafted |
| 7 | Exact mechanism for disclosing mid-cycle PPU charges to customer? | Needs design |
| 8 | Will the "frozen" device status be a new status or reuse an existing one? | Needs technical design |
| 9 | Customer-facing UI flow -- formal UX design? | Needs design |
| 10 | How to handle mid-cycle financial liability transfer (PPU charges on separate invoice)? | Needs design with Adam |

---

## Action Items

| Action Item | Owner | Due |
|---|---|---|
| Create sample Flexible Failover FWA business plan in beta portal | Adam Curcie / Devon D'Andrea | Before next working session |
| Review requirements, identify gaps, prepare follow-up questions | Aksana Rahouski | July 21, 2026 |
| Confirm ThingSpace API supports specifying service plan name on activation | Richard Sacco | Next sync |
| Clarify MVP timeline (soft target: 6-8 weeks, before Sept trade shows) | Adam Curcie | Next sync |
| Schedule follow-up session if needed | Aksana Rahouski | TBD |

---

## Related Resources

- **Existing Cellular Backup PRD:** [COMPLETE: Cellular backup](https://orases.atlassian.net/wiki/spaces/WATM/pages/1928069148)
- **Cellular Backup Feature Docs:** [Device Cellular backup](https://orases.atlassian.net/wiki/spaces/WATM/pages/2185199662)
- **Test Cases:** [Cellular Backup Feature - Comprehensive Test Cases](https://orases.atlassian.net/wiki/spaces/WATM/pages/2185461794)
