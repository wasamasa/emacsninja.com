((title . "Implementing MAL")
 (date . "2016-03-02 13:56:47 +0100")
 (emacs? . #t))

`I did it`_!  For those of you who don't know yet, MAL_ is a
Clojure-like language specially made for getting into the language
implementation business.  It is significantly less work than
implementing R5RS_, comes with `loads of test cases`_, a step-by-step
guide_, many existing implementations [1]_ and `an implementation in
MAL itself`_ which is used to test self-hosting_.

My implementation is fairly standard.  As much as people love `hating
on Emacs Lisp`_, it has proven to be a perfectly viable choice for
writing interpreters.  I doubt picking Common Lisp or CHICKEN would
have made for much nicer code.

If you're wondering whether to implement MAL as well, go for it!  It
will teach you how interpreting Lisp works and that writing an
interpreter is easier than it looks like.  While it doesn't need to be
in a language not featured in its repository, it's nice to contribute
a new one.

To run the implementation, check out the MAL repository, change into
its directory and run ``emacs -Q --batch --load elisp/stepA_mal.el``.
Development was done on Emacs 24.5 with brief testing on Emacs 24.3 as
it is still a fairly popular implementation.

Here's a few random notes I've made while writing this thing:

- ``--batch`` mode is terrible_.  This is not surprising given what
  purpose Emacs Lisp was developed for (an extension language to be
  used *inside* Emacs), but still annoying because I couldn't find any
  way of making readline_-style keybindings work in my REPL.  Neither
  adding hooks to the command loop nor changing keymaps (be it in the
  read functions or ``minibuffer-setup-hook``) did work, so you'll
  need to use ``rlwrap`` for a nicer experience.
- You can read input with ``read-from-minibuffer``.  It cannot read
  more than one line and throws an error on EOF, so I wrapped it into
  ``ignore-errors`` and checked whether it returned nil...
- You can print with the ``print``/``princ``/``prin1`` family.
  ``message`` on the other hand prints to ``stderr``, same for
  everything displaying things in the echo area. All of these return
  their argument, so don't forget to return ``nil`` afterwards.
- To add a final newline, you use ``terpri`` (short for "Terminate
  Print Line").  ``message`` does include a newline,
  ``print``/``princ``/``prin1`` don't.
- The truthiness semantics of Emacs Lisp and MAL are incompatible.
  Not having ``false`` is one thing, the conflation of ``nil`` and
  ``()`` is worse as it is impossible for interop code to tell
  the difference.
- There is no acceptable support for structs/records.  Emacs Lisp
  doesn't allow you to define your own opaque types, so
  ``cl-defstruct`` does encode them in form of vectors (alternatively,
  lists).  This would be OK if one could define custom printer
  functions, but `you can't`_.  Another problem is that it's way too
  simple to fake these structs from other code as they use an interned
  symbol for the tag.
- I did initially use ``cl-defstruct`` for atoms, environments and
  functions, but later I rolled my own thing as I found the Common
  Lisp way of defining custom constructors way too wordy.  In other
  words, there's no longer a dependency on ``cl-lib.el``.
- Later I've found out that while metadata support is only required
  for functions, it is a nice-to-have for the other types, so I
  switched to boxing all types and adding a metadata field.  While
  this made for more visual noise in the internal representation and
  code, debugging became significantly simpler as there's no longer a
  way to mix up types (and get confusing/silent errors later).
- Support for vectors and hash tables is lacking.  While it was
  possible to get by by coercing vectors to lists and using the basic
  hash table functions, the hash table equality code is easily the
  most unreadable part of ``core.el``.
- Unlike what their names suggest, ``throw`` and ``catch`` are for
  control flow (not exception handling) and are used to implement a
  ``return``.
- Exception handling is weird, but usable.  You can use
  ``define-error``, but only in Emacs 24.4.  If you just ``signal`` an
  undefined error and retrieve an error string for it, Emacs turns its
  into the infamous ``Peculiar error``.
- One of the nicer new Emacs Lisp features is the closure support.
  Just add a file-local variable for ``lexical-binding`` with ``t`` as
  value to the file where it needs to be enabled.  An unfortunate side
  effect is that the closures are printed as self-referential lists
  which makes for terrible debug printing.  Don't make the mistake of
  traversing them or you'll run into the recursion limit.
- Actually, it's not called a recursion limit, but rather
  ``max-lisp-eval-depth`` and you'll run into it if your TCO step has
  gone wrong.  Fortunately it's not contagious, so unless you infect
  a regularly executed part of Emacs, your session will remain usable.
- The ability to use Edebug on functions was invaluable.  A weird
  side-effect was that it kept polluting the closures of my editing
  session, so I had to restart Emacs once in a while when inspecting
  them.
- The advanced steps were surprisingly easy to implement, with the
  exception of self-hosting fixes.  Getting to the bottom of those
  took time, I figured out the last one on the ``#mal`` channel.
  Additional checks to catch those are on their way.
- The guide is very helpful, but gets spottier towards the end.  I'm
  currently helping out to get that part on the same level as the rest
  of it.

.. [1] At the time of writing, 46 implementations have been handed in
       already.

.. _I did it: https://github.com/kanaka/mal/pull/180
.. _MAL: https://github.com/kanaka/mal
.. _R5RS: http://www.schemers.org/Documents/Standards/R5RS/
.. _loads of test cases: https://github.com/kanaka/mal/tree/master/tests
.. _guide: https://github.com/kanaka/mal/blob/master/process/guide.md
.. _an implementation in MAL itself: https://github.com/kanaka/mal/tree/master/mal
.. _self-hosting: https://en.wikipedia.org/wiki/Self-hosting
.. _hating on Emacs Lisp: https://www.emacswiki.org/emacs/WhyDoesElispSuck
.. _terrible: http://www.lunaryorn.com/2014/08/12/emacs-script-pitfalls.html
.. _readline: https://en.wikipedia.org/wiki/GNU_Readline
.. _you can't: http://emacshorrors.com/posts/dont-bother.html
