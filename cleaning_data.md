# What issues will you address by cleaning the data?

1) A lot of information was not available, or was null from a few columns. The cleaning process will focus first on cleaning the data by making VIEWS that contain the information needed to calculate revenue, excluding the nullable records or records that are 'not available in demo dataset' or '(not set)'. 

2) The table *all_sessions* had a lot of duplicates, and the table itself did not have a primary key. Therefore, a VIEW called *all_sessions_cleaned* was created to only include the *full_visitor_id*, *product_sku*, *city* and *country*, which is information relevant to the questions to be answered in this project. The code to do this aimed to eliminate any duplicates in the data that was grouped by the attributes just mentioned. 

3) When analyzing the product categories, the overwhelming number of categories made it hard to draw any insights from the data. To try to fix this problem, another VIEW was created, *condesed_categories*, in which all categories that shared the same keywords (such as *Apparel* or *Drinkware*) where considered and condensed into one category.  

4) The VIEW overall_revenue was created to assist in the calculation of the sum of all countries' revenues and then the percentage per country without considering any of the countries that do not have any revenue.

5) The analytics table had over 4 million records and a lot of those attributes were mainly null. Cleaning this table by getting rid of all NULLS would have gotten rid of most of the tables records. 

# Queries: below, provide the SQL queries you used to clean your data.

1) Creating a VIEW to clean the data needed to obtain revenue. With this table only those cities and countries that have a valid name would be considered in all of the following queries used to answer questions in the *Starting with questions* document.

Prices were also divided by 1,000,000 as recommended in the *Cleaning hint* from the *assigment.md* document. They were also rounded to only 4 decimals to avoid long numbers in the outputs. 

```
CREATE OR REPLACE VIEW for_revenue AS
	SELECT		s.country,
			s.city,
			s.product_sku,
			ROUND(s.product_price/1000000, 4) AS price,
			sr.total_ordered
	FROM 	all_sessions AS s
	JOIN	sales_report AS sr
		ON sr.product_sku = s.product_sku
	WHERE	s.city NOT IN ('(not set)', 'not available in demo dataset') 
```

2) VIEW *all_sessions_cleaned* 
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

3) After a few query iterations, all categories were able to be condensed into only 18 different groups. The final queries to obtain that are as follows: 
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

4) The VIEW *overall_revenue*:

```
CREATE VIEW overall_revenue AS
SELECT	country,
		ROUND(SUM(price * total_ordered)) AS Revenue,
		DENSE_RANK() OVER (ORDER BY (SUM(price * total_ordered))DESC) AS Rank_Rev
FROM for_revenue
GROUP BY country
HAVING SUM(price * total_ordered) > 0
ORDER BY DENSE_RANK() OVER (ORDER BY (SUM(price * total_ordered))) DESC
```

5) The following query was written to try to aggregate as much information as possible. All NULL values were minimized by changing the NULL values to zeros for those attributes to be aggregated with the SUM function, that way they would be summed together when they were grouped by the *visit_id*, *visit_start_time*, *visit_date*, *full_visitor_id*, *channel_grouping*, and *social_engagement_type* attributes. The attribute *user_id* was completely dropped from the table because it was verified that all its records were NULL. 

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



