# Configuration Engine V1 — Questions and Clarifications

Questions and open items to discuss with the client (Adam/Devon) as we refine the V1 requirements.

---

## From Requirements Document (Original Open Questions)

1. **Parameter import:** Will the initial ~700 parameters be imported from Adam's spreadsheet, or manually entered via the Grand Schema screen?

2. **Migration path:** How do existing config files map to the new schema? Is there a migration, or do existing configs stay as-is until they're rebuilt in V1?

3. **Mandatory parameter enforcement:** When should mandatory-but-null parameters be flagged? At config creation time? At device provisioning time? Both?

4. **Company config linking:** ~~When Company A links to Company B's config, and Company B changes a value — does Company A get the change automatically, or is it a snapshot?~~ → Superseded by the one-to-many override set proposal (see "Company Override Sets" section below). If we go with named override sets, linking is no longer company-to-company — it's company-to-override-set.

5. **Sub-company unlink behavior:** When a sub-company unlinks from parent config, does it get a copy of the parent's current values, or start blank?

6. **Schema versioning:** If a parameter is removed from the schema, what happens to existing 3-way rules and company overrides that reference it?

---

## Company Overrides — Scoping and Eligibility

### Context

In the current system, company overrides are always relative to a specific 3-way rule (model + carrier + service plan) combination. A company doesn't override "any parameter" globally — it overrides values from the 3-way rule that applies to its devices.

This means the override flow is:

```
3-Way Rule (model + carrier + service plan) produces a set of values
    → Company override modifies specific values from THAT result
```

A company with devices across multiple 3-way rules (e.g., I-22+Verizon+ATM and 4100+T-Mobile+Standard) would have separate override sets for each combination.

### Questions

7. **Override eligibility within a 3-way rule:** Can a company override ANY parameter from the 3-way rule output, or is there still a subset restriction? Are there parameters an admin would never want a company to change, even if they exist in the 3-way rule?

8. **Need for `available_at_company` schema flag:** Given that company overrides are scoped to 3-way rule output (not the full schema), is the `available_at_company` flag on the schema still needed as a guardrail? Or is the 3-way rule scoping sufficient control?

9. **Multiple device groups per company:** If a company has 50 devices all on the same 3-way rule, does one company override apply to all 50? Or can they have different override sets for subgroups of devices within the same 3-way rule?

10. **What the admin sees on the override screen:** When editing a company override, does the admin see the fully compiled values (schema defaults + model defaults + 3-way rule values merged) and modify from that? Or do they only see what the 3-way rule explicitly set?

11. **Company override management screen flow:** Since overrides are per 3-way rule, the company override screen would need to show which 3-way rule combinations the company's devices use, then let the admin pick one and override values. Is this the expected workflow?

---

## Configuration Admin Space — What Belongs Where

### Decisions Made

- **Model Edit Page:** Only the "is configurable" checkbox. A simple flag indicating the model participates in the configuration engine.
- **Configuration Admin Section:** All other config-related management:
  - Grand Schema (parameter definitions, defaults, types, validation)
  - Model Configuration (per-model parameter selection + model default values)
  - 3-Way Rules (model + carrier + service plan value editor)

### Rationale

- Workflow coherence: admins building configurations work in one place (Schema → Model Config → 3-Way Rules)
- Separation of concerns: model admin owns model attributes, config admin owns everything configuration-related
- The model edit page serves many purposes already and would get bloated with ~700 parameter checkboxes

### Questions

12. **Company override UI placement:** Company overrides are managed on the Company Edit page (or a config tab within it). Should there also be a view from the Configuration Admin side that shows all company overrides across companies for a given 3-way rule? This would help admins see the full picture of who has overridden what.

13. **Navigation between spaces:** An admin setting up a 3-way rule may want to quickly see which companies use it and what overrides exist. How tightly should the Config Admin section and Company Admin section be linked?

---

## Company Override Sets — One-to-Many Proposal

### Background (from Meeting 2)

Devon stated the need directly: *"We need the ability to take a company override and be able to mimic it to other companies."* Adam described the Cord Financial example — Cord has custom firewall rules allowing traffic to their RMS servers. When another company (e.g., Loaded ATMs) needs the same access, today they manually build a duplicate config file. Adam suggested linking from the company page: a dropdown to select an existing company's config instead of retyping it, so changes to Cord's config propagate to linked companies.

### Current Problem

Today, company overrides are one-to-one. If 10 companies need the same firewall rules, you create 10 identical override entries. Changing one value means changing it in 10 places — exactly the kind of redundancy the configuration engine is supposed to eliminate.

### Proposed Approach: Named Override Sets

Instead of company-to-company linking (Company B uses Company A's config), we propose **named override sets** as their own entity:

```
Override Set: "Cord Financial RMS Access"
  - Scoped to a 3-way rule (e.g., I-22 + Verizon + ATM)
  - Contains: firewall table entries allowing Cord RMS traffic
  - Assigned to: Cord Financial, Loaded ATMs, Company C, Company D...
```

**How it works:**
1. An admin creates a named override set within the Configuration Admin space
2. The override set is scoped to a specific 3-way rule (it overrides values from that rule)
3. The override set is assigned to one or more companies
4. Editing the override set updates it for ALL assigned companies
5. A company can also have a unique override set (assigned to just that one company) — this covers the one-off case

**Why this is better than company-to-company linking:**
- The override set is its own entity — not "owned" by Cord. If Cord leaves, the override set still exists for other companies.
- Naming is descriptive ("RMS Firewall Access") rather than referential ("Cord's config").
- From the Config Admin space, you can see all override sets, who uses them, and edit them centrally.
- From the Company page, you can see which override sets are assigned and link/unlink.

**Where it's managed:**
- **Configuration Admin Section:** Create, edit, and browse override sets. See which companies are assigned to each. This is the primary management interface.
- **Company Edit Page:** View assigned override sets, link to existing ones, or create a new one. Adam was clear he wants the company page to be a touchpoint for this — and it still is, but as a consumer, not the owner.

### Sub-Company Inheritance

From the meeting, the Miele example: Miele has hundreds of sub-companies that need to inherit the same config. The override set model handles this naturally:

- Parent company is assigned an override set
- Sub-companies inherit the parent's assigned override set by default
- A sub-company can be unlinked and assigned a different override set (or none)
- This matches Adam's request: *"We should have backwards accountability on that, where if we do wanna unlink it on a specific company, we can."*

### Questions for Client

14. **Override set naming and ownership:** Are override sets always created in the Config Admin space, or can an admin create one directly from a company page? (We recommend Config Admin as the primary creation point, with company page as a linking/assignment interface.)

15. **Multiple override sets per company:** Can a company be assigned more than one override set for the same 3-way rule? (e.g., one for firewall rules, another for DNS settings) Or is it one override set per 3-way rule per company?

16. **Override set conflicts:** If a company has its own override set AND inherits one from a parent company, which takes precedence? Does the child's override set merge with or replace the parent's?

17. **Override set scope:** Is an override set always scoped to a single 3-way rule? Or could an override set apply across multiple 3-way rules? (e.g., "Cord RMS Access" might need to apply to I-22+Verizon+ATM and 4100+Verizon+ATM if Cord has both device types.)

18. **Audit trail:** When an override set changes, should there be a log of what changed and which companies were affected? Given that one change can propagate to many companies, this seems important for accountability.
