 Hi team,

  Thank you for the productive discussion today on implementing dual-SIM device management for the new Origin device model and enhancing our
  existing system. Here's a summary of the key ideas, decisions, and next steps from our meeting.

  Key Ideas Discussed

  New Device Model (Origin)
  - New manufacturer and model with eSIM capability (Verizon)
  - Physical SIM slot for T-Mobile
  - No IO digital port (configuration differences from I-22)
  - Requires dual-SIM checkbox enabled in device config for eSIM to register, even for single-carrier deployments

  SIM Management Philosophy
  - Pre-designated inventory approach: devices are imported with intended carrier configuration
  - Rarely redesignate devices post-import (hasn't happened in 3+ years)
  - Warehouse maintains separate inventory for each carrier combination
  - Import determines active status; assignment should not modify it

  Decisions Made

  1. Import Process Enhancement
  - Add active status columns to import template for each carrier (Yes/No format):
    - Verizon SIM / Verizon Active
    - AT&T SIM / AT&T Active
    - T-Mobile SIM / T-Mobile Active
  - Maximum of 2 SIMs can be imported per device (validated by system)
  - If both SIMs imported as active, dual flag automatically set to true
  - Import will NOT trigger carrier API calls - devices imported with SIMs already in appropriate status (test ready, etc.)

  2. Assignment Process Simplification
  - Remove carrier selection options from assignment screen
  - Remove "maintain existing SIM" checkbox
  - Assignment only ties device to company without modifying SIM status
  - Keep assignment process clean

  3. Device Update Process
  - Carrier API calls triggered only during device updates (not import or assignment)
  - Smart status logic for AT&T:
    - If current status is "test ready" → keep test ready (unlimited free period)
    - If current status is "deactivated" → set to active
    - Once test ready is exited, cannot return to test ready
  - Similar logic applies to T-Mobile (6-month test ready grace period)
  - Verizon follows standard activation process

  4. Bulk Operations
  - Use "Update Devices" feature for bulk SIM status changes, and update the system to support it.
  - Do not use the assignment page for bulk device SIMs modifications
  - Import template also needs to supports bulk updates with new active status columns


  Open Items for Tomorrow's Discussion

  Configuration Mapping
  - How to properly map device configurations based on SIM positions and carrier combinations
  - Edge case: AT&T-only device with SIM in position 2 (currently risky scenario)
  - Current limitation: All AT&T-only configs assume SIM in position 1
  - Need to determine: hard-coded system rules vs. configurable layer approach

  Known Constraints
  - I-22 devices: Verizon always in SIM 1, other carrier in SIM 2
  - Origin devices: Verizon as eSIM, T-Mobile as physical SIM
  - Limited carrier combinations reduce complexity



 Let me know if I've missed anything or if clarifications are needed.

 Best regards,
Aksana Rahouski
