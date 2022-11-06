# 💸 Case Study #4 - Data Bank

## Solution C. Data Allocation Challenge

Data bank has 3 different options to allocate data storage space to its customers.

- Option 1: data is allocated based off the amount of money at the end of the previous month

- Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days

- Option 3: data is updated real-time

Now, the Data Bank team wants to estimate how much data storage space will need to be provisioned for each option. It requires the following data elements to help it with the estimation.

- running customer balance column that includes the impact each transaction

````sql
WITH cte AS
 (
   SELECT t.customer_id,txn_date, DATENAME(month,txn_date) AS txn_month,txn_type,
	  (CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
   FROM customer_transactions t
 )
    SELECT *,
      SUM(transaction_amount) 
      OVER (PARTITION BY customer_id ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_balance
    FROM cte
    ORDER BY customer_id;
 ````
 **Output:**
 
 ![Screenshot 2022-11-06 140401](https://user-images.githubusercontent.com/96012488/200161529-b75eeaf0-a771-40e1-90f0-acbb132bd0e7.png)
 
 - customer balance at the end of each month

````sql
WITH amounts_cte AS
(
  SELECT t.customer_id, DATENAME(month,txn_date) AS txn_month,DATEPART(month,txn_date) AS txn_month_id,
         SUM(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
  FROM customer_transactions t
  GROUP BY customer_id,DATENAME(month,txn_date),DATEPART(month,txn_date)
)
  SELECT customer_id,txn_month,
	 SUM(transaction_amount) 
         OVER (PARTITION BY customer_id ORDER BY txn_month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW ) AS closing_balance
  FROM amounts_cte
  ORDER BY customer_id,txn_month_id;
 ````
 
 **Output:**
 
 ![Screenshot 2022-11-06 141610](https://user-images.githubusercontent.com/96012488/200161957-aa6d938f-fa44-4c0e-8e4f-897dd3633c03.png)

- minimum, average and maximum values of the running balance for each customer

````sql
WITH txnamount_cte AS
 (
   SELECT t.customer_id,txn_date, DATENAME(month,txn_date) AS txn_month,txn_type,
          CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
   FROM customer_transactions t
 )
 , running_balance_cte AS
 (
   SELECT *, 
          SUM(transaction_amount) 
          OVER (PARTITION BY customer_id ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_balance
   FROM txnamount_cte
 )
    SELECT 
       customer_id, MIN(running_balance) AS min_balance, AVG(running_balance) AS avg_balance, MAX(running_balance) AS max_balance
    FROM running_balance_cte
    GROUP BY customer_id
    ORDER BY customer_id;
 ````
 
 **Output:**
 
 ![Screenshot 2022-11-06 142223](https://user-images.githubusercontent.com/96012488/200162144-538cfd76-35d4-49e3-a91e-fe378888b667.png)

 
 
