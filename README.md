# Unmatched Online

A browser version of Unmatched with automated rules, pass-and-play, TV mode and online rooms. 48 heroes.

## Files
| File | What it is |
|---|---|
| `index.html` | The game |
| `art-index.js`, `art-1.js` … `art-N.js` | All card art, packed |
| `firebase-config.js` | Your Firebase settings (optional — see below) |
| `.nojekyll` | Tells GitHub Pages to serve files as-is |
| `README.md` | This file |

## 1. Put it on GitHub Pages
1. On github.com, create a new **public** repository (for example `unmatched`).
2. **Add file → Upload files** and upload every file from the zip. Commit.
3. **Settings → Pages → Build and deployment → Deploy from a branch**, pick `main` and `/ (root)`, **Save**.
4. After a minute or two the game is live at `https://YOUR-NAME.github.io/unmatched/`.

Without Firebase the game already works online, peer-to-peer: the device that creates the room hosts it and must stay open. Firebase removes that limit.

## 2. Turn on Firebase (Cloud Firestore)
With Firebase the game state lives in the cloud, so any phone or the TV can sleep, reload or leave and rejoin with the room code, and mobile data works as well as Wi-Fi. Each player's hand still stays only on its own device. Rooms are stored in a Firestore collection called `unmatched_games`.

### a. Project and Firestore
You can reuse a Firebase project you already have (each app keeps its own collection) or create a new one at <https://console.firebase.google.com>.
If the project doesn't have Firestore yet: **Build → Firestore Database → Create database**, pick a location, start in **production mode**.

### b. Rules
**Firestore Database → Rules.** Add this `match` block *inside* the existing `match /databases/{database}/documents { … }` section (keep any rules your other apps use), then **Publish**:
```
    match /unmatched_games/{code} {
      allow read, write: if true;
    }
```
For a brand-new project, the whole rules file is:
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /unmatched_games/{code} {
      allow read, write: if true;
    }
  }
}
```

### c. Web app config
⚙️ **Project settings → Your apps**. If there's no web app yet, click **`</>`** and register one. Under **SDK setup and configuration → Config** you'll see `const firebaseConfig = { … }`.
If you reuse a web app from another of your games, you can copy the same values.

### d. Fill in `firebase-config.js`
On GitHub open `firebase-config.js` → ✏️ pencil, replace the `PASTE_…` values with yours, **Commit changes**:
```js
window.UNMATCHED_FIREBASE = {
  apiKey: "AIza…",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abcd"
};
```

### e. Check it
Reload the site after a minute. The home screen should say **"Online rooms use Firebase"**.

> Firebase web API keys are meant to be public. The rule above only opens the `unmatched_games` collection.
> (Prefer Realtime Database? Add `databaseURL` and `realtime: true` to the config.)

## Playing online
- **TV mode:** open the site on the TV and tap **📺 Use this screen as the TV table**. Each player opens the site on their phone, types the room code and taps **Join**.
- **Two phones:** one player taps **Create online game**; the other joins with the code.
- If anyone reloads or loses signal, reopen the site and join the same code — you get your seat and hand back (on the same phone/browser).
- Tap **📖 Quick rules** in the room for a rules summary and each hero's special rules.

## Housekeeping
Old rooms stay in Firestore (each is a small document). To clear them: Firestore Database → Data → `unmatched_games` → ⋮ → Delete collection.
The free Spark plan (50k reads / 20k writes a day) covers roughly 10–15 full games a day; past that, Firestore just pauses writes until the next day (or switch on the pay-as-you-go plan, which costs pennies at this scale).

## Updating
Upload the new `index.html` and `art-*.js` files over the old ones. **Don't overwrite your filled-in `firebase-config.js`** with the blank one from a new zip.
