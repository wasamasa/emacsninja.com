((title . "nuklear")
 (date . "2016-06-18 00:03:57 +0200"))

It's been two weeks since my last blog post and I've released `my
second CHICKEN egg`_!  Like the previous one, it's a wrapper for a GUI
library, albeit a hugely different one as it follows the immediate
mode (as opposed to the retained mode) approach.  This design decision
resulted in a number of differences:

- Very little hidden state
- No caching involved, all widgets are redrawn at each frame
- The UI is heavily integrated into your program and builds upon its data
- No callbacks, widgets that can be triggered return interesting
  values instead
- It becomes ridiculously simple to create custom widgets
- It's trivial to add custom behavior to widgets

For examples of the last two, see the calendar widget, the password
input box and the user-customizable layouts in the overview demo.
Reimplementing these in a traditional UI toolkit would have been
considerably harder, if not even impossible.  Now consider the
challenge of implementing a GUI editor.  With this approach it is a
matter of creating new data structures on demand that get rendered as
widgets and represent their internal state, then save it to disk when
asked to.  I find this way of thinking incredibly liberating and would
recommend anyone to give it a try, even if just in the form of a
data-driven game.

Unlike with the kiwi egg, I had to step up my C binding skills.  My
old trick to use helper functions that stack-allocate data, then
invoke the real function, did not work as they needed to live across
multiple function calls.  Instead I learned that one can use blobs as
storage managed from Scheme and use a locative to it for work from C.
As long as one doesn't do funny things like `using the results as hash
table keys anywhere`_ it works surprisingly well.  Other than that
I've used the C API here and there to pass around Scheme values that
were impossible to represent directly with the FFI.

The greatest problem I've' encountered was a lack of documentation.
Fortunately I've figured out everything necessary myself, but I still
have the impression that I'm missing out on something.  I hope to get
in contact with upstream regarding the bugs I ran into, otherwise I'll
have to give `dear imgui`_ a try...

.. _my second CHICKEN egg: https://github.com/wasamasa/nuklear
.. _using the results as hash table keys anywhere: http://bugs.call-cc.org/ticket/1293
.. _dear imgui: https://github.com/ocornut/imgui
