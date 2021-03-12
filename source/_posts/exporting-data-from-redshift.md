---
title: exporting data from redshift
date: 2017-01-31 22:05:33
updated: 2017-01-31 22:05:33
tags:
- redshift
- psql
- postgres
- data export
---
I work with a data warehouse in Redshift. I recently needed to export a large set of data out. I started testing querires with smaller result sets using DataGrip's "Export to file..." option. When I moved to running the full queries with larger result sets, my connection to Redshift was timing out before the result set came in.

Redshift has a documented way to export large data sets in the [UNLOAD statement](http://docs.aws.amazon.com/redshift/latest/dg/r_UNLOAD.html). UNLOAD will only export data to S3 buckets though, and I didn't have an S3 bucket easily available.

Searching for solutions brought up some results describing exporting data using postgres's psql tool directly. In the end, I was able to export the data this way, but it took some experimentation to get it in the format I was looking for.

The steps I ended up taking were:
1. Change psql to output in "unaligned" format:
  ```
  \a
  ```
  This changes output from the ASCII-art default format to a comma-separated value format. Or really, separated by whatever character you like. I needed tabs.
2. Change psql to use tab as the delimiter:
  ```
  \f '\t'
  ```
  That's the exact string you need to use. I first tried to pass a literal tab character using the `Control-V, Tab` pattern, but psql interprets the tab literal as whitespace. Quoting the tab changes the delimiter in to the full string of a tab surrounded by two quotes -- not what I was hoping for.
3. Change the output to target a file:
  ```
  \o output_filename.tsv
  ```
  After this `psql` outputs query results to a file with the given name in the current directory. It clears the file before outputting. A simple `\o` without a filename later will change output back to standard output.
4. Run the query as normal:
  ```
  select * ...
  ```
  The first line of the output file is a header row, followed by all the data rows with the specified delimiter, and finally a line reporting how many rows were selected.

In my case, I then tar and bzip2'd the results and was able to `scp` them back to my laptop without trouble.
