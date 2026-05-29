**Subject: Service Plan Updates - Action Items & Data Migration Questions**

Hi Devon and Adam,

Thank you for the productive meeting yesterday. We're moving forward immediately with the decisions made and wanted to confirm our implementation approach and get guidance on the data migration.

---

## Meeting Summary

### Major Decisions Made

**1. Service Plan Pricing Structure - Simplified Approach**

*Decision:* Remove dual carrier pricing options from the service plan management interface

- Keep only 3 carriers in service plan configuration: Verizon, AT&T, T-Mobile
- Remove dual carrier combinations (Verizon/AT&T, Verizon/T-Mobile) from service plan pricing
- Model variations will remain but will primarily be used only for the ATM service plan
- For Tier 1, Tier 2, and Tier 3 plans: pricing variations will be carrier-based only, not model-based

**2. Dual Carrier Fee Handling - Move to Company Level**

*Decision:* Implement two separate dual carrier fee options at the company level

*New Structure:*
- Dual Carrier Fee (AT&T) - remains $4.95 default, customizable per company
- Dual Carrier Fee (T-Mobile) - new field with $3.50 default, customizable per company

*Billing Logic:*
- Devices with dual carrier will always be billed at the primary carrier rate (always Verizon)
- The dual carrier fee is applied based on which secondary carrier is configured
- Total data usage is charged at primary carrier rate regardless of which SIM actually transmitted the data
- Example: Device with Verizon primary + T-Mobile secondary using 10GB total = Verizon data rate + T-Mobile dual fee

**3. UI/UX Approach - Defer Major Redesign**

*Decision:* Keep current horizontal grouping layout, postpone tab-based interface

*Rationale:*
- Tabs would require significant additional development time
- Complexity of maintaining two different code bases for different service plans
- Need to meet February 6th deadline for Origin device billing
- Tab design needs real-world data to validate approach
- Current solution works functionally, even if less visually appealing

*Next Steps:*
- Deploy current horizontal layout to beta for testing
- Client will create realistic scenarios reflecting actual usage patterns
- Re-evaluate UI design after seeing real data in production
- May implement tabs later if business needs justify the investment

**4. Verizon Activation Fix**

*Decision:* Use a single global SKU for all SIM-only activations
- SKU ending in 0018 (from ticket 1936)
- Will work for all cases, updates automatically via OTA when device connects
- Stone to implement this quick fix

### Important Business Rules Clarified

**Dual Carrier Pricing Philosophy**
- Primary carrier is always Verizon for dual carrier devices
- Customers purchase "Verizon service" with a backup SIM
- Failover to secondary carrier doesn't change billing rate
- The dual carrier fee is for having a "hot" backup SIM, not for data routing

**Service Plan Variations**
- ATM Plan: Only plan that will use model-specific pricing variations
- Tier 1/2/3 Plans: Carrier variations only (primarily Verizon discounts)
- Adam estimated ~98% certainty that variations will only be Verizon-based
- Maximum expected tabs/variations per plan: 4-5 (not 20+)
- AT&T-only pricing unlikely to ever be created

**Timeline Constraints**
- Target: Beta deployment by end of week
- Hard Deadline: February 6th for production deployment
- Devon needs time after deployment to retroactively adjust Origin device pricing
- New Verizon Business Account discussion postponed to next week

---

## Implementation Timeline

**This Week - Beta Deployment:**

**1. Service Plan Pricing Structure - Simplified Approach**
- Remove dual carrier pricing options (Verizon/AT&T, Verizon/T-Mobile combinations) from service plan management interface
- Keep only 3 carriers: Verizon, AT&T, T-Mobile
- Maintain model variations capability (primarily for ATM service plan)
- Carrier-based variations only for Tier 1/2/3 plans

**Next Week (Pending Your Input):**

**2. Dual Carrier Fee Handling - Move to Company Level** *(New Ticket)*
- Add second dual carrier fee field to company settings page
- Update invoice logic to display and calculate separate fees based on device configuration
- ⚠️ **This work may extend into next week as we need your guidance on the data migration approach (see below)**

---

## 🚨 ACTION NEEDED: Data Migration Questions

For the dual carrier fee migration, we're moving from **1 field** to **2 fields**:
- **Dual Carrier Fee (AT&T)** - default $4.95
- **Dual Carrier Fee (T-Mobile)** - default $3.50

**We need you to decide how to handle existing customer data:**

**Option 1: Duplicate existing values**
- Copy current dual carrier fee to both AT&T and T-Mobile fields for all customers
- Example: Customer currently has $2.00 → Both new fields become $2.00

**Option 2: Set new defaults for everyone**
- AT&T field = $4.95 for all customers
- T-Mobile field = $3.50 for all customers
- ⚠️ This would override any custom pricing currently in place

**Option 3: Repurpose existing values for AT&T only**
- Keep current dual carrier fee values as "Dual Carrier Fee (AT&T)"
- Leave "Dual Carrier Fee (T-Mobile)" empty/null initially
- You would manually set T-Mobile fees as needed

**Option 4: Custom approach**
- Different strategy based on your business needs

**Please provide the following:**
1. Which option do you prefer for the data migration?
2. If Option 2, confirm the default values ($4.95 AT&T, $3.50 T-Mobile)?
3. Are there any specific customers that need special handling?

---

**Summary:**
- **Service Plan Pricing updates** will be in Beta by end of this week
- **Dual Carrier Fee changes** will be completed once we receive your guidance on data migration (likely next week)
- This timeline still supports your February 6th production deadline

Please let us know your preference on the data migration approach at your earliest convenience.

Thanks,
Aksana
