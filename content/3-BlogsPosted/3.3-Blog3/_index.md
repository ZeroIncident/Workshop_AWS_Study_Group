---
title: "Blog 3"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Optimising Athena: Data Layout Can Matter More Than SQL Changes

Published on 8 November 2026 (Vietnam time). [Original AWS Study Group VN post](https://www.facebook.com/groups/660548818043427/posts/2327599394671686/).

*The experiments in this article are proposed and have no measured results.*

For the same analytical result, how much can storage layout change the amount of data read? This article studies Athena SQL queries billed by scanned data. Other Athena capacity/pricing options exist, so estimates must identify the chosen mode. No comparative benchmark is claimed here. [Athena pricing](https://aws.amazon.com/athena/pricing/)

Two distinct variables are columnar format and partitioning. Parquet allows a query reading selected fields to skip unrelated columns. Its metadata can also help eliminate some data blocks when predicates permit. This supports a hypothesis of improvement over row-oriented text, not a guarantee that every query becomes faster. [Columnar formats](https://docs.aws.amazon.com/athena/latest/ug/columnar-storage.html)

Partitioning narrows the data range. For daily logs, a suitable predicate on the partition key can restrict reads to relevant days. Creating date-based S3 directories alone is insufficient; Athena table configuration and metadata must reflect that organisation. [Partitioning](https://docs.aws.amazon.com/athena/latest/ug/partitions.html)

![Reference pipeline for CloudFront log analytics](/images/aurel/blog-athena.jpg)
*Image: AWS Big Data Blog, a reference pipeline with conversion and partitioning. [Original article](https://aws.amazon.com/blogs/big-data/analyze-your-amazon-cloudfront-access-logs-at-scale/). This older reference illustrates the concept rather than claiming a newly deployed personal architecture.*

A proposed experiment compares identical data in three layouts: unpartitioned CSV, unpartitioned Parquet and date-partitioned Parquet. Contents and types must be equivalent. Use queries selecting a few columns for one day, aggregating a month and reading most data. Record bytes scanned, execution duration and returned results for each case.

Control result reuse, repeat runs and confirm result equivalence before interpreting timing. Otherwise a cached response or a changed dataset can appear to be a layout improvement. Excessively fine partitions and many small files can add metadata and file-open overhead. Partition keys should follow real query patterns. [AWS performance guidance](https://aws.amazon.com/blogs/big-data/top-10-performance-tuning-tips-for-amazon-athena/)

A full comparison includes conversion work, duplicate optimised storage and pipeline maintenance. Use public data for the lab, restrict access to its bucket and manage query output storage. Clean up both intermediate data and query results after retaining the evidence needed for the comparison.

The research goal is to state which query shapes benefit, what trade-offs arise and which measurements support the conclusion. “Parquet is always better” is not an adequate experimental finding.

#AWS #AWSStudyGroup #FCAJ #FirstCloudAIJourney #AmazonAthena #AmazonS3 #DataEngineering #CostOptimization
