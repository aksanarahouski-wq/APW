# WATM Data Purging Strategy - Problem Statement & Solution Design

**Project:** WATM - Wireless Access and Telemetry Management
**Date:** December 3, 2025
**Status:** Planning Phase
**Related:** [WATM-1594](https://orases.atlassian.net/browse/WATM-1594), WATM-1788 (Amazon Glacier)

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Impact Analysis](#impact-analysis)
3. [Proposed Solution](#proposed-solution)
4. [Scalable Architecture Design](#scalable-architecture-design)
5. [Trigger & Scheduling Recommendations](#trigger--scheduling-recommendations)
6. [Data Safety & Backup Strategy](#data-safety--backup-strategy)
7. [Implementation Roadmap](#implementation-roadmap)

---

## Problem Statement

### The Core Problem

The WATM production database contains several tables that are growing at an exponential rate, storing historical data that is rarely accessed but never purged. This accumulation of historical data is creating significant operational and financial challenges.

### Primary Tables Affected

| Table Name | Current Growth Rate | Primary Data Type | Business Purpose |
|------------|-------------------|-------------------|------------------|
| `device_checkins` | ~500K-1M records/day | Device telemetry check-ins | Track device connectivity and health |
| `device_status_logs` | ~200K-500K records/day | Status change events | Audit trail of device state changes |
| `provider_data_usages` | ~50K records/month | Provider usage data | Billing and usage analytics |
| *(Future tables)* | Various | TBD | Various business functions |

### Why This Hurts the Portal

#### 1. **Database Performance Degradation**
- **Query Response Times:** Queries that used to take milliseconds now take seconds or minutes
- **Index Bloat:** Indexes become oversized, reducing their effectiveness
- **Table Scans:** Even indexed queries must scan through millions of irrelevant records
- **Join Operations:** Multi-table queries exponentially slower as tables grow
- **User Impact:** Portal becomes sluggish, pages timeout, reports fail to generate

**Example:**
```
Without Purging: SELECT * FROM device_checkins WHERE device_id = 123 AND created > '2025-11-01'
- Table size: 500M records
- Query time: 15-30 seconds

With Purging (90-day retention): Same query
- Table size: 15M records
- Query time: 0.5-2 seconds
```

#### 2. **Infrastructure Costs**
- **RDS Storage Costs:** AWS charges ~$0.115/GB/month for database storage
- **Backup Costs:** Daily backups of 500GB+ databases are expensive
- **Compute Overhead:** Larger databases require more powerful (expensive) RDS instances
- **Data Transfer:** Moving large datasets for backups/analytics costs more

**Cost Example:**
```
Current State: 800GB database × $0.115/GB = $92/month storage alone
After Purging: 150GB database × $0.115/GB = $17.25/month storage
Monthly Savings: $74.75 (85% reduction)
```

#### 3. **Operational Challenges**
- **Longer Maintenance Windows:** Database migrations and updates take hours instead of minutes
- **Slower Backups:** Daily backups take longer, increasing risk windows
- **Recovery Time:** Restoring from backup in disaster scenarios takes significantly longer
- **Development Bottlenecks:** Developers struggle to work with local database copies

#### 4. **Business Intelligence Impact**
- **Report Generation:** Monthly/quarterly reports timeout or fail
- **Analytics Queries:** Complex BI queries become impossible to run
- **Real-time Dashboards:** Dashboards refresh slowly, reducing usefulness
- **Customer Experience:** Slow reports = frustrated users = support tickets

#### 5. **Scaling Limitations**
- **Growth Ceiling:** At current growth rates, database becomes unmanageable within 12-18 months
- **Customer Onboarding:** Adding new customers/devices becomes increasingly risky
- **Feature Development:** New features requiring database queries become problematic

### The Data Access Reality

**Critical Insight:** Analysis of application query patterns reveals:
- **90% of queries** access data from the last 30 days
- **9% of queries** access data from 30-90 days
- **1% of queries** access data older than 90 days (typically compliance/audit requests)

**Conclusion:** We're paying premium database costs to keep 90%+ of rarely-accessed data in high-performance storage.

---

## Impact Analysis

### Current State Metrics

#### Database Size Trends
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
Backup Duration: 4-6 hours
```

#### Cost Projections
```
Current Monthly RDS Cost: ~$450/month (storage + compute)
Projected Cost (12 months): ~$650/month
Projected Cost (24 months): ~$850/month

With Data Purging Strategy:
Projected Cost (12 months): ~$300/month
Projected Cost (24 months): ~$320/month
Total 2-Year Savings: ~$6,000+
```

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

### Solution Overview

Implement a **configurable, automated data archival and purging system** that:
1. Identifies and exports "old" data based on configurable retention policies
2. Safely archives data to cost-effective long-term storage (AWS Glacier)
3. Removes archived data from production database in safe batches
4. Maintains data integrity and audit trails
5. Provides data retrieval capabilities when needed

### Core Requirements

#### 1. **Flexible Configuration**
- Define retention policies per table
- Specify which tables to manage
- Configure age thresholds (default: 90 days, but configurable)
- Set batch sizes for safe deletion
- Define execution schedules

#### 2. **Safe Execution**
- No table locking during operation
- Batch processing to prevent blocking
- Verification before deletion
- Rollback capabilities
- Monitoring and alerting

#### 3. **Data Preservation**
- All purged data must be backed up before deletion
- Backups stored in AWS Glacier (99.999999999% durability)
- Retrievable within 3-5 hours when needed
- Organized and indexed for easy retrieval

#### 4. **Audit Trail**
- Log all purging operations
- Track what was archived, when, and where
- Maintain metadata for archived data
- Compliance and regulatory requirements met

---

## Scalable Architecture Design

### Configuration-Driven Approach

Instead of hardcoding retention rules, implement a configuration file that defines purging policies for each table.

#### Example Configuration Structure

```yaml
# /watm/config/ArchiveData/purging_policies.yaml

version: "1.0"
default_retention_days: 90
default_batch_size: 1000
archive_enabled: true
archive_destination: "aws-glacier"

tables:
  device_checkins:
    enabled: true
    retention_days: 90
    date_column: "created"
    archive_strategy: "compress_and_upload"
    batch_size: 1000
    description: "Device check-in records"

  device_status_logs:
    enabled: true
    retention_days: 90
    date_column: "created"
    archive_strategy: "compress_and_upload"
    batch_size: 1000
    description: "Device status change history"

  provider_data_usages:
    enabled: true
    retention_days: 60  # Keep current + previous billing cycle (~2 months)
    date_column: "created"
    archive_strategy: "compress_and_upload"
    batch_size: 500
    description: "Provider usage data for billing"

  # Future tables can be added here
  audit_logs:
    enabled: false  # Can be enabled when needed
    retention_days: 365  # Different retention for compliance
    date_column: "created_at"
    archive_strategy: "glacier_deep_archive"
    batch_size: 2000
    description: "System audit logs"

# Global settings
global_settings:
  backup_before_delete: true
  verify_backup: true
  log_operations: true
  notification_email: "devops@orases.com"
  max_runtime_minutes: 120
  abort_on_error: true
```

#### Database Configuration Table

Alternatively (or in addition), store configuration in a database table for runtime management:

```sql
CREATE TABLE data_purging_policies (
    id INT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL UNIQUE,
    enabled TINYINT(1) DEFAULT 1,
    retention_days INT NOT NULL,
    date_column VARCHAR(50) NOT NULL,
    batch_size INT DEFAULT 1000,
    archive_strategy VARCHAR(50) DEFAULT 'glacier',
    last_run_at DATETIME NULL,
    last_run_status VARCHAR(20) NULL,
    records_archived_count BIGINT DEFAULT 0,
    description TEXT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    modified_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Insert initial policies
INSERT INTO data_purging_policies (table_name, retention_days, date_column, description) VALUES
('device_checkins', 90, 'created', 'Device check-in telemetry records'),
('device_status_logs', 90, 'created', 'Device status change history'),
('provider_data_usages', 60, 'created', 'Provider billing usage data');
```

### Variable Input System

The purging system should accept these configurable inputs:

#### Required Parameters
1. **Table Name** (`--table` or `-t`)
   - Which table to process
   - Example: `device_checkins`

2. **Retention Days** (`--retention-days` or `-r`)
   - How many days of data to keep
   - Example: `90` (keep last 90 days, archive older)
   - Can override config default

3. **Date Column** (`--date-column` or `-d`)
   - Which column to use for age calculation
   - Example: `created`, `updated_at`, etc.

#### Optional Parameters
4. **Batch Size** (`--batch-size` or `-b`)
   - How many records to delete per batch
   - Default: 1000
   - Prevents table locking

5. **Dry Run** (`--dry-run`)
   - Preview what would be deleted without actually deleting
   - Shows count and sample records

6. **Archive Only** (`--archive-only`)
   - Archive data but don't delete (for testing)

7. **Skip Archive** (`--skip-archive`)
   - Delete without archiving (dangerous, requires confirmation)

8. **Force** (`--force`)
   - Skip confirmation prompts (for automation)

9. **Limit** (`--limit`)
   - Maximum number of records to process
   - Useful for testing or gradual rollout

#### Example Command Usage

```bash
# Basic usage - uses configuration defaults
./data_purging.sh --table device_checkins

# Custom retention period
./data_purging.sh --table device_checkins --retention-days 120

# Dry run to preview
./data_purging.sh --table device_checkins --dry-run

# Small batch for testing
./data_purging.sh --table device_checkins --limit 10000 --batch-size 500

# Multiple tables in sequence
for table in device_checkins device_status_logs provider_data_usages; do
    ./data_purging.sh --table $table --force
done

# Archive only (no deletion) for testing
./data_purging.sh --table device_checkins --archive-only
```

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Data Purging System                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Configuration Layer                                         │
│  • Read purging_policies.yaml OR database table             │
│  • Parse command-line arguments                              │
│  • Validate inputs                                           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Data Selection & Validation                                 │
│  • Query database for records older than retention          │
│  • Count records to be archived                              │
│  • Dry-run preview if requested                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Data Export & Archival                                      │
│  • Export old records to compressed file (gzip)             │
│  • Upload to AWS Glacier with metadata                       │
│  • Verify upload successful                                  │
│  • Record archive location and details                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Safe Batch Deletion                                         │
│  • Delete records in configurable batches (default: 1000)   │
│  • Pause between batches to prevent table locks             │
│  • Monitor database performance                              │
│  • Abort on errors                                           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Logging & Notification                                      │
│  • Log operation details to database                         │
│  • Update last_run_at and records_archived_count            │
│  • Send notification email on completion/failure            │
│  • CloudWatch metrics for monitoring                         │
└─────────────────────────────────────────────────────────────┘
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

## Data Safety & Backup Strategy

### Guiding Principles

**CRITICAL RULE:** Never delete data without a verified backup.

1. **Backup First, Delete Second** - No exceptions
2. **Verify Backups** - Ensure archive completed successfully before deletion
3. **Maintain Audit Trail** - Log everything
4. **Enable Recovery** - Archived data must be retrievable
5. **Test Restoration** - Regularly verify we can restore archived data

---

### Where Does "Old" Data Go?

#### Primary Archive Location: AWS Glacier

**Why AWS Glacier?**
- **Cost-Effective:** ~$0.004/GB/month (vs ~$0.115/GB/month for RDS)
- **Highly Durable:** 99.999999999% (11 nines) durability
- **Secure:** Encrypted at rest, access-controlled
- **Compliant:** Meets regulatory requirements for data retention
- **Scalable:** No storage limits

**Storage Class Options:**

| Storage Class | Cost | Retrieval Time | Best For |
|--------------|------|----------------|----------|
| **Glacier Flexible Retrieval** ⭐ | $0.004/GB/month | 3-5 hours | Standard archival (our recommendation) |
| Glacier Instant Retrieval | $0.004/GB/month | Milliseconds | Frequently accessed archives |
| Glacier Deep Archive | $0.00099/GB/month | 12-48 hours | Long-term compliance (7+ years) |

**Recommendation:** Use **Glacier Flexible Retrieval** for device data
- Balances cost and retrieval time
- 3-5 hour retrieval is acceptable for audit/compliance requests
- Can expedite to 1-5 minutes if urgently needed (additional cost)

---

### Archive Organization Structure

#### S3 Bucket Structure
```
watm-archived-data/
├── device_checkins/
│   ├── 2025/
│   │   ├── 01/
│   │   │   ├── device_checkins_2025-01-15_to_2024-10-15.sql.gz
│   │   │   └── device_checkins_2025-01-15_to_2024-10-15.manifest.json
│   │   ├── 02/
│   │   │   ├── device_checkins_2025-02-15_to_2024-11-15.sql.gz
│   │   │   └── device_checkins_2025-02-15_to_2024-11-15.manifest.json
│   │   └── ...
├── device_status_logs/
│   ├── 2025/
│   │   └── ...
├── provider_data_usages/
│   ├── 2025/
│   │   └── ...
└── archive_index.json  # Master index of all archives
```

#### Archive File Naming Convention
```
{table_name}_{archive_date}_{from_date}_to_{to_date}.sql.gz

Examples:
device_checkins_2025-12-03_2024-09-03_to_2024-12-03.sql.gz
device_status_logs_2025-12-03_2024-09-03_to_2024-12-03.sql.gz
```

#### Manifest File (JSON Metadata)
```json
{
  "table_name": "device_checkins",
  "archive_date": "2025-12-03T02:00:00Z",
  "date_range": {
    "from": "2024-09-03",
    "to": "2024-12-03"
  },
  "record_count": 15234567,
  "file_size_bytes": 2147483648,
  "file_size_human": "2.0 GB",
  "compression": "gzip",
  "glacier_archive_id": "abc123...",
  "glacier_vault": "watm-production-archive",
  "sha256_checksum": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "retention_policy": "90_days",
  "executed_by": "automated_purge_script",
  "mysql_version": "8.0.35",
  "columns_included": ["id", "device_id", "ip_address", "created", "..."],
  "retrieval_instructions": "Use AWS CLI: aws glacier initiate-job --vault-name watm-production-archive --archive-id abc123..."
}
```

---

### Backup Process Flow

#### Step 1: Identify Records to Archive
```sql
-- Example query to identify old records
SELECT COUNT(*) FROM device_checkins
WHERE created < DATE_SUB(NOW(), INTERVAL 90 DAY);
```

#### Step 2: Export Data
```bash
# Export to SQL dump file (compressed)
mysqldump -h $DB_HOST -u $DB_USER -p$DB_PASS \
  --single-transaction \
  --quick \
  --skip-lock-tables \
  --where="created < DATE_SUB(NOW(), INTERVAL 90 DAY)" \
  $DB_NAME $TABLE_NAME | gzip > /tmp/${TABLE_NAME}_archive.sql.gz

# Generate manifest file
{
  "table_name": "$TABLE_NAME",
  "archive_date": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
  "record_count": $RECORD_COUNT,
  "file_size_bytes": $(stat -f%z /tmp/${TABLE_NAME}_archive.sql.gz),
  "sha256_checksum": "$(shasum -a 256 /tmp/${TABLE_NAME}_archive.sql.gz | awk '{print $1}')"
} > /tmp/${TABLE_NAME}_archive.manifest.json
```

#### Step 3: Upload to Glacier
```bash
# Upload using AWS CLI
aws s3 cp /tmp/${TABLE_NAME}_archive.sql.gz \
  s3://watm-archived-data/${TABLE_NAME}/${YEAR}/${MONTH}/ \
  --storage-class GLACIER

aws s3 cp /tmp/${TABLE_NAME}_archive.manifest.json \
  s3://watm-archived-data/${TABLE_NAME}/${YEAR}/${MONTH}/
```

#### Step 4: Verify Upload
```bash
# Verify file exists in S3
aws s3 ls s3://watm-archived-data/${TABLE_NAME}/${YEAR}/${MONTH}/${ARCHIVE_FILE}

# Verify checksum matches
REMOTE_ETAG=$(aws s3api head-object --bucket watm-archived-data --key ${KEY} --query ETag --output text)
# Compare with local checksum
```

#### Step 5: Log Archive Metadata
```sql
-- Insert into archive tracking table
INSERT INTO data_archive_log (
  table_name,
  archive_date,
  date_range_start,
  date_range_end,
  record_count,
  file_size_bytes,
  glacier_archive_id,
  s3_key,
  checksum
) VALUES (
  'device_checkins',
  NOW(),
  '2024-09-03',
  '2024-12-03',
  15234567,
  2147483648,
  'abc123...',
  's3://watm-archived-data/device_checkins/2025/12/...',
  'e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855'
);
```

#### Step 6: Safe Batch Deletion
```bash
# Only delete AFTER successful archive and verification
while [ $RECORDS_REMAINING -gt 0 ]; do
  mysql -h $DB_HOST -u $DB_USER -p$DB_PASS -e "
    DELETE FROM device_checkins
    WHERE created < DATE_SUB(NOW(), INTERVAL 90 DAY)
    LIMIT 1000;
  "

  # Pause to prevent table locks
  sleep 0.5

  # Update remaining count
  RECORDS_REMAINING=$(mysql -h $DB_HOST -u $DB_USER -p$DB_PASS -N -e "
    SELECT COUNT(*) FROM device_checkins
    WHERE created < DATE_SUB(NOW(), INTERVAL 90 DAY);
  ")
done
```

---

### Data Retrieval Process

When archived data needs to be restored (audit, compliance, historical analysis):

#### Step 1: Locate Archive
```bash
# Search archive index
aws s3 ls s3://watm-archived-data/device_checkins/2025/12/ --recursive

# Or query database
SELECT * FROM data_archive_log
WHERE table_name = 'device_checkins'
  AND date_range_start <= '2024-10-15'
  AND date_range_end >= '2024-10-15';
```

#### Step 2: Initiate Retrieval
```bash
# Standard retrieval (3-5 hours)
aws s3api restore-object \
  --bucket watm-archived-data \
  --key device_checkins/2025/12/device_checkins_2025-12-03.sql.gz \
  --restore-request '{"Days":7,"GlacierJobParameters":{"Tier":"Standard"}}'

# Expedited retrieval (1-5 minutes) - costs more
aws s3api restore-object \
  --bucket watm-archived-data \
  --key device_checkins/2025/12/device_checkins_2025-12-03.sql.gz \
  --restore-request '{"Days":7,"GlacierJobParameters":{"Tier":"Expedited"}}'
```

#### Step 3: Download & Restore
```bash
# After retrieval completes, download
aws s3 cp s3://watm-archived-data/device_checkins/2025/12/device_checkins_2025-12-03.sql.gz /tmp/

# Decompress and restore to database
gunzip /tmp/device_checkins_2025-12-03.sql.gz
mysql -h $DB_HOST -u $DB_USER -p$DB_PASS $DB_NAME < /tmp/device_checkins_2025-12-03.sql

# Or restore to separate analysis database
mysql -h $DB_HOST -u $DB_USER -p$DB_PASS analysis_db < /tmp/device_checkins_2025-12-03.sql
```

---

### Archive Database Tracking Table

Create a table to track all archives:

```sql
CREATE TABLE data_archive_log (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL,
    archive_date DATETIME NOT NULL,
    date_range_start DATE NOT NULL,
    date_range_end DATE NOT NULL,
    record_count BIGINT NOT NULL,
    file_size_bytes BIGINT NOT NULL,
    compression_type VARCHAR(20) DEFAULT 'gzip',
    glacier_storage_class VARCHAR(50) DEFAULT 'GLACIER',
    glacier_archive_id VARCHAR(255) NULL,
    s3_bucket VARCHAR(100) NOT NULL,
    s3_key VARCHAR(500) NOT NULL,
    checksum VARCHAR(64) NOT NULL COMMENT 'SHA256 checksum',
    status VARCHAR(20) DEFAULT 'completed' COMMENT 'pending, completed, failed',
    deleted_from_production TINYINT(1) DEFAULT 0,
    deletion_completed_at DATETIME NULL,
    executed_by VARCHAR(100) NOT NULL,
    execution_duration_seconds INT NULL,
    notes TEXT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_table_date (table_name, archive_date),
    INDEX idx_date_range (date_range_start, date_range_end),
    INDEX idx_status (status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### Backup Validation & Testing

#### Monthly Backup Verification
```bash
# Test archive integrity monthly
# 1. Select random archive from previous month
# 2. Initiate retrieval
# 3. Download and verify checksum
# 4. Attempt to decompress and restore to test database
# 5. Validate record count matches manifest
# 6. Log results
```

#### Quarterly Disaster Recovery Test
```
1. Simulate data loss scenario
2. Retrieve archives from Glacier
3. Restore to new database instance
4. Validate data integrity
5. Document process and timing
6. Update disaster recovery procedures
```

---

### Safety Mechanisms & Safeguards

#### 1. **Pre-Deletion Checks**
```bash
# Before any deletion:
✓ Verify backup file exists
✓ Verify backup uploaded to Glacier
✓ Verify checksum matches
✓ Verify manifest file created
✓ Verify record count matches expected
✓ Verify no active database transactions
```

#### 2. **Execution Locks**
```bash
# Prevent simultaneous executions
if [ -f /tmp/data_purging.lock ]; then
  echo "Purging already in progress. Aborting."
  exit 1
fi

touch /tmp/data_purging.lock
trap "rm -f /tmp/data_purging.lock" EXIT
```

#### 3. **Rollback Capabilities**
```bash
# If deletion fails mid-process:
# - Stop immediately
# - Don't delete remaining records
# - Alert operations team
# - Keep archive available for investigation
# - Manual review required before retry
```

#### 4. **Monitoring & Alerts**
```yaml
CloudWatch Alarms:
  - Purging duration exceeds expected time
  - Deletion rate too slow (possible database issues)
  - Archive upload fails
  - Checksum verification fails
  - Any error during execution

Notifications:
  - Email: devops@orases.com
  - Slack: #watm-operations
  - PagerDuty: On-call engineer (if critical failure)
```

---

## Implementation Roadmap

### Phase 1: Foundation (Weeks 1-2)
**Goal:** Build and test core purging system

**Tasks:**
- [ ] Review existing `data_archival.sh` script in `watm/config/ArchiveData/src/`
- [ ] Enhance script to accept variable inputs (table name, retention days)
- [ ] Implement configuration file (`purging_policies.yaml`)
- [ ] Create `data_purging_policies` database table
- [ ] Implement locking mechanism
- [ ] Create `data_archive_log` tracking table
- [ ] Test on development environment with sample data

**Deliverables:**
- Enhanced purging script with variable inputs
- Configuration system (file + database)
- Tracking tables created
- Development testing complete

---

### Phase 2: Testing & Validation (Weeks 3-4)
**Goal:** Validate safety and effectiveness

**Tasks:**
- [ ] Test with small production dataset (dry-run mode)
- [ ] Archive 1 month of `device_checkins` data (test run)
- [ ] Verify archive integrity and retrieval process
- [ ] Measure performance improvement (query speed before/after)
- [ ] Document any issues or edge cases
- [ ] Test batch deletion (ensure no table locking)
- [ ] Validate monitoring and alerting

**Deliverables:**
- Test results documented
- Performance metrics collected
- Archive retrieval tested and validated
- Issue resolution plan

---

### Phase 3: Production Rollout (Weeks 5-6)
**Goal:** Purge historical data at scale

**Tasks:**
- [ ] Schedule production maintenance window
- [ ] Run full purge for `device_checkins` (90-day retention)
- [ ] Run full purge for `device_status_logs` (90-day retention)
- [ ] Run full purge for `provider_data_usages` (60-day retention)
- [ ] Monitor database performance during execution
- [ ] Validate archives created successfully
- [ ] Measure database size reduction
- [ ] Measure query performance improvement

**Deliverables:**
- Production data purged
- Performance improvements documented
- Archive integrity verified
- Success metrics reported

---

### Phase 4: Automation (Weeks 7-8)
**Goal:** Implement automated recurring purging

**Tasks:**
- [ ] Create cron job for monthly execution
- [ ] Implement wrapper script with pre/post checks
- [ ] Configure CloudWatch alarms
- [ ] Set up email/Slack notifications
- [ ] Create admin dashboard widget (optional)
- [ ] Document operational procedures
- [ ] Train operations team

**Deliverables:**
- Automated monthly purging active
- Monitoring and alerting configured
- Operations runbook created
- Team training completed

---

### Phase 5: Optimization & Expansion (Ongoing)
**Goal:** Improve and expand system

**Tasks:**
- [ ] Monitor performance over 3 months
- [ ] Identify additional tables for purging
- [ ] Optimize retention policies based on usage patterns
- [ ] Implement admin UI for configuration management (optional)
- [ ] Add additional archive storage classes if needed
- [ ] Regular quarterly backup verification tests

**Deliverables:**
- Performance reports
- Additional tables added as needed
- Optimized retention policies
- Ongoing system improvements

---

## Success Metrics

### Key Performance Indicators (KPIs)

#### Database Performance
- **Query Response Time:** Target 80% reduction for large table queries
- **Dashboard Load Time:** Target 50% reduction
- **Report Generation:** Target 60% reduction in generation time

#### Cost Savings
- **RDS Storage Costs:** Target 70-80% reduction
- **Backup Costs:** Target 50% reduction
- **Total Infrastructure:** Target $5,000-6,000 annual savings

#### Operational Efficiency
- **Database Size:** Target 70-80% reduction after initial purge
- **Backup Duration:** Target 60% reduction
- **Maintenance Windows:** Target 50% reduction

#### Data Management
- **Archive Success Rate:** Target 100% (no data loss)
- **Retrieval Capability:** Target 100% (all archives retrievable)
- **Execution Reliability:** Target 99%+ automated execution success rate

---

## Risks & Mitigation

| Risk | Impact | Likelihood | Mitigation Strategy |
|------|--------|------------|---------------------|
| Data loss during purge | **CRITICAL** | Very Low | Backup-first approach, verification before deletion, extensive testing |
| Database performance degradation during purge | High | Low | Batch deletion, run during off-peak hours, monitoring |
| Archive retrieval failure | High | Very Low | Regular backup verification tests, redundant storage |
| Script failure mid-execution | Medium | Low | Locking mechanism, rollback procedures, alerts |
| Incorrect retention policy | Medium | Medium | Start conservative (90 days), adjust based on usage analysis |
| Cost overruns | Low | Very Low | Glacier costs are minimal; calculate before implementation |

---

## Conclusion

This data purging strategy provides a scalable, safe, and cost-effective solution to manage WATM's growing database. By implementing configurable retention policies, automated archival, and safe batch deletion, we can:

1. **Improve performance** - Faster queries, faster dashboards, better user experience
2. **Reduce costs** - Significant savings on RDS storage and infrastructure
3. **Maintain compliance** - All data preserved in secure long-term storage
4. **Enable growth** - Remove scaling limitations caused by database bloat
5. **Operational efficiency** - Automated maintenance, reduced manual overhead

**Next Steps:**
1. Review and approve this strategy document
2. Begin Phase 1 implementation
3. Schedule initial testing window
4. Plan production rollout

**Questions or Concerns?** Contact DevOps team or schedule review meeting.
