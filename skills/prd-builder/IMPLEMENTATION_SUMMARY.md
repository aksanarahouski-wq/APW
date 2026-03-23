# PRD Builder Skill - Implementation Summary

**Date:** March 1, 2026
**Version:** 2.1
**Status:** ✅ Complete

## Latest Update: v2.1 - "Never Make Assumptions" Enhancement

Added robust Q&A workflow borrowed from Epic-Prompt methodology to eliminate assumptions and ensure evidence-based requirements.

---

## What Was Built

A professional, industry-standards-based PRD builder skill that intelligently generates Product Requirements Documents based on context and research provided by the user.

---

## Skill Structure

```
.claude/skills/prd-builder/
├── SKILL.md                           # Main skill definition (v2.1)
│   └── Complete workflow with Q&A phases
├── README.md                          # Overview and maintenance guide (v2.1)
│   └── Quick start, usage, troubleshooting
├── QUICK-REFERENCE.md                 # Quick reference card (v2.1)
│   └── Cheat sheet with Q&A workflow
├── IMPLEMENTATION_SUMMARY.md          # This file
│   └── Build summary and decisions made
└── templates/
    ├── PRD_Template.md                # Canonical PRD template (11KB)
    │   └── Your comprehensive template (moved from /Templates/)
    ├── QA-Template.md                 # Q&A document template (NEW v2.1)
    │   └── Template for blocking questions workflow
    ├── template-reference.md          # Template structure reference (3.5KB)
    │   └── Decision logic and structure documentation
    └── example-output.md              # Example outputs (v2.1)
        └── Q&A example + 3 PRD examples
```

**Total Skill Size:** ~55KB across 8 files

---

## Key Features Implemented

### 1. Intelligent PRD Type Classification
The skill automatically determines the right PRD type based on project scope:

- **Strategic Planning** - Major initiatives needing approval
- **Implementation** - Approved features ready for development
- **Simple Feature** - Minor enhancements or straightforward changes

### 2. TRD Decision Logic
Automatically assesses whether a separate Technical Requirements Document is needed based on:
- Data model complexity
- Architecture changes
- Integration requirements
- Security implementation
- Performance needs
- Infrastructure changes
- Migration strategies

### 3. Industry Best Practices
Based on standards from:
- Product School
- Atlassian (Jira/Confluence PRD patterns)
- Pragmatic Institute
- Modern product management frameworks

### 4. Interactive Workflow
Four-phase process:
1. **Discovery & Classification** - Analyze context, determine PRD type
2. **Interactive Clarification** - Ask targeted questions to fill gaps
3. **Document Generation** - Create complete, structured PRDs
4. **Validation & Output** - Self-review and deliver files

### 5. Comprehensive Documentation
- Complete skill instructions (SKILL.md)
- Maintenance guide (README.md)
- Quick reference card (QUICK-REFERENCE.md)
- Template reference with decision logic
- Three detailed example outputs

---

## Design Decisions

### ✅ Template Usage Guide Moved to Skill
**Rationale:** The usage guide (originally lines 393-487 in template) is workflow logic, not template content. Moving it to the skill:
- Keeps template focused on structure
- Makes skill self-contained
- Allows skill to guide users interactively
- Easier to maintain in one place

### ✅ Template Location in Skill Folder
**Rationale:** Keeping template with skill makes it:
- Self-contained and portable
- Easier to version together
- Clear ownership (skill owns template)
- Simpler for skill to reference

**Original location:** `/Users/aksana/Documents/Projects/Worstpills/Templates/PRD_Template.md`
**New location:** `.claude/skills/prd-builder/templates/PRD_Template.md`
**Status:** Template copied to skill folder, all references updated

### ✅ Separate TRD Decision Logic
**Rationale:** Complex technical requirements deserve dedicated documentation:
- Keeps PRDs focused on WHY and WHAT
- Allows technical depth without bloating PRD
- Separates concerns (product vs. technical specs)
- Matches industry patterns

### ✅ Three Example Outputs
**Rationale:** Concrete examples show:
- What each PRD type looks like in practice
- How detail level varies by type
- When to choose each approach
- Expected output quality

---

## Template Changes

### Original Template (Lines 393-487)
**Removed:**
- 🎯 Document Purpose section
- 📋 Three Usage Scenarios table
- 🔧 When to Create a Separate TRD section
- 📊 Quick Decision Flow diagram
- Key Principles list
- Getting Started steps

**Reason:** All moved to SKILL.md workflow

### Updated Template (Lines 393-406)
**Added:**
- Simple "Template Usage" section
- References to prd-builder skill
- Links to skill documentation
- Manual usage guidance

---

## Quality Standards

### Generated PRDs Must Meet:
- ✅ Clear, unambiguous language
- ✅ Measurable success criteria
- ✅ Testable acceptance criteria
- ✅ Complete sections for PRD type
- ✅ Proper linking to supporting docs
- ✅ Consistent formatting
- ✅ Traceable requirements
- ✅ Actionable specifications

### Skill Documentation Standards:
- ✅ Clear structure with navigation
- ✅ Examples for common scenarios
- ✅ Decision trees and flowcharts
- ✅ Best practices and anti-patterns
- ✅ Troubleshooting guidance
- ✅ Version history tracking

---

## File Locations

### Skill Files (All in `.claude/skills/prd-builder/`)
- Core skill logic and workflow
- Template and examples
- Documentation and reference

### Generated PRD Outputs
- Strategic PRDs: `/docs/planning/PRD-[name]-strategic.md`
- Implementation PRDs: `/docs/requirements/PRD-[name].md`
- Simple PRDs: `/docs/features/PRD-[name]-simple.md`
- TRDs: `/docs/technical/TRD-[name].md`

### Original Template
- Location: `/Users/aksana/Documents/Projects/Worstpills/Templates/PRD_Template.md`
- Status: Still exists, updated with new usage instructions
- Purpose: Can be used manually if needed

---

## How to Use the Skill

### Invocation
```
"Create a PRD for [feature name]"
"Generate a strategic/implementation/simple PRD for [feature]"
"Build a PRD based on this research"
```

### Prerequisites
Gather before invoking:
- ✅ Context (background, business objectives, current state)
- ✅ Research (user research, competitive analysis, constraints)
- ✅ Stakeholder input (goals, success criteria, decisions)
- ✅ Scope clarity (in/out of scope, dependencies)

### Expected Flow
1. User invokes skill with feature name
2. Skill analyzes context and determines PRD type
3. Skill asks clarifying questions if needed
4. Skill generates complete PRD (and TRD if needed)
5. Skill outputs files to appropriate locations
6. Skill provides summary and next steps

---

## Integration with Other Skills

### Upstream (Input Sources)
- Product roadmap and strategy
- User research and feedback
- Technical feasibility studies
- Design explorations and prototypes

### Downstream (Next Steps)
- `/gsd:plan-phase` - Break PRD into implementable phases
- `docs-management` - Maintain PRD as living document
- `ba-toolkit:visual-workflow-diagram-generator` - Create process flows
- Development - Epic and story breakdown

---

## Version History

### v2.1 (2026-03-01) - Current - "Never Make Assumptions" Enhancement
**Borrowed from Epic-Prompt.md methodology**

**Major Changes:**
- ✅ Added "Never Make Assumptions" core principle
- ✅ Added Phase 2: Gap Detection & Blocking Questions workflow
- ✅ Added Phase 2A: Follow-up Questions for iterative clarification
- ✅ Created Q&A document template (QA-Template.md)
- ✅ Enhanced validation checklist (testability, unambiguity, traceability)
- ✅ Added evidence-based requirements rule with confidence levels
- ✅ Added explicit blocking if critical information missing
- ✅ Strengthened quality standards to eliminate vague terms
- ✅ Added Q&A example to example-output.md
- ✅ Updated all documentation to reflect Q&A workflow

**Files Modified:**
- SKILL.md - Added Core Principles, Phase 2, Phase 2A, enhanced Phase 4
- README.md - Updated workflow, file locations, troubleshooting
- QUICK-REFERENCE.md - Added Q&A workflow, enhanced quality checklist
- example-output.md - Added complete Q&A document example
- IMPLEMENTATION_SUMMARY.md - Updated with v2.1 changes

**Files Added:**
- templates/QA-Template.md - Template for blocking questions documents
- CHANGELOG.md - Version history tracker

### v2.0 (2026-03-01)
- ✅ Complete skill rewrite based on industry standards
- ✅ Integrated Template Usage Guide into skill workflow
- ✅ Added interactive clarification phase
- ✅ Added TRD decision logic
- ✅ Enhanced output quality standards
- ✅ Added comprehensive documentation
- ✅ Created three detailed example outputs
- ✅ Moved template to skill folder
- ✅ Updated all template references

### v1.0 (Initial)
- Basic PRD generation capability
- Simple template usage

---

## Testing Recommendations

Before using in production, test with:

1. **Strategic PRD Test**
   - Input: "Create a PRD for enterprise SSO authentication"
   - Expected: Strategic planning PRD with background, implementation plan, success metrics

2. **Implementation PRD Test**
   - Input: "Generate implementation PRD for data export feature"
   - Expected: Detailed requirements, user stories, acceptance criteria, no background section

3. **Simple PRD Test**
   - Input: "Create PRD for copy-to-clipboard button"
   - Expected: Concise PRD with essentials only

4. **TRD Decision Test**
   - Input: Complex feature description with architecture changes
   - Expected: PRD + separate TRD reference

---

## Maintenance Guidelines

### When to Update

**Update SKILL.md when:**
- Workflow changes
- New PRD types added
- Decision logic changes
- Industry best practices evolve

**Update PRD_Template.md when:**
- Section structure changes
- New required fields added
- Format standards change

**Update example-output.md when:**
- Adding new PRD type examples
- Showing new template features
- Demonstrating new patterns

### Versioning
- Increment version in SKILL.md
- Document changes in Version History
- Update IMPLEMENTATION_SUMMARY.md
- Test skill with new version

---

## Success Criteria

This skill is successful if:
- ✅ Users can generate complete PRDs from context/research
- ✅ PRDs meet quality standards consistently
- ✅ Correct PRD type selected automatically
- ✅ TRD created only when needed
- ✅ Generated PRDs are ready for stakeholder review
- ✅ PRDs accelerate development (not block it)
- ✅ Less than 10% of PRDs require significant rework

---

## Known Limitations

1. **Context Dependency** - Quality depends on input context quality
2. **Requires Human Review** - Generated PRDs should be reviewed before approval
3. **Domain Knowledge** - May need clarification for domain-specific requirements
4. **Stakeholder Alignment** - Cannot replace stakeholder conversations

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

## References

### Industry Standards Applied
- **Product School** - PRD structure and best practices
- **Atlassian** - Integration patterns and documentation standards
- **Pragmatic Institute** - Requirements framework and user story formats
- **Diataxis Framework** - Documentation structure principles

### Related Documentation
- Project-level: `/docs/planning/`, `/docs/requirements/`, `/docs/features/`
- Template: `.claude/skills/prd-builder/templates/PRD_Template.md`
- Examples: `.claude/skills/prd-builder/templates/example-output.md`

---

## Acknowledgments

**Author:** Aksana
**Skill Version:** 2.0
**Template Version:** 1.0
**Implementation Date:** March 1, 2026

---

END OF SUMMARY
