((title . "On Transcending Borders")
 (date . "2016-01-22 22:17:09 +0100")
 (emacs? . #t))

Reddit_ reminded me today that the infamous ``xwidget`` branch has
been finally merged into the ``emacs-25`` branch.  As usual there
hasn't been any critical or even informational reports about what
using it is like, so I decided to see for myself what the fuss is
about.  Building it is easy enough:

.. code:: shell

    git checkout emacs-25
    ./configure --with-x-toolkit=gtk3 --with-xwidgets
    make

You can then use ``M-x xwidget-webkit-browse-url`` and must enter a
*full* URL.  ``www.gnu.org`` will only display a blank image,
``http://www.gnu.org/`` on the other hand...

.. image:: /img/xwidgets-gnu-thumb.png
   :target: /img/xwidgets-gnu.png

That's more like it.

According to html5test.com_ things aren't looking too bad:

.. image:: /img/xwidgets-html5-thumb.png
   :target: /img/xwidgets-html5.png

I can even watch YouTube!

.. image:: /img/xwidgets-yt-thumb.png
   :target: /img/xwidgets-yt.png

Once you do more than basic navigation by clicking links, things begin
falling apart.  Every time I interacted with a video, the audio volume
was turned all the way up and the widget flickered.  Searching for
another video didn't work the way you'd expect it either.  You can
click a text form just fine, but any keys you press are interpreted as
potential Emacs commands instead.  Instead you hit ``RET`` and get the
familiar ``read-string`` prompt.  Finishing your input will place it
into the text form.  Except when it doesn't:

.. image:: /img/xwidgets-wp-thumb.png
   :target: /img/xwidgets-wp.png

To keep things short, the integration so far is as basic as it gets.
On top of that you get reminded once in a while that this is still a
hack: Change the window size or scroll off and you get to see extra
buffer text reminding you to readjust the widget size with ``a``.
What it doesn't tell you is that you may need to scroll up as well...

Personally, I find the opposite approach of starting out with
webkit and piling JavaScript on top of it until it resembles what
you're after much more promising.  It may involve less Lisp than
you've hoped for, but leads to significantly more usable results.

.. _Reddit: https://www.reddit.com/r/emacs/comments/4241oy/xwidget_branch_has_been_merged_into_emacs_251/
.. _html5test.com: http://html5test.com/
