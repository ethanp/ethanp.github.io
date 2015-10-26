---
layout: post
title: "Intro to HTTP/2"
date: 2015-10-26 16:11:59 -0500
comments: true
categories: [http, http2, internet, www, networking, tcp]
---
## The problem of HTTP/1.1

### HTTP/1.1 is from an earlier Web

For the past several years, Google, Mozilla, Akamai, the IETF, the academic
research community, and others have been engaged in efforts to reduce PLTs
experienced by users of the WWW. One bottleneck in apparent need of an update
is the now-"ancient" HTTP/1.1 (H1) protocol, published in 1999.

From the beginning, one of the major design goals of the original HTTP protocol
was simplicity to implement and adopt, to encourage growth of the WWW.
Evidently, this technique worked. However, over the past 16 years, the way
people create, distribute, and view pages on the WWW has changed drastically.
Pages today have far more Javascript, CSS, images, and other content to go
along with the vanilla HTML. In the course of this evolution, numerous issues
with H1 have come up, mostly pertaining to the number of sequential round-trips
it requires to fully download the data for each web page. These round-trips
introduce unnecessary latency.

<!-- more -->

Over the past 16 years, countermeasures, "optional" protocol alterations, and
application level workarounds have been suggested and implemented to reduce the
round-trip count and therefore latency of pages retrieved using H1.

### HTTP/1.1 PLT Optimizations

Some optimizations servers and browsers use to lower PLTs seen by clients are
the following:

1. Opening multiple TCP connections to request the download of multiple
   required web page resources in parallel.
2. Inlining and concatenating scripts and stylesheets to reduce the number of
   requests.
3. _HTTP pipelining_, in which multiple requests are sent by the browser over a
   single TCP connection before waiting for responses to any, then the server
   sends all of the responses back in the same order.

HTTP pipelining was an idea that did not work out. It seemed like a logical
approach to improve HTTP performance by reducing latency. However, in reality,
HOL blocking became an even greater issue then it already had been, and it also
led to issues with older proxies that are difficult to update. Nowadays, modern
browsers do not enable pipelining.

## HTTP/2 aims to address these issues

The main reason multiple resources can't be _multiplexed_ (downloaded in
parallel) over a single TCP connection in H1 is because it is an ASCII
protocol. Being an ASCII protocol means that there is no easy way to specify
how to demultiplex those responses with a parser.

To address this, HTTP/2 (H2) is a _binary_ protocol that _can_ easily multiplex
multiple requests and responses through a single TCP connection. It does this
using a new _binary framing layer_, in which responses are broken into separate
_frames_ and interleaved through the TCP socket.

Web designers have discovered that to yield the best user experience, the
server should transmit certain most-important resources first, as soon as they
are available to send (e.g. loaded from disk). H2 frames (and _streams_) have
an (optional) `priority` field to allow the application developer (and server
writer) to bias the ordering of frames sent through the socket. For example,
one may want to ensure that `.html` files have a higher priority than `.jpeg`
files because `.jpeg`s contain a lot of data and are not as crucial for showing
the user a _barebones_ version of the page they requested.

### Brief History

The current H2 specification (RFC 7540) is based primarily on the SPDY protocol
invented and first used at Google. Google is in a rather unique position to
conduct research on this topic because they write the code running large
proportion of both user's browsers, _as well as_ the servers for a large
portion of the web pages those people visit. This allows them to run real
controlled experimental studies on web traffic, to try to guage the best way to
speed up the Web.

Many have voiced concerns that Google's ideas are welcomed to quickly by the
standards committee. However, supporters are eager to get the ball rolling on
any good ideas, and point to the fact that Google already has _results_.

Note that some of the experimental results below were actually obtained using
SPDY rather than H2, but we will not concern ourselves with that because they
are the same in essence. Google Chrome still supports SPDY, but has claimed it
will remove support in 2016 so that there are not two competing
standardizations of basically the same protocol.

In reality, in my Chrome browser, using the "HTTP/2 and SPDY indicator"
extension, I can see that Google's search results and YouTube videos are served
over SPDY/3 running on top of their still-in-research QUIC protocol which is
meant to replace TCP as the transport layer protocol underneath HTTP. The
"indicator" reveals that many websites are in fact already being served over
H2, including Twitter. I hope to explore what exactly QUIC does, promises, and
delivers further in my research if there is time and space for it.

### Experimental Results

Experiments investigating relative performance differences between H1 and H2
have yielded contradictory results (Wang et al.). In (Wang et al.), they found
that H2 generally outperforms H1, except when the task is to transmit _large_
objects over a _high-loss_ connection. One of the goals in defining H2 is that
it is supposed to be easy to swap-in in-place of H1. This finding indicates
that especially with respect to mobile phones, the H1 optimization techniques
of inlining and concatenating web page resources is the wrong way to
improve performance for H2.

### HTTP for Mobile

With respect to mobile devices, there are multiple problems with HTTP that are
_not_ addressed by H2 (Erman et al.).

First of all, TCP congestion window part of congestion avoidance (`cwnd` and
`ssthresh`) makes it so that dropping a single TCP datagram leads to a
drastically reduced throughput. This is because TCP assumes the packet was
dropped due to network congestion, when in reality, it could have been for any
of the wireless-specific issues (multipath propagation, etc.)

Secondly, bad interactions between the 3G & LTE MAC state machine timeouts and
modern default TCP timeouts and mechanisms lead to datagram _retransmissions_
with further resets on congestion window state variables, resulting in a
drastically negative impact on throughput.

### Research is needed

From the system administrators' perspective, it is still difficult to decide
whether enabling H2 on one's servers is going to have a positive impact on
performance _at all_ (Varvello et al.). The following questions still do not
have good enough answers to make answering that decision easy.

1. What changes to the infrastructure and optimization pipeline are needed to
   provide a significant benefit over H1?
2. What are the best algorithms for leveraging H2's new capabilities for
   _server push_ and _prioritization_?
3. Why do experiments by different research groups yield such a wide disparity
   in results?
4. Are we unlikely to see any real PLT improvements until we improve the
   underlying transport layer protocol?
5. In what ways is the problem different for mobile? 
    * What changes need to be made in the mobile-specific scenario, viz. where
      interference, low bandwidth, high latency, and battery life
      considerations are major concerns?

## Glossary

* __IETF__ (Internet Engineering Task Force) -- a standards organization for
  the Internet, which produces "RFC"s (requests for comment) specifying some of
  the crucial internet protocols, such as HTTP, TCP, and TLS.
* __PLT__ (Page Load Time) -- the duration between the time at which a user
  submits a request for a web page, and the time at which she receives the last
  byte of data needed to represent that web page correctly
* __WWW__ (World Wide Web) -- a decentralized collection of resources,
  requested and transmitted using _HTTP_
* __HTTP__ (HyperText Transfer Protocol) -- a protocol specifying the way an
  HTTP _client_ may upload, download, update, etc. documents on an HTTP
  _server_. Often used for downloading _HTML_ in particular.
* __TCP__ (Transmission Control Protocol) -- a protocol which provides an
  application the abstraction of having a reliable pipe across the Internet
  into/from which it may send/receive ordered sequences of bytes
* __HOL blocking__ (head-of-line blocking) -- when multiple messages are being
  sent, but due to imposed FIFO ordering, messages ready to be sent must wait
  for an earlier message which is taking a long time

## References

* Erman, Jeffrey, et al. "Towards a SPDY'ier mobile web?." _Proceedings of the
  ninth ACM conference on Emerging networking experiments and technologies.
  ACM_, 2013.
* Grigorik, Ilya. High Performance Browser Networking: What every web developer
  should know about networking and web performance. _O'Reilly Media, Inc._, 2013.
* Roth, Gregor. "HTTP/2 for Java Developers". Retrieved from
  http://www.javaworld.com/article/2916548/java-web-development/http-2-for-
  java-developers.html on October 8, 2015.
* Stenberg, Daniel aka. bagder. _http2 explained_, retrieved October 8, 2015;
  downloaded as MOBI.
* Varvello, Matteo, et al. "To HTTP/2, or Not To HTTP/2, That Is The Question"
  arXiv preprint arXiv:1507.06562 (2015).
* Wang, Xiao Sophia, et al. "How speedy is SPDY." _Proc. of the 11th USENIX
  Symposium on Networked Systems Design and Implementation (NSDI)_. 2014.
