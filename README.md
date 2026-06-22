# MedMix — Global Medication & Chemical Safety Network

A static, single-file web platform for community-reported chemical/medication
interaction safety, counterfeit medication watch, and a verified reference
database of known interactions.

## What's in this repo
- `index.html` — the entire site (HTML, CSS, JS in one file). No build step.
- `.nojekyll` — tells GitHub Pages not to run Jekyll on the repo (needed since
  the file uses underscores/special characters GitHub's default processor can
  otherwise mishandle).

## 1. Deploy as a static site on GitHub Pages

1. Create a new GitHub repository (or use an existing one).
2. Add `index.html` and `.nojekyll` to the repo root, then commit and push:
   ```bash
   git add index.html .nojekyll
   git commit -m "Add MedMix site"
   git push
   ```
3. In the repo on GitHub: **Settings → Pages**.
4. Under **Source**, choose **Deploy from a branch**, pick `main` (or
   `master`) and `/ (root)`, then **Save**.
5. GitHub will give you a live URL, typically:
   `https://<your-username>.github.io/<repo-name>/`
   It can take 1–2 minutes to go live after the first push.

That's it — no server, no build process. Every time you push a change to
`index.html`, the live site updates automatically within a minute or two.

## 2. Connect a real, shared database (Firebase) — required for global signups

Right now, if Firebase isn't configured, the site falls back to
**browser-only storage** (`localStorage`) — meaning signups and reports only
persist on your own device/browser, not across everyone who visits.

To make signups and reports **real and shared globally**:

1. Go to [https://console.firebase.google.com](https://console.firebase.google.com)
   and create a new project (free tier is enough for this).
2. In the project, click **Add app → Web app** (the `</>` icon). Give it a
   nickname, you don't need Firebase Hosting.
3. Firebase will show you a config object that looks like this:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "yourproject.firebaseapp.com",
     projectId: "yourproject",
     storageBucket: "yourproject.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
4. Open `index.html`, find the `firebaseConfig` block near the top of the
   `<script>` section (search for `YOUR_API_KEY`), and paste your real values
   in.
5. Back in the Firebase console, go to **Build → Firestore Database →
   Create database**. Start in **test mode** for now (it's fine for a
   school/competition project; you can lock down security rules later —
   see note below).
6. Commit and push the updated `index.html`. Reload the live site — the
   small note under "Global risk map" should now say **"Connected — data is
   shared live with everyone who visits this site"** instead of local demo
   mode.

From this point on, every sign-up and report writes to Firestore and is
visible, in real time, to every visitor — which is what makes your member
count and report count genuinely "hundreds of people," not a local fake
number.

**Security note:** Firestore's default "test mode" rules allow anyone to
read/write for 30 days, then lock automatically. For a longer-lived public
site, tighten the rules in Firebase Console → Firestore → Rules, e.g. to
allow only `create` (not arbitrary edits/deletes) on `users` and `reports`.

## 3. Add Google Analytics (after Firebase, or independently)

Google Analytics is separate from Firebase and tracks visits/traffic, not
your app data. Add it once the site is live:

1. Go to [https://analytics.google.com](https://analytics.google.com) →
   **Admin → Create property**. Give it your site name.
2. Create a **Web** data stream, enter your live GitHub Pages URL.
3. Google will give you a snippet like:
   ```html
   <script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXX"></script>
   <script>
     window.dataLayer = window.dataLayer || [];
     function gtag(){dataLayer.push(arguments);}
     gtag('js', new Date());
     gtag('config', 'G-XXXXXXX');
   </script>
   ```
4. Paste that snippet just before the closing `</head>` tag in `index.html`.
5. Commit and push. Traffic will start showing in the Analytics dashboard
   within a few hours (real-time view shows it almost instantly).

## What's real vs. what to be careful claiming

- **Network members, countries, reports filed, growth chart** — all computed
  live from real signups/reports once Firebase is connected. No seeded or
  fake numbers are baked into the code.
- **Verified reference database / chemical connection network** — these 30+
  substance pairs are written by you as the curated baseline, sourced
  conceptually from WHO/FDA-style public guidance. Cite real sources if you
  use this for a competition or application — don't represent the existing
  entries as pulled live from an official API unless you wire one in.
- The site includes a permanent disclaimer that it is a community-reporting
  and educational tool, not a medical or regulatory authority — keep this
  intact in any version you ship.

## Architecture summary

- Single HTML file, vanilla JS (no framework, no build step).
- Tabs/sections are shown/hidden via `goTo(id)` — no router needed for a
  single-page app this size.
- `Chart.js` (CDN) for the dashboard line/bar charts.
- `Leaflet.js` (CDN) for the Global Risk Map.
- Inline SVG (generated in JS) for the Chemical & Medication Connection
  Network — a circular layout connecting every substance pair in
  `INTERACTIONS`.
- `Firebase Firestore` (CDN, compat SDK) for shared, real-time data once
  configured; falls back to `localStorage` automatically if not configured,
  so the site always works even before you set up Firebase.
