---
layout: post
title: "Name According to Function"
date: 2016-05-12 15:08:16 -0700
comments: true
categories: [software engineering, clean code, refactoring, naming, convention, data-flow programming, books, Scala, DSL, Unix, pipeline]
---

Over the past few months, I wrote a kind of crappy program. Now I need to make
additions to that program and there is a lot of internals about the program
that I need to recall to be able to implement the additions efficiently and
robustly. This is not a world I want to continue to live in, so my crappy
program requires improvement. No one I asked had advice that I could put into
practice, so I looked to Amazon for a book with the answers on "what
_specifically_ to do". Based on its cover, title, and reviews, I ended up with
the book __Clean Code: A Handbook of Agile Software Craftsmanship__, put
together from multiple authors by a guy who refers to himself as "Uncle Bob".
The book is frankly a big part of the answer I was looking for.

I don't know a whole lot about Uncle Bob, but he seems to be _very_
experienced with designing, writing, maintaining, refactoring, etc. large Java
projects. He also is very well-read on modern software engineering, but
manages to stay away from the over-use of terminology I'm not familiar with.
He also talks about his conversations with (the only) guys in this arena that
I have heard of, including Fred Brooks and Martin Fowler.

His writing tone can be characterized as follows, "I have read and written
sooo much more code than you, and over time, I have taken the time to think
about the best way to make the code "clean". Herein, I shall share with you
what I have learned, using the best didactic methods I can think of. I hope
you benefit as much as possible from my wisdom."

So far (over the past 3 days) I have read perhaps 1/3 of the book, but I have
already come up with a design for my program that is __so much__ better than
the old design, and a rough prototype implementation is already running. That
is an incredible return on investment.

The main thing I have understood from the book is that __parts of programs
should do what they say they do__. Put most-briefly, there are two parts of
making that possible

1. __Components should be simple enough to be described by just a few words__
2. __The _name_ of a component should be the few words that describe it__

It is embarrassing to say that I never thought of that myself, but the truth
is I didn't, and that is reflected all-too-obviously in the program that I
wrote. I can come up with countless excuses for why I wrote the program that
way, but that doesn't fix the problem. The only way to fix the problem is to
reorganize the program to conform to the above two rules.

<!-- more -->

My program is a pipeline that takes multiple data sources, transforms them,
mashes them together, and writes them to multiple locations. It does this in a
somewhat resilient way by using Kafka as an internal buffer and data bus.
However you can't tell from the structure of the program that that is what is
going on.

Since I'm using Scala, the new design makes this look more like a Unix
program:

```scala
val src1: DataSource[Type1] = Type1Source()
val src2: DataSource[Type2] = Type2Source()
val merger: Merger[Type1, Type2, Type3] = OneAndTwoMerger()
val output1: DataSink[Type3] = Dest1Sink()
val output2: DataSink[Type3] = Dest2Sink()

Merge(src1, src2) | merger tee (output1, output2)

// or desugared:
Merge(src1, src2).|(merger).tee(output1, output2)
```

There's not a lot of code required to create those interfaces and methods.
Basically any number of `DataSink[T]`s can be "observers" of a
`DataSource[T]`. So now we have an asynchronous, "reactive", typesafe pipeline
where components can be swapped in an out by data sources calling their
`publishMsgs(msgs: Seq[T])` method, and data sinks implementing their
`receiveMsgs(msgs: Seq[T])` function.

The biggest influence on this design is the Akka Streaming library, which I
have not tried out yet. Instead of taking the time to learn it, I have
implemented a very basic version of their design. Of course the Akka team
didn't invent the design either, it is similar to "flow-based programming"
which is in-turn inspired by the practice of electrical engineering.

With this approach, each component has a single responsibility: to produce the
data it says it does, or merge the two types it says it does, or write a type
to the place it says it does. Then in the "main" function we just assemble the
data flow of the program.

In this model, a Kafka topic can be implemented as an object (i.e. Singleton)
that has two ends: a producer (which is a DataSink, since it writes data out of
the program), and a corresponding consumer (a DataSource for the program).

So if the program has two "main" functions, one connects to the producer side
of a Kafka topic, and the other connects to the consumer side, all using this
Unix-like Scala DSL, then we have integrated Kafka as a resilient buffer
connecting two stages of the pipeline. This means the consumer side can be
taken offline for fixing or augmentation without losing ephemeral data
collected by the producer side.

Because I haven't thought this through completely or written the code yet,
there is still room for fundamental issues to come up with plugging this model
into my codebase which will make it impossible for this major refactoring to
complete, but what I expect is that this model allows us to express what is
really going in such a way that by looking at the _names_ of the components, we
know _what_ is going on _where_.
