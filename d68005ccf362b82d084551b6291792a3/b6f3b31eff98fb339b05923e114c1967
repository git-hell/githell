/* fold.c - constant folding                            ncc, the new c compiler

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
#include "con.h"
#include "block.h"
#include "fold.h"
#include "prop.h"
#include "opt.h"

/* the front end eliminates most constant expressions from its trees, so
   this pass mostly folds those which are exposed from other optimizations
   (e.g., constant propagation). we handle constant comparisons by carrying
   known condition_code state from the comparisons to their uses. */

static bool changed;

#define FOLD0_I(OP)                                                     \
    CON_FOLD_BINARY_I(insn->dst->ts, &result,                           \
                        &insn->src1->con, &insn->src2->con, OP);

#define FOLD0_ANY(OP)                                                   \
    CON_FOLD_BINARY_ANY(insn->dst->ts, &result,                         \
                        &insn->src1->con, &insn->src2->con, OP);

static blocks_iter_ret fold0(struct block *b)
{
    struct insn *insn;
    union con result;
    struct symbol *sym;
    struct operand *opr;
    ccset ccs;
    bool ccs_valid = FALSE;

    INSNS_FOREACH(insn, &b->insns) {
        insn_normalize(insn);
        sym = 0;

        if (OPERAND_PURE_CON(insn->src1)) {
            switch (insn->op)
            {
            case I_NEG:     CON_FOLD_UNARY(insn->dst->ts, &result,
                                           &insn->src1->con, -);
                            goto fold;

            case I_COM:     CON_FOLD_UNARY_I(insn->dst->ts, &result,
                                             &insn->src1->con, ~);
                            goto fold;

            case I_CAST:    con_cast(insn->dst->ts, &result,
                                     insn->src1->ts, &insn->src1->con);
                            goto fold;
            }
        }

        /* we CAN subtract an impure constant from another, provided
           they reference the same symbol. the symbol disappears, and
           two pure symbols remain, which are processed below. */

        if ((insn->op == I_SUB) && OPERAND_CON(insn->src1)
          && OPERAND_CON(insn->src2) && (insn->src1->sym == insn->src2->sym))
            insn->src1->sym = insn->src2->sym = 0;

        /* we can add a pure constant to (or subtract it from) an impure
           constant, as long as we remember to keep the symbol around. */

        if (OPERAND_PURE_CON(insn->src2)) {
            if (OPERAND_CON(insn->src1)) {
                switch (insn->op)
                {
                case I_ADD:     sym = insn->src1->sym;
                                FOLD0_ANY(+);
                                goto fold;
                            
                case I_SUB:     sym = insn->src1->sym;
                                FOLD0_ANY(-);
                                goto fold;
                }
            }

            if (OPERAND_PURE_CON(insn->src1)) {
                switch (insn->op)
                {
                case I_DIV:     if (!operand_is_zero(insn->src2)) {
                                    FOLD0_ANY(/);
                                    goto fold;
                                }
                                break;

                case I_MOD:     if (!operand_is_zero(insn->src2)) {
                                    FOLD0_I(%);
                                    goto fold;
                                }
                                break;
                    
                case I_MUL:     FOLD0_ANY(*); goto fold;
                case I_SHR:     FOLD0_I(>>); goto fold;
                case I_SHL:     FOLD0_I(<<); goto fold;
                case I_XOR:     FOLD0_I(^); goto fold;
                case I_OR:      FOLD0_I(|); goto fold;
                case I_AND:     FOLD0_I(&); goto fold;
        
                case I_CMP:     changed = TRUE;
                                ccs_valid = TRUE;
                                ccs = con_cmp(insn->src1->ts,
                                              &insn->src1->con,
                                              &insn->src2->con);
                                insn_replace(insn, I_NOP);
                                continue;
                }
            }
        }

        if (I_IS_SET_CC(insn->op) && ccs_valid)
        {
            if (CCSET_IS_SET(ccs, I_CC_FROM_SET_CC(insn->op)))
                result.i = 1;
            else
                result.i = 0;

            goto fold;
        }

        if (insn_defs_cc(insn))
            ccs_valid = FALSE;

        continue;

        fold:
            insn_replace(insn, I_MOVE,
                               operand_dup(insn->dst),
                               opr = operand_con(insn->dst->ts, result));
            opr->sym = sym;
            changed = TRUE;
    }

    if (ccs_valid)
        block_known_ccs(b, ccs);

    return BLOCKS_ITER_OK;
}

void fold(void)
{
    changed = FALSE;
    blocks_iter(fold0);

    if (changed) {
        nop();
        unreach();
        prop();
    }
}

/* vi: set ts=4 expandtab: */
