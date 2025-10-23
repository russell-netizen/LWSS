# Administrator Setup Guide: Lone Worker Safety System

Welcome! This guide will walk you through the entire process of setting up your free, self-hosted Lone Worker Safety System.

This process uses a setup tool to simplify configuration.

## **Part 1: Host the Deployment Tool**

Before you can set up the system, you must host this setup package on a web server. The easiest free method is using GitHub Pages.

1.  **Create a GitHub Repository:**
    * Log in to GitHub and create a new, **public** repository (e.g., "lws-deploy").
2.  **Upload All Files:**
    * Upload all files from this package into your new repository.
    * **Crucially:** You must create a `/templates` folder (lowercase) and put the 8 template files inside it. The `setup.html` file should be in the root.
3.  **Enable GitHub Pages:**
    * In your repository, go to `Settings` > `Pages`.
    * Under "Branch", select your main branch (e.g., `main`) and set the folder to `/root`. Click **Save**.
    * GitHub will give you a public URL for your site (e.g., `https://your-username.github.io/lws-deploy/`). It may take a minute to become active.
4.  **Launch the Tool:**
    * Go to the `setup.html` page at your new URL.
    * **Example:** `https://your-username.github.io/lws-deploy/setup.html`

You will now see the setup wizard.

---

## **Part 2: Use the Setup Wizard**

Follow the steps in the setup tool you just launched.

### **Step 1: Welcome**
* Read the introduction and click "Start Setup".

### **Step 2: Sheet Setup**
* Click the link to open a new Google Sheet (`sheets.new`).
* Click the **"Copy Headers"** button in the wizard.
* In your sheet, click **once** on cell **A1**.
* Paste the copied text.
* With A1 still selected, go to **Data > Split text to columns**. Choose **Comma** if prompted. The headers should now be split across row 1.

### **Step 3: Deploy Script**
* This is the most important part.
* In your Google Sheet, go to `Extensions` > `Apps Script`.
* Delete any existing code in `Code.gs`.
* **Create your Secret Key:** Enter a strong, private password into the "Create a Secret Key" box in the wizard.
* Click the **"Copy Script"** button in the wizard. This copies the script code with your key already inserted.
* Paste the copied code into the empty `Code.gs` editor.
* Follow the on-screen instructions to **Deploy** the script as a **Web app** (Execute as: Me, Who has access: Anyone).
* **Authorize** the script (click "Advanced", "Go to...", "Allow").
* After it's deployed, **copy the "Web app URL"**.
* Finally, follow the instructions to set up the **5-minute trigger** for the `checkOverdueWorkers` function.

### **Step 4: Configure Apps**
* Paste the **Web app URL** (from the previous step) into the first box.
* The **Secret Key** field will be auto-filled for you.
* Configure the other options as you wish.
* Click **"Test Connection & Proceed"**. The tool will check if your URL and Key are correct. If it succeeds, you'll move to the final step.

### **Step 5: Download**
* Click the **"Generate & Download .zip Package"** button.
* Your browser will download a file named `LWS_Deployment_Package.zip`. This file contains your *finished, customized* apps.

---

## **Part 3: Host Your Final Apps**

You are now finished with the setup tool. Your final step is to host the apps from the `.zip` file you just downloaded.

1.  **Unzip the File:**
    * Unzip the `LWS_Deployment_Package.zip` file.
    * Inside, you will find two folders: `LoneWorkerApp` and `MonitoringDashboardApp`. You will also find documentation and the Apps Script code for your records.

2.  **Host the Apps:**
    * You must host these two folders publicly. We recommend using a free, simple hosting service like **Netlify Drop**.
    * Go to [app.netlify.com/drop](https://app.netlify.com/drop).
    * Drag and drop the **`LoneWorkerApp` folder** onto the page. Netlify will give you a unique, public URL. This is your **Worker App URL**.
    * Go back to [app.netlify.com/drop](https://app.netlify.com/drop) again.
    * Drag and drop the **`MonitoringDashboardApp` folder** onto the page. Netlify will give you a *different* public URL. This is your **Monitoring App URL**.

---

## **Part 4: Distribute to Your Team**

### **For Your Safety Monitor:**
* Send them the **Monitoring App URL** (from Step 3.2).
* When they open it, it will work immediately. No login is required.

### **For Your Lone Workers:**
* Send them the **Worker App URL** (from Step 3.2).
* When they open it, they must:
    * Follow the on-screen instructions to **"Add to Home Screen"** or **"Install App"**. This is essential.
    * Open the app from their home screen.
    * Go to **Settings** (gear icon). The "Google Sheet Web App URL" will be pre-filled.
    * Fill in **all other fields**: their name, phone, emergency contacts, and a 4-digit PIN and Duress PIN.
    * Click **Save Settings**.

**Your system is now fully operational.**
