# :pizza: Case Study #2 - Pizza Runner

## :memo: Solution A. Pizza Metrics

### 1. How many Pizzas were ordered?

````sql
SELECT 
   COUNT(pizza_id) AS pizza_order_count
FROM #customer_orders;
 ````
 **Answer:**
 
 
 ![Pizza Metrics 1](https://user-images.githubusercontent.com/96012488/187139840-1ee891f6-c891-4dfa-8275-5e703b996674.png)

 
 - 14 Pizzas were ordered in total. 
 
 ### 2. How many unique customer orders were made? 
 
 ````sql
 SELECT 
	COUNT(DISTINCT customer_id) AS unique_customer_count
 FROM #customer_orders;
 ````
 
 **Answer**
 
 ![Pizza Metrics 2](https://user-images.githubusercontent.com/96012488/187223705-a89831b2-f497-4451-ab3b-5503e9d4ab59.png)


-  5 unique customer orders were made.
 
 ### 3. How many successful orders were delivered by each runner?
 
 ````sql
 SELECT
	runner_id, 
	COUNT(pickup_time) AS successful_order_count
 FROM #runner_orders
 WHERE pickup_time != ''
 GROUP BY runner_id;
 ````
 
 **Answer**
 
 ![Pizza Metrics 3](https://user-images.githubusercontent.com/96012488/187222071-758047bb-116a-4180-944c-9811f6695439.png)

 
 ### 4. How many of each type of pizza was delivered?
 
 ````sql
 SELECT 
	c.pizza_id , 
	COUNT(r.pickup_time) AS pizza_count
 FROM #customer_orders c
 JOIN #runner_orders r
 ON c.order_id =r.order_id
 WHERE r.pickup_time != ''
 GROUP BY c.pizza_id;
 ````
 
 **Answer**
 
 ![Pizza Metrics 4](https://user-images.githubusercontent.com/96012488/187222284-440d594c-378e-4188-ae86-6fd4dd1dcff1.png)

 
 ### 5. How many Vegetarian and Meatlovers were ordered by each customer?
 
 ````sql
 SELECT
	c.customer_id, 
	CAST(p.pizza_name AS VARCHAR(20)) AS pizza_name,
	COUNT(CAST(p.pizza_name AS VARCHAR(20))) AS pizza_count
 FROM #customer_orders c
 JOIN pizza_names p                                                    
 ON c.pizza_id = p.pizza_id
 GROUP BY c.customer_id,CAST(p.pizza_name AS VARCHAR(20));        -- Here, pizza_name (TEXT) is cast as VARCHAR(20) to enable comparisons and calculations.
 ````
 
 **Answer**
 
 ![Pizza Metrics 5](https://user-images.githubusercontent.com/96012488/187222463-3e51d82b-f18c-4552-a546-0b505938199d.png)

 
### 6. What was the maximum number of pizzas delivered in a single order?

````sql
WITH pizza_per_order_cte AS

 (SELECT 
	c.order_id, 
	COUNT(c.pizza_id) AS pizza_count_per_order
 FROM #customer_orders c
 JOIN #runner_orders r
 ON c.order_id = r.order_id
 WHERE r.pickup_time != ''
 GROUP BY c.order_id
 )

 SELECT 
	MAX(pizza_count_per_order) AS max_pizza_count
 FROM pizza_per_order_cte;
 ````
 
 **Answer**
 
 ![Pizza Metrics 6 1](https://user-images.githubusercontent.com/96012488/187222656-2dd47996-39cb-4a9a-b158-485efb661744.png)

![Pizza Metrics 6 2](https://user-images.githubusercontent.com/96012488/187222773-49b81fab-68ba-427e-ba39-17f3b74f425c.png)

-  The maximum number of pizzas delivered in a single order is 3.

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

````sql
SELECT 
 c.customer_id,
	SUM
	(CASE WHEN c.exclusions != '' OR c.extras != '' THEN 1
	ELSE 0
	END) AS at_least_1_change,
	SUM
	(CASE WHEN c.exclusions = '' AND c.extras = '' THEN 1
	ELSE 0
	END) AS no_change
FROM #customer_orders c
JOIN #runner_orders r
ON c.order_id = r.order_id
WHERE r.pickup_time != ''
GROUP BY c.customer_id;
````
 
 **Answer**
 
 ![Pizza Metrics 7](https://user-images.githubusercontent.com/96012488/187222969-7d3e7995-d60d-4f94-866e-dc4966bbbcc6.png)


### 8. How many pizzas were delivered that had both exclusions and extras?

````sql
SELECT 
   COUNT(c.pizza_id) AS pizza_count_w_exclusions_extras
 FROM #customer_orders c
 JOIN #runner_orders r
 ON c.order_id = r.order_id
 WHERE (c.exclusions != '' AND c.extras != '') AND (r.pickup_time != '');
 ````
 
 **Answer**
 
 ![Pizza Metrics 8](https://user-images.githubusercontent.com/96012488/187223216-136799d2-fd60-4303-b698-79f547fb841d.png)

 
 -- Thus, there is just 1 pizza which had both exclusions and extras.

### 9. What was the total volume of pizzas ordered for each hour of the day?

````sql
SELECT 
  DATEPART(HOUR,order_time) AS hour_of_the_day,
  COUNT(pizza_id) as pizza_count
 FROM #customer_orders 
 GROUP BY DATEPART(HOUR,order_time);
 ````
 
 **Answer**
 
 ![Pizza Metrics 9](https://user-images.githubusercontent.com/96012488/187223368-77faea8c-36cd-4671-975d-2a80b22af75c.png)



### 10. What was the volume of orders for each day of the week?

````sql
SELECT 
  FORMAT(DATEADD(DAY, 2, order_time),'dddd') AS day_of_week, -- add 2 to adjust 1st day of the week as Monday
  COUNT(order_id) AS order_count
FROM #customer_orders
GROUP BY FORMAT(DATEADD(DAY, 2, order_time),'dddd');
 ````
 
 **Answer**
 
 ![Pizza Metrics 10](https://user-images.githubusercontent.com/96012488/187223486-1cbf4ec2-200e-40b8-b0f2-d28be93652b3.png)


***Click [here](https://github.com/PriyaPalak/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/B.%20Runner%20and%20Customer%20Experience.md) for solution for B. Runner and Customer Experience!***
