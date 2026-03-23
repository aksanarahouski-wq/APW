# PRD Builder - Quick Reference Card

**Version:** 2.1 | **Core Principle:** NEVER MAKE ASSUMPTIONS

## Invocation

```
"Create a PRD for [feature name]"
"Generate a strategic/implementation/simple PRD for [feature]"
"Build a PRD based on this research"
```

## ⚠️ Important: Q&A Workflow

**If critical info is missing:**
- Skill creates `/docs/qa/PRD-[name]-QA.md` with blocking questions
- **STOPS** and waits for you to fill in answers
- You notify when complete: "Answers provided for [feature name] PRD"
- Skill may ask follow-up questions (FQ-X) if needed
- PRD generation only proceeds after all questions answered

## Before You Start

Gather:
- ✅ Context (background, business objectives, current state)
- ✅ Research (user research, competitive analysis, constraints)
- ✅ Stakeholder input (goals, success criteria, decisions)
- ✅ Scope clarity (in/out of scope, dependencies)

**Note:** Missing info is OK - skill will ask blocking questions

## PRD Type Quick Selection

| Type | When | Includes | Output |
|------|------|----------|--------|
| **Strategic** | Major initiative needing approval | Background, problem, plan, metrics | High-level PRD + future breakdown |
| **Implementation** | Approved feature ready for dev | Detailed req, stories, criteria | Detailed PRD for engineering |
| **Simple** | Minor enhancement | Essentials only | Concise PRD |

## TRD Decision - 30 Second Rule

**Create TRD if ANY:**
- Complex data models
- Architecture changes
- Multiple integrations
- Performance benchmarks needed
- Security implementation details
- Infrastructure changes
- Data migration plans
- Technical diagrams needed

**Otherwise:** Embed technical requirements in PRD

## File Locations

- Strategic: `/docs/planning/PRD-[name]-strategic.md`
- Implementation: `/docs/requirements/PRD-[name].md`
- Simple: `/docs/features/PRD-[name]-simple.md`
- TRD: `/docs/technical/TRD-[name].md`

## Must-Have Sections (All Types)

1. ✅ Overview (what, why, business impact)
2. ✅ Goals & Success Criteria (measurable)
3. ✅ Users & User Stories (persona-driven)
4. ✅ Acceptance Criteria (testable)
5. ✅ Scope (in/out boundaries)
6. ✅ Functional Requirements & Business Rules
7. ✅ Testing Requirements
8. ✅ Dependencies & Risks
9. ✅ References

## Quality Checklist (5-Minute Review)

- [ ] **Testability:** Every acceptance criterion verifiable?
- [ ] **Unambiguity:** No vague terms (fast, easy, user-friendly, soon)?
- [ ] **Traceability:** Every requirement has source/evidence?
- [ ] **No Assumptions:** All blocking questions answered?
- [ ] Success criteria specific and measurable?
- [ ] Acceptance criteria testable?
- [ ] User stories follow format: "As [persona], I need [capability] so that [outcome]"?
- [ ] Business rules clear?
- [ ] Dependencies with mitigation?
- [ ] Risks with impact/probability?
- [ ] Open questions documented (non-blocking only)?
- [ ] TRD decision appropriate?
- [ ] Evidence citations in Notes section?

## Common Patterns

**User Story Format:**
```
As [persona name],
I need [specific capability],
so that [clear outcome/benefit]
```

**Acceptance Criteria:**
```
- [ ] Specific, testable requirement
- [ ] Edge case handling
- [ ] Error scenario coverage
```

**Business Rule:**
```
[Rule]: When [condition], then [behavior]
[Rationale]: [Why this rule exists]
```

## Tips

- 🎯 Start with user value, not technical solution
- 🎯 Be specific - vague = rework
- 🎯 Cover edge cases upfront
- 🎯 Link, don't duplicate content
- 🎯 Mark open questions with owners/deadlines
- 🎯 One PRD = one feature/epic

## Anti-Patterns

- ❌ Technical design in PRD (use TRD)
- ❌ Multiple features in one PRD
- ❌ Ambiguous acceptance criteria
- ❌ Missing dependencies
- ❌ Unanswered blocking questions
- ❌ Making assumptions without validation
- ❌ Vague terms (fast, easy, soon, later)
- ❌ Requirements without evidence/source

## Related Skills

- `docs-management` - Update/maintain PRDs
- `ba-toolkit:visual-workflow-diagram-generator` - Create flow diagrams
- `/gsd:plan-phase` - Break into implementation phases

## Need Help?

See full documentation: `.claude/skills/prd-builder/SKILL.md`
