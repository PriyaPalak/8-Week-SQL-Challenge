# :spaghetti: Case Study #1. Danny's Diner

## Solution

View the raw SQL Syntax [here](https://github.com/PriyaPalak/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/Raw%20SQL%20File).

***

### 1. What is the total amount each customer spent at the restaurant?

**Answer:** 

````sql
SELECT
 s.customer_id, SUM(price) AS total_amount_spent
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
GROUP BY s.customer_id;
````


### 2. How many days has each customer visited the restaurant? 

**Answer:**

````sql
SELECT 
 customer_id , COUNT(DISTINCT order_date) AS days_visited
FROM sales
GROUP BY customer_id;
````


### 3. What was the first item from the menu purchased by each customer?

**Answer:** 

````sql
-- Assuming that if the order_date is same for a customer, the item listed first in the table was purchased first.
````

````sql
-- Using ROW_NUMBER()

SELECT 
 S.customer_id,S.product_name AS first_product
FROM
(SELECT s.*,m.product_name,ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS rnk
FROM sales s
JOIN menu m
ON s.product_id = m.product_id) AS S
WHERE S.rnk = 1;
````


````sql

-- Using RANK()

SELECT 
 S.customer_id,S.product_name AS first_product
FROM
(SELECT s.*,m.product_name,RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS rnk
FROM sales s
JOIN menu m
ON s.product_id = m.product_id) AS S
WHERE S.rnk = 1;
````

````sql
/* 
ROW_NUMBER() and RANK(), both the window functions can be used to assign a rank to the rows of the table. However, the latter considers 
duplicate records and assigns them same rank whereas the former assigns unique ranks to such records.
Here, we want to fetch the row of rank 1 from a window partitioned based on 'customer_id' ordered by 'order_date' to get the first product ordered by the customer.
We have duplicate values of order_date for some customers as they may have ordered more than 1 item in an order.
Now, using RANK() gives two values for customer 'A' and 'C' as both of them ordered two products in a single order due to which two rows got assigned the rank 1.
Using ROW_NUMBER() gives a single result here as even for the same order_date, the product listed first in the table is assigned the rank 1.
*/
````

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

**Answer:** 

````sql
SELECT 
 s.product_id,m.product_name,COUNT(s.product_id) as number_purchases
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
GROUP BY s.product_id,m.product_name;
````



### 5. Which item was the most popular for each customer?

**Answer:** 

````sql
SELECT 
 X.customer_id,X.product_name,X.order_count
FROM
(SELECT s.customer_id,m.product_name,COUNT(s.product_id) as order_count,
RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC ) as rnk
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
GROUP BY m.product_name,s.customer_id) AS X
WHERE X.rnk = 1;
````


### 6. Which item was purchased first by the customer after they became a member?

**Answer:** 

````sql
--Let's assume that if the order_date and join_date are same for a member, they became the member first and then placed the order.
````

````sql
--Nested Query Approach

SELECT 
 X.customer_id,X.product_name
FROM
(SELECT s.customer_id,m.product_name,RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date) as rnk
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
JOIN members me
ON s.customer_id = me.customer_id
WHERE s.order_date >= me.join_date) AS X
WHERE X.rnk = 1;
````

````sql

--CTE Approach

WITH members_first_item_cte AS
(
SELECT s.customer_id,m.product_name,RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date) as rnk
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
JOIN members me
ON s.customer_id = me.customer_id
WHERE s.order_date >= me.join_date
)
SELECT
 customer_id,product_name
FROM members_first_item_cte
WHERE rnk = 1;
````

````sql
--A common application of Nested Query and CTE is that both of them can be used to create temporary tables which exist only within the scope of the query.
````

### 7. Which item was purchased just before the customer became a member?

**Answer:** 

````sql 
SELECT 
 X.customer_id,X.product_name,X.order_date,X.join_date
FROM
(SELECT s.customer_id,m.product_name,s.order_date,me.join_date,RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date DESC) as rnk
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
JOIN members me
ON s.customer_id = me.customer_id
WHERE s.order_date < me.join_date) AS X
WHERE X.rnk = 1;
````



### 8. What is the total items and amount spent for each member before they became a member?

**Answer:** 

````sql
SELECT 
 s.customer_id, COUNT(s.product_id) AS total_items, SUM(m.price) AS amount_spent
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
JOIN members me
ON s.customer_id = me.customer_id
WHERE s.order_date < me.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
````


### My Question: What is the total items and amount spent for each member per item before they became a member?

**Answer:** 

````sql
SELECT 
 s.customer_id, m.product_name,COUNT(s.product_id) AS total_items, SUM(m.price) AS amount_spent
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
JOIN members me
ON s.customer_id = me.customer_id
WHERE s.order_date < me.join_date
GROUP BY m.product_name,s.customer_id
ORDER BY s.customer_id,m.product_name;
````


### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

**Answer:** 

````sql
SELECT 
 X.customer_id, SUM(X.points) AS total_points
FROM
(SELECT s.customer_id, 
CASE WHEN m.product_name = 'sushi' THEN ( 2*m.price*10)
ELSE m.price*10
END AS points
FROM sales s
JOIN menu m
ON s.product_id = m.product_id) AS X
GROUP BY X.customer_id
ORDER BY X.customer_id;
````

````sql

-- Case Statement inside Sum

SELECT 
 s.customer_id, 
SUM(CASE WHEN m.product_name = 'sushi' THEN ( 2*m.price*10)
ELSE m.price*10
END )AS points
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
GROUP BY s.customer_id;
````

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items , not just sushi - how many points do customer A and B have at the end of January?

**Answer:** 

````sql
WITH total_points_cte AS
(
SELECT s.customer_id,s.order_date,m.price,m.product_name,me.join_date, DATEADD(day,6,join_date) AS first_week
FROM sales s
JOIN menu m
ON s.product_id =m.product_id
JOIN members me
ON s.customer_id = me.customer_id
)
SELECT 
 customer_id, 
 SUM(CASE WHEN order_date BETWEEN join_date AND first_week THEN (2*price*10)
         WHEN (order_date NOT BETWEEN join_date AND first_week) AND product_name ='sushi' THEN (2*price*10)
		 ELSE (price*10)
		 END) AS total_points
FROM total_points_cte
GROUP BY customer_id;
````







