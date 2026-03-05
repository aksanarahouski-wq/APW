# UI Design Decision: Radio Buttons for Verizon Account Selector

**Decision Date:** 2026-02-17
**Decision Owner:** Aksana Rahouski
**Status:** ✅ Approved and Documented

---

## Decision

**Selected:** Radio Buttons (Option C from Meeting 2 Prep)

---

## Implementation Specification

### Device Create/Edit Form

**Field Label:** "Verizon Account"

**UI Element:** Radio button group with two options

**Radio Button Options:**
- ⚪ **Regular Account (PPU)**
  - Sublabel/tooltip: "Pay Per Use - Standard per-byte data pricing"
  - Value stored in database: `'regular'`

- ⚪ **Unlimited Internet (FWA)**
  - Sublabel/tooltip: "Fixed Wireless Access - Flat-rate unlimited data"
  - Value stored in database: `'fwa'`

**Default Selection:**
- New devices: "Regular Account (PPU)" pre-selected
- Existing devices: Display current value from `devices.verizon_account_name` field
- Legacy devices (NULL account): Default to "Regular Account (PPU)"

**Validation:**
- Required field (cannot be NULL for Verizon devices)
- Must match service plan compatibility when service plan is assigned

---

## Visual Design Guidance

### Layout
```
Verizon Account: *

⚪ Regular Account (PPU)
   Pay Per Use - Standard per-byte data pricing

⚪ Unlimited Internet (FWA)
   Fixed Wireless Access - Flat-rate unlimited data
```

### Styling Recommendations
- **Radio Buttons:** Use standard HTML radio inputs with custom styling matching existing form elements
- **Labels:** Bold primary text, regular weight for sublabels
- **Spacing:** 16px vertical spacing between radio button options
- **Alignment:** Left-aligned, radio button icon vertically centered with primary label
- **Active State:** Highlight selected option with primary brand color
- **Disabled State:** Gray out both radio buttons and labels when field is read-only

### Responsive Behavior
- **Desktop:** Radio buttons stacked vertically
- **Mobile:** Maintain vertical stack, ensure touch targets are 44px minimum height
- **Tablet:** Same as desktop

---

## Rationale for Radio Buttons

### 1. Scalability
**Problem Solved:** Need to accommodate potential future additions without code changes
- 3rd Verizon account (unlikely but possible)
- T-Mobile Business Internet (mentioned in Meeting 2)
- Other carrier-specific account types

**Why Radio Buttons Win:**
- Adding a 3rd option is trivial: Just add another `<input type="radio">` element
- No refactoring needed (unlike checkbox which would require complete redesign)
- Same pattern works for 2, 3, 4+ options

### 2. User Experience
**Problem Solved:** Users need clear visual indication this is a single-choice selection
- Two Verizon accounts are mutually exclusive (device cannot be on both)
- Visual affordance should match the constraint

**Why Radio Buttons Win:**
- **Checkbox:** Implies multiple selections possible (misleading for mutually exclusive choice)
- **Dropdown:** Hides options behind click/tap (less discoverable, requires interaction to see options)
- **Radio Buttons:** Shows all options immediately, clear visual indication of single choice

### 3. Accessibility
**Problem Solved:** Screen readers and keyboard navigation need semantic correctness
- Assistive technology should understand the input type
- ARIA labels should match the interaction pattern

**Why Radio Buttons Win:**
- Semantically correct: `<input type="radio" role="radio">` communicates mutually exclusive choice
- Built-in keyboard navigation: Arrow keys move between options, Space/Enter selects
- Screen readers announce "radio button, 1 of 2" automatically

### 4. Development Simplicity
**Problem Solved:** Backend logic should be straightforward
- Same data type regardless of UI choice (string: 'regular' or 'fwa')
- Validation is simple

**Why Radio Buttons Win:**
- Checkbox: Requires boolean-to-string conversion logic ('checked' = 'fwa', 'unchecked' = 'regular')
- Dropdown: Requires dropdown component library or custom styling
- Radio Buttons: Native HTML, minimal JavaScript, standard validation

---

## Comparison to Alternatives

| Criteria | Checkbox | Dropdown | Radio Buttons |
|----------|----------|----------|---------------|
| **Scalability (3+ options)** | ❌ Doesn't work | ✅ Scales well | ✅ Scales well |
| **Visual Clarity** | ⚠️ Confusing (implies multi-select) | ⚠️ Options hidden | ✅ Clear, all options visible |
| **Accessibility** | ❌ Semantically incorrect | ✅ Good | ✅ Excellent |
| **Mobile UX** | ✅ Compact | ⚠️ Requires tap to open | ✅ Touch-friendly |
| **Development Time** | ✅ Fast | ⚠️ Requires component | ✅ Fast (native HTML) |
| **Future-Proofing** | ❌ Would need refactor | ✅ No changes needed | ✅ No changes needed |

---

## Technical Implementation Notes

### HTML Structure
```html
<div class="form-group">
    <label for="verizon-account">
        Verizon Account <span class="required">*</span>
    </label>

    <div class="radio-group">
        <div class="radio-option">
            <input
                type="radio"
                id="account-regular"
                name="verizon_account_name"
                value="regular"
                checked
                required
            />
            <label for="account-regular">
                <strong>Regular Account (PPU)</strong>
                <span class="sublabel">Pay Per Use - Standard per-byte data pricing</span>
            </label>
        </div>

        <div class="radio-option">
            <input
                type="radio"
                id="account-fwa"
                name="verizon_account_name"
                value="fwa"
                required
            />
            <label for="account-fwa">
                <strong>Unlimited Internet (FWA)</strong>
                <span class="sublabel">Fixed Wireless Access - Flat-rate unlimited data</span>
            </label>
        </div>
    </div>
</div>
```

### JavaScript Validation (Client-Side)
```javascript
// Ensure radio button is selected before form submission
const form = document.querySelector('#device-form');
form.addEventListener('submit', function(e) {
    const accountSelected = document.querySelector('input[name="verizon_account_name"]:checked');
    if (!accountSelected) {
        e.preventDefault();
        alert('Please select a Verizon Account type.');
        return false;
    }
});
```

### PHP Validation (Server-Side)
```php
// In DevicesController or Device Entity
$validAccounts = ['regular', 'fwa'];
if (!in_array($data['verizon_account_name'], $validAccounts)) {
    throw new ValidationException('Invalid Verizon account type. Must be "regular" or "fwa".');
}
```

---

## Affected Files/Components

Based on PRD v2.1, radio buttons will appear in:

1. **Device Create Form** (`/plugins/Devices/templates/Admin/Devices/add.php`)
2. **Device Edit Form** (`/plugins/Devices/templates/Admin/Devices/edit.php`)
3. **Device Import Template** (not radio buttons, but affects allowed values in CSV)

---

## References

- **PRD v2.1:** `/Users/aksana/Documents/Projects/WATM/Verizon2ndAccount/PRD_Verizon_Second_Account_Support_v2.md`
- **Meeting 2 Prep:** Section 2.3 "Checkbox vs. Dropdown for Account Selection" (recommended radio buttons or dropdown)
- **Meeting 2 Transcript:** Lines 1237-1244 (Devon delegates decision to Aksana)
- **Action Items:** `/Users/aksana/Documents/Projects/WATM/Verizon2ndAccount/Meeting2_ActionItems.md` - Item #1

---

## Communication Plan

**Stakeholders to Notify:**
- ✅ Aksana (decision owner)
- [ ] Development Team Lead (for implementation)
- [ ] UI/UX Designer (if applicable)
- [ ] QA Team (for test plan updates)
- [ ] Devon D'Andrea (for awareness)
- [ ] Adam Curcie (for awareness)

**Communication Method:**
- Email summary with link to this document
- Update in next team meeting
- Add to Jira ticket as comment

---

## Next Steps

1. **Immediate:**
   - [x] Document decision (this file)
   - [x] Update PRD to v2.1
   - [x] Update action items document
   - [ ] Create wireframes/mockups with radio button design

2. **Before Development:**
   - [ ] Review with development team
   - [ ] Confirm CSS/styling approach matches existing form patterns
   - [ ] Add to UI component library if applicable

3. **During Development:**
   - [ ] Implement radio buttons in device create/edit forms
   - [ ] Update validation logic
   - [ ] Write unit tests for radio button selection
   - [ ] Test accessibility (screen reader, keyboard navigation)

4. **Post-Implementation:**
   - [ ] UAT with Devon/Adam
   - [ ] Verify mobile/tablet responsiveness
   - [ ] Confirm consistency across browsers

---

**Document Owner:** Aksana Rahouski
**Last Updated:** 2026-02-17
