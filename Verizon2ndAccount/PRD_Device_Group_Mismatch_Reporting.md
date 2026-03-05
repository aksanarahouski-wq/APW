# Device Group Mismatch Reporting - Requirements

## Problem Statement

**Current Situation:**
- ~47-74 devices in production have device groups that don't match their service plan's carrier group
- This happens intentionally when team manually adjusts device tiers at Verizon/AT&T for customer satisfaction, but doesn't update the portal
- Example: Device uses more data than ATM plan allows, team bumps it to Tier 1 at Verizon side, but keeps customer paying $4.95/month in portal

**Why It Matters:**
- These mismatches are legitimate business workarounds, not errors
- Team needs visibility to review and manually reconcile when appropriate
- No way to identify which devices are out of sync without manual exports and cross-referencing

## Solution

**Build a live admin page that displays mismatched devices on-demand** instead of auto-fixing mismatches in the portal.

### What We're NOT Building:
- ❌ Automatic syncing/patching of device groups
- ❌ Real-time validation during device operations
- ❌ Scheduled background job
- ❌ Carrier API calls to "fix" mismatches

### What We ARE Building:
- ✅ **Admin page that queries mismatched devices on page load** (similar to "Browse Checkin Failures")
- ✅ Displays devices where `vz_group_applied` ≠ Service Plan's Verizon Group (and AT&T equivalent)
- ✅ Live data table view with sortable columns
- ✅ "Download CSV" button to export current results
- ✅ Team manually reviews and decides what action to take outside the system

## Requirements

### 1. Admin Page - "Browse Device Group Mismatches"

**Location:** Admin section (similar to existing "Browse Checkin Failures" page)

**Page Behavior:**
- Query runs on page load to identify all mismatched devices
- No background job or scheduled processing required
- Real-time data - always shows current state

**UI Pattern Reference:**
- Follow "Browse Checkin Failures" pattern (see screenshot)
- Simple table layout with sortable columns
- Clean, minimal design
- Download CSV button at top of page

### 2. Table Display Fields
Include the following columns:

| Field | Description |
|-------|-------------|
| Device Serial Number | Primary identifier |
| Company Name | Which customer owns the device |
| Service Plan Name | Current assigned service plan |
| Carrier | Verizon or AT&T |
| Portal Device Group | `vz_group_applied` or `att_group_applied` |
| Service Plan Group | `verizon_provider_group` or `att_provider_group` |
| Last Updated | When device was last modified |
| Status | Active/Suspended/etc. |

### 3. CSV Export Functionality

**Export Button:**
- Located at top of page (similar to other admin browse pages)
- Label: "Download CSV" or "Export to CSV"
- Exports all current results shown in table

**CSV Contents:**
- All columns from table display
- Same data as visible on page
- Standard CSV format compatible with Excel

### 4. UI/UX Details

**Page Header:**
- Page title: "Browse Device Group Mismatches" or "Device Group Mismatch Report"
- Optional: Show count of total mismatches found (e.g., "47 devices with mismatches")

**Table Features:**
- Sortable columns (click column header to sort)
- Simple pagination if results > 50-100 rows (optional - can defer to Phase 2)
- Clean table styling matching existing admin pages

**Empty State:**
- If no mismatches found: Display message "No device group mismatches found"
- Positive messaging (this is good news!)

## User Workflow

1. Admin navigates to Admin > Browse Device Group Mismatches
2. Page loads and automatically queries for mismatched devices
3. Results display in table (real-time, current data)
4. Admin reviews mismatches on screen
5. (Optional) Admin clicks "Download CSV" to export data for offline review
6. Team decides on case-by-case basis:
   - Keep mismatch (intentional customer accommodation)
   - Update portal to match Verizon/AT&T side
   - Update Verizon/AT&T side to match portal
7. Admin can refresh page anytime to see updated results

## Success Criteria

- ✅ Page loads and queries mismatches in reasonable time (<5 seconds for ~100 devices)
- ✅ Table accurately identifies all device group mismatches (both Verizon and AT&T)
- ✅ CSV export works and includes all displayed data
- ✅ Zero impact on device operations or service plan workflows
- ✅ Team can identify and review ~47-74 existing mismatches
- ✅ Page can be accessed anytime by admins (no scheduling dependency)


## Out of Scope

- ❌ Automated reconciliation/fixing of mismatches
- ❌ Carrier API calls to update device groups
- ❌ Real-time validation during device assignment
- ❌ Integration with billing system
- ❌ Customer-facing visibility
- ❌ Scheduled background jobs or cron tasks
- ❌ Email notifications
- ❌ Historical report archives (only shows current state)


## Acceptance Criteria

- [ ] Admin page accessible via Admin menu navigation
- [ ] Page queries and displays mismatched devices on load
- [ ] Table accurately lists all devices where device group ≠ service plan group (both Verizon and AT&T)
- [ ] Table includes all required fields for team to make decisions
- [ ] Columns are sortable (at minimum: by serial number, company name, carrier)
- [ ] "Download CSV" button exports current table data
- [ ] CSV format is Excel-compatible
- [ ] Page handles errors gracefully (query failures show error message, don't crash page)
- [ ] Empty state displays properly when no mismatches found
- [ ] Existing ~47-74 mismatched devices appear in results
- [ ] Page loads in reasonable time (<5 seconds)
- [ ] UI follows existing admin page patterns (looks consistent with "Browse Checkin Failures")

