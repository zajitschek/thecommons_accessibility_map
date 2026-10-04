# Community Pavement Accessibility Map: Setup & Deployment Guide

## 1. Project Overview & Requirements

### What is this project?
This repository provides a lightweight, open-source web application for community groups to map pavement accessibility issues (such as missing dropped curbs, high curbs, broken paving slabs, and overgrown hedges). 

Volunteers sign in with an email and individual password provided by the project organizer, drop pins on an interactive map, describe the issue, and optionally upload photo evidence. The aggregated data is stored securely in a database and can be exported as reports for ward councillors and local highway officers.

### Requirements & Prerequisites
To run this application, **no paid subscriptions or credit cards are required**. You do **not** need to install terminal software or command-line tools on your local computer. Everything is managed through your web browser.

* **GitHub Account (Free):** Used to host your code repository and automatically publish the live web app via GitHub Actions.
* **Google Account / Gmail (Free):** Required to access the Firebase Console where your map data and user logins are hosted.
* **Bank Details:** **NOT REQUIRED.** Firebase's default "Spark Plan" is 100% free with generous daily usage quotas (50,000 reads/day and 20,000 writes/day), which is more than enough for local community projects.

---

## 2. Directory & File Structure

When you clone or fork this template on GitHub, your repository folder structure should look like this:

```text
/ (Repository Root)
├── .github/
│   └── workflows/
│       └── deploy.yml              # GitHub Actions script for automated deployment
├── public/
│   ├── index.html                  # Main interactive map page
│   ├── css/
│   │   └── styles.css              # Custom map styling
│   └── js/
│       ├── app.js                  # Map logic and Leaflet.js setup
│       └── firebase-config.js      # Firebase project keys and connection logic
├── firestore.rules                 # Security rules for Firestore Database
├── firebase.json                   # Optional deployment config for Firebase Hosting
└── README.md                       # Community overview page
```

---

## 3. Step-by-Step Setup Instructions

### Step 1: Create and Configure Your Firebase Backend (In Web Browser)
1. Open your browser and go to [https://console.firebase.google.com](https://console.firebase.google.com).
2. Log in with your Google / Gmail account.
3. Click **Add Project** (or **Create a Project**):
   * Project Name: Enter something like `formby-accessibility-map`.
   * Disable Google Analytics (optional, not required for this project).
   * Click **Create Project**.
4. Set up **Authentication**:
   * In the left sidebar menu, click **Build** > **Authentication**.
   * Click **Get Started**.
   * Under **Native providers**, click **Email/Password**.
   * Enable the first toggle switch (**Email/Password**). Leave "Email link (passwordless sign-in)" disabled.
   * Click **Save**.
5. Set up **Firestore Database**:
   * In the left sidebar menu, click **Build** > **Firestore Database**.
   * Click **Create Database**.
   * Choose a location close to you (e.g., `eur3` or `europe-west2` for UK).
   * Choose **Start in production mode** and click **Create**.
6. Register your Web App & Get API Keys:
   * Click the **Gear Icon (Project Settings)** in the top left sidebar.
   * Scroll down to the **Your apps** section and click the **Web icon (`</>`)**.
   * App nickname: `Pavement Audit Web App`.
   * Click **Register app**.
   * Copy the `const firebaseConfig` code snippet shown on screen.

---

### Step 2: Configure Your Security & Zero-Cost Guarantees

#### 1. Preventing Unexpected Costs
By default, your Firebase project is created on the **Spark (No-Cost) Plan**. 
* **Hard Stop Protection:** On the Spark plan, if your usage ever reaches the daily free limit (e.g., 50,000 reads in one day), Firebase **automatically pauses service for the rest of the day** rather than charging you. It resets back to zero at midnight.
* Do **not** upgrade to the "Blaze (Pay as you go)" plan, and no billing details will ever be requested or billed.

#### 2. Configure Database Security Rules
To ensure hackers cannot erase or tamper with your map data, set your security rules directly in the browser:

1. Go to **Firebase Console** > **Firestore Database** > **Rules** tab.
2. Replace the text in the code editor with the following:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Pavement issues collection
    match /accessibility_issues/{issueId} {
      // Anyone on the internet can view logged map pins
      allow read: if true;
      
      // Only authenticated volunteers with a validated account can add or edit pins
      allow create, update: if request.auth != null 
                            && request.resource.data.keys().hasAll(['latitude', 'longitude', 'issueType'])
                            && request.resource.data.issueType is string;
                            
      // Only administrators can delete records
      allow delete: if request.auth.token.admin == true;
    }
  }
}
```
3. Click **Publish**.

---

### Step 3: Configure Code Files in GitHub

1. Open your GitHub repository on [https://github.com](https://github.com).
2. Navigate to `public/js/firebase-config.js`.
3. Click the **Pencil icon (Edit this file)** in the top right.
4. Update the file with your Firebase details copied from Step 1:

```javascript
// Located at: /public/js/firebase-config.js
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
import { getAuth } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-auth.js";
import { getFirestore } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "YOUR_FIREBASE_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);
```
5. Scroll down and click **Commit changes...**.

---

### Step 4: Automated Deployment via GitHub Actions & Pages

You can publish the website automatically using **GitHub Pages** whenever you update code in your repository.

#### 1. Create the GitHub Actions Workflow File
In your repository on GitHub:
1. Click **Add file** > **Create new file**.
2. File path: `.github/workflows/deploy.yml`
3. Paste the following configuration code:

```yaml
name: Deploy Pavement Audit Map to GitHub Pages

on:
  push:
    branches:
      - main

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Setup GitHub Pages
        uses: actions/configure-pages@v4

      - name: Upload Pages Artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './public'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
4. Scroll down and click **Commit changes...**.

#### 2. Enable GitHub Pages Settings
1. On GitHub, click the **Settings** tab at the top of your repository.
2. In the left menu, click **Pages** (under the "Code and automation" section).
3. Under **Build and deployment** > **Source**, select **GitHub Actions**.
4. GitHub will automatically trigger your action. Within 1–2 minutes, your live website URL will appear at the top of the Pages settings page (e.g., `https://your-username.github.io/your-repo-name/`).

---

## 4. Operational Guide: Registering Volunteers

Because logging is restricted to approved volunteers, here is how you issue access:

1. Volunteer emails `theformbycommons@protonmail.com` with the subject line **Access**.
2. Go to **Firebase Console** > **Authentication** > **Users** tab.
3. Click **Add user**.
4. Enter the volunteer's email address as their **Email**.
5. Click **Generate password** (or enter a strong custom password).
6. Click **Add user**.
7. Email the generated password back to the volunteer. They can now log into the web map and start auditing pavement issues!