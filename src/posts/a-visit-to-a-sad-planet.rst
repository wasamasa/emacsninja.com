((title . "A Visit to a Sad Planet")
 (date . "2017-04-18 01:42:35 +0200"))

One of the things that always irked me about `my EPUB reader`_ was
that part of it is written in JavaScript, simply because that's the
only way one can script a browser view these days.  I've always wanted
to give the SPOCK_ compiler a try to see how well it does compared to
ClojureScript, so this project gave me the perfect chance to cure me
of delusions.  After an evening, I got it running, it took me a few
more evenings to iron out bugs introduced by the conversion and add a
new bugfix. Here's a list of observations made in that time:

- Both versions are about the same size, with the Scheme version being
  a bit shorter (which is mostly closing parentheses not going on a
  separate line).  I'd expect a greater difference in favor of the
  Scheme version if I had any noteworthy business logic embedded into
  this, but alas.
- Debugging got significantly harder as there is no REPL, no source
  maps integration and no debugger for the Scheme code.  I've had to
  do with classic ``printf``-Debugging, except that it looked more
  like ``(%inline "console.log" (jstring (string-append "Foo "
  (number->string 42))))``.
- While there is documentation (which includes a few working
  examples), it isn't clear how to use the compiler to its fullest
  abilities.  I've resorted to compiling all kinds of code and staring
  at the compiler output to see what works and what doesn't.  This
  experimentation revealed that you'll want to use ``%inline`` for
  most interop, with a bit of dot syntax for property access.
  ClojureScript beats SPOCK easily in this aspect, including its
  ``#js`` reader macro and conversion macros from/to JS data
  structures.
- Error reporting is extremely basic, with some errors being silent
  and merely preventing code from executing any further.
- The supported language is restricted to R5RS with a few useful
  macros and JS-specific helpers.  In other words, while you might
  manage compiling other Scheme libraries to JavaScript, you're better
  off writing your own helpers as needed.
- Tooling is simple and quick.  Recompiling code is instantaneous,
  it's easy to see what part of your own code maps to the generated
  parts.  This is the only benefit I see in SPOCK over ClojureScript.

To summarize, if you want maximum comfort and features, go for
ClojureScript.  The price you pay for it is significant friction while
developing, but other than that it's pretty advanced.  Personally I
think I'll stay with vanilla JavaScript for my other toy projects to
keep things as simple and painless as possible.

I predict that Guile Emacs won't lead to a significant increase in
packages written in Scheme for similar reasons.  Much like in
browsers, the majority of Emacs Lisp usage doesn't have complex
business logic and follows the principle of practicality over purity.
Perhaps it's different for big projects like Magit or Evil, but even
these cases are doubtful to me, simply because they have higher
priorities than speculative rewrites that might as well kill the
project.  I could keep rambling about my reasons for this assessment,
but that is better left for a separate blog post...

.. _my EPUB reader: https://github.com/wasamasa/teapub
.. _SPOCK: http://wiki.call-cc.org/eggref/4/spock
