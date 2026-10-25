# Wine Cellar

The wine you own, what it is worth, and **when to drink each bottle**.

**Live: https://eagleadams86.github.io/wine-cellar/**

A single-file web app in the eagleadams86 family — no build step, no account needed, works
offline. Your cellar lives in your browser; sign in with Google only if you want the same
cellar on your other devices.

## What It Does

- **Cellar tab** — tiles for bottles, value, and how many bottles are *past their window*,
  *drink soon*, *ready* and *holding*; the full cellar table, which opens in drinking order (past
  window first, then the windows that close soonest — so it doubles as the list of what to open
  next), sortable, filterable, with Copy and CSV; and a chart of how many bottles are inside their
  drinking window in each of the coming years.
- **The status tiles are filters.** Press **Past Window**, **Drink Soon**, **Ready** or
  **Holding** to show just those wines in the Cellar list, and again to take it off; several can
  be on at once, each marked with a ✓. The **Status** box in the Cellar is the same filter as a
  list of tick boxes in priority order — Past Window, Drink Soon, Ready, Hold, No Window — and
  **Clear** lets them all go. The page stays where it is when you press a tile.
- **Paid per bottle** in the wine window records what one bottle actually cost (to the cent).
  Once any wine has one, the Cellar table grows a **Paid** column that says, under each price,
  whether you paid **under value**, **in range** or **over value** against the Value Each range —
  in words, never colour. It goes into the CSV, a pasted
  list can carry a **Paid** (or Price Paid / Cost) column, and a share link carries it only if you
  tick **What you paid** in the Share window.
- **Enter saves** the wine window, Opened a Bottle, Paste a List and Check All Wines — anywhere
  but a text area (Notes, the pasted list), where Enter is a new line. On a focused button, Enter
  presses that button.
- **Date added** in the wine window starts on today for a new wine and can be changed to the
  day the bottles really arrived (never later than today). Pasted wines are added today. The
  date is optional for any wine — leave it blank, or tick **Just the year** and give only the year.
- **Came from** is your own words — a wine club, a shop, a gift — and the box offers back the
  ones you have used before. (It used to be three fixed choices; those became words.)
- **Point at a row to see the rest of the wine** — the bubble (Money Map's row tip) lists every
  field the table has no column for: colour, where it came from and when it was added. Then the
  wine's notes, if it has any; a small dot after the name still says it has notes. On a phone,
  tap the wine to see it all in its window.
- **Opened tab** — every bottle you have opened, with the date, the wine's region, grape and what
  you paid (copied when you press Opened One, so the log reads the same after the wine is edited
  or deleted; Paid shows once any bottle has one), your 1–5 rating and a note. **Find** narrows
  the list by any of those words, like the Cellar's.
  Press a line to change **anything** on it — the wine's name, vintage, region, grape, price,
  drinking window, where, date, rating and note. That changes the line only, never the wine in
  the Cellar, and the bottle count is left alone.
  Each line also says its **Timing** and **Age** (see below), and **Where** it was opened once any
  line has a place.
  Point at a line for what the list doesn't show — its drinking window, a hidden Paid or Where,
  and whether it came from your Cellar or was logged elsewhere.
- **Log a Bottle** — for a bottle opened at dinner, a winery or a tasting: type the wine, press
  **Look Up with Claude** to fill its region, grape and drinking window, and save. It goes straight
  onto the Opened tab and its dashboard without ever being in the Cellar.
- **Opened at the Right Time** — tiles at the top of the Opened tab count the bottles opened
  **Early** (before the window opened), **In Window** and **Late** (after it closed), set against
  the window the wine had on the day it was opened, plus the **Average Age** from vintage to
  opening. Press Early, In Window or Late to narrow the list to them, or tick them — and No
  Window — in the list's **Timing** box beside Find; Clear takes them off.
- **What You Like** — your opened bottles grouped by grape and by region, with how many you
  opened, how many you rated and the average stars of those, best-rated first. Press a heading to
  sort. Both cards count the whole log (Find never changes them) and hide until a bottle is opened.
  **Grouped** (the usual view) counts names written different ways together — "Stags Leap
  District, Napa Valley, CA" as Napa Valley, California, and "75% Cabernet Sauvignon, …" as
  Cabernet Sauvignon; point at a row to see what it took in. **As written** counts every spelling
  apart.
- **Consistent names from Look Up** — Claude writes a region as "Area, State or Country", and a
  grape as the single grape when it is 75% or more of a wine labelled by grape, a classic blend's
  name (**Bordeaux blend**, **Rhône blend**, **Port blend**, **Champagne blend**) — a red Bordeaux
  is a Bordeaux blend, not a "Merlot blend" — or else the lead grape and "blend", and is sent the names already in your cellar so it
  reuses your spelling. A finer sub-district goes in the wine's **Sub-region** (e.g. Stags Leap
  District), shown after the region in both lists, copied onto opened bottles, and counted in What
  You Like's own Sub-region table. Run **Check All
  Wines** to bring older wines into line — it only changes what you tick.
- **Folding cards** — At a Glance, Cellar, Drinking Windows Ahead, Opened at the Right Time,
  What You Like and Bottles Opened fold up to their heading
  (press the heading band, as in Money Map). Which ones you folded is remembered on this device.
- **When a list doesn't fit as a table** — on a phone, or a half-screen window — it turns into
  a stack of cards, one to a line so it still reads in its sort order: the wine and its status
  pill on top, the labelled fields below, each at its own width and as many to a line as
  fit, so a long region or grape gets room instead of wrapping. It switches whenever the table could not give each
  text column about two words a line, so it never scrolls sideways or squeezes into tall rows.
  This covers the Cellar, the Opened log and the paste preview. The column headings
  become a row of **Sort by** buttons. It is layout only: Copy and CSV still export the full table.
- **Opened One** and **Delete Wine** are in the wine's own window (press its row): Opened One
  takes a bottle off the count and writes a line in that log, asking first if you had changed
  something in the window without saving. Press a line on the Opened tab to change its date,
  rating or note, or to **Remove This Bottle** — which puts the bottle back on the count, so a
  mistaken press is easy to undo.
- **Paste a List** reads the table Claude writes when you ask it to look wines up — see below.
- **✦ Check All Wines** looks every wine with bottles left up with Claude at once — see below.
- **↩ Undo** (⌘Z / Ctrl+Z) walks back through this sitting's changes, one Save at a time —
  Money Map's undo, ported. Up to twenty steps; an edit, an Opened One, a pasted list, a
  restore and even Delete All Data are each one step. The button asks first, because there
  is no redo; ⌘Z goes straight back. The history lives in memory, so a reload starts it
  fresh, and it clears when another tab or another signed-in device changes the cellar —
  undoing past their work would overwrite it.
- **Accents don't matter when you search.** Every Find — the Cellar's, the Opened list's and ⌘K —
  matches "resonance" to Résonance, "coeur" to Cœur and "rebel's" to Rebel’s, in either direction.
- **Find** (⌘K), **Settings**, **Back Up** (JSON and CSV, Restore, Delete All — which keeps
  your settings, theme and Claude key), **Share**
  (a read-only link with the data inside it), four themes plus Auto — the family's chrome.
- **A switched-off button looks switched off** — Add and Look Up in the paste window before anything is pasted, Save Ticked Changes before anything is ticked, Copy Link while the share window is working: dimmed, with the "not allowed" cursor, rather than looking pressable and doing nothing. It comes from `theme.css` (pack rule 20, 2026-10-23), not this page.

## Look Up with Claude

**✦ Look Up with Claude** in the wine window and the paste window asks Claude (Opus 5.5) to search
the web for a wine and come back with its value, drinking window, region, grape and colour, plus
the pages it used.

- **It fills only what is empty.** Anything you typed or pasted stays. In the wine window each
  line of Claude's answer — value, window, region, grape, colour — has its own **Use this**, so you
  can take its value and keep your window; a line that already matches says "✓ in the box", and
  **Use all of Claude's answer** takes everything. Nothing is saved until you press Save
  or Add, and a one-line "Looked up…" note records when and on whose word.
- **It is told where the bottle came from.** What's in **Came from** goes with the question
  ("Came from, in the owner's words: Wine Access"), and Claude searches that shop first — the
  shop that sold it usually still lists it, at about what you paid. Leave Came from blank to
  send nothing.
- **A non-vintage wine is dated from its bottling.** For an NV wine the **Added** date goes too,
  as "bottled no later than", and Claude counts the drinking window from bottling rather than
  from today — a tawny Port is ready the day it is bottled and does not improve after. A bottling
  date visible in the label photo wins. Enter just the year if that is all you know.
- **It says how sure it is of the bottle** — sure of the wine, probably the right wine, or could
  not find it. That badge is only about *which* bottle; how firm the value and window are is on
  their own lines ("est.", "my estimate"), so a wine nobody sells any more can still be "Sure of
  the wine". A **Label read** badge beside it means a label photo went along and Claude read it.
  When it can't find the wine it fills nothing rather than guess, and only adds a "Looked up… ·
  Claude could not find this exact wine" note, so you can see it was tried.
- **It remembers when.** Each wine keeps the time Claude last answered for it — from this window
  or Check All Wines — saved the moment the answer arrives, even if you change nothing. The wine
  window shows it beside the button ("Checked 12 days ago"). It syncs and backs up with the wine,
  is never in a share link, and Undo leaves it alone. Renaming a wine clears it.
- **A source that names a different wine is left out.** If a page's address names a specific
  bottle (it has a vintage year in it) and shares no word with the wine looked up, it is dropped
  from the sources and the panel says so — Claude may have taken a figure from it, so check the
  value. Found on 2026-10-23: a lookup of the 2019 Albret Lastra Reserva cited a Wine Access page
  for a 2012 Bodegas Resalte de Peñafiel. A search page, a producer page or a region page stays.
- **Windows are usually Claude's estimate.** Critics' published windows are mostly behind
  paywalls, so in testing it found a named critic's window for one wine in twenty; the note says
  "estimate" when that is what it is. Values come from real listings (Wine-Searcher, merchants,
  auction results) when there are any.
- **It needs your own Claude API key** (Settings → Look Up with Claude). The key stays in that
  browser — never synced, backed up or shared — in a password field so a password manager can
  carry it. Every page on `eagleadams86.github.io` can read this browser's storage, so use a key
  made just for this app with a low monthly spend limit in the Claude Console.
- **Cost: about $0.05 a wine** with prompt caching (measured on four wines, 2026-10-25: $0.040–0.053
  and 14–17 seconds each; it was $0.137 uncached). Most of a lookup is Claude reading search results,
  and the search tool re-reads them inside one call — cached, a re-read costs 5% of the normal
  input price. The instructions every wine shares are cached too, so Check All and a pasted list
  reuse them. It is capped at 3 searches and low effort per wine; the paste window runs three lookups at a time. **Closing the window stops
  them** — Cancel, Escape or a click outside — so nothing keeps running (or billing) behind it.
  A key Claude rejects stops the whole paste at once and says why.

### Label Photos

**Add Label Photo** in the wine window takes a photo (the camera and library on a phone, a file
on a Mac); you can also drag a photo file onto the window or paste one (⌘V). **From the Photos
library on a Mac, in Chrome, export first:** in Photos choose File → Export → Export 1 Photo (JPEG)
to the Desktop, then drag that file in. Chrome can't take a photo straight from Photos — the
picker's Photos pane copies it out again each time it is opened, which Chrome refuses, and ⌘C or
a drag from Photos hands over the iPhone's HEIC original, which Chrome can't open. Adding the
photo from the iPhone works directly. The window says which of these happened.

- **Press the thumbnail to see it large.** The photo opens over the window, as big as that
  window allows while keeping the photo's own shape — a tall bottle fills the height, a wide
  shelf the width. Press anywhere, the ✕, or Escape to put it away; the window under it, and
  anything typed there, is untouched. The kept copy is about 800 px on its long side, so on a
  big screen it can look a little soft.
- **Look Up reads it.** The photo goes to Claude with the question, so it matches the exact
  producer, wine and vintage printed on the label instead of what was typed. A photo alone is
  enough: with the Wine and Vintage boxes empty, Look Up fills them from the label (a name you
  typed is kept), and the panel shows "The label reads …".
- **The label's name and vintage are offered, never forced.** When what you typed differs from the
  label — "Vocal" for a "Vöcal" label, say — the panel offers the name with its own
  **Use this** (Use All never renames a wine), and Check All offers it as a line. Check All
  starts it ticked only when it is the same name spelled the label's way. A front label that
  prints only the producer re-spells that word in YOUR name ("Vöcal Cabernet Wheeler Vineyard"),
  rather than cutting the name down to "Vöcal"; a label with words yours lacks is offered whole.
  "The same name" ignores accents, capitals and apostrophes; a longer or different name, or a
  different year, starts unticked. To type an
  accent yourself on a Mac, hold the letter key and pick it, or press Option-U then the letter
  for ö.
- **What it changes, and what it doesn't.** A photo makes *which bottle* surer — fewer typos,
  fewer wrong-wine pages. It does not make the value or the window surer: those still come from
  the web, and a label rarely prints a drinking window.
- **Cost: about a cent more a lookup.** The photo is shrunk to 1568 px on the device first —
  about 1,600 input tokens, roughly $0.006 on Opus 5.5 at full price (less once cached), so it adds
  about a cent to a lookup at most.
- **A small copy is kept with the wine** (800 px, up to about 200 KB) and shown in its window,
  with Replace and Remove. It lives in this browser's IndexedDB — not localStorage, which every
  app on this site shares — and, when signed in, in its own Firestore document beside the cellar
  (`winecellar/{uid}/photos/{wineId}`), fetched only when that wine is opened. The cellar itself
  holds only `photoAt`, the time it was taken, so a cellar full of photos never nears Firestore's
  1 MB-a-document limit.
- **Opened bottles keep one too.** Log a Bottle and an opened line's edit window have the same
  photo row and Look Up reads it the same way; **Opened One** copies the wine's photo onto the
  line, so the Opened tab still has it after the wine is deleted.
- **Never in a share link.** In a backup only if you tick **Include label photos** (the backup
  offered before Delete All always includes them).
- **Deleting waits for the next load.** Removing a photo or deleting its wine takes it away from
  the device and the cloud the next time the app opens, because Undo can bring the wine back
  until then.

### Check All Wines

**✦ Check All Wines** under the Cellar table runs the same lookup on every wine with bottles
left (finished wines are skipped), three at a time. A wine with a label photo sends its kept
copy with the question, as Look Up does, so the label settles which bottle it is (about a cent
more for each such wine; the cost shown up front doesn't count it).

- **You choose which wines.** Every wine has a box and says when it was last checked. The window
  opens with the wines **not checked in the last 90 days** selected; **Select Not Checked In**
  with its menu (30 days, 90 days, 6 months, a year) picks again, and **Select All** /
  **Select None** do what they say. Once a wine's answer arrives its box goes away until the
  check ends — the only tick boxes left in its panel are the changes to save — and the Select
  buttons skip it. A wine whose lookup failed keeps its box, still selected, to try again.
- **It says the cost first** — how many selected wines, about how many dollars and minutes — and
  looks nothing up until you press **Check N Wines** (it can't be pressed with none selected).
  **Stop Checking** keeps what has come back.
- **Then you choose, line by line.** Each wine gets a panel with one ticked line per thing Claude
  says differently, each saying **why** it starts ticked or not. The rule is better evidence wins:
  - **Ticked:** an empty box; a value from current prices (over an estimate or older prices — even
    the same numbers, which clears your "estimate" box); a
    critic's published window over an estimate; your region or grape rewritten the usual way
    when it is the same place or grape ("Stags Leap District, Napa Valley, CA" → "Napa Valley,
    California").
  - **Left unticked:** an estimate replacing real prices or a published window; one estimate
    replacing another; a window you typed yourself; a region or grape that means something else.
    Colour is never pre-ticked.
  Any line can still be ticked or unticked by hand. **Tick All** ticks every line on offer and then becomes **Untick All**. A wine
  Claude could not find offers only its "could not find" note line, ticked, and one that already matches says so.
- **Save Ticked Changes** appears once something is ticked, and saves it all at once, so one **Undo** takes the whole check back. Each
  changed wine's "Looked up…" note line is replaced with today's, not stacked.
- **Closing the window ends the check** — Save, Cancel, Escape or a click outside. The lookups
  stop, and the answers and anything ticked but not saved go with them, so reopening starts fresh.
  That never re-bills a wine by itself: each one checked today opens unselected. A key Claude
  rejects stops the check at once and says why.

Only the wine's vintage, name, region and grape are sent, and only when you press the button.
The page calls `https://api.anthropic.com` directly — the one host added to its security policy
for this, as a data call only, never a script.

## How a Status Is Decided

Nothing is typed in: every status is worked out from the wine's drinking window and the
current year, so the cellar stays right as time passes.

| Status | Means |
|---|---|
| **Hold** | the window hasn't opened yet |
| **Ready** | inside the window, with time to spare |
| **Drink Soon** | inside the window, and it closes within the time set in Settings (a year by default) |
| **Past Window** | the window closed in an earlier year — the one to open next |
| **No Window** | none recorded |

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

Each wine starts with its vintage (`2019 Sample Hill Reserva`, or `NV …`). The parser reads:

- values like `$40-$70`, `$1,200–$1,500`, `~$25–35 (estimate)`, `$20-$25 est.` (both keep the *estimate*
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
value, window, region or grape) rather than making a second row — and so does a wine listed
twice in the same paste, which the preview marks *Adds to the same wine above*. You see every
line before anything is added.

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
or the cloud can never plant anything the app did not expect. Notes are capped at 600
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

Open `http://localhost:8025/` for the app and `/tests.html` for the suite (202 tests; the
`EXPECTED` floor in the file fails the run if any go missing). CI runs the same page headless
on every push, plus CodeQL.

Started from the family starter (`~/claude-starter`) on 2026-10-23.
