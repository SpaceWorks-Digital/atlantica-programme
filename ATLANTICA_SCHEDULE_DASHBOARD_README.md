# ATLANTICA — Schedule Dashboard · README & Handover

> **Canonical handover document.** This supersedes the older `README.md` and `HANDOFF.md`.
> It documents the dashboard **as it actually is today**, and lays the groundwork for the next
> phase: a **database-driven financial / invoicing dashboard** for the company.

- **Project:** 26.015 — Atlantica Wavepool Cascais
- **Live:** https://spaceworks-digital.github.io/atlantica-programme/
- **Repo:** `SpaceWorks-Digital/atlantica-programme` · deploys from **`main`** via GitHub Pages
- **Status:** production prototype, single-file static site + PWA. Per-device data (no shared backend yet).

---

## Part I — The Schedule Dashboard (current build)

### 1. What it is
A self-contained interactive **Gantt / programme dashboard** for the Atlantica project. It is one
file — **`index.html`** — vanilla HTML/CSS/JS, **no build step, no dependencies, no framework**.
Open it locally (double-click) or host the folder on any static host. A small PWA layer makes it
installable and offline-capable on phone and desktop.

### 2. Repository structure
```
index.html               The entire app: markup + CSS + JS in one file
manifest.webmanifest     PWA manifest (installable "ATLANTICA" app)
sw.js                    Service worker — NETWORK-FIRST cache "atlantica-pm-v3"
icon-180/192/512.png     App icons (home screen / install)
README.md                Short legacy readme (kept; see this file for the full picture)
HANDOFF.md               Earlier handover note (kept; superseded by this file)
ATLANTICA_SCHEDULE_DASHBOARD_README.md   ← you are here
```

### 3. Deploy & update flow
- Push to **`main`** → GitHub Pages rebuilds in ~30–60 s.
- `sw.js` is **network-first** (cache `atlantica-pm-v3`): it always fetches the latest when online and
  falls back to cache only when offline. On a new deploy the service worker triggers a one-time reload,
  so phones/desktops pick up changes on next open — no manual cache clearing.
- To force-bust caches after editing static assets, bump the `CACHE` constant in `sw.js`.

### 4. Data model (the important part)
All state lives in a single JSON object persisted to `localStorage` under key **`atlantica_pm_v2`**.
Current schema version **`v: 3`**.

```jsonc
{
  "title":   "Wavepool Cascais — Programme",
  "start":   "2025-10-01",      // project window (From) — timeline left edge
  "end":     "2032-06-30",      // project window (To)   — timeline right edge
  "zoom":    "month",           // "day" | "week" | "month" | "quarter"
  "theme":   "dark",            // "dark" | "light"
  "hideDates": null,            // null = auto (hide on mobile), true/false = forced
  "collapsed": ["design|hotel", ...],   // which group/element rows are collapsed
  "order":   ["masterplan","hub","pool","hotel","apartments","landscape"], // element order
  "tasks": {
    "design|hotel|p2":       { "s": "2026-10-01", "e": "2027-03-01", "label": "optional" },
    "construction|hotel|c3": { "s": "2028-06-01", "e": "2028-12-01" }
  },
  "v": 3
}
```

#### Three-level tree: **Group → Element → Phase**
- **Groups** (`GROUPS`) — two fixed parent folders:
  | id | name | colour |
  |---|---|---|
  | `design` | **DESIGN & TENDERING** | `#38bdf8` |
  | `construction` | **CONSTRUCTION** | `#fb923c` |

- **Elements** (`ELEMENTS`) — the six project elements, shared by **both** groups:
  `masterplan`, `hub`, `pool` (Pool + Concourse), `hotel`, `apartments`, `landscape`.
  Vertical order is user-controllable (drag the ⠿ grip) and stored in `order` (applies to both groups).

- **Phases** (`phasesOf(groupId)`) — differ per group:
  - **DESIGN & TENDERING** → `DESIGN_PHASES` (RIBA 2020 ↔ Ordem dos Arquitetos, dual-labelled):

    | id | RIBA (`t1`) | Ordem dos Arquitetos (`t2`) |
    |---|---|---|
    | `p0` | RIBA 0 · Strategic Definition | Programa Base |
    | `p1` | RIBA 1 · Preparation & Briefing | Programa Base |
    | `p2` | RIBA 2 · Concept Design | Estudo Prévio |
    | `p3` | RIBA 3 · Spatial Coordination | Anteprojeto / Licenciamento |
    | `p4` | RIBA 4 · Technical Design | Projeto de Execução |
    | `pt` | Tender · Procurement | Concurso / Contratação |
    | `p5` | RIBA 5 · Manufacturing & Construction | Assistência Técnica / Obra |
    | `p6` | RIBA 6 · Handover | Conclusão / Receção |
    | `p7` | RIBA 7 · Use | Uso / Pós-Ocupação |

  - **CONSTRUCTION** → `CONSTRUCTION_PHASES` (standard build sequence):

    | id | Phase (`t1`) | Detail (`t2`) |
    |---|---|---|
    | `c0` | Mobilisation | Site set-up |
    | `c1` | Enabling Works | Demolition / Clearance |
    | `c2` | Substructure | Set-out / Foundations |
    | `c3` | Superstructure | Cores / Structure |
    | `c4` | Building Envelope | Facade / Roof |
    | `c5` | MEP · First Fix | Services rough-in |
    | `c6` | Internal Fit-out | Partitions / Finishes |
    | `c7` | MEP & Commissioning | Second fix / Testing |
    | `c8` | External Works | Landscape / Hardstanding |
    | `c9` | Handover / Completion | Snagging / Defects |

#### Task keys
Every bar (task) is stored under a composite key **`"<groupId>|<elementId>|<phaseId>"`** → `{ s, e, label? }`
where `s`/`e` are ISO `YYYY-MM-DD` dates and `label` is an optional custom bar caption.
- **Group** and **element** rows do not store their own dates — their bars are **auto-derived spans**
  (`groupSpan`, `elSpan`) = min start / max end of the tasks beneath them.

#### RIBA 5 ↔ CONSTRUCTION link (bidirectional, per element)
RIBA 5 (the design group's "Manufacturing & Construction" bar) and that element's **CONSTRUCTION**
phases describe the same physical construction window, so they are kept in sync both ways:
- Edit **RIBA 5** → the element's construction phases rescale/shift proportionally to the new window.
- Edit **any construction phase** → RIBA 5 stretches to envelope the construction span (earliest→latest).
- Implemented in `applyLink(key)` / `rescaleConstruction()` / `syncP5FromConstruction()`, hooked into
  every edit path (drag, add, popover apply, delete).

#### Migration & versioning
`migrate(state)` upgrades older saved states without data loss:
- legacy 2-part task keys (`element|phase`) are prefixed into the **design** group;
- the Tender phase (`pt`) is back-filled for very old programmes;
- the **construction** group is **seeded from each element's RIBA 5 window** if absent;
- `theme`, `order`, `collapsed` defaults are normalised.
Bump `STATE_V` when you make a breaking shape change and add the corresponding upgrade step.

### 5. Features & interactions
- **Timeline zoom:** Day / Week / Month / Quarter (weekly shows ISO week numbers `W01…`).
- **Bars:** drag body = move; drag edge = resize; **tap/click = edit** (bottom-sheet on mobile);
  double-tap empty row or `+ add` = create.
- **Rows:** expand/collapse groups and elements; **Expand** / **Collapse** all; drag ⠿ to reorder elements.
- **Dates columns:** frozen Start/End columns (auto-collapse on mobile; **Dates** toggles).
- **Theme:** **Dark/Light** toggle (first toolbar button), persisted; button shows the mode it switches to.
- **Today** marker + jump-to-today; editable title and project From/To window.
- **Mobile/PWA:** installable, offline, opens at the far-left (names + timeline start), single scrollable toolbar.

### 6. Export / import
- **Export Excel** → real `.xlsx` (dependency-free OOXML writer built into the page; opens with no format
  warning). Columns: **Group · Element · Phase · Detail · Start · End · Duration (days)**, hierarchical.
- **Export PDF** → the **current view** (whatever is expanded/collapsed) fitted onto **one A0 landscape
  sheet**, always light-on-white. The timeline is **trimmed to the programme's real end** (snapped to the
  period boundary) so there's no trailing empty space, and the chart **scales up to fill** the sheet.
- **Backup** → downloads the full state as `atlantica_programme.json`.
- **Import** → loads a previously exported JSON (run through `migrate`).
- **Print** → normal browser print (light), default paper.
- **Reset** → restores the seeded ATLANTICA defaults (clears local edits after confirm).

### 7. How to extend the schedule
- **Add/rename an element:** edit the `ELEMENTS` array (`{id, name, col}`). Migration keeps existing tasks.
- **Add/rename a phase:** edit `DESIGN_PHASES` or `CONSTRUCTION_PHASES` (`{id, t1, t2, col}`).
- **Add a group:** extend `GROUPS` and teach `phasesOf()` which phase set it uses.
- **Change the seed programme:** edit `buildSeed()` (per-element start month + phase durations) and
  `seedConstruction()` (how the construction window subdivides).

### 8. Known limitations (today)
- **Per-device only** — state is in `localStorage`; no cross-device sync, no multi-user, no audit trail.
  Move data with Backup/Import. **This is the main thing Part II addresses.**
- No per-task % complete, no baseline-vs-actual, no dependencies/critical path, no resource loading.
- Single project per deployment.

---

## Part II — Toward a Financial / Invoicing Database Dashboard

This is the intended next chapter: evolve from a single-project, browser-stored schedule into a
**multi-project, database-driven dashboard** that manages the company's **fees, invoicing, cashflow and
debtors** — with the programme as the time backbone.

### 9. Why the schedule is the right foundation
For an architecture/engineering practice, **fees are earned and invoiced against stages** (RIBA / OA
phases) of a project. The dashboard already models exactly that spine:

```
Project → Element → Phase → { start, end }
```

Attach money to the **same (project, element, phase)** coordinate and you immediately get:
- a **fee breakdown** by stage and element,
- **invoice triggers** when a phase reaches a % complete or a date,
- a **cashflow forecast** by projecting phase end-dates → expected invoice → expected payment,
- **WIP / earned value** (work done but not yet invoiced).

The schedule is the clock; financials hang off it.

### 10. Conceptual mapping
| Schedule concept (today) | Financial concept (to add) |
|---|---|
| Project (one deployment) | `projects` row — code, client, contract value, currency |
| Element (e.g. Hotel) | cost/fee allocation dimension |
| Phase (RIBA stage) | **fee line** (% of fee) and **invoice milestone** |
| Phase start/end dates | **expected invoice date** → **cashflow forecast** |
| RIBA 5 ↔ Construction link | construction-stage fee vs site-supervision fee tracking |
| Backup JSON | seed/import path into the database |

### 11. Proposed relational schema (starting point)
A normalised Postgres-style sketch (adjust to taste):

```sql
companies(id, name, vat_number, currency_default)
clients(id, company_id, name, vat_number, billing_address, contact)
projects(id, company_id, client_id, code, name, status,
         contract_value, currency, start_date, end_date)
elements(id, project_id, name, sort_order)                 -- Masterplan, Hub, …
phases(id, code, riba_label, oa_label, sort_order)         -- p0..p7, pt, c0..c9 (reference table)
programme_tasks(id, project_id, group, element_id, phase_id,
                start_date, end_date, label, pct_complete)  -- the schedule (was `tasks`)
fee_lines(id, project_id, element_id, phase_id,
          fee_amount, fee_pct, basis)                       -- agreed fee breakdown
milestones(id, project_id, phase_id, element_id, trigger,   -- 'on_complete' | 'on_date' | 'pct'
           trigger_value, amount, description)
invoices(id, project_id, number, issue_date, due_date,
         status, subtotal, vat_rate, vat_amount, total, currency)  -- status: draft|sent|paid|overdue
invoice_lines(id, invoice_id, element_id, phase_id, description, amount)
payments(id, invoice_id, date, amount, method, reference)
-- optional for profitability / WIP:
costs(id, project_id, type, element_id, phase_id, date, amount, description)
timesheets(id, project_id, person_id, date, hours, rate, element_id, phase_id)
```

Key idea: `programme_tasks`, `fee_lines`, `milestones` and `invoice_lines` all share the
`(project_id, element_id, phase_id)` coordinate — so the schedule, the fee, and the invoice line up
automatically.

### 12. Dashboard views to build on top
1. **Project financial summary** — contract value · fees invoiced · paid · outstanding · WIP · % billed.
2. **Cashflow forecast** — derived from `programme_tasks` end-dates + `fee_lines`/`milestones`, plotted on
   the same timeline UI (reuse the Gantt's month/quarter axis).
3. **Invoice register & aged debtors** — outstanding invoices by age bucket (0–30/30–60/60–90/90+).
4. **Stage billing tracker** — per RIBA stage: fee, % complete, invoiced, remaining.
5. **Portfolio roll-up** — all projects: backlog, billed-this-month, forecast cash-in, debtors.

### 13. Suggested architecture & stack
Keep the lightweight, frontend-first ethos; add a managed database with auth so it stays low-ops:

- **Recommended: Supabase** (hosted Postgres + Auth + auto REST/GraphQL + Row-Level Security + realtime).
  Lets the existing vanilla frontend talk to a real multi-user DB with almost no backend code, and RLS
  gives per-company / per-role data isolation. Natural fit for this codebase.
- **Alternatives:** Firebase/Firestore (simpler, less relational — weaker for financial joins/reporting);
  or a small Node/Express + Postgres API if you want full control; or Airtable/NocoDB for a quick
  internal MVP.
- **Auth & roles:** `admin`, `finance`, `project-lead`, `viewer`. Restrict invoice creation/editing to
  finance/admin.
- **Frontend:** the current `index.html` can stay the schedule editor; the financial views can grow as
  additional pages/modules in the same static app, fetching from Supabase. Migrate persistence from
  `localStorage` to the DB per project (Backup/Import becomes the import path).

### 14. Phased build plan
1. **Stand up the DB** (Supabase): `companies, clients, projects, elements, phases, programme_tasks`.
   Import the current Atlantica programme via the Backup JSON.
2. **Move schedule persistence** from `localStorage` → DB (per project); add a project picker.
3. **Fee module:** capture `fee_lines` per element/phase (the agreed fee breakdown).
4. **Invoicing module:** create `invoices` + `invoice_lines` against phases/milestones; statuses & due dates.
5. **Payments & debtors:** record `payments`; aged-debtor view.
6. **Cashflow forecast:** project programme + fees into expected cash-in on the timeline.
7. **Portfolio dashboard** across all projects; then optional costs/timesheets for profitability/WIP.

### 15. Portugal / compliance notes (flag early)
- **IVA (VAT):** standard rate currently **23%** — model `vat_rate` per invoice/line; some services/exports
  differ. Don't hard-code.
- **Legal e-invoicing:** Portuguese fiscal invoices require **certified invoicing software**, **sequential
  numbering**, **ATCUD + QR code**, and **SAF-T (PT)** export. Treat this dashboard initially as the
  **management / forecasting / WIP layer**; either (a) integrate with certified invoicing software (e.g.
  via API) for the legal document, or (b) reconcile against it. Confirm the compliance approach with the
  accountant before issuing real fiscal invoices from a custom system.
- **Multi-currency:** store `currency` per project/invoice even if mostly EUR.

### 16. Security & data
- Move off per-device `localStorage` to a DB with **auth + RLS**; add **audit fields** (created/updated by,
  timestamps) on financial tables.
- Keep a **JSON export/backup** path (the current Backup button is the prototype of this).
- Financial data is sensitive — restrict by role and company; back up the database.

### 17. Glossary
- **RIBA 2020** — UK Royal Institute of British Architects Plan of Work stages 0–7.
- **Ordem dos Arquitetos (OA)** — Portuguese architects' institute phase naming (Programa Base, Estudo
  Prévio, Anteprojeto/Licenciamento, Projeto de Execução, Assistência Técnica/Obra…).
- **WIP** — Work in Progress: work performed but not yet invoiced.
- **Aged debtors** — outstanding (unpaid) invoices grouped by how long they've been overdue.
- **SAF-T (PT)** — Standard Audit File for Tax, the Portuguese fiscal data export standard.

---

*Maintainer note:* the dashboard is intentionally dependency-free and single-file. When adding the
financial layer, keep the schedule editor simple and let the database/financial modules grow alongside it,
sharing the `(project, element, phase)` coordinate that already ties the whole programme together.
