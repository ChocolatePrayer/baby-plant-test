# Baby Plant - Setup Guide

This guide walks you through setting up the Arduino moisture tracking system with Firebase so your GitHub Pages website shows real-time data.

## Architecture

```
Arduino (sensor)
    |  USB serial
    v
Your Laptop  ──runs──>  arduino-bridge.js  ──pushes──>  Firebase Realtime DB
                                                              |
                                                         real-time sync
                                                              |
                                                              v
                                                    GitHub Pages website
                                                       (index.html)
```

---

## Step 1: Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **Add project** and give it a name (e.g. `baby-plant`)
3. You can disable Google Analytics (not needed)
4. Click **Create project**

## Step 2: Enable Realtime Database

1. In the Firebase Console sidebar, click **Build > Realtime Database**
2. Click **Create Database**
3. Choose a location closest to you
4. For security rules, start in **test mode** (allows open read/write for 30 days — see Step 6 for production rules)

## Step 3: Register a Web App

1. In Firebase Console, click the **gear icon > Project settings**
2. Scroll to "Your apps" and click the **</>** (Web) icon
3. Give it a nickname (e.g. `baby-plant-web`). You do NOT need Firebase Hosting
4. Click **Register app**
5. You will see a config object like this:

```js
const firebaseConfig = {
  apiKey:            "AIzaSy...",
  authDomain:        "baby-plant-12345.firebaseapp.com",
  databaseURL:       "https://baby-plant-12345-default-rtdb.firebaseio.com",
  projectId:         "baby-plant-12345",
  storageBucket:     "baby-plant-12345.appspot.com",
  messagingSenderId: "123456789",
  appId:             "1:123456789:web:abc123"
};
```

**Copy these values.** You will paste them into two places.

## Step 4: Add Your Firebase Config

### 4a. Local bridge config (never committed)

Open `firebase-config.js` and replace all the `YOUR_...` placeholders with your real values.

### 4b. Website config (committed to GitHub)

Open `index.html` and replace the same `YOUR_...` placeholders inside the `<script>` tag with your real values.

> **Note:** The API key in `index.html` will be public. This is normal and expected for Firebase client-side apps. The Realtime Database security rules (Step 6) are what actually protect your data, not the API key.

## Step 5: Install Dependencies and Run the Bridge

Make sure Node.js is installed, then:

```bash
cd /path/to/this/repo
npm install
```

Plug in your Arduino via USB, then run:

```bash
node arduino-bridge.js
```

You should see:

```
Firebase connected — sensor marked online.
Listening on /dev/cu.usbmodem1101 at 9600 baud...
Pushed moisture: 42%
Pushed moisture: 41%
...
```

### Troubleshooting serial port path

If you get a "could not open serial port" error, find the correct path:

```bash
ls /dev/cu.*
```

Look for something like `/dev/cu.usbmodem1101` or `/dev/cu.usbmodem14201`. Update the `SERIAL_PATH` variable at the top of `arduino-bridge.js`.

## Step 6: Set Database Security Rules (Recommended)

Once you confirm everything works, go to **Realtime Database > Rules** in the Firebase Console and replace the default rules with:

```json
{
  "rules": {
    "sensor": {
      ".read": true,
      ".write": false
    }
  }
}
```

This allows anyone to **read** the moisture data (your website visitors) but blocks unauthorized **writes**.

To allow your bridge script to still write, you have two options:

### Option A: IP-based (simplest)

Keep rules open (`.write: true`) since only your bridge writes to it, and the data is non-sensitive.

### Option B: Authenticated writes (more secure)

1. In Firebase Console, go to **Authentication > Sign-in method** and enable **Anonymous** sign-in
2. Update the security rules:

```json
{
  "rules": {
    "sensor": {
      ".read": true,
      ".write": "auth != null"
    }
  }
}
```

3. Add authentication to `arduino-bridge.js` by adding these lines after `initializeApp`:

```js
const { getAuth, signInAnonymously } = require('firebase/auth');
const auth = getAuth(app);
await signInAnonymously(auth);
```

## Step 7: Deploy to GitHub Pages

1. Commit and push `index.html`, `main.css`, `homepage.html`, `.gitignore`, and any other HTML pages
2. On GitHub, go to **Settings > Pages**
3. Under "Source", select **Deploy from a branch**
4. Choose `main` branch and `/ (root)` folder
5. Click **Save**
6. Your site will be live at `https://YOUR_USERNAME.github.io/YOUR_REPO_NAME/`

---

## Files Overview

| File | Purpose | On GitHub? |
|---|---|---|
| `index.html` | Moisture tracking page (reads Firebase) | Yes |
| `homepage.html` | Home page with plant categories | Yes |
| `main.css` | Stylesheet | Yes |
| `.gitignore` | Prevents local files from being committed | Yes |
| `SETUP.md` | This guide | Yes |
| `arduino-bridge.js` | Reads Arduino + pushes to Firebase | **No** |
| `firebase-config.js` | Firebase credentials for the bridge | **No** |
| `server.js` | Old local server (no longer needed) | **No** |
| `package.json` | Node.js dependencies | **No** |
| `node_modules/` | Installed packages | **No** |

---

## Daily Usage

1. Plug Arduino into your laptop
2. Run `node arduino-bridge.js`
3. Your GitHub Pages website updates in real time
4. When you unplug the Arduino or stop the script, the website shows "Arduino device offline"
5. Press `Ctrl+C` to stop the bridge
