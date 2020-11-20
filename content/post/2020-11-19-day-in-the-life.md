---
title: "A Day in the Life of a Data Engineer"
date: '2020-11-19 00:12:11 -0400'
categories: SQL
---

The point of this post isn't because I think I'm so awesome that you will all want to know my 1-step smoothie recipe and workout regimen (the 1 step is "add fruit that is near you to a blender and hope it tastes edible" and the workout regime is a strict couch-to-fridge cardio plan); the point is to help people who work with data engineers and the salespeople who market to data engineers understand just what exactly we are looking for in business requirements, project communication, and new products. 


The day of a life of a data engineer can be split into three parts (this is knowledge I've amassed over several companies, not specific to one gig):
1. Handling fires and other errors
2. Heads down infrastructure and ETL work
3. Future project or Agile planning

## Part 1. Wake up! There are errors.

![Wake up](https://www.alisa-in.tech/images/wakeup.png)

I don't want to imply that my calendar reflects the three parts with strict start and end times, but in general, part 1 takes place in the morning. ETL tends to run overnight or in the morning, so there is generally information about the run available when you wake up. Additionally, even if your ETL runs during the day, upstream data often populates overnight and your first morning run can be the most perilous.  Depending on your testing suite, these are the four main groups of errors you could wake up:
* Your orchestration tool or a specific DAG indicate an error occurred
* A dataset has not arrived within its SLA (Service Level Agreement)
* Automated tests or stakeholders indicate a dataset is not what is expected
* Somebody wrote in a Slack channel that dogs are better than cats


This error handling often results in a two-part working morning that you may see in data engineers; signing on to handle some errors or questions, taking a break for breakfast, commuting or other morning activities that were pushed back, and then logging back on to "start" the day. When people make comments about engineers starting their days late, it may not always be simply that they slept in- our day sometimes consists of two phases. Sometimes I do sleep in though. 

To be clear, errors aren't an expected or accepted part of life, but they happen. They happen because code is imperfect, humans are imperfect, and upstream data is definitely imperfect. Data changes, data grows, and data *never* shrinks; people always want more and faster data. For all of these reasons, as we start new projects, we often get errors or warnings to help us adjust the code and infrastructure. Any error we get should be evaluated for a permanent fix (see part 2). 


![Slacking](https://www.alisa-in.tech/images/slacking.png)
*Hey, no Slacking! Kidding, we do a lot of Slacking.*

A typical error may be the `cat_data` has not seen new data in 24 hours but its SLA is 24 hours.  Oh dear.

There are a lot of reasons we may not have received cat_data, and it's up to us to find which it is:
1. There is no new cat data from upstream
2. There is new cat data from upstream, but it is not successfully getting to our database
3. There is new cat data in our database, but our alerting system does not recognize it

![where_did_cat_data_go_wrong](https://www.alisa-in.tech/images/where_did_cat_data_go_wrong.png)
Each of the three places where cat_data reality could be mismatched from expectations

We have to find the error, fix it, and communicate an adjusted SLA to the stakeholders. We also want to make note of the cause of the error so we can implement a long term fix (see part 2).

## Part 2. Heads down time! Write some code!
![Not Funny](https://www.alisa-in.tech/images/not_funny.png)
*My actual office (not kidding)*

When working on big projects or coding, four 15 minute increments is not equal to an hour. I need chunks of time in which I get really into the weeds. For that reason, I block off 2-3 hours on my calendar every day to do heads down work. 

Typically my "actual" job (meeting and emergencies are my actual job too, sadly) consists of:
#### Creating data pipelines from scratch or amending existing ones

This includes hitting an API and writing a dataset with Python to S3 and then making tables in Snowflake with SQL, putting all of this into an Airflow DAG. It can also be simpler, such as making a new table from existing data that helps analysts more easily do analytics or reporting

#### Improving our current infrastructure

We want our infrastructure whether open source, purchased, or internally built to be reliable, scalable, and flexible. <br>
In part 1 we discussed errors; errors are a great indicator that the infrastructure needs a tweak.
How can we improve our infrastructure based on each of the reasons why cat_data could be late? If the problem is #1 - there is no new cat data from upstream, then one solution is to make the testing suite smart enough to not alert if there is no upstream data (and maybe alert stakeholders of this!)

#### Administrative work
Data engineers are often also responsible for administrative duties such as monitoring costs and access. It can't all be pizzazz and glitter, people. 


## Part 3. Ah, meetings. 
![cat_meeting2](https://www.alisa-in.tech/images/cat_meeting2.png)
This is about as close as it gets to a cat meeting

I'm a big believer in the Agile process. My definition of Agile is continuously delivering work and receiving incremental feedback in order to produce the best product possible. This means a lot of meetings and requirements, but it's worth it. The meetings that I think are most important are:
* Daily stand up - Ensure everyone knows what they're doing for the day, ensure there are no overlaps, and help each other with any blockers
* Retro - Reflect how the last sprint could have been better. This allows us to incrementally improve our processes 
* Sprint planning - Make sure the team is all on the same page about the upcoming work and how it relates to our team and company priorities; ensure we all commit to the completing the work next sprint
* Grooming - Ensure all our work tickets meet whatever requirements criteria we agree on, estimate if we are assigning points and keeping track of velocity, and prioritize each ticket.

![grooming](https://www.alisa-in.tech/images/grooming.png)

That's it! Then we go to bed and do it all again tomorrow.
