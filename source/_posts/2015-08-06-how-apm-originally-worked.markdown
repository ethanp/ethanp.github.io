---
layout: post
title: "How apm originally worked"
date: 2015-08-06 20:17:12 -0700
comments: true
categories: [open source, github, oss, atom, development, nodejs, tdd]
---

Over the past few days I have been learning how `apm`, the Atom Package Manager
works under the hood. `apm` is what you use when, in GitHub's (relatively new)
"Atom" text editor, you go to the nice gui package installation interface under
`settings=>packages`.

Atom is a "hackable" text editor built on top of Chromium, using Node.js and
Coffeescript. I believe they call it hackable because all the code is open
source, and you can add plugins to do whatever you want. Your plugins can even
be written in C++ if that's more your style.

My goal was to figure out how `apm` works, and I wasn't sure how best to do
that. My knowledge of Node.js was minimal, and I was no expert in Coffeescript.
What I decided to do was fork the `apm` GitHub repo, clone it onto my computer,
and set my local HEAD to the ["initial commit"][init], and see if I could
understand that. The complete contents are as follows

[init]: https://github.com/atom/apm/commit/b8f4ce0d0cda458853eb280fde39fdeb2de38ebd

```text README.md
# APM - Atom Package Manager

Discover and install Atom packages.
```

At this stage I pretty confident that I understood everthing the author Kevin
Sawicki was doing. Lucky for me, it seems Kevin is rather unique on GitHub's
Atom development team for having smaller commits. I can justify this by noting
that on Atom-core's [list of developer contributions][devcont], he has 2x more
commits than the next guy, but is not in the top 5 in terms of LOC added.

[devcont]: https://github.com/atom/atom/graphs/contributors

So with my head fully wrapped around the "initial commit" I moved my HEAD past
the 2nd commit (a typo fix) into the first commit of any substance, ["Add
initial Gruntfile, binary, and ignores"][addinit]. At this point there was some
investigation to do.

<!-- more -->

I learned how he's setting his default `grunt` task to
compile his Coffeescript source files from the `src` directory into a `lib`
directory to be generated at compile time. I learned the basics of [what a
`package.json` is][expkg] and what it's basic fields do. And I learned how
`require('path')` command works by loading the `module.exports` object of
either the file at the specified path, or in the dependencies, etc. [as
specified here][node-module]. At this point I was good to go on understanding
the first half-hour of development on this project by mister Sawicki.


[addinit]: https://github.com/atom/apm/commit/31294702b11061e60214357f4529fb9b00a7068d
[expkg]: http://browsenpm.org/package.json
[node-module]: https://nodejs.org/api/modules.html

Basically, I continued in this manner, covering test-driven-development and
unit-testing using `jasmine-node`, a primitive API with only two endpoints
using `express.js` as part of a test-case, test fixtures, asynchronous vs
synchronous I/O APIs in various Node modules, and so on, and in the process
learned how `apm` originally _worked_.

### How APM originally worked

It basically just wraps the normal `npm` command to pull from a different
registry set up presumably by Mr. Sawicki. An `npm` registry is a CouchDB
instance, where module names, versions, and other metadata are mapped to the
relevant gzipped-tarball.

At first, there was a bit of complicated code where `apm` was downloading and
installing `node`, `npm`, and `node-gyp` itself, but eventually, these last two
were just [added as dependencies][add-dep]. This involved an
[`async.waterfall`][waterfall] which to my naive judgement about such a simple
script seemed to be a bit of overkill.


[add-dep]: https://github.com/atom/apm/commit/aa480a05e52d14baf56c06517826babd17ae4182
[waterfall]: https://github.com/caolan/async#waterfalltasks-callback


One thing I noticed was that a *lot* of the more complicated bits tended to
disapear over time, getting replaced by packages (e.g. `wrench`, `rimraf`,
etc.) or replaced by finding a simpler way to do the same thing (like passing
the appropriate command line option to the underlying `npm`).

It has felt, watching the first stage of this project come together, like I
have been able to peer over the shoulder of a far superior coder as he writes
what eventually has come to be one of the [main selling points][lolz] of the
Atom text editor itself. I have learned countless lessons. It is like that
tutorial that he never had the chance to write. When someone writes a tutorial
they generally are nervous and keep trying to explain the same thing in
different ways and it never quite makes sense. When someone is writing a
significant piece of a public work, they are strictly getting down to business.
Kevin may not have expected anyone to come along and piece through his thought
process, but he left it out on the table anyway and I just grabbed it.

[lolz]: http://qr.ae/RA68mn

I will continue to go through this git history to find out what happens in the
next chapter. I will also keep this in mind for the future as a way to find out
how something was made. I've got my eye in particular on watching Linus write
[git itself][gitrepo].

[gitrepo]: https://github.com/git/git/commit/e83c5163316f89bfbde7d9ab23ca2e25604af290
