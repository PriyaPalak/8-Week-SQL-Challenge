# 🍜 Case Study #3 : Foodie-Fi

## Business Task: 
Foodie-Fi is a Netflix-type startup but only with cooking shows. Danny created it with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data.The idea is to use data of this subscription based digital business to understand its customers' journey and behavior on the platform to make data driven decisions regarding future investments and new features.

## 📝 Solution A. Customer Journey

#### Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

**Approach:**
- I joined the 2 given tables `plans` and `subscriptions` and selected only the required sample of customers. 
- Additionally, I added a derived column 'days_to_upgarde' by calculating the difference between start_date of each plan and the immediately previous plan for each customer using the window function LAG(). 
 This column provides further insights into the customer's onboarding journey.
 
 ````sql
 SELECT 
  s.customer_id, p.plan_name,start_date,DATEDIFF(DAY,LAG(s.start_date,1) OVER(PARTITION BY customer_id ORDER BY start_date),
  s.start_date) AS days_to_upgrade
 FROM  subscriptions s
 JOIN plans p
 ON s.plan_id = p.plan_id
 WHERE s.customer_id IN (1,2,11,13,15,16,18,19);
   ````
   
   ![CS 3 1](https://user-images.githubusercontent.com/96012488/209833181-ef6368c2-012f-4ede-9bdb-c641f5d828da.png)

**Answer:**

- Customer 1 started off by taking a 7 day free trial, then upgraded to a basic monthly plan.
- Customer 2 also started off by taking a 7 day free trial plan, then upgraded to the pro annual plan.
- Customer 11 took a free 7 day trial and then churned.
- Customer 13 started off by taking a 7 day free trial, then upgraded to basic monthly plan. After 3 months, they upgraded to the pro monthly plan.
- Customer 15 took the trial, upgraded to the pro monthly plan and then cancelled the subscription during the second month.
- Customer 16 started off by taking a trial, then upgraded to a basic montly plan and then during the 5th month, further upgraded to the pro annual plan.
- Customer 18 took the trial and then upgraded to the pro monthly plan.
- Customer 19 started off by taking the trial, then upgarded to the pro monthly plan and after 2 months, further upgarded to the pro annual plan.


## :memo: Solution B. Data Analysis Questions

**1. How many customers has Foodie-Fi ever had?**

````sql
SELECT COUNT(DISTINCT customer_id) AS customer_count
FROM subscriptions;
````

**Answer:**

![CS 3 2](https://user-images.githubusercontent.com/96012488/209834172-7664fc75-6971-40e8-9c44-3de4d61a2b6e.png)

**2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value?**

````sql
SELECT 
  DATEPART(MONTH,start_date) AS Month, COUNT(DISTINCT customer_id) AS number_of_trials_taken 
  --I could use the MONTH() Function as well to get the month from the start_date
FROM subscriptions
WHERE plan_id = 0
GROUP BY DATEPART(MONTH,start_date);
````

**Answer:**

![CS 3 3](https://user-images.githubusercontent.com/96012488/209835132-372dd553-3d8d-4dbe-9e47-15bc1de7f27c.png)


**3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name?**

````sql
SELECT 
  start_date, plan_id
FROM subscriptions
WHERE YEAR(start_date) > '2020'
````
- After the year 2020, no customer has taken the 7 day free trial. Customers have either opted one of the 3 subscription plans or churned out.
- Breakdown by count of occurrence for each plan_name

````sql
SELECT 
   plan_name, COUNT(s.plan_id) AS count_of_occurrence
FROM subscriptions s
JOIN plans p
ON s.plan_id = p.plan_id
WHERE YEAR(start_date) > '2020'
GROUP BY plan_name
ORDER BY count_of_occurrence;
   ````
   
   **Answer:**
   
   ![CS 3 4](https://user-images.githubusercontent.com/96012488/209835606-71f30bee-63f9-4bc3-bbb1-07ebd00cfcf9.png)

- Thus, the breakdown shows that 71 customers have cancelled their subscriptions after the year 2020.
- 63 of the customers have upgraded to the pro-annual plan, 60 took the pro-monthly plan and 8 opted for the basic monthly plan.

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**

- Using JOIN

````sql
SELECT 
  plan_name,COUNT(DISTINCT customer_id) AS churned_customers,
ROUND(100*CAST(COUNT(DISTINCT customer_id) AS FLOAT)/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions),1) AS churn_percentage
FROM subscriptions s
JOIN plans p
ON s.plan_id = p.plan_id
WHERE s.plan_id = 4
GROUP BY plan_name;
````

- Using CTE: I have created a CTE for the two required counts total customers and churned customers and then calculated the churn percentage using the 2 counts.
````sql
WITH counts_cte AS
( 
 SELECT COUNT(DISTINCT customer_id) AS total_customers,
 SUM
 (CASE WHEN plan_id = 4 THEN 1
 ELSE 0 END ) AS churned_customers
 FROM subscriptions 
)
SELECT *, ROUND((100*churned_customers/total_customers),1) AS churn_percentage
FROM counts_cte;
````

**Answer:**

![CS3 5](https://user-images.githubusercontent.com/96012488/209836287-8d6cf86b-1f26-41e9-bf04-f5b738b2ae86.png)

**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number ?**

**Approach 1:** The idea is to create a CTE with the two dates for churn customers - trial_start_date and churn_start_date.
- I have created the cte by joining two derived tables, one containing the trial_start_date and another conatining churn_start_date.
- Then, use it to calculate the number of customers where the difference between the two dates is 7 which means that these customers churned straight after their free trial.

````sql
WITH dates_cte AS
(
SELECT A.customer_id,A.trial_date,B.churn_date
FROM
(SELECT customer_id,start_date AS trial_date
FROM subscriptions
WHERE plan_id = 0 AND customer_id IN (SELECT customer_id FROM subscriptions WHERE plan_id = 4)) AS A
JOIN
(SELECT customer_id,start_date AS churn_date
FROM subscriptions
WHERE plan_id =4) AS B
ON A.customer_id = B.customer_id
)
SELECT 
  COUNT(customer_id) AS churn_after_trial_count,
  100*COUNT(customer_id)/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions) AS churn_percentage
FROM dates_cte
WHERE DATEDIFF(DAY,trial_date,churn_date) = 7;
````

**Approach 2:** Using LEAD() Function 

````sql
WITH next_plan_cte AS
(
SELECT customer_id,plan_id,LEAD(plan_id,1) OVER(PARTITION BY customer_id ORDER BY plan_id) AS next_plan
FROM subscriptions
)
SELECT next_plan, COUNT(customer_id) AS churn_count,
100*COUNT(customer_id)/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions) AS churn_percentage
FROM next_plan_cte
WHERE next_plan = 4
AND plan_id = 0
GROUP BY next_plan;
````

**Answer:**

![CS 3 6](https://user-images.githubusercontent.com/96012488/209839979-9c80033d-f550-48e0-9c45-9b3e91e968c5.png)


**6. What is the number and percentage of customer plans after their initial free trial?**

**Approach:** Create a next_plan_cte containing the next_plan for each plan_id for each customer using the LEAD() function.
- LEAD() is a window function that gives the record following the current record. It takes 3 arguments - the field you wanna obtain, the number of records following the current record and the default value in case of no following record (by default it will show NULL)

````sql
WITH next_plan_cte AS
(
SELECT customer_id,plan_id,LEAD(plan_id,1) OVER(PARTITION BY customer_id ORDER BY plan_id) AS next_plan
FROM subscriptions
)
SELECT 
 next_plan,COUNT(customer_id) AS customer_count,
 ROUND(100*CAST(COUNT(customer_id) AS FLOAT)/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions),1) AS percentage
FROM next_plan_cte
WHERE plan_id = 0
GROUP BY next_plan;
````
**Answer:**

![CS 3 7](https://user-images.githubusercontent.com/96012488/209840118-2a432b21-74f0-4529-ba6e-de6a65a09242.png)


- 9.2% of customers have churned out staright after their free trial.
- The biggest chunk of customers i.e. 54.6 % opted for the basic monthly plan after their free trial, followed by 32.5 % who chose pro-monthly plan.
- Only 3.7 % of the customers went for the pro-annual plan.

**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31 ?**

````sql
WITH next_plan_cte AS
(
SELECT customer_id,p.plan_name,LEAD(p.plan_name,1) OVER (PARTITION BY customer_id ORDER BY s.start_date) AS next_plan 
-- Ordering by start_date ensures that the next_plan value is the next plan opted by the customer and not the next plan_id in serial order (which happens when you order by plan_id)
FROM subscriptions s
JOIN plans p
ON s.plan_id = p.plan_id
WHERE start_date <= '2020-12-31'
)
SELECT 
 plan_name, COUNT(customer_id) AS customer_count,
 ROUND(100*CONVERT(FLOAT,COUNT(customer_id))/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions),1) AS percentage
FROM next_plan_cte
WHERE next_plan IS NULL --This means this is the current plan of the customer (they haven't opted for any other plan or churned after this)
GROUP BY plan_name
ORDER BY customer_count;
````

**Answer:**

![CS 3 8](https://user-images.githubusercontent.com/96012488/209840678-a6d532f9-8afc-4c2d-be1b-8a87799c9061.png)


**8. How many customers have upgraded to an annual plan in 2020?**

````sql
SELECT 
 COUNT(DISTINCT customer_id) AS customer_count
FROM subscriptions
WHERE plan_id = 3
AND start_date <= '2020-12-31'
````

**Answer:**

![CS 3 9](https://user-images.githubusercontent.com/96012488/209840852-9bc09405-2b55-435d-9ce0-04c34eadffb2.png)

**9. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?**

**Approach:** Joining the 'subscriptions' table with annual_plan_day_cte for rows where plan_id = 0. 
- This will return the trial_start_date that will be the joining day and the annual_plan_day in the combined table.
- Then, calculate the average of difference between the 2 dates.

````sql
WITH annual_plan_day_cte AS
(
SELECT customer_id,start_date AS annual_plan_day
FROM subscriptions
WHERE plan_id = 3
)
SELECT 
 ROUND(AVG(CONVERT(FLOAT,DATEDIFF(DAY,start_date,annual_plan_day))),2) AS avg_days
FROM subscriptions s
JOIN annual_plan_day_cte a
ON s.customer_id = a.customer_id
WHERE plan_id = 0;
````
**Answer:**

![CS 3 11](https://user-images.githubusercontent.com/96012488/209949050-a50902e2-5fe5-46a1-9483-5916738fbf0b.png)

![CS 3 10](https://user-images.githubusercontent.com/96012488/209949082-e4f7bf44-725d-45d3-abde-829eb3ceba37.png)


**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc) ?**

- So, we want to break down the number of days it takes for a customer to upgrade to an annual plan into 30 days periods and we have to make 12 such periods.
- Then, we also need to calculate the count of customers for each period.

	**Approach:** We create 3 CTEs. First is trial_date_cte, Second is annual_plan_date_cte. 
 - We use these two CTEs to create a third CTE 'buckets' in which we divide the days_of_upgrade  for different customers into 12 buckets, each of 30 days. 
	- Next, we use CASE WHEN Statement to get the customer count for each bucket.

````sql
WITH trial_date_cte AS
(
SELECT customer_id, start_date AS trial_date
FROM subscriptions
WHERE plan_id = 0
),
annual_plan_day_cte AS
(
SELECT customer_id, start_date AS annual_plan_date
FROM subscriptions
WHERE plan_id = 3
),
buckets AS
(
SELECT a.customer_id,
DATEDIFF(DAY,trial_date,annual_plan_date)/30 + 1 AS bucket 
-- Dividing the days difference by 30 returns the integer part of the quotient that is 1 less than the bucket it is falling in. 
-- Adding 1 returns the bucket number the days difference falls in.
FROM trial_date_cte t
JOIN annual_plan_day_cte a
ON t.customer_id = a.customer_id
)
SELECT bucket,
 CASE WHEN bucket = 1 THEN CONCAT(bucket-1,' - ',bucket*30,' days')
 ELSE CONCAT((bucket-1)*30 + 1 , ' - ', bucket*30,' days')
 END AS avg_days_to_upgrade,
 COUNT(customer_id) AS customers
FROM buckets
GROUP BY bucket;
 ````
 **Answer:**
 
 ![CS 3 12](https://user-images.githubusercontent.com/96012488/209950373-c521fe34-55de-4f39-872b-ecdcd82236ca.png)
 
 **11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**
 
 ````sql
WITH next_plan_cte AS
(
SELECT customer_id,start_date,plan_id, LEAD(plan_id,1) OVER(PARTITION BY customer_id ORDER BY start_date) AS next_plan
FROM subscriptions
WHERE start_date <= '2020-12-31'
)
SELECT 
  COUNT(customer_id) AS customer_count
FROM next_plan_cte
WHERE plan_id = 2
AND next_plan = 1;
 ````
**Answer:**

![CS 3 13](https://user-images.githubusercontent.com/96012488/209950531-2742d2d0-968a-4801-84ae-47675be5bf73.png)

- Thus, no customer downgraded from a pro-monthly to a basic monthly in 2020.


