((title . "Hand-crafted Uberjars")
 (date . "2018-02-26 22:24:25 +0100")
 (emacs? . #f))

While making `a MIDI REPL`_ I ran into the problem of providing people
interested in trying it out something self-contained so that they
wouldn't have to recreate my dev setup.  The solution for this is
making a JAR file containing the `.class` files of your project and
all of its dependencies.  There are tools for this purpose such as
Ant_ and Maven_, but I couldn't figure out how to make
them work for me, so I decided to take a closer look at what happens
behind the scenes and created a simple Makefile_.

A JAR file is just a ZIP archive following `certain rules`_.  It must
contain a ``manifest.txt`` in its root and class files inside
directories mirroring the package structure.  The only difference
between a regular JAR and an Uberjar is that the latter will also
include the class files of its dependencies.  Tools for creating them
will have to extract the class files from all JAR files involved and
combine them into a directory tree before creating a new JAR file
containing all required class files.  Things can get ugly if your
dependencies share the same package prefix (such as ``org.foo`` and
``org.bar``) or if a dependency exists in multiple versions (such as
``org.foo`` depending on ``npm.leftpad-0.0.1`` and ``org.bar``
depending on ``npm.leftpad-0.0.2``) [1]_, I don't even attempt to deal
with these.

The manifest is a text file following a fixed format.  The only thing
you can get wrong here is the entry point which must be the name a
class containing a static main method.  It's required so that a
``java -jar my.jar`` knows where to look, however the entry point can
be changed by running ``java -cp my.jar <classname>`` instead.  This
is useful for debugging and allows you to add other dependency JARs to
the classpath you haven't put into your Uberjar yet.  Just change the
argument to ``-cp`` to be a double colon separated list of JARs.

The dependencies are unzipped into a temporary directory.  The ``jar``
tool supports changing the working directory so that you can switch to
that directory and add the extracted directories without any prefix.
That's everything necessary to create a runnable JAR!

.. _a MIDI REPL: https://github.com/wasamasa/waka
.. _Ant: https://ant.apache.org/
.. _Maven: https://maven.apache.org/
.. _Makefile: https://github.com/wasamasa/waka/blob/master/Makefile
.. _certain rules: https://docs.oracle.com/javase/tutorial/deployment/jar/index.html
.. _yet another product: http://www.jdotsoft.com/JarClassLoader.php

.. [1] The way to deal with them is using a custom class loader, as
       demonstrated by `yet another product`_ in the problem space.
