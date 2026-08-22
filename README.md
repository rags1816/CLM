# CLM Suite

A single-file, self-contained Contract Lifecycle Management tool for tiering
contracts, tracking obligations/SLAs, running QBRs, and rolling everything up
into a CPO-facing governance view. Deliberately an intelligence layer, not a
contract repository or system of record — see `METHODOLOGY.md` for the scope
rationale.

## What it does

- **Document-Centric Workflow (0):** Upload a contract document (.docx or .pdf) at **Step 0 (Tiering)** to establish an active contract context. You can check "Append to active contract" to combine multiple files (e.g. Master Agreement + SLA schedules) under the same supplier context. The app instantly auto-scores the contract across four axes — Value, Risk, Criticality, Business Impact — and recommends a management tier, with support for standard or Gold/Silver/Bronze schemes.
- **Contract Summary Intake:** A no-document alternative to Step 0 for anyone who doesn't want to (or can't) upload the actual signed contract. Type a summary directly — contract identity, tier inputs, obligations/SLAs, rebate terms — and one submission seeds Tiering, the Register, and Rebates in a single pass. An optional AI-assisted step accepts a plain-English description and pre-fills the whole form (tier scores, obligations, SLAs, rebate terms) for review before anything is committed.
- **Framework:** The full Section 0–7 brief with CIPS / Hackett / WorldCC benchmarking references and AI-drafted summaries. Inputs on this screen are persisted automatically in local storage.
- **Obligations/SLA Register:** Automatically stages candidate clauses and obligations parsed from the Step 0 active contract for review. Allows manually tracking status, owners, and due dates.
- **QBR Builder:** Holds a collection of QBRs — one record per supplier/period, switchable via a picker — each extracting SLA/KPI parameters from its active contract via AI (or sandbox regex heuristics) and drafting performance narratives.
- **Rebate Tracker:** Extracts spend volume tiers and rates directly from the active contract and calculates accrued entitlement live from spend inputs.
- **Contract Brief:** A two-page, per-contract handover document — the summary a procurement manager hands to the contract manager taking over ownership. Auto-assembles tier standing, key dates, rebate status, top risks, recent feedback, and recommended next actions from data already on the other tabs; exports to Word.
- **Maturity Assessment:** 15 questions across 6 dimensions, tracked over time.
- **Governance Rollup:** Live CPO-facing rollup (donut tier mix, maturity radar, spend coverage, overdue obligations, risks, and recommendations) exportable to Word or PowerPoint, now including:
  - **Renewal & Obligation Watch Agent** — proactively scans every open Register item across the whole portfolio for anything overdue or due soon, with an optional AI-prioritised action brief.
  - **Cross-Contract Pattern Scan** — a single AI call looks across all contracts on file for recurring themes (repeated issue types, complaint patterns, clustered rebate disputes) that a single-contract view can't surface.

## Status

v2.7.0. Active document-centric single-file app; known gaps documented in `ADMIN_GUIDE.md`. See `CHANGELOG.md` for version history.

## Tech stack

Single `index.html` file — no server, backend, or build step. AI assistance via user-supplied Claude or Gemini API key (dropdown select or custom names, stored in browser `localStorage`), with detailed API response error logs and an offline "Sandbox" mode as default fallback. Optional CDN-loaded libraries (`docx`, `pptxgenjs`, `mammoth`, `pdfjs-dist`) load on-demand for imports/exports.

## How to run

Download and double-click `index.html` — opens in Chrome or Edge. No installation, no account, no login. See `USER_GUIDE.md` for screen-by-screen details.

## Development note

Development assisted by Claude Code (Anthropic) under my direction. The
methodology, product design, and domain expertise reflected in this tool
are my own — see `METHODOLOGY.md` for the original framework.

## Related

See [`METHODOLOGY.md`](./METHODOLOGY.md) for details on the portfolio and maturity models.
