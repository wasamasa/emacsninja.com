((title . "Trapping Attackers With Nyan Cat")
 (date . "2017-01-22 14:54:48 +0100"))

In case you haven't done it yet, I strongly recommend you to give
``telnet nyancat.dakko.us`` a try.  If you don't have a telnet client
ready, head over to `nyancat.dakko.us <http://nyancat.dakko.us/>`_
instead and enjoy the pretty pictures.

I've wondered for quite some time whether it would be possible to run
the same thing on a SSH server.  It turns out that it's not too hard
to do as not only `Kevin Lange's creation`_ can be run in a TTY, but
OpenSSH allows you to do something else than giving you a shell after
a successful authentication attempt.

First of all you'll need to install and test the ``nyancat`` program:

.. code:: shell

    % pacman -S nyancat
    $ nyancat

To restrict the impact of the public-facing service, I decided to
create a new user for it and run a separate SSH daemon with its own
config and service file:

.. code:: shell

    % useradd -m -s /bin/sh anonymous
    % cp /usr/lib/systemd/system/sshd.service /etc/systemd/system/nyanpot.service
    % cp /etc/ssh/sshd_config /etc/ssh/nyanpot_sshd_config

Relevant changed bits in the service file:

.. code:: ini

    [Unit]
    Description=OpenSSH Honeypot
    ...
    [Service]
    ExecStart=/usr/bin/sshd -D -f /etc/ssh/nyanpot_sshd_config
    ...

The SSH config is a bit special as it locks out everyone who isn't the
anonymous user:

.. code::

    Port 22
    PermitRootLogin No
    PermitTTY No
    PasswordAuthentication No
    X11Forwarding No
    AllowTcpForwarding No
    Match User anonymous
        PasswordAuthentication Yes
        PermitTTY Yes
        ForceCommand nyancat

You can test the service with ``systemctl start nyanpot.service`` and
logging in (ideally from a different system) as the anonymous user.
If everything works fine, enable the service permanently with
``systemctl enable nyanpot.service``.  My honeypot is available via
``ssh anonymous@brause.cc`` (PW: anonymous).  Enjoy!

.. _Kevin Lange's creation: https://github.com/klange/nyancat
