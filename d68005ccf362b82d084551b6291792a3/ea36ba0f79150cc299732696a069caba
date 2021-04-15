/* block.c - basic blocks                               ncc, the new c compiler

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

#include <stdio.h>
#include <stdlib.h>
#include <stdarg.h>
#include "cc1.h"
#include "symbol.h"
#include "target.h"
#include "output.h"
#include "tree.h"
#include "insn.h"
#include "live.h"
#include "regs.h"
#include "blks.h"
#include "block.h"

/* a function is built as a collection of basic blocks arranged
   in a control flow graph. these can be globals since we have no
   intention of implementing nested functions or C99-style inline. */

struct symbol *func_sym;        /* symbol of the current function */
struct type func_ret_type;      /* and its return type */
struct symbol *func_strun_ret;  /* hidden struct-return pointer argument */
struct regs func_regs;          /* machine registers used by function */

struct block *entry_block;      /* entry block of current function */
struct block *exit_block;       /* and its exit block */
struct block *current_block;    /* current block; default EMIT() target */

static struct blocks blocks;    /* all currently allocated blocks */

/* generate a new cessor, append
   it to cessors and return it */

static struct cessor *cessor_new(struct cessors *cessors, condition_code cc,
                                 struct block *b)
{
    struct cessor *cessor;

    cessor = safe_malloc(sizeof(struct cessor));
    cessor->cc = cc;
    cessor->b = b;
    CESSORS_APPEND(cessors, cessor);

    return cessor;
}

/* remove a cessor from the list and free it */

static void cessor_free(struct cessors *cessors, struct cessor *cessor)
{
    CESSORS_REMOVE(cessors, cessor);
    free(cessor);
}

/* generate a new basic block, initialize it. */

struct block *block_new(void)
{
    struct block *b;

    b = safe_malloc(sizeof(struct block));
    b->label = ASM_LABEL_NEW();

    INSNS_INIT(&b->insns);
    live_init(&b->live);
    kill_init(&b->kill);
    loop_init(&b->loop);
    BLKS_INIT(&b->dominators);
    CESSORS_INIT(&b->successors);
    CESSORS_INIT(&b->predecessors);
    BLOCKS_APPEND(&blocks, b);

    return b;
}

/* free a block and all its resources */

void block_free(struct block *b)
{
    struct cessor *cessor;

    insns_clear(&b->insns);
    live_clear(&b->live);
    kill_clear(&b->kill);
    blks_clear(&b->dominators);
    loop_clear(&b->loop);
    operand_free(b->control);

    while (cessor = block_get_successor_n(b, 0))
        block_remove_successor(b, cessor);

    BLOCKS_REMOVE(&blocks, b);
    free(b);
}

/* add a new successor to a block and return its cessor. this
   implies the addition of a predecessor to the target block. */

struct cessor *block_add_successor(struct block *b, condition_code cc,
                                   struct block *successor)
{
    struct cessor *cessor;

    cessor = cessor_new(&b->successors, cc, successor);
    cessor_new(&successor->predecessors, CC_NEVER, b);

    return cessor;
}

/* find the (any) cessor entry in block b that points to pred. this
   is only used internally, so it assumes such a predecessor exists. */

static struct cessor *block_find_predecessor(struct block *b,
                                             struct block *pred)
{
    struct cessor *cessor;

    for (cessor = CESSORS_FIRST(&b->predecessors);
      cessor->b != pred; cessor = CESSORS_NEXT(cessor))
        ;

    return cessor;
}

/* remove a successor from a block. again, this implies
   the removal of the predecessor in the target block. */

void block_remove_successor(struct block *b, struct cessor *cessor)
{
    struct block *succ;

    succ = cessor->b;
    cessor_free(&b->successors, cessor);

    cessor = block_find_predecessor(succ, b);
    cessor_free(&succ->predecessors, cessor);
}

/* remove all successors from a block */

void block_remove_successors(struct block *b)
{
    struct cessor *succ;

    while (succ = CESSORS_FIRST(&b->successors))
        block_remove_successor(b, succ);

    operand_free(b->control);
    b->control = 0;
}

/* sort the CC_SWITCH entries of a block in ascending order- this is an
   off-the-cuff insertiony sort, which could definitely be improved. */

static void block_switch_sort(struct block *b)
{
    struct cessor *cessor;
    struct cessor *next;
    bool moved;

again:
    for (cessor = CESSORS_FIRST(&b->successors);
         cessor; cessor = CESSORS_NEXT(cessor)) {

        moved = FALSE;

        if (cessor->cc == CC_SWITCH) {
            while ((next = CESSORS_NEXT(cessor))
              && (next->cc == CC_SWITCH)
              && (next->min < cessor->min)) {
                CESSORS_REMOVE(&b->successors, cessor);
                CESSORS_INSERT_AFTER(&b->successors, next, cessor);
                moved = TRUE;
            }
        
            if (moved)
                goto again;
        }
    }
}

/* coalesce adjacent CC_SWITCH entries when possible. if
   two ranges are adjacent and have the same successor,
   then we collapse them into a single range. */

static void block_switch_coalesce(struct block *b)
{
    struct cessor *cessor;
    struct cessor *next;

    block_switch_sort(b);

    CESSORS_FOREACH(cessor, &b->successors)
        if (cessor->cc == CC_SWITCH) {
            next = CESSORS_NEXT(cessor);

            if (next && (next->cc == CC_SWITCH) && (next->b == cessor->b)
              && (next->min == (cessor->max + 1))) {
                cessor->max = next->max;
                block_remove_successor(b, next);
            }
        }
}

/* coalesce successors (cessor entries) of a block when possible.
   if the block has multiple cessor entries but they all go to the
   same place, then really the block has one CC_ALWAYS successor. */

static void block_coalesce_successors(struct block *b)
{
    struct cessor *cessor;
    struct block *succ_b;

    if (BLOCK_SWITCH(b))
        block_switch_coalesce(b);

    cessor = CESSORS_FIRST(&b->successors);
    
    if ((cessor == 0) || (cessor->cc == CC_ALWAYS))
        return;

    succ_b = cessor->b;
    
    while (cessor = CESSORS_NEXT(cessor)) {
        if (cessor->b != succ_b)
            return;
    }

    block_remove_successors(b);
    block_add_successor(b, CC_ALWAYS, succ_b);

    operand_free(b->control);       /* if it was a switch block ... */
    b->control = 0;                 /* ... it isn't any more */
}

/* the caller knows the state of the condition codes at the end
   of the block, so rewrite the successors with that knowledge */

void block_known_ccs(struct block *b, ccset ccs)
{
    struct cessor *succ;
    struct cessor *next;

    if (block_conditional(b)) {
        for (succ = CESSORS_FIRST(&b->successors); succ; succ = next) {
            next = CESSORS_NEXT(succ);

            if (CCSET_IS_SET(ccs, succ->cc))
                succ->cc = CC_ALWAYS;
            else
                block_remove_successor(b, succ);
        }
    }
}

/* redirect successors to a different block. for each successor in b
   that targets from, rewrite it to target to. we of course need to
   move the corresponding predecessor cessors while we're at it. */

void block_redirect_successors(struct block *b, struct block *from,
                               struct block *to)
{
    struct cessor *succ;
    struct cessor *pred;

    for (succ = CESSORS_FIRST(&b->successors);
      succ; succ = CESSORS_NEXT(succ))
    {
        if (succ->b == from) {
            pred = block_find_predecessor(from, b);
            CESSORS_REMOVE(&from->predecessors, pred);
            CESSORS_APPEND(&to->predecessors, pred);
            succ->b = to;
        }
    }

    block_coalesce_successors(b);
}

/* insert a preheader in before the loop head. we redirect
   all blocks, except those in loop_blks, to the new block,
   and return said new block. */

struct block *block_preheader(struct block *head, struct blks *loop_blks)
{
    struct block *new;
    struct block *b;

    new = block_new();
    block_add_successor(new, CC_ALWAYS, head);

    BLOCKS_FOREACH(b, &blocks) {
        if ((b == new) || BLKS_CONTAINS(loop_blks, b))
            continue;

        block_redirect_successors(b, head, new);
    }

    return new;
}

/* returns TRUE if the block ends in a conditional branch. */

bool block_conditional(struct block *b)
{
    struct cessor *succ;

    succ = block_get_successor_n(b, 0);

    if (succ && CC_CONDITIONAL(succ->cc))
        return TRUE;
    else
        return FALSE;
}

/* returns the set of condition codes used by the block's
   ending conditional branch (empty if it's not conditional) */

ccset block_ccs(struct block *b)
{
    struct cessor *succ;
    ccset ccs;

    CCSET_CLEAR(ccs);

    CESSORS_FOREACH(succ, &b->successors)
        if (CC_CONDITIONAL(succ->cc))
            CCSET_SET(ccs, succ->cc);

    return ccs;
}

/* mark a block as the controlling block of a switch statement. set
   the block's control operand based on the tree given (which must
   be a leaf), and create the CC_DEFAULT successor. this function
   must be called before calling block_switch_case() on a block. */

void block_switch(struct block *b, struct tree *control,
                  struct block *default_block)
{
    block_add_successor(b, CC_DEFAULT, default_block);
    b->control = operand_leaf(control);
}

/* add a switch case to a switch block. at this point (which is
   during parsing) each case label gets its own CC_SWITCH entry.
   we can't do a complete job of collapsing adjacent labels into
   ranges until the optimizer runs, so we don't even try. */

void block_switch_case(struct block *b, long i, struct block *succ)
{
    struct cessor *cessor;

    cessor = CESSORS_FIRST(&b->successors);

    while (cessor = CESSORS_NEXT(cessor))
        if (cessor->min == i)
            error(FATAL, "duplicate case label");

    cessor = block_add_successor(b, CC_SWITCH, succ);
    cessor->min = cessor->max = i;
}

/* called by the parser after completing a switch statement. */

void block_switch_done(struct block *b)
{
    block_coalesce_successors(b);
}

/* return the block that a switch() statement
   switches to for the specified constant. */

struct block *block_switch_lookup(struct block *b, long i)
{
    struct cessor *cessor;
    struct block *def_b;
    unsigned long u;
    
    u = i;

    cessor = CESSORS_FIRST(&b->successors);
    def_b = cessor->b; /* CC_DEFAULT */

    while (cessor = CESSORS_NEXT(cessor))
        if ((u >= cessor->min) && (u <= cessor->max))
            return cessor->b;

    return def_b;
}

/* return the nth successor/predecessor of a block,
   or 0 if not present. indexes start at 0. */

static struct cessor *get_cessor_n(struct cessors *cessors, int n)
{
    struct cessor *cessor;
        
    for (cessor = CESSORS_FIRST(cessors);
      n && cessor; --n, cessor = CESSORS_NEXT(cessor))
        ;

    return cessor;
}

struct cessor *block_get_successor_n(struct block *b, int n)
{
    return get_cessor_n(&b->successors, n);
}

struct cessor *block_get_predecessor_n(struct block *b, int n)
{
    return get_cessor_n(&b->predecessors, n);
}

/* return the number of successors or predecessors */

static int nr_cessors(struct cessors *cessors)
{
    struct cessor *cessor;
    int i;

    for (i = 0, cessor = CESSORS_FIRST(cessors); cessor;
      ++i, cessor = CESSORS_NEXT(cessor))
        ;

    return i;
}

int block_nr_successors(struct block *b)
{
    return nr_cessors(&b->successors);
}

int block_nr_predecessors(struct block *b)
{
    return nr_cessors(&b->predecessors);
}

/* return the successor of this block linked
   to it by condition_code, or 0 if none. */

struct cessor *block_cc_successor(struct block *b, condition_code cc)
{
    struct cessor *succ;

    for (succ = CESSORS_FIRST(&b->successors);
      succ; succ = CESSORS_NEXT(succ))
        if (succ->cc == cc)
            return succ;

    return 0;
}

/* return the CC_ALWAYS successor of a block, or 0
   if the block has no unconditional successor. */

struct cessor *block_always_successor(struct block *b)
{
    struct cessor *succ;

    succ = CESSORS_FIRST(&b->successors);

    if (succ && (succ->cc == CC_ALWAYS))
        return succ;
    else
        return 0;
}

/* return the sole predecessor of a block, if it
   has but one predecessor, 0 otherwise. */

struct cessor *block_sole_predecessor(struct block *b)
{
    struct cessor *pred;

    pred = CESSORS_FIRST(&b->predecessors);

    if (pred && (CESSORS_NEXT(pred) == 0))
        return pred;
    else
        return 0;
}

/* replace Z/NZ conditional branches with the pair indicated
   by nz (its partner implied by its inverse). returns TRUE
   if the rewrite occurred, or FALSE otherwise. */

bool block_rewrite_zs(struct block *b, condition_code nz)
{
    struct cessor *succ;
    bool result;

    result = FALSE;

    CESSORS_FOREACH(succ, &b->successors) {
        switch (succ->cc)
        {
        case CC_Z:  succ->cc = CC_INVERT(nz); break;
        case CC_NZ: succ->cc = nz; break;
        default:    goto out;
        }

        result = TRUE;
    }

out:
    return result;
}

/* remove all of dst's successors, and replace
   them with copies of the successors of src */

void block_dup_successors(struct block *dst, struct block *src)
{
    struct cessor *succ;

    block_remove_successors(dst);

    CESSORS_FOREACH(succ, &src->successors)
        block_add_successor(dst, succ->cc, succ->b);
    
    if (src->control)
        dst->control = operand_dup(src->control);
}

/* replace all appearances in a block of the reg from with reg to.
   returns TRUE if any replacements were made, FALSE otherwise. */

bool block_substitute_reg(struct block *b, pseudo_reg from, pseudo_reg to)
{
    struct insn *insn;
    bool ret = FALSE;
    
    INSNS_FOREACH(insn, &b->insns)
        if (insn_substitute_reg(insn, from, to, INSN_SUBSTITUTE_ALL))
            ret = TRUE;

    return ret;
}

/* replace all appearances everywhere of reg from with reg to. */

bool blocks_substitute_reg(pseudo_reg from, pseudo_reg to)
{
    struct block *b;
    bool ret = FALSE;

    BLOCKS_FOREACH(b, &blocks)
        if (block_substitute_reg(b, from, to))
            ret = TRUE;

    return ret;
}

/* split a block into two blocks; a second block is created as an always
   successor of the current block, and the current block's successors are
   moved to the second block. where is the index of the first insn that
   will be moved to the new block. returns the new block. */

struct block *block_split(struct block *b, insn_index where)
{
    struct block *new;

    new = block_new();
    block_dup_successors(new, b);
    block_remove_successors(b);
    block_add_successor(b, CC_ALWAYS, new);
    insns_push(&new->insns, &b->insns, b->insns.next_index - where);

    return new;
}

/* create a new empty block along the edge between
   b and succ. returns the new block. */

struct block *block_split_edge(struct block *b, struct cessor *succ)
{
    struct block *new;
    struct block *old;
    condition_code cc;

    cc = succ->cc;
    old = succ->b;

    new = block_new();
    block_add_successor(new, CC_ALWAYS, old);
    block_remove_successor(b, succ);
    block_add_successor(b, cc, new);

    return new;
}

/* returns TRUE if any insn in the block writes memory */

bool block_defs_mem(struct block *b)
{
    struct insn *insn;

    INSNS_FOREACH(insn, &b->insns)
        if (insn_defs_mem(insn))
            return TRUE;

    return FALSE;
}

/* returns TRUE if any insn in any of blks writes memory */

bool blocks_def_mem(struct blks *blks)
{
    struct blk *b;

    BLKS_FOREACH(b, blks)
        if (block_defs_mem(b->b))
            return TRUE;

    return FALSE;
}

/* walk the blocks depth-first, starting at the entry node, invoking
   pre and post at the pre- and post-order points, respectively. */

static blocks_iter_ret walk0(struct block *b)
{
    BLOCK_MARK_UNVISITED(b);
    return BLOCKS_ITER_OK;
}

static void walk1(struct block *b, void (*pre)(struct block *),
                                   void (*post)(struct block *))
{
    struct cessor *succ;
    int n;

    if (!BLOCK_VISITED(b)) {
        BLOCK_MARK_VISITED(b);

        if (pre) pre(b);

        for (n = 0; succ = block_get_successor_n(b, n); ++n)
            walk1(succ->b, pre, post);

        if (post) post(b);
    }
}

void blocks_walk(void (*pre)(struct block *),
                 void (*post)(struct block *))
{
    blocks_iter(walk0);
    walk1(entry_block, pre, post);
}

/* iterate over all blocks (in the order they appear in the
   all-blocks list), invoking f on each block. the callback
   returns one of

    BLOCKS_ITER_OK:     nothing interesting to report

    BLOCKS_ITER_ABORT:  stop iterating immediately and
                        return BLOCKS_ITER_ABORT to caller

    BLOCKS_ITER_REITER: continue this iteration, but start
                        another iteration once complete

   returns BLOCKS_ITER_OK unless otherwise noted above. */

blocks_iter_ret blocks_iter(blocks_iter_ret (*f)(struct block *))
{
    struct block *b;
    struct block *next;
    bool reiter;

    do {
        reiter = FALSE;

        for (b = BLOCKS_FIRST(&blocks); b; b = next) {
            next = BLOCKS_NEXT(b);

            switch (f(b)) {
            case BLOCKS_ITER_ABORT:     return BLOCKS_ITER_ABORT;
            case BLOCKS_ITER_REITER:    reiter = TRUE;
            }
        }
    } while (reiter);

    return BLOCKS_ITER_OK;
}

/* sequence the blocks in the all-blocks
   list in reverse post-order */

static void sequence0(struct block *b)
{
    BLOCKS_REMOVE(&blocks, b);
    BLOCKS_PREPEND(&blocks, b);
}

void blocks_sequence(void)
{
    blocks_walk(0, sequence0);
}

/* beginning a new function definition for sym. initialize
   state accordingly. if the function returns a struct, we
   create the hidden pointer-to-struct argument here. */

void func_new(struct symbol *sym)
{
    struct type ret_ptr_type = TYPE_INITIALIZER(ret_ptr_type);

    func_sym = sym;
    TYPE_INIT(&func_ret_type);
    type_deref(&func_ret_type, &sym->type);

    BLOCKS_INIT(&blocks);
    entry_block = block_new();
    current_block = block_new();
    block_add_successor(entry_block, CC_ALWAYS, current_block);
    exit_block = block_new();

    target->func_new();

    if (TYPE_STRUN(&func_ret_type)) {
        type_ref(&ret_ptr_type, &func_ret_type);
        func_strun_ret = symbol_temp(&ret_ptr_type);
        func_strun_ret->ss |= S_ARG;
        target->formal_declare(func_strun_ret);
        type_clear(&ret_ptr_type);
    } else
        func_strun_ret = 0;

    REGS_INIT(&func_regs);
}

/* free all resources belonging
   to the current function */

void func_free(void)
{
    struct block *b;

    while (b = BLOCKS_FIRST(&blocks))
        block_free(b);

    type_clear(&func_ret_type);
    func_strun_ret = 0;
    regs_clear(&func_regs);
}

/* vi: set ts=4 expandtab: */
