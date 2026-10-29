# Late Deliveries and Customer Review Scores on the Olist Marketplace

**Course:** CSE558 Data Science (Monsoon 2026, IIIT-Delhi)  
**Group:** Team tiriri — Divyanshi Gupta, Palak Khatri, Hansika Singh, Rishit Sansanwal, and Aman Pratap Singh  
**Repository:** <https://github.com/hansika24225/Data-Science-Midterm-project>

## Overview & Research Question

**Research question:** Are late deliveries associated with lower customer review scores, and does this relationship hold when comparing orders that experienced similar actual waiting times?

**Report:** [`Final_Project_Report.pdf`](Final_Project_Report.pdf) (8 pages, within the 8-page limit). Its LaTeX source, [`Final_Project_Report.tex`](Final_Project_Report.tex), is formatted on the official CSE558 midterm report template, with Sections 1–6 corresponding directly to the project guidelines.

### Summary of Key Findings
- **Overall rating drop:** Orders delivered after their promised date receive an average of 2.02 fewer stars ($2.27$ vs. $4.29$) and receive 1–2 star ratings 62.4% of the time, compared with 9.2% for on-time deliveries.
- **Controlling for waiting time:** Across comparable waiting-time bands (8–42 days from purchase to delivery), late deliveries still exhibit 7.03 times the odds of a 1–2 star rating (95% CI: 6.48–7.63).
- **Role of review timing:** Nearly 70.0% of late orders are reviewed before the package physically arrives. When restricting the sample to reviews submitted **after** delivery, the association remains statistically significant but drops substantially in magnitude (pooled OR = 1.41, 95% CI: 1.24–1.60), and the correlation between delay duration and review score disappears ($\rho_S = -0.03, p = 0.13$).
- **Observational scope:** Because delivery delays are not randomly assigned, all reported relationships represent observational associations rather than causal effects.

---

## 1. Analysis Pipeline & Notebooks

| Notebook | Stage | Description | Output Directory |
|---|---|---|---|
| `notebooks/01_build_analysis_table.ipynb` | 1. Data Integration | Verifies the nine raw Kaggle CSV files, filters to delivered orders with a single review, aggregates multi-item and multi-payment records to the order level, constructs features and the encoded matrix, and checks join cardinalities. | `outputs/phase1/` |
| `notebooks/02_data_quality_audit.ipynb` | 2. Data Quality & Cleaning | Audits timestamps, extreme values, payment totals, review text, encodings, and missingness; logs all cleaning decisions; excludes 3 anomalous records where the review timestamp precedes order purchase; outputs the final analysis dataset. | `outputs/phase2/` |
| `notebooks/03_eda.ipynb` | 3. Exploratory Analysis | Computes descriptive statistics with 95% Wilson confidence intervals across rating distributions, review timing, waiting-time strata, states, product categories, monthly volume, and Portuguese comment text (12 figures). | `outputs/phase3/` |
| `notebooks/04_hypothesis_testing.ipynb` | 4. Statistical Inference | Runs five pre-specified hypothesis tests with Holm–Bonferroni multiple-testing correction, computes effect sizes and 95% CIs, verifies test assumptions, evaluates sensitivity subsets (S1–S4), and compares primary tests against parametric/non-parametric alternatives. | `outputs/phase4/` |

- **Final analytical sample:** 95,296 unique delivered orders (one row per order: 6,353 late and 88,943 on time or early).
- **3M dimensionality check:** The encoded feature matrix has dimensions $95{,}296 \times 167 = 15{,}914{,}432$ cells, exceeding the course requirement of $\ge 3{,}000{,}000$ cells.

---

## 2. Dataset Setup

The raw dataset is the *Brazilian E-Commerce Public Dataset by Olist* from Kaggle:  
<https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce> (9 relational CSV tables, licensed under CC BY-NC-SA 4.0).

To run the notebooks from scratch, place the dataset in the project root using either option:
- **Option A:** Extract the nine CSV files into the `data/` directory.
- **Option B:** Place the downloaded Kaggle `.zip` archive in the project root; Notebook 01 will automatically extract the CSVs into `data/`.

---

## 3. Running the Code & Recompiling the Report

### Python Environment
Install the required dependencies (`pandas`, `numpy`, `pyarrow`, `matplotlib`, `scipy`, `statsmodels`, `jupyter`) using:

```bash
pip install -r requirements.txt
```

You can execute the full pipeline in order (`01` $\to$ `02` $\to$ `03` $\to$ `04`) either interactively in Jupyter (*Kernel $\to$ Restart & Run All*) or from the terminal:

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/01_build_analysis_table.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/02_data_quality_audit.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/03_eda.ipynb
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=1200 notebooks/04_hypothesis_testing.ipynb
```

All random resampling routines (such as BCa bootstrap confidence intervals) use a fixed random seed (`SEED = 558`) for exact reproducibility.

### Compiling the LaTeX Report
To compile [`Final_Project_Report.tex`](Final_Project_Report.tex):
- **Local TeX installation:** Run `pdflatex Final_Project_Report.tex` twice from the project root directory.
- **Overleaf:** Upload `Final_Project_Report.tex` along with the `outputs/phase3/figures/` and `outputs/phase4/figures/` folders, keeping the relative directory paths intact.

---

## 4. Repository Structure

```text
Data-Science-Midterm-project/
├── README.md                      # Project documentation and instructions
├── Final_Project_Report.pdf       # Compiled 8-page report on the CSE558 template
├── Final_Project_Report.tex       # LaTeX report source
├── requirements.txt               # Python package dependencies
├── data_manifest.csv              # Expected filenames, row counts, and SHA-256 checksums
├── resources/
│   └── stopwords_pt.txt           # Portuguese stopword list for review text EDA
├── notebooks/
│   ├── 01_build_analysis_table.ipynb
│   ├── 02_data_quality_audit.ipynb
│   ├── 03_eda.ipynb
│   └── 04_hypothesis_testing.ipynb
├── outputs/
│   ├── phase1/                    # Integrated order-level tables and join logs
│   ├── phase2/                    # Final cleaned dataset, cleaning log, and feature dictionary
│   ├── phase3/                    # Descriptive summary tables and 12 EDA figures
│   └── phase4/                    # Hypothesis test results, sensitivity checks, and inference plots
└── data/                          # Raw Kaggle CSV files
```

---

## 5. Key Operational Definitions

| Variable | Operational Definition |
|---|---|
| `is_late` | Binary indicator equal to `1` if the customer delivery calendar date is strictly after the promised delivery date (`order_estimated_delivery_date`). Because promised dates are stored at midnight (`00:00:00`), 1,269 orders delivered later during the promised calendar day are properly treated as on time (`is_late = 0`). |
| `wait_days` | Total elapsed waiting time from order purchase timestamp to customer delivery timestamp (in fractional days). |
| `delay_calendar_days` | Whole calendar days between the promised delivery date and the actual delivery date ($\ge 1$ for all late orders). |
| `review_before_delivery` | Binary indicator equal to `1` if `review_answer_timestamp` is earlier than `order_delivered_customer_date`. |
| `low_score` | Binary indicator equal to `1` if `review_score` is 1 or 2 stars. |
| `has_comment` / `has_any_text` | `has_comment = 1` if `review_comment_message` contains non-whitespace text; `has_any_text = 1` if either the message body or review title is non-empty. |

> **Note on floating-point precision for p-values:** In `outputs/phase4/primary_test_results.csv`, raw and Holm-adjusted p-values smaller than roughly $10^{-300}$ underflow standard 64-bit floats to `0.0`. Exact magnitudes are stored in `log10_p_raw` and `log10_p_holm` (e.g., `-538.4` corresponds to $p = 10^{-538.4}$) and are reported in scientific power-of-ten notation in the report.
