/* dealias.c - rewrite aliased/volatile variables       ncc, the new c compiler

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
#include "tree.h"
#include "symbol.h"
#include "gen.h"
#include "dealias.h"

/* if this insn is a function argument, return the first
   argument insn in the sequence. we use this to put any
   loads before the sequence, lest we break the rule that
   arguments and their call insns be contiguous. */

static struct insn *first_arg(struct insn *insn)
{
    struct insn *prev;

    if (I_ARGUMENT(insn->op))
        while ((prev = INSNS_PREV(insn)) && I_ARGUMENT(prev->op))
            insn = prev;

    return insn;
}

/* this pass is invoked immediately after the IR is completed, before any
   optimization is attempted, to force load-before-use and store-after-def
   for aliased variables. in the process, the pseudo-registers associated
   with such variables disappear from the IR, replaced with temps. thus 
   subsequent passes and the optimizer will not see any aliased variables.
   
   note this is very similar to, but subtly different from, how we treat
   volatiles. we rewrite volatiles in-tree before generation even happens.
   (at that point we don't even know what's aliased.) the same benefits
   of split live ranges and volatiles disappearing from the optimizer's
   view still hold. but rewriting the trees creates additional loads and
   stores that are not necessary for aliased variables, e.g.,

         volatile int a;  int b = a + a;
         auto int c; int d = c + c;
  
   generates (correctly) TWO volatile loads for (a + a), whereas this pass
   could legally only generate one for (c + c).

   there's much room for improvement here: we currently load immediately
   before use and store immediately after def, but this often results in
   multiple loads of a value that hasn't changed, or dead stores to memory
   when a variable is updated multiple times in succession. some of this
   is hidden by the optimizer (e.g., load.c) but not all of it, and it
   really shouldn't have to. */

typedef int dealias_flags;  /* DEALIAS_* */

#define DEALIAS_USE     ( 0x00000001 )
#define DEALIAS_DEF     ( 0x00000002 )

static void dealias1(struct block *b, struct insn *insn,
                     struct symbol *sym, dealias_flags flags)
{
    struct tree *addr;
    struct tree *tree;
    struct symbol *temp;

    addr = tree_sym(sym);
    addr = tree_addrof(addr);
    addr = gen(addr, GEN_FLAG_ROOT);

    temp = symbol_temp(&sym->type);
    tree = tree_sym(temp);

    insn_substitute_reg(insn, symbol_reg(sym), symbol_reg(temp),
                        INSN_SUBSTITUTE_ALL);

    insn = first_arg(insn);

    if (flags & DEALIAS_USE) {
        EMIT(insn_new(I_LOAD, operand_sym(temp), operand_leaf(addr)));
        insns_insert_before(&b->insns, insn, &current_block->insns);
    }

    if (flags & DEALIAS_DEF) {
        EMIT(insn_new(I_STORE, operand_leaf(addr), operand_sym(temp)));
        insns_insert_after(&b->insns, insn, &current_block->insns);
    }

    tree_free(addr);
    tree_free(tree);
}

static blocks_iter_ret dealias0(struct block *b)
{
    struct regs regs = REGS_INITIALIZER(regs);
    struct regs defs = REGS_INITIALIZER(defs);
    struct regs uses = REGS_INITIALIZER(uses);
    struct reg *regs_r;
    struct insn *insn;
    struct insn *next;
    struct symbol *sym;
    dealias_flags flags;

    for (insn = INSNS_FIRST(&b->insns); insn; insn = next) {
        next = INSNS_NEXT(insn);

        insn_uses_regs(insn, &uses, 0);
        insn_defs_regs(insn, &defs, 0);
        regs_union(&regs, &uses);
        regs_union(&regs, &defs);

        REGS_FOREACH(regs_r, &regs) {
            sym = reg_symbol(regs_r->reg);

            if (sym && SYMBOL_ALIASED(sym)) {
                flags = 0;
            
                if (REGS_CONTAINS(&uses, regs_r->reg))
                    flags |= DEALIAS_USE;
        
                if (REGS_CONTAINS(&defs, regs_r->reg))
                    flags |= DEALIAS_DEF;
                
                dealias1(b, insn, sym, flags);
            }
        }

        regs_clear(&regs);
        regs_clear(&defs);
        regs_clear(&uses);
    }

    return BLOCKS_ITER_OK;
}

void dealias(void)
{
    current_block = block_new();
    blocks_iter(dealias0);
    block_free(current_block);
}

/* vi: set ts=4 expandtab: */
