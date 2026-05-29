# T-Mobile FWA (Fixed Wireless Access) Support
## Feature Summary for APW

**Date:** May 4, 2026
**Status:** Draft
**Prepared by:** Aksana Rahouski, Orases

---

## What Is This?

This feature adds T-Mobile FWA (Fixed Wireless Access) support to the portal, extending the same FWA capability recently built for Verizon. It allows the portal to properly manage, bill, and configure T-Mobile devices that are on the unlimited $75/month business internet plan.

Today, T-Mobile FWA devices are tracked using flat-rate overrides and admin notes because the portal has no way to distinguish them from regular pay-per-use T-Mobile devices. This feature replaces those workarounds with proper portal support.

---

## Who Is This For?

- **Operations (Devon, Dan)** -- Manage T-Mobile FWA devices through the portal the same way as any other device: assign service plans, deliver configurations, track status
- **Admin (Adam)** -- Designate which T-Mobile devices are on the FWA plan; ensure correct billing without manual flat-rate overrides

---

## What's Included

### 1. Rate Plan Type on T-Mobile Devices

A new "Rate Plan Type" field on T-Mobile devices with two options:
- **PPU** (Pay Per Use) -- the default for all T-Mobile devices
- **FWA** (Fixed Wireless Access) -- for devices on the $75/month unlimited plan

Only super admin users can change this setting. It can be freely toggled between PPU and FWA as needed -- it's an administrative label and does not trigger any changes on the T-Mobile side.

### 2. FWA Service Plan with T-Mobile Pricing

T-Mobile FWA devices use the same FWA service plan already built for Verizon. You'll set T-Mobile-specific pricing through the existing carrier pricing tiers (e.g., $75/month for T-Mobile).

No new service plan needs to be created -- just add T-Mobile pricing to the existing FWA plan.

### 3. Plan-Device Validation

The portal will enforce that:
- FWA devices can only be assigned FWA service plans
- PPU devices can only be assigned PPU service plans

This prevents accidental mismatches that could cause billing errors. Same validation behavior as Verizon FWA.

### 4. Device Import

The device import process will support selecting PPU or FWA for T-Mobile devices. If not specified, devices default to PPU.

---

## What's NOT Included

- **Automated T-Mobile plan changes** -- T-Mobile does not currently support changing a SIM to the FWA plan via API. Plan changes will continue to be handled by emailing T-Mobile. This is a T-Mobile limitation, not a portal limitation.
- **T-Mobile callback notifications** -- We may investigate in the future whether T-Mobile can notify the portal when a plan change is made, but this is not part of the initial release.

---

## How It Compares to Verizon FWA

The T-Mobile version is simpler because T-Mobile FWA runs on the same account and API as regular T-Mobile SIMs. There's no separate provider account, no separate API credentials, and no API routing changes needed.

| | Verizon FWA | T-Mobile FWA |
|---|---|---|
| Separate API account | Yes | No -- same account |
| Portal field name | Provider Account Type | Rate Plan Type |
| Service plan | Shared FWA plan | Shared FWA plan |
| Pricing | Carrier-specific tiers | Carrier-specific tiers |
| Plan change process | Manual (5-step Verizon process) | Email T-Mobile |
| Who can change it | Super admin | Super admin |

---

*Based on the T-Mobile Business Account Discovery meeting, May 4, 2026 -- Aksana, Aaron, Noah (Orases); Adam, Devon (APW).*
