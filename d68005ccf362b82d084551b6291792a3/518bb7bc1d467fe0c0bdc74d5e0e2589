/* block.h - basic blocks                               ncc, the new c compiler

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

#ifndef BLOCK_H
#define BLOCK_H

#include "../common/tailq.h"
#include "insn.h"
#include "type.h"
#include "live.h"
#include "prop.h"
#include "regs.h"
#include "webs.h"
#include "kill.h"
#include "blks.h"
#include "copy.h"
#include "slvn.h"
#include "loop.h"
#include "codes.h"
#include "amd64/target_block.h"

struct symbol;

/* for simplicity we track both successors and predecessors with cessors.
   for successors of a given block, the condition_codes must be exclusive
   and exhaustive. for predecessors, the code is meaningless- use CC_NEVER.

   CC_SWITCH is the oddball here. if the successors are switch cases, then
   the first successor will be CC_DEFAULT, the rest will be CC_SWITCHes for
   the switch cases, and the block control field will be set.

   since switching is fundamentally a test of equality, signedness in the
   ranges has little relevance: we treat all case values as unsigned. */

struct cessor
{
    condition_code cc;
    struct block *b;

    unsigned long min;      /* minimum and maximum values ... */
    unsigned long max;      /* ... in a CC_SWITCH, inclusive */

    TAILQ_ENTRY(cessor) links;
};

TAILQ_HEAD(cessors, cessor);

#define CESSORS_INIT(cs)            TAILQ_INIT(cs)
#define CESSORS_REMOVE(cs, c)       TAILQ_REMOVE(cs, c, links)
#define CESSORS_APPEND(cs, c)       TAILQ_INSERT_TAIL(cs, c, links)
#define CESSORS_FIRST(cs)           TAILQ_FIRST(cs)
#define CESSORS_NEXT(cs)            TAILQ_NEXT(cs, links)
#define CESSORS_FOREACH(c, cs)      TAILQ_FOREACH(c, cs, links)

#define CESSORS_INSERT_AFTER(cs, aft, c)  TAILQ_INSERT_AFTER(cs, aft, c, links)

/* a struct block represents a sequence of insns which can only be entered
   at the beginning and only exited at the end, i.e., a basic block. */

typedef int block_flags;    /* BLOCK_FLAG_* */

#define BLOCK_FLAG_VISITED  ( 0x00000001 )  /* already visited this walk */
#define BLOCK_FLAG_LOOPED   ( 0x00000002 )  /* checked for loop invariants */

struct block
{
    asm_label label;
    block_flags flags;

    struct insns insns;

    struct live live;               /* live-variable data */
    struct kill kill;               /* kill/read data */
    struct prop prop;               /* constant propagation data */
    struct copy copy;               /* copy propagation data */
    struct lvns lvns;               /* value numbering data */
    struct insn *store;             /* redundant load data */
    struct operand *control;        /* controlling value of switch block */
    struct webs webs;               /* allocatable webs data */
    struct blks dominators;         /* dominators of this block */
    struct loop loop;               /* loop data */

    struct cessors successors;
    struct cessors predecessors;

    union
    {
        struct amd64_block amd64;
    };

    TAILQ_ENTRY(block) links;
};

TAILQ_HEAD(blocks, block);      /* struct blocks */

#define BLOCKS_INIT(blks)           TAILQ_INIT(blks)
#define BLOCKS_FIRST(blks)          TAILQ_FIRST(blks)
#define BLOCKS_PREPEND(blks, b)     TAILQ_INSERT_HEAD(blks, b, links)
#define BLOCKS_APPEND(blks, b)      TAILQ_INSERT_TAIL(blks, b, links)
#define BLOCKS_REMOVE(blks, b)      TAILQ_REMOVE(blks, b, links)
#define BLOCKS_NEXT(b)              TAILQ_NEXT(b, links)
#define BLOCKS_FOREACH(b, blks)     TAILQ_FOREACH(b, blks, links)

#define BLOCK_VISITED(b)            ((b)->flags & BLOCK_FLAG_VISITED)
#define BLOCK_MARK_VISITED(b)       ((b)->flags |= BLOCK_FLAG_VISITED)
#define BLOCK_MARK_UNVISITED(b)     ((b)->flags &= ~BLOCK_FLAG_VISITED)

/* if a block ends with a switch-based branch */

#define BLOCK_SWITCH(b)             ((b)->control != 0)

/* the CC_DEFAULT block out of a switch */

#define BLOCK_SWITCH_DEFAULT(b)     (CESSORS_FIRST(&(b)->successors)->b)

#define BLOCK_EMPTY(b)      (INSNS_EMPTY(&(b)->insns) && !BLOCK_SWITCH(b))

extern struct block *block_new(void);
extern void block_free(struct block *);
extern bool block_conditional(struct block *);
extern ccset block_ccs(struct block *);
extern struct block *block_preheader(struct block *, struct blks *);
extern struct block *block_split(struct block *, insn_index);
extern struct block *block_split_edge(struct block *, struct cessor *);

extern struct cessor *block_add_successor(struct block *, condition_code,
                                          struct block *);

extern void block_redirect_successors(struct block *, struct block *,
                                     struct block *);

extern struct cessor *block_get_successor_n(struct block *, int);
extern struct cessor *block_get_predecessor_n(struct block *, int);
extern struct cessor *block_cc_successor(struct block *, condition_code);
extern struct cessor *block_always_successor(struct block *);
extern struct cessor *block_sole_predecessor(struct block *);
extern void block_switch(struct block *, struct tree *, struct block *);
extern void block_switch_case(struct block *, long, struct block *);
extern struct block *block_switch_lookup(struct block *, long);
extern void block_switch_done(struct block *);
extern void block_remove_successor(struct block *, struct cessor *);
extern void block_remove_successors(struct block *);
extern void block_known_ccs(struct block *, ccset ccs);
extern bool block_rewrite_zs(struct block *, condition_code);
extern bool block_substitute_reg(struct block *, pseudo_reg, pseudo_reg);
extern bool blocks_substitute_reg(pseudo_reg, pseudo_reg);
extern void block_dup_successors(struct block *, struct block *);
extern int block_nr_successors(struct block *);
extern int block_nr_predecessors(struct block *);
extern bool block_defs_mem(struct block *);
extern bool blocks_def_mem(struct blks *);

#define BLOCK_APPEND_INSN(b, i)     insn_append(&((b)->insns), (i))
#define BLOCK_PREPEND_INSN(b, i)    insn_prepend(&((b)->insns), (i))

typedef int blocks_iter_ret;    /* BLOCKS_ITER_* */

#define BLOCKS_ITER_OK          0
#define BLOCKS_ITER_ABORT       1
#define BLOCKS_ITER_REITER      2

extern blocks_iter_ret blocks_iter(blocks_iter_ret (*f)(struct block *));

extern void blocks_walk(void (*)(struct block *), void (*)(struct block *));

extern void blocks_sequence(void);

extern struct symbol *func_sym;
extern struct type func_ret_type;
extern struct symbol *func_strun_ret;
extern struct regs func_regs;
extern struct block *entry_block;
extern struct block *exit_block;
extern struct block *current_block;

#define EMIT(insn)          BLOCK_APPEND_INSN(current_block, (insn))

extern void func_new(struct symbol *);
extern void func_complete(struct symbol *);
extern void func_free(void);

#endif /* BLOCK_H */

/* vi: set ts=4 expandtab: */
