/* regs.c - register sets                               ncc, the new c compiler

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

#include <stdio.h>
#include "cc1.h"
#include "output.h"
#include "regs.h"

SET_DEFINE_FREELIST(reg)
SET_DEFINE_ALLOC(reg, 1000)
SET_DEFINE_LOOKUP(reg, pseudo_reg, reg)
SET_DEFINE_REMOVE(reg, pseudo_reg, reg)
SET_DEFINE_OVERLAP(reg, reg)
SET_DEFINE_DIFF(reg, reg)
SET_DEFINE_UNION(reg, reg)
SET_DEFINE_MOVE(reg)
SET_DEFINE_EQUAL(reg, reg)
SET_DEFINE_CLEAR(reg)

/* eliminate all members that have
   the same base as base or bases */

void regs_eliminate_base(struct regs *regs, pseudo_reg base)
{
    struct reg *r;
    struct reg *next;

    r = REGS_FIRST(regs);

    while (r) {
        next = REGS_NEXT(r);

        if (PSEUDO_REGS_SAME_BASE(r->reg, base))
            regs_remove(regs, r->reg);

        r = next;
    }
}

void regs_eliminate_bases(struct regs *regs, struct regs *bases)
{
    struct reg *r;

    REGS_FOREACH(r, bases)
        regs_eliminate_base(regs, r->reg);
}

/* remove all elements that share a
   base with reg, and then add reg */

void regs_replace_base(struct regs *regs, pseudo_reg reg)
{
    regs_eliminate_base(regs, reg);
    REGS_ADD(regs, reg);
}

/* select all elements of src that have the same
   base as base or bases and add them to dst */

void regs_select_base(struct regs *dst, struct regs *src, pseudo_reg base)
{
    struct reg *r;

    REGS_FOREACH(r, src)
        if (PSEUDO_REGS_SAME_BASE(r->reg, base))
            REGS_ADD(dst, r->reg);
}

void regs_select_bases(struct regs *dst, struct regs *src, struct regs *bases)
{
    struct reg *r;

    REGS_FOREACH(r, bases)
        regs_select_base(dst, src, r->reg);
}

/* eliminate elements of dst that do not have
   a base in common with any src bases */

void regs_intersect_bases(struct regs *dst, struct regs *src)
{
    struct regs tmp = REGS_INITIALIZER(tmp);

    regs_select_bases(&tmp, dst, src);
    regs_clear(dst);
    regs_move(dst, &tmp);
}

/* output the regs in human-readable
   form for debugging purposes */

void regs_output(struct regs *regs)
{
    struct reg *r;

    output("{ ");
    
    REGS_FOREACH(r, regs)
        output("%r ", r->reg);

    output("}");
}

/* vi: set ts=4 expandtab: */
