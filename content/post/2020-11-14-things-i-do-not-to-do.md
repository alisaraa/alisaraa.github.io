---
title: "Things I Sometimes Do in Snowflake/SQL That You Should Not Do"
date: '2020-11-14 00:12:11 -0400'
categories: SQL
---

I have turned into a typical old school parent in that I find myself saying "Do what I say, not what I do" This is usually when I’m sharing my screen, and under the perceived time pressure and awkwardness that comes from people watching you work, I take some SQL shortcuts. These shortcuts help me produce a result quickly, but aren’t scalable, flexible, or readable enough for production. In this post, I’ll explain what the shortcuts are, why they can be dangerous, and how they can be replaced for production ready code. 


### $1

In Snowflake, you can refer to columns in a WHERE clause with a reference to their ordinal position. For example:
```
SELECT * FROM cat_table
WHERE $1 = 'Mac'
```

In this case, if I know that the first column in ordinal position is cat_name, then this will give me info about Mac. This seems great, until it doesn't.

What would happen if you added this to your ETL or analytical scripts and someone added a cat_id in the first ordinal to cat_table? Nothing. Nothing would break. Nothing would be returned. The query would just simply give no results since there is no cat_id = 'Mac'.

And that is scary.

The other scary thing about $1 is that it always refers to the underlying table, not the fields listed in your SELECT statement  (as opposed to GROUP BY 1). For example:

```
SELECT 
    cat_name,
    cat_id
FROM cat_table
WHERE $1 = 'Mac'
```
In this case, if cat_id is the first field in cat_table than this will translate to:

```
SELECT 
    cat_name,
    cat_id
FROM cat_table
WHERE cat_id = 'Mac'
```
which will give no results. 

So not only is the code not scalable in the case of a DDL update, you are also forcing the reader to know that your WHERE clause refers to the underlying table, and not the columns you've listed. It's unclear and dangerous and Mac does not like it. What more convincing do you need?

<p><img src="https://www.alisa-in.tech/images/scary_mac.png" alt="scary_macky"></p>

### Division with /

This is a simple one: division can be scary. If you are adding a field that is a division of two or more fields, consider if the denominator can ever be zero and if so, how do you want your field to react.
When writing the query to do some scratch work, we may write:
```
SELECT
    color,
    SUM(cat_cuteness) / SUM(cat_count) AS avg_cuteness 
FROM cat_table
GROUP BY color;
```
That is okay for now, but what happens once you check in this code, go about your life, and someone upstream enters a record like this:

| color  | cat_cuteness  | cat_count |
|---|---| --- |
| Black | 0 | 0 |

Why would someone add this record? Not sure, but it will happen, trust me. As soon as it does, you will get an error to the effect of:

DIVISION BY 0 NOT POSSIBLE, STOP IT. YOU WILL RIP A HOLE IN THE SPACE TIME CONTINUUM.

Not sure if that's the exact error, but you get the point. So what's the workaround? **Always use [DIV0](https://docs.snowflake.com/en/sql-reference/functions/div0.html) when you want to do division that gives 0 in case of a 0 denominator, otherwise use a specific `CASE WHEN`**
For instance, if I wanted avg_cuteness to zero out if there are no cats:

```
SELECT
    color,
    DIV0(SUM(cat_cuteness), SUM(cat_count)) AS avg_cuteness 
FROM cat_table
GROUP BY color;
```

Otherwise, if I want a specific result in case of zero, I may do:


```
SELECT
    color,
    CASE 
        WHEN cat_count = 0 
            THEN NULL
        ELSE DIV0(SUM(cat_cuteness), SUM(cat_count)) 
    END AS avg_cuteness 
FROM cat_table
GROUP BY color;
```

In either case, your code is more robust. Over time, this will decrease the number of errors you wake up to in the morning, and Itzy appreciates that:

<p><img src="https://www.alisa-in.tech/images/scary_itzy.png" alt="scary_itzy"></p>


### Group by 1,2,4

If I'm rapidly adding to a query and trying to get a result I may end up with aggregate fields and non-aggregate fields living amongst each other, for example:

```
SELECT
   cat_id,
   cat_name,
   avg(cat_cuteness),
   cat_color
FROM cat_info
GROUP BY 1,2,4;
```

Hypothetically, I could have written this after being asked for average cuteness by cat, and, at the last minute, someone on the Zoom said "oh we also need to see it by color." To get results quickly, I would have added cat_color last in the query and added a 4 to the group by, but this is not best practice. 

A lot of people ask why they have to explicitly group by all non-aggregate fields when there is no option (in Snowflake) to not group by them. I think it's just convention to ensure the aggregation is intentional; in MySQL, you do not have to do group by all non-aggregated fields, so there is a precedent for more flexibility in aggregation that one day Snowflake may offer. In the interim, you just have to be explicit about your grouping.

But do it in order and with the names of columns, because this improves readability of the code:

```
SELECT
   cat_id,
   cat_name,
   cat_color,
   AVG(cat_cuteness)
FROM cat_info
GROUP BY  
   cat_id,
   cat_name,
   cat_color
```
Notice that all non-aggregate fields appear before the aggregate field and that the group by explicitly names the non-aggregate fields.
There is no improvement here in performance or error prevention; this is merely a matter of readability. In the future, people will read your code and if the group by uses column names, they can determine the grain of your query at a glance. Additionally, they can easily deduce the aggregate fields if they are all together at the end of the query.

And this is less scary to Cheesie. 


<p><img src="https://www.alisa-in.tech/images/scary_cheesie.png" alt="scary_cheese"></p>
