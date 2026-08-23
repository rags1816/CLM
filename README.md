# CLM Suite

A single-file, self-contained Contract Lifecycle Management tool for tiering
contracts, tracking obligations/SLAs, running QBRs, and rolling everything up
into a CPO-facing governance view. Deliberately an intelligence layer, not a
contract repository or system of record — see `METHODOLOGY.md` for the scope
rationale.

## What it does

- **Contract Intake:** The single place to create a contract record — four equally-supported paths, all producing the same downstream record (a tier score, Register items, an optional rebate tier, a QBR): describe it in plain English (AI-assisted, with a live tier-preview badge before you confirm anything), fill it in manually, download/fill/re-upload an offline CSV template (supports multiple contracts per file), or upload the real document.
- **0 · Tiering:** Upload a contract document (.docx or .pdf) here to auto-score it across four axes — Value, Risk, Criticality, Business Impact — with support for standard or Gold/Silver/Bronze schemes; this also sets the active contract context used across QBR, Rebates, and Register. This screen is upload-only — for manual/AI/template entry, use Contract Intake. The portfolio table and Value×Risk heatmap here are the combined *output* of every intake path.
- **Contract Playbook:** The full Section 0–7 brief with CIPS / Hackett / WorldCC benchmarking references and AI-drafted summaries. "+ Add starter" buttons seed Register/QBR/Rebates for a chosen target contract — idempotent, so re-clicking for the same contract won't create duplicates.
- **Obligations/SLA/Change Order Register:** One register for all three types — automatically stages candidate clauses parsed from an active document for review, or add items directly via Contract Intake without a document. Change Orders live here (not a separate screen) so scope drift stays with the obligations/SLAs it affects.
- **Rebate Tracker:** Manual entry, Contract Intake, the offline template, or AI-extraction from an active document — calculates accrued entitlement live from spend × rate.
- **Coverage & Feedback:** Category spend coverage and stakeholder feedback — feeds into QBR (and appears before it in the nav, since QBR reads from it).
- **QBR Builder:** Holds a collection of QBRs — one record per supplier/period, switchable via a picker, with the Supplier field itself a dropdown sourced from the tiered portfolio — each extracting SLA/KPI parameters from its active contract via AI (or sandbox regex heuristics) and drafting performance narratives.
- **Contract Brief:** A two-page, per-contract handover document — the summary a procurement manager hands to the contract manager taking over ownership. Auto-assembles tier standing, key dates, rebate status, top risks, recent feedback, and recommended next actions; optionally includes the latest QBR's narrative for senior briefing (e.g. at renewal or during a crisis) without duplicating data; exports to Word.
- **Maturity Assessment:** 15 questions across 6 dimensions, entirely manual and organisation-wide, tracked over time.
- **Governance Rollup:** Live CPO-facing rollup (donut tier mix, maturity radar, spend coverage, overdue obligations, risks, and recommendations) exportable to Word or PowerPoint, including:
  - **Renewal & Obligation Watch Agent** — proactively scans every open Register item across the whole portfolio for anything overdue or due soon (day cutoffs are user-configurable), with an optional AI-prioritised action brief.
  - **Cross-Contract Pattern Scan** — a single AI call looks across all contracts on the portfolio table for recurring themes (repeated issue types, complaint patterns, clustered rebate disputes) that a single-contract view can't surface.

## Status

v2.8.0. Active document-centric single-file app; known gaps documented in `ADMIN_GUIDE.md`. See `CHANGELOG.md` for version history.

## Tech stack

Single `index.html` file — no server, backend, or build step. AI assistance via user-supplied Claude or Gemini API key (dropdown select or custom names, stored in browser `localStorage`), with detailed API response error logs and an offline "Sandbox" mode as default fallback. Optional CDN-loaded libraries (`docx`, `pptxgenjs`, `mammoth`, `pdfjs-dist`) load on-demand for imports/exports. The offline contract-intake CSV template uses a small hand-written parser — no spreadsheet library dependency.

## How to run

Download and double-click `index.html` — opens in Chrome or Edge. No installation, no account, no login. See `USER_GUIDE.md` for screen-by-screen details.

## Development note

Development assisted by Claude Code (Anthropic) under my direction. The
methodology, product design, and domain expertise reflected in this tool
are my own — see `METHODOLOGY.md` for the original framework.

## Related

See [`METHODOLOGY.md`](./METHODOLOGY.md) for details on the portfolio and maturity models.
