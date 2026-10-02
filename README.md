# Customer Behaviour Analysis

Analysis of a 3,900-customer retail dataset to understand what differentiates high-value
customers, whether the subscription programme earns its discount, and where revenue is
concentrated. Cleaning and feature engineering in Python, business questions answered in
PostgreSQL, results presented in a Power BI dashboard.

## Dataset

[Customer Shopping Trends](https://www.kaggle.com/datasets/iamsouravbanerjee/customer-shopping-trends-dataset)
(Kaggle), included here as `customer_shopping_behavior.csv`.

- 3,900 rows, 18 columns
- **One row per customer, not per transaction.** Purchase history is summarised in a
  `previous_purchases` count, so this is a customer table rather than a transaction log.
- **Synthetic data.** The figures below are real results from this file, but they describe a
  generated dataset, not a real business.
- No purchase dates, so no trend or time-series analysis is possible.
- 37 missing values, all in `Review Rating`.

## Contents

| File | What it is |
|---|---|
| `customer.ipynb` | Loading, EDA, cleaning, feature engineering, load to PostgreSQL |
| `customer_behavior.sql` | Ten business questions answered in PostgreSQL |
| `Analysis_dashboard.png` | Power BI dashboard screenshot |
| `Customer Shopping Behavior Analysis.pdf` | Written report |
| `Customer-Shopping-Behavior-Analysis.pptx` | Stakeholder presentation |
| `customer_shopping_behavior.csv` | Source data |

## Tools & technologies

| Tool | Purpose |
|---|---|
| Python | Data loading, exploration and cleaning |
| pandas | Data manipulation, imputation and feature engineering |
| SQLAlchemy / psycopg | Loading the cleaned table into PostgreSQL |
| PostgreSQL | Business-question analysis (CTEs, window functions, conditional aggregation) |
| Power BI | Interactive dashboard |
| Jupyter Notebook | Analysis environment |
| Gamma | Presentation |

## Project workflow

**1. Loading and exploration**
Imported the CSV with pandas; checked structure, dtypes and summary statistics; identified
the 37 missing `Review Rating` values as the only data quality gap.

**2. Cleaning and feature engineering**
Imputed the missing ratings using the median of each product category rather than a global
median. Standardised column names to snake_case. Derived `age_group` by splitting age into
quartiles and `purchase_frequency_days` by mapping the categorical frequency labels to day
counts. Checked `promo_code_used` against `discount_applied`, found them identical for every
row, and dropped the duplicate column.

**3. Load to PostgreSQL**
Wrote the cleaned frame to a `customer` table via SQLAlchemy so the analysis could be done
in SQL rather than in pandas.

**4. SQL analysis**
Answered ten business questions in `customer_behavior.sql`, covering revenue by gender and
age group, discount behaviour, product ratings, shipping comparison, subscription value,
loyalty segmentation and top products per category.

**5. Dashboard and reporting**
Built the Power BI dashboard, then wrote up the results as a report and a presentation.

## Findings

**The subscription programme is a blanket discount that buys nothing.**
Every subscriber has a discount applied, against 21.9% of non-subscribers. Despite that,
subscribers spend fractionally *less* per order ($59.49 vs $59.87) and account for 26.9% of
revenue from 27.0% of customers. The discount is being given away rather than earning
incremental spend. Either the benefit should be re-priced, or it should be tied to a
behaviour the business actually wants.

**Average order value barely moves across any segment.**
Every cut of the data — gender, subscription status, shipping type, age group, loyalty
segment — lands between $57 and $61. There is no segment to upsell. Revenue growth in this
dataset has to come from purchase frequency and customer count, not from basket size.

**Revenue is concentrated in two categories.**
Clothing contributes $104,264 (44.7%) and Accessories $74,200 (31.8%), so 76.5% of the
$233,081 total sits in two of the four categories. Footwear is 15.5% and Outerwear 7.9%.

**Half of discount users did not need the discount.**
839 of the 1,677 customers who used a discount still spent above the $59.76 average, at a
mean of $79.79 and $66,942 of revenue between them. That is a cohort worth testing a
discount reduction on.

**Repeat buyers do subscribe more, but modestly.**
27.6% of customers with more than five previous purchases subscribe, against 22.4% of the
rest. The direction is right, the gap is small.

**Age is not a useful split here.**
Revenue by age quartile ranges only from $55,763 to $62,143. Reporting an age group as
"highest revenue" would overstate a 10% spread across quartiles of near-identical size.

## Dashboard

![Customer Behaviour Dashboard](Analysis_dashboard.png)

KPI cards for customer count, average purchase amount and average review rating, with
revenue and order volume broken out by category and age group, a subscription split, and
slicers for subscription status, gender, category and shipping type.

## Limitations

- The data is synthetic. Findings demonstrate method, not real market behaviour.
- One row per customer means `purchase_amount` is a single observed order, not lifetime
  value. "Revenue" throughout is the sum of those single amounts.
- `previous_purchases` has no accompanying dates or amounts, so loyalty segments are built
  on a bare count.
- Segment boundaries (New / Returning / Loyal) are a judgement call, not a property of the
  data. The thresholds used are stated in `customer_behavior.sql` and on the segmentation
  slide; different thresholds give materially different percentages.
- Review ratings are imputed for 37 customers, which slightly compresses variance in the
  rating analysis.
- All results are associations within a static snapshot. Nothing here establishes cause.

## Running it

```bash
git clone https://github.com/Sohini-dotcom/Customer_behavior_analysis.git
cd Customer_behavior_analysis
pip install pandas numpy sqlalchemy "psycopg[binary]" jupyter
```

1. Create a PostgreSQL database named `customer_behavior`.
2. Open `customer.ipynb`, update the connection string in the SQLAlchemy cell, and run all
   cells. This cleans the data and writes the `customer` table.
3. Run the queries in `customer_behavior.sql` against that database.

---

**Author:** Sohini Chandra
