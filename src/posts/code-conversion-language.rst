((title . "Code Conversion Language")
 (date . "2019-07-20 22:02:52 +0200")
 (updated . "2019-07-20 23:31:15 +0200")
 (emacs? . #t))

**Update**: I forgot that I did `a brief analysis
<https://gist.github.com/wasamasa/e5f0489676e7ac769e91>`_ on this many
years ago, using ROT13 as example.

Emacs is most famously, a re-imagination of a Lisp machine, with the
Emacs Lisp byte-code interpreter being at its core.  A lesser-known
fact is that there's two more byte-code interpreters in its C sources,
one for compiled regular expressions and another designed for encoding
and decoding text, known as Code Conversion Language (CCL).  This blog
post will focus on the latter as it's largely gone unnoticed and
hasn't seen too much experimentation.

The CCL implementation is split into the byte-code interpreter
(``ccl.c``) and compiler (``ccl.el``) parts.  There is no official
documentation other than comments and docstrings found in these files.
From this I've learned that CCL programs are represented as integer
vectors and that there's a higher-level language compiling to them,
described in the ``ccl-define-program`` docstring.  By reading that
information I've deduced the following:

- The VM has eight integer-sized registers ``r0`` to ``r7`` and an
  instruction counter ``ic``
- Register ``r7`` is used as a status register and may be clobbered at
  any time by an arithmetic operation
- A CCL program can either be run on a string and return a string,
  alternatively it can be run standalone for side effects
- The former mode requires you to provide a nine-element status vector
  representing the registers and instruction counter, the latter an
  eight-element status vector representing the registers only
- As a side-effect, the status vector contains the new state of the
  registers and instruction counter after executing the program
- The VM supports the standard C arithmetic, comparison and assignment
  operators
- The language translates several control flow statements to
  equivalent ``goto`` statements, such as ``if``, ``branch`` (look-up
  table) and ``loop`` with ``repeat`` inside
- Statements are grouped by surrounding them with parentheses
- When operating on a string, they are read in and written out in a
  serial fashion, no random access whatsoever
- It's possible to do a look-up on an array, translation table or hash
  table
- There is a ``call`` operator, but no stack to save/restore arguments
  to/from, so you'll have to come up with a calling convention fitting
  the available registers
- Each CCL program specifies a magnification factor which determines
  the ratio between output and input string size

Armed with that knowledge I wrote some boiler plate code for
experimentation:

.. code:: elisp

    ;; -*- lexical-binding: t; -*-

    (require 'ccl)

    (defvar ccl-status (make-vector 8 0))
    (defvar ccl-status+ic (make-vector 9 0))

    (defun ccl-init-status (status args)
      (let ((i 0))
        (fillarray status 0)
        (dolist (arg args)
          (aset status i arg)
          (setq i (1+ i)))))

    (defun ccl-run (program string &rest args)
      (let ((status ccl-status+ic))
        (ccl-init-status status args)
        (ccl-execute-on-string program status string)))

    (defun ccl-run-pure (program &rest args)
      (let ((status ccl-status))
        (ccl-init-status status args)
        (ccl-execute program status)
        status))

There will be some benchmark numbers, none of these are to be taken
seriously.  Do your own benchmarks before using mine for decisions.

Hello World!
------------

For starters I'll focus on processing strings.  The easiest possible
program that still does something useful reads in output and writes
it out as is:

.. code:: elisp

    (define-ccl-program ccl-identity
      '(1
        (loop
         (read r0)
         (write r0)
         (repeat))))

    (ccl-run 'ccl-identity "Hello World!") ;=> "Hello World!"

Let's go through that program carefully.  The S-Expression starts with
a magnification factor of 1, meaning that the output buffer should be
as large as the input buffer.  If it were zero, no I/O would be
permitted in the first place, whereas a factor greater than one would
allocate enough space to produce a string larger than the input.

The magnification factor is followed by a s-expression that's executed
until it's done or an error occurred, such as there being no more
input.  It may be followed by another s-expression that's executed
after the main one, no matter whether it failed with an error or not.

``ccl-identity`` uses a pattern that will come up a few more times in
this blog post.  It enters a loop, reads a character into the ``r0``
register, writes out a character from the ``r0`` register and jumps to
the beginning of the loop.  If there are no more characters left, the
read operation fails and terminates the loop.  Let's spice things up
by adding an extra processing step before writing out the character:

.. code:: elisp

    (define-ccl-program ccl-xor
      '(1
        (loop
         (read r1)
         (r1 ^= r0)
         (write r1)
         (repeat))))

    (ccl-run 'ccl-xor "Secret" 42) ;=> "yOIXO^"
    (ccl-run 'ccl-xor "yOIXO^" 42) ;=> "Secret"

XOR is the bread and butter operator in modern cryptography.  A text
can be encrypted by replacing each character with the result of XORing
it against a secret byte, similarly it can be decrypted by applying
the same transformation again.  To pass the secret byte as an
argument, I've placed it in the ``r0`` register and read the string
into the ``r1`` register instead.  On each iteration of the loop
``r1`` is set to ``r1 ^ r0`` and written out again.

More on translation
-------------------

In the real world translating characters isn't as simple as applying
some arithmetic to them.  Suppose I wanted to challenge the
``upcase`` built-in:

.. code:: elisp

    (define-ccl-program ccl-upcase
      '(1
        (loop
         (read r0)
         (if (r0 >= ?a)
             (if (r0 <= ?z)
                 (r0 -= 32)))
         (write r0)
         (repeat))))

The processing step is a bit more involved this time.  If the read-in
character appears to be between the ``a`` and ``z`` characters,
transform it by subtracting 32.  Why 32?  Take a look at an ASCII
table and you'll see that this is the distance between uppercase and
lowercase letters.  Unfortunately this implementation cannot challenge
``upcase`` as it fails to translate non-ASCII characters correctly and
is slower than the real deal:

.. code:: elisp

    (ccl-run 'ccl-upcase "Hello World!") ;=> "HELLO WORLD!"
    (ccl-run 'ccl-upcase "Mötley Crüe") ;=> "MöTLEY CRüE"
    (benchmark 100000 '(ccl-run 'ccl-upcase "Hello World!"))
    ;; => "Elapsed time: 0.165250s (0.072059s in 1 GCs)"
    (benchmark 100000 '(upcase "Hello World!"))
    ;; => "Elapsed time: 0.119050s (0.072329s in 1 GCs)"

Let's try again with a different text transformation where I actually
have a chance to win, ROT13_:

.. code:: elisp

    (define-ccl-program ccl-rot13
      '(1
        (loop
         (read r0)
         (if (r0 >= ?a)
             (if (r0 <= ?z)
                 ((r0 -= ?a)
                  (r0 += 13)
                  (r0 %= 26)
                  (r0 += ?a))))
         (if (r0 >= ?A)
             (if (r0 <= ?Z)
                 ((r0 -= ?A)
                  (r0 += 13)
                  (r0 %= 26)
                  (r0 += ?A))))
         (write r0)
         (repeat))))

This time the program needs to recognize two different character
ranges to process, lowercase and uppercase ASCII characters.  In
either case they're translated to their position in the alphabet,
rotated by 13, then translated back to ASCII again.  Surprisingly
enough, this is enough to beat both ``rot13-string`` and
``rot13-region``:

.. code:: elisp

    (ccl-run 'ccl-rot13 "Hello World!") ;=> "Uryyb Jbeyq!"
    (ccl-run 'ccl-rot13 (ccl-run 'ccl-rot13 "Hello World!"))
    ;; => "Hello World!"
    (benchmark 100000 '(ccl-run 'ccl-rot13 "Hello World!"))
    ;; => "Elapsed time: 0.248791s (0.072622s in 1 GCs)"
    (benchmark 100000 '(rot13-string "Hello World!"))
    ;; => "Elapsed time: 6.108861s (2.360862s in 32 GCs)"
    (with-temp-buffer
      (insert "Hello World!")
      (benchmark 100000 '(rot13-region (point-min) (point-max))))
    ;; => "Elapsed time: 1.489205s (1.017631s in 14 GCs)"

I then tried to use translation tables for a final example of a
"Vaporwave" converter, but failed.  Funnily enough this mirrors my
overall experience with Emacs, it's easy to write fun things, but the
moment one tries to write something useful, you discover it's not fun
and sometimes not even up to the task.  At least it's possible to
salvage the translation tables and use them with ``translate-region``
instead, the built-in used by ``rot13-string`` and ``rot13-region``:

.. code:: elisp

    (defvar ccl-vaporwave-table
      (make-translation-table-from-alist
       (cons '(?\s . 12288)
             (mapcar (lambda (i) (cons i (+ i 65248)))
                     (number-sequence 33 126)))))

    (defun vaporwave-it (string)
      (with-temp-buffer
        (insert string)
        (translate-region (point-min) (point-max) ccl-vaporwave-table)
        (buffer-string)))

    (vaporwave-it (upcase "aesthetic")) ;=> "ＡＥＳＴＨＥＴＩＣ"

Edging towards general-purpose computing
----------------------------------------

All examples so far have worked on text.  If you limit yourself to
numbers, you can solve some basic arithmetic problems.  Here's a
classic, calculating the factorial of a number:

.. code:: elisp

    (define-ccl-program ccl-factorial
      '(0
        ((r1 = 1)
         (loop
          (if r0
              ((r1 *= r0)
               (r0 -= 1)
               (repeat)))))))

    (defun factorial (n)
      (let ((acc 1))
        (while (not (zerop n))
          (setq acc (* acc n))
          (setq n (1- n)))
        acc))

While the regular version is more concise, the logic is nearly the
same in both.  Here's some numbers:

.. code:: elisp

    (aref (ccl-run-pure 'ccl-factorial 10) 1) ;=> 3628800
    (factorial 10) ;=> 3628800
    (benchmark 100000 '(ccl-run-pure 'ccl-factorial 10))
    ;; => "Elapsed time: 0.069063s"
    (benchmark 100000 '(factorial 10))
    ;; => "Elapsed time: 0.080212s"

This isn't nearly as much of a speed-up as I've hoped for.  Perhaps
CCL pays off more when doing arithmetic than for looping?  Another
explanation is that the Emacs Lisp byte-code compiler has an edge over
CCL's rather simple one.  Here's a more entertaining example, printing
out the lyrics of 99 Bottles of Beer on the Wall:

.. code:: elisp

    (define-ccl-program ccl-print-bottle-count
      '(1
        (if (r0 < 10)
            (write (r0 + ?0))
          ((write ((r0 / 10) + ?0))
           (write ((r0 % 10) + ?0))))))

    (define-ccl-program ccl-99-bottles
      '(1
        (loop
         (if (r0 > 2)
             ((call ccl-print-bottle-count)
              (write " bottles of beer on the wall, ")
              (call ccl-print-bottle-count)
              (write " bottles of beer.\n")
              (write "Take one down and pass it around, ")
              (r0 -= 1)
              (call ccl-print-bottle-count)
              (write " bottles of beer on the wall.\n\n")
              (repeat))
           ((write "2 bottles of beer on the wall, 2 bottles of beer.\n")
            (write "Take one down and pass it around, 1 bottle of beer on the wall.\n\n")
            (write "1 bottle of beer on the wall, 1 bottle of beer.\n")
            (write "Take one down and pass it around, no more bottles of beer on the wall.\n\n")
            (write "No more bottles of beer on the wall, no more bottles of beer.\n")
            (write "Go to the store and buy some more, 99 bottles of beer on the wall.\n"))))))

    (defun 99-bottles ()
      (with-output-to-string
        (let ((i 99))
          (while (> i 2)
            (princ (format "%d bottles of beer on the wall, %d bottles of beer.\n" i i))
            (princ (format "Take one down and pass it around, %d bottles of beer on the wall.\n\n" (1- i)))
            (setq i (- i 1))))
        (princ "2 bottles of beer on the wall, 2 bottles of beer.\n")
        (princ "Take one down and pass it around, 1 bottle of beer on the wall.\n\n")
        (princ "1 bottle of beer on the wall, 1 bottle of beer.\n")
        (princ "Take one down and pass it around, no more bottles of beer.\n\n")
        (princ "No more bottles of beer on the wall, no more bottles of beer.\n")
        (princ "Go to the store and buy some more, 99 bottles of beer on the wall.\n")))

This example shows a few more interesting things, generating text of
unknown length is rather hard, so I'm using the standard magnification
factor of 1 and estimate how big the buffer will be to create an
appropriately sized input string.  ``call`` is useful to not repeat
yourself, at the cost of having to carefully plan register usage.
Printing out the bottle count can be done if you're limiting yourself
to whole numbers up to 100, a generic solution is going to be hard
without random access to the output string.  The performance numbers
for this one are somewhat surprising:

.. code:: elisp

    (let ((input (make-string 15000 ?\s)))
      (benchmark 1000 '(ccl-run 'ccl-99-bottles input 99)))
    ;; => "Elapsed time: 0.301170s (0.217804s in 3 GCs)"
    (benchmark 1000 '(my-99-bottles))
    ;; => "Elapsed time: 1.735386s (0.507231s in 7 GCs)"

This doesn't make much sense.  Is using ``format`` that expensive?
It's hard to tell in advance whether CCL will make a noticable
difference or not.

But is it Turing-complete?
--------------------------

My experimentation so far left me wondering, is this language
turing-complete?  You can perform arithmetics, there's ``goto``, but
the I/O facilities, amount of registers and memory access are
limited.  The easiest way of proving this property is by implementing
another known turing-complete system on top of your current one.  I
researched a bit and found the following candidates:

- Brainfuck_: A classic, however it requires writable
  memory. Registers could be used for this, but you don't have many to
  play with.  You'd need the ``branch`` instruction to simulate the
  data pointer.
- subleq_: Implementing ``subleq`` looks easy, but suffers from the
  same problem as Brainfuck, it requires you to modify an arbitrary
  memory location.  I've found a compiler from a C subset to
  ``subleq`` that generates code operating beyond the handful of
  registers, so that's not an option either.
- `Rule 110`_: It's basically Game of Life, but one-dimensional and
  can be implemented in a serial fashion.  With some tricks it doesn't
  require random access either.  The proof of it being turing-complete
  looks painful, but whatever, I don't care.  It's perfect.  There are
  more elementary cellular automata, so I'll try to implement it in a
  generic fashion and demonstrate it on `Rule 90`_ which produces the
  `Sierpinski triangle`_.

.. code:: elisp

    (defmacro define-ccl-automaton (n)
      (let ((print-sym (intern (format "ccl-rule%d-print" n)))
            (rule-sym (intern (format "ccl-rule%d" n))))
        `(progn
           (define-ccl-program ,print-sym
             '(1
               ((r4 = 0)
                (if (r0 == ?1)
                    (r4 += 4))
                (if (r1 == ?1)
                    (r4 += 2))
                (if (r2 == ?1)
                    (r4 += 1))
                (branch r4
                        (write ,(if (zerop (logand n 1)) ?0 ?1))
                        (write ,(if (zerop (logand n 2)) ?0 ?1))
                        (write ,(if (zerop (logand n 4)) ?0 ?1))
                        (write ,(if (zerop (logand n 8)) ?0 ?1))
                        (write ,(if (zerop (logand n 16)) ?0 ?1))
                        (write ,(if (zerop (logand n 32)) ?0 ?1))
                        (write ,(if (zerop (logand n 64)) ?0 ?1))
                        (write ,(if (zerop (logand n 128)) ?0 ?1))))))
           (define-ccl-program ,rule-sym
             '(1
               ((r6 = ,n)
                (r0 = 0)
                (read r1)
                (read r2)
                (loop
                 (call ,print-sym)
                 (read r3)
                 (r0 = r1)
                 (r1 = r2)
                 (r2 = r3)
                 (repeat)))
               ((r0 = r1)
                (r1 = r2)
                (r2 = r5)
                (call ,print-sym)))))))

    (define-ccl-automaton 30)
    (define-ccl-automaton 90)
    (define-ccl-automaton 110)

    (defun ccl-sierpinski ()
      (with-output-to-string
        (let ((line "0000000001000000000"))
          (dotimes (_ 20)
            (princ line)
            (terpri)
            (setq line (ccl-run 'ccl-rule90 line))))))

The macro may look scary, but all it does is defining two CCL
programs.  What an elementary cellular automaton does is looking at
the two cells around the current cell, map them to a cell depending to
a rule and emit it.  There are two edge cases with this for the first
and last cell, in my implementation the first cell assumes the
previous one was a zero and the last cell uses the first cell.  Since
there's no random access, it's stored into a spare register at the
beginning and accessed in a S-Expression after the main loop
terminated due to no more input.  The surrounding and current cell are
stored in three registers and rotated every time a new cell is read
in.  The mapping is done in the print program by summing up the ones
and zeroes, then using the ``branch`` instruction to apply the rule to
it.  If you find this hard to follow, here's an Emacs Lisp version of
it using random access and less limited arithmetic to do the job:

.. code:: elisp

    (defun rule--evolve (prev cur next n)
      (let ((acc (+ (if (= prev ?1) 4 0)
                    (if (= cur ?1) 2 0)
                    (if (= next ?1) 1 0))))
        (if (zerop (logand n (ash 1 acc))) ?0 ?1)))

    (defun rule-evolve (line n)
      (let ((out (make-string (length line) ?0)))
        (dotimes (i (length line))
          (cond
           ((zerop i)
            (aset out i (rule--evolve ?0 (aref line i) (aref line (1+ i)) n)))
           ((= i (1- (length line)))
            (aset out i (rule--evolve (aref line (1- i)) (aref line i) (aref line 0) n)))
           (t
            (aset out i (rule--evolve (aref line (1- i)) (aref line i) (aref line (1+ i)) n)))))
        out))

    (defun sierpinski ()
      (with-output-to-string
        (let ((line "0000000001000000000"))
          (dotimes (_ 20)
            (princ line)
            (terpri)
            (setq line (rule-evolve line 90))))))

One more benchmark run, this time with less surprising performance
numbers:

.. code:: elisp

    (benchmark 1000 '(ccl-sierpinski))
    ;; => "Elapsed time: 0.365031s (0.071827s in 1 GCs)"
    (benchmark 1000 '(sierpinski))
    ;; => "Elapsed time: 0.545512s (0.071829s in 1 GCs)"

If you want to see it in action, try evaluating ``(progn (princ
(sierpinski)) nil)`` in ``M-x ielm``.

Final words
-----------

You may ask yourself now whether you should rewrite all of your slow
code to CCL programs.  I don't believe that's the way to go for a
number of reasons:

- Speedups vary wildly, somewhere between -40% to 450%.  There's no
  obvious way of predicting whether the optimization is worth it.
- It's hard to write an equivalent program or sometimes even
  impossible, especially if it requires you to use many variables or
  random access read/write operations.
- It's hard to debug a CCL program.  While the compiler does a good
  job at catching syntax errors, runtime errors are far harder to
  figure out if you can only stare at the registers.  Maybe a debugger
  could be built that uses the "continue" facility and instruction
  counter register...
- It's hard to maintain a CCL program.  Not to mention, there's hardly
  people who know how to write them.  Most of the examples I've found
  online do text encoding/decoding.  The only exception to this is
  ``pgg-parse-crc24-string`` which lives in a file marked as obsolete.
- This leads me to my last point, CCL may be obsoleted as well.
  Granted, it will take time, but so far I haven't seen people
  enthusiastic about it being a thing.

If you still believe that despite of this it's worth giving CCL a try,
please let me know, especially if you're doing something non-standard
with it, like advanced cryptography or number crunching.  Likewise, if
you're not convinced that it's turing-complete or that I could be
doing some things far better than presented, send me a message.

.. _ROT13: https://en.wikipedia.org/wiki/ROT13
.. _Brainfuck: https://en.wikipedia.org/wiki/Brainfuck
.. _Subleq: https://en.wikipedia.org/wiki/One_instruction_set_computer#Subtract_and_branch_if_less_than_or_equal_to_zero
.. _Rule 110: https://en.wikipedia.org/wiki/Rule_110
.. _Rule 90: https://en.wikipedia.org/wiki/Rule_90
.. _Sierpinski triangle: https://en.wikipedia.org/wiki/Sierpi%C5%84ski_triangle
