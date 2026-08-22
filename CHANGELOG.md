# Changelog

All notable changes to CLM Suite are recorded here, most recent first. For
version history prior to this file's creation, see the **Version history**
section of `METHODOLOGY.md`.

## v2.7.1 [current] — Entry-point sequencing fixes

Two inconsistencies identified during UAT review (multiple "add a
contract" / "seed data" entry points producing different downstream
shapes) were fixed:

- **Tiering document-upload now auto-creates/updates its portfolio-table
  row.** Previously, uploading a real contract document at 0 · Tiering
  only set the single-contract score (`tieringSingle`) and opened/created
  a QBR — it did not add a row to the bulk table, so the contract wouldn't
  appear on the portfolio heatmap without a separate manual "+ Add bulk
  row" step. Contract Intake already did this in one action; Tiering-
  upload now matches it via a new `upsertTieringBulkRow()` helper, which
  updates the existing row in place (matched by normalized name) on a
  re-upload rather than creating a duplicate. Register's "Add to Tiering
  table" button (for a document reviewed on the Register screen) now
  routes through the same helper.
- **Framework's "+ Add starter" quick-add now requires a named target
  contract.** Previously, quick-added Register items (obligations, change
  orders) were written with a blank `contract` field, and quick-added
  KPIs/SLAs were written to whichever QBR happened to be open in the
  editor at the time — regardless of which supplier that was. A new
  "Target contract" dropdown on the Framework screen (sourced from the
  Tiering bulk table) is now required before any quick-add button is
  enabled; obligations/change orders/rebates carry that contract's name,
  and KPIs/SLAs are routed to that specific contract's QBR record via
  `openOrCreateQbrFor()`, not the currently-open one.

## v2.7.0 — Multi-step AI intake agent & cross-contract pattern scan

**Multi-step contract intake agent:**
- Added a "0 · Describe it (AI-assisted)" step at the top of Contract
  Summary Intake. Describe a contract in plain English and the AI returns
  a structured suggestion (tier scores, contract identity, obligations,
  SLAs, rebate terms) via a single call, which pre-fills the existing
  intake fields directly for review.
- Nothing is created or saved at this step — the person still reviews and
  edits every field, then commits via the existing "Create contract
  record" action. This chains contract description → AI-suggested
  tiering/obligations/SLAs/rebate terms → one human review/approve step,
  rather than five separate manual actions across screens.

**Cross-Contract Pattern Scan:**
- New card on Governance, below the Watch Agent. Builds a compact
  per-contract summary (tier, overdue count, rebate dispute status,
  average feedback rating, a sample of register/feedback text) across the
  whole portfolio and sends it in a single AI call, looking for recurring
  themes a single-contract view can't surface — e.g. the same issue type
  overdue on multiple suppliers, a repeated complaint theme, clustered
  rebate disputes.
- Explicitly prompted to say "nothing recurs" rather than invent a
  pattern when the data doesn't support one. Requires at least 2 contracts
  with register/feedback data on file; the scan button stays disabled
  otherwise.

## v2.6.0 — Multi-QBR, Contract Brief, and the Renewal & Obligation Watch Agent

**Multi-QBR:**
- Replaced the single `STATE.qbr` object with a `STATE.qbrs[]` collection,
  each keyed by id, holding its own KPIs, SLAs, narrative, and
  recommendations — so multiple suppliers each keep their own QBR record
  instead of one overwriting the last.
- Added a QBR picker at the top of the QBR screen (Open / Delete / + New),
  with a backward-compatible one-time migration that files any pre-v2.6
  single QBR into the new collection on first load, rather than dropping
  it.
- Governance's risk/to-do lists now aggregate across **every** saved QBR
  — flagging suppliers with a breached KPI/SLA on record, and Strategic/
  Gold-tier contracts that have no QBR record at all — instead of only
  seeing whichever QBR happened to be open.
- Demo data now seeds two QBRs (Global Logistics Co, Meridian Cloud
  Hosting) so the multi-QBR picker has something to demonstrate out of
  the box.

**Two-page Contract Brief (new screen):**
- A per-contract handover document distinct from the QBR (a periodic
  performance review) and the Framework brief (the whole methodology) —
  this is the summary a procurement manager hands to the contract manager
  taking over day-to-day ownership.
- Select a contract from a dropdown and it auto-assembles: tier and
  weighted score, the four axis scores, next key date from the Register,
  rebate standing, existing QBR status, average stakeholder rating, top
  risks, open register items, recent feedback, and recommended next
  actions for the incoming owner — all pulled live from Tiering, Register,
  Rebates, Coverage, and QBR data already on file.
- "Draft handover narrative" writes a short AI paragraph grounded in what's
  actually on file (offline-safe recommendations list underneath either
  way); exports to Word.

**Renewal & Obligation Watch Agent:**
- New card on Governance. Scans every open Register item across the
  **whole portfolio** — not just whichever contract's tab happens to be
  open — and buckets anything overdue or due soon into critical
  (overdue) / urgent (≤30 days) / upcoming (≤90 days), sorted soonest
  first.
- "AI prioritise" adds an optional short, directive action brief on top
  of the heuristic list, naming which 3–5 items to act on first and why.

**Also in this pass:**
- About screen reworked as a step-through with chevrons (‹ Previous /
  Next ›) and dot navigation across the 8 pillar cards, rather than one
  long scroll.
- Contract Summary Intake (new screen, first shipped as a no-document
  path): type a contract summary directly instead of uploading the signed
  contract — this app is explicitly not meant to be a contract
  repository. One submission seeds Tiering, the Register, and Rebates in
  a single pass, and sets the new contract as active so QBR/Register/
  Rebates pick it up automatically.
- Version, About "What's new", demo tour, and floater `GUIDE_TIPS` all
  updated to match each of the above.

## v2.5.0 — About screen chevron stepper & Contract Summary Intake

*(Superseded in content by v2.6.0's expansion of the same two features —
recorded here for the version history record.)*

- **About screen → chevron stepper.** The 8 pillar cards became a
  step-through: dots + ‹ Previous / Next › chevrons, one section at a
  time, starting at Section 0. Content unchanged, no longer a long scroll.
- **Contract Summary Intake (new screen).** A guided, no-upload form:
  contract/supplier name, the same 4-axis tier sliders as Tiering,
  one-per-line Obligations/SLAs (with optional `(YYYY-MM-DD)` due-date
  parsing), and optional rebate terms. One submit pushes into
  `tieringBulk`, `register`, and `rebates` in a single pass, and sets it
  as the active contract — without ever touching document parsing or
  storing the actual contract text. Positioned explicitly on the About
  page ("No document to upload? Start here →") and in the demo tour.

## v2.4.0 — Reliability & workflow-integrity pass

**Audit-driven fixes** (from a pre-contract → post-award workflow integrity audit):
- AI calls (Claude/Gemini) now time out after 30s with a clear error and an
  optional fallback provider, instead of hanging indefinitely.
- Fixed a timezone-dependent false positive/negative in the Register's
  "Overdue" status calculation.
- Long obligation/KPI/SLA clauses that exceed the extraction length limit
  are now counted and flagged instead of silently dropped.
- Replaced the old 6,000-character single-call document review with a full
  chunked map-reduce review that covers the entire uploaded document, with a
  cost estimate and a quick/full choice offered for long documents. Always
  uses Gemini directly for this specific call, regardless of the provider
  set in Settings, so the cost estimate stays accurate.

**Contract identity & review caching (Phase 1):**
- The active contract now carries a stable id, threaded through Register and
  QBR so linked items match by identity rather than by supplier-name string
  alone (name-matching is kept as a backward-compatible fallback).
- A single chunked document review is now cached per active contract and
  reused automatically across Tiering, Register, Rebates, QBR, and
  Framework — instead of re-running a full AI review from scratch at each
  of those five trigger points.

**Attribution accuracy:**
- The Maturity/Scorecard screen and Framework brief now cite CIPS, The
  Hackett Group, and WorldCC by name with specific, independently verified
  figures where the app's benchmarking references draw on their published
  work, and soften language where no verbatim source was confirmed.

**Mobile navigation:**
- Replaced the overflowing top nav with a collapsed hamburger/drawer menu
  below a 768px breakpoint, auto-closing on tab selection; desktop nav is
  unaffected.

**Phase 2 — nav, tiering consistency, and data-integrity fixes:**
- Reordered the nav so Register and Rebates (which QBR reads from) come
  before QBR.
- Register's suggested tier now reads from the cached full document review
  when one exists for the active contract, instead of running its own
  independent heuristic/truncated-AI estimate — so Tiering's auto-score and
  what lands in the portfolio table are guaranteed to be the same number.
- Added the same active-contract badge Tiering/Register already had to QBR
  and Rebates.
- `getTier` no longer silently downgrades a corrupted/NaN tiering score to
  the lowest-oversight tier (Routine/Leverage) — it now surfaces a visible
  "Data error" state, and Governance's rollup explicitly flags excluded
  contracts as a named risk rather than mis-tiering them.
- A malformed-but-200 AI response (safety-filtered, empty, or otherwise
  unusable) now raises a clear, visible error instead of silently resolving
  to placeholder text.
- Added optional UNSPSC/CPV manual-entry classification-code fields to the
  Tiering portfolio table, matching the established pattern in CategoryAI
  (manual entry only, no lookup/validation).
