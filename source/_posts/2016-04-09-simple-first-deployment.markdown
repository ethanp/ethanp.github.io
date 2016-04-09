---
layout: post
title: "Simple First Deployment"
date: 2016-04-09 13:45:39 -0700
comments: true
categories: [work, software, deployment, production, systems engineering, shipping, retrospective, milestone]
---

I just had my first experience of "deploying my system into production". I
have been learning about software engineering for a few years now, and I have
seen this term "deploy into production" many times, but never experienced it
myself. The "software system" that was deployed is an internal tool, almost 3
months in development by yours truly. This article is a retrospective of the
development of this system so far.

<!-- more -->

The system itself uses many APIs with which I was previously unfamiliar;
Kafka, Elasticsearch, HDFS, ZooKeeper, Spray, CouchDB, etc.; but I have glued
together unfamiliar APIs on many previous occasions. The system is
"distributed" and has in-built fault-tolerance in various ways; but I have
included that feature in previous projects. The thing that is new for me about
this project is that it was written with maintainability as a goal, and
"released" in such a way that it runs continuously as a service, and its
output can serve as input for the daily work of other human beings. This is my
first project for my first job, and it is finally working.

There were _many_ times during the development of this project when I figured
it would probably be deployed within the next few days. At those points, I
told my manager to begin finding me something interesting to work on next. He
always sounded more skeptical than I expected about the project actually being
nearly done. On each of those occasions, I realized within the next few hours
after announcing my near-completion, that there was still, in fact, a _slew_
of components still needing to be worked out before the software could be
deployed, and that I had _no idea_ how to craft any of these components.

There are _too many_ guides out there about how to develop and deploy software
systems. For me, the most important ended up being [a Medium article][stbl] by
Jeser L. Andersen (no idea who that is). I think I found the article on Hacker
News a month and a half ago. I will summarize my takaways from it

* The developer is responsible for every aspect of the software
* Make sure the project makes sense to tackle
* Choose the right libraries, frameworks etc. to use
    * This requires a small amount of experimentation which should be done up
      front in a separated/extracted code-base
* Limit scope
    * E.g. create a list of problems you're _not_ solving
* Limit external dependencies
* Create a transition plan from the previous system to the new one
* Design for future extension
* Prefer correctness and elegance to efficiency of execution
    * This is especially true for my project because it is network bound anyway
    * But still, understand which parts must be more correct than others
* Collect metrics about system performance
* Write automated tests of whether
    * The components that need to do something correctly are
    * The different components of the system can communicate with each other
      effectively

[stbl]: https://medium.com/@jlouis666/how-to-build-stable-systems-6fe9dcf32fc4

There are still aspects of my system that are not perfect. For example, I
occasionaly see errors about a timeout in some library used by a library I am
using. I don't know whether those errors are corrupting my data, or killing
JVM threads, or whatever else. The system is already a vast improvement over
what was in place before in terms of correctness and latency of the data
produced, so having the occasional issue was not a good enough reason to delay
deployment any further.

The system was deployed in a manner one might describe as "by hand". I rsync
the code onto a server, build the code on the server, open up two `tmux`
windows, run one JVM which produces to Kafka, run another JVM which consumes
from Kafka, make sure everything that is _supposed_ to be happening _is_
happening, and detach from tmux. According to the blog post I linked above, it
is desirable to be able to hit a button and see it deploy itself
automatically. In my case this conflicts with the other suggestion in that
article, which is to stick to limit the number of things you have to learn in
order to get the first version of your software working, which I believe the
author meant to be more important than a one-click deployment scheme.

The system is not exactly "done" in the sense that it doesn't have all the
features it is supposed to have. But at least I know now that the system works
in its production environment and is able to interact successfully and
(demonstrably) correctly with all the external infrastructure that it relies
upon. But to be sure, many corners were cut, many pieces are _kind of_ there
but not really, the metrics are not collected well, and configurations (e.g.
of Kafka, ZooKeeper, and Elasticsearch) were chosen on a "that'll probably get
us running" basis. As it turns out, there have already been issues with
Elasticsearch not being able to handle the load of multiple of my colleagues
querying it at once.

A problem with having deployed it without a larger share of the issues worked-
out is that I now need to be more careful to make my changes behave as
transitions from "working state" to "working state". There are pieces of the
system that if they are left not-working for more than 24 hours, data will be
irretrievably lost, and people will be sad. But by design of the software, the
piece that _must_ be left running properly is mostly disentangled from the
rest of the system, so that it can run while the rest of the system is
offline, and none of the data will be lost. This is accomplished by buffering
data in Kafka.

Since I have no previous experience deploying a software system like this, I
did not have a good basis to evaluate which libraries to build my system upon,
except for the buzz that I see on Hacker News, and my own intuition and
background. I like the Scala programming language best, and because of Spark,
some people I work with already use Scala, so I chose Scala. Is Go a better
tool for the job? According to the relative buzzes of the two ecosystems on
Hacker News, maybe it is. Would Docker have provided a better mechanism for
deployment? Most-likely it would have, but that can be added at any time if it
turns out that deployment is a _problem_ which needs to be _solved_, which at
this simple stage, it is not.

It is exciting to final have real people relying on code that I wrote. I am
excited to continue making this system better-meet the needs of its users. I
am even more excited by the shift in the way I look at writing software now
that I have seen more of the lifecycle of a single project.
