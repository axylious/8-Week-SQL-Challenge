# Case Study #1: Danny's Diner

![Danny's Diner](https://8weeksqlchallenge.com/images/case-study-designs/1.png)

## Table of Contents

* [Problem](https://github.com/axylious/8-Week-SQL-Challenge/tree/main/Case%20Study%20%231%20-%20Danny's%20Diner#problem)
* [Entity Relationship Diagram](https://github.com/axylious/8-Week-SQL-Challenge/tree/main/Case%20Study%20%231%20-%20Danny's%20Diner#entity-relationship-diagram)
* [Question and Solution](https://github.com/axylious/8-Week-SQL-Challenge/tree/main/Case%20Study%20%231%20-%20Danny's%20Diner#question-and-solution)

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
'''SQL
SELECT 
   s.customer_id, 
   SUM(price) tot_spent 
FROM sales s
JOIN menu m
   ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY tot_spent DESC;
'''

**Answer**
customer_id | tot_spent
--- | ---
A | 76
B | 74
C | 36

2. **How many days has each customer visited the restaurant?**
3. **What was the first item from the menu purchased by each customer?**
4. **What is the most purchased item on the menu and how many times was it purchased by all customers?**
5. **Which item was the most popular for each customer?**
6. **Which item was purchased first by the customer after they became a member?**
7. **Which item was purchased just before the customer became a member?**
8. **What is the total items and amount spent for each member before they became a member?**
9. **If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
10. **In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**