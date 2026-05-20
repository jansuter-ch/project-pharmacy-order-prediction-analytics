# Pharmacy Order Prediction Analytics

Predictive analytics pipeline for a dynamically priced mail-order pharmacy, built on the Data Mining Cup 2017 dataset. The pipeline covers two tasks:

- Classification: predict whether a product will be ordered in a session (order = 1 or 0)
- Regression: predict the sales volume if a product is ordered

The pipeline follows the CRISP-DM methodology and evaluates user behavior (clicks, basket interactions, purchases) combined with product attributes and dynamic pricing over time.

---

## Contents

- [Project Structure](#project-structure)
- [Dataset](#dataset)
- [Quick Start](#quick-start)
- [Pipeline Overview](#pipeline-overview)
- [Modeling Approach](#modeling-approach)

---

## Project Structure

```
project-pharmacy-order-prediction-analytics/
├── data/
│   ├── raw/
│   └── processed/
├── src/
│   ├── eda.ipynb
│   ├── classification_task.ipynb
│   └── regression_task.ipynb
├── requirements.txt
└── README.md
```

---

## Dataset

The dataset is from the Data Mining Cup 2017 competition. It is not included in this repository due to competition licensing. Download it directly from [DMC 2017](https://www.data-mining-cup.com/reviews/dmc-2017/) and place the files as follows:

- `data/raw/train.csv`: daily session records (~2.5 million rows) including click, basket, and order events, competitor prices, active pharmacy prices, and revenue
- `data/raw/items.csv`: static product attributes including manufacturer, generic flags, packaging content, category, reference price, and campaign flags

---

## Quick Start

1. Create and activate a virtual environment:
   - macOS/Linux: `python3 -m venv .venv && source .venv/bin/activate`
   - Windows: `python -m venv .venv && .venv\Scripts\activate`

2. Install dependencies:
   `pip install --upgrade pip && pip install -r requirements.txt`

3. Place the raw data files in `data/raw/` as described above.

4. Run the notebooks in `src/` in order:
   - `eda.ipynb`: cleans data, explores distributions and missing values, prepares the processed dataset
   - `classification_task.ipynb`: builds and evaluates order prediction models
   - `regression_task.ipynb`: builds and evaluates sales volume forecasting models

---

## Pipeline Overview

### Data Merging and Sorting

The two source files are merged via the product ID (`pid`), connecting session-level behavior with static product attributes. The merged dataset is sorted chronologically by `day` and `lineID` to enforce time order and prevent data leakage.

### Feature Engineering

Features extracted from the raw data:

- Price features: `priceRatio` (active price vs. RRP), `priceVsCompetitor`, `priceDiscount`
- Time features: cyclical day encoding using sine and cosine transformations
- Interaction features: Advertising x Price, Availability x Price
- Historical behavior features: item order rate, price volatility, prior views across past time lags

### Evaluation Strategy

Instead of a random train-test split, the pipeline uses an expanding time window approach to simulate real-world forecasting:

- Train on Days 1 to N, validate on Day N+1
- Target encodings (e.g. Manufacturer to Purchase Probability) are computed strictly from historical data to avoid leakage

This approach ensures the model is evaluated on data it could not have seen during training, which produces a more realistic estimate of production performance.

---

## Modeling Approach

Three models are compared for each task: Decision Tree, Random Forest, and XGBoost. XGBoost performs strongest across both classification and regression.

Preprocessing steps applied before modeling:

- Robust scaling to handle outliers
- Median imputation for missing values
- Correlation filtering to remove highly collinear features before grid search tuning

Evaluation metrics:

- Classification: F1 Score, ROC AUC
- Regression: RMSE, R²
