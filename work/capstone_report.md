# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Muzamil
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/muzammilsharf/flyrank-ml-internship
- **Date:** September 2026

> Copy this file to `work/capstone_report.md` and fill it in as you build. The eight
> sections mirror the Pass / Needs-Work rubric axes, so nothing here is optional.

## 1. Problem framing

Content teams have many pages. They do not have time to check every page every week. This project helps them pick which pages to check first.

The unit of analysis is **one page, for one client, in one month**. Each row is a snapshot: 90 days of past data, used to guess what happens in the next 30 days.

The output is a **risk score** (a number between 0 and 1) and a **rank**. The highest-risk pages go to the top of the list.

The action a human takes: a content editor opens the top pages on the list. They check if the page really is losing traffic. If yes, they update the title, the content, or add links. If no, they move to the next page.

The cost of a wrong call goes two ways:
- If the model flags a healthy page, the editor wastes time checking it.
- If the model misses a real decline, the page keeps losing traffic and nobody notices.

Why does ML help here? A simple rule (like "flag every old page") is too rough. It flags too many healthy pages. ML can look at many signals at once (traffic, position, clicks, content age) and find a better pattern than one fixed rule.

## 2. Data safety

**Data used:**
- `fact_content_daily_performance` — daily traffic numbers for each page (impressions, clicks, position, sessions).
- `dim_content` — page details (word count, content type, search intent, backlinks, creation date).

**Columns deliberately excluded:**
- `trend_direction` and `trend_pct` — these are label-derived. Using them as features would be leakage (the model would just copy the answer).
- `content_updated_date` (and anything based on it, like "days since last update") — this field only shows the page's *current* state, not its state at each past snapshot. Using it caused a real leak: it made old snapshots show a negative number of days, which is impossible, and it let the model peek at the page's future.
- `content_hash_id` and `client_hash_id` — used only for grouping and joining tables. Never used as model inputs.

**Other leakage risks found and fixed:**
- A filter that required GA4 data to be available accidentally dropped most rows, because most clients only started GA4 tracking in early 2026. Fixed by only requiring Search Console data, and treating missing GA4 numbers as "unknown," not zero.
- `content_age_days` looked safe at first, but it turned out to rise by about 30 days with every monthly snapshot. This made it act like a hidden calendar signal, not a real page-age signal. It was removed.

**Confirmation:** No client names, domains, or URLs appear anywhere in this repo. All IDs are pseudonymous codes used only for grouping.

## 3. Baseline

The baseline is a simple, see-through rule. For each page:

```
score = (expected click rate for its search position − actual click rate) × log(impressions)
```

This flags pages that get fewer clicks than normal for their search position. It is a fair comparison because it uses the same data and the same test set as the model.

**Baseline result: 24.0% Precision@50.**

The baseline has one clear weakness: it often flags pages that rank #1–3 but get zero clicks. These are usually "zero-click" pages, where Google shows the answer directly on the search page. The baseline cannot tell the difference between this and a real decline.

## 4. Model / analysis

**Method:** LightGBM (gradient-boosted trees), compared against Logistic Regression, a single Decision Tree, and Random Forest.

**Why LightGBM fits:** it handles missing values on its own (useful since many pages lack GA4 data), and it can learn non-simple patterns, like "high impressions and low clicks is fine at position #1, but risky at position #5."

**Target (in one sentence):** a page is labeled "declining" if its search impressions drop more than 20% over the next 30 days compared to the past 30 days, and only if it had at least 100 impressions to begin with (too few impressions makes the percent change meaningless).

**Features used (28 total):**
- Traffic: impressions, clicks, click rate, average position, position volatility.
- Engagement (only where GA4 data exists): engaged sessions, engagement time, organic session share, scroll events.
- Content: word count, character count, search volume, competition, backlinks, content type, search intent.

**Left out on purpose:** `content_age_days` and `days_since_last_update` (see Section 2, both caused leakage). `trend_direction` and `trend_pct` (label-derived).

## 5. Evaluation

**Split used:** grouped by client, not by time and not by content item.

**Why:** two earlier splits (a time-based split, and a split grouped by content item) both gave a suspicious Precision@50 of 1.000. This happened because pages from the same client behave alike, so even different pages from the same client "give away" the answer. Holding out whole clients removes this problem.

Only 36 clients exist in this data. One single split can be unlucky or lucky, so the result is the **average of 5 different random splits**, not just one number.

**Results, same split each time:**

| Method | Precision@50 |
|---|---|
| Baseline (heuristic) | 0.240 |
| Logistic Regression | 0.560 |
| Decision Tree (one tree) | 0.920 (unstable, high variance) |
| Random Forest | 0.260 |
| LightGBM, one split | 0.940 (later found to be inflated, see below) |
| **LightGBM, average of 5 splits** | **0.788 (± 0.122, range 0.560–0.940)** |

**Base rate:** in the months used for the final test, about 20.3% of pages were genuinely declining. A model that guessed randomly would score close to that. Our model's 78.8% is about **3.9 times better than random guessing** on this task (a "lift" of roughly 3.9x over the base rate).

**Error analysis:** the single-split 0.940 number looked too good, so it was checked. The top-50 flagged pages were almost all from one single month (March–April 2026), a month with a real, sharp rise in declining pages across many clients at once. The model was partly just "detecting that month," not learning a general pattern about individual pages. After removing the confusing feature (`content_age_days`) and testing outside that month, the more honest number is 0.788.

## 6. Interpretation

The strongest features, in plain words:

1. **How long since the page was last updated** (checked using today's real data, not the leaky historical version) — older, untouched pages are more likely to be flagged.
2. **How many impressions the page gets** — very low or very high traffic changes the risk pattern.
3. **Word count and character count** — shorter pages show up more often in the risk group.
4. **Click rate** — pages getting fewer clicks than expected for their position are riskier.
5. **Position stability** — pages whose ranking position moves around a lot are riskier than pages with a steady position.

**Negative result worth stating clearly:** content age, once we controlled for the calendar-month confusion, was **not** a strong, trustworthy signal on its own. It looked strong only because it accidentally lined up with a real traffic event in two specific months. This is a useful finding: age alone is not a safe signal to lean on without more care.

## 7. Recommendation

The model produces a ranked list. Each page on the list also gets a reason code, so an editor knows *why* it was flagged, not just that it was flagged:

- `SEVERE_CTR_DEFICIT` — good position, but too few clicks. Action: check the title and snippet.
- `PAGE_ONE_OPPORTUNITY` — page ranks #11–20, has real search demand. Action: expand the content, add internal links.
- `LOW_ENGAGEMENT` — clicks arrive, but visitors leave fast. Action: check the page's intro and structure.
- `GENERAL_DECLINE_RISK` — high risk, no single clear reason. Action: general freshness review.

**How an editor uses this tomorrow:** open the top 50 pages each week. Check each page against its reason code. Fix what makes sense. Skip pages that turn out to be false alarms (for example, a page that ranks #1 with zero clicks because Google already shows the full answer).

**Confidence and limits, stated plainly:**
- This tool ranks pages. It does not prove a page will decline, and it does not prove that fixing a page will bring traffic back.
- The score is less trustworthy during unusual periods, like the March–April 2026 spike, which is not yet explained.
- With only 36 clients, the true accuracy could land anywhere between about 56% and 94% on a given month, not always exactly 78.8%.
- About 83% of pages have no GA4 tracking data in this snapshot, so engagement-based reason codes do not apply to most pages yet.

## 8. Reproducibility

**To re-run everything from a fresh clone:**

```bash
git clone https://github.com/muzammilsharf/flyrank-ml-internship.git
cd flyrank-ml-internship
pip install -r requirements.txt
pip install duckdb huggingface_hub lightgbm
hf auth login   # paste a Hugging Face read token with access to FlyRank/internship-warehouse
```

Then open, in order, and run each top to bottom:
1. `work/notebooks/w04_baseline_score.ipynb`
2. `work/notebooks/w05_model.ipynb`
3. `work/notebooks/w06_validation_audit.ipynb`
4. `work/notebooks/w07_action_playbook.ipynb`
5. `work/notebooks/capstone.ipynb`

**Random seeds used:** model training uses `random_state=42`. The multi-split validation check uses five seeds: `42, 7, 123, 2024, 99`.

**Environment:** Python 3.10+, key packages: `duckdb`, `huggingface_hub`, `lightgbm`, `scikit-learn`, `pandas`, `numpy`.

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> language everywhere. No causal claims without an experiment. No claim of finding Google's
> algorithm. No client-identifying details. Numbers in this report match a fresh re-run.
