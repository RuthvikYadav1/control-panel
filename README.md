# Control Panel

A single-file personal dashboard. Open `index.html` in a browser — no build step, no server, no accounts.

## What it tracks

- **Subscriptions** — name, amount, and billing day of the month. Sorted by what's due next, with the real next payment date and a due-soon badge. Totals your monthly spend. Handles short months (a subscription billed on the 31st falls to the 30th in September, the 28th in February).
- **Money I owe** — per-person amounts with optional notes and inline partial payments, tracked against the total until settled.
- **Job deadline** — a target date with a day counter that turns amber inside two weeks and red once it passes.
- **Application tracker** — one button, one tap per job application. Today / last 7 days / per-day average, a 14-day bar chart, and undo.
- **Pomodoro** — 25/5/15 with pause and resume, an audio chime, automatic switch into a break (long break after every 4th focus block), a daily count, and the countdown mirrored into the tab title.

## Your data

Everything is saved to your browser's `localStorage`. Nothing is uploaded and there is no backend — which also means the data lives in one browser on one machine, and clearing site data erases it.

Use **Export** in the header to save a dated JSON backup, and **Import** to restore it or carry it to another machine. Worth doing occasionally.
