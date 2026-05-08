# Beauty-Budget Discovery Engine — CS210 Final Project

## Project Overview

The Beauty-Budget Discovery Engine is a data management and ingredient similarity pipeline designed to identify budget-friendly "dupe" alternatives for luxury Sephora products. As the beauty industry continues to grow into a $130+ billion market, consumers are routinely overpaying for products that share nearly identical formulations with cheaper alternatives — simply because the ingredient data is too messy and unstandardized to compare directly.

This project tackles that problem end-to-end: loading a 7,500+ product Sephora catalog, standardizing every ingredient synonym through a custom-built chemical synonym dataset, storing results in a hybrid database architecture, and applying Jaccard Set Similarity to rank budget alternatives using a composite Value Score.

---

## Objectives

- Load and clean a real-world Sephora product dataset across 8 documented steps
- Build and load a **chemical_synonyms.csv** dataset to standardize ingredient synonym noise (e.g. "Aqua" → "water", "Tocopherol" → "vitamin e")
- Store structured product data in SQLite (mimicking PostgreSQL) and variable-length ingredient documents in JSON (mimicking MongoDB)
- Engineer key features: price tier, price-per-ingredient, luxury markup index
- Perform exploratory data analysis across 12 visualizations
- Apply Jaccard Set Similarity to detect ingredient overlap between luxury and budget product pairs
- Rank all qualifying dupe pairs using a weighted Value Score formula
- Expose results through an Interactive Dupe Finder function

---

## Dataset

**Primary:** https://www.kaggle.com/datasets/nadyinky/sephora-products-and-skincare-reviews?resource=download 

| Column | Description |
|--------|-------------|
| product_id | Unique product identifier |
| product_name | Full product name |
| brand_name | Brand name |
| price_usd | Retail price in USD ($3–$495) |
| rating | Consumer rating (1–5) |
| primary_category | Product category (Skincare, Makeup, Hair, etc.) |
| ingredients | Raw INCI ingredient string |
| cleaned_ingredients | Pre-lowercased, comma-separated ingredient string |

**Secondary (custom-built):** `chemical_synonyms.csv`

| Column | Description |
|--------|-------------|
| raw_name | Raw ingredient token as it appears in the dataset |
| canonical_name | Standardized canonical name for comparison |

- 300+ rows built from the 384 highest-frequency ingredient tokens in the Sephora catalog
- Covers water variants, UV filters, botanical oils, emollients, preservatives, actives (vitamin C, vitamin E, hyaluronic acid, niacinamide, retinol), fragrance allergens, and more
- Loaded at runtime with `pd.read_csv()` — **not hardcoded in Python**

---

## Technologies Used

**Programming Language**
- Python 3.10+

**Libraries**
- Pandas
- NumPy
- Matplotlib
- Seaborn
- SQLAlchemy
- json (standard library)

---

## Methodology

### 1. Data Cleaning & Preprocessing (No Regular Expressions)
The dataset was cleaned across 8 explicit steps:
- Stripping whitespace from all string columns
- Standardizing category casing with `.str.title()`
- Dropping rows missing critical fields (product_id, brand, price)
- Removing duplicate product_id entries
- Coercing price to numeric and removing invalid values
- Clipping price outliers at the 99th percentile
- Imputing missing ratings with each product's **category median**
- Filtering to 4 focus categories: Skincare, Makeup, Hair, Bath & Body → **5,973 rows**

### 2. Chemical Synonym Dataset & Ingredient Standardization
- `chemical_synonyms.csv` is loaded and converted to a Python dictionary for O(1) token lookups
- Each product's ingredient string is split on commas only (`str.split()` — no regex)
- Tokens are looked up in the dictionary; unmapped tokens are kept as-is
- Result per product: a Python **set** of canonical ingredient names (`canonical_ings`)

### 3. Feature Engineering
New columns derived from the cleaned dataset:

| Feature | Formula / Logic |
|---------|----------------|
| `price_tier` | Luxury (≥$60) / Budget (<$30) / Mid-Range |
| `ingredient_count` | Number of canonical ingredients per product |
| `price_per_ingredient` | `price_usd / ingredient_count` |
| `luxury_markup_index` | `price_usd / category median price` |
| `rating_bucket` | Binned rating tiers (Excellent / Good / Average / Poor) |
| `ing_count_bucket` | Binned complexity tiers (Minimal / Lean / Standard / Rich / Complex) |

### 4. Hybrid Database Storage
- **SQLite** (`beauty_budget.db`) — Products table with structured attributes, queried via SQL
- **JSON** (`ingredient_docs.json`) — One document per product storing the full canonical ingredient list (mimicking MongoDB), since products have variable-length lists (5–60+ ingredients)

### 5. Exploratory Data Analysis (EDA)
12 visualizations were produced including:
- Price distribution by category (box plot)
- Price tier stacked bar charts
- Price vs. rating scatter plots with trend lines
- Feature correlation heatmap
- Price-per-ingredient by category and tier
- Ingredient count distributions (Luxury vs. Budget)
- Brand luxury markup rankings
- Synonym coverage chart (from chemical_synonyms.csv)
- Value Score distributions and dupe rankings

### 6. Jaccard Similarity — Dupe Detection Engine
- Every Luxury vs. Budget pair within the same category is compared
- **Jaccard(A, B) = |A ∩ B| / |A ∪ B|**
- Two filters must both pass:
  - Jaccard similarity **≥ 0.60** (60% ingredient overlap)
  - Budget price **≤ 30%** of the luxury price

### 7. Value Score Computation & Ranking
Qualifying dupe pairs are ranked by a composite Value Score:

```
Value Score = (Jaccard Similarity × 0.50)
            + (Normalized Price Saving % × 0.30)
            + (Budget Product Rating ÷ 5 × 0.20)
```

| Weight | Signal | Reason |
|--------|--------|--------|
| 50% | Jaccard similarity | Core measure of ingredient equivalence |
| 30% | Normalized price savings | Primary consumer benefit |
| 20% | Budget product rating | Ensures recommendation quality |

---

## Key Findings

- Price vs. rating correlation was **r ≈ 0.04** for Skincare and **r ≈ 0.02** for Makeup — price is not a reliable signal of product quality
- Luxury Skincare products averaged **~7.6× more per ingredient** than Budget alternatives
- Luxury and Budget products had **similar ingredient count distributions** — premiums are driven by branding, not formulation complexity
- The Makeup category surfaced a **100% ingredient overlap** dupe pair ($205 vs $50) — same formulation, different packaging
- Relaxing the Interactive Dupe Finder threshold to 0.35 similarity returned useful recommendations for most luxury products queried

---

## Project Structure

```
├── beauty_budget_engine.py           # Full pipeline — single script, end-to-end
├── chemical_synonyms.csv             # Custom ingredient synonym dataset (300+ mappings)
├── Cleaned_Sephora_Products.xlsx     # Primary Sephora product dataset
├── beauty_budget.db                  # SQLite products table (generated on run)
├── ingredient_docs.json              # JSON ingredient documents (generated on run)
├── dupe_results.csv                  # All qualifying dupe pairs (generated on run)
├── dupe_results_scored.csv           # Dupe pairs ranked by Value Score (generated on run)
├── project_summary.csv               # Per-category success metrics (generated on run)
├── eda_01_price_by_category.png
├── eda_02_price_tier_stacked.png
├── eda_03_price_vs_rating.png
├── eda_04_correlation_heatmap.png
├── eda_05_price_per_ingredient.png
├── eda_06_ingredient_count_hist.png
├── eda_07_brand_markup.png
├── eda_08_synonym_coverage.png
├── eda_09_luxury_vs_budget_count.png
├── eda_10_value_score_dist.png
├── eda_11_top20_dupes.png
├── eda_12_sim_vs_saving.png
├── dupe_finder_result.png
└── README.md
```

---

## Installation & Setup

### Requirements
- Python 3.10+
- Jupyter Notebook or VS Code

### Setup Steps

**1. Clone the repository**
```bash
git clone https://github.com/yourusername/beauty-budget-discovery-engine.git
```

**2. Go into the project folder**
```bash
cd beauty-budget-discovery-engine
```

**3. Create a virtual environment**
```bash
python -m venv venv
```

**4. Activate the environment**
```bash
# Mac/Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

**5. Install dependencies**
```bash
pip install pandas numpy matplotlib seaborn sqlalchemy openpyxl
```

---

## How to Run the Project

**1. Make sure both input files are in the same folder as the script**
```
Cleaned_Sephora_Products.xlsx
chemical_synonyms.csv
```

**2. Run the pipeline**
```bash
python beauty_budget_engine.py
```

Or open in Jupyter and run all cells top to bottom.

---

## Expected Outputs

**Console:**
```
Sephora Products loaded: 7,519 rows x 8 columns
Dictionary entries loaded from CSV: 300+
Step 8 -- Focus-category subset: 5,973 rows
[DB] Products table -> sqlite:///beauty_budget.db (7,519 rows)
[DB] Ingredient_Docs (JSON) -> ingredient_docs.json (5,973 docs)
Data Cleanliness Rate: ~18.0%
Total dupe pairs: [n]
Value Score computed -> dupe_results_scored.csv
```

**Files generated:**
- `beauty_budget.db` — SQLite products table
- `ingredient_docs.json` — MongoDB-style ingredient documents
- `dupe_results_scored.csv` — All dupe pairs ranked by Value Score
- `project_summary.csv` — Per-category success metrics
- 12 EDA `.png` chart files
- `dupe_finder_result.png` — Interactive Dupe Finder output chart

**Graphs produced:**
- Price distribution by category
- Price tier stacked bar charts
- Price vs. rating scatter (Skincare & Makeup)
- Feature correlation heatmap
- Price-per-ingredient by tier
- Ingredient count histogram (Luxury vs. Budget)
- Brand markup index rankings
- Synonym coverage chart
- Value Score distribution
- Top 20 dupe pairs ranked chart
- Similarity vs. price saving scatter
- Interactive Dupe Finder bar chart

---

## Running the Interactive Dupe Finder

At the end of the script, call `find_dupes_for_product()` with any luxury product ID:

```python
find_dupes_for_product(
    luxury_product_id='P12345',
    top_n=5,
    sim_cutoff=0.35,    # minimum Jaccard similarity (relaxed for interactive use)
    max_budget=30       # optional price ceiling for budget alternatives
)
```

**Example output:**
```
High ingredient overlap, low price  -->  DUPE FOUND
Low ingredient overlap              -->  No match above threshold
```

---

## Important Notes

- Both `Cleaned_Sephora_Products.xlsx` and `chemical_synonyms.csv` must be in the **same folder** as the script
- The `chemical_synonyms.csv` is a proper data artifact — edit it directly to add new synonym mappings without changing any Python code
- Fragrance products are excluded from ingredient analysis — their lists are dominated by allergen disclosure chemicals (limonene, linalool, coumarin) rather than functional actives

---

## Author

**[Your Name]**
CS 210: Data Management for Data Science
