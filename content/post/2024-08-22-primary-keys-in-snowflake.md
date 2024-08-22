---
title: "Primary Keys in Snowflake are Very Demure, Very Mindful"
date: '2024-08-22 00:12:11 -0400'
categories: SQL
---

All database tables should have a primary key, whether its one column or several. Primary keys indicate what defines a row. Without a primary key, the data cannot be understood or maintained correctly; you cannot identify duplicates or updates. So we know tables should have primary keys, but since the uniqueness of them isn't [enforced in Snowflake](https://docs.snowflake.com/en/sql-reference/constraints-overview#constraints-in-get-ddl), why should you define them in Snowflake itself?

## Why should you define primary keys in Snowflake if they aren't enforced?

You should still define primary keys in a table to:
* communicate the grain of the table
* create dynamic SQL checks
* enforce not null constraints

### Communicate the grain of the table
Once a primary key is defined in Snowflake, anyone with access to the table can see its primayr keys by using the `get_ddl()` [command](https://docs.snowflake.com/en/sql-reference/constraints-overview#constraints-in-get-ddl). This helps end users understand the grain of the table they are using. 

If you use a data dictionary product, this metadata can often be ingested and displayed there.

Also, engineers who have to make or edit a `merge` statatement benefit from the primary keys; this helps determine what they should `merge` on when we update this table.

### Create dyanmic SQL checks
Primary keys can also be found with a `show` [command](https://docs.snowflake.com/en/sql-reference/sql/show-primary-keys).

For instance, `show primary keys in account` will show you all your primary keys at once. 
This allows you to use that output to dynamically create any checks you may need, effectively replicating what Snowflake would do if primary keys were enforced.

An example query could be:

```
SELECT 1 FROM table1
QUALIFY COUNT(*) OVER (PARTITION BY <pk list>) > 1
```

These checks may be expensive and, dependending on the cadence, may catch bugs too late for your needs, but knowing that the metadata of the constraints is available can be helpful when designing QA architecture.


### Enforce not null constraints
While the uniqueness of a primary key is not enforced, by default in Snowflake, all columns that make up the primary key cannot have any nulls.

When primarys keys can be one or more columns, called [composite primary keys](https://www.geeksforgeeks.org/composite-key-in-sql/), you may not want to observe this constraint. However, if you insert any nulls into any primary key columns you will get the following error:

`NULL result in a non-nullable column`

If this is a composite key, and you want to remove the null constraint on that column, you can run:

`ALTER TABLE cat_table ALTER COLUMN cat_name DROP NOT NULL;` 

[source](https://docs.snowflake.com/en/sql-reference/sql/alter-table-column)

But in reality, all cats should have names so maybe look into that data source before you run that command.