# 🃏 The Cash Game Ledger

A static, zero-backend bookkeeper for a regular home poker game. Drop each
night's ledger CSV into the repo and the page automatically works out the
weekly result, every player's performance, and the shortest list of payments
that squares everyone up.

Hosted free on **GitHub Pages**. The committed CSV files are the single source
of truth, so the numbers are always auditable.

---

## What it does

- **Settlement** — for each week, the minimum set of "X pays Y $Z" transfers,
  with tick-boxes to track who's paid and a progress bar.
- **Players** — net, ROI, sessions, win–loss, time played, $/hr.
- **Game log** — every night, expandable to a per-player breakdown.
- **Lifetime** — running standings with a per-game trend sparkline.
- **Files & audit** — the raw committed CSVs with a balance check on each.
- **Add a game** — turns a pasted export into the exact file + manifest line to commit.
- **Guide** — in-app navigation help.

Players are matched by **account ID**, not nickname, so one person under two
names merges automatically and two different people with similar names stay
separate.

---

## Setup (one time)

1. Create a new GitHub repository (public).
2. Upload these files, keeping the structure:
   ```
   index.html
   README.md
   ledgers/
     manifest.json
     2026-06-01.csv
     2026-05-31.csv
   ```
3. In the repo: **Settings → Pages → Build and deployment → Source: Deploy from
   a branch**, pick `main` / `root`, save.
4. Wait a minute, then open `https://<your-username>.github.io/<repo>/`.
   Share that link with the group.

---

## Adding a game each week

1. Export the night's ledger **after everyone has cashed out** (so no one is
   still sitting on a live stack).
2. Save the CSV into `ledgers/`, named by date, e.g.
   `ledgers/2026-06-08-friday.csv`.
3. Add one line to the `ledgers` array in `ledgers/manifest.json`:
   ```json
   { "file": "2026-06-08-friday.csv", "label": "Friday night", "date": "2026-06-08" }
   ```
   The **Add a game** tab in the app writes this line for you and lets you
   download the correctly-named file.
4. Commit. Pages redeploys and the new night flows into every tab.

`manifest.json` also sets the currency symbol:
```json
{ "currency": "$", "ledgers": [ ... ] }
```

By default settlement uses the fewest possible payments. To route everything
through one person instead (everyone pays the hub, then the hub pays everyone
out), add:
```json
{ "settleHub": "Clayton" }
```

---

## Merging people across rooms (identity map)

Players are matched by **account ID**. When the same person plays under
different handles or accounts, map each ID to one real name in the `players`
block of the manifest:

```json
"players": {
  "MJhBZs7fJF": "Ethan",
  "mQB12J5va_": "Halim"
}
```

Every ID pointing at the same name is treated as one person everywhere —
weekly stats, settlement, and lifetime standings all merge.

---

## Recording a past settlement

To log a week you already settled by hand, add an entry to the `settlements`
array. Group transactions under a `hub` (everyone pays one person, who then
pays out) and set `"paid": true`:

```json
"settlements": [{
  "week": "2026-W22",
  "date": "2026-05-31",
  "label": "Last week — final settlement",
  "paid": true,
  "groups": [{
    "title": "SD side — everyone pays Clayton",
    "hub": "Clayton",
    "tx": [ { "from": "Ethan", "to": "Clayton", "amount": 211.92 } ]
  }]
}]
```

It shows on the Settlement tab for that week as a finished record.

---

## Legacy / historical log (stats only)

To feed older games into Players and Lifetime without running them through
settlement, drop a `ledgers/legacy.csv` with columns
`game,date,player,buy_in,buy_out,net` (one row per player per game) and point
the manifest at it:

```json
{ "legacyLog": "legacy.csv" }
```

These games are merged into Players, Lifetime, and the Game log by player name,
but are deliberately excluded from the Settlement tab (they were already
settled). Player names here should match the display names from the identity
map so the same person combines across sources. Since the log has no session
times, time-played and $/hr for these games assume 2h each.

---

## How settlement works

`net = buy_out + remaining_stack − buy_in`. A week's nets should sum to exactly
**zero**; if a file is off, the Settlement and Files tabs flag it (usually a
missing buy-out or stack). Sessions in the same Monday–Sunday week are pooled,
so a win on Tuesday cancels a loss on Friday before anyone pays.

---

## The one caveat: paid check-marks

The "paid / not paid" ticks are stored in the **browser on each device** —
a static site has no shared database. The simplest setup is to have one person
(the banker) track payments on their machine. If you want everyone to see the
same live status, that needs a tiny backend (e.g. a free Firebase project) —
the code is structured to make that easy to bolt on later.

---

## Local preview

Because the page fetches files, opening `index.html` directly with a
`file://` path is blocked by the browser. Run a local server instead:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

If no `ledgers/` data is found, the page falls back to a built-in sample night
so the interface still renders.
