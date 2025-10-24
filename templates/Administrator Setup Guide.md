# Administrator Setup Guide: Lone Worker Safety System (Detailed)

Welcome! This guide provides detailed, step-by-step instructions for setting up your free, self-hosted Lone Worker Safety System. We'll use a special web-based setup tool to make this easier.

**Target Audience:** This guide assumes you have average computer skills (using web browsers, copy/pasting text, managing files) but may not have experience with code hosting platforms like GitHub.

**Estimated Time:** 30-45 minutes.

---

## **Part 1: Making the Setup Tool Accessible Online**

The `setup.html` file is a webpage that helps you configure the system. Because modern web browsers have security features that limit what local HTML files can do (like copying text reliably or accessing other files), we first need to put the setup tool and its templates online. The easiest and free way to do this is using GitHub Pages.

### **1.1 Create a Free GitHub Account (If you don't have one)**

* Go to [https://github.com/](https://github.com/).
* Click **"Sign up"**.
* Follow the on-screen instructions to create a free account (choose a username, enter your email, create a password). You do not need any paid options.
* Verify your email address if prompted.

### **1.2 Create a New Repository**

A "repository" is like an online folder for your project files.

* Once logged in to GitHub, click the **"+"** icon in the top-right corner, then select **"New repository"**.
* **Repository name:** Give it a simple name, like `lws-deploy-tool`.
* **Description (Optional):** You can add something like "Deployment tool for Lone Worker Safety system".
* Select **"Public"**. This is required for GitHub Pages to work on a free account.
* **Do Not** initialize with a README, .gitignore, or license for simplicity right now.
* Click **"Create repository"**.

### **1.3 Upload the Setup Files**

You need to upload the `setup.html` file and the `templates` folder (with all its contents) to this new repository.

* On your new repository page, click the **"Add file"** button, then select **"Upload files"**.
* **Drag and Drop:** Find the `setup.html` file on your computer and drag it onto the GitHub upload area.
* **Upload the Templates Folder:** Find the **entire `templates` folder** (which contains the `.txt`, `.md`, `.json`, and `.js` files) on your computer. Drag this **entire folder** onto the same GitHub upload area.
    * **Important:** Make sure the folder is named `templates` (all lowercase). GitHub should show it uploading the folder and the files inside it.
* **Commit changes:** Once the files/folder are listed, scroll down. You can leave the default commit message ("Add files via upload"). Click the **"Commit changes"** button.

### **1.4 Enable GitHub Pages**

This turns your repository into a live website.

* In your repository, click the **"Settings"** tab (usually near the top right).
* On the left sidebar, click **"Pages"**.
* Under the "Build and deployment" section, for the "Source", select **"Deploy from a branch"**.
* Under "Branch", ensure your main branch is selected (usually `main` or `master`). Leave the folder as `/root`. Click **"Save"**.
* **Wait:** The page will reload. At the top, it might say "Your site is live at..." or it might take a minute or two. Refresh the page after a minute if needed. GitHub will provide you with the **public URL** for your site (e.g., `https://your-username.github.io/lws-deploy-tool/`).

### **1.5 Launch the Setup Tool**

* Go to the public URL GitHub gave you in the previous step, but add `setup.html` to the end.
* **Example:** `https://your-username.github.io/lws-deploy-tool/setup.html`
* Bookmark this page – this is your setup tool!

You should now see the "Welcome to the LWS Setup Wizard" screen.

---

## **Part 2: Use the Setup Wizard**

Now, follow the steps presented in the setup tool webpage.

### **Step 1: Welcome**

* Read the introduction. It explains that the tool will fetch the templates (which you just uploaded) and guide you.
* Click the **"Start Setup"** button.

### **Step 2: Sheet Setup**

This step prepares the Google Sheet that will act as your database.

* **Create Sheet:** Click the **`sheets.new`** link. It will open a new, blank Google Sheet in another browser tab. Give this sheet a recognizable name (e.g., "Lone Worker Data").
* **Copy Headers:** Go back to the setup tool tab. Click the **"Copy"** button next to the list of headers. This copies the list to your clipboard, separated by invisible tab characters.
* **Paste Headers:** Go back to your blank Google Sheet tab. Click **once** in the very first cell (cell **A1**). Press `Ctrl+V` (or `Cmd+V` on Mac) to paste. Because the headers were copied with tabs, Google Sheets should automatically place each header into its own cell across the first row (A1, B1, C1, ..., R1). Verify this happened correctly.
* **Proceed:** Go back to the setup tool tab and click the **"Next Step"** button.

### **Step 3: Deploy Script**

This is the most technical part, where you install the "brain" of the system into your Google Sheet. Follow these instructions carefully.

* **Open Script Editor:** In your Google Sheet tab, click the **`Extensions`** menu, then select **`Apps Script`**. A new browser tab will open with the script editor.
* **Clear Existing Code:** Select any code already present in the `Code.gs` window (e.g., `function myFunction() {}`) and delete it.
* **Create Secret Key:** Go back to the setup tool tab (Step 3). Type a strong, unique password into the **"Create a Secret Key"** box. Treat this like any important password. You'll need it again shortly.
* **Copy Customized Script:** Click the **"Copy Script"** button in the setup tool. This copies the entire backend code *with your Secret Key already inserted* into the correct place.
* **Paste Script:** Go back to the Apps Script editor tab and paste (`Ctrl+V` or `Cmd+V`) the copied code into the empty editor window.
* **Save Script:** Click the **Save** icon (looks like a floppy disk). Give the script project a name when prompted (e.g., "LWS Backend Script").
* **Deploy:**
    * Click the blue **`Deploy`** button (top right).
    * Select **`New deployment`**.
    * A dialog box will appear. Click the **gear icon** next to "Select type" and choose **`Web app`**.
    * Fill in the details:
        * Description (optional): e.g., "LWS Backend v1"
        * Execute as: **`Me`** (Your Google Account)
        * Who has access: **`Anyone`** (This is crucial!)
    * Click **`Deploy`**.
* **Authorize:** Google needs your permission to let the script run.
    * Click **`Authorize access`**.
    * Choose the Google account you used to create the sheet.
    * A "Google hasn’t verified this app" screen will appear. This is normal because it's your own script. Click **`Advanced`**.
    * Click **"Go to [Your Script Name] (unsafe)"**.
    * Review the permissions (it will need to manage Spreadsheets and send emails) and click **`Allow`**.
* **Copy Web App URL:** After authorization, a box appears showing the deployment details. **Crucially**, copy the **`Web app URL`** (the one ending in `/exec`). Save this URL somewhere safe temporarily (like Notepad).
* Click **`Done`**.
* **Set Up Trigger:** The script needs to run automatically to check for overdue workers.
    * In the Apps Script editor, click the **Triggers** icon (looks like an alarm clock) on the left sidebar.
    * Click the **`Add Trigger`** button (bottom right).
    * Configure the trigger exactly as follows:
        * Choose which function to run: **`checkOverdueWorkers`**
        * Choose which deployment should run: **`Head`**
        * Select event source: **`Time-driven`**
        * Select type of time based trigger: **`Minutes timer`**
        * Select minute interval: **`Every 5 minutes`**
        * Error notification settings: Choose your preference (e.g., "Notify me immediately").
    * Click **`Save`**. You might need to authorize the script again; if so, repeat the authorization steps.
* **Proceed:** Go back to the setup tool tab and click the **"Next Step"** button.

### **Step 4: Configure Apps**

Here, you provide the details the setup tool needs to build the final Worker and Monitor apps.

* **Web App URL:** Paste the **`Web app URL`** you copied from the Apps Script deployment step into this field. The tool will check if the format looks correct.
* **Secret Key:** This field should be automatically filled with the key you created in Step 3. You can see it here to double-check, but you cannot change it.
* **First Overdue Alert:** Set how many minutes after the anticipated departure time the first alert should be triggered. `15` is standard.
* **Enable Check-ins:** Tick this box if you want workers to be prompted periodically ("Are you OK?").
* **Check-in Interval:** If you enabled check-ins, set how often (in minutes) workers should be prompted. `30` is a common starting point.
* **Company Logo URL (Optional):** If you have a company logo hosted online (must be a direct link to a `.png` or `.jpg` file), you can paste the URL here. It will be used as the icon for the Worker App when installed on a phone. If left blank, a default icon is used.
* **Test Connection:** Click the **"Test Connection & Proceed"** button. The tool will try to contact your deployed Apps Script using the URL and Key.
    * If it shows **"Success! Connection verified."**, you're good to go! The tool will automatically move to the next step.
    * If it shows an error (like "Access Denied" or "Could not connect"), double-check the Web App URL you pasted and the Secret Key you created in Step 3. Ensure your Apps Script is deployed with access set to "Anyone".

### **Step 5: Download**

* If the connection test was successful, you'll reach the final step.
* Click the **"Generate & Download .zip Package"** button.
* Your browser will download a file named `LWS_Deployment_Package.zip`, likely to your computer's "Downloads" folder. This file contains your *finished, customized* apps.

---

## **Part 3: Host Your Final Apps**

You're finished with the setup tool! The `.zip` file contains the actual, customized apps ready for your team. Now you need to put these online.

### **3.1 Unzip the File**

* Find the `LWS_Deployment_Package.zip` file you downloaded.
* Unzip it (usually by right-clicking and selecting "Extract All..." or similar).
* Inside the extracted folder, you'll see:
    * `LoneWorkerApp` folder (contains `index.html`, `manifest.json`, `sw.js`)
    * `MonitoringDashboardApp` folder (contains `index.html`)
    * `Documentation` folder (contains copies of these guides)
    * `01-PASTE_THIS_INTO_APPS_SCRIPT.txt` (a backup copy of the script code you deployed)

### **3.2 Host the Apps (Using Netlify Drop)**

Netlify Drop is a very simple drag-and-drop web hosting service, perfect for this.

* **Host Worker App:**
    * Go to [https://app.netlify.com/drop](https://app.netlify.com/drop) in your browser.
    * Drag the **entire `LoneWorkerApp` folder** (not just the files inside it) from your computer onto the Netlify Drop page where it says "Drag and drop your site folder here".
    * Netlify will upload it and generate a unique public website address (URL), like `https://random-words-12345.netlify.app`.
    * **Copy this URL** and save it. This is your **Worker App URL**.
* **Host Monitor App:**
    * Go back to [https://app.netlify.com/drop](https://app.netlify.com/drop) again (you might need to refresh the page or open it anew).
    * Drag the **entire `MonitoringDashboardApp` folder** onto the Netlify Drop page.
    * Netlify will generate a *different* unique public URL.
    * **Copy this URL** and save it. This is your **Monitoring App URL**.

*(Note: If you already have another web hosting provider, you can simply upload the contents of these two folders there instead of using Netlify Drop.)*

---

## **Part 4: Distribute to Your Team**

Now you provide the links to your staff.

### **For Your Safety Monitor:**

* Send them the **Monitoring App URL** you saved in Step 3.2.
* When they open this link, the monitoring dashboard should load immediately and start checking for active workers. No login or setup is needed for them.

### **For Your Lone Workers:**

* Send them the **Worker App URL** you saved in Step 3.2.
* **Instructions for Workers:**
    1.  Open the link on their smartphone.
    2.  Follow the browser's prompts to **"Add to Home Screen"** (iOS/Safari) or **"Install app" / "Add to Home Screen"** (Android/Chrome). This is crucial for the app to work reliably.
    3.  Open the newly installed app from their phone's home screen.
    4.  Tap the **Settings icon** (gear, top right).
    5.  Confirm the "Google Sheet Web App URL" is already filled in.
    6.  Fill in **all other fields**: their Name, Phone, Emergency Contact details, Escalation Contact details, a 4-digit normal PIN, and a different 4-digit Duress PIN.
    7.  Tap **"Save Settings"**.

---

**Setup Complete!** Your Lone Worker Safety system is now fully configured and operational. Ensure your workers understand how to use the app (refer them to the Info section within their app) and your monitors understand the dashboard and alert procedures.
