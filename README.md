# Control Panel

A single-file personal dashboard. Open `index.html` in a browser — no build step, no server to run yourself.

## What it tracks

- **Subscriptions** — name, amount, and billing day of the month. Sorted by what's due next, with the real next payment date and a due-soon badge. Totals your monthly spend. Handles short months (a subscription billed on the 31st falls to the 30th in September, the 28th in February). Click the pencil on any row to edit its name, amount, or day in place — Enter saves, Escape cancels.
- **Money I owe** — per-person amounts with optional notes and inline partial payments, tracked against the total until settled. Edit (pencil), add, and delete are all available per entry; lowering an amount below what's already been paid clamps the paid total down to match.
- **Job deadline** — a target date with a day counter that turns amber inside two weeks and red once it passes.
- **Application tracker** — one button, one tap per job application, with an optional description field (role, company) that's attached to that entry. Today / last 7 days / per-day average, and a 14-day bar chart. **view log** opens the full history grouped by day — add or edit a description after the fact, delete a single mistaken entry, or clear a whole day at once. **Undo** (or Ctrl+Z) steps back through every application change — a log, a delete, a description edit, or a whole-day clear — and the button names what it will reverse. History is per-session, so a reload starts it fresh.
- **Pomodoro** — 25/5/15 with pause and resume, an audio chime, automatic switch into a break (long break after every 4th focus block), a daily count, and the countdown mirrored into the tab title.

## Your data

**Out of the box** — before you've done the setup below — everything is saved to your browser's `localStorage`. Nothing is uploaded, the data lives in one browser on one machine, and clearing site data erases it. Use **Export** in the header to save a dated JSON backup, and **Import** to restore it or carry it to another machine.

**After the setup below** — your data instead lives in a free cloud database (Firebase/Firestore), behind your own sign-in. It syncs automatically to every device where you sign in with the same account, survives clearing your browser's data, and Export/Import still work as a manual backup on top of that.

## Cloud sync setup (optional, ~10 minutes)

This step is entirely yours to do — it needs your own Google account, and I (Claude) never sign into it on your behalf. Everything below happens at [console.firebase.google.com](https://console.firebase.google.com).

1. **Create a project.** Click "Add project", give it any name, and you can skip Google Analytics — it's not needed here. The free "Spark" plan covers this app comfortably.
2. **Register a web app.** On the project's Overview page, click the `</>` (web) icon. Give it a nickname (anything). You do **not** need Firebase Hosting — skip that checkbox.
3. **Copy the config.** After registering, Firebase shows a `firebaseConfig = { apiKey: "...", ... }` snippet. Copy those six values.
4. **Paste them into `index.html`.** Near the top of the `<script type="module">` block, find:
   ```js
   const firebaseConfig = {
     apiKey: "PASTE_YOUR_API_KEY",
     ...
   };
   ```
   Replace each `PASTE_YOUR_...` placeholder with your real value. This config is not a secret — it's meant to be public in client-side code; your data itself is protected by the rule in step 6, not by hiding this.
5. **Turn on Email/Password sign-in.** In the left sidebar: Build → Authentication → Get started → Sign-in method → Email/Password → enable it → Save.
6. **Create the database and lock it down.** Left sidebar: Build → Firestore Database → Create database → any region → start in production mode. Once created, open the **Rules** tab and replace the contents with:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{uid} {
         allow read, write: if request.auth != null && request.auth.uid == uid;
       }
     }
   }
   ```
   Click **Publish**. This is what actually protects your data — it means a signed-in account can only ever read or write its own document, never anyone else's, regardless of who else signs up.
7. **Push the updated `index.html`** (with your real config) to GitHub, or just open the file locally to test first.

Once that's done, the site shows a sign-in screen. Create an account with any email and password (6+ characters) — it doesn't need to be a real, verifiable email address, since there's no verification step. The first time you sign in, if this browser already had data saved locally, you'll be asked whether to import it into your new account.

If you skip all of this, the file keeps working exactly as it does today — local-only, no sign-in — until you fill in the config.
