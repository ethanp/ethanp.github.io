---
layout: post
title: "Bit Twiddling: Data Structure Alignment"
date: 2015-09-23 13:44:04 -0500
comments: true
categories: [bit twiddling, programming, competitive programming, operating systems, low level, C]
---

In order to align a data structure at the nearest word-alignment to a given
starting address, we'd must find the smallest multiple of N (a power of 2)
which is greater than or equal to integer X. Is there a way we can *make this
fast*, i.e. not use division or modulo?

## First Thoughts

### First of all

```c
if (N > X) return N;
```

### Otherwise, we have two cases

#### First Case

    N: 00100
    X: 01100
    --------
    => 01100

This example indicates the following:

```c
if (!(X & (N-1))) return X;
```

#### Second Case 

    N: 00100       
    X: 01101       
    --------       
    => 10000       

Which can be accomplished by

```c
return (X | (N-1)) + 1;
```

So we have the first-take program of

```c
if (N > X) return N;
if (!(X & (N-1))) return X;
return (X | (N-1)) + 1;
```

<!-- more -->

## Engineering Wisdom

The above solution is in fact correct *(takes a quick bow)*.

When I looked up a better answer on [StackOverflow][StOve], I found a
program that does not use conditionals, and is therefore much faster

```c
return (X + N - 1) & ~(N - 1);
```

However, it was not obvious to me that this program works! Let's peer inside
and see how similar it is to my solution.

First we're going to compute something, and then we're going to "and" it with
`~(N-1)`. Well, "and"ing *anything* against `~(N-1)` is going to round it down
to a multiple of N. So that part is taken care of.

Now we need to get the *right* multiple of N. `if X < N`, then then adding X to
N and rounding down to a multiple N is just going to give us N. That's the
first line of my answer, taken care of.

So `if X > N`, we either do or don't have bits in place values below N. This is
what my last two lines are taking care of. However, we can deal with this by
noting that after we add X to N, if there are bits in place values below N,
when we subtract 1, it will not change any bits that will survive the "and"
stage. But if there are no bits in place values below N, then subtracting one
will decrement our answer wrt bits N and higher.

Hmm, not the best explanation of all time. But that's what's going on in there.

[StOve]: http://stackoverflow.com/questions/19450743
