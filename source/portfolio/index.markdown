---
layout: page
title: "portfolio"
date: 2014-05-14 20:04
comments: true
sharing: true
footer: true
---

## [GitHub](http://github.com/ethanp/)

## [Comment Analyzer](https://github.com/ethanp/programming/tree/master/StuffIWrote/Scala/CommentAnalyzer/CommentAnalyzer_0)

The homepage has a listing of the average sentiments of the videos you've
previously found, as well as their *average comment sentiment*.
![](../../../../../images/Comment_analyzer_homescreen.png)

When you click on a video on the homescreen list, or after you search for a
new video, you get to that video's detail page.

`Depth = 1` means that comment
was posted in reply to another comment. These comments are retrieved from a
completely different database (GooglePlus API) then the base level comments
(YouTube Data API v2), and responses to GooglePlus comments do not contain
which comment they are a response *to* in their metadata. So for now, I am
represent any level depth of commenting as level 1.

This sentiment analysis is performed by the Stanfords *[Recursive Deep Model for Semantic Compositionality]
[Stnfd Sentiment]*, which boasts the ability to
detect sarcasm etc. I think in the case of YouTube comments, something
similar, like sets of known "Good" and "Bad" words would have worked better
than this very sophisticated model because the comments don't usually follow
normal English tree forms.

[Stnfd Sentiment]: (http://nlp.stanford.edu/sentiment/)


![](../../../../../images/Comment_analyzer_coltrane.png)

By clicking on the `+New` button, you are promped to enter the URL of the
YouTube video for which you would like to retrieve, analyze the comments of,
and add to the list on your homepage, e.g.
`https://www.youtube.com/watch?v=fbAohexT0Ho`.

![](../../../../../images/Comment_analyzer_new.png)

Each stored video's detail page, also has a `-delete` button at the bottom,
for removing it from your homepage.

## [javascript-whackamole](https://github.com/ethanp/javascript-whackamole)

It's multiplayer-ready game, but currently only works for a single person against the computer. I made it to have a game to hook up to `playframework` and make it truly multiplayer via `websocket`. That's still `TODO` though....

It uses Bootstrap because I don't know any better and jQuery because it makes
the code look cool.

### Here are some screenshots of the game in action

![](../../../../../images/js-whack/Whole_Game.png)
![](../../../../../images/js-whack/Addnl_Players.png)


## [Django-AudioFiles](https://github.com/ethanp/Django-AudioFiles)

First, use WebRTC to record audio from client's microphone into a Javascript
`Blob` in the browser. Then upload the `Blob` to the server and store it as a
file with Django.

## [Pathfinder](https://github.com/ethanp/Pathfinder)

This is a utility method for those times when you have a bunch of log files
that you need to collect data out of, but they don't always come in the same
directory structure. Or there is a bunch of logged data files and you only want
the ones with a particular filename prefix/suffix/type. **This utility will
allow you to easily and readably express the given directory structure and return to you a
list of the absolute paths of *only* the files you want.**

#### Given this directory structure

    pwd -> upper_dir -> files a -> want it 1 --> dat data1.csv
                    \          \            \
                     \          \            > data I don't want.tsv
                      \          \
                       \          > don't want it
                        \
                         > files b -> want it 2 --> dat data2.csv
                                  \            \
                                   \            > data I don't want.tsv
                                    \
                                     > don't want it

#### Behold the following command

    from pathfinder import pathfinder

    my_files = pathfinder('.', ['upper_dir', '*', '^want', '.csv$='])

    print my_files

            => ['/Absolute...Path/pwd/upper_dir/files 1/want it 1/dat data1.csv',
                '/Absolute...Path/pwd/upper_dir/files b/want it 2/dat data2.csv']

#### Reference

Signature

    def pathfinder(cd_able, path):


* **cd_able** must be *either* a directory one may `cd` into from
  the current directory, *or* not a directory at all.


* **path** is the part that has the following **DSL**:
    * `*`    means any file
    * `^...` means **"starts with"**
    * `...$` means **"ends with"**
    * `...=` means **"is not a directory"**
    * `$` must precede `=` if both are being used as directives
    * There is no escape if you want the directives as literals

Sure you *could* do something like

```bash
find . | egrep "upper_dir.*/want.*\.csv$"
```

but where's the fun in that?
