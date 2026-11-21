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

## 2. Turn on Firebase (recommended for online play)
With Firebase the game state lives in the cloud, so any phone or the TV can sleep, reload or leave and rejoin with the room code, and mobile data works as well as Wi-Fi. Each player's hand still stays only on their own device.

### a. Create the project
1. Go to <https://console.firebase.google.com> and sign in with a Google account.
2. **Create a project** (any name, e.g. `unmatched`). You can turn Google Analytics off.

### b. Create the Realtime Database
1. In the left menu: **Build → Realtime Database → Create Database**.
2. Pick the location closest to you, then **Start in locked mode** → **Enable**.
3. Open the **Rules** tab, replace everything with this, then **Publish**:
   ```json
   {
     "rules": {
       "unmatched": {
         "games": {
           "$code": {
             ".read": true,
             ".write": true,
             ".validate": "newData.isString() && newData.val().length < 400000"
           }
         },
         "$other": { ".read": false, ".write": false }
       },
       "$other": { ".read": false, ".write": false }
     }
   }
   ```
4. On the **Data** tab, copy the database URL at the top (looks like `https://unmatched-xxxx-default-rtdb.firebaseio.com`).

### c. Register the web app
1. Click the ⚙️ gear → **Project settings**. Under **Your apps**, click the **`</>`** (Web) icon.
2. Give it a nickname, leave Hosting unticked, **Register app**.
3. You'll see a `firebaseConfig = { … }` block. Keep this page open.

### d. Fill in `firebase-config.js`
Open `firebase-config.js` (on GitHub: click the file → ✏️ pencil) and replace the `PASTE_…` values with yours:
```js
window.UNMATCHED_FIREBASE = {
  apiKey: "AIza…",
  authDomain: "unmatched-xxxx.firebaseapp.com",
  databaseURL: "https://unmatched-xxxx-default-rtdb.firebaseio.com",
  projectId: "unmatched-xxxx",
  appId: "1:1234:web:abcd"
};
```
`databaseURL` is the one from step b.4 (it isn't always in the config block). **Commit changes.**

### e. Check it
Wait a minute, reload the site. The home screen should say **"Online rooms use Firebase"**. If it still says peer-to-peer, the config file has a typo or a `PASTE_` value left.

> The apiKey isn't a secret — Firebase web keys are meant to be public. The rules above only allow reading and writing game rooms.

## Playing online
- **TV mode:** open the site on the TV and tap **📺 Use this screen as the TV table**. Each player opens the site on their phone, types the room code and taps **Join**.
- **Two phones:** one player taps **Create online game**; the other joins with the code.
- If anyone reloads or loses signal, reopen the site and join the same code — you get your seat and hand back (on the same phone/browser).
- Tap **📖 Quick rules** in the room for a rules summary and each hero's special rules.

## Housekeeping
Old rooms stay in the database (each is small). To clear them: Realtime Database → Data → hover `unmatched/games` → 🗑.
The free Spark plan (1 GB stored, 10 GB/month downloaded) is far more than a game group needs.

## Updating
Upload the new `index.html` and `art-*.js` files over the old ones. **Don't overwrite your filled-in `firebase-config.js`** with the blank one from a new zip.
