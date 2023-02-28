# :convenience_store: Case Study #5: Data Mart

## Business Task:

Data Mart is an online supermarket that specialises in fresh produce. After running its international operations, Danny wants to analyse his sales performance.
Also, in June 2020, the company shifted to using sustainable packaging methods in every single step of the production. Danny needs help to quantify the impact of this change on the sales performance for Data Mart and its separate business areas.

## :memo: Solution 1. Data Cleansing Steps

*First, we perform some data cleaning before we start to answer Danny's key business questions in depth.*

**(a) Convert the week_date to a DATE format.**

````sql
ALTER TABLE dbo.weekly_sales
ALTER COLUMN week_date DATE;
````

**(b) Add week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc.**

````sql
ALTER TABLE weekly_sales
ADD week_number AS DATEPART(week,week_date);
````
- Here, we take help of a computed column to add `week_number` to the existing table `weekly_sales`. This works in SQL Server.
- To position the column after week_date, we can use `AFTER` directive in the `ADD` statement. This works in MySQL Workbench.
However, it doesn't work in SQL Server. So, we need to use the System Interface to reorder the columns as desired.

**(c) Add a month_number with the calendar month for each week_date value as the 3rd column.**

````sql

````
