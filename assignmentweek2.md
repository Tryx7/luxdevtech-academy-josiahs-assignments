![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vr8x5r20nyk0qbnosalb.png)

## Understanding SQL Commands in a Customer-Sales Database
## Title: Building a Customer-Sales Database with SQL
**Subtitle:** A step-by-step breakdown of database creation, table relationships, and querying in SQL.

## Introduction
Hey everyone! 👋 Today, I’ll walk you through a simple SQL database that tracks customers, their products, and sales. We’ll cover:

✅ Database & schema creation

✅ Table relationships (Primary & Foreign Keys)

✅ Inserting data

✅ Querying for insights

Let’s dive in!

## 1. Setting Up the Database & Schema
First, we create a database and schema to organize our tables.

```
sql
-- Create the database
CREATE DATABASE jamii_db;

-- Create a schema for customer-related tables
CREATE SCHEMA customers;
```
**Why schemas?** They help group related tables (like a folder).

## 2. Creating Tables with Relationships
We define 3 tables with primary keys (PK) and foreign keys (FK) to link them.

**Table 1: customer_info**
Stores customer details.

```
sql
CREATE TABLE customers.customer_info (
    customer_id INT PRIMARY KEY,  -- Unique identifier
    fullname    VARCHAR(100),
    location    VARCHAR(100)
);
```
**Table 2: products**
Tracks products linked to customers via customer_id (FK).

```
sql
CREATE TABLE customers.products (
    product_id   INT PRIMARY KEY,
    customer_id  INT,
    product_name VARCHAR(100),
    price        FLOAT,
    FOREIGN KEY (customer_id) REFERENCES customers.customer_info (customer_id)
);
```
**Table 3: sales**
Records sales, linking to both product_id and customer_id.

```
sql
CREATE TABLE customers.sales (
    sales_id    INT PRIMARY KEY,
    product_id  INT,
    customer_id INT,
    total_sales INT,
    FOREIGN KEY (product_id) REFERENCES customers.products (product_id),
    FOREIGN KEY (customer_id) REFERENCES customers.customer_info (customer_id)
);
```
🔑 **Key Takeaway:** Foreign keys ensure data integrity by enforcing relationships.

## 3. Inserting Data
We populate the tables with sample data.

**Inserting Customers**

```
sql
INSERT INTO customers.customer_info (customer_id, fullname, location)
VALUES 
    (1, 'James Mwangi', 'Rwanda'),
    (2, 'Akello Kel', 'Amboseli'),
    (3, 'Judy J', 'Nanyuki'),
    (4, 'Ahab Jez', 'Israel');  -- Customer with no purchases
```
**Inserting Products**

```
sql
INSERT INTO customers.products (product_id, customer_id, product_name, price)
VALUES
    (1, 1, 'Laptop', 20000),
    (2, 2, 'Mouse', 1500),
    (3, 3, 'Charger', 4000);
```
**Inserting Sales Records**

```
sql
INSERT INTO customers.sales (sales_id, product_id, customer_id, total_sales)
VALUES
    (1, 1, 1, 300000),
    (2, 2, 2, 450000),
    (3, 3, 3, 100000),
    (4, 1, 1, 200000),
    (5, 2, 2, 350000),
    (6, 3, 3, 150000);
```
📌 **Note:** Customer 4 (**Ahab Jez**) has no sales yet—we’ll check for this later.

## 4. Running Basic Queries
**A. Fetch All Customers**

```
sql
SELECT * FROM customers.customer_info;
```
**Output:**

| customer_id | fullname     | location  |
|-------------|--------------|-----------|
| 1           | James Mwangi | Rwanda    |
| 2           | Akello Kel   | Amboseli  |
| 3           | Judy J       | Nanyuki   |
| 4           | Ahab Jez     | Israel    |

**B. Get Only Names & Locations**

```
sql
SELECT fullname, location FROM customers.customer_info;
```

## 5. Advanced Queries for Insights
**Query 1: Which Customers Bought What?**

```
sql
SELECT c.fullname, p.product_name, s.total_sales
FROM customers.customer_info c
JOIN customers.sales s ON c.customer_id = s.customer_id
JOIN customers.products p ON s.product_id = p.product_id;
```
**Output:**

| fullname     | product_name | total_sales |
|--------------|--------------|-------------|
| James Mwangi | Laptop       | 300000      |
| Akello Kel   | Mouse        | 450000      |
| Judy J       | Charger      | 100000      |
| ...          | ...          | ...         |

**Query 2: Total Sales per Customer**

```
sql
SELECT ci.fullname, SUM(s.total_sales) AS sales_total
FROM customers.sales s  
JOIN customers.customer_info ci ON s.customer_id = ci.customer_id
GROUP BY ci.fullname
ORDER BY sales_total DESC;
```
**Output:**

| fullname      | sales_total |
|---------------|-------------|
| Akello Kel    | 800000      |
| James Mwangi  | 500000      |
| Judy J        | 250000      |


## Query 3: Highest-Selling Product

```
sql
SELECT p.product_name, SUM(s.total_sales) AS sales_total
FROM customers.sales s
JOIN customers.products p ON s.product_id = p.product_id
GROUP BY p.product_name
ORDER BY sales_total DESC
LIMIT 1;
```
**Output:**

| product_name | sales_total |
|--------------|-------------|
| Mouse        | 800000      |


## Query 4: Customers with No Purchases

```
sql
SELECT fullname
FROM customers.customer_info ci
LEFT JOIN customers.sales s ON ci.customer_id = s.customer_id
WHERE s.sales_id IS NULL;
```
**Output:**

| fullname  |
|-----------|
| Ahab Jez  |


## Conclusion
We built a relational database from scratch, inserted data, and ran queries to extract meaningful insights!

🔹 **Key Learnings:**

✔ **Primary Keys** ensure uniqueness.

✔ **Foreign Keys** maintain relationships.

✔ **JOINs** combine data from multiple tables.

✔ **Aggregations (SUM, GROUP BY)** help analyze trends.

Try this in your own projects! 🚀






