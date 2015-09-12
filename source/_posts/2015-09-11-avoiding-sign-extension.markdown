---
layout: post
title: "Avoiding Sign Extension"
date: 2015-09-11 22:12:42 -0500
comments: true
categories: [code, implementation, specifics, Java, bytes]
---

I was looking at an implementation of file-based mergesort [from GitHub](https://github.com/cowtowncoder/java-merge-sort), and found the following snippet.

```java
/**
 * Simple implementation of comparator for byte arrays which
 * will compare using <code>unsigned</code> byte values (meaning
 * that 0xFF is creator than 0x00, for example).
 */
public class ByteArrayComparator
    implements Comparator<byte[]>
{
    @Override
    public int compare(byte[] o1, byte[] o2)
    {
        final int len = Math.min(o1.length, o2.length);
        for (int i = 0; i < len; ++i) {
            // alas, sign extension means we must do masking...
            int diff = (o1[i] & 0xFF) - (o2[i] & 0xFF);
            if (diff != 0) {
                return diff;
            }
        }
        return o1.length - o2.length;
    }

}
```

What is the meaning of the remark "`// alas, sign extension means we must do
masking...`"? What's the deal with the masking? 

<!-- more -->

References

1. [Supressing sign extension in Java](http://stackoverflow.com/questions/5147738/supressing-sign-extension-when-upcasting-or-shifting-in-java)
2. [Promotion in Java](http://stackoverflow.com/questions/1660856/promotion-in-java)

`ol[i]` is a `byte`, i.e. 8 bits. But if you try to subtract two bytes

```java
byte a = 0x00;
byte b = 0xff;
byte c = a - b;
```
This will produce a __compiler error__, because Java's `-` (minus) operator
_promotes_ `byte`s to `int`s before _performing_ the `-`, thus `-` produces an
`int`, and you can't assign an `int` to a `byte` (without a cast, because you'd lose bits).

So what about

```java
byte a = 0x00;
byte b = 0xff;
int c = a - b;
```

The problem is that (according to the spec in the doc comment), we want `c` to
be _negative_. However note:

```java

// this
int c = 0x00 - 0xff;

// becomes this
int c = 0 - 0xffffffff; // 0 - (-1) = 1 "sign extension"
```

So now `c = 1` instead of `-256`.

```java
// But this
int c = (`0x00` & 0xff) - (`0xff` & 0xff);

// becomes this
int c = (`0x00` & 0x000000ff) - (`0xff` & 0x000000ff);

// which becomes this
int c = 0 - 0x000000ff; // 0 - 256 = -256 (avoided sign extension)

```
So because of the masking, we avoided sign extension, and now `c = -256`, which
is what we wanted.
