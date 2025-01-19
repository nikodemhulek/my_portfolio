# **Nikodem Hulek** 

# 1 Project
## Data Analysis for Industrial Company (SQL, PowerBI)
1.In this project I did a sample data analysis for an industrial company. I had to answer questions and make visualizations based on this company's database. The project was created for learning purposes.
General questions:
1. Has the amount of discounts given translated into increased revenue/products sold?
```sql
WITH SalesWithDiscount AS (
    SELECT 
    	SUM(od.quantity) AS total_quantity_sold,
        SUM(od.unit_price * od.quantity * (1 - od.discount)) AS total_revenue_with_discount,
        AVG(od.discount) AS avg_discount,
        COUNT(o.order_id) AS total_orders_with_discount
    FROM order_details od
    LEFT JOIN orders o ON od.order_id = o.order_id
    WHERE od.discount > 0
),
SalesWithoutDiscount AS (
    SELECT 
        SUM(od.quantity) AS total_quantity_sold,
        SUM(od.unit_price * od.quantity) AS total_revenue_without_discount,
        COUNT(o.order_id) AS total_orders_without_discount
    FROM order_details od
    LEFT JOIN orders o ON od.order_id = o.order_id
    WHERE od.discount = 0
)
SELECT 
    swd.total_quantity_sold AS quantity_with_discount,
    swod.total_quantity_sold AS quantity_without_discount,
    swd.total_revenue_with_discount AS revenue_with_discount,
    swod.total_revenue_without_discount AS revenue_without_discount,
    swd.total_orders_with_discount AS orders_with_discount,
    swod.total_orders_without_discount AS orders_without_discount,
    swd.avg_discount AS average_discount
FROM SalesWithDiscount swd, SalesWithoutDiscount swod
```

2. Does the region have an impact on revenue?
```sql
SELECT
    r.region_description AS region,
    COUNT(o.order_id) AS total_orders,
    SUM(od.unit_price * od.quantity) AS total_revenue
from orders o
left join customers c ON o.customer_id = c.customer_id
left join order_details od ON o.order_id = od.order_id
left join employees e on o.employee_id = e.employee_id 
left join employee_territories et on e.employee_id = et.employee_id 
left join territories t on et.territory_id = t.territory_id 
left join region r ON t.region_id = r.region_id
GROUP by r.region_description
ORDER by total_revenue desc
```

3. Is seasonality in revenue if present the same for each region?
```sql
SELECT
	r.region_description AS region,
    date_part('month', o.order_date) AS month,
    COUNT(o.order_id) AS total_orders,
    SUM(od.quantity) AS total_quantity_sold,
    round(SUM(od.unit_price * od.quantity)) AS total_revenue
FROM orders o
left join order_details od ON o.order_id = od.order_id
left JOIN employees e ON o.employee_id = e.employee_id
left join employee_territories et on e.employee_id = et.employee_id 
left JOIN territories t ON et.territory_id = t.territory_id
left JOIN region r ON t.region_id = r.region_id
GROUP BY r.region_description, date_part('month', o.order_date)
ORDER BY r.region_description, month
```

4. Will a reduction in staff not affect the number of customers/orders served and ultimately sales?
```sql
with wskazniki as (
    select 
        e.employee_id,
        COUNT(o.order_id) as liczba_zamowien,
        SUM(od.quantity * od.unit_price) as wartosc_sprzedazy
    from employees e
 	left join orders o on e.employee_id = o.employee_id
    left join order_details od on o.order_id = od.order_id
    group by e.employee_id
),
srednie_wartosci AS (
    select 
        AVG(liczba_zamowien) as srednia_zamowien,
        AVG(wartosc_sprzedazy) as srednia_sprzedazy
    from wskazniki
)
select 
    e.employee_id,
    CONCAT(e.first_name, ' ', e.last_name) as nazwa_pracownika,
    w.liczba_zamowien,
    round((select srednia_zamowien from srednie_wartosci)) as srednia_zamowien,
    w.wartosc_sprzedazy,
    round((select srednia_sprzedazy from srednie_wartosci)) as srednia_sprzedazy,
    round(w.wartosc_sprzedazy / w.liczba_zamowien) as wartos_sprzedazy_na_zamowienie,
	round((select srednia_sprzedazy from srednie_wartosci) / (select srednia_zamowien from srednie_wartosci)) as srednia_wartos_sprzedazy_na_zamowienie
from  employees e
left join wskazniki w on e.employee_id = w.employee_id
order by wartos_sprzedazy_na_zamowienie desc
```

1.Conduct a detailed analysis of the Northwind database based on the following points:
- check what the trend of the number of customers has been over the years
```sql
select 
	count(distinct c.customer_id) as custmers_number,
	date_part('year', o.order_date) as "year"
from customers c 
left join orders o on c.customer_id = o.customer_id
where date_part('year', o.order_date) is not null
group by date_part('year', o.order_date)
order by year
```
- give a list of customers who shopped in 1996 and 1997, but did not make a single order in 1998
```sql
with tbl1998 as (
select
	distinct o.customer_id as customers1998
from orders o 
where date_part('year', o.order_date) = 1998 
group by o.customer_id 
)
select distinct o.customer_id
from orders o 
where date_part('year', o.order_date) in(1996, 1997)
group by o.customer_id 
having  count(distinct date_part('year', o.order_date)) = 2 --klienci ktorzy robili zakupy w tych 2 latach (2 warunki)
and o.customer_id not in (select customers1998 from tbl1998)
```
- show the TOP 5 customers by revenue for each year and the total discount for the same customers also in the same period
```sql
with ranked_customers as (
	select 
		c.customer_id,
		c.company_name,
		date_part('year', o.order_date) as "year", 
		round(sum(od.quantity*od.unit_price)) as income,
		od.discount as total_discount,
		row_number() over (partition by date_part('year', o.order_date),
		order by sum(od.quantity*od.unit_price*(1 - od.discount)) desc) as rank,
		sum(od.quantity*od.unit_price*od.discount) as discount_amount
	from customers c 
	left join orders o on c.customer_id = o.customer_id
	left join order_details od on o.order_id = od.order_id 
	group by c.customer_id, c.company_name, date_part('year', o.order_date), od.discount
	having sum(od.quantity*od.unit_price*(1 - od.discount)) is not null
)
select
	rc.customer_id,
	rc.company_name,
	rc.year,
	rc.income,
	round(rc.discount_amount),
	rc.total_discount
from ranked_customers rc
where rc.rank <= 5
order by rc.year, rc.income desc
```
- check whether the TOP5 customers also generated the highest number of orders
```sql
TOP 5 INCOME 
with ranked_customers_by_income as (
	select 
		c.customer_id,
		c.company_name,
		date_part('year', o.order_date) as "year", 
		round(sum(od.quantity*od.unit_price)) as income,
		round(sum(od.discount)) as total_discount,
		row_number() over (partition by date_part('year', o.order_date) order by sum(od.quantity*od.unit_price*(1 - od.discount)) desc) as rank
	from customers c 
	left join orders o on c.customer_id = o.customer_id
	left join order_details od on o.order_id = od.order_id 
	group by c.customer_id, c.company_name, date_part('year', o.order_date)
	having sum(od.quantity*od.unit_price*(1 - od.discount)) is not null
)
select rc.customer_id, rc.company_name, rc.year, rc.income, rc.total_discount
from ranked_customers_by_income rc
where rc.rank <= 5
order by rc.year, rc.income desc

TOP 5 ORDER QUANTITY
with ranked_customers_by_orders as (
	select 
		c.customer_id,
		c.company_name,
		date_part('year', o.order_date) as "year", 
		count(o.order_id) as order_count,
		round(sum(od.discount)) as total_discount,
		row_number() over (partition by date_part('year', o.order_date) order by count(o.order_id) desc) as rank
	from customers c 
	left join orders o on c.customer_id = o.customer_id
	left join order_details od on o.order_id = od.order_id 
	group by c.customer_id, c.company_name, date_part('year', o.order_date)
	having count(o.order_id) > 0 -- upewniam sie ze sa jakiekoliwek zamowienia
)
select ro.customer_id,
	ro.company_name,
	ro.year,
	ro.order_count,
	ro.total_discount
from ranked_customers_by_orders ro
where ro.rank <= 5
order by ro.year, ro.order_count desc
```
Answear: 

2. Analyze categories and products carefully
- show the products with the highest sales (TOP 10)
```sql
select 
	p.product_name, 
	sum(od.quantity) as total_sales
from products p
left join order_details od on p.product_id = od.product_id 
group by p.product_name 
order by total_sales desc
limit 10
```
- show what % share each product had in the portfolios of TOP 5 customers
```sql
with top5_customers as (
	select 
		c.customer_id, 
		sum(od.quantity) total_sales
	from customers c 
	left join orders o on c.customer_id = o.customer_id
	left join order_details od on o.order_id = od.order_id
	where od.quantity is not null
	group by c.customer_id
	order by total_sales desc
	limit 5
),
customer_product_share as (
	select
		p.product_name,
		o.customer_id,
		sum(od.quantity) as product_sales, 
		sum(sum(od.quantity)) over(partition by o.customer_id) as customer_total_sales
	from products p 
	left join order_details od on p.product_id = od.product_id
	left join orders o on od.order_id = o.order_id
	group by p.product_name, o.customer_id
)
select 
	cps.product_name,
	cps.customer_id,
	(round(cps.product_sales * 100 / cps.customer_total_sales)) as product_share_percentage
from customer_product_share cps
order by cps.customer_id, product_share_percentage desc
```       				
- check whether there was seasonality in product sales -- if so, which products and when sales were highest and when lowest
```sql
select
	p.product_name,
	date_part('month', o.order_date) as sale_month,
	sum(od.quantity) as total_quantity 
from order_details od 
left join orders o on od.order_id = o.order_id 
left join products p on od.product_id = p.product_id 
group by p.product_name, sale_month
order by 
	p.product_name,
	sale_month
```
Answear:

- show statistics of discounts on individual products
```sql
select 
	p.product_name,
	count(od.discount) as total_discount,
	avg(od.discount) as avg_discount,
	min(od.discount) as min_dscount,
	max(od.discount) as max_discount
from products p 
left join order_details od on p.product_id = od.product_id 
where od.discount > 0 -- tylko produkty na ktorych sotsowano rabat
group by p.product_name 
order by avg_discount desc
```
- check whether the products with the highest income were also the most sold products
```sql
with revenue as( 
	select 
		p.product_name,
		round(sum(od.quantity*od.unit_price)) as total_revenue
	from order_details od 
	left join products p on od.product_id = p.product_id 
	group by p.product_name 
	order by total_revenue desc
	limit 10
), --najczesciej sprzedawane produkty
quantity as (
	select 
		p.product_name,
	sum(od.quantity) as total_quantity
	from order_details od 
	left join products p on od.product_id = p.product_id 
	group by p.product_name 
	order by total_quantity desc
	limit 10
)
select 
	r.product_name as revenue_product,
	r.total_revenue,
	q.product_name as quantity_produt,
	q.total_quantity
from revenue r
left join quantity q on r.product_name = q.product_name
```
Answear:

- check the distribution of product sales by country
```sql
select 
	o.ship_country,
	p.product_name,
	sum(od.quantity) as total_quantity 
from order_details od
left join orders o on od.order_id = o.order_id 
left join products p on od.product_id = p.product_id 
group by o.ship_country, p.product_name 
order by o.ship_country, total_quantity desc
```
3. Analyze employee statistics in detail
- show the size of the customer portfolio for each employee (revenue generated, number of orders, quantity sold)
```sql
select
	e.employee_id,
	concat(e.first_name,' ', e.last_name),
	count(o.order_id) as total_orders,
	sum(od.quantity) as total_quantity_sold,
	round(sum(od.unit_price*od.quantity)) as total_revenue_generated
from employees e 
left join orders o on e.employee_id = o.employee_id 
left join order_details od on o.order_id = od.order_id 
group by e.employee_id, concat(e.first_name,' ', e.last_name)
order by total_revenue_generated desc
```
- check the share of products in each employee's sales. Check whether employees have engaged more strongly based on their performance in selling specific products
```sql
with employee_sales as (
	select 
		e.employee_id,
		concat(e.first_name,' ', e.last_name) as name,
		p.product_name,
		sum(od.quantity) as total_quantity_sold,
		sum(od.unit_price*od.quantity) as total_revenue_genereted
	from employees e 
	left join orders o on e.employee_id = o.employee_id
	left join order_details od on o.order_id = od.order_id
	left join products p on od.product_id = p.product_id
	group by
		e.employee_id,
		concat(e.first_name,' ', e.last_name),
		p.product_name
```  
- check which carriers each employee worked with most often
```sql
select 
	employee_id,
	name,
	product_name,
	total_quantity_sold,
	round(total_revenue_genereted),
	round((total_quantity_sold / sum(total_quantity_sold) over (partition by employee_id)) * 100, 2) as product_sales_share_percentage
from employee_sales
order by employee_id, product_sales_share_percentage desc
```
- come up with and calculate two KPIs that will give a good picture of employee performance
```sql
select
	e.employee_id,
	concat(e.first_name,' ', e.last_name) as employee_name,
	count(o.order_id) as total_orders,
	sum(od.quantity) as total_quantity_sold,
	round(sum(od.unit_price*od.quantity)) as total_revenue_generated,
	round(sum(od.unit_price*od.quantity) / count(o.order_id)) as average_order_value,
	round(sum(od.quantity) / count(o.order_id), 2) as average_items_per_order
from employees e 
LEFT JOIN orders o on e.employee_id = o.employee_id 
LEFT JOIN order_details od on o.order_id = od.order_id 
GROUP BY e.employee_id, employee_name
ORDER BY total_revenue_generated desc
```

4. Analyze the carriers in detail
- which carrier handled the most orders
```sql
SELECT 
    s.shipper_id, 
    s.company_name, 
    COUNT(o.order_id) AS total_orders
FROM 
    shippers s
LEFT JOIN 
    orders o ON s.shipper_id = o.ship_via
GROUP BY 
    s.shipper_id, s.company_name
ORDER BY 
    total_orders desc
```
- which carrier transported the highest number of products
```sql
SELECT 
	s.shipper_id, 
    s.company_name, 
    SUM(od.quantity) AS total_products_shipped
FROM 
    shippers s
LEFT JOIN 
    orders o ON s.shipper_id = o.ship_via
LEFT JOIN 
    order_details od ON o.order_id = od.order_id
GROUP BY 
    s.shipper_id, s.company_name
ORDER BY 
    total_products_shipped desc
```
- total freight per carrier
```sql
SELECT 
    s.shipper_id, 
    s.company_name, 
    SUM(o.freight) AS total_freight
FROM 
    shippers s
JOIN 
    orders o ON s.shipper_id = o.ship_via
GROUP BY 
    s.shipper_id, s.company_name
ORDER by
	total_freight desc
```
5. Analyze the suppliers
- show the suppliers who delivered the largest number of products to us
```sql
select 
    s.company_name, 
    SUM(od.quantity) AS total_quantity_delivered
from suppliers s
left join  products p ON s.supplier_id = p.supplier_id
left   join order_details od ON p.product_id = od.product_id
group by s.company_name 
order by total_quantity_delivered DESC
```
- check the distribution of suppliers by region (how many in each region)
```sql
select 
	s.region,
	count(s.supplier_id) as total_suppliers
from suppliers s 
where s.region is not null
group by s.region
```
- check which region the suppliers who provide the most expensive and cheapest product come from
```sql
with  the_cheapest as (
	select 
		s.region,
		s.company_name,
		p.product_name,
		p.unit_price
	from products p 
	left join suppliers s on p.supplier_id = s.supplier_id
	where p.unit_price = (select min(unit_price) from products)
), 
the_most_expensive as (
	select 
		s.region,
		s.company_name,
		p.product_name,
		p.unit_price
	from products p 
	left join suppliers s on p.supplier_id = s.supplier_id
	where  p.unit_price = (select max(unit_price) from products)
)
select *
from the_cheapest
union all
select *
from the_most_expensive
```

# 2 Project
## Crypto copy trading system (SQL)
2.In this project I made an analysis of purchases and sales of cryptocurrency wallets (on the solana network). The database was created by my programmer friend based on the real public database of the solana network. My task was to find suitable wallets that we could use for copy-trading. To do this, I had to create a query , which would filter the data appropriately, so as to reject portfolios used by bots (they make up the majority of the market), portfolios of inactive users (we only wanted active users from that month), portfolios with no profit, portfolios with no risky plays (for example, for most of their balance), etc. My main goal was to filter out portfolios of "day traders" recording recurring profits, operating on short trades (couple hours, not long trades for couple days).

After filtering the portfolios accordingly, I created a new sub-query to filter the portfolios in even more detail, while this time I only used the top1000 results of the previous filtering. 
That's how I was able to find 68 wallets of interest for manual review from hundreds of thousands of data. After checking these portfolios manually, I was able to select the traders I was interested in so that I could copy their strategies. 
With such a project, it is very important to have constant access to current, as fresh as possible data, as traders often change their wallets.
