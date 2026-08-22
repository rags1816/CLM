# CLM Suite — Methodology

## Origin

Developed by Vijay L Narasimhan, 2025–26, applying the same portfolio-tiering
discipline used in category management (see the Category AI methodology) to
the contract lifecycle — where contracts are typically tracked administratively
(expiry dates, renewal alerts) rather than actively tiered by strategic
importance.

## Problem this solves

Most contract management tooling tracks dates and documents but doesn't
differentiate management effort by contract importance — a strategic supplier
contract and a low-value routine one often get the same (or no) structured
review cadence. This tool applies a weighted scoring model to tier contracts
by actual business importance, then scales governance activity (QBR cadence,
named ownership, review frequency) to match the tier.

## Built on / credited frameworks

- **Framework: Kraljic-style portfolio tiering logic** (as also applied in
  the Category AI tool, adapted here for contracts rather than spend
  categories)
  - What it contributes: the principle of differentiated management effort
    by strategic importance rather than uniform treatment
  - How this tool adapts it: uses a four-axis weighted score (Value, Risk,
    Criticality, Business Impact) specific to contract management, rather
    than the traditional two-axis (supply risk / profit impact) category
    model

- **Framework: Gold/Silver/Bronze (GCF) tiering** — recognised UK
  public-procurement contract management naming convention
  - What it contributes: an alternative, sector-recognised label set for the
    same underlying tiers (a 3-band scheme, so Transactional and Routine
    both map to Bronze)
  - How this tool adapts it: offered as a swappable naming scheme over the
    same underlying score — the label changes, the scoring logic doesn't

- **Reference benchmarks: CIPS / Hackett / WorldCC** — cited in the Framework
  screen as external benchmarking references for the Section 0–7 brief
  (Tiering → Obligations → KPIs → SLAs → Rebates → Change Orders →
  Termination → QBR)

## Core framework (your original model)

**Tiering score:** four axes, each scored 1–5:
- Value — weighted 30%
- Risk — weighted 30%
- Criticality — weighted 20%
- Business Impact — weighted 20%

The weighted composite auto-assigns one of four tiers:

| Tier | Management approach |
|---|---|
| Strategic | Full QBR cadence, named owner, joint business plan |
| Managed | Semi-annual review, automated alerts |
| Transactional | Annual/exception-based review |
| Routine / Leverage | Compliance-only, no active management |

**Document-centric workflow:** Uploading a contract document at **Step 0 (Tiering)** pre-fills the four-axis model based on document content and sets a global active contract context. The app supports uploading multiple documents sequentially (appending their text under a combined title) to analyze entire contract packages (e.g. Master Agreement + SLA schedules) as one. This context then powers downstream AI extraction for QBR KPIs/SLAs, Rebate tiers, and staged Obligations automatically, reducing manual input and duplicate entries.

**No-document intake path (v2.5+):** Recognising that this tool is
deliberately *not* a contract repository or document store, Contract
Summary Intake offers an equally-supported second entry point: a typed
summary (contract identity, tier inputs, obligations/SLAs, rebate terms)
that seeds the same downstream screens the document-upload path does,
without ever requiring the signed contract text itself to be pasted in or
stored. From v2.7, this path also accepts a plain-English description,
which an AI step converts into the same structured suggestion for human
review before anything is committed — see **Agentic layer** below.

**Maturity model:** a separate 15-question, 6-dimension assessment (Foundation
& Data, Obligations, KPIs & SLAs, Rebates, Change & Termination, QBR &
Review, Stakeholder & Spend, Collaboration), scored 1–5 and re-run
periodically to track organisational CLM maturity over time — distinct from
the per-contract tiering score, and rolled up into the Governance view
alongside it.

**Multi-QBR model (v2.6+):** QBRs are held as a collection, one record per
supplier/period, rather than a single overwritable record — reflecting
that a real CLM program runs concurrent, recurring reviews across a
portfolio, not one review at a time. Governance's risk aggregation reads
across the full collection, so a breach on any supplier's QBR — and any
top-tier contract missing a QBR entirely — surfaces at the portfolio level
regardless of which QBR record happens to be open at the time.

## Agentic layer (v2.6+)

Early versions of this tool used AI reactively: a person clicked a button,
one AI call ran against whatever was on the current screen, and the result
came back for review. From v2.6 onward, three functions shift toward
proactive, portfolio-wide, or multi-step AI use — the original contribution
here being *where* and *how* agentic behaviour is scoped, not the
underlying model capability:

- **Renewal & Obligation Watch Agent** — runs a heuristic scan (no AI call
  required) across every open Register item in the whole portfolio,
  independent of which contract's tab is open, and surfaces anything
  overdue or approaching its due/renewal date without the person needing
  to notice a badge. An optional AI call on top turns the ranked list into
  a short, directive action brief.
- **Multi-step contract intake agent** — chains a single plain-English
  description through one AI call into a structured multi-field suggestion
  (tier axes, obligations, SLAs, rebate terms) spanning what would
  otherwise be several separate manual entry steps across different
  screens, collapsing them into one human review/approve step rather than
  removing the review step altogether.
- **Cross-Contract Pattern Scan** — the one function in this layer that is
  genuinely portfolio-wide in its AI reasoning: a single call is given a
  compact summary across *all* contracts on file and asked to identify
  recurring themes across multiple contracts, explicitly instructed to
  report "nothing recurs" rather than manufacture a false pattern when the
  data doesn't support one.

Each of these deliberately keeps a human decision point before anything is
written to persistent state (Register, Tiering, Rebates, QBR) — the agentic
layer accelerates drafting and surfacing, it does not automate commitment.

## Inputs → Process → Outputs

- **Inputs:** manual contract scoring, a typed Contract Summary Intake
  (optionally AI-assisted from a plain-English description), or Word/PDF
  contract text uploaded at Step 0 (supports multiple files, up to 350
  pages per document, with text stored up to 1,200,000 characters);
  maturity questionnaire responses
- **Process:** weighted four-axis tiering; AI-assisted document parsing or
  plain-English intake to seed active contract state; maturity scoring
  across 6 dimensions; portfolio-wide renewal/obligation scanning and
  cross-contract pattern detection
- **Outputs:** contract tier assignment, obligations/SLA register with
  status tracking, extracted QBR KPIs & SLAs held per supplier/period
  across a QBR collection, active rebate tiers, QBR narrative drafts, a
  per-contract two-page Contract Brief for procurement-to-contract-manager
  handover, CPO-facing governance rollup (tier mix, overdue count, weakest
  maturity dimensions, cross-QBR risk aggregation, proactive renewal
  alerts, cross-contract pattern findings)

## Why this approach (rationale)

Applying the same weighted-tiering discipline from category management to
contracts closes a gap most CLM tools leave open — treating contract
governance effort as a byproduct of tier rather than a flat process. Offering
both a default naming scheme and the sector-recognised GCF (Gold/Silver/
Bronze) labels keeps the tool usable in organisations with an established
naming convention, without changing the underlying scoring logic. Keeping
this tool an intelligence layer rather than a system of record — no document
repository, no approval workflow, no audit trail — is a deliberate scope
boundary: those belong to ERP/CLM platforms (Ariba, Atamis, Ivalua and
similar), and this tool is designed to sit on top of that data, or to work
standalone via typed summaries, rather than compete with it.

## Version history

- **v2.7.0 [current]**: Multi-step AI intake agent on Contract Summary
  Intake (plain-English description → AI-suggested tier/obligations/SLAs/
  rebate terms → human review/commit), and a Cross-Contract Pattern Scan
  on Governance that looks for recurring themes across the whole
  portfolio in one AI call. See `CHANGELOG.md` for the full breakdown.
- **v2.6.0**: Multi-QBR (`STATE.qbrs[]` collection with a picker, one
  record per supplier/period, cross-QBR risk aggregation on Governance), a
  new two-page Contract Brief for procurement-to-contract-manager
  handover, and a Renewal & Obligation Watch Agent that proactively scans
  the whole portfolio's Register for anything overdue or due soon.
- **v2.5.0**: About screen reworked as a chevron step-through; Contract
  Summary Intake introduced as a no-document entry path seeding Tiering,
  Register, and Rebates from a typed summary.
- **v2.4.0**: Reliability & workflow-integrity pass — AI-call timeout/fallback handling, timezone-safe overdue calculation, full chunked map-reduce document review (replacing a 6,000-character truncation), stable contract identity with cached review reuse across tabs, verified CIPS/Hackett/WorldCC attribution wording, mobile hamburger nav, tiering-pathway consolidation, `getTier` data-integrity handling, and optional UNSPSC/CPV tagging. See `CHANGELOG.md` for the full breakdown.
- **v2.3.0**: Document-centric workflow enabling contract uploads at Step 0, driving downstream AI/heuristic extraction (KPIs, SLAs, rebate tiers, obligations) across all tabs; persistent Framework screen inputs; detailed API connection error reporting with dropdown model selectors.
- **v2.1.0**: Canvas-based charts rendering inside QBR and Governance screens; automated governance rollups including risks and recommendations from active tab metrics; register linking via supplier name columns.
- **v1.5.0**: Initial release featuring the core 0-7 framework brief, Kraljic-style weighted scoring model, local browser state, and basic file upload staging.
