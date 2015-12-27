---
layout: post
title: "First Glimpse of Compilers"
date: 2015-12-27 00:49:00 -0600
comments: true
categories: [compilers, textbook, homeschooling]
---

I am close to two chapters into "Compilers" (a.k.a. "The Dragon Book"), by Aho,
Lam, Sethi, and Ullman. It is an exciting topic to learn about.

The most fundamental thing I have learned so far is the overall pipeline for
understanding the modular components forming the way a compiler is typically
constructed. It goes

1. Lexical analyzer
2. Syntax analyzer
3. Semantic analyzer
4. Intermediate code generator
5. _Machine-independent code optimizer_ (optional)
6. Code generator
7. _Machine-dependent code optimizer_ (optional)

The input to the compiler pipeline is a "character stream" of the program.
However I don't recall the book ever dealing with specifications of that
stream, except that it supports a generic `getchar()` operation. I can
understand that if the entire source consists of one file, you just read
character-by-character from the file into the compiler. Maybe the language
designer decides how multiple-file projects are to be read by the compiler,
i.e. it is beyond the scope of this book; or maybe they'll go more in-depth on
this later-on.

### The Lexical Analyzer

The character stream is read into first component in the compilation pipeline:
the __lexical analyzer__, which maps the _character stream_ into a _"token"
stream_. It seems like a __token__ knows its _tag_, and its _value_. The
__tag__ is basically the _type_ of this token in the eyes of the compiler.
Possible tags in their example include `NUM`, `ID`, `[keyword]`s, and I added
`COMMENT` as part of an exercise. The __value__ for a token might be the
literal _value_ of a literal; or it might be the name of a variable. More about
that below.

#### Symbol Tables

In addition to creating the token stream, the lexical analyzer builds the
_symbol table_. It seems to that the __symbol table__ maps names to places in
memory. A symbol table also 'by default' implements the language's preferred
form of _block scoping_. Upon entering a new scope, a new symbol table is
created that points to its "parent", the one it is nested inside of. If a
variable is _declared_, it is added to the symbol table to this scope. Later,
when a variable is _referenced_ (i.e. init'd, read, or updated), we will
consult the table for the current innermost scope. If the variable's data is
not found there, we will continue to check ancestors until we find it.
