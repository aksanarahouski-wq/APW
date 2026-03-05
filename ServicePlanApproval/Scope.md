Hi Devon and Adam,

After further investigation (thanks to Stone), here's what we found:

 Issue Summary

  When a service plan is approved, the system automatically attempts to update carrier groups (Verizon/AT&T) for devices with group mismatches. For example, the ATM - Unlimited plan has 68,116 Verizon devices, and 74 of them had a group mismatch. When the Verizon API call to update these 74 devices failed, it triggered a critical issue.

  What Happens When Carrier API Fails

  1. Admin clicks "Approve" on a service plan
  2. Service plan status saves as "approved" in the database ✅
  3. System attempts to update carrier groups via Verizon/AT&T APIs
  4. API call fails (timeout, invalid group, carrier system down, etc.) ❌
  5. System throws an error and stops processing
  6. However, the service plan remains marked as "approved" in the database

  The Problem - Inconsistent State

   ✅ The service plan shows as "approved" in the system, but:
  - ❌ Devices are NOT updated with correct carrier billing groups
  - ❌ Billing cycle data is NOT updated with new pricing
  - ❌ Custom pricing adjustments are NOT recalculated
  - ❌ No clear indication in the UI that the approval is incomplete


  Why We Didn't Catch This in Lower Environments

  The system does NOT make actual API calls to Verizon/AT&T in review/beta - only in production. This means:
  - Lower environments always "succeed" (no real API calls made)
  - The failure scenario is never exercised during testing
  - This inconsistent state only occurs in production when real carrier APIs fail

  Decision Needed

  Should a carrier group update failure abort the entire approval?

  - Option A (Abort): If carrier API fails, service plan should NOT be approved. Status remains "pending" and requires manual retry.
  - Option B (Continue): Service plan approval succeeds regardless. Carrier updates are retried separately, and we add visibility/logging for failures.

  Given the tight deadline, we recommend proceeding with Option B as a temporary fix - approval succeeds, we add better error handling and logging, and carrier sync failures don't block the entire process. This gets you unblocked immediately.

  Proper Long-Term Fix (Post-Deadline)

  Move the entire post-approval workflow to a queue-based async job considering this action makes API calls and updates high volume of data (devices):

  Immediate (Synchronous):
  - Validate and save approval status
  - Queue background job
  - Return success to admin: "Service plan approved. Updates are being processed in background."

  Background Job (Asynchronous):
  1. Update carrier groups - Verizon/AT&T APIs with retry logic, continues even if it fails or rework if must succeed with transaction rollback on failure
  2. Update billing cycle data - Must succeed with transaction rollback on failure
  3. Recalculate custom pricing - Must succeed with transaction rollback on failure
  4. Send email notifications - Success/failure summary with details

  Benefits:
  - No web request timeouts with large device counts
  - Proper retry logic for transient failures
  - Email notifications on success/partial success/failure
  - Detailed logging at each step
  - Failed jobs can be retried without re-approving plan

  Immediate Next Steps

  1. Please confirm: Do you want us to proceed with Option B (temporary fix) to unblock approvals?

Let me know if you have any questions.

Thank you!

Aksana Rahouski
Sr. Product Manager

Orases Custom software to reach your vision
5728 Industry Lane, Frederick, MD 21704
Company: 301.694.8991 • Direct: 301.328.1519
Aksana.Rahouski@orases.com • orases.com




Client response:
There isn't a reason to send the api calls to the carrier at all when making changes to the service plans unless we are changing the values that associate the service plan with the device group to the carries. If we are not changing the carrier device group, (custom field on ATT and tag for TMO)- might be best to send no API calls for actions that are only changing system billing values. It's that possible to separate? 


My response: It's hard for Stone and me to determine why or when this was added, or what the API endpoint actually does or why it's failing (something we should investigate separately) - we're just troubleshooting the code at this point.

What we can see: The code calls both Verizon and AT&T APIs when it finds devices on a plan where vz/att_group_applied doesn't match the Service Plan's Verizon or AT&T group. Perhaps this is intended to update the carrier-side associations between service plans and device groups, like you said. We have 74 Verizon devices in prod, where these do not match right now, why API is being called.

To answer your question: Yes, we can definitely separate this. We can remove these API calls entirely from the 'service plan update’ action. If this functionality still needs to exist, we will need to figure out what should trigger it. We can address the proper implementation separately.

Let me know what you think.

Thank you.


Client response: Yes Devon and I agree to turn that api call off for now
