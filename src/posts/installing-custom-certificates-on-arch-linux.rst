((title . "Installing Custom Certificates on Arch Linux")
 (date . "2018-08-23 09:13:43 +0200")
 (emacs? . #f))

One day a 1337 h4xx0r at the hackerspace [1]_ asked me whether I could
relay a message to the hackint IRC network.  This had one catch, for
some reason the hackint operators insist that you only connect via TLS
and use their self-signed intermediate and root certificates.  I did a
cursory verification [2]_, then looked into how to do a system-wide
installation of the two ``.crt`` files.  To my surprise, there is no
Arch Linux wiki article on the topic, so here's a short guide.

.. code:: shell

    % cp *.crt /etc/ca-certificates/trust-source/anchors
    % update-ca-trust

Merely copying the files into the destination directory isn't enough,
you need to run ``update-ca-trust`` to pound them into a form OpenSSL
and friends can deal with.  If everything went correctly, you'll find
new symlinks in ``/etc/ssl/certs`` with the issuer's name in them.
FWIW, something similar happens on every system update thanks to
Pacman's ``update-ca-trust`` hook.  Check
``/usr/share/libalpm/hooks/update-ca-trust.hook`` for details.

.. [1] As hard as it is to believe, I occasionally meet actual
       hackers there.  The kind that studies exploits, knows their
       way around shell code and breaks into computer systems.
.. [2] They're quick to point out that comparing hashes doesn't help
       you in case you get MITM'd.  I tried verifying the
       certificate's signature with GPG, but failed as I didn't have
       anything useful imported in my local trust store.
