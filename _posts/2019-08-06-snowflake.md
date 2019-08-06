---
layout: post
title: Cool Things in Snowflake
date: '2019-08-06 15:12:11 -0400'
categories: Engineering
---

The title of this post would be “10 Things I Like to Use in Snowflake That You Could Google but First You’d Have to Know They Exist. So Here, They Exist”, but I’m not that good at HTML and CSS so I can’t fit such a wide title on my blog post page. 

Snowflake is a cloud-based data warehouse similar to Redshift. It stores data in a columnar fashion, [see my post on types of data warehouse storage](http://www.alisa-in.tech/eli5/2018/07/03/rowbased.html). I’ve used Snowflake off and on for about four years, starting with a transition from Redshift to Snowflake at $job-3. At $job-2 I went to Vertica. At $job-1, I worked on another transition from Redshift to Snowflake. At $job, I came in to team already using Snowflake. This allowed me to shift my focus from setting up Snowflake to optimizing it. 

Below are the helpful features for efficient and scalable ETL and ad-hoc querying.

## 1. Ingest metadata when you copy


https://docs.snowflake.net/manuals/user-guide/querying-metadata.html

COPY is a great way to get data from an external location (AWS S3 or Microsoft Azure or the newly added feature of Google Cloud). However, Snowflake didn’t invent the COPY command; that feature alone would not make it to this list. It is the metadata feature that differentiates it.

When you copy into a Snowflake table, you can optionally add two fields of metadata:
File name- add METADATA$FILENAME to your SELECT statement
Row number- add METADATA$file_row_number to your SELECT statement

An example from the website linked above on how to use it: 

``` select metadata$filename, metadata$file_row_number, t.$1, t.$2 from @mystage1 (file_format => myformat) t; ```

The file name will appear as the full path of the file after the bucket name. So if your stage points to s3://kittens/ and your file is s3://kittens/cats.csv, the filename will be cats.csv. But if your stage is  s3://kittens/cats/macandcheese.csv, then the query will show cats/macandcheese.csv. Keep this in mind for parsing.

This is beneficial for:
Debugging
Which file did this bad row come from?
How many distinct files are impacted by X?
QA
How many distinct files did we load over time?
What file did we most recently load?
How many files have we loaded?
Data
Metadata is data. If your file names have important information in them, such as the name of the customer or the datetime the file was sent, this can be parsed into a field in the table.

To be honest, I never use the row number feature. I don’t usually care what row number a record was in its original file. I can see no business use for it, but email me your use cases if you have them!

## 2. Merge, if you don’t already

https://docs.snowflake.net/manuals/sql-reference/sql/merge.html

A MERGE is the combination of an update and an insert, sometimes called an upsert. This isn’t a totally new concept (other databases implement the feature differently), but it deserves a mention.

A MERGE allows you to add new records and update existing records without any part of the table being unavailable to end users. If you want to have a production schema that is up 24 hours a day (and you do), use this. 

Be careful:
You need to know the primary key; that will define what it means for a record to be new or existing
You can delete in a merge, but only if the source data does soft deletes, not in the case of hard deletes. This is a bit of a complicated concept, but just know, you may have to do a separate DELETE statement after the MERGE.
Benefits:
You do not have to re-run permissions after a MERGE, as you do in a create and replace
You do not lose any time travel, as you do in a create and replace
The data will never be removed from the table as in a TRUNCATE; so to the end user, the data will always be available
The merges enforce the primary key since duplicated data causes the merge to fail

## 3. Clone a development environment

https://docs.snowflake.net/manuals/sql-reference/sql/create-clone.html

There’s not much to say about cloning, except that it’s fast and efficient and cheap. You can clone any object (database, schema, table, etc). Cloning uses the metadata of the object so it’s fast and takes no new storage. The clone is independent of the source so it can be altered, although once it’s altered it does take up storage but only as much as it needs. So there is very little cost to making a clone of the production database every so often as a development database; the benefit is an up to date sandbox to develop new changes in as accurately as possible. 

## 4. ZEROIFNULL
https://docs.snowflake.net/manuals/sql-reference/functions/zeroifnull.html

I don’t know why I love this function so much, I think it may be because it’s so common and ugly to see  ``` CASE WHEN field == 0 THEN NULL ELSE field END ```  in ETL. ```ZEROIFNULL(field)``` is not faster than the  ``` CASE WHEN ``` statement, but it cuts down on syntactic noise. More readable code is easier to debug code. 

## 5. Unload a dataset to S3 if you need to
https://docs.snowflake.net/manuals/user-guide/data-unload-s3.html

If you have a table you want to backup to S3 (or wherever your cloud storage is), you can write what looks like a COPY statement and run it like any other SQL statement within your ETL. There is no need to write a special Python script to extract and save to S3. 

It is easy to save a file to a location in S3 as CSV, as in the following:
```
copy into s3://mybucket/unload/cats from cats
FILE_FORMAT = (TYPE = CSV)
```

This will create s3://mybucket/unload/cats.csv. Actually it won’t, it will create as many cat files as it wants. In order to create cats.csv and only cats.csv, use the parameter SINGLE=true. In order to refresh the cats every time the COPY runs, use the parameter OVERWRITE=true. 

But CSV isn’t always ideal; datasets change shape and you may want to ingest historic data which does not contain all the same columns. In this case, you can unload a table as JSON object, but you have to convert it to an object using the OBEJCT_CONSTRUCT function: https://docs.snowflake.net/manuals/sql-reference/functions/object_construct.html 
 

## 6. Use ANY_VALUE when any value will do
https://docs.snowflake.net/manuals/sql-reference/functions/any_value.html

There are times in our lives, of which we are not proud, that we must aggregate a field that we truly don’t want to aggregate. For instance, let’s say you have a dataset that looks like this:

| City       | Zip Code | Number of Cats|
| ---------- |---------- | ---------- |
| Boston      | 02120 | 2 |
| Boston      | 02139      |   5 |
| Boston      | 02127      |  25 |
| Worcester | 01061     |    1 |
| Worcester | 01606     |    6 |

Now let’s say you want to display the sum of cats by city, but you still want to display a zip code for reference. You know that you have to either aggregate or group by every field, but you don’t want to aggregate zip code or group by it, since that would not give you the total cats per city. In this case, you may be tempted to do a MAX() on zip code but you do not truly want the max zip code and that requires a calculation. So what do you do?

```
SELECT city, ANY_VALUE(zip_code) as some_zip, SUM(Number_of_cats) AS total_city_cats
FROM  cat_city_table GROUP BY city
```

This is computationally faster than a MAX(). Note though that is non deterministic, which means it will reproduce different results throughout time. Be careful with this. Speaking of which...

## 7. Be careful with ROW_NUMBER
https://docs.snowflake.net/manuals/sql-reference/functions/row_number.html

ROW_NUMBER() is a powerful analytical function which will break ties for you so to speak. RANK() will number the groupings of rows by the partitioned by fields for you; ROW_NUMBER() will number them but ensure there’s no duplicates. This means it will randomly (not really randomly but randomly) pick between two rows with the same field values for all the partitioned fields and the order by field. This can help you deduplicate cleanly, but the results will be non deterministic, meaning each time you run the same query you may get a different result. Since you’re letting Snowflake effectively pick your data, it’s unwise to use this feature in your ETL.

## 8. Be careful with NULLs but use them to your advantage
https://docs.snowflake.net/manuals/sql-reference/functions/equal_null.html

You always have to be careful with nulls. As you may have heard, null is not equal to null. This means you can’t do X = Y and get TRUE when both X and Y are null. This is specifically troubling in a MERGE statement. If one of your primary keys can be null, a merge comparing the source and target values of the primary key will always be false if they’re both null. This means it will always trigger the WHEN NOT MATCHED CLAUSE. This means, dupes.

There are other cases when the null does not equal null thing gets in your way, and this isn’t specific to Snowflake. In fact, I remember having this issue with Vertica and having to do something like

COALESCE(src.cat_field, ‘mac’) = COALESCE(targ.cat_field, ‘mac’)


This is a workaround, yes, but not a good one. Why?
The hardcoded value has to be the same as the data type of your field, so you have to think of strings and ints and dates and datetimes to use as your default
On the off chance that src.cat_field is null but targ.cat_field = ‘mac’, you will get a false match

Snowflake provides you with a function EQUAL_NULL() which will return True if the fields are equal or if they are both null. This is safer and cleaner than the workaround.

How can we use this to our advantage? Let’s say you have some values for cat color, most of which are black, white, torty, orange, but some are null. You want to find all of those who are not black so you do
```
SELECT * from cats
WHERE cat_color != ‘Black’
```

But you notice-- the cats without colors yet are gone! That was not your intention. This is because null != ‘Black’ because null is not equal to anything and therefore cannot be not equal to anything. To get all colors including nulls except black, you can do:

```
SELECT * from cats
EQUAL_NULL( cat_color,  ‘Black’)  = false 
```
(or NOT EQUAL_NULL(cat_color, ‘Black’))

As long as you remember that nothing equals null, including null, then you should be able to use EQUAL_NULL() to write clean code.

## 9 Primary Keys (and other constraints)
https://docs.snowflake.net/manuals/sql-reference/constraints-overview.html

I know what you’re thinking: Snowflake doesn’t enforce constraints, next. Not so fast. Whenever I think about declaring constraints in a database that does not enforce them (looking at you too, Redshift), I think about the Baz Luhrmann quote from Everybody’s Free: “Read the directions, even if you don’t follow them.” (https://genius.com/Baz-luhrmann-everybodys-free-to-wear-sunscreen-lyrics)
 
I know Snowflake doesn’t do the work for you, but the primary key of a table is still metadata, and Snowflake store metadata. If you declare a primary key in the DDL, you can find the primary key from a table by querying:

``` DESC TABLE cats ```

Which will show you a list of each of the fields in the table cats, and which are part of the primary key. It’s not too hard to programmatically use this to find all primary keys in your database and check for the existence or uniqueness of the field. Declare the primary key, even if you don’t enforce them.  

Note that the NOT NULL constraint is enforced and should be used for columns that aren’t ever supposed to be null.

## 10 On that note, use RESULTS SCAN to get non-table results into a table

https://docs.snowflake.net/manuals/sql-reference/functions/result_scan.html

I know what you are thinking: DESC TABLE doesn’t return a real table. I can’t do a WHERE clause or select only a subset of columns. I’d have to ingest it into a Python or an R dataframe in order to manipulate it. 

Not so fast. Snowflake has a workaround for this. Immediately after doing a DESC or a SHOW command, you can run

``` select * from table(result_scan(last_query_id())) ```

and you will receive the results of your last query as a table, which means you can limit your columns or add a WHERE clause.

I admit that it feels janky and it’s per user name, not session, which means you can get in trouble with getting the results of your query.

### Email me to let me know what you think, if you have any other tricks you'e picked up, or if you have any questions!
