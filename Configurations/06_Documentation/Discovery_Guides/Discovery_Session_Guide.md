# Configuration Management - Discovery Session Guide

**Date:** December 9, 2025
**Attendees:** Aksana, Aaron, Stone/Absor, Devon, Adam
**Duration:** 90-120 minutes recommended
**Objective:** Understand current pain points, workflows, and requirements for new configuration management system

---

## Session Structure

### Part 1: Understanding the WHY (20-30 min)
### Part 2: Current State & Pain Points (20-30 min)
### Part 3: Desired Future State (20-30 min)
### Part 4: Technical Deep Dive (20-30 min)
### Part 5: Migration & Risks (10-20 min)

---

## PART 1: Understanding the WHY

**Goal:** Understand the business drivers and pain points motivating this change

### Opening Questions

1. **"Let's start with WHY - what are the biggest problems with how configuration management works today?"**
   - Listen for: pain points, incidents, time waste, business impact
   - Let them talk freely - don't interrupt
   - Take notes on specific examples

2. **"Can you tell me about a recent incident or situation where the current config system caused problems?"**
   - Get specific stories (e.g., the Wednesday outage, the "Pants" incident)
   - Understand business impact: downtime, customer complaints, manual work
   - Ask: "How much time did it take to resolve?" "What was the business impact?"

3. **"What prompted you to prioritize this as your #1 project for 2026?"**
   - Is it scaling issues?
   - Customer complaints?
   - Internal efficiency?
   - Risk/reliability concerns?
   - New business opportunities being blocked?

4. **"If we successfully rebuild this system, what changes for your business?"**
   - Faster customer onboarding?
   - Fewer support tickets?
   - Ability to sell new products?
   - Reduced risk of outages?
   - Time savings for team?

5. **"What happens if we DON'T fix this system? What's the risk?"**
   - Understand urgency
   - Understand consequences

### Expected Insights to Capture:

- [ ] Primary pain points ranked by severity
- [ ] Business impact of current system (time, money, risk)
- [ ] What's blocking them from growing/succeeding
- [ ] Urgency and consequences of inaction
- [ ] Success metrics (how will they measure if new system is better?)

---

## PART 2: Current State & Pain Points

**Goal:** Document how config management works today and where it breaks down

### Configuration Levels & Hierarchy

6. **"Walk me through how configurations work today - what are all the different levels or types of configs you manage?"**
   - Global configs
   - Model-specific configs
   - Carrier-specific configs
   - Service plan configs
   - Company/customer configs
   - Device-level configs
   - Custom configs

7. **"Show me in the system - where do you go to manage each type of configuration?"**
   - Have them screen share and walk through the UI
   - Document: where they click, what screens they use
   - Note: what's confusing, what's hidden, what requires workarounds

8. **"How do these different config levels interact or override each other today?"**
   - Is there a clear hierarchy?
   - What happens when configs conflict?
   - Are the rules clear or confusing?

### Current Workflows

9. **"Walk me through a typical scenario: A new customer (like Altec) needs custom printer translation rules. What do you do today?"**
   - Step-by-step workflow
   - What screens do you use?
   - Where do you create/edit the config?
   - How do you test it?
   - How do you deploy it?
   - What can go wrong?

10. **"What about when you need to make a global change across all devices? How do you do that today?"**
    - What's the process?
    - How long does it take?
    - What's risky about it?
    - How do you validate it worked?

11. **"How do you handle model-specific or carrier-specific configurations today?"**
    - Show examples
    - What's the workflow?

12. **"What can customers see and edit themselves? What can't they?"**
    - Device-level settings?
    - What's exposed in their portal?
    - What do they try to edit that causes problems?

### Pain Points Deep Dive

13. **"What are the most time-consuming or frustrating aspects of managing configs today?"**
    - Listen for: repetitive work, manual processes, error-prone steps
    - Ask: "How much time per week do you spend on config management?"

14. **"Tell me about the 'Browse Configurations' section and the incident with the customer downloading the config file."**
    - What happened?
    - What was exposed that shouldn't have been?
    - What's the security/risk concern?
    - What should customers NOT be able to see/do?

15. **"You mentioned the Wednesday outage where bad config data brought the system down. Tell me more about that."**
    - What exactly happened?
    - What made it hard to diagnose?
    - How long did it take to fix?
    - What would have made it easier to catch/fix?
    - What safeguards do you want?

16. **"What happens when a configuration mistake is made? How do you detect and fix it?"**
    - How quickly do you know something's wrong?
    - What tools help you diagnose?
    - Can you easily rollback?
    - Is there an audit trail?

17. **"Are there configuration changes you avoid making because they're too risky or time-consuming?"**
    - What's being held back?
    - What opportunities are you missing?

### Expected Insights to Capture:

- [ ] Complete list of configuration levels/types
- [ ] Current workflow documentation (step-by-step)
- [ ] UI pain points and confusion areas
- [ ] Time spent on config management (weekly estimate)
- [ ] Specific incidents and their root causes
- [ ] Security/access control gaps
- [ ] Missing features or workarounds being used

---

## PART 3: Desired Future State

**Goal:** Understand their vision for how config management should work

### The Vision

18. **"Adam, you mentioned you have a 'very detailed vision' for this. Walk us through it."**
    - Let Adam describe his ideal system
    - Don't interrupt - just listen and take notes
    - Ask clarifying questions after

19. **"In your ideal world, what does the workflow look like when you need to add custom configs for a new customer?"**
    - Step by step
    - What screens do you see?
    - How is it organized?
    - What validations or safeguards exist?

20. **"Where should different config management happen? On company pages? Service plan pages? A dedicated config management section?"**
    - Refer to how service plans evolved
    - What makes most sense for their workflows?

21. **"You mentioned you DON'T want just a text editor. What DO you want?"**
    - Structured forms?
    - Dropdowns and validations?
    - Visual hierarchy display?
    - What level of flexibility vs. guardrails?

### User Roles & Access

22. **"Who should be able to edit what?"**
    - Admins: what can they edit?
    - Sales/Support staff: what can they edit?
    - Customers: what can they edit?
    - Are there different admin levels?

23. **"What should customers absolutely NOT be able to do or see?"**
    - Download raw config files?
    - Edit certain parameters?
    - See other customers' configs?

### Parameter Organization

24. **"Let's talk about config parameters. Can you show me examples of the different types?"**
    - Global parameters (show examples)
    - Model-specific parameters
    - Carrier-specific parameters
    - Company-specific parameters
    - Device-specific parameters

25. **"Which parameters should be configurable at which levels?"**
    - Start listing them out
    - Note: This might be too detailed for discovery - may need separate session
    - Goal: Understand if they've thought this through or if we need to help them define it

26. **"Are there parameters that should NEVER be overridden at lower levels?"**
    - Security settings?
    - Critical system parameters?

### Validation & Safety

27. **"What kinds of validations or safety checks do you want?"**
    - Prevent invalid parameter combinations?
    - Warning before making dangerous changes?
    - Require approval for certain changes?
    - Testing/staging environment?

28. **"How should the system handle conflicts or invalid configurations?"**
    - Prevent saving invalid configs?
    - Show warnings but allow?
    - Automatic validation checks?

29. **"What visibility do you need into config changes?"**
    - Audit trail: who changed what and when?
    - Ability to compare configs?
    - Ability to see full config for a device (after all overrides)?

### Expected Insights to Capture:

- [ ] Adam's vision documented in detail
- [ ] Desired workflows for common scenarios
- [ ] UI/UX preferences and examples
- [ ] Access control requirements by role
- [ ] Validation and safety requirements
- [ ] Audit and visibility needs

---

## PART 4: Technical Deep Dive

**Goal:** Understand technical requirements and constraints

### Current System Architecture

30. **"How are configs stored today?"**
    - Files on disk?
    - Database?
    - S3?
    - Show me the current structure

31. **"How do devices receive their configurations today?"**
    - On check-in?
    - On-demand?
    - Pushed updates?
    - What's the flow from config creation to device receiving it?

32. **"What's the relationship between configs and other entities?"**
    - Service plans → configs?
    - Device models → configs?
    - Manufacturers → configs?

### Data Structure Requirements

33. **"Let's talk about the data model. In your vision, how would configs be structured?"**
    - Hierarchical?
    - Template-based?
    - Key-value pairs?
    - JSON? XML? Custom format?

34. **"How many config files or config entities exist today?"**
    - Get a sense of scale
    - How many global configs?
    - How many company-specific configs?
    - How many custom configs?

35. **"Are there any configs that are particularly complex or would be challenging to migrate?"**
    - Edge cases
    - Legacy configs
    - Special handling needed?

### Integration Points

36. **"What other parts of the system interact with configurations?"**
    - Device check-in service?
    - Billing system?
    - Notifications/alerts?
    - Reporting?

37. **"How do config changes affect device behavior?"**
    - Immediate effect?
    - Next check-in?
    - Requires device restart?

### Performance & Scale

38. **"How many devices are checking in and pulling configs?"**
    - Current scale
    - Expected growth
    - Performance requirements

39. **"You mentioned the database tables are huge and slow. Which tables? What's the impact?"**
    - Size concerns
    - Query performance issues
    - How does this relate to configs?

### Expected Insights to Capture:

- [ ] Current technical architecture documented
- [ ] Storage and retrieval mechanisms
- [ ] Data model preferences
- [ ] Scale and performance requirements
- [ ] Integration points and dependencies
- [ ] Technical constraints or limitations

---

## PART 5: Migration & Risks

**Goal:** Plan for safe transition from old to new system

### Migration Strategy

40. **"You mentioned you want to migrate gradually, not a hard cutover. Tell me more about what you're envisioning."**
    - Config-by-config migration?
    - Level-by-level (global first, then model, etc.)?
    - Company-by-company?
    - Feature-by-feature?

41. **"What would give you confidence that a migrated config is working correctly?"**
    - Testing approach?
    - Validation steps?
    - Monitoring?
    - Rollback plan?

42. **"How long can the old and new systems run in parallel?"**
    - Weeks? Months?
    - Any constraints?

43. **"What's your biggest fear about this migration?"**
    - Loss of configs?
    - Devices going offline?
    - Customer impact?
    - Business disruption?

### Phased Approach

44. **"What would a 'Phase 1' look like for you? What's the minimum viable functionality?"**
    - What's most critical to get working first?
    - What can wait for later phases?

45. **"Are there any parts of config management that are lower risk to start with?"**
    - Device-level configs first?
    - Start with one company as pilot?
    - Test with one device model first?

### Testing & Validation

46. **"How do you test config changes today?"**
    - Test devices?
    - Staging environment?
    - Beta program?

47. **"What testing capabilities do you need for the new system?"**
    - Sandbox environment?
    - Preview changes before applying?
    - Test configs without affecting production?

### Risks & Concerns

48. **"What are your biggest concerns about this project?"**
    - Timeline?
    - Complexity?
    - Business disruption?
    - Cost?

49. **"What would make this project fail in your eyes?"**
    - What are the deal-breakers?
    - What must we avoid?

50. **"What would make this project a huge success?"**
    - What are the must-haves?
    - What would exceed your expectations?

### Expected Insights to Capture:

- [ ] Migration strategy preferences
- [ ] Risk tolerance and concerns
- [ ] Phasing approach and priorities
- [ ] Testing and validation needs
- [ ] Success criteria and deal-breakers
- [ ] Timeline expectations

---

## Post-Session Actions

### Immediate Next Steps

1. **Synthesize notes** into requirements document
2. **Create visual diagrams:**
   - Current state workflow
   - Future state workflow
   - Configuration hierarchy
   - Data model
3. **Identify gaps** - what questions still need answers?
4. **Schedule follow-up sessions** if needed:
   - Parameter inventory session
   - Technical architecture session
   - UI/UX design session
5. **Share back with client** for validation
6. **Begin high-level estimation** and phasing plan

### Documentation to Create

- [ ] Requirements Document (update existing)
- [ ] Current State Workflow Diagram
- [ ] Future State Workflow Diagram
- [ ] Configuration Hierarchy Diagram
- [ ] User Stories / Use Cases
- [ ] Technical Architecture Proposal
- [ ] Migration Strategy Document
- [ ] Project Phasing Plan
- [ ] Risk Assessment

---

## Tips for Running the Session

### Do's:
- ✅ **Let them talk** - especially in the "WHY" section
- ✅ **Ask "show me"** - have them demonstrate in the system
- ✅ **Dig into specific examples** - concrete stories reveal truth
- ✅ **Take screenshots** when they show things in the UI
- ✅ **Repeat back what you heard** to confirm understanding
- ✅ **Take notes on exact words/phrases** they use
- ✅ **Watch for emotion** - frustration reveals pain points
- ✅ **Ask "why" multiple times** to get to root causes

### Don'ts:
- ❌ **Don't jump to solutions** too quickly
- ❌ **Don't interrupt** when they're describing problems
- ❌ **Don't dismiss concerns** even if they seem small
- ❌ **Don't commit to timeline/scope** in discovery
- ❌ **Don't assume** you understand - ask clarifying questions
- ❌ **Don't skip the "why"** to jump to technical details
- ❌ **Don't let it become a technical spec session** too early

### If You Get Stuck:
- "Tell me more about that..."
- "Can you show me an example?"
- "Walk me through what you do today..."
- "What would you do if you could do anything?"
- "What am I not asking that I should be?"

---

## Key Metrics to Define

Help them think about success metrics:

- **Time savings:** "How much time per week/month do you spend managing configs today? What's acceptable?"
- **Error reduction:** "How many config-related incidents per month? What's the goal?"
- **Speed:** "How long does it take to onboard a new customer with custom configs? What should it be?"
- **Customer satisfaction:** "How many support tickets relate to configs? What's the goal?"
- **Business impact:** "How does this enable new revenue or customers?"

---

## Follow-Up Questions List

Keep a running list of questions that come up during the session that need follow-up:

1.
2.
3.

---

## Session Recording

**Note:** Ask permission to record the session for reference. If yes:
- [ ] Recording started
- [ ] Recording saved to: ________________
- [ ] Transcript generated (if using tool like Otter.ai)

---

## Attendee Notes Section

### Devon's Comments:
-

### Adam's Comments:
-

### Other Stakeholders:
-

---

## Open Questions After Session

_To be filled in after the discovery session completes_

1.
2.
3.

