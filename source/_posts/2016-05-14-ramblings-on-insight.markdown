---
layout: post
title: "ramblings on insight"
date: 2016-05-14 14:46:12 -0700
comments: true
categories: [software engineering, clean code, refactoring, naming, convention]
---

I'm re-designing a program I've been working on for a few months. I've
implemented a prototype for the new design and also started implementing it.
Because I haven't thought this through completely or written much of the code
yet, there is still room for fundamental issues to come up with plugging this
model into my codebase which will make it impossible for this major
refactoring to complete. But really, what I expect is that this model allows
us to express what is really going on in such a way that by looking at the
_names_ of the components, we know _where_ to start looking for code
describing _what_ is really happening. If that is true, and we write function
names from the top-down (i.e. __start with "main" and go down, as a breadth-
first search__), then whole bunches of code will be isolated in that they
survive to create the capability of some unique higher-level function (or a
small set of functions).

Even though I have a clearer conception of how to go forward than I did when
first writing the program. I still find it hard to think through the whole
problem without just trying it. But if I just try it then I don't know what
I'm doing. Then, after I learn what the pieces I need _are_, I can start from
the top-level design and then go down.

I don't know if there's a particular _moment_ when I am _ready_ to see the
problem more fully. In this case, it seemed to happen because I _had_ to find
a better way. The program was going to be a lot of trouble to keep dealing
with if I didn't figure out a better way to name and organize the different
concerns it actually addresses. By naming and organizing things better, we get
a little "registry" of package, class, method, and value names that reveal the
solution's structure.

Refactoring can get a bit tedious. But maybe the most tedious bits are often
not really worth doing. I hope to build up a better intuition for it. But for
now, it's one of those things where I don't know the words to describe what
I'm really making. I can see how knowing a bunch of "patterns" would make this
easier. Especially considering that the crux of the pipeline I implemented is
the "observer" pattern, one of the only patterns I know. But I think the main
thing is to just try things out, see what works, and what doesn't, then go
back and learn the "patterns" after I've read and written a bunch of different
designs _in the code_.
