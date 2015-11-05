((title . "Failing Gracefully")
 (date . "2015-11-05 09:55:38 +0100"))

One of the more annoying problems with Emacs and its init file is that
any error encountered while loading it will prohibit it from
proceeding any further.  There is `a question`_ on `the Emacs
Stackexchange`_, but none of the answers are really satisfactory as
the usage of ``ignore-errors`` or ``with-demoted-errors`` keeps it
from popping up a buffer with the errors.  I'd ideally like the
built-in error reporting to be extended to all errors encountered.
One of the commenters remarks that this level of control is possible,
given a list of individual code snippets to be evaluated.

Come to think of it, this is how my init.org_ does already work!  As I
don't use Org for loading it, I've adapted `Arthur Malabarba's code`_
to find the source blocks and evaluate them one block at a time.  The
basic logic can be summarized as follows:

.. code:: elisp

    (with-temp-buffer
      (insert-file "~/.emacs.d/init.org")
      (goto-char (point-min))
      ;; jump to first relevant header
      (search-forward "\n* Init")
      (while (not (eobp))
        (forward-line 1)
        (cond
         ;; evaluate code blocks
         ((looking-at "^#\\+BEGIN_SRC +emacs-lisp.*$")
          (let ((src-beg (match-end 0)))
            (search-forward "\n#+END_SRC")
            (eval-region src-beg (match-beginning 0))))
         ;; finish on the next level-1 header
         ((looking-at "^\\* ")
          (goto-char (point-max))))))

To report errors, I need to wrap ``eval-region`` into
``condition-case`` and collect data for the error reporting.  If any
errors have been collected after the entire file was processed, a
buffer with a report shall be popped up:

.. code:: elisp

    (let (errors)
      (with-temp-buffer
        (insert-file "~/.emacs.d/init.org")
        (goto-char (point-min))
        ;; jump to first relevant header
        (search-forward "\n* Init")
        (while (not (eobp))
          (forward-line 1)
          (cond
           ;; evaluate code blocks
           ((looking-at "^#\\+BEGIN_SRC +emacs-lisp.*$")
            (let (src-beg src-end)
              (condition-case error
                  (progn
                    (setq src-beg (match-end 0))
                    (search-forward "\n#+END_SRC")
                    (setq src-end (match-beginning 0))
                    (eval-region src-beg (match-beginning 0)))
                (error
                 (push (format "%s for:\n%s\n\n---\n"
                               (error-message-string error)
                               (buffer-substring src-beg src-end))
                       errors)))))
           ;; finish on the next level-1 header
           ((looking-at "^\\* ")
            (goto-char (point-max))))))
      (when errors
        (with-current-buffer (get-buffer-create "*init erorrs*")
          (insert (format "%i error(s) found\n\n" (length errors)))
          (dolist (error (nreverse errors))
            (insert error "\n"))
          (goto-char (point-min))
          (special-mode))
        (display-buffer "*init errors*")))

This could be improved in a number of ways, like by tracking the last
heading and evaluating every single form in the code block on its
own.  Probably not worth it though.

.. _a question: http://emacs.stackexchange.com/questions/669/how-to-gracefully-handle-errors-in-init-file/17818#17818
.. _the Emacs Stackexchange: http://emacs.stackexchange.com/
.. _init.org: https://github.com/wasamasa/dotemacs/blob/master/init.org
.. _Arthur Malabarba's code: http://endlessparentheses.com/init-org-Without-org-mode.html
