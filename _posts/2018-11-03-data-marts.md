---
layout: post
title: ELI5 - Data Marts
date: '2018-11-03 15:12:11 -0400'
categories: ELI5
---
In this post, I will explain the concept of data marts with approachable language, but I won't be talking to my 5 year old nephew Rhys. This time, I will be speaking to anyone who has ever interacted with children before. 

Data marts are an important part of a data warehouse, the details of which you need to erase from your mind before you read this post so we can start fresh. A data mart is a subset of all stakeholder-facing data available (stakeholders are internal users of the warehouse). Why we choose to create a subset and how we decide what goes into each set will be the content of this post.

Some of you may remember Rhys from earlier posts; we have already covered that he is five and has tons of toys. These attributes are core to the analogy, but it is also important to know that he has a three year old sister, Winnie (it is short for Gwendolyn but we are *not* allowed to call her Gwen).

Children ask for toys and often get them. This is good; it's healthy that they are able to express their wants. That being said-- toys. Toys everywhere. Once Rhys gets a toy, he plays with it immediately, but at bedtime it will end up in the toy bin.

The toys are data. Stakeholders ask for the data, which I always appreciate, because no more data requests means no more paychecks. But the toy bin is the data lake, which is a common term for an area that houses all of a company's historic and current data for in whatever forms it comes in.

Have you ever looked for a something in a toy bin? You are likely going to get stabbed by a G.I. Joe's plastic gun thing. I have had Barbie's feet push under my nail bed so far that it made me bleed. You also aren't going to find what you're looking for; you may find a bench that is part of the tool set, but you won't find the large, plastic hammer. Or if you do, you will think it is the hammer for the weird whack-a-mole game and you'll keep looking.

When this happens, Rhys isn't getting as much from his toys as the provider hoped. Think of all the times you see an aunt and they say "oh does Rhysie like that drum set I got him?" and you immediately realize that that little bin that you’ve been ignoring because you thought it was missing its top was actually an upside down drum. In the same way, when I ask an stakeholder “are you using that new data I got you?” and they mumble about not having time to find it or sort through it, I know I messed up. The toy went to the toy bin after the first query.

When we give the stakeholders just a data lake, they may not find what they are looking for, or they may find part of it, or they may get hurt by finding and reporting the wrong metrics. Having lots of data seems great, but if people can't use it correctly, does it really have any benefits?

So we need to divide the toys. If we want to decrease the size of the toy bin by two, the easiest thing to do would be to separate Winnie's and Rhysie's toys. This makes the content a little bit more manageable, increasing the chances that you will find what you are looking for without getting hurt. But Winnie always wants what her older brother has, and Rhys hates when Winnie has something he doesn't. And worst of all, this prevents them from playing together.

In this case, Rhys and Winnie are their own departments; Rhysie is the marketing department and Winnie is the accounting department. When we split the data into an accounting data mart and a marketing data mart, things might seem okay at first. But what happens when the marketing department learns that accounting has spend data that they don't have? What happens when accounting learns that marketing has customer data that they don't have? If the data is available, there is no reason why we should keep it from any department. 

But how do we ensure equality while keeping our toy boxes separate? We can go out and buy two of each toy that they want, but this can get expensive. Plus, what if we mess up and buy the wrong one? Have you ever given a child the wrong toy? 

We don't want to write the same tables to two data marts just like we don't want to buy two times the toys. It is expensive in engineering time and overhead; want to make a change to the table? have fun making it in two places. It also creates two times the chance for errors or inconsistencies. If marketing wants to update the definition of customer conversion but you don't change it for the accounting data mart, you won't have a fun time explaining why people reported two different conversion metrics. 

We want Rhys and Winnie to get the most out of their toys, but we don't want to spend more money than we need to or to disappoint them, and we also don't want a chaotic toy box. We want all stakeholders who can use a dataset to have access to it, but we don't want to increase overhead, and we also don't want to have a data lake that nobody can use. So what do we do? We set up the toys by category. The fake tool box goes in one corner with all the pieces; the weird whack-a-mole game goes in another corner with its hammer. Rhys or Winnie can play together or separately with any toy collection in its own place.

This takes longer to set up all the stations than just one bin per child, but it allows for sharing and decreases the overhead of changing toys. Also, it helps everyone get the most out of the toys because all their parts are together. In the case of a universal toy, like a barbie that can play in a doll house or in a G.I. Jeep, we may have a combined pile, but this will be smaller than the original toy bin and have a strict purpose. An example of a data set that may be in the universal data mart is a calendar table; all departments need to have date information.

Citation note:
This is analogy is what I mean when I say “data marts should be by subject area not by department.” A lot of this is based on my own experience, but that experience began by reading _The Data Warehouse Toolkit_. Kimball refers to the departmental approach (Rhys and Winnie having separate bins) as Independent Data Mart Architecture, writing that “multiple uncoordinated extracts from the same operational sources and redundant storage of analytic data are inefficient and wasteful in the long run. Without any enterprise perspective, this independent approach results in myriad standalone point solutions that perpetuate incompatible views of the organization’s performance, resulting in unnecessary organizational debate and reconciliation. We strongly discourage the independent data mart approach” (27). It’s safe to say I agree, and Winnie and Rhys would too.

