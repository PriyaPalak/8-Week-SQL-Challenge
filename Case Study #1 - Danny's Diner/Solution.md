# :spaghetti: Case Study #1. Danny's Diner

## Solution

View the raw SQL Syntax [here](https://github.com/PriyaPalak/8-Week-SQL-Challenge/blob/main/Case%20Study%20%231%20-%20Danny's%20Diner/Raw%20SQL%20File).

***

### 1. What is the total amount each customer spent at the restaurant?
 

````sql
SELECT
 s.customer_id, SUM(price) AS total_amount_spent
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
GROUP BY s.customer_id;
````
**Answer:**

![1](https://user-images.githubusercontent.com/96012488/193448175-d3c084fb-a249-48e9-a400-7220b93d0233.png)



### 2. How many days has each customer visited the restaurant? 

````sql
SELECT 
 customer_id , COUNT(DISTINCT order_date) AS days_visited
FROM sales
GROUP BY customer_id;
````
**Answer:**

![2](https://user-images.githubusercontent.com/96012488/193448217-58c2da30-f743-4d00-81f7-76f8e7024c78.png)



### 3. What was the first item from the menu purchased by each customer?


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
**Output:**

![3 1](https://user-images.githubusercontent.com/96012488/193448291-d0e05600-20c8-401f-b4bb-8338eb822bea.png)


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

**Output:**

![3 2](https://user-images.githubusercontent.com/96012488/193448301-ec67355f-c78f-4148-92b6-db8d929492dc.png)


````sql
/* 
ROW_NUMBER() and RANK(), both the window functions can be used to assign a rank to the rows of the table.  
However, the latter considers duplicate records and assigns them same rank whereas the former assigns unique ranks to such records.
Here, we want to fetch the row of rank 1 from a window partitioned based on 'customer_id' ordered by 'order_date' to get the first product ordered by the customer.
We have duplicate values of order_date for some customers as they may have ordered more than 1 item in an order.
Now, using RANK() gives two values for customer 'A' and 'C' as both of them ordered two products in a single order due to which two rows got assigned the rank 1.
Using ROW_NUMBER() gives a single result here as even for the same order_date, the product listed first in the table is assigned the rank 1.
*/
````
**Answer:**

Assuming that if the order_date is same for a customer, the item listed first in the table was purchased first.
- The first product purchased by customers A, B and C is sushi,curry and ramen, respectively.

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers? 

````sql
SELECT 
 s.product_id,m.product_name,COUNT(s.product_id) as number_purchases
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
GROUP BY s.product_id,m.product_name;
````

**Answer:**

![4](https://user-images.githubusercontent.com/96012488/193448443-0f29e5e8-0064-4d50-98b2-94567c1f20ea.png)

- Here, as we can see from the table, Ramen is the most purchased item on the menu and overall, it was purchased for 8 times.


### 5. Which item was the most popular for each customer? 

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
**Answer:**

![5](https://user-images.githubusercontent.com/96012488/193448460-69ad3c5d-598c-4582-98f6-959ccceaca7f.png)

- Ramen is the most popular item for customers 'A' and 'C' while 'B' likes all the three items equally.


### 6. Which item was purchased first by the customer after they became a member?
 

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
**Answer:**

![6 1](https://user-images.githubusercontent.com/96012488/193448547-1c68ff91-b0e4-4042-877a-473524804004.png)


````sql
--A common application of Nested Query and CTE is that both of them can be used to create temporary tables which exist only within the scope of the query.
````

### 7. Which item was purchased just before the customer became a member? 

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

**Answer:**

![7](https://user-images.githubusercontent.com/96012488/193448559-a825f276-2016-4c3a-a987-cb6c3e021ac0.png)

-  So 'A' bought sushi and curry before becoming a member. And 'B' bought sushi before becoming a member.

### 8. What is the total items and amount spent for each member before they became a member? 

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
**Answer:**

![8](https://user-images.githubusercontent.com/96012488/193448576-246d8de9-2ae8-418e-83ba-7337dadcdf2a.png)


### My Question: What is the total items and amount spent for each member per item before they became a member? 

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
**Answer:**

![My Question](https://user-images.githubusercontent.com/96012488/193448580-0ce46288-e42b-4880-b0d3-3f951c892024.png)

````sql
-- To aggregate at different levels of granularity of data, use columns in the GROUP BY clause.
-- Increase the number of columns OR Keep adding columns to the GROUP BY clause to increase the level of granularity (Highest to lowest level)
````

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

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
**Answer:**

![9](https://user-images.githubusercontent.com/96012488/193448643-6a38919f-fd64-4361-81eb-82872230f483.png)


### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items , not just sushi - how many points do customer A and B have at the end of January?
 

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

**Answer:**

![10](https://user-images.githubusercontent.com/96012488/193448661-8d70185a-957f-4adb-bdba-d1c8a7a2c3b0.png)


### Bonus Question: Join all the Things

````sql
SELECT 
	s.customer_id,s.order_date,m.product_name,m.price,
 	CASE WHEN (s.order_date >= me.join_date) AND (s.customer_id = me.customer_id) THEN 'Y'
 	ELSE 'N'
 	END AS member
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
LEFT JOIN members me
ON s.customer_id = me.customer_id;
 ````
 
 **Answer:**
 
 ![Bonus Q](https://user-images.githubusercontent.com/96012488/193448744-8e02189f-eba2-490a-aad3-11896ef751c6.png)


### Bonus Question: Rank all the Things

````sql
WITH cte AS
 (
 SELECT s.customer_id,s.order_date,m.product_name,m.price,
 CASE WHEN (s.order_date >= me.join_date) AND (s.customer_id = me.customer_id) THEN 'Y'
 ELSE 'N'
 END AS member
 FROM sales s
 JOIN menu m
 ON s.product_id = m.product_id
 LEFT JOIN members me
 ON s.customer_id = me.customer_id
 )
 SELECT *,
 	CASE WHEN member = 'N' THEN 'null'
 	ELSE RANK() OVER (PARTITION BY customer_id ORDER BY order_date)
 	END AS ranking
 FROM cte
 ORDER BY customer_id;
 ````
 
 **Answer:**
 
| customer_id | order_date | product_name | price | member | ranking | 
| ----------- | ---------- | -------------| ----- | ------ |-------- |
| A           | 2021-01-01 | sushi        | 10    | N      | NULL
| A           | 2021-01-01 | curry        | 15    | N      | NULL
| A           | 2021-01-07 | curry        | 15    | Y      | 1
| A           | 2021-01-10 | ramen        | 12    | Y      | 2
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| B           | 2021-01-01 | curry        | 15    | N      | NULL
| B           | 2021-01-02 | curry        | 15    | N      | NULL
| B           | 2021-01-04 | sushi        | 10    | N      | NULL
| B           | 2021-01-11 | sushi        | 10    | Y      | 1
| B           | 2021-01-16 | ramen        | 12    | Y      | 2
| B           | 2021-02-01 | ramen        | 12    | Y      | 3
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-07 | ramen        | 12    | N      | NULL


***





