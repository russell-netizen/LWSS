# Setup Tool Administrator Guide: Hosting the LWS Deployment Tool

**Target Audience:** This guide is for the person responsible for hosting the `setup.html` deployment tool and its associated template files online, making it accessible to end-user administrators.

**Goal:** To host the `setup.html` tool using GitHub Pages so that Deployment Administrators can easily access and use it.

## **Steps to Host the Deployment Tool**

### **1. Prepare Your Files**

Ensure you have the following files and folder structure ready on your computer. The `Deployment_Admin_Guide.md` file (which the setup tool will include in the generated zip for end-users) should be inside the `templates` folder.

<img width="375" height="277" alt="image" src="https://github.com/user-attachments/assets/5d83c27f-0007-4cd9-82fe-7a6100dd2a7a" />


*(**Note:** The `Deployment_Admin_Guide.md` file should already be correctly named within the `templates` folder as provided in the complete file set).*

### **2. Create a Free GitHub Account (If you don't have one)**

* Go to <https://github.com/>.
* Click **"Sign up"** and follow the instructions to create a free account.
* Verify your email address.

### **3. Create a New Repository**

* Once logged in, click the **"+"** icon (top-right) > **"New repository"**.
* **Repository name:** e.g., `lws-deploy-tool`.
* Select **"Public"**.
* Click **"Create repository"**.

### **4. Upload Your Files**

* On your new repository page, click **"Add file"** > **"Upload files"**.
* Drag the `setup.html` file onto the upload area.
* Drag the **entire `templates` folder** onto the upload area. Ensure the folder name is lowercase `templates`.
* Scroll down and click **"Commit changes"**.

### **5. Enable GitHub Pages**

* Go to your repository's **"Settings"** tab.
* Click **"Pages"** in the left sidebar.
* Under "Build and deployment", set "Source" to **"Deploy from a branch"**.
* Under "Branch", select `main` (or `master`), folder `/root`, and click **"Save"**.
* **Wait:** GitHub will provide a public URL (e.g., `https://your-username.github.io/lws-deploy-tool/`). It might take a minute or two to become active. 

### **6. Get the Tool URL**

* The URL for your Deployment Administrators to use is the GitHub Pages URL with `setup.html` added to the end.
* **Example:** `https://your-username.github.io/lws-deploy-tool/setup.html`
* Share this URL with your Deployment Administrators.

**Hosting Complete:** The setup tool is now live and ready for your administrators to use. They will follow the instructions provided *within* the tool itself.
