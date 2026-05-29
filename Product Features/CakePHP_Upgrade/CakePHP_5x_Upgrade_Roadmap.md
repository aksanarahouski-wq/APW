# CakePHP 5.x Upgrade Roadmap

**Purpose:** Internal team planning for upgrading WATM from CakePHP 4.4 to 5.x
**Status:** Draft
**Last Updated:** March 19, 2026
**TRD Reference:** [CakePHP 5.x Upgrade - TRD](https://orases.atlassian.net/wiki/spaces/WATM/pages/2968289286/CakePHP+5.x+Upgrade+-+TRD)
**QA Test Plan:** [WATM CakePHP 4 to 5 Upgrade — QA Test Plan](https://orases.atlassian.net/wiki/spaces/WATM/pages/2964160521/WATM+CakePHP+4+to+5+Upgrade+QA+Test+Plan)

---

## Executive Summary

This roadmap was developed from findings in the [TRD](https://orases.atlassian.net/wiki/spaces/WATM/pages/2968289286/CakePHP+5.x+Upgrade+-+TRD) (code audit, dependency audit, and phase-level technical analysis) and the [QA Test Plan](https://orases.atlassian.net/wiki/spaces/WATM/pages/2964160521/WATM+CakePHP+4+to+5+Upgrade+QA+Test+Plan) (26-section manual regression plan). The upgrade is split into three independent production releases:

**Release 1 (Phase 0) — CakePHP 4.6 + Deprecation Fixes (~24–40 hrs)**
Stay on CakePHP 4.x but bump from 4.4 → 4.6, then fix all deprecation warnings. This is the "clean the house before the move" phase. The work is mostly mechanical: replacing deprecated method calls like `query()` → `selectQuery()`, `has()` → `hasValue()`, `loadModel()` → `fetchTable()`, plus writing new PHPUnit tests around billing and device status paths to catch regressions. A spike already confirmed 4.6 works (344 tests pass). Low risk, ships to production first.

**Release 2 (Phase 1) — PHP 8.2 + Test Infrastructure (~9–16 hrs)**
Upgrade the runtime from PHP 8.1 → 8.2 (required for CakePHP 5) and modernize test tooling (PHPUnit 8/9 → 10). Also turn on `E_ALL` deprecation visibility and optionally build a webhook replay system for safer testing. This is a runtime/tooling change only — no application code changes. Can run in parallel with Release 1.

**Release 3 (Phases 2–6) — The Actual CakePHP 5 Upgrade (~108–215 hrs)**
The main upgrade, broken into five sub-phases:
- **Phase 2** — Upgrade the 7 private `orases/*` packages to be CakePHP 5 compatible. The `orases/users` + CakeDC/Users v9→v14 jump is the single biggest risk (~25% of total effort), involving auth, RBAC, 2FA, and multi-tenancy. Confidence is low because nobody has started this work and the packages haven't been updated in 5+ years.
- **Phase 3** — Run automated upgrade tools (Rector, dereuromark's tool) to handle bulk renames, type changes, and `FrozenTime` → `DateTime` across the whole codebase. First time the app actually runs on CakePHP 5.
- **Phase 4** — Manual catch-all for everything the automated tools miss: removing `RequestHandlerComponent`, fixing the search plugin syntax, remaining `Entity::has()` call sites, skeleton file updates, and PHPStan cleanup.
- **Phase 5** — Full 26-section manual QA regression (6.5–10 days). This is the production gate. Developer fixes bugs found during QA (0–16 hrs).
- **Phase 6** — Deploy to beta then production, with post-deploy spot checks and 24–48 hr monitoring. Rollback plan is revert to the tagged CakePHP 4.6 release.

**Total estimate: ~141–271 hours** (66–155 dev + 75–116 QA). The key principle: ship the safe stuff (Releases 1 & 2) to production early so the codebase is clean before the risky CakePHP 5 swap, and keep Release 3 on the review environment with targeted QA gates until the full regression passes.

---

## Current State

- **CakePHP:** 4.4.* (spike confirmed 4.6.3 works — all 344 tests pass)
- **PHP:** 8.1.2 (production) — must upgrade to 8.2+
- **PHPUnit:** ~8.5.0 || ^9.3 — needs ^10.x
- **Plugins:** 17 internal + third-party + 7 private `orases/*` packages
- **4.x support:** Bugfixes ended. Security fixes through 2026. On borrowed time.

---

## Two Types of Testing in This Roadmap

This roadmap tracks two separate testing activities. They are done by different people at different times:

1. **Automated tests (PHPUnit, PHPStan, code style)** — Run by developers as part of dev work. These are command-line tools. Time is included in the "Dev Work" estimates.

2. **Manual QA (clicking through the website)** — A QA tester manually walks through the application in a browser, following the [QA Test Plan](https://orases.atlassian.net/wiki/spaces/WATM/pages/2964160521/WATM+CakePHP+4+to+5+Upgrade+QA+Test+Plan). This plan has 26 sections covering login, device management, billing, invoicing, NACHA, commissions, company management, API keys, and more. Each section is a series of steps a person performs in a browser. Time is shown separately in the "Manual QA" column.

> **The QA Test Plan sections referenced throughout this document are all manual, browser-based testing — a person logging in, clicking through pages, verifying data, and comparing invoice totals on screen.**

---

## Deployment Strategy

Not everything ships at once. This upgrade has **three independent production releases**, each with its own QA gate:

```
┌─────────────────────────────────────────────────────────────────────────┐
│  RELEASE 1: CakePHP 4.6 + Deprecation Fixes (still on CakePHP 4.x)   │
│                                                                         │
│  Phase 0 ──► Targeted Manual QA ──► Deploy to Production                │
│                                                                         │
│  Low risk. Non-breaking. Spike already confirmed. Ships first.          │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│  RELEASE 2: PHP 8.2 + Test Infrastructure (still on CakePHP 4.6)      │
│                                                                         │
│  Phase 1 ──► Targeted Manual QA ──► Deploy to Production                │
│                                                                         │
│  Low risk. Runtime upgrade only. Can run in parallel with Release 1.    │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│  RELEASE 3: CakePHP 5.x (the big one)                                  │
│                                                                         │
│  Phase 2 ──► Targeted Manual QA on review ──┐                           │
│  Phase 3 ──► Targeted Manual QA on review ──┤  (review environment only)│
│  Phase 4 ──► Targeted Manual QA on review ──┘                           │
│                          │                                              │
│                          ▼                                              │
│  Phase 5 ──► FULL 26-Section Manual QA on review ──► Go/No-Go decision │
│                          │                                              │
│                          ▼                                              │
│  Phase 6 ──► Deploy to beta ──► Deploy to Production ──► Post-deploy QA│
│                                                                         │
│  High risk. Package upgrades + framework swap. Ships only after full QA.│
└─────────────────────────────────────────────────────────────────────────┘
```

**Why three releases?**
- Release 1 (Phase 0) is a safe, standalone improvement on the current 4.x line. Getting it to production early means the codebase is cleaner when the 5.x work begins.
- Release 2 (Phase 1) is a runtime change, not a code change. It can ship independently.
- Release 3 (Phases 2–6) is the actual CakePHP 5 cutover. Phases 2–4 stay on review with targeted manual QA to catch problems early. The full 26-section manual QA walkthrough in Phase 5 is the gate before this release hits production.

---

## Effort Summary

| Phase | Dev Work | Manual QA (person in browser) | Deploys To | Confidence | Phase Total |
|-------|----------|-------------------------------|------------|------------|-------------|
| | | | | | |
| **RELEASE 1** | | | | | |
| **Phase 0** — Prepare on 4.x | 20–34 hrs | 4–6 hrs | **Production** | High | **24–40 hrs** |
| | | | | | |
| **RELEASE 2** | | | | | |
| **Phase 1** — Infrastructure & environment readiness | 8–14 hrs | 1–2 hrs | **Production** | High | **9–16 hrs** |
| | | | | | |
| **RELEASE 3** | | | | | |
| **Phase 2** — Private package upgrades (`orases/*`) | 24–55 hrs | 6–10 hrs | Review only | Low | **30–65 hrs** |
| **Phase 3** — Automated 5.x upgrade (Rector + tools) | 4–8 hrs | 4–6 hrs | Review only | Medium | **8–14 hrs** |
| **Phase 4** — Manual fixes & compilation | 8–24 hrs | 4–6 hrs | Review only | Medium | **12–30 hrs** |
| **Phase 5** — Full manual QA regression | 0–16 hrs (bug fixes) | 6.5–10 days | Review only | Medium | **52–96 hrs** |
| **Phase 6** — Production deployment | 2–4 hrs | 4–6 hrs | **Production** | High | **6–10 hrs** |
| | | | | | |
| **TOTAL** | **66–155 hrs** | **75–116 hrs** | | | **~141–271 hrs** |

> Manual QA time assumes one tester clicking through the application.

### Confidence Levels

| Level | What It Means | How to Read the Estimates |
|-------|---------------|--------------------------|
| **High** | Work has been spiked, scope is a known list of files/changes, or the task is mechanical. | Estimates are reliable. Likely to land within the range. |
| **Medium** | Scope is defined on paper but the work hasn't been attempted. Tools (Rector) may not cover everything. | Estimates are reasonable but could shift up. Budget toward the high end. |
| **Low** | Significant unknowns. The work depends on third-party code nobody has touched in 5+ years, or on multi-layer dependency jumps that haven't been explored. | Estimates are best guesses. The upper bound could be exceeded. Validate with the team before committing to a timeline. |

### Why Each Phase Has Its Confidence Level

**Phase 0 — High.** The CakePHP 4.6 spike is already done (all 344 tests pass). The deprecation inventory is a concrete grep — exact file counts, exact usages. The fixes are mechanical find-replace patterns. The only area requiring judgment is ~30–50 `Entity::has()` usages in billing paths, but the code audit identified exactly which ones.

**Phase 1 — High.** PHP 8.2 via `ondrej/php` PPA is a well-documented procedure. PHPUnit 10 refactor is mechanical (362 calls already audited). The webhook replay system is optional. These are infrastructure tasks with known, bounded scope.

**Phase 2 — Low.** The `orases/users` + CakeDC/Users v9→v14 task (16–30 hrs by itself) is the biggest unknown in the entire roadmap:
- Nobody has started this work
- The packages haven't been tagged in 5+ years
- There are 3 layers of auth dependency jumps: `cakedc/users` v14, `cakedc/auth` v10, `cakephp/authentication` v3 + `cakephp/authorization` v3
- The TRD identifies 6 critical files and says RBAC is unchanged, but this hasn't been verified hands-on
- The upper bound was raised from the TRD's 45 hrs to 55 hrs, and it could still go higher
- Do we even have write access to these repos on `packages.orases.com`?

**Phase 3 — Medium.** Rector is a known tool with documented CakePHP 5 rules, and there's a reference PR (cakephp-sandbox). But this is the first time the WATM app runs on CakePHP 5.x. Rector won't catch everything — whatever it misses spills into Phase 4.

**Phase 4 — Medium.** The scope looks defined (RequestHandler, search plugin, Entity::has, skeleton files), but this is the catch-all phase. If Rector misses more transforms than expected, Phase 4 absorbs the overflow. The upper bound was raised from 16 to 24 hrs as a buffer for this.

**Phase 5 — Medium.** The QA test plan is detailed (26 sections, step-by-step), but:
- Nobody has timed a full walkthrough before — the 6.5–10 day range is the QA plan author's estimate, not a measured duration
- If the tester finds P0/P1 issues, a developer has to fix them and the tester re-tests — this cycle time (0–16 hrs of dev fixes) wasn't in the original estimate
- Does review have enough test data for billing, tiered pricing, and distributor commissions?

**Phase 6 — High.** Standard deployment. The manual QA is a targeted spot check, not a full walkthrough. Rollback plan is straightforward (revert to tagged CakePHP 4.6 release).

---

## RELEASE 1: CakePHP 4.6 + Deprecation Fixes

> Ships to production independently. Low risk — still on CakePHP 4.x.

### Phase 0: Prepare on 4.x

**Dev Estimate:** 20–34 hours | **Manual QA:** 4–6 hours | **Confidence:** High
**Goal:** Get onto CakePHP 4.6.x, eliminate all deprecation warnings, harden automated test coverage so the 5.x jump is clean.

#### 0a. Automated Test Coverage for Version Bump (2–4 hrs)

*Developer writes PHPUnit integration tests (not manual QA):*

- Add controller integration tests for key request-handling paths
- Cover controllers with complex `->is()` branching
- Verify middleware that registers custom detectors

#### 0b. Bump 4.4 → 4.5 → 4.6 (2 hrs)

- Spike already done (`noah/spike-cakephp-4.5-upgrade`) — all 344 tests pass
- Step through 4.5 then 4.6, commit after each
- Enable `E_ALL` error level on review environment after bump

#### 0c. Automated Test Coverage Hardening (12–20 hrs)

*Developer writes PHPUnit tests against 4.6 behavior before fixing deprecations:*

| Target | Count | Priority |
|--------|-------|----------|
| `Table::query()` usages | 20 in 12 files | High — bulk insert/select/update in Imports plugin |
| `Entity::has()` usages | 240 in 68 files (~30–50 risky) | High — billing, notifications, device status paths |
| `loadHelper()` | 11 in 2 files | Low |
| `loadModel()` | 5 in 3 files | Low |

#### 0d. Fix Deprecations (4–8 hrs)

- Swap `Table::query()` → `selectQuery()`/`insertQuery()`/`updateQuery()`
- Address risky `Entity::has()` usages (use `hasValue()` or null checks)
- Replace `loadHelper()` → `addHelper()`, `loadModel()` → `fetchTable()`
- Upgrade PHPStan from ^0.12 to ^1.x

### Phase 0 — Manual QA by Tester (4–6 hrs)

Deploy 4.6 + deprecation fixes to **review environment**. A QA tester opens the application in a browser and walks through these sections of the QA Test Plan:

| QA Test Plan Section | What the Tester Does | Priority | Est. Time |
|---------------------|----------------------|----------|-----------|
| **Section 1** — Login & Navigation | Log in, click through every menu item, confirm no broken pages or 500 errors | P0 | 0.5–1 hr |
| **Section 5.1** — Invoice Generation | Navigate to Billing, generate invoices, open them, compare totals to pre-upgrade values | P0 | 1–1.5 hrs |
| **Section 7.1** — Billing Cycle Close | Close a billing cycle, verify status transitions (credits, payouts, delayed billing counters) | P0 | 0.5–1 hr |
| **Section 2.1–2.4** — Device CRUD + Status | Browse device list, edit a device, change status to Inactive then back to Active, check status log timestamps | P0 | 1–1.5 hrs |
| **Section 17.1–17.2** — Data Usage | Open a device detail page, check usage numbers, click Daily Usage tab, verify chart renders | P1 | 0.5–1 hr |

**Why not the full 26-section walkthrough?** Phase 0 changes are limited to deprecation swaps (`query()` → `selectQuery()`, `has()` fixes). The risk is concentrated in billing math and device status paths that use those APIs. Everything else is unchanged code.

**Go/No-Go for Production:** Tester confirms invoice totals match pre-upgrade. Device status changes show correct timestamps. No 500 errors on any page.

### Phase 0 — Deploy to Production

After manual QA passes on review:
1. Deploy to beta (optional — client UAT if desired)
2. Deploy to production
3. Monitor for 24 hours — error logs, queue workers, device check-ins

**Rollback:** Revert to previous 4.4 release. No database migration concerns — deprecation fixes are code-only changes.

---

## RELEASE 2: PHP 8.2 + Test Infrastructure

> Ships to production independently. Can run in parallel with Release 1. Low risk — runtime upgrade only.

### Phase 1: Infrastructure & Environment Readiness

**Dev Estimate:** 8–14 hours | **Manual QA:** 1–2 hours | **Confidence:** High
**Goal:** Close test infrastructure gaps and upgrade PHP.

#### 1a. PHP 8.1 → 8.2 Upgrade (2–4 hrs)

- Ubuntu 22.04 needs `ondrej/php` PPA (or upgrade to 24.04)
- Update all environments: review → beta → production
- Required for CakePHP 5.3+

#### 1b. PHPUnit 8/9 → 10 (2–4 hrs)

*Developer refactors automated tests (not manual QA):*

- 362 assertion calls in 21 files — mechanical find-replace
- Already compliant: setUp/tearDown, data providers, no prophecy
- Update `phpunit.xml.dist` schema

#### 1c. Deprecation Visibility (1–2 hrs)

- Enable `E_ALL` on review environment (currently suppressed everywhere)
- Confirm deprecation warnings are visible and actionable

#### 1d. Webhook Replay System (3–4 hrs, optional but recommended)

- Build payload capture middleware for production webhooks
- Create CLI replay command for testing on review/beta
- Curate golden payloads (Verizon callbacks, Adobe Sign, usage data)

### Phase 1 — Manual QA by Tester (1–2 hrs)

After PHP 8.2 is deployed to **review environment**, a QA tester verifies the app still works:

| QA Test Plan Section | What the Tester Does | Priority | Est. Time |
|---------------------|----------------------|----------|-----------|
| **Section 1** — Login & Navigation | Log in, click through every page, confirm the app works on PHP 8.2 | P0 | 0.5–1 hr |
| **Section 23** — Background Jobs | Check a few device detail pages — are "last check-in" timestamps recent? Is usage data updating? | P1 | 0.5–1 hr |

**Why only these two?** PHP 8.2 is a runtime upgrade, not a code change. If the app loads and background jobs are processing, PHP is fine. The PHPUnit 10 refactor is developer-only — no production impact.

**Go/No-Go for Production:** Tester confirms app loads on PHP 8.2. Background job output (check-ins, usage) is still current.

### Phase 1 — Deploy to Production

After manual QA passes on review:
1. Deploy PHP 8.2 to beta first, monitor
2. Deploy PHP 8.2 to production
3. Monitor for 24 hours — queue workers, check-in processing

**Rollback:** Revert PHP to 8.1 via package manager. PHPUnit 10 changes are dev-only, no production rollback needed.

---

## RELEASE 3: CakePHP 5.x Upgrade

> This is the big one. Phases 2–4 stay on the review environment with targeted manual QA after each phase. The full 26-section manual QA walkthrough in Phase 5 is the gate before production. Nothing from Phases 2–4 touches production until Phase 5 passes.

### Phase 2: Private Package Upgrades (`orases/*`)

**Dev Estimate:** 24–55 hours | **Manual QA:** 6–10 hours | **Confidence:** Low
**Deployed to:** Review environment only
**Goal:** Get all private packages CakePHP 5 compatible. This is the critical path.

#### 2a. Ready Packages — Bump Versions (1–2 hrs)

- `orases/files` → ^3.0 (CakePHP 5 ready)
- `orases/helpers` → ^2.0 (CakePHP 5 ready — verify APIs unchanged)
- `orases/theme-limitless` → ^2.0 (CakePHP 5 ready — test with BackendTheme)

#### 2b. Drop or Inline `orases/imports` (1 hr)

- Only 3 files, 1 legacy command (`ImportOldDataCommand`)
- Inline into WATM codebase and remove dependency

#### 2c. Fork/Upgrade `orases/logs` (2–4 hrs)

- 7 files: LogsTable, Log entity, LogsController, LogEventTrait, IndexFilterForm
- Fix deprecated APIs, bump CakePHP dependency
- WATM's Logging plugin extends this — test integration

#### 2d. Fork/Upgrade `orases/users` + CakeDC/Users v9 → v14 (16–30 hrs)

> **Single biggest risk — ~25% of total effort**

**Critical files (6):**
1. `UserBehaviorsController` — login, registration, password reset, 2FA
2. `UserConfigurationMiddleware` — auth config
3. `AuthorizationServiceLoader` — RBAC policy chain
4. `OUsersListener` — login event handler
5. `DefaultOneTimePasswordAuthenticationChecker` — 2FA
6. `OUsersController` — user CRUD

**Integration surface:**
- 30+ controllers with `IsAuthorizedTrait` (likely just type declaration updates)
- 13+ tables with `AuthorBehavior` (unchanged API)
- 4 custom RBAC rule classes extending `OrasesRules`

**What's unchanged (good news):** RBAC format, event constants, user entity structure, AuthorBehavior API.

#### 2e. Fork/Upgrade `orases/sites` (4–8 hrs)

- Blocked until `orases/users` is done (depends on it)
- SitesMiddleware, OSitesTable, SubdomainTrait, MultiSiteBehavior
- 6 direct imports in WATM

```
Dependency chain (must be sequential):

orases/logs ──► orases/users ──► orases/sites
                    │
                    ▼
              cakedc/users v9 → v14
```

### Phase 2 — Manual QA by Tester on Review (6–10 hrs)

After all package upgrades are deployed to **review environment**, a QA tester walks through the features that depend on the upgraded packages:

| QA Test Plan Section | What the Tester Does | Package Being Verified | Est. Time |
|---------------------|----------------------|----------------------|-----------|
| **Section 1** — Login & Navigation | Log in with admin credentials, log out, confirm redirect to login page, try accessing a protected page while logged out | `orases/users` + CakeDC auth | 0.5–1 hr |
| **Section 1** (password reset) | Click "Forgot Password", walk through the reset flow, confirm new password works | `orases/users` | 0.5–1 hr |
| **Section 22.1** — Role-Based Access | Log in as system admin (full menu), log in as company admin (only their data), try URL tampering to access another company | `orases/users` RBAC | 1–1.5 hrs |
| **Section 12.1–12.4** — Company Management | Browse company list, open a company, edit a field, check parent-child relationships display | `orases/sites` multi-tenancy | 1–1.5 hrs |
| **Section 20** — Admin Logging | Navigate to Admin Logs, confirm entries show who/what/when with correct timestamps, try filtering | `orases/logs` | 0.5–1 hr |
| **Section 15.1–15.4** — API Key Management | Enable API access for a company, generate a key, make an API call with it, revoke the key, confirm it's rejected | `orases/users` auth + JSON | 1–1.5 hrs |
| **Section 3.1** — Device Import | Upload a small CSV file, confirm import preview, complete import, verify devices appear in list | `orases/imports` (or inlined version) | 0.5–1 hr |
| **Section 24** — File Operations | Upload a file, download an existing file (invoice PDF, device export) | `orases/files` | 0.5–1 hr |

**Why these sections?** Each row tests a specific upgraded package. No need to test billing or device status — that code wasn't touched in this phase.

**Phase 2 gate (review only — not production):** Tester confirms login works, RBAC blocks unauthorized access, company admin can't see other companies, API keys work, audit logs have correct timestamps, file upload/download works. If these pass, proceed to Phase 3 on review.

---

### Phase 3: Automated 5.x Upgrade

**Dev Estimate:** 4–8 hours | **Manual QA:** 4–6 hours | **Confidence:** Medium
**Deployed to:** Review environment only
**Goal:** Run automated tools to handle the bulk of signature/type/rename changes.

#### 3a. Rector (cakephp50 rules)

- Run `bin/cake upgrade rector --rules cakephp50` on `src/` and each plugin
- Handles: `FrozenTime` → `DateTime`, union types on signatures, renamed classes
- Commit after each plugin

#### 3b. dereuromark's Upgrade Tool

- Templates, config files, non-PHP file transforms
- Commit separately from Rector changes

#### 3c. Composer Updates

- Update `composer.json` with all CakePHP 5 version constraints
- Run `composer update -W`
- Add `require CAKE . 'functions.php'` to `bootstrap.php`
- Update `requirements.php` PHP version check (7.2.0 → 8.2.0)

### Phase 3 — Manual QA by Tester on Review (4–6 hrs)

Deploy to **review environment** after Rector + composer update. **This is the first time the app runs on CakePHP 5.x** — expect some breakage. A QA tester walks through the highest-risk areas:

| QA Test Plan Section | What the Tester Does | Why This Is High Risk | Est. Time |
|---------------------|----------------------|-----------------------|-----------|
| **Section 1** — Login & Navigation (FULL sweep) | Log in, click **every single menu item**, check that each page loads with correct layout and data | Rector changes how every page is routed and rendered | 1–1.5 hrs |
| **Section 2.1** — Device List | Open device list, click column headers to sort, use search/filter, click through pagination pages | CakePHP 5 completely replaces the pagination system | 0.5–1 hr |
| **Section 12.1** — Company List | Same as above — sort, filter, paginate on the company list | Second confirmation that list/pagination behavior works | 0.5 hr |
| **Section 5.1** — Invoice Generation | Generate an invoice, open it, check billing period dates and line item totals, compare to pre-upgrade values | `FrozenTime` → `DateTime` changes all date math | 1–1.5 hrs |
| **Section 15.3** — API Test | Make an API call with a valid key, confirm JSON response comes back with correct data | Content negotiation (JSON rendering) is rebuilt in CakePHP 5 | 0.5–1 hr |
| **Section 18.1** — Dashboard | Confirm dashboard loads, device counts display, charts render | Dashboard uses date-based aggregation queries | 0.5 hr |

**Why these sections?** Rector changes three things everywhere: (1) date/time classes, (2) method signatures, (3) renamed APIs. These manual checks hit the highest-risk surfaces: pages with lists and pagination, financial calculations involving dates, and JSON API responses.

**Phase 3 gate (review only — not production):** Tester confirms all pages load, pagination works on list pages, invoice totals are correct, API returns valid JSON, dashboard renders. If these pass, proceed to Phase 4 on review.

---

### Phase 4: Manual Fixes & Compilation

**Dev Estimate:** 8–24 hours | **Manual QA:** 4–6 hours | **Confidence:** Medium
**Deployed to:** Review environment only
**Goal:** Fix everything the automated tools couldn't handle.

#### Key areas:

| Area | Scope | Action |
|------|-------|--------|
| `RequestHandlerComponent` | 2 files (AppController, ErrorController) | Remove, use `viewClasses()` |
| `friendsofcake/search` | 7 finder calls, 10 tables | Named-arg syntax change |
| `Security` → `FormProtection` | 1 file | Rename component |
| `Entity::has()` remaining | ~30–50 call sites | Replace with `hasValue()` or null checks |
| Skeleton files | bootstrap.php, requirements.php, phpunit.xml.dist | Update to 5.x format |
| PHPStan | Run full analysis | Fix new type errors |
| Remaining compiler errors | Variable | Fix until clean build |

### Phase 4 — Manual QA by Tester on Review (4–6 hrs)

Deploy fixes to **review environment**. A QA tester walks through features that map directly to what the developer just fixed:

| QA Test Plan Section | What the Tester Does | Which Dev Fix This Verifies | Est. Time |
|---------------------|----------------------|-----------------------------|-----------|
| **Section 1** — Login & Navigation | Click through every page again — confirm the fixes didn't break other pages | Baseline sanity check | 0.5–1 hr |
| **Section 2.1** — Device Search/Filter | Search for a device by name, apply status filter, apply company filter | `friendsofcake/search` syntax change | 0.5 hr |
| **Section 12.1** — Company Search/Filter | Search for a company, apply parent company filter | `friendsofcake/search` syntax change | 0.5 hr |
| **Section 2.4** — Device Status Changes | Change a device to Inactive, check status log timestamp, reactivate, check log again | `Entity::has()` → `hasValue()` in device status paths | 0.5–1 hr |
| **Section 5.1–5.5** — Billing (spot check) | Generate an invoice, check tiered pricing, dual SIM upcharge, distributor/subcompany billing, compare totals | `Entity::has()` → `hasValue()` in billing paths | 1–2 hrs |
| **Section 6.1–6.2** — NACHA & Exclusions | Generate NACHA file, click "Exclude from NACHA" button on an invoice, confirm AJAX updates without page reload, confirm filter stays active | `RequestHandlerComponent` removal (AJAX/JSON) | 0.5–1 hr |
| **Section 15.3** — API Test | Make an API call, confirm JSON response | `RequestHandlerComponent` removal (API JSON) | 0.5 hr |

**Why these sections?** Each row is directly tied to a specific fix: search plugin changes → test search/filter in browser, `Entity::has()` fixes → test billing and device status on screen, `RequestHandlerComponent` removal → test AJAX buttons and API responses.

**Phase 4 gate (review only — not production):** Tester confirms search/filter works on all list pages, invoice math is correct, NACHA exclude/include AJAX buttons work without page reload, API returns JSON. If these pass, proceed to full QA in Phase 5.

---

### Phase 5: Full Manual QA Regression

**Dev Estimate:** 0–16 hours (bug fixes found during QA) | **Manual QA:** 6.5–10 days (one tester, full-time) | **Confidence:** Medium
**Deployed to:** Review environment only
**Goal:** One person walks through the entire 26-section QA Test Plan. This is the production gate for the CakePHP 5 upgrade.

> **This is the gate before CakePHP 5 goes to production.** Phases 2–4 used targeted manual QA on review — a tester checking only the features affected by each phase's changes. Phase 5 is the comprehensive walkthrough that catches anything the targeted checks missed.

#### 5a. Automated Tests First (developer, 0.5 day)

Before manual QA begins, a developer runs:
- Full PHPUnit suite (344+ tests)
- PHPStan static analysis
- `composer cs-check` for code style

Any failures here get fixed before handing off to the QA tester.

#### 5b. Full Manual QA Walkthrough

A QA tester walks through **all 26 sections** of the [QA Test Plan](https://orases.atlassian.net/wiki/spaces/WATM/pages/2964160521/WATM+CakePHP+4+to+5+Upgrade+QA+Test+Plan) on the review environment, in priority order:

| QA Round | QA Test Plan Sections | What the Tester Is Clicking Through | Priority | Est. Time |
|----------|----------------------|-------------------------------------|----------|-----------|
| **Smoke** | Section 1 | Log in, navigate every menu item, log out, confirm redirect | P0 | 0.5 day |
| **Core Workflows** | Sections 2–9 | Device CRUD, status changes, imports, bulk ops, invoice generation, tiered/dual SIM/custom/distributor pricing, NACHA files, billing cycle close, commissions, cost breakdowns | P0 | 3–4 days |
| **Secondary Workflows** | Sections 10–21 | Device config assignment, RMA, company management, payment methods, service plans, API keys, tax documents, data usage charts, dashboard, check-in failures, admin logs, notifications | P1 | 2–3 days |
| **Supporting** | Sections 22–26 | Role-based access with different user types, background job output checks, file uploads/downloads, Swagger API docs, carrier test tools | P2 | 1 day |

#### 5c. Critical Regression Check

> **If the tester can only do one thing:** Generate a full billing cycle's invoices on the upgraded system and compare the totals line-by-line against the same cycle's invoices from the current (pre-upgrade) system. If the numbers match, the three biggest risk areas (date calculations, database queries, and billing logic) are all working correctly.

#### Pass/Fail Criteria

**Go for Production:**
- All P0 items pass (tester can log in, manage devices, generate correct invoices, close a billing cycle)
- No P1 items have financial-impact or data-corruption failures
- Invoice totals match pre-upgrade values for the same billing cycle

**Hold for Fixes (go back to Phase 4, fix, re-test):**
- Any P0 failure
- Any P1 failure affecting billing accuracy, commission calculations, or device status integrity
- Any P1 failure where dates/times are incorrect (billing cycles, status logs, usage windows)
- Background carrier usage jobs not collecting data

**Known Issues Acceptable (can ship with these):**
- P2 failures (user management cosmetics, API docs formatting, test suite tools)
- Cosmetic P1 issues (styling, layout that doesn't affect data)
- Minor pagination or sorting glitches that don't affect data accuracy

---

### Phase 6: Production Deployment

**Dev Estimate:** 2–4 hours | **Manual QA:** 4–6 hours | **Confidence:** High
**Goal:** Deploy CakePHP 5 to production with rollback plan.

#### Deployment Steps

1. **Deploy to beta** — client UAT if applicable
2. **Deploy to production** — during low-traffic window
3. **Monitor** — error logs, queue processing, device check-ins, carrier API responses

#### Phase 6 — Post-Deploy Manual QA by Tester (4–6 hrs)

A QA tester walks through these checks on **production** immediately after deploy:

| QA Test Plan Section | What the Tester Does on Production | Est. Time |
|---------------------|-----------------------------------|-----------|
| **Section 1** — Login & Navigation | Log in, click through key pages, confirm app works on production infrastructure | 0.5–1 hr |
| **Section 2.1–2.4** — Device CRUD + Status | Browse device list, edit a device, change status (this triggers real carrier API calls on production) | 1 hr |
| **Section 5.1** — Invoice Generation | Generate one invoice, compare totals to pre-upgrade invoice for same cycle | 1–1.5 hrs |
| **Section 23** — Background Jobs | Check device detail pages for recent "last check-in" timestamps, verify usage data is updating, confirm queues are draining | 1–1.5 hrs |
| **Section 6.1** — NACHA (visual check only) | Confirm NACHA page loads and lists invoices — do NOT generate unless needed | 0.5 hr |

#### 24–48 Hour Monitoring Period (no tester needed — dev/ops watches dashboards)

- Error logs — watch for new exceptions
- Queue worker health — check-in processing, usage collection, notification delivery
- Device check-in flow — UDP → SQS → worker → database
- Carrier API responses — Verizon/AT&T/T-Mobile webhooks processing

#### Rollback Plan

- Keep previous release (CakePHP 4.6 on PHP 8.2) tagged and deployable
- Database migrations must be backward-compatible (or have down migrations)
- Rollback trigger: any P0 failure found in post-deploy manual QA

---

## Manual QA Decision Matrix — Quick Reference

| Phase | Release | Where It Deploys | Which QA Test Plan Sections the Tester Walks Through | Full 26-Section Walkthrough? |
|-------|---------|------------------|------------------------------------------------------|------------------------------|
| **Phase 0** | Release 1 | Review → **Production** | Sections 1, 2.1–2.4, 5.1, 7.1, 17.1–17.2 | **No** — billing + device pages only |
| **Phase 1** | Release 2 | Review → **Production** | Sections 1, 23 | **No** — login + background jobs only |
| **Phase 2** | Release 3 | Review only | Sections 1, 3.1, 12.1–12.4, 15.1–15.4, 20, 22.1, 24 | **No** — one section per upgraded package |
| **Phase 3** | Release 3 | Review only | Sections 1, 2.1, 5.1, 12.1, 15.3, 18.1 | **No** — pagination + dates + API |
| **Phase 4** | Release 3 | Review only | Sections 1, 2.1, 2.4, 5.1–5.5, 6.1–6.2, 12.1, 15.3 | **No** — one section per fix |
| **Phase 5** | Release 3 | Review only | **All 26 sections (1–26)** | **YES — production gate** |
| **Phase 6** | Release 3 | Beta → **Production** | Sections 1, 2.1–2.4, 5.1, 6.1, 23 | **No** — post-deploy spot checks |

---

## Risk Register

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| `orases/users` + CakeDC v14 breaks auth | **High** | Medium | Concentrated in 6 files; RBAC format unchanged; tester verifies auth + RBAC in Phase 2 manual QA on review |
| `Entity::has()` behavior change causes data bugs | **High** | Medium | Audit all 240 usages; tester verifies billing + device status in Phase 0 (production) and Phase 4 (review) manual QA |
| Invoice totals don't match pre-upgrade | **High** | Medium | Tester compares invoice totals in Phase 0 (production), Phase 3 (review), Phase 5 (review), and Phase 6 (production) |
| Shared carrier API accounts — no test isolation | **Medium** | Low | Webhook replay system (Phase 1); careful manual QA on review environment |
| PHP 8.2 upgrade breaks production | **Medium** | Low | Tester verifies on review first; deploy to beta before production in Phase 1 |
| Hidden deprecation issues (warnings suppressed) | **Medium** | Medium | Enable `E_ALL` on review in Phase 1 |
| Pagination completely broken after Rector | **Medium** | Medium | Tester clicks through every list page in Phase 3 manual QA on review |
| Third-party package incompatibility | **Low** | Low | All confirmed compatible per dependency audit |

---

## Key Decisions & Notes

- **Three independent production releases** — Phase 0 and Phase 1 ship to production early; CakePHP 5 (Phases 2–6) ships only after full manual QA
- **No intermediate CakeDC/Users versions needed** — go v9 → v14 directly
- **Queue system requires zero code changes** — identical API across versions
- **Templates (289 files) are already CakePHP 5 compliant** — minimal template work
- **All 5 custom middleware classes are already PSR-15 compliant** — no changes needed
- **`orases/imports` should be inlined and dropped** — only 3 files, not worth maintaining
- **Spike branch:** `noah/spike-cakephp-4.5-upgrade` confirms 4.6 path is clean
- **Full 26-section manual QA walkthrough runs once (Phase 5)** — targeted manual QA in all other phases keeps the feedback loop fast while still catching phase-specific regressions

---

## References

- [CakePHP 5.x Upgrade - TRD](https://orases.atlassian.net/wiki/spaces/WATM/pages/2968289286/CakePHP+5.x+Upgrade+-+TRD) (parent page with child pages for Phase 0, Test Infrastructure, Dependency Audit, Code Audit)
- [WATM CakePHP 4 to 5 Upgrade — QA Test Plan](https://orases.atlassian.net/wiki/spaces/WATM/pages/2964160521/WATM+CakePHP+4+to+5+Upgrade+QA+Test+Plan)
- [CakePHP 5.0 Upgrade Guide - Official](https://book.cakephp.org/5/en/appendices/5-0-upgrade-guide.html)
- [CakePHP 5 Upgrade Guide - DerEuroMark](https://www.dereuromark.de/2023/09/28/cakephp-5-upgrade-guide/)
- [CakePHP End of Life dates](https://endoflife.date/cakephp)
