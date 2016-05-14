---
layout: post
title: "Name According to Function"
date: 2016-05-12 15:08:16 -0700
comments: true
categories: [software engineering, clean code, refactoring, naming, convention, data-flow programming, books]
---

Over the past few months, I wrote a kind of crappy program. Now I need to make
additions to that program and there are a lot of internals about the program
that I need to recall in order to be able to implement the additions
efficiently and robustly. This is not a world I want to continue to live in,
so my crappy program requires some sort of improvement. I looked to Amazon for
a book with the answers on "what _specifically_ to do". Based on its cover,
title, and reviews, I ended up with the book __Clean Code: A Handbook of Agile
Software Craftsmanship__, put together from multiple authors by a guy who
refers to himself as "Uncle Bob" (real name Robert C Martin). The book is
frankly a big part of the answer I was looking for.

I don't know a whole lot about Uncle Bob, but he seems to be _very_
experienced with designing, writing, maintaining, refactoring, etc. large Java
projects. He also is very well-read on modern software engineering, but for
the first quarter of the book, manages to stay away from the over-use of
terminology I'm not familiar with. After that it goes into things like agile,
test driven development, behavior driven development, cohesion, object
oriented programming patterns (e.g. Gang of Four), plain old Java objects,
data access objects, etc. that I don't have much familiarity with. He also
talks about his conversations with (the only) guys in this arena that I have
heard of, including Fred Brooks and Martin Fowler.

His writing tone can be characterized as follows

* I have read and written sooo much more code than you
* Over time, I have taken the time to think about the best way to make the
  code 'clean'
* Herein, I shall share with you what I have learned
* I'll use the best didactic methods I can think of
* I hope you benefit as much as possible from my wisdom

His main didactic method can be characterized as

* Here is an example of some bad code
* This is how it goes wrong
* This is why that is bad
* Let's improve it in these areas
* Here is the improved code
* Note how this new code doesn't have the flaws of the previous

So far I have read perhaps 1/2 of the book, and I have already come up with
and partially implemented a design for my problematic program that is __so
much__ better than the old design, and a rough prototype implementation is
already running, and I have much more "direction/vision" for how it is going
to progress. That shows me that the _Clean Code_ book has produced an
incredible return on investment.

The main thing I have understood from the book is that __parts of programs
should do what they say they do__. Put most briefly, there are two parts of
making that possible

1. __Components should be simple enough to be described by just a few words__
2. __The _name_ of a component should be the few words that describe it__

It is embarrassing to say that I never thought of that myself, but the truth
is I didn't, and that is reflected all-too-obviously in the program that I
wrote. I can come up with countless excuses for why I wrote the program that
way, but that doesn't fix the problem. The only way to fix the problem is to
reorganize the program to conform to the above two rules.




