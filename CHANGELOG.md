# Changelog

v2.6.0
What's in this pass:

Multi-QBR — STATE.qbrs[] collection, picker UI (Open/Delete/+ New), backward-compatible migration from old single-QBR saves, demo data now seeds 2 QBRs, and Governance now aggregates breaches and "top-tier contract with no QBR at all" across every saved QBR, not just whichever one's open.
Two-page Contract Brief — new screen, contract dropdown pulls tiering/register/rebates/feedback/QBR status together automatically, AI-drafted handover narrative, Word export.
Renewal & Obligation Watch Agent — lives on Governance, scans every open register item across the whole portfolio (not just the tab you happen to be on), buckets into critical/urgent/upcoming, with an optional AI-prioritized action brief on top.

Version bumped to 2.6.0, About/What's New, demo tour, and GUIDE_TIPS all updated to match.

v2.5.0
What's in this pass:

About screen → chevron stepper. The 8 pillar cards are now a step-through: dots + ‹ Previous / Next › chevrons, one section at a time, starting at Section 0. Content unchanged, just no longer a long scroll.
Contract Summary Intake (new screen). A guided, no-upload form: contract/supplier name, the same 4-axis tier sliders as Tiering, one-per-line Obligations/SLAs (with optional (YYYY-MM-DD) due-date parsing), and optional rebate terms. One submit pushes into tieringBulk, register, and rebates in a single pass, and sets it as the active contract (so QBR/Register/Rebates pick it up) — without ever touching document parsing or storing the actual contract text. Positioned explicitly on the About page ("No document to upload? Start here →") and in the demo tour, so it's a clear alternative path,

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
