# CLM Suite

A single-file, self-contained Contract Lifecycle Management tool for tiering
contracts, tracking obligations/SLAs, running QBRs, and rolling everything up
into a CPO-facing governance view.

## What it does

- **Document-Centric Workflow (0):** Upload a contract document (.docx or .pdf) at **Step 0 (Tiering)** to establish an active contract context. You can check "Append to active contract" to combine multiple files (e.g. Master Agreement + SLA schedules) under the same supplier context. The app instantly auto-scores the contract across four axes — Value, Risk, Criticality, Business Impact — and recommends a management tier, with support for standard or Gold/Silver/Bronze schemes.
- **Framework:** The full Section 0–7 brief with CIPS / Hackett / WorldCC benchmarking references and AI-drafted summaries. Inputs on this screen are persisted automatically in local storage.
- **Obligations/SLA Register:** Automatically stages candidate clauses and obligations parsed from the Step 0 active contract for review. Allows manually tracking status, owners, and due dates.
- **QBR Builder:** Extracts SLA/KPI parameters from the active contract via AI (or sandbox regex heuristics) and drafts performance narratives.
- **Rebate Tracker:** Extracts spend volume tiers and rates directly from the active contract and calculates accrued entitlement live from spend inputs.
- **Maturity Assessment:** 15 questions across 6 dimensions, tracked over time.
- **Governance Rollup:** Live CPO-facing rollup (donut tier mix, maturity radar, spend coverage, overdue obligations, risks, and recommendations) exportable to Word or PowerPoint.

## Status

v2.3.0. Active document-centric single-file app; known gaps documented in `ADMIN_GUIDE.md`.

## Tech stack

Single `index.html` file — no server, backend, or build step. AI assistance via user-supplied Claude or Gemini API key (dropdown select or custom names, stored in browser `localStorage`), with detailed API response error logs and an offline "Sandbox" mode as default fallback. Optional CDN-loaded libraries (`docx`, `pptxgenjs`, `mammoth`, `pdfjs-dist`) load on-demand for imports/exports.

## How to run

Download and double-click `index.html` — opens in Chrome or Edge. No installation, no account, no login. See `USER_GUIDE.md` for screen-by-screen details.

## Related

See [`clm-METHODOLOGY.md`](./clm-METHODOLOGY.md) for details on the portfolio and maturity models.
