## Prompt to paste into Claude Code (CLI)

This is a **full end-to-end pass**, not a delta since the last review —
run every section below against the app as it stands on `main` right now,
regardless of what any earlier QA prompt or UAT doc assumed. This prompt
supersedes `QA_TESTING_PROMPT_v2.5-2.7.1.md` **and** `CLM_E2E_Test_Script.docx`
(a v2.4.0-era script whose nav-order and feature assumptions predate
Contract Intake, Multi-QBR, Contract Brief, and everything else built
since — its one genuinely valuable piece, a network-level confidentiality
verification gate, has been folded into Section 0 below). Move both
superseded files, plus the earlier UAT test script docx, to `archive/`
once this pass is complete and closed out, per `CLAUDE.md`.

Copy everything in the fenced block below into a Claude Code CLI session
in this repo.

```
Read CLAUDE.md first and follow its workflow rules for this whole session
(no commits without a shown diff and my explicit confirmation, work on a
feature branch, flag anything that looks like a real secret/PII). Pull
latest main before starting, per CLAUDE.md's guidance on direct edits
sometimes landing there outside of PRs.

This is a full end-to-end verification of index.html at v2.8.0 — walk
through the entire app as if you'd never seen it before, not just what
changed recently. Read CHANGELOG.md and METHODOLOGY.md first for context
on what each version was meant to add, then verify the LIVE CODE actually
does it — don't take the changelog's word for it.

## 0. Confidentiality verification (run this FIRST, treat as a hard gate)
Carried forward from an earlier v2.4.0-era test script because it's a genuinely
important check that nothing since has replaced: verify Sandbox mode's "nothing
leaves your browser" claim at the network level, not just by trusting the badge.
- In Settings, select Sandbox as the Primary provider. Confirm it's selected.
- Open your browser's DevTools (F12) and go to the Network tab. Clear it.
- On 0 · Tiering, upload a real document and let it auto-score.
- Watch the Network tab throughout: confirm ZERO requests to any Anthropic
  or Google/Gemini API domain occur while Sandbox is selected.
- If any external AI-provider network request occurs while Sandbox is
  selected, STOP — this is a hard-gate failure, not a minor bug. Do not
  continue with confidential test documents until this is resolved and
  re-verified.
- Repeat the same document upload with Claude or Gemini selected instead,
  and confirm requests DO appear this time (proving the toggle actually
  does something, not just that Sandbox happens to look quiet).

## 1. Static checks (do these first, they're cheap)
- Confirm APP_VERSION ('2.8.0') matches the topbar display and matches
  what CHANGELOG.md calls "[current]".
- Extract the <script> contents and run `node --check` — must pass clean.
- Search for duplicate top-level `function name(` declarations. Flag any
  you can't justify as intentional (e.g. two mutually-exclusive template
  branches emitting the same static id is fine; two real function bodies
  with the same name is not).
- Search for duplicate static `id="..."` attributes across the whole
  file, same caveat.
- Extract every literal `getElementById('X')` / `getElementById("X")` call
  and confirm a matching `id="X"` exists somewhere reachable. Note: a
  handful of ids (app, nav, floaterPanel, floaterBtn, versionTag) live in
  the static HTML shell outside the extracted <script> block — check the
  full file, not just the script, before flagging something as missing.

## 2. Contract creation — the four intake paths (this is the core
   architecture change in this version; test it thoroughly)
Contract Intake is now the SINGLE creation surface — 0 · Tiering no
longer has its own manual sliders. Verify all four paths produce
IDENTICAL downstream shape (a tieringBulk row + Register items + optional
rebate + a QBR record), via the shared `createContractRecord()` function:
- **Manual entry**: fill in name, category (try both a datalist suggestion
  and a typed-in value), start/end dates, sliders, obligations (with and
  without a trailing `(YYYY-MM-DD)`), SLAs, rebate terms. Confirm the live
  tier preview badge updates on every slider move BEFORE you click
  "Confirm & Create Contract Record", and confirm nothing is created until
  that click.
- **AI-assisted**: type a plain-English description, click "Suggest with
  AI" (Sandbox is fine), confirm it pre-fills the same fields the manual
  path uses (including the live preview badge), and confirm — again —
  nothing is created until "Confirm & Create Contract Record".
- **Offline CSV template**: click "Download blank template", confirm the
  downloaded file has a header row and one example row with the exact
  column set the upload parser expects. Fill in 2-3 rows (include at
  least one field with an embedded comma inside quotes, to test the CSV
  parser), re-upload it, confirm the stated count of created contracts
  matches, and confirm each one shows up correctly on Tiering/Register/
  Rebates/QBR. Try uploading a .csv missing the header row or with a
  wrong extension and confirm a clear, non-crashing error.
- **Document upload (0 · Tiering)**: upload a real .docx/.pdf, confirm it
  auto-scores, sets the active contract, opens/creates its QBR, AND now
  automatically creates/updates its portfolio-table row (`tieringBulk`) —
  this was the specific v2.7.1 fix. Re-upload the SAME contract (or
  append a second file) and confirm the EXISTING row updates in place —
  no duplicate row for the same normalized name.
- Confirm 0 · Tiering's "single contract" section is now read-only,
  reflecting only the active uploaded contract — no manual sliders there
  — and links over to Contract Intake when there's no active contract.
- Confirm the portfolio table's "+ Add contract" and "paste names to
  AI-score" still work as lightweight score-only additions, and that the
  copy correctly states they do NOT create Register/Rebate/QBR records
  (unlike the four paths above).

## 3. Nav order & renames
- Confirm nav order is: About, Contract Intake, 0 · Tiering, Contract
  Playbook, Obligations/SLA/CO, Rebates, Coverage & Feedback, QBR,
  Contract Brief, Maturity, Governance, Settings — Coverage must come
  BEFORE QBR (QBR reads stakeholder feedback from Coverage; the old order
  had this backwards).
- Confirm "Framework" is renamed to "Contract Playbook" everywhere
  user-facing (nav label, screen heading, cross-references from other
  screens like Rebates' empty state and the Maturity Scorecard) — internal
  function/variable names and code comments may still legitimately say
  "framework"/"fw" internally; that's expected, not a bug.

## 4. Contract Playbook (formerly Framework) quick-add
- Confirm every "+ Add starter" button is disabled until a contract is
  selected from the "Target contract" dropdown (sourced from the Tiering
  portfolio table).
- With a contract selected, click a quick-add button, confirm the created
  Register/QBR/Rebate items carry that contract's actual name (not blank,
  not "whichever QBR happened to be open").
- **Idempotency**: click the SAME quick-add button again for the SAME
  contract — confirm it reports items already exist and adds nothing new
  (no duplicates). Then switch the target contract dropdown to a
  DIFFERENT contract and click the same button — confirm it adds a fresh
  set for that contract.

## 5. QBR
- Confirm the Supplier field is now a dropdown sourced from the Tiering
  portfolio table, with an "Other" option that reveals a free-text input
  for a name not yet tiered.
- Re-run the v2.6.0 multi-QBR checks: create 2+ QBRs for different
  suppliers via the picker, confirm they don't overwrite each other,
  confirm Delete falls back to a blank draft, confirm Governance
  aggregates risk across ALL saved QBRs not just the open one.
- Confirm the new source-of-data note above the KPI/SLA tables correctly
  states the narrative/exports are built from THIS QBR's own fields plus
  Coverage feedback, and that the "no active contract" warning is visibly
  stronger/more specific than the general Sandbox disclaimer when there's
  no active document.

## 6. Contract Brief
- Confirm the new "Include the latest QBR's narrative" checkbox is OFF by
  default, is disabled when no QBR exists for the selected contract, and
  when checked adds the QBR narrative (not the KPI/SLA tables — those
  would duplicate) to both the on-screen Page 2 and the Word export.
- Re-run the v2.6.0 per-contract accuracy checks: switching the contract
  dropdown refreshes every field with no bleed from the previous
  selection, and the handover narrative saves per-contract.

## 7. Governance
- Confirm 🔔 Renewal & Obligation Watch's urgent/upcoming day-cutoff
  inputs are present, editable, persist after a reload, and actually
  change which items fall into which bucket (test with a value like 15/45
  instead of the 30/90 defaults).
- Confirm 🔍 Cross-Contract Pattern Scan states its scope explicitly
  (only contracts on the portfolio table, with a live count) and stays
  disabled under 2 contracts.
- Confirm the tier-mix donut and maturity radar chart headings now state
  their scope explicitly ("whole portfolio" / "organisation-wide") rather
  than implying a single contract/supplier's view.
- Confirm the Rebates accrued-entitlement bar chart no longer truncates
  long contract/tier-name labels illegibly (check with a deliberately
  long contract name) and has a subtitle explaining what the bars
  represent (spend × rate, not raw spend).

## 8. Register
- Confirm the new explanatory note about WHY Change Order is a Type
  within this register rather than its own screen is present and makes
  sense standalone (without having read CHANGELOG.md).
- Confirm the empty-state copy on both the main table and "Contract
  Analysis & Staging" no longer implies document upload is the ONLY way
  to get items here — it should mention Contract Intake too.

## 9. Rebates
- Confirm the empty-state / active-contract banner copy no longer implies
  a document must be active to use Rebates at all — only "AI Extract
  Rebates" specifically needs one; manual/Intake/template entry doesn't.
- Confirm the new "source of data" sentence under the page intro is
  present and accurate.

## 10. Maturity Scorecard
- Confirm the new "Source of data: entirely manual" note is present and
  clearly states this is a one-time-per-cycle, organisation-wide
  self-assessment — never AI-generated or tied to a specific contract.

## 11. Demo tour
- Confirm the tour's screen sequence now matches the nav order exactly
  (Coverage before QBR) — extract both sequences and diff them.
- Run the tour start to finish; confirm every step's referenced UI
  element actually exists on screen at that point, especially the
  rewritten Tiering/Intake/Playbook steps reflecting the new
  architecture.
- **Resync test**: start the tour, advance a few steps in, then manually
  click a DIFFERENT tab in the top nav (not the tour's own Next/Prev).
  Confirm the tour card updates to narrate the tab you actually navigated
  to, rather than staying stuck on stale text for the old screen. Then
  click Next/Prev on the tour card itself and confirm normal tour
  progression still works correctly afterward (i.e. the resync logic
  doesn't break the tour's own navigation).

## 12. Regression — confirm nothing earlier broke
- Multi-QBR persistence and migration from a pre-v2.6 save (still
  relevant — re-verify quickly, don't skip because it was checked before).
- Rebate/QBR "AI Extract" buttons still pull from the same cached
  document review as Tiering.
- Mobile hamburger nav (below 768px) renders all 12 items including the
  renamed Contract Playbook, and closes correctly on tab selection.
- Governance's Word/PowerPoint exports still produce complete documents
  with charts embedded, including the newly-relabeled chart headings.

## 13. Report back
For each numbered section above, report PASS / FAIL / PARTIAL with a
one-line reason. For any FAIL or PARTIAL, show me the relevant code (file
+ line range) and your proposed fix, but do NOT make the fix yet — wait
for my go-ahead per CLAUDE.md's workflow rules. Don't fix anything
silently while doing this pass, even something that looks trivial.
```
