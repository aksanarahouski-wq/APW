# PRD Builder Skill

A professional, evidence-based Product Requirements Document (PRD) generator with zero assumptions.

## Overview

This skill transforms context and research into comprehensive, actionable PRDs. It intelligently determines document type (Strategic, Implementation, or Simple), decides whether a separate Technical Requirements Document (TRD) is needed, and generates complete specifications ready for stakeholder review and development.

**Core Principle:** NEVER MAKE ASSUMPTIONS. If critical information is missing, the skill creates a Q&A document with blocking questions and halts PRD generation until answers are provided.

## Quick Start

```
# In Claude Code, invoke the skill:
"Create a PRD for [your feature name]"

# Or be specific:
"Generate an implementation PRD for user authentication"
"Build a strategic planning PRD for enterprise SSO"
```

## Skill Structure

```
prd-builder/
├── SKILL.md                    # Main skill definition and workflow
├── README.md                   # This file - overview and maintenance guide
└── templates/
    └── template-reference.md   # Template structure and decision logic reference
```

## How It Works

1. **Discovery & Classification**
   - Analyzes provided context and research
   - Determines PRD type (Strategic, Implementation, or Simple)
   - Assesses if separate TRD is needed
   - Extracts evidence and tracks sources

2. **Gap Detection & Blocking Questions**
   - Identifies missing critical information
   - Creates Q&A document with blocking questions if needed
   - **STOPS and waits** for user to provide answers
   - Reviews answers and asks follow-up questions if needed

3. **Document Generation** (Only after all questions answered)
   - Loads canonical template from skill templates folder
   - Generates complete PRD with all relevant sections
   - Creates TRD if technical complexity warrants it
   - Applies industry best practices
   - Includes evidence citations

4. **Validation & Output**
   - Self-reviews against enhanced quality checklist
   - Validates testability, unambiguity, traceability
   - Outputs files to appropriate locations
   - Provides summary and next steps

## PRD Types

### Strategic Planning
**When:** Major initiative needing approval, budget justification
**Includes:** Background, problem statement, implementation plan, success metrics
**Output:** High-level PRD that will later be broken into Implementation PRDs

### Implementation (Epic/Feature)
**When:** Approved feature ready for development
**Includes:** Detailed requirements, user stories, acceptance criteria, functional specs
**Excludes:** Background (already approved), implementation plan
**Output:** Detailed PRD ready for engineering team

### Simple Feature
**When:** Minor enhancement, straightforward change
**Includes:** Essentials only - overview, goals, stories, criteria, requirements, testing
**Simplified:** Single persona, streamlined sections
**Output:** Concise PRD with just what's needed

## TRD Decision Logic

**Create separate TRD when feature involves:**
- Complex data models with multiple entities/relationships
- System architecture changes or new patterns
- Multiple external integrations with detailed API specs
- Performance optimization requiring benchmarks
- Security implementation (encryption, authentication)
- Infrastructure changes (servers, databases, cloud)
- Data migration strategies or cutover plans
- Detailed technical design requiring diagrams

**Embed technical requirements in PRD when:**
- Small to medium complexity
- Straightforward technical needs
- Changes isolated to specific components
- No significant architecture changes

## File Locations

The skill generates files at:
- **Q&A Documents:** `/docs/qa/PRD-[feature-name]-QA.md` (if blocking questions exist)
- **Strategic PRDs:** `/docs/planning/PRD-[feature-name]-strategic.md`
- **Implementation PRDs:** `/docs/requirements/PRD-[feature-name].md`
- **Simple PRDs:** `/docs/features/PRD-[feature-name].md`
- **TRDs:** `/docs/technical/TRD-[feature-name].md`

## Dependencies

- **Template:** `.claude/skills/prd-builder/templates/PRD_Template.md`
- **Related Skills:**
  - `docs-management` - For maintaining PRDs as living documents
  - `ba-toolkit:visual-workflow-diagram-generator` - For creating process flow diagrams
  - `/gsd:plan-phase` - For breaking PRD into implementable phases

## Maintenance

### Updating the Skill

When making changes:

1. **Update SKILL.md** - Main workflow and instructions
2. **Update template-reference.md** - If template structure changes
3. **Test the skill** - Generate sample PRDs to verify behavior
4. **Update version number** - Increment version in SKILL.md
5. **Document changes** - Add to Version History section

### Updating the Template

The canonical template is at `.claude/skills/prd-builder/templates/PRD_Template.md`

When updating:
1. **Edit the template file**
2. **Update template-reference.md** if structure changed
3. **Test skill** with new template
4. **Update skill version** if workflow needs adjustment

### Quality Standards

Generated PRDs must meet these standards:
- ✅ Clear, unambiguous language
- ✅ Measurable success criteria and acceptance criteria
- ✅ Complete sections for chosen PRD type
- ✅ Proper linking to supporting documents
- ✅ Consistent formatting and terminology
- ✅ Traceable requirements (user story → acceptance criteria → functional req)
- ✅ Actionable specifications ready for development

## Best Practices

**Do:**
- ✅ Start with why - ground requirements in user/business value
- ✅ Be specific - vague requirements cause rework
- ✅ Think edge cases - what happens when things go wrong?
- ✅ Validate assumptions - document assumed vs. confirmed
- ✅ Link, don't duplicate - reference other docs
- ✅ Make it scannable - use formatting effectively
- ✅ Mark open questions clearly

**Don't:**
- ❌ Mix technical design with PRD - use TRDs for complex tech
- ❌ Combine multiple features - one PRD per feature/epic
- ❌ Skip user stories - they bridge goals and requirements
- ❌ Make acceptance criteria ambiguous - must be testable
- ❌ Ignore dependencies - causes delays
- ❌ Forget testing - define approach upfront
- ❌ Leave questions unanswered - document with owners/deadlines

## Examples

See SKILL.md for detailed examples of:
- Strategic Planning PRD for SSO authentication
- Implementation PRD for data export feature
- Simple Feature PRD for copy-to-clipboard button

## Troubleshooting

**Issue:** Skill created Q&A document and stopped
**Fix:** This is expected behavior! Fill in the answers in the Q&A document, then notify: "Answers provided for [feature name] PRD"

**Issue:** Generated PRD missing sections
**Fix:** Check that context provided enough information for that PRD type, or that all Q&A questions were answered

**Issue:** Wrong PRD type generated
**Fix:** Be explicit in your request: "Create a strategic planning PRD..." or "Create an implementation PRD..."

**Issue:** TRD not created when needed
**Fix:** Skill assesses complexity automatically. If you need TRD, mention: "This requires separate TRD for [reason]"

**Issue:** File location incorrect
**Fix:** Ensure target directories exist: `/docs/planning/`, `/docs/requirements/`, `/docs/features/`, `/docs/technical/`, `/docs/qa/`

## Version History

- **v2.1** (2026-03-01) - Added "Never Make Assumptions" methodology with Q&A workflow
- **v2.0** (2026-03-01) - Complete rewrite based on industry standards
- **v1.0** - Initial version

## Author

Aksana - March 2026

## License

Internal project use - Worstpills project
