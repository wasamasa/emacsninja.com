((title . "Atom Feeds With CHICKEN")
 (date . "2015-09-17 22:47:41 +0200")
 (updated . "2015-09-18 00:32:29 +0200"))

I do rarely eat out, but when I do, I visit `Cologne's nicest burger
restaurant`_.  They regularly come up with a new "Burger Of The Week",
but I dislike visiting their website over and over again.  Later I did
find out that it's actually grabbing said weekly offer from their
`Facebook feed`_.  No wonder it errors out for me once in a while,
just to display an outdated image instead...  This made for a
perfect excuse to write some more CHICKEN, so here's a quick
walkthrough of the code.

First comes the scraping part.  http-client_, html-parser_,
uri-common_ and sxpath_ are used for fetching the sources, parsing
them and selecting the relevant data.  The only stumbling block
besides learning XPath_ was making sense of Facebook's HTML.  For some
reason I receive the pages inside a *comment*, therefore I need to
find the most recent post involving the burger in a comment:

.. code:: scheme

    (define (fetch-fragment)
      (let* ((base-url "https://www.facebook.com/DiefetteKuh")
             (document (with-input-from-request base-url #f html->sxml))
             (comment-string ((sxpath "string(//comment()[contains(.,'BURGER')])")
                              document))
             (comment (call-with-input-string comment-string html->sxml)))
        (car ((sxpath "//div[contains(@class,'userContentWrapper') and contains(string(),'BURGER')]")
              comment))))

Only then I can parse the comment again for its metadata:

.. code:: scheme

    (define (burger-metadata fragment)
      (let* ((timestamp ((sxpath "number(//abbr/@data-utime)") fragment))
             (description ((sxpath "string(//div[contains(@class,'userContent')]/p)")
                           fragment))
             (image ((sxpath "string(//div[contains(@class,'uiScaledImageContainer')]/img/@src)")
                     fragment))
             (base-url "https://www.facebook.com")
             (relative-permalink ((sxpath "string(//a/abbr/../@href)") fragment))
             (path (uri-path (uri-reference relative-permalink)))
             (permalink (uri->string (update-uri (uri-reference base-url)
                                                 path: path))))
        (list timestamp description image permalink)))

I did consider inventing my own format for storing the scraped data,
but went with SQLite_ instead for simplicity's sake.  matchable_ is
tremendously useful for destructuring. And sql-de-lite_ is a joy to
use.

.. code:: scheme

    (define db (open-database "db.sqlite3"))

    (define (init-database)
      (if (null? (schema db))
          (exec (sql db "CREATE TABLE burgers(id INTEGER PRIMARY KEY,
                                              date NUMBER,
                                              description TEXT,
                                              url TEXT,
                                              permalink TEXT)"))))

    (define (update-database data)
      (match-let* (((date description url permalink) data)
                   (latest (query fetch-value (sql db "SELECT date FROM burgers
                                                                   WHERE date = ?;")
                                  date)))
        (if (not latest)
            (exec (sql db "INSERT INTO burgers(date, description, url,permalink)
                                  VALUES(?, ?, ?, ?);")
                  date description url permalink))))

The functions presented are used in a straight-forward manner:

.. code:: scheme

    (define (main)
      (init-database)
      (update-database (burger-metadata (fetch-fragment))))

    (main)

This concludes the first part.  All that's left is actually serving
the data for a feed reader.  Besides reading out data from the SQLite
database it's necessary to turn it in an Atom feed with
sxml-serializer_, atom_ and spiffy_.  I use a few more eggs, including
rfc3339_, format_ and spiffy-uri-match_.  First of all, a few helpers:

.. code:: scheme

    (define db (open-database "db.sqlite3"))

    (define news-items 10)

    (define (fetch-latest)
      (query fetch-all
             (sql db "SELECT * FROM burgers ORDER BY date DESC LIMIT ?;")
             news-items))

    (define (updated-at)
      (query fetch-value
             (sql db "SELECT date FROM burgers ORDER BY date DESC LIMIT 1;")))

    (define (unix->datetime seconds)
      (rfc3339->string (seconds->rfc3339 seconds)))

    (define (unix->date seconds)
      (let ((record (seconds->rfc3339 seconds)))
        (format "~4d-~2,'0d-~2,'0d"
                (rfc3339-year record)
                (rfc3339-month record)
                (rfc3339-day record))))

    (define (feed-item url description)
      (serialize-sxml
       `(div
         (p (img (@ (src ,url))))
         (p ,description))))

The feed is merely a serialization of the SXML_ as generated by the
atom egg's API:

.. code:: scheme

    (define (feed)
      (serialize-sxml
       (make-atom-doc
        (make-feed
         title: (make-title "Fette Brause")
         id: "http://fette.brause.cc/"
         updated: (unix->datetime (updated-at))
         authors: (list (make-author name: "Vasilij Schneidermann"))
         links: (list (make-link uri: "http://facebook.com/DiefetteKuh"))
         entries: (map (match-lambda
                        ((id date description url permalink)
                         (make-entry
                          id: permalink
                          title: (make-title (unix->date date))
                          updated: (unix->datetime date)
                          links: (list (make-link uri: permalink))
                          content: (make-content (feed-item url description)
                                                 type: 'html))))
                       (fetch-latest))))))

All that's left now is actually serving the content:

.. code:: scheme

    (define (main)
      (vhost-map
       `((".*" .
          ,(uri-match/spiffy
            `(((/ "")
               (GET ,(lambda (c)
                       (send-response
                        body: (feed)
                        headers: '((content-type "application/xml")))))))))))
      (server-bind-address "127.0.0.1")
      (server-port 8001)
      (set-buffering-mode! (current-output-port) #:line)
      (access-log (current-output-port))
      (start-server))

    (main)

This makes for two moving parts, one being a binary that has to be run
periodically by a scheduler (like, cron_ or `systemd timers`_ or
whatever else floats your boat), the other one being a web server I
just reverse proxy with nginx_:

.. code:: nginx

    server {
        listen 80;
        listen [::]:80;
        server_name fette.brause.cc;

        location / {
            proxy_pass http://127.0.0.1:8001;
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

Yes, it's that simple.  Full sources are on Github_ as usual.

**Update**: I wasn't reading closely enough when it came to the ID of
each atom feed item.  It's important to ensure it is sufficiently
unique and never changes so that a feed reader can use it for
identifying each news item.  In this case the permalink is good
enough, but for blogs made with Hyde one needs to make sure a format
string with two placeholders for tag and date is used.

.. _Cologne's nicest burger restaurant: http://www.fettekuh.de/
.. _Facebook feed: https://www.facebook.com/DiefetteKuh
.. _http-client: http://wiki.call-cc.org/eggref/4/http-client
.. _html-parser: http://wiki.call-cc.org/eggref/4/html-parser
.. _uri-common: http://wiki.call-cc.org/eggref/4/uri-common
.. _sxpath: http://wiki.call-cc.org/eggref/4/sxpath
.. _XPath: http://www.w3.org/TR/xpath/
.. _SQLite: https://www.sqlite.org/
.. _matchable: http://wiki.call-cc.org/eggref/4/matchable
.. _sql-de-lite: http://wiki.call-cc.org/eggref/4/sql-de-lite
.. _sxml-serializer: http://wiki.call-cc.org/eggref/4/sxml-serializer
.. _atom: http://wiki.call-cc.org/eggref/4/atom
.. _spiffy: http://wiki.call-cc.org/eggref/4/spiffy
.. _rfc3339: http://wiki.call-cc.org/eggref/4/rfc3339
.. _format: http://wiki.call-cc.org/eggref/4/format
.. _spiffy-uri-match: http://wiki.call-cc.org/eggref/4/spiffy-uri-match
.. _SXML: https://en.wikipedia.org/wiki/SXML
.. _cron: https://en.wikipedia.org/wiki/Cron
.. _systemd timers: http://www.freedesktop.org/software/systemd/man/systemd.timer.html
.. _nginx: https://www.nginx.com/resources/admin-guide/reverse-proxy/
.. _braune.brause.cc: http://braune.brause.cc/
.. _Github: http://github.com/wasamasa/brause.cc
