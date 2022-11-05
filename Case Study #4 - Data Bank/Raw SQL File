/* Case Study Questions */

 /*  A. Customer Nodes Exploration  */

 -- 1. How many unique nodes are there on the Data Bank system?

 SELECT COUNT(DISTINCT node_id) AS unique_node_count
 FROM customer_nodes;

 -- There are 5 unique nodes.

 -- 2. What is the number of nodes per region?

 SELECT region_id,COUNT(DISTINCT node_id) node_count
 FROM customer_nodes
 GROUP BY region_id;

 -- Each region has 5 unique nodes.

 -- 3. How many customers are allocated to each region?

 SELECT n.region_id,r.region_name,COUNT(DISTINCT customer_id) AS customer_count
 FROM customer_nodes n
 JOIN regions r
 ON n.region_id=r.region_id
 GROUP BY n.region_id,r.region_name
 ORDER BY n.region_id;

 -- 4. How many days on average are customers reallocated to a different node?

 SELECT AVG(DATEDIFF(day,start_date,end_date)) avg_reallocation_days
 FROM customer_nodes
 WHERE end_date NOT LIKE '9999-%';

 -- 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

 WITH diff_cte AS
 (
 SELECT n.region_id,r.region_name,customer_id,DATEDIFF(day,start_date,end_date) AS reallocation_days
 FROM customer_nodes n
 JOIN regions r
 ON n.region_id=r.region_id
 WHERE end_date NOT LIKE '9999-%'
 )
 SELECT DISTINCT region_id,region_name,
		PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY reallocation_days) OVER( PARTITION BY region_id) AS median,
		PERCENTILE_DISC(0.80) WITHIN GROUP (ORDER BY reallocation_days) OVER( PARTITION BY region_id) AS percentile_80th,
		PERCENTILE_DISC(0.95) WITHIN GROUP (ORDER BY reallocation_days) OVER( PARTITION BY region_id) AS percentile_95th
 FROM diff_cte;

 -- Here, I have used PERCENTILE_DISC() function to calculate the  various percentiles. 
 -- It is a window function that calculates the percentile based on a discrete distribution of the column values. The result is equal to a specific column value.
 -- There is another function called PERCENTILE_CONT() which calculates the percentile based on a continuous distribution of column values.


 /* B. Customer Transactions */

 -- 1. What is the unique count and total amount for each transaction type?

 SELECT txn_type,COUNT(txn_type) AS unique_count,SUM(txn_amount) AS total_amount
 FROM customer_transactions
 GROUP BY txn_type;



 -- 2. What is the average total historical deposit counts and amounts for all customers?

 WITH deposit_cte AS
 (
 SELECT customer_id,COUNT(txn_type) AS deposit_count,SUM(txn_amount) AS deposit_amount
 FROM customer_transactions
 WHERE txn_type = 'deposit'
 GROUP BY customer_id
 )
 SELECT AVG(deposit_count) AS avg_deposit_count,AVG(deposit_amount) AS avg_total_amount_deposited
 FROM deposit_cte;

 

 -- 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

 WITH months_cte AS
 (
 SELECT customer_id, DATENAME(month,txn_date) AS txn_month
 FROM customer_transactions
 ),
 counts_cte AS
 (
 SELECT t.customer_id,txn_month,SUM(CASE WHEN txn_type='deposit' THEN 1 ELSE 0 END) AS d,
        SUM(CASE WHEN txn_type='purchase' THEN 1 ELSE 0 END) AS p,
		SUM(CASE WHEN txn_type='withdrawal' THEN 1 ELSE 0 END) AS w
 FROM customer_transactions t
 JOIN months_cte
 ON months_cte.customer_id=t.customer_id
 GROUP BY t.customer_id,txn_month
 )
 SELECT txn_month,COUNT(customer_id) AS customer_count
 FROM counts_cte
 WHERE d>1 AND (p=1 OR w=1)
 GROUP BY txn_month
 ORDER BY txn_month;
 

-- 4. What is the closing balance for each customer at the end of the month?

WITH amounts_cte AS
(
 SELECT t.customer_id, DATENAME(month,txn_date) AS txn_month,
		SUM(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
 FROM customer_transactions t
 GROUP BY customer_id,DATENAME(month,txn_date)
 )
 SELECT customer_id,txn_month,
		SUM(transaction_amount) OVER (PARTITION BY customer_id ORDER BY txn_month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW ) AS closing_balance
 FROM amounts_cte
 ORDER  BY customer_id;

 -- 

 
 /* Data Allocation Challenge */
 
 /* Data Requirements */

 --1. Running Customer balance that includes the impact of each transaction


 WITH cte AS
 (
 SELECT t.customer_id,txn_date, DATENAME(month,txn_date) AS txn_month,txn_type,
		(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
 FROM customer_transactions t
 )
 SELECT *, SUM(transaction_amount) OVER (PARTITION BY customer_id ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_balance
 FROM cte
 ORDER BY customer_id;

 -- Running Total calculated using SUM() as a Window Function and defining the window as Current row and unbounded rows preceding the current row

 --2. Customer Balance at the end of each month


 WITH amounts_cte AS
(
 SELECT t.customer_id, DATENAME(month,txn_date) AS txn_month,
		SUM(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
 FROM customer_transactions t
 GROUP BY customer_id,DATENAME(month,txn_date)
 )
 SELECT customer_id,txn_month,
		SUM(transaction_amount) OVER (PARTITION BY customer_id ORDER BY txn_month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW ) AS closing_balance
 FROM amounts_cte
 ORDER  BY customer_id;

 --3. minimum, average and maximum values of the running balance for each customer

 WITH txnamount_cte AS
 (
 SELECT t.customer_id,txn_date, DATENAME(month,txn_date) AS txn_month,txn_type,
		(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
 FROM customer_transactions t
 )
 ,running_balance_cte AS
 (
 SELECT *, SUM(transaction_amount) OVER (PARTITION BY customer_id ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_balance
 FROM txnamount_cte
 )
 SELECT customer_id, MIN(running_balance) AS min_balance, AVG(running_balance) AS avg_balance, MAX(running_balance) AS max_balance
 FROM running_balance_cte
 GROUP BY customer_id
 ORDER BY customer_id;


 -- Data Storage Space Required in different options
 
 -- Option 1: data is allocated based off the amount of money at the end of the previous month

 -- Data_Allocated based on the closing balance of previous month

 WITH monthlyamounts_cte AS
(
 SELECT t.customer_id, DATENAME(month,txn_date) AS txn_month,DATEPART(month,txn_date) AS txn_month_id,
		SUM(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
 FROM customer_transactions t
 GROUP BY customer_id,DATENAME(month,txn_date),DATEPART(month,txn_date)
 )
 , closing_balance_cte AS
 (
 SELECT customer_id,txn_month,txn_month_id,
		SUM(transaction_amount) OVER (PARTITION BY customer_id ORDER BY txn_month_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW ) AS closing_balance
 FROM monthlyamounts_cte
 )
 ,prev_closing_balance_cte AS
 (
 SELECT *, LAG(closing_balance) OVER(PARTITION BY customer_id ORDER BY txn_month_id) AS prev_closing_balance
 FROM closing_balance_cte
 )
 SELECT *, CASE WHEN prev_closing_balance >= 0 THEN prev_closing_balance ELSE 0 END AS data_allocated
 FROM prev_closing_balance_cte;
 


 -- Total Data Required

  WITH monthlyamounts_cte AS
(
 SELECT t.customer_id, DATENAME(month,txn_date) AS txn_month,DATEPART(month,txn_date) AS txn_month_id,
		SUM(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
 FROM customer_transactions t
 GROUP BY customer_id,DATENAME(month,txn_date),DATEPART(month,txn_date)
 )
 , closing_balance_cte AS
 (
 SELECT customer_id,txn_month,txn_month_id,
		SUM(transaction_amount) OVER (PARTITION BY customer_id ORDER BY txn_month_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW ) AS closing_balance
 FROM monthlyamounts_cte
 )
 ,prev_closing_balance_cte AS
 (
 SELECT *, LAG(closing_balance) OVER(PARTITION BY customer_id ORDER BY txn_month_id) AS prev_closing_balance
 FROM closing_balance_cte
 )
 ,data_cte AS
 (
 SELECT *, CASE WHEN prev_closing_balance >= 0 THEN prev_closing_balance ELSE 0 END AS data_allocated
 FROM prev_closing_balance_cte
 )
 SELECT txn_month,txn_month_id,SUM(data_allocated) AS total_data_required
 FROM data_cte
 GROUP BY txn_month,txn_month_id
 ORDER BY txn_month_id;


 -- Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days

 -- Data_Allocated based on avg_running_balance per month
 
 WITH txnamount_cte AS
 (
 SELECT t.customer_id,txn_date, DATENAME(month,txn_date) AS txn_month,txn_type,DATEPART(month,txn_date) AS txn_month_id,
		(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
 FROM customer_transactions t
 )
 ,running_balance_cte AS
 (
 SELECT *, SUM(transaction_amount) OVER (PARTITION BY customer_id ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_balance
 FROM txnamount_cte
 )
 ,avg_running_balance_cte AS
 (
 SELECT DISTINCT customer_id,txn_month,txn_month_id,AVG(running_balance) OVER(PARTITION BY customer_id,txn_month ORDER BY txn_month) AS avg_running_balance
 FROM running_balance_cte
 )
 ,prev_avg_rb_cte AS
 (
 SELECT *, LAG(avg_running_balance) OVER(PARTITION BY customer_id ORDER BY txn_month_id) AS prev_avg_rb
 FROM avg_running_balance_cte
 )
 SELECT *, CASE WHEN prev_avg_rb >= 0 THEN prev_avg_rb ELSE 0 END AS data_allocated
 FROM prev_avg_rb_cte
 ORDER BY customer_id;

 --Total Data Required

 WITH txnamount_cte AS
 (
 SELECT t.customer_id,txn_date, DATENAME(month,txn_date) AS txn_month,txn_type,DATEPART(month,txn_date) AS txn_month_id,
		(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
 FROM customer_transactions t
 )
 ,running_balance_cte AS
 (
 SELECT *, SUM(transaction_amount) OVER (PARTITION BY customer_id ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_balance
 FROM txnamount_cte
 )
 ,avg_running_balance_cte AS
 (
 SELECT DISTINCT customer_id,txn_month,txn_month_id,AVG(running_balance) OVER(PARTITION BY customer_id,txn_month ORDER BY txn_month) AS avg_running_balance
 FROM running_balance_cte
 )
 ,prev_avg_rb_cte AS
 (
 SELECT *, LAG(avg_running_balance) OVER(PARTITION BY customer_id ORDER BY txn_month_id) AS prev_avg_rb
 FROM avg_running_balance_cte
 )
 ,data_cte AS
 (
 SELECT *, CASE WHEN prev_avg_rb >= 0 THEN prev_avg_rb ELSE 0 END AS data_allocated
 FROM prev_avg_rb_cte
 )
 SELECT txn_month,SUM(data_allocated) AS total_data_required
 FROM data_cte
 GROUP BY txn_month,txn_month_id
 ORDER BY txn_month_id;

 /*  
 AVG() here is used as a Window function. And window functions by default do not collapse the set of rows they operate on. 
 They show the output for each of the row in the set of querying rows. Hence, I have used SELECT DISTINCT to remove dupliate rows in avg_running_balance_cte.
 */


 -- Option 3: data is updated real-time

 -- Data_Allocated based on Running_Balance

 WITH txnamount_cte AS
 (
 SELECT t.customer_id,txn_date, DATENAME(month,txn_date) AS txn_month,txn_type,
		(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
 FROM customer_transactions t
 )
 ,running_balance_cte AS
 (
 SELECT *, SUM(transaction_amount) OVER (PARTITION BY customer_id ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_balance
 FROM txnamount_cte
 )
 SELECT *, (CASE WHEN running_balance >= 0 THEN running_balance ELSE 0 END) AS data_allocated
 FROM running_balance_cte
 ORDER BY customer_id;

 -- Total Data Required

 WITH txnamount_cte AS
 (
 SELECT t.customer_id,txn_date,DATEPART(month,txn_date) AS txn_month_id, DATENAME(month,txn_date) AS txn_month,txn_type,
		(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
 FROM customer_transactions t
 )
 ,running_balance_cte AS
 (
 SELECT *, SUM(transaction_amount) OVER (PARTITION BY customer_id ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_balance
 FROM txnamount_cte
 )
 , data_cte AS
 (
 SELECT *, (CASE WHEN running_balance >= 0 THEN running_balance ELSE 0 END) AS data_allocated
 FROM running_balance_cte
 )
 SELECT txn_month,SUM(data_allocated) AS total_data_required 
 FROM data_cte
 GROUP BY txn_month,txn_month_id
 ORDER BY txn_month_id;
 
 

 ----------------------------------------
