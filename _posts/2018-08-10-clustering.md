---
layout: post
title: ELI5 - Clustering
date: '2018-08-10 15:12:11 -0400'
categories: ELI5
---

This Explain Like I’m 5 will be about clustering, a technique used by Snowflake, a cloud based data warehouse, to improve query performance.  As always, I will be explaining this to nephew Rhys (pronounced Reese) In-tech, who is five and a big fan of toys. The notes in parentheses will be for those of us over 5 who want to know the actual terms.

Okay Rhysie, today we’re going to learn about clustering and micro-partitioning. When a word starts with “micro” it means small, like microscope or a microsecond. A microsecond is even faster than a second! Yes, faster than you can say the alphabet. Yes, faster than you can blink. Yes, okay, stop, it’s fast, we’re moving on.
Before we dive into clustering, we have to talk about micro-partitioning first.

Let’s think about a stack of Monopoly money that you’ve put away after a game (_the data_); because of the way Monopoly is played, there should be no ordering to the dollar bills. Before we put it away, let’s divide the bills into groups, but we won’t sort them (_this is a micro-partition; Snowflake does that automatically_).

So our money is in groups- there is some overlapping values (_several groups may have $500s!_) and some groups may not have any $5 bills, but all our money is there. Let’s say we want to count how many $500 bills we have, a useful metric. Even though our data is in groups, we don’t know anything about them, so we have to look at each bill. We have to check 100% of our bills. That would leave very little time for playing or destroying something in the living room.
What if, after we grouped them, we put a sticky note on top, on which we store some information about each group (_metadata; Snowflake stores this automatically about micropartitions_). We may want to know what the range of dollars we have in this group are; does it include $1s, $5s, and $10s? Or just $500s? 

Now, if we want to grab all the $500s all we have to do is look at each sticky note, rather than through each bill, and grab the piles in which $500 is in range (_this is equivalent to filtering, sometimes called a  WHERE caluse_). There’s a chance each group has at least one $500, so we have to count 100% of our bills again, but hopefully there are some groups we can omit entirely by looking at their sticky notes.

If the game of Monopoly were played such that larger bills were used later in the game, our pile may have some organization to it (_natural data clustering_). This will help our little groups have more organization; more of the large bills will be together and the small bills together. With natural clustering, we may have only have to search through a few groups and all the sticky notes.

But what if we could organize the bills (_clustering_); we could manually reorder the bills (even after they’re grouped!) to get more of the same kind of bill together, so we can check fewer groups when we want a $500 bill. Ideally, we could only have to go through one group, and the rest of the sticky notes tell us there are no $500’s. If there are 10 groups, we could go from searching 100% of the bills to 10% of the bills just by adding the sticky notes and groups. There is some work to be done to make the sticky notes, but if we continue to use them it pays off.
Wow.




Here is a <a href="https://docs.snowflake.net/manuals/user-guide/tables-micro-partitions.html#micro-partitions">hyperlink</a>, Steve. 
