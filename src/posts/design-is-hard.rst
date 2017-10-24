((title . "Design Is Hard")
 (date . "2017-10-20 11:34:31 +0200")
 (emacs? . #t))

This isn't about the pixel pushing kind of design, but the engineering
one.  Given a problematic matter, what choices do you make to create a
tool that enables its user to effectively interact another object?
More importantly, how do you deal with choices that are hard to
rectify afterwards?  While this is going to be a rant, the subject is
one of my more popular Emacs packages, Shackle_.  I thought the 1.0.0
release of it with a new debugging facility to make troubleshooting
easier is just the right moment to ponder a bit about those choices I
made and why I regret some of them.

You may wonder "Wait, what is wrong with Shackle?  It has over a
hundred stars of GitHub, a few thousand downloads on MELPA, dozens of
people using it in their init files and a handful of people
recommending it to others.".  While all of this is true, it's not all
roses.  I occasionally get issues from users that don't understand it
at all and I can't really blame them.  There is a fundamental mismatch
going on here because all this package does is hijacking the
``display-buffer-alist`` variable to invent a similar, but not quite
as powerful mechanism on top of it.  It's an inherently leaky
abstraction which makes for less than ideal debugging: If it ever
breaks down, you'll have to understand both the abstraction and the
underlying code it's built upon.

This project started off with me not understanding how to use this
variable at all.  In hindsight, this should have been the first
warning signal: If you can't fully understand the problem, don't
expect to solve it in a satisfactory manner.  There are a few glaring
problems with ``display-buffer-alist``:

- The docstring for it is hard to parse.  If a newbie asks how to
  customize the display of a certain buffer and is directed to that
  variable, I couldn't blame them for just giving up on this
  altogether.
- It isn't clear how to display a buffer in a certain way.  I've found
  only one example in the elisp manual so far and it's more about
  ``display-buffer`` than ``display-buffer-alist``.
- Conditions may be buffer names and functions, but not major modes.
  This is rather annoying as it means you'll have to write a function
  to check the major mode yourself.  While this is far from fool-proof
  (the code setting up the buffer may enable the desired major mode
  only after displaying it), it works in many cases.
- If your customization of ``display-buffer-alist`` contains a call to
  a function that errors out, the display of that buffer will fail.
  This is particularly annoying if you have a catch-all rule there
  that prevents the source debugger window from appearing, something I
  mostly ran into while developing Shackle.  While you can use ``M-:
  (setq display-buffer-alist nil)``, it's relatively annoying to do
  so.
- The default behavior is rather inscrutable and mostly, but not only
  determined by ``display-buffer-fallback-action``.  Worse, some
  packages rely on the default behavior just to fail with
  customizations to ``display-buffer-alist``.

Now, does Shackle do better?  Well, it does in some ways while being
worse in others:

- Conditions are interpreted as buffer names (if a string) or modes
  (if a symbol) or a list of either.  While this is convenient, the
  original design had the issue of making it impossible to match by
  regex or use a custom function, so I added a ``:regex`` modifier to
  the action (which is just wrong because it changes all of them to
  match by regex) and interpret a list starting with ``:custom`` as a
  function which isn't nice either.  Judging by GitHub's search
  there's about three users of this functionality, with the most
  prolific one being doom_.
- Shackle tries being easier to understand with regards to actions by
  abolishing the alist approach and instead going for a flat plist.
  There is no hierarchy whatsoever which turned out to be a mistake,
  people didn't understand that there were keywords with
  mutually-exclusive behavior, keywords that modified other keywords
  and keywords that work universally.  I've had feature requests where
  I was asked to allow to combine keywords more flexibly, to explain
  how the whole thing works and most surprisingly, to provide a
  grammar of the implemented language.  The latter found its way into
  the README and is more confusing than helpful IMO.  If you want to
  understand the behavior, you're best off with heading to the source.
  I consider this to be the ultimate proof of failing at its design.
- It's way harder to shoot yourself in the foot, in case you do you
  can always bail out with ``M-x shackle-mode`` and revert to vanilla
  Emacs behavior.
- The mere act of enabling Shackle will subtly change the default
  behavior of displaying buffers.  The reason for this is
  ``shackle--display-buffer-popup-window`` which tries to do something
  sensible, but will never behave like the original.
- I've added a feature that doesn't display a window differently, but
  rather modifies the window parameter.  Admittedly it makes things
  more convenient because you'd otherwise need a second package to
  achieve the same effect, but it's the main reason for display of
  buffers intended to not be selected to have weird side effects.
- Debugging Shackle not working as expected is rather tricky.  In the
  best case you'll need to look at the source code of a package to
  check whether it's using ``display-buffer`` or a function using it
  internally (like ``pop-to-buffer``, ``pop-to-buffer-same-window``,
  ``switch-to-buffer-other-window``, etc.).  In the worst case you'll
  need to debug the part of the package displaying such windows or
  Shackle itself while it tries matching conditions and applying
  actions.  I've added a tracing mode to make the former easier, but
  the inherent leaky abstraction remains.
- While Shackle stayed mostly the same, Emacs gained new capabilities
  for ``display-buffer-alist``.  There isn't nearly as much reason for
  using Shackle now, other than laziness.  Other people reached the
  same conclusion_ that it's worth investing some of your time in
  customizing ``display-buffer-alist``.

The bottom line is that I'm not happy with Shackle's design, but am
wise enough to `keep it`_ as is and not do any more invasive changes.
My happiness (or the lack of) isn't worth risking the happiness of its
users.

.. _Shackle: https://github.com/wasamasa/shackle
.. _doom: https://github.com/hlissner/doom-emacs/blob/master/core/core-popups.el
.. _conclusion: https://web.archive.org/web/20160409014815/https://www.lunaryorn.com/2015/04/29/the-power-of-display-buffer-alist.html
.. _keep it: http://blog.npmjs.org/post/141577284765/kik-left-pad-and-npm
