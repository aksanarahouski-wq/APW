# Configurations V2 — Meeting 2 Summary (Configs Sync, P2)

**Date:** April 16, 2026, 4:00 PM ET
**Duration:** ~70 minutes
**Organizer:** Aksana Rahouski
**Attendees:** Aksana Rahouski, Richard Sacco, Aaron Diefes (Orases) · Adam Curcie, Devon D'Andrea, Jon (APW)

---

## Overview

Follow-up deep dive on the Configuration V2 system design, picking up from the morning APW check-in. The team worked through the Confluence documentation, discussing schema management, company override mechanics, migration strategy, distributor/child company hierarchy, and validation rules. Key decisions included renaming "three-way rules" to "MCS pairs," confirming that schema parameters cannot be deleted once defined, and establishing a phased migration approach with a comparison tool for validation.

---

## Key Discussion Topics

### Configuration Builder — Core Components
- **Schema, model, and company overrides** confirmed as the three core layers
- A company can have **multiple override sets**, but the same parameter cannot exist in two overrides for the same company
- Override sets can be scoped to **specific models or "any model"**
- Company-level customization limited to ~20 parameters, controlled via schema availability flags
- Most override sets will contain **1–5 parameters**; firewall rules are the exception with potentially **40+ variations**

### Terminology Change
- **Rename "three-way rules" to "MCS pairs"** (Model–Carrier–Service Plan) across all documentation
- Team agreed this is clearer and more descriptive

### Migration & Testing Strategy
- **Legacy and new config systems will coexist** — users can choose which system to use during transition
- Migration starts with a **subset of pioneer devices** to validate new configs match current DAT files
- Early migration phase will require **significant manual effort** to set up schema, defaults, and MCS pairs
- Beta environment needs **40–100 devices** across multiple models, carriers, and companies for stress testing
- **Comparison tool needed** to preview new configs as key-value pairs against legacy DAT files for validation

### Parameter Management & Schema Design
- Schema parameters **cannot be removed once defined** — no delete button for parameters
- New parameters will be added when **firmware updates** require new functionality support
- Parameters marked as required must have values; **all schema fields** are sent to devices regardless of requirement status
- Validation must prevent required parameters from being set to null at any configuration level
- Parameter availability at company override level controlled through **schema flags**

### Distributor & Child Company Hierarchy
- Distributors with child companies need ability to **apply configs to all children with exceptions** for specific subsidiaries
- Child companies can be **unlinked** from parent override rules and assigned their own configuration sets
- Child companies can be **relinked** to parent rules after being independently configured
- System should validate that **conflicting override sets** cannot be assigned to the same company

### Validation & Conflict Management
- Validation approach should identify **which override sets conflict** with each other, rather than checking all company combinations
- **Firewall configuration** is the primary parameter causing multiple custom company configs due to company-specific access requirements

### Versioning Discussion
- Team discussed whether versioning is needed for model and schema levels
- Model changes occur **infrequently** — unclear if full versioning is necessary
- Consensus: parameters should **not be removed**, reducing the need for complex versioning

---

## Action Items

| Who | What | When |
|-----|------|------|
| Adam | Send sample company configuration files to Aksana for analysis | Soon |
| Aksana | Rename "three-way rules" to "MCS pairs" across all documentation | Next update |
| Team | Review and add comments on Confluence document | Before next meeting |
| Team | Continue discussion on validation rules and override conflict resolution | Next configs meeting |

---

## Decisions Made

- **"Three-way rules" renamed to "MCS pairs"** (Model–Carrier–Service Plan)
- Schema parameters **cannot be deleted** once defined — only new parameters can be added
- Legacy and new config systems will **run in parallel** during migration
- Migration will use **pioneer devices** for validation before wider rollout
- Company-level parameter availability will be **controlled via schema flags**
- Child companies can be **unlinked/relinked** from parent distributor override rules

---

## Open Questions / Follow-ups

- Detailed validation rules for preventing required parameters from being nulled at any level
- Whether company overrides with loose model specifications could override model-specific MCS pair values
- Exact schema flag behavior for controlling which parameters are available at the company override level
- Full versioning strategy for schema and model levels (deferred — low frequency of changes)
- Team unavailable next week (APW traveling Tue–Fri) — next meeting TBD
