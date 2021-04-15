/* slvn.c - super-local value numbering                 ncc, the new c compiler

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
#include "assoc.h"
#include "insn.h"
#include "block.h"
#include "slvn.h"

/* standard value numbering, though a bit odd in implementation.

   a struct lvns maintains an association between pseudo registers and
   insns. if two registers point at the same insn, they have the same
   value number, which is the number recorded in the dst of that insn.
   
   we iterate over a block, first examining the operands and updating their
   number fields by consulting the struct lvns. if there is no such number,
   we invent a new one and insert a new struct lvn (which points at a fake
   I_NUMBER in number_insns, created solely for this purpose). then we scan
   the insns listed in struct lvns to see if this expression was previously
   computed into a register that still holds the result.

   if the expression has previously been computed:
        1. update the struct lvns so the current insn's dst reg
           points at the expression entry's insn (thus assigning
           dst the same value number as the previous computation), and
        2. replace the insn with an I_MOVE from the expression
           entry's register.

   note that multiple registers may point at the same insn (indeed, that
   is the point), and it does not matter which of the existing entries is
   the source of the I_MOVE. copy propagation will unify them later.

   if the expression has NOT been previously computed:
        1. assign a new value number to the dst register, and update
           the struct lvns so the dst now points at this latest insn.

   constants are not assigned value numbers. we compare them directly.

   when we encounter an instruction that is not subject to value numbering,
   we simply invalidate all the DEFd registers in that instruction- purging
   them from the struct lvns. similarly, all entries associated with I_LOAD
   insns are invalidated when any memory stores occur.

   we extend the basic block algorithm to extended basic blocks by simply
   inheriting state from our predecessor when we only have one. we perform
   a preorder walk to ensure the predecessors are processed first.

   as mentioned above, this implementation is a bit odd, and inefficient.
   it is written this way, reusing existing containers and structures, in
   the interests of expediency. it would not hurt to reimplement this on
   a rainy day with better asymptotic performance, which is quadratic with
   respect to block size. todo. */

#define LVN_CONSTRUCT(i)    (*(i) = 0)
#define LVN_DUP(dst, src)   (*(dst) = *(src))

static ASSOC_DEFINE_LOOKUP(lvn, pseudo_reg, reg)

static ASSOC_DEFINE_INSERT(lvn, pseudo_reg, reg, insn,
                           LVN_CONSTRUCT)

static ASSOC_DEFINE_DUP(lvn, pseudo_reg, reg, insn, LVN_DUP)

static ASSOC_DEFINE_UNSET(lvn, pseudo_reg, reg, insn, ASSOC_NULL_DESTRUCT)
static ASSOC_DEFINE_CLEAR(lvn, insn, ASSOC_NULL_DESTRUCT)

#define LVNS_INITIALIZER(ls)        ASSOC_INITIALIZER(ls)
#define LVNS_FOREACH(l, ls)         ASSOC_FOREACH(l, ls)

static struct insns number_insns;

static value_number last_value_number;

#define NEXT_VALUE_NUMBER()     (++last_value_number)

/* if the operand is a pseudo_reg, determine its value_number
   (or allocate a new one) and update the operand accordingly. */

static void assign(struct lvns *lvns, struct operand *o)
{
    struct lvn *l;
    struct insn *insn;

    if (OPERAND_REG(o)) {
        l = lvns_insert(lvns, o->reg);
        
        if (l->insn == 0) {
            insn = insn_new(I_NUMBER, operand_dup(o));
            insn_append(&number_insns, insn);
            insn->dst->number = NEXT_VALUE_NUMBER();
            l->insn = insn;
        }

        o->number = l->insn->dst->number;
    }
}

/* search for the oldest matching expression,
   return its lvn entry, 0 if no match. */

#define MATCH_OPERAND(o1, o2)                                               \
    do {                                                                    \
        if (OPERAND_CLASS(o1) != OPERAND_CLASS(o2))                         \
            goto skip;                                                      \
                                                                            \
        if (OPERAND_REG(o1)) {                                              \
            if ((o1)->number != ((o2)->number))                             \
                goto skip;                                                  \
                                                                            \
            if (((o1)->ts != (o2)->ts))                                     \
                goto skip;                                                  \
        } else                                                              \
            if (!operand_is_same(o1, o2))                                   \
                goto skip;                                                  \
    } while (0)

static struct lvn *match(struct lvns *lvns, struct insn *insn)
{
    struct lvn *l;

    LVNS_FOREACH(l, lvns) {
        if (l->insn->op != insn->op)
            continue;

        if (l->insn->dst->ts != insn->dst->ts)
            continue;

        MATCH_OPERAND(l->insn->src1, insn->src1);
        MATCH_OPERAND(l->insn->src2, insn->src2);

        return l;
skip:   ;
    }

    return 0;
}

/* remove every entry that references an I_LOAD instruction */

static void lvns_invalidate_mem(struct lvns *lvns)
{
    struct lvn *l;

again:
    LVNS_FOREACH(l, lvns) {
        if (l->insn->op == I_LOAD) {
            lvns_unset(lvns, l->reg);
            goto again;
        }
    }
}

/* initialize state */

static blocks_iter_ret init0(struct block *b)
{
    ASSOC_INIT(&b->lvns);
    return BLOCKS_ITER_OK;
}

/* do value numbering for the block */

static void slvn0(struct block *b)
{
    struct insn *insn;
    struct lvn *l;
    struct lvn *m;
    struct cessor *pred;

    if (pred = block_sole_predecessor(b))
        lvns_dup(&b->lvns, &pred->b->lvns);

    INSNS_FOREACH(insn, &b->insns) {
        if (I_SLVN(insn->op) && !(insn->flags & INSN_FLAG_VOLATILE)) {
            assign(&b->lvns, insn->src1);
            assign(&b->lvns, insn->src2);

            if (m = match(&b->lvns, insn)) {
                insn->dst->number = m->insn->dst->number;
                l = lvns_insert(&b->lvns, insn->dst->reg);
                l->insn = m->insn;
                insn_replace(insn, I_MOVE, operand_dup(insn->dst),
                                           operand_dup(m->insn->dst));
            } else {
                insn->dst->number = NEXT_VALUE_NUMBER();
                l = lvns_insert(&b->lvns, insn->dst->reg);
                l->insn = insn;
            }
        } else {
            if (insn->dst && !I_USE_DST(insn->op))
                lvns_unset(&b->lvns, insn->dst->reg);

            if (insn_defs_mem(insn))
                lvns_invalidate_mem(&b->lvns);
        }
    }
}

/* empty the state */

static blocks_iter_ret clear0(struct block *b)
{
    lvns_clear(&b->lvns);
    return BLOCKS_ITER_OK;
}

void slvn(void)
{
    last_value_number = VALUE_NUMBER_NONE;
    INSNS_INIT(&number_insns);

    blocks_iter(init0);
    blocks_walk(slvn0, 0);
    blocks_iter(clear0);

    insns_clear(&number_insns);
}

/* vi: set ts=4 expandtab: */
