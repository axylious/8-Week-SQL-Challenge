# Case Study #1: Danny's Diner

![Danny's Diner](https://8weeksqlchallenge.com/images/case-study-designs/1.png)

## Table of Contents

* [Problem](https://github.com/axylious/8-Week-SQL-Challenge/tree/main/Case%20Study%20%231%20-%20Danny's%20Diner#problem)
* [Entity Relationship Diagram](https://github.com/axylious/8-Week-SQL-Challenge/tree/main/Case%20Study%20%231%20-%20Danny's%20Diner#entity-relationship-diagram)
* [Questions and Solutions](https://github.com/axylious/8-Week-SQL-Challenge/tree/main/Case%20Study%20%231%20-%20Danny's%20Diner#questions-and-solutions)

Original Case Study [here](https://8weeksqlchallenge.com/case-study-1/)

## Problem

Danny wants to use the data he's collected from his customers at the diner to try and answer some questions, like:

1. What are their visiting patterns?
2. How much money have they spent?
3. Which menu items are favorites?

## Entity Relationship Diagram

![Entity Relationship Diagram](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

## Question and Solution

1. **What is the total amount each customer spent at the restaurant?**
```sql
SELECT 
   s.customer_id, 
   SUM(price) tot_spent 
FROM sales s
JOIN menu m
   ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY tot_spent DESC;
```

**Answer:**
customer_id | tot_spent
--- | ---
A | 76
B | 74
C | 36

***

2. **How many days has each customer visited the restaurant?**
```sql
SELECT 
   customer_id, 
   COUNT(DISTINCT order_date) visit_count
FROM sales
GROUP BY customer_id
ORDER BY visit_count DESC;
```

**Answer:**
customer_id | visit_count
--- | ---
B | 6
A | 4
C | 2

***

3. **What was the first item from the menu purchased by each customer?**
```sql
WITH ordered_sales AS (SELECT
   customer_id,
   product_id,
   order_date,
   RANK() OVER(
      PARTITION BY customer_id 
      ORDER BY order_date) purchase
FROM sales)

SELECT os.customer_id, m.product_name FROM ordered_sales os
JOIN menu m
	ON os.product_id = m.product_id
WHERE purchase = 1
GROUP BY os.customer_id, m.product_name;
```

**Answer:**
customer_id | product_name
--- | ---
A | sushi
A | curry
B | curry
C | ramen

***

4. **What is the most purchased item on the menu and how many times was it purchased by all customers?**
```sql
SELECT
   m.product_name,
   COUNT(*) total_sold
FROM menu m
JOIN sales s
   ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY total_sold DESC
LIMIT 1;
```

**Answer:**
product_name | total_sold
--- | ---
ramen | 8

***

5. **Which item was the most popular for each customer?**
```sql
WITH ordered_table AS (SELECT 
   s.customer_id,
   m.product_name,
   COUNT(*) order_count,
   RANK() OVER(
      PARTITION BY customer_id
      ORDER BY COUNT(s.customer_id) DESC) ranking
FROM menu m
JOIN sales s
   ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name
ORDER BY customer_id)

SELECT
   customer_id,
   product_name,
   order_count
FROM ordered_table
WHERE ranking = 1;
```

**Answer:**
customer_id | product_name | order_count
--- | --- | ---
A | ramen | 3
B | curry | 2
B | sushi | 2
B | ramen | 2
C | ramen | 3

***


6. **Which item was purchased first by the customer after they became a member?**
```sql
WITH first_order AS (SELECT 
   m.customer_id,
   s.product_id,
   RANK() OVER(
      PARTITION BY m.customer_id
      ORDER BY s.order_date) ranking
   FROM members m
   JOIN sales s
      ON s.customer_id = m.customer_id
      AND s.order_date > m.join_date)

SELECT
   fo.customer_id,
   m.product_name 
FROM first_order fo
JOIN menu m
   ON fo.product_id = m.product_id
WHERE fo.ranking = 1;
```

**Answer:**
customer_id | product_name
--- | ---
B | sushi
A | ramen

***

7. **Which item was purchased just before the customer became a member?**
```sql
WITH first_order AS (SELECT
   m.customer_id,
   s.product_id,
   RANK() OVER(
      PARTITION BY m.customer_id
      ORDER BY s.order_date) ranking
   FROM members m
   JOIN sales s
      ON s.customer_id = m.customer_id
      AND s.order_date < m.join_date)

SELECT
   fo.customer_id,
   m.product_name 
FROM first_order fo
JOIN menu m
   ON fo.product_id = m.product_id
WHERE fo.ranking = 1;
```

**Answer:**
customer_id | product_name
--- | ---
A | sushi
A | curry
B | curry

***

8. **What is the total items and amount spent for each member before they became a member?**
```sql
WITH first_order AS (SELECT
   m.customer_id,
   s.product_id
   FROM members m
   JOIN sales s
      ON s.customer_id = m.customer_id
      AND s.order_date < m.join_date)

SELECT
   fo.customer_id,
   COUNT(*) order_count,
   SUM(m.price) tot_price
FROM first_order fo
JOIN menu m
   ON fo.product_id = m.product_id
GROUP BY fo.customer_id
ORDER BY fo.customer_id;
```

**Answer:**
customer_id | order_count | tot_price
--- | --- | ---
A | 2 | 25
B | 3 | 40
***

9. **If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
SELECT
   s.customer_id,
   SUM(CASE WHEN m.product_name = 'sushi' THEN m.price * 20
      ELSE m.price * 10 END) points
FROM sales s
JOIN menu m
   ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

**Answer:**
customer_id | points
--- | ---
A | 860
B | 940
C | 360

10. **In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
```sql
WITH first_week AS (SELECT
   m.customer_id,
   s.product_id,
   s.order_date,
   m.join_date
   FROM members m
   JOIN sales s
      ON s.customer_id = m.customer_id
      AND s.order_date)
SELECT
   fw.customer_id,
   SUM(CASE WHEN fw.order_date BETWEEN fw.join_date AND fw.join_date + 6 THEN m.price*20
   WHEN m.product_name = 'sushi' THEN m.price*20
   ELSE m.price*10 END) points
FROM first_week fw
JOIN menu m
   ON fw.product_id = m.product_id
WHERE fw.order_date <= '2021-01-31'
GROUP BY fw.customer_id
ORDER BY fw.customer_id;
```

**Answer:**
customer_id | points
--- | ---
A | 1370
B | 820