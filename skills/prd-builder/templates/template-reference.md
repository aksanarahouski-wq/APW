# PRD Template Reference

This file documents the template structure for the prd-builder skill.

## Template Location

The canonical PRD template is maintained at:
`.claude/skills/prd-builder/templates/PRD_Template.md`

## Template Structure

### Required Sections (All PRD Types)
- Document Metadata (version, date, author, status, tickets)
- Overview (2-3 sentences + key features + business impact)
- Goals and Success Criteria
- Users and User Stories
- Acceptance Criteria
- Scope (In Scope categories)
- Functional Requirements and Business Rules
- Testing Requirements
- Dependencies and Risks
- References
- Notes

### Strategic Planning Only
- Background and Problem Statement
- Implementation Plan
- Success Metrics (KPIs)

### Conditional Sections
- Technical Requirements (inline or TRD reference)
- UI/UX Requirements (if applicable)
- Data Model Overview (if needed)

## Decision Logic

### PRD Type Selection
```
IF major initiative needing approval THEN
  Type = Strategic Planning
  Include = Background, Implementation Plan, Success Metrics
  Create = High-level PRD + link to future Implementation PRDs

ELSE IF approved feature ready for development THEN
  Type = Implementation
  Include = Detailed requirements, stories, criteria
  Exclude = Background, Implementation Plan, Success Metrics
  Create = Detailed PRD ready for engineering

ELSE IF minor enhancement or straightforward change THEN
  Type = Simple Feature
  Include = Essentials only (overview, goals, stories, criteria, requirements, testing)
  Simplify = Single persona, streamlined sections
  Create = Concise PRD
```

### TRD Decision
```
IF feature involves ANY OF:
  - Complex data models (multiple entities/relationships)
  - System architecture changes
  - Multiple external integrations with detailed API specs
  - Performance optimization with benchmarks
  - Security implementation (encryption, auth details)
  - Infrastructure changes
  - Data migration strategies
  - Detailed technical design with diagrams
THEN
  Create separate TRD
  Reference TRD in PRD Technical Requirements section
ELSE
  Embed technical requirements directly in PRD
```

## File Naming

- Strategic: `PRD-[feature-name]-strategic.md`
- Implementation: `PRD-[feature-name].md`
- Simple: `PRD-[feature-name]-simple.md`
- TRD: `TRD-[feature-name].md`

Use kebab-case for feature names.

## File Locations

- Strategic PRDs: `/docs/planning/`
- Implementation PRDs: `/docs/requirements/`
- Simple PRDs: `/docs/features/`
- TRDs: `/docs/technical/`

## Quality Checklist

Before finalizing a PRD, verify:
- [ ] PRD type matches project scope and approval status
- [ ] All required sections completed for chosen type
- [ ] Success criteria are specific and measurable
- [ ] Acceptance criteria cover happy path and edge cases
- [ ] User stories follow "As [persona], I need [capability] so that [outcome]" format
- [ ] Business rules clearly stated and verifiable
- [ ] Dependencies identified with mitigation plans
- [ ] Risks assessed with impact, probability, and mitigation
- [ ] Open questions documented with decision owners and deadlines
- [ ] References include all supporting documentation links
- [ ] Technical complexity appropriately handled (TRD vs inline)
- [ ] Examples provided where helpful
- [ ] Consistent terminology and formatting throughout

## Template Maintenance

When updating the template:
1. Update the canonical template file first
2. Update this reference document if structure changes
3. Test skill with new template structure
4. Update skill version number if significant changes
