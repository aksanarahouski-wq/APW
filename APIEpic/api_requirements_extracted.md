# API Requirements - Extracted from Meeting (November 17, 2025)

## Meeting Overview
**Participants:** Adam Curcie, Devon D'Andrea, Aksana Rahouski (WATM team), Mason Joblinske, David Collison, Jeffrey Foreman (AllPoint/Customer)

**Purpose:** Discuss API requirements for AllPoint to integrate with WATM platform for IoT device management

**Customer Context:** AllPoint will be the first customer piloting the API library

## MVP Requirements (Phase 1 - GET Endpoints Only)

### 1. Device Information

#### 1.1 Get All Devices
- **Endpoint:** GET /devices
- **Requirements:**
  - Return list of all devices for the company
  - Support pagination (per_page parameter)
  - Return device array with basic information
- **Fields to Return:**
  - Serial number
  - Company name
  - Device status
  - Additional fields TBD based on schema review

#### 1.2 Get Device by Serial Number
- **Endpoint:** GET /devices/{serial_number}
- **Requirements:**
  - Return complete device details for a specific device
  - Include device data, location data, and service plan information
  - Return raw data (not pre-aggregated) so customer can apply their own logic

### 2. Signal Strength

#### 2.1 Last Reported Signal Strength
- **Priority:** HIGH - Critical for customer support
- **Requirements:**
  - Return last reported signal strength from device check-ins (every 2 hours)
  - Include timestamp of when signal strength was measured
  - Must indicate whether data is real-time or historical
- **Use Case:** Customer support needs to diagnose connectivity issues and determine if devices are in poor signal locations (Faraday cage situations)

#### 2.2 Real-Time Signal Strength (Phase 2)
- **Priority:** MEDIUM - Future enhancement
- **Requirements:**
  - API endpoint that triggers live ping to device to get current signal strength
  - Not a GET endpoint - requires push/trigger mechanism
  - Returns current signal strength with timestamp
- **Status:** Post-MVP, requires further technical discussion

### 3. Device Status and Health

#### 3.1 Current Health Status
- **Requirements:**
  - Return both primary and secondary status separately (not concatenated)
  - Examples: "active", "offline", "active/offline"
  - Allow customer to parse and use status data as needed
  - Include last check-in timestamp
- **Use Case:** Quick visual indicator if device is online before deeper investigation

#### 3.2 Device Status by Serial Number
- **Endpoint:** GET /devices/{serial_number}/status
- **Requirements:**
  - Granular endpoint to pull only status information
  - Return JSON object with status metadata
  - Include both status values and check-in times

### 4. Carrier Information

#### 4.1 Active Carrier
- **Requirements:**
  - Return carrier that last check-in came from
  - For dual-carrier devices:
    - Indicate if device is dual-carrier
    - List both configured carriers (e.g., Verizon and AT&T)
    - Show which carrier is currently active
  - For single-carrier devices:
    - Return carrier information from device configuration
- **Data Source:** Browse devices screen shows signal strength per carrier (e.g., "47 Verizon" vs "63 AT&T")
- **Edge Cases:** Handle devices with no check-in data (return configured carrier info)

### 5. Location Data

#### 5.1 Device Coordinates
- **Requirements:**
  - Return last known GPS coordinates (cellular triangulation)
  - **DO NOT** return manually entered address (customer has this in their own system)
  - Include timestamp of last location update
- **Important Notes:**
  - Not pinpoint GPS - accuracy up to half mile or even one mile radius
  - Intended for general location verification (e.g., "is device 100 miles away from expected location?")
- **Status:** Timestamp storage needs database review (TBD)

### 6. Data Usage

#### 6.1 Current Data Usage
- **Requirements:**
  - Return current data usage for billing period
  - Return service plan/tier information (e.g., Tier 1 = 3GB, Tier 3 = 10GB)
  - Customer will calculate percentage utilization on their end
- **Note:** All customer devices typically on same plan (unless temporary tier upgrade for high usage)

#### 6.2 Projected Data Usage Price
- **Requirements:**
  - Return projected usage price based on current usage
  - Include extrapolated calculation based on billing cycle position
  - Return raw calculation data so customer can understand the math

#### 6.3 Daily Data Usage (Future)
- **Priority:** HIGH - In current backlog
- **Requirements:**
  - Return daily data usage breakdown
  - **NOT** hourly (carrier limitations prevent accurate hourly reporting)
- **Limitations:**
  - Carrier "buildup protocol" means data not updated in real-time
  - Verizon performs "bill cuts" every 6-24 hours for live sessions
  - Daily granularity provides sufficient value for spike analysis
- **Status:** Feature being prioritized for portal, then will expose via API

### 7. Service Plan Information

#### 7.1 Device Service Plan/Tier
- **Requirements:**
  - Return current tier assignment (Tier 1, Tier 2, Tier 3, etc.)
  - Include data cap for the tier (e.g., Tier 3 = 10GB)
  - Return billing cycle window dates
- **Notes:**
  - Cycle window dates are hardcoded and don't change
  - Can be returned but may not provide significant value to customer

### 8. Network Information

#### 8.1 Private IP Address
- **Requirements:**
  - Return private IP of connected device
  - Already available in current build

#### 8.2 Public IP Address
- **Status:** **NOT POSSIBLE**
- **Reason:** Verizon will not pre-inform customers of public IP space utilization
  - Devices show behind NAT public IP (specific to cell tower or data center)
  - IPs subject to change without notice
  - Verizon's recommendation: whitelist ALL Verizon IPs (impractical)
- **Alternative:** Could route traffic through specific paths if customer has central portal needing data (requires separate conversation about use case)

### 9. Device Type/Model Information

#### 9.1 Modem Model
- **Requirements:**
  - Return modem model information
  - Include any relevant hardware specifications

## Post-MVP Requirements (Phase 2+)

### 10. Configuration Management (PUSH/POST Operations)

#### 10.1 Assign Modem to Machine
- **Type:** POST/PUT
- **Requirements:**
  - Push notification when new modem is connected to machine
  - Use MAC address as identifier (devices can pull MAC address from kiosks automatically)
  - Sync device assignment between WATM portal and customer system
- **Use Case:** Event-based - when machine boots up with new modem, sync connection

#### 10.2 Carrier Switch
- **Type:** POST
- **Requirements:** Allow customer to trigger carrier switch for dual-carrier devices
- **Status:** Post-MVP

#### 10.3 Firewall Settings
- **Type:** POST
- **Requirements:**
  - Push firewall rules to restrict device traffic
  - Support whitelist of approved URLs/IPs
  - Configuration updates applied within 2 hours (next device check-in)
- **Use Case:** Restrict devices to only communicate with customer's machines
- **Example:** ATM customers have 20-30 URL/IP whitelist for all US transaction processing

#### 10.4 Adjust Service Plan Tier
- **Type:** POST
- **Requirements:** Allow customer to change device tier programmatically
- **Status:** Post-MVP

#### 10.5 Remote Configuration Updates
- **Requirements:**
  - Push custom configuration changes to devices
  - Support custom DHCP, netmask, gateways
  - Support port forwarding configurations
  - Support VLAN configurations
  - Support RS-232 serial port configuration (future need for customer)
- **Notes:**
  - Customer currently on "standard configuration"
  - Custom configs would be needed for RS-232 serial port integration with traditional vending machines
  - All config updates applied at next device check-in (every 2 hours)

### 11. Modem Updates and Configuration Status

#### 11.1 Check for Modem Updates
- **Status:** NOT NEEDED as API endpoint
- **Reason:**
  - Devices auto-check for updates every 2 hours
  - All devices should always be up-to-date
  - Updates pushed with low frequency (only when needed)
- **Note:** Customer is on standard configuration (no custom firmware)

#### 11.2 Standard vs Custom Configuration
- **Status:** INFORMATIONAL ONLY
- **Note:** Customer currently uses standard configuration
  - No custom port forwarding, DHCP, netmask, gateway settings
  - Future: May need custom config for RS-232 serial port integration

### 12. Alerts and Notifications

#### 12.1 Security/Compromise Alerts
- **Requirements:**
  - Alert if device attempts to reach hosts outside firewall whitelist
  - Alert on Ethernet tampering (plug/unplug events)
- **Current Status:**
  - No current firmware support for firewall violation alerts
  - Ethernet tampering alarm exists but portal alarm handling incomplete
  - Could request InHand to add firewall traffic drop alarm to firmware
- **Implementation Options:**
  - Email notifications (existing framework)
  - **Webhook solution:** Push notification to customer endpoint when event occurs (recommended for real-time needs)

### 13. Speed and Performance

#### 13.1 Connection Speed
- **Status:** **NOT FEASIBLE VIA API**
- **Reason:**
  - Device has speed test capability but reporting is unreliable
  - Difficult to extract and report back
- **Alternative:** Customer could implement speed test on their end (e.g., speedtest.net) from their Linux/Raspberry Pi devices
- **Status:** Shelved

#### 13.2 Track Data Through Endpoint
- **Status:** **SHELVED**
- **Note:** No details provided in meeting about feasibility

### 14. Visualization and Reporting

#### 14.1 Magnitude Map of Signal Strength (Heat Map)
- **Priority:** WISH LIST item
- **Requirements:**
  - API returns all devices with signal strength data
  - Customer builds heat map visualization showing color-coded signal strength across geographic areas
  - Macro-level view with aggregated average signal strength
- **Use Case:** Visual dashboard showing geographic signal quality (green/yellow/red zones like telecom network status maps)
- **Implementation:** Customer-side visualization using raw device + signal strength data from API

## API Infrastructure Requirements

### 15. Authentication and Security

#### 15.1 API Key Management
- **Implementation:**
  - AWS API Gateway for security and rate limiting
  - API key-based authentication
  - Super admins for each company can manage keys
  - Key management UI in portal under "API Access" section
- **Key Management Features:**
  - Create new keys
  - Edit existing keys
  - Revoke keys
  - View key details

#### 15.2 API Access Control
- **Implementation:**
  - API access enabled per company by WATM super admins
  - Once enabled, company super admins can manage their own keys

### 16. Documentation and Testing

#### 16.1 API Documentation Portal
- **Requirements:**
  - OpenAPI/Swagger documentation
  - Downloadable YAML file
  - Interactive sandbox environment for testing
  - Schema documentation for all endpoints
  - Example responses with field descriptions
  - HTTP status code documentation (200, 401, etc.)

#### 16.2 Sandbox Testing Environment
- **Features:**
  - Try API calls directly from documentation portal
  - Authorize with API key
  - See actual response payloads with test data
  - Parameter testing (pagination, filters, etc.)

## Usage Patterns and Integration

### 17. Data Polling Frequency

#### 17.1 Batch/Scheduled Pulls
- **Customer Plan:** Once or twice daily automated pulls of all device data
- **Use Case:** General dashboard updates and bulk synchronization

#### 17.2 On-Demand Pulls
- **Priority:** MORE IMPORTANT than scheduled pulls
- **Customer Plan:** Real-time lookups when customer support receives calls
- **Use Case:** Support rep gets call about poor connection → immediately pull device status, signal strength, etc.

#### 17.3 Event-Based Integration
- **Customer Plan:** Trigger API calls on specific events
- **Examples:**
  - Machine boots up with new modem → sync device assignment
  - New device provisioned → pull initial device data
- **Implementation:** Customer-side event handlers triggering API calls

### 18. Rate Limiting Considerations
- **Note:** AWS API Gateway will handle rate limiting
- **Customer Pattern:** Mix of low-frequency batch operations and sporadic on-demand lookups
- **No high-volume real-time streaming expected**

## Technical Considerations

### 19. Data Freshness

#### 19.1 Device Check-in Frequency
- **Standard:** Devices check in every 2 hours
- **Implication:** "Last reported" data may be up to 2 hours old
- **Critical:** Always return timestamp with data so customer knows data age

#### 19.2 Real-time vs Historical Data
- **Requirement:** API must clearly indicate if data is:
  - Last reported (potentially hours/weeks old if device offline)
  - Real-time (freshly pulled from device)
- **Example:** Signal strength from 4 weeks ago is still useful if timestamp is provided ("Have you moved device in last 4 weeks?")

### 20. Data Format

#### 20.1 Response Format
- **Format:** JSON
- **Philosophy:** Return raw data, not pre-aggregated
- **Rationale:** Customer can apply their own logic, concatenation, calculations
- **Examples:**
  - Return both status values separately (not concatenated)
  - Return usage + tier (not pre-calculated percentage)
  - Return individual carrier info (not aggregated "dual-carrier" flag)

#### 20.2 Pagination
- **Support:** Optional pagination for large datasets
- **Implementation:** per_page parameter
- **Default:** Return all records if not specified

## Open Questions / TBD Items

### 21. Items Requiring Further Investigation

1. **Date of Last Location Update**
   - Need to verify if timestamp is stored separately in database
   - May need to add this field if not currently tracked
   - Assigned: Richard and Stone to investigate

2. **Signal Strength Data Structure**
   - Confirm exact fields to return
   - Determine if primary/secondary signal strength both needed
   - Review current portal implementation for reference

3. **Cycle Window Dates**
   - Confirm if these are useful to return (they're hardcoded and don't change)
   - Decide if worth including in device payload

4. **Missing Fields Review**
   - Customer to review current API documentation/schemas
   - Provide feedback on missing fields
   - WATM team to send current endpoint documentation

## Implementation Timeline

### Phase 1: MVP (Current - In Testing)
- GET endpoints only
- Core device information
- Signal strength (last reported)
- Status and health
- Basic usage and plan data
- Location coordinates
- Internal testing nearly complete
- **Next Steps:**
  1. WATM team to finish internal testing and minor cleanups
  2. Deploy to production with access enabled only for AllPoint
  3. AllPoint tests in sandbox environment
  4. AllPoint provides feedback on missing data/fields
  5. Iterate based on feedback

### Phase 2: Enhanced GET Endpoints
- Real-time signal strength (requires push mechanism)
- Daily data usage (after portal feature is built)
- Enhanced location tracking with timestamps
- Additional device metrics based on Phase 1 feedback

### Phase 3: POST/PUT Endpoints
- Device assignment/registration
- Configuration management
- Firewall rules
- Tier adjustments
- Carrier switching

### Phase 4: Advanced Features
- Webhook/push notifications for events
- Security alerts
- Custom device alarms
- Advanced reporting endpoints

## Hardware Considerations (Out of Scope for API)

### Related Hardware Discussion
- Discussion about new lower-cost device option
- 6-10 week lead time for specialized devices
- Testing being conducted by John and Larry Stevens
- Follow-up needed on testing timeline
- Not directly API-related but affects device types in system

## Customer Context & Use Cases

### AllPoint's Business Needs
1. **Dashboard Integration**
   - Display device status in their own system
   - Show signal strength alongside machine information
   - Track device connectivity for vending/kiosk machines

2. **Customer Support**
   - Quick device diagnostics during support calls
   - Verify device online status
   - Check signal strength for troubleshooting
   - Historical data for pattern analysis

3. **Device Deployment**
   - Sync new device assignments to machines
   - Track device locations vs expected locations
   - Manage growing device fleet

4. **Data Management**
   - Monitor usage to stay within plan tiers
   - Track costs per device
   - Identify devices with unusually high usage

5. **Security and Control**
   - Restrict device traffic to authorized endpoints
   - Monitor for security issues
   - Control device configurations

### Customer Technical Environment
- Vending machines and kiosks
- Many using Raspberry Pi (Linux)
- Dual-carrier devices (Verizon/AT&T primary)
- Machines often placed in challenging signal environments
- Growing fleet requiring scalable management

## Meeting Action Items

1. **WATM Team (Aksana):**
   - Complete internal testing
   - Review requirements list against current implementation
   - Add missing fields to payloads as identified
   - Send API documentation to AllPoint
   - Enable API access for AllPoint in production
   - Investigate location timestamp storage

2. **WATM Team (Adam/Devon):**
   - Send API documentation to customer
   - Coordinate with Richard and Stone on location timestamp question
   - Consider InHand firmware enhancement for firewall alerts
   - Provide timeline updates as development progresses

3. **AllPoint Team (David/Jeff/Mason):**
   - Test API in sandbox environment once access granted
   - Review API documentation and schemas
   - Provide feedback on missing data/fields
   - Submit additional requirements as identified
   - Follow up with John and Larry Stevens on hardware testing timeline
   - Provide updates on hardware testing to WATM team

4. **Both Teams:**
   - Align on final Phase 1 scope
   - Schedule follow-up after sandbox testing
   - Plan iterative enhancement cycles

## Notes and Constraints

### Technical Constraints
1. **Carrier Limitations:**
   - Hourly data usage not possible (only daily)
   - Public IP addresses cannot be provided reliably
   - Real-time data depends on device being online

2. **Device Behavior:**
   - Check-ins every 2 hours
   - Config updates applied at next check-in
   - GPS accuracy: 0.5-1 mile radius (cellular triangulation)

3. **Firmware Limitations:**
   - Speed test capability exists but unreliable
   - Some alarm types need firmware enhancement
   - Standard vs custom configurations available

### Business Constraints
1. **Lead Times:**
   - Specialized hardware: 6-10 weeks
   - Standard hardware: same-day shipping (8-10K inventory)

2. **Carrier Policies:**
   - Verizon IP policies restrict public IP visibility
   - Carrier data reporting delays (buildup protocol)

3. **Multi-tenant Security:**
   - API access controlled at company level
   - Each company isolated to their own data
   - API key management per company

## Success Criteria

### Phase 1 Success Metrics
1. AllPoint can successfully authenticate and access API
2. AllPoint can retrieve all device data needed for dashboard
3. API response times meet performance requirements
4. Data accuracy matches portal data
5. AllPoint can perform on-demand support lookups
6. No security or data leakage issues
7. API documentation is clear and complete

### Long-term Success Metrics
1. API scales to support multiple customers
2. Reduced support burden through self-service data access
3. Faster customer onboarding with API integration
4. Customer satisfaction with data accessibility
5. Reliable uptime and performance
6. Iterative improvements based on customer feedback
