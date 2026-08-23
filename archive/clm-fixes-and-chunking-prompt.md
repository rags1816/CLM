CLM Suite — Implementation prompt: 4 priority fixes + chunked document review

Implement each of the following as a separate, reviewable step. Show me the
diff and test evidence for each before moving to the next, per the standing
rule (commit locally once tested and reviewed, don't push/PR/merge until I
say so).

---

FIX 1 — AI call timeout

Add an AbortController-based timeout (30 seconds) to both askClaude and
askGemini's fetch calls. On timeout, treat it the same as any other failed
call — trigger the existing fallback chain (primary → fallback → Sandbox),
and surface a clear message ("Request timed out after 30s, trying
fallback...") rather than leaving the UI stuck on "Scoring…"/"Refining…"
indefinitely.

Test: simulate a hung request (e.g. a mock endpoint that never responds)
and confirm the fallback triggers within ~30s rather than hanging forever.

---

FIX 2 — Timezone date comparison bug

statusBadge, the QBR recommendations builder, and the Governance overdue
rollup all currently do `new Date(row.dueDate) < new Date()`, where
dueDate is a plain YYYY-MM-DD string. This parses as UTC midnight,
compared against local now() — producing false-positive "Overdue" flags
in negative UTC-offset timezones and false-negative flags in positive
ones.

Fix: compare dates consistently — e.g. parse dueDate as a local-midnight
Date, or compare only the date portion (year/month/day) rather than full
timestamps.

Test: with a due date of "today" in both a negative-offset timezone
simulation and a positive-offset one, confirm the item does NOT show as
Overdue in either case until the actual local day has passed.

---

FIX 3 — Long-clause silent drop

extractObligationCandidates (skips >300 char sentences) and
extractKpiSlaCandidates (skips >250 char sentences) currently drop long
clauses with no indication to the user.

Fix: track a count of skipped-for-length clauses during extraction, and
show it in the import results ("N obligation-style and M KPI/SLA-style
clauses were skipped for length — review the source document directly for
these").

Test: import a document with at least one clause exceeding each length
threshold, confirm the skip count is accurate and displayed.

---

FIX 4 (redesigned) — Full-document AI review via chunked map-reduce

Replace the current behavior (text.slice(0,6000) sent to AI with no
warning) with the following:

1. On document import, after text extraction, check the extracted text
   length against a single-call threshold (~6,000 characters).

2. If under the threshold: behave as today — one AI call, no chunking
   needed.

3. If over the threshold:
   a. Split the extracted text into chunks (~6,000 characters each,
      breaking at paragraph boundaries where possible, not mid-sentence).
   b. Before starting, show the user: "This document is N pages — it will
      be reviewed in [X] chunks to cover the full text. Estimated cost:
      ~$0.02 per 50 pages (~$Y total for this document). If you already
      have a summary of this contract, you can paste or upload that
      instead for a faster, cheaper review."
   c. Provide a "Use a summary instead" option at this point — a text
      area or file input where the user can paste/upload a shorter
      summary, which then goes through the normal single-call path
      instead of chunking.
   d. If the user proceeds with full chunked review: run each chunk
      through Gemini (map step) to extract a structured partial summary
      (obligations, KPIs/SLAs, financial terms, risk flags found in that
      chunk, with an approximate page range).
   e. Combine all chunk summaries into one final structured document
      review via a single Gemini call (reduce step) — parties, term,
      financial mechanics, obligations/SLAs found anywhere in the
      document, risk flags, each traceable to roughly which page range
      it came from where feasible.
   f. Display the final structured summary to the user for review before
      it feeds into tier suggestions or the obligations/SLA register.

4. Keep a "Quick review" toggle available even for long documents — the
   original fast/cheap first-chunk-only behavior — for cases where the
   user doesn't need full coverage.

5. Cost estimate calculation: use actual Gemini per-token rates (input
   and output) to calculate the estimate shown in 3b; don't hardcode a
   flat number that could drift from real pricing.

Test:
- Import a short document (under threshold): confirm single-call path,
  no chunking message shown.
- Import a long document (the 65-page Accenture PDF, or a synthetic
  equivalent): confirm the chunk-count/cost message displays with
  accurate numbers, confirm chunking actually covers content from late
  in the document (e.g. something only present in the final third of the
  text) that today's truncated version would have missed entirely.
- Confirm the "Use a summary instead" path works as an alternative to
  chunking.
- Confirm "Quick review" toggle still works as a faster/cheaper option
  on a long document.

---

Show me the diff and test evidence for each of the 4 items above,
in order, before moving to the next.
