# :pizza: Case Study #2 - Pizza Runner

## Solution - B. Runner and Customer Experience

### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

````sql
SELECT
  DATEPART(WEEK,registration_date) AS registration_week,
  COUNT(runner_id) AS runner_signups
FROM runners
GROUP BY DATEPART(WEEK,registration_date);
````

**Answer:**

![R C Exp 1](https://user-images.githubusercontent.com/96012488/189708057-7edfdacb-65ab-45a1-9f1e-fa27e338d9f8.png)


### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

````sql
SELECT
  r.runner_id,AVG((DATEDIFF(MINUTE,c.order_time,r.pickup_time))) AS average_arrival_time
FROM #customer_orders c
JOIN #runner_orders r
ON c.order_id = r.order_id
WHERE r.pickup_time != ''
GROUP BY r.runner_id;
````

**Answer:**

![R C Exp 2](https://user-images.githubusercontent.com/96012488/189708604-8243dcff-c93a-4072-8a97-0a76510f05ba.png)


### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

````sql
WITH cooking_time_cte AS
  (
  SELECT c.order_id, COUNT(pizza_id) AS pizza_count, DATEDIFF(MINUTE,c.order_time,r.pickup_time) AS cooking_time
  FROM #customer_orders c
  JOIN #runner_orders r
  ON c.order_id = r.order_id
  WHERE r.pickup_time != ''
  GROUP BY c.order_id, DATEDIFF(MINUTE,c.order_time,r.pickup_time)
  )
  SELECT 
    pizza_count, AVG(cooking_time) AS avg_preparation_time
  FROM cooking_time_cte
  GROUP BY pizza_count;
  ````
  
  **Answer:**
  
  ![R C Exp 3](https://user-images.githubusercontent.com/96012488/189709415-c7be5bd8-de5e-4c46-a5aa-8c837395a9f6.png)

  * Thus, on an average , it takes 12 minutes to prepare an order of 1 pizza.
  * It takes 18 minutes to prepare an order of 2 pizzas.
  * And, it takes 30 minutes to prepare an order of 3 pizzas.
  * So, there is a direct relationship between the number of pizzas oordered and the preparation time.


### 4.  What was the average distance travelled for each customer?

````sql
SELECT 
  c.customer_id, ROUND((AVG(r.distance)),2) AS avg_distance_travelled  --ROUND() to limit the value to 2 decimal places.
FROM #customer_orders c
JOIN #runner_orders r
ON c.order_id = r.order_id
WHERE r.distance != 0
GROUP BY c.customer_id;
````

````sql
SELECT 
  c.customer_id, CAST((AVG(r.distance))AS DECIMAL(5,2)) AS avg_distance_travelled -- CAST() can also do the job here but it keeps the insignificant zeros attached.
FROM #customer_orders c
JOIN #runner_orders r
ON c.order_id = r.order_id
WHERE r.distance != 0
GROUP BY c.customer_id;

 SELECT 
    c.customer_id, 
    FORMAT((AVG(r.distance)),'###.##') AS avg_distance_travelled  --FORMAT() formats a number to the specified format, though it changes the data type to string.
FROM #customer_orders c
JOIN #runner_orders r
ON c.order_id = r.order_id
WHERE r.distance != 0
GROUP BY c.customer_id;
````

````sql
--Any of the above three functions can be used to get the values with desired decimal places but each of them have their own functionalities.
````
  
  **Answer:**
  
  ![R C Exp 4](https://user-images.githubusercontent.com/96012488/189710374-db11bf2e-e306-46cb-acc1-6e69023f9be7.png)

### 5. What was the difference between the longest and shortest delivery times for all orders?

````sql
SELECT 
  order_id,duration
FROM #runner_orders
WHERE duration != 0;

  
SELECT 
  MAX(duration) - MIN(duration) AS delivery_time_difference
FROM #runner_orders
WHERE duration != 0;
````

**Answer:**

![R C Exp 5 1](https://user-images.githubusercontent.com/96012488/189710884-2b45fe71-c091-4fa1-9897-6219f854143e.png)

![R C Exp 5 2](https://user-images.githubusercontent.com/96012488/189710928-5edd9d09-7d05-426f-87ac-5bff05a11c9c.png)


### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

````sql
SELECT 
  runner_id,order_id , ROUND(AVG(distance/duration*60),2) AS avg_speed   
FROM #runner_orders
WHERE distance != 0
GROUP BY order_id,runner_id;
````

**Answer:**

![R C Exp 6](https://user-images.githubusercontent.com/96012488/189711222-c274b863-719a-4477-9206-7dd681a17048.png)

````sql
-- Distance is in KM and Duration is converted into hour. So, Speed is in KM/Hr.
````

* Runner A's average speed varies from 37.5 KM/Hr to 60 KM/Hr.
* Runner B's average speed varies from 35.1 KM/Hr to 93.6 KM/Hr.
* Runner C's average speed is 40 KM/Hr.

### 7. What is the successful delivery percentage for each runner?

````sql
SELECT 
  runner_id,
  100* SUM(CASE WHEN duration = 0 THEN 0
            ELSE 1 END)/COUNT(order_id) AS success_percentage
FROM #runner_orders
GROUP BY runner_id;
````


**Answer:**

![R C Exp 7](https://user-images.githubusercontent.com/96012488/189711446-3fb33474-ec30-436d-8884-bdb594d2555c.png)


* Runner 1 has 100% successful delivery.
* Runner 2 has 75% successful delivery.
* Runner 3 has 50% successful delivery.


***Click [here]() for Solution for Ingredient Optimisation!***
