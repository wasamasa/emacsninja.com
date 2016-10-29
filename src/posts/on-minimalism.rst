((title . "On Minimalism")
 (date . "2016-10-26 23:44:31 +0200"))

`I've implemented MAL for the third time by now`_, this time in
PicoLisp_, a language priding itself on its implementation simplicity.
While it clearly is a Lisp dialect, it has foregone a good amount of
classic Lisp design choices in favor of terse code.  Despite this,
there are practical inclusions for writing application software, like
the GUI system and a distributed database implementation with a
Prolog-style query language.  Other interesting features are an
unobtrusive OOP system, a FFI for C and Java, live debugging
utilities, pattern matching and more in a 1MiB tarball.

You might wonder why I'd be up for learning yet another Lisp dialect,
after having learned Emacs Lisp, Clojure and Scheme.  Furthermore,
Scheme already claims to take minimalism as language design principle
and of course there are more obscure Lisp dialects, like Arc_ and
newLISP_.  I can only blame a friend who told me about this
fascinating talk given by the PicoLisp author demonstrating the
abilities of his programming language.  The overall picture my friend
painted was that of a bizarro world where a crazy German ignored all
established rules for a Lisp dialect, walking the fine line between
insanity and practicability.  Most surprisingly though was that he
used his own invention to write business applications and succeeded in
making a living off it.  Naturally I was intrigued and kept PicoLisp
on my backlog of things to play with.

My implementation is a bit smaller than the Emacs Lisp one, is the
first one to actually make use of GNU readline and went for a purely
Lisp tokenizer as I couldn't figure out how to use PCRE for this task.
It also appears to be the fastest one out of all Lisp family
implementations.  This might change though once the "clisp"
implementation gains support for using SBCL instead of CLISP...

Regarding oddities, here's an incomplete list:

- No lambda.  You pass a quoted list instead.
- Quote returns more than only its first argument.  However ``(quote 1
  2 3)`` is equivalent to ``'(1 2 3)`` so you'll most likely not ever
  notice...
- No macros.  Functions can instead not evaluate their arguments, your
  job is to evaluate them as needed.
- No strings.  String syntax creates so-called "transient" symbols.
  Additionally to doubling as string replacement, these are not equal
  to other symbols with the same contents and are therefore used to
  avoid name clashes in macro-like functions.
- No implicit closures.  If you need to capture something from the
  environment, you must do this explicitly and have the choice between
  mutable and immutable ones.
- Unknown symbols evaluate to ``NIL`` instead of throwing an
  exception.
- Indentation (and even pretty-printing) is a lot easier than in
  classic Lisp dialects as it's basically about increasing the depth
  by three spaces for each level instead of lining up parentheses.
- Closing parentheses that do not belong to the current line are
  separated by spaces.  For convenience's sake, a closing bracket is
  interpreted as "super paren" and closes all remaining parentheses.
- The style guide has an interesting solution to the problem of local
  variables potentially shadowing built-in functions: Capitalized
  identifiers!
- ``NIL`` is not only equivalent to the empty list, but to the empty
  string as well.  This bit me when wrapping readline as an empty
  string couldn't be discerned from ``NULL`` with the naïve
  approach...
- Error handling is close to non-existing.  If you screw up things too
  much, the interpreter will segfault on you.  This is not considered
  a bug.
- Identifiers for built-in functions are very short and at times
  cryptic [1]_.  Clojure got nothing on that!

.. [1] ``read`` does not parse a string into a S-expression, ``str``
       does.  The result of this cannot be handed to ``eval`` either,
       you'll need to ``run`` it instead.  I could go on with this for
       a while...

.. _I've implemented MAL for the third time by now: https://github.com/kanaka/mal/pull/239
.. _PicoLisp: http://picolisp.com/
.. _Arc: http://www.arclanguage.org/
.. _newLISP: http://www.newlisp.org/
