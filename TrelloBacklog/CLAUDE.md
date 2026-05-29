# TrelloBacklog

Jira tickets originally from Trello, in Draft status, pending client review and prioritization.

## Structure

- **`WATM-XXXX/`** folders — per-ticket research and analysis (e.g., `WATM-1501/Analysis.md`)
- **`DeviceAlerts/`**, **`Permissions-Revisions/`**, **`Device New Status/`** — topic-based research for related tickets
- **`Meetings/`** — backlog grooming and review meeting notes
- **`Draft_Trello_Tickets_Report_YYYY-MM-DD.md`** — weekly snapshot reports pulled via `/trello-report`

## Report Generation

Run `/trello-report` to pull current Draft/Trello tickets from Jira. Each run creates a new dated file as a weekly snapshot. Do not overwrite previous reports.

## Current Ticket Count

18 Draft tickets as of 2026-04-03. See latest report for full list.
