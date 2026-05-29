## CakePHP & PHP/Ubuntu Upgrade Strategy — Decision Summary

### Release Structure

We have broken the upgrade work into three releases that must follow a specific sequence:

**Release 1 — CakePHP 4.6 Upgrade**
Upgrade the current CakePHP framework to version 4.6. This is the code-level upgrade and carries the bulk of the testing effort. Noah is leading this work.

**Release 2 — PHP 8.2 + Ubuntu Server Upgrade**
Spin up new AWS servers with the latest Ubuntu (which ships with PHP 8.2 natively), rather than using a third-party PPA to upgrade PHP on existing servers. Richard is leading this work. The approach is:
- Take an AMI of the review server, spin up a new instance
- Use Claude Code to document all configuration differences and generate a repeatable script
- Validate on review first — if it goes poorly, we reassess the approach without having impacted anything
- Once proven on review, apply the same process to beta, then to all three production servers
- Production servers are more configuration-heavy (AWS roles, specific configs), so they will take more effort per server, but each subsequent one should go faster after the first

**Release 3 — CakePHP 5 Upgrade**
The major framework upgrade to Cake 5 (targeting 5.3, which requires PHP 8.2). This includes upgrading all Orases proprietary packages to Cake 5-compatible versions.

### Sequencing Decisions

- **Release 1 and Release 2 can run in parallel.** They are not dependent on each other and can be assigned to different developers working concurrently.
- **Release 3 cannot start until both Release 1 and Release 2 are complete.** The team needs fully functional and testable lower environments before taking on the Cake 5 upgrade.
- **If we start, we commit to going all the way to production.** We will not stop at review or beta. If after the review server attempt we determine the approach is too costly, we fail fast and revert — but we do not leave environments on mismatched versions long-term.

### Pre-Work for Release 3

Even before Release 3 formally kicks off, the team should begin researching the Orases packages (users, sites, files, logs, imports, theme-limitless) to determine if Cake 5-compatible versions exist or are planned. This research can happen independently and in parallel with Release 1 and 2 work, so it does not become a blocker later.

### Testing

The testing effort outlined by Aaron applies primarily to validating the PHP upgrade impact on the application — it does not need to be repeated per server. Once the application is verified on one environment, the remaining servers are a configuration/DevOps exercise.
