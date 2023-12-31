# Explanation:
In cleaning the data, many assumptions were needed to be concluded before work could take place. In going through the dataset and understanding the data, one of the first things to be addressed was the original 'analytics' relation. Many of the rows seemed to repeat themselves for each product viewed - leading to redundant data with the only change being different 'unit_price' of the products viewed. This was first addressed to clean excessive data to a more managable size (query 1 + 2) so that only distinct combinations of the data (without the 'unit_price') were kept. Therefore, the relation 'analytics2' was used for any subsequent work instead of the relation 'analytics'

During the understanding data process, I realized that there were no PRIMARY KEYs to the relations 'all_sessions' and 'analytics' (and therefore 'analytics2'). I decided to tackle this later due to the mess of deciding between a conjuncted PRIMARY KEY or some other iteration. However, what did need to be tackled was that information for 'all_sessions' was somewhat similar to that of 'analytics2' and should be combined to form a more full picture of what the purchases across the ecommerce site looked like. Fortunately, two things made this process a bit easier. Earlier on in the understanding data, I determined that 'visitid' was a more useful metric of determining a user than the 'fullvisitorid'. Therefore I assigned 'visitid' as a foreign key between the two relations. It also happened that a large portion of the 'analytics2' visitid were present within the 'all_sessions' visitid - easing the ability to transform the 'analytics2.visitid's to having their own associated country and city. There were a few outliers and this process will be addressed in the QA part of the project. By combining these two relations with all_sessions still being the main relation, I was able to import all data into 1 relation. Along the way, I also decided to change data_types and change prices to a readable format. A large assumption that I had to make was that if there were a 'all_sessions.totaltransactionrevenue' or 'analytics2.revenue', then there should be a 'all_sessions.productquantity' or 'analytics2.units_sold' respectively and visa versa. This assumption may be incorrect due to not knowing when transaction revenue or quantities are recorded into the data. (more on this in QA / risks). With this assumption however, the 'visitortransactions' relation could be created with the 'revenue' and 'quantity' being calculated from each other. Many extra data points from 'analytics2' were dropped (more in QA) but a cleaned 'visitortransactions' chart was created (query 3 + 4). This relation alone helped me answer couple of the project's required and self made questions.

However, the 'visitortransactions' still needed a PK. An id was assigned to the table acting as a counting number for every user - every visit. This running count served as the PK (query 5 + 6).

In the final step of cleaning data, the products needed to be combined from different sources. The PRIMARY KEY was much easier to identify as the sku and so no further steps were needed to address this issue. With some altering of data types, the main challenge was to identify the 'category' and combine as needed. Though not perfect, my query allowed for a general clean of the 'category' column, enough for analysis to take place (query 7 + 8). Unfortunately, I was unable to determine skus from the 'analytics2' table and therefore they were kept as null to prevent skewing the data.

It was at this moment that I ran into a major snag and could not actually figure out the sku PK for my new 'productdetails' table. This was addressed in a very non-conventional way in which this previous table was used to as a in between to make the final 'productdistinct' table. This new table was almost exactly the same but with a verified unique PK in the sku column (query 9 + 10 + 11 + 12). Again addressing this problem in a nontraditional way, I made a scaffolding from the PK to ensure it was correct, then filled in the rest of the required columns. 

      

Queries:
------------------------------------------
--query 1 + 2
------------------------------------------

	-- to greatly reduce the redundancy of data from the original analytics table
 
 	CREATE TABLE analytics2 (
		visitnumber VARCHAR,		-- all varchar formats in order to speed up the importing process
		visitid VARCHAR,		-- data types will be fixed in the future
		visitstarttime VARCHAR,		--primary keys will be assigned in the future
		date VARCHAR,
		channelgrouping VARCHAR,
		units_sold VARCHAR,
		pageview VARCHAR,
		timeonsite VARCHAR,
		revenue VARCHAR,
		unit_price VARCHAR
		)

------------------------------------------

	--insert these data types. From original analytics table, visitnumber, fullvisitorid, userid, socialenegagementtype
 	--and bounces will not be imported. 
 
	INSERT INTO analytics2 (visitnumber, visitid, visitstarttime, date, channelgrouping, units_sold, pageview, 			timeonsite, revenue, unit_price)
	SELECT DISTINCT			--this allows for distinct combinations of rows to be imported into analytics2
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
	FROM analytics;

------------------------------------------
--query 3 + 4
------------------------------------------

	--visitortransactions will be a join of all_sessions and analytics2
 	--since visitid is still not distinct, the PK will be assigned later
 
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

------------------------------------------

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
		CASE									-- country cleaned of NULL responses
			WHEN country = '(not set)' THEN null				--spelling of countries checked for consistency - no irregularities spotted
			ELSE country
			END AS country, 
		CASE 
			WHEN city = '(not set)' THEN null				--city cleaned of NULL responses
			WHEN city = 'not available in demo dataset' THEN null		--spelling of countries were checked for consistency - no irregularities spotted
			ELSE city
			END AS city,
		channelgrouping, 
		CASE 	
			WHEN date ~ E'^\\d+\\.?\\d*$' THEN 				--strange syntax error. ~ E'^\\d+\\.?\\d*$' allows for detection of any number
				TO_DATE(date, 'YYYYMMDD') 				
			ELSE NULL 
			END AS date, 
		CASE 	
			WHEN timeonsite ~ E'^\\d+\\.?\\d*$' THEN			--timeonsite changed to a HH:MM:SS format
				CAST(	
					(CAST(timeonsite AS int) / 3600)::INTEGER || ':' || 
					((CAST(timeonsite AS int) % 3600) / 60)::INTEGER || ':' || 
					(CAST(timeonsite AS int) % 60)::INTEGER
				AS interval)
			ELSE NULL 
			END AS timeonsite,
		CASE 									--pageviews changed to INT
			WHEN pageviews ~ E'^\\d+\\.?\\d*$' THEN 		
				CAST(pageviews AS int) 
			ELSE NULL 
			END AS pageview, 
		productsku,											--all columns dealing with $ values must be divided by 1000000
		CASE												--if there is revenue but no quantity, this calculates a quantity (rounded down)
			WHEN totaltransactionrevenue ~ E'^\\d+\\.?\\d*$' THEN					--this assumes that all transactions should have some product quantity sold. unsure of how data was gathered
				CAST(ROUND(totaltransactionrevenue :: float) / (productprice :: float) AS INT) 	--must be assumed for now
			WHEN productquantity ~ E'^\\d+\\.?\\d*$' THEN CAST(productquantity AS INT)
			ELSE NULL
			END AS quantity,
		CASE									
			WHEN productprice ~ E'^\\d+\\.?\\d*$' THEN			--productprice present no matter if there is transaction or not
				CAST(productprice AS float)/1000000
			ELSE NULL
			END AS unitprice,
		CASE 									--when there is a quantity, revenue is calculated from it
			WHEN totaltransactionrevenue ~ E'^\\d+\\.?\\d*$' THEN 		--again assumption for now without any other information to work from
				CAST(totaltransactionrevenue AS float)/1000000
			WHEN productquantity ~ E'^\\d+\\.?\\d*$' THEN 
				CAST((productquantity :: int) * (productprice :: int/1000000) AS float)
			ELSE NULL
			END AS revenue
	FROM all_sessions
	UNION										--same as above but for analytics
	SELECT 	
		a2.visitid,
		CASE
			WHEN al.country = '(not set)' THEN null
			ELSE al.country
			END AS country, 
		CASE 
			WHEN al.city = '(not set)' THEN null				--city and country must be taken from all_sessions
			WHEN al.city = 'not available in demo dataset' THEN null	--no data available in analytics2
			ELSE city							--analytics2 is only joined on all_sessions when visitid is present in all_sessions ** big assumption ** will need to QA
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
		CASE 								--quantity all present, do not need to calculate like from all_sessions (above)
			WHEN a2.units_sold ~ E'^\\d+\\.?\\d*$' THEN 
				CAST(a2.units_sold as INT)
			ELSE NULL
			END AS quantity,
		CASE 
			WHEN a2.unit_price ~ E'^\\d+\\.?\\d*$' THEN 
				CAST(a2.unit_price as float)/1000000
			ELSE NULL
			END AS unitprice,
		CASE 								--need to calculate revenue if quantity is present but revenue is null
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
	
 	-- new strategy, insert an id for visitortransactions that serves as primary key
 	
  	ALTER TABLE visitortransactions
	ADD COLUMN id SERIAL PRIMARY KEY

------------------------------------------

	--update with PK with a running number to ensure unique entries
 
 	UPDATE visitortransactions
	SET id = nextval(pg_get_serial_sequence('visitortransactions', 'id'));

------------------------------------------
--query 7 + 8
------------------------------------------
	
 	--new table for products join from products table and all_sessions
  	--productsku is PK. will be able to sort through the duplicates
  	
  	CREATE TABLE productdetails(
		productsku VARCHAR,
		productname VARCHAR,
		category VARCHAR,
		stocklevel INT,
		restockingleadtime INT,
		sentimentscore decimal,
		sentimentmagnitude decimal
 		)

------------------------------------------

	INSERT INTO productdetails(
		productsku,
		productname,
		category,
		stocklevel,
		restockingleadtime,
		sentimentscore,
		sentimentmagnitude
		)
	SELECT 							--distinct sku
	DISTINCT CASE 
		WHEN productsku is not null THEN productsku
		ELSE sku
		END AS productsku,
	CASE WHEN name is not null THEN name			--v2name from all_sessions
		ELSE v2productname				--name from products
		END AS productname,
	SPLIT_PART (v2productcategory, '/', 			
		CASE
        	WHEN v2productcategory = '(not set)' THEN NULL			--this allows to retain the last array from categories to be extracted
		WHEN v2productcategory ~ E'/$'					--also gets rid of "/"
            	THEN CARDINALITY(string_to_array(v2productcategory, '/')) - 1
        	ELSE CARDINALITY(string_to_array(v2productcategory, '/'))
    		END) AS category,
	p.stocklevel :: int,
	p.restockingleadtime :: int,
	sentimentscore :: decimal,
	sentimentmagnitude :: decimal
FROM all_sessions a
LEFT JOIN products p ON a.productsku = p.sku 

------------------------------------------
--query 9 + 10 + 11 + 12
------------------------------------------

	--distinct PK did not work from above. after trial and error. productdetails will become an inbetween table for new table with PK

	CREATE TABLE productdistinct(			--new table with PK
		sku VARCHAR
 		PRIMARY KEY (sku)
 	)

	INSERT INTO productdistinct			--ensured distinct PK first
	SELECT 
		DISTINCT productsku
		FROM all_sessions
	UNION
	SELECT 
		DISTINCT sku
		FROM products

	ALTER TABLE distinctsku				--added the rest of the columns to new table
	ADD COLUMN name VARCHAR,
	ADD COLUMN category VARCHAR,
	ADD COLUMN stocklevel INT,
	ADD COLUMN restockingleadtime INT,
	ADD COLUMN sentimentscore decimal,
	ADD COLUMN sentimentmagnitude decimal

	INSERT INTO productdistinct			--added the rest of the data to new table
	SELECT 	DISTINCT productsku, 			--categories will be slightly off since only 1 category will be chosen
		MAX(productname), 				-- there were many same products under different categories. (may be a future issue to address)
		MAX(category), 
		MAX(stocklevel), 
		MAX(restockingleadtime),
		MAX(sentimentscore),
		MAX(sentimentmagnitude)
	FROM productdetails
	GROUP BY productsku
