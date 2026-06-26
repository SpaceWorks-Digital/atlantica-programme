# HANDOFF — ATLANTICA Programme Dashboard

> ⚠️ **Superseded.** The current, comprehensive handover is
> [`ATLANTICA_SCHEDULE_DASHBOARD_README.md`](./ATLANTICA_SCHEDULE_DASHBOARD_README.md). This note is kept for history.

Resume note for any Claude Code session (e.g. a cloud session started from the phone) that picks up
this repo. Read this first.

## What this is
A self-contained interactive **project programme / Gantt dashboard** for the ATLANTICA Wavepool Cascais
project (26.015). Single static site — **`index.html`** (vanilla JS, no build step, no deps) + a PWA
(`manifest.webmanifest`, `sw.js`, `icon-*.png`).

- **Live:** https://spaceworks-digital.github.io/atlantica-programme/
- **Deploy:** push to `main` → GitHub Pages rebuilds (~30–60 s). The service worker is **network-first**
  (`atlantica-pm-v3`) and auto-reloads clients, so pushes reach phones/desktops on next open — no cache fights.

## Features built
- X-axis timeline zoom: **Day / Week / Month / Quarter** (weekly shows **ISO week numbers** W01…).
- Y-axis rows: 6 elements (Masterplan, Hub, Pool + Concourse, Hotel, Apartments, Landscape), each
  **expand/collapse** to phase rows. Drag the **⠿ grip** to **reorder elements** vertically (touch-friendly).
- Phases are dual-labelled **RIBA 2020 ↔ Ordem dos Arquitetos**, including a **Tender · Procurement**
  (Concurso / Contratação) row between Technical Design and Construction.
- **Start / End date columns** frozen on the left (auto-collapse on mobile to keep the element name legible;
  `Dates` toolbar button toggles).
- Bars: drag to move, drag edges to resize, **tap/click to edit** (bottom-sheet editor on mobile),
  double-tap empty row or `+ add` to create. Element row shows an auto summary span.
- Export / Import JSON, Print, Reset; editable title + project date window.
- Mobile/PWA: installable (Add to Home Screen), offline, single scrollable toolbar.

## Implementation notes
- State persists in `localStorage` key **`atlantica_pm_v2`**; shape `{title,start,end,zoom,collapsed[],
  hideDates,order[],tasks{ "<elId>|<phaseId>": {s,e,label?} }}`. `hideDates:null` = auto (collapse on mobile).
- `migrate()` upgrades older saved states (added the Tender phase + element `order`) without wiping edits.
- Per-device storage — **no cross-device sync yet** (Export/Import to move a programme). A real shared/
  multi-user backend is the obvious next step if needed.

## Open tweaks / next
- Fine-tune the **mobile name-column width** (currently 182px on mobile when dates hidden) to taste.
- Possible: cross-device/shared persistence (backend or URL-encoded state); per-task % complete; milestones;
  baseline vs actual; dependencies.

## NOT in this repo
The broader ATLANTICA Rhino/Grasshopper work lives in the private `SpaceWorks-Digital/SWX-SPACEDESIGN`
repo and is desktop-bound (needs Rhino). This dashboard repo is standalone and cloud-portable.
