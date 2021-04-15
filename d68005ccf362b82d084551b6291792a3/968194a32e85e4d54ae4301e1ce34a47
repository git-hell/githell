/* con.c - constant folding support                     ncc, the new c compiler

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

#include "cc1.h"
#include "type.h"
#include "con.h"

/* the magic of two's complement means we can do all integer operations
   using 64-bit longs and get the right answers, as long as we truncate
   the result to the correct number of significant bits and zero- or
   sign-extend as appropriate. */

void con_normalize(type_bits ts, union con *con)
{
    ts = type_machine_bits(T_BASE(ts));

    switch (ts)
    {
    case T_CHAR:
    case T_UCHAR:   con->i = (unsigned char)   con->i; break;
    case T_SCHAR:   con->i = (signed char)     con->i; break;
    case T_SHORT:   con->i = (short)           con->i; break;
    case T_USHORT:  con->i = (unsigned short)  con->i; break;
    case T_INT:     con->i = (int)             con->i; break;
    case T_UINT:    con->i = (unsigned)        con->i; break;
    }
}

/* cast the src operand with the specified type
   into the dst operand with the specified type.
   src and dst can be aliases of each other. */

void con_cast(type_bits dst_ts, union con *dst, 
              type_bits src_ts, union con *src)
{
    if (T_SAME_CLASS(dst_ts, src_ts))
        *dst = *src;

    if (dst_ts & T_FLOATING) {
        if (src_ts & T_SIGNED)
            dst->f = src->i;
        if (src_ts & T_UNSIGNED)
            dst->f = src->u;
    }

    if (src_ts & T_FLOATING) {
        if (dst_ts & T_UNSIGNED)
            dst->u = src->f;
        if (dst_ts & T_SIGNED)
            dst->i = src->f;
    }

    con_normalize(dst_ts, dst);
}

/* perform a comparison of two constants of the given type, and
   return the condition codes that result. we don't distinguish
   between the signed/unsigned codes, instead setting all that
   might apply. the compiler never issues a comparison followed
   by flag checks that are mismatched, so this causes no problem. */

#define CON_CMP(MEMB)                                                   \
    do {                                                                \
        if (left->MEMB == right->MEMB) {                                \
            CCSET_SET(ccs, CC_AE);                                      \
            CCSET_SET(ccs, CC_GE);                                      \
            CCSET_SET(ccs, CC_BE);                                      \
            CCSET_SET(ccs, CC_LE);                                      \
            CCSET_SET(ccs, CC_Z);                                       \
        }                                                               \
                                                                        \
        if (left->MEMB > right->MEMB) {                                 \
            CCSET_SET(ccs, CC_A);                                       \
            CCSET_SET(ccs, CC_G);                                       \
            CCSET_SET(ccs, CC_AE);                                      \
            CCSET_SET(ccs, CC_GE);                                      \
            CCSET_SET(ccs, CC_NZ);                                      \
        }                                                               \
                                                                        \
        if (left->MEMB < right->MEMB) {                                 \
            CCSET_SET(ccs, CC_B);                                       \
            CCSET_SET(ccs, CC_L);                                       \
            CCSET_SET(ccs, CC_BE);                                      \
            CCSET_SET(ccs, CC_LE);                                      \
            CCSET_SET(ccs, CC_NZ);                                      \
        }                                                               \
    } while (0)

ccset con_cmp(type_bits ts, union con *left, union con *right)
{
    ccset ccs;

    CCSET_CLEAR(ccs);

    if (ts & T_FLOATING)
        CON_CMP(f);
    else if (ts & T_SIGNED)
        CON_CMP(i);
    else
        CON_CMP(u);

    return ccs;
}

/* vi: set ts=4 expandtab: */
