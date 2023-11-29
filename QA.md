What are your risk areas? Identify and describe them.

### Risk Areas
1. Assumptions
   
  i. Since there was limited information about the ecommerce database, there were many assumptions made about the data. The most major assumptions revolved around the visitid and fullvisitid. These were assumed to be users from a specific machine accessing the website since most of the visitid were tied to a specific country and city. However, the risk is if the visitid does not reflect this assumption, the conclusions drawn may not be applicable at all. The visit id was also assumed to be a sort of combination PK along with date and timeonsite. Much of the data cleaning stemmed from this assumption including the decision to join analytics data with all_sessions data.
  
  ii. In this joined data (visitortransactions) it was assumed that much of the transaction records were lost. Many columns had revenues reported by not product quantity and visa versa. The assumption that these columns should be filled in were acted upon during the cleaning phase where subsequently, the data was calculated and inserted back into the table.

  iii. For the products, it was assumed that if products shared the same sku, it was in fact the same product despite other details not matching up including orderedquantity, category and in rare cases, revenue. 

  iv. Other assumptions for the data but were of less importance included the time on site was reported in seconds, $ figures were all supposed to be divided by 1,000,000 , the calculated item quantities were rounded down to the nearest whole number to account for accounting discrepancies, the date was ordered by YYYYMMDD and not by the number of seconds that had passed since a reference date and that sentiment score and sentiment magnitude could be multipled together to get a general preference level.

2. NULL Values

   Many null values existed within this database, this incomplete data set could ultimately affect the results that were gathered from the queries. Though the cleaning process was able to address some of these missing values, there were still many incomplete fields in the cleaned tables. This unfortunately could not be helped as the original data was in bad organizational shape to begin with. Null values that would have a detrimental impact on the analysis queries that were preformed would include: missing both revenue and quantity fields, missing country and / or city fields. Other lesser fields that were still prone to having null values included: timeonsite, pageviews, and sentiment values. Because of the amount of inconsistent data, the validity of the queries should be called into question since even the accuracy of the values that were present, can not be validated and to some extent - trusted.


### QA Process:

The schema of the ecommerce database consists of 5 tables:
all_sessions
analytics
products
sale_by_sku
sales_report

Upon investigation of the data, sales_by_sku, and sales_report will not be considered since it is full of repeated data from other tables. Using the below query, we are able to check each column to see if the column is fully null

      eg.
      SELECT productrefundamount
      FROM all_sessions
      WHERE productrefundamount is not null

Many columns were deleted from the all_sessions and analytics tables. No unique values were found in the remaining 3 tables. 'products.sku' was almost unique and was cleaned to become the PK for the 'productdistinct' table (see cleaning). Conversely, in the all_sessions and analytics table, a combination PK of visitid, date and timeonsite were used to determine different instances of a user visiting the website. With this understanding, the new tables 'analytics2' and ' visitortransactions' were created. 

Since visitor ids were assumed to be tied to the individual, the same user would be able to visit the website more than once. This follow query allows manuel spot checking to make sure that visitors are repeating their visits but ensuring different data is collected (ensuring there are no repeats from a single user on a single visit)

      SELECT *
      FROM visitortransactions
      WHERE visitorid IN (
            SELECT visitorid
            FROM visitortransactions
            GROUP BY visitorid
            HAVING COUNT(*) > 1
      ) ORDER BY visitorid   

Unfortunately, despite our best cleaning efforts not everything could be fixed. Queries like the one below confirms that there are many instances in which columns are blank, potentially leading to issues with analysis.

      SELECT timeonsite 
      FROM visitortransactions 
      WHERE timeonsite is null

In this specific case however, there are no overlaps when a transaction takes place, nullifying the adverse effects that this case may of had.

      SELECT timeonsite 
      FROM visitortransactions 
      WHERE timeonsite is null
            AND revenue is not null

Some cases could not be avoided like sentimentscore and sentimentmagnitude.

      SELECT * FROM productdistinct
      WHERE sentimentscore is null

Among these transgressing columns are the following:  visitortransactions.city,
                                                      visitortransactions.sku,
                                                      productdistinct.sentimentscore,
                                                      productdistinct.sentimentmagnitude
In general, these columns do have some effect on analysis questions and query results but on the whole do not alter the conclusions by too much. They can be dismissed with the understanding that the actual answer to questions may be skewed slightly. With the understanding that these null values are in check, there can still be a semblance of trust in the final query results. 

Other metrics in which to validate our data quality are to check for outliers. Using this example query, no unexpected values were found.

      SELECT * FROM visitortransactions 
      WHERE unitprice  < '0' OR unitprice > '1000'

Spot checks were utilized to make sure that cleaning calculations yielded the correct answer. These were important in the visitortransactions table concerning the timeonsite (seconds changed to HHMMSS format), date formatting, product quantity, unit price and revenue calculations. The PK of the new tables were verified to have unique values. This was done by cross referencing the total number of returned rows to the following example query.

      SELECT COUNT(DISTINCT id)
      FROM visitortransactions

      --returns 62448 = number of rows returned in

      SELECT *
      FROM visitortransactions

However, due to limited time, the rest of the tables were not remade. The original tables are still full of quality issues. This is addressed in the README file under future considerations. The original tables were also not altered to protect the original source of data for future use. However, with the above validity of the 2 new tables: 'visitortransactions' and 'productdistinct', we can be fairly confident of our findings to our questions through queries.
