# Config Management System - Target Users & Personas

## Overview
The config management system serves multiple user types with different needs, permissions, and technical expertise levels. This document defines each user persona, their goals, use cases, and required access levels.

---

## User Personas

### Persona 1: System Administrator (APW Admin)
**Primary Users:** Adam Curcie, John Gates, DevOps Team

**Role Description:**
Full system administrators who manage the entire WATM platform, including all configuration layers for all customers and devices. They have deep technical knowledge of device firmware, networking, and the platform architecture.

**Key Characteristics:**
- **Technical Expertise:** Expert (9/10)
- **Frequency of Use:** Daily
- **Device Count Managed:** 100,000+ devices across all customers
- **Risk Tolerance:** Can make changes that affect entire system
- **Current Pain Points:**
  - Managing 200+ config files manually
  - Manual hostname/timestamp management causing errors
  - Cannot quickly respond to infrastructure changes (DNS, security patches)
  - Difficult to find which configs need updating

**Primary Goals:**
1. Efficiently manage global configurations affecting all devices
2. Define model-specific parameters and defaults
3. Set carrier-specific configurations
4. Respond quickly to security and infrastructure requirements
5. Troubleshoot config-related device issues
6. Migrate devices from old to new config system safely

**Key Use Cases:**

**UC-1: Update Global Configuration Parameter**
- **Scenario:** Need to change DNS servers from Google (8.8.8.8) to private DNS across all 100,000 devices
- **Current Process:** Edit 200 config files manually, high error risk
- **Desired Process:** Change single global parameter, system cascades to all devices
- **Frequency:** Monthly
- **Priority:** Critical

**UC-2: Add Model-Specific Configuration**
- **Scenario:** New i-22 firmware version requires additional alarm I/O parameters
- **Current Process:** Find all i-22 configs, manually edit each one
- **Desired Process:** Update i-22 model config, automatically applies to all i-22 devices
- **Frequency:** Quarterly
- **Priority:** High

**UC-3: Implement Security Requirement**
- **Scenario:** Disable factory reset capability on all devices to prevent unauthorized device repurposing
- **Current Process:** Edit all 200 config files to change one parameter
- **Desired Process:** Set global parameter with one action
- **Frequency:** As needed (security/compliance driven)
- **Priority:** Critical

**UC-4: Troubleshoot Device Config Issues**
- **Scenario:** Device in update loop, need to see effective config and trace where values come from
- **Current Process:** Manually check config file, hostname, logs
- **Desired Process:** View device config with source attribution (which layer each parameter comes from)
- **Frequency:** Weekly
- **Priority:** High

**UC-5: Gradual Config Rollout**
- **Scenario:** Test new carrier parameter on small set of devices before full deployment
- **Current Process:** Not possible - must create separate config files
- **Desired Process:** Apply config to specific device IDs, monitor, gradually expand
- **Frequency:** Monthly
- **Priority:** High

**Required Capabilities:**
- ✅ Full CRUD access to all config layers (Global, Model, Carrier, Service Plan, Company, Device)
- ✅ View effective/final config for any device with source attribution
- ✅ Bulk operations (apply config to device sets)
- ✅ Config version history and audit trail
- ✅ Migration tools (old system → new system)
- ✅ Validation override (with warnings) for advanced scenarios
- ✅ API access for automation
- ✅ Export/import configs for backup/disaster recovery

**Access Level:** Full Administrative Access

---

### Persona 2: Operations/Support Team (APW Operations)
**Primary Users:** Richard Sacco, Support Engineers, NOC Team

**Role Description:**
Technical support staff who monitor device health, troubleshoot issues, and assist customers. They need visibility into configs but typically don't change global/model settings.

**Key Characteristics:**
- **Technical Expertise:** Advanced (7/10)
- **Frequency of Use:** Daily
- **Device Count Managed:** Monitor all, directly manage specific customer devices
- **Risk Tolerance:** Medium - can make customer/device level changes
- **Current Pain Points:**
  - Difficult to identify why device has specific config values
  - Cannot quickly see if device is on correct config version
  - Limited visibility into config change history

**Primary Goals:**
1. Troubleshoot device connectivity and config issues
2. Verify devices are on correct configs
3. Apply device-specific config overrides when needed
4. Support customers with config-related questions
5. Escalate systemic config issues to admins

**Key Use Cases:**

**UC-6: Diagnose Device Issue**
- **Scenario:** Customer reports device not connecting, need to verify config is correct
- **Current Process:** Check logs, download config file, manually compare
- **Desired Process:** View device effective config, see version status, compare to expected
- **Frequency:** Daily (multiple times)
- **Priority:** Critical

**UC-7: Apply Customer-Requested Override**
- **Scenario:** Customer wants specific device to use different LAN IP than portfolio default
- **Current Process:** Escalate to admin or create custom config file
- **Desired Process:** Apply device-level override with validation
- **Frequency:** Weekly
- **Priority:** Medium

**UC-8: Verify Config Deployment**
- **Scenario:** After global config change, verify specific devices received update
- **Current Process:** Check individual device check-in logs and hostnames
- **Desired Process:** Query device config status, see when last updated
- **Frequency:** After each config deployment
- **Priority:** High

**Required Capabilities:**
- ✅ Read access to all config layers
- ✅ View effective config for any device with source attribution
- ✅ Apply device-level overrides (with validation)
- ✅ View config change history and audit trail
- ✅ Search/filter devices by config version or parameters
- ✅ Export device config reports for troubleshooting
- ❌ Cannot modify Global, Model, or Carrier level configs
- ❌ Limited company-level changes (requires approval)

**Access Level:** Operations Access (Read Global/Model/Carrier, Write Device)

---

### Persona 3: End Customer Administrator (APW Client)
**Example Users:** Large fleet managers, enterprise IT administrators

**Role Description:**
APW's customers who manage their own device fleets. They need to configure certain parameters for their devices but should not have access to system-level configs. They may manage anywhere from 10 to 10,000 devices.

**Key Characteristics:**
- **Technical Expertise:** Intermediate (4-6/10) - varies widely
- **Frequency of Use:** Weekly to monthly
- **Device Count Managed:** 10-10,000 devices (their own portfolio only)
- **Risk Tolerance:** Low - must be prevented from breaking their devices
- **Current Pain Points:**
  - Limited ability to configure devices to their needs
  - Must request APW support for simple configuration changes
  - No portfolio-wide settings (must configure each device individually)

**Primary Goals:**
1. Configure device settings that affect their operations (Wi-Fi, scheduling, LAN IPs)
2. Apply consistent settings across their entire device fleet
3. Customize individual devices when needed
4. Avoid breaking their devices with invalid configurations

**Key Use Cases:**

**UC-9: Set Portfolio-Wide Reboot Schedule**
- **Scenario:** Customer with 500 devices wants all devices to reboot at 3:30 AM daily
- **Current Process:** Not possible - must be configured per-device or request custom config file
- **Desired Process:** Set company-level schedule, applies to all their devices
- **Frequency:** Once, then adjust occasionally
- **Priority:** High
- **Reference:** Meeting lines 627-639, 783-796

**UC-10: Configure Device LAN IP**
- **Scenario:** Customer needs device on 10.x.x.x network instead of default 192.168.x.x
- **Current Process:** Limited UI or must contact support
- **Desired Process:** Select from allowed IP ranges via dropdown/validated input
- **Frequency:** Per device during deployment
- **Priority:** Medium
- **Reference:** Meeting lines 663-671

**UC-11: Manage Wi-Fi Networks**
- **Scenario:** Customer needs to add backup Wi-Fi network credentials
- **Current Process:** Works today via UI (use as pattern)
- **Desired Process:** Continue current pattern, extend to other config parameters
- **Frequency:** Occasionally per device
- **Priority:** Medium
- **Reference:** Meeting line 815 (current Wi-Fi functionality is good model)

**UC-12: Configure Power Management**
- **Scenario:** Customer wants to disable cellular backup during certain hours to save costs
- **Current Process:** Must contact APW support
- **Desired Process:** Configure schedule via UI with validation
- **Frequency:** Occasionally
- **Priority:** Low-Medium

**Required Capabilities:**
- ✅ View their own devices only
- ✅ Configure allowed parameters at company level (portfolio-wide)
- ✅ Configure allowed parameters at device level
- ✅ UI-driven configuration with strict validation (no free-form text)
- ✅ Preview changes before applying
- ❌ Cannot download raw config files
- ❌ Cannot see system-level parameters (DNS, time servers, etc.)
- ❌ Cannot access other customers' devices
- ❌ Cannot override certain parameters (security-locked)

**Allowed Configuration Parameters:**
- ✅ Device reboot scheduling (time, frequency)
- ✅ LAN IP addressing (within allowed ranges: 192.168.x.x, 10.x.x.x)
- ✅ Wi-Fi network credentials (already implemented)
- ✅ Power management schedules
- ✅ Device hostname (for their reference)
- ❌ DNS servers (admin only)
- ❌ Time servers (admin only)
- ❌ Firmware parameters (admin only)
- ❌ Carrier settings (admin only)
- ❌ Factory reset enable/disable (admin only)

**Access Level:** Customer Administrator (Limited Write - Own Devices Only)

**Critical Requirement from Meeting (lines 651-657, 812-823):**
> *"has to absolutely be something that will not give them the power to screw themselves because they will, they will, they will take themselves out of business and blame us in a heartbeat."*
> *"I do not want any customers being able to download these files at all."*

---

### Persona 4: End Customer User (APW Client - Limited)
**Example Users:** Field technicians, basic users at customer organizations

**Role Description:**
End users at customer organizations who need to view device status and possibly configure very limited parameters. They are not technical administrators.

**Key Characteristics:**
- **Technical Expertise:** Basic (2-3/10)
- **Frequency of Use:** As needed (sporadic)
- **Device Count Managed:** View only, or single device configuration
- **Risk Tolerance:** Very low - should have minimal config access
- **Current Pain Points:**
  - May not have access to configurations at all currently
  - Depend on administrators for simple changes

**Primary Goals:**
1. View device configuration status
2. Possibly configure very basic parameters (if permitted by their admin)
3. Not break anything

**Key Use Cases:**

**UC-13: View Device Configuration**
- **Scenario:** Check what Wi-Fi networks are configured on device
- **Current Process:** May not have access, or must contact administrator
- **Desired Process:** View-only access to customer-configurable parameters
- **Frequency:** As needed
- **Priority:** Low

**UC-14: Update Single Device Setting**
- **Scenario:** Add Wi-Fi network to specific device (if permitted by customer admin)
- **Current Process:** Contact administrator
- **Desired Process:** Guided UI for single parameter on single device
- **Frequency:** Rare
- **Priority:** Low

**Required Capabilities:**
- ✅ View own company's devices (read-only)
- ✅ Possibly configure single device parameters (if granted by customer admin)
- ✅ Highly restricted, guided UI
- ❌ No portfolio-wide changes
- ❌ No access to technical parameters

**Access Level:** Customer User (Read-Only or Very Limited Write)

**Note:** This persona may not be in scope for initial implementation (MVP). Focus on Admin, Operations, and Customer Administrator personas first.

---

### Persona 5: APW Sales/Business Development
**Primary Users:** Sales team, account managers, pre-sales engineers

**Role Description:**
Non-technical to semi-technical staff who need to understand and demonstrate config capabilities to prospects and customers. They don't configure systems but need visibility for sales purposes.

**Key Characteristics:**
- **Technical Expertise:** Basic to Intermediate (3-5/10)
- **Frequency of Use:** During sales cycles and customer meetings
- **Device Count Managed:** Demo/test environments only
- **Risk Tolerance:** Low - should not modify production
- **Current Pain Points:**
  - Cannot easily demonstrate config flexibility to prospects
  - Limited understanding of what customizations are possible

**Primary Goals:**
1. Demonstrate config capabilities to prospects
2. Understand what customer requirements can be met
3. Provide accurate information during sales process
4. Test configurations in demo environment

**Key Use Cases:**

**UC-15: Demo Config Capabilities**
- **Scenario:** Show prospect how they can manage their device fleet configs
- **Current Process:** Rely on technical team for demos
- **Desired Process:** Access demo environment to show real config UI
- **Frequency:** Weekly during active sales cycles
- **Priority:** Medium

**UC-16: Validate Customer Requirements**
- **Scenario:** Prospect asks if they can set reboot schedules portfolio-wide
- **Current Process:** Escalate to technical team for answer
- **Desired Process:** View documentation/capabilities list, test in demo environment
- **Frequency:** During sales cycles
- **Priority:** Medium

**Required Capabilities:**
- ✅ Access to demo/test environment
- ✅ Documentation of config capabilities
- ✅ View-only access to production (for customer support)
- ❌ No production config changes
- ❌ Limited to demo data

**Access Level:** Demo Environment Access + Production Read-Only

---

## User Access Matrix

| Capability | System Admin | Operations | Customer Admin | Customer User | Sales/BD |
|------------|--------------|------------|----------------|---------------|----------|
| **View Global Configs** | ✅ Full | ✅ Read-only | ❌ No | ❌ No | ✅ Demo only |
| **Edit Global Configs** | ✅ Yes | ❌ No | ❌ No | ❌ No | ❌ No |
| **View Model Configs** | ✅ Full | ✅ Read-only | ❌ No | ❌ No | ✅ Demo only |
| **Edit Model Configs** | ✅ Yes | ❌ No | ❌ No | ❌ No | ❌ No |
| **View Carrier Configs** | ✅ Full | ✅ Read-only | ❌ No | ❌ No | ✅ Demo only |
| **Edit Carrier Configs** | ✅ Yes | ❌ No | ❌ No | ❌ No | ❌ No |
| **View Service Plan Configs** | ✅ Full | ✅ Read-only | ❌ No | ❌ No | ✅ Demo only |
| **Edit Service Plan Configs** | ✅ Yes | ❌ No | ❌ No | ❌ No | ❌ No |
| **View Company Configs** | ✅ All | ✅ All | ✅ Own only | ✅ Own only | ✅ Demo only |
| **Edit Company Configs** | ✅ All | ⚠️ Limited | ✅ Own only (allowed params) | ❌ No | ❌ No |
| **View Device Configs** | ✅ All | ✅ All | ✅ Own only | ✅ Own only | ✅ Demo only |
| **Edit Device Configs** | ✅ All | ✅ All | ✅ Own only (allowed params) | ⚠️ Very limited | ❌ No |
| **Download Raw Config Files** | ✅ Yes | ✅ Yes | ❌ No | ❌ No | ⚠️ Demo only |
| **View Config History/Audit** | ✅ Full | ✅ Full | ✅ Own only | ❌ No | ❌ No |
| **API Access** | ✅ Full | ⚠️ Limited | ⚠️ Future | ❌ No | ❌ No |
| **Migration Tools** | ✅ Yes | ❌ No | ❌ No | ❌ No | ❌ No |
| **Validation Override** | ✅ Yes (with warnings) | ❌ No | ❌ No | ❌ No | ❌ No |

**Legend:**
- ✅ Full access
- ⚠️ Limited/conditional access
- ❌ No access

---

## User Journey Maps

### Journey 1: Admin Updates Global DNS (System Administrator)
**Current State (Pain Points):**
1. Decision made: Need to change DNS servers
2. Open config file repository
3. Manually edit each of ~200 config files
4. Update hostname timestamp in each file (error-prone)
5. Update anticipated hostname in each file
6. Upload files back to system
7. Wait for devices to check in
8. Monitor for update loops if hostname typo occurred
9. **Time Required:** 2-4 hours, high error risk

**Future State (Desired Experience):**
1. Decision made: Need to change DNS servers
2. Log into admin portal
3. Navigate to Global Configs → DNS Settings
4. Update primary/secondary DNS values
5. Preview affected devices (100,000 devices)
6. Click "Apply Changes"
7. System automatically versions configs and schedules updates
8. Dashboard shows rollout progress
9. **Time Required:** 5-10 minutes, minimal error risk

**Success Metrics:**
- Time reduction: 95%+
- Error rate: Near zero
- Admin confidence: High
- Rollback capability: Available

---

### Journey 2: Operations Troubleshoots Device (Operations Team)
**Current State:**
1. Customer reports device not connecting
2. Find device in portal
3. Download device config file
4. Open file, try to understand 200+ parameters
5. Check logs for hostname validation
6. Compare config to similar working devices manually
7. Escalate to admin if config issue suspected
8. **Time Required:** 20-40 minutes

**Future State:**
1. Customer reports device not connecting
2. Find device in portal
3. View "Effective Configuration" tab
4. See config parameters with source attribution:
   - DNS: 8.8.8.8 (Global)
   - LAN IP: 10.1.50.1 (Device Override)
   - Reboot Schedule: 03:30 daily (Company Setting)
5. Compare effective config to expected/other devices
6. Identify issue: Device has wrong carrier setting
7. Update device carrier assignment or apply override
8. **Time Required:** 5-10 minutes

**Success Metrics:**
- Time reduction: 60-70%
- First-call resolution: Improved
- Escalations to admin: Reduced by 50%

---

### Journey 3: Customer Sets Portfolio-Wide Schedule (Customer Admin)
**Current State:**
1. Customer wants all 500 devices to reboot at 3:30 AM
2. Customer contacts APW support
3. Support creates ticket for admin team
4. Admin creates custom config file
5. Admin maps customer devices to new config
6. Wait for devices to check in and receive config
7. Customer tests and confirms
8. **Time Required:** 1-3 days

**Future State:**
1. Customer wants all 500 devices to reboot at 3:30 AM
2. Customer logs into portal
3. Navigate to Company Settings → Scheduling
4. Enable "Automatic Reboot Schedule"
5. Select time from dropdown: 3:30 AM
6. Select frequency: Daily
7. Preview affected devices (500 devices shown)
8. Click "Apply to All Devices"
9. System validates, updates configs, schedules delivery
10. Customer receives confirmation
11. **Time Required:** 5 minutes, self-service

**Success Metrics:**
- Support ticket reduction: 80% for config requests
- Customer satisfaction: Improved
- Time to value: Same-day vs. days
- Support burden: Significantly reduced

---

## Phased User Rollout Strategy

### Phase 1: Internal Users Only (Months 1-6)
**Target Users:**
- System Administrators (Adam, John, DevOps)
- Operations Team (Richard, support engineers)

**Why Start Here:**
- Lowest risk - internal users understand limitations
- Can iterate on UI/UX based on expert feedback
- Validate config engine logic before customer exposure
- Build confidence in system stability

**Success Criteria:**
- 1,000+ devices successfully managed
- Zero critical incidents
- Admin time savings demonstrated
- Operations team trained and confident

---

### Phase 2: Select Customer Beta (Months 6-12)
**Target Users:**
- 3-5 trusted customer administrators
- Continue internal admin/operations use

**Selection Criteria for Beta Customers:**
- Technically savvy
- Good relationship with APW
- 50-500 devices (medium scale)
- Active engagement in beta feedback
- Signed beta agreement

**Why These Customers:**
- Validate customer-facing UI/UX
- Test validation rules with real users
- Gather feedback on desired features
- Build case studies for broader rollout

**Success Criteria:**
- Beta customers successfully self-configure
- Zero instances of customers breaking their devices
- Positive feedback on ease of use
- Documented support ticket reduction

---

### Phase 3: General Availability (Months 12-18)
**Target Users:**
- All customer administrators (opt-in initially)
- All internal users
- Begin training customer users (read-only)

**Rollout Approach:**
- Announce new capabilities
- Provide training materials/webinars
- Opt-in for first 30 days
- Gradually migrate all customers

**Success Criteria:**
- 50%+ customer adoption within 3 months
- Customer satisfaction scores improve
- Support tickets for config requests decrease 70%+
- Sales team actively demos capabilities

---

### Phase 4: Advanced Features (Months 18+)
**Target Users:**
- API users (for automation)
- Customer users (limited read/write)
- Sales/BD (enhanced demo capabilities)

**Advanced Features:**
- API access for customer automation
- More granular customer permissions (admin delegates to users)
- Enhanced reporting and analytics
- Config templates/presets

---

## User Training Requirements

### System Administrators
**Training Topics:**
- Hierarchical config model and cascade logic
- Config editor interface and validation rules
- Migration tools and device ID selection
- Troubleshooting config issues
- Audit trail and version control
- API usage for automation
- Rollback procedures

**Format:** Hands-on workshop, documentation, sandbox environment
**Duration:** 4-8 hours initial, ongoing as features added
**Frequency:** Before each phase rollout

---

### Operations Team
**Training Topics:**
- Viewing effective device configs
- Understanding source attribution
- Applying device-level overrides
- Troubleshooting config issues
- When to escalate to admin
- Customer support workflows

**Format:** Workshop + reference guides
**Duration:** 2-4 hours
**Frequency:** Before customer rollout

---

### Customer Administrators
**Training Topics:**
- What configs they can control
- Company-level vs. device-level settings
- How to use config UI (guided tour)
- Understanding validation messages
- Best practices and common scenarios
- When to contact support

**Format:** Video tutorials, documentation, webinars, in-app guidance
**Duration:** 30-60 minutes self-paced
**Frequency:** At opt-in, ongoing webinars

---

### Sales/Business Development
**Training Topics:**
- Config capabilities overview (what can customers do?)
- Competitive differentiators
- Demo environment walkthrough
- Common customer requirements and how to address
- Pricing implications (if any)

**Format:** Presentation + demo environment access
**Duration:** 1-2 hours
**Frequency:** Quarterly updates

---

## User Feedback & Iteration

### Feedback Collection Methods
- **Admin/Operations:** Weekly standups during Phase 1, monthly retrospectives
- **Beta Customers:** Bi-weekly check-ins, feedback forms, usage analytics
- **General Customers:** In-app feedback, support ticket analysis, surveys
- **Sales/BD:** Quarterly feedback sessions, win/loss analysis

### Key Metrics to Track per User Type
1. **System Administrators:**
   - Time spent on config management (weekly)
   - Number of manual config file edits
   - Config-related incidents
   - User satisfaction score

2. **Operations Team:**
   - Time to resolve config-related tickets
   - Escalation rate to admins
   - First-call resolution rate
   - User satisfaction score

3. **Customer Administrators:**
   - Adoption rate (% using self-service config)
   - Support ticket reduction
   - Configuration error rate
   - Customer satisfaction (CSAT/NPS)

4. **Sales/BD:**
   - Competitive win rate (config flexibility mentioned)
   - Demo effectiveness score
   - Deal velocity (time to close)

---

## Accessibility & Internationalization Considerations

### Accessibility Requirements
- WCAG 2.1 AA compliance for all user interfaces
- Keyboard navigation for all config operations
- Screen reader support for visual elements
- Clear error messages and validation feedback
- Color-blind friendly UI design

### Internationalization (Future)
- **Phase 1:** English only (US-based users)
- **Phase 2+:** Consider localization if international customers need config access
- Language support: English (primary), Spanish (potential future)

---

## Summary: User-Centric Design Principles

### For System Administrators
1. **Power & Control:** Full access with safety nets (validation, audit trails)
2. **Efficiency:** Minimize clicks and time for common operations
3. **Visibility:** Clear understanding of impact before applying changes
4. **Flexibility:** API access for automation, bulk operations

### For Operations Team
5. **Clarity:** Easy to understand device config state and source
6. **Speed:** Quick troubleshooting and resolution
7. **Confidence:** Know when to fix vs. escalate

### For Customer Administrators
8. **Simplicity:** Guided, intuitive UI requiring minimal training
9. **Safety:** Impossible to break devices through config errors
10. **Empowerment:** Self-service for common configuration needs
11. **Trust:** System prevents mistakes, provides confirmation

### For All Users
12. **Reliability:** System always produces correct configs
13. **Transparency:** Audit trail shows who changed what, when, why
14. **Consistency:** Predictable behavior across all config operations

---

*Document created from Meeting 2 discussion and user requirements analysis*
*Last updated: 2026-01-02*
