# :hammer: Handling Data Quality Issues

Danny has already mentioned in the Case Study that some of the tables provided have some data quality issues.
 
 1. The columns "exclusions" and "extras" in the table "customer_orders" have null values.
 2. The columns "pickup_time", "distance" , "duration" and "cancellation" in the table "runner_orders" have null values.
 3. Both the tables "customer_orders" and "runner_orders" have issues relating to data type of the columns.

 So, first we need to deal with these data quality issues before moving on to querying and solving the Case Study questions.

 **Approach:** 
 - We first create a temporary table with the same table_name and make the required changes in the values of the columns (e.g. getting rid of nulls and NULL) 
 - Then we change the datatype of the columns to be able to perform the desired operations. And then we'll use the temporary tables for all our queries.
  
### 1. Cleaning the table "customer_orders"

**Steps:**
  -  exclusions: remove NULL and nulls and replace with ''
  -  extras: remove NULL and nulls and replace with ''

````sql
SELECT 
    order_id,customer_id,pizza_id,
    CASE 
      WHEN exclusions IS NULL OR exclusions LIKE 'null' THEN ''
      ELSE exclusions
      END AS exclusions,
    CASE
      WHEN extras IS NULL OR extras LIKE 'null' THEN ''
      ELSE extras 
      END AS extras,
    order_time
INTO #customer_orders -- #customer_orders is a temporary table which is stored somewhere in a database.
FROM customer_orders;
````
 |Before|After|
 |-------|--------|
 |![Screenshot 2022-12-29 203413](https://user-images.githubusercontent.com/96012488/209971744-266c04ae-e1cf-41b0-b58a-65dc99d4b45c.png)|![CS 2 2](https://user-images.githubusercontent.com/96012488/209969596-5ef1f925-73dc-4554-b342-240b4348350f.png)


### 2. Cleaning the table "runner_orders"

**Step: Cleaning the values**
- pickup_time: remove nulls and replace with ''
- distance: remove km and replace nulls with ''
- duration: remove minutes and replace nulls with ''
- cancellation: remove NULL and nulls and replace with ''
 
 
````sql
SELECT order_id, runner_id,
  CASE 
    WHEN pickup_time LIKE 'null' THEN ''
    ELSE pickup_time
    END AS pickup_time,
  CASE 
    WHEN distance LIKE 'null' THEN ''
    WHEN distance LIKE '%km' THEN TRIM( 'km' FROM distance)
    ELSE distance 
    END AS distance,
  CASE 
    WHEN duration LIKE 'null' THEN ''
    WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
    WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
    WHEN duration LIKE '%minutes' THEN TRIM( 'minutes' FROM duration)
    ELSE duration
    END AS duration,
  CASE 
    WHEN cancellation IS NULL OR cancellation LIKE 'null' THEN ''
    ELSE cancellation
    END AS cancellation
INTO #runner_orders-- #runner_orders is a temporary table which is stored somewhere in a database.
FROM runner_orders;
````

|Before|After|
|---|---|
![Screenshot 2022-12-29 203139](https://user-images.githubusercontent.com/96012488/209972858-a68c0e17-e479-42f0-8309-021de2656589.png)|![Screenshot 2022-12-29 204103](https://user-images.githubusercontent.com/96012488/209972778-42785ab3-4288-417a-a18c-8c3b05b4253d.png)


 **Step: Changing the Dataypes**
- pickup_time: VARCHAR to DATETIME
- distance : VARCHAR to FLOAT
- duration: VARCHAR to INT

````sql
ALTER TABLE #runner_orders
ALTER COLUMN pickup_time DATETIME;

ALTER TABLE #runner_orders
ALTER COLUMN distance FLOAT;

ALTER TABLE #runner_orders
ALTER COLUMN duration INT;
````

- Now, the dataset is ready to go ahead with!

*Click **[here](https://github.com/PriyaPalak/8-Week-SQL-Challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/A.%20Pizza%20Metrics%20.md)** for solution to the first section of case study questions on Pizza metrics!*

