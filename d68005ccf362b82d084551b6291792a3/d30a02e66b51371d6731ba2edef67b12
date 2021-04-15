/* live.c - live variable analysis                      ncc, the new c compiler

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
#include "regs.h"
#include "block.h"
#include "insn.h"
#include "output.h"
#include "live.h"

#define LIVE_INIT(l)                    TAILQ_INIT(l)
#define LIVE_FIRST(l)                   TAILQ_FIRST(l)
#define LIVE_PREV(r)                    TAILQ_PREV(r, live, links)
#define LIVE_NEXT(r)                    TAILQ_NEXT(r, links)
#define LIVE_APPEND(l, r)               TAILQ_INSERT_TAIL(l, r, links)
#define LIVE_INSERT_BEFORE(bef, r)      TAILQ_INSERT_BEFORE(bef, r, links)
#define LIVE_REMOVE(l, r)               TAILQ_REMOVE(l, r, links)
#define LIVE_FOREACH(v, l)              TAILQ_FOREACH(v, l, links)

/* TRUE if r1 is ordered after r2 */

#define RANGE_AFTER(r1, r2)     (((r1)->reg == (r2)->reg)                   \
                                    ? ((r1)->def > (r2)->def)               \
                                    : ((r1)->reg > (r2)->reg))

/* TRUE if the use of reg r at index u occurs before range r1 */

#define RANGE_AFTER_USE(r1, r, u)   (((r1)->reg > (r)) ||                   \
                                        (((r1)->reg == (r))                 \
                                        && ((r1)->def >= (u))))

/* TRUE if the use of reg r at index u is covered by range r1 */

#define RANGE_COVERS_USE(r1, r, u)  (((r1)->reg == (r)) &&                  \
                                        ((r1)->def < (u)) &&                \
                                        ((r1)->last >= (u)))

/* TRUE if range r1 overlaps range r2 */

#define RANGE_OVERLAPS(r1, r2)              (((r1)->last > (r2)->def) &&    \
                                             ((r2)->last > (r1)->def))

/* create a new range with the specified details, insert it
   into the live data at its proper position, and return it. */

static struct range *range_new(struct live *live, pseudo_reg reg,
                               insn_index def, insn_index last)
{
    struct range *new;
    struct range *r;

    new = safe_malloc(sizeof(struct range));
    new->reg = reg;
    new->def = def;
    new->last = last;
    new->uses = 0;

    for (r = LIVE_FIRST(live); r; r = LIVE_NEXT(r))
        if (RANGE_AFTER(r, new)) {
            LIVE_INSERT_BEFORE(r, new);
            break;
        }

    if (r == 0)
        LIVE_APPEND(live, new);

    return new;
}

/* remove the range from the live data and free it */

static void range_free(struct live *live, struct range *r)
{
    LIVE_REMOVE(live, r);
    free(r);
}

/* return the range headed by a def of a reg, 0 if none */

struct range *range_by_def(struct live *live, pseudo_reg reg, insn_index def)
{
    struct range *r;

    LIVE_FOREACH(r, live)
        if ((r->reg == reg) && (r->def == def))
            return r;

    return 0;
}

/* return the range that covers reg at use, or 0 if none */

struct range *range_by_use(struct live *live, pseudo_reg reg, insn_index use)
{
    struct range *r;

    LIVE_FOREACH(r, live) {
        if (RANGE_COVERS_USE(r, reg, use))
            return r;

        if (RANGE_AFTER_USE(r, reg, use))
            break;
    }

    return 0;
}

/* initialize new live data */

void live_init(struct live *live)
{
    LIVE_INIT(live);
    REGS_INIT(&live->in);
    REGS_INIT(&live->out);
    REGS_INIT(&live->def);
    REGS_INIT(&live->use);
}

/* remove all ranges from the live data */

void live_clear(struct live *live)
{
    struct range *r;

    while (r = LIVE_FIRST(live))
        range_free(live, r);

    regs_clear(&live->in);
    regs_clear(&live->out);
    regs_clear(&live->def);
    regs_clear(&live->use);
}

/* live_analyze_local() [and its helpers, live_def() and live_use()]
   perform local live-variable analysis on a block, building its live
   ranges and DEF and USE sets. every register reference in the block
   explicit or implicit (e.g., PSEUDO_REG_CC) is recorded in a range,
   even seemingly dead stores, but the ranges will not be complete
   until the global phase determines what is live in/out. */

static void live_def(struct live *live, pseudo_reg reg, insn_index def)
{
    range_new(live, reg, def, def);
}

/* on use, we update the last of the applicable range (which would be
   the last range whose def is not at or after the use), or, if no such
   def exists, create a new range indicating the register is live-in. */

static void live_use(struct live *live, pseudo_reg reg, insn_index use)
{
    struct range *match;
    struct range *r;

    match = 0;
    r = LIVE_FIRST(live);

    while (r) {
        if (RANGE_AFTER_USE(r, reg, use))
            break;

        if (r->reg == reg)
            match = r;

        r = LIVE_NEXT(r);
    }

    if (match)
        match->last = use;
    else
        match = range_new(live, reg, INSN_INDEX_BEFORE, use);

    ++(match->uses);
}

static blocks_iter_ret live_analyze_local(struct block *b)
{
    struct regs regs = REGS_INITIALIZER(regs);
    struct reg *regs_r;
    struct insn *insn;
    struct range *r;
    pseudo_reg reg;
    ccset ccs;

    live_clear(&b->live);

    INSNS_FOREACH(insn, &b->insns) {
        ccs = insn_uses_cc(insn);

        if (!CCSET_EMPTY(ccs))
            live_use(&b->live, PSEUDO_REG_CC, insn->index);

        if (insn_defs_cc(insn))
            live_def(&b->live, PSEUDO_REG_CC, insn->index);

        if (insn_uses_regs(insn, &regs, 0)) {
            REGS_FOREACH(regs_r, &regs)
                live_use(&b->live, regs_r->reg, insn->index);

            regs_clear(&regs);
        }

        if (insn_defs_regs(insn, &regs, 0)) {
            REGS_FOREACH(regs_r, &regs)
                live_def(&b->live, regs_r->reg, insn->index);

            regs_clear(&regs);
        }
    }

    if (block_conditional(b))
        live_use(&b->live, PSEUDO_REG_CC, INSN_INDEX_BRANCH);

    if (BLOCK_SWITCH(b) && OPERAND_REG(b->control))
        live_use(&b->live, b->control->reg, INSN_INDEX_BRANCH);

    reg = PSEUDO_REG_NONE;

    LIVE_FOREACH(r, &b->live) {
        if (r->reg == reg)
            continue;

        if (r->def == INSN_INDEX_BEFORE)
            REGS_ADD(&b->live.use, r->reg);
        else
            REGS_ADD(&b->live.def, r->reg);

        reg = r->reg;
    }

    return BLOCKS_ITER_OK;
}

/* live_analyze() does the global live variable analysis. after local
   block analysis has been done, standard iterative data flow analysis
   is applied with live_analyze_global() to build live in and live out
   sets. finally, live_analyze_final() amends the local live ranges in
   each block to account for live/in out variables. */

static blocks_iter_ret live_analyze_global(struct block *b)
{
    struct regs prev_in = REGS_INITIALIZER(prev_in);
    struct regs prev_out = REGS_INITIALIZER(prev_out);
    struct cessor *succ;
    blocks_iter_ret ret = BLOCKS_ITER_OK;
    int n;

    regs_move(&prev_out, &b->live.out);

    for (n = 0; succ = block_get_successor_n(b, n); ++n)
        regs_union(&b->live.out, &succ->b->live.in);

    regs_move(&prev_in, &b->live.in);
    regs_union(&b->live.in, &b->live.out);
    regs_diff(&b->live.in, &b->live.def);
    regs_union(&b->live.in, &b->live.use);

    if (!regs_equal(&prev_in, &b->live.in) ||
      !regs_equal(&prev_out, &b->live.out))
        ret = BLOCKS_ITER_REITER;

    regs_clear(&prev_in);
    regs_clear(&prev_out);

    return ret;
}

static blocks_iter_ret live_analyze_final(struct block *b)
{
    struct range *r;
    struct reg *regs_r;

    /* now we add ranges for the live out crowd. for a live out register,
       this either extends its last range to beyond the end of the block,
       or it creates a new range that extends from before this block to 
       beyond it (for registers that are merely transiting the block). */

    REGS_FOREACH(regs_r, &b->live.out)
        live_use(&b->live, regs_r->reg, INSN_INDEX_AFTER);

    /* we never intentionally carry the condition codes across
       block boundaries, so don't clutter the sets with them. */

    regs_remove(&b->live.in, PSEUDO_REG_CC);
    regs_remove(&b->live.out, PSEUDO_REG_CC);
    regs_remove(&b->live.def, PSEUDO_REG_CC);
    regs_remove(&b->live.use, PSEUDO_REG_CC);

    return BLOCKS_ITER_OK;
}

void live_analyze(void)
{
    blocks_iter(live_analyze_local);
    blocks_iter(live_analyze_global);
    blocks_iter(live_analyze_final);
}

/* called to do live analysis when all we're 
   concerned with is condition_codes. for now,
   we actually do a full local analysis, though
   the data for the other variables is bogus
   without global analysis. */

void live_analyze_ccs(struct block *b)
{
    live_analyze_local(b);
}

/* determine which registers interfere with
   reg in live, and add those to regs. as
   usual we exclude PSEUDO_REG_CC here. */

void live_interf(struct live *live, pseudo_reg reg, struct regs *regs)
{
    struct range *r1;
    struct range *r2;
    
    LIVE_FOREACH(r1, live) {
        if (r1->reg == reg) {
            LIVE_FOREACH(r2, live) {
                if ((r2->reg == reg) || (r2->reg == PSEUDO_REG_CC))
                    continue;

                if (RANGE_OVERLAPS(r1, r2))
                    REGS_ADD(regs, r2->reg);
            }
        }
    }
}

/* the caller has killed an insn in the block corresponding to the
   live data. update the live data as best we can, and return TRUE if
   the live data remains valid, FALSE otherwise. in the latter case,
   the data is still usable, it's just overly pessimistic.

   incremental update is possible when:

        1. all the registers DEFd by the instruction are dead, and
        2. this is the only USE of the live ranges for the registers
           USEd in this instruction, and they were not LIVE IN

   the ranges for the DEFd registers are simply removed, the ranges for
   the USEd registers are made dead stores.

   if we tracked ALL uses of a range, rather than just a count, we would
   be successful more often. that may or may not be worth the trouble. */

bool live_kill_insn(struct live *live, insn_index index)
{
    struct range *r;
    struct range *next;

    for (r = LIVE_FIRST(live); r; r = next) {
        next = LIVE_NEXT(r);

        if (r->def == index) {
            if (r->uses != 0)
                return FALSE;

            range_free(live, r);
        } else if (r->last == index) {
            if ((r->uses != 1) || (r->def == INSN_INDEX_BEFORE))
                return FALSE;

            r->uses = 0;
            r->last = r->def;
        }
    }

    return TRUE;
}

/* returns the set of condition_codes defined by the insn
   in the block which are actually used by subsequent code.
   errs pessimistically (CCSET_ALL) if not sure- currently
   we only track a usage count (not all uses) so if there
   is more than one use, that's what the answer will be. */

ccset live_ccs(struct block *b, struct insn *insn)
{
    struct range *r;
    ccset ccs;

    CCSET_CLEAR(ccs);
    r = range_by_def(&b->live, PSEUDO_REG_CC, insn->index);

    if (r && !RANGE_DEAD(r)) {
        if (r->uses == 1) {
            if (r->last == INSN_INDEX_BRANCH)
                ccs = block_ccs(b);
            else
                ccs = insn_uses_cc(insn);
        } else
            ccs = CCSET_ALL;
    }

    return ccs;
}

/* dump the live information for
   debugging purposes */

void live_debug(struct live *live)
{
    struct range *r;

    output("; LIVE    IN: %R\n", &live->in);
    output("; LIVE   OUT: %R\n", &live->out);
    output(";        DEF: %R\n", &live->def);
    output(";        USE: %R\n", &live->use);

    LIVE_FOREACH(r, live)
        output(";      RANGE: %r def=%i uses=%d last=%i\n",
            r->reg, r->def, r->uses, r->last);
}

/* vi: set ts=4 expandtab: */
