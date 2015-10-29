((title . "Zoning Out With Nyan Cat")
 (date . "2015-10-28 21:31:59 +0100"))

For those of you who haven't heard of ``zone`` yet, I urge you to give
it a try with ``M-x zone``.  If you like what you see, ``M-x
zone-when-idle`` makes for a screensaver-like experience.

It shouldn't surprise anyone that Emacs has facilities for playing
tricks with your buffer text while you aren't looking.  I've always
been wondering though whether you couldn't do better and invent a
graphical screensaver with it.  After all, an image in Emacs is just a
string with the appropriate ``display`` property set with the display
engine taking care of the rest.  A week of tinkering later I finally
present you `zone-nyan`_.  Combine with `nyan-mode`_ and
`nyan-prompt`_ for the holy nyan cat trinity in Emacs!

You might be wondering what took me so long.  I've expected turning
the original nyan cat animation into something roughly equivalent to
require an evening, but did later find out that there isn't any actual
pixel art of it, just upscaled images of it with fractional pixel
widths.  GIMP in all of its glory doesn't even support fractional
pixel grids or rulers, so I had to use Photoshop for this one and
transcribe the positions of all the moving objects, then spent the
remaining time on thinking up a good way to encode these in a less
brain-dead manner.

I did decide on giving SVG another try, simply because I was
underwhelmed with generating Bitmaps larger than an icon with Emacs
Lisp.  Dealing with a XML string is yucky, dealing with a nested list
however is close to perfect if you've got something like esxml_ at
hand to convert it to a XML representation.  I've scrapped my previous
approach of designing one big template where I fill in all the parts
and adapted the approach from `Land of Lisp`_ instead where common
building blocks are turned into function calls.  That way you almost
get a DSL for arranging the parts of your image and can easily modify
parts of it.

Thanks to `SVG transforms`_ it's no issue to upscale your image to
something blockier while keeping grid coordinates.  Figuring out how
to clip the canvas was a bit trickier, but turned out to be simpler
than expected with `the clip-path attribute`_.  With this combination
you can resize the window and it will repaint to an appropriately
scaled variant, even while the animation is playing.  I'm pretty sure
now that SVG makes for the most sophisticated way of doing graphics in
Emacs, perhaps I'll even go as far as rewriting retris_ to make use of
it instead and drop the performance hacks...

.. _zone-nyan: https://github.com/wasamasa/zone-nyan
.. _nyan-mode: https://github.com/TeMPOraL/nyan-mode
.. _nyan-prompt: https://github.com/PuercoPop/nyan-prompt
.. _esxml: https://github.com/tali713/esxml
.. _Land of Lisp: http://landoflisp.com/svg.lisp
.. _SVG transforms: https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/transform
.. _the clip-path attribute: https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/clip-path
.. _retris: https://github.com/wasamasa/retris
