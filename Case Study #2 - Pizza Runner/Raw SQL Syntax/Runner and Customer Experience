-

 --This file covers Solutions to Questions regarding Runner and Customer Experience.
 

  /* Q1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)  */

  SELECT DATEPART(WEEK,registration_date) AS registration_week,
  COUNT(runner_id) AS runner_signups
  FROM runners
  GROUP BY DATEPART(WEEK,registration_date); --Use image from Katie Huang's github


  /* Q2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?  */

  SELECT r.runner_id,AVG((DATEDIFF(MINUTE,c.order_time,r.pickup_time))) AS average_arrival_time
  FROM #customer_orders c
  JOIN #runner_orders r
  ON c.order_id = r.order_id
  WHERE r.pickup_time != ''
  GROUP BY r.runner_id;
  

  /* Q3. Is there any relationship between the number of pizzas and how long the order takes to prepare?  */


  WITH cooking_time_cte AS
  (
  SELECT c.order_id, COUNT(pizza_id) AS pizza_count, DATEDIFF(MINUTE,c.order_time,r.pickup_time) AS cooking_time
  FROM #customer_orders c
  JOIN #runner_orders r
  ON c.order_id = r.order_id
  WHERE r.pickup_time != ''
  GROUP BY c.order_id, DATEDIFF(MINUTE,c.order_time,r.pickup_time)
  )
  SELECT pizza_count, AVG(cooking_time) AS avg_preparation_time
  FROM cooking_time_cte
  GROUP BY pizza_count;


  -- Thus, on an average , it takes 12 minutes to prepare an order of 1 pizza.
  -- It takes 18 minutes to prepare an order of 2 pizzas.
  -- And, it takes 30 minutes to prepare an order of 3 pizzas.
  -- So, there is a direct relationship between the number of pizzas oordered and the preparation time.



  /*  Q4. What was the average distance travelled for each customer?  */


  SELECT c.customer_id, ROUND((AVG(r.distance)),2) AS avg_distance_travelled  --ROUND() to limit the value to 2 decimal places.
  FROM #customer_orders c
  JOIN #runner_orders r
  ON c.order_id = r.order_id
  WHERE r.distance != 0
  GROUP BY c.customer_id;

  SELECT c.customer_id, CAST((AVG(r.distance))AS DECIMAL(5,2)) AS avg_distance_travelled -- CAST() can also do the job here but it keeps the insignificant zeros attached.
  FROM #customer_orders c
  JOIN #runner_orders r
  ON c.order_id = r.order_id
  WHERE r.distance != 0
  GROUP BY c.customer_id;

  SELECT c.customer_id, 
  FORMAT((AVG(r.distance)),'###.##') AS avg_distance_travelled  --FORMAT() formats a number to the specified format, though it changes the data type to string.
  FROM #customer_orders c
  JOIN #runner_orders r
  ON c.order_id = r.order_id
  WHERE r.distance != 0
  GROUP BY c.customer_id;


  /* Q5. What was the difference between the longest and shortest delivery times for all orders?  */

  SELECT order_id,duration
  FROM #runner_orders
  WHERE duration != 0;

  SELECT MAX(duration) - MIN(duration) AS delivery_time_difference
  FROM #runner_orders
  WHERE duration != 0;


  /* Q6. What was the average speed for each runner for each delivery and do you notice any trend for these values?  */


  SELECT runner_id,order_id , ROUND(AVG(distance/duration*60),2) AS avg_speed   
  FROM #runner_orders
  WHERE distance != 0
  GROUP BY order_id,runner_id;

  -- Distance is in KM and Duration is converted into hour. So, Speed is in KM/Hr.

  -- Runner A's average speed varies from 37.5 KM/Hr to 60 KM/Hr.
  -- Runner B's average speed varies from 35.1 KM/Hr to 93.6 KM/Hr.
  -- Runner C's average speed is 40 KM/Hr.

  /* Q7. What is the successful delivery percentage for each runner?  */

  SELECT runner_id,
  100* SUM(CASE WHEN duration = 0 THEN 0
            ELSE 1 END)/COUNT(order_id) AS success_percentage
  FROM #runner_orders
  GROUP BY runner_id;

  -- Runner 1 has 100% successful delivery.
  -- Runner 2 has 75% successful delivery.
  -- Runner 3 has 50% successful delivery.

  --(It does not make sense to calculate runner's delivery success here as the unsuccessful deliveries are based on order cancellations which are out of runner's control).


  ------------------------------------------------------------------------------------------------------------------------------------
