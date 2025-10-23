Project Specification: Lone Worker Safety System v2.1

Version: 2.1 Oct 23, 2025

1. Overview

This document specifies a complete, free, self-hosted Lone Worker Safety System designed for organizations to monitor staff working alone. The system includes a worker smartphone app, a monitor dashboard, a Google Apps Script backend, and a web-based deployment tool for easy setup by administrators with average computer skills. The deployment tool generates customized, ready-to-host versions of the worker and monitor applications.

2. System Architecture

The system comprises four main parts:

The Lone Worker App: A Progressive Web App (PWA) for the worker's smartphone.

The Monitoring App: A web-based dashboard for the safety monitor.

The Google Apps Script Backend: A script deployed as a web app from a Google Sheet, acting as the database and logic controller.

The Deployment Tool: A web-based tool (setup.html) hosted online (e.g., GitHub Pages) that generates the customized Worker and Monitor apps.

3. The Lone Worker App (PWA)

This is the primary interface for the lone worker, designed to be installed on their smartphone home screen.

3.1. Core Features

Location Management:

Users can add, edit, and delete visit locations (name and address).

Includes a default, non-deletable "Travelling" location.

Ability to fetch current GPS coordinates and reverse geocode to an address (using OpenStreetMap Nominatim API) when adding/editing locations.

Visit Timer:

User selects a location and a visit duration from predefined steps (0, 10, 15, 20, 30, 45, 60, 75, 90, 105, 120, 150, 180, 240, 300 minutes) using a slider.

Starts the timer by pressing and holding a "Start" button for 1.5 seconds. (Button text changed from "ON SITE").

Sends initial visit data (including contact details from settings) to the backend with status ON SITE.

Locked Screen:

Activated once a visit starts.

Displays a large countdown timer (shows +HH:MM:SS when overdue).

Shows the selected location name and anticipated departure time.

Provides access to "Extend," "Depart," and "SOS" functions via buttons.

Displays an "ALERT ACTIVE" section with an "I AM SAFE" button when an alert status is triggered (e.g., ALERT_SENT, MISSED_CHECKIN, EMERGENCY - PANIC BUTTON).

Panic Button (SOS):

A prominent SOS button on the locked screen.

Triple-tapping this button immediately triggers a high-priority EMERGENCY - PANIC BUTTON alert, sending data (including attempted GPS coordinates) to the backend.

PIN & Duress PIN:

Users define a 4-digit numeric PIN and a separate 4-digit numeric Duress PIN in settings.

Extending a visit requires the normal PIN.

Cancelling an active alert (using the "I AM SAFE" button) requires the PIN.

Entering the Duress PIN instead of the normal PIN when cancelling an alert makes the app appear to end the visit normally but silently sends a high-priority DURESS_CODE_ACTIVATED alert to the backend.

"Are You OK?" Check-ins (Optional & Configurable):

Feature is enabled/disabled via a flag (CHECKIN_ENABLED) injected by the Deployment Tool.

If enabled, the interval (in minutes) is set by a variable (CHECKIN_INTERVAL) also injected by the Deployment Tool.

Prompts the worker with a modal asking "Are you OK?" at the configured intervals during an active visit.

Worker has a 2-minute countdown to tap "I am OK".

Escalating Alert: The prompt includes audible beeps (using Tone.js) and vibrations. The sound volume starts low (-20dB) and increases towards full volume (0dB) during the final minute. Sound and vibration frequency also increase in the final minute.

Tapping "I am OK" resets the check-in timer, resets the synth volume, and dismisses the modal.

Missing the 2-minute window triggers a MISSED_CHECKIN alert sent to the backend and resets the synth volume.

GPS Tracking:

Attempts to get GPS coordinates on triggering any alert (PANIC, DURESS, MISSED_CHECKIN, ALERT_SENT).

When the "Travelling" location is selected, attempts to send a GPS update to the backend every 15 minutes.

GPS retrieval uses navigator.geolocation.getCurrentPosition with high accuracy enabled and a timeout (e.g., 10-20 seconds).

Notifies the user (once via alert, and in info modal) that the app must remain open/in the foreground for reliable GPS tracking.

Low Battery Warning:

Checks phone battery level (using navigator.getBattery) at the start of a visit and periodically during the tick function.

If battery < 15% and not charging, sends a note (LOW BATTERY WARNING: XX%) to the backend (once per visit).

Departure:

Pressing and holding the "DEPART" button sends an update to the backend with status DEPARTED and the actual departure time.

Ends the active visit in the app, clears timers, and returns to the main screen.

Settings:

Accessible via a gear icon.

Allows user to input: Worker Name, Worker Phone, Emergency Contact (Name, Phone, Email), Escalation Contact (Name, Phone, Email), Normal PIN, Duress PIN.

The Google Sheet Web App URL field is pre-filled (via placeholder %%GOOGLE_SHEET_URL%% replaced by setup tool) and disabled after the first save.

Settings are saved to localStorage.

Info/Help Modal:

Accessible via an info icon.

Provides instructions on installation (Add to Home Screen), setup, usage, alerts, duress, GPS limitations, and a privacy notice regarding data collection.

3.2. Technical Specifications

Platform: Single-page HTML application. Must be installable as a Progressive Web App (PWA).

Styling: Tailwind CSS (via CDN). Must be responsive and usable on various smartphone screen sizes. Dark theme UI.

JavaScript: Vanilla JavaScript. ES6+ features allowed.

Audio: Tone.js library (via CDN) for distinct sounds for actions (arrive, depart, extend, safe, warning, panic, check-in) and escalating check-in alerts (volume control via synth.volume.value). Audio context must be initialized on user interaction. Synth volume reset to 0 after check-in alert ends (OK or timeout).

Storage: localStorage for user settings, locations, and active visit state (to allow resuming if app is closed/refreshed).

Backend Communication: Uses fetch API with POST method (mode: 'no-cors') to send data to the Google Apps Script Web App URL. Data sent as URL-encoded form data.

PWA Files: Requires manifest.json (with configurable logo URL) and sw.js (basic service worker for installability) to be present in the same directory as the main index.html.

Error Handling: Use custom modal dialogs for alerts/errors (no native alert() or confirm()). Display informative messages for network errors, GPS failures, incorrect PINs, etc.

Long Press Buttons: Critical actions (Start, DEPART, EXTEND, I AM SAFE) require a 1.5-second press-and-hold, with visual feedback during the press.

4. The Monitoring App

A web-based dashboard for the safety monitor, accessible via a URL generated during setup.

4.1. Core Features

Secure Auto-Login:

The Google Apps Script URL and Secret Key are hardcoded into the final generated index.html file by the Deployment Tool.

No setup or login screen is presented to the monitor; the app loads directly into the dashboard.

Live Dashboard:

Periodically fetches data (every 15 seconds) from the Google Apps Script backend using the JSONP method.

Displays a list of all currently "active" workers (statuses like ON SITE, ALERT SENT, MISSED_CHECKIN, EMAIL_1_SENT etc., but not DEPARTED, SAFE - MANUALLY CLEARED, MONITOR_CLEARED_ALERT).

Status Display:

Each active worker is shown on a card.

Card displays: Worker Name, Location Name, Anticipated Departure Time, Current Alarm Status.

Cards are sorted by alert priority (highest first), then alphabetically by worker name.

Prioritized Alerts & Visual Cues:

Cards are color-coded based on Alarm Status:

Purple (Flashing): DURESS_CODE_ACTIVATED (Highest)

Red (Flashing): EMERGENCY - PANIC BUTTON

Orange/Yellow: ESCALATION_SENT, MISSED_CHECKIN, EMAIL_3_SENT, EMAIL_2_SENT, EMAIL_1_SENT, ALERT SENT

Gray: ON SITE (Normal)

Flashing alerts use CSS keyframe animations.

Audible Alarms:

Uses Tone.js library (via CDN).

Loud, repeating alarm sound triggers for new high-priority alerts (DURESS, PANIC, MISSED_CHECKIN, ALERT_SENT). Requires user interaction to acknowledge/silence via an overlay.

Softer, brief chime sounds for status updates (e.g., EMAIL_1_SENT -> EMAIL_2_SENT).

Alarm overlay displays the name, location, and status of the triggering worker.

Low Battery Indicator:

A distinct icon appears next to the worker's name if their record includes a "LOW BATTERY" note.

Database Size Warning:

If the fetched data array contains 2500 or more rows, a dismissible banner appears warning the monitor to advise the administrator to archive old data.

One-Click Location:

On worker cards with an alert status, a "View Location" link appears.

Links to Google Maps, prioritizing Last Known GPS coordinates if available, otherwise using the Location Address. Opens in a new browser tab.

Declare Safe (Manual Resolve):

Each alert card has a "Manually Resolve Alert" button.

Clicking it opens a confirmation modal.

Monitor must type the worker's name exactly into an input field to confirm.

On confirmation, the Monitor App sends a POST request (mode: 'no-cors') to the backend script, updating that worker's status to MONITOR_CLEARED_ALERT and adding a note.

The dashboard refreshes, removing the resolved worker.

Connection Status:

A visual indicator (e.g., colored dot) shows the status of the connection to the backend (Green: OK, Yellow: Fetching, Red: Error/Denied).

Displays the time of the last successful update.

4.2. Technical Specifications

Platform: Single-page HTML application. Designed primarily for desktop browsers but should be reasonably responsive.

Styling: Tailwind CSS (via CDN). Dark theme UI.

JavaScript: Vanilla JavaScript. ES6+ features allowed.

Audio: Tone.js library (via CDN) for alarms and chimes.

Storage: No localStorage needed, as URL/Key are hardcoded.

Backend Communication:

Uses JSONP method (dynamically creating <script> tags with a callback) for GET requests to fetch data, passing the hardcoded Secret Key as a URL parameter (&token=).

Uses fetch API with POST method (mode: 'no-cors') for sending the "Manual Resolve" update.

Error Handling: Use custom modal dialogs for alerts/errors (no native alert() or confirm()). Display informative messages for connection failures, access denied errors.

5. The Google Apps Script Backend

The central controller deployed from the administrator's Google Sheet.

5.1. Core Functions

doGet(e):

Handles GET requests from the Monitoring App.

Expects a token URL parameter and validates it against the SECRET_KEY.

If valid, reads all data from the active spreadsheet.

Formats the data as a JSON array of objects (one object per row, keys matching headers).

Wraps the JSON in the callback function name provided in the URL parameters (JSONP format).

Returns the JSONP response with ContentService.MimeType.JAVASCRIPT.

Returns an error JSON object if the token is invalid.

doPost(e):

Handles POST requests from the Worker App (status updates, alerts) and the Monitor App (manual resolve).

Uses LockService to prevent race conditions during writes.

Receives data via e.parameter.

Identifies the target row by matching both "Worker Name" and "Arrival Time" from the incoming data against existing rows.

If a matching row is found, updates the values in relevant columns based on the received data.

If no match is found (typically only for the first ON SITE post from a worker), appends a new row with all received data.

If the received Alarm Status is EMERGENCY - PANIC BUTTON, DURESS_CODE_ACTIVATED, or MISSED_CHECKIN, immediately calls sendAlertEmail.

checkOverdueWorkers():

Designed to be run by a time-driven trigger (every 5 minutes recommended).

Uses LockService.

Reads all spreadsheet data.

Iterates through rows, ignoring those with "resolved" statuses (DEPARTED, SAFE - MANUALLY CLEARED, MONITOR_CLEARED_ALERT).

For non-resolved rows, checks if the current time is past the Anticipated Departure Time.

Calculates overdue minutes.

Sends escalating alerts based on overdue time and current status (to prevent duplicate emails):

>= 15 mins & status is ON SITE or ALERT SENT: Set status to EMAIL_1_SENT, call sendAlertEmail.

>= 30 mins & status is not EMAIL_2_SENT, EMAIL_3_SENT, ESCALATION_SENT: Set status to EMAIL_2_SENT, call sendAlertEmail.

>= 45 mins & status is not EMAIL_3_SENT, ESCALATION_SENT: Set status to EMAIL_3_SENT, call sendAlertEmail.

>= 60 mins & status is not ESCALATION_SENT: Set status to ESCALATION_SENT, call sendAlertEmail.

Updates the Alarm Status cell in the sheet for rows where an alert is sent.

sendAlertEmail(workerData, alertType):

Constructs and sends formatted HTML emails via MailApp.sendEmail.

Determines recipient: Escalation Contact Email if alertType is ESCALATION_SENT, otherwise Emergency Contact Email.

Generates subject and body content based on alertType (PANIC, DURESS, MISSED_CHECKIN, EMAIL_1/2/3_SENT, ESCALATION_SENT).

Includes key worker and visit details (Name, Phone, Location Name/Address, Anticipated Departure).

Includes a Google Maps link (prioritizing GPS if available, otherwise address).

Logs success or failure of email sending.

5.2. Spreadsheet Structure

The script assumes the active sheet has headers in Row 1 matching exactly:
Date,Worker Name,Worker Phone Number,Emergency Contact Name,Emergency Contact Number,Emergency Contact Email,Escalation Contact Name,Escalation Contact Number,Escalation Contact Email,Location Name,Location Address,Arrival Time,Anticipated Departure Time,Actual Departure Time,Alarm Status,Notes,Last Known GPS,GPS Timestamp

No other columns should be added, removed, or renamed.

5.3. Deployment & Configuration

Secret Key: A SECRET_KEY variable must be defined at the top of the script. This value is injected by the Deployment Tool during setup.

Deployment: Must be deployed as a Web app (Deploy > New deployment).

Execution: Execute as: Me (the administrator deploying it).

Access: Who has access: Anyone.

Trigger: A time-driven trigger must be manually configured to run checkOverdueWorkers every 5 minutes.

6. The Deployment Tool (setup.html)

A single HTML file designed to be hosted online (e.g., GitHub Pages) that guides an administrator through setting up the system and generating the final app files.

6.1. Functionality

Wizard Interface: Provides a multi-step user interface (using JavaScript to show/hide steps).

Template Fetching: On load, uses the fetch API to retrieve the content of all template files (.txt, .json, .js, .md) located in a ./templates/ subfolder relative to its own location. Displays an error if templates cannot be loaded.

Step-by-Step Guidance: Each step corresponds to a part of the Administrator Setup Guide:

Welcome screen.

Instructions for Google Sheet setup, including a "Copy Headers" button (navigator.clipboard). Includes instructions on using "Split text to columns".

Instructions for Apps Script deployment. Includes a field to Create Secret Key. Includes a "Copy Script" button that copies the script template text with the created Secret Key injected (navigator.clipboard). Displays a preview of the script code. Instructions reordered for clarity. Key terms are bolded.

Configuration fields:

Paste Apps Script Web App URL (with oninput validation for format https://script.google.com/macros/s/.../exec).

Secret Key field (read-only, type="text", auto-filled from Step 3).

First Overdue Alert (Minutes) (number input, default 15).

Enable Check-ins (checkbox).

Check-in Interval (Minutes) (number input, default 30, only shown if checkbox is checked).

Company Logo URL (optional URL input).

"Test Connection & Proceed" button: Performs a JSONP request to the entered Web App URL with the Secret Key to verify the connection and key before allowing progression. Displays success/error status.

Download screen with a "Generate & Download .zip Package" button.

Zip Generation:

Uses the JSZip library (via CDN).

Reads the fetched template content.

Injects the administrator's configured values (URL, Key, Alert Time, Check-in settings, Check-in Interval, Logo URL) into the appropriate placeholders (%%PLACEHOLDER%%, YOUR_SECRET_KEY_HERE) within the worker app, monitor app, manifest, and Apps Script templates.

Removes the setup page, reset button, and login logic from the monitor app template using comment placeholders (<!-- PLACEHOLDER_START -->...<!-- PLACEHOLDER_END -->).

Updates the Administrator Setup Guide.md template text to reflect the simplified workflow (no manual key editing needed).

Packages the modified files into the correct folder structure (LoneWorkerApp/, MonitoringDashboardApp/, Documentation/, 01-PASTE_THIS_INTO_APPS_SCRIPT.txt) within a .zip file.

Triggers a browser download of the generated LWS_Deployment_Package.zip file.

Displays status messages during generation and download.

6.2. Technical Specifications

Platform: Single HTML file. Must work correctly when hosted on a static web server (like GitHub Pages) over HTTPS.

Styling: Tailwind CSS (via CDN). Clean, user-friendly interface. Bold text used for key terms in instructions.

JavaScript: Vanilla JavaScript. Uses fetch for templates, navigator.clipboard for copy buttons, dynamic <script> tag creation for JSONP test, JSZip library for zip creation. type="button" added to all onclick buttons. Includes basic checks for element existence. Robust error handling for template loading and zip generation.

Dependencies: JSZip library (via CDN). Assumes template files are located at ./templates/.

Error Handling: Displays clear error messages if templates fail to load, the connection test fails, or zip generation fails. Includes basic checks for element existence in JavaScript.

6.3. Required Template Files (in ./templates/)

apps_script_template.txt (Contains YOUR_SECRET_KEY_HERE placeholder)

worker_app_template.txt (Contains %%FIRST_ALERT_MINUTES%%, %%CHECKIN_ENABLED%%, %%CHECKIN_INTERVAL%%, %%GOOGLE_SHEET_URL%% placeholders and icon URL)

monitor_app_template.txt (Contains <!-- SETUP_PAGE_START -->, <!-- SETUP_PAGE_END -->, <!-- RESET_BUTTON_START -->, <!-- RESET_BUTTON_END -->, <!-- MONITOR_LOGIN_LOGIC_START -->, <!-- MONITOR_LOGIN_LOGIC_END --> placeholders)

manifest.json (Contains icon URL)

sw.js

Administrator Setup Guide.md

Installer Creation Guide.md

Promotional Blurb.md
