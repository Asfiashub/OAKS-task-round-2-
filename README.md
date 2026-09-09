# Milkvault — Dairy Co-op Milk Chilling & Spoilage Tracker

## Overview

MilkVault is a mobile-friendly web application for dairy collection agents. It helps record milk deliveries, monitor cooling time, identify spoilage risk, and manage milk added to the shared vat.

The application uses a single `index.html` file with Firebase Firestore. When Firestore is connected, all agents can see the same batch information and updates.

**Live demo:** https://asfiashub.github.io/OAKS-task-round-2-/

**Repo:** https://github.com/Asfiashub/OAKS-task-round-2-

## Why a Shared Database

The main problem this project solves is disagreements between agents about which milk batch caused spoilage.

If each agent keeps separate records, it is difficult to know which information is correct. A shared database gives everyone the same batch list and timestamps.

MilkVault uses Firebase Firestore for real-time sharing. The timestamps are also created by the server instead of relying on each phone's clock.

The application also has **Local demo mode**. If Firestore is not configured, the app can still be tested, but the data stays in the browser tab and resets after a refresh. For the actual submission, Firestore should be connected so the app works in **Shared live mode**.

## Setup

### Firebase Project

1. Open the [Firebase Console](https://console.firebase.google.com/) and create a project. Google Analytics can be skipped.
2. From the project overview, select **Databases and storage** from the left sidebar.
3. Select **Firestore** → **Create database**.
4. Choose **Standard edition**, select a nearby region, and choose **Start in test mode**.
5. Open **Project settings** → **Your apps** → web icon `</>` and register a web app.
6. Do not enable Firebase Hosting.
7. Copy the `firebaseConfig` shown by Firebase.

### Connect Firebase

Open `index.html` and find `FIREBASE_CONFIG`. Replace its values with your Firebase configuration:

```js
const FIREBASE_CONFIG = {
  apiKey: "AIza...",
  authDomain: "yourproject.firebaseapp.com",
  projectId: "yourproject",
  storageBucket: "yourproject.appspot.com",
  messagingSenderId: "...",
  appId: "..."
};
```

Save the file and run the project locally:

```bash
python3 -m http.server 8000
```

Open `http://localhost:8000`. If the setup is correct, the connection indicator should turn green and the **"Not connected yet"** message should disappear.

### Secure the Database

Test mode is open, so update the Firestore rules before deployment. In **Firestore → Rules**, use:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /batches/{batchId} {
      allow read, write: if true;
    }
  }
}
```

These rules keep the prototype's simple no-login workflow. They are suitable for the prototype but should be tightened further for a production system.

## Deploying

Since SheetalTrack is a static website, it can be deployed using any static hosting service.

### Netlify Drop

1. Open [Netlify Drop](https://app.netlify.com/drop).
2. Drag your configured `index.html` onto the page.
3. Use the URL provided by Netlify.

### GitHub Pages

1. Push the project to a public GitHub repository.
2. Open **Settings → Pages**.
3. Select the `main` branch and root folder.
4. Save the settings.

The site will be available at:

```text
https://<username>.github.io/<repo-name>/
```

## Publishing the Source Code

Run these commands inside the project folder:

```bash
git init
git add .
git commit -m "SheetalTrack: dairy chilling & spoilage tracker"
git branch -M main
git remote add origin https://github.com/<your-username>/sheetaltrack.git
git push -u origin main
```

Keep the repository **public** before submitting it.

## Local Setup for Reviewers

There are no dependencies or build steps.

```bash
git clone https://github.com/<your-username>/sheetaltrack.git
cd sheetaltrack
python3 -m http.server 8000
```

Then open `http://localhost:8000`.

For shared data, add your own Firebase configuration to `FIREBASE_CONFIG`. Without it, the app runs in Local demo mode.

## How to Use It

1. Enter your name in the **"agent logging in"** field. It is remembered on that device.
2. Record a delivery by selecting a farmer, entering the quantity, and choosing a temperature band. A **"no thermometer"** option is available.
3. The pending list shows the most urgent batches first and provides a cooling countdown.
4. **Pour to vat** marks a batch as completed. If it is **At risk** or **Spoiled**, a reason is required before pouring.
5. **Dispute** records a disagreement without changing the original batch history.
6. **Vat status** shows the current fill level and pickup countdown. **Tanker arrived — empty vat** starts the next vat cycle.
7. **Generate pickup summary for tanker driver** creates a short message with pending volume, vat level, and urgent farmers. It can be copied or opened in the phone's messaging app. No SMS API is required.

## Sample Data

The app includes **15 mock deliveries**, one for each farmer in the roster. The samples cover safe, watch, at-risk, and spoiled states.

They also include a normally poured batch, a risk override with a reason, a disputed batch, and a partially filled vat.

In **Local demo mode**, the data loads automatically. In **Shared live mode**, the app provides a **"Load sample data"** option when the Firestore collection is empty. This adds the 15 sample batches and vat state to the shared database.

## Verifying Your Deployment

The header shows the current mode:

* **Shared live mode** — Firestore is connected and data is shared.
* **Local demo mode** — data is stored only in the current browser tab.

Before submitting:

1. Open the deployed site and confirm **Shared live mode**.
2. Add a test batch.
3. Open the same URL in an incognito window or another phone.
4. Confirm that the test batch appears there too.

If it does not appear, check the `FIREBASE_CONFIG` values and redeploy the site.
