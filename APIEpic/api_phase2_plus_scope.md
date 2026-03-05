# API Development Scope - Phase 2 and Beyond

## Document Overview
This document defines the Phase 2+ scope for the WATM API development. These features include POST/PUT operations, real-time interactions, and future enhancements.

**Customer:** AllPoint (First pilot customer)
**Last Updated:** November 18, 2025
**Based on:** Meeting dated November 17, 2025

---

## Why Two Phases?

1. **Technical complexity:** Real-time pings require async processing, device communication, timeout handling
2. **Infrastructure:** POST operations need queue systems, webhook support
3. **Testing:** GET endpoints safer to test and deploy first
4. **Customer validation:** Ensure data format/content meets needs before building complex features
5. **Incremental delivery:** Get value to customer faster, iterate based on feedback

---

#### Phase 2 - POST/Trigger Endpoint
**Endpoint:** `POST /devices/{serial_number}/ping`

**What it does:**
1. Receives API request from customer
2. WATM portal initiates live ping to device
3. Device responds with current status
4. Portal returns fresh status to customer via API response

**What it returns:**
- **Real-time** device status (not cached)
- Current connection state
- Response time/latency
- Success/failure of ping attempt

**Data characteristics:**
- Fresh data pulled at moment of API call
- Requires device to be online to respond
- Takes longer than MVP GET (must wait for device response)

**Use case:**
- Support rep on call with customer having connectivity issue
- Needs to verify device status RIGHT NOW
- Can diagnose if device is currently reachable

**Technical requirements:**
- POST endpoint (triggers action, not just data retrieval)
- Asynchronous processing (may take seconds for device to respond)
- Timeout handling if device doesn't respond
- Error handling for offline devices

**Implementation complexity:**
- Requires portal to initiate outbound connection to device
- Must handle device response and relay back through API
- Need queue/async processing for longer operations

**Example request/response:**
```json
POST /devices/ABC123/ping

Response:
{
  "serial_number": "ABC123",
  "ping_timestamp": "2025-11-18T16:45:32Z",
  "status": "online",
  "response_time_ms": 1234,
  "success": true
}
```

---

### 2. Integrate Signal Statistics into the Portal

#### Phase 2 - POST/Trigger Endpoint
**Endpoint:** `POST /devices/{serial_number}/signal/refresh`

**What it does:**
1. Customer triggers real-time signal strength check
2. Portal pings device to get current signal strength
3. Device reports current signal strength
4. Portal returns fresh data via API

**What it returns:**
- **Current** signal strength (live)
- Connection status (online/offline)
- Current active carrier (for dual-carrier)
- Signal strength for all available carriers
- Response timestamp

**Data characteristics:**
- Real-time data from device
- Requires device to be online
- Takes longer than GET (must wait for device response)
- More accurate for immediate diagnostics

**Use case:**
- Support rep on call: "Let me check your signal strength right now"
- Customer reports sudden connectivity drop
- Need current signal, not 2-hour-old cached value

**Technical requirements:**
- POST endpoint (triggers device action)
- Portal must ping device for current signal
- Asynchronous processing
- Timeout handling (device may not respond)
- Error states for offline devices

**Implementation complexity:**
- Requires active device communication
- Must handle network latency
- Need to support dual-carrier signal queries
- Error handling for unresponsive devices

**Example request/response:**
```json
POST /devices/ABC123/signal/refresh

Response:
{
  "serial_number": "ABC123",
  "refresh_timestamp": "2025-11-18T16:45:32Z",
  "signal_strength": {
    "verizon": {
      "strength": 52,
      "active": true
    },
    "att": {
      "strength": 61,
      "active": false
    }
  },
  "connection_status": "online",
  "response_time_ms": 1456
}
```

---

## Phase 2 Features - POST/PUT Operations

**Goal:** Add real-time interactions and data modification capabilities

### Real-Time Device Interactions (POST endpoints)

#### 1. Ping the Cell Modem (Real-time)
**Endpoint:** `POST /devices/{serial_number}/ping`
- Trigger live device ping
- Get current connection status
- Return response time and status

#### 2. Refresh Signal Strength (Real-time)
**Endpoint:** `POST /devices/{serial_number}/signal/refresh`
- Trigger live signal strength check
- Get current signal for all carriers
- Return real-time signal data

### Device Configuration Management (POST/PUT endpoints)

#### 3. Assign Modem to Machine (via MAC Address)
**Endpoint:** `POST /devices/{serial_number}/assign`
- Push notification when modem connected to machine
- Use MAC address as identifier
- Sync device-to-machine mapping

**Phase:** Post-MVP (Phase 2) - Confirmed during meeting

**Technical flow:**

1. Customer plugs cell modem into machine (kiosk)
2. Machine boots up
3. WATM device automatically pulls MAC address from kiosk
4. Customer's system pushes MAC address + machine identifier to WATM portal via API
5. WATM portal links the device to the machine/account
6. Synchronized in both systems (WATM portal + AllPoint's IQ Tech system)

**Phase 2 classification reasoning:**
- Requires POST/PUT operation (not just GET)
- More complex than simple data retrieval
- Customer needs to build integration on their end
- Not essential for MVP dashboard/support needs

**Implementation notes:**
- Customer system needs event handler for machine boot-up
- API accepts modem serial number + MAC address + machine identifier
- WATM stores association for device management
- Helps with device tracking and organization

**Example request:**
```json
POST /devices/ABC123/assign

{
  "mac_address": "00:1A:2B:3C:4D:5E",
  "machine_id": "KIOSK-1234",
  "account_id": "ALLPOINT-ACCT-567",
  "assigned_at": "2025-11-18T16:45:32Z"
}
```

**Example response:**
```json
{
  "serial_number": "ABC123",
  "assigned": true,
  "machine": {
    "mac_address": "00:1A:2B:3C:4D:5E",
    "machine_id": "KIOSK-1234",
    "account_id": "ALLPOINT-ACCT-567"
  },
  "assigned_at": "2025-11-18T16:45:32Z"
}
```

#### 4. Push Firewall Settings Remotely / Adjust Firewall Settings
**Endpoint:** `POST /devices/{serial_number}/firewall` or `PUT /devices/{serial_number}/firewall`
- Push firewall rules to device
- Support URL/IP whitelist
- Updates applied at next check-in (within 2 hours)

**Note:** "Push firewall settings remotely" and "Adjust the firewall settings" are the **same feature** - listed separately in requirements document but noted as "Same as above". Both POST and PUT operations supported:
- **POST** - Initial push of firewall rules to device
- **PUT** - Adjust/update existing firewall rules

**Phase:** Post-MVP (Phase 2) - Confirmed during meeting
- **Note:** API Data document originally stated "Launch or Soon After" but meeting discussion confirmed this is a POST/PUT operation deferred to Phase 2 along with other configuration management features

**Related discussion on firewall violation alerts (Phase 3+ consideration):**

**Devon's context:**
**Adam's future enhancement possibility:**
**Key Requirements:**

1. **URL/IP whitelist** - Customer provides list of approved URLs and/or IP addresses
2. **Applied within 2 hours** - Configuration uploaded at next device check-in
3. **Flexible list size** - Can handle 20-35+ URLs/IPs (proven with ATM customers)
4. **All devices can be configured** - Push same rules to entire fleet
5. **POST/PUT operation** - Customer pushes rules via API

**Use Cases:**

1. **Data management** - Restrict modem to only communicate with customer's machines
2. **Traditional vending integration** - Control what devices can access as they expand to traditional vending
3. **Security** - Prevent unauthorized network access
4. **Cost control** - Ensure data only used for legitimate business purposes
5. **Compliance** - Meet security requirements for restricted networks

**Real-world example (ATM customers):**
- 20-30 URLs/IP addresses whitelisted
- Only places on internet required for ATM transaction processing across entire US
- Proven configuration pattern that works at scale

**Technical Implementation:**

- Customer provides whitelist via API
- WATM stores configuration
- Configuration pushed to device at next check-in (every 2 hours)
- Device updates firewall rules
- Only whitelisted URLs/IPs allowed
- All other traffic blocked

**Current Limitations:**
- **No firewall violation alerts** - Device doesn't currently alert when blocking traffic
- Firmware could be enhanced to add this capability (InHand can modify)
- Alternative alarm: Ethernet tampering detection exists (plug/unplug events)

**Future Enhancement (Phase 3+):**
Request InHand to add firewall violation alert to firmware - would generate alarm when firewall drops traffic attempting to reach non-whitelisted destinations.

**Example request:**
```json
POST /devices/ABC123/firewall

{
  "policy": "whitelist",
  "allowed_urls": [
    "api.allpoint.com",
    "updates.allpoint.com",
    "telemetry.allpoint.com"
  ],
  "allowed_ips": [
    "192.168.1.100",
    "10.0.0.50"
  ],
  "apply_to_all": false,
  "notes": "Whitelist for kiosk communication"
}
```

**Example response:**
```json
{
  "serial_number": "ABC123",
  "firewall_update_requested": true,
  "status": "pending",
  "policy": "whitelist",
  "allowed_urls": ["api.allpoint.com", "updates.allpoint.com", "telemetry.allpoint.com"],
  "allowed_ips": ["192.168.1.100", "10.0.0.50"],
  "will_apply_at_next_checkin": true,
  "estimated_completion": "2025-11-18T18:30:00Z",
  "note": "Firewall rules will be applied at next device check-in (within 2 hours)"
}
```

**Batch update for fleet:**
```json
POST /devices/firewall/batch

{
  "device_serials": ["ABC123", "DEF456", "GHI789"],
  "policy": "whitelist",
  "allowed_urls": [
    "api.allpoint.com",
    "updates.allpoint.com"
  ]
}
```

**Phase 2 classification reasoning:**
- POST/PUT operation requiring device configuration
- Customer needs time to define whitelist requirements
- Not essential for MVP dashboard/support needs
- Grouped with other configuration management features
- Can manually configure via portal in meantime

#### 5. Manually Switch Carrier (Dual Carrier Modems)
**Endpoint:** `POST /devices/{serial_number}/carrier/switch`
- Trigger carrier switch for dual-carrier devices
- Specify target carrier
- Monitor switch completion

**Phase:** Post-MVP (Phase 2) - Confirmed during meeting

**Requirements:**

- **For dual-carrier devices only** - Single-carrier devices have no carriers to switch between
- **Manual trigger via API** - Customer initiates carrier switch
- **Specify target carrier** - Verizon → AT&T or AT&T → Verizon (or T-Mobile if configured)
- **Monitor completion** - API should confirm when switch is complete
- **Applied at next check-in** - Like other config updates, takes effect within 2 hours

**Use cases:**

1. **Poor signal on current carrier** - Switch to carrier with better signal strength
2. **Carrier outage/issue** - Failover to secondary carrier
3. **Testing/troubleshooting** - Verify both carriers are working
4. **Network optimization** - Move device to less congested carrier

**Technical considerations:**

- Only applicable to dual-carrier devices (need to check device configuration first)
- Switch command sent to device at next check-in
- Device performs carrier switch and reconnects
- Portal updated with new active carrier
- May cause brief connectivity interruption during switch

**Example request:**
```json
POST /devices/ABC123/carrier/switch

{
  "target_carrier": "Verizon",
  "reason": "Poor signal on AT&T",
  "requested_by": "support_rep_id"
}
```

**Example response:**
```json
{
  "serial_number": "ABC123",
  "switch_requested": true,
  "current_carrier": "AT&T",
  "target_carrier": "Verizon",
  "status": "pending",
  "will_apply_at_next_checkin": true,
  "estimated_completion": "2025-11-18T18:30:00Z",
  "note": "Switch will occur at next device check-in (within 2 hours)"
}
```

**Example status check:**
```json
GET /devices/ABC123/carrier/switch/status

Response:
{
  "serial_number": "ABC123",
  "active_carrier": "Verizon",
  "switch_status": "completed",
  "completed_at": "2025-11-18T18:15:32Z",
  "signal_strength": {
    "verizon": 65,
    "att": 42
  }
}
```

**Phase 2 classification reasoning:**
- Requires POST operation and device action
- More complex than simple data retrieval
- Customer needs to understand dual-carrier device behavior
- Not essential for MVP dashboard/support needs (can manually switch via portal)
- Grouped with other configuration management features

#### 6. Adjust Service Plan Tier / Adjusting Plan Tier
**Endpoint:** `PUT /devices/{serial_number}/plan`
- Change device tier programmatically
- Handle billing implications
- Validate tier options

**Phase:** Post-MVP (Phase 2) - Confirmed during meeting

**Requirements:**

- **Change tier programmatically** - Move device between tiers (Tier 1, Tier 3, Tier 5, etc.)
- **Billing implications** - Tier change affects pricing
- **Validation** - Ensure requested tier is valid/available
- **Immediate or scheduled** - Could apply immediately or at next billing cycle

**Use Cases:**

1. **Usage spike** - Temporarily move device to higher tier for month (as Mason mentioned happens occasionally)
2. **Permanent upgrade** - Customer business grows, needs higher data cap
3. **Downgrade** - Customer reduces usage, can move to lower tier
4. **Batch management** - Move multiple devices to same tier
5. **Cost optimization** - Adjust tiers based on actual usage patterns

**Related context from earlier discussion:**
Mason mentioned they occasionally have to **"up them to another tier for the month"** when there's a sudden spike in usage (like software updates).

**Technical considerations:**

- Tier change may not be immediate (could apply at next billing cycle)
- Need to handle prorated billing if mid-cycle change
- Should return current tier and new tier in response
- May need approval workflow for tier changes
- Track tier change history for auditing

**Example request:**
```json
PUT /devices/ABC123/plan

{
  "new_tier": "Tier 5",
  "effective_date": "2025-12-11",
  "reason": "High usage pattern",
  "requested_by": "admin_user_id"
}
```

**Example response:**
```json
{
  "serial_number": "ABC123",
  "tier_change_requested": true,
  "current_tier": "Tier 3",
  "current_cap_mb": 10240,
  "new_tier": "Tier 5",
  "new_cap_mb": 20480,
  "effective_date": "2025-12-11",
  "billing_impact": {
    "current_monthly_cost": 45.00,
    "new_monthly_cost": 65.00,
    "prorated_charge": 10.00
  },
  "status": "pending",
  "note": "Tier change will take effect at start of next billing cycle"
}
```

**Example batch tier adjustment:**
```json
PUT /devices/plan/batch

{
  "device_serials": ["ABC123", "DEF456", "GHI789"],
  "new_tier": "Tier 3",
  "effective_date": "2025-12-11",
  "reason": "Standard tier alignment"
}
```

**Phase 2 classification reasoning:**
- PUT operation requiring database modification
- Billing system integration required
- Not essential for MVP dashboard/support needs
- Can manually adjust via portal in meantime
- Grouped with other configuration management features

#### 7. Remote Configuration Updates
**Endpoint:** `PUT /devices/{serial_number}/config`
- Push custom configuration changes
- Support: DHCP, netmask, gateway settings
- Support: Port forwarding, VLAN configs
- Support: RS-232 serial port configuration (future need)

---

## Phase 3+ Features (Future Consideration)

### Advanced Features

#### 1. Daily Data Usage Breakdown / Data Usage Broken Down by Day
**Status:** HIGH PRIORITY - In current backlog, will be added to portal first then API

**Phase:** Phase 3+ (After Phase 2) - Portal feature first, then API

**Key Decisions:**

1. **DAILY data usage - FEASIBLE and valuable**
2. **HOURLY data usage - NOT FEASIBLE** (carrier limitations)
3. **High priority** - Devon just prioritized it in backlog
4. **Portal first, then API** - Build in portal, then expose via API
5. **10-11 years of experience** validates daily is sufficient

**Deliverable:**
- Return daily data usage for specified date range
- **NOT hourly** (carrier limitations prevent accuracy)
- Request parameters TBD (date range options to be determined when portal feature is built)

**Carrier limitations (why hourly is NOT feasible):**
- **Verizon "buildup protocol"** - Data usage may not be updated for 6-24 hours
- **Data sessions last weeks** - Can't wait for session termination to get accurate data
- **Bill cuts on live sessions** - Verizon performs periodic "bill cuts" every 6-24 hours
- **Discretionarily inaccurate** - Time windows for updates are not guaranteed
- **Daily is maximum practical granularity** - More granular doesn't add value

**Why daily IS sufficient (Devon's experience):**
- 10-11 years of experience across thousands of devices
- Daily usage is "super awesome tool" for identifying spikes
- More granular than daily never provided additional value
- Even AT&T's extra reporting doesn't justify sub-daily granularity

**Use Cases:**
- **Spike analysis** - Identify which day had unusual data usage
- **Pattern detection** - See usage trends over time
- **Troubleshooting** - Correlate high usage with events/updates
- **Billing verification** - Daily breakdown for customer billing questions
- **Capacity planning** - Understand usage patterns for tier planning

**Implementation approach:**
1. Build daily data usage feature in portal first
2. Design API parameters (date range options)
3. Expose via API endpoint
4. Customer can request X days of history or specific date range

**API Data document note:**
Original document listed this as "Feasibility Issue" stating "We cannot currently provide this. Verizon doesn't guarantee data reporting accuracy within a 6-hour window." Meeting clarified: **DAILY is feasible and in backlog, HOURLY is not feasible**.

**Example future API request:**
```json
GET /devices/ABC123/usage/daily?start_date=2025-11-01&end_date=2025-11-30

Response:
{
  "serial_number": "ABC123",
  "date_range": {
    "start": "2025-11-01",
    "end": "2025-11-30"
  },
  "daily_usage": [
    {
      "date": "2025-11-01",
      "usage_mb": 320
    },
    {
      "date": "2025-11-02",
      "usage_mb": 298
    },
    {
      "date": "2025-11-03",
      "usage_mb": 1450
    }
  ],
  "total_usage_mb": 15234,
  "note": "Daily granularity - hourly breakdowns not available due to carrier limitations"
}
```

**Phase 3+ classification reasoning:**
- Feature not yet built in portal (in backlog)
- Portal implementation required first
- API design depends on portal feature design
- High priority but after MVP and Phase 2 configuration features

#### 2. Webhook/Push Notifications
**Deliverable:**
- Customer registers webhook endpoint
- WATM pushes notifications on events
- Real-time alerts without polling

**Use cases:**
- Device goes offline
- Signal strength drops below threshold
- Data usage exceeds limit
- Security events (firewall violations)

**Implementation options:**
- Email notifications (existing framework)
- Webhook push to customer endpoint (preferred for real-time)

#### 3. Security Alerts
**Deliverable:**
- Alert if device attempts to reach unauthorized endpoints
- Alert on Ethernet tampering (plug/unplug events)
- Firewall violation alerts

**Current status:**
- Ethernet tampering alarm exists (portal handling incomplete)
- Firewall violation alerts need firmware enhancement
- Could request InHand to add this capability

#### 4. Heat Map Data
**Endpoint:** `GET /devices/signal-map` (aggregate endpoint)

**Deliverable:**
- Return all devices with signal strength data
- Customer builds heat map visualization
- Color-coded signal strength by geographic area
- Macro-level aggregated view

**Use case:** Visual dashboard like telecom network status (green/yellow/red zones)

---

## Features NOT Feasible

### 1. Public IP for Connected Devices
**Status:** ❌ NOT POSSIBLE

**Reason:**
- Verizon will not provide public IP information
- Devices NAT behind public IPs specific to cell tower/data center
- IPs subject to change without notice
- Verizon recommendation: whitelist ALL Verizon IPs (impractical)

**Alternative:**
- Could route traffic through specific paths if customer has central portal
- Requires separate conversation about specific use case

### 2. Hourly Data Usage Breakdown / Data Usage Broken Down by Hour
**Status:** ❌ NOT FEASIBLE (Daily is maximum)

### 3. Connection Speed Testing
**Status:** ❌ NOT FEASIBLE VIA API

**Reason:**
- Device has speed test capability but reporting is unreliable
- Difficult to extract and report back via API

**Alternative:**
- Customer could implement client-side speed test (speedtest.net)
- Customer devices run Linux/Raspberry Pi - can add speed test there

### 4. Track Data Through Endpoint
**Status:** ❌ SHELVED

**Reason:** Currently unable to see this data at all

---

### 5. Check for Modem Update / Date of Last Modem Update
**Status:** ❌ NOT NEEDED AS API ENDPOINT

**Why NOT needed as API endpoint:**

- **No manual check needed** - Devices handle this automatically
- **No customer action required** - Updates are managed by WATM
- **Always current** - By design, all devices should be up-to-date
- **Managed service model** - WATM controls when/how updates are deployed
- **Infrequent updates** - Not something customer needs to monitor

**Alternative approach:**
If customer needs update history for audit/compliance, this could be added as a **GET endpoint** returning update history:
- Date of last firmware update
- Current firmware version
- Update history log

**But this was NOT requested during meeting** - customer accepted that devices are always up-to-date and WATM manages updates.

**Related discussion - Standard vs Custom Configuration:**

**Adam's explanation:**
**Devon's confirmation:**
**Future consideration - RS-232 serial port:**

**Devon's example:**
**Adam's note:**
**Summary:**
- ✅ Devices auto-check every 2 hours
- ✅ Updates download automatically
- ✅ Always up-to-date by design
- ❌ No API endpoint needed for checking updates
- ❌ No customer action required
- ⏸️ Could add GET endpoint for update history if needed later (not requested)

**Customer configuration status:**
- **Current:** Standard configuration (no customizations)
- **Future:** May need custom config if RS-232 serial port integration required

---

