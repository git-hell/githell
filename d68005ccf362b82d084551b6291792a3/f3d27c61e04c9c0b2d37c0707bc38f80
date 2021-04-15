/* switch.c - switch statement code generation          ncc, the new c compiler

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
#include "opt.h"
#include "switch.h"

/* find all the switch blocks, decide on a strategy for each one, and go.

   for the moment we only implement two strategies, a straightforward
   test-and-branch for small switches (that is, number of CC_SWITCHes
   < SWITCH_SMALL) and a binary search for larger ones. these do well
   for general use, though the binary search can be murder on pipelines.
   there's obviously plenty of room for improvement here: e.g, there is 
   no attempt at table-driven strategy, the best in many circumstances.

   this is called after the optimizer, which through constant propagation,
   pruning, successor coalescing, etc. has left us with a nice sorted set
   of CC_SWITCH entries attached to all the switch blocks. there's no point
   in calling the optimizer afterwards, since the code we generate would
   not be subject to much target-independent optimization, anyway. we do
   a quick prune afterwards to clean up our empties, though. */

#define SWITCH_SMALL 12

/* recursive binary-search strategy */

static void switch2(struct block *b, struct cessor *first, int count)
{
    struct cessor *cases;
    struct block *lower_b;
    struct block *higher_b;
    struct block *tmp_b;
    int lower;
    int higher;

    if (count == 0)
        block_add_successor(current_block, CC_ALWAYS, 
                                           BLOCK_SWITCH_DEFAULT(b));
    else {
        cases = first;
        lower = 0;
        higher = count - 1;

        while ((lower < higher) && CESSORS_NEXT(cases)) {
            cases = CESSORS_NEXT(cases);
            ++lower;
            --higher;
        }

        tmp_b = current_block;

        lower_b = block_new();
        current_block = lower_b;
        switch2(b, first, lower);

        higher_b = block_new();
        current_block = higher_b;
        switch2(b, CESSORS_NEXT(cases), higher);

        current_block = tmp_b;

        EMIT(insn_new(I_CMP, operand_dup(b->control),
                             operand_i(b->control->ts, cases->min, 0)));

        if (cases->min == cases->max) {
            block_add_successor(current_block, CC_Z, cases->b);

            if (lower || higher) {
                block_add_successor(current_block, CC_B, lower_b);
                block_add_successor(current_block, CC_A, higher_b);
            } else
                block_add_successor(current_block, CC_NZ,
                                                   BLOCK_SWITCH_DEFAULT(b));
        } else {
            tmp_b = block_new();
            block_add_successor(current_block, CC_AE, tmp_b);
            block_add_successor(current_block, CC_B, lower_b);

            current_block = tmp_b;

            EMIT(insn_new(I_CMP, operand_dup(b->control),
                                 operand_i(b->control->ts, cases->max, 0)));

            block_add_successor(current_block, CC_BE, cases->b);
            block_add_successor(current_block, CC_A, higher_b);
        }
    }
}

/* simple linear strategy */

static void switch1(struct block *b, struct cessor *cases, int count)
{
    struct block *no_b;
    struct block *maybe_b;

    while (count--) {
        no_b = block_new();

        EMIT(insn_new(I_CMP, operand_dup(b->control),
                             operand_i(b->control->ts, cases->min, 0)));

        if (cases->min == cases->max) {
            block_add_successor(current_block, CC_Z, cases->b);
            block_add_successor(current_block, CC_NZ, no_b);
        } else {
            maybe_b = block_new();
    
            block_add_successor(current_block, CC_B, no_b);
            block_add_successor(current_block, CC_AE, maybe_b);

            current_block = maybe_b;
            
            EMIT(insn_new(I_CMP, operand_dup(b->control),
                                 operand_i(b->control->ts, cases->max, 0)));

            block_add_successor(current_block, CC_BE, cases->b);
            block_add_successor(current_block, CC_A, no_b);
        }

        current_block = no_b;
        cases = CESSORS_NEXT(cases);
    }

    block_add_successor(current_block, CC_ALWAYS, BLOCK_SWITCH_DEFAULT(b));
}

static blocks_iter_ret switch0(struct block *b)
{
    struct block *new;
    struct cessor *cases;
    int count;

    if (!BLOCK_SWITCH(b))
        return BLOCKS_ITER_OK;

    cases = CESSORS_FIRST(&b->successors);
    cases = CESSORS_NEXT(cases);
    count = block_nr_successors(b) - 1;
    new = block_new();
    current_block = new;

    if (count < SWITCH_SMALL)
        switch1(b, cases, count);
    else
        switch2(b, cases, count);
    
    block_remove_successors(b);
    block_add_successor(b, CC_ALWAYS, new);

    return BLOCKS_ITER_ABORT;
}

void switch_gen(void)
{
    while (blocks_iter(switch0) == BLOCKS_ITER_ABORT)
        ;

    prune();
}

/* vi: set ts=4 expandtab: */
