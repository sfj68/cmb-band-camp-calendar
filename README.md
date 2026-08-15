# CMB Band Camp Calendar — 2026

The 2026 Cyclone Marching Band camp schedule as a single-file, no-build web page,
rebuilt on [Toast UI Calendar](https://github.com/nhn/tui.calendar) v2.1.3.

Eight camp days, Sunday Aug 16 – Sunday Aug 23, 2026, with Drumline / Guard / Winds
tracks side by side and an optional Leadership calendar.

## Running it

Open `index.html` in a browser. There is no build step and no dependencies beyond the
two pinned CDN files.

## Deploying to GitHub Pages

1. Create the repo on GitHub and push this directory to `main`.
2. Repo → **Settings → Pages** → Source: **Deploy from a branch** → `main` / `root`.
3. Live at `https://<username>.github.io/cmb-band-camp-calendar/` in about a minute.

A public repo gets Pages for free; private repos need a paid plan. No `.nojekyll` is
needed — nothing here starts with an underscore.

## Features

- **Day navigation** — prev/next buttons and a day dropdown.
- **Track toggles** — show or hide Drumline, Guard, Winds. Leadership is a separate
  calendar, off by default.
- **Click any block to edit it** — name, day, start and end time, details, and which
  tracks it applies to. **+ New Event** adds one from scratch; **Delete** removes it.
  Changes save to `localStorage` in that browser only — nothing syncs between viewers.
- **Reset to Original** clears every change and restores the published schedule.
- **Print** emits all eight days, one per page, at 8.5×11.

## How it works

Toast UI Calendar has no concept of resources, and its week-view columns are always
dates. The tracks are mapped onto weekday slots instead, with **each camp day rendered
as its own synthetic week** starting from an arbitrary base Monday (2000-01-03), far
from any real date so nothing is highlighted as "today".

A few consequences are worth knowing before editing this file:

- **Week view always renders five columns** (`workweek: true` filters the weekend).
  Tracks fill the leading columns in visible order, and the spare columns are hidden by
  over-widening the calendar inside an `overflow: hidden` wrapper —
  `calc(var(--axis-w) + (100% - var(--axis-w)) * var(--col-scale))`. Everything inside
  Toast UI is percentage-based, so this stays aligned at any width, including in print.

- **Shared events really do span their columns.** Toast UI has no way to place a single
  event across columns, so a full-band row still emits one copy per track column. After
  render, though, those copies are merged: the individual `.toastui-calendar-column`
  elements do not clip their overflow, so the leftmost copy is widened across the whole
  run and the rest are hidden. The result is one continuous bar with one label, which is
  what tells you at a glance that every track is doing this together.

  Only a *run of neighbouring* columns can be merged this way. A match that skips a
  column — Drumline and Winds but not Guard — would swallow the column in between, so it
  keeps the matching colour and stays as separate blocks.

  The merge happens before anything is measured, so text is fitted against the width the
  bar will actually have. It is also why the post-render pass has to remember the
  library's own inline `width`: these blocks are absolutely positioned, so simply
  clearing the width collapses them to shrink-to-fit.

- **What counts as full band is decided by the data, not just the `full:` key.** If
  every track runs byte-identical text over an identical range, that is a full-band
  item and renders cardinal — Tuesday's lunch and dinner arrive as three track rows but
  mean exactly what Monday's `full:` lunch means. Only a *partial* match, where two
  tracks share an activity while the third runs its own, gets the gold "superblock"
  treatment. Toast UI still cannot merge those into one rectangle, so matching colour
  is the ceiling.

- **Compact banners are encoded as time, not pixels.** When a slot has both full-band
  and track-specific rows, the full-band items shrink to 20-minute banners at the top of
  the slot and the track blocks start below them. This has to be done with *disjoint
  time ranges*, because Toast UI lays overlapping events out side by side rather than
  stacked. Each event keeps its true slot range in `raw`, so the editor still shows the
  honest time (the 8 AM Tuesday banner reports 8:00 AM – 9:45 AM).

- **All eight calendar instances stay mounted** so printing can emit every day. Days
  that are not current are hidden with `visibility: hidden`, not `display: none`, so
  Toast UI keeps measuring real widths.

- **Block heights are set as percentages.** The print stylesheet shortens the grid, and
  pixel heights would overflow into the block below.

- **No true black anywhere.** Dark text is a deep cardinal (`--dark: #4A1220`), because
  near-black on gold reads as the wrong school's colours. It holds 9.7:1 contrast on
  gold and 15:1 on white. Neutral greys are warmed to `--muted` for the same reason.

- The calendar is **read-only**: only text is editable and persisted, so drag-and-drop
  would silently revert on reload.

- The print stylesheet pins **portrait**; a 7 AM–11 PM grid does not fit landscape.

### Edits live in an overlay, not in the schedule

`DAYS` stays the source of truth and keeps flowing through the pipeline untouched.
Everything the user does lands in a separate overlay in `localStorage`
(`cmb_band_camp_calendar_v3`), which is why **Reset to Original** can restore the
schedule exactly:

- **overrides** — keyed by the event's edit key. A record carrying a `start` is
  *pinned*: the user gave it explicit times and tracks, so the derived event is
  suppressed and the block is rebuilt from the record instead. A record with only a
  title or details is a plain text edit and stays in the pipeline, so merging and
  cross-track matching still apply.
- **custom** — events added outright.
- **deleted** — edit keys removed from the schedule.

A pinned or custom event deliberately sits outside the merge and cross-track passes.
The user stated exactly what they wanted, so nothing should quietly absorb or reshape
it. Its colour still follows the same rule as the derived schedule: all three tracks is
full band and renders cardinal, two of three renders gold, one gets that track's colour.

Because the editor is prefilled from what is actually on the grid, saving an event
without changing anything leaves the schedule looking identical.

### Schedule data

`DAYS` and `LEADERSHIP_BY_DAY` near the top of the `<script>` are the source of truth,
copied verbatim from the original hand-rolled `CMB_Band_Camp_Schedule_2026_CalendarView.html`.
Times are `"H:MM AM/PM"` strings; `"after mtg"` parses to 8:00 PM. A row's optional
`end` marks an open window (uniform checkout, move-in) that should not be cut off when
the next item starts. `"—"` means nothing scheduled for that track — which normally
means another column split its schedule, so that track's previous block continues.

### Pinned dependency

Toast UI Calendar is pinned to `v2.1.3` rather than `latest`. It is the final release
(published August 2022) and the project is effectively dormant, but a `latest` that
silently changes under a schedule people rely on is a bad day.
