---
layout: post 
title: "🚛 MySQL optimization tips"
tags: ['database', 'sql']
---

This is my **list of the most commonly used and shared ideas on improving MySQL performance**. I will update this periodically because listing all tips at once would take too much time. 😄
I tried to keep the points simple and short.

## Decrease number of connections to database

[RDBMS are ACID](https://mariadb.com/resources/blog/acid-compliance-what-it-means-and-why-you-should-care/) due to that fact most of the databases work on [single thread](https://dba.stackexchange.com/questions/2918/about-single-threaded-versus-multithreaded-databases-performance).
CPU to fake concurrent work (like playing music, moving mouse, and processing video) uses context switching.
Context switching is a process of saving data stored in CPU registers and restoring different data to CPU registers from cache memory. [More info](https://wiki.osdev.org/Context_Switching)
If you try to send many queries to database at once, concurrently by using multiple connections this will decrease performance. 
CPU will start switching the context more often to fake that concurrency and process every query "at the same time". 
Lower number of connections to database will queue queries and let the database to finish work faster.

### What server for a database?
Choosing a good server for a database we can sum up to following points: 
- high cache memory
- can be single-threaded
- high disk I/O (SSD)
- RAM is not that important as CPU and fast disk I/O

## Use indexes 

Instead of creating multiple indexes with one column, create indexes with multiple columns, and use them correctly. 
The column order in an index must be exactly the same as column order in `where` clause.

## Use window functions

All major RDBMS supports window functions and since 2018 MySQL 8.0 also have it!

## Use interval for dates

Let's imagine you can to query transactions between specific dates o from last 30 days. 

```sql
WHERE 
    DATE(created_at) > '2021-02-23'
```

Don't write date comparisons that way. It cannot use an index involving `created_at` because it is hidden in `DATE()` function call. 
Write instead:

```sql
WHERE 
    DATE(created_at) > NOW() - INTERVAL 30 days
```

## Summary tables

Create summary tables with **redundant data**. Sounds crazy? Ya, I know, but the idea isn't new, and I'm not the original author of this concept.
Sometimes when your database is near the edge of performance and there are many reasons to not upgrade it (costs!) then you can consider creating summary tables.

Creating complex queries and optimizing it sounds fun, but once your query have over 140 lines it's getting worse. 
You can pack them into SQL views or common table expresion, but constant updating it makes it really annoying in maintenance. 

Here is how it looks. Let's image I have many online `stores`. Each of them make some `transactions` with different payment types, like `credit_card`, `cash`.
Everything works good when you have only a few stores and number of transactions is fairly low. Reports are generated in a few seconds (max 3 seconds are acceptable).
It's getting slower when you have more and more transactions over time. You start with query optimization, adding indexes, checking the SQL engine and stuff. 
You can spin up a faster instance for a database and double the costs. On the other hand people are querying this data not so much often. 
It would be a huge waste of resources and money.

Summary tables for the rescue 🥸

datetime | `credit_card` | `cash` | average | sum | store_id
--- | --- | --- | --- | ---
2019-10-01 | 543 | 14 | 15.2 | 8263 | 1
2019-10-02 | 263 | 56 | 12.4 | 8472 | 2
2019-10-01 | 853 | 58 | 6.8 | 9272 | 1
2019-10-02 | 873 | 43 | 5.2 | 7930 | 2

Transactions are stored in date groups. Now, you have to deal with much smaller number of rows to fetch and query. 
If store makes ~500 transactions each day, and you group them into days this will decrease number of potentially processed rows by 500. 
Instead of dealing with 100_000_000 rows, now you can just work only 2 millions.

You can update the summary table using SQL triggers, invoke code periodically in application to update it or calculate it in real-time in application code when a new transaction is inserted.

> Checkout also star schema and snowflake schema. It more advanced structure which is used to keep multi-dimensional data in SQL. Above example is just a easy to introduce solution. :) 

## Use different technology

There are many services out there which can help in optimization. Instead of using you SQL database for every search, you can try out combine 2 technologies.

### ElasticSearch

Use ElasticSearch for better full text searches. Using `like` keyword in SQL databases is not that efficient as in specialized tools for that job. 

### Apache Lucene

Instead, using whole ElasticSearch service you can just take advantage from Lucene indexes (which are used in ElasticSearch).
For Java and Hibernate you can use [Hibernate Search](http://hibernate.org/search/) extension.

### Some other topics to read about
- `explain` keyword in SQL
- Differences between SQL engines 
- Common table expressions
- 
### Resources
- [https://stackoverflow.com/questions/54330657/need-some-guidance-to-optimize-reporting-in-mysql](https://stackoverflow.com/questions/54330657/need-some-guidance-to-optimize-reporting-in-mysql)
- [http://mysql.rjweb.org/doc.php/summarytables](http://mysql.rjweb.org/doc.php/summarytables)
- [https://mysqlserverteam.com](https://mysqlserverteam.com)
- [https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)
- [https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing)

