# :pizza: Case Study #2 - Pizza Runner

## 📝 Solution - D. Pricings and Ratings

#### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes how much money has Pizza Runner made so far if there are no delivery fees?

````sql
WITH revenue_cte AS
(	
SELECT 
   p.pizza_name,
   CASE WHEN c.pizza_id = 1 THEN 12*COUNT(c.pizza_id)
   ELSE 10*COUNT(c.pizza_id) 
   END AS Revenue
FROM #customer_orders c
JOIN #pizza_names p
ON c.pizza_id = p.pizza_id
JOIN #runner_orders r
ON c.order_id = r.order_id
WHERE r.distance != 0
GROUP BY c.pizza_id,p.pizza_name
)
SELECT 
  SUM(Revenue) AS Total_Revenue
FROM revenue_cte;
````

**Answer:**

![Screenshot 2022-12-31 170403](https://user-images.githubusercontent.com/96012488/210135187-d1025a35-837d-44c1-895e-214819617cb0.png)

#### 2. What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra.

**Approach:**

- I have created two CTEs, one for the revenue for each kind of pizzas without any other charges and another for the extras charges for each kind of pizza.
- Then, I have joined the two CTEs on 'pizza_name' column and added the Revenue without any other charges and the extra charges.

````sql
WITH revenue_cte AS
(
SELECT 
  p.pizza_name,
  CASE WHEN c.pizza_id = 1 THEN 12*COUNT(c.pizza_id)
  ELSE 10*COUNT(c.pizza_id) 
  END AS Revenue
FROM #customer_orders c
JOIN #pizza_names p
ON c.pizza_id = p.pizza_id
JOIN #runner_orders r
ON c.order_id = r.order_id
WHERE r.distance != 0
GROUP BY c.pizza_id,p.pizza_name
)
, extras_charges_cte AS
(
SELECT 
  p.pizza_name,COUNT(re.extras) AS extras_charges
FROM #records_with_extras re
JOIN #customer_orders c
ON c.record_id = re.record_id
LEFT JOIN #runner_orders r
ON c.order_id = r.order_id
JOIN #pizza_names p
ON c.pizza_id = p.pizza_id
WHERE r.distance != 0
GROUP BY p.pizza_name
)
SELECT 
  (SUM(Revenue) + SUM(extras_charges)) AS Total_Revenue_with_extras_charges
FROM revenue_cte r
JOIN extras_charges_cte e
ON r.pizza_name = e.pizza_name
````

**Answer:**

![Screenshot 2022-12-31 170756](https://user-images.githubusercontent.com/96012488/210135312-e3f34f40-eeed-4e43-8e3f-88049807d203.png)

#### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

**Approach:**
- The ratings table should contain 4 columns: runner_id,order_id,customer_id and the rating.

````sql
-- Creating table `ratings`

DROP TABLE IF EXISTS ratings;
CREATE TABLE ratings
(
runner_id INT,
order_id INT,
customer_id INT,
rating INT 
);


-- Inserting values

INSERT INTO ratings(runner_id,order_id,customer_id,rating)
VALUES  (1,1,101,5),
        (1,2,101,4),
        (1,3,102,5),
        (2,4,103,4),
        (3,5,104,5),
        (2,7,105,4),
        (2,8,102,5),
        (1,10,104,3);
````

**Answer:**

![Screenshot 2022-12-31 171155](https://user-images.githubusercontent.com/96012488/210135416-9853f23c-4ee6-48c6-80e0-58a7be2c2e10.png)

#### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
      
-  **customer_id**
-  **order_id
-  **runner_id**
-  **rating
-  **order_time**
-  **pickup_time**
-  **Time between order and pickup**
-  **Delivery duration**
-  **Average speed**
-  **Total number of pizzas**

````sql
SELECT 
  c.customer_id,
  c.order_id,
  r.runner_id,
  ra.rating,
  c.order_time,
  r.pickup_time, 
  DATEDIFF(MINUTE,c.order_time,r.pickup_time) AS Time_between,
  r.duration,
  ROUND((r.distance/r.duration*60),2)AS average_speed,
  COUNT(c.pizza_id) AS pizza_count
FROM #customer_orders c
JOIN #runner_orders r
ON c.order_id =r.order_id
JOIN ratings ra
ON c.order_id = ra.order_id
WHERE r.duration != 0
GROUP BY c.order_id,c.customer_id,r.runner_id,ra.rating,c.order_time,r.pickup_time,r.duration,DATEDIFF(MINUTE,c.order_time,r.pickup_time),r.duration,(r.distance/r.duration*60)
````
**Answer:**

![Screenshot 2022-12-31 171827](https://user-images.githubusercontent.com/96012488/210135616-616268ca-bd59-4513-adcd-008d9568bc41.png)

#### 5.  If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

**Approach:**
- I have created two CTEs. One for the revenue per order per pizza_id and another for the overall delivery fee. 
- Then, I subtracted the overall delivery fee from the revenue to get the net profit.

````sql
WITH revenue_cte AS
(	
SELECT 
  c.order_id,
  CASE WHEN c.pizza_id = 1 THEN 12*COUNT(c.pizza_id)
  ELSE 10*COUNT(c.pizza_id) 
  END AS Revenue
FROM #customer_orders c
JOIN #runner_orders r
ON c.order_id = r.order_id
WHERE r.distance != 0
GROUP BY c.order_id,c.pizza_id,r.distance
)
,delivery_cte AS
(
SELECT 
  SUM((0.3*distance)) AS delivery_fee
FROM #runner_orders
WHERE distance != 0
)
SELECT 
  (SUM(Revenue) - delivery_fee) AS net_profit
FROM revenue_cte, delivery_cte
GROUP BY delivery_fee;
````
	
- Here, I have not joined the two ctes. I have simply selected the required columns from the two ctes separating them with a comma in the FROM clause.

#### Bonus Question: If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu? 

- The addition of a newSupreme Pizza to the pizza_runner's menu will affect the design of two tables in the database, `pizza_names` and `pizza_recipes`.

````sql
INSERT INTO pizza_names (pizza_id,pizza_name)
VALUES(3,'Supreme');

INSERT INTO pizza_recipes (pizza_id, toppings)
VALUES(3, '1, 2, 3, 4, 5, 6, 7, 8 ,9, 10, 11, 12');

-- Let's see how the new addition has modified the two tables.

SELECT * FROM pizza_names;
SELECT * FROM pizza_recipes;
````

**Result:**

| Before|After|
|---|---|
|![Screenshot 2022-12-31 172641](https://user-images.githubusercontent.com/96012488/210135898-baad0c26-2827-40f9-872a-c77807a4a21f.png)|![Screenshot 2022-12-31 172703](https://user-images.githubusercontent.com/96012488/210135935-b4206100-dc7b-4c4b-bb96-d0a219503a21.png)|
|![Screenshot 2022-12-31 172723](https://user-images.githubusercontent.com/96012488/210135951-8a5a1f85-07bc-4e3f-b0e8-4f792a93b626.png)|![Screenshot 2022-12-31 172745](https://user-images.githubusercontent.com/96012488/210135957-cef7ba69-f8e5-441b-904f-3a5a4e38136e.png)|

***

***The 2nd Challenge comes to an end here. And I must say, I got to learn so many new things in the process of solving this case study😄!***

***

***Click [here](https://github.com/PriyaPalak/8-Week-SQL-Challenge/tree/main/Case%20Study%20%233%20-%20Foodie-Fi) to view the solution for 3rd challenge - Foodie fi!***

