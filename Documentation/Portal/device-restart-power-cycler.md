# Restart Power Cycler

## What It Does

Remotely power-cycles a physical machine (e.g., an ATM) connected to a wireless router's power relay. The user clicks "Restart Power Cycler" on the device detail page, and the system sends a signal through the router to cut and restore power to the connected equipment — effectively rebooting it without anyone on-site.

## Why It Exists

When field equipment like an ATM freezes or malfunctions, the most common fix is a simple power cycle (unplug, wait, plug back in). Without this feature, APW would need to dispatch a technician to each site just to flip a switch — costly ($100-300+ per visit), slow (hours of travel/coordination), and unscalable across hundreds of locations.

This button turns that into a one-click, instant fix from the portal.

## Who Can Use It

The button appears on the device detail page for any device whose model has the `restart_atm` flag enabled. It is visible to users who have access to the device detail view.

## How It Works

### Step-by-Step Flow

1. **User clicks "Restart Power Cycler"** on the device detail page
2. **System pings the device** at its IP address (7-second timeout) to check if the router is online
3. **If the device is offline:** Shows an error — *"The wireless device is not online, so the signal to power cycle your machine could not be sent."*
4. **If the device responds:**
   - Updates the device's `last_ping_time`
   - Verifies the device has credentials (username, password, IP)
   - Determines the router manufacturer (InHand or Systech)
   - Sends the power cycle command via the appropriate API
   - Shows a success message — *"The wireless device is online, therefore the signal to power cycle your machine has been sent to your wireless device. If you have a Power Cycler or Power Relay set up properly at this location, your machine should power cycle at this time."*

### What Happens at the Hardware Level

**InHand routers:**
1. Sets digital I/O output 1 to HIGH (cuts power)
2. Waits 10 seconds
3. Sets digital I/O output 1 to LOW (restores power)
4. Also sends a command to an Ethernet I/O device on port 12345 to power-cycle port 61 with a 10-second delay

**Systech routers:**
1. Sends a single command to port 12345: turn off power on port 61, automatically restore after 10 seconds

### Prerequisites

For this feature to work, the site must have:
- A power cycler or power relay physically connected to the router
- The router must be online and reachable via its IP address
- The device record must have valid credentials (username, password, IP)
- The device model must have `restart_atm` enabled

## Key Code Locations

| Component | File |
|-----------|------|
| Button (UI) | `watm/plugins/Devices/templates/Admin/Devices/view.php` (line ~957) |
| Controller logic | `watm/plugins/Devices/src/Controller/Admin/DevicesController.php` (line ~836) |
| InHand API | `watm/plugins/Devices/src/Util/RouterApi.php` — `restartPowerCycler()` |
| Systech API | `watm/plugins/Devices/src/Util/SystechApi.php` — `restartPowerCycler()` |
