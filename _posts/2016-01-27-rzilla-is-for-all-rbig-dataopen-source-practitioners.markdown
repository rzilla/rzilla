---
layout: post
title: "Rzilla is for all R, big data and open source practitioners"
date: "2016-01-27"
---

Hello everyone! [I](http://piccolboni.info) am happy to announce the creation of Rzilla, a resource for R users of big data and open source software. As of today, Rzilla includes:

- a [forum](https://groups.google.com/forum/#!forum/rzilla)
- a [github organization](https://github.con/rzilla) with a number of useful repos
- a [package archive](https://archive.rzilla.org) to facilitate the distribution of R packages.

The packages currently hosted are [`rmr2`](http://github.com/rzilla/rmr2) (mapreduce in R), [`dplyr.spark.hive`](http://github.com/rzilla/dplyr.spark.hive) (`dplyr` backend for Spark and Hive) and [`quickcheck`](http://github.com/rzilla/quickcheck) (randomized testing). Such selection is simple to explain. `rmr2` is by far the runaway success among the RHadoop packages with more than 100K downloads in a single release and hasn't received any loving commits since March 2015. I also know it in detail as I have written most of the code. `dplyr.spark.hive` is under active development and combines the simplicity of `dplyr` with the power of two big-data backends, Hive and Spark SQL. `quickcheck` is needed for running tests for the other two, but it is a general testing package for any R developer to use. Rzilla is work in progress. Your participation and help is needed!
