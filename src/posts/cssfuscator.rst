((title . "CSSfuscator")
 (date . "2015-10-08 22:17:14 +0200"))

My relationship with CSS is, let's say, complicated.  I know just
enough to style bare-bones websites, yet every time I want to get
fancy I run into this hell of not having any clue what knobs to tweak
to get the desired results, especially if it's got to look the same on
all relevant browsers.  My cure for any hard feelings towards a tool
has been abusing it for something it hasn't been designed for, like
I've been doing way_ too_ often_ with Emacs already.

I do occasionally enjoy `clicking pixels into place`_ and follow other
people doing the same.  Some of them do specialize in small icons and
game sprites, so they usually offer a magnified version of their work,
be it with an extra link or some Javascript to toggle between scaled
versions.  This made me wonder whether you couldn't simply scale the
image itself in the browser without introducing any smoothing.  It
turns out you can with CSS as demonstrated on piCSSel_.

The idea behind this hack is pretty simple.  A block element can have
a ``box-shadow`` property which allows you to put a shadow with a
specific color and position close to it.  As you aren't limited to a
single shadow and can use many of these with distinct colors and
positions, it's possible to arrange them into a picture.  Inspired by
this discovery, I did draw a jar in GIMP_ and wrote some CSS3 by hand
to have an animated version of it on http://brause.cc.

On my way to `ICC 2015`_, it occurred me how nice it would be to not
have to write this CSS by hand again, so I started writing `a CLI
utility`_.  It's using imlib2_ to extract the colors of each pixel,
then turns each of these into a box shadow specification and inserts
them in some CSS and HTML boilerplate which is then written out to
disk.

smugleaf.html_ for instance was generated from `a sprite of a
fictional Pokemon`_ with the following invocation:

.. code:: shell

    cssfuscator -uem -s0.25 -O --html-title=Smugleaf --stylesheet=split \
    --stylesheet-name=smugleaf.css -i smugleaf.gif -o smugleaf.html

You can hit ``C-+`` and ``C--`` for resizing the image up to a certain
limit.  While the rendering isn't perfect yet [1]_, it is
satisfactory.  A side effect of this technique is that it's no longer
possible to download an image of the result.  If you're tempted to use
this tool as an image gallery protection, think again.  This is
probably the most inefficient way to encode an image [2]_, even with
``--optimize`` the resulting files are larger by a factor between 20x
and 2000x.

There's no animation support yet.  imlib2_ does have `an egg`_, but
giflib_ doesn't.  I've been reading the original GIF specifications
and its documentation in order to build `a simple tool`_ and prepare
myself for writing CHICKEN bindings to the library.  Once that's done,
there will be a follow-up post on the gnarly details.

.. [1] Not sure why, but I get white gridlines at some magnifications.
       Another weird thing is that Chromium does allow for resizing
       images with pixel dimensions while Firefox doesn't.
.. [2] Right after HTML obfuscation which would use a HTML table and
       each cell as pixel. See `a speed drawing demo on Youtube`_ for
       someone doing this in Notepad(!) with judicious use of search
       and replace.

.. _way: https://github.com/wasamasa/svg-2048
.. _too: https://github.com/wasamasa/xbm-life
.. _often: https://github.com/wasamasa/retris
.. _clicking pixels into place: https://en.wikipedia.org/wiki/Pixel_art
.. _piCSSel: http://kushagragour.in/lab/picssel-art/
.. _GIMP: https://en.wikipedia.org/wiki/GIMP
.. _ICC 2015: http://wiki.call-cc.org/event/intercontinental-chicken-conference-2015
.. _a CLI utility: https://github.com/wasamasa/cssfuscator
.. _imlib2: https://docs.enlightenment.org/api/imlib2/html/
.. _smugleaf.html: http://brause.cc/smugleaf.html
.. _a sprite of a fictional Pokemon: http://orig13.deviantart.net/3c0b/f/2010/331/7/6/smugleaf_quick_sprite_by_generalcirno-d33r5k1.gif
.. _a speed drawing demo on Youtube: https://youtu.be/FpRcbVXnrds
.. _an egg: http://wiki.call-cc.org/eggref/4/imlib2
.. _giflib: http://giflib.sourceforge.net/index.html
.. _a simple tool: https://github.com/wasamasa/cssfuscator/blob/master/gifinfo.c
