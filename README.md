# **Nikodem Hulek** 

# 1 Project
## Data Analysis for Industrial Company (SQL, PowerBI)
1.In this project I did a sample data analysis for an industrial company. I had to answer questions and make visualizations based on this company's database. The project was created for learning purposes.
General questions
1. Has the amount of discounts given translated into increased revenue/products sold?
2. Does the region have an impact on revenue?
3. Is seasonality in revenue if present the same for each region?
4. Will a reduction in staff not affect the number of customers/orders served and ultimately sales?
5. Does the gender of the employee matter in terms of number/value of orders?

Conduct a detailed analysis of the Northwind database based on the following points:
1. Give the number of customers in the database. 
- check what the trend of the number of customers has been over the years
- give a list of customers who shopped in 1996 and 1997, but did not make a single order in 1998
- show the TOP 5 customers by revenue for each year and the total discount for the same customers also in the same period
- check whether the TOP5 customers also generated the highest number of orders
- check if there was seasonality in sales
2. Analyze categories and products carefully
- show the products with the highest sales (TOP 10)
- show what % share each product had in the portfolios of TOP 5 customers												
- check whether there was seasonality in product sales -- if so, which products and when sales were highest and when lowest
- show statistics of discounts on individual products
- check whether the products with the highest income were also the most sold products
- check the distribution of product sales by country
3. Analyze employee statistics in detail
- show the size of the customer portfolio for each employee (revenue generated, number of orders, quantity sold)
- check the share of products in each employee's sales. Check whether employees have engaged more strongly based on their performance in selling specific products		
- check which carriers each employee worked with most often
- come up with and calculate two KPIs that will give a good picture of employee performance
- outline which manager has the best team in terms of revenue and volume of specific products sold
4. Analyze the carriers in detail
- which carrier handled the most orders
- which carrier transported the highest number of products
- total freight per carrier
5. Analyze the suppliers
- show the suppliers who delivered the largest number of products to us
- check the distribution of suppliers by region (how many in each region)
- check which region the suppliers who provide the most expensive and cheapest product come from

# 2 Project
## Crypto copy trading system (SQL)
2.In this project I made an analysis of purchases and sales of cryptocurrency wallets (on the solana network). The database was created by my programmer friend based on the real public database of the solana network. My task was to find suitable wallets that we could use for copy-trading. To do this, I had to create a query , which would filter the data appropriately, so as to reject portfolios used by bots (they make up the majority of the market), portfolios of inactive users (we only wanted active users from that month), portfolios with no profit, portfolios with no risky plays (for example, for most of their balance), etc. My main goal was to filter out portfolios of "day traders" recording recurring profits, operating on short trades (couple hours, not long trades for couple days).

After filtering the portfolios accordingly, I created a new sub-query to filter the portfolios in even more detail, while this time I only used the top1000 results of the previous filtering. 
That's how I was able to find 68 wallets of interest for manual review from hundreds of thousands of data. After checking these portfolios manually, I was able to select the traders I was interested in so that I could copy their strategies. 
With such a project, it is very important to have constant access to current, as fresh as possible data, as traders often change their wallets.
