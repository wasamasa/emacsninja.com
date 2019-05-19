((title . "Trapping Attackers Without Nyan Cat")
 (date . "2019-05-19 23:00:40 +0200")
 (emacs? . #f))

As of a little over two weeks ago I've stopped offering the service
I've described in `an earlier blog post`_.  Someone™ managed maxing
out my server's resources by establishing as many connections as
possible and keeping them open for hours.  Eventually my hoster sent
me an email about unusually high resource usage.  After a reboot
everything was back to normal, but I didn't feel like allowing this to
happen again, so I shopped for a more efficient SSH honeypot.

endlessh_ fits the bill.  It does as little work as possible, yet
wastes the unsuspecting attacker's time in a rather sneaky way: By
sending an infinite SSH banner.  The downside is that you won't ever
find out what kind of software the attacker is using since it doesn't
proceed beyond that stage.  I was curious how well it works in
practice, so I collected logs for two weeks, wrote a bit less than 100
SLOC of Ruby to do basic statistics and plotted the overall
distribution with ``gnuplot``.

-----

.. code:: text

    Hosts: 21286
    Unique hosts: 1665
    Total time wasted: 17 days, 2 hours, 51 minutes and 40 seconds
    Max time wasted: 7 days, 9 hours, 19 minutes and 39 seconds
    Average time wasted: 20 minutes and 38 seconds
    Total bytes sent: 43.6M
    Max bytes sent: 1.1M
    Average bytes sent: 2.1K
    Max connections established: 58
    Greatest netsplit: 47

    Top 10 hosts:
      112.***.***.***: 12160 connections
       58.***.***.***: 2427 connections
      218.***.***.***: 1792 connections
       58.***.***.***: 358 connections
      195.***.***.***: 174 connections
      185.***.***.***: 109 connections
      222.***.***.***: 65 connections
      223.***.***.***: 58 connections
      115.***.***.***: 52 connections
      180.***.***.***: 51 connections

.. image:: /img/endlessh-thumb.png
   :target: /img/endlessh.png

.. _an earlier blog post: http://emacsninja.com/posts/trapping-attackers-with-nyan-cat.html
.. _endlessh: https://github.com/skeeto/endlessh
