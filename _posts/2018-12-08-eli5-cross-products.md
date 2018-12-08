---
layout: post
title: ELI5 - Cross Products
date: '2018-12-08 15:12:11 -0400'
categories: ELI5
---

In order to understand cross products, you have to understand joins since cross products are a type of join. Also, in order to understand joins, you need to understand cross products since all joins are just cross products. It’s a horrible world we live in.

A cross product is also called a [Cartesian product](https://en.wikipedia.org/wiki/Cartesian_product) by people who trust their ability to spell when a spell checker isn’t available. That particular group of people isn’t inviting me out to lunch any time soon, so I usually stick with cross product. This time, I will be explaining the concept of joins and cross products to my niece, Winnie. Winnie is three and adorable and a terrible flower girl. She didn’t drop the petals until the end of the aisle, and then she tried to pick them up before sitting down because “my mess, my mess.” She’s cute as anything though. I will be explaining to Winnie:
What is a cross product join?
Why is that potentially bad?
What kind of joins do we do instead?<br>
	<figure >
	<img src="https://github.com/alisaraa/alisaraa.github.io/blob/master/images/winnie_wedding.jpg?raw=true" alt="Winnie and Alisa at the Wedding" height="300"><br>
Winnie and Alisa at the wedding <br>Source https://www.momentsphotographyvt.com
	</figure>

**What is a cross product join?**

So Winnie, as we get ready for the wedding, remember that we can choose from several different outfits. Specifically, Grandma bought us two dresses and three pairs of shoes. The dresses are white or ivory, and the shoes are gold, silver, or green. Let’s see how many different ways we can create an outfit, which it to say, put together one dress and one pair of shoes.

|     | Gold        | Silver           | Green  |
| --- | ------------- |-------------| -----|
| White | White dress, gold shoes     | White dress, silver shoes | White dress, green shoes |
| Ivory | Ivory dress, gold shoes      | Ivory dress, silver shoes | Ivory dress, green shoes |

This is a join of the shoes to dresses data. This is a cross product.  So we have six different combinations that we can wear, and that is entirely accurate. 3*2 = 6. Great. So then how come people talk about cross products like they’re a bad thing? Outfit options are cool…

**Why is this potentially bad?**

It’s great that you have six options Winnie, but those are just options. Let’s say the bride wanted a second flower girl (I know, I know, she wouldn’t do that). If you wore the “white dress, gold shoes” option, could Other Flower Girl wear the “ivory dress, gold shoes” option? No. We simply don’t have two pairs of gold shoes. So while all these combinations are valid, they cannot all be true at the same time. 

It is in assuming that the results of a cross product can all be simultaneously true that mistakes are made. Let’s say you know Sales Company did $1000 in sales in March, and there are two sales agents, Agent B and Agent C. You don’t know who made which sales, but you join the sales and agent tables anyway. As a result, you get Agent B made $1000 in sales, and Agent C made $1000 in sales. If you select one or the other, you are attributing sales to one person, but your total will be correct. However, if you leave the data as is, you change the total sales of Sales Company from $1000 to $2000, incorrectly stating sales by 100%.

If we tell people you have six outfit options, that’s okay. But if we tell people you have six outfits, that’s wrong. We cannot clothe six flower girls.

**What kind of joins do we do instead?**

How many outfits do we truly have, assuming we need one pair of shoes and one dress for each outfit? We have two because we only have two dresses. We only want two results when we do our [join](https://en.wikipedia.org/wiki/Join_(SQL)). We have to therefore define how we want to combine our dresses and shoes (this can be called the on clause or the join predicate; all joins can be thought of as cross products that then apply this definition).

So let’s say, for matching purposes, gold can only go with ivory and silver can only go with white. Green can go with another color shoe, but not one that we have. (I am glossing a bit over this definition- how do we know what goes with what? This is the sort of logic that would come from the business. In the Sales Company example, the data would tell us somewhere who made that sale, Agent B or Agent C. If it doesn’t, then we shouldn’t be joining the tables at all).

|     | Gold        | Silver           | Green  |
| --- | ------------- |-------------| -----|
| White | ~~White dress, gold shoes~~     | White dress, silver shoes | ~~White dress, green shoes~~ |
| Ivory | Ivory dress, gold shoes      | ~~Ivory dress, silver shoes~~ | ~~Ivory dress, green shoes~~ |

So now, we have your two outfits! But of course, you can only wear one. Which one do you pick?
<figure >
		<img src="https://github.com/alisaraa/alisaraa.github.io/blob/master/images/winnie_shoes.jpg?raw=true" alt="Winnie and her Shoes"  height="300">
Winnie and her shoes <br>Source https://www.momentsphotographyvt.com
	</figure>

