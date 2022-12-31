# :pizza: Case Study #2 - Pizza Runner

## 📝 Solution C. Ingredient Optimization

*In this section, I first got my hands dirty with solving each query in bits and pieces. Then, I realised there are certain results(tables) which are repeatedly used in many queries. So, I created them as temporary tables.*
  
*While solving the queries, I also realised that data type of certain TEXT columns needs to be converted to Character type (Nvarchar) so that the various String Functions (like STRING_SPLIT(), STRING_AGG(), and CONCAT_WS()) can be performed on them.*

### Data Type Transformations to be able to perform certain operations

- 1. Creating a temporary table `#pizza_recipes` and changing the data type of `toppings` from **TEXT** to **NVARCHAR(100)** to be able to perform STRING_SPLIT() on it.

````sql
SELECT pizza_id , toppings
INTO #pizza_recipes
FROM pizza_recipes;

ALTER TABLE #pizza_recipes
ALTER COLUMN toppings NVARCHAR(100);
````
- 2. Creating a temporary table `#pizza_toppings` and changing the data type of `topping_name` from **TEXT** to **NVARCHAR(100)** to be able to perform STRING_AGG() on it.

````sql
SELECT topping_id,topping_name
INTO #pizza_toppings
FROM pizza_toppings;

ALTER TABLE #pizza_toppings
ALTER COLUMN topping_name NVARCHAR(100);
````
- 3. Creating a temporary table `#pizza_names` and changing the data type of `pizza_name` from **TEXT** to **NVARCHAR(100)** to be able to perform CONCAT_WS() on it.

````sql
SELECT * 
INTO #pizza_names
FROM pizza_names;

ALTER TABLE #pizza_names
ALTER COLUMN pizza_name NVARCHAR(100);
````


*We need to create a few temporary tables with more simple and clear formats using the given tables, so that we can easily answer the given questions.*

### Temporary tables used repeatedly

**1.**  Creating the `#pizzas_with_toppings` table (*It contains pizza ids with separated topping ids and separated topping names with one row for each topping name*).
- STRING_SPLIT() is a user_defined table_valued function that splits a string array into multiple rows of substrings based on the specified separator.
- Input_string (1st argument for this function) should be of the type NVARCHAR, VARCHAR, NCHAR or CHAR.

````sql
SELECT	
 r.pizza_id,TRIM(t.value) AS topping_id,pt.topping_name  
--TRIM() removes any leading or trailing spaces and makes sure the substrings are aligned properly.
INTO #pizzas_with_toppings
FROM #pizza_recipes as r
CROSS APPLY STRING_SPLIT(r.toppings, ',') as t
JOIN #pizza_toppings pt
ON TRIM(t.value) = pt.topping_id;
````
**Before:**

|pizza_recipes|pizza_toppings|
|---|---|
|![Screenshot 2022-12-31 140551](https://user-images.githubusercontent.com/96012488/210130640-70a025ec-e642-4831-a49c-5b08d0c8645d.png)|![Screenshot 2022-12-31 140635](https://user-images.githubusercontent.com/96012488/210130657-104f4396-0d93-4a6f-8461-31ab08e558ac.png)|

**After:   `pizzas_with_toppings`**

![Screenshot 2022-12-31 140130](https://user-images.githubusercontent.com/96012488/210130552-f9b319c6-5aa2-4182-866c-4d541d10da44.png)

**2.**  Adding a column `record_id` to the table `#customer_orders` to select each pizza in an order individually, using IDENTITY() function.
- IDENTITY() is a function used to create auto-incrementing columns, i.e., the value of the column increases automatically everytime a new row is inserted into the table.
- It has two optional arguments Seed and Step, default values for both is 1.

````sql
ALTER TABLE #customer_orders
ADD record_id INT IDENTITY(1,1)
````
| Before|After|
|---|---|
|![Screenshot 2022-12-31 141736](https://user-images.githubusercontent.com/96012488/210130906-5a5d2ffc-5b51-4841-a827-3d3231a68564.png)|![Screenshot 2022-12-31 141930](https://user-images.githubusercontent.com/96012488/210130942-7d14e085-b252-427a-9ddc-6d44eeb52797.png)|



**3.** Creating the table `records_with_exclusions` (*It contains record_ids and exclusions for each record*).

````sql
SELECT 
 record_id, value AS exclusions
INTO #records_with_exclusions
FROM #customer_orders c
CROSS APPLY STRING_SPLIT(c.exclusions,',') 
WHERE exclusions != '';
````

![Screenshot 2022-12-31 142124](https://user-images.githubusercontent.com/96012488/210130980-c378d835-cf1a-4b11-b0ca-7dcf5bbcfca5.png)


**4.**  Creating the table` records_with_extras` (*It contains record_ids and extras for each record*)

````sql
SELECT 
 record_id, value AS extras
INTO #records_with_extras
FROM #customer_orders c
CROSS APPLY STRING_SPLIT(c.extras,',') 
WHERE extras != '';
````

![Screenshot 2022-12-31 142209](https://user-images.githubusercontent.com/96012488/210130991-f06f5273-ed24-41d8-bdaf-5ca702122673.png)


***Let's answer the Questions now!***

#### 1. What are the standard ingredients for each pizza?

**Approach:**
- We will use the newly created temporary table #pizzasa_with_toppings which contains pizza ids with separated topping ids and separated topping names.
- Using this table, we will retrieve a string array consisting of topping names for each of the Pizza id.
- To create a string array of topping names, we need to perform STRING_AGG() on it.
- STRING_AGG() is a user-defined function that concatenates rows of strings into a single string, separated by a specified separator.

````sql
SELECT 
 pizza_id, STRING_AGG(topping_name,', ') AS Toppings
FROM #pizzas_with_toppings
GROUP BY pizza_id;
````

**Answer:**

![Screenshot 2022-12-31 142652](https://user-images.githubusercontent.com/96012488/210131121-00cfd5d9-85c5-4a84-aed7-08196c004ffa.png)


#### 2. What was the most commonly added extra?  

**Approach:**
- We will use the temporary table #records_with_extras and join it with #pizza_toppings for topping_name.

````sql
SELECT 
 pt.topping_name, COUNT(r.record_id) AS number_of_addition
FROM #records_with_extras r
JOIN #pizza_toppings pt
ON r.extras = pt.topping_id
GROUP BY pt.topping_name;
````
**Answer:**

![Screenshot 2022-12-31 142804](https://user-images.githubusercontent.com/96012488/210131142-d6bd8fbe-0740-490d-af51-3094b6fbff7f.png)

- Bacon was the most commonly added extra.

#### 3. What was the most common exclusion?  

**Approach:**
- We will use the temporary table #records_with_exclusions and join it with #pizza_topping for topping_name.

````sql
SELECT 
 pt.topping_name, COUNT(r.record_id) AS number_of_removals
FROM #records_with_exclusions r
JOIN #pizza_toppings pt
ON r.exclusions = pt.topping_id
GROUP BY pt.topping_name;
````

**Answer:**

![Screenshot 2022-12-31 142838](https://user-images.githubusercontent.com/96012488/210131160-a796435c-0583-4bb7-9572-50f42de2a621.png)

- Cheese was the most common exclusion.


#### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
####    Meat Lovers
####    Meat Lovers - Exclude Beef
####    Meat Lovers - Extra Bacon
####    Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers   

**Approach:**
- Here, we are using 3 ctes: exclusions_cte, extras_cte and union_cte. Then we join these with #customer_orders and pizza_names to get the desired result.
- exclusions_cte contains record_id and the exclusions in the 'Exclude ... ' format.
- extras_cte contains record_id and the extras in the 'Extra ... ' format.
- union_cte contains the union of the two ctes.

- And then, we have used CONCAT_WS() function to obtain the pizza_name and optimisation in the required format for each order item.
- CONCAT_WS() function concatenates two or more strings into one string with a separator. CONCAT_WS() means concatenate with separator. Again, the input_string should be NVARCHAR, VARCHAR etc.

````sql
WITH exclusions_cte AS
(
SELECT 
 r.record_id, 'Exclude ' + STRING_AGG(pt.topping_name, ', ') AS optimisation
FROM #records_with_exclusions r
JOIN #pizza_toppings pt
ON r.exclusions = pt.topping_id
GROUP BY r.record_id
),

extras_cte AS

(
SELECT 
 r.record_id, 'Extra ' + STRING_AGG(pt.topping_name, ', ') AS optimisation
FROM #records_with_extras r
JOIN #pizza_toppings pt
ON r.extras = pt.topping_id
GROUP BY r.record_id
),

union_cte AS
(
SELECT * FROM exclusions_cte
UNION
SELECT * FROM extras_cte
) 

SELECT 
 c.record_id,
 CONCAT_WS('-', p.pizza_name, STRING_AGG(optimisation, '-')) AS desired format
FROM #customer_orders c
JOIN #pizza_names p
ON c.pizza_id = p.pizza_id
LEFT JOIN union_cte cte
ON c.record_id = cte.record_id
GROUP BY c.record_id,p.pizza_name
ORDER BY c.record_id;
````
**Answer:**

![Screenshot 2022-12-31 143135](https://user-images.githubusercontent.com/96012488/210131210-e459e7b5-e811-4ee1-970d-4ca72554e102.png)


#### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
#### For example: "Meat Lovers: 2xBacon, Beef, ... , Salami" 

**Approach:**
- Here, we create a cte by joining three tables $customer_orders, #pizza_names and #pizzas_with_toppings so that we have each of the topping name for each of the record.
- Then, we create a case based column 'toppings' where if the ingredient is an extra for a specific record, it should have 2x in front of it. Else, just the topping name.
- And, by the WHERE clause we make sure that if an ingredient is excluded for a record, it should not show up in the toppings column.
- At the end, CONCAT_WS() function is used to get the list of ingredients in the desired format for each order item.

````sql
WITH ingredients_cte AS
(
SELECT
 c.record_id, p.pizza_name,
 CASE WHEN pt.topping_id
 IN (SELECT extras FROM #records_with_extras r WHERE c.record_id = r.record_id) 
 --Here, c.record_id = r.record_id to make sure we are giving 2x to extras for the specific records and not all the records
 THEN '2x ' + pt.topping_name
 ELSE pt.topping_name
 END AS toppings
FROM #customer_orders c
JOIN #pizza_names p
ON c.pizza_id = p.pizza_id
JOIN #pizzas_with_toppings pt
ON c.pizza_id = pt.pizza_id
WHERE pt.topping_id NOT IN (SELECT exclusions FROM #records_with_exclusions r WHERE c.record_id = r.record_id) 
--Here, WHERE c.record_id = r.record_id is like there are some records which have exclusions and for those records, the topping_id should not be in the record's excluded list of toppings.
)
SELECT 
 record_id, CONCAT_WS(' :', pizza_name, STRING_AGG(toppings, ', ')) AS ingredients_list
FROM ingredients_cte
GROUP BY record_id,pizza_name
ORDER BY record_id;
````
- *Note: In this query, we have used the same alias 'r' for both the tables #record_with_extras and #records_with_exclusions but in different subqueries and that works just fine.*

**Answer:**

![Screenshot 2022-12-31 143259](https://user-images.githubusercontent.com/96012488/210131239-0731b809-6682-426d-b7cf-0aaa2099cf38.png)


#### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first? 

**Approach:**
- Here, the approach for cte is very similar to the previous question. 
- But to include only the delivered pizzas, I have added another table #runner_orders and added a WHERE clause with pickup_time not equal to ''.
- The case based column 'times_used' is such that for an extra, the value is 2, for an exclusion, the value is 0 and for a normal ingredient, the value is 1.
- At the end , we have just summed up the 'times_used' column , grouping by 'ingredient' to  get the total_count of number of times an ingredient is used.

````sql
WITH count_cte AS
(
SELECT
 record_id,topping_name AS ingredient,
 CASE -- if extra ingredient add 2
 WHEN pt.topping_id 
 IN (SELECT extras FROM #records_with_extras r WHERE c.record_id = r.record_id) 
 THEN 2 
 -- if exclusion ingredient add 0
 WHEN pt.topping_id
 IN (SELECT exclusions FROM #records_with_exclusions r WHERE c.record_id = r.record_id)
 THEN 0
 -- if normal ingredient add 1
 ELSE 1
 END AS times_used
FROM #customer_orders c
JOIN #pizzas_with_toppings pt
ON c.pizza_id = pt.pizza_id
LEFT JOIN #runner_orders r
ON c.order_id = r.order_id
WHERE r.pickup_time != ''
)
SELECT 
 ingredient, SUM(times_used) AS total_count
FROM count_cte
GROUP BY ingredient
ORDER BY total_count DESC;
````

**Answer:**

![Screenshot 2022-12-31 143351](https://user-images.githubusercontent.com/96012488/210131260-ed356e25-e03d-414a-abc4-fd7f775b48d5.png)

- Thus, **Mushroom** is the most frequently used ingredient.

### Learnings  

-  Various new STRING Manipulation Functions like STRING_SPLIT(), STRING_AGG() , CONCAT with + and CONCAT_WS()
-  The tables provided were not in the format that you can directly fetch the results from. I had to create many new tables, by combining the existing tables, in the desired formats to get the desired results.
-  It was not easy to picture the required CTEs in mind as the designs were quite complex involving many tables and newly created temporary tables. 
So, my approach was this: first what columns do I need, which tables will i fetch those columns from and then how to join all those tables and then any required conditions.


