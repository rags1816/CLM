# Changelog

All notable changes to CLM Suite are recorded here, most recent first. For
version history prior to this file's creation, see the **Version history**
section of `METHODOLOGY.md`.

## v2.8.1 [current] — Fixes from the first full E2E QA pass

The v2.8.0 UAT-driven pass got its own follow-up: a full end-to-end QA
pass (`docs/QA_TESTING_PROMPT_v2.8.0_FULL.md`) against the live app found
6 real issues, all fixed here.

**Contract Intake — AI-assisted suggestion crashed in Sandbox mode:**
- `sandboxAnswer()` always returns a fixed prose template regardless of
  the prompt, but the intake handler always ran `JSON.parse()` on
  whatever came back — so "Suggest with AI" was completely non-functional
  in Sandbox, the default/recommended mode. Added `sandboxSuggestContract()`,
  a proper offline heuristic (keyword-based tier scoring, category
  guessing, name/obligation/SLA extraction from the description text) that
  returns a real structured object without a network call, following the
  same honest-about-being-a-heuristic pattern already used for bulk
  contract scoring. Sandbox output is now visibly labelled as a rough
  heuristic pass, distinct from a genuine AI read.
- The category-guessing heuristic's first version had a real bug of its
  own — matching a keyword derived from the category label as a bare
  substring (`"it"` from `"IT & Technology"`) matched inside unrelated
  words like `"with"`. Replaced with an explicit keyword list per
  category, each matched with proper word boundaries.

**Contract Intake — offline CSV template, two silent-failure bugs fixed:**
- A file with no header row had its first real contract silently dropped
  (the parser always sliced off row 0, assuming it was a header). The
  parser now only treats row 0 as a header if it actually reads like
  "Name" — a headerless file is parsed as all-data instead, with a
  warning shown if no header was detected.
- A non-CSV file (e.g. a binary file renamed `.csv`) previously created
  garbage contract records with mojibake names, no error shown. The
  parser now checks for a high ratio of control characters or the
  Unicode replacement character — a tell-tale sign of binary content read
  as text — and refuses to parse rather than manufacturing records from
  noise. Individual rows with a suspiciously long or corrupted name are
  also filtered out even from an otherwise-valid file.
- The success/error message after a CSV import was being wiped
  immediately by the `render()` call that runs right after — the status
  div's default empty template replaced it before it was ever seen. Now
  persisted in a small module-level variable that survives the re-render.

**"Contract Playbook" rename — 3 missed spots:**
- The About screen's "Read the framework brief" button, the Playbook
  screen's own "Export framework brief to Word" button, and the exported
  Word document's title ("CLM Framework Brief") were all missed by the
  original rename pass's fixed search strings. All three now correctly
  say "Contract Playbook".

## v2.8.0 — UAT feedback pass: architecture consolidation + 31 review points

A first full round of UAT (screen-by-screen, with the demo tour) surfaced
31 distinct points, ranging from a real architectural gap to copy
clarifications. All 31 are addressed in this release.

**Contract creation architecture — the biggest change in this release:**
- **Contract Intake is now the single creation surface.** 0 · Tiering's
  duplicate manual sliders are removed; that screen is now upload-only,
  showing a read-only score for the active contract and linking to
  Contract Intake for anything else. Every creation path — AI-assisted
  description, manual entry, the new offline CSV template, and document
  upload — now runs through one shared `createContractRecord()` function,
  so all four produce an identical downstream shape (tieringBulk row +
  Register items + optional rebate + QBR record) instead of subtly
  different ones.
- **Live tier preview before commit.** Contract Intake's tier sliders now
  show a live-updating tier badge and score as you adjust them — nothing
  is created until "Confirm & Create Contract Record" is clicked, so you
  can trial different inputs freely first.
- **Contract Duration and Category fields added** to Contract Intake
  (start/end date pickers; category is a `<datalist>` — a dropdown of
  common categories with free-text override).
- **Offline CSV template — a 4th intake path.** Download a blank
  template, fill it in offline (Excel, Sheets, anything that saves plain
  CSV), re-upload it. Supports multiple contracts in one file; parsed
  with a small hand-written CSV parser (handles quoted fields with
  embedded commas) rather than pulling in a new charting/spreadsheet
  library.
- **0 · Tiering document-upload now auto-creates/updates its portfolio-
  table row** (this was actually shipped in v2.7.1 — see below — and is
  now the standard behavior of the consolidated architecture too).

**Renames & navigation:**
- **"Framework" renamed to "Contract Playbook"** everywhere user-facing
  (nav label, screen heading, cross-references). Internal function/
  variable names remain `framework`/`fw` — low-risk, code-only.
- **Coverage & Feedback moved ahead of QBR in the nav** — QBR reads
  stakeholder feedback from Coverage, so Coverage now correctly appears
  before it (this was backwards since the screen was first added).
- **QBR's Supplier field is now a dropdown** sourced from the Tiering
  portfolio table (with an "Other" free-text fallback), instead of free
  text that risked a typo creating an orphaned QBR.

**Fixed a real duplicate-entry bug:**
- **Contract Playbook's "+ Add starter" buttons are now genuinely
  idempotent.** Previously, re-clicking the same button (e.g. after
  navigating away and back) could silently add a second copy of the same
  starter obligations/KPIs/SLAs/rebate tiers, because the only guard was
  a UI-level disabled state that reset on re-render. Now checked against
  existing Register/QBR/Rebate entries for that specific contract before
  adding — re-clicking for the same contract adds nothing new; a
  different contract still gets its own set.

**Demo tour:**
- **Resyncs when you navigate manually.** Previously, clicking a
  different tab mid-tour (instead of using the tour's own Next/Prev)
  left the tour card narrating stale content for whichever screen it
  last pushed you to. `setScreen()` now snaps the tour to the nearest
  matching step for wherever you actually navigate, without interfering
  with the tour's own Prev/Next buttons.
- **Reordered to match the corrected nav sequence** (Coverage before
  QBR), and the Tiering/Intake/Playbook steps rewritten to reflect the
  consolidated architecture above.

**Clarity & source-of-data fixes (from UAT points that were "already
correct behavior, just undocumented"):**
- Rebates: empty-state copy no longer implies a document must be active
  for manual entry (only "AI Extract Rebates" needs one); added a
  one-line note on where table data actually comes from.
- QBR: added a note stating the AI narrative and Word/PPT exports are
  built from that QBR's own KPI/SLA/Risk/Action fields plus Coverage
  feedback — and sharpened the "no active document" warning specifically
  for Sandbox mode, since a template narrative with no source document is
  meaningfully weaker than one grounded in real contract text.
- Maturity Scorecard: added a note stating the assessment is entirely
  manual (never AI-generated), one-time-per-cycle, and organisation-wide
  rather than tied to a specific contract.
- Register: added an explanation of why Change Order is a Type within
  this register rather than its own screen, and clarified that clause
  extraction detects obligation/SLA language only, not change-order
  language.
- Cross-Contract Pattern Scan: now states its scope explicitly (only
  contracts on the portfolio table, with a live count).
- Governance charts: tier-mix donut and maturity radar headings now state
  their scope explicitly ("whole portfolio" / "organisation-wide") rather
  than reading as if they could be per-contract.
- Rebates bar chart: fixed label truncation on long contract/tier names
  (added ellipsis truncation + widened the label column) and added a
  subtitle clarifying the bars represent spend × rate, not raw spend.

**New feature — Contract Brief QBR summary option:**
- Added an opt-in (off by default) "Include the latest QBR's narrative"
  checkbox on Contract Brief — useful for senior management briefing
  (e.g. at renewal or during a crisis) without duplicating the KPI/SLA
  tables the Brief already shows. Included in both the on-screen view and
  the Word export when checked.

**Configurability:**
- Renewal & Obligation Watch Agent's urgent/upcoming day cutoffs (30/90
  by default) are now user-editable inputs, persisted in state, instead
  of hardcoded.

## v2.7.1 — Entry-point sequencing fixes

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
