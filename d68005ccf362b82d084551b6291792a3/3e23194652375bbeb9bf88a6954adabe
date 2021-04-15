/* webs.c - allocatable objects (webs)                  ncc, the new c compiler

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
#include "insn.h"
#include "regs.h"
#include "kill.h"
#include "block.h"
#include "target.h"
#include "symbol.h"
#include "output.h"
#include "webs.h"

/* when breaking the IR into webs, we subscript each register in the IR such
   that, if two appearances of a register have different indices, then those
   two appearances are definitely not in the same live range. this is a kind
   of reaching definitions analysis, though it is somewhat more conservative,
   associating DEFs with USEs that a more traditional analysis would not.

   we compute as follows:

   1. label each DEF in every block with a unique index and construct 
      the GEN() sets. a unique web is created for each DEF.

   2. perform forward iterative data analysis to compute IN() and OUT().

   3. examine every USE in each block to determine which definitions
      reach it, and use that information to merge overlapping webs. we
      label the USEs so we can find their webs easily in the next step.

   4. relabel each DEF or USE according to which web it belongs to. all
      the synonyms in a web collapse into one: we arbitarily choose the
      first synonym as the new common synonym.

   physical registers are left untouched (their indices remain 0). those will
   only appear in target-dependent IR analyzed by the graph allocator, and it
   is pointless to break them up as the allocator's hands are tied. for pseudo
   registers, index 0 is reserved to mean "a definition we haven't seen". */

static TAILQ_HEAD(, web) webs = TAILQ_HEAD_INITIALIZER(webs);

struct web
{
    struct regs synonyms;
    TAILQ_ENTRY(web) links;
};

#define WEBS_FIRST()        TAILQ_FIRST(&webs)
#define WEBS_APPEND(w)      TAILQ_INSERT_TAIL(&webs, w, links)
#define WEBS_FOREACH(w)     TAILQ_FOREACH(w, &webs, links)
#define WEBS_REMOVE(w)      TAILQ_REMOVE(&webs, w, links)

#define WEB_FIRST(w)        (REGS_FIRST(&(w)->synonyms)->reg)

/* create a new web populated with
   the given synonym */

static void web_new(pseudo_reg reg)
{
    struct web *w;

    w = safe_malloc(sizeof(struct web));
    REGS_INIT(&w->synonyms);
    REGS_ADD(&w->synonyms, reg);
    WEBS_APPEND(w);
}

/* remove a web and free it */

static void web_free(struct web *w)
{
    regs_clear(&w->synonyms);
    WEBS_REMOVE(w);
    free(w);
}

/* clear all webs. if debugging, dump
   their contents first. */

static void webs_clear(void)
{
    struct web *w;

    if (debug_flag_r)
        output("# REACH WEBS\n");

    while (w = WEBS_FIRST()) {
        if (debug_flag_r)
            output("#        WEB: %R\n", &w->synonyms);

        web_free(w);
    }
}

/* find the web which contains the given synonym */

static struct web *web_find(pseudo_reg reg)
{
    struct web *w;
    struct reg *r;

    WEBS_FOREACH(w) {
        REGS_FOREACH(r, &w->synonyms) {
            if (!PSEUDO_REGS_SAME_BASE(r->reg, reg))
                break; /* definitely not this web */

            if (r->reg == reg)
                return w;
        }
    }

    return 0;
}

/* a USE has been encountered with no candidate DEFs:
   this is a reference to an uninitialized variable.
   create a web associated with the uninitialized reg
   (which has a zero subscript) if necessary. */

static void web_uninit(pseudo_reg reg)
{
    if (web_find(reg) == 0)
        web_new(reg);
}

/* these synonyms share a use: merge
   the webs containing them. returns
   the merged web */

static struct web *webs_merge(struct regs *synonyms)
{
    struct web *w = 0;
    struct web *w2;
    struct reg *r;

    REGS_FOREACH(r, synonyms) {
        w2 = web_find(r->reg);

        if (w == 0)     /* first entry */
            w = w2;
        
        if (w == w2)    /* same entry, no merge */
            continue;

        regs_union(&w->synonyms, &w2->synonyms);
        web_free(w2);
    }

    return w;
}

/* initialize the state. this requires
   the live information be built already. */

static struct regs all_regs = REGS_INITIALIZER(all_regs);

static blocks_iter_ret reset0(struct block *b)
{
    regs_union(&all_regs, &b->live.def);
    regs_union(&all_regs, &b->live.use);

    REGS_INIT(&b->webs.in);
    REGS_INIT(&b->webs.gen);
    REGS_INIT(&b->webs.out);

    return BLOCKS_ITER_OK;
}

static void reset(void)
{
    struct reg *r;

    blocks_iter(reset0);

    REGS_FOREACH(r, &all_regs)
        if (!target->reg_physical(r->reg))
            REG_RESET_IDX(r->reg);

    regs_clear(&all_regs);
}

/* clear the state; we also take the time to
   dump the state if debugging is enabled */

static blocks_iter_ret clear0(struct block *b)
{
    if (debug_flag_r) {
        output("# WEBS, BLOCK %L\n", b->label);
        output("#       GEN: %R\n", &b->webs.gen);
        output("#        IN: %R\n", &b->webs.in);
        output("#       OUT: %R\n", &b->webs.out);
    }

    regs_clear(&b->webs.gen);
    regs_clear(&b->webs.in);
    regs_clear(&b->webs.out);

    return BLOCKS_ITER_OK;
}

/* first pass: find and label all DEFs- creating a
   new web for each- and construct the GEN set. */

static blocks_iter_ret def0(struct block *b)
{
    struct regs defs = REGS_INITIALIZER(defs);
    struct reg *r;
    struct insn *insn;
    pseudo_reg reg;
    int idx;

    INSNS_FOREACH(insn, &b->insns) {
        insn_defs_regs(insn, &defs, 0);

        REGS_FOREACH(r, &defs) {
            reg = r->reg;

            if (!target->reg_physical(reg)) {
                idx = REG_NEXT_IDX(reg);
                PSEUDO_REG_SET_IDX(reg, idx);
                web_new(reg);
                regs_replace_base(&b->webs.gen, reg);
                insn_substitute_reg(insn, r->reg, reg, INSN_SUBSTITUTE_DEFS);
            }
        }

        regs_clear(&defs);
    }

    regs_intersect_bases(&b->webs.gen, &b->live.out);

    return BLOCKS_ITER_OK;
}

/* iterative pass: compute IN() and OUT() sets:

        IN()    = (union of all predecessors' OUT())
                  intersected with LIVE_IN()

        OUT()   = union of (IN() - KILL()) and GEN()

   we converge when the OUT() sets stop growing. */

static blocks_iter_ret compute0(struct block *b)
{
    struct regs tmp = REGS_INITIALIZER(tmp);
    struct cessor *pred;
    int n;

    for (n = 0; pred = block_get_predecessor_n(b, n); ++n)
        regs_union(&tmp, &pred->b->webs.out);

    regs_clear(&b->webs.in);
    regs_select_bases(&b->webs.in, &tmp, &b->live.in);
    regs_clear(&tmp);

    regs_union(&tmp, &b->webs.in);
    regs_eliminate_bases(&tmp, &b->kill.kill);
    regs_union(&tmp, &b->webs.gen);

    if (REGS_COUNT(&tmp) != REGS_COUNT(&b->webs.out)) {
        regs_clear(&b->webs.out);
        regs_move(&b->webs.out, &tmp);
        return BLOCKS_ITER_REITER;
    } else {
        regs_clear(&tmp);
        return BLOCKS_ITER_OK;
    }
}

/* compute which definitions reach every USE, and merge webs accordingly.
   (a web is the maximal intersection of its du-chains, so all DEFs which
   share a common USE must be in the same web.) there are two subtleties:

   1. if no definition from this function reaches a USE, we create a DEF
      from "somewhere outside this function", which has the index 0.
   2. when a register is both DEFd and USEd in the same appearance (e.g.,
      two-address targets like AMD64), the USE will already have an index
      because it was assigned one as a DEF. this DEF is merged with other
      candidates, since obviously we can't split a register in two.

   we assign each USE an index from its web, so we can identify it easily
   in the rewrite pass, rather than having to repeat the computation. */

static blocks_iter_ret merge0(struct block *b)
{
    struct regs current = REGS_INITIALIZER(current);
    struct regs tmp = REGS_INITIALIZER(tmp);
    struct regs regs = REGS_INITIALIZER(regs);
    struct reg *regs_r;
    struct web *w;
    pseudo_reg base;
    struct insn *insn;
    
    regs_union(&current, &b->webs.in);

    INSNS_FOREACH(insn, &b->insns) {
        insn_uses_regs(insn, &regs, 0);

        REGS_FOREACH(regs_r, &regs) {
            if (target->reg_physical(regs_r->reg))
                continue;

            regs_select_base(&tmp, &current, regs_r->reg);
            base = PSEUDO_REG_BASE(regs_r->reg);

            if (REGS_EMPTY(&tmp)) {             /* no reaching DEFs */
                web_uninit(base);
                REGS_ADD(&tmp, base);
            }

            if (regs_r->reg != base)            /* already indexed */
                REGS_ADD(&tmp, regs_r->reg);
            
            w = webs_merge(&tmp);

            insn_substitute_reg(insn, regs_r->reg, WEB_FIRST(w),
                                INSN_SUBSTITUTE_USES);

            regs_clear(&tmp);
        }

        regs_clear(&regs);

        insn_defs_regs(insn, &regs, 0);

        REGS_FOREACH(regs_r, &regs)
            if (!target->reg_physical(regs_r->reg))
                regs_replace_base(&current, regs_r->reg);

        regs_clear(&regs);
    }

    return BLOCKS_ITER_OK;
}

/* rewrite each subscripted variable so they all refer to
   their common name (the first subscript of their web). */

static blocks_iter_ret rewrite0(struct block *b)
{
    struct regs regs = REGS_INITIALIZER(regs);
    struct reg *regs_r;
    struct insn *insn;
    struct web *w;

    INSNS_FOREACH(insn, &b->insns) {
        insn_defs_regs(insn, &regs, 0);
        insn_uses_regs(insn, &regs, 0);

        REGS_FOREACH(regs_r, &regs) {
            if (target->reg_physical(regs_r->reg))
                continue;

            w = web_find(regs_r->reg);

            insn_substitute_reg(insn, regs_r->reg, WEB_FIRST(w),
                                INSN_SUBSTITUTE_ALL);
        }

        regs_clear(&regs);
    }

    return BLOCKS_ITER_OK;
}

/* strip the indices from all registers in the IR. this should be called to
   clean up after using webs, since the IR is expected to be undecorated. */

static blocks_iter_ret strip0(struct block *b)
{
    struct insn *insn;

    INSNS_FOREACH(insn, &b->insns)
        insn_strip_indices(insn);

    return BLOCKS_ITER_OK;
}

void webs_strip(void)
{
    blocks_iter(strip0);
}

void webs_analyze(void)
{
    kill_analyze();
    live_analyze();
    reset();
    blocks_iter(def0);
    blocks_iter(compute0);
    blocks_iter(merge0);
    blocks_iter(rewrite0);
    blocks_iter(clear0);
    webs_clear();
}

/* vi: set ts=4 expandtab: */
