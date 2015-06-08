---
layout: post
title: "Summary of 'The Log'"
date: 2015-05-29 20:57:16 -0700
comments: true
categories: [distributed systems, log, kafka, linkedin, middleware, apache]
---

I read a great article by Jay Kreps, one of the dudes who brought you Apache
Kafka. The article is called [**The Log**: What every software engineer should know about real-time data's unifying abstraction](http://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying). The
article took me a few hours of mildly challenging reading, so I figure you can
spend a few minutes reading my summary and decide more clearly whether you want
to put in the investment of his explanation.

First, he describes the following concept: Let's define a **log** as a *total
order* of functions called and the parameters passed to each of those function-
calls. Let's define a *"commit log"* as a *log* of edits of the contents of a
database. Kreps notes that we can use a commmit log to build the state of a
database at any point in history. This is like `git`; as we patch in each
commit, we obtain the state of the repo at each time. If we feed the same log
to the same program on multiple machines in a cluster, (assuming none of the
functions called are *non- deterministic* and machines behave as we expect), we
will certainly have the same state on each machine after they have each
executed the log. This would be desireable perhaps to provide a "reliable"
*service* for which there are more reads than writes, and more reads than can
be handled by one node; then we can ensure any node is OK to read from by
having all nodes play functions as prescribed by the log.

Then, he describes the following situation: getting every part of a tech-
company's data to every service that needs it is very complicated. In the worst
and most naive case it would be *N^2* because each of *N* places would be sending
data to each of *N* places. That's a lot of network bandwidth, and complexity,
and data formats to understand, and places for things to screw up. So Kreps
suggests
<!-- more -->
just having all data producers standardize a framework for formatting
the data they produce, then just have all producers append to a single shared
log. Now, everyone who wants to read data from someone else can be sure to get
that data in the order it was produced by reading from that one log. Plus there
are N writers and N readers so getting the data to every where is *2N*. Now we
note that 2N < N^2 if N > 2, so most of the time this is advantageous.

Now, the problem is that this single log is not going to be able to handle that
throughput, and it is going to get way too big way too fast. To build Kafka as
an implementation of this "unified log" concept, the key optimization is to
"partition" this log, meaning different pieces of it are written to different
places (machines), and each piece is written in duplicate to multiple machines.
In general we lose the total ordering across all processes, but in general,
this total ordering was literally more strict than could possibly be useful.

> This is the point where I stopped paying as much attention.

Then Kreps goes on to talk about how important stream-processing is, because so
many of the services modern tech companies provide operate on a real-time feed
of events. Then he notes that log-table duality noted in the second paragraph
allows us to provide a reliable enriched event stream, ie. one that takes raw
events, joins each one with data from another table, and inserts some
maintained state like a counter.

Then Kreps notes that Kafka's cleverest provided algorithm for freeing up log
space, is to remove "records whose primary key has a more recent update." The
naive provided algorithm is to discard elements that are more than *x* days
old.

Then he notes how most companies just exist to manipulate data in a distributed
system, and how building distributed systems in Java has to a large extent
become a problem of putting open source lego blocks like Zookeeper, Kafka,
Netty, etc. together. After that he summarizes and concludes.


