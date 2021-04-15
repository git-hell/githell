/* peep.c - AMD64-specific peephole optimizations       ncc, the new c compiler

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

#include "../../common/util.h"
#include "../cc1.h"
#include "../insn.h"
#include "../block.h"
#include "../live.h"
#include "insn.h"
#include "peep.h"

/* these are not all peephole optimizations in the strictest sense,
   but rather a miscellaneous collection of improvements.

   at some point, we may need to create some more general insn-matching
   framework rather than the ad hoc methods here. rainy day, perhaps. */


/* eliminate superfluous sign/zero-extensions.

   the code generator, for several reasons, is blissfully unaware when
   the upper bits of a register are already extended properly.
                                        
        sequences like:                 are replaced with:

        movzbl (%i16q),%i17d            movzbl (%i16q),%i17d
        movzbl %i17b,%i18d              movl %i17d, %18d

   the original sequence above appears because the AMD64 code generator
   automatically extends sub-word loads into the target register to avoid
   partial updates, and the front end casts the result per language rules.

        sometimes we also see:          which should be replaced with:

        movzbl %i22b, %i22d             movzbq %i22b, %i22q
        movslq %i22d, %i23q             movq %i22q, %i23q

   i've only ever seen this arise because of a prior setcc instruction,
   which modifies a byte register, so the code generator manually issues
   the first zero-extend instruction because the result should be an int.
   if that result is cast (to a long, in the above case), then a double
   extension results.

   typically, in cases like the above, the second instruction and reg
   are eliminated by the register allocator through coalescing. */

struct signzero
{
    insn_op first;          /* if the first insn is this .. */
    insn_op second;         /* .. and the second is this .. */
    insn_op new_first;      /* .. replace the first with this .. */
    insn_op new_second;     /* .. and the second with this. */
};

struct signzero signzeros[] =
{
    /* there are surely cases missing. also, note, when the upper 32
       bits are guaranteed to be zero, we can MOVL even for longs */

    /* repeated casts (e.g,. char -> int, char -> int) */

    {   AMD64_I_MOVZBL, AMD64_I_MOVZBL, AMD64_I_MOVZBL, AMD64_I_MOVL    },
    {   AMD64_I_MOVZWL, AMD64_I_MOVZWL, AMD64_I_MOVZWL, AMD64_I_MOVL    },
    {   AMD64_I_MOVSBL, AMD64_I_MOVSBL, AMD64_I_MOVSBL, AMD64_I_MOVL    },
    {   AMD64_I_MOVSWL, AMD64_I_MOVSWL, AMD64_I_MOVSWL, AMD64_I_MOVL    },
    {   AMD64_I_MOVZLQ, AMD64_I_MOVZLQ, AMD64_I_MOVZLQ, AMD64_I_MOVL    },
    {   AMD64_I_MOVSLQ, AMD64_I_MOVSLQ, AMD64_I_MOVSLQ, AMD64_I_MOVQ    },

    /* overlapping casts (e.g., short -> int, short -> long) */

    {   AMD64_I_MOVZBL, AMD64_I_MOVZBQ, AMD64_I_MOVZBQ, AMD64_I_MOVL    },
    {   AMD64_I_MOVSBL, AMD64_I_MOVSBQ, AMD64_I_MOVSBQ, AMD64_I_MOVQ    },
    {   AMD64_I_MOVZWL, AMD64_I_MOVZWQ, AMD64_I_MOVZWQ, AMD64_I_MOVL    },
    {   AMD64_I_MOVSWL, AMD64_I_MOVSWQ, AMD64_I_MOVSWQ, AMD64_I_MOVQ    },

    /* chained casts (e.g., char -> int, int -> long) */

    {   AMD64_I_MOVZBL, AMD64_I_MOVSLQ, AMD64_I_MOVZBQ, AMD64_I_MOVL    },
    {   AMD64_I_MOVZBL, AMD64_I_MOVZLQ, AMD64_I_MOVZBQ, AMD64_I_MOVL    }
};

static bool signzero(struct insn *insn)
{
    struct insn *next;
    int i;
    
    if ((next = INSNS_NEXT(insn)) == 0)
        return FALSE;

    for (i = 0; i < ARRAY_SIZE(signzeros); ++i)
        if ((insn->op == signzeros[i].first)
          && (next->op == signzeros[i].second))
            break;

    if (i == ARRAY_SIZE(signzeros))
        return FALSE;

    if (!AMD64_OPERAND_REG(insn->amd64[1]))
        return FALSE;

    if (!AMD64_OPERAND_REG(next->amd64[0]))
        return FALSE;

    if (insn->amd64[1]->reg != next->amd64[0]->reg)
        return FALSE;

    insn->op = signzeros[i].new_first;
    next->op = signzeros[i].new_second;
    return TRUE;
}

/* the lowest of low-hanging fruit, the oldest optimization in
   the world: replace movl $0, <reg> with xorl <reg>, <reg>.

   we can't do this if the condition codes are live, since a move
   leaves them unaffected, but xor does not. */

static void zero(struct block *b, struct insn *insn)
{
    if (((insn->op == AMD64_I_MOVL) || (insn->op == AMD64_I_MOVQ))
      && AMD64_OPERAND_ZERO(insn->amd64[0])
      && AMD64_OPERAND_REG(insn->amd64[1])
      && (range_by_use(&b->live, PSEUDO_REG_CC, insn->index) == 0))
        insn_replace(insn, AMD64_I_ZERO, amd64_operand_dup(insn->amd64[1]),
                                         amd64_operand_dup(insn->amd64[1]));
}

/* sometimes the code generator creates sequences like:

                    leal 1(%rax), %eax      OR
                    leal (,%rbx,2), %ebx

   when 

                    addl $1, %eax           OR
                    shll $1, %ebx

   would suffice. this results when distinct registers in the IR coalesce
   during allocation. we take the time here to revert these sequences to
   their more natural and slightly-more-efficient counterparts. */

#define REVERT0(l, q)   ((insn->op == AMD64_I_LEAL) ? (l) : (q))

#define LOG_2(s)        ((s == 2) ? 1 : ((s == 4) ? 2 : 3))

static void revert(struct insn *insn)
{
    struct amd64_operand *src;
    struct amd64_operand *dst;

    if ((insn->op != AMD64_I_LEAL) && (insn->op != AMD64_I_LEAQ))
        return;

    src = insn->amd64[0];
    dst = insn->amd64[1];

    if ((src->idx == dst->reg) && (src->reg == PSEUDO_REG_NONE) &&
        (src->sym == 0) && (src->i == 0)) {

            /* lea (%r,s), %r -> shl $c, %r */

            insn_replace(insn, REVERT0(AMD64_I_SHLL, AMD64_I_SHLQ),
                             amd64_operand_con(dst->ts, LOG_2(src->scale), 0),
                             amd64_operand_dup(dst));
            return;
    }

    if ((src->reg == dst->reg) && (src->scale == 1)
      && (src->sym == 0) && (src->i == 0)) {

            /* lea (%r1,%r2), %r1 -> add %r2, %r1 */

            insn_replace(insn, REVERT0(AMD64_I_ADDL, AMD64_I_ADDQ),
                               amd64_operand_reg(dst->ts, src->idx),
                               amd64_operand_dup(dst));
            return;
    }

    if ((src->idx == dst->reg) && (src->scale == 1)
      && (src->sym == 0) && (src->i == 0)) {

            /* lea (%r2,%r1), %r1 -> add %r2, %r1 */

            insn_replace(insn, REVERT0(AMD64_I_ADDL, AMD64_I_ADDQ),
                               amd64_operand_reg(dst->ts, src->reg),
                               amd64_operand_dup(dst));
            return;
    }

    if ((src->reg == dst->reg) && (src->idx == PSEUDO_REG_NONE)) {

            /* lea c(%r), %r -> add $c, %r */

            insn_replace(insn, REVERT0(AMD64_I_ADDL, AMD64_I_ADDQ),
                               amd64_operand_con(dst->ts, src->i, src->sym),
                               amd64_operand_dup(dst));
            return;
    }
}

/* amd64_peep() runs before register allocation,
   whereas amd64_peep_post() runs after, and
   amd64_peep_final() runs last of all. */

static blocks_iter_ret peep0(struct block *b)
{
    struct insn *insn;

again:
    INSNS_FOREACH(insn, &b->insns)
        if (signzero(insn))
            goto again;

    return BLOCKS_ITER_OK;
}

void amd64_peep(void)
{
    blocks_iter(peep0);
}

static blocks_iter_ret post0(struct block *b)
{
    struct insn *insn;

    INSNS_FOREACH(insn, &b->insns)
        revert(insn);

    return BLOCKS_ITER_OK;
}

void amd64_peep_post(void)
{
    blocks_iter(post0);
}

static blocks_iter_ret final0(struct block *b)
{
    struct insn *insn;

    live_analyze_ccs(b);

    INSNS_FOREACH(insn, &b->insns)
        zero(b, insn);

    return BLOCKS_ITER_OK;
}

void amd64_peep_final(void)
{
    blocks_iter(final0);
}

/* vi: set ts=4 expandtab: */
