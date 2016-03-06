---
title: Passing HBase Configurations from Hive Query
tags:
  - Hive
  - HBase
  - MapReduce
date: 2016-03-06 09:40:48
---


In the current company, we employ Hive and HBase for storing and processing data. Although HBase has a good java api to make queries to HBase, Hive's [HBase Integration](https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration) enables us to aggregate information in HBase with simplicity of SQL.

Since Hive converts the query to a mapreduce job, as noted in [HBase's Document](https://hbase.apache.org/book.html#perf.hbase.client.blockcache).
We also would like to set the number of prefetched rows by a scanner (via Scan#setCaching), since it would result in a poor performance if the value is too low.

I first found out [a post on stackoverflow](http://stackoverflow.com/questions/30074734/tuning-hive-queries-that-uses-underlying-hbase-table), but several of the parameters sugggested seems not to be documented on Hive's HBase Integration page. The only way to check what is available seems to search Hive source code on github...

As a result of search, I found 2 classes that gives the answer: [HBaseSerDe](https://github.com/apache/hive/blob/release-1.2.1/hbase-handler/src/java/org/apache/hadoop/hive/hbase/HBaseSerDe.java) and [HiveHBaseInputFormatUtil](https://github.com/apache/hive/blob/release-1.2.1/hbase-handler/src/java/org/apache/hadoop/hive/hbase/HiveHBaseInputFormatUtil.java). According to these classes, 3 parameters are available to us:

  * hbase.scan.cache: a value passed to Scan#setCaching method.
  * hbase.scan.cacheblock: a value passed to Scan#setCacheBlocks method.
  * hbase.scan.batch: a value passed to Scan#setBatch method.

So basically, you would like to set first two parameters, and set the last parameter when the number of columns in HBase is large.
