# 💸 Case Study #4 - Data Bank

## 📝 Solution A. Customer Nodes Exploration

### 1. How many unique nodes are there on the Data Bank system?

````sql
SELECT 
  COUNT(DISTINCT node_id) AS unique_node_count
FROM customer_nodes;
````
**Answer:**

![1 CS 4](https://user-images.githubusercontent.com/96012488/200128877-cb0f5d02-95c1-4611-bb43-075b388541f9.png)

- There are 5 unique nodes.



### 2. What is the number of nodes per region?

````sql
SELECT 
  region_id,COUNT(DISTINCT node_id) node_count
FROM customer_nodes
GROUP BY region_id;
 ````
 
 **Answer:**
 
 ![2 CS 4](https://user-images.githubusercontent.com/96012488/200128911-7a6d64d1-cf52-4b92-97ed-dc652d189f46.png)

- Each region has 5 unique nodes.
 
 ### 3. How many customers are allocated to each region?
 
 ````sql
 SELECT 
  n.region_id,r.region_name,COUNT(DISTINCT customer_id) AS customer_count
 FROM customer_nodes n
 JOIN regions r
 ON n.region_id=r.region_id
 GROUP BY n.region_id,r.region_name
 ORDER BY n.region_id;
 ````
 
 **Answer:**
 
 ![3 CS 4](https://user-images.githubusercontent.com/96012488/200128988-2bd1b62c-f8c5-4302-bf50-eb6920814dc7.png)


 
 ### 4. How many days on average are customers reallocated to a different node?
 
 ````sql
 SELECT 
  AVG(DATEDIFF(day,start_date,end_date)) avg_reallocation_days
 FROM customer_nodes
 WHERE end_date NOT LIKE '9999-%';
 ````
 
 **Answer:**
 
 ![4 CS 4](https://user-images.githubusercontent.com/96012488/200129014-67030e60-2216-4fad-ade5-224bc8e6450d.png)


 ### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
 
 ````sql
 WITH diff_cte AS
 (
 SELECT 
  n.region_id,r.region_name,customer_id,DATEDIFF(day,start_date,end_date) AS reallocation_days
 FROM customer_nodes n
 JOIN regions r
 ON n.region_id=r.region_id
 WHERE end_date NOT LIKE '9999-%'
 )
 SELECT DISTINCT 
      region_id,region_name,
	  PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY reallocation_days) OVER( PARTITION BY region_id) AS median,
	  PERCENTILE_DISC(0.80) WITHIN GROUP (ORDER BY reallocation_days) OVER( PARTITION BY region_id) AS percentile_80th,
	  PERCENTILE_DISC(0.95) WITHIN GROUP (ORDER BY reallocation_days) OVER( PARTITION BY region_id) AS percentile_95th
 FROM diff_cte;
 ````
 
 **Answer:**
 
 ![5 CS 4](https://user-images.githubusercontent.com/96012488/200129032-9cfdd683-166e-4d20-93b5-2a693a72fff3.png)
 
 - The median and the 90th percentile of reallocation days is 15 and 28 respectively for each region.
 - The 80th percentile of reallocation days is 23 for Africa and Europe while 24 for Asia, America and Australia.

***Check out the solution for B.Customer Transactions [here](https://github.com/PriyaPalak/8-Week-SQL-Challenge/blob/main/Case%20Study%20%234%20-%20Data%20Bank/B.%20Customer%20Transactions.md)!***
