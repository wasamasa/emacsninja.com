((title . "Brave New World")
 (date . "2016-11-17 22:17:26 +0100")
 (updated . "2016-11-21 10:32:43 +0100"))

**Update**: Finally figured out the layout after digging a bit more
into the sources, it's a QWERTY-UK (see
``devices/rpi2/uspi/include/uspios.h``).  Looks like I'll have to
modify the bundled `USPI library`_ to include a QWERTY-US layout
before I can make any progress on keyboard remapping in Lisp...

I believe I've found an even greater time sink than writing Lisp
interpreters for fun.  Long time ago, I've read an encouraging blog
post on `the future of the LispM`_, not expecting to find an
implementation of the ideas presented therein.  Turns out I was wrong
about that.  Meet `Interim OS`_!

In case you're wondering why you should possibly care:

- Small and readable codebase (most of the code is device drivers for
  the Raspberry Pi)
- Simple to hack on
- Plan9-style APIs
- Minimal Lisp dialect
- Runs on your favorite desktop OS in hosted mode, that is, safely
  contained to a terminal with the ability to spawn graphical windows
- Runs on bare metal (Raspberry Pi 2)

Getting it to run in hosted mode is simple enough, so I won't explain
it here.  Booting on bare metal however is a different story, so here
we go:

.. code:: shell

    $ git clone https://github.com/mntmn/interim
    $ cp interim/docs/interim-0.1.0-rpi2.tgz ./
    $ bsdtar -xf interim-0.1.0-rpi2.tgz # cry me a river
    % mkdir /media/boot
    % mount /dev/sdXN /media/boot
    % cp release-rpi2/* /media/boot/
    % rm /media/boot/cmdline.txt
    % umount /media/boot

- Plug in the SDHC card, a HDMI monitor and a USB keyboard
- Optionally: Plug in a network cable and/or a USB mouse
- Power up

You'll be greeted by a "Welcome to Interim OS" and dropped into a
promptless shell.  If you're unlucky, the chosen resolution may be
unreadable, so feel free to retry this process a few times.  The
keyboard layout is hardcoded and somewhere between QWERTY-US and
QWERTZ-DE, something I intend to fix soon_.  For basic usage
instructions, type ``(bytes->str (load "/sd/hello.txt"))`` and hit the
enter key.  Happy hacking!

.. _the future of the LispM: https://www.arrdem.com/2014/11/28/the_future_of_the_lispm/
.. _Interim OS: https://github.com/mntmn/interim
.. _soon: https://github.com/mntmn/interim/pull/13
.. _USPI library: https://github.com/rsta2/uspi/
