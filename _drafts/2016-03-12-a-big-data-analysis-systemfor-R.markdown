---
layout: post
title: "A big data analysis system for R"
date: "2016-03-12 15:28:52"
author: Antonio Piccolboni
---

I have recently [announced](http://workstream.piccolboni.info/post/140765564714/tidyr-goes-big) the availability of [`tidyr.big`](https://github.com/rzilla/tidyr.big), a big data backend for package  [`tidyr`](https://github.com/hadley/tidyr). With this post, I would like to put this development into a larger context.

## The Python way

Some time ago I read the announcement of the [Ibis project](http://blog.cloudera.com/blog/2015/07/ibis-on-impala-python-at-scale-for-data-science/),
a big data analysis system for Python, running on [`Impala`](http://impala.io/). If you filter out the marketing-heavy preamble, there's a lot to agree with in that vision, which I understand as follows:

  1. 100% Python experience --- don't mix and match languages.<sup>[1](#fn1)</sup>
  2. Full fidelity data --- no sampling needed.<sup>[2](#fn2)</sup>
  3. Access to existing Python libraries --- new distributed algorithms can be based on sequential ones.<sup>[3](#fn3)</sup>
  4. At big data scale.
  5. At interactive speed --- high throughput and low latency, pick two.

To achieve this vision, several elements are needed, according to the Ibis leaders:

  1. Enable a set of operations with the familiar syntax and semantics of [`pandas`](http://pandas.pydata.org) (data frames for Python) to execute directly on Impala.
  2. Rich type system, including nested types.
  3. Enable users to define functions in Python and execute them on a cluster.
  4. Accelerate Python performance with code generation and columnar formats.

Only the first of these four elements is available in the current release, but the others are planned. With Cloudera's deep pockets and Wes McKinney's leadership, this ambitious vision is realistic.

## The R way

Python users get all the good stuff, don't they? How about R? But the situation is not so dire. Some elements of an Ibis-like system exist already, but they haven't been fully assembled into an integrated data analysis system. Maybe that is what we should make happen.

Let us use [Hive](http://hive.apache.org) or [Spark](http://spark.apache.org) as backends. Hive is used more widely but Spark is growing very quickly and delivers a better interactive experience. This does not imply a negative opinion on Impala, it is just a different starting point. Eventually, I believe, R needs to interface with all the most important big data systems. Likewise, additional backends are also envisioned for Ibis.
The package [`dplyr`](https://github.com/hadley/dplyr) with the backends implemented in [`dplyr.spark.hive`](https://github.com/rzilla/dplyr.spark.hive) allows a similar class of operations as the ones defined by `pandas` to be performed efficiently and scalably on big data.
The package [`tidyr`](https://github.com/hadley/tidyr) defines a complementary set of operations to `dplyr`, also available in `pandas`, but, until recently, it did not have any backends. `tidyr` only operated in memory, on a single thread and a single machine. The package [`tidyr.big`](https://github.com/rzilla/tidyr.big) changes this by implementing most `tidyr` functions on top of Hive or SparkSQL, with the help of a couple of extensions from the [brickhouse project](https://github.com/klout/brickhouse). To the best of my understanding, these four packages, coupled with Hive or Spark,  provide functionality roughly equivalent to the **first element** in the Ibis roadmap. They also implement Hadley Wickham's "[solid pipeline for data analysis](https://github.com/hadley/tidyr)".

The **second element**, a rich type system including nested types, is a bit more far off. R has a list type that allows arbitrary nesting but lacks high performance tools to work on them, save for simple operations like flattening. Hive and SparkSQL support structs, arrays and maps, but not nested types --- unless JSON columns count. There's some initial support for all of them in `dplyr.spark.hive` and `tidyr.big`, but it is just a start.

The **third element**, distributed execution of R functions, is not pie-in-the-sky as it seems. Both Hive and SparkSQL offer the ability to load jar files dynamically and to define functions, known as UDFs, UDAFs and UDTFs, based on classes in those jars. Moreover, there is an alternative R interpreter, [renjin](http://www.renjin.org/),  written completely in java, that can be loaded into both backends. One can thus create a UDF that reads a user-defined R function, executes it on data provided by the backend and returns the results to it. We already have proof of concept that this works, but some obstacles remain. Sure, renjin compatibility with existing packages is incomplete, but it is [improving](http://www.renjin.org/blog/2016-01-31-introducing-gcc-bridge.html) and hundreds of useful functions are part of the core language and base packages anyways. One can write a lot of UDFs with what is already available, in addition to what is built into Spark or Hive (see [this document](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF)  but please note that, confusingly, the document refers to built-in functions as UDFs). Moreover, being able to distribute R code as a jar pretty much eliminates the deployment hurdles that users face with packages [rmr2](http://github.com/rzilla/rmr2)  or [SparkR](https://spark.apache.org/docs/1.6.0/sparkr.html) and, in general, when mixing bytecode and binary. The goal is to make using R against big data a seamless option for the data scientist, with little or no intervention needed from administrators.

The **fourth element**, high performance for R code through code generation and use of columnar formats, is at least in part a pleasant side effect of adopting renjin, which is already [performance oriented](http://www.renjin.org/blog/2013-07-30-deep-dive-vector-pipeliner.html). Of course, one thing is running circles around GNU R and another is achieving hardware speed, but it's a start. Because R would execute within the same JVM as the Spark or Hive process it interacts with, there is no need to talk about columnar formats because no serialization is needed within the same process. Rather, it is a matter of [optimizing](https://issues.apache.org/jira/browse/SPARK-12635) the API to require fewer or no copies and to be vector-oriented (columnar memory layout).
There is some work to do here and some uncertainty as to the time of delivery for these optimizations, but apparently there are no insurmountable technical challenges.

## A language centric point-of-view
While there are many R/big data integration [packages](https://gist.github.com/piccolbo/3d8ac40291f4eaee644b), and many of them are useful, my personal view is that they solve a different problem, such as: how do we make VerticaDB or Spark available to as many users as possible? My point of view is the R equivalent of the one taken by Ibis. As an individual or organization already committed to R, how can I work best with big datasets? I think the language-centric point of view is important. People gravitate towards their favorite language because achieving high proficiency in a language is [hard](http://norvig.com/21-days.html). Juggling multiple languages in the same project leads to code duplication and debugging hell. Communities form around languages with their own subcultures. While I understand no language is the best and each have different strengths, I advocate using different languages for loosely-coupled components that can be debugged independently. Of course, if you are writing a high performance extension for R you may need to mix R and C++ in the same program and, at run-time, process, but that is best left to developers and is not something a data scientist should tackle on a regular basis.

## What next
Where do we go from here? I guess there is more work to do on all dimensions, starting with a good, memorable name and a simplified installation procedure. We need early adopters who can give beta software a spin and provide feedback. This project doesn't have a big corporate backer, so it will have to be a community effort, or not be at all. Come join us on the [forum](https://groups.google.com/forum/#!forum/rzilla) or [github](https://github.com/rzilla) or feel free to send an [email](mailto:rzilla@piccolboni.info) with your suggestions and what you would like to do to help.


<hr>
<small>
<sup><a name="fn1">1</a></sup> If you ever worked and namely debugged at a cross-language interface, you know the pain. <br>
<sup><a name="fn2">2</a></sup> I've heard the argument that sampling is all we need at least a million times. It works for means, but if you are trying to do [speech recognition](http://googleresearch.blogspot.com/2015/08/the-neural-networks-behind-google-voice.html) or beat a 9-[dan](https://en.wikipedia.org/wiki/Dan_(rank))[go player](http://googleasiapacific.blogspot.co.uk/2016/03/alphagos-ultimate-challenge.html), you need [more data](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/35179.pdf) not less.<br>
<sup><a name="fn3">3</a></sup> For example, combine clusterings of subsets of the data into a better clustering, or likewise for quantiles.<br>
