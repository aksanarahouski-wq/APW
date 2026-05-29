# WATM — CakePHP & Infrastructure Upgrade Plan

**Date:** April 7, 2026
**Prepared by:** Orases Team

---

## Summary

The WATM platform currently runs on CakePHP 4.4 and PHP 8.1. Both are approaching or have passed end-of-life:

- **CakePHP 4.x** — Bugfix support has ended. Security patches expire later this year.
- **PHP 8.1** — Reached end-of-life in November 2025.

We have completed a thorough technical audit, built a detailed roadmap, and are ready to begin. The work is structured into **five releases**, designed so that upgrade work can happen incrementally alongside normal feature development without long blocks.

We recommend starting **Release 1 and Release 2 now**, running them in parallel. The remaining releases (3a, 3b, 3c) can be scheduled throughout the year as capacity allows.

---

## Release Plan

### Release 1 — CakePHP 4.6 Upgrade + Deprecation Cleanup

**What it is:** Upgrade the framework from 4.4 to 4.6 and resolve all deprecated code patterns. This is a code-level upgrade that keeps us on the stable CakePHP 4.x line while cleaning up the codebase for the eventual major upgrade.

**Risk level:** Low — a developer spike has already confirmed the upgrade works (all 344 automated tests pass on 4.6).

**Estimated effort:** 24–40 hours (development + targeted QA)

**What you get:** A cleaner, more maintainable codebase on the latest supported 4.x version, with hardened test coverage around billing and device management.

---

### Release 2 — PHP 8.2 + Server Infrastructure Upgrade

**What it is:** Upgrade the server runtime from PHP 8.1 to 8.2 and modernize the test infrastructure. This involves provisioning new AWS servers with the latest Ubuntu (which ships with PHP 8.2 natively) rather than patching existing servers.

**Risk level:** Low — this is a runtime/infrastructure change, not an application code change. We validate on the review environment first. If anything goes wrong, we revert without impacting production.

**Estimated effort:** 9–16 hours (development + targeted QA)

**What you get:** Supported PHP version, improved security posture, and servers on a current OS with a repeatable provisioning process.

---

### Release 3a — Private Package Upgrades (Prep for CakePHP 5)

**What it is:** Upgrade the 7 proprietary Orases packages (authentication, multi-tenancy, file storage, logging, etc.) to be CakePHP 5 compatible. This is the heaviest preparation step and the single biggest technical risk in the entire upgrade — particularly the authentication/authorization package which accounts for ~25% of the total effort.

**Risk level:** Medium-High — the auth packages haven't been updated in 5+ years and involve a multi-layer dependency jump.

**Estimated effort:** 30–65 hours (development + targeted QA on review environment)

**What you get:** All foundational packages ready for CakePHP 5. This work stays on the review environment only — production continues running normally. Regular feature development is not blocked.

**Depends on:** Release 1 and Release 2 complete.

---

### Release 3b — CakePHP 5 Code Migration

**What it is:** Run automated upgrade tools across the entire codebase, then manually fix everything the tools miss. This is where the application actually switches to CakePHP 5 for the first time. Includes pagination changes, date/time class updates, method signature updates, and search plugin syntax changes.

**Risk level:** Medium — automated tools handle the bulk of the work, but there will be manual fixes needed.

**Estimated effort:** 20–44 hours (development + targeted QA on review environment)

**What you get:** The WATM application running on CakePHP 5 on the review environment, validated with targeted QA. Production is still unaffected. Regular feature development continues.

**Depends on:** Release 3a complete.

---

### Release 3c — Full QA Regression + Production Deployment

**What it is:** A comprehensive 26-section manual QA regression covering every major feature of the platform (login, devices, billing, invoicing, NACHA, commissions, company management, API keys, and more). Once QA passes, deploy to beta and then production with a 24–48 hour monitoring period.

**Risk level:** Low — by this point the code is stable on review. This is validation and rollout.

**Estimated effort:** 58–106 hours (bug fixes from QA + full QA regression + production deployment)

**What you get:** CakePHP 5 running in production. Modern, supported framework with active security patches and access to the latest ecosystem packages.

**Depends on:** Release 3b complete.

---

## Recommended Approach

We recommend **starting Release 1 and Release 2 immediately, running in parallel**. The Release 3 work (3a, 3b, 3c) is broken into smaller chunks that can be scheduled throughout the year alongside normal feature development.

This incremental approach:

1. **Eliminates the immediate security and support risk** — gets us off end-of-life PHP and onto the latest stable CakePHP 4.x
2. **Does not block feature development** — upgrade work happens on the review environment while production stays stable
3. **Reduces risk** — each chunk is small enough to validate independently before moving to the next
4. **Provides flexibility** — Release 3a/3b/3c can be scheduled around feature priorities and capacity
5. **Delivers value early** — Releases 1 and 2 ship to production quickly with immediate security benefits

### High-Level Estimates

| Release | Scope | Estimated Effort | Risk | Deploys To |
|---------|-------|-----------------|------|------------|
| **Release 1** | CakePHP 4.6 + deprecation fixes | 24–40 hrs | Low | Production |
| **Release 2** | PHP 8.2 + server upgrade | 9–16 hrs | Low | Production |
| **Release 3a** | Private package upgrades | 30–65 hrs | Medium-High | Review only |
| **Release 3b** | CakePHP 5 code migration | 20–44 hrs | Medium | Review only |
| **Release 3c** | Full QA + production deploy | 58–106 hrs | Low | Production |

**Release 1 + Release 2 (start now): ~33–56 hours**
**Release 3 total (schedule over the year): ~108–215 hours**
**Grand total: ~141–271 hours**

---

## What We've Done So Far

- Completed a full technical audit of the codebase, dependencies, and all 17 plugins
- Built a detailed QA test plan (26 sections covering every major feature)
- Created a phased roadmap with confidence levels and risk mitigations
- Validated the CakePHP 4.6 upgrade path with a working developer spike
- Documented the full strategy and sequencing decisions

All of this planning and documentation is available in our shared Confluence space for full transparency.

---

## Next Steps

With your approval, we will:

1. Begin Release 1 (CakePHP 4.6 upgrade) and Release 2 (PHP 8.2 + server upgrade) in parallel
2. Deploy each to the review environment for QA validation before touching production
3. Ship each release to production independently as it passes QA
4. Schedule Release 3a alongside regular development — it runs on review and does not impact production
5. Continue through 3b and 3c at a pace that works with feature priorities

---

*Questions? We're happy to walk through the roadmap and technical details in more depth.*
