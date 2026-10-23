# Wine Cellar

The wine you own, what it is worth, and **when to drink each bottle**.

**Live: https://eagleadams86.github.io/wine-cellar/**

A single-file web app in the eagleadams86 family — no build step, no account needed, works
offline. Your cellar lives in your browser; sign in with Google only if you want the same
cellar on your other devices.

## What It Does

- **Cellar tab** — tiles for bottles, value, and how many bottles are *drink soon*, *ready*
  and *holding*; a **Drink Next** list of what to open first; the full cellar table (sortable,
  filterable by status, with Copy and CSV); and a chart of how many bottles are inside their
  drinking window in each of the coming years.
- **Opened tab** — every bottle you have opened, with the date, your 1–5 rating and a note.
- **On a phone** every list — Drink Next, the Cellar, the Opened log and the paste preview —
  turns into a stack of cards: the wine and its status pill on top, two labelled fields to a
  line, and the row's buttons at the bottom. The column headings become a row of **Sort by**
  buttons. It is layout only: Copy and CSV still export the full table.
- **Opened One** beside each wine takes a bottle off the count and writes a line in that log.
  Removing a line puts the bottle back, so a mistaken press is easy to undo.
- **Paste a List** reads the table Claude writes when you ask it to look wines up — see below.
- **↩ Undo** (⌘Z / Ctrl+Z) walks back through this sitting's changes, one Save at a time —
  Money Map's undo, ported. Up to twenty steps; an edit, an Opened One, a pasted list, a
  restore and even Delete All Data are each one step. The button asks first, because there
  is no redo; ⌘Z goes straight back. The history lives in memory, so a reload starts it
  fresh, and it clears when another tab or another signed-in device changes the cellar —
  undoing past their work would overwrite it.
- **Find** (⌘K), **Settings**, **Back Up** (JSON and CSV, Restore, Delete All), **Share**
  (a read-only link with the data inside it), four themes plus Auto — the family's chrome.

## How a Status Is Decided

Nothing is typed in: every status is worked out from the wine's drinking window and the
current year, so the cellar stays right as time passes.

| Status | Means |
|---|---|
| **Hold** | the window hasn't opened yet |
| **Ready** | inside the window, with time to spare |
| **Drink soon** | inside the window, and it closes within the time set in Settings (a year by default) |
| **Past window** | the window closed in an earlier year — the one to open next |
| **No window** | none recorded |

The tiles count **bottles you still have**. Value is per-bottle value × count, shown as a
range; wines with no value are left out and the page says how many bottles that is.

## Adding Wines From a List

Press **Paste a List** and paste one wine per line, straight from a chat or a spreadsheet.
Two shapes are understood:

- **A list with a header row** — CSV (like the Cellar table's own ⬇ CSV), tab-separated or a
  Markdown table. Columns are found by name, in any order: **Wine**, Vintage, Region, Grape,
  **Bottles**, **Value** (or Value Each / Price), **Window**, **Where**, Colour, Notes. Other
  columns (Status, Verdict) are read past. Quoted cells may contain commas.
- **Claude's lookup table** with no header needed — **Wine · Approx. value · Drink window ·
  Verdict**, in that order, tab-separated, Markdown, or lined up with spaces.

Each wine starts with its vintage (`2019 Albret Lastra Reserva`, or `NV …`). The parser reads:

- values like `$40-$70`, `~$25–35 (estimate)`, `$20-$25 est.` (both keep the *estimate*
  flag) and `~$36 list, ~$20–27 street` (the street price wins);
- windows like `2027–2034`, `2026-2026`, `Now–2031` (Now = this year), `2030+`, `by 2029`;
  an empty window stays empty (*No window*);
- the bottle count from a Bottles column, or `x2` / `(3 bottles)` in the wine cell; one
  bottle otherwise;
- a Where column per wine — the box in the window fills in only where the list says nothing;
- the colour from a Colour column, or guessed from the grape and name: Port, Tawny, Sherry
  → dessert; Champagne, Brut, Prosecco → sparkling; rosé → rosé; Chardonnay, Sauvignon
  Blanc, Riesling and other white grapes → white; everything else red. The preview shows the
  guess so it can be checked.

A Status or Verdict column is never stored — the status is worked out from the window. The
same vintage and name pasted again adds bottles to the wine you have (filling in a missing
value, window, region or grape) rather than making a second row. You see every line before
anything is added.

## Cloud Sync

Sign-in uses Google Identity Services (a popup straight to `accounts.google.com`), and the
cellar is one Firestore document per Google account: `winecellar/{uid}` holding
`{ state, updatedAt }`. The module is Golf Handicap's, ported whole — including the two rules
the family learned the hard way: the first sign-in with data on both sides **asks** which copy
to keep, and an empty copy never overwrites one with data in it. Nothing from Google is
fetched until you reach for the sync button (or were already signed in on this device).

Firebase project **`winecellar-fa733`** (free Spark plan). Set up 2026-10-23:

- Authentication: Google provider on; `eagleadams86.github.io` in Authorized domains.
- Firestore: `(default)` database in `nam5`; rules as in [`firestore.rules`](firestore.rules)
  — each account may read and write only its own document. Nothing here deploys them; if the
  console rules change, change that file to match.
- Google Cloud → Credentials → *Web client (auto created by Google Service)*: Authorized
  JavaScript origins include `https://eagleadams86.github.io` and `http://localhost:8025`. That
  client ID is `GOOGLE_CLIENT_ID` in `index.html`; serving locally on any other port means
  sign-in only works in production.

The config in `index.html` is Firebase's public client config, not a secret — access is
enforced by the rules. An anonymous probe should be refused:

```bash
curl -s "https://firestore.googleapis.com/v1/projects/winecellar-fa733/databases/(default)/documents/winecellar/probe"
```

(`PERMISSION_DENIED` is right. A probe can't tell correct per-account rules from locked ones —
only a real sign-in can.)

## Privacy and Security

Everything stays in this browser unless you sign in. See [privacy.html](privacy.html). Like
every page on `eagleadams86.github.io`, it ships a Content-Security-Policy and loads no
third-party script except Google's sync code, which is fetched only for sync. Every field is
type-checked and capped at the data boundary (`normalizeState`), so a backup, a share link
or the cloud can never plant anything the app did not expect. Notes are capped at 300
characters, opened-bottle notes at 140.

## Files

| File | What it is |
|---|---|
| `index.html` | The app — markup, styles, logic and the sync module |
| `theme.css` | The family palette, a byte-copy of `~/claude-theme-pack` — never edit it here |
| `tests.html` | The suite: loads the real app in a hidden frame; localhost only |
| `privacy.html` | The privacy policy |
| `sw.js`, `sw-kill.js` | Offline shell (network-first) and its escape hatch |
| `manifest.webmanifest`, `icon-*.png`, `favicon.ico` | Install as an app; `make_icons.py` draws the glass |
| `firestore.rules` | Checked-in copy of the deployed Firestore rules |

## Running It Locally

```bash
python3 -m http.server 8025
```

Open `http://localhost:8025/` for the app and `/tests.html` for the suite (178 tests; the
`EXPECTED` floor in the file fails the run if any go missing). CI runs the same page headless
on every push, plus CodeQL.

Started from the family starter (`~/claude-starter`) on 2026-10-23.
