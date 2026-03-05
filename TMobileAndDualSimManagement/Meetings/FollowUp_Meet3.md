**Subject: Dual-SIM Configuration Mapping - Meeting Summary & Implementation Plan (Meeting 3 of 3)**

Hi team,

Thank you for the excellent progress today on finalizing our dual-SIM configuration mapping strategy. This meeting successfully resolved the primary open item from yesterday's discussion—how to properly map device configurations based on SIM positions and carrier combinations. Here's a summary of today's decisions.

## Recap: What We Previously Decided (Meetings 1-2)

To provide context, our previous meetings established:

**Import Process Enhancement**:
- Add active status columns to import template (Verizon Active, AT&T Active, T-Mobile Active: Yes/No)
- Maximum 2 SIMs per device (validated by system)
- If both SIMs imported as active, dual flag automatically set to true
- Import will NOT trigger carrier API calls (devices imported with SIMs already in test ready status)

**Assignment Process Simplification**:
- Remove carrier selection options from assignment screen
- Remove "maintain existing SIM" checkbox
- Assignment only ties device to company without modifying SIM status

**Device Update Process**:
- Carrier API calls triggered only during device updates (not import/assignment)
- Smart status logic for AT&T: test ready → keep test ready; deactivated → activate
- Similar logic for T-Mobile (6-month grace period)

**Known Constraints**:
- I-22 devices: Verizon always in SIM 1, other carrier in SIM 2
- Origin devices: Verizon as eSIM, T-Mobile/AT&T as physical SIM

## Today's Key Decisions (Meeting 3)

**1. Configuration Mapping Resolution ✅**

We resolved the primary open item from yesterday by creating a comprehensive 18-row configuration mapping matrix that defines exactly how devices map to configuration files based on model, SIM presence, and active status.

**Critical Rule Established**:
- Mapping must consider **BOTH SIM presence AND active status** (not just active status as currently implemented)
- If both SIMs are populated but only one is active → map to **dual carrier config** (not single carrier)
  - Example: Verizon inactive + AT&T active = "DUAL CARRIER **ATT ONLY, VERIZON INACTIVE"
  - Example: Verizon active + T-Mobile inactive = "DUAL CARRIER **VZW ONLY TMO INACTIVE"
  - These special cases are **highlighted in yellow** in the matrix

**Rationale**: Device needs dual carrier config structure when two SIMs are present, regardless of active status. This prevents config remapping issues and keeps the device functional even if carrier status changes.

**2. Dual Carrier vs. Dual SIM Clarification**
- **Dual Carrier Config**: Two SIMs present (determines which config file to use)
- **Dual SIM Billing**: Both SIMs active (determines if customer is charged for two carriers)
- These are now **separate concepts** with different purposes

**3. AT&T-Only Edge Case - DEFERRED ✅**

The open question from yesterday regarding "AT&T-only device with SIM in position 2" has been **deliberately deferred**:
- **Decision**: NOT implementing this scenario at this time
- **Rationale**:
  - Extremely rare (never requested in practice)
  - AT&T is more expensive, business prefers Verizon/dual carrier solutions
  - Configuration engine overhaul is planned (will handle this more elegantly)
- **Current scope**: Focus on T-Mobile implementation for Origin device with minimal changes

**4. Scope Definition**

Today we clearly defined what we're building NOW vs. LATER:
- **Now**: Hard-coded mapping rules based on 18-row matrix (temporary but solid solution)
- **Later**: Configuration engine overhaul using hierarchical parameter system (Global → Model → Carrier)

## Configuration Mapping Matrix (Our "Bible")

Devon created a comprehensive 18-row matrix defining all supported device/carrier combinations:

**I-4100/4500/M5 Models** (Single SIM only - 3 rows):
1. VZW active → VZW ONLY
2. ATT active → ATT ONLY
3. TMO active → TMO ONLY

**I-22/I-52 Models** (9 rows):
4. VZW active only → VZW ONLY
5. VZW active, ATT inactive → VZW ONLY
6. VZW active, TMO inactive → VZW ONLY
7. ATT active only → ATT ONLY
8. **VZW inactive, ATT active → DUAL CARRIER \*\*ATT ONLY, VERIZON INACTIVE** ⚠️
9. **VZW inactive, TMO active → DUAL CARRIER \*\*TMO ONLY VERIZON INACTIVE** ⚠️
10. VZW active, ATT active → VZW/ATT DUAL
11. VZW active, TMO active → VZW/TMO DUAL

**ORIGIN/CR202 Models** (6 rows - Verizon always eSIM):
12. **VZW eSIM active, TMO physical inactive → DUAL CARRIER \*\*VZW ONLY TMO INACTIVE** ⚠️
13. **VZW eSIM active, ATT physical inactive → DUAL CARRIER \*\*VZW ONLY ATT INACTIVE** ⚠️
14. VZW eSIM inactive, ATT physical active → ATT ONLY
15. VZW eSIM inactive, TMO physical active → TMO ONLY
16. VZW eSIM active, ATT physical active → VZW/ATT DUAL
17. VZW eSIM active, TMO physical active → VZW/TMO DUAL

⚠️ = Yellow highlighted special cases requiring dual carrier config despite single active SIM

**Mapping Logic**:
1. Check device model first
2. Check SIM presence (determines if dual carrier config needed)
3. Check active status (determines specific config parameters/carrier priority)

## Complete Implementation Summary

Combining all three meetings, here's what needs to be implemented:

**Import Template Changes** (from Meeting 2):
- Add columns: Verizon Active (Yes/No), AT&T Active (Yes/No), T-Mobile Active (Yes/No)
- Validate maximum 2 SIMs per import row
- Auto-set dual flag if both active
- No carrier API calls on import

**Assignment Screen Changes** (from Meeting 2):
- Remove carrier selection section
- Remove "maintain existing SIM" checkbox
- Assignment only updates company relationship

**Update Devices Feature Enhancement** (from Meeting 2):
- Support bulk updates with active status columns
- This is the preferred method for bulk SIM status changes

**Configuration Mapping Engine Changes** (from Meeting 3):
- Implement 18-row mapping matrix logic
- Consider both SIM presence AND active status
- Add Verizon/T-Mobile dual carrier combination to config builder

**Device SIM Status Update Changes** (from Meetings 2-3):
- **On Import**: Do NOT activate SIMs via carrier API calls
  - Assume all SIMs are in "test ready" status as pre-configured by warehouse
  - Only store SIM numbers and active flags in database
- **On Device Edit Action**: Smart status logic when updating SIM active status
  - Check current SIM status via carrier API
  - If status is "test ready" → keep it in "test ready" (preserve free period)
  - If status is NOT "test ready" → toggle between "active" and "inactive" based on checkbox
  - AT&T: Once exited from "test ready", cannot return (unlimited free period while in test ready)
  - T-Mobile: 6-month grace period in "test ready"
  - Verizon: Standard activation (no test ready period)

**Validation Rules**:
- Prevent saving active SIM flag when no SIM value exists
- All 18 scenarios in matrix represent valid configurations

## Additional Context for Implementation

**Test Ready Status** (confirmed today):
- On import: assume all SIMs are test ready
- Portal does not activate during import
- Richard to implement: "If test ready, keep test ready; otherwise activate" logic

**Edge Case Tabled**:
- Scenario: Dual carrier device on VZW-only config, VZW down, need to switch to ATT remotely
- Cannot solve through portal if device offline
- Will handle operationally (extremely rare)

## Next Steps & Timeline

**Immediate (Current Sprint)**:
1. **Richard**: Implement new mapping logic for all 18 scenarios
2. **Aksana**: Provide detailed ETA for all changes (import, assignment, update devices, mapping)
3. **Team**: Add Verizon/T-Mobile carrier combination to config builder
4. **QA**: Test all mapping scenarios, especially yellow-highlighted special cases

**Next Week**:
- Begin configuration engine overhaul discussion
- Review Adam's parameter analysis (700+ parameters documented with InHand)
- Define hierarchical config building system
- Determine which parameters exposed at which levels

## Action Items

1. **Devon**: Save and share mapping matrix document (screenshot attached)
2. **Richard**: Implement 18-row mapping logic considering SIM presence + active status
3. **Aksana**: Send comprehensive change summary and ETA for entire initiative
4. **Adam**: Share parameter documentation once remaining 36 values clarified by InHand
5. **All**: Review this summary before implementation begins

## Resources

- Meeting 1-3 recordings and transcripts available (Trista managing)
- Configuration Mapping Matrix screenshot attached
- Parameter documentation (pending - Adam has 664 of 700 documented)
- Meeting 2 follow-up email (import/assignment decisions)

This comprehensive plan addresses all decisions from our three-meeting series. The 18-row matrix successfully resolves the configuration mapping challenge identified in Meeting 2, while the import/assignment simplifications provide a clean workflow going forward.

Excellent collaborative work across all three sessions. We now have a clear, implementable plan!

Please review and confirm understanding before Richard begins implementation.

Best regards,
Aksana Rahouski
