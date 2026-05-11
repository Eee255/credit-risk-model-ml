# credit-risk-model-ml

## 📌 Project Overview

**Lauki Finance** is an NBFC (Non-Banking Financial Company) that lends money to individuals and businesses. Unlike banks, they cannot accept deposits — they borrow from banks at ~10% and lend at ~18%. Every bad loan directly destroys their capital.

> **The Problem:** Manual credit evaluation is slow, inconsistent, and error-prone.
> **The Solution:** An ML-powered credit scorecard that scores every applicant before a human decision is made.

This project was built for the **Risk Unit at Lauki Finance** to measure credit risk and automatically categorise loan applicants as **Poor / Average / Good / Excellent** — working as a Phase 1 deployment with Phase 2 monitoring and STP (Straight Through Processing) designed for post-trial automation.

---

## 🎯 Business Success Criteria

| Metric | Target | Achieved |
|--------|--------|----------|
| Recall (Default class) | > 90% | ✅ **91%** |
| Precision (Default class) | > 50% | ✅ **72%** |
| AUC-ROC | > 85% | ✅ **98%** |
| Gini Coefficient | > 85% | ✅ **96%** |
| KS Statistic (top 3 deciles) | > 40% | ✅ **85.98%** |
| Model Interpretability | Required | ✅ Logistic Regression |

---

## 🏗️ Project Architecture

```
Raw Data (3 CSV files)
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│                   DATA PIPELINE                         │
│  customers.csv + loans.csv + bureau_data.csv            │
│         └──── Merge on cust_id ────┘                    │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│                PREPROCESSING                            │
│  • Fix typos  • Impute missing values                   │
│  • Business rule validation (fee ≤ 3%)                  │
│  • Stratified Train/Test Split (75/25) ← BEFORE EDA     │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│             FEATURE ENGINEERING                         │
│  • Loan-to-Income (LTI) Ratio                           │
│  • Delinquency Ratio                                    │
│  • Avg DPD per Delinquency                              │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│              FEATURE SELECTION                          │
│  • VIF → remove multicollinear features (VIF > 10)      │
│  • WOE / IV → remove weak predictors (IV < 0.02)        │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│                  MODELLING                              │
│  • Class imbalance handled with SMOTE-Tomek             │
│  • Logistic Regression | XGBoost | Random Forest        │
│  • Hyperparameter tuning with Optuna (50 trials)        │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│                 EVALUATION                              │
│  • AUC-ROC | Gini | KS Statistic | Decile Table         │
│  • Rank Ordering | Classification Report                │
└─────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────┐
│              CREDIT SCORECARD                           │
│  Score = 300 + (1 − P(default)) × 600                   │
│  Poor (300–499) | Average (500–649)                     │
│  Good (650–749) | Excellent (750–900)                   │
└─────────────────────────────────────────────────────────┘
        │
        ▼
  🚀 Streamlit Application (model_data.joblib)

```



## 🤖 Model Results

### Why Logistic Regression over XGBoost?

| Model | AUC | Gini | Chosen? |
|-------|-----|------|---------|
| **Logistic Regression** | **98%** | **96%** | ✅ **Yes** |
| XGBoost | 99% | 96% | ❌ Explainability gap |
| Random Forest | 97% | 95% | ❌ Baseline reference |

> NBFCs must explain every rejection to the borrower and the RBI. Logistic regression coefficients map directly to feature importance — every decision is fully auditable. XGBoost's marginal 1% AUC gain did not justify losing interpretability in a regulated context.

### Decile Table (Rank Ordering)

| Decile | Event Rate | Cumulative Events | KS |
|--------|-----------|------------------|-----|
| 10 (Top Risk) | **72.0%** | 72.0% | 68.0% |
| 9 | **12.7%** | 98.6% | **85.9%** ← Peak KS |
| 8 | 8.8% | 99.9% | 76.9% |
| 7 | 10.0% | 99.9% | 66.6% |
| 1–6 | 0.0% | 100% | ~0% |

**Business Insight:** Review just the top 20% of applicants (2 deciles) → capture **98.6% of all defaulters**.

---

## 🏆 Credit Scorecard

```
Credit Score = 300 + (1 − P(default)) × 600

P(default) = 0.05  →  Score = 870  →  EXCELLENT  → Auto-approve
P(default) = 0.25  →  Score = 750  →  GOOD        → Standard approval
P(default) = 0.50  →  Score = 600  →  AVERAGE     → Manual review
P(default) = 0.85  →  Score = 390  →  POOR        → Reject
```

| Score Range | Category | Action |
|-------------|----------|--------|
| 750 – 900 | 🟢 **Excellent** | Auto-approve, best rates |
| 650 – 749 | 🔵 **Good** | Approve, standard terms |
| 500 – 649 | 🟡 **Average** | Manual review |
| 300 – 499 | 🔴 **Poor** | Reject / co-applicant needed |

---

## 🚀 Streamlit App

> **Live Demo:** https://credit-risk-model-ml-bbmafg85ayynrks34tzzbn.streamlit.app/

The Streamlit app allows credit officers to input borrower details and get an instant credit score.

**To run locally:**

```bash
# Clone the repository
git clone https://github.com/yourusername/credit-risk-model-lauki-finance.git
cd credit-risk-model-lauki-finance

# Install dependencies
pip install -r requirements.txt

# Launch the app
streamlit run app.py
```

**App inputs:**
- Age, Income, Employment Status
- Loan Amount, Tenure, Purpose
- Credit Utilization Ratio
- Delinquency history

**App output:**
- Probability of Default
- Credit Score (300–900)
- Risk Category (Poor / Average / Good / Excellent)

---
## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **Python 3.10+** | Core language |
| **Pandas / NumPy** | Data manipulation |
| **Scikit-learn** | Logistic Regression, preprocessing, metrics |
| **XGBoost** | Gradient boosting model (challenger) |
| **Imbalanced-learn** | SMOTE-Tomek for class imbalance |
| **Optuna** | Bayesian hyperparameter optimisation |
| **Matplotlib / Seaborn** | EDA visualisations |
| **Streamlit** | Web application deployment |
| **Joblib** | Model serialisation |

---

## 📦 Installation

```bash
# Clone
git clone https://github.com/yourusername/credit-risk-model-lauki-finance.git
cd credit-risk-model-lauki-finance

# Create virtual environment
python -m venv venv
source venv/bin/activate      # Linux/Mac
venv\Scripts\activate         # Windows

# Install
pip install -r requirements.txt
```

**`requirements.txt`**
```
pandas>=1.5.0
numpy>=1.23.0
scikit-learn>=1.2.0
xgboost>=1.7.0
imbalanced-learn>=0.10.0
optuna>=3.0.0
streamlit>=1.20.0
joblib>=1.2.0
matplotlib>=3.6.0
seaborn>=0.12.0
```
