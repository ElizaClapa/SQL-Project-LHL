# ANSWER THE FOLLOWING QUESTIONS AND PROVIDE THE SQL QUERIES USED TO FIND THE ANSWER
 
# **Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

## SQL Queries:  

Information needed to calculate revenue was joined, excluding cities not available or null, from the tables *all_sessions* and *sales_report*, joining the following columns from each table, *country*, *city*, *product_sku*, *product_price* (divided by 1,000,000) and *total_ordered*. 

A VIEW *for_revenue* was created to help calculate revenue. 

``` 

CREATE OR REPLACE VIEW for_revenue AS
	SELECT	s.country,
		s.city,
		s.product_sku,
		ROUND(s.product_price/1000000, 4) AS price,
		sr.total_ordered
	FROM 	all_sessions AS s
	JOIN sales_report AS sr
		ON sr.product_sku = s.product_sku
	WHERE	s.city NOT IN ('(not set)', 'not available in demo dataset') 
```
Using the VIEW above the revenue was calculated with the following query:
```
SELECT	country,
	city, 
	TO_CHAR((SUM(price * total_ordered)), '9G999G999.99') AS revenue
FROM	for_revenue
GROUP BY country, city
HAVING	SUM(price * total_ordered) > 0
ORDER BY revenue DESC
LIMIT 10
```

## Answer:
The top 10 cities with the highest level of transaction revenues on site are the following:
 
TABLE 1 .- Top 10 cities  

|Rank |City | Country  | Revenue |
|:----------|:----------|:----------|:----------|
|1|Mountain View   | United States   |$1,251,427.73    |
|2|San Francisco    | United States    | $378,883.06   |
|3|Sunnyvale  | United States    | $351,486.09   |
|4| Palo Alto    | United States     | $326,388.91    |
|5|New York   | United States     | $310,295.77   |
|6|San Jose    | United States   | $155,708.35   |
|7|London    | United Kingdom   | $140,183.12     |
|8|Chicago   | United States     | $133,738.78   |
|9| Seattle    | United States     | $112,987.16     |
|10| Santa Clara   | United States    | $111,243.45   |

Most cities are located in United States, being London, from United Kingdom, the only exception. 


# **Question 2: What is the average number of products ordered from visitors in each city and country?**


## SQL Queries:  

A Common Table Expression (CTE) was created to calculate the average number of products per customer in each country and city. In the CTE, a window function was used to get the average across all customers in the different country and city groups.    

As expected using the window function, the results were repetitive in the column *average_products_per_visitor* because the average is the same for each group, and because the results are grouped by visitorid as well. 

With the CTE, the information needed was easily accesible without changing the average calculated previously, and grouping the average per country and city. It further allowed to filter the results to include only the records where the average number of products was above 0. 

The CTE calculated the average number of products per country and city considering and including all visitors even if they have not ordered any products. 

To only consider those customers who have ordered products, the following code can be added in the WHERE clause: ```AND sr.total_ordered > 0```. This will make the averages increase slightly. 
```
WITH avg_product_per_customer AS (
	SELECT	s.country,
		s.city,
		s.full_visitor_id AS visitorID,
		SUM(sr.total_ordered) AS total_ordered_per_customer,
		ROUND(AVG(SUM(sr.total_ordered)) OVER (PARTITION BY s.country, s.city), 2) AS average_products_per_visitor
	FROM 	all_sessions AS s
	JOIN sales_report AS sr
		ON sr.product_sku = s.product_sku
	WHERE	s.city NOT IN ('(not set)', 'not available in demo dataset')
	GROUP BY s.country, s.city, s.full_visitor_id
	)

SELECT	country,
	city,
	average_products_per_visitor
FROM avg_product_per_customer
WHERE average_products_per_visitor > 0
GROUP BY country, city, average_products_per_visitor
ORDER BY country, city

```
The following query was considered to answer this question. However, it was decided not to use it because the average was calculated by each group of city and country, which didn't consider the number of visitors to get the average, basically only summing of all ordered products per city and country.


``` 
SELECT	s.country,
		s.city,
		SUM(sr.total_ordered) AS total_ordered_per_customer,
		ROUND(AVG(SUM(sr.total_ordered)) OVER (PARTITION BY s.country, s.city), 2) AS average_products_per_visitor
	FROM 	all_sessions AS s
	JOIN sales_report AS sr
		ON sr.product_sku = s.product_sku
	WHERE	s.city NOT IN ('(not set)', 'not available in demo dataset')
	GROUP BY s.country, s.city

```
## Answer:  

The query with the CTE above was edited to order by *average_products_per_visitor* in descending order and only get the first 10 rows: 

TABLE 2 .- Average products per visitor

| Top 10 | Country  | City  | Average  |
|:----------|:----------|:----------|:----------|
|1| Saudi Arabia   | Riyadh   | 319.00    |
|2| Czechia    | Brno   | 319.00    | 
|3| United States  | Rexburg   | 250.50    | 
|4| United States   | Avon   | 200.00    |
|5| United States   | Sacramento    | 189.00   |
|6| Portugal    | Lisbon   | 189.00    |
|7| Italy   | Rome    | 130.33    |
|8| United States   | Kalamazoo    | 105.00    |
|9| Russia|Saint Petersburg|101.25|
|10|Taiwan|Longtan District|97.00|

Now the bottom 10:

1) United States - Kansas City with an average of 0.67.
 
All of the following have an average of 1:

2) United States - Greer
3) France - Villeneuve-d'Ascq
4) Pakistan Lahore
5) Norway - Oslo
6) Australia - Mountain View
7) United States - Piscataway Township
8) Argentina - Rosario
9) Lithuania - Vilnius
10) India - Patna

# **Question 3:  Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

To answer this question a few queries were ran to explore all the different categories to then try to condense them to draw insights from a more concise data. 

## SQL Queries:    
Firstly, all the names of the different categories in the *all_sessions* table was explored with the following query: 
``` 
SELECT	v2_product_category,
	COUNT (*)
FROM all_sessions
GROUP BY v2_product_category
ORDER BY v2_product_category NULLS FIRST 
```

Which resulted in 74 different category names, including *(not set)* and *${escCatTitle}*, which will not be taken into consideration when checking for patterns, so they will be filtered out.

TABLE 3.- Category Names   
![category names](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Category%20names.png?raw=true)

The first query used try to find any patterns in the types of products ordered in each city and country was the following:

``` 
-- To get Category Names
SELECT	s.v2_product_category,
	fr.country,
	fr.city, 
	COUNT(*) AS num_oders
FROM for_revenue AS fr
JOIN all_sessions AS s USING (product_sku)
WHERE s.v2_product_category NOT IN ('(not set)', '${escCatTitle}')
GROUP BY	s.v2_product_category,
		fr.city,
		fr.country
ORDER BY s.v2_product_category, COUNT(*) DESC
```

With this query, the resulting table contains over 3,600 rows, which makes it hard to find a pattern with such a big volume of data. It could definitely be done with visualization tools, however, the project is focused on using SQL as a tool to answer this questions. Therefore, it was considered that it was best to try exploring other patters using only SQL. 

In the resulting table output, *Category_Names*, it was noted that a lot of the category names shared the same type of words, such as *Apparel* or *Drinkware*. After a few query iterations, all categories were able to be condensed into only 18 different groups. The final query is as follows: 
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

While working in the CASE clause from the query above, the following query was used to see how many categories resulted from the modified VIEW *condensed_categories* as well as the sum of the *category_count*, which will be discussed further more in the next pattern analysis:

```
SELECT	category_name, 
	SUM(category_count) AS total_category_count
FROM condensed_categories
GROUP BY category_name
ORDER BY SUM(category_count) DESC
```

The following image is the result of the previous query using the view. 

TABLE 4.- Condensed Category Names  
![condensed category names](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Condensed%20category%20names.png?raw=true)

After each iteration of the code above, the CASE clause was modified to further decrease the number of categories. Every time an extra string had to be considered in the CASE statement, or two categories needed to be condensed into one, the following query was used to delete the view created after making the pertaining modifications: 
```
DROP VIEW condensed_categories
```

The latter to avoid using up too much space and to easily modify the code. After typing the modifications, the VIEW was created again and then the code above was ran again to get the new results.


Following this process was helpful in noticing those categories that were not very popular among visitors. Moving forward, only those categories with the highest counts, will be investigated to see if any existing patterns are easily spotted. 

Categories to be analyzed:
1) Apparel
2) Brands
3) Electronics
4) Home Office
5) Home Accessories
6) Bags & Backpacks
7) Drinkware
8) Lifestyle

#### Apparel Category Products Per Country
This query joins the *all_sessions* table with the VIEW first presented in this document *for_revenue* to obtain the number of products ordered by customers per country and in the Apparel Category.
 
```
SELECT	fr.country,
	COUNT(*) AS apparel_count
FROM for_revenue AS fr
JOIN all_sessions AS s USING (product_sku)
WHERE s.v2_product_category NOT IN ('(not set)', '${escCatTitle}') AND 
      s.v2_product_category ILIKE '%apparel%' OR s.v2_product_category ILIKE '%wearables%'
GROUP BY fr.country
ORDER BY COUNT(*) DESC
LIMIT 5
``` 

#### Brands Category Products Per Country
This query is essentially the same query as above, using the VIEW *for_revenue* as well. The difference lies in the WHERE clause, used to filter the data and only show those with the specified conditions.
```
SELECT	fr.country,
	COUNT(*) AS brands_count
FROM for_revenue AS fr
JOIN all_sessions AS s USING (product_sku)
WHERE s.v2_product_category NOT IN ('(not set)', '${escCatTitle}') AND 
	  s.v2_product_category ILIKE '%brand%' OR s.v2_product_category ILIKE '%YouTube%'
GROUP BY	fr.country
ORDER BY COUNT(*) DESC
LIMIT 5
```

The following queries will also modify the WHERE clause with the conditions to meet each of the categories to be analyzed.

#### Electronics Category Products Per Country
```
SELECT	fr.country,
	COUNT(*) AS electronics_count
FROM for_revenue AS fr
JOIN all_sessions AS s USING (product_sku)
WHERE s.v2_product_category NOT IN ('(not set)', '${escCatTitle}') AND 
	  s.v2_product_category ILIKE '%electronics%' OR s.v2_product_category ILIKE '%nest%'
GROUP BY	fr.country
ORDER BY COUNT(*) DESC
LIMIT 5
```


#### Home Office Category Products Per Country
```
SELECT	fr.country,
	COUNT(*) AS home_office_count
FROM for_revenue AS fr
JOIN all_sessions AS s USING (product_sku)
WHERE s.v2_product_category NOT IN ('(not set)', '${escCatTitle}') AND 
	  s.v2_product_category ILIKE '%office%'
GROUP BY	fr.country
ORDER BY COUNT(*) DESC
LIMIT 5
```

#### Home Accessories Category Products Per Country
```
SELECT	fr.country,
	COUNT(*) AS home_accessories_count
FROM for_revenue AS fr
JOIN all_sessions AS s USING (product_sku)
WHERE s.v2_product_category NOT IN ('(not set)', '${escCatTitle}') AND 
	  s.v2_product_category ILIKE '%accessories%' AND s.v2_product_category NOT LIKE '%electronics%'
GROUP BY fr.country
ORDER BY COUNT(*) DESC
LIMIT 5
```

#### Bags & Backpacks Category Products Per Country
```
SELECT	fr.country,
	COUNT(*) AS bags_backpacks_count
FROM for_revenue AS fr
JOIN all_sessions AS s USING (product_sku)
WHERE s.v2_product_category NOT IN ('(not set)', '${escCatTitle}') AND 
	  s.v2_product_category ILIKE '%bags%'
GROUP BY fr.country
ORDER BY COUNT(*) DESC
LIMIT 5
```


#### Drinkware Category Products Per Country
```
SELECT	fr.country,
	COUNT(*) AS drinkware_count
FROM for_revenue AS fr
JOIN all_sessions AS s USING (product_sku)
WHERE s.v2_product_category NOT IN ('(not set)', '${escCatTitle}') AND 
	  s.v2_product_category ILIKE '%bottles%' OR s.v2_product_category ILIKE '%drinkware%'
GROUP BY fr.country
ORDER BY COUNT(*) DESC
LIMIT 5
```

#### Lifestyle Category Products Per Country
```
SELECT	fr.country,
	COUNT(*) AS lifestyle_count
FROM for_revenue AS fr
JOIN all_sessions AS s USING (product_sku)
WHERE s.v2_product_category NOT IN ('(not set)', '${escCatTitle}') AND 
	  s.v2_product_category ILIKE '%lifestyle%'
GROUP BY fr.country
ORDER BY COUNT(*) DESC
LIMIT 5
```

## Answer:

#### Apparel Category Products Per Country
In Table 5, products in the apparel category, such as infant, kids, youth, man and woman clothing (including outwear, performance wear, t-shirts and headgear); are most popular in the United States and India. Being these two countries significantly higher than any other country, as the number of products ordered by visitors is a 5-figure number, while the rest of the countries have 4-figure numbers or less. India, with 12,112 products ordered from the apparel category, is about 7,000 products over the number of products ordered by Canada, the third highest. However, United States is the country where visitors consume the most in this category, with 92,172 products ordered, it is more than 5 times the products that are sold in India.

It would be worth looking into what are the factors that make United States so much higher than the rest of the countries. 

TABLE 5.- Apparel Category Products Per Country   
![apparel table](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Apparel%20pattern%20per%20country.png?raw=true)


#### Brand Category Products Per Country

In table 6, United States comes up as the highest consumer of products sold by brands such as Youtube, Google, Android and Waze. India again follows in second place, being United Kingdom, Australia, and Canada in 3rd, 4th, and 5th place, respectively.

TABLE 6.- Brand Category Products  
![brand table](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Brand%20pattern%20per%20country.png?raw=true) 

#### Electronics, Home Office, Home Accessories, Bags & Backpacks, Drinkware, and Lifestyle Categories Per Country

For all other categories (Tables 7 to 12), an overall pattern is observed where United States (U.S.) has the most consumers compared to other countries. India can also be seen in second place for most of the categories, except in the Electronics (Table 7) and Bags & Backpack (Table 10) categories, where it is placed in 5th and 4th place, respectively. Canada being the second place for the Electronics category (by almost 40 times less than U.S.) and United Kingdom for the Bags & Backpacks category (by 20 times less than U.S.). 

  
TABLE 7.-  Electronics Category Products Per Country  
![electronics table](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Electronics%20pattern%20per%20country.png?raw=true)

TABLE 8.-  Home Office Category Products Per Country  
![home office table](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Home%20Office%20pattern%20per%20country.png?raw=true)
 
TABLE 9.-  Home Accessories Category Products Per Country  
![home accessories table](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Home%20Accessories%20patter%20per%20country.png?raw=true)
  
TABLE 10.-  Bags & Backpacks Category Products Per Country  
![bags tables](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Bags%20&%20Backpacks%20pattern%20per%20country.png?raw=true)
 
TABLE 11.-  Drinkware Category Products Per Country  
![drinkware table](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Drinkware%20pattern%20per%20country.png?raw=true)
 
TABLE 12.-  Lifestyle Category Products Per Country  
![lifestyle table](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Lifestyle%20pattern%20per%20country.png?raw=true)

There are a lot other approaches that could be taken to find out if there are any patterns in the different products in the categories. Looking into what the sales behaviour is considering developed, and developing, countries separately would be interesting to explore: are there any categories that are sold more in one or the other? Once any patterns are found, researching what are the determining factors, if any, causing these behaviours should be considered. 

Another approach could involve investigating the sales behaviours in a single country, grouping the data by city, and see which cities have the most consumers. Maybe cities with the highest population or cities with a good-standing level of economic development.

Lastly, another approach could be looking into how many customers are per country, without considering each one of their orders, just the count of customers per country. The exploration could focus on the number of customers in countries with a larger landmass or with a high population density. Are customers buying several products repeatedly or are there many new customers buying products for the first time? Should certain brands focus their efforts to reach consumers in a country where a customer repeatedly buys products, or to countries where many new consumers could potentially make a new purchase? 
 
# **Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


## SQL Queries:  
The following query was used to verify that the products column had valid product names and not nulls or unavailable records is:

```
SELECT v2_product_name,
       COUNT (*)
FROM all_sessions
GROUP BY v2_product_name
ORDER BY v2_product_name NULLS FIRST 
```

It seemed like there were no products with unavailable names or nulls. 

After confirming that none of the records needed to be filtered out, the following queries were used:

A CTE was first created to get the ranking of all products, using the ROW_NUMBER window function grouped by country and ordered by the sum of ordered products. In this CTE, only specific countries were included, and the reason why will be explained later on.  
```
WITH ranked_products AS (
    SELECT	s.v2_product_name AS product,
        	r.country,
        	TO_CHAR(SUM(r.total_ordered), '9G999G999G999') AS total_ordered_sum,
        	ROW_NUMBER() OVER (PARTITION BY r.country ORDER BY SUM(r.total_ordered) DESC) AS rank
    FROM for_revenue AS r
    JOIN all_sessions AS s USING (product_sku)
    WHERE	r.total_ordered > 0
        	AND r.country IN ('United States', 'Russia', 'India', 'Algeria', 'Canada', 'Brazil', 'United Kingdom', 'Australia')
    GROUP BY s.v2_product_name, r.country
)
```
After creating the CTE *ranked_products*, the following query was used to summarized the resulting table by only including those with rank equal to 1. 
```
-- Query using CTE ranked_products
SELECT
    product,
    country,
    total_ordered_sum
FROM
    ranked_products
WHERE
    rank = 1;
```

The following simplified query was used to confirm that the previous query gave the same results:
```
SELECT	s.v2_product_name AS product,
	r.country,
	TO_CHAR(SUM(r.total_ordered), '9G999G999G999') AS total_ordered_sum
FROM for_revenue AS r
JOIN all_sessions AS s USING (product_sku)
WHERE r.total_ordered > 0 AND r.country = 'United States' --> Change the country here for a new result.
GROUP BY s.v2_product_name, r.country
ORDER BY SUM(r.total_ordered) DESC
```
## Answer:

To try to answer this question, only the larger countries per continent were considered:
 
1. Asia: Russia and India 
2. Africa: Algeria
3. North America: Canada  
4. South America: Brazil
5. Antarctica: None (because Antarctica is not a country and is not divided into countries so it is not considered in this exploration)
6. Europe: Russia (because part of Russia is in Europe) and United Kingdom
7. Australia: Australia

United States, India and United Kingdom, were included in the query even though they are not the largest of their continents, because they had most of the consumers for some of the categories that were explored earlier. 

TABLE 13.- Most popular product per country    

| Country  | Product  | Total Ordered  |
|:----------|:----------|:----------|
| United States  | Google 17oz Stainless Steel Sport Bottle    | 476,284    |
| Russia  | YouTube Hard Cover Journal| 14,450   |
| India    | YouTube Hard Cover Journal   |        144,500   |
| Algeria    | No results    | No results    |
| Canada   | YouTube Custom Decals    | 25,320    |
| Brazil  | YouTube Hard Cover Journal   | 14,450   |
| United Kingdom  | 22 oz YouTube Bottle Infuser   | 73,500   |
| Australia   | YouTube Hard Cover Journal   | 57,800   |

# **Question 5: Can we summarize the impact of revenue generated from each city/country?**

## SQL Queries:

To try to answer this question, the VIEW *for_revenue* (created earlier) was used with the following queries:

#### To calculate revenue per City in each Country. 

```
SELECT	country,
		city,
		TO_CHAR(ROUND(SUM(price * total_ordered)), '9G999G999') AS Revenue,
		DENSE_RANK() OVER (PARTITION BY country ORDER BY (SUM(price * total_ordered))DESC) AS Rank_Rev
FROM for_revenue
GROUP BY country, city
HAVING SUM(price * total_ordered) > 0
ORDER BY country, DENSE_RANK() OVER (PARTITION BY country ORDER BY (SUM(price * total_ordered))) DESC
```

#### To obtain only the #1 ranked Cities per Country
```
SELECT	country, 
		city,
		revenue
FROM	(SELECT	country,
		city,
		TO_CHAR(ROUND(SUM(price * total_ordered)), '9G999G999') AS Revenue,
		DENSE_RANK() OVER (PARTITION BY country ORDER BY (SUM(price * total_ordered))DESC) AS Rank_Rev
		FROM for_revenue
		GROUP BY country, city
		HAVING SUM(price * total_ordered) > 0
		ORDER BY country, DENSE_RANK() OVER (PARTITION BY country ORDER BY (SUM(price * total_ordered))) DESC
		)
WHERE rank_rev = 1 
ORDER BY revenue DESC
```

#### To calculate revenue per Country in the World (or across all countries in this dataset).

```
SELECT	country,
	TO_CHAR(ROUND(SUM(price * total_ordered)), '9G999G999') AS Revenue,
	DENSE_RANK() OVER (ORDER BY (SUM(price * total_ordered))DESC) AS Rank_Rev
FROM for_revenue
GROUP BY country
HAVING SUM(price * total_ordered) > 0
ORDER BY DENSE_RANK() OVER (ORDER BY (SUM(price * total_ordered))) DESC
```

The VIEW *overall_revenue* was created to assist in the calculation of the sum of all countries' revenues and then the percentage per country. 

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

The following query was used to calculate the overall revenue from all countries and cities using the VIEW *overall_revenue*:

```
SELECT SUM(revenue)
FROM overall_revenue --> Overall Revenue: $4,290,341
```

Using the VIEW *overall_revenue*, the result from the calculation above ($4,290,341), and the following query, the percentage of individual countries was calculated. 
```
SELECT ROUND(((revenue*100)/4290341))||'%' AS overall_percent_revenue
FROM overall_revenue
WHERE country = 'United States' --> Change name to obtain percentage for a different country.
```
Due to the tables already created to see the ranking of all countries, the purpose of this query is to provide a percentage reference during the discussions in the *Answer* section below, in which only a few countries are mentioned.

With the following query the percentage of the bottom 10 countries was calculated:

```
SELECT SUM(ROUND(((revenue*100)/4290341), 2))||'%' AS overall_percent_revenue
FROM overall_revenue
WHERE country IN ('Lithuania', 'Norway', 'Pakistan', 'Qatar', 'Venezuela', 'Denmark', 'Belgium', 'Finland', 'Hungary', 'Paraguay')```
```
## Answer:

#### Revenue per City in each Country. 

The following table shows a portion of the full output from the query used to obtain this information. It is presented to help visualize how the ranking was calculated. For example, as presented in Table 14, Germany has four different cities, which were ranked according to their individual revenues in comparison to the overall revenue in the country.

From this table is harder to see if there are any patterns, the use of graphs for visualization would be a good place to start.   

TABLE 14.- Ranked Cities per Country based on Revenue  
![rank cities per country](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Ranked%20cities%20per%20country%20based%20on%20revenue.png?raw=true)


#### Top 10 and Bottom 10 of the #1 ranked Cities per Country 

To try to obtain some more insights about which cities contribute the most to the overall revenue across all countries without using any visualization tools, the cities that contribute the most to revenue in their corresponding countries were obtained and they are presented as Top 10 Cities (table 15), and Bottom 10 Cities (table 10). 

Mountain View, United States, is the city that contributes the most ($1,251,428), to the overall revenue compared to all other cities. Vilnius, Lithuania; Lahore, Pakistan; and Oslo, Norway are the cities that contribute the least. 

TABLE 15.- Top 10 Cities ranked \#1 from each Country.    
![top 10 #1 cities](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Top%2010%20%231%20ranked%20Cities%20per%20Country%20based%20on%20revenue.png?raw=true)


TABLE 16.- Bottom 10 Cities ranked \#1 from each Country.   
![bottom 10 #1 cities](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Bottom%2010%20%231%20ranked%20Cities%20per%20Country%20based%20on%20revenue.png?raw=true)


#### Top 10 and Bottom 10 Countries

Taking into consideration the overall revenue (summing the revenue from each city) per country, Table 17 and 18 present the Top 10 and Bottom 10 ranked countries. The major contributing country is United States, which aligns with previous results from this project, contributing 86% of the overall revenue.

The remaining 14% of the revenue comes from the combination of all other countries, as the second and third highest contributors, United Kingdom and India, are only contributing 3% and 2%, respectively. So we can infer that summing all other countries revenues we will get the remaining 9% of the overall revenue.      

TABLE 17.- Top 10 Countries based on Revenue  
![Top 10 Country rev](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Top%2010%20Countries%20based%20on%20revenue.png?raw=true)

Considering all countries from Table 18 as one, they contribute a total of 0.01% to the overall revenue, having close to zero impact per each individual country. 

TABLE 18.- Bottom 10 Countries based on Revenue  
![Bottom 10 Country rev](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Bottom%2010%20Countries%20based%20on%20revenue.png?raw=true)

After all the analyses from different points of view, we can conclude visitors (customers) from United States represent the highest and most significant economic contributors. 





