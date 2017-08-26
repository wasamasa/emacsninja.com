((title . "Parsing the Hard Way")
 (date . "2017-08-26 23:52:39 +0200")
 (emacs? . #t))

Hello again and sorry for the long downtime!  My current Emacs project
is an EPUB reader, something that requires parsing XML and
validating/extracting data from it.  The former can be done just fine
in Emacs Lisp with the help of ``libxml2`` (or alternatively,
``xml.el``), for the latter there is no good solution.  Typically
people go for one of the following approaches:

- Don't parse at all and just use regular expressions on the raw XML.
  This works somewhat okayish if your input is predictable and doesn't
  change much.
- Parse and walk manually through the parse tree with ``car``, ``cdr``
  and ``assoc``.  Rather tedious and requires writing your own tree
  traversal functions for anything less than static XML.
- Invent your own library and use a selector DSL for DOM traversal.
  I've seen a few of those, like `xml+.el`_, `enlive.el`_ and
  `xml-query.el`_, however they support relatively little features in
  their selectors, use their own language instead of a
  well-established one (such as CSS selectors or XPath) and are
  usually not available from a package archive for easy installation.

As I'm a big fan of APIs like Python's lxml_ with the cssselect_
module and used the esxml package before, I decided to implement CSS
selectors for it.  The general strategy was to take parse a CSS
selector into a suitable form, do tree traversal by interpreting the
parse tree and return the nodes satisfying the selector.  Surprisingly
enough, the hardest part of this were the parsing bits, so I'll go
into a bit more of detail on how you'd do it properly without any
dependencies.

The approach taken in `esxml-query.el`_ is recursive-descent parsing,
as seen in software like GCC.  Generally speaking, a language can be
described by a set of rules where the left side refers to its name and
the right size explains what it expands to.  Expansions are sequences
of other rules or constants (which naturally cannot be expanded) and
may contain syntactic sugar, such as the Kleene star (as seen in
regular expressions).  Given an input string described by the grammar,
a parser breaks it down according to its rules until it has found a
valid combination.  The easiest way to turn a grammar into code is by
expressing it with a function for each rule, with each function being
free to call others.  Success and failure can be expressed by
returning a piece of the parse tree, a special sentinel value (I've
chosen to return ``nil`` if the rule wasn't completely matched) or
throwing an error, thereby terminating the computation.  If all
recursively called rule functions returned a bit of the parse tree,
the top-level call returns the complete parse tree and the parsing
attempt has been successful.

Traditionally there is an extra step before parsing the string, as
it's a bit tedious to express the terminating rules as a sequence of
characters, the string is typically preprocessed by a so-called lexer
into a list of tagged tokens.  This is relatively simple to do in
Emacs Lisp by treating the string like a buffer, finding a token that
matches the current position, adding it to the list of found tokens
and advancing the position until the input has been exhausted.  There
is one non-trivial problem though, depending on the token definitions
it can happen that there are two different kinds of tokens for a given
position in the input string.  A simple solution here is picking the
longer match, this is why the tokenization in
``esxml--tokenize-css-selector`` finds all possible matches and picks
the longest one.

The syntactical sugar used for the official CSS grammars consists of
alternation (``|``), grouping (``[...]``), optionals (``?``) and
greedy repetition (``*`` and ``+``).  Given the basic token operations
``(peek)`` (return first token in the stream) and ``(next)`` (pop
first token in the stream), it's straight-forward to translate them to
working code by using conditionals and loops.  For example, the rule
``whitespace: SPACE*`` is consumed by calling ``(next)`` while
``(pop)`` returns a whitespace.  To make things easier, I've also
introduced an ``(accept TYPE)`` helper that uses ``(peek)`` to check
whether the following token matches ``TYPE`` and either consumes it
and returns the value or returns ``nil`` without consuming.  With it
the preceding example can be shortened to ``(while (accept 'space))``.
Similarly, alternation is expressed with ``cond`` and grouping with a
``while`` where the body checks whether the grouped content could be
matched.

This parsing strategy allows for highly flexible error reporting going
beyond "Invalid selector" errors I've seen previously in a browser
console as you immediately know at which place the parser fails and
are free to insert code dealing with the error as you see fit.  Be
warned though that you must understand the grammar well enough to
transform it into a more suitable form, yet equivalent form if you run
into rules that are hard or even impossible to express as code.
Debugging isn't too bad either, you can observe the junctions taken by
your code and quickly spot at which it goes wrong.

I'm looking forward to venture into parser combinators and PEGs next
as they follow the same approach, but involve less code to achieve
similar results.

.. _xml+.el: https://github.com/bddean/xml-plus
.. _enlive.el: https://github.com/zweifisch/enlive
.. _xml-query.el: https://github.com/skeeto/elfeed/blob/master/xml-query.el
.. _lxml: http://lxml.de/
.. _cssselect: http://lxml.de/cssselect.html
.. _esxml-query.el: https://github.com/tali713/esxml/blob/cf54607986a90dd0e33cff961550792e5fef22f1/esxml-query.el
