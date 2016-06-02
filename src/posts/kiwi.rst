((title . "KiWi")
 (date . "2016-06-03 00:58:17 +0200"))

I've released `my first CHICKEN egg`_ today!  It's part of my thesis
on GUI programming approaches and will certainly not be the last one
in my series of new GUI library wrappers.

I actually expected to release `my giflib egg`_ first, but that went
way worse due to deficiencies in the documentation and one very
puzzling bug in the official library interface nobody else than me
appears to be using.  As `it got fixed only recently`_ [1]_,
I might pick up work on that soonish.

kiwi itself is pretty close, but not too close to `the KiWi library`_.
After learning how it works, I wrote bindings to nearly all
identifiers, save the ones that weren't worth the trouble [2]_, then
added a bit of sugar for the most tedious parts of the examples.
Along the way I've handed in over a dozen issues and a few PRs, nearly
all of which were handled quickly.  Therefore, even if you're not too
keen on working with an in-progress library, upstream has always been
helpful and a pleasure to work with.

The other side effect was that I learned to use ``gdb`` for debugging
segfaults, have a much clearer idea just what exactly writing good
wrappers involves and am now in the know how exactly the egg release
process works.  It is much more than just generating the wrapper code
to achieve something that feels rather like Scheme than C.  One
example of this is the rather peculiar use of stack-allocated structs
for specifying coordinates and dimensions of widgets which doesn't
translate well to programming languages that use heap-allocation for
nearly all objects.  I did initially use ``malloc(3)`` to get
heap-allocated and ``free(3)`` with a finalizer to free it after a GC.
This was a pretty bad idea as it did trigger GCs when playing around
with the drag and drop example, so after much experimentation, I
rewrote nearly all functions involving these structs to take their
values, create a stack-allocated struct and use its address in the
respective function, which pretty much mirrors the code style used in
the examples.

I'd love hearing back from you if you happen to use CHICKEN as well
and have a need for widgets in your SDL2 application.  If not, stay
tuned for my other upcoming bindings to nuklear_ and libui_!

.. [1] This does of course only confirm my theory about me being the
       only one using giflib this way...
.. [2] Those were a few helpers that would have required more glue
       code than ports of the algorithm to scheme, procedures for
       implementing new widgets and things that didn't play well with
       the FFI.

.. _my first CHICKEN egg: https://github.com/wasamasa/kiwi
.. _my giflib egg: https://github.com/wasamasa/giflib
.. _it got fixed only recently: https://sourceforge.net/p/giflib/code/ci/ef0cb9b4be572262b49fbc26fb2348683f44a517/
.. _the KiWi library: https://github.com/mobius3/KiWi
.. _nuklear: https://github.com/vurtun/nuklear
.. _libui: https://github.com/andlabs/libui
