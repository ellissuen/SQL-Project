# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
The objective of this project was to take the raw dataset "ecommerce", understand, clean and transform the data - making sure to address quality assurance issues within pgAdmin, and to use the resulting cleaned data in understanding and gaining insight into various questions surrounding this particular ecommerce business. The self made questions in particular, I wanted to focus on the interaction between visitors to the site and what drove them to make purchases. 

## Process
### (Process overview, code and specific relational below)
1. Load raw data from csv files into pgAdmin
2. Understand data columns, data types and how data relates to each other in the raw data (also understanding the questions that are required and which datas will be needed)
3. Recreate new tables that are more useful for analytics
4. Load, join and clean data into the new tables (while excluding not useful data)
5. Examine the new database and check for QA
6. Query the data and draw conclusions

## Results
As limited as the cleaning and organization process of this project was, there were a few key takeaways that can be examined by the data. In general, there was a good sense of the location of users as they browsed and purchased from the website. By the end of the cleaning of relation 'visitortransactions' the table was mostly filled with values that were useful. The product list was also curated to be quite useful at the end of this process. 

When put together, the two cleaned relations 'visitortransactions' and 'productdetails' could be used to gather some important business insights into the success of this ecommerce business. When assessed together, it shows some specific strengths of this company including a huge interest from US markets, success in particular products that could potentially be extrapolated to other products and highlights some area of weaknesses that this company can grow in. 

Specifically, it showed that this ecommerce website signifcantly hosts US based transactions while spread over many other countries as well. Between cities in California and New York, this site generate most of its revenue from these two US States. It was also able to evaluate specific products and categories of products that were the top-sellers among its visitors. The time that visitors spent on the site and the number of pages view played a role in visitors actually making a transaction while the sentiment towards these products had less value in predicting a sale. 

If moving forward with this company, it would make sense to double down on efforts in the US market while promoting apparel and tech/electronic products. This more focused approach may help the ecommerce site in terms of generating more revenue. Another strategy may be to analyze the website for enagement as many visitors seem to leave before viewing the "critical mass" number of pages - since generally more sales are recorded for those users who visit 4 or more pages. Analyze the products and what causes sentiment score may also be worth looking into. Being able to focus efforts into these areas will ultimately strengthen the advantages that this ecommerce company already maintains while decreasing the disadvantages that are currently working against the company.


## Challenges 
The list of challenges for this project were quite extensive. Starting with the initial data, there was not much to work off of since many data points seemed to be duplicates, incorrect and very disorganized. Early on in the project, a majot challenge was just being able to plan out what steps to take. As the project continued however, here are a list of some specific issues that I ran into and how I tried to justify / solve them:

1.visitid / fullvisitorid not coinciding at times and not being a PK. I dismissed the fullvisitorid as it seemed more redundant that the visitid. Without a PK however, I enventually made a new column 'websitevisitid' to designate every new instance of an user visiting the website as the pseudo-PK. I was unable to still assign the PK though values were distinct. This may be a future consideration to look into but did not affect the project (especially the analysis questions) significantly.

2.sku PK not working. Similarily, I was unable to return only distinct values of the sku since there were so many redundancies and strange category types. These category types contained a multitude of syntax difference that were very difficult to group up. Though usable to queries to get a general sense of products and their categories, it is not the more robust system. Without this PK, I was still able to aggregate the values under the sku and it did not affect my analysis. This can also be looked into for future fixes

3. Some challenges came as learning new coding functions of which I was able to learn from ChatGPT. Some challenges were being unable to CAST values as different data types until it was first recognized as a number by inputing the string ~ E'^\\d+\\.?\\d*$'. This may have been due to a importing error on my end when I first transferred the original relations into pgAdmin. Other functions such as CARDINALITY, SPLIT_PART, string_to_array, and COALESCE were taught to me by ChatGPT.

4. In general, this was definitely a "throw you in the deep end" type of project and not truly understanding what to expect or what is expected made it that much more difficult but also helped me learn that much more.

## Future Goals
The first thing to address would be to create proper PK and FK for all relations made. This will undoubtly take a lot of time to sort though the data and make sure that no redundancies are recorded. However, the trade off would be that the analysis of data would be much more accurate.

Along with this goal, a more intricate network of new relations would be helpful in breaking down the data into smaller, more managable tables. This would serve to streamline the analysis front in future projects. When brainstorming for this project, that was an original idea that I had but upon investigation of the data, I realized that this was not a attainable goal, both due to my personal understanding and ability in PosgreSQL and with the time allotted for this project. Ideally, a number of relations would be created including one for different users, a log of all sessions, products, product restocking, product categories, a log of orders and refunds, and a log of website traffic.

More intricate statistical test could be run to see the signifance limit of the analysis queries to get a better understanding of the magnitude of each query result if this were for guiding important decisions for an active company.

## References

Openai.com, https://chat.openai.com/. Accessed 28 Nov. 2023.


