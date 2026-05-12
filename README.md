# Premium Is a Perception, Not a Price
### Marketing Analytics of Campus Starbucks — UIUC Illini Union
Data Analytics Marketing · University of Illinois Urbana-Champaign
**Team 2 · Tanusha Mukund · Suyash Sawant · Avish Shakwala · Priyal Agarwal**

---

## Overview

This project analyzes real point-of-sale data from the Starbucks at the UIUC Illini Union (Sep 2025 – Feb 2026) to understand how premium product strategies perform differently at a campus location compared to standard suburban Starbucks locations.

We test three hypotheses using OLS regression on SKU-level sales data, supported by descriptive analysis and a rules-based textual classifier applied to product names.

**Core finding:** Premium pricing depends on perception, and at a campus Starbucks, perception is shaped by student routines and drink identity — not weather, menu availability, or pricing strategy alone.

---

## Research Questions

1. **Protein Drinks** — Did protein drinks show weak early adoption after their October 2025 launch, despite premium pricing?
2. **Oat Milk** — Does oat milk still generate a statistically significant per-unit price premium once drink type is controlled for?
3. **Iced Format** — Do iced beverages remain the dominant format even during fall–winter, contrary to the conventional seasonal assumption?

---

## Key Findings

| Hypothesis | Finding | Coefficient | P-value |
|---|---|---|---|
| Protein adoption | 77.9% below expected volume at launch; share declining 1.40% → 1.02% | −1.511 | <0.001 |
| Oat milk premium | Premium statistically insignificant once iced/seasonal controls added | $0.30/unit | 0.166 (Model B) |
| Iced format | Cold-format holds 71–82% of volume across all seasons; 47% more units than hot after controls | +0.386 | 0.0002 |

---

## Data

**Source:** Real POS data from Illini Union Starbucks, UIUC  
**Period:** September 1, 2025 – February 26, 2026 (179 days)  
**Scope:** 200+ SKUs across beverages, food, and retail  
**Reporting periods used:**
- Aug–Sep 2025 (pre-launch baseline)
- October 2025 (protein launch month)
- Sep 2025 – Feb 2026 (full 6-month period)

> ⚠️ **Raw data is not included in this repository** — it contains proprietary store-level POS information provided under a data sharing agreement with the Illini Union. Code and analysis notebooks are provided; they will require equivalent data files to run.

**Data fields available:**
- SKU name, quantity sold, gross sales, discounts, net sales, % of total sales
- Category / subcategory (Espresso, Tea, Refreshment, Blended, Brewed Coffee, Modifiers, Food, Retail)
- Reporting period and store-level check count

---

## Methods

### SKU Feature Extraction (Textual Classifier)

All product features were extracted from SKU names using regex-based classifiers:

| Feature | Extraction logic |
|---|---|
| `size` | Prefix match: `GR` (Grande), `VT` (Venti), `TL` (Tall), `SH` (Short) |
| `is_iced` | Contains `\bIC\b`, `ICD`, or `IC ` (case-insensitive) |
| `is_cold_format` | Iced marker OR Blended/Refreshment subcategory OR cold-only SKU types (acai, shaken espresso, cold brew, nitro, pink drink) |
| `is_seasonal` | Contains: `PMKN`, `PECAN`, `CINDL`, `SUG COOKIE`, `APPLE`, `PSTCH`, `PISTACH`, `PEPPERM`, `BRUL` |
| `is_oat` | Contains `OAT` (case-insensitive) |
| `is_protein` | Contains `\bPRO\b` or `PROTEIN` (case-insensitive) |

> **Note on cold-format classifier:** Simple `"IC"` string-matching misses an entire class of functionally cold drinks (Strawberry Acai Lemonade, Pink Drink, Cold Brew, Nitro, Shaken Espresso, Blended beverages). Our rules-based classifier addresses this gap. Strawberry Acai Lemonade alone accounts for 7,870 units — roughly 5% of total beverage volume.

### Regression Models

**Hypothesis 1 — Protein adoption (log-linear demand model)**
```
log(units + 1) ~ is_protein + rpu + C(size) + C(drink_category) + is_iced + is_seasonal
```
- Sample: October 2025 beverage SKUs only
- Baseline model built on non-protein drinks; protein indicator added in second pass
- R² ≈ 0.35 | Durbin-Watson ≈ 0.80 (mild autocorrelation)

**Hypothesis 2 — Oat milk premium (revenue per unit)**
```
Model A: rpu ~ is_oat + C(size) + C(drink_category) + C(period)
Model B: rpu ~ is_oat + is_iced + is_seasonal + C(size) + C(drink_category) + C(period)
```
- Sample: pooled across all three periods, named beverage SKUs with size
- Durbin-Watson ≈ 1.31 (mild autocorrelation)
- Key finding: oat coefficient weakens and loses significance when iced/seasonal controls added

**Hypothesis 3 — Iced format default**
```
log(units + 1) ~ is_iced + rpu + is_seasonal + C(size) + C(drink_category) + C(period)
```
- Sample: pooled across all three periods
- Durbin-Watson ≈ 0.66 (higher autocorrelation; refreshment SKUs cluster together)

### Limitations
- SKU-level aggregate data — no individual transaction tracking, no customer-level behavior
- Autocorrelation in all three models (DW below 2) — standard errors may be slightly underestimated; treat significance thresholds directionally
- Modifier ratio comparison uses periods of unequal length (single months vs. 6-month window)
- Oat premium analysis does not include oat substitution requests — only drinks that come with oat by default
- Unobserved factors: menu placement, staff recommendations, app visibility, promotions

---

## Repository Structure

```
starbucks-campus-analytics/
│
├── notebooks/
│   └── starbucks_final_analysis.ipynb     # Main analysis notebook (all 3 hypotheses)
│
├── outputs/
│   ├── fig_protein_trajectory.png         # Protein adoption bar chart
│   ├── fig_oat_coefficient.png            # Oat premium coefficient plot with CIs
│   ├── fig_oat_modifier_ratio.png         # Oat modifier substitution ratio trend
│   └── fig_iced_share.png                 # Cold-format share by period
│
├── blog/
│   └── starbucks_blog_post.html           # Full op-ed style blog post
│
└── README.md
```

---

## How to Run

1. Clone this repository
2. Obtain the three Excel report files from the data provider (not included):
   - `Daily Operations2.xlsx` — Aug–Sep 2025
   - `Daily Operations3.xlsx` — Oct 2025
   - `Daily Operations (1)(3).xlsx` — Sep 2025–Feb 2026
3. Place them in the root directory or update the `FILES` dict in the notebook
4. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib statsmodels openpyxl
   ```
5. Run `starbucks_final_analysis.ipynb` end-to-end (designed for Google Colab or local Jupyter)

---

## Blog Post

The full op-ed style write-up is available at:  
📝 [Medium — Premium Is a Perception, Not a Price]([https://medium.com/@mukundtanusha/what-our-campus-starbucks-taught-us-about-the-limits-of-premium-pricing-81abb5f5c289])


---

## Citation / Contact

**Tanusha Mukund** — tmukund2@illinois.edu / mukundtanusha@gmail.com
MSBA Student Ambassador, University of Illinois Urbana-Champaign  
BADM 534 — Prof. Unnati Narang — Spring 2026
