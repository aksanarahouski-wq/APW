# APW Check-in — Meeting Summary

**Date:** April 16, 2026, 10:00 AM ET
**Duration:** ~66 minutes
**Organizer:** Aksana Rahouski
**Attendees:** Aksana Rahouski, Laura Perry, Noah Bratzel, Richard Sacco, Stone Marballie, Aaron Diefes (Orases) · Adam Curcie, Devon D'Andrea (APW)

---

## Overview

Weekly APW client check-in covering team introductions, active bug status, feature progress, SIM management issues, and an initial walkthrough of the Configuration V2 system design. The meeting surfaced a SIM import data error requiring immediate correction and led to scheduling a same-day follow-up deep dive on configurations.

---

## Key Discussion Topics

### Team Updates
- **Noah Bratzel** formally introduced to Adam and Devon — previously worked on WATM 2–3 years ago, has been back on the project for ~2 months
- Noah developed the **Verizon second account** feature, now with Aaron for QA testing

### Active Bugs
- **WATM-2070 — SIM card activation failure:** Reported via email. Stone jumped on it yesterday, fix in progress
- **WATM-1953 — Billing cycle job not validating both device and SIM status:** Stone implemented a query update to fix the SIM deactivation discrepancy. Needs QA testing in beta
- Stone created a **report for APW admins** showing devices pulled and flagging items that couldn't be verified

### Feature Progress
- **Commissions feature:** Approved and ready to deploy. Richard pushing the hotfix today
- **CakePHP releases 1 & 2:** Noah and Richard actively working on these
- **Verizon second account:** Aaron testing; estimates a few more days due to routing and service plan complexity
- **Company invitations:** Has feedback from Aaron. Stone to address after completing current bug work

### SIM Management Issues
- **SIM import process clarified:** Import as deactivated → assign to company → customer activates via bulk action
- **SIM import error discovered:** Garrett imported SIMs with "no" in the Active SIM column instead of "yes," causing bulk activation to fail (no enabled SIMs to activate)
- **Terminology change needed:** Rename SIM labels from "Active/Inactive" to **"Enabled/Disabled"** across import, bulk update, device view/edit, and export
- Need to add **Enable/Disable column to the SIM export**
- Bulk update needed to correct "Active SIM" flag from "no" to "yes" for affected Verizon SIMs

### Configuration V2 System Design (Initial Walkthrough)
- Schema contains ~700 parameters with categories, data types, defaults, validation rules, and availability flags
- **Three-way rules** combine model + carrier + service plan to set configuration values
- Configuration engine cascades through: schema defaults → model defaults → three-way rules → company overrides → device overrides
- Required parameters must have values from schema, model, three-way rule, or company override levels
- Model configuration requires selecting a subset of parameters applicable to specific device models
- **Company override sets** allow applying same parameters to multiple companies without model/carrier/service plan specificity; multiple sets per company allowed if parameters don't overlap
- Discussion of validation, push mechanisms, and configuration deployment strategy needed
- Configuration push needs on-demand capability in addition to check-in triggered updates

---

## Action Items

| Who | What | When |
|-----|------|------|
| Stone | Run QA testing on WATM-1953 (SIM deactivation discrepancy) in beta | ASAP |
| Richard | Push commissions hotfix to production | Today (April 16) |
| Stone | Bulk update SIM records — correct "Active SIM" flag from "no" to "yes" for Verizon SIMs | ASAP |
| Aksana | Create ticket to change SIM wording from "Active/Inactive" to "Enable/Disable" across import, bulk update, device view/edit, and export | Soon |
| Aksana | Schedule follow-up configuration design meeting | Done — scheduled for 4:00 PM ET today |
| Team | Read configuration design documents before follow-up meeting | Before 4:00 PM ET today |

---

## Decisions Made

- Commissions feature is **approved for deployment**
- SIM terminology will change from "Active/Inactive" to **"Enabled/Disabled"**
- Enable/Disable column will be **added to SIM export**
- Follow-up configs deep dive scheduled for **4:00 PM ET same day**

---

## Open Questions / Follow-ups

- Validation strategy for ensuring every device receives valid configuration when settings change
- On-demand configuration push mechanism (beyond check-in triggered updates)
- Team unavailable next week (traveling Tue–Fri) — next full sync likely the following week
- Adam to read through configuration documentation and provide Devon cliff notes
