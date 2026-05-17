# KKBox Music Streaming — Churn Prediction

A machine learning project predicting customer churn for KKBox, a Taiwanese music streaming company, using 3 months of behavioral data.

**[📊 View Interactive Dashboard](https://pilee11.github.io/churn-prediction/dashboard.html)** | **[📓 View Notebook](https://colab.research.google.com/drive/1tH9yRc9DWme74MVphQIz4XbkAdLpXp-V)**

---

## Project Overview

The goal of this project is to identify customers who are likely to cancel their subscription, using behavioral patterns observed over January–March 2017. The model enables proactive retention interventions before the subscription expires.

- **671,593 users** analyzed
- **2.4% churn rate** (16,115 churners)
- **Model AUC: 0.764** on held-out test set
- **Recall: 64%** — identifies 64% of churners before they leave

---

## Data

Source: [KKBox Churn Prediction Challenge](https://www.kaggle.com/competitions/kkbox-churn-prediction-challenge) — Kaggle

| File | Description |
|------|-------------|
| `members_v3.csv` | User profile: age, gender, city, registration date |
| `transactions_v2.csv` | Subscription history: plan, price, cancellations |
| `user_logs.csv` / `user_logs_v2.csv` | Daily listening behavior |
| `train_v2.csv` | Churn labels |

**Target population:** Users whose subscription expired in April 2017, registered before January 2017.

---

## Methodology

### 1. Data Collection
Joined 4 raw files using **DuckDB** into a single table with one row per user, aggregating listening behavior by month (January, February, March 2017).

### 2. Missing Values
| Variable | Strategy |
|----------|----------|
| Listening features | Fill with 0 (no data = didn't listen) |
| auto_renew_cancelled | Fill with 0 (no transaction = no action) |
| Gender | Fill with "Unknown" (separate category) |
| Age (invalid) | Fill with median + binary flag `age_valid` |

### 3. Feature Engineering
12 new features created on top of raw variables:

- **Trend features**: % change in listening metrics Jan → Mar
- **Average features**: mean across 3 months
- **Cost per second**: amount paid ÷ total listening seconds
- **Tenure group**: 0–12 / 13–36 / 36+ months since registration

### 4. Model Selection
Compared 4 models using **5-fold stratified cross-validation**:

| Model | AUC |
|-------|-----|
| **LightGBM** | **0.756** ✅ |
| XGBoost | 0.731 |
| Random Forest | 0.674 |
| Logistic Regression | 0.648 |

### 5. Hyperparameter Tuning
RandomizedSearchCV with 30 iterations — best AUC improved from 0.756 → 0.762.

### 6. Feature Importance (Top 5)
| Feature | Importance |
|---------|-----------|
| actual_amount_paid | 423 |
| active_days_trend_pct | 344 |
| age | 266 |
| listen_trend_pct | 236 |
| songs_completed_trend_pct | 222 |

---

## Key Findings

- **Churners are more active listeners** than non-churners — churn is a conscious decision, not passive drift
- **Higher-paying users churn more** — users paying 150–199 TWD churn at 4.7% vs 0.8% for 1–99 TWD
- **Declining activity trend** is a stronger predictor than absolute listening level
- **Two churner types**: active (90%) who need a better value proposition, and passive (10%) who need reminders

---

## Business Recommendations

1. **Focus retention on high-paying subscribers** — offer loyalty discounts at renewal for the 150–199 TWD tier
2. **Build an early-warning trigger** — flag users whose active days drop >20% Jan→Mar and send automated retention offers
3. **Differentiate strategy** — separate campaigns for active vs passive churners
4. **Investigate pricing perception** — survey the premium tier to identify unmet expectations

---

## Tech Stack

`Python` `DuckDB` `LightGBM` `scikit-learn` `pandas` `Plotly` `HTML/CSS/JS`

---

*BGU — Statistics & Data Analysis with Management | Final Year Project*
