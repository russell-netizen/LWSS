# Deployment Administrator Guide: Lone Worker Safety System

Welcome! This guide explains how to use the online Setup Tool to create your customized Lone Worker Safety apps and deploy them using GitHub Pages.

**Goal:** To generate and host your own private versions of the Worker App and Monitoring Dashboard.

**Estimated Time:** 20-30 minutes.

---

## **Part 1: Use the Online Setup Tool**

Your system administrator has provided a link to the LWS Setup Tool website.

1. **Open the Setup Tool Link** in your web browser. You should see the "Welcome to the LWS Setup Wizard".

2. Follow the steps presented in the tool:

   ### **Step 1: Welcome**

   * Read the introduction.

   * Click **"Start Setup"**.

   ### **Step 2: Sheet Setup**

   * **Create Sheet:** Click the **`sheets.new`** link to open a blank Google Sheet. Give it a name (e.g., "My Team LWS Data").

   * **Copy Headers:** Click the **"Copy"** button in the setup tool.

   * **Paste Headers:** In your Google Sheet, click cell **A1** and **Paste**. The headers (Date, Worker Name, ..., Battery Level) should fill row 1 automatically.

   * Click **"Next Step"** in the setup tool.

   ### **Step 3: Deploy Script**

   * **Open Script Editor:** In your Google Sheet, click **`Extensions` > `Apps Script`**. Delete any existing code in `Code.gs`.

   * **Create Secret Key:** In the setup tool, type a **strong, private password** into the "Create a Secret Key" box. **Remember this key!**

   * **Copy Script:** Click the **"Copy Script"** button in the setup tool.

   * **Paste Script:** In the Apps Script editor, paste the copied code.

   * **Deploy:** Click **`Deploy` > `New deployment`**. Select type **`Web app`**, Execute as **`Me`**, Who has access **`Anyone`**. Click **`Deploy`**.

   * **Authorize:** Allow the script access to your Google Account (click "Advanced", "Go to...", "Allow").

   * **Copy URL:** After deployment, **copy the `Web app URL`**.

   * **Set Trigger:** In Apps Script, click "Triggers" (alarm clock icon), click **`Add Trigger`**, set it to run **`checkOverdueWorkers`** on a **`Time-driven`** trigger, **`Minutes timer`**, **`Every 5 minutes`**. Click **`Save`**.

   * Click **"Next Step"** in the setup tool.

   ### **Step 4: Configure Apps**

   * **Web App URL:** Paste the **`Web app URL`** you copied.

   * **Secret Key:** Verify this matches the key you created.

   * **Configure:** Set the First Alert Time, Check-in options, and optional Logo URL.

   * **Test:** Click **"Test Connection & Proceed"**. If successful, you'll move to the last step.

   ### **Step 5: Download**

   * Click the **"Generate & Download App Package (.zip)"** button.

   * Save the `LWS_App_Package.zip` file to your computer.

   ### **Step 6: Finish**

   * Click the link to download the **Full Documentation PDF** for your records.

## **Part 2: Host Your Generated Apps using GitHub Pages**

Now you'll upload the apps from the `.zip` file to your own GitHub repository to make them live.

### **2.1 Prepare Your Files**

* **Unzip:** Find the `LWS_App_Package.zip` file you downloaded and unzip it.

* **Locate Folders:** Inside, find the `LoneWorkerApp` folder and the `MonitoringDashboardApp` folder. These contain your customized apps.

### **2.2 Create a GitHub Account (If you don't have one)**

* Go to <https://github.com/>.

* Click **"Sign up"** and create a free account. Verify your email.

### **2.3 Create a New Repository**

* Log in to GitHub. Click **"+"** > **"New repository"**.

* **Repository name:** Choose a name, e.g., `my-lws-apps`.

* Select **"Public"**.

* Click **"Create repository"**.

### **2.4 Upload App Folders**

* On your new repository page, click **"Add file"** > **"Upload files"**.

* **Important:** Drag the **`LoneWorkerApp` folder** AND the **`MonitoringDashboardApp` folder** from your unzipped package onto the GitHub upload area.

  * *Wait for GitHub to process both folders and all files within them.*

* Scroll down and click **"Commit changes"**.

### **2.5 Enable GitHub Pages**

* Go to your repository's **"Settings"** tab.

* Click **"Pages"** in the left sidebar.

* Under "Build and deployment", set "Source" to **"Deploy from a branch"**.

* Under "Branch", select `main`, folder `/root`, and click **"Save"**.

* **Wait:** GitHub will provide a public URL (e.g., `https://your-username.github.io/my-lws-apps/`). It might take a minute or two to become active.

### **2.6 Get Your App URLs**

Your apps are now live! Your final URLs are:

* **Worker App URL:** `[Your GitHub Pages URL]` + `/LoneWorkerApp/`

  * e.g., `https://your-username.github.io/my-lws-apps/LoneWorkerApp/`

* **Monitor App URL:** `[Your GitHub Pages URL]` + `/MonitoringDashboardApp/`

  * e.g., `https://your-username.github.io/my-lws-apps/MonitoringDashboardApp/`

Save these two URLs.

## **Part 3: Distribute to Your Team**

### **For Your Safety Monitor:**

* Send them the **Monitoring App URL**.

* **IMPORTANT:** On their first visit, they will be prompted to enter the **`Web app URL`** (from Step 3.7) and the **`Secret Key`** (from Step 3.3). They only need to do this once, as it will be saved in their browser.

### **For Your Lone Workers:**

* Send them the **Worker App URL**.

* **Instruct them to:**

  1. Open the link on their smartphone.

  2. Use their browser's menu to **"Add to Home Screen"** or **"Install app"**.

  3. Open the app from their home screen.

  4. Go to **Settings** (gear icon).

  5. Fill in **all fields** (Name, Phone, Contacts, PINs). The URL will be pre-filled.

  6. Tap **"Save Settings"**.

**Setup Complete!** Your system is operational.
