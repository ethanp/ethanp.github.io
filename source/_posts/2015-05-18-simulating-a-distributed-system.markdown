---
layout: post
title: "Simulating a Distributed System"
date: 2015-05-18 19:24:37 -0500
comments: true
categories: distributed, systems, school, akka, java, scala, cluster
---

For my Distributed Computing class at school, there were 3 projects that each
involved implementing a distributed protocol over a simulated distributed
cluster of computers that are only 'connected' to one another in that they know
each other's IP address and must communicate over TCP sockets. For each of the
3 projects, I had a *completely* different method of implementing the system.

### Implementation One: A True Distributed System in Vanilla Java

The first project (Three Phase Commit) was the only one for which I worked with
a partner. We were unclear how to attack the problem of building a distributed
system in a simple way, so we went ahead and implemented a truly distributable
Java application, in which you could spin up processes on any computer and it
would connect in to the master server, who would tell the new node the
listening ports of other servers in the cluster manually, etc. This turned out
to be perhaps overly ambitious. When a node was expecting a message from
someone else, it would fire a timer thread, which on completion would report to
the master to restart the failed process. This system did work properly
(somehow) but it seemed to be living-on-the-edge. It also required two weeks of
a lot of working on it and learning aspects of Java, unit testing, mocks, which
`Exception` would get thrown by which network event & when, etc.

### Implementation Two: Lightweight System Simulation

For the second project (Paxos), I said "Hooey" to all the Java hubbub, which
while I enjoyed its low-level-ness, forcing me to peak further under the hood,
it required a larger time commitment than I was prepared to make. I realized
that my class's requirements only specified that system "nodes" run
concurrently, share no memory, and talk over TCP. So in Scala, I made a `Node`
class state machine, and ran different instances of it on separate threads.
This was too easy though, and I was disappointed to find my implementation
complete more than two weeks before the due date.

### Implementation Three: Akka Cluster

Akka Cluster is a Scala-based framework for building a distributed system, in
which you define the address of a set of "seed" nodes, of whom at least one is
supposed be available at all times. Then when other nodes start up, they become
a "member" of the cluster by contacting any seed node. So for me (the "client"
of Akka Cluster), all there is left to do is define the protocol to run on top
of the cluster. (In this case the protocol was the simple eventually-consistent
database protocol "Bayou".)

Akka nodes follow the "Actor Model", meaning they share no state with any other
part of your program, making them "simple" to run on remote machines (I didn't
try that, the docs say it is easy though). The only way actors can communicate
is via message passing, for which there is a dedicated syntax, `!`. For
example:

```scala
case class MyMessage(data: DataElem)
anotherActor ! MyMessage(theData)
```

The code above would have the actor running this code, send `anotherActor` an
instance of the case class `MyMessage` with the `DataElem` that was passed in.

Inter-actor communcation is guaranteed *FIFO* by Akka, meaning that for any
pair of actors `A, B`, if `A` sends `n` messages to `B`, `B` will receive those
messages in the same order that `A` sent them. NB this says nothing about the
order with respect to other actors in the system.

Using Akka trivialized all sorts of implementation details I'd had to worry
about in the first project, and then ignored for the second project.

If there had been a 4th project for this class, it's clear to me that I would
use Akka again. For some reason my code ran *very* slowly, and it probably had
to do with some configuration-setting that I did not set in the Akka config
file. That was really the only problem I had that I did not fix. One problem I
did fix that took a while, was figuring out a good way to shut the system down.
There were various options discussed online, and eventually I found one that
worked for me, which was to call `system.shutdown()` on every instance System I
instantiate. Another option was to send `self ! PoisonPill` but for some reason
that didn't work. I guess different ones work for different situations.
