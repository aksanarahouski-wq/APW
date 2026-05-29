# Service Plan Filter - Feature Requirements

## Overview
Add a Service Plan filter to the Devices index page, allowing users to filter devices by their assigned service plan. This feature was approved during the Verizon 2nd Account meeting and will be particularly valuable once Verizon Business account service plans are introduced.

## Business Value

### Why This Feature Matters
1. **Verizon Business Account Support** - Users will need to quickly identify and filter devices on business plans vs. regular plans
2. **Customer Value** - Makes it easier for customers to find devices on specific plans without manual workarounds
3. **Eliminates Workarounds** - Current methods (searching by IP patterns, using category flags) require manual setup and training
4. **General Utility** - Useful for all device management scenarios beyond just Verizon Business

### Current State
The Devices index page has filters for:
- Company
- Device Manufacturer & Model
- Device text search (serial numbers, IPs, categories)
- Device Status & Additional Status
- Carrier

**Missing:** Service Plan filter (this feature will add it)

## What We're Building

### New Filter Control
A dropdown filter labeled "Service Plan" that will:
- Display all available service plans alphabetically
- Allow users to filter devices by selecting a plan
- Work alongside existing filters (can combine with company, status, carrier, etc.)
- Include an empty option to show all devices (clear the filter)

### Filter Behavior
- **Selecting a plan:** Shows only devices assigned to that service plan
- **Empty selection:** Shows all devices (no filter applied)
- **Combined filters:** Works with all existing filters simultaneously
- **Pagination:** Filter persists across multiple pages of results
- **Default filter:** Can be saved as a default filter preference

## User Experience

### Where It Appears
The Service Plan dropdown will be added to the existing filter form on the Devices index page, grouped logically with other device property filters.

### How It Works
1. User selects a service plan from the dropdown
2. User clicks "Search"
3. Page displays only devices on that service plan
4. Filter selection is preserved when navigating pages or returning to the page later

### Standard Filter Features (Automatic)
- **Clear Filter:** "Clear Filter" button will reset the service plan selection
- **Set Default Filter:** Selected service plan can be saved as a default filter
- **URL Preservation:** Filter is included in the page URL for sharing/bookmarking

## Functional Requirements

### What Gets Delivered
1. ✅ Service Plan dropdown filter in the Devices index filter form
2. ✅ Dropdown populated with all service plans, ordered alphabetically
3. ✅ Filtering works correctly - shows only devices on selected plan
4. ✅ Filter combines with all existing filters (company, status, etc.)
5. ✅ Filter state persists across pagination
6. ✅ Clear Filter button resets the service plan selection
7. ✅ Set Default Filter saves the service plan preference
8. ✅ Consistent styling with existing filters
9. ✅ No breaking changes to existing functionality

### What's Out of Scope
The following are NOT included in this initial release but could be considered later:
- Visual indicators to distinguish business vs. regular plans
- Multi-select (filter by multiple service plans at once)
- Service plan details on hover
- Quick filter buttons for common plan types

## Testing & Quality Assurance

### Testing Scenarios
We will test the following scenarios before release:

1. **Basic Functionality**
   - Select a service plan and verify correct devices are shown
   - Verify selection persists after page reload

2. **Clear Filter**
   - Verify "Clear Filter" removes the service plan selection
   - Verify all devices are shown after clearing

3. **Combined Filters**
   - Test service plan + company filter
   - Test service plan + status filter
   - Test service plan + manufacturer/model filters

4. **Pagination**
   - Apply filter with multi-page results
   - Verify filter remains applied on page 2, 3, etc.

5. **Default Filter**
   - Save service plan as default filter
   - Verify it auto-applies on return to page

6. **Edge Cases**
   - Service plan with zero devices (empty result)
   - Devices with no service plan assigned

7. **User Roles**
   - Test as Super Admin
   - Test as Customer Admin
   - Verify all roles can use the filter

## Success Metrics

We'll measure success by:
1. Filter works correctly in all test scenarios
2. No performance degradation on Devices index page
3. Positive user feedback from Devon, Adam, and end users
4. Reduction in support requests about finding devices on specific plans

## Related Work

### Verizon 2nd Account Integration
This filter will be especially useful once the Verizon 2nd Account work is complete:
- Business service plans will be identifiable
- Users can quickly filter to "Verizon Business" plans
- Better UX than IP pattern workarounds (e.g., searching for 100.80.x.x)

### Dependencies
- **Standalone feature:** Can be implemented independently
- **No blockers:** Does not depend on other work
- **Does not block:** Other work can proceed in parallel
