# CLAUDE.md — Wine Cellar

A personal wine-collection tracker, built from `~/claude-starter` on 2026-10-23. The
family rules in `~/.claude/CLAUDE.md` all apply (CSP, no third-party script, escape and
sanitize at every boundary, the theme pack). This file records what this app decides that
the family doesn't.

## Shape

- **Dev port 8025** — in both `.claude/launch.json` and `~/.claude/launch.json`.
- Storage keys `wine-state`, `wine-theme`, `wine-tab`, `wine-updated`, `wine-sync-live`,
  `wine-sync-uid`; service-worker cache prefix `wine-shell-`.
- State is `{ version, wines[], opened[], settings }`. `normalizeState()` is the allowlist:
  **a new stored field goes there and SCHEMA is bumped in the same commit**, or it is
  stripped. Tests pin the exact key lists of a wine and an opened bottle.
- Absent is not zero: `cleanYear` / `cleanMoney` test `== null` / `''` before `Number()`,
  because `Number(null)` is 0. A missing value or window stays `null` — never `undefined`
  (Firestore refuses a whole document over one).
- `opened[].label` snapshots the wine's name at the moment it was opened, so the log still
  reads after the wine is deleted. Removing a log line puts the bottle back on the count.

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
- **The table's own order is drinking order** — past the window, drink soon, ready, hold,
  none; then the window that closes soonest. The Wine column sorts by name, then vintage.

## Firebase

Project `winecellar-fa733` (Spark). Google sign-in on; `eagleadams86.github.io` authorized;
Firestore `(default)` in `nam5` with per-uid rules on `winecellar/{uid}` (see
`firestore.rules`); the auto-created web OAuth client allows `https://eagleadams86.github.io`
and `http://localhost:8025`. The console rules were typed as one line — same meaning as the
commented copy in the repo.

## Tests

`tests.html`, `EXPECTED = 170`. Run it headless (Playwright) rather than in the Browser pane:
the pane throttles hidden timers and its focus heuristics turn the 705px keyboard-focus check
red on untouched code.
