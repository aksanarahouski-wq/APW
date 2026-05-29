# Client Feedback - January 22, 2026

## Meeting Summary: Service Plan Management Feature

### Key Topics Discussed

1. **Verizon SIM Activation Issue**
2. **New Verizon Business Account Setup**
3. **Service Plan Management UI/UX** (primary focus)

---

## Major Decisions Made

### 1. **Service Plan Pricing Structure - Simplified Approach**

**Decision:** Remove dual carrier pricing options from the service plan management interface

- **Keep only 3 carriers** in service plan configuration: Verizon, AT&T, T-Mobile
- **Remove dual carrier combinations** (Verizon/AT&T, Verizon/T-Mobile) from service plan pricing
- **Model variations** will remain but will primarily be used only for the ATM service plan
- For Tier 1, Tier 2, and Tier 3 plans: pricing variations will be **carrier-based only**, not model-based

### 2. **Dual Carrier Fee Handling - Move to Company Level**

**Decision:** Implement two separate dual carrier fee options at the company level

**New Structure:**
- **Dual Carrier Fee (AT&T)** - remains $4.95 default, customizable per company
- **Dual Carrier Fee (T-Mobile)** - new field with $3.50 default, customizable per company

**Billing Logic:**
- Devices with dual carrier will **always be billed at the primary carrier rate** (always Verizon)
- The dual carrier fee is applied based on which secondary carrier is configured
- Total data usage is charged at primary carrier rate regardless of which SIM actually transmitted the data
- Example: Device with Verizon primary + T-Mobile secondary using 10GB total = Verizon data rate + T-Mobile dual fee

### 3. **UI/UX Approach - Defer Major Redesign**

**Decision:** Keep current horizontal grouping layout, postpone tab-based interface

**Rationale:**
- Tabs would require significant additional development time
- Complexity of maintaining two different code bases for different service plans
- Need to meet February 6th deadline for Origin device billing
- Tab design needs real-world data to validate approach
- Current solution works functionally, even if less visually appealing

**Next Steps:**
- Deploy current horizontal layout to beta for testing
- Client will create realistic scenarios reflecting actual usage patterns
- Re-evaluate UI design after seeing real data in production
- May implement tabs later if business needs justify the investment

### 4. **Verizon Activation Fix**

**Decision:** Use a single global SKU for all SIM-only activations

- SKU ending in 0018 (from ticket 1936)
- Will work for all cases, updates automatically via OTA when device connects
- Stone to implement this quick fix

---

## Important Business Rules Clarified

### Dual Carrier Pricing Philosophy
- **Primary carrier** is always Verizon for dual carrier devices
- Customers purchase "Verizon service" with a backup SIM
- Failover to secondary carrier doesn't change billing rate
- The dual carrier fee is for having a "hot" backup SIM, not for data routing

### Service Plan Variations
- **ATM Plan:** Only plan that will use model-specific pricing variations
- **Tier 1/2/3 Plans:** Carrier variations only (primarily Verizon discounts)
- Adam estimated **~98% certainty** that variations will only be Verizon-based
- Maximum expected tabs/variations per plan: 4-5 (not 20+)
- AT&T-only pricing unlikely to ever be created

### Timeline Constraints
- **Target:** Beta deployment by end of week
- **Hard Deadline:** February 6th for production deployment
- Devon needs time after deployment to retroactively adjust Origin device pricing
- New Verizon Business Account discussion postponed to next week

---

## Technical Changes Required

1. **Remove dual carrier options** from service plan pricing interface
2. **Add second dual carrier fee field** to company settings page
3. **Update invoice logic** to:
   - Display separate dual carrier fees for AT&T vs T-Mobile
   - Select appropriate fee based on device secondary carrier configuration
4. **Maintain model variations** capability (even if rarely used)
5. **Keep horizontal grouping layout** for all service plans

---

## Open Items for Follow-up

- Verizon Business Account setup (separate meeting next week)
- Service plan differentiation for new business account (likely by service plan assignment)
- Final testing with realistic data scenarios before production deployment

---

## Key Quotes

**On dual carrier pricing variations:**
> "For our Tier 1, Tier 2, and Tier 3 service plans, I don't anticipate us ever making model differentiated pricing changes. It'll just be carrier." - Adam

**On pricing logic:**
> "What they bought is a Verizon primary service plan. So that is $4... to have the backup SIM, regardless if it switches to T-Mobile or stays on Verizon, it's a $3.50 fee." - Devon

**On timeline priority:**
> "I have— we have a lot to do for the ATM customers that have already started buying Origins. Okay, that's my priority." - Devon

---

## Conclusion

The team successfully reached consensus to simplify the approach, move dual carrier pricing to the company level, and defer major UI redesign until real-world usage validates the need for tabs.
