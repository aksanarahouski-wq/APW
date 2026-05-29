# Admin Logs - Decision Summary
**Date:** March 5, 2026
**Participants:** Devon D'Andrea, Adam Curcie, Aksana Rahouski, Richard Sacco

---

## Problem Statement

The `/admin/logging` interface is cluttered with **pages and pages** of system-generated logs, specifically "Device Configuration Log Id changed" entries, making it:
- Difficult to find useful information
- Slow to load
- Hard to identify user-initiated actions
- Frustrating to navigate

### Root Cause

**Technical Issue:**
- Every time a device receives a configuration update, the system automatically updates `device_configuration_log_id` field on the device
- This field change triggers an audit log entry: "Device Configuration Log Id changed from X to Y"
- The field is NOT in the exclusion list in `DevicesTable.php` (unlike similar fields like `device_status_log_id` which ARE excluded)
- For active devices, this happens multiple times per day (every 2 hours in some cases)

**Result:** Thousands of useless audit log entries that provide no actionable value

---

## What Was Agreed Upon

### Main Agreement: Split Logs into Two Categories

**Decision:** Separate system-generated logs from user-triggered logs

**Reasoning:**
- Devon: "Honestly, it's more so user-initiated things [we need to see]"
- Adam: "90% of the time we're gonna click browse user logs"
- Adam: "I feel like there has been instances where [system logs] were useful" - can't completely remove them
- Removing data points entirely "goes against everything I believe in" (Devon)

**What This Solves:**
1. **Performance:** User logs page will load faster (less data to query)
2. **Clarity:** User-initiated actions won't be buried in system noise
3. **Flexibility:** System logs still accessible when needed for troubleshooting
4. **Audit value:** Preserves both types of information

---

## Solution Options Discussed

### ❌ Option 1: Leave As-Is
**Effort:** Zero
**Pros:** No work required
**Cons:** Problem continues, logs remain cluttered and slow
**Decision:** Rejected

---

### ⚠️ Option 2: Remove System-Generated Logs Entirely
**Effort:** Low - Simple exclusion in query
**Implementation:** Add `device_configuration_log_id` to exclusion list in `DevicesTable.php`

**Pros:**
- Fast to implement (one-line code change)
- Immediately reduces log clutter
- Improves page load performance
- No UI changes needed

**Cons:**
- Loses potentially useful troubleshooting data
- No way to see system-level changes when needed
- "Goes against everything I believe in" - Devon prefers not to lose data
- Adam recalls times when system logs were helpful for billing changes and following breadcrumbs

**Decision:** Not chosen, but this is the fallback if split approach is too much effort

---

### ✅ Option 3: Split into Two Log Views (CHOSEN)
**Effort:** Medium - Requires UI changes
**Implementation:**
1. Create two separate log browse pages/links:
   - **"Browse User Logs"** - user-initiated actions (90% use case)
   - **"Browse System Logs"** - system-generated events (10% use case, troubleshooting)
2. Filter logs based on log type/category

**Pros:**
- Solves the clutter problem (user logs are clean)
- Preserves all data (system logs still available when needed)
- Improves performance (each view has less data)
- Better user experience (focused on what they need)
- Flexible for future needs

**Cons:**
- More development effort than simple exclusion
- Requires UI changes (two links instead of one)
- Need to categorize existing logs

**Decision:** ✅ **Agreed upon solution**

**Priority:** Not high priority (can be scheduled after higher priority work)

---

## What User Logs Should Include

Based on the meeting discussion, **user logs** should include:

✅ **User-Initiated Actions:**
- Device assignments, status changes
- Configuration changes (via `configuration_id` field changes)
- Payment method changes, service plan changes
- Device creation, deletion
- Billing-related changes (billing active changed, pending changed)
- Device modifications made by users through the UI

❌ **System-Generated Logs to Move to "System Logs":**
- Device Configuration Log Id changed (the main problem)
- Device status changed from offline to null (device checked in)
- Hostname changed (via config push)
- TID changed (terminal ID updates)
- Password rotated (SysTech devices)
- Other internal tracking field updates

**Key Quote from Devon:** "I don't think I need that, because I already have a check-in log for a device. If I wanted to know when this device checked in, I have a check-ins export."

---

## Additional Context from Meeting

### Use Cases for Logs

**Primary Use Case (90%):**
- Tracking **who** did **what** to a device
- Investigating user-initiated changes
- Auditing administrative actions
- Following up on specific device modifications

**Secondary Use Case (10%):**
- Troubleshooting billing issues (following breadcrumbs)
- Understanding system behavior in edge cases
- Debugging device state changes
- Seeing behind-the-scenes processing

### Performance Issues Mentioned
- Current logs "take years to load at times" (Adam)
- Looking at last 100 log entries goes back only 15 minutes (Devon)
- Need to improve query performance and reduce data volume

---

## Implementation Recommendations

### Quick Win (If Split is Too Complex)
Fall back to **Option 2**: Add `device_configuration_log_id` to exclusion list
- File: `DevicesTable.php` line 1476
- Add to `$dirtyFilterArray`
- One-line change, immediately reduces noise by ~80%
- Safe: follows existing pattern (`device_status_log_id` already excluded)

### Preferred Solution
Implement **Option 3**: Split log views
1. Create "Browse User Logs" and "Browse System Logs" links in admin interface
2. Define log categorization rules (user-triggered vs system-generated)
3. Apply filters to each view
4. Test with real data to ensure correct categorization
5. Monitor performance improvements

### Optional Cleanup (After Implementation)
If client requests: Delete historical "Device Configuration Log Id changed" entries or archive to separate table/storage

---

## Risk Assessment

**Risk Level:** LOW

- Simple query filtering or UI split
- No data loss (system logs preserved)
- Easily reversible if needed
- High impact on user experience and performance
- Follows existing patterns in codebase

---

## Next Steps

1. ✅ Document decision (this file)
2. ⏳ Prioritize work (agreed: not high priority)
3. ⏳ Choose implementation approach:
   - Start with Option 3 (split) if resources allow
   - Fall back to Option 2 (exclude) if time constrained
4. ⏳ Create technical design and estimate effort
5. ⏳ Implement solution when prioritized
6. ⏳ Test with real data
7. ⏳ Deploy and monitor

---

## Key Quotes

**Devon on the problem:**
> "I was looking at 100 right now, and it goes back 15 minutes, right? So on this page, on the last 100 log entries..."

**Adam on the solution:**
> "I'm almost wondering if it would just be, mm, I don't know, logical to, like, split them."

**Adam on use cases:**
> "90% percent of the time we're gonna click browse user logs. And if it loads faster and the logs are less congested, I think that really solves the problem."

**Devon on preserving data:**
> "Removing data points goes against everything I believe in, but I just don't know if... I don't know."

**Final agreement:**
> "If we think that there's some benefit, whether however small it is to having it, then perhaps we just have you move forward with separating it out."

---

**Status:** Decision Made - Implementation Pending
**Priority:** Medium-Low (after higher priority work)
**Agreed Solution:** Split logs into "User Logs" and "System Logs"
