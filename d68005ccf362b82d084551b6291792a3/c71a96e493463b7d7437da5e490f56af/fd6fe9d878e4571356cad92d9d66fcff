/* amd64.c - AMD64 support                              ncc, the new c compiler 

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

#include "../../common/util.h"
#include "../cc1.h"
#include "../type.h"
#include "../insn.h"
#include "../block.h"
#include "../symbol.h"
#include "../output.h"
#include "../opt.h"
#include "insn.h"
#include "reg.h"
#include "peep.h"
#include "fuse.h"
#include "amd64.h"

/* we implement an ABI that is based on, but different from, the SysV ABI.

   functions preserve RBP, RBX, R12-R15, XMM8-15. the remaining registers
   are volatile. the first 6 integer arguments are passed in RDI, RSI, RDX,
   RCX, R8, and R9, and the first 6 floating-point arguments are passed in
   XMM0-5. excess and struct/union arguments are passed on the stack. the
   compiler only preserves bits [63:0] of SSE registers across functions.

   scalar return values appear in RAX or XMM0 as appropriate. the front
   end machinery handles struct return values: the caller passes a hidden
   first argument which is a pointer to the struct temporary where the
   return value should go. this actually matches the SysV ABI, with the
   exception that our functions do not return that pointer in RAX.

   variadic functions expect *all* of their arguments on the stack. the
   complex song and dance required to implement va_arg() and friends in
   the presence of register-passed arguments hardly seems worth it.

   the stack is always aligned to an 8-byte boundary, and all arguments
   passed on the stack are individually aligned to 8 bytes. we employ a
   conventional activation record so the first argument is at 16(%rbp).

   integer/pointer types smaller than 8 bytes are kept denormalized: the
   high bits are not maintained and are assumed to hold junk.

   assembler labels are assumed to fall in either the first or last 2GB
   of the address space, a la gcc "kernel" memory model. we could add a
   compiler switch to restrict this to the first 2GB to save a few bytes. */

int amd64_nr_iargs;             /* number of integer arguments */
int amd64_nr_fargs;             /* number of floating-point arguments */
int amd64_arg_addr;             /* next argument's frame position */
int amd64_local_addr;           /* last local's frame position */

/* called by func_new() to set up initial state
   upon starting a new function definition. */

void amd64_func_new(void)
{
    amd64_arg_addr = 16;
    amd64_local_addr = 0;

    if (TYPE_VARIADIC(&func_sym->type)) {
        /* pretend all the registers are spoken for */
        amd64_nr_iargs = AMD64_NR_IARGS;
        amd64_nr_fargs = AMD64_NR_FARGS;
    } else {
        amd64_nr_iargs = 0;
        amd64_nr_fargs = 0;
    }
}

/* assign frame storage for a local variable */

void amd64_symbol_storage(struct symbol *sym)
{
    amd64_local_addr += type_sizeof(&sym->type, 0);
    amd64_local_addr = ROUND_UP(amd64_local_addr, AMD64_STACK_ALIGN);
    sym->offset = -(amd64_local_addr);
}

/* called for each formal argument, left-to-right, upon entering
   a function definition. determine if the argument is passed in
   a register or on the stack. */

void amd64_formal_declare(struct symbol *sym)
{
    if (TYPE_DISCRETE(&sym->type) && (amd64_nr_iargs < AMD64_NR_IARGS))
        sym->arg_reg = amd64_iargs[amd64_nr_iargs++];
    else if (TYPE_FLOATING(&sym->type) && (amd64_nr_fargs < AMD64_NR_FARGS)) {
        sym->arg_reg = AMD64_REG_XMM(amd64_nr_fargs);
        ++amd64_nr_fargs;
    } else {
        sym->offset = amd64_arg_addr;
        amd64_arg_addr += type_sizeof(&sym->type, 0);
        amd64_arg_addr = ROUND_UP(amd64_arg_addr, AMD64_STACK_ALIGN);
    }
}

/* assign a pseudo register of appropriate class to sym */

void amd64_symbol_reg(struct symbol *sym)
{
    static pseudo_reg next_ireg = AMD64_REG_PSEUDO;
    static pseudo_reg next_freg = AMD64_REG_PSEUDO;

    if (TYPE_DISCRETE(&sym->type)) {
        sym->reg = AMD64_IREG(next_ireg);
        ++next_ireg;
    } else {
        sym->reg = AMD64_FREG(next_freg);
        ++next_freg;
    }
}

/* produce function prolog and epilog. this occurs almost
   immediately before the function assembly is output, so
   the code will not be subject to optimizer manipulation,
   barring sequencing and fusing.
 
   there is plenty of room for improvement here, especially
   for leaf functions, those without local frame storage, etc.
   should also probably attempt to schedule/interleave FP and
   integer non-volatile register saves. todo on a rainy day.

   remember, amd64/gen.c already put RET in the exit block. */

void amd64_logues(void)
{
    struct reg *r;
    struct amd64_operand *o;
    int addr;

    regs_remove(&func_regs, AMD64_REG_RSP);
    regs_remove(&func_regs, AMD64_REG_RBP);
    amd64_regs_remove_trash(&func_regs);

    /* we can't push non-volatile XMM registers directly onto the
       stack, so we grow the frame and effectively spill them. */

    REGS_FOREACH(r, &func_regs)
        if (AMD64_IS_FREG(r->reg))
            amd64_local_addr += AMD64_XMM_BYTES;

    amd64_local_addr = ROUND_UP(amd64_local_addr, AMD64_STACK_ALIGN);

    /* a bog-standard frame; save the existing frame pointer,
       base the new frame pointer off the current stack, and
       then restore the caller's frame pointer on exit. as
       mentioned above, this is not always necessary and could
       be eliminated in some circumstances. we could even omit
       the frame pointer entirely since RSP indexing is a thing,
       but i'm not entirely convinced that's all that beneficial. */

    BLOCK_APPEND_INSN(entry_block,
                      insn_new(AMD64_I_PUSHQ,
                               amd64_operand_reg(T_LONG, AMD64_REG_RBP)));

    BLOCK_APPEND_INSN(entry_block,
                      insn_new(AMD64_I_MOVQ,
                               amd64_operand_reg(T_LONG, AMD64_REG_RSP),
                               amd64_operand_reg(T_LONG, AMD64_REG_RBP)));

    BLOCK_PREPEND_INSN(exit_block,
                       insn_new(AMD64_I_POPQ,
                                amd64_operand_reg(T_LONG, AMD64_REG_RBP)));

    /* if we have any locals, decrement RSP on entry to
       make room for them, and restore RSP on exit. */

    if (amd64_local_addr) {
        BLOCK_APPEND_INSN(entry_block,
                          insn_new(AMD64_I_SUBQ,
                               amd64_operand_con(T_LONG, amd64_local_addr, 0),
                               amd64_operand_reg(T_LONG, AMD64_REG_RSP)));

        BLOCK_PREPEND_INSN(exit_block,
                           insn_new(AMD64_I_MOVQ,
                                 amd64_operand_reg(T_LONG, AMD64_REG_RBP),
                                 amd64_operand_reg(T_LONG, AMD64_REG_RSP)));
    }

    /* save/restore the non-volatile registers on entry/exit. the
       integer regs are stacked as expected, and the XMM regs are
       are spilled into/out of the frame space allocated above. */

    addr = amd64_local_addr;

    REGS_FOREACH(r, &func_regs) {
        if (AMD64_IS_IREG(r->reg)) {
            BLOCK_APPEND_INSN(entry_block,
                              insn_new(AMD64_I_PUSHQ,
                                       amd64_operand_reg(T_LONG, r->reg)));

            BLOCK_PREPEND_INSN(exit_block,
                               insn_new(AMD64_I_POPQ,
                                        amd64_operand_reg(T_LONG, r->reg)));
        } else {
            o = amd64_operand_based(AMD64_REG_RBP, -addr);
            AMD64_OPERAND_INDIRECT(o);
            o->ts = T_DOUBLE;

            BLOCK_APPEND_INSN(entry_block,
                              insn_new(AMD64_I_MOVSD,
                                       amd64_operand_reg(T_DOUBLE, r->reg),
                                       amd64_operand_dup(o)));

            BLOCK_PREPEND_INSN(exit_block,
                               insn_new(AMD64_I_MOVSD,
                                        o,
                                        amd64_operand_reg(T_DOUBLE, r->reg)));

            addr -= AMD64_XMM_BYTES;
        }
    }
}

/* post-allocation optimizations */

void amd64_opt(void)
{
    amd64_peep_post();
    amd64_fuse();
    amd64_peep_final();
    nop();
}

/* output a branch to block b if cc is true */

void amd64_insn_branch(condition_code cc, struct block *b)
{
    static char *ccs[] =    /* keyed to CC_INDEX() */
    {
        "jz",       "jnz",      "jg",       "jle",
        "jge",      "jl",       "ja",       "jbe",
        "jae",      "jb",       "jmp"
    };

    output("\t%s %L\n", ccs[CC_INDEX(cc)], b->label);
}

/* vi: set ts=4 expandtab: */
