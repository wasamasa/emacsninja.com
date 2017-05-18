((title . "Nostalgia")
 (date . "2017-05-18 19:53:05 +0200"))

My first programming experience was with VB6.  I considered trying
TurboPascal, but couldn't figure out how to use it from cmd.exe on my
measly Windows 98 SE machine.  Many things have been said about the
quality of the BASIC family of languages and the subpar programs
created with VB6, but it's easy to forget how convenient they make it
for the beginner to create something workable without losing their
head in irrelevant details in the windowing toolkit.  The workflow
looks mostly like this: Create and arrange widgets, edit their
properties, double-click each widget with custom behavior and write
code for the respective event handler.  Needless to say that the few
original programs I did write weren't of the useful kind.  Only much
later after a brief encounter with C++ for game programming (which
made me quit programming for a few years) I discovered scripting
languages, learned to love Python and eventually spun off into the
polyglossia required by modern software development jobs.

windows93.net_ reminded me of the good ol' times, leaving me wondering
just how hard it would be to recreate the irretrievably lost programs
from back then.  My excursions into GUI programming have taught me
that GNOME offers two separate projects that connect their
introspection for GTK and friends with a JavaScript interpreter, seed_
and gjs_.  Unfortunately ``seed`` is no longer in the Arch
repositories, so I picked ``gjs``, just to find out that there is no
official documentation, aside from `an auto-generated set of API docs
provided by a user`_.

The good news is that with the help of ``#gtk+`` on Freenode, I
managed porting `all applications`_ in less than a day.  The bad news
is that I doubt I'll use ``gjs`` again as its situation reminds me of
LLVM: Lots of potential, intriguing claims (in their case,
introspection giving you language bindings for free) but with outdated
documentation if any (you're better off with reading the source and
experimenting a lot) and tooling being half-assed.  Rewriting teapub_
might be possible, but for what gain?  It's no surprise `Qt is eating
their lunch`_ despite being more complex and requiring you to use C++.
Had I been given this environment ten years ago, I'd probably have
gone for browsers instead.  So, in a way it's not surprising that
environment appears to breed the new generation of programmers.

.. _windows93.net: http://windows93.net
.. _seed: http://live.gnome.org/Seed
.. _gjs: http://live.gnome.org/Gjs
.. _an auto-generated set of API docs provided by a user: https://people.gnome.org/~gcampagna/docs/
.. _all applications: https://github.com/wasamasa/nostalgia
.. _teapub: https://github.com/wasamasa/teapub
.. _Qt is eating their lunch: http://www.phoronix.com/scan.php?page=news_item&px=MTU2ODM
