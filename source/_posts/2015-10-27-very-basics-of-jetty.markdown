---
layout: post
title: "Very basics of Jetty"
date: 2015-10-27 11:09:50 -0500
comments: true
categories: [Jetty, java, web, frameworks, tools, libraries, http, www]
---

### Jetty has _cutting edge_ HTTP/2 support

For my Wireless Networking project, I'm going to compare HTTP/1.1 and HTTP/2
with respect to performance metrics when talking to a mobile phone over the
cellular network. There are not so many [implementations][impl] of HTTP/2 right
now, and some of them seem a bit shaky. At the end of the day, it seems to me
that the easiest way to run this sort of experiment in a reliable fashion is to
use Java's [Jetty][jetty] project. It has well-tested HTTP/1.1 support, many
heavyweight framework users, implements HTTP/2's server push and all sorts of
HTTP/2 negotation mechanisms, and I like static types.

[impl]: https://github.com/http2/http2-spec/wiki/Implementations
[jetty]: http://www.eclipse.org/jetty/

### Jetty is hard to find tutorials for

So I need to learn the basics of Jetty; and there's a lot to learn, and I
haven't found a stellar resource, so I've been reading the [embedded
examples][emb], which are decent. I'm usng "embedded" Jetty because that means
I can write type-safe Java rather than XML. Perhaps XML would be a good choice
for a long-standing app, but I'm making a prototype and it's easier to just
write code.

Here's a brief overview of what I've learned so far, which may come in handy
for someone else wanting to understand the very basics of Jetty (version 9).
This is not a detailed overview because I don't know what I'm talking about.
I'm just explaining the things I have learned from the tutorial linked above
and some mucking around.

[emb]: http://www.eclipse.org/jetty/documentation/current/embedding-jetty.html

<!-- more -->

### Jetty has a few large architectural components

The main things you need to deal with in Jetty are `Server`s, `Handler`s, and
`Connector`s.

The `Server` manages your entire web site. It does this by connecting
`Connector`s to `Handler`s. The default `Connector`, `ServerConnector` responds
to vanilla HTTP/1.1 requests. If you try to make a request via `https` to a
`ServerConnector` without `https` support, in Chrome you will see the following
error message

```text
ERR_SSL_PROTOCOL_ERROR

Unable to make a secure connection to the server. This may be a problem with
the server, or it may be requiring a client authentication certificate that you
don't have.
```

#### Handler

You may want a chain of `Handler`s that will be tried one after another until
one is found to be appropriate for handling the current request

```java
HandlerList handlers = new HandlerList();
handlers.setHandlers(new Handler[] { resource_handler, new DefaultHandler() });
```

The `DefaultHandler` will simply return `404`s for everything. Jetty will keep trying `Handler`s in the list until one of them performs

```java
baseRequest.setHandled(true);
```

Looking at the `DefaultHandler`'s source, we can see that indeed, in the
`handle` method, this is the _first_ thing it does.

#### Connector

Each `Connector` instance can respond to a particular _protocol_ at a
particular _host_ on a particular _port_. So if you want to respond to `https`
requests as well as vanilla `http`, you're going to need another `Connector`.

#### Routing

A `ContextHandler` is used to do what in Rails would be called "routing". This
is where you associate path's in the HTTP request header to the right
controller. After you've created your `ContextHandler`s and set their
`contextPath`s, you need to do the following

```java
Server server = new Server(8080);
//...
ContextHandlerCollection contexts = new ContextHandlerCollection();
contexts.setHandlers(new Handler[] { context0, context1, ... });
server.setHandler(contexts);
```

This means the server is going to ask the `ContextHandlerCollection` for the
first `ContextHandler` that matches the path specified on the incoming request.

### Conclusion: Jetty feels dated compared to Ruby on Rails

So far, I find Jetty surprisingly nice. I'm
starting to appreciate the fact that it is written in Java and it is open
source. It's so different from Ruby on Rails and Node.js because there is not a
lot of magical mystery code running in the background whose source is difficult
to find. The advantage of those other frameworks is that they have much higher
level APIs. I find myself wrapping Jetty methods into my own methods that do
what in Rails would have been done automatically. Now I understand the draw of
Rails's "convention over configuration" and "don't repeat yourself". I didn't
realize how much configuration and repeating yourself was necessary back in the
Java days. And that leads me to my next point, that writing Jetty feels like
writing 10 year old code. I wasn't coding 10 years ago, but I imagine you had
to explicitly wire every last thing together by hand over and over again, as
you do in Jetty.

Unlike Rails or iOS, basic Jetty is not a Model View Controller framework. It's
just handlers on a server, more like Node.js. Personally, I like MVC out of
ignorance because I assume people who came up with it thought the "application
framework" way of doing thigs was 'bad', and I tend to trust people who don't
like the 'old' way of doing things. That said, I am not making a database based
application here so I will not evaluate this difference any further.
