---
layout: post
title: "Some Java Network Programming Fundamentals"
date: 2016-02-28 15:15:13 -0800
comments: true
categories: [java, networking, programming, fundamentals, nio, TCP, UDP, TCP/IP, buffers, streams, I/O]
---

Most of what I've learned and discussed here comes from _TCP/IP Sockets in
Java_, a highly recommended book about this stuff by Calvert and Donahoo. Some
of it also comes from _Java Network Programming_ by Elliotte Rusty Harold.

## Overview

* The *only* transport-layer protocols Java supports are TCP & UDP; for
  anything else, you must link to native code via the Java Native Interface
* TCP uses *stream* sockets, through which one generally just writes to an
  `OutputStream` and reads from an `InputStream` of bytes that remain in-order
  and uncorrupted and are (practically) guaranteed delivery by the
  implementation of the protocol by the operating system.
    * Unless you're using NIO; see below for more on that
* UDP uses *datagram* sockets, through which you `send` and `receive` objects
  called `DatagramPacket`s, which are just a length, a destination, and data
* Unless you're using NIO, everything _blocks_: e.g. connecting to servers,
  listening for clients, reads, writes, and disconnecting (for TCP)
    * By default most of these actions may block _indefinitely_
    * For reading and connecting, you can configure a timeout, after which you
      will receive an `InterruptedIOException`
    * For _writing_ to a TCP stream, you _cannot_ configure a timeout

### Handling multiple clients

* Deal with one at a time, which is simplest, especially if there's some state
  that is shared by all potential clients. Speed may become problematic
  quickly.

```java
void mainLoop() {
    while (true) {
        Socket s = serverSocket.accept();
        handle(s);
    }
}
void handle(Socket s) {
    InputStream in = s.getInputStream();
    // process request, etc.
    s.close();
}
```
* Create a new thread to handle each incoming client. This is still pretty
  simple, but will lead to massive overhead if you have many concurrent
  clients, and therefore you're context-switching all the time.

```java
void mainLoop() {
    while (true) {
        Socket s = serverSocket.accept();
        new Thread() {
            @Override public void run() {
                InputStream in = s.getInputStream();
                // process request, etc.
                s.close();
            }
        }.start()
    }
}
```

* Use a thread pool to handle requests. Java has abstracted the thread pool
  concept into the `Executors` factory class. There are a multitude of
  executors to choose from. This `newCachedThreadPool()` one will execute each
  task on an existing thread if one is idle, and will create a thread
  otherwise. Threads sitting idle in the cache for over one minute are
  terminated.

```java
ExecutorService executor = Executors.newCachedThreadPool()

void mainLoop() {
    while (true) {
        Socket s = serverSocket.accept();
        executor.execute(new TheHandler(s));
    }
}
static class TheHandler implements Runnable {
    Socket s;
    public TheHandler(Socket s) { this.s = s; }
    @Override public void run() {
        InputStream in = s.getInputStream();
        // process request, etc.
        s.close();
    }
}
```

* Use NIO (rather complicated) to allow `N` threads to service `M`
  clients, where `N` is small and `M` is huge. This uses event-based
  programming. We can set all network operations to be non-blocking, and
  only wait as long as we want to for them. An extensive example can be
  found below.
* Use a framework like Netty, Akka, etc. that wraps the NIO stuff up in a
  ribbon and a tie


### 10K feet above NIO

<!-- more -->

* If you're using NIO, you create `Channel`s of bytes into and out of sockets
  (or file handles)
* You register a `Selector` to be notified when the `Channel` is ready to be
  read from or written to
* You query the `Selector` to tell you which `Channel` are ready, and may then
  take action on those that are
* You get data in and out by passing a `Buffer` to the `Channel`

Here's an example based on TCP/IP Sockets in Java, a highly recommended book
about this stuff by Calvert and Donahoo.

```java
Selector slctr = Selector.open(); // factory
ServerSocketChannel chnl = ServerSocketChannel.open(); // factory
chnl.socket().bind(inetAddr); // set address to listen on

// For some reason Channels block by default. If we want to
// register with the Selector for notifications, we must turn
// that off.
chnl.configureBlocking(false);

// Notify Selector whenever this Channel has a new connection
// ready to be "accepted". Such a notification still does
// *not* guarantee it will work immediately.
chnl.register(slctr, SelectionKey.OP_ACCEPT);

while (true) {
    // wait configurable period of time to be notified
    // by any registered channel
    int numNotifications = slctr.select(timeoutMS);
    if (numNotifications == 0) {
        // We timed out without any notification.
        // We could do whatever we want here because we're
        // no longer blocked.
    } else {
        // numNotifications different channels have notified us of
        // being available for Connect, Read, Accept, or Write.
        // It is OK to use these keys in concurrent threads.
        for (SelectionKey key : slctr.selectedKeys()) {
            // We're not sure which channel this key belonged to.
            // Also, notification was just a "hint" and we need to
            // check again whether the Channel is available.
            if (key.isAcceptable()) {
                // here's the actual call to accept()
                SocketChannel clientChnl =
                    ((ServerSocketChannel) key.channel()).accept();

                // similar to the ServerSocketChannel
                clientChnl.configureBlocking(false);
                // Except that here we register to notify Selector
                // about being "readable", and
                clientChnl.register(
                    key.selector(),
                    SelectionKey.OP_READ,
                    // We must associate an "attachment" with this
                    // channel. This is the buffer that will be
                    // filled with the incoming bytes rcvd via TCP.
                    ByteBuffer.allocate(NUM_BYTES) // eg 256?
                );
            }
            if (key.isReadable()) {
                // retrieve the readable client socket's channel
                SocketChannel client =
                    (SocketChannel) key.channel();

                // retrieve the ByteBuffer we associated with
                // that channel
                ByteBuffer buf = (ByteBuffer) key.attachment();

                // Attempt to read `buf.remaining()` bytes _from_
                // the Channel _into_ the ByteBuffer.
                int bytesRead = client.read(buf);

                // -1 from read() means end-of-stream, which in
                // this case means the client closed their output
                // side of the TCP connection. We may still be
                // able to send data if that side of the connection
                // has not been closed yet.
                if (bytesRead == -1) client.close();

                else if (bytesRead > 0) {
                    // if our application has data to write back
                    // to the client, we must tell the selector
                    // that we've now become interested in writing
                    key.interestOps(SelectionKey.OP_READ
                                    | SelectionKey.OP_WRITE);
                }
            }
            // socket not closed, and is writable
            if (key.isValid() && key.isWritable()) {
                // beyond the scope of this post.
            }
        }
    }
}
```


## Tips for Traps

* Don't write to the network through a `PrintStream`
    * It chooses end-of-line chars based on your platform, not the protocol
      (HTTP uses `\r\n`)
    * It uses the default char encoding of your platform (likely UTF-8), not
      whatever the server expects (likely UTF-8)
    * It eats all exceptions into this `boolean checkError()` method, when
      you're better off just using the normal exception hubbub
