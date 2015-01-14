# GS2 documentation

## What is GS2?

GS2 is a stack-based, concatenative programming language with functional influences, inspired by [GolfScript](http://www.golfscript.com/) and [J](http://jsoftware.com/). Its primary purpose is doing well at [code golf](https://en.wikipedia.org/wiki/Code_golf) contests; it achieves this by supplying many built-in commands and syntactical shortcuts that are each only one byte long.

Programming in GS2 is conceptually similar to programming in GolfScript or similar languages: the stack initially contains a string representing standard input, and its contents are printed to standard output when the program is finished.

## Values

GS2 values are integers, blocks, or lists. Blocks contain unevaluated code, representing "functions", like in GolfScript. There is no dedicated string type: strings are just lists of integers.

## Tokens

Throughout this document mnemonics are listed next to some token names; these
are compatible with the included gs2c utility.

Special tokens are:

Token | Meaning
- | -
$01 $xx | Push unsigned byte $xx to stack.
$02 $yy $xx | Push signed short $xxyy to stack.
$03 $zz $yy $xx $ww | Push signed short $wwxxyyzz to stack.
$04 _ss_ $zz | Push string(s) to stack. _ss_ is separated by $07; the action performed is decided by the string end byte $zz, which is one of:

*   $05: push strings to stack, sequentially
*   $06: push strings to stack in array
*   $9b: sprintf (pops format from _ss_, pops fitting number of items from stack, pushes formatted string)
*   $9c: regex match (pops regex from _ss_, pops string from stack, pushes 1 on match, else 0)
*   $9d: regex replace (pops replacement string from _ss_, pops regex from _ss_, pops string from stack and pushes re.sub result)
*   $9e: regex find (like $9c, but calls re.findall)
*   $9f: regex split (like $9c, but calls re.split)
The regexes used by these operations may be prefixed by special characters to set a special variable _c_: by default it is 0, prefixing the regex by $5D sets it to 1, prefixing it by $7D $xx sets it to xx. _c_ affects the operations as follows:

*   $9c: match whole string if _c_ &gt; 0
*   $9d: perform at most _c_ substitutions (unlimited if _c_ = 0)
*   $9e: find first matching substring only if _c_ &gt; 0 (else array of matching substrings)
*   $9f: perform at most _c_ splits (unlimited if _c_ = 0)
Furthermore, if the first character of a program is $04, it may be omitted; an unmatched string closing token will automatically be paired up.

$07 $cc | Push single-character string "$cc" to stack.
$08 | Opens a block.
$09 | Closes a block.
%111xxyyy (hex values $e0 through $fb) | Wraps last y+1 values into a block. x is the final token (what do we do with this block?): 0=nop, 1=map (0x34), 2=filter (0x35), 3=apply on top 2 elements (0x38)
$xx $fc (dm1, dump-map1) | Map single token inside lists. Short for $08 $90 $xx $09 $34.
$xx $fd (df1, dump-filter1) | Filter single token inside lists. Short for $08 $90 $xx $09 $35.
$fe (m:) | Open rest-of-program map block.
$ff (f:) | Open rest-of-program filter block.

The following single-byte tokens have special meaning at the start of a program:
Mode | Meaning
- | -
$30 (line-mode) | Line mode: [program] &rarr; [lines, map program, unlines]
$31 (word-mode) | Word mode: [program] &rarr; [words, map program, unwords]
$32 (line-mode-skip-first) | Like $30, but ignore first line.

All other tokens are simple operations that pop values from the stack and push results back.

## Constants

The following single-byte tokens push constants to the stack:
Byte | Constant
- | -
$0a (new-line) | [$0a]
$0b (empty-list) | []
$0c (empty-block) | {}
$0d (space) | [$20]
$10 - $1a | 0 - 10
$1b | 100
$1c | 1000
$1d | 16
$1e | 64
$1f | 256

## Functions

Opcode | Meaning
- | -
$0e (make-array) | Pop n, then pop n elements and push them back in an array.
$20 (negate, reverse, eval) | Negates numbers, reverses lists, evaluates blocks.
$21 (bnot, head) | Bitwise-negates numbers, extracts first element from lists.
$22 (not, tail) | Boolean negation for numbers, drops first element from lists.
$23 (abs, init) | Absolute value for numbers, drops last element from lists.
$24 (digits, last) | Push array of base 10 digits for numbers, extracts last element from lists.
$25 (random) | Push random.randint(0, x-1) for numbers x, chooses random element for lists.
$26 (dec, left-uncons) | Subtracts 1 from numbers, pushes tail and then head for lists.
$27 (inc, right-uncons) | Adds 1 to numbers, pushes init and then last for lists.
$28 (sign, min) | Pushes sign for numbers, minimum for lists.
$29 (thousand, max) | Multiplies numbers by 1000, maximum for lists.
$2a (double, lines) | Multiplies numbers by 2, splits over newlines for lists.
$2b (half, unlines) | Divides numbers by 2, joins with newlines for lists.
$2c (square, words) | Squares numbers, splits over space for lists.
$2d (sqrt, unwords) | Integer square root for numbers, joins with space for lists.
$2e (range, length) | Pushes [0..n-1] for numbers n, pushes length for lists.
$2f (range1, sort) | Pushes [1..n] for numbers n, sorts lists, sortBy for blocks.
$30 (add, cat, +) | Adds numbers, catenates lists/blocks.
$31 (sub, diff, -) | Subtracts numbers, set difference for lists.
$32 (mul, join, times, fold, \*) | Multiplies numbers, repeats list+number, joins two lists, folds block.
$33 (div, chunks, split, each, /) | Divides numbers, chunksOf for list+number, splits two lists, applies block over each element of list.
$34 (mod, step, clean-split, map, %) | Modulo numbers, each nth element for list+number, splits two lists removing empty lists, maps block over list.
$35 (and, get, when, filter, &amp;) | Bitwise ands numbers, indexes for list+number, evals number+block only when underlying number is nonzero, filters for list+block.
Et cetera, et cetera, et cetera. You know what, just read the mnemonics in gs2.py and mess around with them. If you need to know anything ask me via [mail](mailto:0nooodl@gmail.com) or IRC (I'm mauris in #anagol on freenode.)

## Example usage and gs2c

The included gs2c.py script is an "assembler" that compiles a more readable representation of gs2 opcodes and constants. It reads mnemonics for gs2 functions from the source code of gs2.py itself! Let's write a simple program, compile it using gs2c.py, and run it.

A very simple golf challenge is the following: given a positive integer n on standard input, print a triangle of asterisks of size n:

    *
    **
    ***
    ****
    *****

We read the number, then map a block over [1..n], turning each number into asterisks followed by a newline:

    read-num range1 m: "*" times new-line

Then gs2c can compile this, and gs2 can run the resulting file:

    $ python gs2c.py &lt; stars &gt; compiled &amp;&amp; echo 7 | python gs2.py compiled
    *
    **
    ***
    ****
    *****
    ******
    *******

Our solution is 7 bytes long: 56 2f fe 07 2a 32 0a. This is pretty good compared to GolfScript's 11.