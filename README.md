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
- **Click any block to edit its text.** Edits save to `localStorage` in that browser
  only — nothing syncs between viewers.
- **Reset to Original** clears all edits.
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

- **Full-band bars are duplicated, not spanned.** Toast UI cannot span one event across
  columns, so each full-band row emits one identical event per visible track column.
  Adjacent, identically coloured blocks read as a single bar. The same limitation
  applies to cross-track "superblocks" (two or more tracks running the identical
  activity), which are matched by gold styling rather than merged into one rectangle.

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

- The calendar is **read-only**: only text is editable and persisted, so drag-and-drop
  would silently revert on reload.

- The print stylesheet pins **portrait**; a 7 AM–11 PM grid does not fit landscape.

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
