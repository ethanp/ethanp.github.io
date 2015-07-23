---
layout: post
title: "Inside Java's BufferedReader"
date: 2015-07-01 13:22:50 -0700
comments: true
categories: [source code, Java, buffering, I/O, library, performance, explanation]
---

`BufferedReader` is suprisingly fast for parsing large text files. Why is that?
In my experience, it is faster for this task than a `BufferedInputStream`.
StackOverflow says this is because the `BufferedReader` uses `char` internally
instead of `byte`.

What follows is a high-level breakdown of what's going on "under the hood" of
the `BufferedReader`, i.e. an overview of the implementation details.
Details about `mark` support will be omitted.

For me, the most significant takeaways are the following

1. **There is no magic** -- every time I find this out about something I am
   surpised. This class is probably *very* similar to the way I would have
   naively written a buffering wrapper for a `Reader` object.
2. **Utmost efficiency is sacrificed for code clarity** -- my guess is that the
   reason they left in the inefficiencies I mentioned above is because having
   special cases would have clouded the code. If someone wants an even *more*
   efficient `Reader` they are always welcome to write their own.
3. **Small amount of code** -- there's really not a whole lot to this class. It
   basically just reads into a buffer, then services incoming reads from that
   buffer. There are no "niceties" or asynchronous callbacks etc. I think the
   attitude of the auther is that if you want to find that, simply look
   elsewhere.
    * Of particular note: there is no a single mention of *character encodings*
      anywhere in the class.

<!-- more -->

Starting with the obvious, a `java.io.BufferedReader` is `java.io.Reader` that
holds a buffer to make operations faster. A `Reader` is an object that reads
character streams. Any subclasser of the `abstract class Reader` must implement
(at a minimum) `read(char[], int, int)` and `close()`. Other reader methods
basically just use these methods to put together the rest of the interface. The
only public method that `BufferedReader` adds to the `Reader` interface is
`String readLine()`, which is described below.

Let's look at some of the fields, with my annotations about what they actually
mean.

### Reader in

```java
private Reader in;
```

This is the `Reader` whose reads we are buffering. `BufferedReader` is an
example of the *"Decorator Pattern"* and this `Reader` is whom we're
decorating. Both `BufferedReader` constructors require `in` to be passed as a
parameter, and it cannot be changed. Calling `this.close()` calls

```java    
in.close();
in = null;
```

Setting `in = null` might be to help prevent garbage accumulation when the
client of `BufferedReader` doesn't properly get rid of the reference to the
`BufferedReader`.


### char[] cb

```java
private char[] cb = new char[8192];
```

This of course is the char buffer itself. We respond to read requests directly
from here as often as possible. We fill it up by issuing `in.read(cb, int,
int)` requests on the underlying `Reader in` annotated above.

Here I'm showing *default* internal char buffer size of 8kb. This happens to be
2 disk sectors of an HDD using the first generation of the Advanced Format. I
don't know if that's how 8kb was chosen; perhaps it was chosen using benchmark
comparisons.

### int nChars

```java
private int nChars
```

This is the index in `cb` corresponding to the last unread `char` that we may
`read()` before we need to `fill()` it again from `in`.

### int nextChar

```java
private int nextChar
```

This is the index in `cb` corresponding to the next `char` that should be
returned by a `read()`.

### void fill()

```java
private void fill() throws IOException;
```

Here we block until successfully reading from `in` into `cb`, and set `nChars`
to the end of the valid range of `cb` and `nextChar` to zero.

### int read()

```java
public int read() throws IOException;
```

Returns a single character of the stream, or -1 if the end of the stream has
already been reached.

Basically just does

```java
return cb[nextChar++];
```

However if `nextChar >= nChars`, we have already read to the end of the
existing `cb`, so we call `fill()`. Now if `fill()` fails, we've reached the
end of the stream, so `return -1`.

### int read(char[], int, int)

```java
public int read(char cbuf[], int off, int len) throws IOException;
```

This entire method is `synchronized` behind a `lock` owned by the parent
`Reader` class. In `Reader` it says to use this `lock` instead of synchronizing
on `this` "for efficiency". I'm not sure why that would be any more efficient.

After acquiring the lock, we do bounds checks. I'm not sure why this isn't done
*before* acquiring the lock because all checks are done on the parameters,
which can't change between the time the method is called and the lock is
acquired. So if we did this check before acquiring the lock, we would never
wait on the lock for no reason. Maybe this is so that any concurrent operation
using the lock gets to finish before the exception is thrown on this thread?
That's my best guess at this point.

The documentation says 

> As an additional convenience, it attempts to read as many characters as
> possible by repeatedly invoking the `read` method of the underlying stream.

This is accomplished by first copying the rest of the current buffer into the
passed-in `cbuf`. If this has not yet filled `cbuf`, keep calling `fill()` and
then dumping the new buffer contents into `cbuf`. Seems to me there is an
**extra copying step** into the buffer in this case, where it would perhaps be
faster and more memory efficient copy *directly* into `cbuf`. But hey, what do
I know.

### String readline()

```java
public String readLine() throws IOException;
```

Returns the next segment of text from the stream terminated by `\n`, `\r`, or
`\r\n`.

This method operates inside a synchronized endless loop building up a
`StringBuffer s`.

The loop is like this:

1. `fill()` the underlying `cb` buffer
2. If the stream ended
    * `return s.toString()`
3. Otherwise, iterate through `cb` looking for a line ending
4. If a line ending is found
    * append `cb` through the line ending to `s` and return it.
5. Otherwise, repeat the loop.

### long skip(long)

```java
public long skip(long n) throws IOException;
```

In a synchronized loop:

1. If the buffer has all been read `fill()` it
    * **Why?** Why copy memory into the buffer unecessarily? We're just going
      to skip some or all of these bytes!
2. If the buffer is larger than the number of bytes remaining to skip
    * Skip that many bytes into the buffer and return
3. Otherwise skip over the entire buffer and repeat the loop

### boolean ready()

```java
public boolean ready() throws IOException;
```

> Tells whether the stream is ready to be read

After acquiring the lock, return try if either the buffer still has unread
data, or `in.ready()` is true (on the underlying `Reader in` described above).

### void close()

```java
public void close() throws IOException;
```

Inside the lock, close and release the underlying `Reader in`, and release
`cb`.

### Conclusion

Those are all the important methods of `BufferedReader` that don't involve
`mark` support.

At this point the reader may have a somewhat deeper understanding of how
`BufferedReader` manages to achieve such high speeds. I also pointed out
potential inefficiencies in `read(char[],int,int)` and `skip(long)`, as well as
an seemingly unecessary block on a lock before bounds checking in
`read(char[],int,int)`.

You may want to refer back to the takeaways in the introduction.
