# 🛡️ SafeCart: AI Agents for Product Safety & Indian Listing Compliance

SafeCart is a four-agent AI system. It reads marketplace reviews for physical-harm signals and checks product listings against Indian Legal Metrology, BIS and FSSAI rules and restricted-claim rules. It then escalates the riskiest products, with an evidence quote and a recommended action for each.

> Portfolio project by **Priyanka G**. Built with Amazon Quick agents and skills, and validated on a synthetic test set in the Amazon Reviews 2023 format.

## 📄 Start here
- **[PRD (PDF)](docs/SafeCart_PRD.pdf):** problem, solution, how each agent works, results and FAQ
- **[Interactive dashboard](demo/SafeCart_dashboard.html):** download the file and open it in a browser
- **[Working model](demo/SafeCart_working_model.html):** type a review or a listing and see how it's scored, step by step

![Orchestration](diagrams/SafeCart_orchestration.png)

## 📊 Results (synthetic validation set: 100 reviews, 20 listings)
| Metric | Target | Result |
|---|---|---|
| Safety recall | ≥ 90% | **100%** (30/30) |
| Safety precision | ≥ 70% | **100%** (0 false alarms) |
| Harm-type accuracy | — | **97%** (29/30), up from 23% after the fix |
| Planted listing violations caught | 20/20 | **20/20** |
| Compliance pass rate | — | 25% (5 PASS · 11 FIX · 4 BLOCK) |

⚠️ The test set is synthetic, so these scores are optimistic. Next step: hand-labelled real reviews.

## 🤖 The four agents
| Agent | Role |
|---|---|
| Coordinator | Checks the inputs, sends work to the specialists, combines one workbook |
| Review Safety Specialist | Pre-filter → classify harm → risk score → tier → 3-flag boost |
| Listing Compliance Specialist | Legal Metrology / BIS / FSSAI checks plus restricted claims with rewrites |
| Digest & Validation Specialist | Weekly digest, recall/precision, planted-violation check |

**Design principle:** code handles the clear-cut checks (filters, scoring, mandatory fields), so results are repeatable and auditable. The model is used only for judgement calls (is this harm? is this claim misleading?).

## 🛠️ Tech stack
| Layer | Technology | Used for |
|---|---|---|
| AI agents | AWS Quick agents | Coordinator + 3 specialist agents |
| Reusable logic | AWS Quick skills | Review scoring, listing compliance, recall/precision (each with a Python script) |
| Shared knowledge | AWS Quick space | Rules reference, validation set and answer keys |
| Judgement | LLM (inside the agents) | Harm classification, restricted-claim detection, compliant rewrites |
| Data processing | Python, pandas, regex | Pre-filter, risk scoring, tiers, 3-flag boost, compliance scores, metrics |
| Spreadsheets | openpyxl, xlsxwriter | Validation set, results workbooks, dashboard data |
| Dashboard and demo | HTML, CSS, JavaScript, Highcharts | Animated dashboard and interactive working model |
| Docs and diagrams | ReportLab, SVG, resvg | PRD (PDF), orchestration diagram |
| Data format | Amazon Reviews 2023 (McAuley Lab) | Format of the synthetic test set |

**Planned:** Lovable + Google Sheets live dashboard (prompts and template in `lovable/`), Supabase for live scoring.

## 📁 Repo structure
```
docs/        PRD (PDF), rules reference
demo/        Interactive dashboard + working model (HTML)
agents/      Full instructions for all 4 agents (reusable as system prompts)
skills/      3 skills, each with a SKILL.md and Python scoring script
data/        Validation set + answer keys, results, demo data, raw agent outputs
lovable/     Dashboard data (JSON/Excel/CSV), Google Sheet template, Lovable prompts
diagrams/    Orchestration diagram (PNG + SVG)
```

## ▶️ Run the scoring scripts
```bash
pip install pandas openpyxl
python -c "import sys; sys.path.insert(0,'skills/safecart-review-scoring/scripts'); import score_reviews as s; print(s.KEYWORDS)"
```

## 🧭 Key decisions
- **Recall before precision.** A missed hazard costs far more than a false alarm.
- **3-flag rule.** 3 or more safety flags on one product escalates it to CRITICAL.
- **Missing BIS/FSSAI = automatic BLOCK.** Validation showed that score thresholds alone let these through.
- **Fixed 8-label harm list.** The model had invented 17 labels; locking the list raised accuracy from 23% to 97%.

## ⚖️ Limitations
No live Amazon review feed (platform terms) · check the BIS category list against current notifications · time and cost per run not yet measured.
