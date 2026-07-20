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

**Document-derived tiering:** when a contract is imported (Word/PDF), the same
four-axis model is pre-filled from extracted document text (obligation
language, KPI/SLA clauses) rather than manual entry, with an AI-refine option
before the suggested tier is accepted into the portfolio.

**Maturity model:** a separate 15-question, 6-dimension assessment (Foundation
& Data, Obligations, KPIs & SLAs, Rebates, Change & Termination, QBR &
Review, Stakeholder & Spend, Collaboration), scored 1–5 and re-run
periodically to track organisational CLM maturity over time — distinct from
the per-contract tiering score, and rolled up into the Governance view
alongside it.

## Inputs → Process → Outputs

- **Inputs:** manual contract scoring, or Word/PDF contract text (obligation
  and KPI/SLA clause extraction, capped at 60 pages per document); maturity
  questionnaire responses
- **Process:** weighted four-axis tiering; AI-assisted document extraction
  and tier suggestion; maturity scoring across 6 dimensions
- **Outputs:** contract tier assignment, obligations/SLA register with
  status tracking, QBR narrative drafts, CPO-facing governance rollup
  (tier mix, overdue count, weakest maturity dimensions)

## Why this approach (rationale)

Applying the same weighted-tiering discipline from category management to
contracts closes a gap most CLM tools leave open — treating contract
governance effort as a byproduct of tier rather than a flat process. Offering
both a default naming scheme and the sector-recognised GCF (Gold/Silver/
Bronze) labels keeps the tool usable in organisations with an established
naming convention, without changing the underlying scoring logic.

## Version history

- v1.5.0 [current]: full 0–7 framework, four-axis tiering with dual naming
  schemes, AI-assisted document import/tiering, maturity model, governance
  rollup
