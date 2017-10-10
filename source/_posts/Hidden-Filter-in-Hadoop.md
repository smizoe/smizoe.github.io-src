---
title: Hidden Filter in Hadoop
tags:
  - Hadoop
  - MapReduce
date: 2016-03-28 22:16:02
comments: true
---


When I was testing a mapreduce job written in [Apache Crunch](https://crunch.apache.org), I encountered a strange error. Although a file exists, the job won't find the file; i.e., although it appears in the result of FileSystem#listStatus, an exception is thrown saying 'Input path does not exist' when I pass the file name to crunch's Pipeline#readTextFile.

After about one hour of struggle, I finally found the cause: if a file name starts with _ (underscore), FileInputFormat silently skips the file. FileInputFormat has a private field named [hidden\_filter](https://github.com/apache/hadoop/blob/release-2.7.1/hadoop-mapreduce-project/hadoop-mapreduce-client/hadoop-mapreduce-client-core/src/main/java/org/apache/hadoop/mapreduce/lib/input/FileInputFormat.java#L90) and this is automatically applied when input paths of a job are read.

I hope this saves someone's time...
