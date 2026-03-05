# Carrier API Calls - Discussion & Analysis

## **Current Situation**

### **What We Did:**
- The carrier API calls (Verizon/AT&T) were removed from the service plan approval workflow as requested
- The functionality still exists in the codebase but is now disconnected
- **74 Verizon devices in production** have a mismatch: `vz_group_applied` ≠ Service Plan's Verizon group

### **What the Code Was Doing:**
- During service plan approval, the system checked for devices where `vz/att_group_applied` (device level) didn't match the Service Plan's Verizon or AT&T group
- When mismatches were found, it triggered API calls to Verizon/AT&T to update carrier-side device groups
- When these API calls failed, they blocked the entire approval process

### **Your Guidance:**
- Only call carrier APIs when actually changing values that associate service plans with carrier device groups
- Don't call APIs for changes that are "only changing system billing values"

---

## **Questions to Address**

### **1. Understanding the Functionality**

**"What does this carrier API call actually do?"**
- Does it update device group associations on the Verizon/AT&T side?
- Does it change billing groups in the carrier's system?
- Does it affect device configurations or just metadata?
- Is this required for proper billing to flow through from the carrier?

**"Why do we have 74 devices with group mismatches?"**
- Is this normal/expected state?
- Did these devices fail to update during a previous approval?
- Are they orphaned from old processes?
- Do they represent actual billing problems?

### **2. Determining When It Should Trigger**

**"When SHOULD we update carrier device groups?"**

Let's walk through these scenarios together:

#### **Scenario A: Service Plan Changes**
- Adding new pricing tiers (like Origin T-Mobile $3.50)
- Changing existing prices
- Adding/removing device group names
- **Question:** Do any of these require carrier API calls?

#### **Scenario B: Device Group Field Changes**
- Admin explicitly changes the "Verizon Group" or "AT&T Group" field on the service plan
- **Question:** Is THIS the only time we should call the carrier API?

#### **Scenario C: Device Assignment**
- Device gets assigned to a service plan for the first time
- Device switches from one service plan to another
- **Question:** Should the carrier API be called during device assignment instead?

#### **Scenario D: Mismatch Reconciliation**
- We discover 74 devices where `vz_group_applied` doesn't match the service plan
- **Question:** Should there be a separate admin tool to reconcile these mismatches?

### **3. Understanding the Business Impact**

**"What breaks if these carrier groups are NOT synchronized?"**
- Does billing fail or become inaccurate?
- Do devices lose connectivity?
- Is it just metadata that gets out of sync?
- How do you currently detect and fix these issues?

**"What's the correct source of truth?"**
- Should the Service Plan's Verizon/AT&T group always match `vz/att_group_applied` on devices?
- Or can they legitimately differ in some cases?

---

## **Discussion Approach**

### **Opening:**
We successfully removed the carrier API calls from the service plan approval workflow as requested. Now we need to determine where this functionality belongs long-term. Before we talk about 'where,' let's make sure we understand 'what' and 'why.'

### **Discovery:**
1. Review what the code was doing (documented above)
2. Clarify: "What does this carrier API endpoint actually do on Verizon/AT&T's side?"
3. Understand: "Why do we have 74 devices with group mismatches right now?"
4. Determine impact: "What happens if we never sync these groups? What breaks?"

### **Scenario Review:**
Walk through Scenarios A-D above to understand when carrier API calls should be triggered.

### **Implementation Options:**

#### **Option 1: Service Plan Field-Specific Trigger**
- Only call carrier APIs when the "Verizon Group" or "AT&T Group" field is explicitly changed
- Skip API calls for all other service plan changes (pricing, names, etc.)

#### **Option 2: Separate Device Assignment Workflow**
- Call carrier APIs during device assignment, not service plan approval
- Ensures devices get synced when they're actually being assigned to plans

#### **Option 3: Admin Reconciliation Tool**
- Create a separate admin interface to find and fix mismatches
- Allows manual review before triggering carrier APIs
- Useful for the 74 existing mismatched devices

#### **Option 4: Async Background Process**
- Queue a background job that periodically checks for mismatches
- Attempts to sync them with proper retry logic
- Logs failures for admin review

#### **Option 5: Do Nothing**
- If carrier group sync isn't critical, maybe it's not needed at all
- Archive the functionality as dead code

### **Decision:**
Based on our discussion, which option makes the most sense? And for the 74 devices currently out of sync - should we handle those separately?

---

## **Information to Gather**

### **For This Discussion:**
- [ ] Screenshots of the Service Plan edit page showing Verizon/AT&T group fields
- [ ] Example of what `vz_group_applied` looks like on a device record
- [ ] List of the 74 mismatched devices (or at least a sample)
- [ ] Code snippet showing what the carrier API call was doing (if available)

### **Questions We'll Need to Answer:**
- "How long will each option take to implement?"
- "What's the risk of leaving this functionality out completely?"
- "Can we test the carrier API calls in review/beta environments?"

---

## **Next Steps**

1. Document decisions made about when/where carrier API calls should trigger
2. Update project scope documentation with the agreed-upon solution
3. Create implementation tickets based on chosen option
4. Address the 74 existing mismatched devices (separate ticket if needed)

---

## **Related Documentation**
- See: ServicePlanApproval/Scope.md for original issue exchange
- Context: 74 Verizon devices in production with group mismatches
- Decision: API calls removed from approval workflow (Feb 2026)
