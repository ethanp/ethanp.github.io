---
layout: post
title: "Python Class Equivalence"
date: 2014-06-29 17:32:24 -0700
comments: true
categories: [Python, OOP, reflection]
---

## `vars(self) == vars(other)`, it seems too easy

I implemented a class

```python
class A(object):
    def __init__(self):
        self.a = 123
        self.b = 436
```

Then when I created two equivalent instances and compared them, I got false.

```python
>>> a = A()
>>> b = A()
>>> a == b
False

>>> c = a
>>> c == a
True
```

This must be because the `==` operator performs reference equality by default
which is what I should have expected, so I implemented a simple equivalence operator

```python
def __eq__(self, other):
    return self.a == other.a and \
           self.b == other.b
```

Then I figured there must be an easier way to do something so simple. So I
looked for what **reflection** methods are available on my new class, and
found the method `vars(object)`, which for this thing would return

```python
{'a': 123, 'b': 436}
```

This is just what I needed for a very simple equivalence operator

```python
def __eq__(self, other):
    return vars(self) == vars(other)
```

Unfortunately, `vars(a)` doesn't just work on *everything*:

```python
>>> vars([1,2,3])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: vars() argument must have __dict__ attribute
```

So I'm assuming `vars(a)` just calls `a.__dict__`.

I looked on Google and GitHub and didn't find too many examples of people
doing this, but one example said

```python
def __eq__(self, other):
    return self.__class__ == other.__class__ and \
           self.__dict__ == other.__dict__
```

So, until I find a reason not to, my default equality implementation shall be

```python
def __eq__(self, other):
    return self.__class__ == other.__class__ and \
           vars(self) == vars(other)
```

Assuming this is semantically what is desired of class equality, is there any
reason this is a Bad Way to implement it? Why has no one told me about this?
It seems too easy, but I guess sometimes things are just simple.
