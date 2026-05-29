# Product Requirements Document (PRD)
## Invoice Template Redesign

**Document Version:** 1.0
**Date:** January 13, 2026
**Author:** Aksana Rahouski
**Status:** Draft for Review
**Document Owner:** Aksana Rahouski

---

## Executive Summary

Client feedback from the January 13, 2026 meeting indicated that the current invoice format feels "too crowded" and cumbersome. This project will redesign the invoice PDF template to improve readability and scannability by grouping line items by service plan tier and simplifying descriptions.

### Key Changes
- **Tier Grouping**: Organize line items by Service Plan/Tier with clear section headers
- **Simplified Descriptions**: Remove redundant tier information from line item descriptions
- **Compact Layout**: Reduce whitespace and consolidate to single page where possible

### Business Impact
- Improved customer experience with easier-to-read invoices
- Reduced confusion and billing-related support questions
- More professional presentation of billing information

---

## Background and Problem Statement

### Current State

The current invoice template displays all line items in a flat list with verbose descriptions. Each line item includes:
- Full Service Plan name and limit (e.g., "Tier 3 (Up to 10GB)")
- Actual usage amount (e.g., "used less than 1.50 GB")
- Additional fees (e.g., "+ Dual Sim Device Fee")

Example: `Tier 3 (Up to 10GB), used less than 1.50 GB + Dual Sim Device Fee`

### Problems

**Problem 1: Poor Scannability**
- Line items are difficult to parse quickly due to verbose descriptions
- No visual grouping makes it hard to see patterns (e.g., all Tier 3 devices)
- 2-page invoices for moderate-sized accounts feel overwhelming

**Problem 2: Redundant Information**
- Tier limits are repeated on every line item even when they're the same
- Makes descriptions unnecessarily long and cluttered

### Impact if Not Addressed

- Customer dissatisfaction with invoice presentation
- Increased billing support questions
- Unprofessional appearance compared to industry standards

---

## Goals and Objectives

### Primary Goals

1. **Improve Invoice Readability**: Make invoices easier to scan and understand at a glance
2. **Reduce Visual Clutter**: Eliminate redundant information from line item descriptions
3. **Maintain Data Integrity**: Keep all essential billing information accurate and complete

### Success Criteria

- ✅ Line items are grouped by a Service Plan
- ✅ Line item descriptions are reduced to essential information only (usage + modifiers)
- ✅ Invoice layout is more compact without losing essential data
- ✅ All financial totals, quantities, and footer notes remain accurate
- ✅ Client approves new invoice format

### Non-Goals (Out of Scope)

- ❌ Changing invoice data structure or billing calculation logic
- ❌ Adding new invoice fields or metrics
- ❌ Email delivery format changes
- ❌ Invoice preview in customer portal (admin portal only)

---

## Target Users

### Primary Users

**1. Customer Finance/Billing Personnel**
- **Role**: Review and process invoices for payment
- **Need**: Clear, accurate billing breakdown they can quickly verify
- **Pain Point**: Current format is too crowded and hard to scan
- **Benefit**: Faster invoice review and approval process

**2. WATM Account Managers**
- **Role**: Explain invoices and answer customer billing questions
- **Need**: Professional invoice format that's easy to explain
- **Pain Point**: Customers complain about confusing invoice layout
- **Benefit**: Fewer billing questions and easier customer conversations

---

## Scope

### In Scope

**Invoice Template Changes**
- ✅ Group line items by a Service Plan (e.g., "Tier 1", "Tier 3", "Super Tier")
- ✅ Add visual section headers for each tier grouping
- ✅ Simplify line item descriptions to show only usage amount and modifiers
- ✅ Reduce whitespace and optimize layout for single-page format where possible
- ✅ Update invoice PDF generation logic

**Data Maintained**
- ✅ All pricing, quantities, and totals
- ✅ Billing method information
- ✅ Footer notes (Dual SIM fees, credit card fees, etc.)
- ✅ Total quantity summary

### Out of Scope (Future Enhancements)

**UI/UX Changes**
- ❌ Invoice preview in customer-facing portal
- ❌ Email template changes
- ❌ Additional invoice formats (CSV, Excel, etc.)

**Data/Functionality**
- ❌ Changes to billing calculation logic
- ❌ New data fields or breakdowns
- ❌ Historical invoice regeneration

---

## Functional Requirements

### FR-1: Service Plan Grouping

**FR-1.1: Group Line Items by Service Plan**
- System must organize invoice line items into sections based on Service Plan
- Sections should appear in consistent order: Tier 1, Tier 3, Super Tier (or by Service Plan hierarchy)
- Each section must display all line items belonging to that Service Plan

**FR-1.2: Section Headers**
- Each Service Plan section must have a clear visual header (e.g., bold text, background color, spacing)
- Header format: `[Service Plan Name] ([Usage Limit])`
  - Example: "Tier 3 (Up to 10GB)"
  - Example: "Super Tier (Up to 35GB)"

### FR-2: Simplified Line Item Descriptions

**FR-2.1: Usage Description Format**
- Remove Service Plan name and limit from individual line items (shown in section header)
- Display only: `<[usage amount]` + optional modifiers
- Examples:
  - `<1.50GB + Dual` (for items with Dual SIM fee)
  - `<100MB` (for items without additional fees)
  - `<1MB` (for minimal usage)

**FR-2.2: Modifier Display**
- Dual SIM Device Fee indicator: Display as "+ Dual" suffix
- Maintain consistent formatting across all line items
- Ensure modifiers are clearly distinguishable

### FR-3: Layout Optimization

**FR-3.1: Compact Header Area**
- Reduce whitespace in invoice header section
- Maintain all required information: Company name, billing address, invoice number, date

**FR-3.2: Table Spacing**
- Use tighter line spacing within sections while maintaining readability
- Target: Fit invoices with <25 line items on single page

**FR-3.3: Column Structure**
- Maintain existing columns: Description, Billing Method, Price, Qty, Total
- Adjust column widths to accommodate shorter descriptions

### FR-4: Data Validation and Integrity

**FR-4.1: Financial Accuracy**
- All pricing, quantities, and totals must remain mathematically accurate
- Subtotal and Total Price calculations must be unchanged
- Section subtotals are optional (can be added if helpful)

**FR-4.2: Footer Notes Preservation**
- Maintain all footer notes:
  - `*3% FEE APPLIED TO ALL CREDIT CARD TRANSACTIONS`
  - `++MINIMUM $7.95 PER PAYMENT METHOD`
  - `§§Dual SIM fee is $2.00`
- Maintain Total Qty summary

---

## Testing Requirements

### Integration Test Cases

**Test Case IT-1: Invoice Generation with Multiple Service Plans**
1. Generate invoice for account with devices across Tier 1, Tier 3, and Super Tier service plans
2. Verify line items are grouped by Service Plan with proper section headers
3. Verify all line items appear under correct Service Plan section
4. **Verify:** Subtotal and Total match expected amounts

**Test Case IT-2: Simplified Description Format**
1. Generate invoice with various usage patterns (GB, MB, with/without Dual SIM)
2. **Verify:** Descriptions show only usage amount (e.g., "<1.50GB + Dual")
3. **Verify:** Service Plan information does NOT appear in line item descriptions
4. **Verify:** Dual SIM modifier displays as "+ Dual"

**Test Case IT-3: Single Service Plan Invoice**
1. Generate invoice for account with devices all on same Service Plan
2. **Verify:** Section header appears correctly
3. **Verify:** Layout is clean and readable
4. **Verify:** No blank sections or formatting issues

**Test Case IT-4: Large Invoice (100+ line items)**
1. Generate invoice with 100+ devices across multiple Service Plans
2. **Verify:** Grouping works correctly with large data sets
3. **Verify:** Page breaks occur logically (not mid-section)
4. **Verify:** All line items are included, none missing

