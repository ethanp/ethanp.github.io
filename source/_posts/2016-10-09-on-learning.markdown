---
layout: post
title: "On Learning"
date: 2016-10-09 15:03:32 -0700
comments: true
categories: [learning, school, life, career, books, reading]
---

[I like to learn things](https://www.youtube.com/watch?v=Zjm8JeDKvdc), but I
often feel like time I spend learning things is wasted.

### It's hard to tell what info is going to be useful and what is fluff

Especially, classes in school (at all levels) often led me to a lot of wasted
effort.

First of all, a lot of the stuff you're supposed to learn is simply not
useful knowledge. People say (especially in the liberal arts): "well just going
through the process is teaching you how to think." But I say, "why don't you
teach me how to think using stuff that's actually useful?"

In college computer science classes, I quite often didn't understand _why_ what
I was learning was important. E.g. during my first Operating Systems course. I
thought I would never end up needing anything I was learning in that class.
Virtual memory, threading, scheduling, networking, etc. I thought, "This is way
lower-level than anything I'll ever need to know." For me, the material should
have been motivated with an example like, "let's build a program for running a
medical device." Now we're talking; we're going to need to get all the
different low-level components right and make them fit together so that we can
save lives. Instead, we were expected to read a very long and dry paper on the
Therac-25, a radiation machine with concurrency issues that killed a few people
in the 1980s. I spent a few hours trying to read the paper. One could say I was
spending that time "learning to think", but I think I spent that time "getting
nowhere". In the end I learned about the Therac-25 from Wikipedia, a source of
information whose expected readership has a level of background knowledge more
commensurate with my own. The whole time I was reading the paper, I was
thinking: it would take me so long to get to the point of this paper, and I'm
not even going to get a whole lot out of it. No good. A few years later, I had
to read that paper again for my graduate operating systems class, and at that
point I had sufficient background knowledge to simply read the paper and feel
like I understood its content and learned important lessons about software
engineering. Completely different story for a different level of student.

### Some keys to not wasting effort

* Have something in mind that you want to accomplish with your newly-obtained knowledge
    * E.g. I want to build a medical radiation device
* Have someone (colleague) or some place (e.g. stack overflow) where you can ask questions when you get confused
* Learn the relevant vocabulary for the field on wikipedia
    * This may take at least half an hour of doing a depth-first search on the linked topics until you have a general grasp of the concerns and their names

<!-- more -->

### Qualities of time

During college, I learned from [Aaron
Swartz](http://www.aaronsw.com/weblog/productivity) about how one can't just
spend all day reading tough material because it's too strenuous. Instead, you
have to choose the right activity for the mood you're in, and there's not a
whole lot you can do if you're just not in the mood to learn anything. Aaron
says we should have a todo list that is aware of the fact that different tasks
require different levels of mental effort. Maybe after reading your compilers
textbook for an hour you're mentally tired and physcally peppy, so you go for a
jog. Then you're relaxed and ready to work on writing code for a program. Then
you'll be overwhelmed so you step back plan out some next steps for the
program. Then you're sick of the program so you meet some friends. Then you're
tired and you take a nap. And so on. The point is you're spending time
usefully, but still cognizant of what you're actually capable of achieving with
your current mental and physical state. When people aren't aware of this
technique, they end up not starting the project until the night before, and
have to do one strenuous activity all night, fighting exhaustion, and not
really getting much out of it because it's all slapped together. Granted,
plenty of people are really proficient at getting that strategy to work out. To
me, it's pointlessly stressful, when I can just start the project on-time and
finish way before the duedate, and move on with my life.

### Find the right source

The main ways that I learn things are from podcasts, youtube, wikipedia, books,
and blogposts. In terms of amount of deep and useful knowledge gained per
minute, reading a great book wins hands-down. 

Finding a great book can be pretty challenging, because you may spend some time
reading the wrong books. After looking for some time, I have finally found a
[compilers book](https://www.amazon.com/Writing-Compilers- Interpreters-
Software-Engineerin
g/dp/0470177071/ref=sr_1_sc_1?ie=UTF8&qid=1476048360&sr=8-1-spell&keywords=writ
ing+compilers+and+interpretere) from which I am learning what I wanted to know.
Before that I was reading more "traditional" (theoretical) books, but instead
of learning how to build a compiler, they taught me about various parsing
algorithms and how to get your language to work for each of them. Frankly, if I
want to write my own little programming language, I don't care which parsing
algorithm I use, I just want to parse my syntax and move on to the backend.

I went through a similar process for learning the fundamentals of algorithms. I
kept rotating through different books, until I finally decided to read the
"Algorithm Design Manual" (only the "tutorial" section). I was able to complete
the exercises but didn't feel like I really got it. So I tried to read "CLRS".
That book was going more deep into the subject than I wanted, so I started
"Algorithms" by Sedgewick and Wayne. It started off pretty slow, so I found
that by skimming, I was able to read a few hundred pages in the first few
hours. Then I got to a lot of material that I didn't already know. The book has
code, diagrams, exercises, lecture videos, etc. With this mix, I was finally
able to achieve a high learning per minute score. The whole time I was reading
this book, whenever I found a concept hard to grasp, based on my bad
experiences with other books and classes and tutorials, I could rest-assured
that it would be unlikely that I could find a better explanation elsewhere.
This attitude strengthened my resolve, and I ended up reading and understanding
almost everything in the book.

### When to skim

In my experience, skimming material does not lead to understanding the
material. Instead, it only leads to that feeling later of "yeah I think I've
seen this stuff before". That said, skimming is often crucial, especially for
reading blogposts. 

Blogposts may be written with any of an assortment of aims.
For example, the author may be trying to assert her expertise in an area by
providing a good explanation, in order to further her career. Those are usually
the best ones, and can be read deeply for understanding. Or the author may be
trying to clear up his own thoughts on some subject. That's what I'm trying to
do here. In that case, skim to find the parts that are relevant, and read
those. Or it may be written for the author to remember something, like how to
get HDFS running locally on Mac. Thank heavens for such blogposts. Or it may be
some sort of "war story", which is basically a combination of the above three
types.

Lectures are generally only worth skimming. It takes me less effort to
understand a lecturer than to understand a book. But even when it's clearly
stated, the lecturer is generally not divulging much information. They may also
get lost on side-tracks or try to pull things in that they think are cool. That
stuff can be the most fun to hear, but it's generally not time spent really
learning anything.

One useful question to ask before reading something is "Can I just skim it?"
The answer to this is yes if you _don't_ want to be able to actively use that
knowledge in the future. Often skimming is useful for verifying that you
understand something. If I were to skim a tutorial on parsing theory, I would
be able to roughly guage how much of that stuff I actually understand based on
how much I'm "agreeing with the author" as I skim. A frustrating situation is
to read something deeply when you could have just skimmed it. Another time
skimming is useful is when you're stuck. When I got stuck reading about
computer architecture, I would go back to the table of contents. "How did I get
to where I am?" "Where are we going with this stuff?" "What do I find
interesting about this?" Then I would skip to some figures. Look at the figure.
Is it cool looking? Wouldn't it be fun to understand what it means? Let's look
at some of the keywords in the text near the figure. Oh there's some words I've
seen before but didn't understand, I guess they were important. Now I can go
back to their definitions and see if I can place them properly in the figure.
And so on.

### Is learning important

I find myself to be in the position that people want me to help their companies
make money. They want to trade my time and energy for their investment money.
It's a "good spot" to be in, someone once assured me, and I agree. It's fun
helping people make a business, and make cool products, and work together to
solve interesting problems along the way. I am convinced that the reason they
want to trade my time for their money is because I have, over time, stashed
away some knowledge into my brain in such a way that it is accessible to be
linked together and molded to fit the constraints of their problems. And I
enjoy doing the linking and molding and so on even though it requires effort.
So, what I'm trying to say is that I have developed some methods that I believe
to have improved my ability to affix knowledge into my brain accessibly, and I
have tried to share some aspects of those methods above, mainly for my own
benefit, but hopefully for someone else's benefit too.
