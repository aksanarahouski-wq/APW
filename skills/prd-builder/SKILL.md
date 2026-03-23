# PRD Builder Skill

**Version:** 2.1
**Purpose:** Generate comprehensive, evidence-based Product Requirements Documents (PRDs) with zero assumptions

---

## Skill Overview

This skill creates professional PRDs following best practices from Product School, Atlassian, and Pragmatic Institute. It intelligently determines the appropriate document type (Strategic Planning, Implementation, or Simple Feature) and generates complete, actionable requirements.

**Core Principle: NEVER MAKE ASSUMPTIONS.** If information is missing, the skill will create blocking questions and halt PRD generation until answers are provided. This ensures PRDs are accurate, evidence-based, and aligned with stakeholder intent.

---

## When to Use This Skill

Invoke this skill when you need to:
- Create a PRD for a new feature or product initiative
- Document requirements for an approved epic ready for development
- Formalize specifications for a simple feature enhancement
- Transform research, user feedback, or stakeholder input into structured requirements

**Invocation patterns:**
- "Create a PRD for [feature name]"
- "Generate product requirements for [initiative]"
- "Document the requirements for [project]"
- "Build a PRD based on this research/context"

---

## Prerequisites

Before invoking this skill, gather:

✅ **Context** - Project background, business objectives, current state
✅ **Research** - User research, competitive analysis, technical constraints
✅ **Stakeholder Input** - Goals, success criteria, key decisions
✅ **Scope Clarity** - What's in/out of scope, dependencies, constraints

**Note:** The skill may create a Q&A document with blocking questions if critical information is missing. PRD generation will not proceed until all questions are answered.

---

## Core Principles

### Rule #1: Never Make Assumptions

- **If information is missing, ASK.** Do not invent details or make assumptions.
- **If evidence is ambiguous, FLAG IT.** Document uncertainty and ask for clarification.
- **If requirements conflict, STOP.** Create blocking question to resolve conflict.
- **Block PRD generation** if critical questions remain unanswered.
- **No placeholders (TBD, TODO)** without explicit user approval.

This ensures PRDs are accurate, evidence-based, and aligned with stakeholder intent.

### Rule #2: Evidence-Based Requirements

- Every requirement must trace to a source (context, research, user answer)
- Include evidence citations in Notes section
- Document confidence level if evidence is ambiguous:
  - ✅ **High confidence:** Explicitly stated in provided context
  - ⚠️ **Medium confidence:** Reasonably inferred from context
  - ❓ **Low confidence:** Assumed or unclear - requires validation
- Use precise, testable language (avoid vague terms: fast, easy, user-friendly, soon)

---

## Workflow

### Phase 1: Discovery & Classification

1. **Analyze provided context and research**
   - Review all background materials
   - Identify feature scope and complexity
   - Understand stakeholders and users
   - Extract evidence and track sources

2. **Determine PRD Type**
   ```
   Strategic Planning → Major initiative needing approval
   ├─ Includes: Background, Problem Statement, Implementation Plan, Success Metrics
   ├─ Use case: New product features, major initiatives, budget approval needed
   └─ Output: High-level PRD + link to detailed Implementation PRDs

   Implementation (Epic/Feature) → Approved feature ready for dev
   ├─ Includes: Detailed requirements, acceptance criteria, user stories
   ├─ Excludes: Background/problem (already approved), implementation plan
   └─ Output: Detailed PRD ready for engineering

   Simple Feature → Minor enhancement or straightforward change
   ├─ Includes: Overview, goals, stories, criteria, requirements, testing
   ├─ Simplified: Single persona, streamlined sections
   └─ Output: Concise PRD with essentials only
   ```

3. **Assess Technical Complexity**

   **Create Separate TRD if feature involves:**
   - Complex data models with multiple entities/relationships
   - System architecture changes or new patterns
   - Multiple external integrations with detailed API specs
   - Performance optimization requiring benchmarks
   - Security implementation with encryption/authentication details
   - Infrastructure changes (servers, databases, cloud services)
   - Data migration strategies or cutover plans
   - Detailed technical design requiring architecture diagrams

   **Embed technical requirements in PRD if:**
   - Small to medium complexity
   - Straightforward technical needs
   - Changes isolated to specific components
   - No significant architecture changes

### Phase 2: Gap Detection & Blocking Questions

1. **Analyze provided context for completeness:**
   - Identify missing critical information for chosen PRD type
   - Flag ambiguous or conflicting statements
   - Detect assumptions that need validation
   - Review evidence quality and confidence levels

2. **Generate blocking questions** if critical information is missing:

   Each question must include:
   - **Question ID:** Q-1, Q-2, Q-3, etc.
   - **What decision/data is needed:** The specific question
   - **Why it matters:** Which PRD sections/requirements are impacted
   - **Evidence (if any):** Quotes from provided context with citations
   - **Answer:** [Blank - to be filled by user/stakeholders]

   **Critical areas to check:**

   **For Strategic Planning PRDs:**
   - Business problems and impact if not addressed
   - Primary business goals and success metrics
   - Implementation timeline and phasing
   - Key stakeholders and decision makers
   - Budget/resource constraints

   **For Implementation PRDs:**
   - Primary user personas and pain points
   - Detailed acceptance criteria and edge cases
   - Functional requirements and business rules
   - UI/UX requirements or design references
   - Critical dependencies and risks
   - Testing strategy and validation approach

   **For Simple Feature PRDs:**
   - Core user need being addressed
   - Success definition (1-3 measurable criteria)
   - Must-have vs. nice-to-have requirements

   **For All PRDs:**
   - Technical complexity (TRD decision)
   - Related features or existing work
   - Compliance/regulatory requirements

3. **Create Q&A Document** if blocking questions exist:

   **Location:** `/docs/qa/PRD-[feature-name]-QA.md`

   **Structure:**
   ```markdown
   # [Feature Name] - PRD Questions & Answers

   **PRD:** [PRD-feature-name.md](../requirements/PRD-feature-name.md)
   **Status:** Awaiting Answers
   **Created:** [Date]

   _This document captures questions that need answers before drafting the PRD._

   ---

   ## Blocking Questions

   ### Q-1: [Question Topic]

   **What decision/data is needed:**
   [Specific question text]

   **Why it matters:**
   - [Impact on PRD section/requirement]
   - [Dependencies on this decision]
   - [Risk if not clarified]

   **Evidence:**
   [Quotes from provided context, if any. Format: "Quote text" from [source]]
   [If no evidence: "No direct evidence in provided context"]

   **Answer:**
   _[To be filled in by user/stakeholders]_

   ---
   ```

4. **STOP and inform user** if blocking questions exist:
   ```
   ⚠️ BLOCKING QUESTIONS DETECTED

   I've identified [X] critical questions that must be answered before generating the PRD.

   Q&A document created at: /docs/qa/PRD-[feature-name]-QA.md

   Next steps:
   1. Review the Q&A document
   2. Fill in the Answer sections for each question
   3. Notify me when complete: "Answers provided for [feature name] PRD"
   4. I'll review your answers and either:
      - Ask follow-up questions (FQ-1, FQ-2) if clarification needed
      - Proceed to generate the PRD if all is clear

   Do NOT proceed to Phase 3 until all questions answered.
   ```

5. **If no blocking questions:** Proceed directly to Phase 3

### Phase 2A: Follow-up Questions (After Initial Answers)

After user provides initial answers:

1. **Review answers for:**
   - Ambiguities or incomplete information
   - References to standards/features needing clarification
   - Assumptions that need validation
   - Conflicts or contradictions

2. **Generate follow-up questions** (FQ-1, FQ-2, etc.) if needed:
   - Insert after related blocking question in Q&A doc
   - Format: Same structure as blocking questions
   - Header: `### FQ-1: [Topic] (Follow-up to Q-X)`
   - Reference which blocking question this clarifies

3. **Update Q&A document** with follow-up questions:
   - Add follow-ups immediately after related blocking questions
   - Update Status to "Awaiting Follow-up Answers"
   - Leave Answer sections blank

4. **Notify user:**
   ```
   I've reviewed your answers and need [X] follow-up clarifications.

   Q&A document updated with follow-up questions (FQ-1, FQ-2, etc.)

   Please fill in the follow-up answers, then notify me to proceed.
   ```

5. **Wait for follow-up answers** before proceeding to Phase 3

6. **If all answers clear:** Proceed to Phase 3

### Phase 3: Document Generation

1. **Load the appropriate template** from `.claude/skills/prd-builder/templates/PRD_Template.md`

2. **Generate complete PRD** following this structure:

   **Header Metadata:**
   - Document version, date, author, status
   - Related tickets/epics with links
   - Document owner and last updated date

   **Core Sections (all types):**
   - Overview with key features and business impact
   - Goals and success criteria (measurable)
   - Users and user stories (persona-driven)
   - Acceptance criteria (specific, testable)
   - Scope (in/out boundaries)
   - Functional requirements and business rules
   - Testing requirements
   - Dependencies and risks
   - References and notes

   **Strategic-Only Sections:**
   - Background and problem statement
   - Implementation plan with phases
   - Success metrics (KPIs)
   - Rollout strategy

   **Conditional Sections:**
   - Technical requirements (inline or TRD reference)
   - UI/UX requirements (if applicable)
   - Data model overview (if needed)

3. **Apply PRD Best Practices:**
   - Use clear, active language ("The system shall...")
   - Make criteria measurable and testable
   - Link to external docs (personas, designs, TRDs)
   - Include realistic examples and edge cases
   - Document assumptions and open questions
   - Reference evidence sources (research, transcripts)
   - Use consistent formatting (checklists, tables, headings)

4. **Create TRD if needed:**
   - Reference TRD location in PRD Technical Requirements section
   - Create placeholder or outline for TRD with key sections
   - Document technical approach, architecture, data models, APIs
   - Include performance benchmarks, security specs, infrastructure needs

### Phase 4: Validation & Output

1. **Self-review checklist:**
   - [ ] All required sections completed for PRD type
   - [ ] **Testability:** Every acceptance criterion is verifiable
   - [ ] **Unambiguity:** No vague terms (fast, easy, user-friendly, soon, later)
   - [ ] **Traceability:** Every requirement mapped to source/evidence
   - [ ] **Consistency:** Terminology consistent throughout document
   - [ ] **Completeness:** All template sections appropriately filled
   - [ ] **Conciseness:** Follows template's level of detail (not overly verbose)
   - [ ] Success criteria are specific and measurable
   - [ ] Acceptance criteria cover happy path and edge cases
   - [ ] User stories follow "As [persona], I need [capability] so that [outcome]" format
   - [ ] Business rules are clearly stated and verifiable
   - [ ] Dependencies identified with mitigation plans
   - [ ] Risks assessed with impact and probability
   - [ ] **No assumptions documented** - all blocking questions answered
   - [ ] Open questions documented with decision owners (non-blocking only)
   - [ ] References include all supporting documentation and evidence sources
   - [ ] Technical complexity appropriately handled (TRD vs inline)
   - [ ] Evidence citations included in Notes section

2. **Generate PRD file** at appropriate location:
   - Strategic: `/docs/planning/PRD-[feature-name]-strategic.md`
   - Implementation: `/docs/requirements/PRD-[feature-name].md`
   - Simple: `/docs/features/PRD-[feature-name].md`

3. **Generate TRD if needed** at `/docs/technical/TRD-[feature-name].md`

4. **Provide summary** to user:
   - PRD type created and rationale
   - File location(s)
   - Key decisions made
   - Open questions requiring stakeholder input
   - Next steps (review, approval, breakdown into stories)

---

## Output Standards

### Document Quality Expectations

**Clarity:**
- Use precise, unambiguous language
- Define domain-specific terms on first use
- Avoid jargon unless necessary (and then explain it)

**Completeness:**
- Cover all sections relevant to PRD type
- Provide sufficient detail for development team
- Include examples where helpful for understanding

**Consistency:**
- Follow template structure exactly
- Use consistent terminology throughout
- Maintain consistent formatting (bullet styles, table formats)

**Traceability & Evidence:**
- Link user stories to acceptance criteria
- Link functional requirements to user needs
- Every requirement must trace to source (context, research, user answer)
- Include evidence citations in Notes section
- Document confidence level if evidence is ambiguous:
  - ✅ **High confidence:** Explicitly stated in context
  - ⚠️ **Medium confidence:** Inferred from context
  - ❓ **Low confidence:** Assumed, needs validation
- No placeholders (TBD, TODO) without explicit user approval

**Actionability:**
- Make requirements implementable
- Provide clear success criteria
- Include enough detail to estimate effort

### File Naming Conventions

- Strategic PRDs: `PRD-[feature-name]-strategic.md`
- Implementation PRDs: `PRD-[feature-name].md`
- Simple Feature PRDs: `PRD-[feature-name]-simple.md`
- Technical Requirement Docs: `TRD-[feature-name].md`

Use kebab-case for feature names (e.g., `user-authentication`, `payment-processing`)

---

## Integration with Other Processes

**Upstream Inputs:**
- Product roadmap and strategy
- User research and feedback
- Technical feasibility studies
- Design explorations and prototypes

**Downstream Outputs:**
- Epic and story breakdown for development
- Technical design documents (TRDs)
- Test plans and test cases
- Release and rollout plans

**Related Skills:**
- Use `docs-management` skills for maintaining PRD documentation
- Use `ba-toolkit:visual-workflow-diagram-generator` for process flows
- Use `/gsd:plan-phase` to break PRD into implementable phases

---

## Examples

### Example 1: Strategic Planning PRD
```
User provides: "We need to add SSO authentication to support enterprise customers"

Skill determines:
- Type: Strategic Planning (major initiative, budget approval needed)
- TRD needed: Yes (complex security implementation, multiple integrations)

Output:
- PRD-sso-authentication-strategic.md
  ├─ Background: Enterprise customers require SSO, losing deals without it
  ├─ Goals: Enable enterprise sales, improve security, reduce support load
  ├─ Implementation Plan: 3 phases over 6 months
  ├─ Success Metrics: 80% enterprise customers adopt, reduce auth tickets by 50%
  └─ Links to: TRD-sso-authentication.md for detailed specs
```

### Example 2: Implementation PRD
```
User provides: Context about approved "Export Data" feature ready for development

Skill determines:
- Type: Implementation (feature approved, needs detailed requirements)
- TRD needed: No (straightforward feature, standard patterns)

Output:
- PRD-export-data.md
  ├─ Detailed user stories for 3 personas
  ├─ 15 specific acceptance criteria
  ├─ Functional requirements for CSV, Excel, PDF export
  ├─ Business rules for data filtering and permissions
  ├─ Technical requirements embedded (file formats, size limits, performance)
  └─ Testing scenarios for integration and UAT
```

### Example 3: Simple Feature PRD
```
User provides: "Add a 'Copy to Clipboard' button for sharing links"

Skill determines:
- Type: Simple Feature (minor enhancement, single interaction)
- TRD needed: No (trivial technical implementation)

Output:
- PRD-copy-link-button-simple.md
  ├─ Overview: One-click link copying for easier sharing
  ├─ Single user persona (all users)
  ├─ 3 acceptance criteria
  ├─ Basic functional requirements
  └─ Simple testing approach
```

---

## Best Practices

1. **Start with Why** - Always ground requirements in user/business value
2. **Be Specific** - Vague requirements lead to rework and confusion
3. **Think Edge Cases** - What happens when things go wrong?
4. **Validate Assumptions** - Document what you're assuming vs. what's confirmed
5. **Keep it DRY** - Link to other docs instead of duplicating content
6. **Make it Scannable** - Use formatting to help readers find what they need
7. **Involve Stakeholders** - Mark open questions and decision points clearly
8. **Iterate** - PRDs are living documents, update as understanding evolves

---

## Anti-Patterns to Avoid

❌ **Don't write technical designs in PRDs** - Use TRDs for complex technical details
❌ **Don't mix multiple features** - One PRD per feature/epic
❌ **Don't skip user stories** - They're the bridge between business goals and requirements
❌ **Don't make acceptance criteria ambiguous** - They must be testable
❌ **Don't ignore dependencies** - Unidentified dependencies cause delays
❌ **Don't forget about testing** - Define testing approach upfront
❌ **Don't leave questions unanswered** - Document open items with owners and deadlines

---

## Skill Metadata

**Author:** Aksana
**Created:** March 2026
**Template Version:** 1.0
**Skill Version:** 2.0
**Last Updated:** March 1, 2026

**Template Location:** `.claude/skills/prd-builder/templates/PRD_Template.md`

---

## Version History

### v2.1 (2026-03-01)
- **MAJOR:** Added "Never Make Assumptions" core principle
- **MAJOR:** Added Phase 2 Gap Detection with blocking questions
- **MAJOR:** Added Q&A document creation workflow
- **MAJOR:** Added Phase 2A Follow-up Questions for iterative clarification
- Enhanced validation checklist with testability, unambiguity, traceability checks
- Added evidence-based requirements rule with confidence levels
- Added explicit blocking if critical information missing
- Strengthened quality standards to eliminate vague terms

### v2.0 (2026-03-01)
- Complete skill rewrite based on industry standards
- Integrated Template Usage Guide into skill workflow
- Added interactive clarification phase
- Added TRD decision logic
- Enhanced output quality standards
- Added examples and anti-patterns

### v1.0 (Initial)
- Basic PRD generation capability
