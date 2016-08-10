((title . "Lisping the ChucKian Way")
 (date . "2016-08-10 22:23:30 +0200"))

`I did it again`_!  For the uninitiated, ChucK_ is a special-purpose
programming language intended to be used for sound synthesis.  While
working on `a college assignment`_, it occurred me that the language
had just the minimum amount of features to be leveraged for my second
MAL_ implementation (previously_): Console/File I/O, an object system
reminiscent of both Java and C++, regular expressions and arrays.  How
hard could it possibly be to write the probably most advanced ChucK
program in existence?

This turned out to be a good deal more annoying than my first
implementation.  The major problem besides the language giving you
minimal support for non-audio programming was that clearly nobody else
did use it for a project of this size.  I came to this conclusion
after reporting `a number of bugs`_ that should have been absolutely
obvious to anybody using the standard library for anything else than
creating music.

Modules
-------

I didn't expect the first obstacle to be the very act of loading code
from another file, something that should surely be supported well for
a system also used in live programming.  It turns out that live
programming is a *very* stretchable term; what they do is more akin to
hotswapping invidual code units in terms of files which is cute, but
not what I need.  There is a ``Machine.add(file)`` facility available
from code, however it does not immediately load code from the
specified file and instead post-pones loading after the current file
has been loaded up.

I've pondered whether to create a file solely consisting of these
instructions via ``make``, then decided against it in favor of an
upfront loading approach where a runner script extracts "magic"
comments indicating the dependencies and boots ``chuck`` with all of
them in that order and the file they originate from.  While this isn't
ideal [1]_, it works surprisingly well.

I/O and time
------------

Getting user input was also tricky.  Initially I tried out the HID
example just to find out that it would only work for special input
devices as opposed to console input.  Therefore I tried out the
``ConsoleInput`` class which gives a "hacked" thing to read from.  Due
to it having some oddities such as not accepting ``C-d`` (and
therefore only being terminable with ``C-c`` which brings down the
entire process), behaving incorrectly when nested and adding an extra
space to the prompt, I hacked my own thing together with the ``KBHit``
class, an ASCII table and some flushing to standard output.

Speaking of printing, you would expect the manual to tell you how one
does that...  Instead you are told about the debug output syntax ``<<<
foo >>>;``, I've had to consult the ``VERSIONS`` file to learn that
one can send strings to ``chout``.  Another wonderful gotcha was that
due to some RtAudio bug, you can get spurious errors about your audio
stream still running which mess up the prompt.  The only way to get
rid of them is starting the process with ``--silent`` which just runs
everything as fast as possible resulting in 100% CPU usage.  Fun.

Finally, I've hunted for a way to measure time for the performance
tests, but learned that ChucK's notion of it is more about
coordination of sound.  In silent mode, it is therefore useless.
However not all hope is lost as you can shell out to ``date`` and
retrieve its output in a hacky way: ``Std.system(command)`` doesn't
return anything useful, so you need to redirect to a file and read
from it instead...

Exceptions
----------

ChucK is somewhere between a traditional compiled and interpreted
language.  I did run into a good amount of exceptions, but was
surprised that I couldn't throw, create or catch any.  This is
saddening me as it forces one to return errors and check for them very
often, be it in form of integers and out parameters (hello, C!) or a
dedicated error object.

Functions
---------

I'm pretty used to have at least some way to pass functions around and
expected ChucK to be the same considering that the debug syntax had a
way of printing functions.  However the language doesn't have any
function type, so while you can coerce one into an ``Object``, it
won't do you any good.  Therefore I went for the C# solution and
implemented functors, that is, classes with a ``call(args)`` method
which one can instantiate and pass around.  Yuck.

OO
---

While one can do Java-like OOP, it is severely limited.  Considering
that there are no generics, no ``super``, no interfaces, no unions, no
casting to arrays, no self-references, no automatic boxing or boxed
primitives, an impoverished ``static`` keyword and no ``private``, the
resulting code is clumsy, yet has a certain air to it which I'd call
"ChucKian", like a few other people did on the ``chuck-users`` mailing
list.  The most impressive collection I've found so far is LicK_.

Types
-----

There is a strict distinction between reference and value types which
require using either the ``@=>`` or ``=>`` operators.  While the
manual insists strings are reference types, one can use ``=>`` just
fine on them...

While there are no hashmaps, one can use an array with string keys and
store or retrieve objects of the array type.  What doesn't work though
is retrieving or even iterating over the keys.  For this reason I
wrote a terrible hash table implementation using regular arrays.

Other
-----

I'm not impressed by the compiler.  It doesn't catch things like
missing returns, blatant type system abuse and OOP mistakes that
result in segfaults or thrown assertions.  While many hate Java
backtraces, not having any is worse.  Scoping is most certainly not
lexical and forced me to pick more unique names in a few places.

Summary
-------

I've dragged out this project far too long, but have been happy to
learn that it is possible to implement MAL in a language as weird as
ChucK.  Next time I'll hopefully pick a more featureful language, like
SuperCollider_...

.. [1] The most obvious reason is that files may not contain cyclical
       dependencies, a less obvious one is that due to the approach of
       at most one class per file, one ends up with comically large
       command lines.

.. _I did it again: https://github.com/kanaka/mal/pull/229
.. _ChucK: http://chuck.cs.princeton.edu/
.. _a college assignment: https://github.com/wasamasa/theremin
.. _MAL: https://github.com/kanaka/mal
.. _previously: http://emacsninja.com/posts/implementing-mal.html
.. _a number of bugs: https://github.com/ccrma/chuck/issues/created_by/wasamasa
.. _LicK: https://github.com/heuermh/lick
.. _SuperCollider: https://supercollider.github.io/
