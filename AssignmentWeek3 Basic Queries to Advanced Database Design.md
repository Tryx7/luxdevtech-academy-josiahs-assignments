## 🗄️ Complete SQL Assignment: From Basic Queries to Advanced Database Design

A comprehensive guide covering core SQL concepts, advanced techniques, query optimization, and database modeling with practical examples.

---

## Database Schema Overview
Our assignment uses a simple e-commerce database with three main tables:

- **customer_info:** Customer details (id, name, location)
- **products:** Product catalog (id, name, price, customer_id)
- **sales:** Sales transactions (id, total_sales, product_id, customer_id)

---

## <u> Section 1: Core SQL Concepts</u>

## Q1: Write a SQL query to list all customers located in Nairobi. Show only full_name and location.

**Query:**

```
sql
SELECT customer_id, full_name, location
FROM customer_info
WHERE location = 'Nairobi';
```
**Purpose:** Basic filtering with WHERE clause to find customers in a specific location.

| customer\_id | full\_name        | location |
| ------------ | ----------------- | -------- |
| 3            | Jane Robinson     | Nairobi  |
| 10           | Michael Rodriguez | Nairobi  |
| 35           | Michael Harris    | Nairobi  |
| 59           | Robert Brown      | Nairobi  |
| 71           | Olivia Robinson   | Nairobi  |
| 86           | Michael Thompson  | Nairobi  |
| 94           | James Thompson    | Nairobi  |



---

## Q2: Write a SQL query to display each customer along with the products they purchased. Include full_name, product_name, and price.
**Query:**

```
select customer_info.customer_id, full_name, product_name, price
from customer_info
inner join products
where customer_info.customer_id = products.customer_id;
```
**Purpose:** Inner join to combine customer and product data.

| customer\_id | full\_name       | product\_name | price   |
| ------------ | ---------------- | ------------- | ------- |
| 1            | Joseph Brown     | Router        | 1063.32 |
| 1            | Joseph Brown     | Webcam        | 80.30   |
| 1            | Joseph Brown     | Tablet        | 1227.13 |
| 1            | Joseph Brown     | Headphones    | 57.43   |
| 2            | William Jackson  | Headphones    | 1431.80 |
| ...          | ...              | ...           | ...     |

---


## Q3: Write a SQL query to find the total sales amount for each customer. Display full_name and the total amount spent, sorted in descending order.
**Query:**

```
sql
SELECT c.customer_id, c.full_name, SUM(s.total_sales) AS total_amount
FROM customer_info c
JOIN sales s ON c.customer_id = s.customer_id
GROUP BY c.customer_id, c.full_name
ORDER BY total_amount DESC;
```
**Purpose:** Aggregation with GROUP BY and sorting with ORDER BY.

| customer\_id | full\_name      | total\_sales |                                          
| ------------ | --------------- | ------------ | 
| 28           | Thomas White    | 9906.30      |                                          |
| 78           | Michael Brown   | 9802.85      |                                          |
| 14           | John Harris     | 8967.80      |                                          |
| 56           | Mia Lee         | 8935.90      |                                          |
| 70           | Joseph Doe      | 8816.10      |                                          |
| ...          | ...             | ...          | 



---


## Q4: Write a SQL query to find all customers who have purchased products priced above 10,000.
**Query:**

```
sql
SELECT DISTINCT c.customer_id, c.full_name, p.product_name, p.price
FROM customer_info c
JOIN products p ON c.customer_id = p.customer_id
WHERE p.price > 10000;
```
**Purpose:** Filtering with JOIN conditions and using DISTINCT to avoid duplicates.

| customer\_id | full\_name   | product\_name | price   |
| ------------ | ------------ | ------------- | ------- |
| ...              | ...           | ...          | ...


---


## Q5: Write a SQL query to find the top 3 customers with the highest total sales.
**Query:**

```
sql
SELECT c.customer_id, c.full_name, SUM(s.total_sales) AS total_sales
FROM customer_info c
JOIN sales s ON c.customer_id = s.customer_id
GROUP BY c.customer_id, c.full_name
ORDER BY total_sales DESC
LIMIT 3;
```
**Purpose:** Using LIMIT to get top results after sorting.

| customer\_id | full\_name    | total\_sales |
| ------------ | ------------- | ------------ |
| 3            | Jane Robinson | 4934.09      |
| 1            | Joseph Brown  | 3678.47      |
| 4            | Sophia Doe    | 3024.08      |


---


## <u> Section 2: Advanced SQL Techniques</u>

## Q6: Write a CTE that calculates the average sales per customer and then returns customers whose total sales are above that average.

**Query:**

```
sql
WITH customer_totals AS (
    SELECT 
        c.customer_id,
        c.full_name,
        SUM(s.total_sales) AS total_sales
    FROM customer_info c
    JOIN sales s ON c.customer_id = s.customer_id
    GROUP BY c.customer_id, c.full_name
),
average_sales AS (
    SELECT AVG(total_sales) AS avg_sales
    FROM customer_totals
)
SELECT 
    ct.customer_id,
    ct.full_name,
    ct.total_sales
FROM customer_totals ct
CROSS JOIN average_sales a
WHERE ct.total_sales > a.avg_sales
ORDER BY ct.total_sales DESC
Limit 3;
```
**Purpose:** Common Table Expressions (CTEs) for complex calculations and data reuse.

| customer\_id | full\_name    | total\_sales |
| ------------ | ------------- | ------------ |
| 40           | Thomas Thomas | 58669.20     |
| 3            | Jane Robinson | 53281.58     |
| 78           | Michael Brown | 49229.30     |
| 85           | Amelia White  | 47714.29     |
| 66           | James Lewis   | 37832.48     |



---


## Q7: Write a Window Function query that ranks products by their total sales in descending order. Display product_name, total_sales, and rank.

**Query:**

```
SELECT 
    product_name,
    total_sales,
    RANK() OVER (ORDER BY total_sales DESC) AS rankk
FROM (
    SELECT 
        p.product_name,
        SUM(s.total_sales) AS total_sales
    FROM products p
    JOIN sales s 
        ON p.product_id = s.product_id
    GROUP BY p.product_name
) AS sub
ORDER BY total_sales DESC;
```
**Purpose:** Window functions for ranking without grouping limitations.

| product_name | total_sales | rankk |
| ------------ | ----------: | ---:  |
| Microphone   |   110515.77 |    1  |
| Desk Chair   |   102157.46 |    2  |
| Monitor      |   101817.33 |    3  |
| Camera       |    98305.35 |    4  |
| Drone        |    97830.93 |    5  |
| Hard Drive   |    97668.82 |    6  |
| Tablet       |    79636.73 |    7  |




---


## Q8: Create a View called high_value_customers that lists all customers with total sales greater than 15,000.
**Query:**

```
sql
CREATE VIEW high_value_customers AS
SELECT 
    c.customer_id,
    c.full_name,
    SUM(s.total_sales) AS total_sales
FROM customer_info c
JOIN sales s ON c.customer_id = s.customer_id
GROUP BY c.customer_id, c.full_name
HAVING SUM(s.total_sales) > 15000;

-- Use the view
SELECT * FROM high_value_customers;
```
**Purpose:** Views for reusable complex queries and data abstraction.

| customer_id |    full_name    |        total_sales |
| ----------- | --------------- | -----------------: |
| 29          | John Rodriguez  |    19249.669921875 |
| 58          | John Garcia     |   32605.1396484375 |
| 20          | Amelia Thompson | 27058.469848632812 |
| 17          | Joseph Martinez |  33605.50061035156 |
| 35          | Michael Harris  | 15800.829956054688 |
| 16          | Daniel White    | 27310.069458007812 |
| 66          | James Lewis     | 37832.479736328125 |


---


## Q9: Create a Stored Procedure that accepts a location as input and returns all customers and their total spending from that location.
**Query:**

```
sql
CREATE PROCEDURE GetCustomersByLocation(IN p_location VARCHAR(100))
BEGIN
    SELECT 
        c.customer_id,
        c.full_name,
        c.location,
        SUM(s.total_sales) AS total_spending
    FROM customer_info c
    JOIN sales s ON s.customer_id = c.customer_id
    WHERE c.location = p_location
    GROUP BY c.customer_id, c.full_name, c.location
    ORDER BY total_spending DESC;
END;

-- Call the procedure
CALL GetCustomersByLocation('Nairobi');
```
**Purpose:** Stored procedures for reusable business logic with parameters.

| customer_id |     full_name     | location |        total_sales |
| ----------- | ----------------- | -------- | -----------------: |
| 3           | Jane Robinson     | Nairobi  | 53281.579833984375 |
| 86          | Michael Thompson  | Nairobi  | 26463.729522705078 |
| 35          | Michael Harris    | Nairobi  | 15800.829956054688 |
| 71          | Olivia Robinson   | Nairobi  |  12385.88037109375 |
| 10          | Michael Rodriguez | Nairobi  |  6196.950225830078 |
| 59          | Robert Brown      | Nairobi  |    3138.1201171875 |



---


## Q10: Write a recursive query to display all sales transactions in order by sales_id, along with a running total of sales.

**Query:**

```
sql
WITH RECURSIVE Sales_CTE AS (
    -- Base case: first sales record
    SELECT 
        s.sales_id,
        s.total_sales,
        s.total_sales AS running_total
    FROM sales s
    WHERE s.sales_id = (SELECT MIN(sales_id) FROM sales)

    UNION ALL

    -- Recursive case: add next sales record
    SELECT 
        s.sales_id,
        s.total_sales,
        c.running_total + s.total_sales AS running_total
    FROM sales s
    JOIN Sales_CTE c ON s.sales_id = c.sales_id + 1
)
SELECT * FROM Sales_CTE;
```
**Purpose:** Recursive CTEs for hierarchical data and cumulative calculations.

| sales\_id | total\_sales | running\_total |
| --------- | ------------ | -------------- |
| 1         | 5249.91      | 5249.91       |
| 2         | 5368.44      | 10618.30       |
| 3         | 1413.70      | 12032.00        |
| 4         | 5696.52      | 17728.60      |


---


## <u>⚡ Section 3: Query Optimization & Performance</u>

## Q11: The following query is running slowly:
**SELECT * FROM sales WHERE total_sales > 5000;
Explain two changes you would make to improve its performance and then write the optimized SQL query.**

**Answer**

**Problem:** This query runs slowly:

```
sql
SELECT * FROM sales WHERE total_sales > 5000;
```
**Solution:**

1. **Create an Index:**

 ```
 sql
CREATE INDEX idx_total_sales ON sales(total_sales);
 ```

2. **Select only needed columns:**
 

```
sql
SELECT sales_id, customer_id, total_sales
FROM sales
WHERE total_sales > 5000;
```
**Why it works:**

- Index allows fast lookups instead of full table scans
- Selecting fewer columns reduces I/O and memory usage

---


## Q12: Create an index on a column that would improve queries filtering by customer location, then write a query to test the improvement.
**Create Index:**

```
sql
CREATE INDEX idx_customer_location ON customer_info(location);
```
**Test the improvement:**

```
sql
EXPLAIN
SELECT customer_id, full_name, location
FROM customer_info
WHERE location = 'Nairobi';
```
**Purpose:** EXPLAIN shows if the query uses the index efficiently.

---


## <u> Section 4: Database Design & Modeling</u>

## Q13: Redesign the given schema into 3rd Normal Form (3NF) and provide the new CREATE TABLE statements.

**Normalized Tables:**

```
sql
-- Customers table
CREATE TABLE customer_info (
    customer_id INT PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    location VARCHAR(200)
);

-- Products table (removed customer_id - not normalized)
CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    product_price DECIMAL(10,2) NOT NULL
);

-- Orders table
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customer_info(customer_id)
);

-- Order Details (many-to-many relationship)
CREATE TABLE OrderDetails (
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```
**3NF Benefits:**

- Eliminates redundancy
- Prevents update anomalies
- Maintains data integrity

---



## Q14: Create a Star Schema design for analyzing sales by product and customer location. Include the fact table and dimension tables with their fields.
**Fact Table:**

```
sql
CREATE TABLE SalesFact (
    sales_id INT PRIMARY KEY,
    product_id INT NOT NULL,
    customer_id INT NOT NULL,
    date_id INT NOT NULL,
    quantity INT NOT NULL,
    total_amount DECIMAL(12,2) NOT NULL,
    FOREIGN KEY (product_id) REFERENCES ProductDim(product_id),
    FOREIGN KEY (customer_id) REFERENCES CustomerDim(customer_id),
    FOREIGN KEY (date_id) REFERENCES DateDim(date_id)
);
```
**Dimension Tables:**

```
sql
-- Product Dimension
CREATE TABLE ProductDim (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(100),
    price DECIMAL(10,2)
);

-- Customer Dimension
CREATE TABLE CustomerDim (
    customer_id INT PRIMARY KEY,
    full_name VARCHAR(100),
    location VARCHAR(100),
    region VARCHAR(100)
);

-- Date Dimension
CREATE TABLE DateDim (
    date_id INT PRIMARY KEY,
    full_date DATE,
    day INT,
    month INT,
    quarter INT,
    year INT
);
```
**Star Schema Benefits:**

- Optimized for analytical queries
- Fast aggregations
- Easy to understand and maintain

---


## Q15: Explain a scenario where denormalization would improve performance for reporting queries, and demonstrate the SQL table creation for that denormalized structure.

**Scenario:** Monthly sales reports need data from multiple tables. Joining 4-5 tables repeatedly is slow.

**Denormalized Table:**

```
sql
CREATE TABLE SalesReport (
    sales_id INT PRIMARY KEY,
    customer_id INT,
    full_name VARCHAR(100),
    location VARCHAR(100),
    region VARCHAR(100),
    product_id INT,
    product_name VARCHAR(100),
    category VARCHAR(100),
    price DECIMAL(10,2),
    order_date DATE,
    quantity INT,
    total_amount DECIMAL(12,2)
);

```
**When to Denormalize:**

- Read-heavy reporting workloads
- Complex joins are performance bottlenecks
- Data warehouse/analytics scenarios
- Trade storage space for query speed


**Key Takeaways**

1. **Start Simple:** Master basic queries before advanced techniques
2. **Optimize Wisely:** Index frequently queried columns
3. **Choose the Right Model:** OLTP vs OLAP requirements
4. **Test Performance:** Always measure before and after optimizations
5. **Balance Trade-offs:** Normalization vs performance needs







