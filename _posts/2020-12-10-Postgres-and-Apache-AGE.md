---
layout: post
title: Postgres and Apache AGE
tags: [Postgres, AGE, PostgreSQL]
---

# {{ page.title }}

Apache AGE is a very new and incubating extension for Postgres that gives it graph database capabilities. It is a really neat project that is inspired by [Bitnine's AgensGraph](https://bitnine.net/agensgraph/), which is a PostgreSQL 10 (10.4 right now) fork which provides graph functionality as well.

I've been fortunate to have talked to some of the core AGE team members and I'm really excited about the direction of the project.

That said, I thought it would be fun to take AGE for a test run to show some of its raw power, which is the expressive Cypher Query Language.

## Baseline Setup

The first thing in the journey I needed to do was to get AGE setup and running locally. I highly recommend just cloning the `age-compose` repository that I mention below, as it provides some sample relational data and some sample graph data and helper functions. 

Since AGE is still in alpha, I decided I want to keep the experience containerized, so I created a couple of Docker images:  one [Debian based](https://hub.docker.com/repository/docker/sorrell/agensgraph-extension), and one [Alpine based](https://hub.docker.com/repository/docker/sorrell/agensgraph-extension-alpine). These containers are based on the Postgres 11 official images, but also will clone, build, and install AGE. AGE is locked in PG11 right now, but they are working on making it compatible with newer version of PG. 

One of the great things that Bitnine offers in their [agensbrowser Docker image](https://hub.docker.com/r/bitnine/agensbrowser) is some sample data in the form a Northwind database. This data is traditional RDBMS stuff - tables and relations. No special graph database nodes or relationships.

But since Northwind isn't built into the Docker images I created, I decided I would put that data into a Docker compose project called [age-compose](https://github.com/sorrell/age-compose). Upon adding this data though, I started looking for a way to turn the traditional RDBMS tables of employees and territories into graph nodes that I could use with AGE.

In AgensGraph, they created some helpers with a `LOAD FROM (table)` syntax. Since that functionality is still in the works with AGE, I decided I would create a plpython function to help with this task (because loading the graph row-by-row would be insanity). I also added this function to the `age-compose` project, [and you can see it here](https://github.com/sorrell/age-compose/blob/master/docker-entrypoint/initdb.d/20-initgraph.sql#L10). (**Note:** as of this writing, this functionality only works in the Debian-based Docker image.)

I certainly don't claim that to be the most beautiful or performant implementation, but it really scratches the itch when you want to turn a table into a bunch of nodes with the same label. It made that process as easy as calling `SELECT load_graph_from_table('employee', 'employees');`.

## Querying The Graph

Now that I had a running container with some sample data loaded, it was easy to query the graph. To query the graph using AGE, we need to stuff the Cypher query into a function call that looks like this:

```sql
SELECT * from cypher('my_graph_name', $$
  CypherQuery
$$) as (a agtype);
```

So if we wanted to see an employee with the last name of "Buchanan", we could do something like the following:

```sql
select * from cypher('northwind_graph', $$ 
  MATCH (a:employee { lastname: 'Buchanan' }) 
  RETURN a 
$$) as (employee agtype);

-----RESULT (extended display on)
-[ RECORD 1 ]---------------
employee | {"id": 1407374883553285, "label": "employee", "properties": {"city": "London", "notes": "Steven Buchanan graduated from St. Andrews University, Scotland, with a BSC degree in 1976.  Upon joining the company as a sales representative in 1992, he spent 6 months in an orientation program at the Seattle office and then returned to his permanent post in London.  He was promoted to sales manager in March 1993.  Mr. Buchanan has completed the courses Successful Telemarketing and International Sales Management.  He is fluent in French.", "photo": "\\x", "title": "Sales Manager", "region": null, "address": "14 Garrett Hill", "country": "UK", "hiredate": "1993-10-17", "lastname": "Buchanan", "reportto": 2, "birthdate": "1955-03-04", "extension": "3453", "firstname": "Steven", "homephone": "(71) 555-4848", "photopath": "http://accweb/emmployees/buchanan.bmp", "employeeid": 5, "postalcode": "SW1 8JR", "titleofcourtesy": "Mr."}}::vertex
```

Notice that the return type is `agtype`, which is saying that we are going to return a vertex (node) or an edge (relation). This could be changed, for example, to `text` if you were to just `RETURN a.firstname`. Notice that the result of this specific query is of type `vertex`. 

If we wanted to explore who Buchanan reports to, we could do the following:

```sql
select * from cypher('northwind_graph', $$ 
  MATCH (a:employee { lastname: 'Buchanan' })-[r:REPORTS_TO]->(b:employee) 
  RETURN r
$$) as (rel agtype);

-----RESULT (extended display on)
-[ RECORD 1 ]---------------
rel | {"id": 3940649673949188, "label": "REPORTS_TO", "end_id": 1407374883553282, "start_id": 1407374883553285, "properties": {}}::edge
```

Notice a couple things here. First, that `start_id` matches Buchanan's `id`, which makes sense. The `end_id` is the `id` of the employee that Buchanan reports to, which we could also see by executing `RETURN b` instead of `RETURN r`.

## Comparing Apples and Oranges 

Now that we've kicked the wheels a little bit, I wanted to run some baseline comparisons. One of the canonical examples when it comes to comparing graph and relational databases is the "employee hierarchy" query. This query shows the user who manages different employees. Here's what the query should return, a list of subordinates and their manager (name and title for both):

```sql
-----RESULT
 subord_lastname |       subord_title       | mgr_lastname |       mgr_title
-----------------+--------------------------+--------------+-----------------------
 Buchanan        | Sales Manager            | Fuller       | Vice President, Sales
 Callahan        | Inside Sales Coordinator | Fuller       | Vice President, Sales
 Davolio         | Sales Representative     | Fuller       | Vice President, Sales
 Dodsworth       | Sales Representative     | Buchanan     | Sales Manager
 King            | Sales Representative     | Buchanan     | Sales Manager
 Leverling       | Sales Representative     | Fuller       | Vice President, Sales
 Peacock         | Sales Representative     | Fuller       | Vice President, Sales
 Suyama          | Sales Representative     | Buchanan     | Sales Manager
```



In the relational world, this consists of a recursive CTE (common table expression). Here's what that query looks like:

```sql
WITH RECURSIVE recursive_cte
AS ( 
  SELECT employeeid, lastname, title, reportto
  FROM employees
  WHERE reportto IS NULL
  UNION ALL
  SELECT e.employeeid, e.lastname, e.title, e.reportto
  FROM employees e
  INNER JOIN recursive_cte r 
    ON e.reportto = r.employeeid
  WHERE e.reportto IS NOT NULL 
)
SELECT r.lastname AS subord_lastname
  , r.title       AS subord_title
  , e.lastname    AS mgr_lastname
  , e.title       AS mgr_title
FROM recursive_cte r
LEFT JOIN employees e 
ON e.employeeid = r.reportto
WHERE r.reportto IS NOT NULL 
ORDER BY r.lastname;
```

And in the AGE-powered relational/graph world, the **Raw AGE query** looks like this:

```sql
SELECT * FROM cypher('northwind_graph', $$
  MATCH (n:employee),(m:employee)
  WHERE n.reportto = m.employeeid
  RETURN n.lastname, n.title, m.lastname, m.title
  ORDER BY n.lastname
$$) AS (subord_lastname agtype, subord_title agtype, mgr_lastname agtype, mgr_title agtype);
```

We could make this a bit more expressive and performant by creating a relation here though, and that's already done in the `age-compose` setup step that looks like this:

```sql
DO $$ BEGIN RAISE NOTICE 'CREATING Employee-Mgr Relationship'; END $$;
SELECT * FROM cypher('northwind_graph', $$
  MATCH (n:employee),(m:employee)
  WHERE m.employeeid=n.reportto
  CREATE (n)-[r:REPORTS_TO]->(m) 
  RETURN toString(count(r)) + ' relations created.'
$$) AS (a agtype);
```

Utilizing that new relation, we can run the **Relation AGE query** below:

```sql
SELECT * FROM cypher('northwind_graph', $$
  MATCH (n:employee)-[r:REPORTS_TO]->(m:employee)
  RETURN n.lastname, n.title, m.lastname, m.title
  ORDER BY n.lastname
$$) AS (subord_lastname agtype, subord_title agtype, mgr_lastname agtype, mgr_title agtype);
```

It's pretty clear that the Cypher is much more concise and easier to maintain. And this is a HUGE win when it comes to working with code and data - maintainability is key. Recursive CTEs are often confusing because you don't encounter them a lot, and when you need to modify them there is often some Googling involved. This is much different compared to Cypher, which is very expressive and concise.

## Performance

I was curious how the two queries would perform against each other in terms of query time. This is a very unscientific comparison, and probably unfair since AGE is very alpha right now, but worth looking at.

I simply ran each query from the `psql` command line with `\timing` on and averaged 10 runs. Here's how they performed:

```
 Raw AGE query: 3.681 ms
 Relation AGE query: 1.489 ms  
 CTE query: 0.801 ms
```

##  Conclusions

While the CTE is faster, on this small data set it's really negligible. Especially when you factor in the readability improvements the AGE query provides. AGE is an exciting project that will be getting a ton of improvements in the next year, and I'm excited to see where it goes.




