---
layout: post
title: "Brief Overview of Garbage Collection in the HotSpot JVM"
date: 2015-06-08 18:50:33 -0700
comments: true
categories: [JVM, implementation detail, performance, optimization, Java, virtual machine, HotSpot, garbage collection, systems]
---

__Disclaimer:__ This post only claims to represent my _current understanding_
of the workings of the HotSpot VM's garbage collector. This understanding comes
from reading the _references_ listed below, as well as making some assumptions.
It is not necessarily fully correct.

### Brief Overview

__The garbage collector _separates the heap into 5 sections of 4 types_.__ I
don't know whether these spaces are "virtual" (i.e. remapped to discontiguous
"pages" like Linux's *virtual memory addresses*) or not. I suspect they are
_not_.

1. __Eden__ --- where objects go when they're first allocated in the running
   program
2. __Survivors 1 & 2__ a.k.a. "young space" --- objects in Eden are moved here
   if they survive a minor ("young") GC
3. __Tenured__ (a.k.a. __Old__) --- long-lived objects (that have survived [a
   configurable number of] minor GCs) are moved and then live in here
    * We can tell the JVM to allocate all objects larger than `n` bytes
      directly into the *old* space.
4. __Permanent__ --- this is where the JVM's own objects live (e.g. classes and
   JITed code). It behaves just like the *tenured* space.

<!-- more -->

The GC is arranged "generationally" because (according to the "__generational
hypothesis__") it is assumed the longer objects live, the longer into the
future their life expectancy is. So if we move the older objects into a
separate bin, we can do quick, efficient, lucrative "minor GCs" in which we
only garbage collect from Eden and the Survivor spaces.

__Minor GC__ is triggered _when Eden becomes full_. It uses the root references
to collect the reference set, and moves all live objects from Eden and one
survivor space into the other survivor space (a.k.a. "mark-and-sweep". I guess
this means we're physically moving the object in RAM because we have to update
all the objects references to the new location.

__Root references__ for minor GC are from the stack (I think this means entire
stacks for all running threads) and old space. HotSpot uses "dirty cards" as an
optimization to not have to trace through references from all members of the
old space, only the modified ones.

__Full GC__ would _intuitively_ be triggered by running out of space in the
"tenured" or "permanent" bins, but this is not _necessarily_ the case.

#### References
* [Grid Dynamics](http://blog.griddynamics.com/2011/06/understanding-gc-pauses-in-jvm-hotspots.html)
* [Cubrid Blog](http://www.cubrid.org/blog/dev-platform/understanding-java-garbage-collection/)
* [StOve](http://stackoverflow.com/questions/9546392/)
