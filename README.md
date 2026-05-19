# Mail-Order Pharmacy Analytics Project

This project builds a predictive analytics pipeline for a dynamically priced mail-order pharmacy. The goal is two-fold:
1. **Classification:** Predict whether a product will be bought in a session (`order = 1` or `0`).
2. **Regression:** Predict the sales volume/quantity if a product is bought.

Based on the Data Mining Cup 2017 scenario, the pipeline evaluates user behavior (clicks, basket interactions, purchases) combined with product attributes and dynamic pricing over time to optimize revenue and understand the interactions between different product features.

## Project Structure

```text
analytics-project/
├── data/
│   ├── raw/                  # Place raw CSV files here (items.csv, train.csv)
│   └── processed/            # Intermediary datasets and modeling evaluations
├── src/
│   ├── eda.ipynb             # Exploratory Data Analysis & quality checks
│   ├── classification_task.ipynb  # Purchasing probability prediction
│   └── regression_task.ipynb # Sales/revenue forecasting
├── requirements.txt          # Python dependencies
└── README.md                 # Project documentation
```

## Quick Start

1. **Create and activate a virtual environment:**
   - macOS/Linux: `python3 -m venv .venv && source .venv/bin/activate`
2. **Install dependencies:**
   - `pip install --upgrade pip && pip install -r requirements.txt`
3. **Place input data in the `data/raw/` folder:**
   - `data/raw/train.csv`
   - `data/raw/items.csv`
4. **Run the notebooks in `src/` sequentially:**
   - First, run `src/eda.ipynb` to clean the data and explore missing values.
   - Second, run `src/classification_task.ipynb` for predicting order classification.
   - Third, run `src/regression_task.ipynb` for predicting sales quantity based on orders.

## Data Overview

The data connects temporal behavior events (`train.csv`) to static item attributes (`items.csv`).

- **items.csv**: Contains fixed attributes like the manufacturer, generic flags, packaging content, category, reference price, and campaign flags.
- **train.csv**: Tracks daily session information over ~2.5 million records, recording `click`, `basket`, or `order` events. Also documents daily competitor's price, the pharmacy's active price, and resulting revenue.

### 1. Data Merging and Sorting
- **Merge**: Connects runtime behavior attributes and static product details via the product ID (`pid`).
- **Time Sorting**: Crucial step to enforce time logic; data is chronologically sorted by `day` and `lineID` to prevent data leakage (the model must not see future timelines during training).

### 2. Feature Engineering
Extracts complex insights from basic fields:
- **Price Features**: `priceRatio` (Price vs. RRP), `priceVsCompetitor`, `priceDiscount`.
- **Time Features**: Cyclical cyclical day representation (sine/cosine).
- **Interactions**: Advertising × Price, Availability × Price.
- **Historical Behaviors**: Item order rate, price volatility, and prior views over past lags.

### 3. Evaluation Strategy (Rolling Time Window)
Instead of a simple random split, the model is trained with **Expanding/Rolling Time Windows** to simulate real-world production accurately:
- Example: Train on Days 1–7 -> Validate on Day 8.
- Target Encoded mappings (like Manufacturer -> Purchase Probability) strictly rely on retrospective data mapped recursively.

### 4. Classification & Regression Modeling
Models are pitted against each other (Decision Tree, Random Forest, XGBoost). XGBoost typically emerges as the strongest candidate. Optimization utilizes robust scaling, missing value imputation via median strategies, and correlation filters to prevent overfitting prior to final grid search tuning.

### Core Philosophy
* **No Time Leakage**: Features are historically strict.
* **Realistic Testing**: Expanding windows mimic real future conditions.
* **Data-driven Storytelling**: F1 Score and ROC AUC reflect tangible business impact factors.
