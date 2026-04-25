# Pellet Arena 🟡

A multiplayer Pac-Man-style arena game. Real-time multiplayer powered by Firebase. Hosted free on GitHub Pages.

## Quick overview

- One static `index.html` file
- Firebase Realtime Database keeps players, ghosts, pellets, and scores in sync
- Anyone who visits your URL joins the same shared room
- Free tier of both services is plenty for casual play

---

## Setup (about 10 minutes total)

### Part 1 — Create your Firebase project (5 min)

1. Go to <https://console.firebase.google.com> and sign in with your Google account.
2. Click **Add project**. Name it anything (e.g. `pellet-arena`). You can disable Google Analytics — not needed.
3. Once the project is created, in the left sidebar click **Build → Realtime Database**.
4. Click **Create Database**.
   - Pick the location closest to you (e.g. `us-central1`).
   - Choose **Start in test mode**. (This makes the DB readable/writable by anyone for 30 days. Fine to start; we'll lock it down properly in Part 4.)
5. Now grab your config:
   - Click the gear icon (top left) → **Project settings**.
   - Scroll to **Your apps** → click the `</>` web icon.
   - Give the app a nickname (e.g. `pellet-web`). You do **not** need Firebase Hosting — leave that unchecked.
   - Click **Register app**.
   - You'll see a `firebaseConfig` block. **Copy the whole object.**

### Part 2 — Paste your config into the game (1 min)

1. Open `index.html` in a text editor.
2. Find the block near the top of the `<script>` that says:
   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     ...
   };
   ```
3. Replace it with the config you just copied from Firebase. **Important:** make sure your config object includes `databaseURL`. If it's missing, look for it on the Realtime Database page (it looks like `https://your-project-default-rtdb.firebaseio.com`) and add it manually.
4. Save the file.

### Part 3 — Push to GitHub Pages (3 min)

1. On <https://github.com>, click **+ → New repository**. Name it whatever you want (e.g. `pellet-arena`). Set it to **Public**. No README, no `.gitignore` (we already have one).
2. Upload the files. Easiest way:
   - On the new empty repo page, click **uploading an existing file**.
   - Drag `index.html`, `README.md`, and `.gitignore` into the upload box.
   - Click **Commit changes**.
3. Turn on Pages:
   - In the repo, go to **Settings → Pages** (in the left sidebar).
   - Under **Source**, pick **Deploy from a branch**.
   - Branch: `main`, folder: `/ (root)`. Click **Save**.
4. Wait about a minute. Refresh the Pages settings page — you'll see a green box with your live URL like:
   ```
   https://YOUR_USERNAME.github.io/pellet-arena/
   ```
5. Open that URL. You should see the join screen. Type a name, click join, and play!

### Part 4 — Lock down your database rules (2 min, important)

By default, Firebase test mode lets **anyone** read or write **any** data in your database, and it expires in 30 days. Let's tighten it to just our game data so it doesn't expire and isn't abused.

1. Back in the Firebase console, go to **Realtime Database → Rules** tab.
2. Replace the contents with:
   ```json
   {
     "rules": {
       "rooms": {
         "$room": {
           ".read": true,
           ".write": true
         }
       }
     }
   }
   ```
3. Click **Publish**.

This still allows anyone to read/write the game room (which is what you want for an open multiplayer game) but blocks access to anything outside `rooms/`. It also won't expire.

> **Heads up:** these rules are open by design — anyone with your URL can join and modify game state. For a small toy game shared with friends, that's fine. For something more serious, you'd add Firebase Authentication and stricter rules.

---

## Sharing the game

Send people the GitHub Pages URL. Everyone who visits joins the same room (`main`) and shows up on the leaderboard.

Want to set up multiple separate rooms? In `index.html`, change:
```js
const ROOM = "main";
```
to whatever you want (e.g. `"alice-and-bob"`). You could read it from the URL hash if you want different rooms for different links — say the word and I can add that.

---

## How it plays

- **Movement:** Arrow keys, WASD, or the on-screen pad on mobile
- **Yellow pellets:** +10 each
- **Pulsing power pellets (corners):** +50, plus 6 seconds of ghost-eating power
- **Eat ghost while powered:** +200
- **Caught by ghost:** -100 and you respawn
- Map refills when all pellets are gone — round counter ticks up

---

## How the multiplayer works (under the hood)

- Each player runs the game loop locally and sends their position to Firebase 8x/sec.
- Firebase pushes updates to every other connected player in real time (no polling).
- Pellets are removed from the shared DB the instant someone eats them, so eating is "first come first served."
- One player at a time is elected to drive the ghosts (a 6-second lease that auto-renews). If they leave, another player takes over within 6 seconds.
- Disconnected players are auto-removed via Firebase's `onDisconnect` (clean leave when you close the tab).

---

## Troubleshooting

**"Setup needed" warning on load:** You haven't replaced the Firebase config in `index.html`.

**"Connection failed" or stuck on connecting:** Most likely your `databaseURL` is missing or wrong. Double-check it matches what's shown in your Firebase Realtime Database console (top of the Data tab).

**Permission denied errors in browser console:** Your database rules are blocking access. Re-do Part 4 above.

**Game works locally but not on GitHub Pages:** Hard refresh (Cmd+Shift+R / Ctrl+Shift+F5) — Pages caches aggressively. Also make sure you committed the latest `index.html` after editing the config.

**Other people don't see me:** Verify you're both opening the exact same URL (including the trailing slash). Check that `ROOM` is the same value in everyone's copy.

---

## Free tier limits

Firebase Realtime Database free tier: 100 simultaneous connections, 1 GB stored, 10 GB/month bandwidth. For a casual game with friends, you'll never come close.

GitHub Pages free tier: 100 GB/month bandwidth, 1 GB site size. Plenty.
