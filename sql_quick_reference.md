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


### Intro  <a name="intro"></a>
In this project, I plan to add examples of SQL queries that cover different topics, for instance, recursive queries, window function.

This is a living project, i.e. I will from time to time add new useful examples

#### To give credits
The most of examples and SQL queries are based or taken from blog-posts or my university courses. I will at each section add also the corresponding reference so that in case of more interest you can read in more depth the corresponding reference.

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
This table models the edges of the below graph:
<p align="center">
<img src="/img/sql_queries_recursive.jpg" alt="geo" width="300" height="150"/>
</p>
So, the table contains for instance the entry: `(1,2)` because there is an edge coming from the vertex $1$ and going to the vertex $2$.

Now, the goal is to figure out what everything we can reach by starting at the vertex $1$. To solve this problem we use recursive queries.

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
Find all paths starting at $1$.
```sql
select S,T
from paths
where S = 1
```

This example is based on a lecture for the course Einsatz und Realisierung von Datenbanksystemen held by Professor Kemper at the Technical University Munich.