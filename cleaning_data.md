      In cleaning the data, many assumptions were needed to be concluded before work could take place. In going through the dataset and understanding the data, one of the first things to be addressed was the original 'analytics' relation. Many of the rows seemed to repeat themselves for each product viewed - leading to redundant data with the only change being different 'unit_price' of the products viewed. This was first addressed to clean excessive data to a more managable size (query 1 + 2) so that only distinct combinations of the data (without the 'unit_price') were kept. Therefore, the relation 'analytics2' was used for any subsequent work instead of the relation 'analytics'

      During the understanding data process, I realized that there were no PRIMARY KEYs to the relations 'all_sessions' and 'analytics' (and therefore 'analytics2'). I decided to tackle this later due to the mess of deciding between a conjuncted PRIMARY KEY or some other iteration. However, what did need to be tackled was that information for 'all_sessions' was somewhat similar to that of 'analytics2' and should be combined to form a more full picture of what the purchases across the ecommerce site looked like. Fortunately, two things made this process a bit easier. Earlier on in the understanding data, I determined that 'visitid' was a more useful metric of determining a user than the 'fullvisitorid'. Therefore I assigned 'visitid' as a foreign key between the two relations. It also happened that a large portion of the 'analytics2' visitid were present within the 'all_sessions' visitid - easing the ability to transform the 'analytics2.visitid's to having their own associated country and city. There were a few outliers and this process will be addressed in the QA part of the project. By combining these two relations with all_sessions still being the main relation, I was able to import all data into 1 relation. Along the way, I also decided to change data_types and change prices to a readable format. A large assumption that I had to make was that if there were a 'all_sessions.totaltransactionrevenue' or 'analytics2.revenue', then there should be a 'all_sessions.productquantity' or 'analytics2.units_sold' respectively and visa versa. This assumption may be incorrect due to not knowing when transaction revenue or quantities are recorded into the data. (more on this in QA / risks). With this assumption however, the 'visitortransactions' relation could be created with the 'revenue' and 'quantity' being calculated from each other. Many extra data points from 'analytics2' were dropped (more in QA) but a cleaned 'visitortransactions' chart was created (query 3 + 4). This relation alone helped me answer couple of the project's required and self made questions.

      To address the issue of the missing PK in 'visitortransactions', a seperate process of adding in a 'websiteviewid' was inserted after the relation was made. This was based on the 'date', 'visitorid', and 'timeonsite' metrics with the assumption that no single user would be on the site more than once per day and log the exact same timeonsite (to the second - more in QA). This allowed the chart to have a PK to identify instances of users visiting this ecommerce website. (query 5 + 6)
    
      In the final step of cleaning data, the products needed to be combined from different sources. The PRIMARY KEY was much easier to identify as the sku and so no further steps were needed to address this issue. 

      





Queries:
------------------------------------------
--query 1 + 2
------------------------------------------
CREATE TABLE analytics2 (
	visitnumber VARCHAR,
	visitid VARCHAR,
	visitstarttime VARCHAR,
	date VARCHAR,
	channelgrouping VARCHAR,
	units_sold VARCHAR,
	pageview VARCHAR,
	timeonsite VARCHAR,
	revenue VARCHAR,
	unit_price VARCHAR
	)

INSERT INTO analytics2 (visitnumber, visitid, visitstarttime, date, channelgrouping, units_sold, pageview, timeonsite, revenue, unit_price)
SELECT DISTINCT
  visitnumber, 
	visitid, 
	visitstarttime, 
	date, 
	channelgrouping, 
	units_sold, 
	pageviews, 
	timeonsite, 
	revenue,
	unit_price
FROM
    analytics;

------------------------------------------
--query 3 + 4
------------------------------------------
CREATE TABLE visitortransactions (
	visitorid VARCHAR,
	country	VARCHAR,
	city VARCHAR,
	channelgrouping VARCHAR,
	date DATE,
	timeonsite TIME,
	pageview INT,
	sku VARCHAR,
	quantity INT,
	unitprice float,
	revenue float
	)


INSERT INTO visitortransactions (
	visitorid,
	country,
	city,
	channelgrouping,
	date,
	timeonsite,
	pageview,
	sku,
	quantity,
	unitprice,
	revenue   
	)

SELECT 	
	visitid, 
	CASE
		WHEN country = '(not set)' THEN null
		ELSE country
		END AS country, 
	CASE 
		WHEN city = '(not set)' THEN null
		WHEN city = 'not available in demo dataset' THEN null
		ELSE city
		END AS city,
	channelgrouping, 
	CASE 	
		WHEN date ~ E'^\\d+\\.?\\d*$' THEN 		
			TO_DATE(date, 'YYYYMMDD') 
		ELSE NULL 
		END AS date, 
	CASE 	
		WHEN timeonsite ~ E'^\\d+\\.?\\d*$' THEN		
			CAST(	
				(CAST(timeonsite AS int) / 3600)::INTEGER || ':' || 
				((CAST(timeonsite AS int) % 3600) / 60)::INTEGER || ':' || 
				(CAST(timeonsite AS int) % 60)::INTEGER
			AS interval)
		ELSE NULL 
		END AS timeonsite,
	CASE 	
		WHEN pageviews ~ E'^\\d+\\.?\\d*$' THEN 		
			CAST(pageviews AS int) 
		ELSE NULL 
		END AS pageview, 
	productsku,
	CASE
		WHEN totaltransactionrevenue ~ E'^\\d+\\.?\\d*$' THEN
			CAST(ROUND(totaltransactionrevenue :: float) / (productprice :: float) AS INT) 
		WHEN productquantity ~ E'^\\d+\\.?\\d*$' THEN CAST(productquantity AS INT)
		ELSE NULL
		END AS quantity,
	CASE
		WHEN productprice ~ E'^\\d+\\.?\\d*$' THEN	
			CAST(productprice AS float)/1000000
		ELSE NULL
		END AS unitprice,
	CASE 
		WHEN totaltransactionrevenue ~ E'^\\d+\\.?\\d*$' THEN 
			CAST(totaltransactionrevenue AS float)/1000000
		WHEN productquantity ~ E'^\\d+\\.?\\d*$' THEN 
			CAST((productquantity :: int) * (productprice :: int/1000000) AS float)
		ELSE NULL
		END AS revenue
FROM all_sessions

	UNION

SELECT 	
	a2.visitid,
	CASE
		WHEN al.country = '(not set)' THEN null
		ELSE al.country
		END AS country, 
	CASE 
		WHEN al.city = '(not set)' THEN null
		WHEN al.city = 'not available in demo dataset' THEN null
		ELSE city
		END AS city,
	a2.channelgrouping,
	CASE 	
		WHEN a2.date ~ E'^\\d+\\.?\\d*$' THEN 		
			TO_DATE(a2.date, 'YYYYMMDD') 
		ELSE NULL 
		END AS date,
	CASE 	
		WHEN a2.timeonsite ~ E'^\\d+\\.?\\d*$' THEN		
			CAST(	
				(CAST(a2.timeonsite AS int) / 3600)::INTEGER || ':' || 
				((CAST(a2.timeonsite AS int) % 3600) / 60)::INTEGER || ':' || 
				(CAST(a2.timeonsite AS int) % 60)::INTEGER
			AS interval)
		ELSE NULL 
		END AS timeonsite,
	CASE 	
		WHEN a2.pageview ~ E'^\\d+\\.?\\d*$' THEN 		
			CAST(a2.pageview AS int) 
		ELSE NULL 
		END AS pageview,
	NULL,
	CASE 
		WHEN a2.units_sold ~ E'^\\d+\\.?\\d*$' THEN 
			CAST(a2.units_sold as INT)
		ELSE NULL
		END AS quantity,
	CASE 
		WHEN a2.unit_price ~ E'^\\d+\\.?\\d*$' THEN 
			CAST(a2.unit_price as float)/1000000
		ELSE NULL
		END AS unitprice,
	CASE 	
		WHEN a2.revenue ~ E'^\\d+\\.?\\d*$' THEN 		
			CAST(a2.revenue AS float)/100000
		WHEN a2.units_sold ~ E'^\\d+\\.?\\d*$' THEN
			CAST((units_sold :: int) * (unit_price :: float)/1000000 AS float)
		ELSE NULL 
		END AS revenue
FROM 
	analytics2 a2
JOIN 
	all_sessions al ON a2.visitid = al.visitid;
 
 ------------------------------------------
--query 5 + 6
------------------------------------------

------------------------------------------
--query 7 + 8
------------------------------------------
CREATE TABLE productdetails(
	productsku VARCHAR,
	productname VARCHAR,
	category VARCHAR,
	stocklevel INT,
	restockingleadtime INT,
	sentimentscore decimal,
	sentimentmagnitude decimal
	)

INSERT INTO productdetails(
	productsku,
	productname,
	category,
	stocklevel,
	restockingleadtime,
	sentimentscore,
	sentimentmagnitude
	)
SELECT 	DISTINCT CASE 
			WHEN productsku is not null THEN productsku
			ELSE sku
			END AS productsku,
		CASE WHEN name is not null THEN name
			ELSE v2productname
			END AS productname,
		SPLIT_PART (v2productcategory, '/', 
			CASE
        		WHEN v2productcategory = '(not set)' THEN NULL
				WHEN v2productcategory ~ E'/$'
            	THEN CARDINALITY(string_to_array(v2productcategory, '/')) - 1
        		ELSE CARDINALITY(string_to_array(v2productcategory, '/'))
    		END) AS category,
		p.stocklevel :: int,
		p.restockingleadtime :: int,
		sentimentscore :: decimal,
		sentimentmagnitude :: decimal
FROM all_sessions a
LEFT JOIN products p ON a.productsku = p.sku 
