# Project Specification: Lone Worker Safety System v2.7

**Version:** 2.7 

## 1. Overview

This document specifies a complete, free, self-hosted Lone Worker Safety System designed for organizations to monitor staff working alone. The system includes a worker smartphone app, a monitor dashboard, a Google Apps Script backend, and a web-based deployment tool for easy setup by administrators with average computer skills. The deployment tool generates customized, ready-to-host versions of the worker and monitor applications.

## 2. System Architecture

The system comprises four main parts:

1.  **The Lone Worker App:** A Progressive Web App (PWA) for the worker's smartphone.
2.  **The Monitoring App:** A web-based dashboard for the safety monitor.
3.  **The Google Apps Script Backend:** A script deployed as a web app from a Google Sheet, acting as the database and logic controller.
4.  **The Deployment Tool:** A web-based tool (`setup.html`) hosted online (e.g., GitHub Pages) that generates the customized Worker and Monitor apps.

## 3. The Lone Worker App (PWA)

This is the primary interface for the lone worker, designed to be installed on their smartphone home screen.

### 3.1. Core Features

* **Location Management:** Add, edit, delete locations (name/address). Default "Travelling". GPS lookup via Nominatim.
* **Visit Timer:** Select location, duration (slider, predefined steps). Hold **"Start"** button (1.5s) to begin. Sends `ON SITE` status.
* **Locked Screen:** Shows countdown (+ overdue time), location, departure time. Buttons for Extend, Depart, SOS. Shows "ALERT ACTIVE" section when needed.
* **Panic Button (SOS):** Triple-tap sends `EMERGENCY - PANIC BUTTON` alert + GPS attempt.
* **PIN & Duress PIN:** 4-digit numeric PINs set in settings. Normal PIN for Extend/Safe. Duress PIN on "I AM SAFE" sends silent `DURESS_CODE_ACTIVATED` alert.
* **"Are You OK?" Check-ins (Optional & Configurable):** Enabled via `CHECKIN_ENABLED` flag. Interval set via `CHECKIN_INTERVAL` (minutes). Both injected by Deployment Tool. Modal prompt with 2-min countdown. **Escalating Alert:** Sound volume (`Tone.js`) increases from -20dB to 0dB in last minute. Sound/vibration frequency increases in last minute. OK resets timer/volume. Timeout sends `MISSED_CHECKIN` alert.
* **GPS Tracking:** Attempts GPS on alerts. Every 15 mins if "Travelling". `navigator.geolocation`, high accuracy, timeout. User warned about foreground requirement.
* **Low Battery Warning & Reporting:**
    * Checks battery (`navigator.getBattery`) at visit start and periodically (e.g., every 15 mins via `tick`).
    * Sends initial battery level in `Notes` (e.g., "Battery: 85%").
    * If level drops < 15% (and not charging), sends a note (`LOW BATTERY WARNING: 14%`) **once per visit** and sets internal flag (`batteryAlertSent`).
    * Sends regular battery level updates in `Notes` (e.g., "Battery: 60%") periodically (e.g., 15 mins) *only if* the low warning hasn't been sent for the current visit.
* **Departure:** Hold "DEPART" button (1.5s). Sends `DEPARTED` status. Ends visit.
* **Settings:** Inputs for Name, Phone, Contacts (Emergency/Escalation), PINs. URL field pre-filled (`%%GOOGLE_SHEET_URL%%`) and disabled after save. Uses `localStorage`.
* **Info/Help Modal:** Instructions on install, setup, use, alerts, duress, GPS, privacy.

### 3.2. Technical Specifications

* **Platform:** Single-page HTML PWA.
* **Styling:** Tailwind CSS (CDN). Responsive dark theme.
* **JavaScript:** Vanilla JS, ES6+.
* **Audio:** `Tone.js` (CDN) for actions and escalating check-in alerts (volume control). Synth volume reset after check-in.
* **Storage:** `localStorage`.
* **Backend Communication:** `fetch` API, `POST`, `mode: 'no-cors'`, URL-encoded data.
* **PWA Files:** Requires `manifest.json` (configurable logo) and `sw.js` (basic) in the same directory.
* **Error Handling:** Custom modal dialogs.
* **Long Press Buttons:** 1.5s hold for Start, DEPART, EXTEND, I AM SAFE.

## 4. The Monitoring App

Web-based dashboard for the safety monitor.

### 4.1. Core Features

* **Secure Auto-Login:** URL and Secret Key hardcoded. No login screen.
* **Header Display:** Includes Logo, title, connection status, last update time, notification status/button.
* **Session Alert History Log:**
    * Collapsible `<details>` panel below header.
    * Logs key events (Session start, Worker active, Status change, Alert acknowledged, Manual resolve, Notification status) with timestamps for the current session.
    * Limited to max entries (e.g., 50), stored in-memory. Clears on reload.
* **Live Dashboard:** Fetches data (JSONP, 15s interval). Displays "active" workers in a grid layout.
* **Status Display & Sorting:** Cards show Name, Location, Departure Time, Status. Sorted by priority, then name.
* **Prioritized Alerts & Visual Cues:** Color-coded cards: Purple/Flashing (Duress), Red/Flashing (Panic), Orange/Yellow (Overdue/Missed), Gray (On Site). CSS animations.
* **Enhanced Highlighting for Overdue (Pre-Alert) Workers:**
    * If status is `ON SITE` but `Anticipated Departure Time` is past, card gets amber border (`border-pre-alert`) and "OVERDUE" badge.
* **Audible Alarms:** `Tone.js`. Loud repeating alarm for new critical alerts, requires acknowledgment via overlay. Softer chime for status updates. Overlay shows trigger details.
* **Browser/Desktop Notifications:**
    * Uses `Notification` API. Button to request permission if needed.
    * Shows system notification for new critical alerts (`PANIC`, `DURESS`, `MISSED_CHECKIN`, `ALERT_SENT`) with worker name, status, location, and logo.
* **Worker Details Displayed on Card:**
    * **Battery Level:** Displays current percentage (parsed from `Notes`). Color-coded. Shows "(Batt: N/A)" if unavailable.
    * **Time Since Last GPS:** Displays relative time since `GPS Timestamp`.
    * **Stale GPS Warning (Travelling):** If "Travelling" and GPS > 30 mins old, GPS age text highlighted.
* **Database Size Warning:** Dismissible banner if >= 2500 rows.
* **One-Click Location:** Link on alert cards to Google Maps (GPS priority). Opens in new tab.
* **Direct "Call Worker" Button:** "Call" button on each card using `tel:` link.
* **Declare Safe (Manual Resolve):** Button on alert cards. Requires typing worker's name. Sends `POST` to update status to `MONITOR_CLEARED_ALERT`. Logs action.
* **Connection Status:** Indicator dot and last update time.

### 4.2. Technical Specifications

* **Platform:** Single-page HTML app. Primarily desktop, responsive grid.
* **Styling:** Tailwind CSS (CDN). Dark theme. Includes styles for battery colors, stale GPS (`gps-stale`), pre-alert overdue (`border-pre-alert`), log panel. Logo in header.
* **JavaScript:** Vanilla JS, ES6+. Logic for session log array, Notification API, parsing battery %, calculating GPS/Time differences, calculating pre-alert status.
* **Audio:** `Tone.js` (CDN). Error handling for audio context.
* **Storage:** None needed (session log in-memory).
* **Backend Communication:** **JSONP** for `GET`. `fetch`, `POST`, `mode: 'no-cors'` for "Manual Resolve". Increased JSONP timeout (20s).
* **Error Handling:** Custom modals. Logs connection errors/timeouts, updates status indicator. Handles invalid GPS/Date timestamps gracefully. Displays notification permission status.

## 5. The Google Apps Script Backend

Central controller deployed from Google Sheet.

### 5.1. Core Functions

* **`doGet(e)`:** Handles `GET`. Validates `token`. Returns sheet data as JSONP (Dates as ISO strings).
* **`doPost(e)`:** Handles `POST`. Uses `LockService`. Finds row by Name & Arrival Time to update, or appends. Saves `Notes`. Parses numeric battery level from `Notes` into `Battery Level` column. Handles date parsing errors. Fetches full row for immediate alerts email.
* **`checkOverdueWorkers()`:** Runs via trigger (5 mins). Uses `LockService`. Ignores resolved statuses. Calculates overdue minutes. Sends escalating alerts (`EMAIL_1`...`ESCALATION`) via `sendAlertEmail`. Updates sheet status. Handles invalid dates.
* **`sendAlertEmail(workerData, alertType)`:** Sends HTML email via `MailApp`. Selects recipient. Formats subject/body. Includes details & Maps link. Handles missing data.

### 5.2. Spreadsheet Structure

* Headers must match **exactly**: `Date,Worker Name,Worker Phone Number,Emergency Contact Name,Emergency Contact Number,Emergency Contact Email,Escalation Contact Name,Escalation Contact Number,Escalation Contact Email,Location Name,Location Address,Arrival Time,Anticipated Departure Time,Actual Departure Time,Alarm Status,Notes,Last Known GPS,GPS Timestamp,Battery Level`
* **New Column Added:** `Battery Level` (Column S).

### 5.3. Deployment & Configuration

* **Secret Key:** `SECRET_KEY` variable, value injected by Deployment Tool.
* **Deployment:** As **Web app**. Execute as: **Me**. Access: **Anyone**.
* **Trigger:** **Time-driven trigger** for `checkOverdueWorkers` (5 mins).

## 6. The Deployment Tool (`setup.html`)

Single HTML file hosted online for administrator setup.

### 6.1. Functionality

* **Wizard Interface:** Multi-step UI.
* **Template Fetching:** Uses `fetch` to get templates from `./templates/` on load. Handles errors. `templatesLoaded` flag.
* **Step-by-Step Guidance:**
    1.  Welcome.
    2.  Sheet setup: "Copy Headers" copies **tab-delimited** headers (including `Battery Level`). Instructions simplified for direct paste.
    3.  Script deployment: Field to **Create Secret Key**. "Copy Script" injects key. Instructions reordered. Script preview. Bold key terms.
    4.  App config: Input for **Web App URL** (validation). Read-only `type="text"` **Secret Key**. Inputs for **First Alert Time**, **Enable Check-ins**, **Check-in Interval** (conditional), **Logo URL**. "Test Connection" button uses JSONP. Bold key terms.
    5.  Download: "Generate & Download .zip" button.
* **Zip Generation:**
    * Uses `JSZip` (CDN). Checks `templatesLoaded`. Validates inputs.
    * Injects config values into placeholders (`%%...%%`, `YOUR_SECRET_KEY_HERE`) in worker app, monitor app, manifest, script templates. Injects `%%LOGO_URL%%` into Monitor App template.
    * Removes setup page/logic from monitor app using comment placeholders.
    * Packages files (`LoneWorkerApp/`, `MonitoringDashboardApp/`, `Documentation/`, `01-...txt`) into `.zip`. Includes `Deployment_Admin_Guide.md`.
    * Triggers download. Displays status. Handles errors.

### 6.2. Technical Specifications

* **Platform:** Single HTML file, hosted via HTTPS.
* **Styling:** Tailwind CSS (CDN). Bold key terms.
* **JavaScript:** Vanilla JS. `fetch`, `navigator.clipboard`, JSONP test, `JSZip`. `type="button"`. Robust template loading & URL construction. Error handling.
* **Dependencies:** `JSZip` (CDN). Assumes `./templates/` folder.
* **Error Handling:** Clear alerts/console logs for errors. Disables buttons if setup fails.

### 6.3. Required Template Files (in `./templates/`)

* `apps_script_template.txt` (`YOUR_SECRET_KEY_HERE`)
* `worker_app_template.txt` (`%%...%%` placeholders, icon URL)
* `monitor_app_template.txt` (`<!-- ... -->` placeholders, `%%LOGO_URL%%`)
* `manifest.json` (icon URL)
* `sw.js`
* `Deployment_Admin_Guide.md`
* `Installer Creation Guide.md`
* `Promotional Blurb.md`
