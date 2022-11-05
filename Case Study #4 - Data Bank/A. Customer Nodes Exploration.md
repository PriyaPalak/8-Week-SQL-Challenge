# 💸 Case Study #4 - Data Bank

## 📝 Solution A. Customer Nodes Exploration

### 1. How many unique nodes are there on the Data Bank system?

````sql
SELECT 
  COUNT(DISTINCT node_id) AS unique_node_count
FROM customer_nodes;
````
**Answer:**
