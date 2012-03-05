<link rel="stylesheet" href="/css/markdown.css"></link>

## What's The Buzz

One of initial descriptions of the project, then hosted at
[github.com/nojs](http://github.com/nojs) was
> Sane language to produce javascript code that works in browser and
> on node.js

Some time passed, more experiments followed, and it became obvious
that this no-js kind of approach applies to a variety of topics and has a
number of pleasant implications. That was the time when project was
killed and organization spawned at github under the same [url](http://github.com/nojs/).


Here are some in-progress experiments:

* [CLOS](/clos) CLOS-like object system for javascript

Actually, it's prototype-based implementation of
[c3 linearization](http://en.wikipedia.org/wiki/C3_linearization)
algorithm found in python or perl. It does allow to have multiple
inheritance in prototype-coherent javascriptish way.

* [gg](/gg) Simple Interactive Grammar Generator

It's metalua's gg inspired, or, to be more precise, metalua's gg rip
implementation in javascript. It was so awesome to read all of
[Fabien's posts](http://metalua.blogspot.com/) on it, and I wanted it so badly in
javascriptland, that I couldn't resist having it here ripped-off.

* [jscc](/jscc) Javascript compiler compiler

I don't quite get what that title could stand for, but it's extensible processing
tool for js that allows you to have any and all of that fancy or not
syntax extensions and all AST-manipulation and macro-defining
facilities.

* Some interesting extensions:
    * Structural pattern matching
    * Destructuring assignment
    * Destructuring properties access

* [Actor](/actor) Erlang-like message passing

  Allows erlang-line processes to run on nodes evenly in browser and on
  server. Consumed well with syntax bindings.


### Some bookmarks for future works

* [ast diff](/ast-diff) Diff algorithm applicable to AST
* Minimal emacs for noder
* Minimal linux for noder

