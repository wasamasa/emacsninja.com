((title . "libui")
 (date . "2016-06-28 21:30:14 +0200"))

I've released `the third and last egg`_ in my series of wrappers for
GUI libraries!  Maybe there will be more, but none I'll work on
full-time.

This time I haven't had the joy of working with a well thought-through
API and ran into more issues than before.  I'll focus on the
memory-related ones as it's a topic I haven't found much about online.
To summarize, I've identified at least three different ways of
managing memory in my wrapper:

1. Foreign-managed pointers to heap-allocated memory:

   This is the easiest case.  Some foreign function is called and it
   returns a pointer towards the data you're interested in.
   Fortunately, you don't need to clean up afterwards as it firmly
   remains under its control.

   .. code:: scheme

       (define widget-table (make-hash-table))

       (define (dispatch-event! widget* type)
         (match-let* ((widget (hash-table-ref widget-table widget*))
                      (handlers (widget-handlers widget))
                      ((handler . args) (hash-table-ref handlers type)))
           (apply handler widget args)))

       (define-external (libui_WindowClosingHandler (uiWindow* window*) (c-pointer data)) bool
         (dispatch-event! window* 'closing))

2. Self-managed pointers to heap-allocated memory:

   Slightly bit harder.  A foreign function expects a pointer to a
   long-lived object that you maintain yourself.  First you allocate
   heap memory with good ol' ``malloc``, then expose ``free``
   explicitly, implicitly with a finalizer or in a way combining
   both:

   .. code:: scheme

       (define (new-area-handler draw-handler mouse-event-handler mouse-crossed-handlerdrag-broken-handler key-event-handler)
         (let* ((area-handler* (allocate uiAreaHandler-size))
                (_ ((foreign-lambda* void ((uiAreaHandler* handler))
                      "uiAreaHandler *h = handler;"
                      "h->Draw = libui_AreaDrawHandler;"
                      "h->MouseEvent = libui_AreaMouseEventHandler;"
                      "h->MouseCrossed = libui_AreaMouseCrossedHandler;"
                      "h->DragBroken = libui_AreaDragBrokenHandler;"
                      "h->KeyEvent = libui_AreaKeyEventHandler;")
                    area-handler*))
                (area-handler (make-area-handler area-handler* draw-handler mouse-event-handler mouse-crossed-handler drag-broken-handler key-event-handler)))
           (hash-table-set! area-table area-handler* area-handler)
           (set-finalizer! area-handler free)))

   The latter was a bit difficult as libui is too smart and attempts
   detecting memory leaks on its own when the ``uiUninit`` function is
   called.  If that precedes the finalizers, the application crashes.
   Not too hard to hack around though:

   .. code:: scheme

       (define (new-path #!optional alternate?)
         (let* ((flag (if alternate? uiDrawFillModeAlternate uiDrawFillModeWinding))
                (path* (uiDrawNewPath flag)))
           (set-finalizer! (make-path path*) path-free!)))

       (define (path-free! path)
         (and-let* ((path* (path-pointer path)))
           (uiDrawFreePath path*)
           (path-pointer-set! path #f)))

       (define (uninit!)
         ;; run all pending finalizers
         (gc #t)
         (uiUninit))

3. Self-managed pointers to stack-allocated memory:

   Now things get weird.  In C, the default mode of operation is using
   the stack for allocating data.  There isn't really an equivalent in
   the vast majority of dynamic languages where nearly everything is a
   heap-allocated object.  I've tackled this problem previously by
   using the ``free``/``malloc`` strategy, later by writing
   code that unpacked the stack-allocated struct into simple values,
   then reassembled them to a struct at call-time.  The former ignores
   the semantics of stack-allocation and the latter is limited as it
   falls apart when the data needs to live on for longer than a
   function call.  Clearly I needed a different solution!

   It turns out that you can use CHICKEN's garbage collector in your
   favor.  It is fairly fast when it comes to allocating more memory
   as it is using the C stack and ``alloca`` to store Scheme objects.
   Therefore, all we'd need is a way to use a Scheme object in foreign
   code as it will be subject to Scheme's scoping rules and can be
   reclaimed once it's no longer accessible.  For this you create a
   blob of the right size, then hand a locative pointing to it to the
   foreign function expecting a pointer and fill your data in:

   .. code:: scheme

       (define-record brush storage)

       (define (brush-pointer brush)
         (make-locative (brush-storage brush)))

       (define uiDrawBrush-size (foreign-type-size (struct "uiDrawBrush")))

       (define (new-solid-brush r g b a)
         (let* ((brush (make-brush (make-blob uiDrawBrush-size)))
                (brush* (brush-pointer brush)))
           ((foreign-lambda* void ((uiDrawBrush* br) (double r) (double g) (double b) (double a))
              "br->Type = uiDrawBrushTypeSolid, br->R = r, br->G = g, br->B = b, br->A = a;")
            brush* r g b a)
           brush))

   An alternative way is storing the locative inside the record and
   omitting the pointer procedure.  While this variant has the benefit
   of always having the same pointer, it will break after garbage
   collection has moved the blob around, effectively invalidating the
   pointer.

.. _the third and last egg: https://github.com/wasamasa/libui
