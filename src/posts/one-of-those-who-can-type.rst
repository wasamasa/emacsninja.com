((title . "One of Those Who Can Type")
 (date . "2017-07-11 08:31:55 +0200"))

.. code::

    <paluche> wasamasa: how many times have you done MAL?
    <wasamasa> paluche: this is the fourth time
    <paluche> So...
    <wasamasa> so, three times so far
    <paluche> you're a MAL-coholic?

I don't even need to answer this one, do I?  `PR #4`_ is for
Smalltalk, one of those languages I always wanted to try out because
of their influence.  What convinced me to finally do it was that I had
to learn a bit of Objective-C for work and this made the second
obviously Smalltalk-inspired language I've encountered, other than
Ruby.  Overall, it was a pretty nice experience, despite not using an
image-based and IDE-centric workflow with `GNU Smalltalk`_.  As usual,
here's my list of random notes:

- Smalltalk has relatively minimalistic syntax
- My biggest stumbling points with it were sequencing syntax
  (semicolon and period) and precedence rules
- It's kind of telling that the standard all implementations go by
  omitted how the syntax for defining a method looks like, this makes
  it more difficult than it should to share self-contained code
  examples on the web
- Documentation or rather, finding the right method is a big problem,
  the canonical solution would be to use the browser, however the
  search function there errors out
- I've therefore resorted to searching the locally installed sources
  and internets for things not specific to the implementation
- Apply (aka `valueWithArguments`) is supported, variadic args aren't
  or rather, there is no syntax for them
- There is no cond/case construct, instead you're supposed to either
  return from every conditional, do a dictionary lookup, use
  polymorphism or nest boolean tests
- There is no continue/break construct, but it can be emulated easily
  with the non-standard ``valueWithExit`` method
- It's awesome just how much of the language is implemented in itself
- It's interesting that there are only five keywords and no special
  forms, only methods that are implemented in terms of VM primitives
- The class hierarchy makes more sense than in Java, generally this is
  the cleanest implementation of OO I came across (though Self dares
  challenging this by replacing metaclasses with a prototyping
  mechanism as seen in JS)
- The debugging experience is less than stellar, I have to try out
  Pharo for the real deal
- Blocks are considerably cleaner than in Ruby where you have three
  options for subtly different purposes with subtly different
  semantics
- Block syntax however is imperfect, you cannot return from them
  (attempting to do so gives you a silly error message), instead the
  last expression is used as return value
- This gets stranger if you consider that (non-local) returns are
  exclusive to methods; in other words blocks aren't intended to be as
  powerful as method bodies
- There is no stack overflow in step 5, however if you push far
  enough, you can reach an unrecoverable OOM condition
- The class library is far less forgiving than in Ruby, for instance
  slicing/access errors out instead of returning an empty array or
  `nil`
- String syntax is a bit weird with regards to quoting as strings are
  delimited with `'`, no backslash escapes exist and escaping of `'`
  is done by doubling it
- If there's a language that invented monkey-patching, this is the
  one; I've successfully made use of this capability to fix a runtime
  bug that only happens in the CI environment

.. _PR #4: https://github.com/kanaka/mal/pull/264
.. _GNU Smalltalk: http://smalltalk.gnu.org/
