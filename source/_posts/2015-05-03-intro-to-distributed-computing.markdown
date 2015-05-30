---
layout: post
title: "Intro to Distributed Computing"
date: 2015-05-03 21:18:38 -0500
comments: true
categories: [distributed, systems, school]
---

This semester I have been taking Distributed Computing with Professor Lorenzo
Alvisi. Distributed computing has been something I have been curious about for
a long time, and aside from writing some MapReduce jobs during a summer
internship 2 summers ago, I had no real experience thinking about distributed
systems. The class was very enjoyable.

The main lesson was that **distributed systems are very flaky places**, but you
can arrive at some **surpringly simple solutions** to most of the problems once
you find a better way of expressing what you really want. The other main lesson
was that **finding a better way of expressing what you really want can be a
confusing** thing to do.

## Scenario 1: “Two Generals’ Problem”
Persons 1 and 2 would like to be sure that both agreed on a time to
meet, but neither can tell whether the other is listening to what is being
said.

> Person 1: Let’s meet up at 2:00pm

Now person 1 doesn’t know whether person 2 was listening though. So person 2
feels compelled to respond (while still multi-tasking).

> Person 2: Yeah

<!-- more -->

Now person 1 knows 2 heard, but 2 doesn’t know whether 1 trusts him. So person
1 feels compelled to respond (in a very serious tone).

> Person 1: OK, great.

Now person 1 doesn’t know whether person 2 was listening though. So person 2
feels compelled to respond (while still text-messaging).

> Person 2: Cool.

Now person 1 knows 2 heard, but 2 doesn’t know whether 1 trusts him. So person
1 feels compelled …

**This situation never resolves itself** in a way satisfactory to both
individuals, so either they’ll stand there doing this until 2:00pm, or they’ll
give up and both skip the meeting because each was never *sure* the other had
really agreed to go.

### Take Away:
Think about it: when you text-message someone, you don’t know whether

1.  they’ve received the text yet (maybe they haven't looked at their phone)
2.  they’ll ever receive the text (there could be some “cell-phone issue”)
3.  they’re going to properly act on it even if they did receive it, or
4.  they’ve literally died since you last heard from them [unlikely but
    certainly possible].

What makes things especially difficult is that the other person has the exact
same problem when they’re texting to you. This means that even if you got the
message, they don’t know that you got it. And so you can try to reassure them,
and then they will know that you got it, but you still don’t know if they know
that you get it. And that circle goes on forever.
