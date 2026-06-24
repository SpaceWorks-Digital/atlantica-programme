# ATLANTICA — Project Programme Dashboard

A self-contained interactive Gantt / programme dashboard for project **26.015 Atlantica Wavepool
Cascais**. Single `index.html`, no build step, no dependencies — open it locally or host it anywhere
static (GitHub Pages, Netlify, S3, an intranet).

## What it does
- **X axis (timeline):** toggle **Daily / Weekly / Monthly / Quarterly** zoom.
- **Y axis (rows):** the 6 project elements — **Masterplan, Hub, Pool + Concourse, Hotel, Apartments,
  Landscape** — each **expand/collapse** to reveal its phase rows.
- **Phases:** dual-labelled **RIBA 2020 stage ↔ Ordem dos Arquitetos (PT)** phase:

  | RIBA 2020 | Ordem dos Arquitetos |
  |---|---|
  | 0 Strategic Definition | Programa Base |
  | 1 Preparation & Briefing | Programa Base |
  | 2 Concept Design | Estudo Prévio |
  | 3 Spatial Coordination | Anteprojeto / Licenciamento |
  | 4 Technical Design | Projeto de Execução |
  | 5 Manufacturing & Construction | Assistência Técnica / Obra |
  | 6 Handover | Conclusão / Receção |
  | 7 Use | Uso / Pós-Ocupação |

## Interaction
- **Drag a bar body** → move it in time.
- **Drag a bar edge** → set its duration.
- **Double-click a bar** → edit exact start/end dates, rename, or delete.
- **Double-click an empty phase row** → create a task there.
- **Element summary bar** spans the min–max of its phases (auto, read-only).
- Edits **auto-save to the browser** (`localStorage`). **Export / Import** round-trips the whole
  programme as JSON. **Reset** restores the seeded ATLANTICA defaults. **Print** → PDF-friendly.

## Notes
- The seeded durations are a plausible placeholder programme to start from — drag to your real dates.
- State is per-browser. Share a programme by **Export** → send the JSON → **Import**.
- Status: v1, local prototype. For online deployment just publish this folder as static files.
