# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
1) To applied the knowledge obtained during the past 6 weeks in the course about SQL. 
	
	a) To create a Database, importing all the tables data from CSV files. 
	
	b) To understand the data and find any inconsistencies, missing values, invalid values, duplicates and relationships between the tables. 
	
	c) To write queries that answer the questions in this project by joining relevant information from different tables and making sure to use cleaned data and following a Quality Assurance (QA) process to ensure that the data and the results are reliable. 

	d) To create a ERD for the *ecommerce* database created, specifying the types of relationships between tables. 
	
	e) To gain more insights from the data creating 3 more questions, focusing on using tables that we not heavily used to answer previous questions.   
    
2) To learn and gain more SQL coding skills, while implementing and reinforcing the skills already learned. 

3) To learn from other classmates' insights and approaches they took to overcome the challenges in this project. 

4) To gain feedback on the technical skills applied to answer the questions in the project, but also about the overall understanding of the data and what could be learned from it.  

## Process
### 1. Creating the Database *ecommerce* and answer the questions in the project
For this step, making sure that the dataset was created successfully and contained all the values from the CSV files was the first challenge.

After that, cleaning the data seemed to be the first thing to start with, however, this would have taken too long as there is a lot of information that needed to be cleaned. For that reason, queries were written to try to answer the questions first and then, depending on the data needed for this, a cleaning process was implemented to make sure the answers from the queries were pulled from cleaned data. This process of writing queries to answer questions and writing queries to clean the data was repeated back and forth throughout the completion of this project.

The results were then documented in the *starting_with_questions.md* file, including tables and queries used to get the results. During this process, a lot of potential QA approaches came up as the results were explained more thoroughly and risk areas were identified. 

The *cleaning_data.md* document was completed at the same time as working on the *starting_with_questions.md* file. All the VIEWS used to clean the data can be found there and a more detailed explanation on the decision making that resulted in the creation of these VIEWS is presented as well.     

### 3. Obtaining more insights from the data with new questions
For this step, three questions were created to get more information from the data. These questions also focused on using the *analytics* table because other tables were used more during the previous steps. 

### 4. Completing the QA process
The QA process was a working progress during the steps mentioned above. But was completed and documented closer to the end. The project presentations helped to strengthen the QA process as other approaches were taken into consideration. 

### 5. Creating the Entity Relationship Database (ERD) schema
The ERD was created after the QA process to ensure the relationships defined were correct. 

All the cleaning was done using VIEWS as to not alter the raw data in case important information for another team was deleted in the cleaning process. However, the VIEWS were not generated in the ERD schema, so it only includes the original tables. 
 
## Results
### 1) The top 10 cities with the highest level of transaction revenues on site are the following:
 
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

### 2) Patterns in the product categories ordered by visitors per country

In Table 5, products in the apparel category, such as infant, kids, youth, man and woman clothing (including outwear, performance wear, t-shirts and headgear); are most popular in the United States and India. Being these two countries significantly higher than any other country, as the number of products ordered by visitors is a 5-figure number, while the rest of the countries have 4-figure numbers or less. India, with 12,112 products ordered from the apparel category, is about 7,000 products over the number of products ordered by Canada, the third highest. However, United States is the country where visitors consume the most in this category, with 92,172 products ordered, it is more than 5 times the products that are sold in India.

It would be worth looking into what are the factors that make United States so much higher than the rest of the countries. 

TABLE 2.- Apparel Category Products Per Country   
![appareltable](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Apparel%20pattern%20per%20country.png?raw=true)


For all other categories explored in the project, an overall pattern is observed where United States (U.S.) has the most consumers compared to other countries. India can also be seen in second place for most of the categories, except in the Electronics (Table 7) and Bags & Backpack (Table 10) categories, where it is placed in 5th and 4th place, respectively. Canada being the second place for the Electronics category (by almost 40 times less than U.S.) and United Kingdom for the Bags & Backpacks category (by 20 times less than U.S.). 

  
TABLE 3.-  Electronics Category Products Per Country  
![electronicstable](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Electronics%20pattern%20per%20country.png?raw=true)

TABLE 4.-  Bags & Backpacks Category Products Per Country  
![bagstables](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Bags%20&%20Backpacks%20pattern%20per%20country.png?raw=true)

There are a lot other approaches that could be taken to find out if there are any patterns in the different products in the categories. Looking into what the sales behaviour is considering developed, and developing, countries separately would be interesting to explore: are there any categories that are sold more in one or the other? Once any patterns are found, researching what are the determining factors, if any, causing these behaviours should be considered. 

Another approach could involve investigating the sales behaviours in a single country, grouping the data by city, and see which cities have the most consumers. Maybe cities with the highest population or cities with a good-standing level of economic development.

Lastly, another approach could be looking into how many customers are per country, without considering each one of their orders, just the count of customers per country. The exploration could focus on the number of customers in countries with a larger landmass or with a high population density. Are customers buying several products repeatedly or are there many new customers buying products for the first time? Should certain brands focus their efforts to reach consumers in a country where a customer repeatedly buys products, or to countries where many new consumers could potentially make a new purchase? 

### 3. Impact of revenue generated per country
Taking into consideration the overall revenue (summing the revenue from each city) per country, Table 17 and 18 present the Top 10 and Bottom 10 ranked countries. The major contributing country is United States, which aligns with previous results from this project, contributing 86% of the overall revenue.

The remaining 14% of the revenue comes from the combination of all other countries, as the second and third highest contributors, United Kingdom and India, are only contributing 3% and 2%, respectively. So we can infer that summing all other countries revenues we will get the remaining 9% of the overall revenue.      

TABLE 5.- Top 10 Countries based on Revenue  
![Top 10 Country rev](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Top%2010%20Countries%20based%20on%20revenue.png?raw=true)

Considering all countries from Table 6 as one, they contribute a total of 0.01% to the overall revenue, having close to zero impact per each individual country. 

TABLE 6.- Bottom 10 Countries based on Revenue  
![Bottom 10 Country rev](https://github.com/ElizaClapa/SQL-Project-LHL/blob/main/Pictures/Bottom%2010%20Countries%20based%20on%20revenue.png?raw=true)

After all the analyses from different points of view, we can conclude visitors (customers) from United States represent the highest and most significant economic contributors. 

### 4. Top selling product per country

Specific countries were considered to answer this, only those countries with the biggest landmass per continent were considered, plus United States, India and United Kingdom. The latter because they were the highest consumers per category. 

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


## Challenges 
1. There were several problems importing the CSV file into PgAdmin into the tables created there.   

2. Not understanding the meaning of each attribute, specifically the attributes in *all_sessions* table. This became a very important factor in considering the approaches to take with the data, what information to use, and how to clean it. With more information available the results could potentially be very different. 

3. The limited understanding of how specific clauses, functions and overall queries worked was very challenging. Visualizing what was happening when using a query helped, but when the query resulted in an error, it was really hard to figure out why. The timing it took to write one working query was really long, but a lot of learning came out from this.

4. The biggest challenge during this project was to be able to see the full scope of the data and what it could potentially answer. More practice working with data and analyzing it is definitely needed so, with time, it will be easier to identify what questions are worth spending time answering with the data.

5. Another significant challenge was the QA process, not knowing exactly what steps to take to create a process that ensures that the data is reliable. More work in this area should probably be done, and more practice and better understanding about how to do this is worth considering. 
   

## Future Goals
Future goals include studying and learning more about the challenges that were presented in this project. Familiarized myself with other real raw datasets that needs a lot of cleaning and being comfortable with this and knowing how to tackle the cleaning and QA processes. 

Improving the technical skills applied in this project, learning from the feedback to avoid making any mistakes found in this project. 

Modify the queries, or the approaches to obtain the information needed, after reviewing the feedback from experts assessing my project. Comparing the previous results to the modified ones to see the impact of an improved procedure. 

Apply concepts and skills learned before, during, and after this project, to upcoming projects, either academically or professionally. 