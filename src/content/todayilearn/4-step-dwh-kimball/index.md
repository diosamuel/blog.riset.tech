---
title: "Four-Step dimensional modelling design process"
description: "4-Step as DWH methods for data modelling"
date: "February 19 2026"
---


> Source: The Data Warehouse Toolkit: Third Edition

# I assume you already understand how data warehouse works
---

The first Four-Step dimensional data modelling was introduced by Ralph Kimball and Margy Ross in book called **The Data Warehouse Toolkit** (you can check in this [The Datawarehouse Toolkit: Third Edition](https://www.oreilly.com/library/view/the-data-warehouse/9781118530801/9781118530801toc.xhtml)).

Its a great resource to help you understand about how data was modelled and teach you the high concept about how the data warehouse works in several industries.

What i love this book, it doesnt teach you about coding and tools, you can jump the tools but you cant hop the fundamental.

There was 4 Step as the title of this content to design of a dimensional model, include:

## First step: select the business process

Business process are the operational activites performed by the organization, some of the business process are:

1. Taking an order
2. Sign up a new member
3. Analyzing some consumer behaviour
4. Anything you do in your companies

These business process will generate some of the event that will be translated into facts in a fact table. Most fact table focus on the result of a single business process or event, that we call it 'grain' later

Choosing correct grain will help you make correct structured dimensions table later.

# Two step: Grain

The grain itself, as we discussed before, is the pivotal step in dimensional design.

The grain must be declared before choosing dimensions or facts, because every candidate of dimensions or fact that we will form in the next step must be consistent with the grain.

Imagine you have a transaction table (OLTP) consist of 1000 rows that each rows tell you when exactly the time your consumer bought your *Fruits*, each row correspond into a another table called *Fruit* that tell you what the fruit information. The grain must be the transaction of buy and the metadata of the fruit itself.

Focus on atomic data, yeah you can use aggregate data and store in somewhere table can be accessed easly, read more: [Aggregate Fact Table: Cube](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/aggregate-fact-table-cube/)

<img src="https://www.kimballgroup.com/wp-content/uploads/1999/12/The-Matrix-Fig-111.png"/>

# Third Step: 5W+1H are answered in Dimension Table

Dimension provide the "who, what, where, when, why and how" context in the business process event. Dimension table called the "soul" in the DWH as they are hold of more lot information that already drill down and more spesific

Example of Dimenstion Table:

```
+-------------------+----------------+
| Column Name       | Type           |
+-------------------+----------------+
| project_key       | BIGINT (PK)    |
| project_id        | VARCHAR        |
| project_name      | VARCHAR        |
| project_type      | VARCHAR        |
| faculty_name      | VARCHAR        |
| start_date        | DATE           |
| end_date          | DATE           |
| status            | VARCHAR        |
| created_timestamp | TIMESTAMP      |
+-------------------+----------------+
```

it explain what does project_key fact table do.

# Last step: Fact Table, Baby

A fact table contains the numeric measures produced by an operational measurement event in the real world.

The fundamental design of fact table are focused on physical activity and the grain that we choose before. The candidate of fact key table are will be joined into dimensional table, so we must carefully choose fact table and dimensional table to achieve the best analytical result that we can query on the table.

example of fact table:

```
+--------------------------+-------------+
| Column Name              | Type        |
+--------------------------+-------------+
| project_key              | BIGINT (FK) |
| date_key                 | INT (FK)    |
| total_budget_project     | DECIMAL     |
| student_count            | INT         |
+--------------------------+-------------+
```

project_key and date_key will be joined into the dimensional table called project and date.


The fact table and dimensional table are in separated table is for help drill-down for data for the highest analysis.

Read this for comprehensive material [The Datawarehouse Toolkit: Third Edition](https://www.oreilly.com/library/view/the-data-warehouse/9781118530801/9781118530801toc.xhtml).