# CLAUDE.md — Wine Cellar

A personal wine-collection tracker, built from `~/claude-starter` on 2026-10-23. The
family rules in `~/.claude/CLAUDE.md` all apply (CSP, no third-party script, escape and
sanitize at every boundary, the theme pack). This file records what this app decides that
the family doesn't.

## Shape

- **Dev port 8025** — in both `.claude/launch.json` and `~/.claude/launch.json`.
- Storage keys `wine-state`, `wine-theme`, `wine-tab`, `wine-folded`, `wine-updated`, `wine-sync-live`,
  `wine-sync-uid`; service-worker cache prefix `wine-shell-`.
- State is `{ version, wines[], opened[], settings }`. `normalizeState()` is the allowlist:
  **a new stored field goes there and SCHEMA is bumped in the same commit**, or it is
  stripped. Tests pin the exact key lists of a wine and an opened bottle.
- Absent is not zero: `cleanYear` / `cleanMoney` test `== null` / `''` before `Number()`,
  because `Number(null)` is 0. A missing value or window stays `null` — never `undefined`
  (Firestore refuses a whole document over one).
- `opened[].label` snapshots the wine's name at the moment it was opened, so the log still
  reads after the wine is deleted — and since SCHEMA 5 (2026-10-24) so do `region`, `grape` and
  `paid`, written by Opened One. A line LACKING them (`!('region' in it)`) is filled from its
  wine inside normalizeState, which covers old saves, backups, the cloud copy and share links
  (the share payload's opened map does not carry them; the receiver backfills from the shared
  wine, and `paid` only travels when What you paid is ticked). A line that has one, even blank,
  keeps it. The Opened list's Paid column hides while no line has a price, like the Cellar's;
  its Find (`#openedFind`) is page-life only. Removing a log line puts the bottle back on the count.
- SCHEMA 6 (2026-10-24) adds `vintage`, `drinkFrom`, `drinkTo` to an opened line by the same
  snapshot-and-backfill rule; a line whose wine is gone reads its vintage off the label's leading
  year (`wineLabel`'s "2010 …" / "NV …" shape). The share payload still omits them — the
  receiver backfills. **Timing (`timingOf`: early / inwin / late / none) and age at opening are
  derived, never stored**, against the line's OWN window, so editing a wine never rewrites history.
- The Opened tab's dashboard (`#timingCard`, `#likeCard`) reads only `computeOpened()`, over the
  WHOLE log. Timing tiles toggle `timingPick` (page-life, like `statusPick`; Clear empties it too);
  No Window has no tile, the warn line counts it. What You Like groups grape/region case-blind
  (`likeGroups`), averages rated bottles only, counts blanks aloud, and never reads notes.

## Decisions — Don't Undo

- **Status is derived, never stored** (`statusOf(w, year, soonYears)`). The verdict column in
  a pasted Claude table is read past on purpose: a verdict is right for one year, a window
  for all of them.
- **Counts are bottles, not wines**, and only bottles still owned. Finished wines (qty 0)
  stay on record, hidden unless Settings → *Show wines with no bottles left*, and are never in
  a figure.
- **Free-text notes are allowed here** — this is a personal app, not one of the two work apps
  — but capped (300 / 140) and single-line where the field is single-line.
- **Wine Access's descriptions are their copyrighted text.** Notes seeded from them are short
  summaries in our own words, never pasted copy.
- **Charles's real bottles are never committed** — the repo is public. They go in
  `my-shipment-*.json` (gitignored) and are loaded through Back Up → Restore, or pasted.
- **Sync is Golf Handicap's module, ported whole** (GIS sign-in, `shouldBootSync`, the push
  gate, `syncDecision`, `forCloud`, `describeSyncError`, the relisten timer). A fix to sync
  starts in Golf Handicap or is mirrored there. The CSP's external hosts are exactly Golf's
  list, and a test pins them.
- **Paste reads by header NAME when there is a header row** (`headerMap`), else positionally
  as Claude's lookup table (wine, value, window). The separator is decided ONCE from the first
  line — a comma inside Claude's value cell must not turn its space-aligned table into a CSV.
  Test fixtures for paste are invented wines; Charles's real list never goes in the repo.
- **A focused number box ignores the wheel** — one capture-phase `wheel` listener blurs it.
  Chrome steps a focused `type=number` on every wheel/trackpad tick over it; scrolling the tall
  editor to reach Save turned Bottles 2 into 0 (2026-10-23). Every year and price box is a
  number box too. And taking a wine to 0 bottles in the editor asks first, because 0 hides it.
- **Select-on-focus never cancels the mouse release on a number box.** Chrome stops a spin
  arrow repeating only on the release; cancelling it made ONE click on Bottles run 1 → 109 in
  three seconds, and a click on the down arrow ran it to 0 — Charles's actual bug, 2026-10-23.
  The starter and its siblings carry the same handler; check them before trusting a number box.
  The FIRST fix (number boxes skip the cancel) was not enough: the flag saying "this focus came
  from the mouse" was a bare boolean nothing reset, so a click on a table ROW armed the cancel,
  the editor's first box took the focus, and the cancel ate the Bottles arrow's release. It is
  now `pressedOn` (the element pressed, cleared on pointerup), and the cancel fires only on a
  release over the box it was armed for.
- **A `table.cards` that doesn't fit is a stack of cards** — CSS keyed on `.cardview`, which
  `fitCards()` sets (2026-10-24; it was a fixed 640px media query before, and a half-screen
  window got a table squeezed into tall rows with the × cut off). "Fits" = laid out with every
  TEXT column at least 10rem (`.fitprobe`; numbers, nowrap cells, the pill and the buttons
  excluded), no wider than its box. A MutationObserver on each host (`CARD_HOSTS`) refits a
  replaced table before paint; a ResizeObserver refits on a WIDTH change, a frame later (the
  switch changes the host's height, which would loop). A hidden box (zero width) is decided when
  it shows. A new card table must sit in a host listed in `CARD_HOSTS`. **One card to a line at
  every width** — Charles, for the sort order; a 2-up grid was tried and rejected. A wide card
  spends its width on fields: `flex: 0 1 auto`, each field its own one-line width, packed with a
  22px gap (equal 7.5rem shares wrapped a long region to four lines beside a Bottles holding
  "1"; letting fields GROW left holes before the next label — both rejected 2026-10-24), and
  an empty full-width `tr::before` breaks the line after the name and pill so fields start in the
  same place on every card. Each `<td>` carries `data-label` (the card shows it via `::before`,
  so exports never see it) — a new column needs one, and a test checks every cell. The name
  `<th>` leads (`order: -2`), the Status pill rides its line (`order: -1`), sortable headings
  become Sort-by chips.
- **Undo is Money Map's in-memory ring** (`undoRing`/`undoSnapshot`, cap 20; `pushSnapshot`
  and `undoBankable` pure and pinned). save() banks the state it REPLACES unless the save is
  itself an undo (the `undoing` latch) or nothing changed (`coreOf`, which ignores `version`).
  Every window here saves once, so one window is one step without Money Map's per-field
  latch — keep it that way: a window that saved per field would need that latch.
  `undoStartOver()` runs at boot, in the storage listener and in `wcAdopt`: a step back past
  another tab's or device's work would overwrite it. Never persisted, no redo; the button
  confirms, ⌘Z doesn't and is gated off fields and open windows. Delete All's and a wine
  delete's wording says Undo brings it back THIS SITTING — keep the words true.
- **Look Up with Claude** (2026-10-23): the page calls `api.anthropic.com/v1/messages` with raw
  `fetch` (no SDK — no build step, no third-party script), the reader's own key from
  `localStorage['wine-claude-key']` (NEVER `state`), and `anthropic-dangerous-direct-browser-access`.
  Opus 5.5, effort `low`, web search `max_uses: 3`, `fallbacks: "default"`; the answer comes back
  through ONE strict tool, `record_wine` (tool_choice `auto` — forced is a 400 on this model), and
  `readLookup()` is the boundary (http(s)-only sources). `pause_turn` resumes with no added
  message; a silent finish is nudged once. Lookups fill EMPTY boxes only. Measured on Charles's 20
  wines: $0.144 a wine, ~36 s; Sonnet 5.5 was $0.065 and nearly as good — Charles chose Opus.
  0 of 20 windows came from a named critic, so the note line says "estimate". The terminal test
  harness bridges fetch to Node so the key (`~/.config/anthropic_winecellar_key`) never enters the
  page. privacy.html says what is sent; keep it true if the request changes. **Closing either
  window stops its lookups** — a `MutationObserver` on `open`, not the `close` event (which some
  engines never deliver), because Escape and a click outside never pass through Cancel; before
  2026-10-23 a closed paste window went on billing through every row.
- **Enter saves** through ONE document listener and the `ENTER_SAVES` map (dialog id → its save
  button). A window that gains a save button goes in the map; Settings (saves as you go) and
  Delete All (never one key away) stay out on purpose. Skipped: textarea, a focused
  button/link (it presses itself), the key `<form>`, IME composition, a hidden/disabled save.
- **`paid`** (SCHEMA 3 / SHARE_PAYLOAD_V 3, 2026-10-23): one bottle's real cost, `cleanMoney`,
  nullable, cents kept (`fmtPaid`). `dealOf()` sets it against the value range (ends inclusive)
  as words — under value / in range / over value — never a colour (red-green colourblind). The
  Paid column HIDES while no wine has a price paid, so column counts and `shown` keys come from
  the rendered `cols`, not `COLUMNS`. Shares only with `sharePaid` ticked (off by default).
  The estimate box was kept on purpose: a deal against a guessed range is shakier.
- **`added` is a day, a bare year or ''** (SCHEMA 2 / SHARE_PAYLOAD_V 2, 2026-10-23): the last
  two only for "Already in the cellar", chosen in the wine window by leaving the date blank or
  ticking Just the year (`dressAdded()`). `cleanAdded()` is the boundary — an ABSENT field is
  still today, so old records are unchanged; `fmtAdded()` is the one way to print it. Any
  other source requires the day, and switching to one puts today in an empty box.
- **Wrong-wine sources** (2026-10-23): `readLookup(found, asked)` drops a source whose PATH has a
  vintage year and shares no distinctive word (`wineWords`, minus `SOURCE_COMMON`) with the asked
  name or Claude's `matched_wine`; `r.dropped` is said on screen (`droppedLine`). Every caller passes
  the asked wine. Kept narrow on purpose: on the 58 real pages from that day's re-runs it dropped
  exactly the one wrong page. It cannot undo a figure Claude took from that page.
- **Check All Wines** (2026-10-23) reuses that lookup across every wine with `qty > 0`.
  `lookupDiffs`/`applyLookupTo` are the saved-wine twins of `LOOKUP_BOXES`/`applyLookup` (those
  read the FORM; these read a wine) — change a field's rule in both. Pre-tick rule: an EMPTY
  field starts ticked, a replacement unticked, colour never. Answers live in `checkFound`
  (wine id → `{key: wineKey, r}`) until the window closes ANY way — `checkEnd()` (from the
  `open`-attribute observer) clears answers, failures, ticks and picks, so reopening starts fresh
  (2026-10-24; before, they lived for the page's life). `checkedAt` + the not-checked-lately
  default selection are what stop a re-bill now. Ticks live in
  `checkTicks` so a re-render keeps them, and an arriving answer redraws only its own panel.
  Save is one `save()` = one Undo step; it re-reads each wine by id at save time. `__plant`
  clears all three maps.
- **`checkedAt`** (SCHEMA 4, 2026-10-24): when Claude last ANSWERED for a wine — an exact UTC
  moment (`nowStamp()`, built from UTC parts because `toISOString` is banned by a test) or null;
  `cleanChecked()` is the boundary (strict shape, not over a day in the future). Charles chose to
  stamp **on the answer, not on Save** (`stampChecked(id)`, its own `save()`): the check was paid
  for, and a stray close must not make the wine look unchecked and re-bill it. So `coreOf()`
  ignores it (a stamp mints no Undo step) and `undoLast()` keeps the later of the two times. A
  failed lookup is not a check; a low-confidence answer is. Not in share links
  (`buildSharePayload` is a whitelist — SHARE_PAYLOAD_V stays 3). The editor's Look Up stamps at
  once only when the form's name matches the saved wine (`wineKey`); otherwise `lookupStamp`
  waits for Save, and Save clears the stamp on a renamed wine. `fmtChecked()` is the one printer.
- **Check All picks vs ticks:** `checkPicks` (wine ids, "Select …" buttons, `data-pick`) say what
  Start looks up; `checkTicks` (`data-tick`, Tick All) say what Save changes. Never share the
  words. Picks fill on open from `#checkSince` (not checked in N days, plus this sitting's
  failures — page-life only, not `state`); a wine leaves the picks when its answer arrives.
  A wine WITH an answer has no pick box at all (`checkSelectable()`), and every Select button and
  the count skip it — a hidden box Select All could tick would re-bill invisibly. A failed lookup
  has no answer, so it keeps its box.
- **Not found still leaves a note** (2026-10-24): `lookupNoteLine` says "Claude could not find
  this exact wine" for `low`, with no sources; `putLookupNote()` is the ONE rule (replace an older
  "Looked up" line, else append, only within 300) for the editor, Check All (a pre-ticked `note`
  pseudo-line from `lookupDiffs`) and paste.
- **An installed iPhone draws under the status bar** (black-translucent), so the header pads by
  `env(safe-area-inset-top)`, a fixed `body::before` band in `--surface` keeps scrolled content
  out from under the clock (the header is not sticky on a phone), and a dialog's max-height takes
  the larger inset off BOTH ends (in `vh` — `dvh` is 0 in some embedded engines). And `openModal()` sends focus to the dialog on touch
  for anything `opensOnFocus()` — a `<select>` too, because iOS 26 opens a focused select's menu
  at once. All three found on Charles's iPhone, 2026-10-23.
- **Tests that stub storage stub `W.Storage.prototype.setItem`**, never
  `localStorage.setItem = …`, which Chrome stores as an item named "setItem" while the real
  write still happens.
- **Notes are the ROW's hover text** (Money Map's `rowTip` + `noteDot` + `wireGridTip`, as
  `#rowTip` wired once on `<main>`): `data-tip` on the `<tr>`, never a `title`; a dot after the
  name, behind a U+2060 word joiner that `cellText()` strips. The dot goes INSIDE the name button
  and in a `.nameend` nowrap span with the last word (`wineNameCell()`): after the button it
  wrapped alone (a button is an atomic box), and the joiner alone still let it go 57 times in a
  940-case width sweep. Glued: 0. Touch shows no tip on a row that
  opens the editor. Since 2026-10-23 EVERY row has a tip: `wineFacts(w, shown)` lists each field
  the table has no column for (`shown` = its column keys), then
  the notes, then "Click to edit" LAST (moved from the top 2026-10-24; pointer-only, the name
  button is the keyboard path). A new column must be added to that table's keys, or the tip repeats it. The dot
  still means notes only.
- **No Drink Next; the status tiles ARE the filter** (2026-10-24, Charles): the Cellar's default
  order is already drinking order, so the Drink Next card went (an old `next` in `wine-folded` is
  inert). At a Glance is Bottles · Value · Past Window · Drink Soon · Ready · Holding — Past has
  its own `.stat.err` tile (Drink Soon no longer adds it in) and Opened left the row (still six
  tiles, so the tile-row table is unchanged). The four status tiles are `button.stat`
  `aria-pressed` toggles; the Cellar's Status box is a `<details>` of tick boxes in `STATUS` rank
  order. Both press ONE `statusPick` Set through `setStatusPick()`, which redraws the list and
  repaints both controls — never the tiles' markup, or the pressed tile loses focus. A tile press
  unfolds a folded Cellar and holds the page still (a scroll to the Cellar was considered and put
  aside). Page-life only, like the select it replaced.
- **Folding is device-local** (`wine-folded`, a list of `data-fold` names), not in `state` as
  Money Map keeps it: it is screen layout, so it neither syncs nor rides in a share link, and
  needs no SCHEMA bump. The full-screen chart button stays a child of `#chartCard` (it is
  positioned against the card) and `maxiReady()` takes it away while the card is folded.
- **Opened One and Delete are in the wine window, not on the row** (2026-10-24, Charles): the
  whole row opens the window, so the Cellar table has no actions column. Both show only for a
  SAVED wine; Opened One is disabled at 0 bottles. Opened One CLOSES the wine window first — left
  open behind, its Bottles box would hold the old count and its Save would put the bottle back —
  and asks first if the boxes differ from when it opened (`wineFormSnap`/`wineFormAtOpen`). The
  Opened tab followed the same day: **Remove This Bottle** (`#openRemove`) is in the line's edit
  window, shown only when editing a line, and no table has an actions column (`td.rowacts` is
  gone, with its CSS).
- **An opened line is edited in the Opened One window** (`openBottleEdit`, `editingBottleId`):
  date, rating and note only, never the count and never the label (the name as it was).
- **A wine name wraps and carries no underline** (2026-10-24): `.rowbtn` overrides the base
  button rule's `nowrap` — before that the Wine column alone never wrapped, squeezed every other
  column and pushed the × off the edge. No underline because the whole ROW is the target (it
  hovers and opens as one); keyboard focus keeps the ring. A test pins both.
- **The table's own order is drinking order** — past the window, drink soon, ready, hold,
  none; then the window that closes soonest. The Wine column sorts by name, then vintage.

## Firebase

Project `winecellar-fa733` (Spark). Google sign-in on; `eagleadams86.github.io` authorized;
Firestore `(default)` in `nam5` with per-uid rules on `winecellar/{uid}` (see
`firestore.rules`); the auto-created web OAuth client allows `https://eagleadams86.github.io`
and `http://localhost:8025`. The console rules were typed as one line — same meaning as the
commented copy in the repo.

## Tests

`tests.html`, `EXPECTED = 245`. Run it headless (Playwright) rather than in the Browser pane:
the pane throttles hidden timers and its focus heuristics turn the 705px keyboard-focus check
red on untouched code.
