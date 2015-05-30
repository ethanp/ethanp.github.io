---
layout: post
title: "Paxos as a Classroom"
date: 2015-05-18 19:28:49 -0500
comments: true
categories: [distributed systems, paxos, analogy, metaphor]
---

    multi-paxos as a classroom:
    teacher := proposer
    students := acceptors
    all := learners

    teacher to students:
        may I have your attention for the next 1:15 hrs?
        I'd like to teach youz guys all about "the paxos protocol"
        here is what I'm assuming you already know

    each student to teacher:
        if not already busy for that time:
            yes, here's how much I actually already know about paxos
        else:
            my apologies, but I will be busy skimming facebook for the next 1:15
            so if there's something you want to tell me, post it on there

    teacher to each student:
        given how far behind you are, these are the things you need to catch up on
        but also this is what I'm teaching now

    each (alive & correct) student to teacher:
        oh very cool!
        this is how much I now understand of all the stuff you've said

    teacher:
        if the majority understands more than before:
            set new start point for next time's lesson
