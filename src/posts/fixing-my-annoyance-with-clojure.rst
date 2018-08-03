((title . "Fixing My #1 Annoyance With Clojure")
 (date . "2018-08-03 20:53:26 +0200")
 (emacs? . #f))

Clojure is quite something. Immutable by default, functional style
programming being encouraged and with many useful libraries written by
a nice community.  There's ugly sides to it as well, like the error
reporting being terrible, dealing with Java things and waiting for
your tooling to boot.  However, my greatest annoyance with it is that
my preferred debugging workflow, printing out the problematic thing,
is far more painful than it should be [1]_.  Suppose you have the
following piece of code and want to check the result of the last form
in the ``let``.

.. code:: clojure

    (defn frobnicate [x]
      (let [y ...
            z ...]
        (filter foo? (map bar (baz x)))))

This won't do as the function would return ``nil``:

.. code:: clojure

    (defn frobnicate [x]
      (let [y ...
            z ...]
        (prn "XXX" (filter foo? (map bar (baz x))))))

This works, but what if the expression has a side effect or is
computationally expensive?

.. code:: clojure

    (defn frobnicate [x]
      (let [y ...
            z ...]
        (prn "XXX" (filter foo? (map bar (baz x))))
        (filter foo? (map bar (baz x)))))

This is quite good, but annoying to type out and undo after it's no
longer needed:

.. code:: clojure

    (defn frobnicate [x]
      (let [y ...
            z ...
            xxx (filter foo? (map bar (baz x)))]
        (prn "XXX" xxx)
        xxx))

I'd love to have this:

.. code:: clojure

    (defn frobnicate [x]
      (let [y ...
            z ...]
        (dbg (filter foo? (map bar (baz x))))))

The idea is borrowed from Smalltalk, more specifically Super Collider
where you can just call ``.debug`` on an object to see its value
printed.  Unlike a regular print function this one returns the object
and does therefore allow inserting into a call chain just fine.
Bonus: It accepts an optional argument for printing a prefix so that
you can identify the debug output easily.  Translating this to Clojure
is as easy as it gets [2]_:

.. code:: clojure

    (defn dbg
      ([thing]
       (dbg "XXX" thing))
      ([prefix thing]
       (println prefix)
       (print (with-out-str (clojure.pprint/pprint thing)))
       thing))

Now, this is useful. You can surround anything useful with ``(dbg
...)`` and if you wish, add a prefix as first argument.  Removing the
debug print is as easy as raising the enclosed S-Expression [3]_.  But
how do you get this into a Clojure session?  The thing is that once
you eval this in a namespace, the function belongs to that namespace
and to use it in another one you'd need to either import it from that
namespace or define it in the other namespace.  Another issue is that
you'd need to edit the sources of a Clojure project to make use of
this helper.  Surely you can do better than that?

Studying the Leiningen documentation and the official Clojure docs on
namespaces I learned about two more things:

- Leiningen supports the ``:injections`` key which evaluates a vector
  of forms in the project context [4]_
- The lowest-level function to manipulate a different namespace than
  the current one is ``intern`` [5]_

Put both together and you get the following snippet for your
``~/.lein/profiles.clj``:

.. code:: clojure

    {:user {:injections [(defn dbg ...)
                         (intern 'clojure.core 'dbg dbg)]}}

This is close to perfect.  It will only work for projects using
Leiningen obviously and displays a warning as the code is run twice,
but it works nicely in any namespace!

.. [1] I've got to admit, this is quite petty. If I managed learning
       how to read ugly Java backtraces and studied the wonders of the
       Java class path, how could ``printf``-debugging annoy me to
       this extent?  I believe the conventional wisdom that it's
       hardly worth automating tasks with little time savings misses
       the point, fixing long-standing annoyances however is a worthy
       goal.  Better be happy than bitter about your setup.
.. [2] The eagle-eyed reader will notice that you could get by with a
       mere ``(clojure.pprint/pprint thing)``.  The reason for the
       above is that I've encountered rather discomforting behavior in
       a codebase where pretty-printed output ended up interleaved
       with logging output.  The easy workaround is making the
       pretty-printing atomic by collecting it into a string, then
       printing it out.
.. [3] If you use Paredit or Smartparens, it's as easy as hitting
       ``M-r`` with point on the form you want to replace the outside
       one with.
.. [4] This doesn't say anything about how often it's actually
       evaluated, so better put an idempotent expression there.
.. [5] ``intern`` in Emacs Lisp does merely convert a string to a
       symbol.  In CL it does the same, but allows specifying what
       package the symbol should belong to.  In Clojure it takes a
       namespace, a name and a value...
