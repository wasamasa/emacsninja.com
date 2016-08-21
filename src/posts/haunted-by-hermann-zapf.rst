((title . "Haunted by Hermann Zapf")
 (date . "2016-08-21 19:06:46 +0200"))

Linux gives me, for worse or better, the ability to coerce nearly
every application into picking any font.  The respective component in
`its font stack`_ is the aptly named Fontconfig_ software.  This is
good because some applications insist on fonts I don't like and bad
because sometimes my configuration does the wrong thing, resulting in
bizarre text rendering problems.

One of those has plagued me for months.  When using i3lock_, the
unlock indicator used a swirly, cursive font from `the TeX Gyre
project`_ instead of my default sans-serif font.  I've eventually
identified it as `TeX Gyre Chorus`_ which appears to be a libre
variant of `ITC Zapf Chancery`_.  Why would i3lock ever get the idea
to use *that* of all the fonts?  It didn't make much sense, the mere
act of making TeX fonts available to the rest of the system shouldn't
do something this drastic...

Eventually I've had the idea to use the ``FC_DEBUG`` environment
variable with 4096 as value, this revealed that the query was empty or
in other words, no font was set at all.  For some inexplicable reason,
``fc-match ""`` returns something entirely different.

Thanks to Cairo_ and its simple API, it didn't take me long `to figure
out a fix`_.  While it recommends to use Fontconfig directly for
serious usage, I'd rather not.  Just see for yourself at
FcPatternBuild_ and recoil in horror.

.. _its font stack: http://behdad.org/text/
.. _Fontconfig: https://www.freedesktop.org/wiki/Software/fontconfig/
.. _i3lock: https://github.com/i3/i3lock
.. _the TeX Gyre project: http://www.gust.org.pl/projects/e-foundry/tex-gyre
.. _TeX Gyre Chorus: http://www.tug.dk/FontCatalogue/texgyrechorus/
.. _ITC Zapf Chancery: https://en.wikipedia.org/wiki/ITC_Zapf_Chancery
.. _Cairo: https://www.cairographics.org/manual/cairo-text.html
.. _to figure out a fix: https://github.com/i3/i3lock/pull/89
.. _FcPatternBuild: https://www.freedesktop.org/software/fontconfig/fontconfig-devel/fcpatternbuild.html
