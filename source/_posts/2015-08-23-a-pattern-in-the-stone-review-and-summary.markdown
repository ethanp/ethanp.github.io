---
layout: post
title: "A Pattern in the Stone: Review and Summary"
date: 2015-08-23 16:56:25 -0500
comments: true
categories: [book, review, computers, bits and bytes, Feynman]
---

### How I Found it

I was watching a biography of Richard Feynman, and they interviewed this guy W.
Daniel Hillis, and he seemed like a cool dude so I looked him up on Google, and
came across his book, __*The Pattern in the Stone: The Simple Ideas that Make
Computers Work*__ (1999). On Amazon it was compared to __*Code: The Hidden
Language of Computer Hardware and Software*__, by Charles Petzold, an
encredibly well- written book about how computers work. I would gladly read any
book considered comparable in lucidity to _Code_, so I got _The Pattern in the
Stone_.

### What it says

The first few chapters are meant to give a basic understanding of what a
computer is actually doing, and he spends some time noting the Universality of
computers, which he says shows that "all computers are alike in what they can
and cannot do". Personally, my intuition of the workings of a computer comes
mainly from an explanation by Richard Feynman himself ([as seen on the
YouTubes][FeyComp]) which is "heuristic" rather than mechanical. Basically
Feynman gradually turns a human being into a computer, and then talks about how
this mechanistic person can be implemented using logic gates built from water
pipes that he sketches on a whiteboard. In _Pattern in the Stone"_, he builds
logic gates out of parallel and series wires, and also out of springs and
pivots.

[FeyComp]: https://www.youtube.com/watch?v=EKWGGDXe5MA

Then he introduces finite state machines and programming in LOGO. Then he
mentions how machine code can be thought of as _control instructions_,
specifying the next instruction to fetch and execute, and _processing
instructions_, moving data to and from memory, and through the Arithmetic Logic
Unit.

Then he starts really getting into what I think is the main point of the book,
to convince the reader that there is no magical process occurring in our brains
that a mechanical computer cannot replicate, meaning that

> As far as we know, no device built in the physical universe can have any more
> computational power than a Turing machine...[so] a universal computer with
> the proper programming should be able to simulate the function of a human
> brain.

<!-- more -->

Then he explains a bit about quantum computing. He notes that when a water
molecule is formed, its atoms must "know"/"compute" the appropriate angle
between their bonds. Computational approximation methods of determining this
angle are slow, but the water molecule does it "almost instantaneously".

> One way of explaining how the water molecule can make the same calculation is
> to imagine it trying out every possible configuration simultaneously— in
> other words, using parallel processing. Could we harness this simultaneous
> computing capability of quantum mechanical objects to produce a more powerful
> computer? Nobody knows for sure.

He notes that although it would certainly be possible, "there is no evidence
whatsoever" of the human brain leveraging quantum mechanics in its operation.

Then he defines an algorithm as a procedure for provably solving a problem, and
a heuristic as a rule that _tends_ to get the right answer. Then he uses this
to describe a chess-playing AI using alpha-beta pruning.

After discussing encryption and encodings, he moves on to parallelism. He notes
that a computer analyzing a photo has to look at the image pixel-by-pixel, but
the human brain seems to process the entire photo, even comparing it to
previously seen photos, very quickly, even though neurons themselves are
comparatively slow.

Then he goes into one of those bold claims of an expert that are very fun to
read:

> As computers on the network begin to exchange interacting programs instead of
> just electronic mail, I suspect that the Internet will start to behave less
> like a network and more like a parallel computer. I suspect that the emergent
> behavior of the Internet will get a good deal more interesting.

> As the information available on the Internet becomes richer, and the types of
> interaction among the connected computers become more complex, I expect that
> the Internet will begin to exhibit emergent behavior going beyond any that
> has been explicitly programmed into the system.

To me this statement seems vague enough to _definitely_ be true. I would
imagine that already, the interaction between different high-frequency-trading
systems display emergent behaviors, though I have no idea.

Then he gets to machine learning. He notes that computers need three
ingredients for learning: a goal, a measure of how far off they are from that
goal, and a way to reduce that distance. He describes the similarities of
"artificial neural networks" with the human brain. He mentions that by building
up a multi-layer artificial neural network larger abstractions can be encoded
in the weights connecting the neurons.

Finally, he gets to my favorite part of the book: how to build AI that understands humanity

> I believe that we may be able create an artificial intelligence long before
> we understand natural intelligence, and I suspect that the creation process
> will be one in which we arrange for intelligence to emerge from a complex
> series of interactions that we do not understand in detail— that is, a
> process less like engineering a machine and more like baking a cake or
> growing a garden. We will not engineer an artificial intelligence; rather, we
> will set up the right conditions under which an intelligence can emerge.

> [The practice in systems engineering] of "divide and conquer" works very
> well, but an evolved object like the brain does not necessarily have this
> kind of hierarchical structure.

> The brain is much more complicated than a computer, yet it is much less prone
> to catastrophic failure. The contrast in reliability between the brain and
> the computer illustrates the difference between the products of evolution and
> those of engineering. A single error in a computer’s program can cause it to
> crash, but the brain is usually able to tolerate bad ideas and incorrect
> information and even malfunctioning components.

> So, in creating an artificial intelligence, what is the alternative to
> engineering? One approach is to mimic within the computer the process of
> biological evolution.

He notes how he created a sorting algorithm that outperformed quicksort by _evolving_ it using sequences of computer instructions. _Whoa!_

> One of the interesting things about the sorting programs that evolved in my
> experiment is that I do not understand how they work. I have carefully
> examined their instruction sequences, but I do not understand them.
> 
> If the safety of the airplane depended on sorting numbers correctly, I would
> rather depend on an evolved sorting program than on one written by a team of
> programmers.

This book is super cool.

> I suspect that consciousness is a consequence of the action of normal
> physical laws, and a manifestation of a complex computation

This is the same conclusion me and some friends reached this past spring at the
end of a weeks-long discussion.

> but to me this makes consciousness no less mysterious and wonderful— if
> anything, it makes it more so. Between the signals of our neurons and the
> sensations of our thoughts lies a gap so great that it may never be bridged
> by human understanding. So when I say that the brain is a machine, it is
> meant not as an insult to the mind but as an acknowledgment of the potential
> of a machine. I do not believe that a human mind is less than what we imagine
> it to be, but rather that a machine can be much, much more.

What a beauty.
