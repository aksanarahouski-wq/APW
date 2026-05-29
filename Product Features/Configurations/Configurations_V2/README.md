# Configuration Engine V2

Replaces the legacy file-based (.DAT) configuration system with a schema-driven parameter management engine. Layered inheritance model: Schema Defaults → Model Defaults → 3-Way Rules → Company Override Sets → Device Overrides.

**Status:** In progress — requirements finalized, prototype built, client feedback incorporated.

## Files

| File | Purpose |
|------|---------|
| `Requirements_Config_Builder.md` | **Primary document.** Full requirements spec for Phase 1 — inheritance hierarchy, override sets, resolution logic, conflict detection, versioning & approval workflow, change log, UI screens, and all design decisions. Start here. |
| `Requirements_Config_Push.md` | Phase 2 requirements — config delivery to devices, enhanced push model, hostname verification, per-device push status tracking, retry strategy, .DAT compilation. |
| `Outstanding_Client_Questions.md` | Open questions needing client validation (Adam/Devon) before dev work begins. Also contains decisions made during the April 2026 internal review that are documented in Requirements_Config_Builder.md. |
| `Current_Config_System_Summary.md` | Reference doc — how the legacy V1 config system works today. Hierarchy, delivery mechanism, key code locations, database tables, and known limitations. |
| `Prototype_Build_Prompts.md` | Step-by-step prompts used to build the UI prototype in Lovable. Covers Grand Schema, Model Configuration, 3-Way Rules, Override Sets, Device Preview, and Config Push screens. |
| `Resolution_Logic_Diagram.html` | Interactive Mermaid diagram showing the 5-level resolution logic. Open in a browser. |

## Folders

| Folder | Contents |
|--------|----------|
| `Meetings/` | Client meeting transcripts and summaries. `Meeting1.md` = full transcript (Mar 27, 2026). `Meeting1_Summary.md` = key decisions, action items, and open questions from that meeting. |
| `Archive/` | Superseded documents. `Questions_and_Clarifications.md` was replaced by `Outstanding_Client_Questions.md` + resolved items folded into `Requirements_Config_Builder.md`. `Distributor_SubCompany_Override_Discussion.md` decisions were migrated into Requirements_Config_Builder.md Level 4 section. |

## Phases

| Phase | Scope | Status |
|-------|-------|--------|
| **Phase 1** | Config Builder — schema, model config, 3-way rules, override sets, resolution logic, versioning & approval, device preview | In progress |
| **Phase 2** | Config Push & Delivery — enhanced push with check-in fallback, on-demand manual push, hostname verification, per-device status tracking, retry strategy | Requirements defined |
