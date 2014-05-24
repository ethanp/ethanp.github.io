---
layout: post
title: "Java inheritance"
date: 2014-05-23 14:06:59 -0700
comments: true
categories: [Java, inheritance, Comparable, Scala]
---

I spent quite a while struggling with the Java class declaration syntax, and
now I am starting to get it.

### Problem simplification

I started out writing this post because I was *totally right* and the Java
compiler was *wrong*, and so on, but now I have cleared it up and I was wrong
and the compiler was right after all (surprise!). I was initially intending to
use this post as a way of getting satisfaction out of how much smarter I am
than the compiler, and instead I will use this space to explain to the next
how to answer similar questions. I started by *heavily* simplifying my issue
down to its core, and attacking the simplified problem instead of the original
one with extraneous variables and interconnections. I've never approached a
programming problem like that, and let me tell you: seems like it worked. By a
twist of fate, the new version looks like a LinkedList, that's not what I
started out doing, but who knows what'll happen when you keep smushing the
problem from a larger context into a little jar.

First, we have a `Base`, which holds an element `elem` of any type, and a
reference to another `Base`, quite like a *linked list*.


``` java Base.java
class Base<T> {
    T elem;
    Base<T> next;

    Base(T elem) {
        this.elem = elem;
    }
}
```

<!-- more -->

Next, we have an `Ord`, which is a `Base` that is `Comparable`. We require
that the element contained by an `Ord` is itself comparable, and furthermore
that the method of comparing `Ord`s is to compare their respective `elem`s:

``` java Ord.java
class Ord<U extends Comparable<U>> extends Base<U> implements Comparable<Ord<U>>
{
    Ord(U elem) {
        super(elem);
    }

    @Override
    public int compareTo(Ord<U> o) {
        return this.elem.compareTo(o.elem);
    }
}

```

This works fine, but note the following limitation:

``` java main.java
public static void main(String[] args) {
    Ord<Integer> a = new Ord<Integer>(3);
    Ord<Integer> b = new Ord<Integer>(4);

    a.compareTo(b);         // this works just fine

    a.next = b;

    // this does not work because a.next is of type Base<Integer>
    a.compareTo(a.next);

    // this DOES work though because a.elem is of type Integer which is Comparable
    a.elem.compareTo(a.next.elem);
}
```

After writing and re-writing this blog post over and over, this makes sense
now. I used to be confused: "shouldn't the compiler *know* that a.next is an
Ord<Integer>?" And the answer is (now obvious) "Of course not." So I had the
following code, which I considered a "workaround", where we inform the
compiler that `next` is an `Ord` not a `Base`.

``` java Ord.java
class Ord<U extends Comparable<U>> extends Base implements Comparable<Ord<U>>
{
    Ord<U> next;

    Ord(U elem) {
        super(elem);
    }

    @Override
    public int compareTo(Ord<U> o) {
        return this.elem.compareTo(o.elem);
    }

```

It is now quite clear that there is no way the compiler would know your
intention of having the `next` component of descendents of `Base` be (static)
instances of their own respective classes. In fact, I'd bet most of the time
that's *not* what people want. But I was blind, and I believe I now see.

## Let's talk about Scala

Here is the (rough) equivalent to *all of that* (second Java version) in Scala:

``` scala InheritanceTest.scala

class Base[T](val elem: T, val next: Base[T])

class Ord[U <% Ordered[U]](override val elem: U, override val next: Ord[U])
                        extends Base[U](elem, next) with Ordered[Ord[U]] {
  override def compare(that: Ord[U]) = this.elem compare that.elem
}

object Run extends App {
  val a = new Ord(3, null)
  val b = new Ord(4, null)
  println(a < b)
}

```

The funny thing about this Scala version is that Intellij actually indicates a
compiler error, but SBT is quite happy to (correctly) compile it and produce a
result. This is one of those *opportunities* to spend forever looking for the
reason it won't compile, only to find that it compiles just fine.

Note that in the Scala version we were forced to say `override`, which is
*great* because in Java there is no indication that this is an overriden
field, and no resulting check of whether the override was successful. Of
course, in this example, no big deal. But what if there were a bunch of
instance variables that needed to be overridden, and they were private with
getters and setters, and split across multiple classes, etc.?

The other thing is that in the Scala version we must use a "view bound" `<%`
rather than my first thought, an "upper type bound" `<:`. (I put the English
names of those operators so you can google them. Good luck googling `<:`
alone!) In any case, it seems to me that `[A <: B]` means that `<A extends B>`, while
`[A <% B]` means that *either* `[A <: B]` *or* `A` has some implicit conversion defined to some type `C` s.t. `[C <: B]`, though I'm not 100% about that. In our Scala example above `<%` is necessary because (for example) it is **not** the case that `[Int <: Seq[Int]]`
[because Int has a primitive represenation][om-nom-nom].

[om-nom-nom]: http://stackoverflow.com/questions/16001010/why-does-int-not-inherit-extend-from-orderedint

## Extravagant insight

So in response to my confused pleas for "a clean solution" (before my epiphany
for why-it-must-be-so) my roommate came up with a clever scheme to configure
the type of the `next` pointer within the class declaration:

``` java Base.java
public class Base<T, E extends Base> {
    T elem;
    E next;

    Base(T elem) {
        this.elem = elem;
    }
}

```

``` java Ord.java
class Ord<U extends Comparable<U>> extends Base<U, Ord<U>>
                            implements Comparable<Ord<U>> {
    Ord(U elem) {
        super(elem);
    }

    @Override
    public int compareTo(Ord<U> o) {
        return this.elem.compareTo(o.elem);
    }

    public static void main(String[] args) {
        Ord<Integer> a = new Ord<Integer>(3);
        Ord<Integer> b = new Ord<Integer>(4);

        // still works, as before
        System.out.println(a.compareTo(b) < 0);

        a.next = b;

        // NOW WORKS, before it didn't!
        System.out.println(a.compareTo(a.next));

        // still works, as before
        System.out.println(a.elem.compareTo(a.next.elem));
    }
}

```

For whatever reason, when we put it into my existing code it didn't work and
we were both totally confused because it makes perfect sense. But now, when I
put it into my way simplified version, it works! So his idea just went from
being "just clever" to being "applicably awesome".
