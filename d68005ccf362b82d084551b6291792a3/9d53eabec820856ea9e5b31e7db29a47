/* loop.c - natural loop computation                    ncc, the new c compiler

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
#include "codes.h"
#include "assoc.h"
#include "block.h"
#include "output.h"
#include "target.h"
#include "live.h"
#include "dom.h"
#include "stack.h"
#include "loop.h"

/* the usual init/clear for block loop data */

void loop_init(struct loop *loop)
{
    BLKS_INIT(&loop->blks);
    loop->depth = 0;
}

void loop_clear(struct loop *loop)
{
    blks_clear(&loop->blks);
    loop->depth = 0;
}

/* in this analysis, we find natural loops using dominators. once identified,
   each block is assigned a loop depth, indicating its loop nesting level.
   the blocks that comprise each loop are collected into the loop.blks field
   of that loop's header block. when multiple loops share the same header,
   we conservatively assume they are part of the same loop, not nested. */

STACK_DECLARE(bstack, struct block *)
static STACK_DEFINE_PUSH(bstack, struct block *)
static STACK_DEFINE_POP(bstack, struct block *)

static struct bstack bstack = STACK_INITIALIZER(bstack);

/* given a back edge from tail to head, compute the
   blocks that comprise the implied natural loop, and
   add those blocks to the head's loop.blks. */

#define INSERT(b)                                                           \
    do {                                                                    \
        if (!BLKS_CONTAINS(&blks, (b))) {                                   \
            BLKS_ADD(&blks, (b));                                           \
            bstack_push(&bstack, (b));                                      \
        }                                                                   \
    } while (0)

static void loop_blocks(struct block *tail, struct block *head)
{
    struct blks blks = BLKS_INITIALIZER(blks);
    struct cessor *pred;
    struct block *b;
    int n;

    BLKS_ADD(&blks, head);

    if (head != tail) {
        INSERT(tail);

        while (!STACK_EMPTY(&bstack)) {
            b = bstack_pop(&bstack);

            for (n = 0; pred = block_get_predecessor_n(b, n); ++n)
                INSERT(pred->b);
        }
    }

    blks_union(&head->loop.blks, &blks);
    blks_clear(&blks);
}

/* initialize before analysis */

static blocks_iter_ret loop0(struct block *b)
{
    blks_clear(&b->loop.blks);
    b->loop.depth = 0;

    return BLOCKS_ITER_OK;
}

/* invoke loop_blocks() for each back edge from this block */

static blocks_iter_ret loop1(struct block *b)
{
    struct cessor *succ;
    int n;

    for (n = 0; succ = block_get_successor_n(b, n); ++n)
        if (DOMINATES(succ->b, b))
            loop_blocks(b, succ->b);

    return BLOCKS_ITER_OK;
}

/* assign a loop depth to every block */

static blocks_iter_ret loop2(struct block *b)
{
    struct blk *blk_b;

    BLKS_FOREACH(blk_b, &b->loop.blks)
        ++(blk_b->b->loop.depth);

    return BLOCKS_ITER_OK;
}

void loop_analyze(void)
{
    dom_analyze();

    blocks_iter(loop0);
    blocks_iter(loop1);
    blocks_iter(loop2);
}

/* loop-invariant code motion. detect computations which do not
   change in the body of a loop, and hoist them to a preheader. */

static struct block *head;          /* of loop currently being processed */
static struct block *preheader;     /* preheader we made for loop, if any */
static bool load_ok;                /* no stores occurred; loads invariant */

/* known invariants in this loop */

static struct regs invariants = REGS_INITIALIZER(invariants);
    
/* blocks in this function outside the current loop */

static struct blks out_blks = BLKS_INITIALIZER(out_blks);

/* when a computation is potentially invariant, we record the location
   (block/insn) of the definition as a maybe_inv for later inspection. */

struct loc
{
    struct block *b;
    struct insn *insn;
};

ASSOC_DECLARE(maybe_inv, pseudo_reg, reg, struct loc, loc)

static struct maybe_invs maybe_invs = ASSOC_INITIALIZER(maybe_invs);

static ASSOC_DEFINE_LOOKUP(maybe_inv, pseudo_reg, reg)

static ASSOC_DEFINE_INSERT(maybe_inv, pseudo_reg, reg, loc,
                           ASSOC_NULL_CONSTRUCT)

static ASSOC_DEFINE_UNSET(maybe_inv, pseudo_reg, reg, loc,
                           ASSOC_NULL_DESTRUCT)

static ASSOC_DEFINE_CLEAR(maybe_inv, loc, ASSOC_NULL_DESTRUCT)

#define MAYBE_INVS_FIRST()          ASSOC_FIRST(&maybe_invs)
#define MAYBE_INVS_NEXT(m)          ASSOC_NEXT(m)

/* before we start an analysis, reset BLOCK_FLAG_LOOPED on all
   blocks- it simply tracks which loop heads we've already done. */

static blocks_iter_ret invariants0(struct block *b)
{
    b->flags &= ~BLOCK_FLAG_LOOPED;
    return BLOCKS_ITER_OK;
}

/* scan blocks and pick the best candidate to
   process next. do innermost loops first. */

static blocks_iter_ret nexthead0(struct block *b)
{
    if (!BLKS_EMPTY(&b->loop.blks) && ((head == 0)
      || (b->loop.depth > head->loop.depth))
      && !(b->flags & BLOCK_FLAG_LOOPED))
        head = b;

    return BLOCKS_ITER_OK;
}

/* if this register has exactly one definition in b, and
   that definition domininates any/all uses in b (i.e.,
   the DEF precedes all USEs in b), return the insn where
   the DEF occurs, otherwise return 0. */

static struct insn *unique_def(struct block *b, pseudo_reg reg)
{
    struct regs defs = REGS_INITIALIZER(defs);
    struct regs uses = REGS_INITIALIZER(uses);
    struct insn *insn;
    struct insn *def;
    bool busted;

    def = 0;
    busted = FALSE;

    INSNS_FOREACH(insn, &b->insns) {
        insn_defs_regs(insn, &defs, 0);

        if (REGS_CONTAINS(&uses, reg) && (def == 0))
            busted = TRUE;

        if (REGS_CONTAINS(&defs, reg))
            if (def)
                busted = TRUE;
            else
                def = insn;
        
        regs_clear(&defs);
        regs_clear(&uses);

        if (busted)
            break;
    }

    if (busted)
        return 0;
    else
        return def;
}

/* we've found a new loop head, so do an initial analysis. we scan the loop
   for registers which are definitely invariant, and definitions which might
   also be invariant, populating invariants and maybe_invs accordingly.

   (remember that we're dealing with webs here, not naked registers. there is
   implied reachability in webs, i.e., a DEF of a register with some index can
   potentially reach any USE of that register with the same index. registers
   with different indexes look like different registers to us here, so we
   loosely use the term 'register' when, more precisely, we should say 'web'.)

   for the purposes of this routine:

   1. if a register is never defined in the body of the loop,
      it is (self-evidently) loop-invariant ( -> invariant )

   2. if a register has exactly one definition in the body of the loop,
      and this definition dominates all uses of the register inside and
      outside the loop, it might be loop-invariant ( -> maybe_invs )

   the dominance qualifications on #3 really speak to our desire to move the
   invariant definition out of the loop: we can't do that if the definition
   doesn't always occur before every possible use. */

static void loop_scan(void)
{
    struct regs candidates = REGS_INITIALIZER(candidates);
    struct blks out_defs = BLKS_INITIALIZER(out_defs);
    struct blks in_defs = BLKS_INITIALIZER(in_defs);
    struct blks read_blks = BLKS_INITIALIZER(read_blks);
    struct blk *def_b;
    struct insn *def_insn;
    struct reg *cand_r;
    struct maybe_inv *m;
    ccset ccs;
    
    kill_gather(&head->loop.blks, 0, &candidates);

    REGS_FOREACH(cand_r, &candidates) {
        kill_gather_blks(cand_r->reg, &out_blks, &out_defs, &read_blks);
        kill_gather_blks(cand_r->reg, &head->loop.blks,
                         &in_defs, &read_blks);

        if (BLKS_EMPTY(&in_defs)) {
            REGS_ADD(&invariants, cand_r->reg);
            goto skip; /* only defined outside loop, good */
        }

        if (!BLKS_EMPTY(&in_defs) && !BLKS_EMPTY(&out_defs))
            goto skip; /* multiple definitions, bad */

        if (BLKS_COUNT(&in_defs) > 1)
            goto skip; /* multiple definitions (inside loop), bad */

        /* we've narrowed it down to one block, make sure there's only
           one definition in that block and that it dominates all uses
           in that block, and that that block dominates all other uses. */

        def_b = BLKS_FIRST(&in_defs);
        def_insn = unique_def(def_b->b, cand_r->reg);

        if (!def_insn || !dominates_all(def_b->b, &read_blks))
            goto skip;

        /* if the insn isn't eligible for LICM, it's not eligible */

        if (!I_LICM(def_insn->op))
            goto skip;

        if (def_insn->flags & INSN_FLAG_VOLATILE)
            goto skip; /* never reorder these */

        /* insns that write memory are disqualified. if there are any
           stores in the loop at all, so are insns that read memory. */

        if (insn_defs_mem(def_insn))
            goto skip;
        
        if (!load_ok && insn_uses_mem(def_insn))
            goto skip;

        m = maybe_invs_insert(&maybe_invs, cand_r->reg);
        m->loc.b = def_b->b;
        m->loc.insn = def_insn;

skip:
        blks_clear(&out_defs);
        blks_clear(&in_defs);
        blks_clear(&read_blks);
    }

    regs_clear(&candidates);
}

/* a convenient routine to make a preheader for
   the current loop if none exists. takes care of
   setting cfg_changed to tell everyone the CFG
   has changed and data may need recomputation */

static bool cfg_changed;

static void make_preheader(void)
{
    if (preheader == 0) {
        preheader = block_preheader(head, &head->loop.blks);
        cfg_changed = TRUE;
    }
}

/* returns TRUE if this instruction is movable,
   in light of the current invariants. */

static bool movable(struct insn *insn)
{
    struct regs uses = REGS_INITIALIZER(uses);

    switch (insn->op)
    {
    case I_ADD:
    case I_SUB:
    case I_SHL:
        if (insn->dst->ts & (target->ptr_int | target->ptr_uint | T_INTS))
            return FALSE;
    }

    insn_uses_regs(insn, &uses, 0);
    regs_diff(&uses, &invariants);

    if (REGS_COUNT(&uses) == 0)
        return TRUE;
    else {
        regs_clear(&uses);
        return FALSE;
    }
}

/* here's where we perform loop-invariant code motion. we look at each
   possibly-invariant definition in the body of the loop (maybe_invs)
   and, if its operands are invariant and the insn is movable, we move
   the insn to a preheader (creating one if necessary) and declare the
   target of the definition an invariant register. rinse and repeat. */

static void loop_motion(void)
{
    struct maybe_inv *m;
    struct maybe_inv *next_m;
    bool moved;

    do {
        moved = FALSE;

        for (m = MAYBE_INVS_FIRST(); m; m = next_m) {
            next_m = MAYBE_INVS_NEXT(m);

            if (movable(m->loc.insn)) {
                make_preheader();
                BLOCK_APPEND_INSN(preheader, insn_dup(m->loc.insn));
                insns_remove(&m->loc.b->insns, m->loc.insn);
                REGS_ADD(&invariants, m->reg);
                maybe_invs_unset(&maybe_invs, m->reg);
                moved = TRUE;
            }
        }
    } while (moved);
}

void loop_invariants(void)
{
    bool changed;

    webs_analyze();
    blocks_iter(invariants0);
    cfg_changed = TRUE;

    for (;;) {
        if (cfg_changed) {
            kill_analyze();
            loop_analyze();
            cfg_changed = FALSE;
        }

        head = 0;
        blocks_iter(nexthead0);
        
        if (head == 0)
            break;
        else {
            head->flags |= BLOCK_FLAG_LOOPED;
            preheader = 0;
            load_ok = !blocks_def_mem(&head->loop.blks);
            blks_all(&out_blks);
            blks_diff(&out_blks, &head->loop.blks);

            loop_scan();
            loop_motion();

            blks_clear(&out_blks);
            regs_clear(&invariants);
            maybe_invs_clear(&maybe_invs);
        }
    }

    webs_strip();
}

/* vi: set ts=4 expandtab: */
