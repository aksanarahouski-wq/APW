# Configuration Engine V1 — Open Questions & Decisions

Consolidated tracking of all open questions, client answers, and design decisions. This is the single source for question status — the requirements doc references this file but does not duplicate the questions.

Last Updated: July 19, 2026

---

## Open — Needs Client Input

### Q1: Parameter source for "Model = Any" override sets

**Context:** When an admin creates an override set scoped to a specific model (e.g., I-22), the parameter list comes from that model's selected parameters. But when the override set scope is "Any" model, what parameters are available to choose from?

**Options:**
- **A: Schema parameters flagged `available_at_company`** — Only parameters the schema designates as appropriate for company-level overrides. This is what the current requirements doc specifies.
- **B: Union of all model parameter lists** — Every parameter that appears in at least one model's selected list. Broader than A.
- **C: Full schema** — Show everything, let silent filtering drop irrelevant parameters at resolution time.

**Related:** Adam's Confluence comment on question #7 said "the preview should include values regardless if they are relevant to the model or not" — but that may refer to the display/preview, not to which parameters are selectable when building the override set. Need to confirm these are two separate concerns.

**Ask Adam:** When building a "Model = Any" override set, which parameters should the admin be able to select from?

---

### Q2: Override set editor — model applicability indicators

**Context:** When an override set is scoped to "Any" model and an admin adds a parameter like `io_enable`, that parameter only exists on I-22 devices, not on 4100s. At resolution time, devices whose model doesn't have that parameter silently ignore it.

**Ask Adam/Devon:** Should the override set editor show which models each parameter applies to? For example, a note like "Applies to: I-22, I-52" next to `io_enable`. Or is the device-level config preview sufficient for understanding what actually lands on a device?

---

## Open — Needs Internal Design Decision

### Q3: Config Builder + Config Push integration

**Context:** The requirements are currently separated into Phase 1 (Config Builder) and Phase 2 (Config Push & Delivery). However, without understanding how configs will be pushed to devices, we cannot fully validate the building piece. The two phases need to be considered together.

**Key dependencies:**
- Compilation timing (at push time vs. pre-compile on change) affects whether the builder needs to track compiled state
- Push status affects the device page UI (which is part of the builder scope)
- The legacy coexistence model spans both phases
- Hostname versioning (push concern) affects how we validate configs landed correctly (builder concern)

**Decision needed:** Should we merge the Phase 1 and Phase 2 requirements into a single document, or at minimum review them together as a unified design before development starts?

---

## Resolved — Applied to Requirements Doc

These decisions have been applied inline to `Requirements_Config_Builder.md`. Preserved here for reference.

### Migration path
**Decision:** Migration is manual, one company/device at a time. Existing configs stay as-is until rebuilt in V1. No automated migration tooling for MVP.
**Source:** Adam, Confluence (Mar 27 + Apr 16 2026). Internal decision (Jul 19 2026) to treat as settled for MVP.

### Mandatory parameter enforcement
**Decision:** Enforcement happens at the time MCS Rules are published.
**Source:** Adam, Confluence (Apr 17 2026)

### Schema versioning / parameter removal
**Decision:** Schema is append-only for MVP. Parameters cannot be removed. If removal is ever needed, client will consult with dev team.
**Source:** Adam, Confluence (Apr 17 2026)

### Cellular backup as matching dimension
**Decision:** Not a matching dimension. APW stopped using that flag. 3-way rule (Model + Carrier + Service Plan) is confirmed correct.
**Source:** Adam, Confluence (Apr 17 2026)

### Conflict detection UX
**Decision:** Override sets should cross-reference against all other overrides so the system can identify which sets can/cannot be used for the same companies. The Build vs. Assign separation (Version 6) addresses the UX: validation at assignment time with per-company blocking, and post-edit re-validation with flagging (not blocking).
**Source:** Adam, Confluence (Apr 17 2026). Internal design (Jul 19 2026).

### Soft conflicts
**Decision:** No. Strict rejection confirmed. No soft overrides.
**Source:** Adam, Confluence (Apr 17 2026)

### Override set browse/filter UI
**Decision:** Current proposal accepted. Browse page includes assignment status column and conflict indicators (added in Version 6).
**Source:** Adam, Confluence (Apr 17 2026)

### Distributor inheritance granularity
**Decision:** Show effective reach as **companies**, not devices. "12 of 47 companies" is sufficient.
**Source:** Adam, Confluence (Apr 17 2026)

### Auto-recompile notification
**Decision:** Yes, admin should see a change summary. Should also have historical changes in a change log / audit trail.
**Source:** Adam, Confluence (Apr 17 2026)

### Override sets per company
**Decision:** A company CAN have multiple override sets, as long as no parameter overlap between sets for overlapping scopes.
**Source:** Adam, Confluence (Mar 27 2026)

### Override set scope — "Any" option
**Decision:** Override sets need "Any" option for Model, Carrier, and Service Plan selectors.
**Source:** Adam, Confluence (Mar 27 2026)

### Legacy source selector
**Decision:** Keep legacy as a disabled fallback — don't remove it. In case of unforeseen issues, client wants ability to re-enable.
**Source:** Adam, Confluence (Mar 27 2026)

### Parameter import
**Decision:** Initial ~700 schema parameters and initial rules will be seeded directly into the database by the dev team, not entered through the UI.
**Source:** Adam, Confluence (Mar 27 2026)

### Company config linking
**Decision:** Override sets replace company-to-company linking. One set, many companies. Edits propagate automatically.
**Source:** Internal design decision.

### Sub-company unlink behavior
**Decision:** See Distributor Sub-Company Inheritance under Level 4 in requirements doc.
**Source:** Internal design decision.

### Override set cross-3-way-rule reuse
**Decision:** Override sets have their own scope with "Any" option. Not tied to specific 3-way rules.
**Source:** Internal design decision (Version 2 changelog).

### `available_at_company` flag
**Decision:** Keep the flag. It serves both as a guardrail and as the parameter source for model-agnostic override sets.
**Source:** Internal design decision.

### Build vs. Assign separation
**Decision:** Override set lifecycle split into build phase (no validation) and assign phase (per-company conflict validation). Edits to assigned sets trigger re-validation with flagging, not blocking. See Version 6 changelog.
**Source:** Internal design review (Jul 19 2026), informed by Altech/Cord/Baum override analysis.
