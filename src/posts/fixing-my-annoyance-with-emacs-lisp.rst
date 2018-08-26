((title . "Fixing My #1 Annoyance With Emacs Lisp")
 (date . "2018-08-26 20:12:05 +0200")
 (emacs? . #t))

Ah, Emacs Lisp.  There are many reasons for loving and hating it.  I
disagree with most people name when they argue why the language sucks
[1]_, for me it's mostly two things that end up mattering in practice:

1. The APIs are terrible.  Font-locking is an enigma.  It's common for
   packages to use synchronous APIs because it's far easier to do than
   The Right Thing™.  Moving through buffers and editing them makes
   for incomprehensible *and* stateful code.  I could go on, but most
   of these can be mitigated by writing your own APIs as you figure
   things out.  This is not what this blog post is about.
2. There is no namespace or module system.  This means that every
   global identifier could end up clashing with another one unless you
   emulate namespacing by adding a unique prefix.  While this could be
   fixed, it's unlikely to happen [2]_.  Interestingly enough this
   situation is similar to C, but worse as there's no visibility
   control, only the convention of using a double dash for global
   identifiers not considered public.  This annoys me as I have to
   type out a potentially long prefix every time.  This is what this
   blog post is about.

I initially considered one of the namespace packages.  It would make
for as little typing as possible, however this would require an
additional dependency and break my existing workflows.  Therefore I
went for the alternative route, writing a command that inserts the
package prefix of the current buffer at point.  Bind that command to
an easily reachable key binding and you'd save nearly as much effort
with typing.

.. code:: elisp

    (defvar-local my-current-package-prefix nil)

    (defun my-ensure-trailing-dash (string)
      (if (and (not (zerop (length string)))
               (not (= (aref string (1- (length string))) ?-)))
          (concat string "-")
        string))

    (defun my-guess-current-package-prefix (arg)
      (save-excursion
        (goto-char (point-min))
        (if (and (not arg)
                 (re-search-forward "^(defgroup \\(\\w+\\)" nil t))
            (setq my-current-package-prefix
                  (my-ensure-trailing-dash (match-string 1)))
          (setq my-current-package-prefix
                (my-ensure-trailing-dash
                 (read-string "Package prefix: "
                              my-current-package-prefix))))))

    (defun my-insert-current-package-prefix (arg)
      (interactive "P")
      (when (or (not my-current-package-prefix) arg)
        (my-guess-current-package-prefix arg))
      (insert my-current-package-prefix))

    (with-eval-after-load 'elisp-mode
      (define-key emacs-lisp-mode-map (kbd "C-.")
                  'my-insert-current-package-prefix))

Guessing the prefix is done by looking for a ``(defgroup ...)`` form
which is a good enough indicator for a prefix [3]_.  In case it's not
given, the above code prompts for a prefix and allows resetting it
with a prefix argument.  The trickiest part is ensuring the prefix
ends with a dash.  You could optimize this even further by looking
whether a prefix has already been inserted, but honestly, undoing the
change is simple enough.

Let's see whether this reignites my drive to write more Emacs
packages...

.. [1] Who cares if it's slow?  Who cares about the lack of regex
       literals?  Yes, it's not <insert your favorite language>.
       Despite all of this people wrote lots of it, far more than any
       of the haters would.  Feel free to dream about an Emacs
       rewritten in something else, but it's going to stay a pipe
       dream if that's all you do.  The topic deserves a separate blog
       post because it's a common phenomenon in the Emacs community to
       place irrational hopes in a re-implementation to succeed the
       status quo.
.. [2] The topic came up on emacs-devel before, the main problem is
       that the tooling would need to be updated.  Simple workflows
       the core team is used to (such as grepping the qualified name)
       would completely break apart.
.. [3] An even better indicator would be the ``:prefix`` option inside
       ``(defgroup ...)``, but let's not go overboard.
