/* dead.c - dead store elimination                      ncc, the new c compiler

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
#include "block.h"
#include "symbol.h"
#include "dead.h"
#include "opt.h"

/* the hard work in identifying dead stores is live-variable analysis,
   which is done elsewhere. we have to further qualify them to be sure
   they're not used in instructions with side effects, but that's it.

   I_CALL instructions for functions that return values that are unused
   are not recognized as dead here, because they have side effects. this
   is harmless: we make another pass after the target code is generated
   and the dead store will be eliminated then.
 
   we don't remove instructions, we replace them with I_NOPs to keep the
   live data intact. even if we've partially-invalidated the live data,
   we continue through an iteration until we've eliminated all the dead
   stores we can before recomputing it. theoretically this keeps the
   number of recomputations to a minimum, though it does require that
   we subsequently kill the I_NOPs with a [low-cost] nop() pass. */

static bool changed;
static bool invalidated;

static blocks_iter_ret dead0(struct block *b)
{
    struct regs regs = REGS_INITIALIZER(regs);
    struct reg *reg;
    struct range *r;
    struct insn *insn;
    int changes;

    do {
        changes = 0;

        INSNS_FOREACH(insn, &b->insns) {
            if (insn->op == I_NOP) goto skip;
            if (insn_defs_mem(insn)) goto skip;
            if (insn_side_effects(insn)) goto skip;

            insn_defs_regs(insn, &regs, 0);

            if (insn_defs_cc(insn))
                REGS_ADD(&regs, PSEUDO_REG_CC);

            REGS_FOREACH(reg, &regs) {
                r = range_by_def(&b->live, reg->reg, insn->index);
                if (!RANGE_DEAD(r)) goto skip;
            }
    
            ++changes;
            changed = TRUE;
            insn_replace(insn, I_NOP);

            if (!live_kill_insn(&b->live, insn->index))
                invalidated = TRUE;
skip:
            regs_clear(&regs);
        }
    } while (changes);

    return BLOCKS_ITER_OK;
}

void dead(void)
{
    bool need_nop = FALSE;

    invalidated = TRUE;

    do {
        if (invalidated)
            live_analyze();

        changed = FALSE;
        invalidated = FALSE;
        blocks_iter(dead0);

        if (changed)
            need_nop = TRUE;
    } while (changed && invalidated);

    if (need_nop)
        nop();
}

/* vi: set ts=4 expandtab: */
