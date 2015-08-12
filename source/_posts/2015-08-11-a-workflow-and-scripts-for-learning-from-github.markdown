---
layout: post
title: "a workflow and scripts for learning from github"
date: 2015-08-11 17:48:41 -0700
comments: true
categories: [git, github, learning, programming, bash, shell, scripts]
---

### My "Learning" Workflow

As I wrote about [before][before], I have developed an interesting method of learning
from experts, which can be summarized as follows:

[before]: http://ethanp.github.io/blog/2015/08/06/how-apm-originally-worked/

1. Fork their repo on GitHub
2. Clone the repo locally and "detach the `HEAD`" to the "inital commmit"
3. Now repeat the following `while (curious)`
    1. Open the working tree in an editor/IDE
    2. If there's something runnable, run it
    3. Understand everything going on in the working tree
        * Take hints from the commit message
    3. Advance `HEAD` one commit
    4. View the diff from the previous commit

<!-- more -->

### SourceTree cannot handle this workload

I have always used Atlassian's Git GUI called SourceTree for all of my Git
usage because it is a _great_ application, but the fact is, it will not work
for the workflow above. SourceTree has to rebuild the list of commits *all the
time* (startingfrom the most recent), and when there are thousands of commits,
that can take about a minute. But since I'm always operating at the beginning
of the reverse end of the commit log, using SourceTree is untenable. For
whatever reason they decided it was a better idea not to keep all these commits
in memory after they are loaded the first time. I'm not saying that was a bad
decision, because my workflow here may be _atypical_.


### Git is a DAG pointing back in time

From what I understand--which is not (yet) a whole lot--Git is a DAG of
"snapshots" of the state of your project. Each snapshot points to its parents.

In a typical case, you'll load a snapshot, edit your working directory, add the
changes into the staging area, and __commit__ a new snapshot which equals the
previous snapshot, plus the staged changes. Now your new commit has _one_
parent: the previous snapshot.

A __merge commit__ would have __two parents__: the previous commit on the
branch being committed to, and the previous commit on the branch being merged-
in.

### The Learning workflow flows the wrong way

The Learning worflow would require pointers to the _next_ commit, not just
back-pointers. So it doesn't quite fit the Git mold, and we require a
workaround.

That's where StackOverflow saves the day. [Here][git traverse] someone modified
a StackOverflow answer by someone else to a _related_ question, and produced
wrappers that help you traverse back and forth between commits. I reproduce an
explicated and slightly simplified version below which may be added to your
shell config file.

```scala
# checkout prev (older) revision
git_prev() {
    # move HEAD "one" generation back along *this* branch
    git checkout HEAD~1
}

# checkout next (newer) commit
git_next() {
    # git show ref:
    #       show (commit-sha, ref-name) pairs for current-versions of all 
    #       "refs"; i.e. tags, remote branches, and local branches
    # 
    # git show-ref -s HEAD:
    #       get sha of latest commit on branch pointed to by HEAD
    #
    # BRANCH=...:
    #       get the name of the branch HEAD is on
    #
    # HASH=`git rev-prase $BRANCH`
    #       get the hash of the latest commit on the current branch
    #
    # git rev-list --topo-order HEAD..$HASH:
    #       list all commit sha's in order on the current branch from
    #       HEAD until now 
    #
    # PREV=...:
    #       get the commit sha for the commit after HEAD on branch
    #
    # git checkout $PREV
    #       move head to the next commit
    #
    BRANCH=`git show-ref | grep $(git show-ref -s HEAD) | sed 's|.*/||' | grep -v HEAD | sort -u`
    HASH=`git rev-parse $BRANCH`
    PREV=`git rev-list --topo-order HEAD..$HASH | tail -1`
    git checkout $PREV
}
```

[git traverse]: http://stackoverflow.com/questions/2121230/git-how-to-move-back-and-forth-between-commits/23172256#23172256

With that knowledge in tow, I also made a little command to jump to the _i_'th
commit on master. If you're trying to jump to the _i_'th commit on the current
branch, get the current branch using the `BRANCH=...` code above, and pass it
to `rev-list`.

```scala
gitj() {
    git checkout `git rev-list master | tail -n$1 | head -n1`
}

$ gitj 1    # jump to initial commit of master
$ gitj 3    # jump to third commit
```

### Wrappin it up

So yeah, now the workflow is simpler to use and it was made possible by gaining
a better understanding of how Git works.
