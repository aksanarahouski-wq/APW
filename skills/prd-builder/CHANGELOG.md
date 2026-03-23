# PRD Builder Skill - Changelog

All notable changes to the PRD Builder skill will be documented in this file.

---

## [2.1.0] - 2026-03-01

### Added - "Never Make Assumptions" Enhancement
Borrowed methodologies from Epic-Prompt.md to eliminate assumptions and ensure evidence-based requirements.

#### New Features
- **Core Principles section** with two explicit rules:
  - Rule #1: Never Make Assumptions
  - Rule #2: Evidence-Based Requirements
- **Phase 2: Gap Detection & Blocking Questions**
  - Identifies missing critical information
  - Generates structured blocking questions with Q-IDs
  - Creates Q&A document with blank answers
  - STOPS PRD generation until questions answered
- **Phase 2A: Follow-up Questions**
  - Reviews initial answers for ambiguities
  - Generates follow-up questions (FQ-X format)
  - Iterative clarification loop
- **Q&A Document Template** (QA-Template.md)
  - Structured format for blocking questions
  - Status tracking table
  - Follow-up questions section
- **Enhanced Validation Checklist**
  - Testability check
  - Unambiguity check (no vague terms)
  - Traceability check (evidence citations)
  - No assumptions check
- **Evidence Confidence Levels**
  - ✅ High confidence: Explicitly stated in context
  - ⚠️ Medium confidence: Reasonably inferred from context
  - ❓ Low confidence: Assumed or unclear, requires validation

#### Enhanced
- **Phase 1: Discovery & Classification**
  - Now extracts evidence and tracks sources
- **Phase 4: Validation & Output**
  - Expanded checklist from 10 to 20 items
  - Added checks for testability, unambiguity, traceability, consistency, completeness, conciseness
- **Output Standards**
  - Enhanced traceability section with evidence requirements
  - No placeholders (TBD, TODO) without approval
  - Explicit prohibition of vague terms: fast, easy, user-friendly, soon, later

#### Documentation
- **README.md** - Updated workflow, file locations, troubleshooting
- **QUICK-REFERENCE.md** - Added Q&A workflow section at top
- **example-output.md** - Added complete Q&A document example (Example 0)
- **IMPLEMENTATION_SUMMARY.md** - Updated with v2.1 changes
- **CHANGELOG.md** - New file tracking version history

#### Files Modified
- `SKILL.md` - Added ~150 lines (Core Principles, Phase 2, Phase 2A, enhanced Phase 4)
- `README.md` - Added ~30 lines (workflow updates, Q&A documentation)
- `QUICK-REFERENCE.md` - Added ~25 lines (Q&A workflow, enhanced checklist)
- `example-output.md` - Added ~230 lines (complete Q&A example)
- `IMPLEMENTATION_SUMMARY.md` - Added ~50 lines (v2.1 details)

#### Files Added
- `templates/QA-Template.md` - Q&A document template (~100 lines)
- `CHANGELOG.md` - This file (version history tracker)

### Changed
- **Skill version** from 2.0 to 2.1
- **Workflow** now has explicit blocking phase if info is missing
- **File locations** now include `/docs/qa/` for Q&A documents
- **Quality standards** significantly strengthened

### Impact
- **Before v2.1:** Skill made assumptions when context incomplete → PRDs needed rework
- **After v2.1:** Skill asks blocking questions → PRDs accurate from the start
- **Result:** Better stakeholder alignment, less rework, faster development

---

## [2.0.0] - 2026-03-01

### Added - Complete Skill Rewrite
Major rewrite based on industry standards (Product School, Atlassian, Pragmatic Institute).

#### New Features
- **Intelligent PRD type classification**
  - Strategic Planning PRDs
  - Implementation PRDs
  - Simple Feature PRDs
- **TRD decision logic**
  - Automatic assessment of technical complexity
  - Separate TRD creation for complex features
- **Four-phase workflow**
  - Phase 1: Discovery & Classification
  - Phase 2: Interactive Clarification (basic)
  - Phase 3: Document Generation
  - Phase 4: Validation & Output
- **Comprehensive template** (PRD_Template.md)
  - Moved from `/Templates/` to skill folder
  - All PRD sections structured
- **Quality standards** and best practices
- **Three detailed examples**
  - Strategic Planning PRD (SSO authentication)
  - Implementation PRD (data export)
  - Simple Feature PRD (copy-to-clipboard)

#### Documentation
- `SKILL.md` - Complete skill definition (~360 lines)
- `README.md` - Overview and maintenance guide (~190 lines)
- `QUICK-REFERENCE.md` - Quick reference card (~120 lines)
- `templates/template-reference.md` - Template structure reference
- `templates/example-output.md` - Three complete PRD examples
- `IMPLEMENTATION_SUMMARY.md` - Build summary

#### Template Usage Guide
- Moved from template file into skill workflow
- Added decision flows and scenarios
- Integrated TRD creation guidelines

---

## [1.0.0] - Initial Release

### Added
- Basic PRD generation capability
- Simple template usage
- Minimal documentation

---

## Version Numbering

This project uses [Semantic Versioning](https://semver.org/):
- **MAJOR version** (X.0.0) - Incompatible workflow changes
- **MINOR version** (0.X.0) - New features, backward compatible
- **PATCH version** (0.0.X) - Bug fixes, documentation updates

---

## References

- **v2.1 Inspiration:** `/Users/aksana/Documents/Projects/Worstpills/Templates/Epic-Prompt.md`
- **Industry Standards:** Product School, Atlassian, Pragmatic Institute
- **Documentation Framework:** Diataxis principles

---

## Future Enhancements

Potential v3.0 features:
- [ ] Integration with Jira/Confluence for direct PRD creation
- [ ] Template variations for different industries (SaaS, Healthcare, Finance)
- [ ] Automated PRD-to-Epic breakdown
- [ ] Version comparison for PRD updates
- [ ] PRD quality scoring and recommendations
- [ ] Integration with design tools (Figma) for UI/UX requirements
- [ ] Automated stakeholder review routing

---

**Maintained by:** Aksana
**Project:** Worstpills
**Location:** `.claude/skills/prd-builder/`
