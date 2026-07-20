# Prototype Update Prompts — Build vs. Assign Separation (July 19, 2026)

These prompts update the existing Configuration Management prototype to reflect the new override set design: separating the **build** phase (defining scope, parameters, values) from the **assign** phase (attaching companies, with conflict validation).

Feed these into your Lovable prototype sequentially. They modify existing screens — do not create new pages unless specified.

---

## Prompt U1 — Override Set Editor: Remove Company Assignment from Build Flow

```
Update the Company Override Set creation and edit workflow. Currently, the override set editor includes company assignment as part of the same form. We need to separate these into two distinct phases.

Changes to the Override Set Add/Edit form:

1. REMOVE the company assignment section from the override set editor form. The editor should only contain:
   - Override set name
   - Scope selectors (Model, Carrier, Service Plan — each with "Any" or a specific value)
   - Parameter list with value inputs
   - Save / Cancel buttons
   - Draft/publish workflow (unchanged)

2. The override set editor is now purely about WHAT the override does — not WHO it applies to.

3. When creating a new override set, the flow is:
   - Admin clicks "Create Override Set"
   - Fills in name, scope, selects parameters, sets values
   - Saves (as draft or published)
   - Override set is created with zero company assignments
   - A banner at the top shows: "This override set has no company assignments and is currently inactive."

4. When editing an existing override set:
   - The editor shows the same fields (name, scope, parameters, values)
   - No company assignment UI in the editor itself
   - If companies ARE assigned, show a read-only info line in the header: "Assigned to 3 companies" (clickable, links to the Assignments tab)

Keep the Configuration Preview button (P3.3) — it still works the same way, previewing the resolved configuration through the company override level. The preview now requires the admin to select a "preview context" company from a dropdown if the override set has assignments, or shows resolution without company context (schema → model → 3-way rule → this override set) if unassigned.
```

---

## Prompt U2 — Override Set Detail Page: Add Company Assignments Tab

```
Add a "Company Assignments" tab to the Override Set detail/edit page. This is a new tab alongside the existing parameter editor and change history tabs.

The Company Assignments tab has two sections:

**Section 1: Assigned Companies (top)**

A table showing all companies currently assigned to this override set:

| Column | Description |
|--------|-------------|
| Company Name | The company name (link to company page) |
| Type | "Direct" or "Inherited" (inherited = from distributor parent) |
| Assigned By | Admin who made the assignment |
| Assigned Date | When the assignment was made |
| Status | "Active" or "Conflict" (red badge if conflicts detected) |
| Actions | "Remove" button to unassign |

If no companies are assigned, show an empty state: "No companies assigned. This override set is inactive — assign companies to activate it."

Below the table, show a count: "3 companies assigned (2 active, 1 conflict)"

**Section 2: Assign Company (bottom)**

A company picker to add new assignments:

- Search/autocomplete input: "Search companies..."
- When admin selects a company and clicks "Assign":
  1. System runs conflict validation:
     - Get all other override sets already assigned to this company
     - Check each for scope overlap AND parameter overlap with THIS override set
  2. If NO conflict: assignment succeeds. Company appears in the table above with "Active" status. Show success toast: "Cord Financial assigned successfully."
  3. If CONFLICT found: assignment is BLOCKED. Show an error panel below the picker:
     - "Cannot assign Cord Financial — conflicts detected:"
     - For each conflict:
       - "Parameter 'fw_acl' conflicts with override set 'Cord Firewall' (Any + Any + Any)"
     - "Resolve the conflict by editing one of the override sets to remove the overlapping parameter, or remove the conflicting override set from this company first."
  4. The error panel stays visible until dismissed or the admin tries a different company.

**Validation detail panel:**

Add a "Validate All" button at the top of the assigned companies section. When clicked, the system re-runs conflict validation for ALL assigned companies and updates the Status column. This is useful after editing the override set's parameters or scope.

Sample data:
- Override Set "Cord Firewall Rules" (Any + Any + Any) → fw_acl, fw_enable
  - Assigned to: Cord Financial (Active), Loaded ATMs (Active), Vance RMS (Active)
- Override Set "MQTT Disable" (Any + Any + Any) → mqtt_enable, mqtt_center, mqtt_username, mqtt_keepalive, mqtt_tls
  - Assigned to: Altech (Active), Baum (Active)
- Override Set "I-22 IO Config" (I-22 + Any + Any) → io_enable, io_chip, iodigital_input_data
  - Assigned to: Baum (Conflict — io_chip conflicts with "Baum Custom IO" set)
```

---

## Prompt U3 — Override Set Editor: Post-Edit Re-Validation Banner

```
When an admin saves changes to an override set that already has companies assigned (editing parameters, values, or scope), add a re-validation step after the save.

**Flow:**

1. Admin edits an override set (adds a parameter, changes scope, etc.)
2. Admin clicks Save
3. Save succeeds — the override set is updated
4. System immediately runs conflict re-validation against all assigned companies
5. Results appear as a banner at the top of the page:

**If all companies pass:**
- Green banner: "Changes saved. All 3 assigned companies validated — no conflicts."

**If conflicts are found:**
- Amber/warning banner:
  "Changes saved. Conflict validation found issues:"
  - "Cord Financial: parameter 'fw_acl' conflicts with 'Cord Firewall' (Any + Any + Any)"
  - "This override set will not be compiled or pushed for conflicting companies until resolved."
  - [View Assignments] button linking to the Company Assignments tab
  - [Dismiss] button

6. The Company Assignments tab updates the Status column for affected companies to show "Conflict" with a red badge.

**Scope change specific messaging:**

If the admin changed the scope (e.g., widened from I-22 + Verizon + Any to Any + Any + Any):
- Banner additionally notes: "Scope was changed — this may affect which devices are covered and which company assignments conflict."

If the admin narrowed the scope:
- Banner notes: "Scope was narrowed. Previously conflicting companies may now be resolved."
- System re-validates and clears "Conflict" status for companies that are now clean.
```

---

## Prompt U4 — Override Set Browse Page: Assignment Status Column

```
Update the Company Override Sets browse page (the table listing all override sets) to reflect the build vs. assign separation.

**Add columns:**

| Column | Description |
|--------|-------------|
| Assigned Companies | Count of companies assigned. Show "0 — Inactive" in muted text if none. Show "3 companies" as a link if assigned. |
| Conflict Status | "OK" (green) if all assignments pass validation. "1 conflict" (red badge) if any company has conflicts. Hidden if 0 companies. |

**Update existing columns:**
- Keep: Name, Scope (Model + Carrier + Service Plan), Parameters (count), Status (Draft/Published)
- The "Companies" column from the old design (if it existed) is replaced by the new "Assigned Companies" column above.

**Add filter:**
- Add a filter option: "Assignment Status" with values: All / Assigned / Unassigned / Has Conflicts
- "Unassigned" shows override sets with 0 companies — these are inactive/draft templates

**Sample data for browse page:**

| Name | Scope | Parameters | Status | Assigned Companies | Conflict Status |
|------|-------|------------|--------|-------------------|----------------|
| Cord Firewall Rules | Any + Any + Any | 2 | Published | 3 companies | OK |
| MQTT Disable | Any + Any + Any | 5 | Published | 2 companies | OK |
| I-22 IO Config | I-22 + Any + Any | 3 | Published | 1 company | 1 conflict |
| Altech NAT Rules | Any + Verizon + Any | 4 | Draft | 0 — Inactive | — |
| TMO DNS Override | Any + T-Mobile + Any | 2 | Published | 5 companies | OK |
| VPN Config Template | Any + Any + Any | 8 | Draft | 0 — Inactive | — |
```

---

## Prompt U5 — Company Edit Page: Override Set Assignments View

```
Update the Company Edit page to show which override sets are assigned to this company and allow assignment/removal from the company side.

**Section: "Configuration Override Sets"**

Place this section on the Company Edit page (or as a tab if using tabs). It shows all override sets assigned to this company.

**Table columns:**

| Column | Description |
|--------|-------------|
| Override Set Name | Name (link to the override set editor) |
| Scope | Model + Carrier + Service Plan display |
| Parameters | Count of parameters in the set |
| Status | "Active" or "Conflict" (red badge) |
| Source | "Direct" or "Inherited from [Parent Company Name]" |
| Actions | "Remove" (for direct assignments) or "Unlink" (for inherited) |

**Conflict detail:**
If any override set shows "Conflict" status, show an expandable detail below it:
- "Conflicts with: [other override set name] on parameter(s): fw_acl"
- The admin can click through to either override set to resolve the conflict.

**Add Override Set button:**
- "Assign Override Set" button opens a picker/search showing all published override sets
- When the admin selects one, conflict validation runs immediately (same logic as Prompt U2)
- If conflict: show error inline with details
- If clean: assignment succeeds, set appears in the table

**Empty state:**
"No override sets assigned to this company. Configuration uses the base 3-way rule values only."

**Inherited sets section:**
If this company is a sub-company of a distributor, show inherited override sets in a separate sub-section:
- Header: "Inherited from [Parent Company Name]"
- Each inherited set can be "Unlinked" (stops inheriting) or left as-is
- An unlinked set disappears from this company; a "Re-link" option can restore it

**Sample data for Cord Financial:**
| Override Set Name | Scope | Parameters | Status | Source |
|-------------------|-------|------------|--------|--------|
| Cord Firewall Rules | Any + Any + Any | 2 | Active | Direct |
| RADIUS Auth Config | Any + Verizon + Any | 2 | Active | Direct |

**Sample data for Loaded ATMs (sub-company of Cord):**
| Override Set Name | Scope | Parameters | Status | Source |
|-------------------|-------|------------|--------|--------|
| Cord Firewall Rules | Any + Any + Any | 2 | Active | Inherited from Cord Financial |
| Loaded ATMs RMS | Any + Any + ATM | 3 | Active | Direct |
```

---

## Prompt U6 — Update Device Page V1 Context Info for Multiple Override Sets

```
Update the Device Page V1 Configuration section context info to reflect that a device's company may have multiple override sets (and some may be in conflict status).

**Changes to the V1 Engine context info block:**

When set to V1 Engine, update the override set display:

Before (single override set line):
  Override Set: Cord Financial RMS Access

After (multiple override sets, with status):
  Override Sets:
    - Cord Firewall Rules (Any + Any + Any) — Active
    - RADIUS Auth Config (Any + Verizon + Any) — Active

If no override sets match this device (either none assigned to the company, or assigned sets' scopes don't match this device's model/carrier/service plan):
  Override Sets: None matching this device

If a conflict exists on any matching override set:
  Override Sets:
    - Cord Firewall Rules (Any + Any + Any) — Active
    - I-22 IO Config (I-22 + Any + Any) — Conflict (not applied)
  Warning badge: "1 override set has conflicts and is not being applied"

**Changes to the V1 Configuration Preview table:**

In the Source column, when a parameter value comes from a company override, show which override set it came from:
- Before: "Company Override" (orange badge)
- After: "Company Override: Cord Firewall Rules" (orange badge with set name)

This helps the admin understand which override set is responsible for each parameter value, especially when multiple sets are active.

**Sample device data updates:**

- Device "ATM-VZW-002" — V1 Engine, Rule: I-22 + Verizon + [ATM, Standard], Override Sets: Cord Firewall Rules (Any+Any+Any), RADIUS Auth Config (Any+Verizon+Any), Company: Cord Financial
- Device "ATM-TMO-003" — V1 Engine, Rule: I-22 + T-Mobile + [ATM], Override Sets: Cord Firewall Rules (Any+Any+Any), Company: Loaded ATMs
- Device "ATM-VZW-004" — V1 Engine, Rule: 4100 + Verizon + [ATM], Override Sets: none, Company: Miele
```

---

## Prompt U7 — Override Set Editor: Conflict Validation Summary Panel

```
Add a "Conflict Validation" summary panel to the Override Set editor page. This gives the admin visibility into the health of all company assignments without navigating to the Assignments tab.

**Placement:**
Right sidebar or a collapsible panel below the scope selectors, above the parameter list.

**Content:**

If no companies assigned:
  "No companies assigned — this override set is inactive."
  [Assign Companies] button

If all companies pass validation:
  "3 companies assigned — all validated, no conflicts."
  Small green indicator dot.

If conflicts exist:
  "3 companies assigned — 1 conflict detected"
  Red indicator dot.
  Expandable detail:
    "Cord Financial: 'fw_acl' conflicts with 'Cord Firewall' (Any + Any + Any)"
  [View Assignments] link to the Assignments tab

**Live update behavior:**
As the admin edits parameters or scope in the editor (before saving), the panel shows a notice:
  "Unsaved changes — conflict status may change after save."
After saving, the panel updates with fresh validation results (matching the banner from Prompt U3).
```

---

## Prompt U8 — Update Side-by-Side Comparison for Multiple Override Sets

```
Update the Device Page Side-by-Side Legacy vs. V1 Comparison view (from Prompt P4.4) to handle multiple override sets in the V1 column.

**Changes to the V1 column header:**

Replace the single override set reference with a list:
- Before: "Override Set: Cord Financial RMS Access"
- After:
  "Override Sets (2 active):"
  "- Cord Firewall Rules (Any + Any + Any)"
  "- RADIUS Auth Config (Any + Verizon + Any)"

**Changes to the parameter table V1 Source column:**

When a parameter's source is "Company Override", show the override set name:
- Before: "Company Override"
- After: "Company Override (Cord Firewall Rules)"

This makes it clear which override set is responsible for each value difference when comparing against the legacy configuration.

**If an override set is in conflict status:**
- Show it in the header with a strikethrough and "(conflict — not applied)" note
- Parameters from that set are excluded from the V1 resolved values
- Those parameters fall back to whatever the next layer provides (3-way rule, model default, or schema default)
```

---

## Prompt U9 — Separate Edit Actions: Values/Scope vs. Company Assignments

```
The override set detail page currently has a single "Edit Draft" button that opens the full editor. Split this into two distinct entry points so admins clearly understand whether they're editing WHAT the override does vs. WHO it applies to.

**Changes to the Override Set detail page header:**

Replace the single "Edit Draft" button with two buttons:

1. **"Edit Values & Scope"** (primary button)
   - Opens the override set editor with: name, scope selectors (Model, Carrier, Service Plan), parameter list with value inputs
   - This is the "build" action — modifying what the override set contains
   - Save triggers re-validation for all assigned companies (per Prompt U3)

2. **"Manage Assignments"** (secondary/outline button)
   - Navigates to the Company Assignments tab (from Prompt U2)
   - This is the "assign" action — modifying who gets this override set

**Visual treatment:**
- "Edit Values & Scope" = solid primary button (blue)
- "Manage Assignments" = outline/secondary button
- Both buttons visible in the header bar, next to each other
- If the override set is in Published state, "Edit Values & Scope" should show "Create New Draft" instead (follows the existing draft/publish workflow)

**Tab structure on the override set detail page:**
Ensure the page has clear tabs:
- **Overview** — read-only view of scope, parameters, values, and the conflict validation summary panel (Prompt U7)
- **Company Assignments** — the full assignment management UI (Prompt U2), now also reachable via "Manage Assignments" button
- **Change History** — audit log of edits

The "Edit Values & Scope" button opens an edit mode on the Overview tab (or a modal/drawer), NOT a separate page.
```

---

## Prompt U10 — Company Assignments Tab: Standalone vs. Distributor Sections

```
The Company Assignments tab (from Prompt U2) currently has a single flat company picker. Split it into two sub-sections to handle standalone companies and distributors differently — distributors require inheritance decisions for their sub-companies.

**Replace the current assignment UI with a tabbed layout inside the Assignments tab:**

Two sub-tabs within Company Assignments:
- **"Standalone Companies"** tab (with count badge, e.g., "Standalone Companies (14)")
- **"Distributors"** tab (with count badge, e.g., "Distributors (2)")

---

**Sub-tab A: Standalone Companies**

For companies that are NOT distributors (no sub-companies). Simple assign/unassign flow.

Top section — "Assigned Standalone Companies" table:

| Column | Description |
|--------|-------------|
| Company Name | Link to company page |
| Assigned By | Admin who assigned |
| Assigned Date | When assigned |
| Status | "Active" or "Conflict" (red badge) |
| Actions | "Remove" button |

Bottom section — "Assign Company" picker:
- Search/autocomplete: "Search standalone companies..."
- Only non-distributor companies appear in search results
- On assign: run conflict validation (same as Prompt U2)
- If conflict: block with error detail
- If clean: company appears in table above

---

**Sub-tab B: Distributors**

For distributor companies with sub-company inheritance management.

**Top area: Assigned Distributors list**

Each assigned distributor appears as an expandable card:

┌─────────────────────────────────────────────────────────────┐
│ ▼ Miele                                          [Remove]  │
│   Distributor · 248 sub-companies                          │
│                                                             │
│   ☑ Apply to sub-companies                                  │
│                                                             │
│   ┌───────────────────────────────────────────────────────┐ │
│   │ Sub-Company          │ Status       │ Action          │ │
│   │──────────────────────│──────────────│─────────────────│ │
│   │ Miele Sub-Co 1       │ ● Inheriting │ [Unlink]        │ │
│   │ Miele Sub-Co 2       │ ● Inheriting │ [Unlink]        │ │
│   │ Miele Sub-Co 3       │ ● Inheriting │ [Unlink]        │ │
│   │ Miele Sub-Co 4       │ ○ Unlinked   │ [Re-link]       │ │
│   │                      │  → "Sub-Co 4 Custom"           │ │
│   │ Miele Sub-Co 5       │ ○ Unlinked   │ [Re-link]       │ │
│   │                      │  → (base rule fallback)        │ │
│   └───────────────────────────────────────────────────────┘ │
│   Summary: 3 inheriting, 2 unlinked                         │
│   [Search sub-companies...]                                 │
└─────────────────────────────────────────────────────────────┘

**Distributor card details:**
- Header: distributor name, sub-company count, [Remove] button
- "Apply to sub-companies" checkbox:
  - Checked (default): sub-company table visible, new sub-companies auto-inherit
  - Unchecked: sub-companies stop inheriting. Confirmation before unchecking: "This will stop X sub-companies from inheriting this override set."
- Sub-company table columns: Name, Status (● Inheriting / ○ Unlinked), Detail for unlinked (other override set name or "base rule fallback"), Action (Unlink / Re-link)
- Search/filter within sub-company table
- Collapsed by default for distributors with 20+ sub-companies

**Bottom area: "Add Distributor" picker**
- Search/autocomplete: "Search distributors..."
- Only distributor companies appear
- On selecting a distributor, show onboarding modal (see Prompt U11)

**Sample data:**
- Miele: 248 sub-companies, expanded, 3 inheriting / 2 unlinked
- National ATM Services: 34 sub-companies, collapsed, "32 inheriting, 2 unlinked"
```

---

## Prompt U11 — Distributor Assignment: Onboarding Flow

```
When an admin adds a distributor from the Distributors sub-tab (Prompt U10), show an onboarding modal before the distributor card appears.

**Step 1: Inheritance choice**

Modal:
┌─────────────────────────────────────────────────────────────┐
│ Assign Distributor: Miele                                   │
│                                                             │
│ Miele has 248 sub-companies.                                │
│                                                             │
│ How should sub-companies be handled?                        │
│                                                             │
│ ● Apply to all sub-companies                                │
│   All 248 sub-companies will inherit this override set.     │
│   You can unlink individual sub-companies afterward.        │
│                                                             │
│ ○ Assign distributor only                                   │
│   Only Miele itself gets this override set.                 │
│   Sub-companies keep their current configuration.           │
│                                                             │
│                              [Cancel]  [Assign Distributor] │
└─────────────────────────────────────────────────────────────┘

Default: "Apply to all sub-companies"

**Step 2: Conflict resolution (only if "Apply to all" AND conflicts exist)**

If some sub-companies already have a different override set with overlapping parameters:

┌─────────────────────────────────────────────────────────────┐
│ Sub-Company Conflicts                                       │
│                                                             │
│ 2 of Miele's sub-companies have conflicting override sets:  │
│                                                             │
│ │ Sub-Company      │ Conflicting Set        │ Action    │   │
│ │──────────────────│────────────────────────│───────────│   │
│ │ Miele Sub-Co 4   │ Sub-Co 4 Custom         │ ○ Keep    │  │
│ │                  │ (fw_acl overlaps)       │ ○ Replace │   │
│ │ Miele Sub-Co 5   │ Miele Legacy Firewall   │ ○ Keep    │  │
│ │                  │ (mqtt_enable overlaps)  │ ○ Replace │   │
│                                                             │
│ "Keep" = sub-company stays on its current set (Unlinked)    │
│ "Replace" = sub-company inherits this override set          │
│ Default: Keep                                               │
│                                                             │
│                                    [Back]  [Confirm & Add]  │
└─────────────────────────────────────────────────────────────┘

After confirmation, the distributor card appears expanded in the Distributors sub-tab.

If "Assign distributor only" was selected: no conflict step. Card appears with "Apply to sub-companies" unchecked.
```

---

## Prompt U12 — Fix Assignments Tab Route + Build Out the Page

```
The override set detail page has a "Company Assignments" tab that currently navigates to a 404 page. Fix the routing so the /assignments path resolves correctly and renders the full Company Assignments UI.

**Route fix:**
Ensure that `/admin/config-mgmt/company-override-sets/{id}/assignments` is a valid route that renders the Company Assignments tab content (Prompts U2, U10, U11).

If the override set detail page uses client-side tabs (no route change), make the tab work as a hash route (e.g., `#assignments`) or as a nested route. Either way, the tab must be navigable both by clicking the tab header AND by direct URL.

**Page content when routed directly:**
When a user navigates directly to the assignments URL, show:
- The override set header (name, scope, status — same as the detail page)
- The Company Assignments tab selected/active
- The full Standalone Companies + Distributors sub-tab UI from Prompt U10

**Breadcrumb:**
Company Override Sets > [Override Set Name] > Assignments
```

---

## Prompt U13 — Validation Scenarios: Re-Validate After Override Set Edit

```
Add realistic validation scenarios to the prototype to demonstrate how conflict detection works when an override set is edited after companies are already assigned.

**Scenario 1: Adding a parameter that causes a new conflict**

Setup:
- Override Set "MQTT Disable" (Any + Any + Any) with parameters: mqtt_enable, mqtt_center, mqtt_username, mqtt_keepalive, mqtt_tls
- Assigned to: Altech (Active), Baum (Active)
- Override Set "Baum Custom IO" (I-22 + Any + Any) with parameters: io_enable, io_chip, iodigital_input_data
- Assigned to: Baum (Active)

Action: Admin edits "MQTT Disable" and adds parameter `io_chip` to the set.

Result after save:
- Re-validation runs automatically
- Amber banner appears: "Changes saved. Conflict validation found issues:"
  - "Baum: parameter 'io_chip' conflicts with 'Baum Custom IO' (I-22 + Any + Any)"
  - "This override set will not be compiled for conflicting companies until resolved."
  - [View Assignments] button
- Company Assignments tab: Baum's status changes from "Active" to "Conflict" with red badge
- Altech remains "Active" (no conflict)

**Scenario 2: Changing scope that resolves a conflict**

Setup:
- Override Set "I-22 IO Config" (I-22 + Any + Any) with parameters: io_enable, io_chip
- Assigned to: Baum (Conflict — io_chip conflicts with "Baum Custom IO")

Action: Admin edits scope from (I-22 + Any + Any) to (4100 + Any + Any).

Result after save:
- Re-validation runs
- Green banner: "Changes saved. Scope was narrowed. All 1 assigned companies validated — no conflicts."
- Baum's status changes from "Conflict" to "Active" (because "Baum Custom IO" is scoped to I-22 and no longer overlaps with the 4100 scope)

**Scenario 3: Widening scope that creates new conflicts**

Setup:
- Override Set "TMO DNS Override" (Any + T-Mobile + Any) with parameters: dns_static, ntp_server
- Assigned to: 5 companies (all Active)
- Override Set "Altech DNS Config" (Any + Verizon + Any) with parameters: dns_static
- Assigned to: Altech (Active)

Action: Admin edits "TMO DNS Override" scope from (Any + T-Mobile + Any) to (Any + Any + Any).

Result after save:
- Re-validation runs
- Amber banner: "Changes saved. Scope was changed — this may affect which devices are covered and which company assignments conflict."
  - "Altech: parameter 'dns_static' conflicts with 'Altech DNS Config' (Any + Verizon + Any)"
- Altech's status: "Conflict"
- Other 4 companies: still "Active"

**Scenario 4: Removing a conflicting parameter resolves the issue**

Setup:
- Override Set "MQTT Disable" (Any + Any + Any) with parameters: mqtt_enable, mqtt_center, mqtt_username, mqtt_keepalive, mqtt_tls, io_chip
- Assigned to: Altech (Active), Baum (Conflict — io_chip conflicts with "Baum Custom IO")

Action: Admin removes `io_chip` from "MQTT Disable".

Result after save:
- Re-validation runs
- Green banner: "Changes saved. All 2 assigned companies validated — no conflicts."
- Baum's status changes from "Conflict" to "Active"

**Add these as interactive walkthroughs:**
In the prototype, create a "Validation Scenarios" section accessible from the config management page (e.g., a help/demo button). Each scenario should be a clickable card that sets up the state and walks the admin through the edit → save → re-validation flow, showing the banners and status changes in real time.
```
