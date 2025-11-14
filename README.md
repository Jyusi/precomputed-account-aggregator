# Precomputed Account Aggregator for Fraud Detection

This repository is an **archive of the Fraud Account Detection Project**, with a special focus on scalable dataset preparation for machine learning and analysis. The core workflow here is designed to process, aggregate, and transform large-scale transactional data from blockchain and/or financial ledgers, with the aim of exposing features and patterns indicative of fraud or abnormal account behavior.

---

## Project Overview

Financial and transactional systems create massive logs of operations between accounts. Fraud detection in such environments requires not just isolating suspicious transactions but also understanding **behavioral context**—how accounts interact over time and across the network. This project tackles that need by:

- Efficiently extracting features from transaction graphs
- Categorizing accounts based on ground-truth or predicted flags (e.g., normal/abnormal)
- Aggregating and transforming data for advanced analysis and predictive modeling

---

## Workflow: Data Preparation to Model Building

### 1. **Data Trimming & Aggregation (`main_aggregator.ipynb`)**

#### a. **Import and Clean Raw Data**
- Loads transaction data (`transactions.csv`) and account flag data (`train_acc.csv`, `test_acc_predict.csv`) with robust type overrides using [Polars](https://pola.rs/) for speed and memory efficiency.
- Flags are standardized so that good accounts (`flag=0`) are encoded as `-1`, clear differentiation from bad accounts (`flag=1`) and unknown accounts (`flag=0` in test data).

#### b. **Feature Engineering**
- **Transaction-level features** (profit, cost, ratios, temporal tags): 
    - For each transaction: profit (`value - gas * gas_price`), net value, gas cost, value/gas ratios, and binary features such as whether the transaction is profitable, on weekends, at night, etc.
    - **Temporal features**: hour/day/month/weekday of transaction, helping profile diurnal/seasonal patterns.

#### c. **Account-level Graph Construction**
- **Accounts encoded as categorical variables** for compact integer mapping.
- **Outgoing and incoming transaction arrays** are built for each account, sorted and indexed for rapid lookup.
- Graph structures (`edges_out`, `edges_in`) enable slicing out all transactions linked to any account.
- Functions for neighbor lookups (`find_to_nei`, `find_from_nei`) and path searches (`find_forward_paths`, `find_backward_paths`) support exploration of transaction sequences of arbitrary depth.

#### d. **Aggregating Features for Downstream Analysis**
- **Streaming feature accumulation** (via `RunningStats`): Means, variances, min/max for key numeric features are built efficiently in a streaming manner.
- Per-account aggregates are computed for different flags and types (‘normal’, ‘abnormal’, A/B directionality, temporal bins).
- Data is further pruned, deduplicated, and restructured to produce wide tabular summaries with hundreds (or thousands) of features per account.

#### _Result_: **A highly structured, feature-rich account-level dataset.**

---

### 2. **Analysis & Model Building (`main_f1.ipynb`)**

#### a. **Advanced Feature Engineering**
- The dataset from **main_aggregator** is loaded and processed further:
    - **Derived ratios, contrasts, and population-relative features** (e.g., abnormal-to-normal ratios, z-scores, quartile/season contrasts).
    - **Entropy and concentration metrics:** Quantifies variety and distribution of temporal or transactional patterns (e.g., how scattered an account’s activity is across hours/days/months).
    - **Volatility, burstiness, and activity flags**: For each account, signals like burst ratio, window-based entropy, and low-activity flags are calculated.

#### b. **Data Consolidation**
- Data from multiple sources (`data1_df`, `data2_df`, etc.) is loaded, featured, and concatenated into a single large table.
- Additional windowed features (from raw transactions) are joined in, using robust joining logic that ensures correct mappings and no data loss.

#### c. **Supervised Modeling**
- **CatBoost Classifier** (or similar) is tuned with **Optuna** for fast yet robust hyperparameter optimization, including dynamic weighting for minority (fraudulent) class.
- Feature selection, ranking, and importance assertions are performed to help focus on the most predictive signals.
- Cross-validation and advanced threshold tuning (maximizing F1 at precision-recall curve best points) ensure that fraudulent accounts are optimally detected.

#### _Result_: **A ready-to-use set of features and workflows for high-performance classification, anomaly detection, or further analytics.**

---

## Getting Started

### Installation

```bash
# Clone the repository
git clone https://github.com/Jyusi/precomputed-account-aggregator.git
# Install dependencies
pip install -r requirements.txt
```
See [requirements.txt](https://github.com/Jyusi/precomputed-account-aggregator/blob/main/requirements.txt) for full package list (Polars, Numpy, CatBoost, Optuna, Scikit-learn, etc).

---

## Key Repository Elements

- **main_aggregator.ipynb:** Data trimming, cleaning, transaction graph construction, feature extraction. _Preparation stage._
- **main_f1.ipynb:** Feature engineering, tabular consolidation, model training (CatBoost, F1-centric optimization), ranking. _Modeling stage._
- **README.md:** Project documentation and workflow guidance.
- **requirements.txt:** All dependencies for dataset preparation, analysis, and modeling.

---

## Why this pipeline?

- **Scalable**: Handles millions of transactions efficiently, using parallelism and columnar data structures.
- **Flexible**: Facilitates both graph-based and tabular analysis.
- **Rich Features**: Aggregates behavioral, temporal, and statistical signals critical for fraud detection.
- **Actionable**: Outputs are ready for machine learning but also usable for audit, analytics, and further network investigation.

---

## Citation & Reuse

If you use this dataset preparation or modeling workflow, please cite or reference this repository. For extensions, contributions, or collaboration, open an issue or submit a PR.

---

_This archive is the result of extensive research and engineering for fraud analytics. For questions, reach out via GitHub Issues._
