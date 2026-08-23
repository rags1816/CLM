CLM Suite — Consolidated fix batch: mobile nav, Phase 2 findings, remaining pending items

Work through these as separate, reviewable steps, in this order. Show diff
and test evidence for each before moving to the next, per the standing
rules (commit locally once tested and reviewed, don't push/PR/merge until
told; check for divergence from main at the start; keep this branch to
this one batch of work, don't let it run indefinitely).

---

STEP 1 — Mobile hamburger nav (already scoped, proceed as agreed)

Implement option (b): collapsed hamburger/drawer menu below a mobile
breakpoint. Trace anything that depends on current nav DOM structure
(demo tour references, any JS finding tabs by structure) before changing
markup. Test at 375px/390px/414px plus a desktop-width check. Confirm the
demo tour's Tiering/Register steps still work if the tour needs to
interact with the collapsed nav on mobile.

While rebuilding the nav, also do STEP 2's reordering as part of the same
markup change — don't touch nav structure twice.

---

STEP 2 — Nav reordering (Phase 2 Medium finding)

Reorder the nav so Register (Obligations/SLA/CO) and Rebates come before
QBR, since QBR reads from both and is currently empty on first visit.
Suggested order: About → Tiering → Framework → Register → Rebates → QBR →
Coverage → Scorecard → Governance → Settings. Confirm this doesn't break
anything that assumes the old tab order (e.g. demo tour step sequencing).

---

STEP 3 — Consolidate the two disconnected tiering pathways (Phase 2 High finding)

Register currently has its own separate heuristic + old-truncated-AI
tiering path (heuristicTierFromText / the un-chunked aiSuggestTier) that
can disagree with Tiering's chunked auto-score, and only Register's path
reaches the portfolio table Governance depends on.

Fix: make Register's tiering read from the same cached chunked review
(DOC_REVIEW_CACHE, from Phase 1) when an active contract with a cached
review exists, rather than running its own separate heuristic/truncated
call. Keep the heuristic path only as the fallback for manually-entered
contracts with no active-contract upload at all (i.e. when there's
nothing to cache). Confirm only one tier number exists per contract going
forward when a document was uploaded.

Test: upload a document, confirm Tiering's auto-score and what lands in
the portfolio table via Register are now the same number, sourced from
the same review — not two independent calculations that happen to agree
by chance.

---

STEP 4 — Visible active-contract indicator on QBR/Rebates (Phase 2 Medium finding)

Add the same kind of visible indicator Tiering and Register already have
(a name/badge showing which contract is active) to QBR and Rebates, so a
user landing on either tab can see at a glance which document the
"AI Extract..." button would operate on, rather than only inferring it
from a button's presence.

---

STEP 5 — getTier NaN silent fallback (deferred Medium finding, original audit)

getTier currently silently defaults corrupted/NaN axis scores to the
lowest-oversight tier via `|| TIERS[TIERS.length-1]`. Make this throw or
surface a visible error/repair state instead of silently under-reporting
risk. Test against a deliberately corrupted state object.

---

STEP 6 — Malformed-but-200 AI response placeholder (deferred Low finding, original audit)

Where a malformed-but-200 AI response currently resolves to placeholder
text rather than an explicit error, make the failure visible to the user
(e.g. a clear "AI response could not be parsed" message) rather than
silently showing placeholder content that could be mistaken for a real
answer.

---

STEP 7 — CPV tagging (portfolio consistency with CategoryAI)

Add optional UNSPSC and/or CPV manual-entry fields to contracts, matching
the pattern already built in CategoryAI (manual entry only, no lookup/
validation, not copied on any duplicate/copy actions). Add to the Tiering
composition table and/or Register, wherever contracts are listed.

---

STEP 8 — Changelog entry

Add a CHANGELOG.md entry (or entries) covering today's full batch of work:
the 4 original audit fixes, the direct-edit reconciliation, the
consistency pass, Phase 1 (contract identity + review caching), the
CIPS/Hackett/WorldCC attribution work, and this batch (mobile nav, Phase 2,
remaining fixes). Use the existing versioning convention in the file.

---

Not in scope for this batch (flag if you disagree, but don't build):
contract intent/purpose capture — that's a bigger feature needing its own
requirements spec, scoped separately later. Scanned-PDF detection warning
— lower priority now that real test documents aren't scanned; leave for a
future pass.

Show me diff and test evidence for each step in order before proceeding
to the next.
