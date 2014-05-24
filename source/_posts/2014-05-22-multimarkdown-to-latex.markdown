---
layout: post
title: "Markdown to LaTeX"
date: 2014-05-22 18:32:09 -0700
comments: true
categories: [installation, markdown, multimarkdown, latex, tex, fletcher penney]
---

### Installation guide for Mac OSX

One day, I thought, "Wouldn't it be nice to have a Markdown to LaTeX
converter." Theoretically, one could combine these two tools to quickly create
beautiful documents. That's how I found [Multimarkdown][] which is basically a
souped up version of regular markdown. The documentation made it sound like
this would be a breeze, but it actually took me way to long to get this to
work on my machine. So I made a step-by-step guide.

For the uninitiated, [Markdown][] is a simplified way of writing html markup.
If you don't know about it please check it out. It is in fact how this very
webpage was created.

Then today I had to install it all over again on a different computer. To
save my future self and you a lot of trouble, this time I noted what I did to get it to
work (Mac OSX only).

<!-- more -->

### Install a lot of software

First you must install [homebrew][] the lifesaving Mac package manager.

``` bash
ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
```

Then you should install multimarkdown et al. by running

``` bash
brew install multimarkdown pkg-config glib gettext
```

I'm not sure every one of those is necessary, but I don't have another
computer to test which ones can be left out. Probably at least the first one
is necessary.

Now go to the Mac App Store and buy [MultiMarkdown Composer][] for $12. Or
don't, it isn't actually necessary, but I like that it live-renders LaTeX
equations and has a hotkey for exporting markdown to different formats. You
can export using the command-line too and that's also quite simple.

Install [TeXworks][]. This is a gui for running LaTeX commands, and it's free,
but again you can use the terminal for anything you can do in here if that's
your thing.

### Get the base latex installation and multimarkdown template files

Download the [multimarkdown latex support][] files from github. And put them
in a (likely *new*) directory called `~/Library/texmf/tex/latex/mmd`.
Personally, I put the files in .../texmf/tex**t**/latex/... and that cost me a
half-hour of my life so do yourself a favor and just copy and paste the
following command

``` bash
mkdir -p ~/Library/texmf/tex/latex/mmd
mv myunzipped_latex_support_files ~/Library/texmf/tex/latex/mmd
```

Install [mactex-2013][]. For whatever amazing reasons, you can't use homebrew
for this. You have to go to that website, download a 4+ GB file and run the
installer.

### Create your first latex-ready markdown document

Now create a file Yayaya.md in Multimarkdown Composer and paste the following
header at the top, verbatim:

    latex input:        mmd-article-header
    Title:      Hello Dr. Fourier
    Author:     My Name
    Base Header Level:      1
    latex mode:     memoir
    Keywords:       Math, DSP, Digital Signal Processing, Fourier Transform
    CSS:        http://fletcherpenney.net/css/document.css
    xhtml header:       <script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
    </script>
    copyright:          2014 My Name
    latex input:        mmd-natbib-plain
    latex input:        mmd-article-begin-doc
    latex footer:       mmd-memoir-footer

That was just telling Multimarkdown how to format the LaTeX output. After that paste the actual contents of the document, e.g.

    # Simpler LaTeXing #

    ## The Fourier Transform ##

    **Sometimes** *this* formula comes in quite handy.

    ### The Formulation ###

    What follows is the formula for the Fourier transform.

    \\[FT\{f(x)\}:=\frac{1}{\sqrt{2\pi}}\int_{-\infty}^{\infty}\!f(x)e^{-iwx}dx\\]

Then go to `file->export-> "Export as: asdf" , "Format: LaTeX"`

This *should* have produced the following raw latex file:

    \input{mmd-article-header}
    \def\mytitle{Hello Dr. Fourier}
    \def\myauthor{My Name}
    \def\latexmode{memoir}
    \def\keywords{Math, DSP, Digital Signal Processing, Fourier Transform}
    \def\mycopyright{2014 My Name}
    \input{mmd-natbib-plain}
    \input{mmd-article-begin-doc}
    \part{Simpler LaTeXing}
    \label{simplerlatexing}

    \chapter{The Fourier Transform}
    \label{thefouriertransform}

    \textbf{Sometimes} \emph{this} formula comes in quite handy.

    \section{The Formulation}
    \label{theformulation}

    What follows is the formula for the Fourier transform.

    \[FT\{f(x)\}:=\frac{1}{\sqrt{2\pi}}\int_{-\infty}^{\infty}\!f(x)e^{-iwx}dx\]

    \input{mmd-memoir-footer}

    \end{document}

Open `asdf.tex` in TeXworks, hit the green "play" button in the top-left
corner. A bunch of garbage will pile up in the Console Output area, but then
your beautiful PDF will have been generated. This is just stellar, I'm telling
you. Now you can open the `asdf.pdf` file in your favorite pdf viewer.

### Bask in its glory

How about that. Hmm, indeed.

### Comprehensive troubleshooting guide

If in the console of TeXworks, you get something like `mmd-article-header.tex:
not found`, it means you put the multimarkdown-latex-support files in the
wrong place or you forgot to download them or something. Because I kept
getting that error message I was *convinced* there was some command I needed
to run to tell mactex that this new directory exists on my machine and
contains latex templates. Believe: there is no such command, mactex just looks
in this directory of its own accord.

[mactex-2013]: http://www.tug.org/mactex/
[multimarkdown latex support]: https://github.com/fletcher/peg-multimarkdown-latex-support/zipball/master
[TeXworks]: https://www.tug.org/texworks/
[Multimarkdown]: http://fletcherpenney.net/multimarkdown/
[Markdown]: http://daringfireball.net/projects/markdown/
[Multimarkdown Composer]: https://itunes.apple.com/us/app/multimarkdown-composer/id593294811?ls=1&amp;mt=12
[homebrew]: http://brew.sh/
