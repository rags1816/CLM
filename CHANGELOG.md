# Changelog

All notable changes to CLM Suite are recorded here, most recent first. For
version history prior to this file's creation, see the **Version history**
section of `METHODOLOGY.md`.

## v2.4.0 [current] — Reliability & workflow-integrity pass

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
