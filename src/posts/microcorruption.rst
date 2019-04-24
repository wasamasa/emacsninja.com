((title . "Microcorruption")
 (date . "2019-04-24 11:40:24 +0200")
 (emacs? . #f))

Hey folks.  After successfully completing the original Cryptopals_
exercises, I've decided to try the Microcorruption_ challenge Thomas
Ptacek helped designing.  They are about binary exploitation, the art
of turning a `buffer overflow`_ into arbitrary code execution.
There's plenty of write-ups about solving the challenges, so mine will
be about the low-level debugging involved.  While the notes below are
specific to the challenge, you may find them helpful for understanding
debuggers in general.  For more debugging insights, I can recommend
reading "The Art of Debugging" by Norman Matloff and Peter Jay
Salzman.

Challenge-specific points
-------------------------

Every program in the challenge follows the same pattern of munging
some data, asking for user input and conditionally unlocking a door if
all requirements have been met.  Your task is figuring out user input
that will unlock the door in a reproducible manner.  The big
revelation is that unlike in a reverse engineering challenge you don't
even have to provide a valid password, all that matters is that you
trick the system into unlocking the door.

The challenge tries to make it as simple as possible to figure out
what's going on.  The programs run on a MSP430_ microcontroller, a
RISC_ architecture so simple that the full list of mnemonics fits on
two manual pages.  Most of the reverse engineering work has been done
for you by giving you a disassembled view of the program code, with
function names for each section.  There is a separate page for
assembling and disassembling code.  The debugger is graphical and
features a dashboard of widgets to follow the program's control flow.
This makes it easy to spot patterns, as opposed to textual debuggers
where you need to keep all relevant information in your head.  It's a
bit like the difference between Nethack_ (top-down rogue-like) and
Zork_ (classic text adventure).

Using the debugger
------------------

The most fundamental tool you have in your toolbox is the breakpoint.
With it you put stop signs on your program and can skip the boring
parts easily.  Typically you put breakpoints on functions, but you can
put them on absolute addresses as well.  Note that if you ``reset``
the debugger (as opposed to rebooting it), breakpoints are preserved,
this makes it easy to try a different way of executing the program.
Another pattern that comes up is creating a breakpoint to skip ahead
to some point of the program, then undoing it later to inspect that
part of the program more closely.

Stepping through code is crucial to get right.  You can ``continue``
execution to proceed to the next breakpoint or user input action.  To
proceed one instruction, use ``step`` which is also known as ``step
in``.  This is because, when faced with a call to a function, ``step``
would step into it, with the next instruction being inside that
function.  To avoid this there's ``next`` which is also known as
``step over``, it behaves almost the same except that, if faced with a
call, it will not enter the function, but step over it so that you
stay at the same code snippet.  If you find yourself in a function
you'd want to get out quickly, use ``finish``.  This will execute
instructions until the function's end.

There are short forms of these commands, ``c`` (``continue``), ``s``
(``step``), ``n`` (``next``) and ``f`` (``finish``).  To repeat the
last command, use the enter key.  It's possible to repeat commands
with a numerical argument and to define macros (mostly useful for more
complex repetitive patterns), but I haven't made use of these yet.

Poking the memory
-----------------

To understand how memory can be corrupted in ways useful to an
attacker, it's important to know how a running program is represented
in memory.  For this, it's useful to track the memory and register
views as the program is running.  You'll find that in this challenge
there are separate regions of memory dedicated to the stack, heap and
program code.  If the memory layout permits it, interesting things
can happen when the attacker writes beyond the designated boundaries.
For example, if the stack pointer points to attacker-controlled
memory, anything reading memory from the stack will be influenced,
such as the return address to the last function (which could be
changed to jump to a different function).  Another example is
overwriting adjacent stack memory to change a local variable's
contents.  It's even possible to put machine code (also known as
shellcode because it's typically designed to spawn a shell) on the
stack, then jump into it and continue execution from there on,
provided that the stack is executable.

Endianness is a subtle point here.  The MSP430 architecture is
little-endian, so the least significant byte comes first.  The number
256 for example is encoded as the bytes ``0x00`` and ``0x01``.  This
is important to know when trying to make sense of numbers in the
disassembly and interpreting user input (like, when entering an
address).

.. _Cryptopals: http://emacsninja.com/posts/cryptopals.html
.. _Microcorruption: http://microcorruption.com/
.. _buffer overflow: https://en.wikipedia.org/wiki/Buffer_overflow
.. _MSP430: https://en.wikipedia.org/wiki/TI_MSP430
.. _RISC: https://en.wikipedia.org/wiki/Reduced_instruction_set_computer
.. _Nethack: https://en.wikipedia.org/wiki/NetHack
.. _Zork: https://en.wikipedia.org/wiki/Zork
