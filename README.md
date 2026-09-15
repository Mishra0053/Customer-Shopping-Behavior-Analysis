Absolutely — here is the **entire README as one single copy-paste block**. Just copy everything inside it into your GitHub `README.md`.

````markdown
# 🛍️ Customer Shopping Behavior Analysis

An end-to-end data analytics project that analyzes customer shopping behavior using **Python, Pandas, PostgreSQL, Neon, SQL, and Power BI**. The project transforms raw customer data into meaningful business insights through data cleaning, feature engineering, database management, SQL analysis, customer segmentation, and interactive visualization.

---

## 📌 Project Overview

Understanding customer shopping behavior is important for businesses because customer data can reveal valuable patterns related to purchasing habits, product preferences, customer loyalty, discounts, subscriptions, shipping methods, and revenue.

This project analyzes a dataset containing **3,900 customer records** and answers important business questions such as:

- Which gender generates more revenue?
- Which products have the highest average ratings?
- Do Express-shipping customers spend more than Standard-shipping customers?
- Do subscribers spend more than non-subscribers?
- Which products receive discounts most frequently?
- How many customers are New, Returning, or Loyal?
- Which products are most purchased within each category?
- Are repeat buyers more likely to subscribe?
- Which age group contributes the most revenue?

The complete project follows this workflow:

```text
Raw CSV Dataset
       ↓
Python + Pandas
       ↓
Data Cleaning & Preprocessing
       ↓
Feature Engineering
       ↓
PostgreSQL / Neon
       ↓
SQL Business Analysis
       ↓
Power BI Visualization
       ↓
Business Insights & Recommendations
````

---

## 🎯 Objectives

The main objectives of this project are:

1. Analyze customer shopping behavior.
2. Clean and preprocess raw customer data.
3. Handle missing values appropriately.
4. Standardize dataset columns and formats.
5. Create useful analytical features.
6. Store processed data in PostgreSQL.
7. Integrate PostgreSQL with Neon.
8. Perform business-oriented SQL analysis.
9. Identify customer purchasing patterns.
10. Analyze product ratings and popularity.
11. Analyze discount usage.
12. Compare subscribers and non-subscribers.
13. Segment customers based on previous purchases.
14. Analyze repeat customer behavior.
15. Study revenue contribution by age group.
16. Build an interactive Power BI dashboard.
17. Generate actionable business recommendations.

---

## 📂 Dataset

**Dataset:** `customer_shopping_behavior.csv`

**Number of Records:** 3,900

The dataset contains customer-level shopping information including:

* Customer ID
* Age
* Gender
* Item Purchased
* Category
* Purchase Amount
* Location
* Size
* Color
* Season
* Review Rating
* Subscription Status
* Shipping Type
* Discount Applied
* Previous Purchases
* Payment Method
* Frequency of Purchases

Additional features were created during preprocessing:

* `age_group`
* `purchase_frequency_days`

---

## 🛠️ Technologies Used

### Programming & Data Analysis

* Python
* Pandas

### Database

* PostgreSQL
* Neon PostgreSQL

### Query Language

* SQL

### Visualization & Business Intelligence

* Microsoft Power BI
* DAX

### Development Environment

* Jupyter Notebook
* Google Colab

---

# 🔄 Data Processing

## 1. Loading the Dataset

The CSV dataset was loaded using Pandas.

```python
import pandas as pd

df = pd.read_csv('customer_shopping_behavior.csv')

df.head()
```

The dataset was initially inspected using:

```python
df.info()
df.describe(include='all')
df.isnull().sum()
```

These commands were used to understand the dataset structure, data types, statistical information, and missing values.

---

## 2. Handling Missing Values

Missing values in the `Review Rating` column were handled using category-wise median imputation.

```python
df['Review Rating'] = df.groupby('Category')['Review Rating'].transform(
    lambda x: x.fillna(x.median())
)
```

### Why Median Imputation?

Median was selected because it is less affected by extreme values and is therefore more robust when dealing with potential outliers.

---

## 3. Standardizing Column Names

Column names were converted to lowercase and spaces were replaced with underscores.

```python
df.columns = df.columns.str.lower()
df.columns = df.columns.str.replace(' ','_')
```

The purchase amount column was renamed:

```python
df = df.rename(columns={
    'purchase_amount_(usd)': 'purchase_amount'
})
```

This made the column names easier to use in Python and SQL.

---

# 🧮 Feature Engineering

## Age Group

Customers were divided into four age groups using quartile-based segmentation.

```python
labels = ['Young Adult','Adult','Middle-aged','Senior']

df['age_group'] = pd.qcut(
    df['age'],
    q=4,
    labels=labels
)
```

The resulting groups were:

* Young Adult
* Adult
* Middle-aged
* Senior

---

## Purchase Frequency in Days

The categorical purchase frequency was converted into numerical day values.

```python
frequency_mapping = {
    'Fortnightly': 14,
    'Weekly': 7,
    'Monthly': 30,
    'Quarterly': 90,
    'Bi-weekly': 14,
    'Annually': 365,
    'Every 3 months': 90
}

df['purchase_frequency_days'] = (
    df['frequency_of_purchases'].map(frequency_mapping)
)
```

This makes purchase frequency easier to analyze quantitatively.

---

# 🧹 Redundant Data Removal

The relationship between `discount_applied` and `promo_code_used` was checked before removing redundant information.

```python
(df['discount_applied'] == df['promo_code_used']).all()
```

Since both columns were found to contain equivalent information, `promo_code_used` was removed.

```python
df = df.drop('promo_code_used', axis=1)
```

This helped reduce unnecessary duplication in the dataset.

---

# 🗄️ PostgreSQL & Neon Integration

After preprocessing, the cleaned dataset was uploaded to a PostgreSQL database hosted using **Neon**.

The final database table was:

```text
customer
```

SQLAlchemy was used to connect Python with PostgreSQL.

```python
from sqlalchemy import create_engine

DATABASE_URL = "postgresql://neondb_owner:[PASSWORD]@[NEON_HOST]/neondb?sslmode=require"

engine = create_engine(DATABASE_URL)

print("Connected successfully!")
```

The database connection was verified using:

```python
from sqlalchemy import text

with engine.connect() as conn:
    result = conn.execute(text("SELECT version();"))
    print(result.fetchone())
```

The processed DataFrame was uploaded using:

```python
table_name = "customer"

df.to_sql(
    table_name,
    engine,
    if_exists="replace",
    index=False
)

print(f"Data successfully loaded into table '{table_name}'.")
```

> **Security:** Database passwords and credentials should never be committed to GitHub. Use environment variables or a `.env` file instead.

---

# 📊 SQL Business Analysis

The PostgreSQL database was used to answer 10 important business questions.

---

## Q1. Total Revenue by Gender

### Question

**What is the total revenue generated by male vs. female customers?**

### SQL

```sql
SELECT 
    gender,
    SUM(purchase_amount) AS revenue
FROM customer
GROUP BY gender;
```

### Result

| Gender | Revenue |
| ------ | ------: |
| Male   | 157,890 |
| Female |  75,191 |

### Insight

Male customers generated higher total revenue than female customers in the dataset.

---

# 💰 Q2. Discount Users Who Spent Above Average

### Question

**Which customers used a discount but still spent more than the average purchase amount?**

### SQL

```sql
SELECT 
    customer_id,
    purchase_amount
FROM customer
WHERE discount_applied = 'Yes'
AND purchase_amount >= (
    SELECT AVG(purchase_amount)
    FROM customer
);
```

The dataset contains:

| Discount Applied | Customers |
| ---------------- | --------: |
| No               |     2,223 |
| Yes              |     1,677 |

The query identifies individual customers who used a discount and made a purchase at or above the overall average purchase amount.

---

# ⭐ Q3. Top 5 Products by Average Review Rating

### SQL

```sql
SELECT 
    item_purchased,
    ROUND(AVG(review_rating)::numeric, 2) AS "Average Product Rating"
FROM customer
GROUP BY item_purchased
ORDER BY AVG(review_rating) DESC
LIMIT 5;
```

### Result

| Rank | Product | Average Rating |
| ---: | ------- | -------------: |
|    1 | Gloves  |           3.86 |
|    2 | Sandals |           3.84 |
|    3 | Boots   |           3.82 |
|    4 | Hat     |           3.80 |
|    5 | Skirt   |           3.78 |

### Insight

Gloves had the highest average rating among the top five products identified.

---

# 🚚 Q4. Standard vs. Express Shipping

### Question

**How does average purchase amount differ between Standard and Express shipping?**

### SQL

```sql
SELECT 
    shipping_type,
    ROUND(AVG(purchase_amount),2)
FROM customer
WHERE shipping_type IN ('Standard','Express')
GROUP BY shipping_type;
```

### Result

| Shipping Type | Average Purchase |
| ------------- | ---------------: |
| Standard      |            58.46 |
| Express       |            60.48 |

### Insight

Customers using Express shipping had a slightly higher average purchase amount than customers using Standard shipping.

> This represents an observed association in the dataset and does not prove that Express shipping causes higher spending.

---

# 👤 Q5. Subscribers vs. Non-Subscribers

### Question

**Do subscribed customers spend more?**

### SQL

```sql
SELECT 
    subscription_status,
    COUNT(customer_id) AS total_customer,
    ROUND(AVG(purchase_amount),2) AS avg_spend,
    ROUND(SUM(purchase_amount),2) AS total_revenue
FROM customer
GROUP BY subscription_status
ORDER BY total_revenue, avg_spend DESC;
```

### Result

| Subscription Status | Customers | Average Spend | Total Revenue |
| ------------------- | --------: | ------------: | ------------: |
| Yes                 |     1,053 |         59.49 |        62,645 |
| No                  |     2,847 |         59.87 |       170,436 |

### Insight

Non-subscribers generated substantially more total revenue because they represent a much larger customer group.

The average purchase amount was also slightly higher for non-subscribers.

---

# 🏷️ Q6. Products with Highest Discount Rate

### Question

**Which five products have the highest percentage of purchases with discounts applied?**

### SQL

```sql
SELECT 
    item_purchased,
    ROUND(
        100.0 * SUM(
            CASE 
                WHEN discount_applied = 'Yes' THEN 1
                ELSE 0
            END
        ) / COUNT(*),
        2
    ) AS discount_rate
FROM customer
GROUP BY item_purchased
ORDER BY discount_rate DESC
LIMIT 5;
```

### Result

| Rank | Product  | Discount Rate |
| ---: | -------- | ------------: |
|    1 | Hat      |        50.00% |
|    2 | Sneakers |        49.66% |
|    3 | Coat     |        49.07% |
|    4 | Sweater  |        48.17% |
|    5 | Pants    |        47.37% |

### Insight

Hat had the highest discount rate among the products analyzed.

---

# 👥 Q7. Customer Segmentation

Customers were segmented based on the number of previous purchases.

### Classification

```text
New       → previous_purchases <= 1
Returning → previous_purchases BETWEEN 2 AND 10
Loyal     → previous_purchases > 10
```

### SQL

```sql
WITH customer_type AS (
    SELECT
        CASE
            WHEN previous_purchases <= 1 THEN 'New'
            WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
            ELSE 'Loyal'
        END AS customer_segment,
        COUNT(*) AS customer_count
    FROM customer
    GROUP BY customer_segment
)
SELECT *
FROM customer_type
ORDER BY customer_count DESC;
```

### Result

| Customer Segment | Customers |
| ---------------- | --------: |
| Loyal            |     3,116 |
| Returning        |       701 |
| New              |        83 |

### Insight

The Loyal segment represents the largest customer group in the dataset.

---

# 🛒 Q8. Top Products Within Each Category

A SQL window function was used to rank products within each category.

### SQL

```sql
WITH item_counts AS (
    SELECT 
        category,
        item_purchased,
        COUNT(customer_id) AS total_orders,
        ROW_NUMBER() OVER(
            PARTITION BY category
            ORDER BY COUNT(customer_id) DESC
        ) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT 
    item_rank,
    category,
    item_purchased,
    total_orders
FROM item_counts
WHERE item_rank <= 3;
```

### Result

| Category    | Rank | Product    | Orders |
| ----------- | ---: | ---------- | -----: |
| Accessories |    1 | Jewelry    |    171 |
| Accessories |    2 | Sunglasses |    161 |
| Accessories |    3 | Belt       |    161 |
| Clothing    |    1 | Blouse     |    171 |
| Clothing    |    2 | Pants      |    171 |
| Clothing    |    3 | Shirt      |    169 |
| Footwear    |    1 | Sandals    |    160 |
| Footwear    |    2 | Shoes      |    150 |
| Footwear    |    3 | Sneakers   |    145 |
| Outerwear   |    1 | Jacket     |    163 |
| Outerwear   |    2 | Coat       |    161 |

> The provided query output contained two Outerwear products, so no third Outerwear product is assumed.

---

# 🔁 Q9. Repeat Buyers and Subscription

### Question

**Are customers with more than five previous purchases also likely to subscribe?**

### SQL

```sql
SELECT 
    subscription_status,
    COUNT(customer_id) AS repeat_buyers
FROM customer
WHERE previous_purchases > 5
GROUP BY subscription_status;
```

### Result

| Subscription Status | Repeat Buyers |
| ------------------- | ------------: |
| No                  |         2,518 |
| Yes                 |           958 |

### Total Repeat Buyers

**3,476**

### Insight

Among customers with more than five previous purchases, the majority were non-subscribers.

---

# 👶 Q10. Revenue by Age Group

### SQL

```sql
SELECT 
    age_group,
    SUM(purchase_amount) AS total_revenue
FROM customer
GROUP BY age_group
ORDER BY total_revenue DESC;
```

### Result

| Age Group   | Total Revenue |
| ----------- | ------------: |
| Young Adult |        62,143 |
| Middle-aged |        59,197 |
| Adult       |        55,978 |
| Senior      |        55,763 |

### Insight

Young Adults contributed the highest total revenue among the four age groups.

---

# 📈 Power BI Dashboard

The processed customer data was connected to Power BI to create an interactive business intelligence dashboard.

## Key Performance Indicators

| KPI                     |  Value |
| ----------------------- | -----: |
| Number of Customers     |   3.9K |
| Average Purchase Amount | ₹59.76 |
| Average Review Rating   |   3.75 |

## Dashboard Visualizations

The dashboard includes:

* Customer % by Subscription Status
* Revenue by Category
* Sales by Category
* Sales by Age Group
* Revenue by Age Group

## Interactive Filters

The dashboard provides filters for:

* Subscription Status
* Gender
* Category
* Shipping Type

These filters allow users to explore customer behavior across different segments.

---

# 📐 DAX Measures

## Average Review Rating

```DAX
Average Review Rating =
AVERAGE('public customer'[review_rating])
```

## Average Purchase Amount

```DAX
Average Purchase Amount =
AVERAGE('public customer'[purchase_amount])
```

## Number of Customers

```DAX
Number of Customers =
COUNT('public customer'[customer_id])
```

---

# 🔍 Key Findings

The project produced several important findings.

### 💰 Revenue

Male customers generated **157,890** in total revenue, compared with **75,191** generated by female customers.

### 🏷️ Discount Usage

* 1,677 customers had discounts applied.
* 2,223 customers did not have discounts applied.
* Hat had the highest discount rate at 50%.

### ⭐ Product Ratings

Gloves had the highest average rating among the top five products with a rating of **3.86**.

### 🚚 Shipping

Express-shipping customers had an average purchase amount of **60.48**, compared with **58.46** for Standard shipping.

### 👥 Customer Loyalty

* Loyal customers: 3,116
* Returning customers: 701
* New customers: 83

### 🔁 Repeat Buyers

Customers with more than five previous purchases:

* Non-subscribers: 2,518
* Subscribers: 958

### 👶 Age Groups

Young Adults generated the highest revenue among the age groups with **62,143**.

---

# 💡 Business Recommendations

Based on the observed patterns, the following strategies can be considered:

## 1. Improve Subscription Conversion

The subscriber group is considerably smaller than the non-subscriber group. Businesses could experiment with stronger subscription benefits, loyalty rewards, and exclusive offers.

## 2. Reward Loyal Customers

Since loyal customers represent the largest segment, personalized offers and loyalty programs could be used to maintain customer engagement.

## 3. Promote Highly Rated Products

Products such as Gloves, Sandals, and Boots have strong average ratings and could be highlighted in promotional campaigns.

## 4. Optimize Discount Strategies

Products with high discount rates should be monitored to determine whether discounts are supporting sales effectively without unnecessarily reducing profit margins.

## 5. Target High-Revenue Age Groups

Young Adults generated the highest revenue among the defined age groups and could be considered an important target segment.

## 6. Promote Category Leaders

Products such as Jewelry, Blouse, Sandals, and Jacket performed strongly within their respective categories and could be considered for targeted promotions.

## 7. Analyze Express-Shopping Customers

Express-shipping customers showed a slightly higher average purchase amount. Further analysis could determine whether premium shipping preferences are associated with higher-value customers.

> These recommendations are based on patterns observed in the dataset and should not be interpreted as causal conclusions.

---

# 📁 Suggested Repository Structure

```text
customer-shopping-behavior-analysis/
│
├── README.md
│
├── data/
│   └── customer_shopping_behavior.csv
│
├── notebooks/
│   └── customer_shopping_analysis.ipynb
│
├── sql/
│   └── customer_analysis.sql
│
├── powerbi/
│   └── customer_shopping_dashboard.pbix
│
├── screenshots/
│   ├── data_preprocessing.png
│   ├── database_connection.png
│   ├── sql_analysis.png
│   └── powerbi_dashboard.png
│
├── requirements.txt
│
└── .gitignore
```

---

# ⚙️ Installation

Clone the repository:

```bash
git clone <YOUR_REPOSITORY_URL>
cd customer-shopping-behavior-analysis
```

Install the required Python libraries:

```bash
pip install -r requirements.txt
```

Or install them directly:

```bash
pip install pandas sqlalchemy psycopg2-binary
```

---

# 🚀 How to Run the Project

## Step 1: Dataset

Place the dataset inside the `data` folder:

```text
data/customer_shopping_behavior.csv
```

## Step 2: Run Python Notebook

Open:

```text
notebooks/customer_shopping_analysis.ipynb
```

Run the notebook to:

* Load the dataset
* Inspect the data
* Handle missing values
* Standardize column names
* Create age groups
* Convert purchase frequency into days
* Remove redundant columns
* Connect to PostgreSQL
* Upload the processed data

## Step 3: Configure PostgreSQL / Neon

Create a PostgreSQL database using Neon and configure the database connection securely.

Recommended approach:

```python
import os
from sqlalchemy import create_engine

DATABASE_URL = os.getenv("DATABASE_URL")

engine = create_engine(DATABASE_URL)
```

Do not hard-code passwords in the repository.

## Step 4: Run SQL Analysis

Open:

```text
sql/customer_analysis.sql
```

Run the queries against the `customer` table.

## Step 5: Open Power BI

Connect Power BI to the PostgreSQL/Neon database and load the `customer` table.

Use the dashboard to explore:

* Customer behavior
* Sales
* Revenue
* Product categories
* Age groups
* Subscription behavior
* Shipping preferences

---

# 🔐 Security

Never upload sensitive credentials to GitHub.

Do not commit:

```text
.env
DATABASE_URL
DB_PASSWORD
API_KEYS
```

Recommended `.gitignore`:

```gitignore
.env
*.env
__pycache__/
.ipynb_checkpoints/
```

If a database password has previously been exposed, rotate the password before making the repository public.

---

# 📸 Project Screenshots

Recommended screenshots for the repository include:

### Dataset Preview

Show:

* `df.head()`
* Dataset structure
* Data preprocessing

### Database Connection

Show:

* Successful PostgreSQL/Neon connection
* `SELECT version()`
* `customer` table

### SQL Analysis

Show selected SQL queries and their outputs.

### Power BI Dashboard

Show the complete interactive dashboard with KPIs, charts, and filters.

---

# 🔄 Complete Project Workflow

```text
                  CUSTOMER DATA
                        │
                        ▼
                 ┌─────────────┐
                 │    Python   │
                 │   Pandas    │
                 └──────┬──────┘
                        │
                        ▼
              Data Preprocessing
                        │
              ┌─────────┴─────────┐
              │                   │
              ▼                   ▼
       Missing Values      Feature Engineering
              │                   │
              └─────────┬─────────┘
                        ▼
                 Clean Dataset
                        │
                        ▼
               ┌─────────────────┐
               │ PostgreSQL/Neon │
               └────────┬────────┘
                        │
                        ▼
                  SQL Analysis
                        │
                        ▼
                  Business KPIs
                        │
                        ▼
                 ┌─────────────┐
                 │  Power BI   │
                 └──────┬──────┘
                        │
                        ▼
             Interactive Dashboard
                        │
                        ▼
                Business Insights
```

---

# 🚀 Future Scope

The project can be extended with advanced analytics and machine learning techniques.

### Customer Churn Prediction

Predict customers who are likely to stop purchasing.

### Customer Lifetime Value

Estimate the long-term value of individual customers.

### Product Recommendation

Build recommendation systems based on customer purchase behavior.

### Advanced Customer Segmentation

Apply clustering algorithms such as:

* K-Means
* Hierarchical Clustering
* DBSCAN

### Sales Forecasting

Use historical customer data to forecast future sales.

### Real-Time Analytics

Connect live transactional data to a continuously updated dashboard.

### Personalized Marketing

Use customer segments and purchasing patterns to create personalized marketing campaigns.

---

# ⚠️ Limitations

* The analysis is based only on variables available in the dataset.
* The dataset does not provide complete long-term customer histories.
* Observed relationships do not prove causation.
* Subscription analysis does not explain why customers subscribe or do not subscribe.
* External factors such as competitors, economic conditions, and market trends are not included.
* Customer segmentation thresholds were defined specifically for this project.
* Discount analysis identifies purchasing patterns but does not measure profitability or margin impact.
* Q2 produces customer-level qualifying records rather than a summarized aggregate count.

---

# 📊 Project Outcomes

This project demonstrates a complete end-to-end data analytics workflow:

**Data Collection → Data Cleaning → Feature Engineering → Database Management → SQL Analysis → Data Visualization → Business Insights**

The project demonstrates practical skills in:

* Python
* Pandas
* SQL
* PostgreSQL
* Neon
* Power BI
* DAX
* Data Visualization
* Customer Analytics
* Business Intelligence

The analysis converts raw customer shopping data into structured information that can support decisions related to **customer retention, marketing, product promotion, discount strategy, subscription growth, and sales analysis**.

---

# 👨‍💻 Author

**Anurag Mishra**

Computer Science and Business Systems (CSBS)
VIT-AP University

---

# ⭐ Acknowledgement

This project was developed as an academic and practical data analytics project to demonstrate the complete process of transforming raw customer data into actionable business insights using modern data analytics and business intelligence tools.

---

# 📜 License

This project is intended primarily for **educational and academic purposes**.

```
```
