# Hierarchical Configuration Management System
## Client Presentation Deck

**Presented by:** Aksana Rahouski / Orases Team
**Date:** January 2026
**Version:** 1.5

---

## SLIDE 1: The Challenge

### Current State: Managing 100,000+ Devices with 200 Config Files

**The Problem:**
- **Hours of manual work** for simple global changes (e.g., updating DNS servers)
- **200 config files** require manual editing for every infrastructure change
- **High error risk** - manual hostname management causes device update loops
- **No flexibility** - cannot respond quickly to security threats or customer needs
- **Cannot scale** - approaching architectural limits

**Real-World Impact:**
- 5-10 hours/week on config file maintenance
- Service disruptions from config errors
- Lost deals where config flexibility was a requirement
- Cannot implement security measures quickly (e.g., disable factory reset across all devices)

---

## SLIDE 2: The Solution

### Intelligent Hierarchical Configuration Management

**Transform 200 files into a smart configuration engine**

**Core Concept:**
Set it once at the top, customize where needed

```
Global (all devices)
  ↓
Model-specific (i-22, 4100, etc.)
  ↓
Carrier-specific (Verizon, AT&T, etc.)
  ↓
Service Plan (ATM, Tier 1, etc.)
  ↓
Company-wide (customer portfolio)
  ↓
Individual Device
```

**Example:**
- Set DNS = 8.8.8.8 globally → applies to 100,000 devices
- One customer needs custom DNS = 10.1.1.1 → override at Company layer
- One special device needs DNS = 192.168.1.1 → override at Device layer

---

## SLIDE 3: Key Features

### What Makes This System Powerful

**1. Six-Layer Configuration Hierarchy**
- Global → Model → Carrier → Service Plan → Company → Device
- Set defaults high, override low
- Automatic inheritance and cascading

**2. Smart Conditional Rules**
- Handle complex business logic automatically
- 2-way, 3-way, and 4-way conditional combinations
- Example: "VZW i-22 devices enable Device Manager, but AT&T i-22 devices don't"

**3. 11-Level Priority Resolution Engine**
- Intelligent conflict resolution
- Always knows which value to use
- Complete transparency (see exactly where each value comes from)

**4. Customer Self-Service**
- Safe UI-driven configuration for end customers
- No more support tickets for simple changes
- Impossible to break devices through config errors

**5. Automated Version Control**
- System-managed versioning (no more manual hostname typing)
- Automatic device synchronization
- Complete audit trail

**6. Gradual Migration**
- Test on 5-10 devices, expand gradually
- Both systems run in parallel during transition
- Zero disruption to operations

---

## SLIDE 4: Business Impact

### The Results That Matter

**Operational Efficiency**
- **95% time reduction**: Hours → Minutes for global changes
- Update DNS across 100,000 devices in <10 minutes (vs. hours/days)
- Free up 5-10 hours/week of admin time

**Risk Reduction**
- Eliminate manual hostname errors
- Near-zero config-related incidents
- High confidence deployments

**Business Agility**
- Respond to security threats in minutes
- Infrastructure migrations become low-risk
- Support complex customer requirements without config file sprawl

**Customer Satisfaction**
- Customer self-service (70% reduction in support tickets)
- Same-day changes vs. multi-day wait
- Flexible customization competitive advantage

**Scalability**
- Support 200,000+ devices without architectural changes
- No limits on growth

---

## SLIDE 5: How It Works - Simple Example

### DNS Server Change

**OLD SYSTEM:**
1. Open 200 config files manually
2. Edit DNS value in each file
3. Type hostname/timestamp in each file (manual version tracking)
4. Save each file
5. One typo → device update loop → customer outage
6. **Time: 2-4 hours | Risk: High**

**NEW SYSTEM:**
1. Open Global Configuration page
2. Change DNS value: 8.8.8.8 → 1.1.1.1
3. Click Save
4. System automatically:
   - Updates version for all 100,000 devices
   - Identifies affected devices
   - Pushes configs on next check-in
5. **Time: <2 minutes | Risk: Near-zero**

**Exception Handling:**
- Customer XYZ needs different DNS? → Override at Company layer (30 seconds)
- One device needs special DNS? → Override at Device layer (30 seconds)

---

## SLIDE 6: How It Works - Complex Example

### Service Plan Features with Customer Exceptions

**Scenario:** ATM service plan baseline + customer-specific customization

**Three-Way Rule (Baseline for ALL ATM customers):**
- Model i-22 + AT&T + ATM Plan → 40 firewall whitelist rules

**Four-Way Rule (Customer Exception):**
- Model i-22 + AT&T + ATM Plan + CORD Company → 50 firewall rules (custom)

**Result:**
- All ATM customers get 40 rules automatically
- CORD gets 50 rules because they need extras
- No manual file editing
- One change updates all affected devices instantly

---

## SLIDE 7: Customer Self-Service Portal

### Empower Your Customers Safely

**Phase 1: Device-Level Configuration**
- Customers configure individual devices
- UI-driven (dropdowns, validated inputs)
- Only allowed parameters visible
- Cannot break their devices

**Allowed Customer Actions:**
- Configure device LAN IP (with validation)
- Manage Wi-Fi networks
- Set reboot schedules
- Adjust power management

**Safety Guardrails:**
- Cannot access system settings (DNS, firmware, carrier configs)
- Cannot download raw config files
- Cannot enter invalid values
- Preview before apply
- Confirmation dialogs

**Future Enhancement (Phase 2+):**
- Portfolio-wide settings (apply to all customer devices at once)

---

## SLIDE 8: Admin Capabilities

### Power Tools for System Administrators

**Complete Control:**
- Manage 600+ configuration parameters
- Full access to all 6 layers
- Conditional rules management
- Migration tools

**Safety Features:**
- Preview affected devices before changes
- "Dry run" mode (calculate impact without committing)
- Confirmation for changes affecting >10 devices
- Double confirmation for changes affecting >1000 devices
- Complete audit trail

**Visibility:**
- See effective config for any device
- Source attribution for every parameter
- Migration status dashboard
- System health indicators

**Testing & Rollout:**
- Test on 5-10 devices first
- Expand gradually to full fleet
- Rollback capability if issues occur

---

## SLIDE 9: The Smart Resolution Engine

### How the System Decides Which Value to Use

**11-Level Priority Hierarchy** (Highest → Lowest)

1. **Device Override** - Individual device setting
2. **Four-Way Rule** - Customer-specific exception (Model+Carrier+Plan+Company)
3. **Company Override** - Customer portfolio setting
4. **Three-Way Rule** - Service plan baseline (Model+Carrier+Plan)
5. **Service Plan Layer** - Plan defaults
6. **Two-Way Rules** - 6 conditional combinations (Model+Carrier, Carrier+Plan, etc.)
7. **Carrier Layer** - Carrier defaults
8. **Model Layer** - Model defaults
9. **Global Layer** - System-wide defaults
10. **Schema Default** - Hardcoded fallback
11. **Required Validation** - Error if nothing found

**The system checks each level in order and uses the first value found**

**Example:**
- DNS not set at Device? → Check Four-Way Rule
- No Four-Way Rule? → Check Company Override
- Company has DNS = 10.1.1.1? → **STOP, use 10.1.1.1**

---

## SLIDE 10: Conditional Rules Framework

### Handle Complex Business Logic Automatically

**The Challenge:**
Some parameters depend on multiple device characteristics simultaneously

**The Solution:**
Conditional rules that override standard inheritance when conditions match

**Rule Types:**

**Two-Way Rules** (6 combinations):
- Model + Carrier: "VZW i-22 devices enable advanced mode"
- Carrier + Service Plan: "VZW ATM plan gets 3.5GB daily, AT&T ATM gets 350MB"
- Model + Service Plan: "i-22 on ATM plan gets 350MB, 4500 gets 5GB"
- Carrier + Customer: "VIP customer negotiated 10GB VZW allowance"
- Model + Customer: "Miele's i-22 devices need specific I/O config"
- Service Plan + Customer: "Premium customer gets upgraded plan limits"

**Three-Way Rules:**
- Model + Carrier + Service Plan: "i-22 + AT&T + ATM = 40 firewall rules"
- Baseline for ALL customers

**Four-Way Rules:**
- Model + Carrier + Service Plan + Company: "CORD's custom firewall rules"
- Customer-specific exceptions to baselines

---

## SLIDE 11: Migration Strategy

### Zero-Disruption Transition

**Parallel Operation:**
- Both OLD and NEW systems run simultaneously
- No impact to existing devices during migration

**Gradual Rollout:**

**Phase 1: Pilot (1,000 devices)**
- Test on small set first
- Validate system works correctly
- Refine processes

**Phase 2: Expansion (50,000 devices)**
- Add customer self-service
- Expand device migration
- Monitor performance

**Phase 3: Complete Migration (All devices)**
- Migrate all remaining devices
- Retire OLD system
- Clean up code

**Safety Features:**
- Rollback capability (switch device back to old system if issues)
- Migration status dashboard
- Device-by-device or batch migration
- Compare old vs new config output for validation

---

## SLIDE 12: Implementation Phases

### Structured Delivery Approach

**Phase 1: MVP (Core System)**
- ✅ Config Key Schema Management (600+ parameters)
- ✅ 6-Layer Hierarchical Config Engine
- ✅ Conditional Rules Framework (2-way, 3-way, 4-way)
- ✅ 11-Level Priority Resolution Engine
- ✅ Both systems operational in parallel
- ✅ 1,000 devices migrated successfully
- ✅ Zero critical incidents

**Phase 2: Customer Self-Service**
- ✅ Customer-facing Device configuration UI
- ✅ Customers can edit device-level configs
- ✅ UI-driven with strict validation
- ✅ 50,000 devices migrated
- ⚠️ Future: Company-wide configuration access

**Phase 3: Complete Migration**
- ✅ All devices on new system
- ✅ OLD system retired
- ✅ Code cleanup completed

---

## SLIDE 13: Success Metrics

### How We Measure Success

**Operational Metrics:**
- Config change time: **Hours → <10 minutes** (95% reduction)
- Admin time savings: **5-10 hours/week freed**
- Config-related incidents: **Target near-zero**
- Support tickets: **70% reduction** (customer self-service)

**Migration Metrics:**
- Phase 1: **1,000 devices** migrated with zero critical incidents
- Phase 2: **50,000 devices** migrated
- Phase 3: **All 100,000+ devices** on new system

**Business Metrics:**
- Customer satisfaction: **Same-day changes** vs. multi-day wait
- Competitive advantage: **Flexible customization** competitors can't match
- Scalability: **Support 200,000+ devices** without limits
- Time-to-market: **Minutes** to implement security/infrastructure changes

**Quality Metrics:**
- Configuration errors: **Eliminate manual errors**
- Audit compliance: **Complete audit trail** for all changes
- Deployment confidence: **High confidence** in rollouts

---

## SLIDE 14: ROI Summary

### Investment vs. Return

**Time Savings:**
- Admin time: **5-10 hours/week** → **$15,000-$30,000/year**
- Incident response: **Reduce config-related outages** → **$10,000-$50,000/year**
- Support tickets: **70% reduction** → **$20,000-$40,000/year**

**Risk Reduction:**
- Eliminate manual errors causing device update loops
- Prevent service disruptions from config mistakes
- Enable rapid security response (minutes vs. days)

**Business Enablement:**
- Win deals requiring config flexibility
- Support customer growth without scaling issues
- Respond to infrastructure/security requirements instantly

**Competitive Advantage:**
- Customer self-service (differentiator)
- Flexible customization competitors cannot match
- Professional, scalable platform

**Total Annual Value: $45,000-$120,000**

**One-Time Investment:** Development + Migration effort

**Payback Period:** 6-12 months

---

## SLIDE 15: Why Now?

### Critical Drivers for This Project

**Operational Pain:**
- Config management consuming 5-10 hours/week of skilled admin time
- Manual errors causing customer-impacting incidents
- Cannot respond quickly to infrastructure changes

**Business Requirements:**
- Security: Need to disable factory reset across all devices instantly
- Customer demands: Config flexibility is a competitive requirement
- Scalability: Approaching limits of current system (100,000+ devices)

**Competitive Pressure:**
- Lost deals where config flexibility was required
- Customer churn risk from poor self-service

**Infrastructure Evolution:**
- DNS migrations, time server changes are high-risk operations
- Need safe way to test changes before full deployment

**Risk Exposure:**
- Current system poses significant outage risk
- No audit trail for compliance
- Manual hostname management creates update loops

**The Cost of Waiting:**
- Continued operational inefficiency
- Ongoing incident risk
- Competitive disadvantage grows
- Technical debt increases

---

## SLIDE 16: Questions & Discussion

### Key Topics

**Technical Questions:**
- How does the resolution engine handle conflicts?
- What happens if a required parameter has no value?
- How do conditional rules integrate with layer inheritance?

**Migration Questions:**
- How long will migration take?
- What's the rollback plan if issues occur?
- Can we migrate specific customers first?

**Customer Impact:**
- Will customers notice the transition?
- When will self-service portal be available?
- What training is needed?

**Business Questions:**
- What's the total investment required?
- What are the risks?
- What happens after full migration?

---

## SLIDE 17: Next Steps

### Moving Forward

**1. Review & Approve Approach**
- Executive Summary and Business Impact
- Technical architecture overview
- Migration strategy

**2. Detailed Requirements Review**
- Functional requirements walkthrough
- User stories and acceptance criteria
- Technical design specifications

**3. Project Planning**
- Resource allocation
- Timeline development
- Budget approval

**4. Pilot Preparation**
- Select 5-10 test devices
- Define success criteria for pilot
- Plan monitoring and validation

**5. Kickoff Phase 1**
- Begin development
- Set up parallel systems
- Prepare migration tools

---

## SLIDE 18: Appendix - User Access Summary

### Who Can Do What

| Capability | System Admin | Customer Admin |
|------------|--------------|----------------|
| **Edit Global/Model/Carrier Configs** | ✅ Full Access | ❌ No Access |
| **View System Configs** | ✅ All | ❌ No Access |
| **Edit Company Configs** | ✅ All Companies | ⚠️ Phase 2+ |
| **Edit Device Configs** | ✅ All Devices | ✅ Own Only (Allowed Params) |
| **Download Raw Config Files** | ✅ Yes | ❌ NEVER |
| **Migration Tools** | ✅ Yes | ❌ No Access |
| **Conditional Rules Management** | ✅ Yes | ❌ No Access |
| **Audit Trail Access** | ✅ Full | ⚠️ Own Only |

**Phase 1 Focus:** Device-level customer configuration only
**Phase 2+ Enhancement:** Company-wide customer configuration

---

## SLIDE 19: Appendix - Configuration Layers Explained

### The Six-Layer Hierarchy

**1. Global Layer**
- System-wide master configuration
- Infrastructure settings (DNS, time servers, security)
- Applies to ALL devices unless overridden

**2. Model Layer**
- Device model-specific (i-22, 4100, Origin, etc.)
- Hardware capabilities, model features
- Example: i-22 alarm I/O settings

**3. Carrier Layer**
- Cellular carrier-specific (Verizon, AT&T, T-Mobile)
- Network settings, carrier features
- Example: Verizon APN settings

**4. Service Plan Layer**
- Service tier-specific (ATM, Tier 1, Tier 2, etc.)
- Feature differentiation, data allowances
- Example: ATM plan has higher data thresholds

**5. Company Layer**
- Customer portfolio-wide settings
- Operational preferences, network integration
- Example: ACME Corp custom DNS, reboot schedules

**6. Device Layer**
- Individual device overrides
- Site-specific settings, one-off customizations
- Example: Device #12345 custom LAN IP

**Flow:** Global → Model → Carrier → Service Plan → Company → Device
**Override:** More specific layers override more general layers

---

## SLIDE 20: Appendix - Real-World Scenarios

### Example Use Cases

**Scenario 1: Infrastructure Change**
- **Task:** Migrate from Google DNS (8.8.8.8) to private DNS (10.0.0.1)
- **OLD:** Edit 200 files manually (4 hours, high error risk)
- **NEW:** Change Global layer value (2 minutes, zero errors)

**Scenario 2: Security Response**
- **Task:** Disable factory reset on all devices to prevent unauthorized repurposing
- **OLD:** Edit 200 files, deploy over days
- **NEW:** Set Global parameter, applies in minutes

**Scenario 3: Customer Customization**
- **Task:** Customer needs custom reboot schedule for 5,000 devices
- **OLD:** Create new config file, test, deploy manually
- **NEW:** Set Company layer value, applies automatically

**Scenario 4: Service Plan Feature**
- **Task:** ATM plan customers get advanced firewall rules
- **OLD:** Manually configure each customer's config files
- **NEW:** Three-Way Rule applies automatically to all ATM customers

**Scenario 5: Customer Exception**
- **Task:** One VIP customer needs custom data allowance
- **OLD:** Create one-off config file, maintain separately
- **NEW:** Four-Way Rule for that customer, inherits everything else

**Scenario 6: Testing New Feature**
- **Task:** Test new DNS servers on 10 devices before full rollout
- **OLD:** Manually edit 10 device configs, hope for the best
- **NEW:** Device layer override for 10 test devices, monitor, expand gradually

---

**END OF PRESENTATION**

---

## Document Information

**Version:** 1.5 (Based on PRD v1.5)
**Last Updated:** January 2026
**Full PRD Document:** Config_Management_System_PRD.md
**Functional Requirements:** Config_Management_System_PRD_Functional_Requirements.md

**Contacts:**
- **Product Manager:** Aksana Rahouski
- **Engineering Lead:** TBD
- **Business Stakeholder:** Adam Curcie (APW)
