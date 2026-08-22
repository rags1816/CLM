## Prompt to paste into Claude Code

Copy everything in the fenced block below into a Claude Code session in
this repo. It's written to be run as-is. A companion manual UAT test
script (`CLM_Suite_v2.7.1_UAT_Test_Script.docx`) covers the same ground
for human click-through testing — this prompt is the automated/code-level
equivalent for Claude Code to run alongside it.

```
Read CLAUDE.md first and follow its workflow rules for this whole session
(no commits without a shown diff and my explicit confirmation, work on a
feature branch, flag anything that looks like a real secret/PII).

I need a verification pass on index.html covering everything that changed
between v2.4.0 (the last version I manually reviewed line-by-line) and the
current v2.7.1. Do NOT assume the changes are correct — actually check
each item below against the live code, not against CHANGELOG.md's
description of what was supposed to happen. Read CHANGELOG.md and
METHODOLOGY.md first for context on what v2.5.0–v2.7.1 were meant to add,
then verify the code actually does it.

## 1. Static checks (do these first, they're cheap)
- Confirm APP_VERSION matches the version displayed in the topbar and
  matches what CHANGELOG.md calls "[current]".
- Extract the <script> contents and run `node --check` on them — must
  pass clean.
- Search for duplicate `function name(` declarations at the top level —
  flag any (other than genuinely intentional ones you can point to and
  justify, e.g. two mutually-exclusive template branches that happen to
  emit the same static id).
- Search for duplicate static `id="..."` attributes across the whole
  file — same caveat as above.
- Grep every `getElementById('X')` / `document.getElementById("X")` call
  and confirm a matching `id="X"` exists somewhere it could actually be
  rendered before that code path runs. Flag any that don't.

## 2. Multi-QBR (v2.6.0)
- Confirm STATE.qbr is still the single "currently open/editing" record,
  and STATE.qbrs is the array of saved records — not conflated.
- Confirm loadState() migrates a pre-v2.6 single-QBR save (STATE.qbr with
  a real supplier name but no STATE.qbrs) into STATE.qbrs on first load,
  without losing the data. Test this specifically: hand-construct an old-
  shape localStorage value (qbr populated, no qbrs key) and confirm it
  loads correctly.
- In the running app: create two QBRs for two different suppliers via the
  picker's "+ New QBR", confirm both persist independently after a page
  reload, confirm editing one doesn't bleed into the other.
- Confirm "Delete" on a QBR removes it from the list and, if it was the
  open one, falls back to a blank draft rather than erroring.
- Confirm Governance's Risks/To-do lists reflect breaches from ALL saved
  QBRs, not just whichever one is currently open on the QBR screen — test
  by breaching a KPI on QBR #2 while QBR #1 is the one open, then check
  Governance shows it.
- Confirm a Strategic/Gold-tier contract with no matching QBR record shows
  up as a named risk on Governance.

## 3. Contract Summary Intake (v2.5.0) + AI intake agent (v2.7.0)
- Manual path: fill in the form without touching "Suggest with AI", submit,
  confirm a Tiering row, Register items (with correct due-date parsing
  from the "(YYYY-MM-DD)" suffix), and a rebate entry (only if
  threshold/rate were filled) are created, and the new contract becomes
  the active contract.
- Confirm submitting via Intake creates or reuses a QBR record via the
  same supplier-name matching QBR uses elsewhere (openOrCreateQbrFor) —
  submitting twice for the same name should not create two QBRs.
- AI-assisted path: type a plain-English description, click "Suggest with
  AI" (Sandbox mode is fine for this check), confirm the returned JSON is
  parsed and populates name/category/tier sliders/obligations/SLAs/rebate
  fields WITHOUT creating anything yet — nothing should appear on Tiering/
  Register/Rebates until "Create contract record" is clicked afterward.
- Confirm a malformed/non-JSON AI response is caught and shown as a clear
  error rather than throwing an unhandled exception or silently leaving
  stale field values.

## 4. Contract Brief (v2.6.0)
- Select a contract with real Register/Rebate/Feedback/QBR data linked to
  it and confirm every section (tier & score, next key date, rebate
  standing, QBR status, stakeholder rating, top risks, open register
  items, recent feedback, recommended actions) actually reflects that
  contract's data, not another contract's.
- Select a contract with NO linked data and confirm the screen degrades
  gracefully (no data required to be present, no thrown errors).
- Confirm "Draft handover narrative" saves per-contract (switching the
  dropdown to a different contract should show that contract's own saved
  narrative, not the one just drafted).
- Confirm Word export produces a document with all the same sections as
  the on-screen view.

## 5. Renewal & Obligation Watch Agent (v2.6.0)
- With register items at various due dates (overdue, due in <30 days, due
  in 31-90 days, due >90 days, and no due date), confirm the watch agent
  buckets them correctly into critical/urgent/upcoming and excludes
  anything >90 days out or with status "Complete".
- Confirm items are sorted soonest-due-first within and across buckets.
- Confirm "AI prioritise" is disabled when there are zero alerts, and
  produces a reasonable prioritised brief when there are some.

## 6. Cross-Contract Pattern Scan (v2.7.0)
- Confirm the scan button is disabled with fewer than 2 contracts on the
  Tiering bulk table, and enabled with 2+.
- Load the demo data (2+ suppliers) and run a scan — confirm it completes
  without error in both Sandbox and (if you have a key configured) live
  AI mode.
- Check the prompt construction sends a genuinely compact per-contract
  summary (not the full register/feedback dump) — flag it if token usage
  looks disproportionate to what's needed.

## 7. About screen chevron stepper (v2.5.0)
- Confirm ‹ Previous is disabled on the first pillar and › Next is
  disabled on the last, dot navigation jumps directly to the clicked
  section, and the step counter ("X of 8") is accurate throughout.

## 8. Sequencing fixes (v2.7.1) — these are the newest and least-tested code
- **Tiering document-upload → auto bulk-table row.** Upload a document at
  0 · Tiering and confirm a row is automatically added to `tieringBulk`
  (visible on the bulk table and heatmap) via `upsertTieringBulkRow`,
  without a separate manual "+ Add bulk row" step. Then re-upload the
  SAME contract (or append a second file for it) and confirm the EXISTING
  row is updated in place — no duplicate row for the same normalized
  contract name. Also check Register's "Add to Tiering table" button
  (`btnAddSuggestedTier`) now routes through the same `upsertTieringBulkRow`
  helper rather than its own separate `.push()`.
- **Framework quick-add → requires a target contract.** Confirm every
  "+ Add starter" button on the Framework screen is disabled until a
  contract is selected from the new "Target contract" dropdown
  (`fwTargetContract`). With a contract selected, confirm: (a) Register
  items created via "+ Add starter obligations/change orders" carry that
  contract's name in the `contract` field, not an empty string; (b) KPIs/
  SLAs added via "+ Add starter KPIs/SLAs to QBR" land on THAT contract's
  QBR record specifically (check via the QBR picker) — including when a
  DIFFERENT QBR was already open on the QBR screen at the time, which
  must NOT receive the new items; (c) rebate tiers added carry that
  contract's name in `contractName`, not blank.
- Confirm `FW_TARGET_CONTRACT` resets to blank if the previously-selected
  contract no longer exists in `tieringBulk` (e.g. after a data reset),
  rather than pointing at a stale name.

## 9. Regression checks — confirm v2.4.0 behavior still works
- Register's "AI document review" and candidate clause staging still
  work end to end, including the button that adds a candidate as an SLA
  onto the currently-open QBR.
- Rebate and QBR "AI Extract" buttons still pull from the same cached
  document review as Tiering, not a separate/possibly-disagreeing call.
- Mobile hamburger nav (below 768px) still renders and closes correctly
  with the two new nav items (Contract Intake, Contract Brief) added.
- Run the full demo tour end to end (About → Intake → Tiering → Framework
  → Register → Rebates → QBR → Coverage → Brief → Scorecard → Governance
  → Settings) and confirm every step's referenced UI element actually
  exists at that point in the tour.

## 10. Report back
For each numbered section above, report PASS / FAIL / PARTIAL with a one-
line reason. For any FAIL or PARTIAL, show me the relevant code (file +
line range) and your proposed fix, but do NOT make the fix yet — wait for
my go-ahead per CLAUDE.md's workflow rules. Don't fix anything silently
while doing this pass, even something that looks trivial.
```

