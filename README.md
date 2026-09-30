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

**After the setup below** — your data instead lives in a free cloud database (Supabase), behind your own sign-in. It syncs automatically — live, not just on refresh — to every device where you sign in with the same account, survives clearing your browser's data, and Export/Import still work as a manual backup on top of that.

## Cloud sync setup (optional, ~10 minutes)

This step is entirely yours to do — it needs your own account, and I (Claude) never sign into it on your behalf. Everything below happens at [supabase.com](https://supabase.com) (free, no credit card required).

1. **Create a project.** Sign up, then "New project" — give it any name and a database password (Supabase asks for one; you won't need it day-to-day, just save it somewhere). Pick any region.
2. **Copy the API config.** Once the project finishes provisioning: Settings (gear icon) → API. Copy the **Project URL** and the **`anon` `public`** key (not the `service_role` key — that one must never appear in client-side code).
3. **Paste them into `index.html`.** Near the top of the `<script type="module">` block, find:
   ```js
   const supabaseConfig = {
     url: "PASTE_YOUR_PROJECT_URL",
     anonKey: "PASTE_YOUR_ANON_KEY"
   };
   ```
   Replace both placeholders with your real values. Neither is a secret — the anon key is meant to be public in client-side code; your data itself is protected by the policy in step 5, not by hiding this.
4. **Turn off "Confirm email"** (recommended for a personal tool). Authentication → Providers → Email → toggle off "Confirm email" → Save. Without this, signing up sends a confirmation email and you can't sign in until you click the link in it — extra friction with no real benefit for an account only you will ever use.
5. **Create the table and lock it down.** Left sidebar → SQL Editor → New query → paste this in and run it:
   ```sql
   create table user_data (
     user_id uuid primary key references auth.users(id) on delete cascade,
     data jsonb not null default '{}'::jsonb,
     updated_at timestamptz not null default now()
   );
   alter table user_data enable row level security;
   create policy "read own data" on user_data for select using (auth.uid() = user_id);
   create policy "insert own data" on user_data for insert with check (auth.uid() = user_id);
   create policy "update own data" on user_data for update using (auth.uid() = user_id);
   alter publication supabase_realtime add table user_data;
   ```
   The three `create policy` lines are what actually protect your data — they mean a signed-in account can only ever read or write its own row, never anyone else's, regardless of who else signs up. The last line turns on live sync, so a change on one device shows up on another without a refresh.
6. **Push the updated `index.html`** (with your real config) to GitHub, or just open the file locally to test first.

Once that's done, the site shows a sign-in screen. Create an account with any email and password (6+ characters). The first time you sign in, if this browser already had data saved locally, you'll be asked whether to import it into your new account.

If you skip all of this, the file keeps working exactly as it does today — local-only, no sign-in — until you fill in the config.
