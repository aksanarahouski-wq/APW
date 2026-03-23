# Distributor / Sub-Company Override Set Inheritance — Discussion Notes

Captured from working sessions (March 2026) discussing how company override sets interact with the distributor → sub-company hierarchy.

---

## Background

In WATM, companies can be distributors with hundreds of sub-companies (e.g., Miele). Today, the legacy system handles this with a `apply_to_children` flag on custom company configurations — if checked, all sub-companies inherit the parent's config.

From Meeting 2, Adam described the current workflow:
- Build a config for Miele (the distributor)
- Check a box that says sub-companies inherit it
- All of Miele's hundreds of sub-companies get that same config
- If one sub-company needs something different, they manually build a separate config for that sub-company
- Adam: *"We should have backwards accountability on that, where if we do wanna unlink it on a specific company, we can."*

Devon called out the Miele example specifically — Miele has hundreds of sub-customers that need to inherit the same config, and this inheritance is critical to the workflow.

---

## Options Considered

### Option A: Explicit Assignment Only (simple)

- When you create an override set for Miele, you explicitly assign Miele AND all of Miele's sub-companies to that override set
- No automatic inheritance — the admin picks all the companies
- If a sub-company needs something different, create a different override set and assign just that sub-company
- **Pro:** Simple, no inheritance logic needed
- **Con:** If Miele gets a new sub-company, someone has to remember to assign it to the override set

### Option B: Inherit from Parent with Unlink (matches current system) ← SELECTED

- When you assign an override set to a distributor, there's an "Apply to sub-companies" flag
- If enabled, all sub-companies automatically inherit the parent's override set assignment
- Any individual sub-company can be unlinked — removing the inherited override set
- If a new sub-company is added under the distributor, it automatically gets the parent's override set
- **Pro:** Matches the current workflow Adam is used to, handles new sub-companies automatically
- **Con:** Adds inheritance logic to the override set assignment

**Decision: Option B selected** — it matches the existing workflow and handles the automatic inheritance for new sub-companies, which is important for distributors with hundreds of subs.

---

## How It Works

```
Override Set: "Miele Network Config" (scoped to I-22 + Verizon + [ATM])
├── Assigned to: Miele (distributor)
│   └── Apply to sub-companies: YES
│
├── Inherited by: Miele Sub-Co 1 (automatic)
├── Inherited by: Miele Sub-Co 2 (automatic)
├── Inherited by: Miele Sub-Co 3 (automatic)
├── UNLINKED: Miele Sub-Co 4 → assigned to "Miele Sub-Co 4 Custom" override set instead
├── UNLINKED: Miele Sub-Co 5 → no override set (falls back to 3-way rule)
└── Inherited by: Miele Sub-Co 6 (automatic, newly added)
```

---

## Decisions Made

1. **Unlinked sub-company behavior:** When a sub-company is unlinked from the parent's override set, it falls back to the 3-way rule values (no company-level override), unless it has its own separate override set assigned.

2. **Re-linking supported:** An admin can put a sub-company back on the parent's override set after it was previously unlinked.

3. **Distributor vs. non-distributor UI:** Distributor override sets may need a different view than non-distributor override sets, because distributors need visibility into sub-company inheritance status — who's inheriting, who's unlinked, who has their own override set.

---

## UI Implications

The override set editor's company assignment panel needs to handle the distributor case differently:

**For non-distributor companies:** The existing flat two-panel layout works (available companies ↔ assigned companies).

**For distributor companies:** The assignment view needs a hierarchical display:
- The distributor is directly assigned
- Below it, show all sub-companies with their inheritance status:
  - "Inheriting" — using the parent's override set (automatic)
  - "Unlinked" — not using this override set (falls back to 3-way rule)
  - "Own Override" — unlinked and assigned to a different override set (show which one)
- Unlink/re-link controls per sub-company

---

## Open Questions — RESOLVED

1. **Can a sub-company be assigned to a different override set while the parent's "apply to sub-companies" is on?**
   - **Answer: Yes.** Assigning a sub-company to its own override set effectively acts as an unlink from the parent's inheritance. The sub-company's explicit assignment wins over the parent's inheritance. This is consistent with how unlink already works — if you can unlink a sub-company (which falls back to the 3-way rule), you can also unlink and assign a different override set. Every company — whether a subsidiary or standalone — can have its own dedicated custom configuration.
   - Status: **Confirmed** (March 2026)

2. **What if a distributor has sub-companies on different 3-way rules?**
   - **Answer: The override set is scoped to a specific 3-way rule, so only sub-companies whose devices use that same 3-way rule would be eligible for inheritance.** Sub-companies on a different 3-way rule are unaffected by this override set. The inheritance only flows within the scope of the 3-way rule the override set targets.
   - Status: **Confirmed** (March 2026)

3. **Nested distributors — can a sub-company itself be a distributor with its own sub-companies?**
   - **Answer: No nested distributors.** The system does not have nested distributor hierarchies. Inheritance is one level only: distributor → direct sub-companies.
   - Status: **Confirmed** (March 2026)

---

## UI Design: Distributor Override Set Management

### The Core Challenge

The distributor case adds complexity to the override set's company assignment panel. A non-distributor company is a simple assign/unassign. A distributor company brings along potentially hundreds of sub-companies, each with their own inheritance status.

### Proposed UI Approach

#### Override Set Editor — Company Assignment Panel (Distributor View)

When a distributor is assigned to an override set with "Apply to sub-companies" enabled, the assignment panel shows a hierarchical view:

```
┌─────────────────────────────────────────────────────────────┐
│ Companies Assigned to "Miele Network Config"                │
│ (Scoped to: I-22 + Verizon + [ATM])                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ▼ Miele (Distributor)                    [Remove]          │
│    ☑ Apply to sub-companies                                 │
│    ┌───────────────────────────────────────────────────────┐│
│    │ Sub-Company          │ Status       │ Action          ││
│    │──────────────────────│──────────────│─────────────────││
│    │ Miele Sub-Co 1       │ ● Inheriting │ [Unlink]        ││
│    │ Miele Sub-Co 2       │ ● Inheriting │ [Unlink]        ││
│    │ Miele Sub-Co 3       │ ● Inheriting │ [Unlink]        ││
│    │ Miele Sub-Co 4       │ ○ Unlinked   │ [Re-link]       ││
│    │                      │   → "Sub-Co 4 Custom" override ││
│    │ Miele Sub-Co 5       │ ○ Unlinked   │ [Re-link]       ││
│    │                      │   → (no override, 3-way rule)  ││
│    │ Miele Sub-Co 6       │ ● Inheriting │ [Unlink]        ││
│    └───────────────────────────────────────────────────────┘│
│    Sub-companies: 4 inheriting, 2 unlinked                  │
│                                                             │
│  ─ Loaded ATMs (standalone)               [Remove]          │
│  ─ Company C (standalone)                 [Remove]          │
│                                                             │
│                              [+ Assign Company]             │
└─────────────────────────────────────────────────────────────┘
```

#### Key UI Behaviors

1. **Distributor row is expandable/collapsible.** Collapsed shows summary count ("4 inheriting, 2 unlinked"). Expanded shows the full sub-company table.

2. **Status indicators:**
   - **● Inheriting** (green) — sub-company uses this override set via parent inheritance
   - **○ Unlinked** (gray) — sub-company does not use this override set
     - If unlinked sub-company has its own override set, show which one (clickable link)
     - If no override set, show "(no override, falls back to 3-way rule)"

3. **Unlink action** removes the sub-company from inheritance. The sub-company falls back to the 3-way rule values unless it has its own override set assigned elsewhere.

4. **Re-link action** puts the sub-company back on the parent's override set.

5. **Assigning a sub-company to a different override set** (done from the other override set's assignment panel or the company page) automatically unlinks it from the parent's override set here. The status updates to "Unlinked → [name of other override set]".

#### Company Edit Page — Override Set Assignment (Distributor View)

From the company page, the admin sees which override sets are active:

```
┌─────────────────────────────────────────────────────────────┐
│ Configuration Overrides for: Miele                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 3-Way Rule                  │ Override Set        │ Status  │
│ ────────────────────────────│─────────────────────│──────── │
│ I-22 + Verizon + [ATM]      │ Miele Network Config│ Direct  │
│                              │ ☑ Applied to subs   │         │
│ 4100 + Verizon + [Standard] │ (none)              │ —       │
│                                                             │
│                     [+ Assign Override Set]                  │
└─────────────────────────────────────────────────────────────┘
```

#### Company Edit Page — Override Set Assignment (Sub-Company View)

For a sub-company, the page shows whether it's inheriting or has its own:

```
┌─────────────────────────────────────────────────────────────┐
│ Configuration Overrides for: Miele Sub-Co 4                 │
│ Parent: Miele (Distributor)                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ 3-Way Rule                  │ Override Set        │ Source  │
│ ────────────────────────────│─────────────────────│──────── │
│ I-22 + Verizon + [ATM]      │ Sub-Co 4 Custom     │ Direct  │
│                              │ (Parent: Miele      │         │
│                              │  Network Config)    │         │
│                              │ [Use Parent's] [Change]      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Key elements for sub-company view:
- Shows the parent's override set for reference (grayed out)
- **[Use Parent's]** button re-links to the parent's override set
- **[Change]** lets them pick a different override set
- If currently inheriting, shows "Inherited from: Miele" with an **[Override]** button to assign a different set

---

## What Still Needs to Be Done

- [x] Answer the three open questions above
- [ ] Update Requirements.md with the distributor inheritance model under Level 4 (Company Overrides)
- [ ] Update Questions_and_Clarifications.md — mark distributor questions as resolved
- [ ] Add prototype prompt for the distributor override set view (hierarchical sub-company display with unlink/re-link controls)
