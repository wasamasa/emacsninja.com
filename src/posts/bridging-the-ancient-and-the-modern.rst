((title . "Bridging the Ancient and the Modern")
 (date . "2018-03-23 09:53:50 +0100")
 (emacs? . #t))

I tried out some new social networks lately.  Mastodon_ I quite like
(it's like what I've wanted Twitter to be), Discord_, not so sure.
So, if you've wondered about my reduced presence on IRC, that's why.

Writing an IRC bot is one of the classic programming exercises that
can be done in pretty much every programming language offering you
some way to open TCP sockets and manipulate strings.  I started doing
one in Emacs Lisp long time ago, although not from scratch (rather by
leveraging `an existing IRC client`_) and wondered whether there is
anything to learn from doing the equivalent with a "modern" IM
platform like Discord.  Could it be still be done from scratch? What
else is different about it?

First I had to find a meaningful thing for the bot to do.  I chose
Eliza_, the classic demonstration of a chatter bot that managed
fooling people into having prolonged conversations with them.  The
version I'm familiar with is ``M-x doctor`` which is part of Emacs.
So, first of all, I wrote some code to interface with that command in
a REPL-style fashion.  A companion shell script boots up Emacs in
batch mode for interfacing with the doctor from the shell.  Much like
the original mode, you terminate your input by pressing ``RET`` twice.
This is an intentional design decision to allow for multi-line input
as seen on Discord (but absent from IRC, where you could get away with
making it single-line input).

I briefly entertained the thought of writing the rest of the bot from
scratch in Emacs Lisp, but abandoned it after learning that I'd need
to use websockets with zlib compression to subscribe and respond to
incoming messages.  While there is `an existing library for websocket
support`_, I'd rather not figure out its nitty-gritty details, let
alone with the lack of zlib compression.  It doesn't help that
Discord's official API docs are inconclusive and fail answering
questions such as how you can set your current status (and more
importantly, why it fails getting updated).  So, an officially
recommended Discord library it is.

The choice on which one it's going to be depended on whether the
programming language it's implemented with allowed me to communicate
with my shell script.  I tried out `discord.js`_ first, battled a
fair bit with Node.js, but gave up eventually.  There doesn't seem to
be a way to spawn a child process and read from / write to its
stdout / stdin pipes as you see fit.  Instead you can only add a
callback for the process output and good luck if you want to figure
out what piece of output corresponds to the input you wrote earlier.
This is why I went for discordrb_ instead, wrote some glue code for
subprocess communication and started figuring out their API to react
to incoming messages.

There are a few lessons to be learned from their API:

- Allow adding as many event handlers as you want for specific events,
  with convenience options for narrowing down to commonly needed
  situations (like messages starting with a prefix)
- Inside these event handlers, provide an object containing all
  context you'd need, including the information who to respond to
- Keep the bot alive when an event handler errors out

Now, to test the bot I needed to unleash it on a server.  It turns out
that unlike on IRC bot accounts are handled specially.  You must:

- Register an application and obtain an ID for authorization purposes
- Enable a bot account and obtain an authorization token
- Generate an URL for inviting the bot to a server
- Share that URL with someone wielding the necessary permissions
- Hope for the best and wait for them to invite the bot

This meant that I had to create my own test server to check whether my
code worked at all.  For this reason I haven't been able to get it
running on the server I was invited on.  If you want to, you can try
it on your own server, `the sources`_ are on GitHub as always.

.. _Mastodon: https://joinmastodon.org/
.. _Discord: https://discordapp.com/
.. _an existing IRC client: https://github.com/jorgenschaefer/circe
.. _Eliza: https://en.wikipedia.org/wiki/ELIZA
.. _an existing library for websocket support: https://github.com/ahyatt/emacs-websocket
.. _discord.js: https://discord.js.org/
.. _discordrb: https://github.com/meew0/discordrb
.. _the sources: https://github.com/wasamasa/eliza-bot
