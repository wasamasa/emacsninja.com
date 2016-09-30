((title . "Small-time Patching")
 (date . "2016-09-22 23:13:35 +0200")
 (updated . "2016-09-30 17:58:23 +0200"))

**Update**: Added the improved backtrace

**Update**: `Merged <http://git.savannah.gnu.org/cgit/emacs.git/commit/?id=d1890a3a4a18f79cabf4caf8d194cdc29ea4bf05>`_!

Today ``#emacs`` reminded me of an oddity in Emacs I've sort of
learned to live with:  Backtraces are, well, see for yourself:

.. code::

    Debugger entered--Lisp error: (wrong-type-argument number-or-marker-p t)
      +(1 t)
      eval((+ 1 t) nil)
      eval-expression((+ 1 t) nil)
      call-interactively(eval-expression nil nil)
      command-execute(eval-expression)

I can live with errors printed as a list.  What I can't live with is
none of the call stack lines being printed as a S-Expression...  To
fix this, one must dive a bit deeper than usual as only the debugger
porcelain is implemented in Emacs Lisp.  Its workhorse is
``backtrace`` in ``eval.c``:

.. code:: c

    DEFUN ("backtrace", Fbacktrace, Sbacktrace, 0, 0, "",
           doc: /* Print a trace of Lisp function calls currently active.
    Output stream used is value of `standard-output'.  */)
      (void)
    {
      union specbinding *pdl = backtrace_top ();
      Lisp_Object tem;
      Lisp_Object old_print_level = Vprint_level;

      if (NILP (Vprint_level))
        XSETFASTINT (Vprint_level, 8);

      while (backtrace_p (pdl))
        {
          write_string (backtrace_debug_on_exit (pdl) ? "* " : "  ");
          if (backtrace_nargs (pdl) == UNEVALLED)
            {
              Fprin1 (Fcons (backtrace_function (pdl), *backtrace_args (pdl)),
                      Qnil);
              write_string ("\n");
            }
          else
            {
              tem = backtrace_function (pdl);
              Fprin1 (tem, Qnil);   /* This can QUIT.  */
              write_string ("(");
              {
                ptrdiff_t i;
                for (i = 0; i < backtrace_nargs (pdl); i++)
                  {
                    if (i) write_string (" ");
                    Fprin1 (backtrace_args (pdl)[i], Qnil);
                  }
              }
              write_string (")\n");
            }
          pdl = backtrace_next (pdl);
        }

      Vprint_level = old_print_level;
      return Qnil;
    }

Despite the rather unusual look and weird naming, it's not too hard to
find the culprit.  Most of the time is spent inside a loop that walks
through the call stack, accesses the top-most function and args with
``backtrace_function`` and ``backtrace_args``, prints a lisp object
with ``Fprin1`` (which is just another way to use ``prin1`` from C
code) and writes out normal strings with ``write_string``.  ``Qnil``
refers to the global ``nil`` symbol, ``tem`` is a naming convention
for a temporary variable.  It should be sufficient to print the
opening paren first, then the function, a space and proceed normally
from that point:

.. code:: c

    tem = backtrace_function (pdl);
    write_string ("(");
    Fprin1 (tem, Qnil);	/* This can QUIT.  */
    write_string (" ");

Time to recompile Emacs with ``make``, boot the binary with
``src/emacs -Q``, and trigger a backtrace with ``M-: (+ 1 t)``.
Unfortunately that does not yield a prettier backtrace yet, but rather
a ``Search failed: "\n debug("``.  As the debugger is busted, I had to
resort to ``ag`` to find where exactly the breakage occurs.  It's not
too hard to fix, mind you, all you have to do is to patch
``debugger-setup-buffer`` in ``debug.el`` to search for ``"\n
(debug"`` instead.

The result is the following teensy patch:

.. code:: diff

    diff --git a/lisp/emacs-lisp/debug.el b/lisp/emacs-lisp/debug.el
    index 22a3f39..4020620 100644
    --- a/lisp/emacs-lisp/debug.el
    +++ b/lisp/emacs-lisp/debug.el
    @@ -279,7 +279,7 @@ That buffer should be current already."
       (goto-char (point-min))
       (delete-region (point)
     		 (progn
    -		   (search-forward "\n  debug(")
    +		   (search-forward "\n  (debug")
     		   (forward-line (if (eq (car args) 'debug)
                                          ;; Remove debug--implement-debug-on-entry
                                          ;; and the advice's `apply' frame.
    diff --git a/src/eval.c b/src/eval.c
    index 72facd5..e32e7a1 100644
    --- a/src/eval.c
    +++ b/src/eval.c
    @@ -3409,8 +3409,9 @@ Output stream used is value of `standard-output'.  */)
           else
     	{
     	  tem = backtrace_function (pdl);
    -	  Fprin1 (tem, Qnil);	/* This can QUIT.  */
     	  write_string ("(");
    +	  Fprin1 (tem, Qnil);	/* This can QUIT.  */
    +	  write_string (" ");
     	  {
     	    ptrdiff_t i;
     	    for (i = 0; i < backtrace_nargs (pdl); i++)

And an IMHO vastly improved backtrace:

.. code::

    Debugger entered--Lisp error: (wrong-type-argument number-or-marker-p t)
      (debug error (wrong-type-argument number-or-marker-p t))
      (+ 1 t)
      (eval (+ 1 t) nil)
      (eval-expression (+ 1 t) nil)
      (funcall-interactively eval-expression (+ 1 t) nil)
      (call-interactively eval-expression nil nil)
      (command-execute eval-expression)

Not sure whether to bother submitting this...  Let me know what you
think!
