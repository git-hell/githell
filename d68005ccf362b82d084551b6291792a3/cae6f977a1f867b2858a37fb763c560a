/* codes.h - condition codes                            ncc, the new c compiler

Copyright (c) 2021 Charles E. Youse (charles@gnuless.org). All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE. */

#ifndef CODES_H
#define CODES_H

/* we assume an architecture with condition codes, which maps nicely onto
   AMD64, ARM, SPARC, i386, 68K, etc. (not so nicely onto MIPS, RISC-V...).
   condition_code is a bit of a misnomer: these are not condition codes,
   but rather the states that can be deduced from them.

   we map unsigned and floating-point comparisons onto the same set of codes.
   this reflects the behavior of both x86 SSE and ARM VFP floating-point; if
   a target comes along that makes this awkward, we can separate them. */

typedef int condition_code;     /* CC_* */

#define CC_Z            0           /* zero/equal */
#define CC_NZ           1           /* not zero/equal */
#define CC_G            2           /* (signed) > */
#define CC_LE           3           /* (signed) <= */
#define CC_GE           4           /* (signed) >= */
#define CC_L            5           /* (signed) < */
#define CC_A            6           /* (unsigned/float) > */
#define CC_BE           7           /* (unsigned/float) <= */
#define CC_AE           8           /* (unsigned/float) >= */
#define CC_B            9           /* (unsigned/float) < */

#define CC_ALWAYS       10
#define CC_NEVER        11

    /* separate the condition_codes that are actually dependent
       on the state of the CPU from the ones that aren't */

#define CC_CONDITIONAL(cc)  ((cc) < CC_ALWAYS)

    /* "real" condition codes are those above, which actually indicate some
       condition, as opposed to the pseudo-CCs below used for switch blocks */

#define CC_NR_REAL      (CC_NEVER + 1)
#define CC_REAL(cc)     ((cc) < CC_NR_REAL)

    /* invert the truth value of a (real) condition_code. notice that the
       values of the CC_* constants are arranged such that opposite meanings
       differ in their lsbs only; this makes CC_INVERT() trivial. */

#define CC_INVERT(cc)       ((cc) ^ 1)

    /* used for controlling block of switch statements,
       see struct cessor in block.h */

#define CC_SWITCH       12
#define CC_DEFAULT      13

    /* sometimes we need to index by condition_codes, e.g., to map them
       onto their equivalent I_SETcc instructions. for the moment the CC_*
       values are agreeable, but they might not always be, so use this. */

#define CC_INDEX(cc)    (cc)

    /* sometimes it's useful to group all condition_codes as a set */

typedef int ccset;

#define CCSET_CLEAR(s)              ((s) = 0)
#define CCSET_SET(s, cc)            ((s) |= 1 << CC_INDEX(cc))
#define CCSET_IS_SET(s, cc)         ((s) & (1 << CC_INDEX(cc)))
#define CCSET_EMPTY(s)              ((s) == 0)
#define CCSET_SAME(s1, s2)          ((s1) == (s2))
#define CCSET_INTERSECT(s1, s2)     ((s1) & (s2))
#define CCSET_UNION(s1, s2)         ((s1) | (s2))
#define CCSET_ALL                   ( 0xFFFFFFFF )

#endif /* CODES_H */

/* vi: set ts=4 expandtab: */
