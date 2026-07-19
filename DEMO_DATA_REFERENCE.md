# CLM Suite — Demo Data Reference
**For testing v2.2.0** · use this to cross-check what you see on screen against what should be there.

## How to load it
On the **About** screen, click **▶ Demo tour**. This seeds all the data below *once* — only if your Tiering bulk table is currently empty (it won't overwrite real data). If you want to reload fresh demo data later, go to ⚙ Settings → **Reset all local data** first, then run the tour again.

---

## Supplier 1: Global Logistics Co — Strategic / 🥇 Gold
*The flagship demo — has data on every screen, including a live QBR.*

**Tiering:** Value 5, Risk 4, Criticality 5, Impact 4 → score **4.5** → Strategic / Gold

**Register (3 items):**
| Type | Description | Owner | Due | Status |
|---|---|---|---|---|
| SLA | On-time delivery must be ≥98% of scheduled loads | Ops | 10 days ago | Open → shows **Overdue** |
| Obligation | Insurance certificate renewal — annual | Procurement | +20 days | Open |
| Change Order | Fuel surcharge pricing amendment | Procurement | +45 days | Open |

**Rebates:** Tier 2, threshold $500,001–$1,000,000, rate 2%, spend to date $720,000 → **accrued $14,400**, status Open

**Coverage & Feedback:**
- James (Ops Director): 4/5 — "Reliable overall, but the fuel surcharge change was communicated late."
- Priya (Warehouse Lead): 5/5 — "Excellent on-time performance this quarter."
- Collaboration: Joint business plan — "Explore regional consolidation to cut empty-mile costs." (In progress)

**QBR (pre-loaded, this is the one you'll see when you open the QBR tab):**
| KPI | Target | Actual | Status |
|---|---|---|---|
| On-time delivery | 98% | 95% | At risk |
| Damage / defect rate | <1% | 0.6% | On track |

| SLA | Target | Actual | Status |
|---|---|---|---|
| Incident response time | 4 hours | 6 hours | **Breached** |
| Tracking system uptime | 99.5% | 99.7% | On track |

- Rebates note: "Tier 2 volume rebate tracking at 2%..."
- Risks: "Fuel surcharge pass-through communicated late last cycle; single-source risk on the Northeast lane."
- Actions: "Agree 30-day advance notice for surcharge changes; scope a backup carrier for the Northeast lane."
- **Check:** Recommendations panel should show *"Escalate 1 breached SLA(s)..."* generated automatically from the breached SLA above.
- **Check:** KPI/SLA status donut chart should show 1 On track, 1 At risk, 1 Breached.

---

## Supplier 2: Meridian Cloud Hosting — Managed / 🥈 Silver
*Built to make Governance's Risks/To-do say something real — has a disputed rebate and a low rating.*

**Tiering:** Value 3, Risk 3, Criticality 4, Impact 3 → score **3.2** → Managed / Silver

**Register (2 items):**
| Type | Description | Owner | Due | Status |
|---|---|---|---|---|
| SLA | System uptime must be ≥99.9% | IT | +5 days | Open → shows **Due soon** |
| Obligation | Annual SOC2 audit report submission | IT | +60 days | Open |

**Rebates:** Volume rebate, threshold >$250,000 annual, rate 3%, spend to date $410,000 → accrued $12,300, status **Disputed**

**Coverage & Feedback:**
- Alex (IT Manager): **2/5** — "Uptime dipped twice this quarter and support response was slow."
- Collaboration: Innovation — "Pilot AI-based auto-scaling to reduce compute spend." (Idea)

**QBR:** none pre-loaded (only one QBR can be open at a time — this one shows on the QBR tab if you switch the Supplier field to "Meridian Cloud Hosting"; its linked register/rebate/feedback data should populate automatically once you do).

**Check:** on Governance, this supplier's disputed rebate and 2/5 rating should each generate one line in the Risks and To-do lists.

---

## Supplier 3: Apex Office Supplies — Transactional-Routine / 🥉 Bronze
*The low-touch contrast case — minimal data, no rebate program.*

**Tiering:** Value 1, Risk 2, Criticality 1, Impact 1 → score **1.3** → Routine/Leverage / Bronze

**Register (1 item):**
| Type | Description | Owner | Due | Status |
|---|---|---|---|---|
| Obligation | Catalogue price list annual refresh | Procurement | +90 days | Open |

**Rebates:** none — deliberately, to show not every contract needs rebate tracking.

**Coverage & Feedback:**
- Dana (Office Manager): 4/5 — "Good value, occasional stock-outs on paper."

---

## Spend categories (Coverage tab)
| Category | Annual spend | % under contract | Expected badge |
|---|---|---|---|
| Logistics & Freight | $1,200,000 | 85% | Covered |
| IT & Cloud Services | $900,000 | 60% | Partial |
| Office Supplies | $150,000 | 40% | **White space** |
| Facilities | $300,000 | 20% | **White space** |

**Check:** the result strip at the top of Coverage should show a weighted **% spend under contract** — work it out yourself as a sanity check: (1,200,000×.85 + 900,000×.60 + 150,000×.40 + 300,000×.20) ÷ 2,550,000 ≈ **66%**.

---

## Maturity Scorecard
Deliberately uneven (not all 3s), so the radar chart has a real shape:
`{Foundation:4,3 · Obligations:3,2 · KPIs/SLAs:4,3 · Rebates:2,2 · Change/Termination:4,3 · QBR/Review:4,3 · Stakeholder/Spend:3,2 · Collaboration:3}`

**Check:** Rebates dimension (2,2 → avg 2.0) should be at or near the bottom of the maturity radar — that's intentional, it should also show up in Governance's "weakest dimension" flag.

---

## What to specifically check on Governance
With all the above loaded, the Governance tab's live-computed sections should show:
- **Tier mix donut:** 1 Gold, 1 Silver, 1 Bronze
- **Spend under contract:** ~66%, with 2 white-space categories flagged
- **Rebate exposure:** ~$26,700 total accrued, 1 of 2 tiers not reconciled
- **Risks:** overdue register item(s), the disputed rebate, 2 white-space categories, the low feedback rating, weakest maturity dimension
- **To-do:** close the overdue item, reconcile the disputed rebate, scope contracts for the two white-space categories, run an improvement action on Rebates maturity, follow up with Alex before next QBR
- **Maturity radar:** visibly lower on the Rebates axis than the others

If any of those don't show up as described, that's exactly the kind of mismatch worth flagging back to me.
