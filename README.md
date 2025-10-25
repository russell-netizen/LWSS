# Project Specification: Lone Worker Safety System v2.13

**Version:** 2.13 

## 1. Overview
This document specifies a complete, free, self-hosted Lone Worker Safety System. It includes a worker PWA, a monitor dashboard, a Google Apps Script backend, and a web-based deployment tool. The tool generates customized, ready-to-host versions of the apps. Full documentation is provided separately.

## 2. System Architecture
* The Lone Worker App (PWA)
* The Monitoring App
* The Google Apps Script Backend
* The Deployment Tool (`setup.html`)
* Documentation (Separate PDF)

## 3. The Lone Worker App (PWA)

*(No changes from v2.12)*

### 3.1. Core Features
* Location Management
* Visit Timer ("Start" button, 1.5s hold)
* Locked Screen
* Panic Button (SOS - Triple Tap)
* PIN & Duress PIN (Duress signal on Extend and I Am Safe)
* "Are You OK?" Check-ins (Optional, Configurable Interval, Escalating Alert, Vibration Confirmation)
* GPS Tracking (Foreground dependent)
* Low Battery Warning & Reporting
* Departure (1.5s hold)
* Settings (URL Pre-filled)
* Info/Help Modal
* Start Timer Reminder (5 min)

### 3.2. Technical Specifications
* PWA, Tailwind, Vanilla JS
* `Tone.js` (Audio)
* `navigator.vibrate`
* `localStorage`
* `fetch` POST (`no-cors`)
* `manifest.json`, `sw.js`
* Custom Modals
* Long Press Buttons (1.5s no-PIN, 1s PIN)

## 4. The Monitoring App

### 4.1. Core Features

* **Secure Login (Reverted):**
    * The Monitor App **no longer** has credentials hardcoded.
    * On first launch, it displays a **setup page** (`setupPage`) prompting the user for the **Google Apps Script URL** and the **Secret Key**.
    * On successful connection (tested via JSONP `fetchData` call), these values are saved to the browser's `localStorage`.
    * On subsequent visits, the app reads from `localStorage` and loads directly to the dashboard.
* **Header Display:** Includes Logo, title, connection status, last update time, notification status/button. **Includes a "Reset" button** to clear `localStorage` and return to the setup page.
* **Session Alert History Log:** Collapsible, in-memory log of session events.
* **Live Dashboard:** Fetches data (JSONP, 15s interval). Displays "active" workers in a grid.
* **Status Display & Sorting:** Cards sorted by priority, then name.
* **Prioritized Alerts & Visual Cues:** Color-coded cards (Purple/Red/Orange/Gray).
* **Enhanced Highlighting (Pre-Alert Overdue):** Amber border/badge for `ON SITE` workers past their departure time.
* **Audible Alarms:** `Tone.js`. Loud repeating alarm for new critical alerts, requires acknowledgment. Softer chime for status updates.
* **Browser/Desktop Notifications:** `Notification` API for new critical alerts.
* **Worker Details Displayed on Card:**
    * **Battery Level:** Parsed from `Notes`, color-coded.
    * **Time Since Last GPS:** Relative time, stale warning for "Travelling".
* **Database Size Warning:** Dismissible banner if >= 2500 rows.
* **One-Click Location:** Google Maps link.
* **Direct "Call Worker" Button:** `tel:` link.
* **Declare Safe (Manual Resolve):** Button, requires name confirmation, sends `POST`.
* **Connection Status:** Indicator dot and last update time.

### 4.2. Technical Specifications
* **Platform:** Single-page HTML app.
* **Styling:** Tailwind CSS (CDN).
* **JavaScript:** Vanilla JS, ES6+.
* **Audio:** `Tone.js` (CDN).
* **Storage:** **Uses `localStorage`** to store the `sheetUrl` and `secretKey` after first-time setup.
* **Backend Communication:** **JSONP** for `GET`. `fetch`, `POST`, `mode: 'no-cors'` for "Manual Resolve".
* **Error Handling:** Custom modals.

## 5. The Google Apps Script Backend

*(No changes from v2.7)*

## 6. The Deployment Tool (`setup.html`)

### 6.1. Functionality
* **Wizard Interface:** Multi-step UI (6 steps).
* **Template Fetching:** Uses `fetch` to get *code* templates from `./templates/` on load.
* **Step-by-Step Guidance:**
    1.  Welcome (includes PDF download link).
    2.  Sheet setup: "Copy Headers" (tab-delimited, includes `Battery Level`).
    3.  Script deployment: "Create Secret Key" field, "Copy Script" button.
    4.  App config: Inputs for URL, Key (read-only), Alert Time, Check-ins, Interval, Logo URL. "Test Connection" button.
    5.  Download Apps: "Generate & Download App Package (.zip)" button.
    6.  Finish: Final PDF download link.
* **Zip Generation:**
    * Uses `JSZip` (CDN).
    * Injects config values into placeholders in `worker_app_template.txt`, `apps_script_template.txt`, and `manifest.json`.
    * **Crucially, it NO LONGER modifies `monitor_app_template.txt` to inject credentials.** It only injects the `%%LOGO_URL%%` placeholder.
    * The `monitor_app_template.txt` file is packaged *as-is* with its `setupPage` and `localStorage` logic intact.
    * Packages `LoneWorkerApp/`, `MonitoringDashboardApp/`, and `01-...txt` into `.zip`.
    * Documentation is NOT included in the zip.

### 6.2. Technical Specifications
* **Platform:** Single HTML file, hosted via HTTPS.
* **Styling:** Tailwind CSS (CDN).
* **JavaScript:** Vanilla JS. `fetch`, `navigator.clipboard`, JSONP test, `JSZip`.
* **Dependencies:** `JSZip` (CDN).
* **Error Handling:** Clear alerts/console logs for errors.

### 6.3. Required Template Files (in `./templates/`)
* `apps_script_template.txt` (`YOUR_SECRET_KEY_HERE`)
* `worker_app_template.txt` (`%%...%%` placeholders)
* `monitor_app_template.txt` (`<!-- ... -->` placeholders, `%%LOGO_URL%%`)
* `manifest.json` (icon URL)
* `sw.js`
* `Deployment_Admin_Guide.md` (Source text for PDF, not fetched by `setup.html`)
* `Installer Creation Guide.md` (Source text for PDF, not fetched by `setup.html`)
* `Promotional Blurb.md` (Source text for PDF, not fetched by `setup.html`)

## 7. Documentation (Separate PDF)
* A comprehensive PDF document must be created and hosted by the Setup Tool Admin.
* The `setup.html` tool links to this PDF.
* The documentation (based on `Deployment_Admin_Guide.md`) **must be updated** to instruct the Deployment Admin that their Monitor will need to enter the Web App URL and Secret Key on their first use of the Monitor App.
