/* load.c - redundant load elimination                  ncc, the new c compiler

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
#include "regs.h"
#include "load.h"

/* redundant load elimination. sometimes the compiler stores a value
   to a memory location only to retrieve it a short time later. we
   do a simple track here of the last store encountered, and eliminate
   a load if it's just reloading the value (as the same type). we do
   this over extended basic blocks, inheriting the last store from a
   predecessor if it's the only one.

   because of aliasing, we can't be more extensive than this at present
   (e.g., tracking more I_STOREs than simply the last one). any write to
   memory must be treated as invalidating all memory.

   this only solves half the problem. the compiler also generates some
   redundant or useless stores (stores which are overwritten before they
   could possibly be read). some of these arise within the compiler and
   should (or must) be addressed at their sources (dealias.c, graph.c),
   but not all. dealing with these stores is more difficult than loads,
   at least beyond the basic block level, because a store only renders
   a previous store useless if it postdominates it. todo.

   this pass must occur after SLVN (and a follow-up copy propagation),
   otherwise it will miss most opportunities- address computations that
   resolve to the same address aren't apparent beforehand. */

static blocks_iter_ret init0(struct block *b)
{
    b->store = 0;

    return BLOCKS_ITER_OK;
}

static void load0(struct block *b)
{
    struct regs regs = REGS_INITIALIZER(regs);
    struct insn *insn;
    struct cessor *pred;

    if (pred = block_sole_predecessor(b))
        b->store = pred->b->store;

    INSNS_FOREACH(insn, &b->insns) {
        if ((insn->op == I_LOAD) && b->store
          && operand_is_same(insn->src1, b->store->dst)
          && (insn->dst->ts == b->store->src1->ts)) {

            insn_replace(insn, I_MOVE, operand_dup(insn->dst),
                                       operand_dup(b->store->src1));
            continue;
        }

        if (insn->op == I_STORE)
            b->store = insn;
        else if (insn_defs_mem(insn))
            b->store = 0;
        else if (b->store) {
            insn_defs_regs(insn, &regs, 0);

            if ((OPERAND_REG(b->store->dst)
                    && REGS_CONTAINS(&regs, b->store->dst->reg))
              || (OPERAND_REG(b->store->src1)
                    && REGS_CONTAINS(&regs, b->store->src1->reg)))
                b->store = 0;

            regs_clear(&regs);
        }
    }
}

void load(void)
{
    blocks_iter(init0);
    blocks_walk(load0, 0);
}

/* vi: set ts=4 expandtab: */
