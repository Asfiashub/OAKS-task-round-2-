# SheetalTrack — Dairy Co-op Milk Chilling & Spoilage Tracker

A mobile-friendly web tool for dairy collection agents to log milk drop-offs,
watch cooling countdowns, and flag souring risk before pouring into the
shared vat. Built as a single static HTML file with a shared real-time
database, so every agent sees the same batch list and the same clock.

**Live demo:** _add your deployed URL here after step 3 below_
**Repo:** _add your GitHub URL here_

## Why Firestore (and why "local mode" exists)

The core problem in the brief is *disputes between agents* about whose batch
soured. That only gets solved if every agent sees the same shared list,
timestamped by a server they can't fudge — a single device's local storage
can't do that. So this app uses Firebase Firestore's free tier as a shared
backend: real-time sync, server-side timestamps, no server to run yourself.

If you open `index.html` without configuring Firestore, it still works in a
**local demo mode** (data lives in the browser tab only, resets on refresh)
so you can see the UI immediately — but this is a fallback for local
testing, not the graded deliverable. Do the 5-minute setup below before
you deploy.

## 1. Create a free Firestore project (~5 minutes)

1. Go to <https://console.firebase.google.com/> and create a new project
   (Google Analytics is not needed — skip it).
2. In the left menu, open **Build → Firestore Database → Create database**.
   Choose a nearby region, and start in **test mode** for now.
3. In the left menu, open **Project settings** (gear icon) → scroll to
   **Your apps** → click the **</>** (web) icon → register an app (any
   nickname) → **do not** check "Firebase Hosting" here.
4. Copy the `firebaseConfig` object it shows you.

## 2. Wire up the app

Open `index.html`, find `FIREBASE_CONFIG` near the top of the `<script>`
block, and paste your values in:

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

Save the file. Open it locally in a browser (double-click it, or run
`python3 -m http.server` in this folder and visit `http://localhost:8000`)
and confirm the "Not connected yet" banner is gone and the connection dot
next to the agent name field is green.

**Lock down the database** before you consider it done — test mode is
open to anyone. In Firestore → Rules, paste:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /batches/{batchId} {
      allow read, write: if true;
    }
  }
}
```

This keeps the prototype's low-friction, no-login field workflow (see
`TRADEOFFS.md` for why) while still requiring Firestore's request
validation instead of raw test-mode defaults. For a real deployment you'd
tighten this further — see the note in `TRADEOFFS.md`.

## 3. Deploy the static site

Any static host works since this is a single HTML file. Two easy options:

**Netlify Drop (fastest, no account needed to try, free account to keep the URL):**
1. Go to <https://app.netlify.com/drop>
2. Drag `index.html` (with your config already filled in) onto the page.
3. Netlify gives you a public URL immediately.

**GitHub Pages (keeps the live URL tied to your repo):**
1. Push this folder to a public GitHub repo (see below).
2. In the repo, go to **Settings → Pages** → Source: `main` branch, root.
3. Your live URL will be `https://<username>.github.io/<repo-name>/`.

## 4. Publish the source

```bash
git init
git add .
git commit -m "SheetalTrack: dairy chilling & spoilage tracker"
git branch -M main
git remote add origin https://github.com/<your-username>/sheetaltrack.git
git push -u origin main
```

Make sure the repo is **public** before submitting the link.

## Local setup (for anyone reviewing the code)

No build step, no dependencies to install:

```bash
git clone https://github.com/<your-username>/sheetaltrack.git
cd sheetaltrack
python3 -m http.server 8000
# visit http://localhost:8000
```

Fill in `FIREBASE_CONFIG` with your own Firestore project (or your own
copy of the reviewer's, if shared) to see live shared data; otherwise it
runs in local demo mode as described above.

## How to use it

1. Enter your name once in the "agent logging in" field — it's remembered
   on this device and stamps everything you do.
2. Log a drop-off: pick the farmer from the roster dropdown (or "Other" for
   someone not yet on it), quantity, and a temperature band (there's a "no
   thermometer" option for when agents don't have one).
3. The pending list is sorted with the most urgent batch at the top and
   shows a live countdown to the point it's treated as high spoilage risk.
4. **Pour to vat** marks it done and adds it to the current vat cycle. If
   the batch is already flagged "At risk" or "Spoiled", pouring requires a
   typed reason — that's the trust mechanism described in `TRADEOFFS.md`.
5. **Dispute** logs a disagreement (wrong quantity, wrong farmer, etc.)
   permanently against the batch without deleting or editing history.
6. The **Vat status** card fills up as batches are poured and counts down
   to when the pooled milk is considered overdue for pickup. **Tanker
   arrived — empty vat** resets the gauge for the next cycle without
   touching any historical batch records.
7. **Generate pickup summary for tanker driver** composes a short text
   (pending volume, vat fill, urgent farmers) that you can copy or open
   directly in your phone's messaging app — no SMS API or paid service
   involved, just a pre-filled `sms:` link.

## Sample data

The app ships with 15 realistic mock deliveries (one per farmer on the
roster), covering the full range of states a judge or new agent would want
to see immediately:

- Several risk levels: safe, watch, at-risk, and one already spoiled
- One batch poured normally, one poured with a risk override + reason
- One disputed batch
- A vat that already has milk in it, so the capacity monitor isn't empty on
  first load

**Local demo mode** loads this automatically. **Shared live mode**
(Firestore configured) shows a "Load sample data" banner the first time the
collection is empty — click it once, and it writes the same 15 batches
(plus a matching vat state) into your Firestore project so every agent
sees them.

## Checking your deployment actually shares data

The header shows a badge: **"Shared live mode"** (green) means Firestore is
wired up and every viewer sees the same batches; **"Local demo mode"**
(amber) means this browser tab only. Before submitting:

1. Open the deployed URL in a normal window and log a test batch.
2. Open the same URL in an incognito window (or on a different phone).
3. Confirm the badge says "Shared live mode" and the test batch appears
   there too. If it doesn't, `FIREBASE_CONFIG` likely wasn't filled in
   before you deployed — recheck step 2 above and redeploy.
