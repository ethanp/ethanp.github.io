---
layout: post
title: "Lesson Learnt about Collaboration"
date: 2015-07-22 21:16:25 -0700
comments: true
categories: [life, coding, learning, pair programming, collaboration, growing up]
---

Perhaps the greatest lesson of this summer has been in taking advantage of the
power of collaboration. 

When my boss decided to hire me, he told me that it was in large part because
of my communication skills. I don't know if he still agrees with this, but I
like to think it is true.

One of my greatest prides is the ability to openly lose an argument. For a long
time, this has quite often been my reason for entering an argument, and I try
to make it easy to lose. If someone seems to know they're right, we must
together find the bridge of what I'm missing that will be convincing beyond a
reasonable doubt of their correctness. Making it easy to lose means figuring
out what you *actually* think, making that clear, and not wavering from that
initial point of view even when more facts come to light. Or at least
acknowledging that the original viewpoint was incorrect, and now *this* is what
I [honestly] believe to be true. A regrettable human tendancy is to change
one's opinion during an argument as the facts come to light because "with these
facts, my original point of view was wrong, and clearly I wasn't wrong, so that
couldn't have been my real point of view." This needs to be consciously
avoided.

<!-- more -->

But still, I haven't always taken advantage of opportunities to collaborate.
There is a lot of overhead when working with a partner. A lot of your time is
spent explaining to them stuff you already know. These could be things which
took quite a while of staring at text trying to understand, and now it feels
like you're letting them off the hook by just explaining it outright. Sometimes
there are petty disagreements in which I like to code things my way and she
hers, etc.

One small example is that in one project I wanted the code for a very
complicated function to have the following structure

```scala
def complicated1() {
    val precondition1 = state1 != invalid && state2 != invalid
    val precondition2 = state3 != invalid || state4 != invalid
    val preconditionsMet = precondition1 && precondition2
    if (!preconditionsMet) {
        logError("error 1")
        return // early escape
    }
    val computationResult = doSomeComputing()
    if (!isValid(computationResult)) {
        logError("error 2")
        return // another early escape
    }
    // the main logic here
}
```

whereas my partner wanted something more along the lines of the following

```scala
def complicated2() {
    if (state1 != invalid && state2 != invalid
     && (state3 != invalid || state4 != invalid)) 
    {
        val computationResult = doSomeComputing()
        if (isValid(computationResult)) {
            // the main logic here
        }
        else {
            logError("error 2")
        }
    }
    else {
        logError("error 1")
    }
}
```

It's a petty issue; either way the *exact* same code is going to execute, and
we each had very good reasons for finding our own way easier to understand. We
didn't bicker over it because we got along very well, and in the end we
actually did it my way. But in reality the actual function was considerably
more complex, and perhaps there were entirely better ways of structuring it
that neither of us noticed.

Now, at work, there is a new-hire considerably more senior and experienced than
I, who started 2 days ago, and who is taking over my project once I go back to
school in a few weeks; so it has been my responsibility to teach him how
everything works. Very often, he feels the need to obsessively ask me: "Your
code that says `3 + 4`, that means you're adding 4 to 3, right?" Sometimes he
says "right?" and I say yes, then he repeats what he said and I repeat "yes,
you are correct", and this can continue and I wonder what he is getting out of
that.

At first I resented this because it sometimes felt like no one taught *me* much
about the codebase when I arrived; for the more complex bits, I just stared at
it until it clicked. This was fun to me, and no one ever seemed to have the
time to go in depth with it anyway. It was good practice at reading others'
code and understanding what it does. Learning new ways of accomplishing many
things that were unfamiliar. Getting used to seeing code that isn't formatted
*my way*, and accepting it for *what it is*. Learning how to utilize their code
from mine even when there is no explicitly exported clean API, and so forth.

But well it was the new-guy's day 3 today, and it seems we have really begun to
click as a duo. It is abundantly clear that he has a *wealth* of skills I lack.
Sure, he could definitely learn this code without my help, but I benefit a
great deal from seeing how he navigates it. What are the parts that he finds
especially confusing? The importance of variable names shines immediately
through. There are a few patches of code where I worked some Scala feature in
there just to try them out. Some of these turned out to be easier to understand
and reuse code from, and for some of them, something more cut-and-dry would
have been better.

My new partner's oft-stated goal is to get to a point where he can start
contributing to our project without my help, so that our concurrent efforts
will speed things along. People with a smart, helpful attitude like that have
been rare in my experience, so if he's being honest, I'm all for taking him to
that place as quickly as I can.

I think maybe I just have a bias that everyone who doesn't know what I know
just wasn't smart enough to realize it. That is the thinking pattern of a real
asshole and this experience is making that abundantly clear. Everyone knows
that when learning something new, everything is always surprisingly confusing,
and then once it clicks, everything becomes "trivial." However, I have not
spent much time teaching others, so I haven't had the opportunity to see this
"click" moment in others, and how dumb they all look before that moment occurs.
I do remember teachers explaining something, and me looking at it sideways in
confusion, and asking them to repeat it slower. This happened on many
occasions, and they must have thought I was a real idiot on each of those
occasions, but I was always so relieved when they did repeat, and I often would
strain my brain muscles and actually (partially) "get it" the second time
through.

The new-guy has taught me a lot from his attitude in joining a new team at a
new workplace. He's not trying to just "set to work" writing code willy-nilly
as quickly as possible. Compared to him, that's basically what I did. He wants
to *truly* understand what's going on. What does the architecture look like?
Who is the expert on what? Who is good at explaining things? Who is willing to
take the time to explain things? What do our users want? What do they expect?
What is the timeline? What has been built? What is he responsible for? What are
the alternative solutions that have been under consideration? Who has decision-
making power? Which teams do we collaborate with?

These are all questions that I know the answers to, as they are the necessary
fundamentals of how to operate and move forward. I started actively learning
the answers to these questions as soon as I arrived as well. However, I wasn't
as conscious and certainly not nearly as thorough at answering them as he is.
And over time I may have slipped a bit into a position of people trusting me to
do the right thing, so I don't feel as pressured to be on the ball with
everything at all times. But his arrival has been a reality check in the
importance of having a firm grip on the fundamentals of what are we doing and
why and how best to accomplish it in a strictly-business manner. There is no
excuse for losing one's professionalism, including having a "casual" workplace.

So far it has been 3 days. On a personal level of course I have learned more
during those 3 days of working with the new guy than I did during the previous
few weeks of working on my own. During this week I have probably also been
slower to accomplish those tasks remaining to finish before I leave. It is
firmly clear that this is a worthy tradeoff, and I will continue to spend the
rest of my time making sure that I am playing as great a part as possible in
his success in moving our project forward into the future. There maybe some
hiccups in our working relationship, because alas I barely know the guy, and
we're from different societal cultures, and my first impressions of people are
always notoriously off-base. However, now it is clear that the responsibility
is my own to make sure any future road-bumps are known to be my fault and my
problem and mine to fix with his help.
