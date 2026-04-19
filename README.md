# 🛍️ Customer Shopping Behavior — End-to-End Data Analysis Project

> A complete data analysis pipeline — from raw customer data to interactive Power BI dashboard — uncovering purchasing patterns, subscription behavior, and revenue drivers across customer segments.

---

## 📌 Overview

This project delivers a **full-stack data analysis workflow** on real customer shopping data. It covers data ingestion in Python, exploratory analysis, structured cleaning, relational database querying, and an executive-ready Power BI dashboard — demonstrating end-to-end analyst proficiency in a single, reproducible project.

**Business Goal:** Analyze customer shopping behavior to identify revenue contributors, segment customers by loyalty, evaluate discount effectiveness, and surface actionable insights for marketing and retention strategy.

---

## 📂 Dataset

| Attribute        | Details                                                   |
|------------------|-----------------------------------------------------------|
| **File**         | `customer_shopping_behavior.csv`                          |
| **Records**      | 3,900 rows × 18 columns                                   |
| **Source**       | Retail customer transaction data                          |
| **Key Fields**   | Customer ID, Age, Gender, Item Purchased, Category, Purchase Amount (USD), Location, Season, Review Rating, Subscription Status, Shipping Type, Discount Applied, Previous Purchases, Payment Method, Frequency of Purchases |

> 📁 Raw data stored in `/data/raw/`. Cleaned data exported to `/data/cleaned/` and loaded into PostgreSQL.

---

## 🛠️ Tools & Technologies

| Category           | Tool / Technology                                  |
|--------------------|----------------------------------------------------|
| **Language**       | Python 3.x                                         |
| **Libraries**      | Pandas, NumPy, Matplotlib, Seaborn, SQLAlchemy     |
| **Database**       | PostgreSQL (`customer_behavior` database)          |
| **Visualization**  | Power BI Desktop                                   |
| **Presentation**   | Gamma.app                                          |
| **Environment**    | Jupyter Notebook                                   |
| **Version Control**| Git & GitHub                                       |

---

## 🔄 Project Steps

### 1. 📥 Loaded the Dataset
- Ingested raw CSV into a Pandas DataFrame using `pd.read_csv()`
- Inspected schema, column types, and value distributions across 3,900 customer records and 18 attributes using `df.info()` and `df.head()`

### 2. 🔍 Performed Exploratory Data Analysis (EDA)
- Computed descriptive statistics across all numeric and categorical fields using `df.describe(include='all')`
- Audited null values per column with `df.isnull().sum()` to identify data quality issues
- Profiled distributions of purchase amount, review ratings, gender, and season to surface early patterns

### 3. 🧹 Cleaned & Engineered the Data
- **Imputed** missing `Review Rating` values using **category-level medians** to prevent bias and preserve segment integrity
- **Standardized** all column names to `snake_case` for consistency and error-free querying
- **Renamed** `purchase_amount_(usd)` → `purchase_amount` for cleaner code
- **Dropped** the redundant `promo_code_used` column after confirming 100% overlap with `discount_applied`
- **Engineered** two new features:
  - `age_group` — segmented customers into `Young Adult`, `Adult`, `Middle-aged`, `Senior` using quartile-based binning (`pd.qcut`)
  - `purchase_frequency_days` — converted text frequency labels (e.g., *Weekly*, *Fortnightly*, *Quarterly*) to numeric day intervals for quantitative analysis

### 4. 🗄️ Loaded Data into PostgreSQL & Ran SQL Queries
- Connected Python to a local **PostgreSQL** instance (`customer_behavior` DB) via **SQLAlchemy** + `psycopg2`
- Loaded the cleaned DataFrame directly into the `customer` table
- Executed **10 analytical SQL queries** covering:

| # | Query Focus |
|---|-------------|
| Q1 | Total revenue split by gender |
| Q2 | Discount users who still exceeded average spend |
| Q3 | Top 5 products by average review rating |
| Q4 | Average purchase amount — Standard vs. Express shipping |
| Q5 | Subscriber vs. non-subscriber spend and revenue comparison |
| Q6 | Top 5 products by discount usage rate |
| Q7 | Customer segmentation — New / Returning / Loyal (via CTE) |
| Q8 | Top 3 products per category using `ROW_NUMBER()` window function |
| Q9 | Subscription rate among repeat buyers (5+ purchases) |
| Q10 | Revenue contribution breakdown by age group |

```sql
-- Q7: Segment customers by purchase history
WITH customer_type AS (
  SELECT customer_id, previous_purchases,
    CASE
      WHEN previous_purchases = 1              THEN 'New'
      WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
      ELSE 'Loyal'
    END AS customer_segment
  FROM customer
)
SELECT customer_segment, COUNT(*) AS number_of_customers
FROM customer_type
GROUP BY customer_segment;
```

```sql
-- Q8: Top 3 most purchased products per category
WITH item_counts AS (
  SELECT category, item_purchased,
    COUNT(customer_id) AS total_orders,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
  FROM customer
  GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
```

### 5. 📊 Built the Power BI Dashboard
- Connected **Power BI Desktop** directly to the PostgreSQL `customer_behavior` database
- Designed `CUSTOMER_BEHAVIOR_DASHBOARD.pbix` with interactive visuals:
  - Revenue breakdown by gender and age group
  - Subscriber vs. non-subscriber spend comparison
  - Top-selling products and categories
  - Discount impact analysis across customer segments
  - Shipping type performance and seasonal purchase trends
- Applied DAX measures to compute dynamic KPIs across filter selections

### 6. 📝 Created the Analysis Report
- Documented methodology, key findings, data limitations, and business recommendations
- Structured around an executive summary, EDA highlights, SQL insights, and actionable next steps
- Exported as a shareable PDF for stakeholder distribution

### 7. 🎨 Designed the Presentation with Gamma
- Crafted a concise, visually polished slide deck on [Gamma.app](https://gamma.app)
- Translated SQL and dashboard findings into business-ready narratives for non-technical audiences
- Embedded KPI callouts and visual highlights directly into slides for maximum clarity

---

## 📈 Dashboard Preview

> 📌 *Open `CUSTOMER_BEHAVIOR_DASHBOARD.pbix` in Power BI Desktop to explore the full interactive report.*

![Dashboard Preview](assets/dashboard_preview.png)

**Dashboard highlights:**
- KPI cards — total revenue, average spend, subscription rate
- Revenue by gender, age group, and season with slicer controls
- Customer segmentation view (New / Returning / Loyal)
- Product performance ranked by category and review rating
- Discount usage vs. revenue impact analysis

---

## 📊 Key Results

| Metric | Finding |
|--------|---------|
| **Dataset Size** | 3,900 customers × 18 features |
| **New Features Engineered** | 2 (`age_group`, `purchase_frequency_days`) |
| **Redundant Column Removed** | `promo_code_used` — 100% overlap with `discount_applied` |
| **Missing Data Resolved** | `Review Rating` imputed with category-level medians |
| **SQL Queries Executed** | 10 — using CTEs, window functions, subqueries, and aggregations |
| **Customer Segments Identified** | New, Returning, Loyal — derived from `previous_purchases` |
| **Repeat Buyer Insight** | Analyzed whether customers with 5+ purchases are also subscribers |

---

## ▶️ How to Run

### Prerequisites

- Python 3.8+
- PostgreSQL (running locally or via Docker)
- Power BI Desktop
- Git

### Setup Instructions

```bash
# 1. Clone the repository
git clone https://github.com/your-username/customer-behavior-analysis.git
cd customer-behavior-analysis

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # Mac/Linux
venv\Scripts\activate           # Windows

# 3. Install dependencies
pip install -r requirements.txt
```

### Configure Database Connection

Update the credentials in the notebook or create a `.env` file:

```python
username = "postgres"
password = "your_password"
host     = "localhost"
port     = "5432"
database = "customer_behavior"
```

### Run the Notebook

```bash
jupyter notebook notebooks/customer_shopping_behavior_analysis.ipynb
```

The notebook will automatically load, clean, engineer features, and push data to the PostgreSQL `customer` table.

### Run SQL Queries

```bash
psql -U postgres -d customer_behavior -f customer.sql
```

### Open the Dashboard

1. Launch **Power BI Desktop**
2. Open `CUSTOMER_BEHAVIOR_DASHBOARD.pbix`
3. Go to **Home → Transform Data → Data Source Settings**
4. Update PostgreSQL server and database credentials
5. Click **Refresh** to load live data

---

## 📁 Repository Structure

```
📦 customer-behavior-analysis/
├── 📂 data/
│   ├── raw/
│   │   └── customer_shopping_behavior.csv
│   └── cleaned/                              # Generated after notebook run
├── 📂 notebooks/
│   └── customer_shopping_behavior_analysis.ipynb
├── 📂 sql/
│   └── customer.sql                          # All 10 analytical queries
├── 📂 dashboard/
│   └── CUSTOMER_BEHAVIOR_DASHBOARD.pbix
├── 📂 report/
│   └── customer_behavior_report.pdf
├── 📂 presentation/
│   └── gamma_slides_link.txt
├── 📂 assets/
│   └── dashboard_preview.png
├── requirements.txt
├── .env.example
└── README.md
```

---

## 🤝 Connect with Me

**Shivangi Parihar**

📧 Email: [pariharshivangi0807@gmail.com](mailto:pariharshivangi0807@gmail.com)

🔗 LinkedIn: [View Profile](https://www.linkedin.com/in/shivangipariharmits)

💼 Portfolio: [Visit My Portfolio](https://shivangi08-ai.github.io)

💻 GitHub: [My GitHub](https://github.com/shivangi08-ai)
---

*Built with Python · PostgreSQL · Power BI · Gamma — turning 3,900 customer records into business-ready decisions.*
