# CLM Suite

A single-file, self-contained Contract Lifecycle Management tool for tiering
contracts, tracking obligations/SLAs, running QBRs, and rolling everything up
into a CPO-facing governance view.

## What it does

- **Tiering (0):** scores a contract across four axes — Value, Risk,
  Criticality, Business Impact — and auto-assigns a management tier
  (Strategic / Managed / Transactional / Routine), with an optional
  Gold/Silver/Bronze (GCF-style UK public-procurement) naming scheme
- **Framework:** the full Section 0–7 brief (Tiering → Obligations → KPIs →
  SLAs → Rebates → Change Orders → Termination → QBR), with CIPS / Hackett /
  WorldCC benchmarking references and AI-drafted executive summaries
- **Obligations/SLA register:** manual entry or AI-assisted import from Word/
  PDF contracts — extracts obligation language and KPI/SLA clauses, stages
  them for review, and suggests a tier from the document text
- **QBR builder:** structured Quarterly Business Review (KPIs, SLAs, rebates,
  risks, actions) with AI-drafted narrative
- **Maturity assessment:** 15 questions across 6 dimensions, tracked over time
- **Governance rollup:** live CPO-facing view — tier mix, overdue obligations,
  weakest maturity dimensions — exportable to Word or PowerPoint

## Status

v1.5.0. Working single-file app; known gaps documented in `ADMIN_GUIDE.md`
(no OCR for scanned PDFs, no shared/multi-user workspace, no server-side key
management).

## Tech stack

Single `index.html` file — no server, backend, or build step. AI assistance
via user-supplied Claude or Gemini API key (stored in browser `localStorage`
only), with an offline "Sandbox" mode as default fallback. Optional CDN-loaded
libraries (`docx`, `pptxgenjs`, `mammoth`, `pdfjs-dist`) for document
import/export — core tiering, QBR, register, and governance features work
fully offline.

## How to run

Download and double-click `index.html` — opens in Chrome or Edge. No
installation, no account, no login. See `USER_GUIDE.md` for a screen-by-screen
walkthrough and `ADMIN_GUIDE.md` for deployment, AI provider setup, and
troubleshooting.

## Related

See [`METHODOLOGY.md`](./METHODOLOGY.md) for the tiering and maturity
frameworks this tool applies.
