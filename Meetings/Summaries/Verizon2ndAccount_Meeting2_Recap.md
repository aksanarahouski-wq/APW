# Meeting 2 Recap - Verizon 2nd Account Project

## Meeting Topics Summary

### 1. **Credit Card Payment Automation**
- Currently processing ~200 manual credit card transactions monthly through QuickBooks
- Discussed challenges with automating dynamic charges (varies by usage)
- Agreed to research solutions but not an immediate priority (can wait 3 months)

### 2. **Confluence/Documentation Organization**
**Devon's Feedback:**
- "Sometimes I have a really hard time navigating through the Confluence pages to find stuff that I'm looking for"
- Asked for a page listing roadmap items from recent meetings
- Mentioned there are existing resources John created about credit card processing

**Aksana's Response:**
- Acknowledged Confluence is currently "a dump" and work in progress
- Explained current structure:
  - **Client Review folder**: Items up for discussion, alignment, discovery (what/why/how before building)
  - **Platform Documentation folder**: Post-release executive summaries and user guidance (Aaron doing most lately)
  - Internal folder vs. client-accessible folders
- Q1 roadmap exists in "yet another software" - needs to be added to Confluence
- Committed to improving organization given the growing list of items

**Action Items:**
- Clean up Confluence organization/navigation
- Add Q1 roadmap to Confluence from external tool
- Make it easier for Devon/Adam to find roadmap items and documentation

### 3. **Project Prioritization & Roadmap**
- Current priorities: Data purging feature (in development), service plan updates (completed)
- Verizon 2nd account addition is next priority (smaller scope, more urgent)
- Config management system is the next big initiative
- Need to balance budget/speed - options for faster delivery with more hours vs. slower/steadier pace

### 4. **Verizon Second Account (Primary Focus)**
- **Scope**: Support Verizon Business Internet (FWA - Fixed Wireless Access) account alongside regular pay-per-use account
- **Upgrade/Downgrade Process**: Manual off-portal (5-step physical process), portal only tracks post-migration
- **Device Management**: Add account type flag at device level, enable filtering by service plan
- **Service Plans**: Business plans simpler - remove usage limits & device group names, keep checkboxes, use flat-rate pricing
- **Terminology**: "Unlimited Internet (FWA)" for business, "Regular Account (PPU)" for pay-per-use
- **Migration**: 15 existing devices to be manually updated post-launch (no automated migration)

### 5. **Mid-Cycle Billing Challenge**
- Issue: When switching between accounts mid-billing cycle, need to bill for usage on both service plans
- Pay-per-use data charges must be captured before switching to unlimited plan
- Decided to defer full automation (low volume currently ~5-10/month)
- Short-term: Manual handling or restrict changes to billing cycle start dates

### 6. **Device Group Mismatch Issue**
- Removed carrier API calls from service plan approval workflow (was causing failures)
- Need alternative solution: Quarterly/monthly automated report of device group mismatches
- Report should be downloadable (like commission reports), not live portal view
- Manual review/correction by team rather than automated patching

### 7. **Small Item**
- Card 1965 checkbox feature is dev-complete, ready to ship soon

## Action Items for Aksana:
1. **Improve Confluence organization and navigation**
2. **Add Q1 roadmap to Confluence** (currently in external tool)
3. Finalize Verizon 2nd account scope and begin development
4. Research credit card automation options (3-month timeline)
5. Continue config management discovery/planning
6. Create device group mismatch reporting functionality
7. Ship card 1965 feature
8. Schedule budget/speed alignment meeting with Devon, Adam, and Laura
