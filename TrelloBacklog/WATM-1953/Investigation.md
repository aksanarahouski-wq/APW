# Investigation Notes — WATM-1953: Deactivated Device CleanUp Command

## What Happened

On 11/12/25, the `CleanUpDeactivatedDevicesForCurrentCycleCommand` set `charge_for_current_cycle = 0` on device RF3022109011248 (Digital Music Systems, Inc). However, the device was not actually deactivated — it remained active and consumed 47GB of data that was never billed.

## How the Command Works (Current Behavior)

Based on prior analysis of the billing pipeline (`Billing/Command/CleanUpDeactivatedDevicesForCurrentCycleCommand.php`):

1. The command finds devices it considers "deactivated" and sets `charge_for_current_cycle = 0` on both the device and its `company_device_usages` records
2. It intentionally **excludes** devices deactivated during the current billing cycle (so they still get billed for partial-cycle usage)
3. It does **not** verify the device's actual `device_status_id` before flipping the billing flag

## The Problem

The command assumes that if a device matches its query criteria, it is truly deactivated. In this case, that assumption was wrong — the device was still active. The command has no safety check to confirm the device's status before removing it from billing.

This is the opposite side of a related gap we've seen in the company deactivation flow (see company deactivation analysis), where the `waive_final_bill` flag on the company doesn't propagate to device-level billing fields. Both issues stem from a disconnect between device status and billing flags.

## What Needs Investigation

1. **What query/condition does the command use to identify devices?** — We need to review the exact query to understand why RF3022109011248 was included. Is it checking `device_status_id`, or inferring deactivation from some other field (e.g., a log entry, a company status)?

2. **Why was this device flagged as deactivated when it wasn't?** — Was there a failed carrier deactivation? A status that was set and then reverted? A company-level status change that didn't cascade correctly to the device?

3. **What was the device's actual status at the time the command ran?** — Check `o_logs` for device RF3022109011248 around 11/12/25 to see its status history.

## Recommended Fix Direction

The command should:
- **Verify `device_status_id` is actually "Deactivated"** before setting `charge_for_current_cycle = 0`
- **If a device matches the cleanup criteria but is NOT deactivated**, skip it and either:
  - Log a warning/alert for manual review, or
  - Force-deactivate the device (with notification)
- Consider adding a reconciliation check: if a device has `charge_for_current_cycle = 0` but is still active and consuming data, flag it
