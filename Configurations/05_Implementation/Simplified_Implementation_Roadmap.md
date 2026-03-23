# Implementation Roadmap - Configuration Management System
## Milestone-Based Delivery Plan

**Document Purpose:** High-level milestone roadmap for client alignment and team planning
**Status:** Proposed - Awaiting Approval
**Last Updated:** March 5, 2026
**Document Owner:** Aksana Rahouski

---

## Executive Summary

This roadmap breaks down the Configuration Management System into **value-driven milestones** that deliver business benefits incrementally. Each milestone solves specific operational pain points and builds the foundation for subsequent capabilities.

The system is built around a **six-layer configuration hierarchy** defined in the PRD:

```
Global → Model → Carrier → Service Plan → Company → Device
```

Each layer is introduced as its own milestone, in hierarchy order, so each phase builds directly on the previous one. Conditional rules (Two-Way, Three-Way, Four-Way) are introduced after all six layers are established.

**Approach:**
- Deliver working functionality incrementally
- Solve highest-impact pain points first
- Validate with real devices before expanding scope
- Maintain safe rollback throughout migration

---

## Pain Points & Priority

### Critical Priority (Solve in Phase 1)
1. **Global configuration changes take 5+ hours** - Manual file editing across 200 config files
2. **Configuration errors cause device loops** - No version control or safe testing
3. **No safe testing mechanism** - All changes go directly to production

### High Priority (Solve in Phases 2-3)
4. **Model-specific settings applied incorrectly** - Wrong parameters on wrong device types
5. **Carrier-specific settings mixed up** - APN and carrier configs confused

### Medium-High Priority (Solve in Phase 4)
6. **Service plan configurations not isolated** - Feature sets and data limits tied to billing plan have no dedicated configuration layer

### Medium Priority (Solve in Phases 5-6)
7. **No company-level customization** - Cannot set company-wide defaults
8. **Complex configuration rules** - Multi-factor conditional logic (Two-Way, Three-Way, Four-Way rules)

### Lower Priority (Future Enhancement)
9. **Customer self-service portal** - End-user configuration access (Company layer)
10. **Advanced versioning workflow** - Draft/review/approve process
11. **Automated testing & validation** - Multi-level defense system

---

## Milestone Overview

| Milestone | Layer Introduced | Business Value | Complexity |
|-----------|-----------------|----------------|------------|
| **Phase 1: Global Layer** | Global | 95% time reduction for global changes | Low |
| **Phase 2: Model Layer** | Model | Prevent model-specific config errors | Medium |
| **Phase 3: Carrier Layer** | Carrier | Separate carrier-specific settings | Medium |
| **Phase 4: Service Plan Layer** | Service Plan | Isolate plan-based feature/data configs | Medium |
| **Phase 5: Company Layer** | Company | Enable company-wide customizations | Medium-High |
| **Phase 6: Conditional Rules** | Rules Engine | Handle complex multi-factor logic | High |
| **Phase 7: Versioning & Workflow** | Future | Enterprise change management | High |

---

## Phase 1: Global Layer (MVP)
**Priority:** CRITICAL
**Risk:** LOW
**PRD Reference:** Layer 1 of 6 — Global Layer

### Business Objectives
- **Primary:** Eliminate 5+ hour manual process for global configuration changes
- **Secondary:** Establish the foundation layer of the six-layer hierarchy
- **Tertiary:** Validate the approach with minimal risk before expanding scope

### Capabilities Delivered
1. **Configuration Parameter Library**
   - Centralized catalog of all 600+ parameters
   - Parameter definitions, validation rules, default values
   - Layer availability matrix (which layers can set each parameter)

2. **Global Layer Configuration Interface**
   - Web-based admin interface for editing global default values
   - Change audit trail (who changed what and when)
   - Parameters flagged as Global-only vs. available at lower layers

3. **Config Resolution Engine (Foundation)**
   - Six-layer hierarchy engine built and ready (Global through Device)
   - Phase 1 populates Global layer; remaining layers resolve to empty/device overrides
   - Source attribution for every resolved parameter value

4. **Device Override Preservation**
   - Existing device-specific overrides (Wi-Fi, equipment) respected
   - Device overrides take precedence over Global defaults
   - No disruption to currently working device configurations

5. **Controlled Migration System**
   - Test with 10 devices, then expand gradually
   - Old system continues running in parallel
   - Instant rollback capability at any stage

### Success Criteria
- Admin can change global DNS servers in under 5 minutes (vs. 5+ hours today)
- Changes propagate to 100+ devices successfully
- Zero device update loops or configuration errors
- Old system remains operational for unmigrated devices
- Admin team trained and comfortable with new interface

### Business Value
- **Time savings:** 95% reduction for global changes (5 hours → 5 minutes)
- **Error reduction:** Eliminate manual file editing mistakes
- **Operational agility:** Respond to infrastructure changes immediately
- **Risk reduction:** Gradual, controlled rollout with rollback

---

## Phase 2: Model Layer
**Priority:** HIGH
**Risk:** LOW
**PRD Reference:** Layer 2 of 6 — Model Layer

### Business Objectives
- **Primary:** Prevent model-specific configurations from being applied to wrong device types
- **Secondary:** Simplify configuration management across different device models (i-22, 4100, 4500, etc.)

### Capabilities Delivered
1. **Model Layer Configuration Interface**
   - Define default configurations per device model
   - Model-specific settings override Global layer defaults
   - Display inherited Global values (grayed out with source label)

2. **Model-Aware Parameter Definitions**
   - Parameters marked as available at Model layer (per schema)
   - Validation ensures model-appropriate settings only

3. **Updated Resolution Engine**
   - Resolution order: Device → Company → Service Plan → Carrier → **Model** → Global
   - Model layer values override Global; Device overrides still take precedence

### Success Criteria
- i-22 devices receive i-22-specific parameters only
- 4100 devices receive 4100-specific parameters only
- Model-specific changes affect only devices of that model
- Global changes still affect all devices appropriately

### Business Value
- **Error prevention:** Stop applying incompatible settings across device types
- **Efficiency:** Model-specific changes don't require touching all devices
- **Scalability:** Easy to onboard new device models

---

## Phase 3: Carrier Layer
**Priority:** HIGH
**Risk:** LOW
**PRD Reference:** Layer 3 of 6 — Carrier Layer

### Business Objectives
- **Primary:** Separate carrier-specific settings (APN, network parameters) into their own layer
- **Secondary:** Enable rapid carrier onboarding and carrier configuration changes

### Capabilities Delivered
1. **Carrier Layer Configuration Interface**
   - Define configurations per carrier (Verizon, AT&T, T-Mobile, etc.)
   - Carrier-specific settings override Model and Global defaults
   - Display inherited values with source labels

2. **Carrier-Aware Validation**
   - Ensure carrier-appropriate settings are applied to correct devices
   - Prevent carrier configuration mix-ups (e.g., Verizon APN on AT&T device)

3. **Updated Resolution Engine**
   - Resolution order: Device → Company → Service Plan → **Carrier** → Model → Global
   - Carrier layer overrides Model and Global; device/company overrides still take precedence

### Success Criteria
- Verizon devices receive Verizon APN settings only
- AT&T devices receive AT&T APN settings only
- Carrier-specific changes affect only devices on that carrier
- New carriers can be onboarded without touching other configurations

### Business Value
- **Correctness:** Right carrier settings on right devices, automatically
- **Agility:** Quick carrier onboarding and carrier-wide config changes
- **Scale:** Manage multi-carrier fleet efficiently

---

## Phase 4: Service Plan Layer
**Priority:** MEDIUM-HIGH
**Risk:** LOW-MEDIUM
**PRD Reference:** Layer 4 of 6 — Service Plan Layer

### Business Objectives
- **Primary:** Isolate service plan-specific configuration into its own dedicated layer
- **Secondary:** Enable plan-based feature sets and data limit enforcement through configuration
- **Tertiary:** Establish the Service Plan layer as the foundation for Three-Way and Four-Way conditional rules (introduced in Phase 6)

### Capabilities Delivered
1. **Service Plan Layer Configuration Interface**
   - Define configurations per service plan (e.g., ATM plan, FWA plan, custom tiers)
   - Service plan settings override Carrier, Model, and Global defaults
   - Display inherited values with source labels (Carrier, Model, Global)
   - Contextual access to conditional rules involving this service plan (added in Phase 6)

2. **Plan-Based Parameter Definitions**
   - Parameters marked as available at Service Plan layer (per schema)
   - Examples: data limits, feature flags, traffic thresholds, plan-specific feature toggles

3. **Updated Resolution Engine**
   - Full resolution order: Device → Company → **Service Plan** → Carrier → Model → Global
   - Service Plan layer overrides Carrier, Model, and Global
   - Device and Company layers still take highest precedence

4. **Plan Change Propagation**
   - When a Service Plan layer config changes, only devices on that plan are affected
   - Other devices unaffected regardless of model or carrier

### Success Criteria
- Devices on the ATM plan receive ATM-specific parameters (data limits, feature flags)
- Devices on different plans receive their plan-specific parameters independently
- Service plan configuration changes affect only devices assigned to that plan
- Changing a device's assigned service plan correctly picks up the new plan's configuration
- Global, Model, and Carrier layer changes still propagate correctly through Service Plan layer

### Business Value
- **Correctness:** Plan-specific features and limits enforced through configuration, not manual overrides
- **Flexibility:** New service plans can define their own configuration baseline without touching other layers
- **Foundation for rules:** Service Plan layer is required before Three-Way and Four-Way conditional rules can be implemented (Phase 6)

---

## Phase 5: Company Layer
**Priority:** MEDIUM-HIGH
**Risk:** MEDIUM
**PRD Reference:** Layer 5 of 6 — Company Layer

### Business Objectives
- **Primary:** Enable company-specific configuration defaults for the full device fleet of each customer
- **Secondary:** Support portfolio-level customizations and parent/child company inheritance

### Capabilities Delivered
1. **Company Layer Configuration Interface**
   - Define configurations per company (customer portfolio-wide settings)
   - Company-specific settings override Service Plan, Carrier, Model, and Global
   - Admin view: full access to all keys available at Company layer
   - Display inherited values with source labels

2. **Multi-Tenant Hierarchy**
   - Parent/child company relationships respected
   - Child companies inherit from parent company settings
   - Override cascade through company hierarchy

3. **Updated Resolution Engine**
   - Full resolution order: Device → **Company** → Service Plan → Carrier → Model → Global
   - Company layer is second-highest priority after Device overrides

4. **Customer Self-Service Foundation (Phase 1 Admin Only)**
   - Company layer is admin-managed only at this phase
   - Customer self-service access to Company layer deferred to future enhancement

### Success Criteria
- Company A devices receive Company A-specific settings without affecting Company B
- Child companies can inherit parent company settings
- Company-specific changes affect only devices within that company
- All previous layers (Global, Model, Carrier, Service Plan) continue functioning correctly

### Business Value
- **Customization:** Support customer-specific requirements without device-level overrides
- **White-label:** Enable portfolio company configurations
- **Efficiency:** Company-wide changes in one place instead of per-device overrides

---

## Phase 6: Conditional Rules Engine
**Priority:** MEDIUM
**Risk:** HIGH
**PRD Reference:** FR-3a — Conditional Rules Framework; 11-Level Priority Hierarchy

### Business Objectives
- **Primary:** Handle complex configuration logic where parameter values depend on multiple device attributes simultaneously
- **Secondary:** Implement Two-Way, Three-Way, and Four-Way conditional rules that override standard layer inheritance

### Capabilities Delivered
1. **Two-Way Rules (6 combinations)**
   - Rules based on any two of: Model, Carrier, Service Plan, Company
   - Sub-priority order (highest to lowest):
     1. Carrier + Service Plan (carrier data policies by plan)
     2. Model + Service Plan (model pricing tier by plan)
     3. Model + Carrier (model+carrier-specific behavior)
     4. Carrier + Company (customer carrier agreements)
     5. Model + Company (customer model configs)
     6. Service Plan + Company (customer plan exceptions)

2. **Three-Way Rules**
   - Rules based on Model + Carrier + Service Plan combinations
   - Example: All i-22 + ATT + ATM devices get `traffic_threshold=350MB`

3. **Four-Way Rules**
   - Rules based on Model + Carrier + Service Plan + Company
   - Enable customer-specific exceptions to Three-Way Rule baselines
   - Example: CORD company gets custom firewall rules overriding standard ATM plan rules

4. **Rule Management Interface**
   - Create, edit, and test rules through admin UI
   - Test rules before deploying to production
   - Track rule execution and outcomes
   - Conflict detection and resolution

5. **Full 11-Level Priority Hierarchy**
   - Level 1: Device Layer override
   - Level 2: Four-Way Rule (Model + Carrier + Service Plan + Company)
   - Level 3: Company Layer
   - Level 4: Three-Way Rule (Model + Carrier + Service Plan)
   - Level 5: Service Plan Layer
   - Level 6: Two-Way Rules (6 sub-priorities, 6.1–6.6)
   - Level 7: Carrier Layer
   - Level 8: Model Layer
   - Level 9: Global Layer
   - Level 10: Static defaults
   - Level 11: NULL / no value

### Success Criteria
- Two-Way, Three-Way, and Four-Way rules execute correctly during configuration generation
- Complex multi-factor scenarios are handled automatically without manual intervention
- No rule conflicts or ambiguous outcomes
- Rules can be tested safely before production use
- All previous layer behaviors (Phases 1–5) continue functioning correctly

### Business Value
- **Automation:** Complex configuration logic handled by rules, not manual device overrides
- **Correctness:** Fewer configuration errors from edge cases and multi-factor scenarios
- **Completeness:** All 11 levels of the priority hierarchy fully operational

**Note:** This is the most complex milestone. May be sub-phased (Two-Way first, then Three-Way + Four-Way) based on team capacity.

---

## Phase 7: Versioning & Workflow (Future)
**Priority:** LOW
**Risk:** MEDIUM
**PRD Reference:** Future enhancement

### Business Objectives
- **Primary:** Enterprise-grade change management for configuration updates
- **Secondary:** Draft, review, and approve workflow before publishing configuration changes

### Capabilities Delivered (Proposed)
1. **Configuration Versioning**
   - Save configuration changes as drafts before publishing
   - Review and approve changes before they affect devices
   - Rollback to previous configuration versions at any layer

2. **Change Approval Workflow**
   - Multi-level approval process
   - Complete change history and audit trail
   - Scheduled change deployment

3. **Testing & Validation**
   - Test configuration changes on a subset of devices before full rollout
   - Automated validation before deployment
   - Gradual rollout with monitoring

### Business Value
- **Governance:** Control over who can publish configuration changes
- **Safety:** Review changes before they affect the device fleet
- **Auditability:** Complete change history and approval trail

**Note:** This milestone is future-scoped and will be detailed based on business priorities after Phases 1–6 are complete.

---

## Migration Strategy

### Gradual Device Migration (Applies to All Phases)

**Phase-In Approach:**
1. **Pilot (10 devices)** — Monitor before expansion
2. **Expansion (100 devices)** — Monitor before further scaling
3. **Scale (1,000 devices)** — Monitor before full rollout
4. **Full rollout** — Batches of 1,000+ over time

**Safety Measures:**
- Old system remains operational throughout all phases
- Instant rollback capability at any point
- No changes to unmigrated devices
- Thorough testing at each expansion stage

---

## Risks & Mitigation

### Phase 1 Risks
**Risk:** Configuration errors impact devices during migration
**Mitigation:** Start with 10 devices, gradual expansion, instant rollback

**Risk:** Admin team adoption challenges
**Mitigation:** Training, clear documentation, demonstrate time savings immediately

### Phases 2–5 Risks
**Risk:** Layer complexity causes confusion or misconfiguration
**Mitigation:** Add one layer at a time, validate each layer fully before proceeding to next

**Risk:** Performance issues as layers accumulate
**Mitigation:** Performance testing after each milestone; resolution engine optimized with proper indexing

### Phase 4 (Service Plan Layer) Specific Risks
**Risk:** Service plan configurations overlap with carrier or company settings already set
**Mitigation:** Source attribution UI makes layer conflicts visible; admin review required before publishing

### Phase 6 Risks
**Risk:** Rule conflicts or ambiguous outcomes in the rules engine
**Mitigation:** Conflict detection built into rule management UI, testing framework, phased rollout (Two-Way first)

**Risk:** Rule engine complexity exceeds sprint capacity
**Mitigation:** Sub-phase this milestone: Two-Way rules first, then Three-Way, then Four-Way

---

## Resource Requirements (High-Level)

### Per Phase (Typical)
- **Development:** 1-2 developers per phase
- **QA/Testing:** 1 QA resource for validation
- **Admin Team:** 1-2 admins for testing and feedback
- **Project Management:** Ongoing coordination and tracking

**Note:** Detailed resource planning, effort estimation, and sprint breakdown will occur during technical planning for each phase.

---

## Success Metrics

### Operational Metrics
- Time to make global configuration changes
- Configuration error rate
- Device update loop incidents
- Admin team productivity per phase

### Technical Metrics
- Number of devices successfully migrated per phase
- Configuration generation performance (resolution engine latency)
- System uptime and reliability
- Rollback frequency (target: zero)

### Business Metrics
- Admin team satisfaction
- Operational cost reduction
- Time to onboard new device models, carriers, or service plans
- Configuration consistency across fleet

---

## Next Steps

### Immediate
1. **Client review** of milestone roadmap
2. **Alignment on priorities** — confirm phase order and service plan layer placement
3. **Budget and resource approval** for Phase 1 (Global Layer MVP)

### Following
4. **Phase 1 kickoff:**
   - Detailed technical planning and estimation
   - UI/UX design for admin interface
   - Parameter library inventory and schema design
   - Test plan development
5. **Team assignment** and sprint planning

### Ongoing
6. **Phase completion reviews** before proceeding to next phase
7. **Continuous feedback** from admin team during each phase

---

## Appendix: Phase Dependencies

```
Phase 1: Global Layer (MVP)
    ↓ Foundation layer established; resolution engine built for all 6 layers
Phase 2: Model Layer
    ↓ Model awareness added; 2nd layer in hierarchy operational
Phase 3: Carrier Layer
    ↓ Carrier awareness added; 3rd layer operational
Phase 4: Service Plan Layer
    ↓ Plan-based configuration isolated; 4th layer operational
    ↓ Required before Three-Way and Four-Way rules in Phase 6
Phase 5: Company Layer
    ↓ Full hierarchy operational (all 6 layers); customer portfolio customization enabled
Phase 6: Conditional Rules Engine
    ↓ Two-Way, Three-Way, Four-Way rules active; full 11-level priority hierarchy complete
Phase 7: Versioning & Workflow (Future)
```

**Key Principle:** Each phase introduces exactly one layer of the six-layer hierarchy in order. Conditional rules (Phase 6) require all six layers to be operational first. Phases cannot be skipped.

---

**Document Status:** High-Level Roadmap - Ready for Client Review
**Next Action:** Client approval of milestone approach and Phase 1 scope
**Document Owner:** Aksana Rahouski
**Last Updated:** March 5, 2026
