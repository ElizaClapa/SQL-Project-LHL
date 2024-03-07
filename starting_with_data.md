# Question 1: What is the percentage of visitors that actually make an order compared to those who don't? 

## SQL Queries: 

To answer this question, a CTE named *ordered* was created to obtain the count of visitors who had placed orders, the *sales_report* was joined to the *all_sessions_cleaned* and *analytics_aggregated* VIEWS to obtain the total number of products ordered. The left join was used to also considered those customers that have been active but have not placed any orders.

*all_sessions_cleaned* VIEW: 
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
*analytics_aggregated* VIEW:
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
Query with CTE *ordered*:
```
WITH ordered AS (
	SELECT	ag.visit_id, 
		sc.full_visitor_id,  
		SUM(sr.total_ordered) AS total_ordered_sum,
		ROW_NUMBER() OVER (PARTITION BY (CASE WHEN SUM(sr.total_ordered) = 0 THEN 'zero'
                           		 	      ELSE 'nonzero'
                         		 	  END) ORDER BY (SUM(sr.total_ordered)))AS visitor_count
	FROM analytics_aggregated AS ag
	LEFT JOIN all_sessions_cleaned AS sc USING (full_visitor_id)
	JOIN sales_report AS sr USING (product_sku) 
	GROUP BY ag.visit_id, sc.full_visitor_id
	)

SELECT	MAX(visitor_count) AS customers_count,
	'No' AS Ordered
FROM ordered
WHERE total_ordered_sum = 0
UNION 
SELECT	MAX(visitor_count) AS customers_count,
	'Yes' AS Ordered
FROM ordered
WHERE total_ordered_sum > 0
```
## Answer: 

The total number of visitors that have placed orders is more than 70% of the total number of visitors. Which means that more than half of the people actively visiting the websites are engaging and making purchases.   

TABLE 1.- Visitors with orders  vs Visitors with no orders    

| Visitors Count |Ordered    | Percentage 
|:----------     |:----------|:----------|
| 797            | No        | 28%       |
| 2006           | Yes       | 72%       |




# Question 2: What are the most viewed products that have not been purchased?  

## SQL Queries:

Using the following query, the product name was obtained by joining the *products* table to the *sales_report* table and the *all_sessions_cleaned* and *analytics_aggregated* VIEWS. 

The query is meant to obtain those products directly related to *visitor_id* and *full_visitor_id*s from the *analytics_aggregated* VIEW, which gives information about the online sessions from each visitor. 

```
SELECT	p.productname,
	COUNT(*)
FROM analytics_aggregated AS ag
LEFT JOIN all_sessions_cleaned AS sc USING (full_visitor_id)
JOIN sales_report AS sr USING (product_sku) 
JOIN products AS p 
	ON sr.product_sku = p.sku
GROUP BY p.productname
HAVING SUM(sr.total_ordered) = 0
ORDER BY COUNT(*) DESC
```

## Answer:
The top 10 products that have not been ordered after viewing them are a variety of accesories and clothing. Being the Men's Vintage Tank one of the most viewed but not ordered products. 

Flashlights would be the 10th most viewed prodcuts that have not been ordered. Although in table 2 only the top 10 products are presented, the total number of products goes up to 56 distinct products. 

TABLE 2.- Top 10 viewed products but not ordered. 
![](Image)


# Question 3: What are the days of the week where there is a higher activity online? 

## SQL Queries:
To answer this question the *analytics_aggregated* VIEW was used to obtain the total *page_views* and *time_on_site* per day of the week. All dates with the same day of week were aggregated and grouped. Saturday was not included in the CASE statement because during the QA process, it was concluded that there were no dates with that day of the week. 

```
SELECT	CASE EXTRACT(DOW FROM visit_date)
     		WHEN 1 THEN 'Sunday'
   		WHEN 2 THEN 'Monday'
     		WHEN 3 THEN 'Tuesday'
     		WHEN 4 THEN 'Wednesday'
     		WHEN 5 THEN 'Thursday'
     		WHEN 6 THEN 'Friday'
   	 	END AS day_of_week,
		COUNT(*) AS count_of_dates,
		SUM(page_views_sum) AS page_views_per_day,
		SUM(time_on_site_sum) AS time_on_site_per_day
FROM analytics_aggregated
WHERE visit_date IS NOT NULL
GROUP BY day_of_week
ORDER BY SUM(time_on_site_sum) DESC
```

# Answer:
The day of the week that seems to be the most active in terms of time spent on the site per day is Sunday. Followed by Monday and Tuesday. The least day visitors spend time in the website is Friday, maybe because after a long week of work a lot of people are looking forward to going out, like grabbing dinner or going to the movie theather.

In table 3, it is observed that there is a NULL value, which means that the data is not completely clean and explored further to find the best way to work with those values, or dropp them if necessary. However, when exploring the data from the *analytics_aggregated* to look for any null values in the *visit_date* attribute, none of the them were NULL. The data was reviewed as well to see if there were any values that did not meet the date format YYYY-MM-DD, but nothing outside of that was found. So the NULL value could not be explained.  
  
TABLE 3.- Visitors activity measured by page views and time on site per each day of the week  
![](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Day%20of%20week%20with%20page%20views%20and%20time%20on%20site.png?raw=true)
