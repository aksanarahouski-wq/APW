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
> "If device isn't even online and that's the problem, pulling a signal strength that might be from, let's say weeks ago, showing 100%, that's going to be misleading. Even if you can only give me signal strength from four weeks ago, but you at least tell me it's from four weeks ago, that at least allows us to convey that."

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

**Meeting Discussion:**
This feature was demonstrated during Aksana's sandbox demo and confirmed as fundamental to the API design.

**Aksana's Demo (showing bulk device list):**
> "You can use a sandbox to kind of see what comes back. Like for this company, it has some devices. So like in this case, it returned this array, right? And this is what come back, like when you get them out of bulk, right? Give me all, right? **Serial number, company name status.**"

**David's follow-up:**
> "Then you can pull it by serial number."

**Aksana's confirmation:**
> "So in this case, let's say I will take one of these. I'll take this guy. **You could pull by serial number, and then it gives you a device.** This JSON that comes back that has device data, location data, plan."

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

**Meeting Discussion:**
Aksana asked whether to return status concatenated (as portal shows) or separately. Team consensus was **separate values**.

**Mason's use case:**
> "Yeah, so that gives us a quick glance, just like, hey, is this machine online or not, before we click into it and ping it to see what our signal strengths are."

**Devon's recommendation:**
> "I would suggest taking both statuses and then you guys can parse it if you want to or not. But I would suggest taking the additional status because like Mason said, I mean, if you, you know, getting it, getting a status that says active versus active slash offline tells you a lot right off the bat."

**David's confirmation:**
> "Yep, I'm happy to get it as two separate pieces of information."

**Why separate values (Aksana's explanation):**
> "It's easier that way because then you get to do whatever, you get to use one, use both, use none, right, like concatenate them, put any condition around them rather than us kind of pre-packaging it for you and then you kind of stack with pre-packaged version."

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

**Meeting Discussion:**
This discussion revealed technical considerations for handling both single and dual-carrier devices.

**Aksana's question (explaining current data model):**
> "Carrier, the modem is using. Okay, so that's the one. So today, Adam and Devin, you know, we just did this, like, uplift. Basically, today on a device, we're storing, basically SIM and whether which SIM is active and like so active SIM tells us what carrier it is, right?"

**Devon's clarification:**
> "We are going to have to look at, we're going to, Exana, we're going for their, I'm fairly certain you guys have dual carrier devices."

**Adam's MVP recommendation:**
> "We probably just want to, I think for launch, kick back the carrier that the last check-in came from for them. Like, you know, if you go to the browse screen where you can see a check-in from a high level, it indicates the signal strength based on which carrier that sent that."

**Devon's explanation of data source:**
> "That's going to be on the browse devices screen, Exana. We should have that in the device page itself. We maybe need to look at adding that. Yeah, like on the signal strength where it says like **47 Verizon compared to 63 AT&T**. So that tells you exactly which one of your duals is actually, you know, being utilized."

**Aksana's confirmation:**
> "So whatever, however, this decision is made, that's what we want to map to the carrier."

**Devon's important edge case (devices without check-in data):**
> "Now, having said that, if there's a device that has no check-in data, thus, you know, up to this point, you may all, we may also want to, you know, look at, you know, like you, let's say it's just a single carrier device. Well, you know what carrier it is in that sense. You don't need to look at it. You don't need to look at a check-in to know which carrier that is."

**Devon's conclusion on dual-carrier needs:**
> "Now, it's just specifically with the dual carrier, if they want to know, not just that it's dual carrier, but they also want to know, okay, of Verizon and AT&T, or if they eventually get Verizon and T-Mobile, which carrier is it actually online right now? Gotcha. So we probably, we might want to pull both."

**Implementation Requirements:**

1. **Data Model:** System stores SIM data with active SIM indicator
2. **For MVP:** Return carrier from last check-in
3. **Data Source:** Browse devices screen shows signal per carrier (e.g., "47 Verizon" vs "63 AT&T")

**Two scenarios to handle:**

**Scenario 1: Single-carrier device (no check-in data)**
- Return configured carrier from device configuration
- Don't need check-in data to know carrier
- Example: Device just provisioned, never checked in

**Scenario 2: Dual-carrier device**
- Return BOTH configured carriers (e.g., Verizon AND AT&T)
- Show which carrier last check-in came from (the active one)
- Customer wants to know: "Which carrier is it actually online right now?"
- Include signal strength per carrier if available

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

**Meeting Discussion:**
This discussion clarified exactly what location data the customer needs and explicitly ruled out returning manual address data.

**Aksana's initial question:**
> "Carrier location. So location is going to be the address, right?"

**Devon's critical clarification:**
> "I apologize, Exona. Can we go back to the location for a second? This could be one of two things. This could be if they are looking for, you know, their manually entered address information, which, you know, I'm not sure if that's even helpful because they probably already have that on their side. Or if they're looking for the last known coordinates of the device's actual, you know, **cellular triangulation, cell tower triangulation**."

**David's clear preference:**
> "Personally, for me, Jeff, correct me if you have a difference of opinion, but I want the **current coordinates**."

**Mason's agreement:**
> "Yep, I agree with you guys."

**Adam's critical accuracy warning:**
> "And also just to just to be just so you are aware again, just making sure the team all knows our cell, our coordinates that we pull is **not GPS. It is not pinpoint. It is can be anywhere up to a half a mile or even in some cases closer to a mile**."

**David's acknowledgment:**
> "Yep, I get it. Yeah. **It's just a tool to get you in the ballpark if you're just, you know, scratching your head wondering where the hell box is.**"

**Adam's real-world example:**
> "I tracked down one of their boxes that they lost the other day. That's right. Yeah. Over in North Jersey."

**Adam's follow-up question (timestamp requirement):**
> "And then so touching on the location, there's another request to have the **date data for the last location update, which I don't know if we even keep that** like as its own piece of data anywhere in the database."

**Devon's technical consideration:**
> "Right now. So we'd have to kind of think about that internally, which is definitely TBD. Yeah, like, if we go into a device, Exana, and there's already like, the map is already loaded. Is there something cash[ed] there that we know when that when the last time that was updated, because obviously, we can have it so that we go the the API is to refresh it. And then we know it's real time. But if a device is not online, but they're still looking for that last known location, we'd have to look and see if we can pull that."

**Aksana's response:**
> "Okay, yeah, we could definitely let's look at the database to see how we store that. Okay, and then we can, again, we could too pull actual physical location, like as an address, and then the coordinates too, like it could be either or both."

**Devon's pushback on address field:**
> "I don't know. Is that even valuable? Is that valuable at all to you guys to pull the Xana? Are you referring to the address field that is in our portal currently? Yes, yes, yes. So is that it? Would that even be valuable to you guys? Because if you know, if you know everything else, you know, serial number, whatever your piece of equipment, whatever, like, is a piece of information regarding address. **That you would have had to type in there. Is that even valuable for you to pull it out?**"

**David's definitive answer:**
> "**I don't think so, because we have the address that the machine was installed in our system. Right. So to be able to get the coordinates from you. Yeah. And then be able to compare it to what we have.** Yes. And then be able to detect and say, you know, I understand there may be a half mile circle or a mile circle around where there's some question marks."

**Devon's confirmation of use case:**
> "**But if it's 10 miles away or 100 miles away, right?** Right. Right. Yeah. Right. OK. OK. Sounds good on that one."

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
> "And then so touching on the location, there's another request to have the **date data for the last location update, which I don't know if we even keep that** like as its own piece of data anywhere in the database."

**Devon's response:**
> "Right now. So we'd have to kind of think about that internally, which is definitely TBD."

**Aksana's commitment:**
> "Okay, yeah, we could definitely let's look at the database to see how we store that."

**Second discussion (when reviewing requirements list) - Aksana's clarification:**
> "Okay um so date of last location update and i'm not sure what this like tbd push is that like just for later is that what it means or..."

**Devon and Adam's clarification:**
> "That was what we were just talking about, Oksana. We're not, we're not, yeah, we have to get with store that as its own piece of data anywhere in the. **Yeah, we got to get with Richard and Stone on that one.** All right, sounds good."

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
  "address": "123 Main St, City, State",  // ❌ DO NOT RETURN - Customer has this
  "manual_address": "..."  // ❌ DO NOT RETURN - No value to customer
}
```

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
> "It's basically, and I hope the team understands just how that number is kind of extrapolated. You know, obviously, it's pretty, it's a pretty basic kind of just, you know, mathematic calculation based on where we're at in the billing cycle."

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
> "**Data usage is going to be an MVP.**"

This simple statement confirmed that data usage is a core GET endpoint for Phase 1, in contrast to the tier adjustment feature which was classified as Phase 2.

**Meeting Discussion:**
The substantial discussion about "Percent utilization for period" clarified that WATM should NOT pre-calculate percentages - customer will do the math themselves.

**Adam's explanation of current state:**
> "Yeah, so if I remember correctly, this was kind of presented as like, you know, a numeric representation of like, okay, we've used 50% of data, which **we do track technically, but we don't store or display it as a percentage right now**, which is why, again, I'm just not entirely sure."

**Adam's suggested approach:**
> "Well, if we can get the account that if we can get the entire account to have some consistency or uniformity with respect to which tier they use, I'm not sure if that is or isn't that way today, then they can set it up where **they read in the data usage and they can make their own calculation** based on if everything's on tier three, then you know it's out of 10 gigs."

**Devon's clarification question:**
> "You know it's out of 10 gigs. A percentage of 10 gigs, is that what you're saying?"

**Devon's suggestion:**
> "I mean, we can certainly look at making that calculation, or we could just get all on the same page about what usage, **the usage you're reading in is a percentage of a cap that is predefined**."

**Aksana's key insight (raw data approach):**
> "Yeah, because if you get basically **your max plan, right, and then you are at whatever, two gigs out of 10, you could do your own math to calculate percent as long as you kind of, you have both values**."

**David's important context question:**
> "Right, correct me if I'm wrong, Jeff, everyone on our, or Mason, **everyone on our system uses the same plan, right?**"

**Mason's answer (mostly uniform, but not always):**
> "Yeah, **for the most part, unless there's some sort of sudden spike in usage and we have to up them to another tier for the month or whatever, but yeah, they're all on the same plan**."

**Devon's final solution (return tier in API):**
> "And you can even go further with that, too. And you could even because **we're going to give you the service plan. We can that can be a part of the get anyway.** So like, you could say if you had all of your stuff on tier one, which is a three gig cap, but then you have a handful of stuff that you guys have had to move up because of software updates or whatever, then you can just have something in your system that's like, okay, **reading in the tier, it's now on tier three, which is out of 10 gigs, and then create your own calculation of percent, we know the tier, we know the utilization, correct, generate that**."

**Aksana's agreement:**
> "Okay."

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
      "tier": "Tier 5",  // Temporarily upgraded
      "data_cap_mb": 20480,
      "tier_description": "20GB plan"
    }
  }
}
```

Customer system reads tier dynamically and calculates: `8500 / 20480 = 41.5% utilized`

---

### 9. Private IP for Connected Devices
**Endpoint:** `GET /devices/{serial_number}/network`

**Phase:** MVP (Phase 1) - Confirmed during meeting

**Deliverable:**
- Return private IP address of connected device
- Already available in current build
- Straightforward GET endpoint

**Meeting Discussion:**
This was one of the briefest discussions - just a quick confirmation that the data already exists.

**Aksana's question:**
> "Data usage is going to be an MVP. **Private IP for connected devices. Is that pulling it?**"

**Devon's confirmation:**
> "Yeah, **we're already giving them that. We have it.**"

**Aksana's acknowledgment:**
> "Okay, sounds good."

**Key Points:**

1. **MVP feature** - Simple GET endpoint
2. **Data already exists** - No development needed, just expose via API
3. **Quick confirmation** - Shortest discussion in the meeting (3 lines)
4. **Straightforward** - No debate, no complications
5. **Ready to go** - Already have the data in portal

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

**Meeting Discussion:**
This was a very brief discussion acknowledging the dates are static and don't change.

**Aksana's observation:**
> "Okay, then cycle window dates. So I see the node because it is, it's **hard coded**. Is it just the window dates? Well, we, we could return that definitely, but like **to your point, this is not something that's changing**."

**Key Points:**

1. **Hardcoded value:** The cycle window is always 11th - 10th of each month
2. **Static data:** This almost never changes
3. **Can return it:** Aksana confirmed they can include it in API response
4. **Low priority:** Since it's hardcoded, not critical to return via API

**From API Data document:**
> "Our cycle window is the 11th - 10th. We can add this, but it will almost never change, so hardcoding is an option."

**Decision:**
- **Include in API responses for completeness** - makes API self-documenting
- **Customer may choose to hardcode on their side** - since value is static
- **Low effort to implement** - just return the hardcoded dates
- **Benefit:** Customer doesn't need to maintain hardcoded value if it ever changes

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

**Meeting Discussion:**
This feature was discussed in two contexts: first when talking about usage patterns, then when reviewing the requirements list.

**Mason's initial description of use case:**
> "I never really thought about kind of the timeframe of it. I know, I think it would be **more event based**. And my thinking like, hey, **we just plugged in a cell modem to this machine. Maybe it's on the boot up of that machine. It connects and is able to sync over or pull some of that information from it**. I think it would be nice to have some sort of way to, at least maybe, and it wouldn't be to the router, but to the website to **push like, hey, this router is connecting to this machine**. We're going to push the router or connect that router to. What machine it's connected to, but also on the web portal side, connect it to on, like, AllPoint's website, being able to put that account that it's linked to on IQ Tech."

**Adam's explanation of MAC address as identifier:**
> "I feel like there was something stated in a previous conversation, Mason, I can't remember if it was with you or with John, where there was a reference to **utilizing the MAC address** to kind of, because **our devices can kind of pull the MAC address of your kiosks automatically**, and if you guys were kind of **pushing back to the portal of the MAC address**, it might help either **synchronize or organize things** in that regard. **It's like an identifier**. So I know that was something that we did establish could be like a **push API, where basically you're sending the portal the MAC address** for one reason or another of the kiosk that either the wireless box is on or even like a wireless box that you might be trying to find in the event you can't find one."

**When reviewing requirements list - Aksana:**
> "Yeah. OK, this one modems are assigned to a machine."

**Devon's phase classification:**
> "That would be a put, that would be a put, right? Or a push that we're talking about maybe in, you know, **post, yeah, post MVP**."

**Aksana's agreement:**
> "Okay."

**Key Requirements:**

1. **Event-based trigger:** When machine boots up with new modem
2. **MAC address as identifier:** WATM devices can automatically pull MAC address from customer's kiosks
3. **Push API (POST endpoint):** Customer pushes MAC address to WATM portal
4. **Synchronization:** Links modem to specific machine in both systems
5. **Use cases:**
   - New modem plugged into machine
   - Associate modem with customer's machine/account
   - Help locate wireless boxes

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

**Meeting Discussion:**
This feature had substantial discussion covering use cases, capabilities, and related security concerns.

**Initial phase confirmation - Aksana:**
> "Okay, so this one, carrier switch, also sounds like it's a post. **Push firewall setting, same thing**. Object, so update post, post, so we're just **kind of pushing post for now until, for after MVP**."

**Mason's use case (data management for traditional vending):**
> "And with that, the idea kind of behind that is, as we push out more of these down the road, we're looking to get these involved in the traditional side of things. And so that's where it kind of becomes hit or miss and what the cell modems could potentially be used for. And so I guess the idea behind that would be **setting up firewall restrictions so that the modems can only be used for our machine so we can help manage that data better on our side of things**."

**Adam's capability confirmation:**
> "And we can certainly put **any type of firewall rules in that you want**. I mean, you could give me a list of **35 URLs or IPs** that you say, well, this is everything possible we could ever need. And then that **configuration can be uploaded to every single one of your devices within two hours**. I mean, **as soon as they check in, they'll take that update and they'll have updated firewall settings**. So we have something similar like that for our **ATM customers where they have a list of about, I think it's 20 to 30 combination of URLs and IP addresses** that are basically the only, you know, 20 or 30 places on the whole internet that are actually required to support any ATM processing, any transaction anywhere in the United States."

**Related discussion on firewall violation alerts (Phase 3+ consideration):**

**Devon's context:**
> "So, this goes into, Oksana, this is, again, a bigger conversation that we'll have to have, but we can, you know, we can obviously do, like, **a firewall for you guys**, like Adam said earlier, you know, and then you just kind of know that the **devices are only able to reach those hosts**. We **don't get any kind of alerts right now if the device detects that it's trying to reach something outside of that firewall and is blocking that traffic**. We are, however, utilizing some pretty neat alarm features that are built into the firmware of the devices."

**Adam's future enhancement possibility:**
> "I mean, to be honest, knowing the way that our current, you know, kind of, like, inventory of configurable alarms operates, **I don't think it would be out of reason that InHand could just modify the firmware to allow an alarm to be generated in the event the firewall is dropping traffic**. Sure. That doesn't seem to be just knowing, wow, I, you know, do know about the way it all works. I don't think that would be difficult for them. So if it is something that you guys feel in the future would be of great benefit, certainly have that conversation."

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

**Meeting Discussion:**
This was briefly discussed when reviewing the requirements list.

**Aksana's observation (grouping POST operations):**
> "Okay, so this one, **carrier switch, also sounds like it's a post**. Push firewall setting, same thing. Object, so update post, post, so we're just **kind of pushing post for now until, for after MVP**."

**Key Points:**

1. **POST operation** - Not a GET, requires action/trigger
2. **Post-MVP** - Deferred to Phase 2 along with other POST/PUT operations
3. **Grouped with other configuration changes** - Firewall settings, device assignment, etc.

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

**Meeting Discussion:**
This was a very brief discussion confirming it's a push operation for Phase 2.

**Aksana's question:**
> "Okay, so then there's another bucket. So **adjusting plan tier**. So that would be, **that's a push, right?**"

**Devon's confirmation:**
> "Yeah, **that's a push**, yeah. So like, **not MVP**."

**Aksana's acknowledgment:**
> "Data usage is going to be an MVP." (moved on to next item)

**Key Points:**

1. **Push operation (PUT)** - Not a GET, requires data modification
2. **Post-MVP** - Deferred to Phase 2
3. **New bucket** - Aksana noted this starts "another bucket" of features
4. **Very brief discussion** - Quick confirmation, no debate needed

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

**Meeting Discussion:**
This feature had substantial discussion clarifying the difference between daily (feasible) and hourly (not feasible) data usage breakdowns.

**Aksana's question (from requirements list):**
> "All right, and then **data usage broken by hour**."

**Devon's clarification (daily, not hourly):**
> "So, we are, we have in our, in our backlog, which could be hopefully done, worked on soon, **daily data usage**. Yeah, I just, yeah, I just prioritized that one, but **it's daily though, and now we need hourly**. Well, I don't know that we're going to be able to provide that level of, you know, just, I just don't know that we're going to be able to get to, I think, I think, honestly, I think **daily is probably going to be**, I mean, Adam, unless you think I'm wrong, I just don't know of a really, I guess, you know, **super accurate way of being able to present hourly**."

**Adam's comprehensive explanation on why hourly is NOT feasible:**
> "Yeah, so to be totally transparent with you, as I kind of stated, we **can't really get accurate hourly updates**. According to Verizon, they have what is referred to as a **buildup protocol**, which basically states that from, you know, any one point in time, **your data usage may not be updated for six to up**. I mean, I've heard some reps say even **24 hours**, but I mean, it's probably like **six hours**, because the way that the sessions work on the network, it's difficult for them to 100% accurately tally a data session until, like, **it terminates**, which, you know, is easy to say, but in reality, we all know, like, **some data sessions last for, I don't know, weeks**. So it's like, that doesn't really work. I can't wait weeks to get to know how much my data is. So they have, they have got made ways that they perform **build cuts on live data sessions** to get accurate updates in between the actual data session start and end. But they're, you know, discretionarily inaccurate with like a, like, you know, this, this time window that they refer to as a bill cut, which I've heard, again, I've heard it's six, I've heard it's 24."

**Devon's experience-based endorsement of daily (10-11 years):**
> "And just, and just for the record in, in, in the, you know, **in the 10, 11 years that Adam and I have been doing this**, we've, we've, like, **looking at daily usage has always been a super, you know, a really, really awesome, you know, tool to use when looking at spikes**. Even, even with the, a little bit of extra reporting that we can see sometimes, like on the AT&T side, you know, **getting more granular than daily just doesn't, doesn't really, never really even provided much more of a, of a value add**. So I think **daily**, and if we, if we build that in and then **create the API for daily** in a way that however we want to look at it, if it's, you're asking for X amount of days or you're just asking for how, you know, for looking back, we'll, we'll have to, we'll hash that out **once we actually get that built in our portal**."

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

**Meeting Discussion:**
This was discussed when Aksana asked about hourly data usage. Team clarified that HOURLY is not feasible, but DAILY is.

**Aksana's question:**
> "All right, and then **data usage broken by hour**."

**Devon's response:**
> "We have in our backlog...daily data usage...but **it's daily though**, and now we need hourly. Well, I don't know that we're going to be able to provide that level...I just don't know of a really...super accurate way of being able to present **hourly**."

**Adam's detailed explanation:**
> "Yeah, so to be totally transparent with you...we **can't really get accurate hourly updates**. According to Verizon, they have what is referred to as a **buildup protocol**...your data usage may not be updated for **six to up...24 hours**...some **data sessions last for...weeks**...they perform **build cuts on live data sessions**...I've heard it's six, I've heard it's 24."

**Devon's 10-11 years of experience:**
> "In the 10, 11 years that Adam and I have been doing this...looking at daily usage has always been a super...awesome...tool to use when looking at spikes...getting **more granular than daily just doesn't...never really even provided much more of a value add**."

**Reason (Technical - Carrier Limitations):**
- **Verizon "buildup protocol"** - Data usage may not be updated for 6-24 hours
- **Data sessions can last weeks** - Can't wait for session termination
- **Bill cuts on live sessions** - Verizon performs periodic "bill cuts" every 6-24 hours
- **Cannot guarantee accuracy at hourly level** - Time windows are discretionarily inaccurate
- **Daily is maximum practical granularity** - More granular doesn't provide value

**Reason (Practical - Experience-Based):**
- 10-11 years of experience managing thousands of IoT devices
- Daily usage has always been sufficient for spike analysis
- Even AT&T's more granular reporting doesn't justify sub-daily breakdowns
- Industry best practice: daily is the standard

**Available:** Daily data usage (HIGH PRIORITY in backlog) - See "Phase 3+ Features: Daily Data Usage Breakdown" for full details

**Cross-reference:** See Phase 3+ Features section for detailed discussion of Daily Data Usage Breakdown (feasible and in backlog)

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

**Meeting Discussion:**
This feature was discussed and determined to be unnecessary as an API endpoint because devices already auto-check for updates.

**Aksana mentioned when reviewing requirements:**
> "Check for modern updates, date of last update."

**Devon's initial response:**
> "That, that I don't know. Like we, as a managed services company, like we have full control, you know, obviously we make any change, make any and all changes that we feel are necessary that are custom for, for our customers. But I don't know, you know, how much."

**Adam's comprehensive explanation:**
> "You guys would need to there's really not like a **check for modem update everything that you have with us is yeah it should be up to date** so to give you a little bit more elaboration on what devin just said **every device checks for an update every two hours when it checks in** so if there's if there's an update needed it will notice on every single check-in every two hours and download one as needed we **don't load updates um typically um with the high degree of frequency** uh i mean unless there's something that we like know needs to be addressed like um so you know **not very common**"

**Key Points:**

1. **Automatic checking:** Devices auto-check for updates every 2 hours at each check-in
2. **Automatic download:** If update needed, device notices and downloads automatically
3. **Low frequency:** Updates loaded infrequently - only when needed
4. **Managed service:** WATM has full control over updates
5. **Always up-to-date:** Customer devices should always be current
6. **Standard configuration:** AllPoint uses standard config (not custom firmware)

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
> "Because there was the other message right below which is a **standard configuration** which i mean i would need some type of you know conversation to kind of put in place we've got tons of customers that run their own **custom configuration** you guys don't have anything custom so by default you are running a **standard** what we what we refer to as a standard configuration like i have customers who use um port forwarding for stuff like telnet directly into their kiosks i have customers who interact with printers inside of vlans directly through our modems on you know different ports and other types of stuff **custom dhcp custom netmass custom gateways all types of different customizations**"

**Devon's confirmation:**
> "But you guys would have to have asked for something that would be custom but so **you guys are running a standard config** if that makes sense right"

**Future consideration - RS-232 serial port:**

**Devon's example:**
> "And just you know maybe maybe an example of something like that like that could come in the future is the um you know the interest in possibly figuring out if there's anything you guys can do with that **serial port** right um"

**Adam's note:**
> "Um that **may require custom configuration** i don't know it's just you know 100 yeah if there is anything with that serial port that you guys ever decide that you're going to be leaning on in the future any specifics regarding the way that those protocols um i have like very limited to be honest um experience using using it or supporting it um but i would imagine it would all have to be pre-configured um to an extent um to an extent so that it works exactly how you expect it in the field so yeah i mean that would definitely be something that would be **non-standard** um in the terms of the fact you know other customers you know probably wouldn't ever use it"

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
