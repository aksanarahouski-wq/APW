# Meeting Summary: Verizon Second Account Discussion

## Overview
This meeting covered multiple topics including a service plan bug, daily usage charts, and most significantly, the implementation requirements for supporting Verizon's second business account.

---

## **Verizon Second Account: Key Discussion Points**

### **What You Discussed**

The team needs to implement support for a second Verizon account (Verizon Business Internet/FWA) alongside the existing standard Verizon account. The discussion focused on three main areas:

#### **1. Device-Level Configuration**
- **Agreement:** Devices with Verizon SIMs need a way to specify which Verizon account they belong to (Account A - standard, or Account B - business)
- This information must be captured during:
  - Manual device creation
  - Device updates
  - Device imports (bulk operations)
- The device needs this information to know which Verizon API endpoint to call for activation/deactivation

#### **2. Service Plan Restrictions**
- **Agreement:** Business Internet devices require special service plans that are dramatically different from standard plans
- Service plans for Business Internet accounts won't need:
  - Usage limits
  - Device group names
  - Most standard required fields
- Only basic fields are needed: name, a few configuration options (enable WiFi, enable firewall)

#### **3. Device-to-Service Plan Matching**
- **Agreement:** The system must enforce that Business Internet devices can **only** be assigned to Business Internet service plans
- This validation needs to occur in:
  - Device assignment page (bulk operations)
  - Single device service plan changes
  - Any workflow where devices get mapped to service plans

### **What You Agreed On**

1. **Two-tier approach:** Store account information at the device level AND create account-specific service plans
2. **Validation enforcement:** The system must prevent mismatches between device account type and service plan account type
3. **UI simplicity:** A checkbox approach could work for the immediate need (FWA checkbox), though scalability beyond two accounts was noted as a concern
4. **No customer self-service:** Customers cannot upgrade/downgrade between standard and Business Internet accounts themselves - only admins can perform this
5. **Manual migration process:** Moving devices between accounts is highly complex, time-sensitive, and cannot be automated (requires simultaneous APN changes, deactivation on one account, reactivation on the other within a 2-3 minute window)

### **Outstanding Discussion Items**

1. **Implementation details for "the magic":**
   - Exactly what happens when the Business Internet checkbox is checked/unchecked
   - How to handle the 15 existing Business Internet devices already in the portal
   - Proper validation rules and error handling for bulk operations
   - How to prevent accidental configuration pushes to Business Internet devices during rollout

2. **Scalability considerations:**
   - T-Mobile may need similar treatment with different plan codes (not separate accounts)
   - The checkbox solution works for two options but breaks down if a third account type is added
   - Should the solution be designed more generically now or wait until needed?

3. **Import/update workflow details:**
   - How device updates should handle the new account field
   - Which spreadsheet columns need to be added
   - How to migrate existing Business Internet devices to the new system

4. **Service plan creation workflow:**
   - Exact UI changes needed for the "Add Service Plan" page
   - Whether to make most fields optional when Business Internet is selected
   - How to clearly indicate which service plans are for which account type

### **Next Steps**

- Schedule a 30-minute follow-up meeting (week after next, after Adam returns) to finalize the Business Internet implementation approach
- Create a ticket for the device update import behavior (separate issue discussed at end of meeting)
- Stone and Richard to continue investigating the service plan pricing bug that was reproduced during the meeting

---

## **Other Topics Covered**

**Service Plan Bug:** A critical bug was confirmed where custom pricing on child service plans gets overwritten when the parent ATM Unlimited plan is modified. The bug was reproduced during the meeting. The team rolled back to a previous plan version temporarily.

**Daily Usage Chart:** The backend calculations are already running in production (as of Feb 5). The frontend will be held back for 2-4 weeks to allow sufficient data accumulation before showing customers the feature. The team agreed this was the right approach to avoid customer confusion about missing historical data.
