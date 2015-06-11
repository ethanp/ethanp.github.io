---
layout: post
title: "What Akamai does"
date: 2015-06-10 19:23:31 -0700
comments: true
categories: [papers, Akamai, networking, internet]
---

I met someone at a hack night worked for Akamai. He was a cool dude, so I was
curious what Akamai does. For whatever reason I ended up reading a paper cited
as so:

> Nygren, Erik, Ramesh K. Sitaraman, and Jennifer Sun. “The akamai network: a
> platform for high-performance internet applications.” ACM SIGOPS Operating
> Systems Review 44.3 (2010): 2–19.

For whatever reason, everything I know about Akamai is from this paper. I'm not
in any way afiliated with Akamai, etc., though the paper is basically a badly
designed advertising pamphlet for their services. A small number of sentences
in this summary may have been lifted directly from that paper. You can consider
this whole post a paraphrasing of the paper. **The intent of this post is to
tell you everything important that I learned from the paper** in a form that is
*way* easier to digest than you reading the paper itself. To that end, this
document is maybe 2 pages long, and the original is 17. Clearly much of the
detail has been removed, but the gist remains. If that appeals to you, welcome
aboard! Note that the paper is from 2010, so is probably already out of date.

### Introduction

Akamai invented the *Content Delivery Network* (CDN) concept in the 1990s,
because the raw naive Internet implementation is too slow to conduct a global
business. As of 2010, Akamai delivers 15-20% of global Web traffic.

#### The Interwebs is a dangerous place

<!-- more -->

For a web app, downtime and high latency is *very* costly. Customers won't go
to your page unless they can see your content *right this second*. The Internet
itself can provide no reliability or performance gaurantees.

The Internet is composed of 1000s of individual networks, the largest having
only 5% of Internet access traffic. It takes 650 networks to get to 90% of all
access traffic. The Internet-access pricing structure happens to result in a
situation in which connections between networks ("peering points") are
bottlenecks causing packet loss and increased latency.

The protocol ISPs use to exchange rounting information (BGP) is bad and subject
to misuse. Internet outages and partitions are regular occurrences. The strict
ACK policy for TCP adds enough overhead to make video streaming over TCP
impractical. The high proportion of people still using IE6 means any improved
algorithms that get implemented have to be backwards compatible with that one
silly browser.

### Content Delivery Networks

Originally a CDN's purpose was to cache static site content at the "edge" of
the Internet, close to end users, to avoid middle-mile bottlenecks. Now they
accelerate entire web apps and provide HD live streaming media. They provide
security, logging, diagnostics, reporting, and management tools

A **delivery network** is a *virutal network*, i.e. a software layer over the
actual Internet, to provide reliability, performance, scalability, and
security. It requires no client software or changes to the underlying networks.
Akamai's network is made from tens of thousands of globally deployed servers
running sophisticated algorithms to enable faster content delivery.

#### In Greater Detail

When a user enters a URL into their browser, their DNS routes them to an Akamai
DNS, which in turn gives them the IP address of an **edge server** chosen via
machine learning algorithm. The edge server acts as a cache for the **origin
server**, and if it needs new data, it uses an especially reliable and
performant **transport system** (described later on), which is a network of
internal nodes who know best how to connect the *edge* back to the *origin*.

The **transport system** connects *edge* to *origin* servers with a cache
pyramid of clusters. Streaming video is *branched out* to the edge clusters by
intermediate layers of *reflectors*.

Edge servers communicate to each other via their own chosen path optimization
rather than the default one chosen by BGP, and use dynamically optimized TCP
parameters. They can also intelligently prefetch and cache content from the
origin right before before it is requested by the user.

Akamai's EdgeComputing product allows you to distribute your web application
*itself* "to the *edge*", which is great for content aggregation, product
catalogs (dynamic access of static content), data validation, and data input
batching. It's not great for apps relying heavily on transactional databases
because they still must communicate with the origin server.
