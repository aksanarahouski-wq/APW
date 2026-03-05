# Verizon SIM Activation Process Change

**Date Identified:** December 9, 2025 (Meeting 2)
**Reporter:** Adam Curcie
**Priority:** High
**Status:** In Progress - Needs API Implementation

---

## ⚠️ CRITICAL INFORMATION

**SIM Activation Method:** SIM AND SKU
**Required SKU:** `VZW120004440018`

**Key Points:**
- ❌ **Cannot use IMEI + ICCID** for SIM-only activations
- ✅ **Must use SKU + ICCID** (SIM AND SKU method)
- ⚠️ Verizon rejects IMEI+ICCID requests but **does NOT log them as failures** (silent rejection)

---

## Problem Summary

The portal's current reactivation process fails for **Verizon SIM cards** that are inserted into customer-owned devices (not WATM boxes) because the system lacks the IMEI required by the current API call, and Verizon rejects IMEI+ICCID activation attempts for standalone SIMs.

---

## Technical Details

### Current Implementation (Not Working for SIM-Only)

**Activation Method:** "Device and SIM"

**Required Parameters:**
- `IMEI` - Device hardware identifier
- `ICC ID` - SIM card identifier

**When This Fails:**
- Customer puts WATM SIM into their own device
- WATM system only has the ICC ID (SIM card number)
- No IMEI exists in the system for the customer's device
- API call is incomplete and rejected by Verizon
- **Failure is silent** - doesn't appear in logs as the request is simply dropped

### Example from Meeting

**Device:** NGC Group customer device
**Issue:** Reactivation attempted multiple times through portal - all failed
**Workaround:** Adam had to manually reactivate through Verizon's system
**Root Cause:** Missing IMEI in reactivation API call

---

## Required Solution: "SIM AND SKU" Activation

Verizon supports an alternative activation method specifically for SIM-only scenarios.

### New Implementation Needed

**Activation Method:** "SIM AND SKU" (also referred to as "Skew and SIM 2" or "Skew Activation")

**Required Parameters:**
- `ICC ID` (ICCID) - SIM card identifier ✅ Already available in system
- `SKU` - **`VZW120004440018`** (Universal SKU for all SIM-only activations)

**When to Use:**
- Activating or reactivating standalone SIM cards
- No IMEI available in system
- Customer using their own hardware (non-WATM device)

**Important:** This is the **ONLY** method that works for SIM-only activations. IMEI+ICCID method will be rejected by Verizon for standalone SIMs.

### API Changes Required

1. **Detect SIM-only scenarios:**
   - Check if device has IMEI in system
   - If no IMEI → use "SIM AND SKU" method with **SKU: `VZW120004440018`**
   - If IMEI exists → use current "Device and SIM" method (IMEI + ICCID)

2. **Find correct Verizon API endpoint:**
   - Research Verizon API documentation for "SIM AND SKU activation"
   - Alternative names: "skew activation", "skew and SIM 2"
   - Implement new API call structure

3. **Configuration:**
   - Hardcode SKU value: `VZW120004440018` (universal for all SIM-only activations)
   - Or store as configuration parameter if flexibility needed for future changes
   - Add proper error handling for rejected activations (currently silent failures)

---

## Quote from Adam (Meeting Transcript: Lines 300-303)

> "when you do a activation In Verizon. The preferred method is device and SIM which is IMEI and ICC ID, but it accepts this skew and SIM 2 which is what you do. If you only have ICC IDs and then you just use this one skew for all devices."

---

## Impact

### Current State
- ❌ Cannot reactivate SIM-only Verizon devices through portal
- ❌ Failures are silent (don't appear in logs)
- ❌ Requires manual intervention by Adam via Verizon system
- ❌ Poor customer experience (reactivation doesn't work)

### After Fix
- ✅ Portal can reactivate both device+SIM and SIM-only scenarios
- ✅ Proper error handling and logging
- ✅ No manual intervention needed
- ✅ Seamless customer experience

---

## Additional Context

### Carrier Comparison

**Verizon:**
- Very strict about activation requirements
- Requires specific API calls for different scenarios
- "Very, very specific about especially like activations" - Adam

**AT&T & T-Mobile:**
- More flexible
- "Oh, you want Sims on? Turn them on here, go." - Adam
- Easier activation process

### Related Issues

**Similar Bug Noted by Richard (Meeting Transcript: Lines 293-302):**
> "And this is similar to that other bug that you were doing because that also didn't really have a true IMEI, It was just another Verizon device with an ICC ID."

Suggests this is a pattern - multiple instances of Verizon devices/SIMs without IMEIs causing activation issues.

---

## Implementation Plan

### Step 1: Investigation
- [ ] Research Verizon API documentation for "SIM AND SKU" activation method
  - Alternative names: "skew activation", "skew and SIM 2"
- [ ] Identify correct API endpoint for SKU-based activation
- [ ] Document required parameters and request format
- [x] ~~Obtain/confirm universal SKU value from Verizon account~~ **COMPLETED: `VZW120004440018`**

### Step 2: Development
- [ ] Add logic to detect SIM-only scenarios (no IMEI present)
- [ ] Implement new API call for "SIM AND SKU" activation using SKU: `VZW120004440018`
- [ ] Update reactivation flow to choose correct method based on IMEI presence
- [ ] Add proper error handling and logging for rejected activations (prevent silent failures)
- [ ] Add logging to track which activation method was used (Device+SIM vs SKU+SIM)

### Step 3: Testing
- [ ] Test with SIM-only devices (no IMEI)
- [ ] Test with regular devices (with IMEI) - ensure no regression
- [ ] Verify activation logs show proper success/failure status
- [ ] Test deactivation flow (ensure no issues there)

### Step 4: Deployment
- [ ] Deploy to beta/UAT environment
- [ ] Test with real Verizon SIM cards
- [ ] Monitor logs for any issues
- [ ] Deploy to production
- [ ] Monitor initial production activations

---

## Testing Scenarios

### Scenario 1: SIM-Only Reactivation (Primary Fix)
**Given:** Device has ICCID but no IMEI in system
**When:** User attempts to reactivate through portal
**Then:** System uses "SIM AND SKU" API call with SKU: `VZW120004440018` + ICCID
**Expected:** Activation succeeds and is logged properly

### Scenario 2: Device with IMEI (Regression Test)
**Given:** Device has both ICC ID and IMEI in system
**When:** User attempts to reactivate through portal
**Then:** System uses existing "device and SIM" API call
**Expected:** Activation succeeds (no change from current behavior)

### Scenario 3: Failed Activation Logging
**Given:** Any activation scenario
**When:** Verizon API call fails for any reason
**Then:** Failure is logged in system with clear error message
**Expected:** Admin can see failure in logs and investigate

### Scenario 4: Multiple Reactivation Attempts
**Given:** Failed activation (for any reason)
**When:** User retries reactivation
**Then:** System doesn't create duplicate requests or loops
**Expected:** Each attempt is logged separately and doesn't interfere with others

---

## Success Criteria

1. ✅ Portal successfully reactivates Verizon SIM cards without IMEI
2. ✅ No regression in existing device+IMEI activation flow
3. ✅ All activation attempts (success/failure) logged properly
4. ✅ No manual intervention required by Adam for SIM-only reactivations
5. ✅ Clear error messages when activation fails (not silent failures)

---

## Questions for Verizon / Follow-up

1. ✅ ~~What is the universal SKU value for WATM's account?~~ **ANSWERED: `VZW120004440018`**
2. What is the exact API endpoint for "SIM AND SKU" activation?
3. Are there any rate limits or restrictions on SKU activations?
4. What error responses should we expect and handle?
5. Is there any difference in deactivation flow for SKU-activated SIMs?
6. Why doesn't Verizon log IMEI+ICCID rejections as failures for SIM-only activations?
7. Are there any specific API headers or authentication requirements for SKU activations?

---

## Related Documentation

- **Ticket:** Created by Adam (Meeting 2, timestamp ~18:00-26:00)
- **Meeting Notes:** `/Users/aksana/Documents/Projects/WATM/Configurations/Meetings/Meeting2.md`
- **Timestamp in Meeting:** Lines 259-354 (approximately 19:10 - 26:54)

---

## Technical Notes

### Current Flow (Simplified)
```
User clicks "Reactivate"
  → Portal checks device info
  → Calls Verizon API with (IMEI + ICC ID)
  → If IMEI missing → API call incomplete → Verizon drops request
  → No error shown to user
```

### Proposed Flow
```
User clicks "Reactivate"
  → Portal checks device info
  → IF IMEI exists:
      → Call Verizon API with "Device and SIM" method
      → Parameters: (IMEI + ICCID)
  → ELSE (SIM-only, no IMEI):
      → Call Verizon API with "SIM AND SKU" method
      → Parameters: (SKU: VZW120004440018 + ICCID)
  → Handle response (success or error)
  → Log result with activation method used
  → Show clear message to user
  → If rejected, log as failure (not silent)
```

---

## Priority Justification

**High Priority** because:
- Blocks customer self-service reactivation
- Requires manual intervention (Adam's time)
- Silent failures are bad UX
- Affects business operations (customers can't reactivate their own SIMs)
- Growing use case as more customers bring their own devices

---

---

## Update Log

### December 9, 2025 - Customer Note Added
**Source:** Customer/Client feedback
**Update:** Added critical SKU information

**Customer Note:**
> "SIM Needs to be activated as SIM AND SKU - Use SKU VZW120004440018. Cannot be activated as IMEI and ICCID, as no IMEI is used with SIMs, and is rejected from verizon, not logged as failures."

**Changes Made:**
- Added SKU value: `VZW120004440018`
- Clarified activation method name: "SIM AND SKU"
- Emphasized that IMEI+ICCID method cannot be used for SIMs
- Noted that rejections are not logged as failures (silent failures)
- Updated implementation plan with SKU value
- Updated testing scenarios with specific parameters

---

**Last Updated:** December 9, 2025 - Added specific SKU value from customer
**Next Review:** After API endpoint identification
