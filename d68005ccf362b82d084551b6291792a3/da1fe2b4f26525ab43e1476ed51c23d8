/* sched.c - instruction scheduling                     ncc, the new c compiler

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
#include "insn.h"
#include "block.h"
#include "output.h"
#include "sched.h"

/* a struct deps is attached to every insn, holding a set of pointers to
   other insns (in the same block) which must be scheduled before this
   insn. viewed collectively, the struct deps from all insns in a block
   form the conventional dependency DAG. */

SET_DEFINE_FREELIST(dep)
SET_DEFINE_ALLOC(dep, 1000)
SET_DEFINE_LOOKUP(dep, struct insn *, insn)
SET_DEFINE_CLEAR(dep)

#define DEPS_INIT(ds)           SET_INIT(ds)
#define DEPS_ADD(ds, i)         deps_lookup((ds), (i), SET_LOOKUP_CREATE)
#define DEPS_CONTAINS(ds, i)    (deps_lookup((ds), (i), 0) != 0)

/* initialize/empty the scheduling data
   when an insn is allocated or freed */

void sched_init(struct sched *sched)
{
    DEPS_INIT(&sched->deps);
}

void sched_clear(struct sched *sched)
{
    deps_clear(&sched->deps);
}

/* recompute the dependencies in block b. conventional rules: for two
   instructions I and Ip, where Ip precedes I in the basic block, Ip
   must remain before I (thus making I "dependent" on Ip) if:

        1. Ip DEFs any registers USEd by I, OR
        2. I DEFs any registers USEd by Ip, OR
        3. I and Ip DEF any of the same registers.

   for our purposes, the condition codes and memory are treated as registers
   (PSEUDO_REG_CC and PSEUDO_REG_MEM, respectively). we also have the added
   rule that volatile insns are dependent on all previous volatile insns,
   thus preserving their ordering. */

static blocks_iter_ret analyze0(struct block *b)
{
    struct insn *insn;
    struct insn *prev;
    struct regs uses = REGS_INITIALIZER(uses);
    struct regs defs = REGS_INITIALIZER(defs);
    struct regs prev_uses = REGS_INITIALIZER(prev_uses);
    struct regs prev_defs = REGS_INITIALIZER(prev_defs);

    INSNS_FOREACH(insn, &b->insns) {
        sched_clear(&insn->sched);

        insn_uses_regs(insn, &uses, INSN_DEFSUSES_CC | INSN_DEFSUSES_MEM);
        insn_defs_regs(insn, &defs, INSN_DEFSUSES_CC | INSN_DEFSUSES_MEM);

        for (prev = INSNS_PREV(insn); prev; prev = INSNS_PREV(prev)) {
            insn_uses_regs(prev, &prev_uses, INSN_DEFSUSES_CC
                                           | INSN_DEFSUSES_MEM);
            insn_defs_regs(prev, &prev_defs, INSN_DEFSUSES_CC
                                           | INSN_DEFSUSES_MEM);

            if (regs_overlap(&uses, &prev_defs)
              || regs_overlap(&defs, &prev_uses)
              || regs_overlap(&defs, &prev_defs)
              || ((insn->flags & INSN_FLAG_VOLATILE)
              &&  (prev->flags & INSN_FLAG_VOLATILE)))
                DEPS_ADD(&insn->sched.deps, prev);

            regs_clear(&prev_uses);
            regs_clear(&prev_defs);
        }

        regs_clear(&uses);
        regs_clear(&defs);
    }

    return BLOCKS_ITER_OK;
}

void sched_analyze(void)
{
    blocks_iter(analyze0);
}

/* this delay optimization is about as old as C itself. we attempt to push
   I_ADD and I_SUB instructions past I_LOADs and I_STOREs when possible. the
   intent is delay post-increment/decrement operations, so the access can be
   done with pointer in its original register, rather than copying it to a
   temporary, incrementing/decrementing, and then accessing via the temporary.

   we also push past a following I_CAST because of shortcomings in the AMD64
   sign-/zero-extension peephole optimizations. we will remove this special
   case once better (target-independent) sign-/zero-extension is in place.

   this optimization doesn't really need full scheduling data, and in fact,
   the data is misleading when run on the target-independent IR. it would be
   faster, but messier, to inspect the involved instructions directly. */

static blocks_iter_ret delay0(struct block *b)
{
    struct insn *insn;
    struct insn *next;

again:
    INSNS_FOREACH(insn, &b->insns) {
        if ((next = INSNS_NEXT(insn)) == 0)
            break;

        if ((insn->op != I_ADD) && (insn->op != I_SUB))
            continue;

        if ((next->op != I_LOAD) && (next->op != I_STORE)
          && (next->op != I_CAST))
            continue;

        if (DEPS_CONTAINS(&next->sched.deps, insn))
            continue;

        insns_swap(&b->insns, insn);
        goto again;
    }

    return BLOCKS_ITER_OK;
}

void sched_delay(void)
{
    sched_analyze();
    blocks_iter(delay0);
}

/* vi: set ts=4 expandtab: */
