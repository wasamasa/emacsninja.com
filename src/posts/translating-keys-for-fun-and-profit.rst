((title . "Translating Keys For Fun And Profit")
 (date . "2015-09-11 23:59:29 +0200"))

I've got to admit that Vim is pretty clever when it comes to its key
bindings.  Never did I notice that ``TAB`` and ``C-i`` are the same
thing in the terminal, simply because I've always used ``C-i`` in
normal mode for traversing the jump list and ``TAB`` in insert mode
for inserting indentation.  With Emacs it's quite different.  ``TAB``
doesn't just insert a preset amount of indentation, no, it's usually
bound to a command that adjusts the indentation of the current line or
region to be just about right.  Therefore this key makes sense outside
of Evil's insert state, simply because it is a convenient way of
fixing the indentation of a piece of code.  That's why one of the
first tweaks to my Evil setup was unbinding ``TAB`` (and ``SPC`` and
``RET``) in its motion and visual state map.  Too bad this gets rid of
``C-i`` for jumping forward.  Surely it's possible to have my cake and
eat it?

Turns out it is if I use the GUI version as it distinguishes between
the key symbol ``<tab>`` which is exclusive to the tab key and
``TAB``, a key code both ``C-i`` and ``TAB`` can resolve to unless
``<tab>`` has been bound.  My first attempt did look like this:

.. code:: elisp

    (with-eval-after-load 'evil-maps
      (define-key evil-motion-state-map (kbd "TAB") 'evil-jump-forward))

This undoes my earlier unmapping, simply because in modes where
``<tab>`` hasn't been bound yet, both ``TAB`` and ``C-i`` will now
jump forward.  Back to square one.

After studying the Emacs Lisp manual a good bit, I've learned Emacs
comes with not one, not two, but three special keymaps specially
designed for translating keypresses, sorted in order of lookup:

``input-decode-map``
    Mainly used for working around terminal-specific oddities.
    Terminal-local.

``local-function-key-map``
    Turns unbound key symbols into more preferable keys.
    Terminal-local.  ``function-key-map`` is the global variant.

``key-translation-map``
    Generic map for translating one key into another.  The manual
    recommends using it for self-inserting keys...

Since I'm neither interested in working around the peculiarities of
terminal-local keymaps nor translating unbound symbols,
``key-translation-map`` it is!  Any of its bindings can either take a
key or a function receiving a ``PROMPT`` argument you're unlikely to
use:

.. code:: elisp

    (defun evil (prompt)
      (if prompt (kbd "C-c") (kbd "C-x")))

    (define-key key-translation-map (kbd "C-c") 'evil)

Good luck figuring out why ``C-x C-c`` doesn't work although ``F1 k
C-x C-c`` shows a correctly looked up command!

Back to the original problem.  First of all, I need something less
ambiguous than ``TAB`` and morally equivalent to ``<tab>``.
Evaluating ``(kbd "<tab>")`` confirms that this key binding is a
vector holding ``tab`` as its only element.  Therefore it should be
fine to use ``(kbd "<C-i>")`` for translation purposes.

Next, the key shouldn't be translated unconditionally.  A check
whether it's not part of a longer key sequence and used in Evil's
normal state should suffice:

.. code:: elisp

    (defun my-translate-C-i (_prompt)
      (if (and (= (length (this-command-keys-vector)) 1)
               (= (aref (this-command-keys-vector) 0) ?\C-i)
               (bound-and-true-p evil-mode)
               (eq evil-state 'normal))
          (kbd "<C-i>")
        (kbd "TAB")))

With the translation function returning ``<C-i>`` for the desired
special case, it can be bound to the correct action in one of Evil's
many keymaps:

.. code:: elisp

    (define-key key-translation-map (kbd "TAB") 'my-translate-C-i)

    (with-eval-after-load 'evil-maps
      (define-key evil-motion-state-map (kbd "<C-i>") 'evil-jump-forward))

-----

Evil is using a similiar trick for a different purpose: Distinguishing
the Escape key from key bindings involving Meta.  Vim avoids this
problem again by not using the Meta key in any of its stock key
bindings.  If you hit, say ``M-O`` in insert mode, this will exit
insert mode, open a line above the current one and enter insert
mode again [1]_.  In Emacs hitting ``ESC`` will either trigger
``keyboard-escape-quit`` or allow for entering a command containing a
meta key in an alternative manner.

Their implementation differs from mine in a few important aspects:
First of all, ``input-decode-map`` is used to allow for doing the
translation in terminals only.  This results in a minor inconvenience:
The binding needs to be done for every terminal with the terminal
selected.  Second, ``evil-esc`` checks for more than just the state
and length of the binding: It waits for a customizable amount of time
to ensure the key event wasn't part of a key chord.  Finally, they
cannot make use of a function like I did because if the translation
cannot be applied, translating with the original key sequence will
screw up more complex chords.

So, how did they solve this?  Well, ``define-key`` accepts more
definitions than just keyboard macros, symbols and commands.  It holds
part of the secret of how one defines new menu bindings, simply
because in Emacs a menu item is just another key binding with its own
set of rules.  One of them is a ``:filter`` property that takes a
function for translating a ``MAP`` argument to something different,
including another key binding.  In case the translation doesn't end up
being chosen, ``MAP`` can be simply returned.  I'm not sure whether
this couldn't be emulated with one of the many functions returning the
keys associated to the current command, but anyway, `it`_ works pretty
well as is.

.. [1] Some people even abuse this feature on a regular basis which
       led to `a bug report`_ on Evil's issue tracker by someone
       seeking to have it in Emacs as well!

.. _it: https://bitbucket.org/lyro/evil/src/b12fd659f1affadcef74fb882c4d6912512d4692/evil-core.el?at=default&fileviewer=file-view-default#evil-core.el-552
.. _a bug report: https://bitbucket.org/lyro/evil/issues/549/holding-down-alt-in-insert-mode-doesnt
