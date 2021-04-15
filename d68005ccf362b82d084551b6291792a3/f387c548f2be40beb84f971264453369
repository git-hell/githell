/* con.h - constant folding support                     ncc, the new c compiler

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

#ifndef CON_H
#define CON_H

#include "type.h"
#include "block.h"

extern void con_normalize(type_bits, union con *);
extern void con_cast(type_bits, union con *, type_bits, union con *);
extern ccset con_cmp(type_bits, union con *, union con *);

#define CON_FOLD_UNARY_I(TS, DST, SRC, OP)                                  \
    do {                                                                    \
        (DST)->i = OP (SRC)->i;                                             \
        con_normalize((TS), (DST));                                         \
    } while (0)

#define CON_FOLD_UNARY(TS, DST, SRC, OP)                                    \
    do {                                                                    \
        if ((TS) & T_FLOATING)                                              \
            (DST)->f = OP (SRC)->f;                                         \
        else                                                                \
            (DST)->i = OP (SRC)->i;                                         \
                                                                            \
        con_normalize((TS), (DST));                                         \
    } while (0)

#define CON_FOLD_BINARY_I(TS, DST, LHS, RHS, OP)                            \
    do {                                                                    \
        if ((TS) & T_SIGNED)                                                \
            (DST)->i = (LHS)->i OP (RHS)->i;                                \
        else                                                                \
            (DST)->u = (LHS)->u OP (RHS)->u;                                \
                                                                            \
        con_normalize((TS), (DST));                                         \
    } while (0)

#define CON_FOLD_BINARY_ANY(TS, DST, LHS, RHS, OP)                          \
    do {                                                                    \
        if ((TS) & T_FLOATING)                                              \
            (DST)->f = (LHS)->f OP (RHS)->f;                                \
        else if ((TS) & T_SIGNED)                                           \
            (DST)->i = (LHS)->i OP (RHS)->i;                                \
        else                                                                \
            (DST)->u = (LHS)->u OP (RHS)->u;                                \
                                                                            \
        con_normalize((TS), (DST));                                         \
    } while (0)

#endif /* CON_H */

/* vi: set ts=4 expandtab: */
