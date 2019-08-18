---
layout: post
title: Functions in Where Clauses
date: '2019-08-18 15:12:11 -0400'
categories: SQL
---
This post is part of a new series of mini-blogs I’m doing on SQL performance tuning tips. They aren’t in any particular order. You may have woken up this morning wondering “how does my database deal with functions in WHERE statements?” and that’s a really good question. Let’s say you have a table of kitten data, and you want to filter down for kittens adopted in the past 30 days:

You might write the following:

```
SELECT * FROM KITTEN_DATA
WHERE dateadd(days, 30, adoption_date) > current_date
and adoption_date < current_date
```

Out of 1,033,397,442 rows in kitten_Data, the results show only 932385 were adopted in the past 30 days. Results:
	<figure >
	<img src="https://github.com/alisaraa/alisaraa.github.io/blob/master/images/kittens_results.png?raw=true" height="300"><br>
	</figure>

You may think that this a great performance improvement since you decreased the table by 9%. But what did you really do?

The truth is, your database does not know which rows to throw away or keep until they’ve made this comparison, and it can’t make this comparison until it has completed the calculation on every single row. So when you asked it to cut down Y% of the data, first it did the following on all the data:
	<figure >
	<img src="https://github.com/alisaraa/alisaraa.github.io/blob/master/images/kittens_new_field.png?raw=true" height="300"><br>
	</figure>


Then, once it had this new third field, it filtered and only returned 9% of the data. So, although you thought you were cutting down the data, you were first calculating over all the data, even the fields you were going to throw out!

What you should do instead is put the function on the other side of the filter:

```
SELECT * FROM KITTEN_DATA
WHERE adoption_date > dateadd(days, -30, current_date)
and adoption_date < current_date
```
I ran both queries (avoiding cached results) and found that the latter took less time than the former, although it was such a fast query it’s hard to quanify how big the impact is.

If you can’t find a way to change your code in this way, try to filter before the function filter. So if I really couldn’t avoid using my function I could first cut the table down with:

```
SELECT * FROM (
SELECT * FROM KITTEN_DATA
WHERE adoption_date > ‘2019-01-01
and adoption_date < current_date
) WHERE  dateadd(days, 30, adoption_date) > current_date
```
This would at least decrease the number of records on which I performed that operation. 

However, this is a pretty well-known concept; other sites such as [MS SQL Tips](https://www.mssqltips.com/sqlservertutorial/3204/avoid-using-functions-in-where-clause/) will tell you the same thing. In fact, I learned it five years ago from a textbook that was probably written twenty years ago.

But, as the MS SQL Tip site says, there is another reason to be wary of functions in WHERE clauses: it prevents your ability to capitalize on indexing which is so important in performance. But I could not find in Snowflake’s [documentation](https://docs.snowflake.net/manuals/user-guide/tables-clustering-keys.html) if this is true of clustering too. I set a cluster key on adoption_date, reclustered fully, and ran both queries from this post and honestly...they both said they only scanned 2 out of 279 (7%) of the partitions. What does this mean? This means that Snowflake **might** be smart enough to use the clustering key for certain functions in the where clause. Barring official documentation from them though, I can’t say that’s consistently true. 
