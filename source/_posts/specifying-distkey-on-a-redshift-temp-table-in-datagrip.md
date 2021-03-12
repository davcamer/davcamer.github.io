---
title: specifying distkey on a redshift temp table in DataGrip
date: 2017-01-17 22:24:09
updated: 2017-01-17 22:24:09
tags:
- redshift
---
I've been doing some work in our Redshift data warehouse lately. Creating Temp Tables has simplified my queries dramatically. I've been using the DataGrip IDE to connect to Redshift. It has a [minor bug](https://youtrack.jetbrains.com/issue/DBE-1078) affecting `create table as` that I tripped over repeatedly while working out my queries.

If the query includes a `distkey` or `sortkey`, DataGrip won't recognize that the query continues after the `as` if there is nothing further on the line.

But, I was able to quickly work around this by just putting `as` and `select` on the same line.

```
CREATE TABLE table_name DISTKEY (id) SORTKEY (id) AS SELECT
  DISTINCT(t.id) AS id
FROM ...
```
