# CLAUDE.md — Wine Cellar

A personal wine-collection tracker, built from `~/claude-starter` on 2026-10-23. The
family rules in `~/.claude/CLAUDE.md` all apply (CSP, no third-party script, escape and
sanitize at every boundary, the theme pack). This file records what this app decides that
the family doesn't.

## Shape

- **Dev port 8025** — in both `.claude/launch.json` and `~/.claude/launch.json`.
- Storage keys `wine-state`, `wine-theme`, `wine-tab`, `wine-folded`, `wine-updated`, `wine-sync-live`,
  `wine-sync-uid`, `wine-cloud-base`, `wine-cloud-sent`, `wine-cloud-base-copy`; service-worker cache prefix `wine-shell-`.
- State is `{ version, wines[], opened[], pairings[], settings }`. `normalizeState()` is the allowlist:
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
  its Find (`#openedFind`) is page-life only. Removing a log line puts the bottle back on the count,
  and a pairing had with that line (`openedId`) goes back to `planned` with `openedId: null` and a
  `back` History entry — the confirm says so first (2026-10-26 review). The Opened tab's Wine sort
  is by the label's NAME, then vintage — `compareWines`' rule.
- SCHEMA 6 (2026-10-24) adds `vintage`, `drinkFrom`, `drinkTo` to an opened line by the same
  snapshot-and-backfill rule; a line whose wine is gone reads its vintage off the label's leading
  year (`wineLabel`'s "2010 …" / "NV …" shape). The share payload still omits them — the
  receiver backfills. **Timing (`timingOf`: early / inwin / late / none) and age at opening are
  derived, never stored**, against the line's OWN window, so editing a wine never rewrites history.
- The Opened tab's dashboard (`#timingCard`, `#likeCard`) reads only `computeOpened()`, over the
  WHOLE log. Timing tiles toggle `timingPick` (page-life, like `statusPick`; Clear empties it too);
  No Window has no tile, the warn line counts it. What You Like groups grape/region case-blind
  (`likeGroups`), averages rated bottles only, counts blanks aloud, and never reads notes.
- SCHEMA 7 / SHARE_PAYLOAD_V 4 (2026-10-24): `place` (Where, 40 chars) on an opened line. The share
  payload's opened map now carries the line's own snapshot (region, grape, vintage, window; place
  with the notes; paid when ticked), because a logged bottle has no wine to backfill from.
- **`#openDialog` has three modes** (`dressOpenDialog`): Opened One (`drinkingId`, wine boxes
  hidden, copies the wine), Edit (`editingBottleId`, every box) and Log Bottle (`loggingBottle`,
  `wineId: null`, no count). Edit and Log rebuild `label` from the name + vintage boxes. **A line
  edit never touches a Cellar wine** (Charles, 2026-10-24). Its Look Up (`obLookupBtn`) reuses
  `lookupWine`/`readLookup`, fills only empty region/grape/window boxes and offers Claude's for the rest.
- **Lookup naming** (2026-10-24): `record_wine` pins region to "Area, State or Country" and grape to
  the lead grape / "X blend"; `appellation` (strict ⇒ in `required`) carries the sub-district into
  the note line. `lookupWine` sends `knownNames(state)` — only names ALREADY in the house shape, so
  the old mess is never offered back. A **statewide/country-wide wine** ("California" on a Josh
  Cellars label) has that state ALONE as its region: the schema says so, `readLookup` moves a
  lone appellation UP when region came back empty, and `knownNames` accepts a bare US state. privacy.html says these names are sent; keep them in step.
- **The grouping tables count names AS WRITTEN** (trimmed, case-blind) — the Grouped / As written
  switch was REMOVED 2026-10-31 (Charles: on his cellar Grouped only stripped "blend", putting a
  Tempranillo blend in with Tempranillo; Look Up already writes tidy names). Gone with it:
  `likeGrouped`, `setLikeView`, the `grouped` argument of facetName/facetKey/facetMatch/likeGroups/
  makeupGroups/computeOpened/computeMakeup/computePairings, the "Grouped from" row tip, `.likeview`.
  `groupedRegion` / `groupedGrape` / `CLASSIC_BLENDS` STAY — Check Wines' `tickRule` uses them to tell
  "the same name, written another way" from a different one. A stale `wine-like-view` key may sit
  in old browsers; nothing reads it. Don't bring a Grouped view back.
- **SCHEMA 8 / SHARE_PAYLOAD_V 5: `subregion`** on a wine and (snapshot) an opened line. Look Up's
  `appellation` becomes `r.subregion`; it is a LOOKUP_FIELDS/LOOKUP_BOXES row, NOT in the note line
  any more (and since SCHEMA 12 there is no note line). A wine with no `subregion` key has the old second part of its Looked up line moved in
  (`subregionFromNotes`, guarded against window/could-not-find/hosts parts). What You Like's
  Sub-region table is never folded.
- **Check Wines' default ticks are `tickRule`** (Charles, 2026-10-24: "an estimate overwriting an
  actual" must not be pre-ticked). Better evidence wins: listings (`!r.valueEst`) tick; a published
  window ticks only over YOUR estimate. "Estimate" is `ESTIMATE_WORDS` on the basis text; yours is
  read off History by `windowBasisOf` (the newest entry whose `c` changed the window: its `wb`, else
  null) — null means typed by hand, never overwritten unasked. Each line shows its `why`. Don't go back to "only empty boxes start ticked".
- **SCHEMA 10: `photoAt` on an opened line too** (2026-10-24, Charles: the Opened tab had no
  photo). `photoRow(ids)` is the ONE photo row, made twice — `winePhoto` and `bottlePhoto`
  (`ob_photo*`, in `#openWineBox`, so Opened One's mode hides it). Opened One COPIES its wine's
  photo to the line under the line's own id (same snapshot rule as label and window), so the
  log keeps it after the wine goes. Every photo walk goes through `photoRecs(st)` (wines +
  opened): bookkeeping, sweep, backup, restore, flush, Delete All's wording. A new record kind
  that keeps a photo goes there. The opened Look Up sends the photo and fills an EMPTY name and
  vintage from the label, like the wine window. Share payload's opened map is a whitelist — no photoAt.
- **SCHEMA 9: `photoAt`** (2026-10-24) — when a wine's label photo was taken (`cleanChecked`
  shape), or null. **The photo is never in `state`**: a JPEG data URL (`cleanPhoto`, ≤300 000
  chars; the sharp copy SENT to Claude has its own `PHOTO_SEND_MAX`) in IndexedDB `wine-photos`
  (never localStorage — the origin's 5 MB is shared by every app) and in
  `winecellar/{uid}/photos/{wineId}` = `{ data, at }`, its own Firestore doc because the cellar doc
  is capped at 1 MB. `firestore.rules` has the sub-match — the parent match does NOT cover it
  (published to the console 2026-10-24; a refused write leaves the photo local and on
  `wine-photo-unsent`). Deletes are DEFERRED: `save()` → `photoBookkeeping` puts a
  wine that lost its photo on `wine-photo-gone`, and only the list as it stood at boot
  (`photoGoneAtBoot`) is deleted — Undo can bring a wine back until a reload. `sweepPhotos()` runs
  after the reveal, never in a shared view or the harness (`inTests()`). **With sync on, the local
  copy sweep (`sweepPhotoCopies`) and the cloud deletes (`deleteGonePhotos`) wait for the sync
  verdict and re-read the state it left** — a newer cloud copy can bring a deleted wine back, photo
  and all (2026-10-26 review). The queues (`flushPhotoQueue`, `deleteGonePhotos`) live in the MAIN
  script and take a `cloud` of `{ send, remove }` from the module, so the suite drives them with a
  fake; a photo retaken mid-send stays on `wine-photo-unsent` and goes round again. Not in share links
  (whitelist); in a backup with **Include label photos**, ticked by default since 2026-10-25 (`photos` map, restored by
  `backupPhotoEntries`). Tests swap the store with `__photoStore` — the suite never writes the disk.
  `accept` on the file input lists types: `image/*` opened a "comment" for the tests' comment
  stripper and failed three source tests. It leaves HEIC OUT on purpose: Chrome cannot decode
  HEIC at all and Claude's API takes none, so the picker is asked for a compatible copy, and a
  HEIC that still arrives (Charles's first try, from Photos in Chrome on the Mac) gets
  `HEIC_HELP` — `isHeic()` reads the `ftyp` brand, then the type and name. `readPicked()` READS
  before it decodes: the Mac picker's Photos pane exports into a temp `uuid=` folder and RE-EXPORTS
  on every open, and Chrome refuses a file that changed after the pick — so a Photos pick can
  NEVER be read in Chrome, picked once or twice (Charles, 2026-10-24). That gets
  `PHOTO_REPICK_HELP`. Paste and drop are wired on `#wineDialog` (`imageFromTransfer()` →
  `takePhotoFile()`), but **Photos' ⌘C and a drag from Photos both hand Chrome the original HEIC
  FILE** (Charles, 2026-10-24), and with a file URL on the pasteboard Chrome ignores any picture
  beside it — `navigator.clipboard.read()` came back EMPTY with file-url + TIFF (measured; a
  clipboard-picture fallback was built and removed). So from the Photos library, Chrome needs a
  JPEG exported to a real folder (File → Export → Export 1 Photo); both help texts say so. Only
  pure image data (no file URL) pastes as a PNG. **No HEIC decoder** (Charles, 2026-10-24:
  "exporting a JPEG or using the iPhone is fine") — libheif WASM would need `'wasm-unsafe-eval'`
  in the CSP on the shared origin; don't re-offer it. Synthetic ⌘V in the Browser pane
  fires no paste event; test pastes in headful Playwright with a pasteboard built in JXA.

- **SCHEMA 18 (2026-10-31, Charles): a pairing's `food`** — a `FOOD` code (redmeat, poultry, pork,
  seafood, veg, spicy, cheese, dessert, other) or null; `cleanFood` is the boundary. Pair Meal's
  strict tool has a required `food_kind` enum built from the same list (a test pins them equal),
  `readPairing` → `r.food`, carried by Plan for Later and Opened One (`drinkPairing.food`); the
  pairing window has `#p_food`. History code `food`. What Pairs Well shows **Food beside Grape**
  (`PAIR_LIKE_TABLES`, `renderPairLikeTable`, per-table `pairLikeSort`; `pairLikePick` has both
  fields, AND across) — grape alone averaged a ribeye with a lava cake (Cabernet 2.8). Charles
  asked for it "next to the existing what pairs well for now": the Grape table may go later.
  Pairings are not in share links, so no SHARE_PAYLOAD_V bump.
- **SCHEMA 17 / SHARE_PAYLOAD_V 6 (2026-10-31, Charles): half-point ratings and `heldUp`.** Ratings
  (opened lines AND pairings) are 1–5 in steps of .5 — `cleanRating` is the one boundary, `starsOf`
  the one printer ("★★★★½"). `heldUp` on an opened line is a CODE from `HELD_UP` (great / fine /
  faded / gone, or null) — how the bottle AGED, kept apart from how good it was, so the rating is
  never a curve for age. History code `heldup`; the Opened list's Held Up column hides until a line
  has one (Paid/Where's rule) and is a pill like Timing's, in the status tokens already used
  (great ok, fine hold, faded warn, gone err — no new colour); CSV has it; share links carry it.
- **`--page-w` is 2400px, Money Map's** (2026-10-31, Charles: "match money map"), not the family's
  1500px. The test accepts either.

## Decisions — Don't Undo

- **The 📌 pins the tab row** (2026-10-26, Charles: "like my other apps") — League Night's shape:
  `data-pin` set by the head script (`wine-pin`), `.pinbtn` a sibling of the tablist, `--pin-top` /
  `--pin-clear` MEASURED by `measurePinTop()` (which replaced the old `--head-h` observer; the root's
  `scroll-padding-top` reads `--pin-clear`). Unlike League Night it pins on a phone too: the header is
  `position: static` under 560px, so the row stops under the notch band (`body::before`) instead.
  `.pinbtn` needs `min-height: 0` here, or the page's button floor stretches it to 26×38.5. A change
  belongs in all five pin apps. Tests set `data-pin` + `measure()` rather than `applyPin()`, which
  writes storage; a sticky test needs its spacer INSIDE `<main>` (the row only sticks in its parent).
- **A refresh lands in one place** (2026-10-26, Charles: "refreshing the Cellar while scrolled to
  the bottom flickers more"). Measured frame by frame: the browser painted the TOP for a frame,
  restored short, then `footRows()` (a rAF later) grew the page 56px and it jumped again — three
  positions in ~50ms at 756px, the Cellar worst because of its foot row and card fit. Now `boot()`
  runs `fitCards(); footRows(); restoreScroll();` between `render()` and the gate release, and
  `history.scrollRestoration = 'manual'` with the place kept in sessionStorage `wine-scroll`
  (`{y, tab}`, written on pagehide, only for a reload/back-forward to the SAME tab; never in
  tests or a shared view). Re-measured: first visible frame is the final place on every tab, in
  Chromium and WebKit iPhone. Anything that changes page height after first render belongs
  before that release. Other family apps were not checked.
- **Every tab is dashboard → groupings → list** (2026-10-27, Charles — it was dashboard → list →
  groupings from 10-26 until the groupings became filters): Cellar = At a Glance, Cellar Makeup,
  Cellar; Opened = timing tiles, What You Like, Bottles Opened; Pairings = Pairings at a Glance,
  What Pairs Well, Pairings. A test pins each panel's card order. A grouping that FILTERS sits
  above what it filters; a read-only one would still go below the list.
- **The grouping tables are filters** (2026-10-27, Charles's idea). Press a row → the tab's list
  narrows to it. Picks are page-life maps `makePick` / `likePick` / `pairLikePick` (field →
  Map(key → name)), pressed only through `setFacet` / `clearFacets` (`FACET_TABS` says what each
  card narrows and redraws — that card and its list, never the tiles). **OR within a table, AND
  across** (Charles chose it). `facetWritten` / `facetName` / `facetKey` are the ONE place a
  record's group is worked out — `makeupGroups`, `likeGroups` and every list filter read them, so a
  row and the list can't disagree. Each table is counted over what the OTHER tables' picks leave
  (`facetMatch(…, skip)`), so its siblings stay pressable — and FADE (`tr.unpicked`, --text-muted)
  once the table has a pick (Charles, same day: undimmed they read as "this table didn't filter"); a row that pool empties stays at 0,
  `dead`: dimmed (opacity .4) and its button `disabled` — it USED to drop out, and on a phone a press
  moved the page ~140px (Charles, same day: "dim and make unclickable"). A picked row is never dead.
  **No row may change height on a press**: the ✓'s room is reserved on every row
  (`.facetbtn::after` hidden) and the name doesn't turn bold; the Clear button hides by
  `visibility`, not `hidden`. Measured at iPhone 13 size: the pressed row moves 0px in every table;
  only the warn line under the last table changes. Share
  is of the table's own pool (`g.total`). The tiles and every other dashboard figure stay
  whole-set; the list notes say so ("always counts …"), and a source test pins the three phrases.
  **The tiles narrow the groupings too** (Charles, same day): `statusPick` / `timingPick` /
  `pairingPick` reach the tables as `keep` (makeupNow / openedNow / pairingsNow), the same
  dead-not-gone rule; each set*Pick redraws its grouping card. Find does NOT reach them. Planned
  or Overdue alone leaves What Pairs Well all dead — it counts had pairings only; honest, kept.
  Each list's Clear empties its tab's picks (the card's Clear sends focus to its first row); `__plant` empties them all. A pairing's grape
  is `pairingGrape` (opened line's, else wine's), for What Pairs Well and the list alike; the list
  filter covers planned pairings too. **The Drinking Windows Ahead chart was REMOVED the same day** (Charles),
  with its SVG, hover panel, `#chartMaxi` full-screen overlay, `computeAll().years`, `inWindow`
  and 8 tests — so this app is no longer one of the family's ⤢ full-screen-chart apps.
- **Cellar Makeup** (2026-10-26, Charles): `#makeupCard` (above the Cellar list since 2026-10-27) — What You
  Like for the cellar as it stands (above the list since 2026-10-27). `computeMakeup(st, yr, picks, keep)` → `makeupGroups` (likeGroups'
  naming, but weighted by `qty`; counts wines, bottles, `ready` = ready/soon/past). Share is
  of EVERY bottle, blanks included. Columns Wines · Bottles · Share · Ready (5, not What You Like's
  4) — so the grid is 23rem min (two across only when they fit; half-screen 756px is one across)
  and under 30rem the cells lose half their side padding and the headings their letter spacing;
  measured: all four tables fit 313px exactly. Don't add a sixth column without re-measuring.
  **Six tables on purpose** (Charles, same day: Colour sat
  alone on a second row with four): + Drink By (`drinkTo` as a year; its OWN order is the calendar,
  soonest first — it stands in for the removed Drinking Windows chart) and Where It Is (`location`,
  case-blind). Six fills 3-across and 2-across evenly; keep the count divisible by 6 or re-check.
  `MAKE_TABLES[f][3]` is what a left-out bottle lacks, for the warn line and the empty table.
- **Pair Meal + the Pairings tab (SCHEMA 15, 2026-10-26, a friend of Charles's idea).**
  `buildPairRequest` is pure and pinned: Opus 5.5 effort `low` (Charles chose it over Sonnet),
  ONE strict tool `recommend_wines`, **no web search and no `cache_control`** (a cache write costs
  1.25× and a dinner question is almost never followed by another inside 5 minutes — don't "fix"
  the missing cache). The cellar goes as numbered lines (`pairWines`: qty > 0, drinking order, cap
  150) — Claude answers by NUMBER and `readPairing` maps it back to the id sent, dropping strays,
  repeats and anything past 3. Recipe cut at `PAIR_TEXT_MAX` 4000 and never stored; `pairLast` is
  page-life. Measured on 4 real meals: ~1600 in / ~450 out, **$0.013–0.017**, 5–10 s.
  `state.pairings[]` `{id, wineId, label, meal ≤PAIR_MEAL_MAX 60, date '' or day, status
  planned|had, rating, note, openedId, why, at}` — `label` snapshots the wine like an opened line;
  the PAIRING's rating is how the food and wine went together, never the bottle's. Pairings tab is
  fourth in `TABS` (before History), hidden in a shared view like History, not in share links.
  Opened One for a meal goes through `drinkPairing` (set by `setDrinkPairing` AFTER `openDrink`,
  cleared by `dressOpenDialog`): Save flips a planned pairing to had or writes a new had one,
  linked by `openedId`, and the wine's `open` History entry gains a `meal` change. Plan for Later
  writes a `plan` History kind (LOG_KINDS) with `meal`; History's What filter has **Pairings**.
  **SCHEMA 16: a pairing keeps its own History** (Charles: "marked a pairing as actual and rated
  it, then hit undo — that didn't get logged"). `p.log` via `cleanLog`; `PAIR_FIELDS` + `diffPairing`
  (meal, wine, status, day, rating, notes; codes `wine`/`status`/`day` added to LOG_CODES).
  Writers: `addPairing` (plan / open / add), the Opened One flip (`open`, Planned → Had), the
  pairing window's Save and We Had It (`edit`, only when something changed), and `undoLast`
  (`undo`/`revive`, captured in `wasPair` BEFORE `state` is replaced — the opened-line trap).
  `coreOf` nulls pairing logs. History rows carry `pairingId` → `data-pairing` → `openPairing`;
  `#pairingHistory` in the pairing window. **Any new record kind that Undo can touch needs the
  same three things: a log, a diff, and an undoLast branch.**
  **Its dashboard** (same day, Charles chose tiles + Best and Worst): `computePairings(st, today)`
  is the one place its figures are worked out, over the WHOLE list. `pairingKind` splits planned /
  overdue / had (disjoint — the list's own words). The Planned, Overdue and Had tiles and the
  list's **Status box** (`#pairingPickBox`) are ONE filter, `pairingPick`, pressed only through
  `setPairingPick` (repaints aria-pressed + the box, redraws the list alone — never the tile
  markup) — the Cellar's tiles + Status box and the Opened tab's tiles + Timing box, the same shape.
  The tiles were first shipped as a REPLACEMENT for a filter (wrong; Charles caught it): every tab
  keeps its dropdown beside its tiles. **What Pairs Well**
  (`computePairings(st, today, picks, keep).grapes` → `likeGroups`, rendered by `renderPairLike`)
  REPLACED a Best and Worst list the same day — Charles: "if there are five 5-star reviews, which
  one is the best?" It picked the three newest and silently dropped the rest; don't bring a
  ranked top-N back. Only HAD pairings; a pairing's grape is its opened line's copy, else its
  wine's, else blank (counted in the warn line).
  **Pair With a Meal + Ask Claude (2026-10-31, Charles had a 2009 Stags' Leap Cab with frozen
  pizza and wanted notes).** No SCHEMA bump: `why` already existed — it is now an editable
  textarea (`#p_why`, "Claude's Notes", maxlength 250) and is LOGGED (`why` in LOG_CODES and
  PAIR_FIELDS, values never recorded — like notes). `#openPair` in `#openDialog` (edit mode only)
  calls `openPairing(id, line)`: `pairingFromLine` set, `p_wine` disabled, status had, date = the
  line's; Save goes through `addPairing` with `label` + `openedId` from the line (`addPairing` now
  takes `w` null and writes the wine's log only when there is one). The button reads **Open Its
  Pairing** when a pairing already has that `openedId`. **✦ Ask Claude** (`#p_ask`) →
  `judgePairing` → `callTool(body, 'note_pairing', …)` — `callTool` is Pair Meal's call loop pulled
  out and shared. `JUDGE_TOOL` {meal_name, food_kind enum, notes}; `buildJudgeRequest` sends meal,
  `judgeBottle` (name/vintage/grape/region/window + opened date, age, timing) and `told`
  {bottleNote, heldUp, rating, note} — **never price or place**. Fills `p_why`, and `p_food` only
  while Not said; nothing saved until Save; `judgeStop` on dialog close. Measured on the real
  bottle: ~1050 in / ~465 out, 4–10 s, ~1–2¢. **Ask for WORDS, not characters**: "under 250
  characters" came back at ~280 and got clipped mid-sentence; "about 35 words" came back whole.

- **Lookup `confidence` is WHICH bottle, nothing else** (2026-10-25, Charles). The schema and
  `LOOKUP_SYSTEM` both say so; how firm the value/window are lives in `value_is_estimate` and
  `window_basis` on their own lines. Before, one word did both, and a label Claude had just read
  came back "Probably the right wine" only because no merchant listed that vintage. Badges:
  `LOOKUP_CONF` (Sure of the wine / Probably / Could not find) plus `labelPill(r)` — a neutral
  **Label read** when `r.labelName` came back. **Check Wines sends each wine's kept photo**
  (`loadPhoto(w)`, the ≤800px copy), as the editor's Look Up does; privacy.html says so.
- **The label's name/vintage are OFFERED** (`LABEL_KEYS`, 2026-10-25, Charles: "Vocal" typed for
  "Vöcal"): first rows of `LOOKUP_BOXES` and `LOOKUP_FIELDS`. An EMPTY box is still filled by the
  Look Up handler; a typed one gets Use This only — `applyLookup(all)` skips LABEL_KEYS, and the
  panel's Use All count ignores them. Taking one moves `lookupStamp.key` to the new name so Save
  keeps the check stamp. Check Wines' `tickRule`: name ticks only when equal after `searchFold`
  minus non-alphanumerics; vintage never ticks. The Opened window's Look Up does not offer them.
  The offered name is `labelNameFor(typed, label)`: a label whose words are all in the typed
  name (a front label printing only "Vöcal") RE-SPELLS those words in the typed name; only a
  label with words of its own is offered whole. Never offer the raw label_name over a typed one.
- **Lookups send Came from** (2026-10-25, Charles: Wine Access listed the 2023 Vöcal at $95
  while Claude guessed $45–75): `buildLookupRequest` adds "Came from, in the owner's words: …"
  (never the old "Already in the cellar"), and `LOOKUP_SYSTEM` says to search that shop first.
  The editor's Look Up passes `f_source`; Check Wines and paste pass the wine. privacy.html says so.
- **A non-vintage lookup sends Added** (2026-10-25, Charles's decade-old 30-year tawnies): only
  when `!w.vintage` and `added` is a day or bare year — "in the owner's cellar since X, so bottled
  no later than that". `LOOKUP_SYSTEM` says an NV window runs from bottling (a bottling date read
  off the photo first). The editor passes the Added box as the form holds it. privacy.html says so.
- **Lookup text caps** (2026-10-25, Charles: "100 is clearly not enough"): Claude's value/window
  explanation `BASIS_MAX = 250` (`clipWords`, whole word + "…"), wine notes `NOTES_MAX = 600`. Both,
  and History's `LOG_*` constants, are declared ABOVE normalizeState — boot reads them (TDZ).
  (`LOOKUP_LINE_MAX` went with the Looked up line in SCHEMA 12.)
- **SCHEMA 13: `color` may be `''`, "Not set"** (2026-10-25, Charles). The Add window starts
  there instead of Red; normalizeState keeps it (and no longer guesses red for a missing one);
  Look Up fills it like an empty box and Check Wines counts it as empty. The paste list's guess
  from grape and name still ends in red — that is a parse of what was pasted, not a default.
- **SCHEMA 12: History (`w.log`)** (2026-10-25, Charles). A Use This on the NAME rewrote the old
  "Looked up …" note line with Claude's window explanation beside a window he had KEPT, and
  `windowBasisOf` trusted that line. So nothing writes notes any more: each wine keeps its last
  `LOG_CAP = 20` entries `{at, k, c?, x?, cf?, src?, wb?}` (`cleanLogEntry` is the allowlist —
  optional keys OMITTED, never undefined; `c`/`x` values ≤60 chars, notes/photo carry none). Kinds:
  add, paste, edit, ai, check, look, open, back, undo, revive. `lookupEntry(r, before, after)` is
  the ONE reading of an answer: taken = the saved wine now says it (`c`), left = it doesn't (`x`),
  `wb` ONLY when the window was taken. An answer for a saved wine writes a `look` at once
  (`stampChecked(id, r)`); the Save after it REPLACES that entry by its `at` (`logLookup`), so one
  lookup is one entry. Editor Save also writes an `edit` for anything else changed (`handEdits`).
  `coreOf` ignores `log`; `undoLast` carries the current logs forward and appends `undo`
  (or `revive` for a wine it brings back). Old Looked up lines are DELETED, not migrated
  (Charles: "we can't really trust them"), in normalizeState for a save older than 12, after
  `subregionFromNotes`. Not in share links. **History tab** (`#historyPanel`, third in `TABS`) is
  `computeHistory` + the Opened tab's machinery: `sortHead`, Find via `foldHas`, page-life
  `historyPick` (kinds) / `historyFields` sets, a host in `CARD_HOSTS`, rows open the wine.
  The wine window shows the same list (`renderWineHistory`) under Notes.
- **SCHEMA 14: an opened line keeps its own History** (2026-10-25, Charles: "history doesn't seem
  to be tracking changes to the opened bottles"). `o.log`, same `cleanLog` allowlist; `LINE_FIELDS`
  + `diffLine` are the line's version of LOG_FIELDS/diffFields, adding codes `opened` (date),
  `rating`, `place`; `ALL_LOG_FIELDS` feeds `LOG_LABEL` and the Field filter. Writers: the line's
  window Save (`edit`, only when something changed) and Log Bottle (`open`); Opened One logs on
  the WINE only (no duplicate row). `coreOf` nulls line logs; `undoLast` carries them forward
  (capture `wasLine` BEFORE `state` is replaced — reading it after wiped every line's log) and a
  line made by Opened One this sitting has NO log yet, so `(before.log || [])`. History rows for a
  line carry `lineId` → `data-bottle` → `openBottleEdit`; `#bottleHistory` in the Opened window.
  Share links already whitelist line fields, so `log` never travels.
- **The thumbnail enlarges** into `#photoView`, a TOP-LEVEL dialog (inside the edit window its
  clicks would trip that window's click-outside and close both). `photoFit` sizes it to the
  photo's own ratio within the opener's width × the screen height. Any click, ✕ or Escape
  closes it. A scripted `.click()` opens it with no user activation, and Chrome's close watchers
  then group it with the window under it — one Escape closes BOTH; a real click doesn't.

- **Every Find folds text through `searchFold()`** (2026-10-24, Charles: "resonance" must find
  Résonance) — NFD minus marks, lower case, `FOLD_LETTERS` (œ æ ß ø ł đ ð þ ı) spelled out, curly
  apostrophes straightened, applied to the query AND each field via `foldHas`. The Cellar's Find,
  the Opened Find and `searchApp` (⌘K) all use it; a new search must too.

- **Status is derived, never stored** (`statusOf(w, year, soonYears)`). The verdict column in
  a pasted Claude table is read past on purpose: a verdict is right for one year, a window
  for all of them.
- **Counts are bottles, not wines**, and only bottles still owned. Finished wines (qty 0)
  stay on record, hidden unless Settings → *Show wines with no bottles left*, and are never in
  a figure.
- **Free-text notes are allowed here** — this is a personal app, not one of the two work apps
  — but capped (600 for a wine's notes and an opened bottle's, NOTES_MAX) and single-line where the field is single-line.
- **Wine Access's descriptions are their copyrighted text.** Notes seeded from them are short
  summaries in our own words, never pasted copy.
- **Charles's real bottles are never committed** — the repo is public. They go in
  `my-shipment-*.json` (gitignored) and are loaded through Back Up → Restore, or pasted.
- **Sync is Golf Handicap's module, ported whole** (GIS sign-in, `shouldBootSync`, the push
  gate, `syncDecision`, `forCloud`, `describeSyncError`, the relisten timer). A fix to sync
  starts in Golf Handicap or is mirrored there. **Since 2026-10-26 that includes Golf's
  offline-overwrite fix, verbatim**: `cloudTag`/`sameTag`/`syncPlan`/`makeCloudSync` in the
  classic script, every cellar push a `runTransaction` that writes only while the cloud is
  still this device's base (`wine-cloud-base`, `{uid, rev, at}`; dirty = `wine-updated` ≠
  `wine-cloud-sent`), the which-copy question when both sides moved, and `catchUp()` on
  `online` / visible / a minute after a failed push. The doc carries `rev`. The suite runs
  Golf's `runSyncSim` verbatim. Golf's CLAUDE.md has the full note. Photos still `setDoc` their
  own small docs — they are keyed per record and re-sent from the unsent list. The CSP's external hosts are exactly Golf's
  list, and a test pins them.
- **Two changed copies are MERGED, not chosen between (2026-10-29, Money Map's e73c28b,
  ported).** Nearly every press saves, so both devices were "changed" far more often than either
  had work the other lacked, and the whole-copy choice threw one side's edits away. Now:
  - **`wine-cloud-base-copy`** (`{uid, rev, at, json}`, json = the doc's `state`) is the CONTENT of
    the engine's base, recorded in the module's io hooks only — `adopt`, a successful `transact`
    write (`.then`), a `read`/snapshot whose tag IS the base, and a settled merge (`take()`
    records the cloud copy it merged, which keep() then makes the engine's base). The engine is
    untouched, byte for byte. `baseCopy()` returns it only while its tag equals `wine-cloud-base`.
    Delete All Data removes it.
  - **`settleTwoCopies`** is the `ask` hook. Data on both sides + a base → `mergeCellars(base,
    local, remote, prefer)` (classic script, pure, pinned): wines, opened lines and pairings
    merge BY ID, field by field; a one-side change is kept; a delete lands unless the other side
    edited the record (then a conflict). **Never a conflict:** `version`; a record's `log`
    (union, time order, LOG_CAP); `checkedAt`/`photoAt` (the later wins — the same rule
    `loadPhoto` fetches by, so a photo kept only here is never orphaned); and a wine's `qty`,
    which is a COUNT — both moves land (`syncMergeCount`), so an Opened One on each device takes
    two bottles off. No conflicts → adopted through `adoptInto` (the ONE `wcAdopt` call in the
    module) and keep() pushes; toast "Combined changes from both devices". Conflicts → the
    dialog lists only those (`describeChanges`: `Wine “NV Name” · Bottle count`, `Opened bottle
    “…”, date · Tasting note`, `Pairing “meal” · Day`; a value only for a count or a rating).
    No base → the old whole-copy choice, listing `diffCellars` (History and check stamps left
    out) and both save times; silent when nothing else differs. An empty copy never merges; a
    NEWER-build copy is never merged (`mergeCellars` → null) so `wcAdopt`'s halt sees it.
  - `syncNormal` = `normalizeState` round-tripped, so a filled-in default is never a change.
    `forCloud` is a plain JSON copy (it strips only `undefined`); photos were never in `state`,
    so the merge compares like with like. There is no view state in `state` — tab, folds, sort,
    the lookup key are their own keys and never sync — so nothing is "this device's own".
  - The dialog's device time is `wine-updated`, which any save moves — it says "last saved".
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
  same place on every card. **Superseded on the three tab lists (2026-10-25, Charles: "rough when
  things aren't in the same place while scrolling")**: there `tr` is a GRID of `--cols` equal
  columns set by `@container` on `#listWrap/#openedWrap/#historyWrap` (2 → 6 at 34/48/62/76rem;
  the container rules must carry the same `:is(#…)` specificity or the 2 default wins), so a field
  is in the same cell on every card. `td.cardtop` (Status / Opened / When) shares row 1 with the
  name, or sits under it at 2 columns; Region, Changed, Left span 2; Note spans all. An empty td
  keeps its cell (`visibility: hidden`) unless nothing after it is filled. The paste preview keeps
  the flex line (an inline-size container inside a content-sized dialog sizes to 0). The phone test
  checks `drift` — fields with more than one left edge — is empty. **`fitCards(only)` refits only the hosts an
  observer saw change, and puts `scrollY` back** (2026-10-25): refitting every host when Paste a
  List's preview got its first width probed the Cellar behind the window as a table, the page got
  shorter for a moment, and iPhone Safari (no scroll anchoring) clamped it to the foot. Measured in
  Playwright WebKit with `devices['iPhone 13']` + `touchscreen.tap` — Chromium never showed it, and
  moving focus around was a red herring.
- **Redraw only what a press changed — iPhone Safari jumps otherwise** (2026-10-25). What You
  Like's view switch (since removed) and its sort headings called `renderOpenedDash`, which rewrote the
  timing tiles; with the tiles half above the screen WebKit dropped the page ~700px (a final-layout
  effect: any forced layout in between hid it, so checkpointing scrollY "fixed" it). They now call
  `renderLikeTables`, and the tiles are written only when their HTML changed. Reproduce such jumps
  in Playwright WebKit + `devices['iPhone 13']` + `touchscreen.tap`; Chromium never shows them. Each `<td>` carries `data-label` (the card shows it via `::before`,
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
  **Prompt caching** (2026-10-25, after Anthropic's low-hit-rate email): `system` is one marked
  block (shared by every wine) plus top-level automatic `cache_control`. Web search re-reads its
  results inside one call; cached, those reads are 5% price. Measured on 4 real wines, same
  answers: $0.137 uncached → $0.056 system mark only → **$0.046 both**. `LOOKUP_COST`/`LOOKUP_SECS`
  drive Check Wines' estimate; `lookupWine` usage counts `cacheRead`/`cacheWrite` apart from input.
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
- **`added` is a day, a bare year or ''** (SCHEMA 2 / SHARE_PAYLOAD_V 2, 2026-10-23), chosen in the
  wine window by leaving the date blank or ticking Just the year (`dressAdded()`) — for EVERY wine
  since 2026-10-24 (Charles chose "date optional for all" when Came From became free text; it was
  "Already in the cellar" only). `cleanAdded()` is the boundary — an ABSENT field is still today,
  so old records are unchanged; `fmtAdded()` is the one way to print it. A new wine starts on today.
- **`source` (Came from) is free text, 40 chars** (SCHEMA 11, 2026-10-24, Charles: "someone else
  may use this app" — no shop's name built in). `cleanSource()` is the boundary and turns the old
  keys into words (`wineaccess` → Wine Access, `cellar` → Already in the cellar, `other` → '')
  wherever they arrive — old saves, backups, share links. `#sourceList` offers back used values
  (`renderLocations`); paste's `#pasteSource` is a text box too. Find searches it.
- **Wrong-wine sources** (2026-10-23): `readLookup(found, asked)` drops a source whose PATH has a
  vintage year and shares no distinctive word (`wineWords`, minus `SOURCE_COMMON`) with the asked
  name or Claude's `matched_wine`; `r.dropped` is said on screen (`droppedLine`). Every caller passes
  the asked wine. Kept narrow on purpose: on the 58 real pages from that day's re-runs it dropped
  exactly the one wrong page. It cannot undo a figure Claude took from that page.
- **Check Wines** (2026-10-23) reuses that lookup across every wine with `qty > 0`.
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
- **Check Wines picks vs ticks:** `checkPicks` (wine ids, "Select …" buttons, `data-pick`) say what
  Start looks up; `checkTicks` (`data-tick`, Tick All) say what Save changes. Never share the
  words. Picks fill on open from `#checkSince` (not checked in N days, plus this sitting's
  failures — page-life only, not `state`); a wine leaves the picks when its answer arrives.
  A wine WITH an answer has no pick box at all (`checkSelectable()`), and every Select button and
  the count skip it — a hidden box Select All could tick would re-bill invisibly. A failed lookup
  has no answer, so it keeps its box.
- **Not found still leaves a record** (2026-10-24; History since 2026-10-25): a `low` answer is a
  `look` entry with `cf: 'low'` and nothing else. Check Wines offers nothing for it.
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
  needs no SCHEMA bump.
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
- **A shared snapshot shows no History tab** (2026-10-25): `shownTabs()` drops it when
  `viewOnly`; setTab clamps to it and hides the button, the arrow keys walk it, and a clamped
  tab is NOT written to `wine-tab`, so the reader's own copy keeps its remembered tab.
- **Delete Wine is folded under `#wineDeleteZone`** (a `details.danger-zone`, as Start Again
  is in Back Up; 2026-10-25), shut on every open. Hide the ZONE for a new wine, not the button.
  The Opened window's Remove This Bottle is folded the same way, under `#openRemoveZone`.
  In every dialog row with a left-hand action, Cancel + the primary button sit in one
  `.act-end` span pushed right, so a narrow row wraps the pair whole — never Save alone.
- **Windows open at their top; the record windows keep the page's place** (2026-10-25):
  `openModal` sets `scrollTop = 0` (iOS Safari keeps a closed dialog's scroll, so every wine
  after the first opened at Save), and takes `placeY` for `KEEPS_PLACE` (wine + opened
  windows), which a watcher on `open` puts back when the last of them closes. Save's render()
  and close()'s focus hand-back both move the page. Opt-in on purpose: a Find hit moves it.
- **The card buttons are at the TOP; a foot copy is MEASURED** (2026-10-26, Charles: scrolling
  past every wine to add one). `#cellarTop` / `#openedTop` sit above Find, ids unchanged.
  `#cellarFoot` / `#openedFoot` show only while the top row → end of list is taller than the
  window (`footRows()`, a ResizeObserver + resize). Not a row count like FM/SP's — a wine is a row
  or a tall card depending on width. Foot buttons are `data-same` stand-ins that press the top one
  (one handler per action) and copy its hidden/disabled; no Load Sample Data copy.
  **No Paste a List button** (Charles, 2026-10-26): it is reached from the Add window's
  `#wineToPaste` link only — keep that link, it is now the only way in.
- **The table's own order is drinking order** — past the window, drink soon, ready, hold,
  none; then the window that closes soonest. The Wine column sorts by name, then vintage.

## Firebase

Project `winecellar-fa733` (Spark). Google sign-in on; `eagleadams86.github.io` authorized;
Firestore `(default)` in `nam5` with per-uid rules on `winecellar/{uid}` (see
`firestore.rules`); the auto-created web OAuth client allows `https://eagleadams86.github.io`
and `http://localhost:8025`. Since 2026-10-24 the console holds `firestore.rules` exactly,
comments and all (put in through the editor's CodeMirror view, then Publish) — before that it
was a one-line version of the cellar rule.

## Tests

`tests.html`, `EXPECTED = 345`. Run it headless (Playwright) rather than in the Browser pane:
the pane throttles hidden timers and its focus heuristics turn the 705px keyboard-focus check
red on untouched code.
