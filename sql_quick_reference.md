---
layout: "page"
title: Example based SQL Reference
show-avatar: false
---

<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    extensions: ["tex2jax.js"],
    jax: ["input/TeX", "output/HTML-CSS"],
    tex2jax: {
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      displayMath: [ ['$$','$$'], ["\\[","\\]"] ],
      processEscapes: true
    },
    "HTML-CSS": { availableFonts: ["TeX"] }
  });
</script>
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>


# Table of contents
1. [Intro](#intro)
2. [Basic Table Manipulations](#paragraph1)
    1. [Deleting Data](#subparagraph1)
    2. [Updating Data](#subparagraph2)
    3. [Table Declaration & Inserting Data](#subparagraph3)
3. [Recursive Queries](#paragraph2)
4. [Grouping Sets, Rollup and Cube Aggregations](#paragraph3)
5. [Window Functions](#paragraph4)

### Intro  <a name="intro"></a>
In this project, I plan to add examples of SQL queries that cover different topics, for instance, recursive queries, the window function

This is a living project, i.e. I will from time to time add new useful examples.

#### Acknowledgment
The examples and SQL queries are based on or taken from other blog-posts or my university courses. Therefore, I will add to each section the corresponding reference.

### Table manipulation <a name="paragraph1"></a>
Here we will look at how to delete, update data from a table, and how to declare a table and insert data into the table.
All examples for deleting data, updating data, and table declaration and insertion were overtaken and adjusted from **Gabriel Cánepa**'s post in [pluralsight](https://www.pluralsight.com/guides/manipulating-data-using-insert-update-delete-sql-server).

#### Deleting Data <a name="subparagraph1"></a>
_Syntax_:
```sql
Delete From table_name
Where field = z;
```
*Example*:
```sql
Delete From customer
Where customer_id = 10;
```

#### Updating Data <a name="subparagraph2"></a>
_Syntax_:
```sql
Update table_name
Set field1 = x, field2 = y
Where field1 = z;
```

*Example*:
```sql
Update customers
Set customer_name = "Thomas and Frank Gluecklich", customer_address = "Strasse Neben an 34"
Where customer_name = "Thomas Gluecklich"
```


#### Table Declaration & Inserting Data <a name="subparagraph3"></a>
*Table declaration:*
```sql
Create Table customers (
	customer_id INT AUTO_INCREMENT,
	customer_name VARCHAR(50) NOT NULL,
	customer_address VARCHAR(100) NOT NULL,
	PRIMARY KEY (customer_id)
);
```

*Data insertion:*
```sql
Insert Into table_name (field2, field1, ...)
Values (value2, value1, ...), (value3, value3, ...);
```
*Example*
```sql
INSERT INTO customers (customer_name, customer_address)
Values 
	("Hans Klobasa", "Stredni ulice"),
	("Thomas Schmidt", "Velka ulice");
```

### Recursive Queries <a name="paragraph2"></a>
Suppose we have a table that was declared using the following query
```sql
Create Table edges (
	vertex_from INT,
	vertex_to INT);
```
This table models the edges of the following graph:
<p align="center">
<img src="/img/sql_queries_recursive.jpg" alt="geo" width="300" height="150"/>
</p>
The table contains for instance the entry: `(1,2)` because there is an edge coming from the vertex $1$ and going to the vertex $2$.

Now, the goal is to figure out which vertices we can reach if we start at the vertex $1$. 

To solve this problem we use the recursive queries.

Firstly, we declare a `view` that will contain all possible paths. Then, we check out all paths starting at vertex $1$.
```sql
Create View paths(S, T) (
	(select vertex_from as S, vertex_to as T
	from edges) 
	Union all
	(select edges.vertex_from, paths.T
	from edges, paths
	where edges.vertex_to = paths.S));
```
Finally, using the table `paths` we can list all vertices that are reachable from the vertex $1$.
```sql
select S,T
from paths
where S = 1
```

This example is based the course ["Einsatz und Realisierung von Datenbanksystemen"](https://db.in.tum.de/teaching/ss20/impldb/?lang=de).

### Grouping Sets, Rollup and Cube Aggregations <a name="paragraph3"></a>
Suppose a table that was created using the following declaration:

```sql
Create Table smartphone_sales (
	Brand Varchar(30),
	Shop Varchar(30),
	Year Int
);
```
If a smartphone was sold, then we insert into the table `customers` the smartphone's brand, in which shop it was purchased, and the year of the purchase. Of course, it would be more useful to enter instead of a year the exact date, but to understand the **aggregation operators** it is simpler to consider only the year.

Now, we would like to discover:
1. total amount of purchases per brand and year
2. total amount of purchases per brand
3. total amount of all purchases

To obtain these results we can use three regarding output equivalent SQL queries.

We could use the `group by` and `union all` to obtain all three requests:
```sql
-- first request:
(select Brand, Year, count(*) as Sales
from smartphone_sales
group by Brand, Year)
union all 
-- second request:
(select Brand, Null, count(*) as Sales
from smartphone_sales
group by Brand)
union all 
-- third request:
(select Null, Null, count(*) as Sales
from smartphone_sales);
```
However, it can be quite tedious to write every time all possible aggregations instead we can use `grouping sets`:

```sql
select Brand, Year, count(*) as Sales
from smartphone_sales
group by grouping sets ((Brand, Year), (Brand), ());
```
So, the `grouping sets` causes that SQL iterates the `group by` over $((Brand, Year),~(Brand),~())$. 

The tuple $((Brand, Year),~(Brand),~())$ obtains a sequence that goes from very specific overview $(Brand,~Year)$ to the global overview $()$, i.e. $()$ stands for the overall amount of purchases. This type of transition from specific to global is called **roll-up**, and the opposite is called **drill-down**.

The `grouping sets` is quite useful; however, it can be still tedious in case of a very specific overview with the goal to perform roll-up. Imagine determining all combinations for `grouping sets` to perform roll-up for $(Brand,~Shop,~Year)$. For this reason, SQL introduces the operation `rollup`:
```sql
select Brand, Year, count(*) as Sales
from smartphone_sales
group by rollup (Brand, Year);
```

Great! We now know how to solve the above requests. But, we are still curious and would like to discover:
1. total amount of purchases per brand, shop
2. total amount of purchases per brand
2. total amount of purchases per shop

To solve these requests we can use `cube` operation:
```sql
select Brand, Shop, count(*) as Sales
from smartphone_sales
group by cube (Brand, Year);
```

### Window Functions <a name="paragraph4"></a>
The following operations can be easily performed using window functions:
- computing moving averages
- computing cumulative sums
- finding top k

To understand the window functions, suppose that we add the following with clause to each query 
```sql
with smartphone_sales(Shop, Purchase_date, Purchase_price) as (
values ('A', date '2019-10-20',323.8),
('B', date '2018-06-24',500.20),
('A', date '2019-09-01',600.99),
('C', date '2019-01-05',250.90),
('C', date '2018-07-18',400.90),
('B', date '2019-03-27',360.90),
 ('B', date '2019-07-20',760.90),
('B',date '2018-03-05',189.99),
('A',date '2018-06-09',198))
```

Now, we would like to know the cumulative amount of sold smartphones of each shop in the year 2019. To do this execute such a query:
```sql
select shop, purchase_date, count(Purchase_Price) over w as Cumulative_Amount_per_Shop -- over starts the window function
from smartphone_sales
where extract(year from purchase_date) = 2019
window w as (partition by shop -- partition clause
             order by purchase_date asc -- ordering clause
			 rows between unbounded preceding -- framing clause
			and current row);
```
The following picture nicely explains each clause in the syntax. The picture
was taken from the course [**Foundations in Data Engineering**](https://db.in.tum.de/teaching/ws1819/foundationsde/?lang=de) from the slides [**Advanced SQL**](https://db.in.tum.de/teaching/ws1819/foundationsde/chapter3_updated.pdf?lang=de).

<p align="center">
<img src="/img/window_clauses.jpg" alt="geo" width="300" height="180"/>
</p>

To compute the running average price of sold smartphones that include the sold smartphones itself and a smartphone sold right after and before:
```sql
select shop, purchase_date, purchase_price, sum(purchase_price) over w as running_average -- over starts the window function
from smartphone_sales
window w as (order by purchase_date asc
			 rows between 1 preceding
			and 1 following);
```

To obtain a shop that had best revenues in 2019 we could use the rank() function:
```sql
select shop, revenue, rank() over w 
from (select shop, sum(purchase_price) as revenue
      from smartphone_sales
      where extract(year from purchase_date) = 2019
      group by shop)
window w as (order by revenue desc);
```
Suppose shops A and B had in year the 2019 both the same highest revenue. So, the function `rank()` gives them both the same rank 1. If shop C had the second-largest revenue,
then it will obtain rank 3 and not rank 2! If we instead want to obtain rank 2 we could use `dense_rank()`. 

Now, add there still many other funcatinalities that provides the window function, but the goal of this project is to refresh basic functionalities of each sql topic

change to sql_quick_refresh, does in english refrence make sense?

Todos:
- based on the images explain the window clauses
- based on movig average once price and once amout explain the range and rows
- explain the lag by computing the performance, and the lead
- based on the amount of sold phones explain the rank(), dens_rank()

perhaps introduce the Hyper DB

