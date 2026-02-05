# Data Purging Strategy - Technical Review & Feedback

**Reviewer:** Technical Lead Review
**Date:** January 22, 2026
**Documents Reviewed:**
- PRD: Data Purging Strategy PRD
- TRD: Data Purging Strategy - Technical Requirements Document

---

## Executive Summary

The technical approach is **generally solid** with good attention to prerequisites (indexes) and FK safety. However, there are **critical gaps** in error handling, data consistency verification, and operational procedures that must be addressed before implementation.

**Multi-tenancy concern resolved:** All companies use uniform 90-day retention with no exceptions.

**Recommendation:** Address remaining critical issues before proceeding to development.

---

## Critical Issues (Must Fix)

### 1. Foreign Key Safety - Incomplete Solution

**Issue:** The TRD identifies FK relationships but the proposed solution has a **race condition**.

```php
// TRD proposes:
$referencedIds = $this->Devices->find()
    ->select(['device_checkin_id'])
    ->toArray();

$deleted = $this->DeviceCheckins->deleteAll([
    'created <' => $cutoffDate,
    'id NOT IN' => $referencedIds
]);
```

**Problem:** Between reading `$referencedIds` and executing the DELETE, a device could update its `device_checkin_id` to point to a record being deleted.

**Questions:**
- How will you prevent this race condition?
- Should the purge run in a transaction with appropriate isolation level?
- Should you use `FOR UPDATE` locks on the devices table?
- What happens if a device check-in arrives during purge execution?

**Suggested Mitigation:**
```php
// Option A: Use LEFT JOIN to exclude at query time
DELETE dc FROM device_checkins dc
LEFT JOIN devices d ON dc.id = d.device_checkin_id
WHERE dc.created < :cutoff
  AND d.id IS NULL;

// Option B: Add safety buffer (don't purge most recent 100 days instead of 90)
// This gives you 10-day buffer against the race condition

// Option C: Temporarily lock the devices table (not recommended for production)
```

**Severity:** CRITICAL - Could cause FK constraint violations in production

---

### 2. Multi-Tenancy / Company Isolation - ✅ RESOLVED

**Issue:** The WATM system is **multi-tenant** with strict company-based data isolation. The TRD doesn't mention how purging respects company boundaries.

**Resolution:** Confirmed that **all companies must retain exactly 90 days of data with no exceptions**. This means:
- No per-company retention overrides needed
- No compliance variations between companies
- Global 90-day purge policy applies uniformly

**Validation:**
The TRD's approach correctly handles this by:
```sql
-- All records older than 90 days are purged regardless of company
DELETE FROM device_checkins
WHERE created < DATE_SUB(NOW(), INTERVAL 90 DAY)
  AND id NOT IN (SELECT device_checkin_id FROM devices WHERE device_checkin_id IS NOT NULL);
```

**Recommendation:** Document this as an explicit business rule in the TRD to prevent future confusion.

**Severity:** ~~CRITICAL~~ → **RESOLVED** (no changes needed, documentation only)

---

### 3. Data Consistency Verification - Missing

**Issue:** The TRD mentions "verify upload (checksum)" but doesn't specify **what** gets verified or **how**.

**Questions:**
- How do you verify the archive contains ALL records that should be deleted?
- What if mysqldump succeeds but skips rows due to a transient error?
- What if the archive file is corrupted during compression?
- What if Glacier upload succeeds but data is corrupted in transit?
- Do you verify row count matches between export and archive?

**Missing Verification Steps:**
```bash
# Before deletion, verify:
1. Row count in SQL dump matches database count
2. File integrity (SHA256 matches before/after upload)
3. Glacier archive ID is valid and retrievable
4. Spot-check: Can you retrieve and parse the archive?
5. Foreign key exclusions are correct
```

**Suggested Implementation:**
```php
// Step 1: Count records to be archived
$expectedCount = $this->DeviceCheckins->find()
    ->where(['created <' => $cutoffDate])
    ->count();

// Step 2: Export and count rows in dump
$actualCount = $this->countRowsInSqlDump($dumpFile);

// Step 3: Verify counts match
if ($expectedCount !== $actualCount) {
    throw new Exception("Row count mismatch: expected $expectedCount, got $actualCount");
}

// Step 4: Verify Glacier upload integrity
$uploadChecksum = hash_file('sha256', $dumpFile);
$glacierChecksum = $glacierClient->getArchiveChecksum($archiveId);
if ($uploadChecksum !== $glacierChecksum) {
    throw new Exception("Checksum mismatch - archive may be corrupted");
}

// Only then proceed to deletion
```

**Severity:** CRITICAL - Without this, you could delete data without valid backup

---

### 4. Rollback / Recovery Procedure - Missing

**Issue:** The TRD says "archive-before-delete" but doesn't define **how to recover** if something goes wrong.

**Questions:**
- What if you discover after deletion that the archive is corrupted?
- What if you accidentally purge data that shouldn't have been purged?
- What's the recovery SLA (how fast can you restore)?
- Who is authorized to request data restoration?
- What's the step-by-step procedure to restore from Glacier?

**Missing Documentation:**
```markdown
# Recovery Procedure

## Scenario 1: Archive Corruption Detected
1. Immediately halt any further purging
2. Check data_archive_logs for affected date ranges
3. Notify engineering team
4. Determine if data is still in production or already deleted
5. If deleted, restore from RDS snapshots (if within retention)

## Scenario 2: Accidental Over-Purge
1. Identify affected date range and table
2. Locate Glacier archive ID in data_archive_logs
3. Initiate Glacier retrieval (3-5 hours)
4. Download archive, decompress, restore via mysql
5. Verify restoration integrity
6. Update application to reflect restored data

## Scenario 3: Customer Requests Historical Data
1. Support team submits ticket with date range needed
2. Ops team queries data_archive_logs for archive ID
3. Initiate Glacier retrieval
4. Provide data to customer (CSV export or read-only access)
```

**Severity:** HIGH - Without recovery procedures, you're unprepared for failures

---

### 5. Partial Failure Handling - Not Defined

**Issue:** The TRD shows a status flow (`pending → archiving → archived → deleting → completed`) but doesn't explain **what happens on partial failure**.

**Failure Scenarios Not Addressed:**
- Glacier upload succeeds for 500MB, fails at 600MB (multipart upload)
- Delete completes for 80% of records, then MySQL crashes
- Archive completes, but disk fills up before deletion starts
- Network failure during Glacier upload
- EC2 instance terminated mid-purge

**Questions:**
- Is the purge operation idempotent (can you re-run safely)?
- How do you resume from partial failure?
- What if status is "archived" but deletion never happened?
- What if status is "deleting" but only 50% of records were deleted?

**Suggested Implementation:**
```php
// Make purge idempotent by tracking progress
CREATE TABLE data_purge_batches (
    id INT AUTO_INCREMENT PRIMARY KEY,
    archive_log_id INT,
    batch_number INT,
    start_id BIGINT,
    end_id BIGINT,
    records_deleted INT,
    status ENUM('pending', 'completed', 'failed'),
    deleted_at DATETIME
);

// On resume, skip completed batches
$completedBatches = $this->DataPurgeBatches->find()
    ->where(['archive_log_id' => $archiveLogId, 'status' => 'completed'])
    ->extract('batch_number')
    ->toArray();

// Only delete batches not yet completed
foreach ($batches as $batchNumber => $batch) {
    if (in_array($batchNumber, $completedBatches)) {
        continue; // Already done
    }
    $this->deleteBatch($batch);
    $this->markBatchCompleted($batchNumber);
}
```

**Severity:** HIGH - Production systems WILL fail; you need graceful handling

---

## High Priority Issues (Should Fix)

### 6. Index Creation Timing Estimates - Overly Optimistic

**Issue:** TRD estimates `device_checkins` index creation at "30-60 minutes" but acknowledges "worst case 2-4 hours" in a footnote.

**Reality Check:**
- Current database size: ~800GB
- `device_checkins` table: Estimated 200-500M rows
- AWS RDS I/O contention during peak hours
- Index build on 500M rows on a loaded production database

**Questions:**
- What's the actual current row count in each table?
- Have you tested index creation on a production snapshot?
- What's the RDS instance type and IOPS limit?
- Will you need to scale up the instance temporarily?

**Suggested Approach:**
```bash
# Phase 0A: Test on production snapshot
1. Create RDS snapshot
2. Restore to separate instance
3. Test index creation, measure actual time
4. Measure I/O impact on replica lag

# Phase 0B: Production rollout
1. Schedule during absolute lowest traffic (3am Sunday)
2. Increase RDS IOPS temporarily if needed
3. Monitor replica lag closely
4. Have rollback plan if lag becomes critical
```

**Severity:** HIGH - Underestimating could cause unexpected production impact

---

### 7. Scheduling Conflict - Weekly vs. Monthly Inconsistency

**Issue:** PRD recommends "Monthly" schedule, TRD recommends "Weekly" schedule. Documents contradict each other.

**PRD Says:**
> Automated Monthly: 1st Sunday @ 2 AM

**TRD Says:**
> Weekly cron job... Why Weekly (Not Monthly): With 90-day retention and monthly runs, you'd actually have 90-120 days

**Questions:**
- Which is correct?
- Has this been discussed and decided?
- What's the tradeoff analysis?

**Analysis:**
| Aspect | Weekly | Monthly |
|--------|--------|---------|
| Retention accuracy | 90-97 days | 90-120 days |
| Batch size | Smaller (1 week of data) | Larger (1 month of data) |
| Performance impact | More frequent, shorter runs | Less frequent, longer runs |
| Operational overhead | 4x more executions | Simpler |

**Recommendation:** Start with **monthly** (as PRD suggests), then move to weekly if retention accuracy becomes critical.

**Severity:** MEDIUM - Needs alignment between PRD and TRD

---

### 8. Batch Size - No Testing Plan

**Issue:** TRD proposes `batch_size: 5000` but PRD proposes `batch_size: 1000`. No rationale or testing plan.

**Questions:**
- What's the actual optimal batch size?
- Have you tested on production to measure lock contention?
- What's the tradeoff between speed and lock duration?
- Does batch size vary by table (e.g., smaller for `device_checkins`)?

**Suggested Testing:**
```bash
# Test on production replica
for batch_size in 500 1000 2000 5000 10000; do
    mysql -e "DELETE FROM device_checkins
              WHERE created < DATE_SUB(NOW(), INTERVAL 100 DAY)
              LIMIT $batch_size"
    # Measure:
    # - Execution time
    # - Lock wait time (SHOW ENGINE INNODB STATUS)
    # - Impact on concurrent queries
done
```

**Severity:** MEDIUM - Incorrect batch size could cause production issues

---

### 9. Glacier Storage Class - Cost vs. Retrieval Tradeoff Not Analyzed

**Issue:** PRD recommends "Glacier Flexible Retrieval" but doesn't justify vs. alternatives with actual cost analysis.

**Missing Analysis:**

| Storage Class | Storage Cost | Retrieval Cost | Retrieval Time | 2-Year Total Cost (500GB) |
|--------------|-------------|---------------|----------------|--------------------------|
| S3 Standard | $0.023/GB/mo | $0 | Instant | **$276/year** |
| S3 Infrequent Access | $0.0125/GB/mo | $0.01/GB | Instant | $150/year |
| Glacier Flexible | $0.004/GB/mo | $0.03/GB + $0.01/1000 requests | 3-5 hours | **$48/year** |
| Glacier Deep Archive | $0.00099/GB/mo | $0.02/GB + $0.025/1000 requests | 12-48 hours | **$12/year** |

**Questions:**
- How often do you expect to retrieve archives? (Once a year? Never?)
- If retrieval is rare, why not Deep Archive for 75% additional savings?
- Have you considered S3 Infrequent Access with lifecycle transition?

**Suggested Strategy:**
```yaml
# Hybrid approach
Recent archives (< 1 year): S3 Infrequent Access (faster retrieval)
Old archives (> 1 year): Glacier Deep Archive (cheapest)
```

**Severity:** MEDIUM - Cost optimization opportunity

---

### 10. Monitoring and Alerting - Insufficient Detail

**Issue:** TRD mentions "CloudWatch alarms" but doesn't specify **what** to monitor or alert on.

**Missing Monitoring:**
```yaml
CloudWatch Alarms:
  - Purge job hasn't run in 10 days (scheduled job failure)
  - Archive upload duration > 2 hours (performance degradation)
  - Deletion batch failure rate > 5% (database issues)
  - Disk space < 20% during purge (out of disk risk)
  - Glacier upload checksum mismatch (data corruption)

Custom Metrics:
  - Records archived per run
  - Records deleted per run
  - Archive file size
  - Execution duration
  - Batch deletion time

Notifications:
  Success:
    - Email summary (daily digest, not per-run)
    - Slack notification (optional)

  Failure:
    - Immediate PagerDuty alert
    - Email to ops team
    - Slack #watm-alerts channel
```

**Severity:** MEDIUM - Production reliability depends on proper monitoring

---

## Medium Priority Issues (Nice to Have)

### 11. Testing Strategy - Underdeveloped

**Issue:** TRD estimates "2 sprints" but doesn't detail testing approach.

**Missing Testing Plan:**
```markdown
# Unit Tests
- [ ] FK exclusion query correctness
- [ ] Batch deletion logic
- [ ] Checksum verification
- [ ] Status transitions
- [ ] Error handling

# Integration Tests
- [ ] Full purge cycle on test database
- [ ] Archive upload and retrieval
- [ ] Partial failure recovery
- [ ] Concurrent purge prevention (lock file)

# Load Tests
- [ ] Purge 1M records, measure impact
- [ ] Test on production-sized dataset
- [ ] Measure query performance during purge

# Compliance Tests
- [ ] Verify company retention overrides work
- [ ] Verify FK-referenced records aren't deleted
- [ ] Verify archive integrity after retrieval
```

**Severity:** MEDIUM - Inadequate testing could cause production issues

---

### 12. Performance Impact During Purge - Not Measured

**Issue:** TRD claims "no portal downtime" but doesn't quantify performance impact during purge execution.

**Questions:**
- What's the expected query latency increase during batch deletion?
- Will users notice slowdowns during the 2-hour purge window?
- Should you reduce batch size during peak hours?
- Should you pause purging if query latency exceeds threshold?

**Suggested Monitoring:**
```php
// Adaptive batch sizing
while ($recordsToDelete > 0) {
    $currentLoad = $this->measureDatabaseLoad();

    if ($currentLoad > 0.8) {
        // Reduce batch size or pause
        $batchSize = 1000;
        sleep(10);
    } else {
        $batchSize = 5000;
    }

    $this->deleteBatch($batchSize);
}
```

**Severity:** MEDIUM - User experience consideration

---

### 13. Dry-Run Output - Not Specified

**Issue:** TRD mentions `--dry-run` mode but doesn't show **what the output looks like**.

**Suggested Output:**
```bash
$ bin/cake data_purge --table=device_checkins --dry-run

DRY RUN MODE - No changes will be made
================================================

Table: device_checkins
Cutoff Date: 2025-10-24 (90 days ago)

Analysis:
  Total records in table: 487,293,581
  Records older than cutoff: 312,442,198
  Records referenced by devices: 43,291 (will be excluded)
  Records to be archived: 312,398,907
  Estimated archive size: 142 GB (compressed)
  Estimated batches: 62,480 (at 5000 per batch)
  Estimated execution time: ~2.5 hours

Foreign Key Safety Check:
  ✓ 43,291 records excluded (currently referenced by active devices)
  ✓ No FK constraint violations expected

Disk Space Check:
  ✓ Available disk space: 500 GB
  ✓ Required for archive: 142 GB
  ✓ Sufficient space available

Would archive to:
  s3://watm-archived-data/device_checkins/2026/01/device_checkins_2026-01-22.sql.gz

Proceed with actual purge? (Use --force to skip this prompt)
```

**Severity:** LOW - Quality of life improvement

---

### 14. Execution Duration Estimates - Missing

**Issue:** TRD says "complete within 2-hour window" but doesn't show how this was calculated.

**Missing Analysis:**
```bash
# Purge Time Calculation

Table: device_checkins
Records to delete: 312M records
Batch size: 5000 records
Batches: 62,400 batches

Time per batch:
  - DELETE query: ~50ms (estimated)
  - Sleep between batches: 100ms
  - Total: 150ms per batch

Total deletion time:
  62,400 batches × 150ms = 9,360 seconds = 2.6 hours

Archive upload time:
  File size: 142 GB
  Upload speed: 100 Mbps (12.5 MB/s)
  Time: 142,000 MB ÷ 12.5 MB/s = 11,360 seconds = 3.2 hours

Total estimated time: 5.8 hours (exceeds 2-hour window!)
```

**Questions:**
- Is the 2-hour window realistic?
- Should archival and deletion run in parallel?
- Should you increase batch size to meet the window?

**Severity:** MEDIUM - Unrealistic estimates could cause operational issues

---

### 15. Glacier Retrieval Procedure - Missing

**Issue:** TRD says "3-5 hour restore time" but doesn't document the actual procedure.

**Missing Documentation:**
```bash
# Glacier Data Retrieval Procedure

## Step 1: Locate Archive
mysql> SELECT glacier_archive_id, glacier_vault_name, file_size_bytes
       FROM data_archive_logs
       WHERE table_name = 'device_checkins'
         AND date_range_start <= '2025-06-15'
         AND date_range_end >= '2025-06-15';

## Step 2: Initiate Retrieval
aws glacier initiate-job \
  --vault-name watm-archived-data \
  --account-id - \
  --job-parameters '{
    "Type": "archive-retrieval",
    "ArchiveId": "abc123...",
    "Tier": "Standard"
  }'

## Step 3: Wait for Completion (3-5 hours)
aws glacier describe-job \
  --vault-name watm-archived-data \
  --account-id - \
  --job-id xyz789

## Step 4: Download Archive
aws glacier get-job-output \
  --vault-name watm-archived-data \
  --account-id - \
  --job-id xyz789 \
  output.sql.gz

## Step 5: Restore Data
gunzip output.sql.gz
mysql watm_db < output.sql

## Step 6: Verify Restoration
mysql> SELECT COUNT(*) FROM device_checkins
       WHERE created BETWEEN '2025-06-01' AND '2025-06-30';
```

**Severity:** LOW - Operational documentation gap

---

## Questions for Engineering Team

### Architecture & Design

1. **Configuration Management:** The PRD asks "YAML vs database table" but the TRD chooses hardcoded table-specific logic. Why this decision? Does it make sense to skip configuration entirely?

2. **CakePHP Command vs Shell Script:** The TRD chooses CakePHP Command. Does this require the app to be deployed to the same server where purging runs? What if you want to run purging from a separate maintenance server?

3. **Transaction Isolation:** Should the purge run in a transaction? What isolation level? REPEATABLE READ to prevent FK race conditions?

4. **Glacier vs S3 Lifecycle:** Why use Glacier API directly? Why not upload to S3 and use lifecycle policies to transition to Glacier? (Simpler, more flexible)

### Data & Compliance

5. **Multi-Tenant Retention:** Do all companies have the same retention requirements? Are there contractual obligations we're missing?

6. **Regulatory Compliance:** Are there GDPR, CCPA, HIPAA, or other regulations that affect retention policies?

7. **Audit Trail:** Is `data_archive_logs` sufficient for audit purposes? Do we need immutable logs?

8. **Data Ownership:** Who owns archived data? Can customers request their archived data? What's the SLA?

### Operations & Scheduling

9. **Scheduling Frequency:** Monthly (PRD) or Weekly (TRD)? Which is final?

10. **Execution Window:** Is 2am Sunday truly the lowest-traffic time? Have you analyzed actual traffic patterns?

11. **Conflict Avoidance:** TRD mentions "avoids 2am Verizon job" - what other scheduled jobs could conflict?

12. **Lock Mechanism:** How do you prevent simultaneous executions? File lock? Database lock? What if lock file isn't cleaned up after a crash?

### Error Handling & Recovery

13. **Partial Failure:** What happens if purging fails halfway through? Is it idempotent? Can you resume?

14. **Archive Corruption:** What if you discover archive corruption **after** deletion? What's the recovery procedure?

15. **FK Constraint Violations:** What if the FK exclusion logic has a bug and deletion fails with FK constraint violation? How do you recover?

16. **Disk Space:** What if disk space runs out during export? Does the script check available space before starting?

### Testing & Validation

17. **Production Testing:** How will you validate the purge worked correctly without breaking production?

18. **Rollback Plan:** If the first production purge causes issues, what's the rollback plan?

19. **Performance Testing:** Have you load-tested batch deletion on production-sized data?

20. **Index Testing:** Have you tested index creation on a production snapshot to verify timing estimates?

---

## Recommendations

### Before Development Starts

- [ ] **Align PRD and TRD** - Resolve contradictions (monthly vs weekly, batch size 1000 vs 5000)
- [ ] **Investigate multi-tenancy** - Verify no companies have special retention requirements
- [ ] **Test index creation** - Run on production snapshot to validate timing estimates
- [ ] **Define recovery procedures** - Document step-by-step Glacier retrieval process
- [ ] **Specify monitoring** - Define exact CloudWatch metrics and alarms

### During Development

- [ ] **Fix FK race condition** - Use LEFT JOIN or transaction locking
- [ ] **Add verification** - Row count matching, checksum validation
- [ ] **Make idempotent** - Support resuming from partial failure
- [ ] **Add dry-run output** - Clear, detailed preview of what will be purged
- [ ] **Document error handling** - What happens on each failure mode

### Before Production Deployment

- [ ] **Create runbook** - Step-by-step operational procedures
- [ ] **Test on snapshot** - Full purge cycle on production data copy
- [ ] **Validate monitoring** - Ensure alerts trigger correctly
- [ ] **Train ops team** - How to run, monitor, and recover from failures
- [ ] **Get compliance approval** - Legal/compliance sign-off on retention policies

---

## Overall Assessment

| Aspect | Rating | Notes |
|--------|--------|-------|
| **Index Prerequisites** | ✅ Excellent | Well thought out, addresses root cause |
| **FK Safety** | ⚠️ Needs Work | Race condition, needs transaction handling |
| **Multi-Tenancy** | ❌ Missing | Critical gap for WATM's multi-tenant architecture |
| **Verification** | ⚠️ Needs Work | Checksum mentioned but not detailed |
| **Error Handling** | ❌ Missing | Partial failure scenarios not addressed |
| **Recovery Procedures** | ❌ Missing | No documented restoration process |
| **Testing Strategy** | ⚠️ Needs Work | Underdeveloped, needs more detail |
| **Monitoring** | ⚠️ Needs Work | High-level only, needs specific metrics |
| **Documentation** | ⚠️ Needs Work | Operational runbook missing |

**Final Recommendation:** Address critical issues (FK safety, multi-tenancy, verification, error handling) before starting development. The core approach is sound, but implementation details need refinement.

---

## Next Steps

1. **Schedule alignment meeting** with product, engineering, and ops teams to resolve open questions
2. **Revise TRD** to address critical issues identified in this review
3. **Create testing plan** with specific test cases and acceptance criteria
4. **Get compliance review** to validate retention policies
5. **Build operational runbook** for deployment and ongoing maintenance

**Target:** Revised TRD ready for development kickoff within 1 week.
