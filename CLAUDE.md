# CLAUDE.md — WATM Project Root

## Doc Index

| Folder | What's here |
|--------|------------|
| `watm/` | Source code — CakePHP app (`watm/`), Node.js check-ins (`checkins/`), Playwright E2E tests, DDEV config. Has its own CLAUDE.md with architecture, commands, and coding standards. |
| `Product Features/` | Feature work — PRDs, research, implementation docs. Active features at top level, shipped features in `COMPLETE/`. |
| `Meetings/` | All meeting transcripts and summaries. `Transcripts/` for raw recordings, `Summaries/` for condensed notes, action items, and preps. Covers client, partner, and feature-specific meetings. |
| `Sprint Status Updates/` | Biweekly sprint board reports — dated snapshots generated via `/sprint-report`. |
| `TrelloBacklog/` | Jira/Trello backlog tickets — research folders per ticket, weekly snapshot reports. |
| `Documentation/` | Standalone reference docs (roles & permissions, system docs). |
| `SQL/` | Ad-hoc SQL scripts and queries. |

## Project Overview

WATM (Wireless Access and Telemetry Management) — IoT device management and billing platform. CakePHP 4.x multi-tenant web app + Node.js UDP check-ins service. Manages devices across carriers (Verizon, AT&T, T-Mobile), tracks usage, generates invoices, processes ACH payments, white-label multi-tenant.

**Client:** APW (American Power and Water) — primary stakeholder for all feature work.

## Team

### Orases (Internal)
- **Aksana Rahouski** — Senior Product Manager / BA. Leads client relationship, PRDs, backlog, roadmap.
- **Laura Perry** — Project Manager. Scheduling, budget/scorecard tracking, Trello board, sponsor updates.
- **Richard Sacco** — Developer. Senior dev on WATM since inception (~7 yrs). Device check-ins, router configs, portal.
- **Aaron Diefes** — Developer. T-Mobile/dual-SIM, executive summaries, QA/release coordination.
- **Noah** — Developer. 5G/Verizon API integration, T-Mobile bug investigations.
- **Stone Marballie** — Developer. (stone.marballie@orases.com)

### APW / Allpoint Wireless (Client)
- **Devon D'Andrea** — Operations exec. Day-to-day ops, device management, portal usage, backlog prioritization.
- **Adam Curcie** — Technical lead / exec. Carrier integrations, service plans, billing logic, budget approval.
- **Rick** — President & CEO. Business direction, investment decisions. Devon and Adam report to him.
- **Vince** — Billing / Accounting. Processes ~200 credit card transactions/month in QuickBooks.
- **John** — Operations / Support. Creates tickets, provides device examples, some Confluence docs.
- **Dan** — Operations. SIM card provisioning and activation.

### InHand Networks (Hardware Partner)
- **Kenneth Hunter** — VP of Sales, Americas. Business relationship, ~12 yrs at InHand.
- **Zeming Wang** — Head Engineer, US. Translates requirements to engineering specs. Goes by "Ziming" / "Jim."

## Key Tools & Integrations

- **Jira:** `WATM` project — [orases.atlassian.net](https://orases.atlassian.net) | [Sprint Board](https://orases.atlassian.net/jira/software/c/projects/WATM/boards/89) | [Kanban Board](https://orases.atlassian.net/jira/software/c/projects/WATM/boards/198)
- **Confluence:** [WATM space](https://orases.atlassian.net/wiki/spaces/WATM/overview) for technical docs and meeting notes
- **Harvest:** Time tracking for WATM project
- **Slack:** Project communication — Channel ID `C03BWTE5SJD`

## Skills & Commands

- `/sprint-report` — Query Jira sprint board, generate status update in `Sprint Status Updates/`
- `/trello-report` — Pull Draft/Trello tickets from Jira, generate dated snapshot in `TrelloBacklog/`
- `/prd-builder` — Generate PRDs from requirements
- `/analyze-workflow` — Review Claude Code session efficiency

## Current Date

Today's date is 2026-04-27.
