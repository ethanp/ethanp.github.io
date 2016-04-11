---
layout: post
title: "Bigtable paper summary"
date: 2016-04-10 22:02:12 -0700
comments: true
categories: [systems, distributed systems, bigtable, google, hadoop, apache, cassandra, hbase]
---

When looking into what Cassandra and HBase are, and their relative strengths
and weaknesses, people often seem to think they can get away with the
following very succinct characterizations: "Cassandra is like is Dynamo plus
Bigtable, and HBase is just Bigtable". I don't know much about Dynamo _or_
BigTable because we skipped those papers in my systems courses. So to get
started understanding what's going on with all this mess, I decided to read
the Bigtable paper. What follows is a brief summarization/retelling of the
[Bigtable paper][btp]. It follows roughly the form of the paper, especially in
that it starts high level, and then digs slightly lower-down into the
implementation.

### What problem are we solving?

Bigtable provides an API for storing and retrieving data.

It is most useful if

* there is a _lot_ of data coming in at a high rate over time
* there is no need to join each data table with another
* data might need to be updated
* range queries are common

Bigtable is a distributed database. It is a database management system which
allows you to define tables, write and update data, and run queries against
the stored data. It is similar to a relational database, except that it is not
"relational" in the sense denoted by the term "relational database". Instead,
it brings its own type of data model, where instead of storing data in normal
two-dimensional table cells, you store it according to a new set of rules that
allows for flexibility in the shape of each record, while still enabling
overall efficiency.

<!-- more -->

### Go on...

In the Bigtable data model, data is grouped in tables. Each record in one
table must have a string identifier unique in that table. Each table is
defined to have a set of "column families". This is where the data goes. A
record has values, that is the point of having a record. Each of those values
is associated with a timestamp when it was created. This can either be defined
by Bigtable or by the user inserting the data. Having two versions of a record
means it was updated. Each value has its own key (a string) pointing to it.
That key has two components, a "family" and a "qualifier". Each table has a
finite set of families with known names. However, the qualifier segment of a
value's key may be an arbitrary string. There may be any number of anchors for
each family in each record. You can hint whether you want Bigtable to prefer
disk locality when writing many values in the same column family.

Rows are stored on disk in alphabetically by row key, and for each value, in
_descending_ order of timestamps (i.e. most recent first). All reads and
writes over a single key are atomic with respect to each other, but not with
respect to changes in other rows. (This is something the client must be
careful about.) All the rows for a given table are chopped up (dynamically)
into "tablets" which each are stored in by exactly one "tablet server" node in
the cluster. One can specify garbage collection to either only keep the last
_n_ versions, or to only keep everything for a specified duration.

When reading the data, one might e.g. just fetch the data for one column
family within a particular range of row names, but get all currently known
versions of that data. This type of query is what was optimized by the
creators of Bigtable. One might for example want to execute this type of query
to feed as input to a MapReduce job. One could do that, and then also be able
to enjoy the intentionally efficient writing of MapReduce results into a
different Bigtable table.

Bigtable data is stored in the SSTable file format. In this format, each file
represents what in Scala would be called a `immutable.Map[String, String]`. A
file is made of blocks, each containing a continuous interval of the keys that
is listed in the file's "block index". Each SSTable file is stored in Google
File System (GFS), which I will not describe in detail here, and is similar to
the Hadoop File System (HDFS).

Bigtable relies on Chubby (similar to Apache Zookeeper) to ensure there is "at
most one active aster at any time" and for bootstrapping.

A Bigtable instance has a single master node, and a potentially dynamic number
of tablet servers. A tablet is a complete alphabetical interval of all the
data in a table. The master manages tablet assignment, server cluster
membership changes (with help from Chubby), load-balancing of tablets among servers, garbage
collection, and schema changes. The tablet server handles read and write
requests to its tablets, and splits tablets when they get too big.

Clients only need to communicate with the master to find the relevant tablet
servers. The relative amount of this sort of communication compared to the
actual data transfers is such that this practice is not a system-wide
bottleneck. Clients also cache this data until a server says the information
is stale.

Tablet location information (across all tables being managed by a single
Bigtable instance) is stored in one big 3-level hierarchical B+-tree type
thing (i.e. it's a sorted _n_-way tree, where you go down the appropriate
pointer from an internal node based on what it says about the interval of row
IDs contained in its leaves. The root is stored in Chubby. At the third level
I think you have the locations of all the tablets and what keys are owned by
each.

As previously mentioned, persistent tablet state is stored in GFS, but what is
that "tablet state". Each write or update is appended a commit log (with a
unique mutation identifier to resolve duplication caused by tricky crash
scenarios). It also modifies the (copy-on-write) in-memory representation of
the current state of this chunk of the table. One can always reconstruct the
in-memory representation by playing back the commit log, but ideally most
reads are serviced from memory. Every once in a while the in-memory state gets
too big, and _it_ is written out as an SSTable into GFS. Every once in a
while, multiple SSTables are read back into, and since some might be updates
to others, they are merged together, and the sources deleted.

In practice, the authors found that applying a very computationally efficient
compression algorithm (rather than one with a top-notch compression ratio)
increased overall system efficiency in many cases. They also added bloom
filters to the SSTables to reduce the number of unecessary disk reads of non-
existent data.

The paper says Google has used Bigtable as a backend for its Google Analytics
product, Google Earth, Personalized Search, and storing websites for
retrieving results for its Search Engine.

As future work they want to be able to provide better (but not full) support
for transactions (I think this is addressed in the Spanner paper released a
few years later).

Their lessons learned include

* "delay adding new features until it is clear how the new features will be
  used"
* "the importance of proper system-level monitoring"
* "the value of simple designs...code and design clarity are of immennse
  help in code maintenance and debugging"

They conclude by noting that this system bears some resemblance to the C-Store
column-oriented database system, for which there is another good paper that I
ought to finish reading at some point. They also note that people who are used
to "traditional" relational databases find the Bigtable interface awkward, but
that it works so much better than the alternatives that people end up using it
anyway.

### References

* Chang, Fay, et al. "Bigtable: A distributed storage system for structured
  data." ACM Transactions on Computer Systems (TOCS) 26.2 (2008): 4.

[btp]: http://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf
