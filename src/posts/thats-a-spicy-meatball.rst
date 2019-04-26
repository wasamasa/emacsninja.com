((title . "\"That's a spicy meatball!\"")
 (date . "2019-04-26 19:43:39 +0200")
 (emacs? . #f))

QEMU is pretty neat, particularly when combined with KVM for
hardware-accelerated virtualization.  I'm currently using it with an
Ubuntu guest for research purposes and missed a few convenience
features that just magically work in VMware and VirtualBox: Seamless
resizing and copy/paste between guest and host.  I've found a few
guides online telling me to use SPICE, but nearly all of them assumed
you start the VM via ``virt-manager``, a nice frontend with an ugly
XML configuration format.  Personally I prefer writing a small shell
script, so here's mine, adapted from `the relevant Arch Wiki
article`_:

.. code:: shell

    #!/bin/bash
    socket='/tmp/vm_spice.socket'
    qemu-system-x86_64 \
        -m 2G -enable-kvm -drive file=disk.qcow2,format=qcow2 \
        -vga qxl -device virtio-serial-pci \
        -device virtserialport,chardev=spicechannel0,name=com.redhat.spice.0 \
        -chardev spicevmc,id=spicechannel0,name=vdagent \
        -spice "unix,addr=$socket,disable-ticketing" &

    sleep 5
    remote-viewer -f "spice+unix://$socket"

The first line of options is stuff specific to my setup.  The guest
won't work comfortably without at least 2G of RAM, KVM makes
virtualization run close to native speed and the drive points to a
qcow2 file.  The remaining options are for setting up SPICE with a QXL
video device and all the guest interop jazz, listening on a UNIX
socket.  After waiting for a while, ``remote-viewer`` is spawned
against the UNIX socket in full-screen mode.  You might want to
experiment here, for example you can configure it to use your favorite
mouse cursor release key combination.

Inside the guest you'll have to do a few more things:

.. code:: shell-session

    [wasa@box ~]# apt-get install spice-vdagent xserver-xorg-video-qxl
    [wasa@box ~]# systemctl enable spice-vdagentd
    [wasa@box ~]# systemctl start spice-vdagentd
    [wasa@box ~]$ spice-vdagent

The last line is about running the client which communicates clipboard
requests and alike to the daemon.  I discovered the hard way that the
``spice-vdagent`` package comes with autorun entries for popular
desktop environments, so instead I went for launching it inside ``i3``
as soon as my session starts.

To see changes to screen resolution you'll have to restart your
session and check ``/var/log/Xorg.0.log`` to correctly detect and use
the QXL driver.  If that's indeed the case, ``xrandr`` will show a
``Virtual-0`` device and ``xrandr --output Virtual-0 --auto`` changes
its resolution to the best fitting one.

Finally, there's one more underappreciated feature this setup gives
you, changing focus is seamless so you'll no longer need a dedicated
key combination for releasing or capturing the mouse cursor.  If
you're in windowed mode focus is back to the host once you hover over
anything outside the VM's screen, in fullscreen mode you can hover
over the top middle part of its screen, an OSD appears (with options
such as disabling fullscreen mode) and focus is back to the host
again.  Switching focus between host and guest no longer sends
spurious keys to the guest, for example pressing ``$mod+3`` to switch
to a VM on workspace 3 used to enter the number 3 into whatever
application had focus inside the VM.  This is no longer the case with
``remote-viewer`` and makes for a smooth user experience.

.. _the relevant Arch Wiki article: https://wiki.archlinux.org/index.php/QEMU#SPICE
