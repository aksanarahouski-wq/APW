# WATM Completed Work Report
## All Completed Tickets (October 2024 - Present)

**Total Tickets Completed:** 146

---

# 2024 (30 tickets)

## Improvements & Features (24)

### Epic: WATM-1027 -- P2 - Device Management

#### WATM-1655 -- Add TMO SIM fields to the Manage Device View/Edit pages
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** Normal
**What was accomplished:** Added TMO SIM Number fields to the Manage Device View and Edit pages, enabling T-Mobile SIM tracking across device management workflows including dashboard counts, search, and exports.
**Business objective delivered:** Enabled the business to support T-Mobile carrier devices alongside existing Verizon and AT&T offerings, expanding the addressable market for IoT connectivity services.

#### WATM-1654 -- Add new TMO SIM option to device models
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** Normal
**What was accomplished:** Added a new "TMO SIM" device model option to the Add Device and Edit Device pages, allowing users to designate devices with T-Mobile SIM cards.
**Business objective delivered:** Extended the platform to support T-Mobile as a carrier option, paving the way for multi-carrier device management and new revenue streams.

#### WATM-1653 -- Add TMO SIM field to device import and device exports
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** Normal
**What was accomplished:** Added the TMO SIM Number column to both the device import template and device export, enabling bulk management of T-Mobile SIM data.
**Business objective delivered:** Streamlined bulk device provisioning for T-Mobile SIMs, reducing manual data entry and improving operational efficiency for large-scale deployments.

#### WATM-1652 -- Add SINR to the device Export Checkins
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** Normal
**What was accomplished:** Added a SINR (Signal-to-Interference-plus-Noise Ratio) column to the Export Checkins report on the Manage Devices page.
**Business objective delivered:** Provided customers with deeper signal quality diagnostics, enabling better troubleshooting of intermittent connectivity issues in the field.

#### WATM-1618 -- Revise how null signal strength values are displayed in the signal strength chart
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** Normal
**What was accomplished:** Implemented improved handling of null signal strength values in the signal strength chart so that missing data no longer causes the chart line to spike off the visible area.
**Business objective delivered:** Improved the accuracy and readability of signal strength charts, giving customers reliable visual data for diagnosing connectivity issues.

#### WATM-1608 -- Changes to Manage Billing Cycles page
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** High
**What was accomplished:** Removed the expand/collapse feature from the Manage Billing Cycles page and added a horizontal scroll bar for easier data navigation.
**Business objective delivered:** Improved the usability of the billing cycles interface, making it easier for billing staff to review all columns without losing context.

#### WATM-1198 -- Fix issue with new Excel library
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** Normal
**What was accomplished:** Closed as no longer needed after the Excel library data issues with Last Check-in timestamp and Warranty Start date resolved themselves.
**Business objective delivered:** Issue resolved without code changes, confirming the stability of the new Excel export library.

#### WATM-1074 -- Changes to Browse Devices section
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** High
**What was accomplished:** Removed the expand/collapse feature from the Browse Devices page and added a horizontal scroll bar, allowing users to see all columns including the critical "Last Check-in" column.
**Business objective delivered:** Eliminated a major usability barrier where customers on various screen sizes could not access important device columns, particularly the Last Check-in sort feature critical to their workflows.

### Epic: WATM-1031 -- P2 - Invoicing

#### WATM-1630 -- Recover all Instyle ATM invoices from 2023
**Epic:** WATM-1031 -- P2 - Invoicing | **Priority:** Normal
**What was accomplished:** Closed at client request as recovery of 2023 Instyle ATM invoices was no longer needed.
**Business objective delivered:** Client-initiated closure confirmed no further action required for historical invoice recovery.

### Epic: WATM-1033 -- P2 - Dashboard

#### WATM-1084 -- Add an export button on the "Alerts in the last 7 days" section of the dashboard
**Epic:** WATM-1033 -- P2 - Dashboard | **Priority:** Normal
**What was accomplished:** Added an export button to the "Alerts in the last 7 days" dashboard section, allowing download of alert data including Company, Alert Name, Date, Serial Number, and Location.
**Business objective delivered:** Enabled administrators and company managers to export and analyze recent alert data for reporting and operational review purposes.

#### WATM-1080 -- Add an export button on the "Notifications in the last 7 days" section of the dashboard
**Epic:** WATM-1033 -- P2 - Dashboard | **Priority:** Normal
**What was accomplished:** Added an export button to the "Notifications in the last 7 days" dashboard section with proper role-based access for WATM Super Admins and Company Super Admins.
**Business objective delivered:** Gave users the ability to export notification data for external analysis, compliance tracking, and operational reporting.

### Epic: WATM-1035 -- P2 - Companies

#### WATM-1631 -- Internal error message when trying to access the Browse Employees or Add Employee buttons as a WATM Super Admin
**Epic:** WATM-1035 -- P2 - Companies | **Priority:** Normal
**What was accomplished:** Fixed an internal error that occurred when WATM Super Admins accessed Browse Employees or Add Employee functions, and refined the tax document access flag behavior for different user roles.
**Business objective delivered:** Restored critical employee management functionality and ensured proper access control for sensitive tax document features across user roles.

#### WATM-1629 -- Update email templates for when a Credit Card is suspended
**Epic:** WATM-1035 -- P2 - Companies | **Priority:** Normal
**What was accomplished:** Updated credit card suspension email templates for all account types (Distributors, Customers, Subcustomers) to include payment method nickname and subcustomer name for better identification.
**Business objective delivered:** Improved the clarity and specificity of credit card suspension notifications, enabling faster resolution of payment issues by identifying the exact account and payment method affected.

### Epic: WATM-1064 -- P2 - Logging

#### WATM-1620 -- Create New "Browse Jobs" section under Admin section
**Epic:** WATM-1064 -- P2 - Logging | **Priority:** Normal
**What was accomplished:** Created a new "Browse Jobs" section under the Admin menu that displays job titles, start/end times, and status with filtering capabilities for WATM administrators.
**Business objective delivered:** Provided administrators with visibility into background job execution history, enabling monitoring and troubleshooting of automated system processes.

### Epic: WATM-1123 -- P2 - Technical Debt

#### WATM-1605 -- Odd Response from Check-In Server
**Epic:** WATM-1123 -- P2 - Technical Debt | **Priority:** Normal
**What was accomplished:** Investigated an unusual check-in server response and determined no adverse effects, closing the ticket without code changes needed.
**Business objective delivered:** Confirmed system stability after investigating anomalous server behavior reported by the client.

### Epic: WATM-1374 -- P2 - Account/Permissions changes

#### WATM-1645 -- Enable “allow payment method creation” flag for all of Miele’s sub customers except for L&M
**Epic:** WATM-1374 -- P2 - Account/Permissions changes | **Priority:** Normal
**What was accomplished:** Enabled the "allow payment method creation" flag for all of Miele Manufacturing sub-customers except L&M, timed to go live on 11/18.
**Business objective delivered:** Empowered Miele sub-customers to add their own ACH payment information, streamlining the payment onboarding process for a key distributor.

#### WATM-1644 -- Reassign all of Miele's sub customers devices to the sub customers primary payment method
**Epic:** WATM-1374 -- P2 - Account/Permissions changes | **Priority:** Normal
**What was accomplished:** Executed a one-time script to reassign all of Miele sub-customer devices to each sub-customer primary payment method after bank account setup was completed.
**Business objective delivered:** Completed a critical data migration for Miele Manufacturing, ensuring all devices are billed to the correct sub-customer payment methods.

### Epic: WATM-1408 -- Commissions P2 - Invoicing

#### WATM-1424 -- Automatically set Commissions Payment Method when a WATM admin is upgrading an account to a distributor company type
**Epic:** WATM-1408 -- Commissions P2 - Invoicing | **Priority:** Normal
**What was accomplished:** Implemented automatic setting of the Commissions Payouts payment method when a WATM Super Admin upgrades an account to distributor status.
**Business objective delivered:** Eliminated a manual step in the distributor onboarding process, reducing administrative overhead and potential for payment configuration errors.

### Epic: WATM-1486 -- Commissions P2 - Commissions module (company)

#### WATM-1487 -- Add summary page to earnings report
**Epic:** WATM-1486 -- Commissions P2 - Commissions module (company) | **Priority:** Normal
**What was accomplished:** Added a summary page to the company earnings report showing an aggregate total of commissions earned per subcustomer by the distributor.
**Business objective delivered:** Provided distributors with a clear, consolidated view of their earnings across all subcustomers, simplifying commission tracking and financial planning.

### Epic: WATM-1489 -- Commissions P2 - Companies

#### WATM-1490 -- Add flag to Company Employees for tax documents
**Epic:** WATM-1489 -- Commissions P2 - Companies | **Priority:** Normal
**What was accomplished:** Added a flag to the Company Employees section that grants individual employees access to the Document Management module and tax documents submodule.
**Business objective delivered:** Enabled fine-grained access control for tax documents, allowing primary account owners to delegate document access to specific team members.

### Epic: WATM-1500 -- Documents Management

#### WATM-1554 -- Revisions to the 1099 consent form
**Epic:** WATM-1500 -- Documents Management | **Priority:** Normal
**What was accomplished:** Implemented revisions to the 1099 consent form so distributors who do not consent to electronic delivery are excluded from electronic distribution, with a list maintained in the portal for WATM Admins.
**Business objective delivered:** Ensured legal compliance for 1099 tax document distribution by properly handling consent opt-outs and providing administrators visibility into non-consenting distributors.

### Epic: WATM-1691 -- Service Plan Management

#### WATM-1646 -- Revise how service plan customizations are handled(WATM Super Admin)
**Epic:** WATM-1691 -- Service Plan Management | **Priority:** Normal
**What was accomplished:** Built a new "Manage Company Service Plan" interface for WATM Super Admins to add and modify service plan customizations across all companies and distributor subcustomers, with an "Apply to all Subcustomers" option.
**Business objective delivered:** Centralized service plan management for administrators, enabling efficient bulk pricing adjustments and reducing the time to configure distributor subcustomer pricing.

#### WATM-1641 -- Revise how service plan customizations are handled(Company Super Admin)
**Epic:** WATM-1691 -- Service Plan Management | **Priority:** Normal
**What was accomplished:** Added service plan customization capabilities for Company Super Admins (distributors) including a Company column in browse view, "Apply to all Subcustomers" toggle, and per-subcustomer selection.
**Business objective delivered:** Empowered distributors to independently manage service plan pricing for their subcustomers, reducing dependency on WATM administrators for routine pricing changes.

#### WATM-1570 -- Distributor items that needed update for Miele
**Epic:** WATM-1691 -- Service Plan Management | **Priority:** Normal
**What was accomplished:** Coordinated and tracked the Miele Manufacturing distributor setup across three related tickets covering payment method flags, service plan customizations, and device reassignment.
**Business objective delivered:** Successfully onboarded a major distributor (Miele Manufacturing) with properly configured sub-customer relationships, payment methods, and service plan pricing.

## Bug Fixes (6)

### Epic: WATM-1031 -- P2 - Invoicing

#### WATM-1648 -- Flat rate devices are not showing up on October's invoice
**Epic:** WATM-1031 -- P2 - Invoicing | **Priority:** Normal
**What was accomplished:** Fixed a billing issue where flat rate devices were not appearing on monthly invoices, specifically resolving the case where devices with flat rate charges but no service plan were excluded.
**Business objective delivered:** Ensured accurate invoicing for all flat rate devices, preventing revenue leakage and customer billing discrepancies.

### Epic: WATM-1064 -- P2 - Logging

#### WATM-1640 -- Logs are taking a long time to load in Prod
**Epic:** WATM-1064 -- P2 - Logging | **Priority:** Normal
**What was accomplished:** Investigated and resolved slow log loading times in Production that appeared after a recent deployment.
**Business objective delivered:** Restored normal performance for the logging module, ensuring administrators can efficiently review system activity and troubleshoot issues.

### Epic: WATM-1408 -- Commissions P2 - Invoicing

#### WATM-1628 -- Earnings Report Contains all Companies and not just Distributors
**Epic:** WATM-1408 -- Commissions P2 - Invoicing | **Priority:** Normal
**What was accomplished:** Fixed the Earnings Report to correctly filter only distributor companies, resolving an issue where all company types were included in the report.
**Business objective delivered:** Ensured the Earnings Report provides accurate distributor-specific commission data, critical for financial reporting and payout calculations.

### Epic: WATM-1588 -- Commissions P2 - Payouts

#### WATM-1573 -- Add form to the credit card suspension notification email
**Epic:** WATM-1588 -- Commissions P2 - Payouts | **Priority:** Normal
**What was accomplished:** Attached the billing change form to the credit card suspension email notification and updated templates with subcustomer name and payment method nickname.
**Business objective delivered:** Streamlined the credit card suspension resolution process by providing the required form directly in the notification email, reducing turnaround time for payment corrections.

### Epic: WATM-1693 -- Invoicing

#### WATM-1662 -- If a device has a flat rate then they are being charged for the flat rate twice
**Epic:** WATM-1693 -- Invoicing | **Priority:** Normal
**What was accomplished:** Fixed a critical billing bug where devices with flat rate charges were being double-billed, and ensured flat-rate-only devices appear correctly on the Admin Billing report.
**Business objective delivered:** Eliminated invoice overcharges for flat rate customers and restored complete visibility in billing reports, protecting both revenue accuracy and customer trust.

### Epic: WATM-1699 -- Device Management

#### WATM-1700 -- Issues with device w48284
**Epic:** WATM-1699 -- Device Management | **Priority:** Normal
**What was accomplished:** Fixed an internal error on the device edit page that prevented enabling dual SIM or modifying delayed billing for certain devices related to SIM number validation.
**Business objective delivered:** Restored the ability to edit dual SIM and billing settings for affected devices, removing a blocker for device configuration management.

---

# 2025 (76 tickets)

## Improvements & Features (38)

### Epic: WATM-1027 -- P2 - Device Management

#### WATM-1638 -- Add Parent Company column to the device export
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** Normal
**What was accomplished:** Added a Parent Company column to the device export for WATM admin and super admin users, showing the parent company for customer and distributor accounts with sub-customers.
**Business objective delivered:** Improved organizational visibility for administrators managing multi-tier company hierarchies, making it easier to audit device assignments across the distributor network.

#### WATM-1331 -- Signal Strength Revisions
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** Normal
**What was accomplished:** Added SINR signal strength values to the device export, signal strength chart, and info icons with explanatory text for both RSSI and SINR metrics.
**Business objective delivered:** Gave customers and support teams comprehensive signal quality visibility with both RSSI and SINR metrics, enabling proactive identification of connectivity problems.

### Epic: WATM-1586 -- VPN

#### WATM-1593 -- POC for VPN whitelist
**Epic:** WATM-1586 -- VPN | **Priority:** Normal
**What was accomplished:** Completed a proof of concept for VPN access list integration with PaloAlto firewalls, generating IP address lists from active devices for a specific company.
**Business objective delivered:** Validated the technical feasibility of automated VPN access control, laying the groundwork for secure per-customer device network access management.

### Epic: WATM-1588 -- Commissions P2 - Payouts

#### WATM-1330 -- Email notification for 1099s
**Epic:** WATM-1588 -- Commissions P2 - Payouts | **Priority:** Normal
**What was accomplished:** Built email notification functionality for 1099 tax documents, allowing WATM Admins to send links to all eligible distributors so they can download their 1099NEC forms from the portal.
**Business objective delivered:** Automated the 1099 distribution process, replacing manual delivery with a scalable digital workflow that ensures timely tax document delivery to distributors.

### Epic: WATM-1679 -- Cellular backup

#### WATM-1633 -- Cellular Back Up Enhancement
**Epic:** WATM-1679 -- Cellular backup | **Priority:** Normal
**What was accomplished:** Enhanced the cellular backup feature to support WiFi as the primary internet link (Station mode), adding SSID and password configuration fields to the portal with persistence across configuration pushes.
**Business objective delivered:** Expanded connectivity options for customers by enabling WiFi-primary deployments with cellular backup, supporting new use cases where location-provided WiFi is preferred over wired connections.

### Epic: WATM-1681 -- Device connected equipment and firewall rules.

#### WATM-1310 -- Connected Equipment Revisions
**Epic:** WATM-1681 -- Device connected equipment and firewall rules. | **Priority:** Normal
**What was accomplished:** Redesigned the Connected Equipment feature to show real-time device status, eliminate duplicate entries from mixed-case MAC addresses, and handle multiple IP addresses per MAC address correctly.
**Business objective delivered:** Delivered accurate connected equipment visibility to customers, enabling reliable monitoring of downstream devices and proper management of locked equipment.

#### WATM-1290 -- Firewall Rules Column/Functionality
**Epic:** WATM-1681 -- Device connected equipment and firewall rules. | **Priority:** Normal
**What was accomplished:** Built out firewall rules management for connected equipment including Allow/Deny policies, a "Deny All Except Locked" option, lock/unlock functionality, and device-level firewall enable/disable capability.
**Business objective delivered:** Gave customers granular network security control over their connected equipment, enabling them to restrict unauthorized device access while maintaining approved connections.

### Epic: WATM-1685 -- Device offline checker quality, reliability, and accuracy

#### WATM-1706 -- Enhance Offline Checker
**Epic:** WATM-1685 -- Device offline checker quality, reliability, and accuracy | **Priority:** Normal
**What was accomplished:** Refactored the offline device checker from a single-threaded process to a multi-process parallel system capable of processing all devices within a 20-minute cycle.
**Business objective delivered:** Dramatically improved the reliability and speed of offline device detection, ensuring customers receive timely notifications when devices go offline.

#### WATM-1656 -- Add Google Analytics 4 for apcommand.com and allpointcommand.com
**Epic:** WATM-1685 -- Device offline checker quality, reliability, and accuracy | **Priority:** Normal
**What was accomplished:** Added Google Analytics 4 tracking tags to both apcommand.com and allpointcommand.com portals.
**Business objective delivered:** Enabled web analytics tracking for both portal domains, providing the business with user behavior insights for data-driven product decisions.

#### WATM-1649 -- Change checkins nodejs so that it buffers up many checkins to put many messages in redis rather than putting them in one at a time
**Epic:** WATM-1685 -- Device offline checker quality, reliability, and accuracy | **Priority:** High
**What was accomplished:** Refactored the Node.js check-in server to batch multiple device check-ins together before writing to SQS queues, rather than processing them one at a time.
**Business objective delivered:** Significantly improved check-in processing throughput and reduced database bottleneck risk by batching device telemetry data.

#### WATM-1647 -- Bulk Insert Checkins
**Epic:** WATM-1685 -- Device offline checker quality, reliability, and accuracy | **Priority:** Normal
**What was accomplished:** Implemented bulk insert processing for device check-ins, enabling multiple check-ins to be inserted into the database simultaneously rather than one at a time.
**Business objective delivered:** Improved system performance and scalability by reducing database I/O overhead during high-volume check-in processing periods.

### Epic: WATM-1687 -- Dual SIM Changes

#### WATM-1535 -- Revisions to pricing for Other info on Manage Device page
**Epic:** WATM-1687 -- Dual SIM Changes | **Priority:** Normal
**What was accomplished:** Updated the Other Info section on the Manage Device page to display the combined projected price including both service plan and dual SIM charges as a single total.
**Business objective delivered:** Provided clearer pricing visibility on device pages by showing the total projected cost including all applicable charges.

### Epic: WATM-1693 -- Invoicing

#### WATM-1664 -- WATM-1664 ⁃ Email tracking?
**Epic:** WATM-1693 -- Invoicing | **Priority:** High
**What was accomplished:** Fixed a bug where only the last email address in the Invoice CC list was receiving invoice emails, ensuring all CC recipients now receive invoices correctly.
**Business objective delivered:** Restored proper invoice email distribution to all designated recipients, ensuring companies receive complete billing communications.

#### WATM-1660 -- Need to be able to update payment method and have it apply to current invoice
**Epic:** WATM-1693 -- Invoicing | **Priority:** Normal
**What was accomplished:** Fixed the invoice generation process so that payment method changes made after the billing cycle start date but before invoice generation are properly reflected on invoices.
**Business objective delivered:** Enabled billing staff to make payment method corrections up until invoice generation without data inconsistencies, improving billing flexibility and accuracy.

### Epic: WATM-1743 -- APW Support 2025

#### WATM-1949 -- Add Manufacturer Warranty Dates to Device Import
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Added Manufacturer Warranty Start Date and End Date columns to the device import template and processing logic, alongside existing customer warranty fields.
**Business objective delivered:** Enabled bulk import of manufacturer warranty data, improving warranty tracking accuracy and reducing manual data entry for large device deployments.

#### WATM-1943 -- Update Devices upload edits
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Added Primary Status and Additional Status columns to the bulk device update upload feature, allowing administrators to change device statuses in bulk via CSV.
**Business objective delivered:** Improved operational efficiency by enabling bulk device status changes, reducing the time required to manage large device fleet status transitions.

#### WATM-1928 -- Update Device Update Import
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Added Manufacturer and Model fields to the device bulk update page, including updated sample data and template files.
**Business objective delivered:** Enabled bulk updates of device manufacturer and model information, streamlining fleet management for large-scale device transitions.

#### WATM-1890 -- Remove Flat Rate Devices from Distributor Earnings Reports
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Excluded flat rate devices (those with no service plan) from distributor earnings reports to eliminate incorrect negative earnings calculations.
**Business objective delivered:** Improved earnings report accuracy by removing devices that do not contribute to distributor commissions, eliminating confusing negative values.

#### WATM-1888 -- Devices Billed for Cycles When Inactive Throughout Period
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Updated billing logic so devices are only invoiced for billing cycles where they were active at any point during that cycle, preventing charges for fully deactivated periods.
**Business objective delivered:** Eliminated incorrect charges for devices that were deactivated throughout an entire billing cycle, resolving invoice discrepancies and customer complaints.

#### WATM-1850 -- Looking into wrong config issue
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Investigated and resolved a device configuration mismatch issue where devices were incorrectly assigned cellular backup configurations during the assign process.
**Business objective delivered:** Fixed a configuration assignment bug affecting 42+ devices, ensuring devices receive the correct configuration files based on their actual setup.

#### WATM-1848 -- Add mobile disclaimer to device details page
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Added a mobile-specific disclaimer message to the device details page informing users that the page is optimized for desktop viewing.
**Business objective delivered:** Set appropriate user expectations for mobile viewing while maintaining full functionality, reducing confusion about layout on smaller screens.

#### WATM-1830 -- Customize notification recipients
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Added notification opt-out options at the employee level, allowing users to opt out of all notifications, global notifications, or company notifications independently.
**Business objective delivered:** Gave customers granular control over notification preferences, reducing unwanted communications while maintaining critical alerts for those who need them.

#### WATM-1802 -- Edit invoice email and invoice header
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Critical
**What was accomplished:** Updated the invoice email template and invoice header to reflect the company rebranding from "ATM Partner" to "Allpoint Wireless | Wireless ATM Store", including updated contact information and logos.
**Business objective delivered:** Ensured all customer-facing billing communications reflect the current company branding after the ATM Partner divestiture.

#### WATM-1795 -- Remove SMS Notification Option from UI
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Removed the SMS notification option from the UI and cleaned up existing SMS notification records, since SMS functionality had been disabled on the backend.
**Business objective delivered:** Eliminated customer confusion caused by a non-functional SMS option in the notification setup, preventing misconfigured alerts that would never be delivered.

#### WATM-1740 -- Bulk Actions "Internal Error"
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Critical
**What was accomplished:** Resolved a critical internal error that occurred when using Bulk Actions to change device status or service plan, caused by a null entity reference.
**Business objective delivered:** Restored bulk device management functionality, enabling administrators to efficiently update multiple devices simultaneously.

#### WATM-1729 -- Added functionality to assign device
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Enhanced the device assignment workflow to exclude deactivated companies from search results and display parent company names in parentheses for sub-companies.
**Business objective delivered:** Improved the device assignment experience by showing clearer company hierarchy information and preventing assignment to inactive companies.

#### WATM-1710 -- Assigning Devices to Sub-company with payment method
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Updated the device assignment flow so that when assigning devices to a sub-company with its own payment method, the sub-company primary payment method is shown as the default selection.
**Business objective delivered:** Reduced assignment errors and saved time by automatically defaulting to the correct payment method when assigning devices to sub-companies.

#### WATM-1666 -- Add feature to allow who should receive ACH auth form
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Added a recipient selection feature for ACH authorization forms when distributors create payment methods for subcustomers, with options to send to the distributor or subcustomer with confirmation notifications.
**Business objective delivered:** Streamlined the ACH authorization process for distributor-subcustomer relationships, enabling subcustomers to sign forms directly while keeping distributors informed.

### Epic: WATM-1787 -- APC Support 2024

#### WATM-1712 -- Add tools in APC to create and maintain Access Lists
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** High
**What was accomplished:** Built VPN Access Control List (ACL) management tools including a company-level VPN ACL flag, list naming, automatic IP address maintenance, and password-protected list endpoints for PaloAlto firewall integration.
**Business objective delivered:** Delivered automated VPN access management integrated with the firewall infrastructure, enabling per-customer device network isolation and security policy enforcement.

#### WATM-1665 -- Odd log item
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** High
**What was accomplished:** Investigated an unusual device modification log entry and closed the ticket after the client confirmed no action was needed.
**Business objective delivered:** Addressed client concern about unexpected log entries, confirming normal system behavior.

#### WATM-1657 -- Plan for upcoming larger items
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** Normal
**What was accomplished:** Created a prioritized planning document on Confluence for upcoming larger feature initiatives including alert handling, TMO APIs, API library, firewall rules, and cellular backup.
**Business objective delivered:** Established a strategic roadmap for major upcoming feature development, aligning development priorities with business objectives.

#### WATM-1650 -- Need pairing of serial number and device ID.
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** High
**What was accomplished:** Fulfilled a one-time data pull request providing serial number and device ID pairings for Bitcoin Depot devices to support their internal system URL mapping.
**Business objective delivered:** Supported a key customer integration need by providing device reference data, bridging a gap until the customer API integration is completed.

#### WATM-1639 -- Browse Logs page won't load
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** Normal
**What was accomplished:** Resolved the Browse Logs page loading failure.
**Business objective delivered:** Restored administrator access to system logs for operational monitoring and troubleshooting.

#### WATM-1632 -- Routing for APC to reach the following subnet:
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** Normal
**What was accomplished:** Configured network routing for APC to reach the 100.80.0.0/17 subnet through the Orases tunnel.
**Business objective delivered:** Extended network connectivity to a new device subnet, enabling the platform to communicate with additional IoT devices.

### Epic: WATM-1807 -- Device Power Management Scheduling Feature Delivery

#### WATM-1800 -- IoT Relay Feature with Timer - Scope/Estimate for a New Feature
**Epic:** WATM-1807 -- Device Power Management Scheduling Feature Delivery | **Priority:** High
**What was accomplished:** Built a device power management scheduling feature allowing customers to set automated on/off schedules for IoT relay devices by day of week, with override capabilities.
**Business objective delivered:** Delivered a new revenue-generating feature that enables customers to schedule automated power cycling of their ATM equipment, reducing energy costs and supporting operational workflows.

### Epic: WATM-1827 -- Billing Module Redesign: Generation of billing cycle invoices and other documents.

#### WATM-1840 -- Billing Module Redesign
**Epic:** WATM-1827 -- Billing Module Redesign: Generation of billing cycle invoices and other documents. | **Priority:** Normal
**What was accomplished:** Redesigned the billing module with a unified billing cycle management interface using a card-based layout, ensuring all billing documents are generated from a single data snapshot to prevent misalignment.
**Business objective delivered:** Transformed the billing workflow to eliminate data inconsistencies between invoices, NACHA files, and reports, significantly reducing billing errors and improving user productivity.

### Epic: WATM-1843 -- T-Mobile Sim Support in APW

#### WATM-1937 -- TMobile Device Usage Issues
**Epic:** WATM-1843 -- T-Mobile Sim Support in APW | **Priority:** Normal
**What was accomplished:** Fixed T-Mobile device usage API calls to use current date instead of billing cycle start date, and planned cleanup of accumulated incorrect usage data.
**Business objective delivered:** Restored accurate T-Mobile data usage reporting, ensuring correct billing for T-Mobile devices and preventing continued accumulation of false data.

### Epic: WATM-1859 -- APW API Access

#### WATM-1617 -- API Access
**Epic:** WATM-1859 -- APW API Access | **Priority:** Normal
**What was accomplished:** Researched and designed an external API library for WATM customers, with endpoint planning, authentication design, and customer requirements gathering.
**Business objective delivered:** Laid the foundation for a customer-facing API, enabling WATM clients to integrate device data into their own systems and reducing dependence on the portal for routine data access.

## Bug Fixes (38)

### Epic: WATM-1027 -- P2 - Device Management

#### WATM-1651 -- Pending status
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** Normal
**What was accomplished:** Resolved an issue where devices were stuck in a "pending" status due to invalid ICCIDs according to Verizon API, preventing the automated status update script from processing them.
**Business objective delivered:** Ensured accurate device status tracking so customers always see the true state of their devices, preventing confusion and unnecessary support inquiries.

#### WATM-1613 -- w45290 - failed activation
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** High
**What was accomplished:** Added enhanced logging for device activation failures to better diagnose situations where the portal status and Verizon status become out of sync.
**Business objective delivered:** Improved troubleshooting capability for device activation issues, reducing the time to diagnose and resolve carrier API failures.

#### WATM-1524 -- Devices with Flat Rate only are not being billed
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** Critical
**What was accomplished:** Fixed a billing bug where devices assigned with a flat rate charge but no service plan were not being included in customer invoices.
**Business objective delivered:** Ensured all billable devices are properly invoiced, preventing revenue loss from devices with flat rate billing configurations.

### Epic: WATM-1029 -- Theme Issues

#### WATM-1591 -- Browse Device Set Filter Bug
**Epic:** WATM-1029 -- Theme Issues | **Priority:** Normal
**What was accomplished:** Fixed a production-only bug where clicking "Set Default Filter" on the Browse Devices page incorrectly redirected users to the Dashboard instead of staying on the Browse Devices page.
**Business objective delivered:** Restored the ability for users to save their preferred device view settings without disruption, improving daily workflow efficiency.

### Epic: WATM-1374 -- P2 - Account/Permissions changes

#### WATM-1497 -- When Distributors/Companies Create Subcustomer Company accounts the user has the wrong role/permissions.
**Epic:** WATM-1374 -- P2 - Account/Permissions changes | **Priority:** Normal
**What was accomplished:** Investigated and closed a role/permissions issue where subcustomer company accounts created by distributors had incorrect user roles; issue could not be reproduced.
**Business objective delivered:** Confirmed that the subcustomer account creation process is functioning correctly, with no reproducible defects in role assignment.

### Epic: WATM-1408 -- Commissions P2 - Invoicing

#### WATM-1634 -- Critical bug - pricing wrong for Tier 1 devices
**Epic:** WATM-1408 -- Commissions P2 - Invoicing | **Priority:** High
**What was accomplished:** Investigated a pricing discrepancy for Tier 1 devices showing incorrect prices; the issue resolved itself and was traced to a recent service plan change.
**Business objective delivered:** Confirmed pricing accuracy after investigation, ensuring customers are billed at the correct service plan rates.

### Epic: WATM-1693 -- Invoicing

#### WATM-1702 -- Earnings report isn't displaying the correct amount
**Epic:** WATM-1693 -- Invoicing | **Priority:** Normal
**What was accomplished:** Resolved an earnings report calculation error where dual SIM upcharge prices were not being correctly loaded for sub-companies during invoice generation.
**Business objective delivered:** Ensured accurate commission calculations for distributors, preventing incorrect payout amounts due to missing dual SIM pricing data.

#### WATM-1701 -- Financing amount is showing up on invoice even though it isn't listed in the General Info section
**Epic:** WATM-1693 -- Invoicing | **Priority:** Normal
**What was accomplished:** Investigated a reported financing charge discrepancy on invoices and confirmed the system was functioning as designed, with the "0 months remaining" display working correctly.
**Business objective delivered:** Confirmed billing accuracy for financing charges, with documentation provided to the client on how the remaining months display logic works.

### Epic: WATM-1743 -- APW Support 2025

#### WATM-1950 -- Device Group Assignment Should be based on Active and not Presence.
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Fixed the device group assignment process to skip SIM assignment for deactivated (not just absent) SIMs, resolving failures when assigning Origin devices with inactive Verizon SIMs.
**Business objective delivered:** Eliminated device assignment failures for Origin devices, allowing smooth onboarding of new device types with mixed carrier SIM configurations.

#### WATM-1930 -- Failed Deactivation of SIM
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Low
**What was accomplished:** Closed as a false alarm; the client reported an error with SIM deactivation that did not require investigation.
**Business objective delivered:** No action required; client confirmed the ticket should be disregarded.

#### WATM-1898 -- Individual Company Earnings Reports are not generated in new Billing Cycle Workflow
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Fixed a regression where individual company earnings reports were no longer being generated as part of the new billing cycle workflow.
**Business objective delivered:** Restored individual company earnings report generation, ensuring distributors receive their per-company commission breakdowns.

#### WATM-1820 -- Research Billing Cycle Discrepancies
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Researched root causes of billing cycle discrepancies including double billing, pricing errors, and invoice-to-NACHA mismatches identified in meeting notes.
**Business objective delivered:** Identified and documented the root causes of billing discrepancies, providing a roadmap for systematic resolution of invoice accuracy issues.

#### WATM-1817 -- Cannot remove logo in whitelabeling section after adding one.
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** High
**What was accomplished:** Fixed an issue where company logos added through the white-labeling feature could not be removed, and resolved the specific case for AZ ATM Expert.
**Business objective delivered:** Restored full white-labeling management capability, allowing administrators to add and remove company logos as needed.

#### WATM-1801 -- Bad Data in Device Export
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** High
**What was accomplished:** Investigated incomplete device export data where certain date ranges were missing from Excel filter options, indicating potential data gaps.
**Business objective delivered:** Identified and addressed device export data completeness issues to ensure administrators have reliable data for fleet management.

#### WATM-1799 -- Can't import devices
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Critical
**What was accomplished:** Closed as a false alarm; the device import issue reported by the client was not reproducible.
**Business objective delivered:** No action required; client confirmed the import issue was temporary.

#### WATM-1790 -- Account for TMO SIM field in the Bulk Action > Service Plan update process
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Fixed the Bulk Action Service Plan update process to account for a hidden TMO SIM field that was preventing configuration values from updating correctly.
**Business objective delivered:** Ensured bulk service plan changes properly update device configurations, critical for large-scale fleet management operations.

#### WATM-1781 -- Customer cannot log in
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Critical
**What was accomplished:** Resolved a critical login issue for company users caused by the Signal Strength chart query timing out when cached data was invalid, by showing an empty chart instead of blocking login.
**Business objective delivered:** Restored customer access to the portal by implementing a graceful fallback for the dashboard signal strength chart, eliminating a login-blocking performance issue.

#### WATM-1780 -- Invoices Did Not Generate/Send 4/11/25
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Fixed invoice generation failures caused by database timeouts during PDF/Excel file creation by implementing a retry queue mechanism for failed file generations.
**Business objective delivered:** Improved invoice generation reliability by automatically retrying failed file creation, ensuring all companies receive their invoices.

#### WATM-1760 -- Duplicates in March/April Billing Cycle
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Documented duplicate device entries found in the billing report for further root cause investigation in a future effort.
**Business objective delivered:** Identified billing report duplicates for tracking and future resolution, maintaining awareness of data quality issues.

#### WATM-1759 -- Config Value on Browse device screen not updating
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Critical
**What was accomplished:** Fixed a critical bug where changing a device service plan did not update the configuration file displayed on the browse device screen, caused by a missing hidden TMO field in the service plan change form.
**Business objective delivered:** Ensured device configurations correctly update when service plans change, maintaining consistency between portal display and actual device settings.

#### WATM-1746 -- Device is not loading it's company  configs
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** High
**What was accomplished:** Investigated a device configuration loading issue on production where the hostname mismatch was identified as a test device anomaly, confirmed resolved in a client meeting.
**Business objective delivered:** Confirmed device configuration loading works correctly, with the reported issue traced to a test device hostname mismatch.

#### WATM-1745 -- Device connected equipment 'lock' functionality issue.
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** High
**What was accomplished:** Fixed the connected equipment lock functionality where locking multiple devices simultaneously would only save one, traced to a semicolon encoding issue in API calls.
**Business objective delivered:** Restored the ability to lock multiple pieces of connected equipment in a single operation, critical for managing device network access rules.

#### WATM-1725 -- W34394
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Critical
**What was accomplished:** Fixed a bug where the portal kept overriding a device status to "Data Suspension" after it had been set to "Deactivated", by preventing deactivated devices from being affected by data suspension logic.
**Business objective delivered:** Stopped erroneous status overrides for deactivated devices, ensuring device statuses remain stable once set by the customer.

#### WATM-1711 -- Cap SINR Percentage on Browse Devices Screen
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Capped the SINR percentage display on the Browse Devices screen to never exceed 100% or drop below 0%, while preserving the actual SINR value.
**Business objective delivered:** Corrected the signal strength percentage display to show meaningful, bounded values that customers can trust for quick device health assessment.

#### WATM-1709 -- Odd API behavior for device W58371
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Fixed a bug where deactivated devices were incorrectly affected by data suspension logic and billing cycle resets, preventing erroneous API calls to Verizon.
**Business objective delivered:** Prevented failed carrier API calls and incorrect status changes for deactivated devices, improving system reliability and carrier integration accuracy.

#### WATM-1708 -- Discrepancies between watm billing report and what's actually on the invoices
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Investigated billing report discrepancies and determined they were caused by timing differences between report and invoice generation, not a system bug.
**Business objective delivered:** Confirmed billing system accuracy through investigation, providing the client with clear guidance on report-to-invoice timing dependencies.

#### WATM-1707 -- Admin billing report doesn't contain sub-companies
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Fixed the admin billing report to include devices assigned to sub-companies, added an "Invoiced Company" column to clarify billing relationships.
**Business objective delivered:** Provided complete billing visibility by including sub-company device data in the admin report, essential for accurate financial oversight of the entire device fleet.

#### WATM-1705 -- Can't seem to add RMAs
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** High
**What was accomplished:** Fixed a critical bug preventing RMA creation that was causing error messages despite valid device data, and deployed the fix to Production on an expedited timeline.
**Business objective delivered:** Restored the ability to create RMAs, unblocking the device replacement workflow that was falling behind due to the defect.

#### WATM-1704 -- Data notifications not working?
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** High
**What was accomplished:** Identified that data notifications were not being sent for sub-company devices by design, and captured the client requirement to add configurable notification routing options for parent/sub-company scenarios.
**Business objective delivered:** Documented and scoped a new notification routing feature to allow distributors to control how device alerts are shared with their sub-companies.

### Epic: WATM-1787 -- APC Support 2024

#### WATM-1783 -- Primary User Not Assigned to Company
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** Normal
**What was accomplished:** Identified that 250+ companies without primary users were imported historically, implemented notifications for failed invoice emails, and updated logic to fall back to Invoice CC emails when no primary user exists.
**Business objective delivered:** Ensured invoice delivery reliability by implementing fallback email logic and administrative alerts, preventing invoice emails from silently failing.

#### WATM-1779 -- Send Invoice to All Companies, Regardless of Status
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** Normal
**What was accomplished:** Removed company status filtering from invoice generation so all companies with a balance due receive invoices regardless of their status, and added a company status filter to the invoice search.
**Business objective delivered:** Ensured complete invoice coverage for all accounts with outstanding balances, preventing missed collections due to company status filtering.

#### WATM-1738 -- Ensure Users that have the "User" Role Cannot see WATM Dashboard
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** Normal
**What was accomplished:** Implemented access control to prevent users with the "User" role from viewing the WATM Dashboard.
**Business objective delivered:** Strengthened role-based access controls to ensure only authorized personnel can access administrative dashboard data.

#### WATM-1694 -- DUAL CARRIER CHECKBOX ON BROWSE DEVICES SCREEN
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** Critical
**What was accomplished:** Fixed the dual carrier checkbox on the Browse Devices screen to be read-only, preventing users from accidentally toggling it when it should only be an informational display.
**Business objective delivered:** Prevented potential confusion and data integrity issues by making the dual carrier display non-interactive on the browse screen.

#### WATM-1658 -- Device checkin issues after recent prod push
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** Critical
**What was accomplished:** Resolved a critical issue where no device check-ins were being stored after a production deployment, and the Export Checkins feature was displaying an internal error.
**Business objective delivered:** Restored device telemetry data collection after a production incident, ensuring continuous monitoring of the IoT device fleet.

#### WATM-1627 -- Approved Commissions Payout is still showing $0.00
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** Normal
**What was accomplished:** Investigated the Available Commissions page showing $0.00 after payouts were approved, clarified the page purpose with the client, and identified a fix needed for line item display.
**Business objective delivered:** Resolved confusion around commission payout visibility and ensured distributors can see pending commissions before payout approval.

#### WATM-1626 -- Check In Failed / Ping Verification Inquiry
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** Normal
**What was accomplished:** Investigated a false offline notification triggered by a check-in failure, but logs were too old to diagnose; noted that batch check-in improvements should prevent recurrence.
**Business objective delivered:** Documented the incident for tracking while confirming that ongoing system improvements address the underlying reliability concern.

#### WATM-1623 -- Upstate ATM
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** Critical
**What was accomplished:** Investigated and resolved a critical invoice generation error where an incorrect company (Upstate ATM) received invoices for another company (Vogel Vending), traced to a one-time data import error.
**Business objective delivered:** Eliminated a billing error that caused invoices to be sent to the wrong company, confirming it was an isolated data issue from initial system launch.

#### WATM-1614 -- W75052 - why isn't the status data suspension?
**Epic:** WATM-1787 -- APC Support 2024 | **Priority:** High
**What was accomplished:** Investigated why device W75052 was not placed in Data Suspension despite exceeding its ATM plan data threshold, traced to a data auto-pull issue on production.
**Business objective delivered:** Identified and fixed the automated data usage monitoring system to properly enforce data suspension thresholds, protecting against over-usage charges.

---

# 2026 (40 tickets)

## Improvements & Features (26)

### Epic: WATM-1027 -- P2 - Device Management

#### WATM-1212 -- Button on assign that prohibits Company admin/super-admin from changing status of device to "Deactivated" for a certain number of billing cycles.
**Epic:** WATM-1027 -- P2 - Device Management | **Priority:** Normal
**What was accomplished:** Implemented a configurable field on the device assign and edit screens that prevents company admins from deactivating devices for a specified number of billing cycles after sale.
**Business objective delivered:** Protected the business from equipment losses by enforcing minimum activation periods after hardware sales, ensuring recovery of subsidized device costs.

### Epic: WATM-1691 -- Service Plan Management

#### WATM-1443 -- Service Plan Change modification idea - workload estimate request.
**Epic:** WATM-1691 -- Service Plan Management | **Priority:** Normal
**What was accomplished:** Implemented automatic device configuration updates when service plans are changed, setting the correct 24-hour data usage threshold based on the service plan parameters.
**Business objective delivered:** Automated a previously manual process of updating device configuration limits when service plans change, reducing human error and ensuring devices operate within their plan parameters.

### Epic: WATM-1743 -- APW Support 2025

#### WATM-1927 -- Config download on device screen
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** High
**What was accomplished:** Restricted the ability to download device configuration files from the browse device screen to super admin users only, while keeping the configuration name visible to all users.
**Business objective delivered:** Enhanced security by preventing non-admin users from downloading sensitive device configuration files while maintaining configuration visibility.

#### WATM-1854 -- Hide Average Signal Strength Graph Area
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Hidden the empty Average Signal Strength graph from the APC dashboard as a temporary measure while a long-term solution is planned.
**Business objective delivered:** Improved the dashboard presentation by removing a non-functional empty graph, providing a cleaner user experience.

#### WATM-1841 -- add logic for 50% data notifications
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** High
**What was accomplished:** Added a configurable "days after cycle start" field to data notifications, allowing 50% data usage alerts to be suppressed near the end of billing cycles.
**Business objective delivered:** Reduced unnecessary alert fatigue for customers by preventing late-cycle 50% data notifications when 90% and 100% alerts are more relevant.

#### WATM-1832 -- RMA for dual sim
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Implemented automatic dual SIM flag transfer during the RMA process, so replacement devices inherit the dual SIM setting when appropriate SIM combinations are present.
**Business objective delivered:** Streamlined the RMA workflow by automatically configuring replacement devices with dual SIM settings, reducing manual steps and potential configuration errors.

#### WATM-1826 -- Process offline devices job PING reorchestration
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Implemented a 3-attempt ping retry mechanism for the offline device checker with delayed retry processing via a new SQS queue, reducing false positive offline detections.
**Business objective delivered:** Significantly reduced false offline alerts (targeting 60%+ reduction), improving the reliability of device status monitoring and reducing unnecessary support escalations.

#### WATM-1794 -- Eradicate Device
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Designed and began implementation of a device eradication feature for WATM Super Admins with dual authorization, audit logging, and email notifications.
**Business objective delivered:** Provided administrators with a secure, audited process for permanently removing devices from the platform, with appropriate safeguards against accidental deletion.

#### WATM-1580 -- Payment Method Update Notifications for Distributor - SubCustomer scenario.
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Implemented email notifications to distributor Super Admins when subcustomers create or modify payment methods, including a direct link to the payment method in the portal.
**Business objective delivered:** Gave distributors real-time visibility into subcustomer payment changes, supporting financial oversight and fraud prevention across the distributor network.

#### WATM-1563 -- Deactivate/Reactivate Company
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Built company deactivation/activation functionality that preserves device states during deactivation and offers two restoration options: full activation or selective restoration of previous device statuses.
**Business objective delivered:** Provided administrators with a safe, reversible company deactivation process that preserves device history and prevents data loss during account management operations.

### Epic: WATM-1788 -- Data Purging Strategy

#### WATM-1594 -- Archive Data onto Amazon Glacier
**Epic:** WATM-1788 -- Data Purging Strategy | **Priority:** Normal
**What was accomplished:** Planned and began implementation of data archival to Amazon Glacier for device_checkins, device_status_logs (3+ months old), and provider_data_usages (2+ months old) to reduce database size.
**Business objective delivered:** Addressed growing database performance concerns by implementing a data archival strategy, improving query performance for active data while preserving historical records.

### Epic: WATM-1843 -- T-Mobile Sim Support in APW

#### WATM-1804 -- SIMPL / TMO API Build
**Epic:** WATM-1843 -- T-Mobile Sim Support in APW | **Priority:** High
**What was accomplished:** Built the T-Mobile/SIMPL API integration for managing T-Mobile SIM cards including activation, deactivation, and data usage retrieval.
**Business objective delivered:** Enabled the platform to programmatically manage T-Mobile SIM cards, supporting the expansion into multi-carrier IoT connectivity services.

### Epic: WATM-1865 -- Device Manufacturer → Device Model association management feature

#### WATM-375 -- Revisions to the Device manufacturer and model functionality
**Epic:** WATM-1865 -- Device Manufacturer → Device Model association management feature | **Priority:** Low
**What was accomplished:** Designed device manufacturer-to-model association management with validation rules for imports, placed on hold pending further evaluation of data structure compatibility.
**Business objective delivered:** Scoped a data organization improvement to enforce proper manufacturer-model relationships, improving data quality for device management.

### Epic: WATM-1914 -- Service Plan Management Enhancements

#### WATM-1919 -- Service Plan Management Enhancements Trello
**Epic:** WATM-1914 -- Service Plan Management Enhancements | **Priority:** Normal
**What was accomplished:** Delivered Service Plan Management Enhancements enabling custom tier pricing by carrier and/or model, with default options for broad application and phase 2 planning for multi-tier needs.
**Business objective delivered:** Gave administrators flexible service plan pricing tools that support carrier-specific and model-specific customization, enabling more competitive and granular pricing strategies.

#### WATM-1904 -- Distributor Single Device Minimum - Billing Rule Changes
**Epic:** WATM-1914 -- Service Plan Management Enhancements | **Priority:** Normal
**What was accomplished:** Implemented a $4.25 minimum distributor wholesale price for single-device-per-payment-method scenarios, with clear messaging to distributors about how the rule affects their commissions.
**Business objective delivered:** Protected business margins on single-device accounts by enforcing minimum wholesale pricing that covers ACH fees and carrier costs, while providing transparent communication to distributors.

#### WATM-1483 -- Change customized service plan - adjusted price logic.
**Epic:** WATM-1914 -- Service Plan Management Enhancements | **Priority:** Normal
**What was accomplished:** Changed service plan customization logic so adjusted prices are treated as absolute values rather than relative adjustments, preventing unintended price changes when base prices are modified.
**Business objective delivered:** Ensured custom pricing stability for distributor accounts, preventing unexpected price changes when base service plan prices are adjusted by administrators.

### Epic: WATM-1964 -- APW Support 2026

#### WATM-1984 -- Dual Carrier Fee Handling edits
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Normal
**What was accomplished:** Implemented separate dual carrier fee fields for AT&T ($4.95 default) and T-Mobile ($3.50 default) with company-level customization and automatic fee selection based on device secondary carrier.
**Business objective delivered:** Enabled differentiated pricing for dual carrier configurations by carrier type, supporting competitive T-Mobile dual carrier rates while maintaining existing AT&T pricing.

#### WATM-1965 -- checkbox for power relay
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** High
**What was accomplished:** Added a "Power Relay Installed?" informational checkbox to the device page and a corresponding column in the device export for customer self-service tracking.
**Business objective delivered:** Enabled customers to track power relay installation status within the portal, improving asset management visibility across their device fleet.

#### WATM-1954 -- Data suspension status - customer view
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** High
**What was accomplished:** Restricted the "Data Suspension" status option to Super Admin users only, preventing regular customers from setting this status on their devices.
**Business objective delivered:** Tightened access controls on device status management, ensuring only authorized administrators can apply data suspension status.

#### WATM-1939 -- Verizon SIM Activation Process Change
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** High
**What was accomplished:** Implemented a SIM-AND-SKU activation method for Verizon SIM-only devices that lack an IMEI, using the universal SKU identifier instead of the previously failing IMEI+ICCID method.
**Business objective delivered:** Resolved silent Verizon activation failures for standalone SIM cards, eliminating the need for manual intervention and enabling automated SIM management.

#### WATM-1931 -- Export for Failed Check-ins
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Normal
**What was accomplished:** Added an Export to Excel feature for the Browse Device Check-in Failures screen with date range filtering and email delivery, following the existing device export pattern.
**Business objective delivered:** Gave administrators the ability to export and analyze check-in failure data, enabling better diagnostics of device connectivity issues across the fleet.

#### WATM-1920 -- Clean up email distribution list
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Normal
**What was accomplished:** Removed the former ATM Partner email address from the company pending deactivation notification distribution list, updating it to only go to the current Allpoint Wireless service email.
**Business objective delivered:** Updated system notifications to reflect current business contact information after the company restructuring.

#### WATM-1842 -- Distributors adding ACH information
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Normal
**What was accomplished:** Placeholder ticket for distributors adding ACH information, awaiting further requirements from the client.
**Business objective delivered:** Ticket created for future ACH workflow improvements pending client specifications.

#### WATM-1581 -- Dynamic Access Lists for VPN
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Normal
**What was accomplished:** Advanced the Dynamic Access Lists for VPN feature from proof of concept to implementation planning, building on the initial POC validated with PaloAlto firewall integration.
**Business objective delivered:** Progressed the VPN access control solution toward production deployment, moving beyond validation to active development of the security infrastructure.

### Uncategorized

#### WATM-1980 -- Annual Commissions Report: Add Tax ID and Update Addresses
**Epic:** No Epic -- Uncategorized | **Priority:** Normal
**What was accomplished:** Added Tax ID and updated company addresses in the Annual Commissions Report export.
**Business objective delivered:** Enhanced the annual commissions report with essential tax filing data, streamlining year-end compliance processes.

#### WATM-1977 -- 2FA logic on configuration changes
**Epic:** No Epic -- Uncategorized | **Priority:** Normal
**What was accomplished:** Planned 2FA logic implementation for device configuration changes to add an additional security layer.
**Business objective delivered:** Defined requirements for enhanced security controls on sensitive device configuration operations.

## Bug Fixes (14)

### Epic: WATM-1125 -- P2 - Admin

#### WATM-1107 -- Issue with phone number when creating WATM Admin on mobile device
**Epic:** WATM-1125 -- P2 - Admin | **Priority:** Normal
**What was accomplished:** Investigated a phone number formatting issue when creating WATM Admin users on mobile devices, providing guidance on the required phone format for mobile browsers.
**Business objective delivered:** Addressed mobile usability concern for admin user creation, ensuring administrators can manage users from any device.

### Epic: WATM-1743 -- APW Support 2025

#### WATM-1912 -- Bug on Connected Equipment and Firewall Functionality
**Epic:** WATM-1743 -- APW Support 2025 | **Priority:** Normal
**What was accomplished:** Fixed a bug where interacting with the firewall function on ATM plan devices would incorrectly clear the firewall configuration by sending an empty fw_acl parameter.
**Business objective delivered:** Prevented accidental clearing of ATM device firewall rules, protecting the security configuration of devices on the ATM service plan.

### Epic: WATM-1964 -- APW Support 2026

#### WATM-2008 -- Tmobile Signal on Browse Devices Screen/Last Signal Strength
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Normal
**What was accomplished:** Fixed the Browse Devices screen to correctly show T-Mobile as the active carrier for dual-SIM devices where the VZW SIM is inactive and TMO SIM is active, using IP address pattern detection.
**Business objective delivered:** Improved carrier identification accuracy on the device browse screen, ensuring the correct carrier label is displayed for dual-SIM devices.

#### WATM-2007 -- Email addresses exist but can't be seen?
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** High
**What was accomplished:** Fixed the Browse Companies email search to exclude soft-deleted and inactive users, which were incorrectly appearing in search results due to wildcard matching on deleted email suffixes.
**Business objective delivered:** Corrected the company search functionality to return only valid, active employee records, preventing confusion when searching by email address.

#### WATM-2005 -- Bulk Action - change service plan
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** High
**What was accomplished:** Fixed a critical bug in the bulk action service plan change process that caused an internal error while still sending API calls to Verizon, leaving the portal out of sync with carrier data.
**Business objective delivered:** Restored reliable bulk service plan management, ensuring portal data stays synchronized with Verizon when processing bulk changes.

#### WATM-2000 -- Company users should not be able to edit device model
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Critical
**What was accomplished:** Restricted the device model field to be non-editable by company users, limiting model changes to WATM administrators only.
**Business objective delivered:** Strengthened data integrity controls by preventing non-admin users from modifying device model assignments.

#### WATM-1983 -- Last 30 Days Signal Chart Bug
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** High
**What was accomplished:** Fixed a timezone-related bug causing the Last 30 Days Signal Strength chart to display data points one day into the future.
**Business objective delivered:** Corrected signal strength chart date alignment, ensuring data points appear on the correct date for accurate historical signal analysis.

#### WATM-1975 -- Great Lakes Amusement - subs see whitelabeled invoice but they didn't set up a whitelabeled name
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Critical
**What was accomplished:** Fixed an issue where subcustomer invoices for Great Lakes Amusement incorrectly displayed the distributor name in the invoice header even though white-labeling was not configured.
**Business objective delivered:** Ensured invoice branding is only applied when explicitly configured through white-labeling, preventing unauthorized distributor name exposure on subcustomer invoices.

#### WATM-1969 -- Critical: DC Configuration Check Box BUG
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Critical
**What was accomplished:** Investigated a reported configuration checkbox bug and confirmed the behavior was correct per the existing device configuration matrix; client acknowledged the expected behavior.
**Business objective delivered:** Confirmed system accuracy by validating the dual carrier configuration logic against established business rules.

#### WATM-1968 -- Critical: Sub Customer Browse Devices Screen Error
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** High
**What was accomplished:** Investigated a reported browse devices screen error for a sub-customer and was unable to reproduce the issue; the client was asked to verify if the problem persisted.
**Business objective delivered:** Provided responsive troubleshooting support for a customer-reported display issue, with the problem appearing to be temporary.

#### WATM-1966 -- update to annual commissions report page
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Normal
**What was accomplished:** Fixed the "Generate Tax Documents" button to work as a background process with email confirmations, added company address to the export, and improved the default year selection.
**Business objective delivered:** Streamlined the annual tax document generation process with background processing, better notifications, and improved usability defaults.

#### WATM-1952 -- ATT Sim Card Active->Inactive status switch bug
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Normal
**What was accomplished:** Fixed the AT&T SIM deactivation process for dual carrier devices during customer suspension, ensuring active SIMs in "Test Ready" status are properly deactivated while preserving inactive SIM states.
**Business objective delivered:** Ensured reliable carrier API integration for mass device suspension operations, preventing SIM status inconsistencies across hundreds of dual carrier devices.

#### WATM-1936 -- Verizon Status Switch Bug
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Normal
**What was accomplished:** Fixed a Verizon status switch bug where the true device state was being read incorrectly, with a "deactive" device being returned as "active" in the API response parsing.
**Business objective delivered:** Corrected carrier status detection to accurately read Verizon device states, preventing incorrect status changes and failed API calls.

#### WATM-1893 -- Fix Pagination Duplicate Entries in Browse Commissions Details View
**Epic:** WATM-1964 -- APW Support 2026 | **Priority:** Normal
**What was accomplished:** Fixed a pagination bug in the Browse Commissions Details view where the same invoice entries appeared duplicated across multiple pages due to a sorting/offset issue.
**Business objective delivered:** Corrected the commission detail display so distributors see accurate, non-duplicated data when reviewing their commission breakdowns.

---

# Summary of Key Business Outcomes

## 2024 Highlights

### Multi-Carrier Expansion
T-Mobile SIM support was added to device management, import/export, and dashboard views (WATM-1653, WATM-1654, WATM-1655), enabling the business to expand beyond Verizon and AT&T into a three-carrier ecosystem.

### Signal Quality Visibility
SINR signal strength metrics were added to device exports, charts, and reports with informational tooltips (WATM-1331, WATM-1652, WATM-1618), giving customers comprehensive tools to diagnose connectivity issues.

### Billing System Corrections
Critical billing defects were resolved including flat rate devices not being invoiced (WATM-1524, WATM-1648), double-billing for flat rate charges (WATM-1662), and earnings report accuracy (WATM-1628). These fixes protected revenue integrity and customer trust.

### Distributor Commission Infrastructure
Key commission features were delivered including earnings report summary pages (WATM-1487), automatic commission payment method setup (WATM-1424), 1099 consent management (WATM-1554), and tax document email notifications (WATM-1330).

### Service Plan Customization
New interfaces for both WATM Admins (WATM-1646) and Company Super Admins (WATM-1641) were built for managing service plan customizations, including the ability to apply pricing across all subcustomers of a distributor.

### UI/UX Improvements
Browse Devices and Billing Cycles pages were overhauled with horizontal scrolling (WATM-1074, WATM-1608), dashboard export buttons were added (WATM-1080, WATM-1084), and email templates were updated with improved specificity (WATM-1629).

## 2025 Highlights

### System Performance & Reliability
Major infrastructure improvements were completed including bulk check-in processing (WATM-1647, WATM-1649), the enhanced multi-process offline device checker (WATM-1706), and a 3-attempt ping retry mechanism (WATM-1826). These changes dramatically improved system throughput, reduced false offline alerts, and improved monitoring reliability across 80,000+ devices.

### Billing Module Redesign
The billing module was redesigned with a unified card-based interface (WATM-1840) that generates all billing documents from a single data snapshot, eliminating data misalignment between invoices, NACHA files, and reports. Invoice generation reliability was improved with retry mechanisms (WATM-1780), and payment method update timing issues were resolved (WATM-1660).

### Connected Equipment & Firewall
The Connected Equipment feature was redesigned (WATM-1310) with real-time status visibility and firewall rule management (WATM-1290), giving customers granular network security control. Device lock functionality was fixed for multi-device scenarios (WATM-1745).

### VPN Access Control
VPN Access Control Lists were implemented (WATM-1712) with automatic IP address maintenance and PaloAlto firewall integration, progressing from proof of concept (WATM-1593) to production tooling.

### Device Power Management
A new power management scheduling feature (WATM-1800) was delivered, enabling customers to set automated on/off schedules for IoT relay devices with day-of-week granularity and override capabilities.

### Critical Bug Resolution
Numerous critical defects were addressed including login failures from dashboard chart queries (WATM-1781), RMA creation errors (WATM-1705), bulk action failures (WATM-1740, WATM-1759), device status override bugs (WATM-1725, WATM-1709), and invoice delivery failures for companies without primary users (WATM-1783). Invoice email CC distribution was fixed (WATM-1664).

### T-Mobile Integration
The T-Mobile/SIMPL API integration was built (WATM-1804) for managing T-Mobile SIM lifecycle including activation, deactivation, and data usage retrieval, and T-Mobile data usage API calls were corrected (WATM-1937).

### Data & Notifications
Data notification logic was enhanced to suppress late-cycle 50% alerts (WATM-1841), notification opt-out options were added at the employee level (WATM-1830), and the SMS notification option was removed from the UI since the feature was disabled (WATM-1795). Sub-company notification routing was scoped (WATM-1704).

## 2026 Highlights (Year to Date)

### Service Plan Management Enhancements
Major improvements to service plan pricing were delivered including carrier/model-specific tier pricing (WATM-1919), absolute value customization logic (WATM-1483), and distributor single-device minimum pricing rules (WATM-1904). These changes provide more granular and accurate pricing control across the distributor network.

### Dual Carrier Fee Differentiation
Separate dual carrier fee fields were implemented for AT&T and T-Mobile (WATM-1984), enabling competitive pricing for T-Mobile dual carrier while maintaining existing AT&T rates. Carrier identification on the browse screen was fixed for dual-SIM devices (WATM-2008).

### Data Archival & Performance
Data archival to Amazon Glacier continued (WATM-1594) to address growing database size and query performance, targeting device check-ins, status logs, and provider data usage records.

### Security & Access Controls
Device model editing was restricted to administrators only (WATM-2000), Data Suspension status was limited to Super Admins (WATM-1954), configuration download was restricted (WATM-1927), and Verizon SIM activation was updated to use the SIM-AND-SKU method (WATM-1939) for devices without IMEIs.

### Annual Reporting & Tax Compliance
The annual commissions report was improved with background processing, email confirmations, and company address data (WATM-1966). Tax ID and address fields were added to the report export (WATM-1980), and the email distribution list was updated for current company contacts (WATM-1920).

### Operational Improvements
Bulk action service plan changes were fixed (WATM-2005), company email search was corrected to exclude deleted users (WATM-2007), pagination duplicates in commission views were resolved (WATM-1893), and the signal strength chart timezone alignment was corrected (WATM-1983). White-labeling invoice branding was fixed for non-configured distributors (WATM-1975).
