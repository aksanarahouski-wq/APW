# WATM Data Purging - Product Requirements Document

**Project:** WATM - Wireless Access and Telemetry Management
**Date:** December 3, 2025
**Status:** Planning - Developer Discussion Draft
**Related Tickets:** [WATM-1594](https://orases.atlassian.net/browse/WATM-1594), WATM-1788 (Amazon Glacier)
**Document Owner:** [Your Name]

---

## Executive Summary

The WATM production database has three tables growing at 500K-1M records per day, causing query performance degradation and increasing infrastructure costs. We need a scalable, configurable data purging system that safely archives old data to AWS Glacier and removes it from production database.

**Goal:** Improve database performance by 80% and reduce storage costs by 70% through automated data archival.

---

## Problem Statement

### The Issue
- Database size: ~800GB (growing 300GB/year)
- Query response times: 8-15 seconds (used to be sub-second)
- 90% of queries access last 30 days of data, but we store years of historical data
- Current monthly RDS cost: ~$450 (projected $850 in 24 months)

### Tables Affected
| Table | Daily Growth | Retention Needed |
|-------|-------------|------------------|
| `device_checkins` | 500K-1M records | Last 90 days |
| `device_status_logs` | 200K-500K records | Last 90 days |
| `provider_data_usages` | ~50K records/month | Last 60 days |

### Impact if Not Addressed
- Database becomes unmanageable within 12-18 months
- Portal performance continues degrading
- Increasing infrastructure costs
- Customer reports/dashboards timing out

---

## Impact Analysis

### Current State Metrics

#### Database Growth Trends
```
Current Database Size: ~800GB
Annual Growth Rate: ~300GB/year
Projected Size (12 months): ~1.1TB
Projected Size (24 months): ~1.4TB
```

#### Performance Metrics
```
Average Query Response Time (device_checkins): 8-15 seconds
Average Dashboard Load Time: 12-20 seconds
Report Generation Time (monthly): 3-5 minutes (often timeout)
Database Backup Duration: 4-6 hours
```

#### Cost Projections

**Without Data Purging:**
```
Current Monthly RDS Cost: ~$450/month
Projected Cost (12 months): ~$650/month
Projected Cost (24 months): ~$850/month
```

**With Data Purging Strategy:**
```
Projected Cost (12 months): ~$300/month
Projected Cost (24 months): ~$320/month
Total 2-Year Savings: ~$6,000+
```

### Data Access Reality

**Critical Insight:** Analysis of application query patterns reveals:
- **90% of queries** access data from the last 30 days
- **9% of queries** access data from 30-90 days
- **1% of queries** access data older than 90 days (typically compliance/audit requests)

**Conclusion:** We're paying premium database costs to keep 90%+ of rarely-accessed data in high-performance storage.

### Risk Assessment Without Action

| Risk | Timeframe | Severity | Likelihood |
|------|-----------|----------|------------|
| Database performance becomes unacceptable | 6-12 months | **CRITICAL** | High |
| Portal becomes unusable during peak hours | 3-6 months | **CRITICAL** | Medium |
| Unable to onboard large customers | 6-12 months | **HIGH** | High |
| Database maintenance windows exceed acceptable downtime | 3-6 months | **HIGH** | Medium |
| Storage costs double | 12-18 months | **MEDIUM** | Very High |
| Backup/restore processes fail | 12-18 months | **CRITICAL** | Low |

---

## Proposed Solution

### High-Level Approach
1. **Archive** old data to AWS Glacier (cheap, secure storage)
2. **Verify** backup was successful
3. **Delete** archived records from production database in safe batches
4. **Enable retrieval** if historical data is needed (3-5 hour restore time)

### Key Principle
**Never delete data without verified backup.**

---

## Requirements

### Functional Requirements

#### 1. Configurable Retention Policies
- Define **per-table** retention rules (e.g., 90 days for `device_checkins`)
- Specify which **date column** to use for age calculation
- Support **variable retention periods** (not hardcoded)
- Enable/disable purging per table

#### 2. Safe Batch Deletion
- Delete records in **small batches** (e.g., 1000 at a time)
- Prevent database table locking
- Pause between batches
- Abort gracefully on errors

#### 3. Data Archival
- Export old records to compressed files (gzip)
- Upload to **AWS Glacier** for long-term storage
- Generate **manifest files** with metadata
- Track archive location in database

#### 4. Verification & Audit Trail
- Verify backup before deletion
- Log all operations to database table
- Track: what was archived, when, where, record counts
- Enable data retrieval when needed

#### 5. Execution Modes
- **Dry-run mode**: Preview what would be deleted (no actual deletion)
- **Archive-only mode**: Backup without deleting (for testing)
- **Manual execution**: On-demand via command line
- **Automated execution**: Scheduled runs (future)

### Non-Functional Requirements
- No portal downtime during execution
- No table locking or blocking
- Reversible process (can restore archived data)
- Monitoring and alerting on failures
- Execution time: Complete within 2-hour window

---

## Technical Design (High-Level)

### Configuration Approach

**Option A: YAML Configuration File**
```yaml
tables:
  device_checkins:
    enabled: true
    retention_days: 90
    date_column: "created"
    batch_size: 1000

  device_status_logs:
    enabled: true
    retention_days: 90
    date_column: "created"
    batch_size: 1000
```

**Option B: Database Configuration Table**
```sql
CREATE TABLE data_purging_policies (
    id INT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL,
    enabled TINYINT(1) DEFAULT 1,
    retention_days INT NOT NULL,
    date_column VARCHAR(50) NOT NULL,
    batch_size INT DEFAULT 1000,
    last_run_at DATETIME NULL
);
```

### Command-Line Interface
```bash
# Basic usage
./data_purging.sh --table device_checkins

# Custom retention
./data_purging.sh --table device_checkins --retention-days 120

# Dry run (preview only)
./data_purging.sh --table device_checkins --dry-run

# Archive without deleting (testing)
./data_purging.sh --table device_checkins --archive-only
```

### Archive Storage Structure
```
s3://watm-archived-data/
├── device_checkins/
│   ├── 2025/
│   │   └── 12/
│   │       ├── device_checkins_2025-12-03.sql.gz
│   │       └── device_checkins_2025-12-03.manifest.json
├── device_status_logs/
└── provider_data_usages/
```

### Execution Flow
```
1. Read configuration (which table, retention policy)
2. Query: Count records older than retention period
3. Export old records to compressed file
4. Upload to AWS Glacier
5. Verify upload successful (checksum validation)
6. Delete records in batches (1000 at a time, with pauses)
7. Log operation to database
8. Send notification (email/Slack)
```

### Archive Tracking Table
```sql
CREATE TABLE data_archive_log (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL,
    archive_date DATETIME NOT NULL,
    date_range_start DATE NOT NULL,
    date_range_end DATE NOT NULL,
    record_count BIGINT NOT NULL,
    file_size_bytes BIGINT NOT NULL,
    s3_key VARCHAR(500) NOT NULL,
    checksum VARCHAR(64) NOT NULL,
    status VARCHAR(20) DEFAULT 'completed',
    deleted_from_production TINYINT(1) DEFAULT 0,
    execution_duration_seconds INT NULL
);
```

---

## Trigger & Scheduling Recommendations

### Options Analysis

We have three primary trigger strategies, each with trade-offs:

#### Option 1: Automated Monthly Schedule ⭐ **RECOMMENDED**

**Description:** Purging runs automatically on a fixed schedule (e.g., 1st Sunday of each month at 2 AM)

**Pros:**
- ✅ Set it and forget it - no manual intervention
- ✅ Predictable database maintenance windows
- ✅ Consistent data retention policies
- ✅ Prevents accumulation of old data
- ✅ Allows planning for maintenance windows

**Cons:**
- ❌ Less flexibility if business needs change
- ❌ Requires monitoring to catch failures
- ❌ Fixed schedule may conflict with peak usage times

**Implementation:**
```bash
# Cron job (runs 1st Sunday of each month at 2:00 AM)
0 2 1-7 * 0 /path/to/watm/config/ArchiveData/src/data_purging.sh --auto --force >> /var/log/data_purging.log 2>&1

# Or use AWS EventBridge for better monitoring
Rule: "First Sunday of each month at 2:00 AM EST"
Target: EC2 Run Command or Lambda function
```

**Best For:**
- Production environments with predictable data patterns
- Organizations wanting hands-off maintenance
- Stable retention policies

---

#### Option 2: On-Demand Manual Execution

**Description:** Operations team runs purging manually when needed

**Pros:**
- ✅ Full control over timing
- ✅ Can run during known low-traffic periods
- ✅ Flexibility for special circumstances
- ✅ Easier to monitor during execution

**Cons:**
- ❌ Requires someone to remember to run it
- ❌ Risk of forgetting, leading to data accumulation
- ❌ Inconsistent execution = inconsistent performance
- ❌ Manual overhead

**Implementation:**
```bash
# Run manually via SSH
ssh production-server
cd /path/to/watm/config/ArchiveData/src/
./data_purging.sh --table device_checkins --dry-run
./data_purging.sh --table device_checkins --force
```

**Best For:**
- Initial testing and validation phases
- Organizations with unpredictable usage patterns
- Situations requiring manual oversight

---

#### Option 3: Hybrid - Automated with Manual Override ⭐ **RECOMMENDED FOR PRODUCTION**

**Description:** Automated monthly runs with ability to run on-demand when needed

**Pros:**
- ✅ Best of both worlds - automation + flexibility
- ✅ Automated baseline prevents data accumulation
- ✅ Manual runs available for special circumstances
- ✅ Can adjust retention policies per execution

**Cons:**
- ❌ Slightly more complex setup
- ❌ Need to prevent simultaneous executions

**Implementation:**
```yaml
# Automated base schedule
Cron: Monthly automated run with standard retention (90 days)

# Manual execution available via:
Option A: SSH + command line
Option B: Admin dashboard UI button (future enhancement)
Option C: API endpoint with authentication

# Locking mechanism
- Check for .lock file before execution
- Create lock file at start
- Remove lock file on completion
- Abort if lock file exists (another run in progress)
```

**Best For:**
- Production environments (our recommendation)
- Organizations wanting automation with flexibility
- Situations requiring occasional policy adjustments

---

### Detailed Recommendation: Hybrid Approach

#### Phase 1: Initial Rollout (Months 1-3)
**Strategy:** On-demand manual execution

1. **Month 1:** Run on-demand with small limits for testing
   ```bash
   ./data_purging.sh --table device_checkins --limit 100000 --dry-run
   ./data_purging.sh --table device_checkins --limit 100000
   ```

2. **Month 2:** Run larger purges, validate performance improvements
   ```bash
   ./data_purging.sh --table device_checkins
   ./data_purging.sh --table device_status_logs
   ```

3. **Month 3:** Full purge of all tables, measure results

#### Phase 2: Automation (Month 4+)
**Strategy:** Automated monthly with manual override

**Automated Schedule:**
- **Frequency:** Monthly
- **Day:** First Sunday of the month (lowest traffic)
- **Time:** 2:00 AM EST (off-peak hours)
- **Tables:** All enabled tables in configuration
- **Retention:** Standard 90-day policy

**Manual Override Available:**
- Operations team can run on-demand if needed
- Useful for emergency performance issues
- Custom retention periods for special cases

**Monitoring:**
- CloudWatch alarm if purging fails
- Email notification on completion
- Slack notification for any errors
- Dashboard showing last run status

#### Execution Schedule Example

```bash
# /etc/cron.d/watm-data-purging

# Monthly automated purge - First Sunday at 2:00 AM
0 2 1-7 * 0 watm /path/to/data_purging_wrapper.sh >> /var/log/watm/purging.log 2>&1

# Wrapper script includes:
# - Lock file check
# - Pre-execution health check
# - Execution of purge for all enabled tables
# - Post-execution validation
# - Notification on completion/failure
```

---

### Trigger Scheduling Matrix

| Scenario | Recommended Trigger | Frequency | Rationale |
|----------|-------------------|-----------|-----------|
| Production (stable) | Automated Monthly | 1st Sunday @ 2 AM | Hands-off, predictable, prevents accumulation |
| Production (high-growth) | Automated Bi-weekly | 1st & 3rd Sunday @ 2 AM | More frequent purging for rapid data growth |
| Testing/Staging | On-Demand | As needed | Manual control for testing scenarios |
| Initial Rollout | On-Demand → Automated | Monthly after validation | Start manual, automate once proven |
| Emergency Cleanup | On-Demand | Immediate | Manual execution for urgent performance issues |

---

### Additional Scheduling Considerations

#### 1. **Time Zone Considerations**
- Schedule based on EST/EDT (primary user base)
- Avoid scheduling during business hours (9 AM - 6 PM EST)
- Consider international users if applicable

#### 2. **Resource Contention**
- Don't schedule during:
  - Backup windows
  - Billing cycle processing
  - Other scheduled maintenance
  - Known peak traffic times

#### 3. **Seasonal Adjustments**
- Month-end/quarter-end: Avoid (billing processes)
- Holidays: Defer purging to avoid issues
- Fiscal year-end: Extra caution

#### 4. **Notification Strategy**
```
Before Execution:
  - Send alert to ops team 1 hour before scheduled run

During Execution:
  - Real-time progress updates via CloudWatch

After Execution:
  - Success: Email summary (records archived, time taken, space freed)
  - Failure: Immediate Slack alert + PagerDuty escalation
```

---

## Open Questions for Team Discussion

### 1. Configuration Management
- **Question:** YAML file vs database table for configuration?
- **Options:**
  - **YAML file**: Easier to version control, requires code deploy to change
  - **Database table**: Dynamic changes without deploy, requires UI/SQL access
  - **Hybrid**: YAML for defaults, database for overrides
- **Recommendation needed from team**

### 2. Execution Trigger Strategy
- **Question:** How should purging be triggered?
- **Options:**
  - **Manual on-demand**: Operations team runs when needed
  - **Automated monthly**: Cron job runs 1st Sunday @ 2 AM
  - **Hybrid**: Automated baseline + manual override capability
- **Discussion points:**
  - Do we want "set it and forget it" automation?
  - What time/day minimizes user impact?
  - Who gets notified of success/failure?

### 3. Retention Policies
- **Question:** Are 90-day (device tables) and 60-day (billing) retention periods correct?
- **Discussion points:**
  - Do we have compliance requirements for longer retention?
  - Do any reports/dashboards need older data?
  - Should we start conservative (120 days) and reduce later?

### 4. Archive Storage Strategy
- **Question:** AWS Glacier storage class?
- **Options:**
  - **Glacier Flexible Retrieval**: $0.004/GB/month, 3-5 hour restore (recommended)
  - **Glacier Deep Archive**: $0.00099/GB/month, 12-48 hour restore (cheapest)
- **Recommendation:** Flexible Retrieval (balanced cost/retrieval time)

### 5. Batch Size & Performance
- **Question:** What batch size prevents table locking without being too slow?
- **Options:** 500 / 1000 / 2000 records per batch
- **Discussion:** Need to test on production to determine optimal size

### 6. Rollout Strategy
- **Question:** How to safely test and roll out?
- **Proposed phased approach:**
  - **Phase 1:** Manual testing with small dataset (1 month of data)
  - **Phase 2:** Manual purge of all tables (full 90-day retention)
  - **Phase 3:** Automated monthly execution
- **Does team agree with this approach?**

### 7. Monitoring & Alerting
- **Question:** Who gets notified and how?
- **Discussion points:**
  - Email to: devops@orases.com?
  - Slack channel: #watm-operations?
  - CloudWatch alarms for failures?
  - Dashboard showing last run status?

### 8. Data Retrieval Process
- **Question:** Who can request archived data restoration? What's the process?
- **Discussion points:**
  - Support team submits ticket?
  - Operations team handles directly?
  - SLA for retrieval (24-48 hours)?

---

## Success Metrics

### Performance Improvements (Post-Purge)
- Database size reduced by **70-80%** (800GB → 150-200GB)
- Query response time reduced by **80%** (15 sec → 2-3 sec)
- Dashboard load time reduced by **50%**
- Report generation time reduced by **60%**

### Cost Savings
- RDS storage costs reduced by **70%** (~$75/month savings)
- Glacier storage costs: ~$2-5/month (minimal)
- **Annual savings: $5,000-6,000**

### Operational Goals
- Archive success rate: **100%** (no data loss)
- Automated execution reliability: **99%+**
- Retrieval capability: **100%** (all archives accessible)

---

## Existing Implementation

**Note:** A shell script already exists at `watm/config/ArchiveData/src/data_archival.sh` that handles archival for specific tables. This PRD proposes enhancements to make it more scalable and configurable.

### Current Script Capabilities
- Exports table data to compressed files
- Uploads to AWS Glacier
- Deletes in batches to prevent locking

### Proposed Enhancements
- Variable inputs (table name, retention days)
- Configuration file support
- Multiple execution modes (dry-run, archive-only)
- Enhanced logging and tracking
- Automated scheduling capability

---

## Implementation Phases

### Phase 1: Enhancement & Testing (2 weeks)
- [ ] Enhance existing script with variable inputs
- [ ] Create configuration system (YAML or database)
- [ ] Create `data_archive_log` tracking table
- [ ] Test on dev environment

### Phase 2: Production Validation (1 week)
- [ ] Run dry-run on production
- [ ] Manual test: Archive 1 month of `device_checkins`
- [ ] Verify archive and retrieval process
- [ ] Measure performance improvement

### Phase 3: Full Rollout (1 week)
- [ ] Purge all three tables (90/60 day retention)
- [ ] Monitor database performance
- [ ] Validate results

### Phase 4: Automation (1 week)
- [ ] Set up automated monthly execution
- [ ] Configure monitoring/alerting
- [ ] Document operational procedures

**Total Timeline: 5-6 weeks**

---

## Risks & Mitigation

| Risk | Mitigation |
|------|-----------|
| **Data loss during purge** | Backup-first approach, verification before deletion, extensive testing |
| **Table locking during deletion** | Batch deletion (1000 records at a time), tested on current production |
| **Archive not retrievable** | Regular verification tests, checksum validation |
| **Script failure mid-execution** | Locking mechanism, rollback procedures, alerts |

---

## Next Steps

1. **Developer Review** - Discuss open questions and technical approach
2. **Architecture Alignment** - Agree on configuration approach and trigger strategy
3. **Estimate Effort** - Size each implementation phase
4. **Approve PRD** - Sign off on approach and timeline
5. **Begin Phase 1** - Enhance existing script with new capabilities

---

## Questions or Feedback?

Please provide input on:
- Configuration approach preference (YAML vs database table)
- Execution trigger strategy (manual, automated, hybrid)
- Retention period validation (90 days correct?)
- Rollout phasing approach
- Any technical concerns or suggestions

**Contact:** [Your Name/Team] for questions or to schedule alignment meeting
