# 💸 Case Study #4 - Data Bank

## 📝 Solution A. Customer Nodes Exploration

### 1. How many unique nodes are there on the Data Bank system?

````sql
SELECT 
  COUNT(DISTINCT node_id) AS unique_node_count
FROM customer_nodes;
````
**Answer:**

### 2. What is the number of nodes per region?

````sql
SELECT 
  region_id,COUNT(DISTINCT node_id) node_count
FROM customer_nodes
GROUP BY region_id;
 ````
 
 **Answer:**
 
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
 
 ### 4. How many days on average are customers reallocated to a different node?
 
 ````sql
 SELECT 
  AVG(DATEDIFF(day,start_date,end_date)) avg_reallocation_days
 FROM customer_nodes
 WHERE end_date NOT LIKE '9999-%';
 ````
 
 **Answer:**
 
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
