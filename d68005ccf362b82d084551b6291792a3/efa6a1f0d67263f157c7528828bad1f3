/* opt.c - optimization                                 ncc, the new c compiler

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

#include "../common/util.h"
#include "cc1.h"
#include "insn.h"
#include "block.h"
#include "dead.h"
#include "algebra.h"
#include "fold.h"
#include "loop.h"
#include "copy.h"
#include "prop.h"
#include "testz.h"
#include "slvn.h"
#include "load.h"
#include "testz.h"
#include "sched.h"
#include "opt.h"

/* remove unreachable code. our technique is simple: do a depth-first
   walk of the tree and eliminate all blocks that weren't visited. */

static blocks_iter_ret unreach0(struct block *b)
{
    if ((b != exit_block) && !BLOCK_VISITED(b))
        block_free(b);

    return BLOCKS_ITER_OK;
}

void unreach(void)
{
    blocks_walk(0, 0);
    blocks_iter(unreach0);
}

/* pruning is trivial: if a block is empty and it has but one CC_ALWAYS
   successor (which should always be the case for an empty block), then
   redirect all its predecessors to that successor and delete the block.
   beware the corner case where an empty block is its own successor: that
   is a (valid) infinite loop, and can't be removed. */

static blocks_iter_ret prune0(struct block *b)
{
    struct cessor *succ;
    struct cessor *pred;

    if ((b != entry_block) && BLOCK_EMPTY(b)) {
        if ((succ = block_always_successor(b)) && (succ->b != b))
        {
            while (pred = block_get_predecessor_n(b, 0))
                block_redirect_successors(pred->b, b, succ->b);

            block_free(b);
        }
    }

    return BLOCKS_ITER_OK;
}

void prune(void)
{
    blocks_iter(prune0);
}

/* remove no-ops: some optimizations leave I_NOPs where instructions used
   to be. we also take care of no-op self copies and mutual copies which
   typically arise from within the compiler rather than user constructs. */

static bool need_prune;

static blocks_iter_ret nop0(struct block *b)
{
    struct insn *insn;
    struct insn *prev;
    struct insn *next;
    pseudo_reg dst;
    pseudo_reg src;
    pseudo_reg prev_dst;
    pseudo_reg prev_src;

again:
    INSNS_FOREACH(insn, &b->insns) {
        prev = INSNS_PREV(insn);

        if (insn->op == I_NOP)
            goto remove;

        if (insn_copy(insn, &dst, &src)) {
            if (dst == src)
                goto remove;

            if (prev && insn_copy(prev, &prev_dst, &prev_src)
              && (dst == prev_src) && (src == prev_dst))
                goto remove;
        }

        continue;
remove:
        insns_remove(&b->insns, insn);
        goto again;
    }

    if (BLOCK_EMPTY(b))
        need_prune = TRUE;

    return BLOCKS_ITER_OK;
}

void nop(void)
{
    need_prune = FALSE;
    blocks_iter(nop0);

    if (need_prune)
        prune();
}

/* if a block has one always successor, and that successor has but
   one precedessor, then we coalesce them into one basic block. */

static blocks_iter_ret coal0(struct block *b)
{
    struct cessor *succ;
    struct block *succ_b;

    if ((b != entry_block) && (succ = block_always_successor(b))
      && (succ->b != b) && (succ->b != exit_block)
      && (block_nr_predecessors(succ->b) == 1)) {
        succ_b = succ->b;
        insns_append(&b->insns, &succ_b->insns);
        block_dup_successors(b, succ_b);
        block_free(succ_b);
        return BLOCKS_ITER_ABORT;
    }

    return BLOCKS_ITER_OK;
}

void coal(void)
{
    while (blocks_iter(coal0) == BLOCKS_ITER_ABORT)
        ;
}

/* really need to think out what order we're going to invoke these
   passes, under what circumstances to repeat them, and when passes
   might re-invoke other passes and such. the interactions between
   them are real, but we don't want to make compile times long but
   needlessly invoking passes that don't have anything left to do. */


/* early optimization occurs almost immediately after the IR generation for
   a function is complete. the IR has been dealiased, and at this stage is
   devoid of target machine registers, and is missing the target arg-loading
   preamble. switch blocks have not yet been generated: CC_SWITCHes remain. */

void opt_early(void)
{
    unreach();
    prune();
    dead();
    nop();
    testz_early();
    algebra();
    copy();
    fold();
    prop();
    slvn();
    copy();
    load();
    slvn();
    prop();
    copy();
    coal();
    testz_middle();
    sched_delay();
    copy();
    loop_invariants();
}

/* late optimization occurs immediately after the target IR is generated,
   but before register allocation. obviously, all these passes must be
   capable of working on the machine-dependent IR. */

void opt_late(void)
{
    testz_late();
}

/* post optimization occurs immediately before the function is output.
   target-specific function prolog/epilog are present. do final cleanup. */

void opt_post(void)
{
    coal();
    blocks_sequence();
}

/* vi: set ts=4 expandtab: */
