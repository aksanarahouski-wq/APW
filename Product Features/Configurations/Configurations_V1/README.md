# Configuration Engine V1

Replaces the legacy file-based (.DAT) configuration system with a schema-driven parameter management engine. Layered inheritance model: Schema Defaults -> Model Defaults -> 3-Way Rules -> Company Override Sets -> Device Overrides.

**Status:** In progress — requirements finalized, prototype built, client feedback incorporated.

**Lovable Prototype:** https://device-heartbeat-hub.lovable.app/admin/config-mgmt
Subpages: Schema Concept, Model Defaults, Three-Way Rule, Company Overrides.

## Files

| File | Purpose |
|------|---------|
| `Requirements_Config_Builder.md` | **Primary document.** Full requirements spec for Phase 1 — inheritance hierarchy, override sets, resolution logic, conflict detection, versioning & approval workflow, change log, UI screens, and all design decisions. Start here. |
| `Requirements_Config_Push.md` | Phase 2 requirements — config delivery to devices, enhanced push model, hostname verification, per-device push status tracking, retry strategy, .DAT compilation. |
| `Current_Config_System_Summary.md` | Reference doc — how the legacy config system works today. Hierarchy, delivery mechanism, key code locations, database tables, and known limitations. |
| `Resolution_Logic_Diagram.html` | Interactive Mermaid diagram showing the 5-level resolution logic. Open in a browser. |
| `Open_Questions.md` | Tracked open questions needing client confirmation (Adam/Devon) and answered questions from Confluence comments not yet applied to requirements. |

## Folders

| Folder | Contents |
|--------|----------|
| `Lovable prototype/` | Prototype build prompts. `Prototype_Build_Prompts.md` = original prompts covering all screens. `Prototype_Prompts_2026-07-19_BuildAssignSeparation.md` = update prompts for the Build vs. Assign override set redesign. |
| `DataSamples/` | Real .DAT config files for analysis. `Verizon_I-22_Sp1-Sp2-Sp3/` contains a base 3-way rule and three company override files (Altech, Cord, Baum) with an `Override_Analysis.md` documenting parameter diffs. |
| `Client Emails/` | Email drafts and client correspondence related to config V1 design. |
| `Archive/` | Superseded documents. `Questions_and_Clarifications.md` was replaced by resolved items folded into `Requirements_Config_Builder.md`. `Distributor_SubCompany_Override_Discussion.md` decisions were migrated into Requirements_Config_Builder.md Level 4 section. |

## Phases

| Phase | Scope | Status |
|-------|-------|--------|
| **Phase 1** | Config Builder — schema, model config, 3-way rules, override sets, resolution logic, versioning & approval, device preview | In progress |
| **Phase 2** | Config Push & Delivery — enhanced push with check-in fallback, on-demand manual push, hostname verification, per-device status tracking, retry strategy | Requirements defined |
