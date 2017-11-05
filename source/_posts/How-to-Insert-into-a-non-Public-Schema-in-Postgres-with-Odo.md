---
title: How to Insert into a non-Public Schema in Postgres with Odo
comments: true
tags:
  - Python
  - Odo
date: 2017-11-04 21:28:14
---


[Odo](http://odo.pydata.org/en/latest/)([the github repository of Odo](https://github.com/blaze/odo)) is a great tool for moving data from one place/format to another. But when I tried to transfer data in the csv format into a table in a non-public schema in a PostgreSQL database, it took some time to find out how to do it with odo.

At first I could not load data even to a table in the public schema since it uses `COPY :tbl FROM :path` (see [this line](https://github.com/blaze/odo/blob/0.5.0/odo/backends/sql_csv.py#L157)).
Fortunately loading csv files into PostgreSQL is available on master branch or v0.5.1 branch, so it's resolved by specifying these branches at installation.

Another problem was it's unclear how to specify the name of the schema. When I looked at [the source code of odo CLI](https://github.com/blaze/odo/blob/0.5.1/bin/odo), it turned out it's very straight-forward with it:

```shell
$ odo path/to/data.csv postgres://user:pass@host/dbname::table_name --schema schema_name
```
As you can see from the source, you can specify any keyword arguments that are accepted by `odo` function.
