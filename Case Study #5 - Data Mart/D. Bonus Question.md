# 🏪 Case Study #5: Data Mart

## 📝 Solution D. Bonus Question

### Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?

- region
- platform
- age_band
- demographic
- customer_type

 The approach is similar to comparing 12 weeks period before and after change, with the focus on a specific area of business.
 So, we'll just break down the numbers by that specific area of business, e.g., region.

### 1. Region

````sql
WITH sales_24_weeks AS
(
SELECT 
  region,week_date,week_number,SUM(sales) AS total_sales
FROM clean_weekly_sales
WHERE (week_number BETWEEN 13 AND 36)
AND (calendar_year = 2020)
GROUP BY region,week_date,week_number
)
, before_after_change AS
(
SELECT 
  region,
	SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS sales_before_change,
	SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sales END) AS sales_after_change
 FROM sales_24_weeks
 GROUP BY region
)
SELECT 
  region,
	(sales_after_change - sales_before_change) AS impact_on_sales,
	ROUND(100*((sales_after_change - sales_before_change)/sales_before_change),2) AS impact_percentage
FROM before_after_change
ORDER BY impact_percentage;
````
**Answer:**

![Screenshot 2023-06-16 123043](https://github.com/PriyaPalak/8-Week-SQL-Challenge/assets/96012488/b3dfc587-c543-489b-94f8-3c4ab0e28eb1)


- Asia saw the highest negative impact on sales in 2020, with a decline of 3.26% in sales after packaging change.
- The metrics show that the sales declined in  every region except Europe. In fact, the sales saw a growth of 4.73% in Europe after the sustainable changes were introduced.
- This might be possible due to more inclination of Europeans toward sustainability. Maybe, they prefer sustainable products and don't mind paying slightly more in return.


### 2. Platform

````sql
WITH sales_24_weeks AS
(
SELECT 
  platform,week_date,week_number,SUM(sales) AS total_sales
FROM clean_weekly_sales
WHERE (week_number BETWEEN 13 AND 36)
AND (calendar_year = 2020)
GROUP BY platform,week_date,week_number
)
, before_after_change AS
(
SELECT 
  platform,
	SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS sales_before_change,
	SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sales END) AS sales_after_change
 FROM sales_24_weeks
 GROUP BY platform
)
SELECT 
   platform,
	(sales_after_change - sales_before_change) AS impact_on_sales,
	ROUND(100*((sales_after_change - sales_before_change)/sales_before_change),2) AS impact_percentage
FROM before_after_change
ORDER BY impact_percentage;
````
**Answer:**

![Screenshot 2023-06-16 123202](https://github.com/PriyaPalak/8-Week-SQL-Challenge/assets/96012488/c53d7094-2b75-42e0-9c3f-67641f236472)


- Offline Retail stores saw a drop in sales by 2.43%, while Online Shopify stores observed a rise in sales by 7.18%.
- A possible explanation for this could be that Online customers of Data Mart are more inclined toward sustainable products compared to offline customers.




### 3. Age_band

````sql
WITH sales_24_weeks AS
(
SELECT 
  age_band,week_date,week_number,SUM(sales) AS total_sales
FROM clean_weekly_sales
WHERE (week_number BETWEEN 13 AND 36)
AND (calendar_year = 2020)
GROUP BY age_band,week_date,week_number
)
, before_after_change AS
(
SELECT 
  age_band,
	SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS sales_before_change,
	SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sales END) AS sales_after_change
 FROM sales_24_weeks
 GROUP BY age_band
)
SELECT 
  age_band,
	(sales_after_change - sales_before_change) AS impact_on_sales,
	ROUND(100*((sales_after_change - sales_before_change)/sales_before_change),2) AS impact_percentage
FROM before_after_change
ORDER BY impact_percentage;
````

**Answer:**

![Screenshot 2023-06-16 123253](https://github.com/PriyaPalak/8-Week-SQL-Challenge/assets/96012488/8b81ca1d-e750-4c9a-b71a-0a37c05e597b)


- There has been a decline in sales across all the age groups after the packaging changes were introduced. However, the decline is highest in the unknown
- age group at a negative of 3.34%, whereas the 'Young Adults' group saw the lowest drop in sales due to packaging change at a negative of 0.92%.

### 4. Demographic

````sql
WITH sales_24_weeks AS
(
SELECT 
  demographic,week_date,week_number,SUM(sales) AS total_sales
FROM clean_weekly_sales
WHERE (week_number BETWEEN 13 AND 36)
AND (calendar_year = 2020)
GROUP BY demographic,week_date,week_number
)
, before_after_change AS
(
SELECT 
  demographic,
	SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS sales_before_change,
	SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sales END) AS sales_after_change
 FROM sales_24_weeks
 GROUP BY demographic
)
SELECT 
  demographic,
	(sales_after_change - sales_before_change) AS impact_on_sales,
	ROUND(100*((sales_after_change - sales_before_change)/sales_before_change),2) AS impact_percentage
FROM before_after_change
ORDER BY impact_percentage;
````

**Answer:**

![Screenshot 2023-06-16 123336](https://github.com/PriyaPalak/8-Week-SQL-Challenge/assets/96012488/3685a931-bc1e-4a01-89af-5bd7b61cdeb0)

- Again, there has been a decline in sales across all demographics after the packaging changes, with the highest decline in the unknown demographic at a negative of 3.34%.
- While the 'families' demographic saw a drop of 1.82%, the drop in 'Couples' demographic was only 0.87%. 
- This might be because Families usually have more expenses and responsibilities compared to Couples. Hence, they find it difficult to pay a higher price for sustainable products and thus opt for cheaper alternatives.Couples, having less expenses and responsibilities, find it easier to afford slightly expensive sustainable products and thus stay loyal to the brand.

### 5. Customer_type

````sql
WITH sales_24_weeks AS
(
SELECT 
  customer_type,week_date,week_number,SUM(sales) AS total_sales
FROM clean_weekly_sales
WHERE (week_number BETWEEN 13 AND 36)
AND (calendar_year = 2020)
GROUP BY customer_type,week_date,week_number
)
, before_after_change AS
(
SELECT 
  customer_type,
	SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS sales_before_change,
	SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sales END) AS sales_after_change
 FROM sales_24_weeks
 GROUP BY customer_type
)
SELECT 
  customer_type,
	(sales_after_change - sales_before_change) AS impact_on_sales,
	ROUND(100*((sales_after_change - sales_before_change)/sales_before_change),2) AS impact_percentage
FROM before_after_change
ORDER BY impact_percentage;
````
**Answer:**

![Screenshot 2023-06-16 123430](https://github.com/PriyaPalak/8-Week-SQL-Challenge/assets/96012488/1efba620-91c4-46e1-9ef1-45f12957e0f0)

- Sales reduced by 3% in the Guest Customers category, and by 2.27% in Existing Customers category. Meanwhile, 'New Customer' Category saw a rise in sales by 1.01%.
- A possible explanation could be that the brand, after making these changes, attracted new customers who are more into sustainable products and thus the sales increased in this category.

***

*The possible explanations I've provided may or may not be the real reason for the drop. Further analysis and data on customers' attitude is required to come up with a more reliable explanation.*

***Thanks for checking out my solutions!***
***Hope you find it useful :)***
