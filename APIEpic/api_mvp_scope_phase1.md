# API Development Scope - MVP (Phase 1)

## Document Overview
This document defines the MVP (Phase 1) scope for the WATM API development. The MVP focuses on GET endpoints only, providing read-only access to existing portal data.

**Customer:** AllPoint (First pilot customer)
**Last Updated:** November 18, 2025
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
