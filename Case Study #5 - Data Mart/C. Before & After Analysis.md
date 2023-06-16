# 🏪 Case Study #5: Data Mart

## :memo: Solution C. Before & After Analysis

This technique is usually used when we inspect an important event and its impact on the business performance.

*Here, the event is introduction of sustainable packaging methods in every single step from the farm all the way to the customer. Danny needs help to quantify the impact of this 
change on the sales performance for Data Mart and it’s separate business areas.*

*Taking the week_date value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect,we 
would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before.*

So, let's first see which week the changes were introduced.

````sql
SELECT DATEPART(week,'2020-06-15') AS week_number;
````
![Screenshot 2023-06-16 111852](https://github.com/PriyaPalak/8-Week-SQL-Challenge/assets/96012488/f68ea47e-676f-4ff6-8d0f-cefbd578b50b)

- The changes were introduced in `week number 25`.
- 12 weeks before 25th week : period `before` change
- 12 weeks including and after 25th week: period `after` change

### 1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?

````sql
WITH sales_8_weeks AS
(
SELECT 
  week_date,week_number,SUM(sales) AS total_sales
FROM clean_weekly_sales
WHERE (week_number BETWEEN 21 AND 28)
AND (calendar_year = 2020)
GROUP BY week_date,week_number
)
, before_after_change AS
(
SELECT 
	SUM(CASE WHEN week_number BETWEEN 21 AND 24 THEN total_sales END) AS sales_before_change,
	SUM(CASE WHEN week_number BETWEEN 25 AND 28 THEN total_sales END) AS sales_after_change
 FROM sales_8_weeks
)
SELECT 
	(sales_after_change - sales_before_change) AS impact_on_sales,
	ROUND(100*((sales_after_change - sales_before_change)/sales_before_change),2) AS impact_percentage
FROM before_after_change;
````
**Answer:**

![Screenshot 2023-06-16 112638](https://github.com/PriyaPalak/8-Week-SQL-Challenge/assets/96012488/b688d3a4-0b1a-436b-925c-72216ab36b0f)


- Thus, the total sales reduced in the period after change by an amount of $26,884,188, reflecting a negative change by 1.15%.
- The possible reasons could be : customers didn't recognise the product on the shelves due to new packaging,
 or they opted for a cheaper alternative as sustainable changes often lead to higher prices.

### 2. What about the entire 12 weeks before and after?

A similar approach can be used to address this question.

````sql
WITH sales_24_weeks AS
(
SELECT 
  week_date,week_number,SUM(sales) AS total_sales
FROM clean_weekly_sales
WHERE (week_number BETWEEN 13 AND 36)
AND (calendar_year = 2020)
GROUP BY week_date,week_number
)
, before_after_change AS
(
SELECT 
	SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS sales_before_change,
	SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sales END) AS sales_after_change
 FROM sales_24_weeks
)
SELECT 
	(sales_after_change - sales_before_change) AS impact_on_sales,
	ROUND(100*((sales_after_change - sales_before_change)/sales_before_change),2) AS impact_percentage
FROM before_after_change;
````

**Answer:**

![Screenshot 2023-06-16 112742](https://github.com/PriyaPalak/8-Week-SQL-Challenge/assets/96012488/403b9124-8641-4739-b045-27d5b9d9272f)

- If we consider the entire 12 weeks before and after the change, the decline in sales in the after period is even higher at a negative of 2.14%.

### 3. How do the sales metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

 Let's first compare the sales metrics for `4 weeks` before and after with the previous years 2018 and 2019.

- The same approach can be used here. We just need to add `calendar_year` into the context.

````sql
WITH sales_8_weeks AS
(
SELECT 
  calendar_year,week_date,week_number,SUM(sales) AS total_sales
FROM clean_weekly_sales
WHERE (week_number BETWEEN 21 AND 28)
GROUP BY calendar_year,week_date,week_number
)
, before_after_week_25 AS
(
SELECT calendar_year,
	SUM(CASE WHEN week_number BETWEEN 21 AND 24 THEN total_sales END) AS sales_before_change,
	SUM(CASE WHEN week_number BETWEEN 25 AND 28 THEN total_sales END) AS sales_after_change
 FROM sales_8_weeks
 GROUP BY calendar_year
)
SELECT calendar_year,
	(sales_after_change - sales_before_change) AS impact_on_sales,
	ROUND(100*((sales_after_change - sales_before_change)/sales_before_change),2) AS impact_percentage
FROM before_after_week_25;
````
**Answer:**

![Screenshot 2023-06-16 112908](https://github.com/PriyaPalak/8-Week-SQL-Challenge/assets/96012488/489a1e98-47a2-4ce0-af6c-b3b7942c9f44)


- In 2018, the sales increased by an amount of $ 4,102,105, indicating a positive change of 0.19% in the period after changes were introduced.
- Similarly, in 2019, the sales saw a rise amounting to $ 2,336,594, corresponding to a positive change of 0.1% in the period after packaging change.

- However, in 2020, there has been a significant drop in sales following the introduction of sustainable packaging. The sales declined by $ 26,884,188, reflecting a percentage drop of 1.15%.This reduction shows a significant impact on sales compared to the previous years 2018 and 2019.

Now, let's repeat this analysis for a period of `12 weeks`.



- For this, we just change the week number to `13 to 24` for before period and `25 to 36` for after period.

````sql
WITH sales_24_weeks AS
(
SELECT 
  calendar_year, week_date,week_number,SUM(sales) AS total_sales
FROM clean_weekly_sales
WHERE (week_number BETWEEN 13 AND 36)
GROUP BY calendar_year,week_date,week_number
)
, before_after_change_week AS
(
SELECT 
  calendar_year,
	SUM(CASE WHEN week_number BETWEEN 13 AND 24 THEN total_sales END) AS sales_before_25,
	SUM(CASE WHEN week_number BETWEEN 25 AND 36 THEN total_sales END) AS sales_after_25
 FROM sales_24_weeks
 GROUP BY calendar_year
)
SELECT calendar_year,
	(sales_after_25 - sales_before_25) AS impact_on_sales,
	ROUND(100*((sales_after_25 - sales_before_25)/sales_before_25),2) AS impact_percentage
FROM before_after_change_week
ORDER BY calendar_year;
````
**Answer:**

![Screenshot 2023-06-16 113410](https://github.com/PriyaPalak/8-Week-SQL-Challenge/assets/96012488/7e183564-975e-43b2-8a3a-7ae5be118338)

- As we can see above, in 2018, the sales increased by 1.63% in the period after change. While in 2019, a slight drop of 0.3% was observed.
- In 2020, however, the drop is even more significant at a staggerring 2.14%!

- Thus, there has been a significant impact on sales in 2020 due to packaging changes compared to the previous years, 2018 and 2019.

- When we compare the best year (2018) wih the worst year (2020), the variation in sales becomes even more apparent with a drop of 3.77% (1.63% + 2.14%).

***

*I thoroughly enjoyed solving this section of the case study - `Before-After analysis`.*
*Hadn't done this kind of analysis before. Got to learn something new and completely applicable in real world. Will use it as a guide when faced with a similar problem at workplace.*

***

***Click here to check out my solution for D. Bonus Question [here]()!***







