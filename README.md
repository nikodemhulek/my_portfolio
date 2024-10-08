# **Nikodem Hulek** 

# 1 Project
## Data Analysis for Industrial Company (SQL, PowerBI)
1.In this project I did a sample data analysis for an industrial company. I had to answer questions and make visualizations based on this company's database. The project was created for learning purposes.
# 2 Project
## Crypto copy trading system (SQL)
2.In this project I made an analysis of purchases and sales of cryptocurrency wallets (on the solana network). The database was created by my programmer friend based on the real public database of the solana network. My task was to find suitable wallets that we could use for copy-trading. To do this, I had to create a query , which would filter the data appropriately, so as to reject portfolios used by bots (they make up the majority of the market), portfolios of inactive users (we only wanted active users from that month), portfolios with no profit, portfolios with no risky plays (for example, for most of their balance), etc. My main goal was to filter out portfolios of "day traders" recording recurring profits, operating on short trades (couple hours, not long trades for couple days).

After filtering the portfolios accordingly, I created a new sub-query to filter the portfolios in even more detail, while this time I only used the top1000 results of the previous filtering. 
That's how I was able to find 68 wallets of interest for manual review from hundreds of thousands of data. After checking these portfolios manually, I was able to select the traders I was interested in so that I could copy their strategies. 
With such a project, it is very important to have constant access to current, as fresh as possible data, as traders often change their wallets.
