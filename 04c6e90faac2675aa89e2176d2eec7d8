April 14, 2021

## ncc, the _new_ C compiler

NCC is an ANSI/ISO-compliant optimizing C compiler that is intended to be
the system compiler for a forthcoming lightweight POSIX system for embedded
and desktop computing. Thus it is but a small piece of a much larger project.

The compiler is retargetable by design, but, at present, it produces binaries
for Linux/x86_64. This is a matter of development convenience and not, as
indicated above, its ultimate purpose. As the compiler ABI differs somewhat
from the System V ABI used by Linux, it does not produce binaries that can be
linked against Linux system libraries. It does, however, provide its own
(still incomplete) standard ANSI/Posix C library.

### status

The compiler is complete in the sense that it recognizes the entirety of the
language (as outlined below) and produces reasonable object code. The standard
library is substantial but still incomplete. The compiler is still in its
infancy and is rather crude. Even so, it is relatively stable, compiles itself
and its own libraries with no difficulty, and produces code that performs
within 10-20% of GCC -O2 (at least, according to my own not-exactly-scientific
tests).

There are undoubtedly many lurking bugs.

### history and acknowledgements

This is actually a complete rewrite of an early compiler (also called NCC)
which I wrote for a different purpose a couple of years ago; it was a K&R
compiler for building old BSD sources. This is a different animal, the same
in name only, though given their shared authorship, this NCC and its earlier
namesake undoubtedly show many similarities.

Shout out to Paul Sokolovsky [@pfalcon](https://github.com/pfalcon), who was
briefly a colleague of mine on another project, who dropped me a line a few
months ago to ask what ever happened to the previous incarnation of NCC.
He (inadvertently) got the ball rolling on this one.

### building/using

There's a simple makefile. See the embedded instructions there for building.

### supported C dialect

The compiler is compliant with ANSI C89/ISO C90, except that
* there is no support for wide characters/string literals and
* there is no locale support.

these reflect the compiler's purpose, which is for writing system software,
not applications (the ANSI committee really got these wrong, anyway).

The compiler also supports a few non-standard extensions:
* C99-style flexible array members
* C11-style anonymous structs and unions

Planned, but not yet in the compiler are:
* inline assembly (with a function similar to, but form different from,
GCC/LLVM)
* statement expressions (again, a la GCC/LLVM)

Note that there is no plan to support C99, except, perhaps, designated
initializers, compound literals, and _maybe_ the __restrict__ qualifier.
Much of the rest of C99 is either highly specialized (e.g., complex types),
unnecessary (e.g., __long long__, NCC __long__ is always 64 bits) or has
been implicitly acknowledged to be garbage (e.g., variable-length arrays,
which have effectively been withdrawn). Inline functions will be subsumed
by the preprocessor combined with statement expressions.

### the standard library

While the compiler and preprocessor are original works, the standard library
is mostly a curated collection of code with BSD-style licenses. the code is
drawn from the Amsterdam Compiler Kit/Minix libraries, 4BSD, and even
Coherent (much of which is really high-quality code, so I'm happy it can live
on in ncc).

### brief (simplified) notes about internals

The front end uses a syntax-directed approach: an unremarkable
recursive-descent parser builds a CFG from the source input,
expressions being the only constructs that are temporarily
represented as ASTs. The target-independent IR looks much like
a load-store architecture with condition codes (so, e.g, an ARM,
which is not a coincidence).

There are several target-independent optimization phases, including
the usual constant folding, algebraic reassociation/simplification,
constant and copy propagation, value numbering, loop-invariant code
motion, etc. An interpretive code generator then translates the IR
into machine-dependent IR, which is effectively assembly language
with mostly-undetermined register assignments. Target-dependent (and
repeat target-independent) optimizations are run, then the graph
allocator assigns registers and the output is rendered.

Notably missing are global common subexpression elimination, and anything
requiring SSA form. GCSE is planned, I just haven't gotten around to it
yet. SSA, on the other hand, may or may not happen. The infrastructure to
support it is there -- the IR already employs register subscripts, there
is a carve-out for phi functions, extending the existing dominator code
to find frontiers, etc. would be straightforward -- but I'm not yet
convinced that the overall improvements would justify the cost.

There is some primitive instruction scheduling, but only applied (quite
awkwardly, actually) to the target-independent IR.

### criticisms/improvements needed

The compiler is hog-tied, as all C compilers are, by aliasing problems.
(Would that history had turned out differently, and we all wrote code
in Ada.) The compiler takes a very pessimistic view of memory accesses.
It would be fairly unintrusive to take advantage of the Standard's strict-
aliasing provisions to improve this, but that tends to make programmers
angry. To go beyond that, doing serious aliasing analyses, and then using
the results, would be a lot more work.

The code is messier than I'd like. Some parts of the compiler are better
thought-out than others. There are phasing problems between optimization
passes. Many of the algorithms are implemented poorly and slowly. The list
goes on. I should/will open issues in the repository to enumerate.

Many "optimizations" in the compiler are ad hoc code improvements that really
deserve to be properly analyzed and applied more generally. This is especially
true of the target-dependent optimizations for the AMD64. (There are many,
many improvements to be made in the AMD64 code generator. It interacts poorly
with other phases of the compiler and still produces some really bone-headed
sequences in the output. This is partly because I designed the IR for RISC
architectures, but still ...)

Probably the worst (but by no means only) offender is the register allocator.
It's a fairly textbook bottom-up allocator, with absolutely no finesse when
it comes to spills: not only does it reserve registers for spilling (which
causes spills), it spills entire ranges rather than splitting ranges, and
to make matters worse, the spill code naively inserts loads-before-uses and
stores-after-defs for _every_ reference in those spilled ranges. My guess is
that a little effort on the allocator will go along way toward improving
the output.
