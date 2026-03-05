# T-Mobile Device Usage Issues - Ticket Summary

## Problem
T-Mobile devices are incorrectly showing **zero usage** in the system. This is a **two-fold issue**:

1. **API is being called incorrectly** - The system passes the billing cycle start date (11/10) instead of the current date
2. **False data is being stored** - For the past ~2 weeks since release, incorrect usage data has been accumulated

## Root Cause
When querying the T-Mobile API for device usage:
- **Current behavior**: System passes APW's billing cycle start date (11/10) as a parameter
- **Result**: API returns data from 10/19 to 11/19 every single day (same values repeatedly)
- **Impact**: Since the same cumulative value is retrieved daily, the difference calculation shows zero usage

**Key insight**: The T-Mobile API requires a billing cycle date parameter. The date passed determines which billing cycle data you receive. By passing a static historical date, we keep getting the same billing cycle window.

## Technical Details
- APW's billing cycle: 10th through 10th
- T-Mobile's billing cycle: 19th through 19th
- System stores cumulative usage values and calculates daily differences
- ~20 T-Mobile devices currently affected
- Postman collection exists at: Carrier APIs > T-Mobile > Usage endpoint

## Solution Required

### 1. Fix the Bug
**Change the API call to pass `today's date` instead of the billing cycle start date**
- This ensures we always grab the most current billing cycle data
- Going forward, usage calculations will work correctly

### 2. Clean Up Existing Data
**Zero out the false data accumulated since release (~2 weeks)**
- Recommended approach: Insert a zero value as starting point, then grab current data
- Alternative: Backfill by calling API for each past day (more complex, may be excessive)
- Can be done via cake command (one-time execution)
- Current impact is limited since the UI only shows 3 past billing cycles (not daily granularity)

## Additional Context
- This should be **one ticket** (not split) since both tasks are related and should be done together
- **Priority: High** - Needs to be fixed before next billing cycle
- Richard has already researched the issue (1 hour invested)
- Code location is known
- Testing can be done in production using test devices

## Estimate
**8 hours** - includes both fixing the API call and cleaning up existing data

## Testing Notes
- Test devices are available in Frederick office
- Can use Postman to verify API responses
- Must test in production (Verizon callbacks only work in one environment)

## Team Discussion Notes
- Date discussed: During grooming session (transcript recorded)
- Participants: Aksana, Richard, Noah, Stone
- Decision: Keep as single ticket, prioritize before next billing cycle
- Richard to consult with developer taking the ticket
