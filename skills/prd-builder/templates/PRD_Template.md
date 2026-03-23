# Product Requirements Document - Consolidated Template
## [Feature/Project Name]

**Document Version:** 1.0
**Date:** [Date]
**Author:** [Author Name/Team]
**Status:** [Draft | In Review | Approved | In Progress | Done]
**Related Tickets:** [JIRA-XXXX](link)
**Document Owner:** [Name]
**Last Updated:** [DD.MM.YYYY]

---

## Table of Contents

1. [Overview](#overview)
2. [Background and Problem Statement](#background-and-problem-statement) _(Strategic Only)_
3. [Goals and Success Criteria](#goals-and-success-criteria)
4. [Users and User Stories](#users-and-user-stories)
5. [Acceptance Criteria](#acceptance-criteria)
6. [Scope](#scope)
7. [Functional Requirements and Business Rules](#functional-requirements-and-business-rules)
8. [Technical Requirements](#technical-requirements)
9. [UI/UX Requirements](#uiux-requirements) _(if applicable)_
10. [Testing Requirements](#testing-requirements)
11. [Dependencies and Risks](#dependencies-and-risks)
12. [Implementation Plan](#implementation-plan) _(Strategic Only)_
13. [Success Metrics](#success-metrics) _(Strategic Only)_
14. [Open Questions](#open-questions)
15. [References](#references)
16. [Notes](#notes)

---

## Overview

_2-3 sentences: What is this feature? Why does it matter to the business/users?_

[Describe what the feature is, why it's needed, and what problem it solves]

### Key Features
- **Feature 1**: [Brief description]
- **Feature 2**: [Brief description]
- **Feature 3**: [Brief description]

### Business Impact
- [Impact on revenue, costs, users, etc.]
- [How this aligns with business goals]
- [Expected outcomes]

---

## Background and Problem Statement
_**Strategic Planning Only** - Omit for implementation-focused documents_

### Current State
[Describe how things work today and what the current limitations are]

### Problems
**Problem 1: [Problem Title]**
- [Specific issue or pain point]
- [Who is affected and business/user impact]

**Problem 2: [Problem Title]**
- [Specific issue or pain point]
- [Who is affected and business/user impact]

### Impact if Not Addressed
[Consequences of not solving this problem - include metrics if available]

---

## Goals and Success Criteria

### Primary Goals
1. **[Goal 1]**: [Description of what we want to achieve]
2. **[Goal 2]**: [Description of what we want to achieve]
3. **[Goal 3]**: [Description of what we want to achieve]

### Success Criteria
_Measurable outcomes that define success_

- ✅ [Specific, measurable outcome 1]
- ✅ [Specific, measurable outcome 2]
- ✅ [Specific, measurable outcome 3]

### Out of Scope
- ❌ [Feature/capability explicitly not included]
- ❌ [Future enhancement deferred]
- ❌ [Related but separate concern]

_See [Product Roadmap](link) for future enhancements_

---

## Users and User Stories

### Primary Users

**[User Persona 1]** - [Link to persona document]
- **Role**: [Job title or role description]
- **Need**: [What they need from this feature]
- **Pain Point**: [Current problem they face]
- **Benefit**: [How this feature helps them]

**[User Persona 2]** - [Link to persona document]
- **Role**: [Job title or role description]
- **Need**: [What they need from this feature]
- **Pain Point**: [Current problem they face]
- **Benefit**: [How this feature helps them]

### User Stories

**As [Primary Persona Name]:**
- As [persona], I need [capability] so that [outcome]
- As [persona], I need [capability] so that [outcome]
- As [persona], I need [capability] so that [outcome]

**As [Secondary Persona Name]:**
- As [persona], I need [capability] so that [outcome]
- As [persona], I need [capability] so that [outcome]

---

## Acceptance Criteria

- [ ] [Specific, testable requirement]
- [ ] [Specific, testable requirement]
- [ ] [Specific, testable requirement]
- [ ] [Edge case handling requirement]
- [ ] [Edge case handling requirement]

---

## Scope

### In Scope

**[Category 1 - e.g., UI Changes]**
- ✅ [Specific deliverable or change]
- ✅ [Specific deliverable or change]

**[Category 2 - e.g., Backend Changes]**
- ✅ [Specific deliverable or change]
- ✅ [Specific deliverable or change]

**[Category 3 - e.g., Data/Integration]**
- ✅ [Specific deliverable or change]
- ✅ [Specific deliverable or change]

---

## Functional Requirements and Business Rules

### FR-1: [Requirement Category]

**FR-1.1: [Specific Requirement]**
- [Detailed description of what the system must do]
- [Any constraints or rules]
- [Expected behavior]

**Business Rules:**
- [Rule describing when/how this behavior occurs]
- [Rule describing validation or constraint for this requirement]

**FR-1.2: [Specific Requirement]**
- [Detailed description of what the system must do]
- [Any constraints or rules]
- [Expected behavior]

**Business Rules:**
- [Rule describing workflow or state transitions]
- [Rule describing authorization or access control]

### FR-2: Data Validation and Integrity

**FR-2.1: Input Validation**
- [Validation rules for user inputs]
- [Format requirements]
- [Error handling approach]

**Business Rules:**
- [Rule describing data handling or processing policy]
- [Rule describing compliance requirements]

---

## Technical Requirements

> **NOTE:** For complex features, create a separate **Technical Requirements Document (TRD)**. Reference it here and include only high-level requirements in this section.
>
> _See [Technical Requirements Document](link-to-TRD) for detailed specifications_

### Performance & System
- [Response time requirement with measurable target]
- [Throughput/scalability requirement]
- [Browser/platform support requirements]

### Integration
- [External system/service integration name and purpose]
- [API requirements and data exchange formats]

### Security & Compliance
- [Authentication/authorization requirement]
- [Data encryption/transmission requirement]
- [Regulatory compliance (HIPAA, GDPR, etc.)]
- [Audit/logging requirement]

_See [Security Standards](link) and [Technical Standards](link) for full requirements_

### Data Model Overview
> **NOTE:** For complex data models, create detailed specifications in the TRD. Include only essential information here.

**[Primary Entity Name]**
| Field | Type | Required | Validation | Notes |
|-------|------|----------|------------|-------|
| id | UUID | Yes | Auto-generated | Primary key |
| field1 | String(100) | Yes | Max 100 chars | Description |
| status | Enum | Yes | See below | State machine |

**[Status/Type Enum]**
- STATE_ONE → Description of what this state means
- STATE_TWO → Description of what this state means

_See [Data Models Documentation](link) for full schema and relationships_

---

## UI/UX Requirements
_Omit if no UI changes. See [Mock Up/Design Files](link) for detailed designs_

### Key Interface Components
- **[Component 1]**: [Brief description and purpose]
- **[Component 2]**: [Brief description and purpose]

### Key User Flows
- **[Flow 1]**: [Critical path description]
- **[Flow 2]**: [Alternative/error path description]

### Accessibility
- [WCAG compliance level required]
- [Specific accessibility considerations]

---

## Testing Requirements

### Test Plan Overview
**Testing Phases:**
1. Unit Testing (Development team)
2. Integration Testing (QA team)
3. User Acceptance Testing (UAT) (Business stakeholders)
4. Regression Testing (QA team)
5. Performance Testing _(if applicable)_

### Key Test Scenarios

**IT-1: [Integration Test Scenario Name]**
1. [Setup/preconditions]
2. [Action steps]
3. **Verify:** [Expected outcome]

**UAT-1: [User Acceptance Test Scenario Name]**
- **Persona:** [User type]
- **Scenario:** [Real-world use case description]
- **Steps:** [User actions]
- **Success Criteria:** [What indicates success]

**RT-1: [Regression Test Scenario Name]**
- **Verify:** [Existing feature still works correctly]
- **Verify:** [No unintended side effects]

### Testing Notes
- [Testing dependencies on other features]
- [Special environment requirements]
- [Performance testing thresholds]

---

## Dependencies and Risks

### Dependencies

**Must Exist Before Development:**
1. **[Dependency Name]** - [Description] | **Mitigation:** [Fallback plan]
2. **[External System/Service]** - [Description] | **Mitigation:** [Fallback plan]

**Required for Testing:**
- [Related Feature/Epic](link) - [Why needed for testing]

**Integrates With:**
- [System/Service](link) - [Integration purpose]

### Risks

**[HIGH/MEDIUM/LOW] RISK: [Risk Title]**
- **Description:** [What could go wrong]
- **Impact:** [High/Medium/Low] - [Consequences if occurs]
- **Probability:** [High/Medium/Low]
- **Mitigation:** [Action to reduce risk and contingency plan]

---

## Implementation Plan
_**Strategic Planning Only** - Omit for implementation-focused documents_

### Phase 1: [Phase Name] (Duration)
- [ ] [Specific deliverable or task]
- [ ] [Specific deliverable or task]

### Phase 2: [Phase Name] (Duration)
- [ ] [Specific deliverable or task]
- [ ] [Specific deliverable or task]

**Total Estimated Timeline:** [X weeks/months]

### Rollout Strategy
- **Deployment Approach**: [All at once / Phased / Feature flag / Canary]
- **User Communication**: [How users will be informed]
- **Training Needed**: [Documentation, training sessions, etc.]
- **Rollback Plan**: [How to revert if issues occur]

---

## Success Metrics
_**Strategic Planning Only** - Omit for implementation-focused documents_

### Key Performance Indicators (KPIs)

**Performance Metrics:**
- [Metric name]: Baseline [X] → Target [Y]

**User Adoption Metrics:**
- [Metric name]: Target [X]

**Business Metrics:**
- [Metric name]: Baseline [X] → Target [Y]

---

## Open Questions

### 1. [Question Category]
- **Question:** [Specific question]
- **Options:**
  - **Option A**: [Description with pros/cons]
  - **Option B**: [Description with pros/cons]
- **Decision Needed By**: [Date or milestone]
- **Decision Owner**: [Who will decide]
- **Status**: [Open / Decided / Deferred]

---

## References

### Related Features
- [Related Feature 1](link) - [Relationship description]
- [Related Feature 2](link) - [Relationship description]

### Supporting Documentation
- [Technical Requirements Document (TRD)](link)
- [User Persona Documents](link)
- [Design/Mockup Files](link)
- [Data Models Documentation](link)
- [Technical Standards](link)
- [Security Standards](link)
- [Product Roadmap](link)

---

## Notes

### Evidence Sources
- [List transcript files and source documents used]
- [User research findings]

### Key Decisions
- [Decision made with rationale and reference]
- [Decision made with rationale and reference]

### Research Applied
- **[Standard/Compliance]**: [Specific requirement derived from research]

### Q&A Document
- [Link to Q&A document if available]

---

END OF DOCUMENT

---

## Template Usage

This template is designed to be used with the **prd-builder skill** in Claude Code.

**To generate a PRD using this template:**
1. Gather context and research about your feature/project
2. Invoke the skill: `Create a PRD for [feature name]`
3. The skill will guide you through an interactive process
4. A complete PRD will be generated based on your project needs

**For manual usage:**
- Use `/docs-management` skills to maintain PRDs as living documents
- See `.claude/skills/prd-builder/SKILL.md` for detailed guidance on PRD types and decision logic
- See `.claude/skills/prd-builder/templates/template-reference.md` for usage scenarios and quality checklist
