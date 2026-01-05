---
title: "Understanding Data Lakehouse: Here's the Progress"
description: "New year, new buzzword"
date: "January 06 2026"
---
Gonna update this article as these concepts start to make sense.

<img width="100%" src="https://substackcdn.com/image/fetch/$s_!0gXh!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4d1d2f29-23b3-4b89-8050-6a1895ec3b27_666x375.jpeg"/>

This article was referenced by main paper *Lakehouse: A New Generation of Open Platforms that Unify Data Warehousing and Advanced Analytics* and the youtube video below, enjoy.

<iframe width="100%" height="500" src="https://www.cidrdb.org/cidr2021/papers/cidr2021_paper17.pdf"/>

## 1980s: Data Warehouse was introduced

In the 1980s, data warehouses were introduced. This was the era when databases started to separate transactional data from analytical data. Data was split into "fact" and "dimension" views to better serve analytics needs. The process was relatively straightforward: operational data known as OLTP (Online Transaction Processing), which is the database used by regular apps or services would undergo ETL (Extract, Transform, Load) to aggregate and move the data into a data warehouse.

This made it look like there were two separate databases: one for transactions and one for analytics. Data warehouses offered rich management features and high-performance SQL analytics, including support for schemas, indexes, and transactions, which worked very well for structured data at the time.

## 2010's New Problems for Data Warehouse
Data warehouses are great for tables and SQL, but they struggle to handle semi-structured or unstructured data, like files, timeseries, images, and documents. Nowadays, a lot of data comes in these formats. Data warehouses also become very expensive when storing large datasets, mainly because they're designed for tabular data. On top of that, they don’t offer much support for data science or machine learning experiments, since the stored data isn’t very rich or flexible.

## Here come Data Lakes, to solve the problem?
Alongside data warehouses, a new paradigm emerged called the data lake, a low-cost data storage solution designed to hold all types of raw data using file APIs like Amazon S3 (Cloud) and HDFS Hadoop (Local). You can store any kind of data in a data lake, and it’s accessible from anywhere. Data lakes are often integrated with open file formats like Apache Parquet, making the data directly accessible for machine learning and data science engines.

## Wait, does it really solve the problem? 
Not really, the problem with today’s architecture, while storage is cheap and easy, it creates a two-tier system that’s much more complex to implement and manage. Here are some of the trade-offs:
1. Data reliability suffers because multiple storage systems use different semantics and SQL engines.
2. There’s extra ETL required before data becomes available in the data warehouse, which means more processes to double-check and potentially higher costs due to continuous ETL and duplicated storage.
3. How to track file in lake?


## 2021, Lakehouse systems were introduced
Lake + House, you didnt hear it wrong!
basically, a lakehouse combines data lake and data warehouse architectures into a single unified system. The first layer consists of various data sources (structured, unstructured, and semi-structured), which are then pipelined into data lake storage.


## State-of-the-art performance 

<iframe width="100%" height="315" src="https://www.youtube.com/embed/RU2dXoVU8hY?si=FHdPLymazCk9rWzc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
