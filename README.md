# Beauty-Budget Discovery Engine
## CS 210: Data Management for Data Science

---

## Project Overview

A data pipeline that finds budget "dupe" alternatives for luxury Sephora products by comparing ingredient lists using Jaccard Set Similarity. The engine standardizes messy ingredient data (e.g. "Aqua" = "Water" = "Eau"), stores results in a hybrid database, and ranks budget alternatives using a composite Value Score.

---

## Dataset

**Cleaned_Sephora_Products.xlsx** — 7,519 Sephora products with prices, ratings, categories, and ingredient lists (sourced from Kaggle)

**chemical_synonyms.csv** — Custom-built synonym dataset (300+ mappings) that standardizes ingredient names. Loaded with pd.read_csv() — not hardcoded in Python.

---

## Technologies Used

- Python
- Pandas, NumPy, Matplotlib, Seaborn
- SQLAlchemy (SQLite)
- JSON

---

## Methodology

1. **Data Cleaning** — 8 steps: null removal, deduplication, price capping, rating imputation by category median
2. **Ingredient Standardization** — Load chemical_synonyms.csv as a dictionary, split ingredient strings on commas, map each token to its canonical name
3. **Feature Engineering** — price_tier, price_per_ingredient, luxury_markup_index
4. **Hybrid Database** — SQLite for structured product data, JSON for variable-length ingredient lists
5. **EDA** — 12 visualizations covering price distributions, correlations, and brand markups
6. **Jaccard Similarity** — Compare every Luxury vs Budget pair; qualify if similarity ≥ 60% AND budget price ≤ 30% of luxury price
7. **Value Score** — Rank pairs: Jaccard (50%) + Price Savings (30%) + Budget Rating (20%)

---

## Key Findings

- Price vs. rating correlation ≈ 0.04 — price does not predict quality
- Luxury Skincare costs ~7.6× more per ingredient than Budget alternatives
- Top dupe: Valentino Radiant Powder ($205) vs Refill ($50) — 100% ingredient overlap, $155 saved

---

## Project Structure

```
├── beauty_budget_engine.py       # Full pipeline
├── chemical_synonyms.csv         # Ingredient synonym dataset
├── Cleaned_Sephora_Products.xlsx # Sephora product data
└── README.md
```

---

## Setup & Installation

```bash
git clone https://github.com/yourusername/beauty-budget-discovery-engine.git
cd beauty-budget-discovery-engine
pip install pandas numpy matplotlib seaborn sqlalchemy openpyxl
python beauty_budget_engine.py
```

> Both `Cleaned_Sephora_Products.xlsx` and `chemical_synonyms.csv` must be in the same folder as the script.

---

## Author
**[Saanvi Koti]** — CS 210: Data Management for Data Science
