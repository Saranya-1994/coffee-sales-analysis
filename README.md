# ☕ Coffee Sales Analysis — SQL & Power BI

> **End-to-end analysis of vending machine coffee sales (2024–2025)**
> Data cleaning, SQL querying, and interactive Power BI dashboard built on real transaction data.

---

## 📋 Project Snapshot

| Field | Details |
|-------|---------|
| **Domain** | Retail / Vending Machine Sales |
| **Tools** | PostgreSQL, Valentina Studio, Power BI |
| **Currency** | US Dollar (USD) |
| **Data Period** | March 2024 – March 2025 (ongoing) |
| **Total Records** | 3,898 transactions |
| **Status** | ✅ Completed |

---

## 💡 What This Project Does

This project consolidates, cleans, and analyzes **two years of coffee vending machine sales data** (2024 & 2025) using SQL to uncover business performance trends, customer purchasing habits, and product-level insights — visualized through an interactive Power BI dashboard.

---

## 📂 Dataset Structure

Two CSV files were imported into PostgreSQL and merged into a unified `coffee_sales` table.

| Column | Type | Description |
|--------|------|-------------|
| `date` | DATE | Date of transaction |
| `datetime` | TIMESTAMP | Full timestamp of purchase |
| `cash_type` | VARCHAR | Payment method: cash or card |
| `card` | VARCHAR | Anonymous card number (2024 only) |
| `money` | NUMERIC | Amount in Ukrainian hryvnias (USD) |
| `coffee_name` | VARCHAR | Name of coffee product |

---

## 🎯 Objectives

- ✅ Combine 2024 & 2025 datasets into a unified PostgreSQL database
- ✅ Analyze total revenue, monthly growth trends, and YoY performance
- ✅ Identify top-performing coffee products by revenue and order count
- ✅ Understand customer payment preferences (cash vs. card)
- ✅ Visualize findings through an interactive Power BI dashboard

---

## 🔍 SQL Analysis Workflow

### 1. Combine Tables
```sql
CREATE TABLE coffee_sales AS
SELECT date, datetime, cash_type, money::NUMERIC, coffee_name FROM index_1
UNION ALL
SELECT date, datetime, cash_type, money::NUMERIC, coffee_name FROM index_2;
```

### 2. Convert Data Types
```sql
ALTER TABLE index_1 ALTER COLUMN money TYPE NUMERIC USING money::NUMERIC;
ALTER TABLE index_2 ALTER COLUMN money TYPE NUMERIC USING money::NUMERIC;
```

### 3. Total Revenue by Year
```sql
SELECT SUM(money) AS total_sales_2024 FROM public.index_1;
SELECT SUM(money) AS total_sales_2025 FROM public.index_2;
```

### 4. YoY Growth Difference
```sql
WITH totals AS (
  SELECT '2024' AS year, SUM(money) AS sales FROM index_1 UNION ALL
  SELECT '2025', SUM(money) FROM index_2
)
SELECT
  MAX(CASE WHEN year='2024' THEN sales END) AS sales_2024,
  MAX(CASE WHEN year='2025' THEN sales END) AS sales_2025,
  MAX(CASE WHEN year='2025' THEN sales END) -
  MAX(CASE WHEN year='2024' THEN sales END) AS growth_difference
FROM totals;
```

### 5. Top 10 Coffee by Revenue
```sql
SELECT coffee_name, SUM(money) AS revenue
FROM coffee_sales
GROUP BY coffee_name
ORDER BY revenue DESC LIMIT 10;
```

### 6. Payment Method Analysis
```sql
SELECT COALESCE(a.cash_type, b.cash_type) AS cash_type,
       COALESCE(a.total_2024, 0) AS count_2024,
       COALESCE(b.total_2025, 0) AS count_2025
FROM (SELECT cash_type, COUNT(*) AS total_2024 FROM index_1 GROUP BY cash_type) a
FULL JOIN
     (SELECT cash_type, COUNT(*) AS total_2025 FROM index_2 GROUP BY cash_type) b
ON a.cash_type = b.cash_type;
```

### 7. Monthly Revenue Comparison
```sql
SELECT COALESCE(a.month, b.month) AS MONTH,
       COALESCE(a.total_2024, 0) AS sales_2024,
       COALESCE(b.total_2025, 0) AS sales_2025
FROM (SELECT TO_CHAR(TO_DATE(date,'YYYY-MM-DD'),'YYYY-MM') AS MONTH,
             SUM(money::NUMERIC) AS total_2024 FROM index_1 GROUP BY MONTH) a
FULL JOIN
     (SELECT TO_CHAR(TO_DATE(date,'YYYY-MM-DD'),'YYYY-MM') AS MONTH,
             SUM(money::NUMERIC) AS total_2025 FROM index_2 GROUP BY MONTH) b
ON a.month = b.month ORDER BY MONTH;
```

### 8. Revenue Summary
```sql
SELECT COUNT(*)       AS total_orders,
       SUM(money)     AS total_revenue,
       MIN(money)     AS min_order,
       MAX(money)     AS max_order,
       AVG(money)     AS avg_order
FROM coffee_sales;
```

---

## 📊 Key Findings

| Metric | Value |
|--------|-------|
| **Total Revenue** | 2,935.72 USD |
| **Total Orders** | 3,898 |
| **Average Order Value** | 0.75 USD |
| **Top Product by Revenue** | Latte (687.80 USD) |
| **Top Product by Orders** | Americano with Milk (824 orders) |
| **Peak Month** | October 2024 (333.39 USD) |
| **Card Payment Share** | 95.66% |
| **Cash Payment Share** | 4.34% |

### 🏆 Top 5 Coffee Products

| Rank | Product | Revenue (USD) | Orders |
|------|---------|---------------|--------|
| 1 | Latte | 687.80 | 806 |
| 2 | Americano with Milk | 606.46 | 824 |
| 3 | Cappuccino | 444.34 | 517 |
| 4 | Americano | 370.49 | 593 |
| 5 | Hot Chocolate | 244.14 | 282 |

---

## 📈 Power BI Dashboard

The dashboard includes:

- 📊 **KPI Cards** — Total Sales, Order Count, Avg Order Value, Sales by Year
- 📉 **Sales Trend Line Chart** — Monthly revenue across all periods
- 📊 **Products Bar Chart** — Revenue ranking across all coffee types
- 🏅 **Popular Items Chart** — Top products by number of orders
- 📅 **Monthly by Coffee Type** — Grouped bar chart for top 5 products
- 🥧 **Payment Donut Chart** — Card vs cash breakdown
- 🔽 **Slicers** — Filter by year, product, and payment type

---

## 💡 Recommendations

1. **Stock Optimization** — Prioritize Latte, Americano with Milk, and Cappuccino; top 3 account for >50% of revenue
2. **Menu Rationalization** — Remove low-volume specialty items (Irish whiskey, 21 orders) to free machine space
3. **Card Infrastructure** — With 95.66% card usage, ensure card reader reliability and consider contactless upgrades
4. **Replicate October Peak** — October 2024 was the strongest month; investigate what drove it and replicate in 2025
5. **Continue Monitoring** — Re-run monthly as 2025 data grows for a true YoY comparison
6. **Price Testing** — Americano has the lowest avg order value (~26 USD); test modest price increase to lift blended average

---

## 🗂️ Repository Structure

```
📦 coffee-sales-analysis
 ┣ 📄 coffee_sales_queries.sql          ← All SQL queries used in analysis
 ┣ 📄 Coffee_Sales_Analysis_Report.pdf  ← Full analysis report (10 pages)
 ┣ 📄 Coffee_Sales_Dashboard.pbix       ← Power BI dashboard file
 ┗ 📄 README.md                         ← This file
```

---

## 🛠️ Tools & Technologies

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat&logo=postgresql&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![SQL](https://img.shields.io/badge/SQL-CC2927?style=flat&logo=microsoftsqlserver&logoColor=white)

| Tool | Purpose |
|------|---------|
| **PostgreSQL** | Database, data cleaning, all SQL queries |
| **Valentina Studio** | CSV import and database management UI |
| **Power BI Desktop** | Interactive dashboard and visualization |

---

*Dataset: Open vending machine coffee sales data · 3,898 transactions · 18 product types · March 2024 – March 2025*

---


📷 **Dashboard Preview**

<img width="1235" height="699" alt="Image" src="https://github.com/user-attachments/assets/d84df2d9-1163-47a4-966f-f8365a381b27" />







