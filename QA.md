# What are your risk areas? Identify and describe them.

1. For the table *all_sessions*, there were a lot missing values, with columns with mostly null values or unavailable, which makes it hard to use them for analysis. There was also a lot of duplicate data that needed to be cleaned. Although the *all_sessions* table needed some cleaning and queries were written for this purpose, it was hard to know if the data was properly cleaned, in terms of duplicates. Being the reason that, for this particular table, there is no primary key, not even a composite primary key, meaning that there is a potential for a lot of repetitive data. However, it makes sense that there are repetitive *full_visitor_id* as they could belong to several visits during different days and times. 

2. The product categories were a long list of distinct categories. To help draw insights from the data from a more manageable number of categories, an approach of condensing categories was considered and used. In doing this, the insights obtained are not as accurate in terms of what exact products are included in each of the categories. Reducing the number of categories for this project helped with the analysis to draw conclusions and summarize the data. However, it could have been more specific if all the existing categories were analyzed as they were.  

3. Another area that must be considered as risky, is the cities and countries being a valid combination. For example, one of the Country-City combinations found is 'Australia - Los Angeles'. This is likely a mistake, unless there is a city in Australia called 'Los Angeles'. A bit of searching was done to find out whether this is a valid city for Australia, however, nothing was found. Creating a country-city mapping to ensure that all combinations are correct would help to ensure that the results are as accurate as possible.   

4. Understanding what each of the tables and its attributes meant was another challenge. Exploring the data types and what the values were for each of the tables took a significant amount of time. Finding relationships between tables was also another important area to look into.

5. Another main risk area to consider is the QA process itself, which still needs a lot of work and could potentially be limited at this point. Learning more about QA techniques or methods and about what are the right questions to ask when working on this would help improve the overall project's results. 
  
# QA Process: describe your QA process and include the SQL queries used to execute it.

1. Thinking about what information this table is providing, and what information is needed from it to answer the questions from this project, the decision to group everything by distinct groups of a combination of the following attributes was decided: *full_visitor_id*, *product_sku*, *country*, and *city*, when creating the VIEW *all_sessions_cleaned*. The decision to only include *product_sku* and not the product names or the category names, is because that information could be easily accessible by joining the products table and the original *all_sessions* table using the *product_sku* attribute.

	The following query was created to check the duplicates from the table grouped as mentioned above:

``` 
SELECT	full_visitor_id,
	product_sku, 
   	city, 
    	country, 
    	ROW_NUMBER() OVER(PARTITION BY	full_visitor_id, product_sku, city, country ORDER BY full_visitor_id) AS duplicatecount
FROM all_sessions
ORDER BY ROW_NUMBER() OVER(PARTITION BY	full_visitor_id, product_sku, city, country ORDER BY full_visitor_id) DESC 
```

The query above is meant to show all duplicates based on the combination of attributes specified before, and then used as a subquery to create the VIEW *all_sessions_cleaned*, where only the minimum values of the *duplicatecount* were included (and all the other records from the attributes related to that record). In the VIEW, further filtering was conducted to clean the cities that were not valid. 

```
CREATE VIEW all_sessions_cleaned AS
	SELECT	full_visitor_id,
		product_sku,
		city,
		country,
		MIN (duplicatecount) AS duplicate_count
	FROM	(SELECT	full_visitor_id,
			product_sku, 
   			city, 
    		country, 
    		ROW_NUMBER() OVER(PARTITION BY	full_visitor_id, product_sku, city, country ORDER BY full_visitor_id) AS duplicatecount
   		FROM all_sessions
		ORDER BY ROW_NUMBER() OVER(PARTITION BY	full_visitor_id, product_sku, city, country ORDER BY full_visitor_id) DESC
		)
WHERE	city NOT IN ('(not set)', 'not available in demo dataset')
GROUP BY full_visitor_id, product_sku, country, city
```

After reviewing the code and the table one last time, something else was careful considered which would potentially make the results from the queries above, innacurate. The fact that duplicates were taken off the resulting table, means that the *product_sku*'s from those records were also eliminated, although one visitor could have more than one combination of these attributes *full_visitor_id*, *product_sku*, *country*, and *city*, due to several visits in different dates, basically multiple orders. However, because this two attributes were not considered, the elimination of these duplicates could potentially be a mistake. To fix this, the query could be re-written to include this other attributes in the partition. 

Even though this process was done, when answering the questions from this project, another VIEW, *for_revenue* was used for almost all questions. The reasoning behind this is that the questions were based on revenue from products ordered by online visitors. And the information needed to obtain those results were obtained using the table *sales_report* and the names and category names for all products could be obtained by joining the corresponding table through the *product_sku*. 

2. For the categories in the *all_sessions* table were explored to know how many distinct categories there were and how many duplicates there were. Two results were invalid categories so they were filtered out. The remaining valid categories were 74 distinct values.

```
SELECT	v2_product_category,
	COUNT (*)
FROM all_sessions
GROUP BY v2_product_category
ORDER BY v2_product_category NULLS FIRST -- (not set) x 757 rows and {escCatTitle} x 19 rows
```

With the following VIEW *condensed_categories*, the number of categories was reduced to only 18 using the next query after this one. 

```
CREATE OR REPLACE VIEW condensed_categories AS
	SELECT	CASE	WHEN v2_product_category ILIKE '%brand%' OR v2_product_category ILIKE '%YouTube%' THEN 'Brands'
			WHEN v2_product_category ILIKE '%bottles%' OR v2_product_category ILIKE '%drinkware%' THEN 'Drinkware'
			WHEN v2_product_category ILIKE '%electronics%' OR v2_product_category ILIKE '%nest%' THEN 'Electronics'
			WHEN v2_product_category ILIKE '%apparel%' OR v2_product_category ILIKE '%wearables%' THEN 'Apparel'
			WHEN v2_product_category ILIKE '%office%' THEN 'Home Office'
			WHEN v2_product_category ILIKE '%accessories%' AND v2_product_category NOT LIKE '%electronics%' THEN 'Home Accessories'
			WHEN v2_product_category ILIKE '%bags%' THEN 'Bags & Backpacks'
			WHEN v2_product_category ILIKE '%lifestyle%' THEN 'Lifestyle'
			ELSE v2_product_category
		END AS category_name,
		COUNT (*) AS category_count
FROM all_sessions
WHERE v2_product_name NOT IN ('(not set)', '${escCatTitle}')
GROUP BY v2_product_category
ORDER BY COUNT(*) DESC 
```
Query to check the distinct number of categories referencing the VIEW *condensed_categories*:
```
SELECT	category_name, 
	SUM(category_count) AS total_category_count
FROM condensed_categories
GROUP BY category_name
ORDER BY SUM(category_count) DESC
```
After exploring a bit more the individual product names to answer one of the questions from the project, it was noted that some of the products included words such as 'Youtube', 'Google' and 'Android' in their names, but also words like 'shorts' and 't-shirts' that could have been considered in the *Apparel* category. However, they were probably included in the category *Brands*. The categories are still technically correct, as those products are from said brands, and are included in one of the categories explored. However, considering to separate the products included in the *Brands* category and allocate the different types of products into the other categories like *Apparel*, *Electronics*, *Bags & Backpacks* and *Drinkware* could potentially have an impact in the insights obtained in Question 3 from the *starting_with_questions.md* document.  

```
SELECT	v2_product_name,
	COUNT (*)
FROM all_sessions
GROUP BY v2_product_name
ORDER BY v2_product_name NULLS FIRST -- A lot of products with brand names and clothing combinations.
```

3. A country-city mapping was attempted using the *world-cities* dataset from GitHub (https://github.com/datasets/world-cities.). However, because of technical difficulties and not enough time to troubleshoot, the table wasn't imported into the created Database in PgAdmin, so the mapping wasn't completed. ChatGPT was a great AI tool used for this specific situation, where instructions on how to create a reference dataset to ensure all country-city combinations in our analysis are valid where given. The mapping would be a great addition to the QA process, so it would be worth checking in to see what went wrong with the import of the data, and then try one more time. 

4. The tables were explored by a number of individual queries to see how many nulls, or duplicates there were per column in different tables, also to define Primary and Foreign Keys. The following are some of the queries used for this. 
```
SELECT	sku,
	COUNT(*) AS num_duplicates
FROM products 
GROUP BY sku
HAVING COUNT(*) > 1 -- No duplicates, sku is primary key for Products table
```
```
SELECT	product_sku,
	COUNT(*) AS num_duplicates
FROM sales_by_sku
GROUP BY product_sku
HAVING COUNT(*) > 1 -- No duplicates, product_sku is primary key for sales_by_sku
```
```
SELECT	product_sku,
	COUNT(*) AS num_duplicates
FROM sales_report
GROUP BY product_sku
HAVING COUNT(*) > 1 -- No duplicates, product_sku is primary key to sales_report
```
```
SELECT	full_visitor_id,
	COUNT(*) AS num_duplicates
FROM all_sessions 
GROUP BY full_visitor_id
HAVING COUNT(*) > 1 -- has 794 visitors_id with duplicates, so full_visitor id is not primary key
```
When trying to find a Primary Key from the analytics table, each of the attributes were checked first to see if they had any duplicates. Because all of them had duplicates, a composite primary key was explored. While looking for the right combination of attributes, the following query resulted in an error that said that the *visit_Number* attribute didn't exist, but it would appear in the output when retrieving all attributes using *\**. Although an output was not possible from the query below, it would have probably resulted in many rows with duplicates as each of the *visit_Number*'s were allocated to many records at a time.  
```
SELECT	visit_Number, -- It says this column doesn't exist 
	visit_id,
	full_visitor_id,
	COUNT(*) AS num_duplicates
FROM analytics
GROUP BY visit_Number, visit_id, full_visitor_id
HAVING COUNT(*) > 1

SELECT *
FROM analytics -- The visit_Number shows when running this code.
```
After trying the following group combinations to try to find a composite Primary key, it was concluded that the table doesn't include any, so an autogenerated Primary Key was considered and included in the table as the *analytics_id*. This primary key, however, won't be of any use for the purposed of answering the questions in the project. 
```
SELECT	visit_id,
full_visitor_id,
visit_start_time,
visit_date,
time_on_site,
COUNT(*) AS num_duplicates
FROM analytics
GROUP BY visit_id, full_visitor_id, visit_start_time, visit_date, time_on_site
HAVING COUNT(*) > 1 --- There are no attributes in this table that could serve as a Primary key, not even a composite primary key.
```
Cleaning the table *analytics* involved a lot of aggregating and changing the NULL values to zeros for those attributes that were aggregated with the SUM function. However, the lack of information about what each of the attributes really mean, how they were collected, and what is measured with that data, makes it difficult to know if aggregating all those attributes was the right choice. Although the number of total records in the VIEW *analytics_aggregated* reduced significantly, we could not know for sure what the impact of the decisions made to clean the table is until more information about the attributes is obtained. 
```
CREATE VIEW analytics_aggregated AS
SELECT	visit_id,
		visit_start_time,
		visit_date,
		full_visitor_id,
		channel_grouping,
		social_engagement_type,
		SUM(COALESCE(units_sold, 0)) AS units_sold_sum,
		SUM(COALESCE(page_views, 0)) AS page_views_sum,
		SUM(COALESCE(time_on_site,0)) AS time_on_site_sum,
		SUM(COALESCE(bounces, 0)) AS bounces_SUM,
		SUM(COALESCE(revenue, 0)) AS revenue_sum,
		SUM(COALESCE(unit_price, 0)) AS unit_price_sum
FROM	analytics
GROUP BY visit_id,
		 visit_start_time,
		 visit_date,
		 full_visitor_id,
		 channel_grouping,
		 social_engagement_type
```
The dates in the *visit_date* attribute were also explored to ensure all its records are valid. With the following query the date range May 1st, 2017 to August 1st, 2017, was obtained: 

```
SELECT	MIN(visit_date) AS min_date, -- 2017-05-01
	MAX (visit_date) AS max_date -- 2017-08-01
FROM analytics_aggregated 
```
The following query was ran to know if all days in the week were included in the *visit_date*. However, it was observed that Saturdays were not in the data. 

```
SELECT CASE EXTRACT(DOW FROM visit_date)
     		WHEN 1 THEN 'Sunday'
   		WHEN 2 THEN 'Monday'
     		WHEN 3 THEN 'Tuesday'
     		WHEN 4 THEN 'Wednesday'
     		WHEN 5 THEN 'Thursday'
     		WHEN 6 THEN 'Friday'
     		WHEN 7 THEN 'Saturday'
   	 	END AS day_of_week
FROM analytics_aggregated
WHERE EXTRACT(DOW FROM visit_date) = 7 --> Change from 0(Sun)-7(Sat) to check for other days.
```

The following are independent queries ran to explore the data and try to understand more about how they were related:

```
SELECT COUNT(country), country
FROM all_sessions
GROUP BY country
ORDER BY country --136 different countries including "not set" with 24 records

SELECT COUNT(city), city
FROM all_sessions
GROUP BY city
ORDER BY city --266 different cities including "not set" with 354 records and 8302 not available in demo dataset

------------------------------------------------------------------------
SELECT	COUNT(DISTINCT (v2_product_name)) AS product_count,
		'all_sessions' AS table_names
FROM all_sessions   -- 471 rows

UNION

SELECT	COUNT(DISTINCT (productname)) AS product_count,
		'products' AS table_names
FROM products   -- 313 rows
------------------------------------------------------------------------
SELECT	v2_product_name
FROM all_sessions
WHERE v2_product_name NOT IN (SELECT productname
			 FROM products)
GROUP BY v2_product_name -- 379 product names that are not included in the products table.
------------------------------------------------------------------------
SELECT	v2_product_name
FROM all_sessions
WHERE v2_product_name IN (SELECT productname
			 FROM products)
GROUP BY v2_product_name -- 92 rows, only this many products have been ordered and that's why they are included in the products table.
-----------------------------------------------------------------------
SELECT DISTINCT(productname)
FROM products
WHERE ordered_quantity = 0  -- There are 172 products here that have not been ordered. 
------------------------------------------------------------------------

SELECT	DISTINCT(v2_product_name)
FROM all_sessions
WHERE v2_product_name NOT IN (SELECT productname
			      FROM products)
ORDER BY v2_product_name -- 379 distinct products in all_sessions table that are not included in the products table

SELECT DISTINCT(productname)
FROM products
WHERE productname NOT IN (SELECT v2_product_name
			  FROM all_sessions)
ORDER BY productname -- 221 distinct products in products table that are not included in the all_sessions table

------------------------------------------------------------------------

SELECT	DISTINCT(v2_product_name)
FROM all_sessions
WHERE v2_product_name IN (SELECT productname
			  FROM products)
ORDER BY v2_product_name -- 92 distinct products in all_sessions table that are included in the products table

SELECT DISTINCT(productname)
FROM products
WHERE productname IN (SELECT v2_product_name
		      FROM all_sessions)
ORDER BY productname -- 92 distinct products in products table that are included in the all_sessions table

------------------------------------------------------------------------

--------------- Check products from all_sessions and sales_report tables ---------------
SELECT	DISTINCT(v2_product_name)
FROM all_sessions
WHERE v2_product_name IN (SELECT productname
		          FROM sales_report)
ORDER BY v2_product_name -- 77 distinct products in all_sessions table that are included in the sales_report table

SELECT DISTINCT(productname)
FROM sales_report
WHERE productname IN (SELECT v2_product_name
		      FROM all_sessions)
ORDER BY productname -- 77 distinct products in products table that are included in the all_sessions table

------------------- Check products from sales_report and products ---------------------
SELECT	DISTINCT(productname)
FROM products
WHERE productname IN (SELECT productname
		      FROM sales_report)
ORDER BY productname -- 237 distinct products in products table that are included in the sales_report table

SELECT DISTINCT(productname)
FROM sales_report
WHERE productname IN (SELECT productname
		      FROM products)
ORDER BY productname -- 237 distinct products in products table that are included in the all_sessions table

-----------------------------

SELECT	DISTINCT(productname)
FROM products
WHERE productname NOT IN (SELECT productname
			 FROM sales_report)
ORDER BY productname -- 76 distinct products in products table that are NOT included in the sales_report table

SELECT DISTINCT(productname)
FROM sales_report
WHERE productname NOT IN (SELECT productname
		          FROM products)
ORDER BY productname -- 0 distinct products in sales_report table that are NOT included in the products table

-- I should be looking at the product_skus or names, and the total_ordered from sales_report for revenue -----
```

Here are other queries that were used to explore the data in some of the columns, they were helpful in figuring out how many NULL values for each of the attributes in comparison to the total number of records. They are all commented out as they were mainly exploratory and deeper explorations were done, which are all specified in the queries above or in the *cleaning_data.md* document. 
   
```
SELECT total_transaction_revenue
FROM all_sessions 
WHERE total_transaction_revenue IS NOT NULL -- 81 rows

SELECT transactions
FROM all_sessions 
WHERE transactions IS NOT NULL -- 81 rows, 1 transaction only for all. 

SELECT product_quantity
FROM all_sessions 
WHERE product_quantity IS NOT NULL -- 53 rows

SELECT product_price
FROM all_sessions 
WHERE product_price IS NOT NULL -- 15134 rows

SELECT DISTINCT(product_sku)
FROM all_sessions 
WHERE product_sku IS NOT NULL -- 15134 rows no distinct, distinct 536 rows
 
SELECT product_revenue
FROM all_sessions 
WHERE product_revenue IS NOT NULL -- 3 rows

SELECT product_sku
FROM all_sessions
WHERE product_sku IS NOT NULL -- 15134 rows

SELECT item_quantity
FROM all_sessions
WHERE item_quantity IS NOT NULL -- 0 rows

SELECT item_revenue
FROM all_sessions
WHERE item_revenue IS NOT NULL -- 0 rows

SELECT transaction_revenue
FROM all_sessions
WHERE transaction_revenue IS NOT NULL -- 4 rows

SELECT transaction_id
FROM all_sessions
WHERE transaction_id IS NOT NULL -- 9 rows

SELECT unit_price
FROM analytics
WHERE unit_price IS NOT NULL -- 4,301,122 rows

SELECT units_sold
FROM analytics
WHERE units_sold IS NOT NULL -- 95147 rows
```


