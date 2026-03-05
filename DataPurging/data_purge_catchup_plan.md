# Data Purge Catch-Up Plan

**Goal:** Get all three tables purged up to their retention cutoffs within ~1 week using overnight cron jobs.

**Created:** 2026-02-25
**Updated:** 2026-02-26 (with real 30-day chunk numbers)
**Status:** Ready to deploy

---

## Current State (as of 2026-02-26 morning)

| Table | Now starts at | Retention cutoff | Gap (days) | Est. total remaining |
|---|---|---|---|---|
| `device_checkins` | 2023-09-09 | 2025-11-28 | ~811 | ~430M |
| `device_status_logs` | 2023-12-31 | 2025-11-28 | ~697 | ~386M |
| `provider_data_usages` | 2024-10-23 | 2025-12-28 | ~431 | ~265M |

**Total estimated remaining: ~1.08 billion rows**

---

## Confirmed Performance (30-day chunks, real production data)

| Table | Records | Export | Delete | Total | Notes |
|---|---|---|---|---|---|
| `device_checkins` | 15.9M | 17.6m | 19.1m | 37m 25s | Includes ~14m trailing scan tax (shrinking each run) |
| `device_status_logs` | 16.8M | 6.2m | 16.1m | 22m 22s | Clean linear scaling |
| `provider_data_usages` | 21.0M | 2.5m | 31.5m | 33m 47s | Clean linear scaling |

**Full `--all` run: ~93 minutes** (all three tables sequential).

DB load stayed at 25-44% throughout — well within safe limits.

The `device_checkins` trailing scan is caused by high-ID rows with old `created` dates. This tax is shrinking with each run as stragglers get purged and the table gets smaller.

---

## Strategy: 30-Day Chunks, 4 Runs Per Night

### Runs needed:

| Table | Gap (days) | Runs to catch up |
|---|---|---|
| `device_checkins` | 811 | ~27 |
| `device_status_logs` | 697 | ~23 |
| `provider_data_usages` | 431 | ~15 |

**27 runs** total (shorter tables report "no purgeable records" once caught up).

At **4 runs/night = ~7 nights** to clear the backlog.

---

## Off-Peak Window Analysis

Server runs UTC. Off-peak target: **11 PM - 5 AM ET = 4:00 - 10:00 UTC**.

### Existing jobs in this window (UTC):

| UTC Time | ET Time | Job | Concern |
|---|---|---|---|
| 4:00 Wed | 11:00 PM | `pm2 flush` | Trivial, no conflict |
| 5:15 daily | 12:15 AM | `detect_data_spike` | DB reads, avoid overlap |
| 5:30 daily | 12:30 AM | `aggregate_daily_usage` | DB-heavy, avoid overlap |
| 8:00 11th | 3:00 AM | `get_aggregate_verizon --prev` | Monthly only, minor |
| 8:15 11th | 3:15 AM | `activate_data_suspended` | Monthly only, minor |

The `*/5` and `*/10` minute worker jobs run 24/7 but are lightweight queue workers - no conflict.

### Catch-up schedule: 4 runs, ~90 min spacing

| Cron (UTC) | ET Time | Notes |
|---|---|---|
| `0 4 * * *` | 11:00 PM | Run 1 - starts the night, finishes ~5:30 UTC |
| `0 6 * * *` | 1:00 AM | Run 2 - after aggregate_daily_usage (5:30) and detect_data_spike (5:15) finish |
| `30 7 * * *` | 2:30 AM | Run 3 - 90 min gap from Run 2 |
| `30 9 * * *` | 4:30 AM | Run 4 - finishes ~11:00 UTC (6:00 AM ET) at worst |

`run-one` prevents any overlap. If Run 1 is still going at 6:00 UTC, Run 2 simply skips.

---

## Cron Setup

### Phase 1: Catch-up mode

Replace the current single test run with these 4 lines:

```crontab
# Data purge catch-up - 4 runs per night during off-peak (11pm-5am ET)
# run-one prevents overlapping runs. Expect ~7 nights to clear backlog.
0 4 * * * run-one /var/www/watm/bin/cake data_purge --all --chunk-days 30 --force >> /var/www/watm/logs/purge_catchup.log 2>&1
0 6 * * * run-one /var/www/watm/bin/cake data_purge --all --chunk-days 30 --force >> /var/www/watm/logs/purge_catchup.log 2>&1
30 7 * * * run-one /var/www/watm/bin/cake data_purge --all --chunk-days 30 --force >> /var/www/watm/logs/purge_catchup.log 2>&1
30 9 * * * run-one /var/www/watm/bin/cake data_purge --all --chunk-days 30 --force >> /var/www/watm/logs/purge_catchup.log 2>&1
```

### Phase 2: Maintenance mode (once caught up)

Replace the 4 lines with a single weekly run:

```crontab
# Data purge - weekly maintenance (Sunday 1am ET / 6am UTC)
0 6 * * 0 run-one /var/www/watm/bin/cake data_purge --all --chunk-days 30 --force >> /var/www/watm/logs/purge.log 2>&1
```

At ~615K rows/day max, 7 days = ~4.3M rows - trivial single run.

---

## Expected Nightly Progress

Each night clears ~120 days (4 runs x 30-day chunks) per table.

Assuming all 4 runs fire (runs take ~93 min, spacing is 90 min, so occasional skips possible via run-one):

| Night | device_checkins | device_status_logs | provider_data_usages |
|---|---|---|---|
| 1 | Sep 9 - Jan 8, 2024 | Dec 31 - Apr 30, 2024 | Oct 23, 2024 - Feb 20, 2025 |
| 2 | Jan 8 - May 8, 2024 | Apr 30 - Aug 28, 2024 | Feb 20 - Jun 21, 2025 |
| 3 | May 8 - Sep 5, 2024 | Aug 28 - Dec 27, 2024 | Jun 21 - Oct 19, 2025 |
| 4 | Sep 5 - Jan 4, 2025 | Dec 27 - **caught up** | Oct 19 - **caught up** |
| 5 | Jan 4 - May 5, 2025 | - | - |
| 6 | May 5 - Sep 2, 2025 | - | - |
| 7 | Sep 2 - **caught up** | - | - |

**provider_data_usages**: done in ~4 nights.
**device_status_logs**: done in ~4 nights.
**device_checkins**: done in ~7 nights.

Once the shorter tables finish, those runs get faster (just checkins + "no purgeable" for the others), so the later nights might squeeze in extra runs.

---

## Monitoring During Catch-Up

### Morning check query

```sql
SELECT id, table_name, date_range_start, date_range_end,
  FORMAT(record_count, 0) as records,
  ROUND(file_size_bytes / 1048576, 1) as size_mb,
  ROUND(execution_time_seconds / 60, 1) as minutes,
  status
FROM data_archive_logs
WHERE created >= CURDATE() - INTERVAL 1 DAY
ORDER BY id;
```

### Other checks

```bash
# How many runs completed overnight?
grep -c "Purge Summary" /var/www/watm/logs/purge_catchup.log

# Any failures?
mysql watm_db -e "SELECT * FROM data_archive_logs WHERE status = 'failed' ORDER BY id DESC LIMIT 5;"

# Check DB load from overnight in RDS Performance Insights console
```

### Warning signs

- **DB load spiked > 70% overnight**: Drop to 3 runs (remove the 9:30 UTC slot)
- **Runs consistently overlapping** (run-one skipping): Increase spacing or drop a slot
- **A run failed mid-delete**: Check `data_archive_logs`, follow recovery in `data_purge_production_runbook.md`
- **device_checkins scan time growing instead of shrinking**: Investigate the ID distribution

### If things go sideways

```bash
# Comment out all 4 catch-up crons
crontab -e

# Check for stuck archive logs
mysql watm_db -e "SELECT * FROM data_archive_logs WHERE status IN ('pending', 'archiving', 'deleting');"

# See recovery procedures in data_purge_production_runbook.md
```

---

## Known Quirk: Straggler Records

On the first overnight run, `device_checkins` wasted an iteration purging a single record from 2023-02-20 - a high-ID row with an old `created` date. These stragglers cause the chunk-days logic to find a sparse date range and purge almost nothing.

This appears to be rare (only happened once so far). If it becomes a recurring problem, we may need to add a minimum record count threshold that skips sparse chunks and advances to the next dense date range.

---

## Appendix: Why 30-Day Chunks?

Smaller chunks (7-15 days) waste the device_checkins scan tax on fewer rows.

Bigger chunks (60-90 days) would mean:
- 500MB+ archive files for device_checkins
- 30+ minute delete phases per table
- Longer failure window if something goes wrong

30 days is the sweet spot: amortizes the scan tax, keeps individual runs under 40 min per table, and archive files stay under 600 MB.

If runs prove rock-solid after a few nights, scaling to 45 days would shave a night or two off the timeline.
