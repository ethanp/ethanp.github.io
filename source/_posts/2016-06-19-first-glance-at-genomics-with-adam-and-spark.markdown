---
layout: post
title: "First glance at Genomics with ADAM and Spark"
date: 2016-06-19 14:15:22 -0700
comments: true
categories: [genomics, adam, spark, avro, parquet, aws, s3, science, dna sequencing]
---

At work, we have a Spark cluster. One of my first responsibilities was to make
it more reliable and efficient. So I looked on Github to see how people
actually use Spark, and what magic they use to get their clusters not to crash
in the process. This is how I found [ADAM][], "a genomics analysis platform
built using Avro, Spark, and Parquet". Then I looked in the repo's
_contributors_ list, and watched a few lectures by Frank Nothaft and Matt
Massie, two of the project's main contributors. What I heard there was pretty
cool.

In short, they're looking to build systems that will "one day" recommend more
effective treatments for diseases including cancer and Alzheimer's within an
hour of receiving a patient's DNA sample. They describe several components of
what needs to be done to make [research toward] this possible.

<!-- more -->

### ADAM according to YouTube

See the __references__ at the bottom of this post.

They didn't explain much of the actual genomics, though what they did explain
of that, they did with laudable clarity. In essence, the first step is to
assemble ("sequence") a large number of small strips of DNA into a single
massive (250 GB) chain (per sample per person), even in the presence of small
errors in the small strips. Ok now the the second step is to take the data from
multiple samples from multiple people, and find anomalies that correlate
historically with the likelihood to end up being diagnosed with a particular
disease. These anomalies may be on the scale of a few single-character changes
scattered around the DNA, again in the presence of mistakes in the
transcription of the DNA. So statistical techniques are used (from what I
remember, Poisson random sampling methods and binomial Bayesian methods, and
Chi-squared independence tests) to evaluate the confidence in the correlations.

According to the lecturers, currently, techniques for doing this sort of
computation and analysis relies on code written by PhD students in genomics,
who are not so interested in writing high quality code, as learning about
genomics and finishing their dissertations. Therefore, there are many
(compressed) file formats, and processing subsystems written in every
programming language you can think of. These subsystems are assembled together
by piping data through the filesystem between each compoenent. Many of these
subsystems are inherent unscalable.

All these are issues the researchers at the _Big Data Genomics_ project are
trying to solve using __ADAM__. They [reported in _SIGMOD 2015_][sigmod] to
have achieved a 28x speedup and 63% cost reduction over current genomics
pipelines. The _Big Data Genomics_ project is a collaboration between
researchers at the AMPLab at UC Berkeley, and other genetics and cloud-
computing research institutions and companies. They note that their ADAM
pipeline infrustructure is able to accommodate analyses from other areas of
scientific research as well, including astronomy and neuroscience.

Producing a high-quality human genome currently takes 120 hours using a
"single, beefy node". Their improvements involve using map-reduce (via Spark),
and columnar storage (via Avro & Parquet) to distribute the workload across a
scalable cluster, so their 28x speedup is probably something people are happy
about.

A major issue faced by genomics researchers is the proprietary nature of the
data. This means that researchers must collect and process their own data,
which is heavily human and computer labor intensive. The humongous (I mean
really...) nature of the datasets means that the only practical means of
transferring them is by shipping boxes of hard drives. As data collection gets
easier, and the amount of data available explodes, even shipping boxes will
become impractical. So the vision is to keep the data in the Simple Storage
Service (S3) hosted by Amazon Web Services (AWS) so that it can be operated on
via Spark without being copied into an institution's private data center. This
is currently more dream than reality, but seems like the logical step because
of how Big the Data is.

Another major issue faced by genomics researchers is the large number of (open
and proprietary) genomics data formats, and the quirks/bugs in their
implementations. The ADAM team's solution to this is their large, fixed schema,
created using Apache Avro. This schema is designed to be able to accommodate
whatever genetics research you may want to do. To allow ADAM to ingest your
format, you write a transformation from your format to their standard Avro
schema. Then all the applications built-in to ADAM for analyzing the data are
available to you.


[ADAM]: https://github.com/bigdatagenomics/adam/
[sigmod]: https://amplab.cs.berkeley.edu/wp-content/uploads/2015/03/adam.pdf

### ADAM, according to the paper

The software architecture is (explicitly) based to a large extent on the
Internet's OSI model. It also has 7 layers, starting with a few storage layers,
going to a schema layer, going to compute and transform layers, then to an
application layer. The point is, like the OSI model, to make it easier to
implement on top of existing components, to make it portable in the important
dimensions across scientific disciplines and execution environments; and also
to allow the implementations of each layer to be swapped out over time as the
hardware, compute software, and relevant scientific algorithm ecosystems
evolve.

They proceed to discuss several pipelines (one genomics, the other astronomy)
that they implemented (mainly) around the needs of Spark and their AWS
(virtual) hardware. They note that their re-implementation provides several
bug-fixes with respect to algorithm implementations in reference genetics
software components. Plus, each of the reference components can only
communicate through disk I/O, while the Spark data pipeline keeps data in
memory until the end. This is reminiscent of the original insight of Spark:
speedup over Hadoop MapReduce by keeping as much data in memory as possible
throughout the job.

Datasets in genomics, astronomy, and many other scientific fields involve a
coordinate system. In genomics, it is generally a 1-dimensional string of
_nucleotides_ (A, C, G, T), which serves as a convenient abstraction over what
is really a collection of molecules, each containing a packaged collection of
DNA polymers in a complex 3D shape, crammed into the nucleus of a cell (I
think, pg. 8). There is an assumed independence of data between distant regions
of the 1D string.


#### The Region Join Algorithm

To figure out which regions of the 1D space were acquired by this sample from
this "non-reference" human being, it is matched up with a "reference" (a.k.a.
"idealized") human genome, generated by aggregating many people together. What
they use Spark for is to figure out which regions of the reference genome were
sequenced in _this_ real DNA sample. They call it "convex hull" (see below for
definition), because to generalize to multi-dimensional spaces that's the right
way to see it, but for 1-Dimension, you're really just looking to line-up sub-
lines along a big line, and that would have been a simpler way to explain it if
I'm not mistaken.

This "region join" can be implemented by (a) shipping the reference genome to
each node, and having them output which part was matched for each input data-
strip, or (b) repartitioning and sorting the datasets in such a way that puts
the joinable data from each dataset on the same machine in sorted order (which
I believe is part of the Spark API), and then iterating through both, and
"zipping" them together.

The output of that stage is a powerful primitive for higher-level algorithms
that researchers will want to run.


#### Storage

Part of the Hadoop concept is that you get data locality because MapReduce (or
Spark) schedules your computations on the HDFS nodes that already happen to be
holding the relevant data. However, this conflicts with their goal of being
able to store like 100s of _petabytes_ of data. They can't really maintain the
cost of having enough (even virtual) machines up and storing their giant
datasets. So they opted to store the data in Amazon S3 (distributed storage)
"buckets". This increases job start (and sometimes finish) duration, but lowers
costs.

This reminds me of how the default method for using the "Databricks cloud
platform" assumes that you are storing your dataset in S3 buckets, and the data
will be loaded (remotely) into the relevant EMR (virtual machine) nodes when
the user of an interactive Spark session (or scheduled script) asks for them.

They store the data as in Parquet files. They wrote their own Parquet "row
chunk" index file, that Spark then uses to figure out how to intelligently
paralellize file [system block] reads, and not read (too much) more of the
files than is really necessary. This is done with the help of a query predicate
pushdown mechanism. If I'm not mistaken, all this stuff is really cool, but now
it's built-in to Spark with the help of the Catalyst optimizer, which I think
came out after this paper did.


#### Cost and Performance

In their experiments, ADAM is way faster and cheaper than existing
implementations of each of its components in almost all genomic situations.
Taking the whole pipeline as a single system, then it's always way cheaper and
faster. This is also true for the astronomy dataset and task. They achieve near
linear performance scaling (i.e. when adding more machines to the cluster) in
almost all components.

### Diversion: Convex Hull

* __Convex set/shape__ -- for any two points in the shape, the line connecting
  them is also part of the shape
* __Convex hull__ -- the smallest convex set that contains a given subset of
  points in the space
* __Convex hull in 1-Dimension__ 
    * given a set of lines, the smallest range of the line that contains the
      start and end of every given line (right?)
    * given a set of points, just find the line connecting the smallest and
      largest point (right?)



### References

* Nothaft, Frank Austin, et al. ["Rethinking data-intensive science using scalable analytics systems." Proceedings of the 2015 ACM SIGMOD International Conference on Management of Data. ACM, 2015.][gsadam]
* Youtube: [Next-Generation Genomics Analysis Using Spark and ADAM- Timothy Danford (AMPLab, UC Berkeley)][ngn]
* Youtube: [ADAM: Fast, Scalable Genomic Analysis - Frank Austin Nothaft (UC Berkeley)][advid]
* Youtube: [sfspark.org: Frank Nothaft, Scalable Genome Analysis With ADAM ][sfsa]
* [Github project][githubproj]

[sfsa]: https://www.youtube.com/watch?v=ctLyjYw0BOg
[advid]: https://www.youtube.com/watch?v=dJX-bphMZRs
[ngn]: https://www.youtube.com/watch?v=axLEBM_PZeI
[gsadam]: https://scholar.google.com/scholar?hl=en&q=Next-Generation+Genomics+Analysis+Using+Spark+and+ADAM-+Timothy+Danford+(AMPLab%2C+UC+Berkeley)&btnG=&as_sdt=1%2C44&as_sdtp=
[spkpqt]: https://github.com/apache/spark/blame/master/sql/core/src/main/scala/org/apache/spark/sql/execution/datasources/parquet/ParquetReadSupport.scala
[githubproj]: https://github.com/bigdatagenomics/adam
