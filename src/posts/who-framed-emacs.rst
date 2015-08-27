((title . "Who Framed Emacs?")
 (date . "2015-08-27 14:10:54"))

To be honest, I don't really like frames [1]_ in Emacs.  One reason is
how bad they play with i3_, my preferred tiling window manager.  Any
call of ``raise-frame`` fails drawing my attention to it unless the
frame has been on the same workspace already which makes it rather
useless for me.  The other reason is that unlike windows [2]_ frames
do not have first-class support.  It is considerably hard convincing
Emacs to prefer frames over windows, so hard in fact that there is `a
package for it`_ with an `optional dependency list from hell`_.
Supporting frames programmatically isn't simple for me either.  I try
very hard to do so in my own packages, but given my aforementioned
setup, I simply won't run into more subtle bugs involving them.

That being said, there are situations where I'm OK with frames.  Emacs
does always create an initial frame, no matter whether you run a
normal instance, use ``--batch`` or go for the `Emacs daemon`_.  This
can be easily verified with ``(> (length (frame-list)) 0)``.
Additionally to the initial frame, extra frames can be created.  I
usually don't do this by hand, so this only happens to me when
``emacsclient`` requests a new one.

The distinction between the initial frame and every other frame
becomes quite important as soon as you wish to run code after a frame
was created.  I do this for three things, adjusting the default
fontset to display Emoji properly, resetting the
``ansi-color-names-vector`` variable for my theme and handling ``C-i``
independently from ``TAB``.  My first attempt is making use of
``after-make-frame-functions``:

.. code:: elisp

    (defun my-modify-frame ()
      ...)

    (add-hook 'after-make-frame-functions 'my-modify-frame)

This will not work because ``after-make-frame-functions`` is a
so-called `abnormal hook`_, but will nevertheless not error out.
Unlike other hooks every function in it is run with an argument, in
this case, the created frame.  If you rely on the frame in question to
be selected (like, for the theme case), you must select it explicitly:

.. code:: elisp

    (defun my-modify-frame (frame)
      (with-selected-frame frame
        ...))

    (add-hook 'after-make-frame-functions 'my-modify-frame)

Much better.  For some reason though, it only seems to work in
``emacsclient`` and on creation of subsequent frames in normal Emacs
instances.  A quick search of the Emacs Lisp sources reveals that
``make-frame`` ends up running the hook, but is ``make-frame``
actually used for the initial frame?  ``command-line`` in
``startup.el`` suggests it does use ``frame-initialize`` which uses
``make-frame`` internally, but any attempts at getting a replacement
function displaying debug information fails for me, so I'm just going
to assume the initial frame is very special and requires a stupid
workaround: Unconditionally executing the function *and* adding it to
the hook.  To make this a bit more convenient, I'll make the ``frame``
parameter optional.

.. code:: elisp

    (defun my-modify-frame (&optional frame)
      (with-selected-frame (or frame (selected-frame))
        ...))

    (my-modify-frame)
    (add-hook 'after-make-frame-functions 'my-modify-frame)

And there you go.  This time with the actual code I'm running:

.. code:: elisp

    (defun my-filter-C-i (map)
      (if (and (bound-and-true-p evil-mode) (eq evil-state 'normal))
          (kbd "<C-i>")
        map))

    (with-eval-after-load 'evil-maps
      (define-key evil-motion-state-map (kbd "<C-i>") 'evil-jump-forward))

    (defun my-modify-frame (&optional frame)
      (with-selected-frame (or frame (selected-frame))
        (set-fontset-font "fontset-default" nil "Symbola" nil 'append))
        (load-theme 'my-solarized t)
        (define-key input-decode-map [?\C-i]
          `(menu-item "" ,(kbd "TAB") :filter my-filter-C-i)))

     (my-modify-frame)
     (add-hook 'after-make-frame-functions 'my-modify-frame)

Next up: Explaining what the hell I'm doing there with Evil_.

.. _i3: http://i3wm.org/
.. _RMS agrees they'd better be called panes: https://lists.gnu.org/archive/html/emacs-devel/2014-01/msg00496.html
.. _a package for it: http://www.emacswiki.org/emacs/OneOnOneEmacs
.. _optional dependency list from hell: http://www.emacswiki.org/emacs/OneOnOneEmacs#toc5
.. _Emacs daemon: https://www.gnu.org/software/emacs/manual/html_node/emacs/Emacs-Server.html
.. _abnormal hook: https://www.gnu.org/software/emacs/manual/html_node/emacs/Hooks.html
.. _Evil: https://bitbucket.org/lyro/evil/wiki/Home

.. [1] This unfortunate naming choice is the result of Emacs predating
       more common naming systems.  The rest of the world refers to
       them as windows.
.. [2] Another unfortunate naming choice, even `RMS agrees they'd
       better be called panes`_.
