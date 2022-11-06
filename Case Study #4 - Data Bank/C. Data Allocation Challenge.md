# 💸 Case Study #4 - Data Bank

## Solution C. Data Allocation Challenge

Data bank has 3 different options to allocate data storage space to its customers.

- **Option 1:** data is allocated based off the amount of money at the end of the previous month

- **Option 2:** data is allocated on the average amount of money kept in the account in the previous 30 days

- **Option 3:** data is updated real-time

### :dart: Goal: To estimate Data Storage Space that will need to be provisioned per month in each option

### Requirements

Now, since the Data Bank team wants to estimate how much data storage space will need to be provisioned per month for each option, it requires the following data elements to help it with the estimation.

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
 
 I have shown the output for a few customers to save space here.
 
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
 
 Again, I have shown output for just 20 customers to save up the space.
 
 ![Screenshot 2022-11-06 142223](https://user-images.githubusercontent.com/96012488/200162144-538cfd76-35d4-49e3-a91e-fe378888b667.png)
 
 
 ### Data Storage Space Per Month
 
 Now, the amount of Data Storage Space required per month in each of the 3 options are as follows:  
 
 - **Option 1** 
 
 **Data_Allocated per customer per month based on the closing balance of previous month**
 
 
 ````sql
 WITH monthlyamounts_cte AS
 (
  SELECT t.customer_id, DATENAME(month,txn_date) AS txn_month, DATEPART(month,txn_date) AS txn_month_id,
	 	SUM(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
  FROM customer_transactions t
  GROUP BY customer_id,DATENAME(month,txn_date),DATEPART(month,txn_date)
 )
 ,closing_balance_cte AS
 (
  SELECT customer_id,txn_month,txn_month_id,
	 	SUM(transaction_amount) 
	 	OVER (PARTITION BY customer_id ORDER BY txn_month_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW ) AS closing_balance
  FROM monthlyamounts_cte
 )
 ,prev_closing_balance_cte AS
 (
  SELECT *, 
 	LAG(closing_balance) OVER(PARTITION BY customer_id ORDER BY txn_month_id) AS prev_closing_balance
  FROM closing_balance_cte
 )
   SELECT *, 
	   (CASE WHEN prev_closing_balance >= 0 THEN prev_closing_balance ELSE 0 END) AS data_allocated
   FROM prev_closing_balance_cte;
   ````
   
   ![Screenshot 2022-11-06 161202](https://user-images.githubusercontent.com/96012488/200166205-dfbd4917-1249-4724-8a58-bcf6cc81ede6.png)


:arrow_right: **Total Data required per month**

````sql
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
	SUM(transaction_amount) 
	OVER (PARTITION BY customer_id ORDER BY txn_month_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW ) AS closing_balance
  FROM monthlyamounts_cte
 )
 , prev_closing_balance_cte AS
 (
  SELECT *, 
  	LAG(closing_balance) OVER(PARTITION BY customer_id ORDER BY txn_month_id) AS prev_closing_balance
  FROM closing_balance_cte
 )
 ,data_cte AS
 (
  SELECT *,
  	CASE WHEN prev_closing_balance >= 0 THEN prev_closing_balance ELSE 0 END AS data_allocated
  FROM prev_closing_balance_cte
 )
  SELECT 
  	txn_month,txn_month_id,SUM(data_allocated) AS total_data_required
  FROM data_cte
  GROUP BY txn_month,txn_month_id
  ORDER BY txn_month_id;
````
![Screenshot 2022-11-06 161439](https://user-images.githubusercontent.com/96012488/200166326-6bc9a7c7-9766-4074-a246-4b376d2ff4fc.png)

**Note:** Since there is no previous closing balance for January, data required for this month is 0.

- **Option 2**

**Data_Allocated per customer per month based on average running balance of the previous 30 days**

````sql
WITH txnamount_cte AS
 (
  SELECT t.customer_id,txn_date, DATENAME(month,txn_date) AS txn_month,txn_type,DATEPART(month,txn_date) AS txn_month_id,
	(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END ) AS transaction_amount
  FROM customer_transactions t
 )
 , running_balance_cte AS
 (
  SELECT *, 
  	SUM(transaction_amount) 
	OVER (PARTITION BY customer_id ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_balance
  FROM txnamount_cte
 )
 , avg_running_balance_cte AS
 (
  SELECT DISTINCT 
  	customer_id,txn_month,txn_month_id,
	AVG(running_balance) 
	OVER(PARTITION BY customer_id,txn_month ORDER BY txn_month) AS avg_running_balance
  FROM running_balance_cte
 )
 , prev_avg_rb_cte AS
 (
  SELECT *, 
 	LAG(avg_running_balance) OVER(PARTITION BY customer_id ORDER BY txn_month_id) AS prev_avg_rb
  FROM avg_running_balance_cte
 )
  SELECT *, 
 	CASE WHEN prev_avg_rb >= 0 THEN prev_avg_rb ELSE 0 END AS data_allocated
  FROM prev_avg_rb_cte
  ORDER BY customer_id;
 ````
![Screenshot 2022-11-06 162551](https://user-images.githubusercontent.com/96012488/200166715-f885ebf9-5d92-435c-8500-c4ccb1bbfe7e.png)

➡️ **Total Data required per month**

````sql
WITH txnamount_cte AS
 (
  SELECT t.customer_id,txn_date, DATENAME(month,txn_date) AS txn_month,txn_type, DATEPART(month,txn_date) AS txn_month_id,
	(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
  FROM customer_transactions t
 )
 , running_balance_cte AS
 (
  SELECT *, 
   	SUM(transaction_amount) 
	OVER (PARTITION BY customer_id ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_balance
  FROM txnamount_cte
 )
 , avg_running_balance_cte AS
 (
  SELECT DISTINCT 
  	customer_id, txn_month, txn_month_id,
  	AVG(running_balance) 
	OVER(PARTITION BY customer_id,txn_month ORDER BY txn_month) AS avg_running_balance
  FROM running_balance_cte
 )
 , prev_avg_rb_cte AS
 (
  SELECT *, 
 	LAG(avg_running_balance) 
	OVER(PARTITION BY customer_id ORDER BY txn_month_id) AS prev_avg_rb
  FROM avg_running_balance_cte
 )
 , data_cte AS
 (
  SELECT *, 
  	CASE WHEN prev_avg_rb >= 0 THEN prev_avg_rb ELSE 0 END AS data_allocated
  FROM prev_avg_rb_cte
 )
  SELECT 
  	txn_month,SUM(data_allocated) AS total_data_required
  FROM data_cte
  GROUP BY txn_month,txn_month_id
  ORDER BY txn_month_id;
````

![Screenshot 2022-11-06 162950](https://user-images.githubusercontent.com/96012488/200166894-6ec4c7d6-a472-496a-b4ad-690b97bb5a1e.png)

**Note:** Since there are no transactions before January, data required for this month is 0.

- **Option 3**

**Data_Allocated per customer per transaction based on the running balance**

````sql
WITH txnamount_cte AS
 (
  SELECT t.customer_id,txn_date, DATENAME(month,txn_date) AS txn_month,txn_type,
	(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
  FROM customer_transactions t
 )
 , running_balance_cte AS
 (
  SELECT *, 
  	SUM(transaction_amount) 
  	OVER (PARTITION BY customer_id ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_balance
  FROM txnamount_cte
 )
  SELECT *, 
  	(CASE WHEN running_balance >= 0 THEN running_balance ELSE 0 END) AS data_allocated
  FROM running_balance_cte
  ORDER BY customer_id;
````

![Screenshot 2022-11-06 164254](https://user-images.githubusercontent.com/96012488/200167363-248a1894-3c9f-4285-9dde-496276e2bcff.png)


➡️ **Total Data required per month**

````sql
 WITH txnamount_cte AS
 (
  SELECT t.customer_id,txn_date,DATEPART(month,txn_date) AS txn_month_id, DATENAME(month,txn_date) AS txn_month,txn_type,
	(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
  FROM customer_transactions t
 )
 , running_balance_cte AS
 (
  SELECT *, 
  	SUM(transaction_amount) 
	OVER (PARTITION BY customer_id ORDER BY txn_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_balance
  FROM txnamount_cte
 )
 , data_cte AS
 (
  SELECT *, 
  	(CASE WHEN running_balance >= 0 THEN running_balance ELSE 0 END) AS data_allocated
  FROM running_balance_cte
 )
  SELECT 
  	txn_month, SUM(data_allocated) AS total_data_required 
  FROM data_cte
  GROUP BY txn_month,txn_month_id
  ORDER BY txn_month_id;
````

![Screenshot 2022-11-06 164131](https://user-images.githubusercontent.com/96012488/200167319-ef54654b-dfd8-4f61-b19a-91ce1d9d04f2.png)


### Conclusion

➡️ Option 1 and 2 have almost the same data requirements, whereas Option 3 requires much more data to be provisioned in all the months.
 
 
