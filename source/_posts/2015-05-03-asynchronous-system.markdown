---
layout: post
title: "Asynchronous System"
date: 2015-05-03 21:27:36 -0500
comments: true
categories: distributed, systems, school
---

Before taking a Distributed Computing course at school, I had written for
various program environments:

1.  a script that trains a neural network to recognize whether an English
    sentence is grammatical
2.  an iOS app that involves a fair bit of multi-threading
3.  a command line interpreter that forks and runs commands as background
    processes
4.  the Raft distributed consensus protocol inside a professor’s multi-threaded
    distributed system simulator
5.  some asynchronous features for a simplified database system where threads
    communicate solely via Java’s `PipedInputStreams` instead of using shared
    variables

Perhaps you are familiar with these situations, and like me thought they might
be “distributed”. Reality check: they’re not. Similarly, in a distributed
system, one has multiple “threads” processing data at once. However,
these “threads” may be each on different machines; this adds some complexity.
None of the situations outlined above have the sort of issues one can expect
fairly regularly when talking amongst machines.

### Different machines don’t share
<!-- more -->
1.  heap space
2.  scheduler
3.  clock time (i.e. at any moment each machine's "watch" reads a different
    time)
4.  clock tick speed (i.e. clock time's "drift" apart from one another)
5.  stable storage
6.  etc.

### They can’t differentiate between friends in any of the following states

1.  dead
2.  became slow
3.  the network literally broke down between the two
4.  the network become slow between the two

This is because all they know is "I've sent my buddy a message and have yet to
hear back."

## If you told me all this, my response might be:

> Why not just run your program under the expectation things will work, and
> hope for the best?

### The answer (as far as I can guess) is several fold:

1.  “If you are the company who has figured out a better answer than ‘hoping
    for the best,’ people will pay you for access to your software” -- Adam
    Smith
2.  Your Users will be *very* pissed if their stuff seems to randomly
    disappear; even if it only happens to a few of them every once in a while.
     * This becomes especially likely if you have a lot of users with a lot of
       data
3.  You’re going to have to be super-over-cautious about not losing important
    data. It would be great to speed things up in such a way that you're still
    not losing any important data, but also not wasting time being overly
    cautious.

For the research community, the method they have derived for coming up with
better solutions than the never-failing “hope for the best,” is **formalism**.
By giving ideas with a lot of moving parts names, they simplify the task of
using the human brain to derive solutions. The most important formalism is the
**asynchronous model**.

## The Setting: Asynchronous Message-Passing and Assumptions

In an **asynchronous system** (as opposed to a *synchronous* one), a few
properties of the system are explicitly assumed. The basic assumptions are
listed above under the names **“Different machines don’t share”** and **“They
can’t differentiate between when each another machine has done any of the
following”**.

However, in an effort to always find the simplest solution under the smallest
set of assumptions possible, different authors use variations on the above
assumptions. Commonly they will say something like "channels are reliable", by
which they mean the network connection between any two nodes will always
*eventually* transmit every message it is asked to. The word *eventually* is
thrown around a lot, and it means there is some time before *never* when it
will happen.

By adding additional assumptions (e.g. "packets sent between any pair of
processes cannot be reordered en route") simpler solutions can often be found.
By removing assumptions, upper bounds can be established on the costs of
solving certain problems.
