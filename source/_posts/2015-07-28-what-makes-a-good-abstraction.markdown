---
layout: post
title: "What makes a good abstraction"
date: 2015-07-28 19:39:27 -0700
comments: true
categories: [design, abstraction, coding]
---

Creating good abstractions may have more value than anything else. The more I
learn about creating a company and writing code, the more important I realize
proper abstractions are.

Perhaps I should start by listing a few abstractions that I find to be "good"
ones so that we're more on the same page, because this an abstract topic, and
everyone probably has a different conception of it. Surely I've missed some of
the best ones, but these roll off my head:

1. Programming languages (in that they are a higher-level representation of
   machine-code)
2. Object-oriented programming
3. Design patterns & architectural patterns
4. The mouse (input device)
5. A "process" as used in Unix
6. Most mathematical notation

So what do these have in common? They give us a mental representation of what
we're doing that vastly simplifies what is actually going on. This means that
in order to come up with something which is at its base level very complex, we
only need to manipulate simplified mental objects. These means we must perform
fewer total mental operations, using less mental short-term memory, to achieve
the same result.

<!-- more -->

My Operating Systems professor asked, "How do you teach/learn elgance [in
design abstractions]?" His answer was two-fold:

1. "Study case studies, try to learn lessons"
2. "When you build a system, ask yourself 'What would Ritchie and Thompson
   say about my design?'"

Sometimes when we define a good abstraction as one that facilitates mental
operations (as I've done above) we can say that an elegant abstraction should
reframe what we're doing in terms that a more primitive human would be more
familiar with. A Unix "process" is basically saying that each concurrently
running program is its own discrete creature who doesn't occupy any parts of
the world that other creatures are occupying, and is capable of thinking and
effecting changes to its external environment, but cannot directly affect other
creatures, except to kill them, wait for them to die, and send mail to their
doorstep.

However, mathematical notation is in a different category, it doesn't really
personify anything, it most often just provides a symbol for something that
would require many words to describe. In the math library at school, I once
picked up some very old mathematical text, and it said something like

> The value ascribed to the first object can be said to vary in direct
> proportion to the inverse of the square of the second.

Oh, you mean `x = 1/y^2`? The real statement was actually much longer and more
confusing, and the representation in mathematical notation would have added
only a few more components. This notational description gets the point across
much faster, although it doesn't really connect with my "inner humanity".

With each of these abstractions, we must take (perhaps a lot of) time to build
up facility with them so that they then --

Oh, I should add __spoken and written language__ to that list above, duh!

Anyway, before we begin using an abstraction "naturally" we must internalize
what it represents. This is generally not easy. It took surprisingly long to
understand the basics object-oriented programming, and hopefully over the next
years I will continue to have zillions of a-ha moments in which more
complexities are bundled up into Lego bricks that can be assemble into assets
of object-oriented programs. But as stated by both Venkat Subramanian and
Martin Odersky with respect to Scala, just because something replaces
complexity with simplicity, doesn't mean it is easy to understand upon first
seeing it.

Perhaps noteworthy of the abstractions in my list above, with the exception
perhaps of item 4, they took _thousands_ of people conceptualizing the same
things over and over again, each refining the abstraction, to achieve the level
of simplicity and generality with which they can be applied today. (Maybe 5 is
an exception too, I'm not sure.)

So how does one learn to make a good abstraction? Well, perhaps we can forget
about being the _one_ to invent a really fundamental abstraction. Maybe
information theory was an abstraction almost fully envisioned by one person,
I'm not sure. Regardless, a small team of people is surely capable of
leveraging the innovations around them to contribute a further small
abstraction. E.g. sometimes one encounters an especially nice API, which lays
out just the right Lego bricks to be able to create a slew of useful "things".

I want to try another case study, which is the Yelp application, because I'm
not sure where this is going to go. Does this application fit the hypothetical
definition of an abstraction, reducing the number of mental operations required
to do accomplish an otherwise complicated task? Sure it does; I can find a good
place to by tapping a few buttons that are easy to locate without much thought.
Another thing it does is allow me forget about _how_ I want to find the place
to eat by dealing with that for me. In this case, I'm leveraging the effort
provided by Yelp's actual programmers, as well as the data provided by all
those people whom Yelp's algorithms think I am similar to. But I don't have to
think about all that, all I have to do is ask the application where I should
eat.

I want to try another case study, the hammer. I can be cheeky here and note
that my hammer makes every problem look like a nail, so it simplifies my
problems to the case of dealing with a nail, which I simply must whack at to
deal with. But sometimes my problem doesn't fit well into the "nail" box, and
the hammer will do the wrong thing. For example, when opening up those
impossible plastic packages, a hammer would potentially open the package, but
might break the contents in the process. It successfully reduced the creativity
I needed to solve the problem, but was still in the end an inelegant, or worse,
insufficient, solution.

Abstractions are powerful. They're hard to come up with. Strive for elegance.
Use them to reduce complexity. Use them to reduce the amount of mental
calculations and short term memory required to solve a problem. Just because it
takes a while to understand the abstraction doesn't mean it is bad.
