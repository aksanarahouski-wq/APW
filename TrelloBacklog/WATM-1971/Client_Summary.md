# WATM-1971: Invoice Template Redesign — Client Summary

**Ticket:** WATM-1971 — Invoice Template Redesign
**Status:** Ready for Development
**Parent:** WATM-1964: APW Support 2026
**PRD:** [Invoice Template Redesign - PRD](https://orases.atlassian.net/wiki/spaces/WATM/pages/2713255971/Invoice+Template+Redesign+-+PRD)
**Jira:** [WATM-1971](https://orases.atlassian.net/browse/WATM-1971)

---

## Why

Feedback from the January 13, 2026 meeting indicated that the current invoice format feels "too crowded" and hard to read. Customers and account managers spend more time than necessary reviewing invoices, and the layout doesn't look as polished as it could.

## What's Changing

1. **Organized by Service Plan** — Line items will be grouped under clear section headers (e.g., "Tier 1", "Tier 3", "Super Tier") instead of appearing in one long flat list. This makes it easy to see at a glance which devices are on which plan.

2. **Shorter, cleaner descriptions** — Today each line item repeats the full service plan name and data limit. In the new format, that information moves to the section header, and each line item just shows the usage amount and any extras (e.g., "<1.50GB + Dual" instead of "Tier 3 (Up to 10GB), used less than 1.50 GB + Dual Sim Device Fee").

3. **More compact layout** — Reduced whitespace and tighter spacing so that typical accounts (under ~25 devices) fit on a single page instead of spilling onto two.

## What's NOT Changing

- All pricing, totals, and quantities stay exactly the same
- Billing method information stays the same
- Footer notes (credit card fees, minimum payment fees, Dual SIM fees) stay the same
- How invoices are delivered or accessed stays the same
- No changes to billing calculations or data

## What We'll Deliver for Review

Before going live, we'll provide sample invoices for approval:

- A multi-plan account (showing how grouping looks across different service tiers)
- A single-plan account (showing the clean layout when all devices are on the same tier)
- A large account (100+ devices, showing that the format scales well)

## What's Out of Scope

- No changes to the customer portal invoice view (this is the PDF only)
- No changes to invoice email templates
- No new data fields or metrics on the invoice
- No changes to how past invoices look (only new invoices going forward)
