---
layout: post
title: "What is an abstraction"
date: 2015-07-28 23:23:05 -0700
comments: true
categories: [mental meandering, semantics]
---

After some further thought, it has become clear that my previous post about
abstractions and what makes them good got it all wrong. There, I said roughly
that an abstraction is something that allows you to simplify a hard mental task
to make that task easier. This needs to be reevaluated. The earlier post
claimed that a hammer is an improper abstraction for opening up a package. That
just doesn't sound right. A hammer is nothing more than a bad _tool_ for
opening up a package. A _tool_ is something that makes a task easier. A tool
and an abstraction are not equivalent. 

My Software Architecture professor began the semester by giving us about 20
very similar definitions of what software architecture is, to give us a general
sense of what people are talking about. I left the lecture equally uninformed
about what software architecture is, but did eventually come to grips with it.
With the Sisyphean nature of the task now understood, I would like to propose a
new definition:

> An abstraction is a way of conceptualizing something without having to think
> about everything that is actually going on. If we think in terms of the
> abstraction, we can arrive at the same conclusions as could have been derived
> using the "underlying truth of the matter" without as much mental effort.
> After communicating the chain of logic using the abstraction, the listener
> would also be able to derive the result by substituting the abstraction out
> for the "underlying truth".

It is basically a mental shorthand.

Here is a revised list of abstractions

1. Programming languages -- the "truth" is generally a sequence assembly
   operations
2. The Unix filesystem -- the truth is locations of files on disk
3. Mathematical notation -- the truth is the actual mathematical operations
   ascribed to the notation; this is related to the programming languages item
4. Design patterns -- the truth is a particular organization of logical
   constructs provided by a programming language
5. Object oriented programming -- the truth is a set of structs and
   corresponding procedures
6. A compressed file -- the truth is the file that was compressed

It is this sense of abstraction which I believe to be of monumental importance
for making progress scientifically, technologically, culturally, etc.

One thing that completely surprised me when starting to learn about computers,
is that in programming, one builds "layers of abstraction." But those layers
are built on layers, are built on layers, etc. until you get down to the logic
gates, etc. So a programming language is an abstraction with an implementation:
there is a "function" (known as a compiler) which maps the written program into
a sequence of instructions. But an abstraction needn't have an function mapping
it to its underlying truth.

How did I get caught up in some sort of philosphical meandering? It feels
important to have a firmer grasp of what an abstraction is, what makes one
good, and how to go about creating them. The goal is to reduce mental effort in
creating new ideas and communicating them to other humans or to machines. To be
able to do this, it may be important to know what an abstraction is and what it
is not. Now I am sure it is _not_ a hammer, in contrary to what I stated in the
previous post. It is also not Yelp.

So what makes one good? It should allow questions to be asked and answered of
the truth it represents in the terms of the abstraction. It may even invite new
useful questions to be asked. It may also lead to better abstractions. It
should justify the mental effort required to begin thinking in its terms.

How does one create one? It depends

1. Sometimes by extracting what is the same out of multiple instances of
   related items (e.g. design patterns)
2. Sometimes by replacement with a symbol (math notation).
3. Sometimes by stating the problem in more "human" terms
    * The Unix filesystem is just a bunch of "folders" that contain "files",
      and folders _are_ files too
    * Object oriented programming allows us to mentally group code into a
      _thing_ which _does_ stuff, even though the machine doesn't "see" it that
      way
