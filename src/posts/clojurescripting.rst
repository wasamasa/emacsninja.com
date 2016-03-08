((title . "Clojurescripting")
 (date . "2016-03-08 21:42:33 +0100"))

After seeing the announcement for `ClojureScript's self-hosting feature`_
and planck_, I kept wondering whether one couldn't have the same on
your typical GNU/Linux machine.  Turns out it is possible thanks to
the cljs-bootstrap_ project!

I've initially tried building it from source, but as the standalone
file produced by the `Closure compiler`_ wasn't functional, I went for
the lazy route instead:

.. code:: shell

    npm install -g cljs-repl
    npm install -g source-map-support

Installing ``source-map-support`` is not required, but still done to
avoid the ``Could not load source-map support`` error.  Most likely a
missing dependency...

If you've configured ``npm`` correctly, you'll have access to a
``cljs`` executable now.  It appears to be functional:

.. code:: clojurescript

    cljs.user=> (+ 1 1)
    2

Now for the scripting part.  You can either pipe code into the
``cljs`` executable or pass an extra argument that's interpreted as a
file to read code from.  What's less obvious is that the code must be
a single form, otherwise it will result in an incomprehensible
error...

With this knowledge, I was able to rewrite the following:

.. code:: javascript

    require('process');

    process.stdin.setEncoding('utf8');
    process.stdin.on('data', function(data) {
        process.stdout.write(data);
    })

To something more lispy:

.. code:: clojurescript

    (do
      (js/require "process")

      (.setEncoding js/process.stdin "utf8")
      (.on js/process.stdin "data"
           (fn [data]
             (.write js/process.stdout data))))

I'm sort of underwhelmed.  While you can get pretty far, anything
outside the realm of manipulating data structures will require
interop, be it with what Node.js provides directly or one of the
numerous modules out there.  You can of course write your own
libraries for more idiomatic access to commonly used functionality,
but that again would require tooling to be easily usable.

Maybe I should reconsider doing my own take on Clojure based on `my
Emacs Lisp MAL implementation`_.  Doing that would be incredibly
pointless, but then I'd finally know how my ideal scripting language
would look like.  Alternatively, I could just stay with CHICKEN and
contribute some more eggs...

.. _ClojureScript's self-hosting feature: https://swannodette.github.io/2015/07/29/clojurescript-17/
.. _planck: https://github.com/mfikes/planck
.. _cljs-bootstrap: https://swannodette.github.io/2015/07/29/clojurescript-17/
.. _Closure compiler: https://developers.google.com/closure/compiler/
.. _my Emacs Lisp MAL implementation: http://emacsninja.com/posts/implementing-mal.html
