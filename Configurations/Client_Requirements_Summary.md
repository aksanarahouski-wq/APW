# Configuration Management Framework - Client Requirements Summary

**Date:** December 9, 2025
**Based on:** Meeting1.md (meeting transcript)
**Priority:** Top priority for 2026

## Overview

The client needs a complete overhaul of the current configuration management system. This is identified as a "big, meaty" project that will likely span multiple phases throughout 2026.

---

## Core Requirements

### 1. Multi-Level Configuration Hierarchy

The system needs to support configuration management at multiple levels with a clear override priority:

1. **Global Config** - Base configuration applied to all devices
2. **Model Level** - Configurations specific to device models
3. **Carrier Level** - Configurations specific to carriers (Verizon, T-Mobile, AT&T, etc.)
4. **Service Plan Level** - Configurations tied to specific service plans
5. **Company Level** - Custom configurations for specific companies/customers
6. **Device Level** - Individual device-specific configurations

**Override Priority:** Lower levels override higher levels (Device > Company > Service Plan > Carrier > Model > Global)

### 2. Company-Level Custom Configurations

**Admin-Managed (Not Customer-Accessible):**
- Custom translation rules (e.g., Altec Enterprises printer configuration)
- DHCP range settings
- LAN IP configurations
- Extensive firewall rules (for ATM devices)
- Other company-specific parameters that customers request

**Key Point:** Customers do NOT configure these themselves. Sales/support staff (like Adam) take customer requirements and build these configurations for them.

**Example Use Case:**
> "Alan from Altec says 'I need to talk to my printers' → Adam builds translation rules → These rules apply to ALL Altec devices regardless of model or carrier"

### 3. Device-Level Configurations

**Customer-Accessible (Limited Options):**
- Wi-Fi settings
- Basic firewall rules (for non-ATM devices)
- Other simple device-specific settings

**Restrictions:**
- Very few parameters should be customer-manageable
- ATM device customers should NOT be able to manipulate firewalls
- Must prevent customers from breaking their devices

### 4. Remove Current "Browse Configurations" System

**Current Pain Points:**
- File-based configuration browsing must be eliminated
- Customers should NOT be able to download raw config files
- **Incident:** Customer "Pants" downloaded config file and started asking questions about it
- Current system is too open-ended and dangerous

**Requirement:** Remove or restrict the "Browse Configurations" section entirely.

### 5. UI/UX Requirements

#### NOT Desired:
- Simple text editor approach where users can freely edit config files
- Open forum where users can "self-destruct" by breaking configurations
- Ability for customers to see/download raw config text files

#### Desired Features:
- **Structured parameter management** - Prevent assigning parameters to wrong levels
  - Example: Firewall-level parameters cannot be added to carrier-level configs
- **Validation and restrictions** - Users can't accidentally misconfigure
- **Text editor with constraints** - For admin use, but with proper formatting and validation
- **Visual organization** - See configurations organized by level/hierarchy
- **Manage configurations from relevant pages:**
  - Company-level configs accessible from company management pages
  - Similar to how service plans evolved (used to be company-page only, now has dedicated management section)

### 6. Parameter Organization

**Concept:**
- Global config = massive list of key-value pairs
- Each level (company, model, carrier, etc.) gets a **subset** of applicable parameters
- Not all parameters are relevant at all levels
- Need to define which parameters can be configured at which levels

**Example:**
- Company level might only configure: firewall settings, DHCP ranges, translation rules
- They should ONLY see/access those specific parameters, not the entire global config

### 7. Migration Strategy (Critical Requirement)

**Requirements:**
- **NO hard cutover** - Cannot lose all configurations during migration
- **Gradual migration** - Ability to move configurations from old system to new system incrementally
- **Safe rollback** - Must be able to revert to old system if issues arise
- **Dual operation** - Keep current system operational while building/testing new system
- **Manual migration control** - Ability to select which configs to migrate and when
- **Testing per config** - Migrate a config, monitor, validate before moving to next one

**Quote from Adam:**
> "Keep the current configuration management system in place while we integrate or move away... have a way where you could select or like almost like phase these over one by one to the new system."

**Rationale:** Config management is the "heartbeat" of their business - must not disrupt operations.

### 8. Phased Development Approach

**Requirements:**
- Break project into phases
- Release functionality incrementally
- Cannot be "under construction for three months"
- Need usable functionality as development progresses

**Planning Needed:**
- Define phases clearly during kickoff
- Identify what can be released early
- Plan migration path for each phase

---

## Technical Considerations

### Data Structure Requirements

1. **Define configuration entities and their relationships:**
   - Which entities can have configs/sub-configs?
   - How do configs inherit/override?
   - What's the data model?

2. **Parameter categorization:**
   - Create comprehensive list of all config parameters
   - Group parameters by applicable level (global, model, carrier, company, device)
   - Define which parameters can be overridden at which levels

3. **Validation rules:**
   - What parameters are required?
   - What are valid values for each parameter?
   - What combinations are invalid?

### System Integration

**Current Pain Point:**
- Recent outage caused by bad config data bringing system "to its knees"
- System should be bulletproof against bad config data
- Need better logging and monitoring to quickly identify config-related issues

**Requirements:**
- Config changes should not crash the system
- Better error handling for invalid configs
- Monitoring/alerting for config-related problems

---

## Open Questions for Kickoff

1. **Parameter inventory:** Need complete list of all current config parameters and categorization

2. **Parameter-level mapping:** Which parameters belong at which levels? (global/model/carrier/company/device)

3. **Use cases:** Document all current configuration scenarios and workflows

4. **Edge cases:** What happens when configs conflict? What are invalid combinations?

5. **Access control:**
   - Who can edit global configs?
   - Who can edit company configs?
   - What can customers edit?

6. **Validation requirements:** What validations are needed for each parameter?

7. **Migration scope:**
   - How many config files exist currently?
   - How complex are they?
   - What's the migration effort estimate?

8. **Testing strategy:** How to test without impacting production?

---

## Success Criteria

1. **Admin users can:**
   - Manage global configurations
   - Create model-specific configs
   - Create carrier-specific configs
   - Create company-specific custom configs
   - Override configs at appropriate levels
   - See clear hierarchy of config inheritance

2. **Customers can:**
   - Manage limited device-level settings (Wi-Fi, basic firewall)
   - NOT see or download raw config files
   - NOT break their devices through misconfiguration

3. **System reliability:**
   - Bad config data does not bring system down
   - Clear logging and monitoring of config issues
   - Fast troubleshooting when config problems occur

4. **Migration:**
   - Smooth transition from old to new system
   - No service disruption
   - Ability to rollback if needed
   - Phased approach with validation at each step

---

## Next Steps

1. **Schedule kickoff session** with development team (Aksana, Aaron, Stone/Absor)
2. **Gather requirements:**
   - Client to show analysis/documentation they've prepared
   - Discuss "vision" for how config management should work
   - Walk through current pain points in detail
3. **Technical discovery:**
   - Inventory all current config parameters
   - Map parameters to appropriate levels
   - Document current config file structure
4. **Create detailed plan:**
   - Define phases
   - Design data model
   - Plan migration strategy
   - Identify risks

---

## Timeline

- **Priority:** Top priority for 2026
- **Urgency:** Client ready to start ASAP (rest of month/year available)
- **Approach:** Phased delivery throughout 2026
- **Note:** This is described as likely the biggest project for the year

---

## Additional Context

**Client Pain Point:**
> "There's so much of this stuff that is like I wouldn't even want to see it" - Devon

This quote captures that the current system is overwhelming and too granular. The new system should abstract away complexity while still providing necessary control.

**Client Concern:**
> "We've been manually fixing [devices]" after config issues

Configuration problems currently require extensive manual intervention. New system must be more resilient and provide better tooling for fixing config-related issues.
