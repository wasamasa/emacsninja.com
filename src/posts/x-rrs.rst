((title . "7 x R7RS")
 (date . "2017-09-14 19:49:20 +0200"))

The nice thing about Scheme is that not only it comes in more than
seven standards [1]_, but with tons of implementations as well.  I've
heard from various people that R7RS-small is good enough to write
portable code in it that will work on more than one implementation, so
I decided to try `implementing MAL`_ in it.  This has been mostly
successful as I've started out with 11 implementations, narrowed it
down to 7 and found a new one for future experimentation.  During this
time I've reported 14 bugs for two implementations, Cyclone_ and
Foment_.  My impressions of the implementations can be summarized as
follows:

Chibi_
  The recommended implementation if you're looking for a fully
  standards-compliant one.  It comes with its own comprehensive test
  suite that's borrowed by others.  I've had zero issues with it,
  although the overall speed could be better, earning it the last
  place (where it's tied with Foment).  This is somewhat weird
  considering that it's advertised as small yet embeddable, Picrin
  would be another candidate for that purpose.

Kawa_
  It proudly wears the GNU banner (despite not being the Scheme
  officially endorsed by GNU), but is otherwise completely unknown.
  This is a shame because unlike the other JVM languages I've worked
  with it comes with a fast compiler, has a small boot time and didn't
  pose any issues for me, other than having to learn how this
  classpath thing works in Java.  Speed is decent as it comes third
  place in the toy benchmarks, interop doesn't look painful either.
  Its author did a few more interesting things with it, such as
  implementing other languages like Emacs Lisp (which forms the basis
  of JEmacs_).  I might just as well invest more time into it and use
  it to write things where Clojure would otherwise be a bad idea.

CHICKEN_
  My favorite implementation for writing all kinds of small,
  standalone programs.  I occasionally stumble upon issues, but none
  for this run.  It's not completely obvious how to create R7RS
  programs using libraries from anywhere else than the current
  directory, this will be hopefully addressed in the next major
  release, among with a more R7RS-like reorganization of the
  namespaces and improvements to the package system.  Speedwise it's
  the best, despite the ``r7rs`` egg being known for incurring a
  huge speed penalty in numerical benchmarks.

Gauche_
  The domain says it all.  I haven't expected it to support R7RS
  considering how long it has been around, but it does!  Thanks to
  this it contains many contrib libraries and performs fairly well for
  an interpreter-only system.  If I hadn't discovered CHICKEN, this
  might have been my preferred Scheme to go for.

Picrin_
  The other contender in the embeddable domain.  Unfortunately I ran
  into several problems, the most problematic ones being that it's
  `currently not buildable from master`_ and that there are no
  official plans to support loading user-written libraries.  For this
  reason I wouldn't recommend using it over Chibi.

Sagittarius_
  A fork of Gauche with more features, some changes (build system,
  command-line switches) and a bit of performance improvements.

Cyclone_
  Judging from the design documents this is basically CHICKEN, but
  with a different approach to GC and threading that allows it to do
  native threads.  It's a relatively young project and therefore bound
  to have countless bugs, yet it's second place in the speed
  department.

Foment_
  Someone's personal project for learning Scheme.  It's about the same
  speed as Chibi and not quite there yet.

Guile_
  The Scheme officially endorsed by GNU.  It had a long time in
  making, but its R7RS support is minimal and only covers reader
  adjustments.

Racket_
  I'm somewhat surprised there is inofficial support for R7RS, given
  that the language itself has been derived from R6RS.  Not tested
  yet.

Larceny_
  Research project that supports R5RS, R6RS and R7RS to varying
  degrees.  Due to R7RS incompatibilities, a somewhat outdated
  documentation and a ridiculous toolchain to set it up I've decided
  to test it at some later point to see whether it becomes the fastest
  implementation for MAL.

Gerbil_
  I've learned about this one from `a blog post by Fare`_, the
  maintainer of ASDF, a build system for CL packages.  It turns out
  that not only he likes Racket, no, he also switched to this Scheme
  implementation because it's made by a close friend who not only
  ported Racket's module system to Gambit_, but also added R7RS
  support to it.  I expect it to become usable soon and to make for
  another contender to the fastest Scheme implementation for MAL.

.. _implementing MAL: https://github.com/kanaka/mal/pull/273
.. _Cyclone: https://github.com/justinethier/cyclone
.. _Foment: https://github.com/leftmike/foment
.. _Chibi: http://synthcode.com/scheme/chibi/
.. _Kawa: https://www.gnu.org/software/kawa/
.. _JEmacs: http://jemacs.sourceforge.net/
.. _CHICKEN: http://call-cc.org/
.. _Gauche: https://practical-scheme.net/gauche/index.html
.. _Picrin: https://github.com/picrin-scheme/picrin
.. _currently not buildable from master: https://github.com/picrin-scheme/picrin/issues/351
.. _Sagittarius: https://bitbucket.org/ktakashi/sagittarius-scheme/wiki/Home
.. _Guile: https://www.gnu.org/software/guile/
.. _Racket: https://racket-lang.org/
.. _Larceny: http://www.larcenists.org/
.. _Gerbil: https://github.com/vyzo/gerbil
.. _a blog post by Fare: http://fare.livejournal.com/188429.html
.. _Gambit: http://gambitscheme.org/wiki/index.php/Main_Page

.. [1] Additionally to RnRS there's the IEEE standard (which mostly
       resembles R5RS) and the upcoming R7RS-large standard that aims
       at giving Scheme a big standard library.
