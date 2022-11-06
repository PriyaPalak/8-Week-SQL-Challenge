# 💸 Case study #4 - Data Bank

## 📝 Solution B. Customer Transactions

### 1. What is the unique count and total amount for each transaction type?

````sql
SELECT 
  txn_type,COUNT(txn_type) AS unique_count,SUM(txn_amount) AS total_amount
FROM customer_transactions
GROUP BY txn_type;
````

**Answer:**

![cs 4 1](https://user-images.githubusercontent.com/96012488/200153699-11ce1fd0-51e6-4ced-9244-46b891e209c0.png)



### 2. What is the average total historical deposit counts and amounts for all customers?

````sql
WITH deposit_cte AS
 (
  SELECT customer_id,COUNT(txn_type) AS deposit_count,SUM(txn_amount) AS deposit_amount
  FROM customer_transactions
  WHERE txn_type = 'deposit'
  GROUP BY customer_id
 )
 SELECT
  AVG(deposit_count) AS avg_deposit_count,AVG(deposit_amount) AS avg_total_amount_deposited
 FROM deposit_cte;
 ````
 
 **Answer:**
 
 ![Screenshot 2022-11-06 092833](https://user-images.githubusercontent.com/96012488/200153705-1d7d1b89-6ce0-4b67-9117-2257b230c03c.png)

 
 ### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
 
 ````sql
 WITH months_cte AS
 (
  SELECT customer_id, DATENAME(month,txn_date) AS txn_month,DATEPART(month,txn_date) AS txn_month_id
  FROM customer_transactions
 )
 ,counts_cte AS
 (
  SELECT t.customer_id,txn_month,txn_month_id,SUM(CASE WHEN txn_type='deposit' THEN 1 ELSE 0 END) AS d,
         SUM(CASE WHEN txn_type='purchase' THEN 1 ELSE 0 END) AS p,
		SUM(CASE WHEN txn_type='withdrawal' THEN 1 ELSE 0 END) AS w
  FROM customer_transactions t
  JOIN months_cte
  ON months_cte.customer_id=t.customer_id
  GROUP BY t.customer_id,txn_month,txn_month_id
 )
    SELECT 
       txn_month,COUNT(customer_id) AS customer_count
    FROM counts_cte
    WHERE d>1 AND (p=1 OR w=1)
    GROUP BY txn_month,txn_month_id
    ORDER BY txn_month_id;
 ````
   
   **Answer:**
   
   ![Screenshot 2022-11-06 093022](https://user-images.githubusercontent.com/96012488/200153709-ff4ae689-2206-47f6-b759-295e2fba584d.png)

   
   ### 4. What is the closing balance for each customer at the end of the month?
   
   ````sql
   WITH amounts_cte AS
  (
    SELECT t.customer_id, DATENAME(month,txn_date) AS txn_month,DATEPART(month,txn_date) AS txn_month_id,
		SUM(CASE WHEN txn_type='deposit' THEN txn_amount ELSE -txn_amount END )AS transaction_amount
    FROM customer_transactions t
    GROUP BY customer_id,DATENAME(month,txn_date),DATEPART(month,txn_date)
   )
      SELECT 
        customer_id,txn_month,
        SUM(transaction_amount) OVER (PARTITION BY customer_id ORDER BY txn_month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW ) AS closing_balance
      FROM amounts_cte
      ORDER  BY customer_id,txn_month_id;
 ````
 
 **Answer:**
 
 ![Screenshot 2022-11-06 093249](https://user-images.githubusercontent.com/96012488/200153717-3fbfb8d3-839d-4e3f-b8a3-2abbb296df5f.png)

   
 
