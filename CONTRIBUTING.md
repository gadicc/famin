# famin CONTRIBUTING guidelines

For those who want to get involved, or, as guidelines for users
wanting to write performant code (i.e. it's advantageous to
emulate the patterns we use in the library itself).

See also: https://hackpad.com/Famous-Stuff-akPoyPRvt9o (TODO,
merge in the relevant sections).

## Getting started

TODO

## General

* **Comments** are part of good code.  You should especially
  comment anything that might not be immediately obvious.
  Put a block comment at the beginning of each file explaining
  its purpose and make sure that public properties & methods
  have jsdoc comments.

* **Every `commit` or `PR` *must* include appropriate tests.**
  On every test run, a coverage report is created in
  `coverage/`, let's at least *aspire* for 100% coverage.

  [As the code base matures:]  Our test runner, karma, already
  allows to run the tests on local browsers, and each push
  will (soon) run the entire suite on 500+ browsers / versions
  / operating systems (mobile included) via SauceLabs, and
  update the browser support matrix in the README.

* **Prefix methods with an underscore** if they are internal,
  non-public, or use an API that may still change.

* **This is a *SEMVER* project**.  Bump the patch version for
  small fixes, the minor version for new features, and the
  major version **for API changes**.  Always avoid API changes
  where possible.  Always strive for backwards compatibility.

  When deprecating code, leave a comment with the original
  deprecation date (and proposed removal date).  Have the
  original function use `log.warn()` to explain why the
  code is deprecated and offer an alternative to users to
  upgrade in their own time.

  With backwards incompatible changes, again, bump the
  major version, but more importantly, *clearly mark the
  change as **BREAKING** in History.md*, and include
  instructions on 1) what changed, 2) why it changed, and
  3) upgrade path for users.

* **Every `commit` or `PR` *must* update History.md**,
  noting if it is a feature, bugfix, BREAKING change,
  and what is does and how it works, with a reference
  to any relevant GitHub issue.

## Pull Requests

* Should be made against the `devel` branch.

* Should adhere to everything in this doc (comments,
  data structures, garbage collection, backwards
  compatibility, proper deprecation process, include
  tests, make an entry in History.md, etc.)

* Should clearly state if they are ready for merging
  or are there for discussion purpuses.

* Should be "OK"'d by at least one additional team
  member, who should review the diff.

## Architecture

### General

* **Avoid use of ES6 features (e.g. set/map)** for high traffic calls.
  This will often be run with polyfills which aren't well optimized.

* When **bumping dependency versions** (e.g. in `package.json`),
  ensure that all tasks still work as expected, e.g. `build`,
  `dev` and `test`.

### Efficiency & Garbage Collection

* **Don't create new instances** (`new`), objects (`{}`,
  `obj.create()`, etc) or arrays (`[]`, `arr.slice()`, etc) in
  isolate code; instead create a factory that can recycle past
  instances.  *Justify any new data creation with comments*.

* **Use the right data structure for the job**.  We're used
  to using arrays and objects obsessively, which is a great
  convenience for "normal code".  For high traffic performant
  code, we need to be smarter, and the versions of these
  classes we provide all pool/recycle past instances.

  e.g. always prefer `SinglyLinkedList()` if it answers all
  your needs, e.g. you need to keep a list of items to iterate
  through but (almost) never need to access a specific index
  directly.   Use `Vec3()` instead of a 3x1 array of primitives
  for size, position, etc.

* **Avoid high traffic anonymous functions**.  For anything
  that will be run often, keep a reference around for that
  function and always pass that reference.  e.g. `onNextTick()`
  takes optional arguments for context (`this`) and data to
  avoid constantly creating new functions and garbage collecting
  them.  Iterating through these kinds of functions with a 
  SinglyLinkedList is highly efficient.

* **Don't use *bind()***.  It's not performant.  Make your
  own closure (using `.call()` is fine).  It's possible this
  will get optimized better in the future, but there are still
  all the old browsers.
  http://stackoverflow.com/questions/17638305/why-is-bind-slower-than-a-closure

* (?) Import the `Trash` class and **call `trash` on any object
  you wish to discard** (before removing your references to it).
  This will keep references to all garbage and avoid garbage
  collection at inconvenient times.  When the engine is sure
  it is, and will remain, idle, it calls `Trash.discard()` to
  really make the garbage available to the GC (Note: the GC is
  meant to only collect during idle times, but this isn't
  always the case... we're a better judge.).  This also helps
  keep track of (potentially unnecessary) garbage.

* Controversial: for super high traffic functions, suggest
  operating directly on _properties rather than gaving
  getProp() and setProp() methods, to avoid the overhead.
