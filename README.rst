================
 rpn-calculator
================

-----------------------------------------------
 One of the few calculators made in pure Troff
-----------------------------------------------

Introduction
============
This project is a simple calculator that evaluates expressions put in `Reverse
Polish Notation`_ (RPN) or Postfix notation as some call them. The program is
aptly named *rpn-calculator*. The entirety of the program is written in pure and
plain Troff with no macros or preprocessors used whatsoever.

Originally written by `Stephanie Björk`_, it is hoped that this project will
help one realize the capabilities of the Troff typesetting language, that it is
not merely a typesetting language but perhaps a fully-fledged, Turing-complete
programming language; and also inspire other maintainers and contributors
whoever they are.

.. _Reverse Polish Notation: https://en.wikipedia.org/wiki/Reverse_Polish_notation
.. _Stephanie Björk: https://katt64.github.io

Capabilities
============
As of now, the calculator is capable of handling integers of fairly large sizes.
The size of *large* largely depends on the capabilities of Troff's numerical
register (``.nr``).

It is also capable of handling up to 26 tokens in its token stack (Alice stack),
and subseqently 26 machine-sized values in its evalulation stack (Birch stack).
This limit is semi-artificial. If you can read Troff source files and understand
my intentions, you'll know my reasonings. I had been thinking about giving a
total of 676 (26*26) spaces in the token stack, but I feel that it would be too
*loud* and would cause *unintended interference* towards other numerical
registers.  Nonetheless, this program serves as a *practical demonstration*; so,
a token stack size of more than 26 is usually unnecessary nonetheless.

The calculator currently handles 5 basic binary operations:

- Addition (+)
- Subtraction (-)
- Multiplication (*)
- Division (/)
- Modulo (%)

It takes input expressions translated into Reverse Polish Notation, from a Troff
source file interspersed with requests that push each token into the token
stack, and a request to evaluate the expression in the token stack. It outputs
the result and certain errors (if any) into a Postscript file.

Modulo is the remainder of dividing two operands. So, if you share 23 cookies
among 7 friends, each of them will get 3 cookies equally with 2 remaining for
further fractioning. This can be expressed as ``23 / 7 = 3 cookies for
everyone`` and ``23 % 7 = 2 cookies for further processing``.

Usage
=====
To begin using the calculator, make sure you have the 3 necessary files
listed below, obtainable from this repository. The directory hierarchy **must**
be preserved.

- ``rpn.roff``
  
  This is the Troff source file that contains macros to handle input
  expressions, handle the token stack, evaluate said stack, and print the
  result (and errors) into a Postscript file.

- ``stacks/alice.roff``

  This is the Troff source file that defines a basic numerical stack of 26
  available slots along with the methods to handle (push/pop) them. It also
  comes with a method (``.APS``) to print the stack counter (``aC``) for
  diagnostic reasons. This stack and its methods are labelled beginning with an
  ``A`` or ``a``, which can be "backronymed" into Alice.

- ``stacks/birch.roff``

  This is the Troff source file that defines a basic numerical stack of 26
  available slots along with the methods to handle (push/pop) them. It also
  comes with a method (``.BPS``) to print the stack counter (``bC``) for
  diagnostic reasons. This stack and its methods are labelled beginning with an
  ``B`` or ``b``, which can be "backronymed" into Birch.

Input
-----
In the same directory as ``rpn.roff``, prepare a Troff source file as input to
the calculator. The ``test.roff`` file, though not necessary for the functioning
of the calculator, is a very good example of an input file and also contains
comments on what things are in the file. No skills in Troff are required.

Following this paragraph is another example of an input file, showing only the
absolutely necessary lines for a successful calculation. The expression to be
evalulated is ``(7 + 9) / (6 - 4)``, which can be converted into Reverse Polish
Notation as ``7 9 + 6 4 - /``. Comments begin with ``\"`` and are ignored by the
compiler. Commands are CaSE-SenSItIve and begin with a ``.`` (dot) on the first
column.

.. code:: nroff

  .so rpn.roff \" Source the RPN processor module.
  .RPNPUSH 7 \" Push operand 7 into the token stack.
  .RPNPUSH 9 \" Push operand 9 into the token stack.
  .RPNPUSH + \" Push operator + into the token stack.
  .RPNPUSH 6 \" Push operand 6 into the token stack.
  .RPNPUSH 4 \" Push operand 4 into the token stack.
  .RPNPUSH - \" Push operator - into the token stack.
  .RPNPUSH / \" Push operator / into the token stack.
  .RPNEVAL \" Evaluate the token stack and print the result.

We will be using this source file throughout the entirety of this README file.

Processing
----------
To process the source file and get our result, we must use a Troff compiler.
A popular Troff compiler is GNU Troff, which comes with most distributions
of GNU/Linux and Cygwin, so you might not have to explicitly install it. The
reason for its being installed by default is because most manual pages are
typeset using Nroff when output to the terminal.

Henceforth assuming the input file is saved to ``input.roff``, to get your
output in Postscript, run the following command::

  groff input.roff > output.ps

This will evaluate the expression in the source file and output the numerical
result straight to a Postscript file named ``output.ps``. This file must be
openned using a Postscript interpreter like Ghostscript. Technically, if your
output is Postscript, the typesetting is done by Troff (not Nroff).

If you want the output on TTY (terminal) instead, run the following command::

  groff -Tutf8 input.roff | head -n 2

This will evaluate the expression in the source file and output the numerical
result straight to the terminal in UTF-8 encoding. The last blank lines are
stripped using the intrinsic ``head`` utility. Technically, if your output is
the terminal, the typesetting is done by Nroff (not Troff).

Compilation warnings and errors spit out by ``groff`` can be very hard to
understand. If you typed the source file well, you will not likely find any
errors. If you have typed the source file well without any mistakes, but there
is still a warning or error, then the calculator output should **not** be
trusted and the fault is very likely mine (the programmer's).

Output
------
If you typed the source file correctly without any syntactical or spelling
mistakes, the output will definitely contain the numerical result of the
evaluation of the token stack.

However, the result could be wrong if there was a warning or an error during
compilation, an error during evaluation, an error before printing the result, or
if you have (in)advertently pushed the calculator to or beyond its limits (see
Capabilities_).  Also, even if there were no machine-noticeable errors, the
result is only as good as the input given by the human.  Run-time errors are
output directly to the resulting document before the evaluation result is
printed (I'm considering outputing errors to stderr instead, though).

If everything went well, the output should look something like this::

  8

This is the correct answer because ``(7 + 9) / (6 - 4) = 16 / 2 = 8``.

As of the time of this writing, the only error the calculator is programmed to
complain about is when there is/are some number(s) left behind unevaluated in
the evaluation stack that cannot be evaluated further. This is likely due to
human error in the Input_ stages. If that is so, the output should look
something like this::

  ERROR: Malformed input expression in token stack. 8

This is likely to happen if you accidentally push an extra value into the token
stack. As you can see, you still get the answer, but depending on how the error
occurred, it could be incorrect. So, fix the error first!

Motivation
==========
As a typical teenager going through chronic depression and anxiety, I seldom
have a lot of motivation. However, if there is something that can motivate me,
it is UNIX and its quirks. Troff is one of the quirks I love the most. It is not
just a language for typesetting (although it is extremely good at it), it also
has enough capabilities to perform other more complex computational tasks that
other more mainstream programming languages can handle as well.

Before this calculator, I have coaxed Troff into evaluating factorials and
printing Fibonacci sequences. It was fun, but got boring pretty quickly. This
calculator is by far the most interesting thing yet.

It is my decision to create this repository in order to demonstrate this
capability within Troff that actually transcends Troff's core values. To quote
Professor Brian W. Kernighan from the book, *Troff User's Manual* from November
1992:

  Joe Ossanna's *troff* remains a remarkable accomplishment. For fifteen years,
  it has proven a robust tool, taking unbelievable abuse from a variety of
  preprocessors and being forced into uses that were never conceived of in the
  original design, all with considerable grace under fire.

With that said, I am deeply indebted to Joe Ossanna for creating such a
magnificent masterpiece in typesetting (Troff) and Professor Brian W. Kernighan
for his well-written books on Troff.

Getting help
============
If you need help with this program or Troff, you can contact me personally. I'm
Stephanie Björk (Katt) <katt64@tuta.io>.

I am no expert in Troff typesetting. I've just started out this year, so please
bear in mind that I could very well be as clueless as you are. :p
