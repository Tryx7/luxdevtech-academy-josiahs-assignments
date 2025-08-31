
### ETL Pipeline Code Explanation - Progressive Learning Guide

## 📚 Table of Contents
1. [Introduction to ETL](#introduction-to-etl)
2. [Code Walkthrough by Level](#code-walkthrough-by-level)
3. [Key Business Insights](#key-business-insights)
4. [Best Practices](#best-practices)
5. [Learning Path](#learning-path)

---

## 🔄 Introduction to ETL

**ETL** stands for **Extract, Transform, Load** - a fundamental data engineering process:

| Phase | Purpose | Example |
|-------|---------|---------|
| **Extract** | Get data from sources | API calls, database queries, file reading |
| **Transform** | Clean, modify, and enrich data | Remove duplicates, convert currencies, categorize |
| **Load** | Store processed data | Save to database, data warehouse, or files |

---

## 💻 Code Walkthrough by Level

### Level 1: Basic Setup and Imports

```python
import requests      # For making HTTP API calls
import pandas as pd  # For data manipulation and analysis
from sqlalchemy import create_engine  # For database connections
import os           # For accessing environment variables
from dotenv import load_dotenv  # For loading .env files
```

> 💡 **What's happening**: We're importing the tools we need. Think of this like gathering all your cooking utensils before starting a recipe.
> 
> 🔑 **Key concept**: `pandas` is like Excel but for programming - it helps us work with tables of data.

---

### Level 2: Environment Setup

```python
load_dotenv()  # Load environment variables from .env file
```

> 💡 **What's happening**: This loads sensitive information (like database passwords) from a hidden `.env` file instead of putting them directly in the code.
> 
> 🔒 **Security note**: Never put passwords or API keys directly in your code! Always use environment variables.

---

### Level 3: 📥 EXTRACTION - Getting Data from an API

```python
base_url = "https://data-liart.vercel.app/data"
all_products = []
page = 1
has_more = True

while has_more:
    response = requests.get(f"{base_url}?page={page}")
    data = response.json()
    all_products.extend(data.get('data', []))
    has_more = data.get('hasMore', False)
    page += 1
```

> 💡 **What's happening**: 
> - We're calling an API that returns product data page by page
> - The `while` loop continues until there are no more pages
> - Each page's data gets added to our `all_products` list
> 
> 🔑 **Key concept**: APIs often return data in "pages" to avoid overwhelming the server or client. This is called **pagination**.
> 
> 🌍 **Real-world analogy**: Like flipping through pages of a catalog, collecting items from each page until you reach the end.

---

### Level 4: Converting to DataFrame

```python
df = pd.DataFrame(all_products)
```

> 💡 **What's happening**: We convert our list of products into a pandas DataFrame (think of it as a spreadsheet).
> 
> ⚡ **Why DataFrames**: They make it easy to clean, analyze, and manipulate tabular data.

---

### Level 5: 🔄 TRANSFORMATION - Data Cleaning Basics

```python
df_clean = df.copy()
df_clean = df_clean.drop_duplicates()
```

> 💡 **What's happening**: 
> - We make a copy to preserve the original data
> - Remove any duplicate rows
> 
> ✅ **Best practice**: Always work on a copy of your data so you can go back if something goes wrong.

---

### Level 6: Handling Missing Values

```python
df_clean['product_num_ratings'] = df_clean['product_num_ratings'].fillna(0).astype(int)
df_clean['product_star_rating'] = df_clean['product_star_rating'].fillna(0)
```

> 💡 **What's happening**: 
> - `fillna(0)` replaces missing values (NaN) with 0
> - `astype(int)` converts the data type to integer
> 
> ⚠️ **Why this matters**: Missing data can break calculations. We need to decide what missing values mean (here, 0 ratings means no reviews yet).

---

### Level 7: Advanced Data Cleaning - Price Normalization

```python
def clean_price(price_value):
    if pd.isna(price_value):
        return 0.0
    if isinstance(price_value, (int, float)):
        return float(price_value)
    
    price_str = str(price_value)
    cleaned_str = ''.join(char for char in price_str if char.isdigit() or char == '.')
    
    try:
        return float(cleaned_str) if cleaned_str else 0.0
    except ValueError:
        return 0.0
```

> 💡 **What's happening**: This function handles messy price data that might have:
> - Currency symbols ($, £, €)
> - Commas (1,000.50)
> - Other random characters
> 
> 🔑 **Key concept**: Real-world data is messy! You need robust cleaning functions.

---

### Level 8: Currency Conversion

```python
country_to_currency = {
    'US': 'USD', 'CA': 'CAD', 'UK': 'GBP', 'DE': 'EUR', 'FR': 'EUR', ...
}
exchange_rates = {'USD': 1.0, 'CAD': 0.73, 'GBP': 1.22, 'EUR': 1.07, ...}

df_clean['product_currency'] = df_clean['country'].map(country_to_currency)
df_clean['price_usd'] = df_clean.apply(
    lambda row: row['product_price'] * exchange_rates.get(row['product_currency'], 1),
    axis=1
)
```

> 💡 **What's happening**: 
> - Map each country to its currency
> - Convert all prices to USD for consistent comparison
> - `lambda` is a small function that works on each row
> 
> 💰 **Business value**: Now we can compare prices across different countries fairly!

---

### Level 9: Creating Derived Variables

```python
df_clean['revenue_estimate'] = df_clean['price_usd'] * df_clean['product_num_ratings']

def rating_bucket(rating):
    if rating >= 4.5: return 'Excellent'
    elif rating >= 3.5: return 'High'
    elif rating >= 2.0: return 'Medium'
    else: return 'Low'

df_clean['rating_bucket'] = df_clean['product_star_rating'].apply(rating_bucket)
```

> 💡 **What's happening**: 
> - **Revenue estimate**: Multiply price by number of ratings (proxy for sales)
> - **Rating buckets**: Group ratings into categories for easier analysis
> 
> 🔑 **Key concept**: **Feature engineering** - creating new variables that provide business insights.

---

### Level 10: Text Processing and Categorization

```python
def extract_category(title):
    title_lower = title.lower()
    if 'game' in title_lower:
        return 'Gaming'
    elif 'security' in title_lower or 'antivirus' in title_lower:
        return 'Security'
    elif 'office' in title_lower or 'microsoft' in title_lower:
        return 'Productivity'
    # ... more conditions
    else:
        return 'Other'

df_clean['category'] = df_clean['product_title'].apply(extract_category)
```

> 💡 **What's happening**: We're using keyword matching to automatically categorize products based on their titles.
> 
> 🔑 **Key concept**: **Text processing** - extracting meaningful information from unstructured text data.

---

### Level 11: 📤 LOADING - Database Connection

```python
db_connection_url = f"postgresql+psycopg2://{os.getenv('DB_USER')}:{os.getenv('DB_PASSWORD')}@{os.getenv('DB_HOST')}:{os.getenv('DB_PORT')}/{os.getenv('DB_NAME')}"
engine = create_engine(db_connection_url)
```

> 💡 **What's happening**: 
> - Building a connection string to a PostgreSQL database
> - Using environment variables for sensitive information
> 
> 🔑 **Key concept**: **Database connections** require specific formats (called connection strings) that include server location, credentials, and database name.

---

### Level 12: Loading Data to Database

```python
table_name = 'amazon_bestsellers'
df_clean.to_sql(table_name, engine, if_exists='replace', index=False)
```

> 💡 **What's happening**: 
> - Save our cleaned DataFrame to a PostgreSQL table
> - `if_exists='replace'` means overwrite the table if it already exists
> - `index=False` means don't save the row numbers
> 
> 🏢 **Why databases**: Databases are better than files for:
> - Handling large amounts of data
> - Multiple people accessing data simultaneously
> - Complex queries and analysis

---

### Level 13: 📊 SQL Analysis Queries

```sql
SELECT product_title, country, revenue_estimate
FROM amazon_bestsellers
ORDER BY revenue_estimate DESC
LIMIT 5;
```

> 💡 **What's happening**: SQL queries let us ask specific questions of our data:
> - "Show me the top 5 products by revenue"
> - "What's the average price per category?"
> - "How many products are in each rating bucket?"
> 
> 🔑 **Key concept**: SQL is the language for talking to databases. It's like asking questions in a very structured way.

---

### Level 14: Advanced Analysis Examples

#### Revenue Analysis
```sql
SELECT country_region, SUM(revenue_estimate) AS total_revenue
FROM amazon_bestsellers
GROUP BY country_region
ORDER BY total_revenue DESC;
```

#### Review Density Analysis
```sql
SELECT product_title, review_density
FROM amazon_bestsellers
WHERE price_usd > 0
ORDER BY review_density DESC
LIMIT 5;
```

> 💡 **What's happening**: 
> - **GROUP BY**: Combines rows with the same value (like Excel pivot tables)
> - **Review density**: Number of reviews per dollar - shows which products get lots of attention relative to their price

---

## 📈 Key Business Insights from This Pipeline

| Insight | Value | Implication |
|---------|-------|-------------|
| **European market dominates** | €2.09 billion revenue | Focus marketing efforts in Europe |
| **Creative software most expensive** | $8,298 average price | High-value niche market |
| **Quality ratings distribution** | 671/999 products rated "High" | Generally good product quality |
| **Security product engagement** | High review density | Customers research security heavily |

---

## 🏗️ Why This Code Structure Matters

### 1. **Separation of Concerns**
- ✅ Extraction code is separate from transformation
- ✅ Each phase has a clear purpose
- ✅ Easy to debug and modify

### 2. **Error Handling**
- ✅ Functions handle edge cases (missing data, invalid formats)
- ✅ Database operations are wrapped in try-catch blocks

### 3. **Scalability**
- ✅ Pagination handles large datasets
- ✅ Database storage supports growth
- ✅ Modular functions can be reused

### 4. **Business Value**
- ✅ Currency normalization enables global comparison
- ✅ Derived metrics provide actionable insights
- ✅ Automated categorization saves manual work

---

## 🎯 Learning Path Summary

| Level | Focus Area | Skills Developed |
|-------|------------|------------------|
| **🟢 Beginner** | Imports and basic data loading | Python basics, pandas fundamentals |
| **🟡 Intermediate** | Data cleaning and transformation | Data quality, feature engineering |
| **🟠 Advanced** | Database operations and SQL | Database design, query optimization |
| **🔴 Expert** | End-to-end pipeline design | Architecture, scalability, business value |

---

## 🚀 Next Steps for Students

1. **Practice with different APIs** - Try other data sources
2. **Experiment with transformations** - Create your own derived variables
3. **Learn more SQL** - Master joins, subqueries, and window functions
4. **Add error handling** - Make the pipeline more robust
5. **Automate the pipeline** - Schedule it to run regularly

