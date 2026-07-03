# Customer Segmentation (RFM + KMeans)

End-to-end retail customer segmentation from POS receipt data: RFM feature engineering, KMeans on Frequency and Monetary, rule-based segment labels, rolling monthly history, and optional SQL Server export for Power BI.

Built as a first analytics project - exploratory comparison of raw vs transformed features, then a production-style engine you can rerun each month.

---

## What this project does

Retail teams need to know *who* their customers are, not just how much they sold. This notebook:

1. Aggregates receipt lines to customer-level **Recency, Frequency, Monetary (RFM)** metrics
2. Clusters **Frequency and Monetary** with 1-D KMeans (after comparing raw vs √ transform)
3. Bins **Recency** with fixed day thresholds (not clustered - easier to explain to stakeholders)
4. Maps R/F/M scores to named segments (Champion, Loyal, At Risk, Dormant, New, etc.)
5. Runs a **6-month rolling history** so you can track segment transitions over time
6. Exports five tables to SQL Server for dashboarding (optional)

---



## Business questions

1. **Which customers are high-value and still active?** (Champions, Loyal)
2. **Who is drifting away?** (At Risk, Dormant - driven by Recency bins)
3. **Who is new vs one-time vs returning after a long gap?** (uses First Purchase Date, not just last visit)
4. **Does sqrt-transforming skewed spend improve cluster separation?** (validated with Silhouette and Davies–Bouldin)
5. **How stable are segments month over month?** (rolling window + transition table)

---



## Key results (from the included run)


| Metric                             | Value                                                                             |
| ---------------------------------- | --------------------------------------------------------------------------------- |
| Customers segmented (latest month) | ~31,500 customer-month rows across 6 rolling months                               |
| Segment distribution (example)     | Low activity ~9.4k, One Time Visitor ~8k, Dormant ~3.5k, New ~3.3k, Champion ~228 |
| Raw F/M clustering                 | Silhouette ~0.78, DB ~0.57 (high Silhouette partly from scale dominance)          |
| Sqrt F/M clustering                | Silhouette ~0.53, DB ~0.68 — worse on paper, clearer separation in plots          |
| Rolling stability                  | Silhouette on F and M stays ~0.73–0.78 across six month-end runs                  |
| Negative recency rows              | 0 after pipeline checks                                                           |


The sqrt comparison is worth keeping: raw metrics look better numerically, but scatter plots show most customers crushed into one corner until spend is compressed.

---



## Segmentation approach


| Component           | Method                                                                        |
| ------------------- | ----------------------------------------------------------------------------- |
| Recency             | Manual bins: 0–15, 16–35, 36–60, 60+ days → scores 4 down to 1                |
| Frequency           | 1-D KMeans (4 clusters), labels ordered low→high by centroid                  |
| Monetary            | Same as Frequency                                                             |
| Final label         | Rule table on R/F/M scores + New (first purchase ≤30 days) + One Time Visitor |
| Rolling window      | 6 months of receipts per month-end cutoff                                     |
| Threshold stability | Optional 12-month reference fit → frozen F/M thresholds for the rolling year  |


**Segments:** Champion, Loyal, Loyal Low Spender, Loyal not frequent, Low activity, At Risk, Dormant, New, One Time Visitor, Other.

---



## Data privacy (anonymized)

**The dataset bundled or referenced by this repo is anonymized.** I verified the notebook outputs directly:


| Field         | What you see                               | Real PII?                                                                             |
| ------------- | ------------------------------------------ | ------------------------------------------------------------------------------------- |
| `Account No_` | Sequential IDs (`10000000`, `10000001`, …) | No — synthetic surrogate keys                                                         |
| `Phone No_`   | Same numeric pattern as account ID         | No — not a dialable phone number                                                      |
| `Receipt No_` | `REC0000000000000001` format               | No — sequential receipt tokens                                                        |
| `Store Name`  | `Store 1`, `Store 2`                       | No — generic labels                                                                   |
| `Item`        | Numeric codes (`1000000`, …)               | No — product names removed                                                            |
| `Name`        | Not present in sample outputs              | Column exists in schema for SQL export; populate only from your own anonymized source |


You cannot reverse-engineer a real customer name, phone number, or original receipt ID from the values shown in the notebook. Monetary amounts and dates are retained for realistic RFM behavior but are not tied to identifiable individuals in this export.

---

## How to run

**Requirements:** Python 3.10+, pandas, scikit-learn, matplotlib, seaborn, openpyxl (for Excel). SQL export additionally needs `sqlalchemy`, `pyodbc`, and a local SQL Server instance.

```bash
pip install -r requirements.txt
```

1. Add anonymized `CustomerData.xlsx` to `data/` (see column expectations in the notebook intro).
2. Open `RFM - Kmeans Segementation Advanced.ipynb`.
3. Run all cells through the segmentation engine.
4. Optional: configure SQL Server connection in the export cell and run cell 31.

---



## Limitations (honest)

- Analysis covers a **subset of stores/categories**, not the full chain -segment counts and metrics will shift at full scale.
- `k=4` for F and M is a practical default, not tuned exhaustively.
- Segment rules are **hand-crafted** for interpretability; they are not optimized for a single KPI.
- KMeans on raw F/M inflates Silhouette when Monetary dominates -always inspect plots, not just metrics.
- SQL export assumes a local `RFM` database and ODBC Driver 17; paths and credentials are environment-specific.
- Phone numbers in the export path are identifiers for CRM join-back in a private environment - in this public repo they are anonymized tokens only.

---



## Tools

Python, pandas, NumPy, scikit-learn, matplotlib, seaborn, SQLAlchemy, Power BI (downstream).