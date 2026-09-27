# Retail Operations Database Analysis

## Project Summary
This repository contains the database design, data population scripts, and analytical queries for an e-commerce retail platform. The project models store transaction workflows, validates transactional data integrity, and runs multi-level business queries to evaluate customer behavior and sales performance.

## Relational Schema Design
The database structure is built on two master tables and four dependent operational tables:
* Master Tables: `customers`, `products`
* Transactional Tables: `orders`, `order_items`, `payments`, `product_reviews`

### Data Integrity Rules
* Orders to Order Items: Configured with `ON DELETE CASCADE` to remove line items automatically if a parent order is deleted.
* Customers to Orders: Configured with `ON DELETE SET NULL` to preserve historical order financial records if a customer account is removed.
* Value Constraints: Implemented table-level `CHECK` constraints on product prices, item quantities, payment amounts, and review ratings to prevent data corruption at the entry point.

---

## Script Verification Metrics
The script populates over 1,200 transactional records. Run this verification block to confirm table hydration totals:

```sql
SELECT 'customers' AS table_name, COUNT(*) AS row_count FROM customers
UNION ALL SELECT 'products', COUNT(*) FROM products
UNION ALL SELECT 'orders', COUNT(*) FROM orders
UNION ALL SELECT 'order_items', COUNT(*) FROM order_items
UNION ALL SELECT 'payments', COUNT(*) FROM payments
UNION ALL SELECT 'product_reviews', COUNT(*) FROM product_reviews;
```

### Table Audit Log

| Table Name | Row Count | Status |
| :--- | :--- | :--- |
| customers | 30 | Verified |
| products | 50 | Verified |
| orders | 400 | Verified |
| order_items | 1201 | Verified |
| payments | 400 | Verified |
| product_reviews | 50 | Verified |

---

## SQL Queries: Level 1 (Data Extraction)

### 1. Customer contact list for email marketing
```sql
SELECT name, email FROM customers;
```

### 2. Complete product catalog details
```sql
SELECT * FROM products;
```

### 3. Unique product categories
```sql
SELECT DISTINCT category FROM products;
```

### 4. Products priced above 1,000
```sql
SELECT * FROM products WHERE price > 1000;
```

### 5. Products within the 2,000 to 5,000 price range
```sql
SELECT * FROM products WHERE price BETWEEN 2000 AND 5000;
```

### 6. Target customer data filtration via profile IDs
```sql
SELECT * FROM customers WHERE customer_id IN (30, 29, 25, 12);
```

### 7. Identify customers with names starting with 'A'
```sql
SELECT * FROM customers WHERE name LIKE 'A%';
```

### 8. Electronics products priced under 3,000
```sql
SELECT * FROM products WHERE category = 'Electronics' AND price < 3000;
```

### 9. Products sorted by price in descending order
```sql
SELECT name, price FROM products ORDER BY price DESC;
```

### 10. Products sorted by price (descending) and name (ascending)
```sql
SELECT name, price FROM products ORDER BY price DESC, name ASC;
```

---

## SQL Queries: Level 2 (Filtering and Formatting)

### 1. Identify orders with missing customer assignments
```sql
SELECT * FROM orders WHERE customer_id IS NULL;
```

### 2. Format customer columns with frontend display aliases
```sql
SELECT name AS 'Customer Name', email AS EmailId FROM customers;
```

### 3. Calculate total gross value per order line item
```sql
SELECT *, quantity * item_price AS 'Grand Total' FROM order_items;
```

### 4. Merge customer name and phone into a single column
```sql
SELECT CONCAT(name, phone) AS NameAndPhone FROM customers;
```

### 5. Extract date part from order timestamps
```sql
SELECT DATE(order_date) FROM orders;
```

### 6. List products out of stock
```sql
SELECT * FROM products WHERE stock_quantity = 0;
```

---

## SQL Queries: Level 3 (Aggregations)

### 1. Total number of orders placed
```sql
SELECT COUNT(*) AS TotalOrders FROM orders;
```

### 2. Cumulative store revenue from all orders
```sql
SELECT SUM(total_amount) AS TotalRevenue FROM orders;
```

### 3. Calculate average order value across the platform
```sql
SELECT AVG(total_amount) AS AverageRevenue FROM orders;
```

### 4. Count of distinct customers who have placed an order
```sql
SELECT COUNT(DISTINCT customer_id) AS active_customers FROM orders;
```

### 5. Order volume generated per customer
```sql
SELECT customer_id, COUNT(order_id) AS NumberOfOrdersPlaced FROM orders GROUP BY customer_id;
```

### 6. Cumulative sales revenue generated per customer
```sql
SELECT customer_id, SUM(total_amount) AS TotalSalesAmount FROM orders GROUP BY customer_id ORDER BY TotalSalesAmount DESC;
```

### 7. Total quantity of items sold per category
```sql
SELECT p.category AS Category, SUM(oi.quantity) AS ProductsFromCategorySold FROM products as p INNER JOIN order_items as oi ON p.product_id = oi.product_id GROUP BY Category;
```

### 8. Average item price per product category
```sql
SELECT category, ROUND(AVG(price), 2) AS AverageItemPrice FROM products GROUP BY category;
```

### 9. Daily transaction volume log
```sql
SELECT DATE(order_date) AS OrderDate, COUNT(order_id) FROM orders GROUP BY OrderDate;
```

### 10. Total revenue collected split by payment method
```sql
SELECT method, SUM(amount_paid) AS TotalPaymentReceived FROM payments GROUP BY method;
```
## SQL Queries: Level 4 (Multi-Table Joins)

### 1. Retrieve order details mapped to customer names
```sql
SELECT c.name AS CustomerName, o.*
FROM customers c 
INNER JOIN orders o ON c.customer_id = o.customer_id;
```

### 2. Isolate products that have recorded a transaction history
```sql
SELECT DISTINCT(p.name) AS ProductName
FROM products as p 
INNER JOIN order_items as oi ON p.product_id = oi.product_id;
```

### 3. Match active orders with their respective payment methods
```sql
SELECT o.*, p.method
FROM orders o 
INNER JOIN payments p ON o.order_id = p.order_id;
```

### 4. Fetch comprehensive customer profiles alongside complete order histories
```sql
SELECT c.*, o.*
FROM customers c 
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

### 5. Review product catalog records along with actual transaction line item quantities
```sql
SELECT p.*, oi.quantity AS quantity
FROM products as p 
LEFT JOIN order_items oi ON p.product_id = oi.product_id;
```

### 6. Audit payment transactions to capture entries without matched parent orders
```sql
SELECT p.method, o.*
FROM orders o 
RIGHT JOIN payments p ON o.order_id = p.order_id;
```

### 7. Extract a master cross-reference log mapping customers, orders, and payment IDs
```sql
SELECT c.customer_id, o.order_id, p.payment_id
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
INNER JOIN payments p ON o.order_id = p.order_id;
```

---

## SQL Queries: Level 5 (Subqueries)

### 1. Filter products priced above the historical system catalog average
```sql
SELECT *
FROM products
WHERE price > (
    SELECT AVG(price) 
    FROM products
)
ORDER BY price ASC;
```

### 2. Isolate customer records matching profiles with at least one recorded order
```sql
SELECT *
FROM customers
WHERE customer_id IN (
    SELECT customer_id 
    FROM orders
    WHERE order_id IS NOT NULL
);
```

### 3. Extract orders where the specific transaction amount exceeds that customer's personal average spending
```sql
SELECT order_id, customer_id, status, total_amount
FROM orders AS o1
WHERE total_amount > (
    SELECT AVG(total_amount)
    FROM orders AS o2
    WHERE o1.customer_id = o2.customer_id
)
ORDER BY customer_id ASC;
```

### 4. Isolate inactive profiles where zero orders have been processed
```sql
SELECT *
FROM customers
WHERE customer_id NOT IN (
    SELECT customer_id 
    FROM orders
    WHERE customer_id IS NOT NULL
);
```

### 5. Identify inventory items that have never been purchased
```sql
SELECT *
FROM products
WHERE product_id NOT IN (
    SELECT product_id 
    FROM order_items
    WHERE product_id IS NOT NULL
);
```

### 6. Track the peak transaction amount processed for each unique client account
```sql
SELECT customer_id, total_amount
FROM orders AS o1
WHERE total_amount = (
    SELECT MAX(total_amount)
    FROM orders AS o2
    WHERE o1.customer_id = o2.customer_id
)
ORDER BY customer_id ASC;
```

### 7. Identify the peak transaction profile for each client, including full account profiles
```sql
SELECT c.*, o.total_amount
FROM customers AS c 
INNER JOIN orders AS o ON c.customer_id = o.customer_id
WHERE o.total_amount = (
    SELECT MAX(total_amount)
    FROM orders AS o2
    WHERE o.customer_id = o2.customer_id
);
```

---

## SQL Queries: Level 6 (Set Operations)

### 1. Compile a unified list of accounts that have either purchased an item or left a product review
```sql
SELECT customer_id, name
FROM customers
WHERE customer_id IN (
    SELECT customer_id 
    FROM orders
)
OR customer_id IN (
    SELECT customer_id 
    FROM product_reviews
    WHERE review_text IS NOT NULL
);
```

### 2. Isolate active accounts that have completed both a purchase and a matching review matrix
```sql
SELECT c.customer_id, c.name
FROM customers AS c
WHERE EXISTS (
    SELECT 1 
    FROM orders AS o 
    WHERE c.customer_id = o.customer_id
)
AND EXISTS (
    SELECT 1 
    FROM product_reviews AS pr 
    WHERE c.customer_id = pr.customer_id
);
```
