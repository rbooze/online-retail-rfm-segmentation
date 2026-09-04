# Online Retail II — Customer Segmentation with RFM Analysis

Customer segmentation on real UK e-commerce transaction data, using RFM (Recency, Frequency, Monetary) analysis to identify high-value customers, at-risk customers, and everything in between — then visualized in an interactive Power BI dashboard.

## Overview

This project analyzes ~1M invoice-level transactions from a UK-based online gift/homeware retailer to answer a core business question: **which customers matter most, and what should the business do about each group?**

The result is a six-segment customer classification (Champions, Loyal Customers, New Customers, Needs Attention, At Risk, Lost) built from real purchase behavior, with revenue concentration analysis showing exactly how much each segment contributes to the bottom line.

## Dataset

- **Source:** [Online Retail II Data Set from ML Repository](https://www.kaggle.com/datasets/mathchi/online-retail-ii-data-set-from-ml-repository)
- **Size:** ~1 million transaction line items
- **Time range:** December 2009 – December 2011
- **Granularity:** Invoice-level, one row per product line item
- **Fields:** Invoice, StockCode, Description, Quantity, InvoiceDate, Price (GBP), Customer ID, Country

## Data Cleaning

Real transactional data is messy, and this dataset in particular required several rounds of investigation rather than a single cleaning pass. Key steps:

- **Missing Customer IDs (~25% of rows):** Preserved separately rather than discarded — these are real transactions, just unattributable to a specific customer, so they were split into their own dataframe before being excluded from RFM.
- **Cancellations:** Rows with an Invoice number prefixed "C" were identified and removed, since they represent reversed orders rather than completed purchases.
- **Orphaned cancelled-originals:** Two extreme outlier transactions (one customer ordering 80,995 units of a single item, another 74,215) turned out to have matching cancellations that weren't caught by the initial "C"-prefix filter. Investigation revealed that cancellation invoice numbers in this dataset are *not* derived from their original invoice number — they're assigned independently — so a digit-matching approach failed silently. These were identified manually and removed once confirmed.
- **Negative quantities:** Checked separately from cancellations; in this dataset, negative-quantity rows were almost entirely already captured by the cancellation filter.
- **Type mismatches:** Customer ID and Invoice both loaded with inconsistent types across cleaning steps (e.g., `16446.0` vs `16446` as string vs float), which silently broke several `isin()` and merge operations before being caught and standardized.

This last issue — join-key type/formatting mismatches causing silent failures — resurfaced again later in Power BI (a `Customer ID` relationship that looked correctly configured but matched zero rows until whitespace was trimmed from the key column). It's the throughline data-quality lesson of this project: **type and formatting consistency on join keys matters as much in a BI tool as it does in pandas.**

## Methodology

**RFM Feature Engineering**
For each customer, three metrics were calculated as of a snapshot date (one day after the dataset's final transaction):

- **Recency** — days since the customer's most recent purchase
- **Frequency** — number of distinct invoices (purchases)
- **Monetary** — total amount spent

**Segmentation: Manual Quintile Scoring**
Each metric was split into five quintiles using `pd.qcut`, with Recency scored in reverse (lower days = higher score, since recency is "better" when smaller). The three scores were combined into segment labels using a rule-based mapping:

| Segment | Definition |
|---|---|
| Champions | Recent, frequent, high spenders |
| Loyal Customers | Reliably recent and frequent |
| New Customers | Very recent, but low frequency so far |
| Needs Attention | Mixed signals, doesn't cleanly fit other groups |
| At Risk | Previously frequent, now going quiet |
| Lost | Long inactive, low frequency |

## Key Findings

- Customer counts were fairly evenly spread across segments, with **Lost (1,512)** and **Champions (1,283)** as the two largest groups — a typical "barbell" shape for retail data (a strong loyal core, plus a long tail of one-time buyers).
- **At Risk customers had the second-highest average Monetary value of any segment** (higher than Loyal Customers) — meaning this group isn't just "used to buy often," they were high-value spenders, making them a higher-priority win-back target than their segment name alone suggests.
- Revenue is concentrated: a disproportionate share of total revenue comes from the Champions segment relative to their share of the customer base — the classic 80/20 pattern that makes RFM segmentation actionable rather than just descriptive.
- **Revenue Concentration:**
	| Segment | Total Revenue % |
	|---|---|
	| Champions | 69.1% |
	| Loyal Customers | 15.5% |
	| New Customers | 1.4% |
	| Needs Attention | 4.1% |
	| At Risk | 6.2% |
	| Lost |3.7% |

## Recommendations

- **Champions** — protect this segment with loyalty perks; they generate outsized revenue relative to their numbers.
- **At Risk** — prioritize win-back campaigns here specifically because of their high historical spend, not just their past frequency.
- **Loyal Customers** — upsell opportunity; already engaged, room to increase spend per order.
- **New Customers** — nurture toward repeat purchases with onboarding-style follow-up.
- **Lost** — low-cost reactivation only, or deprioritize given low historical value.

## Visuals

- Customer count by segment (bar chart)
- Revenue by segment (bar chart)
- Recency vs. Frequency scatter plot, colored by segment
- Segment profiling table (mean/median Recency, Frequency, Monetary per segment)
- Interactive Power BI dashboard with segment slicers and a revenue-over-time trend line

## Tools Used

- **Python** — pandas, matplotlib
- **Jupyter Notebooks** — split into `01_data_cleaning.ipynb` and `02_rfm_segmentation.ipynb`
- **Power BI** — interactive dashboard for exploring segments and revenue trends

## Project Structure

```
├── data
│   └── cancelled_orders.csv
│   └── cleaned_online_retail.csv	# Cleaned transaction-level output
│   └── datacard.txt
│   └── duplicate_rows.csv
│   └── invalid_stockcode.csv
│   └── missing_customerid.csv
│   └── negative_qty.csv
│   └── online_retail_II.xlsx
│   └── orphaned.csv
│   └── price_is_zero.csv
│   └── recency.csv
│   └── rfm_segments.csv			# Final customer-level RFM table with segments
├── notebooks
│   └── 01_data_cleaning.ipynb	  	# Data loading, cleaning, and quality checks
│   └── 02_rfm_segmentation.ipynb	# RFM feature engineering, scoring, visualization
├── power-bi
│   └── retail_rfm_segmentation.ipynb
└── README.md
```

## Possible Extensions

- K-means clustering as an alternative segmentation method, compared against the manual quintile approach
- Cohort analysis by acquisition month
- Country-level segment breakdown
