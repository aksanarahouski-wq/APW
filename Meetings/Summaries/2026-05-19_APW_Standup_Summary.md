# APW Stand-up — Meeting Summary

**Date:** May 19, 2026, 3:00 PM UTC
**Duration:** ~47 minutes
**Organizer:** Laura Perry
**Attendees:** Aksana Rahouski, Laura Perry, Noah Bratzel, Richard Sacco, Stone Marballie, Aaron Diefes (Orases)

---

## Overview

Internal stand-up covering Verizon FWA service plan type changes, MySQL upgrade planning, blocked client bugs requiring follow-up, CakePHP branch management, ticket estimation process discussion, and scheduling adjustments for Memorial Day week.

---

## Key Discussion Topics

### Verizon FWA Service Plan Type Restrictions
- Service plans currently allow three states: no restriction, PPU, or FWA. Team agreed it should be only two: **PPU or FWA** (no "no restriction" option)
- All existing Verizon service plans on beta were temporarily set to PPU by Noah as a workaround
- Client was testing PPU-to-FWA assignment errors but wasn't seeing expected error messages because their service plans were set to null (no restriction) rather than PPU
- The PPU/FWA restriction is **carrier agnostic** -- set on the device and service plan, not tied to carrier. The UI groups them under carrier for client convenience only
- AT&T devices would always be PPU-only since AT&T has no business/FWA devices
- T-Mobile FWA work (coming later) will follow the same pattern
- A migration script would handle the production cutover, setting all existing plans to PPU by default
- Aksana to draft a comment to the Trello ticket (WATM-1978) explaining the change and validating the assumption with the client before Noah proceeds

### MySQL Upgrade Planning
- **Deadline:** July 31st -- takes priority over PHP/CakePHP upgrade
- Richard plans to upgrade the review environment first (simpler since it's not on RDS), then beta, then prep production
- **Reserved keyword scan** already completed with no issues found in the codebase; Richard wants to run one more thorough scan with Claude
- **Database users** will be recreated with the new password hashing strategy rather than migrated
- **Smoke testing:** Richard wants to use the Claude-based automated click-through testing (previously done by Laura/Aksana for CakePHP) to validate the MySQL upgrade
- **Production cutover approach:** Richard proposed taking an RDS snapshot, launching it on MySQL 8.4, then doing a differential sync after stopping services during the downtime window
- **Table compacting** (from the data purging project) should be included during the downtime window
- Richard to review the upgrade plan with Swiz before implementation
- Richard to share the finalized plan with the full team for visibility and learning

### CakePHP Upgrade Branch Management
- The **review branch is frozen** and reserved exclusively for the CakePHP upgrade
- New features and bug fixes should go into **feature branches merged to the integration branch**, not review
- This keeps the CakePHP upgrade isolated so issues can be tracked to their source

### Blocked Client Bugs (3 tickets)
- **Device export showing $0 price:** Normal behavior until the billing cycle closes and new values populate. Some outlier devices with unusual pricing may have been intentionally configured by the client. Needs more communication
- **Deactivated device marked as suspended:** One device was found as suspended rather than deactivated. Client hasn't responded; needs clarification since they don't want deactivated devices charged in the next cycle
- **Device check-in time accuracy:** Devices can send inaccurate check-in times. Client hasn't responded about whether they want action taken
- Team agreed to hold these for the client meeting next week, but Richard should prepare recommendations (do something vs. do nothing) rather than just waiting

### Ticket Estimation Process Discussion
- Team debated individual vs. group estimation approaches
- **Richard's position:** Client doesn't hold team accountable to estimates, so group estimation is overhead that slows output
- **Aksana's position:** Estimates are needed for backlog sizing and capacity planning (knowing if there's 2 hours vs. 20 hours of work in the queue); T-shirt sizing is sufficient
- **Noah's position:** Group estimation catches hidden complexity (a dev might estimate 2 hours on something that's actually 20), but acknowledges it takes time and devs don't enjoy it
- **Laura's position:** 95% of projects she's worked on use individual estimation by the assigned dev
- **No final decision** reached; team will continue current approach where individual devs estimate their own tickets

### Schedule and Meeting Changes
- **Monday May 26th** is Memorial Day -- office closed
- **Backlog grooming meeting** moved from Monday to Friday at noon
- Meeting renamed from "backlog grooming" to **"project planning" or "backlog planning"** to better reflect its purpose (soft sprint planning, work clarity review)
- **Thursday working session** cancelled due to scheduling conflicts (Aksana's Thursdays blocked by Duke)
- Team will reassess whether a second weekly session is needed if standups start running long

### Aaron's Availability
- Aaron left the meeting early; team noted he's mostly on ABC this week with limited APW availability
- Noah already commented on Aaron's post that T-Mobile FWA testing is ready when he has time
- Team to ping Aaron in the channel about T-Mobile testing (~2 hours of work)

---

## Action Items

| Owner | Action | Priority |
|-------|--------|----------|
| Noah | Add ticket and remove "no restriction" option from service plan types, keeping only PPU and FWA | High |
| Aksana | Draft comment to Trello ticket (WATM-1978) explaining service plan type changes to client | High |
| Richard | Fix invoice to NACHA discrepancy bug | High |
| Richard | Perform MySQL upgrade on review environment with Claude-based smoke testing | High |
| Richard | Review MySQL upgrade plan with Swiz before production implementation | Medium |
| Richard | Share finalized MySQL upgrade plan with team for visibility | Medium |
| Richard | Prepare recommendations for three blocked client bugs before next week's meeting | Medium |
| Laura | Move backlog grooming meeting from Monday to Friday at noon | Medium |
| Laura | Rename backlog grooming meeting to "project planning" or "backlog planning" | Low |
| Team | Ping Aaron about T-Mobile FWA testing availability this week | Medium |

---

## Status Updates

- **Noah:** Distracted by CP issues but expects quick resolution. Working on Verizon FWA service plan type fix -- will add a ticket and implement the PPU/FWA-only change once CP work is cleared
- **Richard:** Resolved the critical invoice issue yesterday and sent client email. Priorities are: (1) fix invoice-to-NACHA discrepancy bug, (2) begin MySQL upgrade starting with review environment. Has three blocked bugs awaiting client discussion
- **Stone:** Still deep in Etink work. Expects to have APW availability this afternoon after reaching a good stopping point on his current Etink ticket
- **Aaron:** Mostly on ABC this week per his standup post. T-Mobile FWA testing is ready for him when he has availability (~2 hours)
- **Aksana:** Holding off on adding new work to the backlog given current capacity constraints. Will draft the Verizon service plan type change communication to the client
