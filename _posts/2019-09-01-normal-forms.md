---
layout: post
title: Normal Forms for Normal People
date: '2019-09-01 15:12:11 -0400'
categories: SQL
---
This post is for my friend Mitch, who, despite his persistent belief that he annoys me with his questions, has reminded me why it’s important that we stay hungry for theoretical knowledge even as we get caught up in our day-to-day work.

## What are normal forms?
Normal forms are properties one can apply to a database to normalize it. Does that not help? I felt like it helped but then I re-read it and I realized it was meaningless. Okay, Wikipedia said database normalization “is the process of structuring a relational database [. . .] in order to reduce data redundancy and improve data integrity.” ([source](https://en.wikipedia.org/wiki/Database_normalization)). This process is a collection of steps you can take. These steps are called normal forms. They each have a name. This is a significantly better definition. Basically, you want the design of your database to allow for easy inserts and updates that don’t cause duplicates and break everything over time, and, to do this, you walk through the normal forms and apply them to your database.

There are ten normal forms; each time a database adheres to an increasingly normalized form, it is said to “satisfy” that form. So, in the process of normalizing your database, you satisfy first normal form (1NF), second normal form (2NF), third normal form (3NF), then Boyce–Codd normal form. Yeah, that is where it gets weird. We are going to focus on only the first three normal forms, as those are by far the most commonly satisfied because, as we will see, the forms get increasingly hard to satisfy and they correct for increasingly rare potential bugs. 

## Are normal forms still relevant?
Normal forms hold a special part in my heart; like others who have taken a database class, I lived and died by them for an entire semester. I read Kimball’s book. I memorized the definition and an example and counterexample for each form. I learned then and I still believe now that Kimball’s work on data warehouses, including normalization, revolutionized data warehousing and paved the way for the rise of big data and data science.

However, I treat the topic differently in a professional setting. Who has time to talk about normal forms when things are breaking? Who has time to explain a normal form to a stakeholder who doesn’t want to do 10 joins? Just add the column to the table!

It is a contradiction that I firmly hold the following two beliefs: normal forms are one of the most important concepts of data warehousing and that normal forms are interview questions unrelated to our “real” job. I have to reconcile these two beliefs.

## What can I add to the conversation about normal forms?
I have to sort out my contradictory feelings on normal forms. To do this, I’m going to walk through the normal forms, not for a five year old (Rhys is six now, by the way, he didn't get a gift from you), but for normal people; for people who talk to stakeholders every day; for people who don’t have infinite time to consider the impact of database design; for people who read the blog posts on this site. So, in short, very, very few people.

My fear in writing a post is that I will simply rewrite the Wikipedia page in a lower grade level. I don’t want to do that. I know you can read Wikipedia. But can I write a post for Mitch who wants to know the theory but only as he can apply it to his job?

## How can I memorize the forms?
You can’t, that’s crazy. Lower your ambitions in life, jeez. I can only help you remember the first three.
I’ve heard this phrase in a few different ways but Wikipedia attributes the following to William Kent:  "[every] non-key [attribute] must provide a fact about the key, the whole key, and nothing but the key". ([source](https://en.wikipedia.org/wiki/Third_normal_form#cite_note-Kent-7)). You say this in the same cadence as “the truth, the whole truth, and nothing but the truth.” This has helped me keep the forms straight in stressful situations like interviews or tests (“the key, the whole key, and nothing but the key”). We will walk through what this means.

If you need to brush up on primary keys, please see the post [here](http://www.alisa-in.tech/eli5/2018/08/15/pks.html) before continuing.

## First Normal Form
### What is 1NF?
The key. The table must have a key, all other fields must depend on that key, and each field value must be atomic.
The concept of fields depending on the primary key may be a little vague, but Wikipedia has a great [page](https://en.wikipedia.org/wiki/Functional_dependency) on functionality dependency. For now, I think it’s sufficient to say that the table has a primary key and this primary key is unique.

### What does violating 1NF look like?
A table in first normal form will look like a table with a primary key that each of the non-primary key fields depend on in some way, meaning they will describe the primary key field and contain one value each. A *violation* of first normal form looks like this:

| Cat ID (PK)  | Cat Name  |  Parents |
|---------------|---------------|---------------|
| 1  | Mac  | Alisa, Adam  |
|  2 | Cheese  | Alisa, Adam |
|  3 | Sleepy  | Alisa  |

An additional violation would be if the table simply did not have a primary key or it had a bunch of unrelated fields, but that is harder to represent visually, and, in my opinion, less likely to happen.

<figure >
<img src="https://github.com/alisaraa/alisaraa.github.io/blob/master/images/adoption_day.png?raw=true" alt="When we got Mac and Cheese"  height="300">
The day we adopted Mac and Cheese
</figure>
 
### What happens if I violate 1NF?
The cat table above has an obvious drawback: it’s extremely hard to filter for the cats that Alisa owns. For instance, you can’t get the right results from:

```
SELECT cat_name FROM not_first_normal_form_cat_table
WHERE parents = 'Alisa'
```

This will actually result in 1 row returned. In order to get the correct answer, you will have to do something like this:

```
SELECT cat_name FROM not_first_normal_form_cat_table
WHERE parents ilike 'Alisa'
```

Which is less performant and less intuitive. Stakeholders may forget to use this syntax and incorrectly report metrics.

If you go back to the original reasoning for why we normalize our databases, you’ll see our concern was redundancy and data integrity.  How is that impacted by first normal form?

Well, let’s try to make an update to not_first_normal_form_cat_table. Let’s say I changed my name to Alisa In-Tech. If this table satisfied 1NF we could do:
<br>
```
UPDATE not_first_normal_form_cat_table
SET parents = ‘Alisa In-Tech’’
WHERE parents = ‘Alisa’
```

But in this case, that would not work; this would only change the parent of Sleepy. We would have to do something like:
<br>
```
DELETE FROM not_first_normal_form_cat_table
WHERE parents ilike ‘%Alisa%’;

INSERT INTO not_first_normal_form_cat_table
SELECT  * FROM source_data
WHERE parents ilike ‘%Alisa In-Tech%’;
```
<br>
This works for the sample data I provided, but what if I had a sister named Analisa? I don’t, but I could if my parents were weird. All the Analisa owned cats would be deleted in the DELETE statement, then not inserted in the INSERT statement. So we have killed off Analisa’s cats. Great job. Hope you feel good about what you did, cat killer.

### Are there any benefits of violating 1NF?
The last section was to bring awareness about the dangers of violating first normal form, but once you know the risks of making a choice, you are then prepared to know when to make that choice. One common violation I see a lot is putting the entire semi-structured source data into a field.<br>
Now that database systems including [Snowflake](https://docs.snowflake.net/manuals/user-guide/semistructured-concepts.html) have gotten better at ingesting semi-structured data such as JSON and Parquet, it has become trivial to store the entire record in one field. Unlike CSV ingestion, JSON ingestion will not break if there are missing keys, a key name change, or field values that change types. With less sensitive ingestion into the write schema, it becomes tempting to just provide this record of the complete data set in the read schema. Since Snowflake also makes it easy for end users to parse JSON (if you know what you are looking for), this cuts down on column add requests when a new key is set to the source.
Is this a good idea for every data source? No. This can cause havoc and JSON blobs are large, making returning results take even longer. I only recommend it when:<br>
* You know you will never do an update, insert, or delete based on the values in the JSON field
* You have a data set that does not change shape much between the source and the production data (so it’s easy to just tack on the JSON field to the end of each record without doing much transformation on it)
* The underlying data set has frequently changing field and keys values and types that stakeholders want to access shortly after a change. If the key names and field data types don’t change often, why not just parse them?

## Second Normal Form
### What is 2NF?
The whole key. This means that every non-key attribute (every field that isn’t the primary key) depends on all the parts of the key. If a table has only one primary key and satisfies 1NF then the table automatically satisfies 2NF. This is the best case scenario for a test question. It saves so much time for that answer. <br>
But let’s say we have a composite [primary key](https://en.wikipedia.org/wiki/Compound_key) which, despite what a candidate once told me during an interview, is possible. 2NF says we should be sure that no non-key field depends on only part of our composite key.
### What does violating 2NF look like?
Without loss of generality, let’s assume our composite key has two fields in our table that tracks cat weights: bad_2nf_cat_weight

| Cat ID (PK)  | Weigh in Date (PK)  |  Cat Weight | Cat Name |
|---------------|---------------|---------------|---------------|
| 1  | 2019-01-01  | 10 pounds  | Mac |
| 2 | 2019-01-01  | 8 pounds | Cheese |
| 1 | 2019-01-02  | 11 pounds  | Mac |
| 2 | 2019-01-05  | 8 pounds  | Cheese |

Technically, the names Mac and Cheese can be determined by (depend on) the primary key {Cat ID, Weigh in Date}. However, Mac and Cheese can also be unique determined by Cat ID (see not_first_normal_form_cat_table). So this is partial dependency; it violates 2NF.

Note: In this case, we would say that the cat's name is “denormalized” on to the weight table. If there is a lot of instances of this, we may say that the database schema is “heavily denormalized.”

### What happens if I violate 2NF?

So is this really bad? Don’t people who want who use the cat table want to know which cat we are weighing? The true 2NF form would be two tables:

cat_weight_fact

| Cat ID (PK)  | Weigh in Date (PK)  |  Cat Weight |
|---------------|---------------|---------------|
| 1  | 2019-01-01  | 10 pounds  |
| 2 | 2019-01-01  | 8 pounds |
| 1 | 2019-01-02  | 11 pounds |
| 2 | 2019-01-05  | 8 pounds  |

cat_dim

| Cat ID (PK)  | Cat Name  |
|---------------|---------------|
| 1  | Mac  | 
|  2 | Cheese |
|  3 | Sleepy |

We would expect the stakeholder to join cat_weight_fact to cat_dim on cat_id in order to find the name. <br>
Why are two tables better than one? Well, let’s say we want to change Mac’s name to Macaroni (that is his full name, by the way, Macaroni In-Tech). To update the violation of 2NF, we would need to update 2 records, while in the 2NF example, we would only need to update one record in cat_dim. <br>
Updating two records versus one is not that big of a deal, but data tends to grow, with fact tables growing much faster than dimension tables. So do you want to continue to update larger and larger tables with table scans? Or do you want to update one record and put the onus on stakeholders to do the join?

### Are there any benefits of violating 2NF?
So when we do we make a calculated decision to disregard 2NF? There’s no hard and fast rule but in general, the following may make you consider violating 2NF: <br>
* When you know the stakeholders will virtually always make this join 
* When the join to get the additional information will be expensive for stakeholders (post on expensive joins it forthcoming)
* When the cat's name (the denormalized field) will rarely if ever be updated 

## Third Normal Form
### What is 3NF?
And nothing but the key. If we know all non-key fields already depend on the whole key, so how could the fields depend on anything other than the key? Well, there’s such a concept of [transitive dependency](https://en.wikipedia.org/wiki/Transitive_dependency) which we want to eliminate in this step of normalization. A transitive dependency is when a non-key field does depend on all parts of the key, but it could also be determined by another non-key field.
### What does violating 3NF look like?
Let’s say we wanted to keep track of what the cats like:
<br>
Bad_3nf_cat_favorite_food

| Cat ID (PK)  | Meal Name (PK) | Favorite Food | Favorite Food Brand | Favorite Food Flavor |
|---------------|---------------|---------------|---------------|---------------|
| 1  | Dinner  |  Blue Buffalo Wilderness Indoor Chicken Recipe Grain-Free Dry Cat Food | Blue Buffalo | Chicken |
|  2 | Dinner | Blue Buffalo Wilderness Indoor Chicken Recipe Grain-Free Dry Cat Food | Blue Buffalo | Chicken |
|  3 | Dinner | Fancy Feast Gourmet Naturals Beef Canned Cat Food | Fancy Feast | Beef |
| 1  | Snack |  Pill Pockets Treats Chicken Flavor | Greenies | Chicken |
| 2  | Snack |  Treats Tuna & Cheese Flavor | Greenies | Tuna |

Technically, every non-key attribute in this table depends on both parts of the primary key {Cat ID, Meal}, but are those two fields combined the only way we can determine each field? What I mean is-  can we only determine that Mac’s favorite dinner food brand is Blue Buffalo from knowing that those two fields? Or could we determine that from the fact that his favorite food is Blue Buffalo Wilderness Indoor Chicken Recipe Grain-Free Dry Cat Food? The latter; that canned food is only made from one brand, Blue Buffalo. So we have a transitive dependency.
The solution would be, once again, making two tables (normalization tends to create more tables and denormalization tends to collapse or combine tables):<br>

cat_favorite_food

| Cat ID (PK)  | Meal Name (PK) | Favorite Food |
|---------------|---------------|---------------|
| 1  | Dinner  |  Wilderness Indoor Chicken Recipe Grain-Free Dry Cat Food |
| 2 | Dinner | Wilderness Indoor Chicken Recipe Grain-Free Dry Cat Food |
| 3 | Dinner | Gourmet Naturals Beef Canned Cat Food | 
| 1  | Snack |  Pill Pockets Treats Chicken Flavor | 
| 2  | Snack |  Treats Tuna & Cheese Flavor | 

cat_food_dim

| Food Name (PK) | Food Brand | Food Flavor |
|---------------|---------------|---------------|
| Wilderness Indoor Chicken Recipe Grain-Free Dry Cat Food | Blue Buffalo | Chicken |
| Wilderness Indoor Chicken Recipe Grain-Free Dry Cat Food | Blue Buffalo | Chicken |
| Gourmet Naturals Beef Canned Cat Food | Fancy Feast | Beef |
|  Pill Pockets Treats Chicken Flavor | Greenies | Chicken |
| Treats Tuna & Cheese Flavor | Greenies | Tuna |

Note that in reality, we may use a surrogate key for Food Name since it’s so long, but I didn’t want to add additional numbers to this example lest I take away from the point.

Once again, we’re asking the stakeholders to join these two tables if they want to know the brand of Mac’s favorite dinner food. 
### What happens if I violate 3NF?
Let’s say you go with the first version of the table: bad_3nf_cat_favorite_food. This will get 2x bigger for every cat we get (assuming cats only eat dinner and snacks). So if a brand updated its brand name, it would take two times the compute to update bad_3nf_cat_favorite_food rather than cat_food_dim.
 ### Are there any benefits of violating 3NF?
The benefit of violating 3NF if similar to that of 3NF: fewer joins for stakeholders. So if the join would be common, expensive, or the only way someone would view the data, consider denormalizing.
