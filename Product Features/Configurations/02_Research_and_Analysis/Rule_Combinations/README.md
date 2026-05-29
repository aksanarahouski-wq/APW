# Rule Combinations Analysis

**Last Updated:** March 1, 2026

This folder contains analysis of all possible conditional rule combinations for the configuration management system.

---

## Current Documents

### Two_Way_Rule_Combinations_Analysis.md
**Status:** VALIDATED
**Conclusion:** Support ALL 6 two-factor combinations

Analysis demonstrates that different parameters require different two-factor combinations:
- Model + Carrier (mqtt_enable, advanced features)
- Carrier + Service Plan (traffic thresholds by carrier policy)
- Model + Service Plan (traffic thresholds by model tier)
- Carrier + Customer (customer carrier agreements)
- Model + Customer (customer model configurations)
- Service Plan + Customer (customer plan exceptions)

### Three_Way_Rule_Design_Validation.md
**Status:** VALIDATED - FINAL DESIGN
**Conclusion:** Support ONLY Model + Carrier + Service Plan (no other three-way combinations)

Comprehensive validation analysis proving that:
- Other three-way combinations (M+C+Customer, M+SP+Customer, C+SP+Customer) are not needed
- All scenarios covered by existing architecture (Two-Way or Four-Way Rules)
- Real-world evidence shows no patterns requiring other three-way types
- Complexity vs. benefit trade-off favors current design

### Four_Way_Rule_Combinations_Analysis.md
**Status:** VALIDATED
**Conclusion:** Support Model + Carrier + Service Plan + Customer (only combination possible)

Analysis of when to use Four-Way Rules:
- Customer-specific exceptions to service plan policies
- Only when all 4 factors matter simultaneously
- Relatively rare (estimated 10-20 rules)
- Higher priority than Three-Way Rules (Level 2 vs Level 4)

---

## Design Summary

### Validated Rule Architecture

**Layer-Based Configuration (Single Factor):**
1. Global Layer
2. Model Layer
3. Carrier Layer
4. Service Plan Layer
5. Company Layer
6. Device Layer

**Two-Way Conditional Rules (2 Factors) - 6 Types:**
1. Model + Carrier
2. Carrier + Service Plan
3. Model + Service Plan
4. Carrier + Customer
5. Model + Customer
6. Service Plan + Customer

**Three-Way Conditional Rules (3 Factors) - 1 Type:**
1. Model + Carrier + Service Plan (service plan baselines for all customers)

**Four-Way Conditional Rules (4 Factors) - 1 Type:**
1. Model + Carrier + Service Plan + Customer (customer-specific exceptions)

---

## 11-Level Priority Hierarchy

When resolving configuration parameters:

1. Device Override (highest priority)
2. Four-Way Rule (customer exception)
3. Company Override
4. Three-Way Rule (service plan baseline)
5. Service Plan Layer
6. Two-Way Rule (6 sub-priorities)
   - 6.1: Carrier + Service Plan
   - 6.2: Model + Service Plan
   - 6.3: Model + Carrier
   - 6.4: Carrier + Customer
   - 6.5: Model + Customer
   - 6.6: Service Plan + Customer
7. Carrier Layer
8. Model Layer
9. Global Layer
10. Schema Default
11. Required Validation (lowest priority)

---

## Key Findings

### Real-World Evidence
- Production config analysis validates current design
- ATM plan baselines are often model+carrier independent
- Customer exceptions are service-plan-specific
- Most parameters use simple patterns (layers or two-way rules)

### Design Principles
- Use SIMPLEST rule type that accurately models the dependency
- Three-Way Rules for service plan baselines (all customers)
- Four-Way Rules for customer-specific exceptions
- Two-Way Rules for simpler multi-factor dependencies

### Coverage Validation
- All identified use cases covered by current design
- No gaps found in production configuration analysis
- System provides complete flexibility without unnecessary complexity

---

## Archive

**Three_Way_Rule_Combinations_Analysis.md.archive** - Earlier analysis document; superseded by Three_Way_Rule_Design_Validation.md

---

**Document Status:** Analysis Complete - Design Validated
