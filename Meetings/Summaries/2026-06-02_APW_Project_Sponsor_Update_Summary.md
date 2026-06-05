# APW Project Sponsor Update — June 2, 2026

**Date:** June 2, 2026
**Duration:** ~55 minutes
**Organizer:** Laura Perry
**Recording:** [tldv](https://tldv.io/app/meetings/6a1f1a18234d0000133d2bc7)

## Attendees

- Devon D'Andrea (APW)
- Adam Curcie (APW)
- Laura Perry (Orases)
- Aksana Rahouski (Orases)

## Meeting Overview

Biweekly status/scorecard meeting covering the company rebranding to IDM Wireless, May spending and budget projections, progress on backend infrastructure upgrades (MySQL, server, CakePHP), status of feature development including T-Mobile functionality and device status enhancements, a post-mortem on an unauthorized email template deployment, and upcoming trade shows and a potential quarterly in-person meeting.

## Topics Discussed

### Project Branding / IDM Wireless Name Change

- APW is officially rebranding to **IDM Wireless** (International Device Management).
- Vince created several logo concepts; the team responded positively, particularly to one featuring a globe.
- The "international" angle is justified by existing connections in Canada and emerging connections in Australia.
- Devon is planning a marketing blast covering the rebrand, a July 4th / 250th anniversary device sale promotion, and new portal features — described as "the loudest blast we've probably ever put out."
- A 60-day deadline was imposed in early May (by opposing legal counsel in a trademark lawsuit), targeting early July to demonstrate rebranding efforts. APW's lawyers requested 180 days; they expect a compromise around 90 days, pushing to approximately August.
- SEO company Funnel Boost was paid $3,500 to handle website content changes. Chad may only need oversight involvement.
- Remaining work includes creating new email addresses, domain redirects, and content replacement across all properties.

### Budget & Spending Update

- May spending was lower than typical due to Noah being pulled to other client projects and Stone's other project having fires.
- Year-to-date spend through end of May: approximately $310K.
- Annual projection adjusted down from slightly over $700K to slightly under $700K due to the lighter May.
- Key May accomplishments: CakePHP 4.6 upgrade, MySQL upgrade, Verizon second account, invoice template, and several bug fixes.

### Backend Infrastructure & Upgrades

- **MySQL upgrade** completed the day of the meeting.
- **Server upgrade** scheduled for Thursday (June 4).
- **CakePHP 4.6 minor version upgrade** is live in production and stable.
- **CakePHP 5 major version upgrade** is the next infrastructure milestone. Noah will lead it once he wraps T-Mobile work.
- Team is intentionally interleaving upgrade work with feature development to combine testing efforts and avoid redundant QA passes.
- Larger features like Configurations V2 will wait until after the CakePHP 5 upgrade is complete and tested.

### Feature Development & Testing

- **T-Mobile functionality**: In UAT. Adam commented it looked good. Devon to complete testing; Aksana urged thorough testing given FWA support and multi-carrier complexity.
- **Additional device status & admin-configurable status**: In QA with Aaron; expected ready for client review later that day or the next morning.
- **Combined production push** of all approved items targeting early the following week, after MySQL and server upgrades settle.
- **Power relay detection (WATM-2163)**: Devon and Adam discovered that toggling the InHand router IO mode from output to input allows detection of whether a power relay is physically connected (via closed electrical circuit detection). This has been a customer pain point for ~6 years. Needs an additional command to check whether the router has an IO port. Applies to the multi-outlet power relay feature as well. Ticket moved to top of prioritized backlog.
- **Signal strength reporting fix**: The i4100 device on newer firmware reports a null/empty value for the standard signal parameter (WAN1 RSSI), causing the portal to display an incorrect 183% signal strength. Agreed approach: cascading fallback — check WAN1 dBm first, then WAN1 RSSI, then WAN1 SIG RSSI. Handle null values properly. InHand will not rewrite firmware for the legacy i4100 device.
- **Twilio account integration**: Queued as next priority after IO/power relay detection work.

### Deactivated Device Email Template Incident

- An updated email template for the "admin deactivated" device status was pushed to production without client review.
- Root cause: Stone redesigned the email to improve UX while working on the deactivated status feature. Due to being pulled off the project for a week-plus, the CakePHP release, and "many cooks in the kitchen," it was accidentally included in the production deployment.
- The email contained an incorrect statement about billing, which was fixed immediately once discovered.
- Devon acknowledged the template "looked awesome" aside from the billing error.
- Aksana confirmed the team discussed that all changes must be reviewed, approved, and tested before release.
- June billing provided free as compensation.

### Trade Shows & Quarterly Meeting

- APW planning trade shows from September through April (potentially one per month) to promote the IDM Wireless rebrand.
- Teams discussed scheduling a quarterly in-person meeting — overdue and want to meet before trade show season. Targeting summer (July/August).

## Action Items

- **Devon**: Finalize the rebranding timeline/deadline once legal negotiations settle (60 vs. 90 vs. 180 days) and communicate to Orases.
- **Devon**: Contact Chad regarding oversight needs for the website content change (Funnel Boost handling execution).
- **Devon & Adam**: Complete UAT testing of T-Mobile functionality (target: end of day or next day).
- **Devon**: Update WATM-2163 (power relay detection) ticket with the additional command to check if the router has an IO port.
- **Adam & Devon**: Test the WAN1 dBm signal parameter on non-upgraded devices and provide findings to Orases.
- **Laura**: Move WATM-2163 (power relay IO detection) to the top of the prioritized backlog.
- **Aksana / dev team**: Add third signal strength key (WAN1 dBm) to the check-in parsing logic with fallback decision tree and null-value handling.
- **Laura**: Coordinate combined production push of approved features targeting early next week.
- **Aksana**: Propose dates for a quarterly in-person meeting (summer timeframe, before September trade show season).
- **Orases team**: Reinforce process — all changes, including email templates, must go through review/approval/testing before production deployment.

## Key Decisions

- Company will rebrand to **IDM Wireless**, with "International Device Management" as the positioning angle.
- **WATM-2163 (power relay detection)** moved to top of prioritized backlog; should be addressed before or bundled with the multi-outlet power relay feature.
- **Signal strength logic** will use a cascading fallback approach: check new key first, then existing keys, and properly handle null values. No admin-side configuration needed.
- **CakePHP 5 upgrade** will proceed next (led by Noah), and large features like Configurations V2 will wait until after completion.
- **Combined production deployment** of T-Mobile, device status, and other approved features after MySQL and server upgrades settle.
- **Twilio account integration** queued as next priority after IO/power relay detection work.
