# Configurations - Device Configuration Management

**Status:** Active - V1 design in progress
**Last Updated:** July 19, 2026

---

## Directory Structure

### Configurations_V1/
**Current working version.** Schema-driven configuration engine replacing the legacy .DAT file system. Layered inheritance: Schema Defaults -> Model Defaults -> 3-Way Rules -> Company Override Sets -> Device Overrides.

Contains requirements, prototype prompts, resolution logic, and client correspondence. See `Configurations_V1/README.md` for full details.

**Lovable Prototype:** https://device-heartbeat-hub.lovable.app/admin/config-mgmt
Subpages cover: Schema Concept, Model Defaults, Three-Way Rule, Company Overrides.

### Client_Provided_Documents/
Source data from APW — production .DAT config files, CSV exports by layer (Global, Carrier, Model, Service Plan, Customer, Device), and Verizon I-22 standard/custom baselines. Reference material for understanding the current config landscape.

### Research_and_Analysis/
Evidence-based analysis supporting design decisions — rule combination analysis (two-way, three-way, four-way), parameter layer mapping, and dependency analysis. Much of this informed the original (OLD) approach but remains useful reference for understanding config parameter behavior.

### Configurations-OLD/
**Archived — do not use.** Original V0 pitch with 6-layer hierarchy, 11-level priority resolution, and conditional rules framework. Client feedback: too complicated. Preserved for historical reference only. All active work is in `Configurations_V1/`.

### CLEANUP_SUMMARY.md
Log of a previous folder reorganization (March 2026). Documents what was moved, archived, or removed during that cleanup pass.

---

## Quick Reference

| Need to find... | Look in... |
|-----------------|-----------|
| **Current requirements & design** | `Configurations_V1/Requirements_Config_Builder.md` |
| **Config push/delivery spec** | `Configurations_V1/Requirements_Config_Push.md` |
| **How legacy system works** | `Configurations_V1/Current_Config_System_Summary.md` |
| **UI prototype prompts** | `Configurations_V1/Lovable prototype/Prototype_Build_Prompts.md` |
| **Production .DAT files** | `Client_Provided_Documents/Configuration_Files/` |
| **Parameter analysis** | `Research_and_Analysis/` |
| **Old/archived approach** | `Configurations-OLD/` |
