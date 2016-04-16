((title . "ich1 the CSI Killer")
 (date . "2016-04-16 23:04:31 +0200"))

One of the optional parts of `implementing MAL`_ was line editing for
easier interaction with the interpreter.  Most implementations either
load an existing binding to readline_ or make use of it directly with
FFI_ magicks, but neither were an option for me as there simply was no
such library (yet) and Emacs module support isn't `terribly well
documented`_ (just like the C API), not `terribly well looking`_ and
requires a recent build plus a compile-time option to be made use of.
So, not an option other than for `a few enthusiasts`_.

The naïve way of doing event handling wasn't an option either as one
can verify by looking at ``read_minibuf_noninteractive`` and how it
reads a string outside the command loop.  By that time I concluded it
to be impossible to reimplement a fancier variant of it with Emacs
Lisp, yet asked around for help on ``#emacs``.  Truth to be told, I
didn't expect to find any, but then the unlikely happened and `Alain
Kalker`_ proposed a hack crazy enough to work.  One thing led to the
other and after two weeks of hacking I can finally present you
emacsrepl_, the Emacs Lisp REPL you've all been waiting for!  It's
fairly useless at the moment as it doesn't have any redeeming features
(yet), but has been an interesting exercise and could not only make it
into my MAL implementation, but even pave the way for standard input
handling in Emacs...

-----

You may wonder what could possibly be complicated about letting the
cursor dance in the terminal.  As it is with most problems, it's not a
single reason, but rather a combination of unfortunate factors.  In
this case, featuritis, legacy hardware support, arcane documentation
and the curse of existing work being good enough to not bother walking
`a different path`_.  Or maybe it's just people not giving a damn
about how smelly widely used code can be:

.. code:: c

    /* Delete the string between FROM and TO.  FROM is inclusive, TO is not.
       Returns the number of characters deleted. */
    int
    rl_delete_text (from, to)
         int from, to;
    {
      register char *text;
      register int diff, i;

      /* Fix it if the caller is confused. */
      if (from > to)
        SWAP (from, to);

      /* fix boundaries */
      if (to > rl_end)
        {
          to = rl_end;
          if (from > to)
            from = to;
        }
      if (from < 0)
        from = 0;

      text = rl_copy_text (from, to);

      ...
    }

This is just not right.  Fixing API usage mistakes reeks of Windows 95
programming practices.  Even worse if you consider that this function
is not part of the external API and therefore the "confused caller" is
something inside readline itself that prompted this addition.  Why one
would not simply debug the codebase to not require that way of doing
things is beyond me.

Another problem I've got with this is that readline clocks in at about
23k SLOC.  Fortunately I'm not the only one considering that fact a
problem: `Salvatore Sanfilippo`_ wrote linenoise_ as minimal
replacement for it.  The code is clean, very readable and was
consulted extensively for getting the design and implementation of
``emacsrepl`` right.

The program itself can be split into two parts, a shell script
conjuring the spirits of the terminal and the Emacs Lisp code
receiving input and printing output.

-----

To react to every single input of the user, two conditions must be
fulfilled:  Characters are read in raw mode, a terminal state that
deactivates any special-casing that would keep us from detecting a
``C-c`` and characters are read in one at a time.  The former is
typically done in C with the ``termios.h`` family and a menagerie of
flags (which must be undone on exit).  Fortunately it can be done with
the ``stty`` command and trapping exit.

The next problem is getting one character at a time into Emacs.  As
I'm leveraging ``emacs --batch`` to be able to read from standard
input in the first place [1]_, I can only read lines with it, so the
next part of the hack is printing each character on its own line,
piping that into Emacs and ensuring line buffering for it to work as
expected.  This obviously introduces overhead, but nothing noticable
for this usecase.

-----

Now for the Emacs side of things.  Characters are read in
successfully, but not everything of interest can be expressed as a
single byte.  Experimenting with ``cat -A`` reveals that key
combinations involving modifiers are different, even more so special
keys like ``<up>`` and ``<end>``.  These sequences are decoded with a
simple state machine.  The same approach was chosen for UTF-8 data as
anything beyond ASCII is represented with more than one byte, with the
state machine being a faithful port of `prior art`_.

To move the cursor around and update the edited line, one needs to
print out characters, some of which are special and terminal-specific.
I did initially reach out to the ``terminfo`` database, but gave up
quickly due to the lacking documentation on what sequences like
``ich1`` do and unexpected interactions such as typing being messed up
after exiting the REPL.  Fortunately linenoise goes for an easier
approach: Picking a small set of primitives from the nearly
ubiquitiously supported `CSI codes`_ and redrawing with these only.
This works reasonably well at the cost of not being completely
compatible with everything out there [2]_ and should be the slower
approach, but again it isn't an issue in practice.

As the appearance of edited text is manipulated, the underlying text
representation of the user input so far must be kept up to date.  For
this Emacs offers the perfect data structure:  The buffer!  This helps
keeping the code size small as boundaries, contents and the position
of point are tracked for you.  I don't have much to complain here,
save that some essential operations like replacing text must be
implemented as deletion and insertion.  Another benefit of it is that
more complex editing operations (think Paredit_) don't need to be
reimplemented, only the redisplay of them.

.. _implementing MAL: http://emacsninja.com/posts/implementing-mal.html
.. _readline: https://en.wikipedia.org/wiki/GNU_Readline
.. _FFI: https://en.wikipedia.org/wiki/Foreign_function_interface
.. _terribly well documented: http://diobla.info/blog-archive/modules-tut.html
.. _terribly well looking: https://lists.gnu.org/archive/html/emacs-devel/2016-01/msg00222.html
.. _a few enthusiasts: https://lists.gnu.org/archive/html/emacs-devel/2016-03/msg01693.html
.. _Alain Kalker: https://github.com/ackalker
.. _emacsrepl: https://github.com/tetracat/emacsrepl
.. _a different path: https://en.wikisource.org/wiki/The_Calf_Path
.. _Salvatore Sanfilippo: http://invece.org/
.. _linenoise: https://github.com/antirez/linenoise
.. _prior art: http://bjoern.hoehrmann.de/utf-8/decoder/dfa/
.. _CSI codes: https://en.wikipedia.org/wiki/ANSI_escape_code#CSI_codes
.. _Paredit: http://mumble.net/~campbell/emacs/paredit.release

.. [1] Printing messages to standard error is its other exclusive
       ability, it would have helped though to not make it the default
       mode of printing...
.. [2] ``M-x ansi-term`` and DOS, I'm looking at you!  Surprisingly
       enough, the Linux console supports everything implemented so
       far flawlessly.
