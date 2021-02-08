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
2. [Table Declaration and Basic Table Manipulations](#paragraph1)
    1. [Table Declaration & Inserting Data](#subparagraph1)
    2. [Deleting Data](#subparagraph2)
    3. [Updating Data](#subparagraph3)
3. [Recursive Queries](#paragraph2)
4. [Grouping Sets, Rollup and Cube Aggregations](#paragraph3)
5. [Window Functions](#paragraph4)

### Intro  <a name="intro"></a>
In this project, I plan to add SQL queries covering different topics, for instance, recursive queries or window functions. The aim of the presented examples is mainly to refresh the topics covered by the examples. So, I will not in detail explain each line of the SQL queries. However, I believe that the examples can be used as a good starting point to understand the covered concept in more depth.

This project is a living project, i.e., I will add new useful examples from time to time.

#### Acknowledgment
The examples are based on or taken from other blog-posts or my university courses. Therefore, I will add to each section the corresponding source.

### Table Declaration and Basic Table Manipulations <a name="paragraph1"></a>
In this section, we start with basic commands like table declaration or update of table's data.

The following examples are based on **Gabriel Cánepa**'s post in [pluralsight](https://www.pluralsight.com/guides/manipulating-data-using-insert-update-delete-sql-server).

#### Table Declaration & Inserting Data <a name="subparagraph1"></a>
*Table declaration:*
```sql
Create Table customers (
	customer_id int Auto_Increment,
	customer_name varchar(50) Not Null,
	customer_address varchar(100) Not Null,
	Primary Key (customer_id)
);
```

We set the customer_id as a *primary key*, i.e., each row in the table customers can be uniquely identified by the field customer_id. The `Auto_Increment` command means that the customer_id is automatically incremented if a new row is inserted.

*Data insertion:*
```sql
Insert Into table_name (field1, field2, ...)
Value (value11, value12, ...), (value21, value22, ...);
```
*Example*
```sql
Insert Into customers (customer_name, customer_address)
Value 
	("Thomas Example", "Example address"),
	("Peter Example1", "Example1 address");
```
The `Auto_Increment` causes that the "Thomas Example" obtains customer_id 1 and "Peter Example" obtains 2.

#### Deleting Data <a name="subparagraph2"></a>
_Syntax_:
```sql
Delete From table_name
Where field = z;
```
*Example*:
```sql
Delete From customer
Where customer_id = 2;
```

#### Updating Data <a name="subparagraph3"></a>
_Syntax_:
```sql
Update table_name
Set field1 = x, field2 = y
Where field3 = z;
```

*Example*:
```sql
Update customers
Set customer_name = "Thomas Frank Example", customer_address = "Great Address"
Where customer_name = "Thomas Example"
```

### Recursive Queries <a name="paragraph2"></a>
Suppose we have a table that was declared using the following query:
```sql
Create Table edges (
	vertex_from int,
	vertex_to int);
```
This table models the edges of the following graph:
<p align="center">
<img src="/img/sql_queries_recursive.jpg" alt="geo" width="300" height="150"/>
</p>
For instance, the table contains the row (1, 2) because there is an edge from the vertex $1$ to the vertex $2$.

The goal is to figure out which vertices we can reach if we start at the vertex $1$. 

To solve this problem, we use recursive queries.

Firstly, we declare a `view` that will contain all possible paths. Then, we check out all paths starting at the vertex $1$.
```sql
Create View paths(S, T) (
	(Select vertex_from As S, vertex_to As T
	From edges) 
	Union All
	(Select edges.vertex_from, paths.T
	From edges, paths
	Where edges.vertex_to = paths.S));
```
Finally, using the table `paths` we can list all vertices that are reachable from the vertex $1$.
```sql
Select S,T
From paths
Where S = 1
```

This example is based on the course ["Einsatz und Realisierung von Datenbanksystemen"](https://db.in.tum.de/teaching/ss20/impldb/?lang=de).

### Grouping Sets, Rollup and Cube Aggregations <a name="paragraph3"></a>
Suppose a table that was created using the following declaration:

```sql
Create Table smartphone_sales (
	product_id int Not Null,
	brand varchar(30),
	store varchar(30),
	year int,
	Primary Key (product_id)
);
```
If a smartphone was sold, then we insert into the table smartphone_sales the smartphone's product-ID, brand, in which store it was purchased, and the year of the purchase. Of course, it would be more useful to enter instead of a year the exact date, but to understand the **aggregation operators** it is simpler to consider only the year.

Now, we would like to discover:
1. total amount of purchases per brand and year
2. total amount of purchases per brand
3. total amount of all purchases

To obtain these results we can use three equivalent approaches:
- a combination of `Group By` and `Union All` operations
- `Grouping Sets` operation
- `Rollup` operation

The following SQL query leverages the `Group By` and `union all` operations:
```sql
-- first request: total amount of purchases per brand and year
(Select Brand, Year, count(*) As Sales
From smartphone_sales
Group By Brand, Year)
Union All
-- second request: total amount of purchases per brand
(Select Brand, Null, count(*) As Sales
From smartphone_sales
Group By Brand)
Union All 
-- third request: total amount of all purchases
(Select Null, Null, count(*) As Sales
From smartphone_sales);
```
The above SQL query is straight forward and easy to understand. However, it can be quite tedious to write every time all possible aggregations, instead we can use `Grouping Sets`:

```sql
Select Brand, Year, count(*) as Sales
From smartphone_sales
Group By Grouping Sets ((Brand, Year), (Brand), ());
```
`Grouping Sets` causes that SQL iterates the `Group By` over: $(Brand, Year),~(Brand),~()$. 

The sequence: $(Brand, Year),~(Brand),~()$ goes from a very specific overview $(Brand,~Year)$ to a global overview $()$, i.e., $()$ stands for the overall amount of purchases. This type of transition from specific to global is called **roll-up**, and the opposite is called **drill-down**.

The `Grouping Sets` is quite useful; however, it can be still tedious in case of a very specific overview with the goal to perform roll-up. Imagine determining all combinations for `Grouping Sets` to perform roll-up for $(Brand,~store,~Year)$. We would have to type: $(Brand,~Store,~Year),~(Brand,~Store),~(Brand),~()$. For this reason, SQL introduces the operation `Rollup`:
```sql
Select Brand, Year, count(*) as Sales
From smartphone_sales
Group By Rollup (Brand, Year);
```
The `Rollup` operation generates for $(Brand, Year)$ the following sequence: $((Brand, Year),~(Brand),~())$.

Note that if we apply `Rollup` to $(Year, Brand)$, then we obtain the following: $(Year, Brand),~(Year),~()$. As we can se the order of field names matters, i.e., the `Rollup` generates for $(Year,~Brand)$ a different sequence than for $(Brand,~Year)$.

Great! We now know how to solve the above requests. In addition to the above requests, we would like to know:
1. total amount of purchases per brand, store
2. total amount of purchases per brand
2. total amount of purchases per store

To solve these requests we can use `cube` operation:
```sql
Select Brand, Store, count(*) As Sales
From smartphone_sales
Group By Cube (Brand, Year);
```

The operation cube applies `count(*)` to all possible combinations of $(Brand, Year)$, i.e., $(Brand,~Year),~(Brand),~(Year),~()$.

Finally, as in the [recursive case](#paragraph2) is the above content based on the course ["Einsatz und Realisierung von Datenbanksystemen"](https://db.in.tum.de/teaching/ss20/impldb/?lang=de).

### Window Functions <a name="paragraph4"></a>
The following operations can be easily performed using window functions:
- computing moving averages
- computing cumulative sums
- finding top k

We create using the with-clause a temporary table smartphone_sales. This table will be used to explain the window functions. With the following query we declare the temporary table:
```sql
-- with-clause; START
With smartphone_sales(Store, Purchase_date, Purchase_price) As (
Values ('A', date '2019-10-20',323.8),
('B', date '2018-06-24',500.20),
('A', date '2019-09-01',600.99),
('C', date '2019-01-05',250.90),
('C', date '2018-07-18',400.90),
('B', date '2019-03-27',360.90),
 ('B', date '2019-07-20',760.90),
('B',date '2018-03-05',189.99),
('A',date '2018-06-09',198))
-- with-clause; END
```
The with-clause must be followed by a `select` query, which is shown demonstratively in the following query:
```sql
-- with-clause; START
With smartphone_sales(Store, Purchase_date, Purchase_price) As (
Values ('A', date '2019-10-20',323.8),
('B', date '2018-06-24',500.20),
('A', date '2019-09-01',600.99),
('C', date '2019-01-05',250.90),
('C', date '2018-07-18',400.90),
('B', date '2019-03-27',360.90),
 ('B', date '2019-07-20',760.90),
('B',date '2018-03-05',189.99),
('A',date '2018-06-09',198))
-- with-clause; END

Select store, purchase_date, count(Purchase_Price) Over w As Cumulative_Amount_per_Store -- over starts the window function
From smartphone_sales
Where extract(year From purchase_date) = 2019
window w As (Partition By store -- partition clause
             Order By purchase_date Asc -- ordering clause
			 Rows Between Unbounded Preceding -- framing clause
			And current Row);
```
To keep the following queries uncluttered, we omit the with-clause. However, keep in mind, to run the code, you have to add first the with-clause.

With the above query, we determined the cumulative amount of sold smartphones in each store in 2019.

The following figure, taken from the slides [**Advanced SQL**](https://db.in.tum.de/teaching/ws1819/foundationsde/chapter3_updated.pdf?lang=de) presented in the course [**Foundations in Data Engineering**](https://db.in.tum.de/teaching/ws1819/foundationsde/?lang=de), nicely explains each clause: partition, ordering, and framing clause that were used in the above query.

<p align="center">
<img src="/img/window_clauses.jpg" alt="geo" width="300" height="180"/>
</p>

The following query computes the running average price of sold smartphones that include the sold smartphones themselves and a smartphone sold right after and before.
```sql
Select store, purchase_date, purchase_price, sum(purchase_price) Over w As running_average -- over starts the window function
From smartphone_sales
window w As (Order By purchase_date Asc
			 Rows Between 1 Preceding
			And 1 Following);
```

To obtain a ranking of stores where each rank is based on the revenue generated in 2019, we could use the `rank()` function:
```sql
Select store, revenue, rank() Over w 
From (Select store, sum(purchase_price) As revenue
      From smartphone_sales
      Where extract(year From purchase_date) = 2019
      Group By store)
window w As (Order By revenue Desc);
```
Note that if there are stores whose revenues were the largest and were the same, then function `rank()` assigns both the first rank. The store with the second-largest revenue obtains the third rank and not the second rank! On the other hand, the operation `dense_rank()` would assign the store with the second-largest revenue the second rank.

As the above figure hints, the example is based on the course [**Foundations in Data Engineering**](https://db.in.tum.de/teaching/ws1819/foundationsde/?lang=de).

The window functions offers more functionalities, but I think the above examples are for a refresh sufficient.