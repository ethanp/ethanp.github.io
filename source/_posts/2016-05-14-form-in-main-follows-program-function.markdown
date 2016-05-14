---
layout: post
title: "form in 'main' follows program function"
date: 2016-05-14 14:44:11 -0700
comments: true
categories: [software engineering, clean code, refactoring, naming, convention, data-flow programming, Scala, DSL, Unix, pipeline]
---

My program is a pipeline that takes multiple data sources, transforms them,
mashes them together, and writes them to multiple locations. It does this in a
somewhat resilient way by using Kafka as an internal buffer and data bus.
However you would have no idea from the structure of the program that that is
what is going on. In the "main" method, all that happens is a few
configuration settings are overridden, and a server is started. That doesn't
tell the reader _anything_ about what's happening.

Since I'm using Scala, the new design makes the "main" function look more like
a Unix program:

```scala
val src1: DataSource[Type1] = Type1Source()
val src2: DataSource[Type2] = Type2Source()
val merger: Merger[Type1, Type2, Type3] = OneAndTwoMerger()
val output1: DataSink[Type3] = Dest1Sink()
val output2: DataSink[Type3] = Dest2Sink()

Merge(src1, src2) | merger tee (output1, output2)

// desugared to show there is no magic
Merge(src1, src2).|(merger).tee(output1, output2)
```

Ok, so data streams emanating from source-1 and source-2 are merged together
by a type-compatible "merger" class, who writes its output stream into both
output-1 and output-2.

There's not a lot of code required to create those interfaces and methods.
Basically any number of `DataSink[T]`s can be "observers" of a
`DataSource[T]`. Whenever a `DataSource` finds itself with data to publish, it
calls the `receiveMsgs(msgs: Seq[T])` method of all the observing `DataSink`s.
So now we have a "reactive" (sources produce data whenever it is available to
them), and typesafe pipeline where components can be swapped in an out.
Communication between sources and sinks by default is just function calls
(i.e. synchronous), but their calls could be wrapped with Futures or Akka
actors. Using function-calls makes coding, testing, debugging easier, has
better type-checking, and doesn't need backpressure. Increased asynchrony
would allow for higher speeds, but is not needed yet, and will be hooked-in
as-needed.

The biggest influences on this design are the Unix shell, and the Akka
Streaming library, which I saw some presentations about. I think both were
inspired by electrical engineering (e.g. circuits and signal processing).

With this approach, each component has a single responsibility: to ingest,
filter, transform, aggregate, or output streams of data. Then in the "main"
function we just assemble the data flow of the program by hooking components
together. This means to test the program, we just need to test that each
component produces or consumes the data that it says it does properly.

Before, almost all of my tests involved at least three separate major program
components. I think I will start by re-writing those, and wherever things
don't work, write lower-level tests of one thing, and keep zooming in like
that. That way, testing effort is spent on the parts that are hard to get
right. I'm not writing the tests first because most of the code for the
program is simple hooking things into each other. Testing that would be an
unecessary duplication of effort. If the main logic is so plain to see and
understand and will not undergo heavy modification, it does not need to be
written twice. Then there are a few bits that use some pretty difficult
external APIs that can be used well and can be used badly. I want to make sure
that I'm using those at least as well as is necessary for the program to
function properly. Most of the issues I've had in the past are with the HDFS
API. With HDFS, it takes to take a little while sometimes for opens, writes,
and closes to propagate properly to all the replicas. Before I knew that, I
was using the API sub-optimally, and the program would crash every twelve
hours or so. That problem itself would not be simple to test against, but it
gives the impression that interaction ith these external APIs is where the
main complexity in my program lives.

In this new "source-to-sink" program model, a single Kafka "topic" can be
implemented as an `object` (i.e. Singleton) that has two ends (fields): a
`Producer` (which is a `DataSink[T]`, since it writes data out of the
program), and a corresponding `Consumer` (a `DataSource[T]` for the program).

So if the program has two "main" functions, one connects to the `Producer`
side of a Kafka topic, and the other connects to the `Consumer` side, all
using this Unix-like Scala DSL, then we have integrated Kafka as a resilient
buffer connecting two stages of the pipeline. This means the computation
subgraph connected within-JVM to the Consumer side can be taken offline for
fixing or augmentation without losing ephemeral data being collected by the
Producer side.
