**Question 1: How many visitors came to the website and how many purchases were made**

SQL Queries:

      SELECT COUNT(visitorid), COUNT(revenue)
      FROM visitortransactions
  
Answer: 

There were a total of over 62,000 visits to the site but only 468 purchases were made. This conversion ratio is even smaller when we address the total number of website visits that were accounted for because their visitid did not have a record of their country or city. 


**Question 2: What correlation, if any, to time spent on the site does whether or not a visitor choose to make a purchase?**

SQL Queries:

      SELECT AVG(timeonsite) 
      FROM visitortransactions

      SELECT COUNT(timeonsite) 
      FROM visitortransactions
      WHERE timeonsite > '00:04:42.379928'        --AVG(timeonsite)
        AND revenue is not null

      SELECT COUNT(timeonsite) 
      FROM visitortransactions
      WHERE timeonsite < '00:04:42.379928'
        AND revenue is not null
      
Answer:

When considering the 'timeonsite' in general, more purchases are made when the user is spending more than the average amount of time ( 4 minutes 42 seconds) as opposed to less than the average amount of time. The ratio is 297 (spending above average time) to 171 (spending below average time).


**Question 3: What correlation, if any, to number of pages viewed does whether or not a visitor chooses to make a purchase?**

SQL Queries:

      SELECT AVG(pageview)
      FROM visitortransactions

      SELECT pageview, COUNT(pageview), COUNT(revenue)
      FROM visitortransactions
      GROUP BY pageview
      ORDER BY pageview DESC
      
Answer:

When considering the number of pages viewed against the prescence of a purchase, most purchases are made on visits that are between 4 to 19 pages viewed. The average number of pages viewed by users is 5. However the mode of pages viewed is 1. In business considerations, this ecommerce site looks to have a low retention rate of interest from its users and may be lacking in attention grabbing stimulus. There also exists outliers in which users view more than 30 pages and have a very high rate of making purchase transactions.


**Question 4: Which products are the visitor's favorites and least favorites?**

SQL Queries:

      SELECT productname, category, sentimentscore * sentimentmagnitude AS preference
      FROM productdetails
      WHERE sentimentscore is not null
        AND sentimentmagnitude is not null
      ORDER BY sentimentscore * sentimentmagnitude DESC

Answer:

The most preferred product is a tie between: "Men's Quilted Insulated Vest Black" and " Women's Colorblock Tee White". With the top 20 most preferred products being of various apparel categories. The least preferred product is the "Women's Convertible Vest-Jacket Sea Foam Green". Therefore it does not seem that there is a pattern in which category of products are generally preferred. Instead, it may make more sense to physically examine the product and better understand the causation of these preference scores given by visitors to the site.




**Question 5: How does the visitor's preference of products affect revenue of those products?**

SQL Queries:

      SELECT productname, category, sentimentscore * sentimentmagnitude AS preference, COUNT(revenue)
      FROM productdetails p
      JOIN visitortransactions v ON p.productsku = v.sku 
      WHERE revenue is not null
        AND sentimentscore is not null
        AND sentimentmagnitude is not null
      GROUP BY productname, category, sentimentscore * sentimentmagnitude
      ORDER BY COUNT(revenue) DESC

      SELECT productname, category, sentimentscore * sentimentmagnitude AS preference, COUNT(revenue)
      FROM productdetails p
      JOIN visitortransactions v ON p.productsku = v.sku 
      WHERE revenue is not null
        AND sentimentscore is not null
        AND sentimentmagnitude is not null
      GROUP BY productname, category, sentimentscore * sentimentmagnitude
      HAVING sentimentscore * sentimentmagnitude < 0         -- ONLY change from previous query
      ORDER BY COUNT(revenue) DESC

Answer:
In general personal preference of a product does not seem to have a major factor of if a certain product is bought or not. However, there is significantly less amount of products that are bought from products that have a negative sentiment. This may be useful in identifying products that need to be improved so that customers are more drawn to purchasing them.
