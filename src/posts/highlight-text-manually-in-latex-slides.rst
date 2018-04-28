((title . "Highlight Text Manually In LaTeX Slides")
 (date . "2018-04-29 00:58:24 +0200")
 (emacs? . #f))

Sometimes I've been in the situation that I have a text snippet where
a splash of color would explain things in a far easier way than having
to create a visualization or use a laser pointer.  Imagine something
like a backtrace, with different parts highlighted in different
colors.  This thing can be done pretty easily, even when using LaTeX
indirectly, like when compiling it from Org in Emacs.

The following trick relies on using the minted_ package for
highlighting.  It supports an option for embedding LaTeX escapes into
your code snippets.  The documentation shows off a mathematic formula
in a comment, however we can do far more, like using the
``\textcolor`` command from the ``xcolor`` package to insert colored
text.  Have a silly example:

.. code:: latex

    \setminted{escapeinside=||}
    \definecolor{green}{HTML}{218A21}

    ...

    \begin{minted}[]{text}
    |\textcolor{red}{RR}|
    |\textcolor{green}{GG}|
    |\textcolor{blue}{BB}|
    \end{minted}

In an Org file you'd have to do a bit less typing (assuming you
customized Org to always use ``minted``):

.. code:: text

    #+LATEX_HEADER: \setminted{escapeinside=||}
    #+LATEX: \definecolor{green}{HTML}{218A21}

    ...

    #+BEGIN_SRC text
    |\textcolor{red}{RR}|
    |\textcolor{green}{GG}|
    |\textcolor{blue}{BB}|
    #+END_SRC

.. _minted: https://github.com/gpoore/minted
