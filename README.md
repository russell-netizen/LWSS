# Project Specification: Lone Worker Safety System v2.12

**Version:** 2.12

## 1. Overview
This document specifies a complete, free, self-hosted Lone Worker Safety System. It includes a worker PWA, a monitor dashboard, a Google Apps Script backend, and a web-based deployment tool. The tool generates customized, ready-to-host versions of the apps. Full documentation is provided separately.

## 2. System Architecture
* The Lone Worker App (PWA)
* The Monitoring App
* The Google Apps Script Backend
* The Deployment Tool (`setup.html`)
* Documentation (Separate PDF)

## 3. The Lone Worker App (PWA)

### 3.1. Core Features
* **Location Management:** Add, edit, delete locations. Default "Travelling". GPS lookup.
* **Visit Timer:** Select location/duration. Hold **"Start"** button (**1.5s**) to begin. Sends `ON SITE` status.
* **Locked Screen:** Shows countdown (+ overdue), location, departure. Buttons: Extend, Depart, SOS. Shows "ALERT ACTIVE" section when needed.
* **Panic Button (SOS):** Triple-tap sends `EMERGENCY - PANIC BUTTON` alert + GPS attempt.
* **PIN & Duress PIN:**
    * Users define a 4-digit Normal PIN and a 4-digit Duress PIN.
    * "Extend 10 Mins" and "I AM SAFE" buttons require PIN.
    * **Duress Signal:** Entering the **Duress PIN** instead of the normal PIN when prompted (for "Extend" OR "I AM SAFE") makes the app *appear* to perform the action normally but silently sends a high-priority `DURESS_CODE_ACTIVATED` alert to the backend.
* **"Are You OK?" Check-ins (Optional & Configurable):**
    * Enabled via `CHECKIN_ENABLED` flag. Interval set via `CHECKIN_INTERVAL` (minutes). Both injected by Deployment Tool.
    * Modal prompt with 2-min countdown.
    * **Escalating Alert:** Sound volume (`Tone.js`) increases from -20dB to 0dB in last minute. Sound/vibration frequency increases in last minute.
    * **Confirmation:** Tapping "I am OK" triggers a distinct, short vibration pattern (e.g., `[50, 50, 50]`), resets the timer/volume, and dismisses the modal.
    * Missing the 2-minute window triggers a `MISSED_CHECKIN` alert.
* **GPS Tracking:** Attempts GPS on alerts. Every 15 mins if "Travelling". `navigator.geolocation`, high accuracy, timeout. User warned about foreground requirement.
* **Low Battery Warning & Reporting:**
    * Checks battery (`navigator.getBattery`) at visit start and periodically (e.g., every 15 mins).
    * Sends initial battery level in `Notes`.
    * If level drops < 15% (and not charging), sends a `LOW BATTERY WARNING: XX%` note **once per visit**.
    * Sends regular battery level updates periodically if low warning hasn't been sent.
* **Departure:** Hold "DEPART" button (**1.5s**). Sends `DEPARTED` status. Ends visit.
* **Settings:** Inputs for Name, Phone, Contacts (Emergency/Escalation), PINs. URL field pre-filled (`%%GOOGLE_SHEET_URL%%`) and disabled after save. Uses `localStorage`.
* **Info/Help Modal:** Instructions on install, setup, use, alerts, duress, GPS, privacy.
* **Start Timer Reminder:**
    * If the user is on the main screen and has not started a visit for 5 minutes, a non-intrusive banner appears.
    * Timer resets on navigation or visit start.

### 3.2. Technical Specifications
* **Platform:** Single-page HTML PWA.
* **Styling:** Tailwind CSS (CDN). Responsive dark theme.
* **JavaScript:** Vanilla JS, ES6+.
* **Audio:** `Tone.js` (CDN).
* **Vibration:** `navigator.vibrate` for check-in confirmation.
* **Storage:** `localStorage`.
* **Backend Communication:** `fetch` API, `POST`, `mode: 'no-cors'`.
* **PWA Files:** `manifest.json` (configurable logo), `sw.js` (basic).
* **Error Handling:** Custom modal dialogs.
* **Long Press Buttons:** Actions require press-and-hold with visual feedback.
    * **1.5 seconds (1500ms):** "Start", "DEPART" (actions *without* PIN).
    * **1 second (1000ms):** "Extend 10 Mins", "I AM SAFE" (actions *requiring* PIN).

## 4. The Monitoring App

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
* **Platform:** Single HTML, Tailwind, Vanilla JS
* **Styling:** Tailwind CSS (CDN). Dark theme. Includes styles for battery colors, stale GPS (`gps-stale`), pre-alert overdue (`border-pre-alert`), log panel. Logo in header.
* **JavaScript:** Vanilla JS, ES6+. Logic for session log array, Notification API, parsing battery %, calculating GPS/Time differences, calculating pre-alert status.
* **Audio:** `Tone.js` (CDN). Error handling for audio context.
* **Storage:** None needed (session log in-memory).
* **Backend Communication:** **JSONP** for `GET`. `fetch`, `POST`, `mode: 'no-cors'` for "Manual Resolve". Increased JSONP timeout (20s).
* **Error Handling:** Custom modals. Logs connection errors/timeouts, updates status indicator. Handles invalid GPS/Date timestamps gracefully. Displays notification permission status.

## 5. The Google Apps Script Backend

### 5.1. Core Functions
* **`doGet(e)`:** Validate token, return data as JSONP (ISO Dates).
* **`doPost(e)`:** Save data, parse/save Battery Level from Notes.
* **`checkOverdueWorkers()`:** Triggered check, send escalating emails.
* **`sendAlertEmail()`:** Format and send emails.

### 5.2. Spreadsheet Structure
* Headers (A-S): `Date`, `Worker Name`, `Worker Phone Number`, `Emergency Contact Name`, `Emergency Contact Number`, `Emergency Contact Email`, `Escalation Contact Name`, `Escalation Contact Number`, `Escalation Contact Email`, `Location Name`, `Location Address`, `Arrival Time`, `Anticipated Departure Time`, `Actual Departure Time`, `Alarm Status`, `Notes`, `Last Known GPS`, `GPS Timestamp`, `Battery Level`

### 5.3. Deployment & Configuration
* `SECRET_KEY` (injected).
* Deploy as Web App, Execute=Me, Access=Anyone.
* 5-min Time-driven trigger for `checkOverdueWorkers`.

## 6. The Deployment Tool (`setup.html`)

### 6.1. Functionality
* **Wizard Interface:** Multi-step UI (6 steps).
* **Template Fetching:** Uses `fetch` to get *code* templates from `./templates/` on load.
* **Step-by-Step Guidance:**
    1.  Welcome.
    2.  Sheet setup: "Copy Headers" copies **tab-delimited** headers (including `Battery Level`).
    3.  Script deployment: Field to **Create Secret Key**. "Copy Script" injects key.
    4.  App config: Input for **Web App URL** (validation). Read-only `type="text"` **Secret Key**. Inputs for **First Alert Time**, **Enable Check-ins**, **Check-in Interval** (conditional), **Logo URL**. "Test Connection" button.
    5.  **Download Apps:** "Generate & Download App Package (.zip)" button.
    6.  **Documentation:** Link to download a separate, comprehensive documentation PDF.
* **Zip Generation:**
    * Uses `JSZip` (CDN).
    * Injects config values into placeholders in code templates.
    * Removes setup page/logic from monitor app.
    * Packages *only* `LoneWorkerApp/`, `MonitoringDashboardApp/`, and `01-...txt` into `.zip`.
    * **Documentation is NOT included in the zip.**

### 6.2. Technical Specifications
* **Platform:** Single HTML file, hosted via HTTPS.
* **Styling:** Tailwind CSS (CDN). Bold key terms.
* **JavaScript:** Vanilla JS. `fetch`, `navigator.clipboard`, JSONP test, `JSZip`. `type="button"`.
* **Dependencies:** `JSZip` (CDN). Assumes `./templates/` folder.
* **Error Handling:** Clear alerts/console logs for errors.

### 6.3. Required Template Files (in `./templates/`)
* `apps_script_template.txt` (`YOUR_SECRET_KEY_HERE`)
* `worker_app_template.txt` (`%%...%%` placeholders, icon URL)
* `monitor_app_template.txt` (`<!-- ... -->` placeholders, `%%LOGO_URL%%`)
* `manifest.json` (icon URL)
* `sw.js`
* `Deployment_Admin_Guide.md` (Source text for PDF, not fetched by `setup.html`)
* `Installer Creation Guide.md` (Source text for PDF, not fetched by `setup.html`)
* `Promotional Blurb.md` (Source text for PDF, not fetched by `setup.html`)

## 7. Documentation (Separate PDF)
* A comprehensive PDF document (e.g., `LWS_Documentation.pdf`) must be created by the Setup Tool Administrator.
* This PDF must be hosted online (e.g., in the `templates` folder or root of the host repository).
* The download link `href` in Step 6 of `setup.html` must point to this PDF's absolute URL.
* The PDF contains the formatted content of the Deployment Admin Guide, Installer Guide, and Promotional Blurb.
