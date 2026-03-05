# API Development Scope - MVP vs Phase 2

## Document Overview
This document defines the scope and deliverables for the WATM API development, clearly separating MVP (Phase 1) features from Phase 2 enhancements. The phased approach prioritizes GET endpoints for MVP, with POST/trigger mechanisms deferred to Phase 2.

**Customer:** AllPoint (First pilot customer)
**Last Updated:** November 18, 2025 (Enhanced with detailed meeting discussions)
**Based on:** Meeting dated November 17, 2025

---

## MVP (Phase 1) - GET Endpoints Only

**Goal:** Provide read-only API access to existing portal data
**Timeline:** In internal testing, ready for customer testing soon
**Authentication:** API key-based via AWS API Gateway

### Core Principle
MVP returns **last reported data** from the portal - no real-time device pings or data modifications. All endpoints are GET requests that return cached/stored data.

---

## Feature-by-Feature Breakdown

### 1. Ping the Cell Modem

#### MVP (Phase 1) - GET Endpoint
**Endpoint:** `GET /devices/{serial_number}/status`

**What it returns:**
- Last reported connection status from device check-in
- Status values: Active, Suspended, Deactivated, Offline
- **Timestamp of last check-in** (CRITICAL - must be included)
- Device check-ins occur every 2 hours

**Data characteristics:**
- Returns stored data from database
- Could be from latest check-in (within 2 hours) OR from weeks ago if device offline
- Timestamp allows customer to determine data freshness

**Use case:**
- Support rep can see if device was online at last check-in
- If timestamp is old (e.g., 4 weeks), indicates device has been offline
- Can correlate with customer's issue timing

**Limitations:**
- Not real-time - shows last known status
- If device is currently offline, returns last successful check-in data
- No active ping to device

**Implementation:**
- Query existing check-in data from database
- Return JSON with status + timestamp
- Standard GET endpoint

**Example response:**
```json
{
  "serial_number": "ABC123",
  "status": "active",
  "last_check_in": "2025-11-18T14:30:00Z",
  "connection_status": "online"
}
```

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

#### MVP (Phase 1) - GET Endpoint
**Endpoints:**
- `GET /devices/{serial_number}` (includes signal strength in device details)
- `GET /devices/{serial_number}/signal` (dedicated signal endpoint)

**What it returns:**
- Last reported signal strength from device check-in
- Signal strength value (numeric, e.g., 47, 87)
- **Timestamp of when signal was measured** (CRITICAL)
- Carrier information (which carrier reported this signal strength)

**For dual-carrier devices:**
- Signal strength for both carriers
- Indicator of which carrier is currently active
- Example: "47 Verizon" vs "63 AT&T"

**Data characteristics:**
- Returns stored data from last check-in
- Updated every 2 hours (standard check-in frequency)
- Could be weeks old if device offline
- Timestamp is ESSENTIAL for data context

**Use case:**
- Support diagnoses poor connectivity
- Can see if device is in low-signal area (e.g., Faraday cage)
- With timestamp, can ask: "Have you moved device since [date]?"
- Even 4-week-old signal data is useful if timestamp provided

**Why timestamp is critical (Devon's point):**
**Limitations:**
- Not real-time signal strength
- Accuracy: GPS coordinates are cellular triangulation (0.5-1 mile radius)
- Shows last known signal, not current

**Implementation:**
- Query device check-in logs from database
- Return signal strength + timestamp + carrier
- Include both carriers for dual-carrier devices

**Example response:**
```json
{
  "serial_number": "ABC123",
  "signal_strength": {
    "primary": {
      "carrier": "Verizon",
      "strength": 47,
      "timestamp": "2025-11-18T14:30:00Z",
      "active": true
    },
    "secondary": {
      "carrier": "AT&T",
      "strength": 63,
      "timestamp": "2025-11-18T14:30:00Z",
      "active": false
    }
  },
  "is_dual_carrier": true,
  "last_check_in": "2025-11-18T14:30:00Z"
}
```

---

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

## Additional MVP Features (Phase 1)

### 3. Get Serial Number of Modem
**Endpoint:** `GET /devices` or `GET /devices/{serial_number}`

**Deliverable:**
- Return device serial number in all device endpoints
- Include in device list and individual device details
- Use as primary identifier for device-specific queries

**Implementation:**
Serial number serves as the **primary identifier** throughout the API:

1. **Returned in bulk device list:** `GET /devices`
   - Response includes array of devices with serial numbers
   - Allows customer to iterate through all their devices

2. **Used as path parameter:** `GET /devices/{serial_number}`
   - Serial number identifies specific device for detailed queries
   - Used across all device-specific endpoints (status, signal, location, etc.)

3. **Fundamental to API design:**
   - No debate needed - obvious must-have
   - Essential for device identification and tracking
   - Matches customer's device management needs

**Example responses:**

**Bulk list:**
```json
{
  "devices": [
    {
      "serial_number": "ABC123",
      "company_name": "AllPoint",
      "status": "active"
    },
    {
      "serial_number": "DEF456",
      "company_name": "AllPoint",
      "status": "active"
    }
  ],
  "total": 2
}
```

**Individual device:**
```json
{
  "serial_number": "ABC123",
  "device_data": {...},
  "location_data": {...},
  "plan": {...}
}
```

---

### 4. Current Health Status
**Endpoint:** `GET /devices/{serial_number}/status`

**Deliverable:**
- Return both primary and secondary status **separately** (not concatenated)
- Status values: Active, Suspended, Deactivated, Offline
- Example: `"primary_status": "active"`, `"secondary_status": "offline"`
- Allow customer to parse/use as needed (concatenate on their side if desired)
- Include last check-in timestamp

**Why separate values (Aksana's explanation):**
**Benefits of separate values:**
- Maximum flexibility for customer
- Can use one status field or both
- Can concatenate on their side if needed ("active/offline")
- Can apply custom business logic
- Not locked into WATM's formatting
- Aligns with API philosophy of returning raw data

**Example response:**
```json
{
  "serial_number": "ABC123",
  "status": {
    "primary_status": "active",
    "secondary_status": "offline",
    "last_check_in": "2025-11-18T14:30:00Z"
  }
}
```

---

### 5. See What Carrier the Modem is Using
**Endpoint:** `GET /devices/{serial_number}`

**Deliverable:**

**For single-carrier devices:**
- Return configured carrier (Verizon, AT&T, T-Mobile)
- Even if no check-in data exists

**For dual-carrier devices:**
- Return both configured carriers
- Indicate which carrier last check-in came from
- Show signal strength per carrier (from browse devices screen)
- Flag device as dual-carrier

**Technical consideration:**
Portal currently stores this data and displays on browse devices screen. May need to add to device detail page if not already there.

**Example responses:**

**Single-carrier device:**
```json
{
  "serial_number": "ABC123",
  "carrier_info": {
    "is_dual_carrier": false,
    "carrier": "Verizon",
    "configured_via": "device_config"
  }
}
```

**Dual-carrier device:**
```json
{
  "serial_number": "DEF456",
  "carrier_info": {
    "is_dual_carrier": true,
    "configured_carriers": ["Verizon", "AT&T"],
    "active_carrier": "AT&T",
    "last_check_in_carrier": "AT&T",
    "signal_per_carrier": {
      "verizon": 47,
      "att": 63
    }
  }
}
```

---

### 6. Get Location Information
**Endpoint:** `GET /devices/{serial_number}/location`

**Deliverable:**

**Return GPS coordinates (cellular triangulation):**
- Latitude/longitude of last known location
- **DO NOT** return manually entered address (customer has this already)
- Include timestamp of last location update (TBD - need to verify if stored)

**Important limitations:**
- **Not pinpoint GPS** - accuracy up to 0.5-1 mile radius
- Uses cellular tower triangulation
- Intended for general location verification

**Key Decisions Made:**

1. **Return GPS coordinates (cellular triangulation)** - NOT manual address
2. **Customer already has address** in their own system - no value in returning it
3. **Accuracy is 0.5-1 mile radius** - customer understands and accepts this
4. **Use case:** Compare coordinates to expected location to detect major discrepancies
5. **Timestamp of last location update:** TBD - need database investigation

**Use case clarified:**
- Customer has installation address in their system
- Get coordinates from WATM API
- Compare coordinates to expected location
- Half-mile variance acceptable (GPS accuracy limitation)
- 10-100 mile variance indicates device moved or problem
- Real-world use: Adam successfully tracked down lost box in North Jersey using this method

**TBD item (IMPORTANT - "Date of last location update"):**

This was discussed twice in the meeting - first in the location context, then explicitly when reviewing the requirements list.

**First discussion (during location conversation) - Adam's question:**
**Devon's response:**
**Aksana's commitment:**
**Second discussion (when reviewing requirements list) - Aksana's clarification:**
**Devon and Adam's clarification:**
**Status Summary:**
- **TBD (To Be Determined)** - not confirmed if stored in database
- **Action:** Assigned to **Richard and Stone** to investigate database schema
- **May need to add this field** if not currently tracked as separate timestamp
- **Critical importance:** For knowing if coordinates are current or from weeks ago (same issue as signal strength timestamp)
- **Without timestamp:** Coordinates could be from weeks ago if device offline, misleading without context

**Technical consideration:**
- Portal shows map with cached coordinates
- Need to determine if timestamp exists for when map was last updated
- For offline devices, last known location still valuable **IF** timestamp provided
- If timestamp not stored: Need to add this to database schema before exposing via API

**Example response:**
```json
{
  "serial_number": "ABC123",
  "location": {
    "latitude": 40.7128,
    "longitude": -74.0060,
    "accuracy_radius_meters": 800,
    "method": "cellular_triangulation",
    "last_updated": "2025-11-18T14:30:00Z",
    "notes": "Accuracy 0.5-1 mile radius. Not pinpoint GPS."
  }
}
```

**What NOT to include:**
```json
{
  "address": "123 Main St, City, State",
  "manual_address": "..."
}
```

**Note:** DO NOT RETURN address or manual_address fields - Customer already has this data in their system.

---

### 7. Projected Data Usage/Price
**Endpoint:** `GET /devices/{serial_number}/billing` or include in device details

**Deliverable:**
- Return projected usage price for current billing cycle
- Include extrapolated calculation based on current usage and cycle position
- Show raw data so customer understands the math

**Current Status:**
- **Already exists in portal** - displayed on device page
- Data is already calculated and stored
- Straightforward to expose via API

**How it works (Devon's explanation):**
**Calculation logic:**
- Takes current data usage
- Divides by days elapsed in billing cycle
- Multiplies by total days in cycle to project final usage
- Applies pricing tier to projected usage to calculate projected price
- Example: 5GB used in 10 days of 30-day cycle → projected 15GB total → apply Tier 3 pricing

**Implementation:**
- Query existing billing calculation from portal database
- Return both current and projected values
- Include billing cycle context (dates, days elapsed/remaining)
- Let customer see the raw data behind the projection

**Use case:**
- Customer can see if device is trending toward exceeding tier cap
- Support can identify devices with abnormally high usage patterns
- Billing forecasting for cost management

**Example response:**
```json
{
  "serial_number": "ABC123",
  "billing": {
    "current_usage_mb": 5120,
    "billing_cycle": {
      "start_date": "2025-11-11",
      "end_date": "2025-12-10",
      "days_elapsed": 7,
      "days_remaining": 23
    },
    "projected_usage_mb": 15360,
    "projected_price": 45.00,
    "current_tier": "Tier 3",
    "tier_cap_mb": 10240
  }
}
```

---

### 8. Data Usage (includes "Percent utilization for period")
**Endpoint:** `GET /devices/{serial_number}/usage`

**Phase:** MVP (Phase 1) - Confirmed during meeting

**Deliverable:**
- Return current data usage for billing period
- Return service plan/tier information (e.g., "Tier 1", "Tier 3")
- Return tier data cap (e.g., Tier 1 = 3GB, Tier 3 = 10GB)
- **DO NOT** pre-calculate percentage - provide raw values
- Customer will calculate percentage on their end

**MVP Confirmation:**
When reviewing the requirements list after discussing "Adjusting plan tier" (a Phase 2 feature), Aksana explicitly confirmed:

**Aksana's statement:**
This simple statement confirmed that data usage is a core GET endpoint for Phase 1, in contrast to the tier adjustment feature which was classified as Phase 2.

**Key Decisions Made:**

1. **Do NOT pre-calculate percentage** - WATM doesn't currently store/display as percentage
2. **Return raw values:** Current usage + tier + cap
3. **Customer calculates percentage** on their end
4. **Flexibility:** Handles devices on different tiers gracefully
5. **Service plan included in response** so customer knows which tier/cap to use

**Why not pre-calculate percentage:**
- WATM doesn't currently store it as percentage
- Most customer devices on same tier (but exceptions exist for usage spikes)
- Customer can handle logic: "if tier = Tier1 (3GB cap), calculate % = usage/3GB"
- Provides flexibility for different business logic
- Raw data allows customer to handle tier variations dynamically
- Example: Some devices temporarily moved to higher tier for software updates

**Customer's actual usage pattern:**
- **Typical:** All devices on same plan
- **Exception:** Occasional devices moved to higher tier temporarily due to usage spikes
- Customer needs to handle both scenarios

**Implementation:**
Simple - return current usage (MB), tier name, and tier cap (MB). Customer does:
```
percentage = (current_usage_mb / tier_cap_mb) * 100
```

**Example response:**
```json
{
  "serial_number": "ABC123",
  "usage": {
    "current_usage_mb": 5120,
    "billing_cycle_start": "2025-11-11",
    "billing_cycle_end": "2025-12-10",
    "service_plan": {
      "tier": "Tier 3",
      "data_cap_mb": 10240,
      "tier_description": "10GB plan"
    }
  }
}
```

**Customer calculation:**
```
5120 MB / 10240 MB = 50% utilized
```

**Handles edge case (device moved to different tier):**
```json
{
  "serial_number": "DEF456",
  "usage": {
    "current_usage_mb": 8500,
    "service_plan": {
      "tier": "Tier 5",
      "data_cap_mb": 20480,
      "tier_description": "20GB plan"
    }
  }
}
```

**Note:** This device was temporarily upgraded to Tier 5 due to usage spike.

Customer system reads tier dynamically and calculates: `8500 / 20480 = 41.5% utilized`

---

### 9. Private IP for Connected Devices
**Endpoint:** `GET /devices/{serial_number}/network`

**Phase:** MVP (Phase 1) - Confirmed during meeting

**Deliverable:**
- Return private IP address of connected device
- Already available in current build
- Straightforward GET endpoint

**Implementation:**
No special considerations needed - data is already available in the system, just needs to be included in API response.

**Use Cases:**
- Network troubleshooting
- Verify device network configuration
- Confirm device connectivity details
- Support diagnostics

**What it returns:**
- Private IP address assigned to the connected device
- Optionally: Gateway, subnet mask, DNS servers, network configuration details

**Note:** This is **private IP**, not public IP. Public IP was discussed separately and determined to be not feasible due to Verizon limitations (see "Features NOT Feasible" section).

**Example response:**
```json
{
  "serial_number": "ABC123",
  "network": {
    "private_ip": "192.168.1.105",
    "gateway": "192.168.1.1",
    "subnet_mask": "255.255.255.0",
    "dns_primary": "8.8.8.8",
    "dns_secondary": "8.8.4.4",
    "dhcp_enabled": true
  }
}
```

**Contrast with Public IP:**
- ✅ **Private IP** - Returned by API (this feature)
- ❌ **Public IP** - NOT possible (Verizon restrictions, see Features NOT Feasible section)

---

### 10. Cycle Window Dates
**Endpoint:** Include in billing/usage endpoints

**Deliverable:**
- Return billing cycle dates (11th - 10th)
- These are hardcoded and rarely change
- Can be included but low value since consistent

**Implementation:**
Return billing cycle start/end dates in billing and usage endpoints. Even though dates are predictable (11th-10th), including them makes the API response complete and self-contained.

**Example (already included in billing/usage responses):**
```json
{
  "billing_cycle": {
    "start_date": "2025-11-11",
    "end_date": "2025-12-10",
    "window": "11th - 10th"
  }
}
```

**Why include despite being static:**
- API completeness - all billing context in one response
- Future-proofing - if cycle ever changes, no customer code updates needed
- Self-documenting API - customer doesn't need external documentation for cycle dates
- Minimal cost - trivial to include in existing responses

---

## MVP Summary - What Customer Gets

**All MVP endpoints are GET requests that return stored/cached data:**

1. ✅ Device list and device details
2. ✅ Last reported status with timestamp
3. ✅ Last reported signal strength with timestamp
4. ✅ Carrier information (single/dual carrier)
5. ✅ Location coordinates
6. ✅ Current and projected data usage
7. ✅ Service plan/tier information
8. ✅ Private IP address
9. ✅ Billing cycle dates
10. ✅ Health status (primary/secondary separate)

**What MVP does NOT include:**
- ❌ Real-time device pings
- ❌ Real-time signal strength checks
- ❌ POST/PUT operations (no data modification)
- ❌ Webhook/push notifications
- ❌ Device configuration changes
- ❌ Firewall rule updates
- ❌ Carrier switching
- ❌ Tier adjustments

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

## MVP Implementation Details

### Authentication & Security
- **API Key Management Portal** in WATM application
- API access enabled per company by WATM super admins
- Company super admins manage their own keys
- AWS API Gateway for rate limiting and security

**Key Management Features:**
- Create new keys
- Edit existing keys
- Revoke keys
- View key details

### Documentation & Testing
- **OpenAPI/Swagger documentation**
- **Interactive sandbox environment**
- Downloadable YAML file
- Schema documentation for all endpoints
- Example responses
- HTTP status code documentation

### Sandbox Features
- Try API calls directly from portal
- Authorize with API key
- See actual response payloads (test data)
- Test pagination, filters, parameters

---

## Customer Integration Patterns

### Batch/Scheduled Pulls
**Frequency:** Once or twice daily
**Use case:** General dashboard updates, bulk synchronization
**Endpoints:** `GET /devices` with pagination

### On-Demand Pulls
**Priority:** MORE IMPORTANT than scheduled pulls
**Use case:** Support rep gets call → immediately lookup device
**Endpoints:** Individual device endpoints by serial number

### Event-Based Integration
**Use case:** Machine boots with new modem → trigger API call
**Implementation:** Customer-side event handlers
**Endpoints:** Various device detail endpoints

---

## MVP Success Criteria

### Technical
1. ✅ AllPoint can authenticate with API key
2. ✅ All device data accessible via GET endpoints
3. ✅ Response times meet performance requirements
4. ✅ Data accuracy matches portal data
5. ✅ Timestamps included for all time-sensitive data

### Business
1. ✅ AllPoint can build their dashboard with API data
2. ✅ Support team can perform on-demand lookups
3. ✅ No security or data leakage issues
4. ✅ Documentation clear and complete
5. ✅ Sandbox testing successful

### Customer Feedback
1. ✅ Identify missing data/fields
2. ✅ Validate data format and structure
3. ✅ Confirm API meets immediate needs
4. ✅ Prioritize Phase 2 features based on usage

---

## Development Timeline

### Current Status: MVP in Internal Testing
**Next Steps:**
1. Complete internal testing and minor cleanups
2. Deploy to production with access enabled only for AllPoint
3. AllPoint tests in sandbox environment
4. Gather feedback on missing data/fields
5. Iterate based on feedback
6. Begin Phase 2 planning based on MVP usage patterns

### Phase 2 Timeline
- To be determined based on MVP feedback
- Prioritize real-time features (ping, signal refresh) first
- Add configuration management features incrementally
- Consider customer's actual usage patterns

---

## Key Takeaways

### MVP Philosophy
- **Read-only access** to existing portal data
- **No real-time device interactions** (requires infrastructure for Phase 2)
- **No data modifications** (POST/PUT deferred to Phase 2)
- **Fast to implement** - leverages existing database queries
- **Low risk** - no device impact, only reads cached data

### Why Two Phases?
1. **Technical complexity:** Real-time pings require async processing, device communication, timeout handling
2. **Infrastructure:** POST operations need queue systems, webhook support
3. **Testing:** GET endpoints safer to test and deploy first
4. **Customer validation:** Ensure data format/content meets needs before building complex features
5. **Incremental delivery:** Get value to customer faster, iterate based on feedback

### Critical Success Factor
**Always include timestamps** with time-sensitive data (status, signal strength, location) so customer can determine data freshness and make informed decisions.
