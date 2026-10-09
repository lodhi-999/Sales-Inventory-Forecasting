# Sales & Inventory Forecasting

End-to-end analysis of a stationery retailer's sales: data quality checks, rule-based cleaning, anomaly detection, a 6-month demand forecast for each product, and an inventory plan that shows when each product will run out of stock.

**Tools:** Python · Pandas · NumPy · Matplotlib · Seaborn · Jupyter Notebook

---

## Business questions

1. What is the quality of the raw data, and what anomalies does it contain?
2. Which products, regions and customers drive revenue?
3. How many units of each product will sell over the next 6 months?
4. Will current stock cover that demand, and how much needs to be reordered?

## Dataset

`Sales_Inventory_Dataset.xlsx` has two sheets:

| Sheet | Rows | Columns |
|---|---|---|
| `sales` | 238 orders (1 Jan – 26 Aug 2025) | order_id, order_date, product_id, product_name, quantity, sold_price, customer_id, region |
| `inventory` | 7 products | product_id, product_name, quantity_on_hand |

The data covers 7 products, 4 regions and 8 customers. `sold_price` is the total value of the order line (quantity × unit price), not a unit price.

## Pipeline

```
Load data  →  Data quality checks (EDA)  →  Rule-based cleaning  →  Anomaly detection
          →  Monthly aggregation  →  Model backtest  →  6-month forecast  →  Inventory plan
```

### 1. Data quality checks
- Missing values, duplicates, negative values
- Consistency of text categories and the product_id ↔ product_name mapping
- Unit price per product (each product sells at one fixed price)
- Date coverage, including the partial last month
- Repeating patterns in each column

### 2. Rule-based cleaning (no manual row edits)
Every fix is a rule written in code and marked with a flag column, so the notebook works the same on 238 rows or millions.

| Issue | Rule applied |
|---|---|
| 1 row missing product_id and product_name | Recovered from its unit price (₹3), which belongs to only one product: blue pen. Rows whose price is shared by several products would be labelled `Unknown`. |
| 3 rows missing customer_id | Labelled `UNKNOWN`. The customer sequence was tested and gives contradictory answers, so guessing would invent data. Customer is not used in the forecast. |
| August has only 26 of 31 days | Scaled to a full month before modelling, so the model doesn't read it as a drop in demand. |
| Months with no sales for a product | Filled with 0, since no sales is a real zero. |

### 3. Anomaly detection
Outliers are checked **within each product** (unit-price mismatches and the IQR rule on quantity), not across all products. A global check would flag every scale order just because scales cost more.

### 4. Forecasting
Each product has only 8 monthly data points, too few for Prophet, SARIMA or Holt-Winters to learn trend or seasonality. Three simple models were compared with a **walk-forward backtest**: each model predicts a month it hasn't seen, and accuracy is measured with **WAPE** (total absolute error ÷ total actual sales).

| Model | Backtest WAPE |
|---|---|
| **3-month moving average** | **35.7%** (selected) |
| Simple exponential smoothing | 37.7% |
| Naive (repeat last month) | 37.9% |

Units are forecast first, then converted to revenue using each product's fixed price.

### 5. Inventory plan
For each product: months of stock cover, the month stock runs out, and the units to reorder to cover 6 months of demand.

## Key findings

**Sales**
- **Scale** brings in the most revenue (30%) despite low volume, because of its higher price. **Pencil** sells the most units.
- **West** is the strongest region with 47% of revenue.
- **Sharpener** was launched in June 2025.
- **Scale** has had no sales since early June and only 5 units in stock. This is either a discontinued product or a stock-out hiding real demand, which needs a business check.

**Forecast (Sep 2025 – Feb 2026)**
- About **1,320 units** and **₹4,440 in revenue**, roughly ₹740 per month.

**Inventory**

| Product | In stock | Forecast per month | Months of cover | Runs out | Reorder for 6 months |
|---|---|---|---|---|---|
| Sharpener | 25 | 37.4 | 0.7 | Sep 2025 | 200 |
| Pencil | 85 | 58.3 | 1.5 | Oct 2025 | 265 |
| Scale | 5 | 2.4 | 2.1 | Oct 2025 | 9 |
| Red pen | 46 | 22.1 | 2.1 | Oct 2025 | 86 |
| Blue pen | 97 | 43.2 | 2.2 | Nov 2025 | 162 |
| Eraser | 105 | 36.8 | 2.9 | Nov 2025 | 116 |
| Black pen | 75 | 19.7 | 3.8 | Dec 2025 | 43 |

**Sharpener and pencil need reordering first.** Reorder quantities are the minimum needed and do not yet include safety stock or supplier lead time.

## Limitations
- Only 8 months of history, so seasonality can't be measured.
- The data looks synthetic: exactly one order per day, consecutive order IDs, and columns that repeat on fixed cycles. Results should be read as a demonstration of the method.

## Repository structure

```
├── Sales_Inventory_Forecasting_Pipeline.ipynb   # full analysis
├── Sales_Inventory_Dataset.xlsx                 # raw data
└── README.md
```
