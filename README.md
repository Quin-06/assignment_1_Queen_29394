# assignment_1_Queen_29394

# PLSQL Assignment One — Sunrise Supermarket

**Student Name:** UMUTONIWASE Nizeyimana Queen
**Student ID:** 29394
**Repository:** `assignment_1_Queen_29394`
**DBMS:** Oracle Database

## 1. Business Scenario

Sunrise Supermarket sells different products to customers. Customers can place orders containing one or more products. Each product belongs to a category and has a selling price.

The supermarket management wants to understand customer purchasing behavior, product sales, customer spending, order frequency, and revenue trends over time.

For this assignment, the database was created using four tables:

* `customers` — stores customer information.
* `products` — stores products, categories, and prices.
* `orders` — stores customer orders and order dates.
* `order_items` — stores the products and quantities included in each order.

The database contains 6 customers, 10 products across 4 categories, 15 orders, and 30 order items.

## 2. How to Run

1. Open Oracle SQL Developer, Oracle Live SQL, or another Oracle-compatible SQL environment.
2. Open the `assignment.sql` file.
3. Run the table creation statements first.
4. Run the INSERT statements to populate the database.
5. Run the JOIN, CTE, and window-function queries.
6. Review the results produced by each query.
7. Screenshots of the query results can be added to this README where required.

## 3. JOIN Queries

### JOIN Query 1 — Orders and Customers

This query uses an `INNER JOIN` between the `orders` and `customers` tables.

It displays the order ID, customer's name, city, and order date.

The JOIN is performed using `customer_id`, which is the relationship between the two tables.

```sql
SELECT
    o.order_id,
    c.customer_name,
    c.city,
    o.order_date
FROM orders o
INNER JOIN customers c
    ON o.customer_id = c.customer_id
ORDER BY o.order_date;
```

**Business interpretation:**
This query allows management to see which customers placed each order and where those customers are located. It can help the supermarket analyze ordering activity by customer and city.

**Result:**
The query returns the 15 orders together with the corresponding customer information.

---

### JOIN Query 2 — Order Items and Products

This query joins `order_items` with `products` using `product_id`.

It displays the product name, category, price, and quantity ordered.

```sql
SELECT
    oi.order_item_id,
    oi.order_id,
    p.product_name,
    p.category,
    p.price,
    oi.quantity
FROM order_items oi
INNER JOIN products p
    ON oi.product_id = p.product_id
ORDER BY oi.order_id, oi.order_item_id;
```

**Business interpretation:**
Management can use this information to understand which products are included in customer orders and the quantities purchased.

**Result:**
The query returns all 30 order-item records with their corresponding product information.

---

### JOIN Query 3 — All Customers and Their Orders

This query uses a `LEFT JOIN` so that every customer is displayed, even if the customer has not placed an order.

```sql
SELECT
    c.customer_id,
    c.customer_name,
    c.city,
    o.order_id,
    o.order_date
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_date;
```

**Business interpretation:**
A LEFT JOIN is useful because management may want to identify customers who have not yet made any purchases. These customers could potentially be contacted through customer-retention or marketing activities.

**Result:**
All customers are returned together with their orders where orders exist.

## 4. CTE Query

### Customers Above Average Spending

A Common Table Expression (CTE) called `customer_totals` is used to calculate the total amount spent by each customer.

The calculation is:

`quantity × product price`

The outer query then compares each customer's spending with the average spending of all customers.

```sql
WITH customer_totals AS (
    SELECT
        c.customer_id,
        c.customer_name,
        SUM(oi.quantity * p.price) AS total_spend
    FROM customers c
    JOIN orders o
        ON c.customer_id = o.customer_id
    JOIN order_items oi
        ON o.order_id = oi.order_id
    JOIN products p
        ON oi.product_id = p.product_id
    GROUP BY
        c.customer_id,
        c.customer_name
)
SELECT
    customer_id,
    customer_name,
    total_spend
FROM customer_totals
WHERE total_spend > (
    SELECT AVG(total_spend)
    FROM customer_totals
)
ORDER BY total_spend DESC;
```

**Business interpretation:**
This helps management identify customers whose spending is above the average. These customers contribute a relatively large amount of revenue and their purchasing behavior can be studied further.

**Result:**
The query returns only customers whose total spending is greater than the average customer spending.

## 5. Window Functions

### Window Query 1 — Rank Customers by Spending

The `RANK()` window function ranks customers according to their total spending.

```sql
RANK() OVER (
    ORDER BY total_spend DESC
)
```

Customers with higher spending receive a higher position in the ranking.

**Business interpretation:**
The query allows management to compare customer spending and identify the customers who generate the most sales revenue.

---

### Window Query 2 — Number Each Customer's Orders

The `ROW_NUMBER()` function numbers each customer's orders according to the order date.

```sql
ROW_NUMBER() OVER (
    PARTITION BY customer_id
    ORDER BY order_date, order_id
)
```

`PARTITION BY customer_id` restarts the numbering for every customer.

**Business interpretation:**
This allows management to see whether an order is a customer's first, second, third, or later purchase.

---

### Window Query 3 — Running Revenue Total

The query first calculates the revenue generated by each order. It then uses the `SUM()` window function to calculate cumulative revenue.

```sql
SUM(order_revenue) OVER (
    ORDER BY order_date, order_id
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

**Business interpretation:**
The running total shows how revenue accumulates over time. Management can use this to monitor sales growth throughout the period.

---

### Window Query 4 — Days Between Customer Orders

The `LAG()` function is used to obtain the previous order date for each customer.

```sql
LAG(order_date) OVER (
    PARTITION BY customer_id
    ORDER BY order_date, order_id
)
```

The previous date is then subtracted from the current order date to calculate the number of days between orders.

**Business interpretation:**
This helps management understand customer purchasing frequency. A shorter gap indicates that a customer is ordering more frequently, while a longer gap indicates less frequent purchasing.

## 6. Overall Business Interpretation

The database provides information that can help Sunrise Supermarket understand its customers and sales activity.

The JOIN queries connect customer, order, and product information. The CTE identifies customers spending above the average. The ranking window function allows customer spending to be compared, while `ROW_NUMBER()` shows the sequence of each customer's purchases.

The running revenue calculation provides a view of how sales accumulate over time. Finally, the `LAG()` calculation provides information about the time between customer purchases.

Together, these queries provide management with useful information about customer spending, purchase frequency, and revenue trends.

## 7. Challenges and Resolutions

### Challenge 1: Calculating customer spending

Customer spending is not stored directly in the database. It must be calculated from the quantity of each product and its price.

**Resolution:**
I joined `customers`, `orders`, `order_items`, and `products`, then calculated:

`quantity × price`

and used `SUM()` to calculate each customer's total spending.

### Challenge 2: Finding customers above average spending

The average spending must be calculated after customer totals have been generated.

**Resolution:**
I used a CTE to first calculate the total spending for each customer. The outer query then compared each total with the average of those totals.

### Challenge 3: Calculating running revenue

An order can contain multiple products, so revenue needs to be calculated at the order level before calculating the running total.

**Resolution:**
I created an `order_revenue` CTE that calculates revenue for each order and then applied a windowed `SUM()` to calculate cumulative revenue.

### Challenge 4: Calculating the time between orders

The current order date must be compared with the previous order date for the same customer.

**Resolution:**
I used the `LAG()` window function with `PARTITION BY customer_id` and ordered the customer's orders by date.

## 8. Conclusion

The assignment demonstrates the use of relational database concepts, JOINs, Common Table Expressions, aggregate functions, and window functions in Oracle Database.

These techniques allow Sunrise Supermarket to transform its transactional data into useful information for understanding customers, orders, products, and revenue.
