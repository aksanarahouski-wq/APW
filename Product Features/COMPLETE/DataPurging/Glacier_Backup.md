# AWS Glacier Data Archival - Project Status

**JIRA Ticket:** [WATM-1594](https://orases.atlassian.net/browse/WATM-1594)
**Epic:** WATM-1788 (Amazon Glacier)
**Status:** Development Complete - Awaiting Testing
**Last Updated:** November 26, 2025

---

## Executive Summary

We have developed a solution to archive older database records to AWS Glacier, which will improve database performance and reduce storage costs. The archival script is ready for testing on production with a small dataset. Once validated, we can implement it at scale.

---

## The Problem

The WATM production database has three tables that are growing very large, which is causing:

* Database queries to run slower
* Increased hosting costs
* Longer backup and maintenance times
* Greater risk of operational issues

### Tables Requiring Archival

| Table Name | What It Stores | Why It's Growing | What We Need Active |
| --- | --- | --- | --- |
| `device_checkins` | Device check-in records | Every device checks in regularly | Last 3 months only |
| `device_status_logs` | Device status change history | Status changes tracked continuously | Last 3 months only |
| `provider_data_usages` | Provider data usage records | Monthly usage data accumulates | Current + previous billing cycle only |

**The Reality:** Most of this historical data is rarely accessed, but we can't delete it permanently due to compliance and audit requirements.

---

## Our Solution

We've developed an automated archival system that:

1. **Safely backs up** old records to AWS Glacier (long-term storage)
2. **Verifies** the backup was successful
3. **Removes** the archived records from the active database
4. **Preserves** the ability to retrieve archived data if needed

### What We Agreed On

After analyzing the data patterns and business needs, we determined these retention policies:

**device_checkins & device_status_logs:**

* Keep the last 3 months in the database
* Archive everything older than 3 months

**provider_data_usages:**

* Keep current billing cycle + previous cycle in the database
* Archive older billing cycles

---

## Development Journey & Current Status

### What We Built

We created a flexible archival script (`data_archival.sh`) located at `watm/config/ArchiveData/src/` that can safely archive any table's data based on custom query conditions. The script:

1. Exports matching records from the database
2. Uploads the backup to AWS Glacier for long-term storage
3. Deletes the archived records in small batches to prevent database locks
4. Handles large files (supports multipart uploads for files over 3GB)
5. Includes safety mechanisms to abort gracefully if interrupted

### Early Challenges We Solved

**Initial Problem:** The first version of the script caused database table locks, which could have disrupted the production application.

**Solution Implemented:** We redesigned the script to delete records in batches of 1,000 at a time instead of all at once. This prevents any locking issues and allows the database to continue serving the application normally during archival.

**Current Status:** The script is ready for testing but hasn't been run on production yet due to competing priorities.

---

## What's Next - Testing & Implementation

### Proposed Next Steps

**Step 1: Small Production Test**

* Run the script on production with a very small query (e.g., archive 1 month of old data from one table)
* Monitor database performance during execution
* Verify the archive was created successfully
* Confirm records were deleted safely

**Step 2: Validate Results**

* Check that archived data can be retrieved from Glacier
* Confirm database performance improved (even slightly)
* Review any errors or warnings

**Step 3: Scale Up (if test succeeds)**

* Run larger archival queries for all three tables
* Implement on a recurring schedule (monthly)
* Monitor ongoing performance and cost savings

### Why Start Small?

Testing with a small dataset on production will:

* Give us confidence the script works correctly in the real environment
* Identify any edge cases we haven't considered
* Allow us to fine-tune the process before committing to large-scale archival
* Provide concrete performance metrics to validate the benefit

---

## Expected Benefits

### Performance Improvements

* **Faster Queries:** Smaller tables mean faster searches and reports
* **Improved Responsiveness:** Database can handle more concurrent users
* **Reduced Maintenance Time:** Backups and updates complete faster

### Cost Savings

* **Database Storage:** Reduced RDS storage costs (current database is paying for storage it rarely uses)
* **Glacier Storage:** Very inexpensive long-term storage (~$0.004 per GB per month)
* **Future-Proofing:** Prevents tables from becoming unmanageably large

### Data Accessibility

* **Archived data is NOT deleted** - it's moved to secure long-term storage
* **Can be retrieved** if needed for audits, analysis, or compliance
* **Retrieval time:** 3-5 hours for standard retrieval, faster options available for urgent needs

---

## How the Script Works (Non-Technical Overview)

The archival script accepts two inputs:

1. **Table name** - which table to archive from (e.g., `device_checkins`)
2. **Query condition** - what records to archive (e.g., "older than 3 months")

Then it automatically:

1. **Backs up** the matching records to a file
2. **Uploads** that backup to AWS Glacier
3. **Verifies** the upload was successful
4. **Deletes** the backed-up records from the database (in safe batches)
5. **Cleans up** temporary files

The entire process is designed to be safe and reversible. If anything fails, the script stops and doesn't delete data.

---

## Questions We Should Answer During Testing

1. **Performance:** How much faster are queries after archiving a small amount of data?
2. **Safety:** Does the batch deletion approach work smoothly without locking tables?
3. **Timing:** How long does it take to archive different amounts of data?
4. **Automation:** How often should we run this (monthly, quarterly)?
5. **Retrieval:** How quickly can we restore archived data if needed?

---

## Recommendations

### Immediate Action Items

1. **Schedule a small production test**

    * Choose a low-traffic time window (e.g., weekend morning)
    * Archive just 1-2 months of old data from `device_checkins` table
    * Monitor and document results

2. **After successful test, create full implementation plan**

    * Define exact archival schedules for each table
    * Set up monitoring and alerting
    * Document retrieval procedures for the team

3. **Consider automation**

    * Once validated, set up monthly automated archival
    * Create a dashboard to track archived data and savings
    * Establish procedures for data retrieval requests


---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| Script fails during execution | Low | Low | Script has rollback mechanisms; start with small test |
| Database performance during archival | Low | Low | Batch deletions prevent locks; run during low-traffic window |
| Archived data is not retrievable | Very Low | Medium | Test retrieval before deleting large datasets |
| Cost higher than expected | Very Low | Low | Glacier storage is extremely cheap (~$0.004/GB/month) |

---

## Summary

We have a working solution ready to test. The script has been refined to handle the early issues (table locking) and now safely processes archival in batches.

**The next logical step is a small production test to validate the approach before implementing at scale.**

Once validated, this will provide ongoing performance improvements and cost savings while maintaining full data retention for compliance.
