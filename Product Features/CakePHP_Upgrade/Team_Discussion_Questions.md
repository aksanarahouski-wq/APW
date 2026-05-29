# CakePHP 5.x Upgrade — Team Discussion Questions

**Purpose:** Supporting document for roadmap review meeting with lead dev and lead QA
**Meeting Prep For:** Aksana Rahouski
**Roadmap Reference:** [CakePHP_5x_Upgrade_Roadmap.md](./CakePHP_5x_Upgrade_Roadmap.md)
**Date:** March 17, 2026

---

## How to Use This Document

Questions are grouped by topic, not by phase. For each question there's context on why it matters and what the roadmap currently assumes. Bring this to the meeting, work through the sections, and capture decisions to fold back into the roadmap.

---

## 1. Deployment Strategy & Release Sequencing

The roadmap proposes three independent production releases. Need team alignment on whether this is the right approach.

**For Lead Dev:**

- [ ] **Do you agree with shipping Phase 0 (CakePHP 4.6 + deprecation fixes) to production separately?** The spike on branch `noah/spike-cakephp-4.5-upgrade` shows all 344 tests pass. Is there any reason NOT to ship this early and get it out of the way?

- [ ] **Can PHP 8.2 (Release 2) ship independently of the CakePHP 4.6 bump (Release 1), or do they need to go together?** The roadmap says they can run in parallel, but is there a dependency we're missing?

- [ ] **For Release 3 (CakePHP 5), should Phases 2–4 be on a feature branch, or do we work directly on a long-lived upgrade branch?** How do we handle other feature work and bug fixes shipping to production while Phases 2–4 are in progress on review?

- [ ] **What's the branching strategy during the upgrade?** If a critical production bug comes in while we're mid-Phase 3 on review, how do we hotfix production (still on CakePHP 4.6) while the upgrade branch has CakePHP 5 code?

**For Lead QA:**

- [ ] **Are you comfortable with targeted manual QA after Phases 0 and 1 being the gate for production?** Or do you want a broader manual QA pass before shipping those releases?

- [ ] **For Phases 2–4 (review only), is targeted manual QA sufficient to proceed between phases, or do you want a broader check each time?** The roadmap limits QA to sections directly affected by each phase's changes.

---

## 2. The `orases/*` Packages — Ownership & Access

The biggest risk in this upgrade is the private `orases/*` packages, especially `orases/users`. Need clarity on logistics.

**For Lead Dev:**

- [ ] **Do we have access to fork/modify the `orases/*` packages?** The TRD says "fork/upgrade" for `logs`, `users`, and `sites`. Do we have write access to these repos on `packages.orases.com`, or do we need to request access / create our own forks?

- [ ] **Who owns these packages today?** Are there other projects using them that could be affected by our changes? Do we need to coordinate with anyone?

- [ ] **For `orases/imports` — the roadmap says inline 3 files and drop the dependency. Is there any reason to keep it as a package?** Is any other project using it?

- [ ] **For `orases/users` + CakeDC/Users v9 → v14 — have you looked at the CakeDC v14 changelog?** The TRD lists 6 critical files and estimates 16–30 hours. Does that feel right based on what you've seen in the code? Any hidden complexity?

- [ ] **The dependency chain is sequential: `logs` → `users` → `sites`. Can any of this be parallelized?** For example, can someone start on `orases/sites` scaffolding while `orases/users` is still in progress?

---

## 3. Test Coverage & Automated Tests

Phase 0c calls for 12–20 hours of writing new PHPUnit tests before fixing deprecations. Need alignment on scope.

**For Lead Dev:**

- [ ] **Is 12–20 hours realistic for the test coverage hardening in Phase 0c?** The TRD identifies 240 `Entity::has()` usages and 20 `Table::query()` usages. The roadmap says focus on billing, notifications, and device status paths — not every usage. Does the dev team agree with that scoping?

- [ ] **Which `Entity::has()` usages are the scariest?** The code audit calls out `PowerScheduleEvaluator`, `Company` entity, `DeviceCheckinsTable`, and `WatmRules`. Are there others that worry you?

- [ ] **After the upgrade is complete, what's the target test coverage?** Right now there are 344 tests. Phase 0c will add more. Should we set a coverage target, or just focus on the high-risk areas and move on?

**For Lead QA:**

- [ ] **Do the existing 344 automated tests give you confidence, or do you feel there are major gaps?** Are there areas of the application that you know break frequently but have no automated test coverage?

---

## 4. Manual QA — Scope, Timing, and Resources

The roadmap assumes one tester full-time for Phase 5 (6.5–10 days). Need to validate this.

**For Lead QA:**

- [ ] **Is one tester realistic for the full 26-section walkthrough in Phase 5?** Or do we need two testers to hit the 6.5–10 day estimate? Would two testers cut it to ~4–5 days?

- [ ] **Does the QA Test Plan cover everything you'd want to test?** Are there sections missing? The plan was written by Aaron — have you reviewed it and do you agree with the P0/P1/P2 prioritization?

- [ ] **For the targeted manual QA in Phases 2–4, do you want to be the one doing it, or can a developer do those checks?** They're short (4–10 hours each) and focused on specific features. A developer who knows what they changed might be faster, but a fresh set of eyes might catch more.

- [ ] **How do you want to handle QA failures in Phase 5?** If the tester finds a P0 issue on day 3 of a 7-day pass, does the developer fix it and the tester re-tests just that section? Or does the tester restart the full walkthrough from the beginning?

- [ ] **Do we need test data seeded on review for Phase 5?** The Test Infrastructure page flags that review/beta have minimal data compared to production's hundreds of millions of rows. Is that enough to test billing cycles, tiered pricing, distributor commissions, etc.? Or do we need the data seeding scripts from Phase 1d before QA can be effective?

- [ ] **The "Critical Regression Check" says compare invoice totals line-by-line against pre-upgrade values. Do we have a way to do that?** Do we need to save a snapshot of current invoice data before starting the upgrade so the tester has something to compare against?

---

## 5. Environment & Infrastructure

Several assumptions about environments need validation.

**For Lead Dev:**

- [ ] **PHP 8.2 on Ubuntu 22.04 via `ondrej/php` PPA — has ops done this before?** Or should we consider upgrading to Ubuntu 24.04 (ships PHP 8.3)? What's the ops team's preference?

- [ ] **The review and beta environments share real carrier API accounts (Verizon, AT&T, T-Mobile) with production.** Is there any risk that manual QA testing on review could affect production carrier state? For example, if the tester changes a device status on review, does that trigger a real carrier API call?

- [ ] **Deprecation warnings are suppressed on ALL environments today.** Phase 1c says enable `E_ALL` on review. Is there a reason they were suppressed? Will enabling them flood the error logs and make it hard to spot real issues?

- [ ] **The webhook replay system (Phase 1d) is marked "optional but recommended." Should we make it required?** Without it, how do we test Verizon callback handling and Adobe Sign webhooks on review?

---

## 6. Timing & Scheduling

The roadmap doesn't specify calendar dates. Need to discuss sequencing with other work.

**For Both:**

- [ ] **When do we start?** CakePHP 4.x security fixes end in 2026. Are we starting this quarter, next quarter?

- [ ] **How does this fit alongside the Configuration Management project?** The existing roadmap in the repo (`Simplified_Implementation_Roadmap.md`) tracks a separate multi-phase initiative. Are we doing these in parallel with different developers, or sequentially?

- [ ] **Can the dev work on Phases 2–4 be done by one developer, or do we need two?** The TRD estimates 2–3 weeks focused dev time. Is one developer going to be dedicated to this, or splitting time with other work?

- [ ] **What's the calendar impact of Phase 5 (6.5–10 days of full-time QA)?** Does the QA tester have other commitments? Do we need to book them in advance?

- [ ] **Is there a hard deadline?** CakePHP 4.x security fixes through 2026 — but is there a business deadline (audit, compliance, client requirement) that's sooner?

---

## 7. Risk & Rollback

The roadmap identifies risks but some need team validation.

**For Lead Dev:**

- [ ] **The rollback plan for Release 3 says "keep previous release tagged and deployable." Are the CakePHP 5 database migrations backward-compatible?** The CakeDC/Users v14 upgrade adds 4 new columns (last_login, lockout_time, failed_password_attempts, avatar type). Can CakePHP 4.6 code run against a database with those columns, or does rollback require a migration down?

- [ ] **What happens to data written by CakePHP 5 if we roll back to CakePHP 4.6?** For example, if CakePHP 5 writes `DateTime` objects differently than `FrozenTime`, could CakePHP 4.6 misread that data after rollback?

- [ ] **The `Entity::has()` behavior change (Phase 0d + Phase 4) — are there any billing cycle close or invoice generation paths where getting this wrong could corrupt financial data?** The code audit flags `WatmRules` (6 usages) and `Company` entity (5 usages) as critical. How bad is the worst case?

**For Lead QA:**

- [ ] **If we deploy Release 3 to production and discover a P0 issue after 24 hours (e.g., carrier usage collection stopped), what's the QA expectation?** Rollback immediately, or hotfix forward? Does the tester need to re-run the full 26-section walkthrough after a rollback?

---

## 8. Assumptions to Validate

These are things the roadmap assumes to be true based on the TRD. Need the team to confirm.

| # | Assumption | Source | Who Should Confirm |
|---|-----------|--------|-------------------|
| 1 | The spike branch (`noah/spike-cakephp-4.5-upgrade`) is still valid and up-to-date with current main | TRD Phase 0 | Lead Dev |
| 2 | All third-party packages have CakePHP 5 compatible versions (confirmed by Composer Upgrade Checker) | Dependency Audit | Lead Dev |
| 3 | Queue system (26 jobs, 24 queues) requires zero code changes | Dependency Audit | Lead Dev |
| 4 | Templates (289 files) are already CakePHP 5 compliant | Code Audit | Lead Dev |
| 5 | All 5 custom middleware classes are already PSR-15 compliant | Code Audit | Lead Dev |
| 6 | `orases/files` ^3.0, `orases/helpers` ^2.0, `orases/theme-limitless` ^2.0 are truly CakePHP 5 ready (just bump, no code changes) | Dependency Audit | Lead Dev |
| 7 | RBAC format, event constants, and AuthorBehavior API are unchanged in CakeDC/Users v14 | Dependency Audit | Lead Dev |
| 8 | The QA Test Plan (26 sections) covers all critical business workflows | QA Test Plan | Lead QA |
| 9 | One tester can complete the full 26-section walkthrough in 6.5–10 days | QA Test Plan | Lead QA |
| 10 | Review environment has enough data to meaningfully test billing, invoicing, and commissions | Test Infrastructure page | Lead QA |

---

## Meeting Output

After the meeting, capture:

1. **Decisions made** — fold back into the roadmap
2. **Open items** — assign owners and due dates
3. **Estimate adjustments** — update hours if the team disagrees with any estimates
4. **Calendar dates** — when does each release start and target completion?
5. **Resource assignments** — who is doing what?
