/* insn.c - instructions                                ncc, the new c compiler

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

#include <stdlib.h>
#include <stdarg.h>
#include <string.h>
#include "cc1.h"
#include "con.h"
#include "target.h"
#include "tree.h"
#include "symbol.h"
#include "sched.h"
#include "live.h"
#include "regs.h"
#include "insn.h"

#define INSNS_REMOVE(is, i)             TAILQ_REMOVE(is, i, links)
#define INSNS_PREPEND(is, i)            TAILQ_INSERT_HEAD(is, i, links)
#define INSNS_APPEND(is, i)             TAILQ_INSERT_TAIL(is, i, links)
#define INSNS_INSERT_AFTER(is, aft, i)  TAILQ_INSERT_AFTER(is, aft, i, links)
#define INSNS_INSERT_BEFORE(bef, i)     TAILQ_INSERT_BEFORE(bef, i, links)
#define INSNS_CONCAT(dst, src)          TAILQ_CONCAT(dst, src, links)

/* allocate a new operand. note that it is here that T_PTRS and
   T_LDOUBLEs decay permanently into their machine counterparts.
   any stray front-end type_bits are stripped out as well. */

static struct operand *operand_new(operand_class class, type_bits ts)
{
    struct operand *opr;

    opr = safe_malloc(sizeof(struct operand));
    opr->class = class;
    opr->ts = type_machine_bits(ts);
    opr->ts = T_BASE(opr->ts);
    opr->number = VALUE_NUMBER_NONE;

    return opr;
}

/* return a duplicate instance of the
   operand. safe to use on a null src. */

struct operand *operand_dup(struct operand *src)
{
    struct operand *dst = 0;

    if (src) {
        dst = safe_malloc(sizeof(struct operand));
        *dst = *src;
    }

    return dst;
}

/* convenience functions for generating operands */

struct operand *operand_reg(type_bits ts, pseudo_reg reg)
{
    struct operand *opr;

    opr = operand_new(O_REG, ts);
    opr->reg = reg;

    return opr;
}

struct operand *operand_con(type_bits ts, union con con)
{
    struct operand *opr;

    opr = operand_new(O_CON, ts);
    opr->con = con;

    return opr;
}

struct operand *operand_f(type_bits ts, double f)
{
    struct operand *opr;

    opr = operand_new(O_CON, ts);
    opr->con.f = f;

    return opr;
}

struct operand *operand_i(type_bits ts, long i, struct symbol *sym)
{
    struct operand *opr;

    opr = operand_new(O_CON, ts);
    opr->con.i = i;
    opr->sym = sym;

    return opr;
}

/* return an operand corresponding to
   constant zero (0) or one (1) */

#define OPERAND_CONSTANT(V)                                             \
    if (ts & T_FLOATING)                                                \
        return operand_f(ts, V);                                        \
    else                                                                \
        return operand_i(ts, V, 0);

struct operand *operand_zero(type_bits ts) { OPERAND_CONSTANT(0); }
struct operand *operand_one(type_bits ts) { OPERAND_CONSTANT(1); }

/* return an O_REG that corresponds to the symbol */

struct operand *operand_sym(struct symbol *sym)
{
    return operand_reg(TYPE_BITS(&sym->type), symbol_reg(sym));
}

/* return an operand congruent to the tree leaf */

struct operand *operand_leaf(struct tree *tree)
{
    type_bits ts;

    ts = TYPE_BITS(&tree->type);

    switch (tree->op)
    {
    case E_CON:         if (ts & T_FLOATING)
                            return operand_f(ts, tree->con.f);
                        else
                            return operand_i(ts, tree->con.i, tree->sym);

    case E_SYM:         return operand_reg(ts, symbol_reg(tree->sym));
    }
}

/* return TRUE if the operand represents
   a constant zero (0) or one (1) value */

#define OPERAND_IS(V)                                                       \
    if (OPERAND_PURE_CON(opr)) {                                            \
        if (opr->ts & T_FLOATING)                                           \
            return (opr->con.f == V);                                       \
        else                                                                \
            return (opr->con.i == V);                                       \
    } else                                                                  \
        return FALSE;

bool operand_is_zero(struct operand *opr) { OPERAND_IS(0); }
bool operand_is_one(struct operand *opr) { OPERAND_IS(1); }

/* returns TRUE if the operands are identical */

bool operand_is_same(struct operand *opr1, struct operand *opr2)
{
    if ((opr1 == 0) && (opr2 == 0))
        return TRUE;

    if ((opr1 == 0) || (opr2 == 0) || (opr1->ts != opr2->ts))
        return FALSE;

    if (OPERAND_REG(opr1) && OPERAND_REG(opr2) && (opr1->reg == opr2->reg))
        return TRUE;

    if (OPERAND_CON(opr1) && OPERAND_CON(opr2)) {
        if ((opr1->ts & T_INTEGRAL)
          && (opr1->sym == opr2->sym)
          && (opr1->con.i == opr2->con.i))
            return TRUE;
    
        if (opr1->ts & T_FLOATING)
            return (opr1->con.f == opr2->con.f);
    }

    return FALSE;
}

/* this seems pointless, but it's likely that in
   the future operands will carry more information. */

void operand_free(struct operand *opr)
{
    if (opr)
        free(opr);
}

/* given a (zeroed) data area, construct a properly initialized insn */

static void insn_construct(struct insn *insn, insn_op op, va_list args)
{
    insn->op = op;

    if (I_TARGET(op))
        target->insn_construct(insn, args);
    else {
        if (op & I_FLAG_DST) insn->dst = va_arg(args, struct operand *);
        if (op & I_FLAG_SRC1) insn->src1 = va_arg(args, struct operand *);
        if (op & I_FLAG_SRC2) insn->src2 = va_arg(args, struct operand *);
    }

    sched_init(&insn->sched);
}

/* release the resources associated with the insn */

static void insn_destruct(struct insn *insn)
{
    if (I_TARGET(insn->op))
        target->insn_destruct(insn);
    else {
        operand_free(insn->dst);
        operand_free(insn->src1);
        operand_free(insn->src2);
    }

    sched_clear(&insn->sched);
}

/* allocate a new insn and construct it with
   the given op and (optional) operands */

struct insn *insn_new(insn_op op, ...)
{
    struct insn *insn;
    va_list args;

    va_start(args, op);
    insn = safe_malloc(sizeof(struct insn));
    insn_construct(insn, op, args);
    va_end(args);

    return insn;
}

/* return an exact duplicate of insn */

struct insn *insn_dup(struct insn *src)
{
    struct insn *insn;

    if (I_TARGET(src->op))
        return target->insn_dup(src);
    else {
        insn = insn_new(I_NOP);
        insn->op = src->op;
        insn->dst = operand_dup(src->dst);
        insn->src1 = operand_dup(src->src1);
        insn->src2 = operand_dup(src->src2);
        
        return insn;
    }
}

/* replace an insn with a new one with
   the given op and operands */

void insn_replace(struct insn *insn, insn_op op, ...)
{
    va_list args;
    insn_index index;
    insns_entry links;

    index = insn->index;
    links = insn->links;

    va_start(args, op);
    insn_destruct(insn);
    memset(insn, 0, sizeof(struct insn));
    insn_construct(insn, op, args);
    va_end(args);

    insn->index = index;
    insn->links = links;
}

/* release insn resources and free it */

void insn_free(struct insn *insn)
{
    insn_destruct(insn);
    free(insn);
}

/* remove an instruction from the list, and free it. the work
   is in updating the indexes for the following insns. */

void insns_remove(struct insns *insns, struct insn *insn)
{
    struct insn *next;
    
    next = INSNS_NEXT(insn);
    INSNS_REMOVE(insns, insn);

    while (next) {
        --(next->index);
        next = INSNS_NEXT(next);
    }
    
    --(insns->next_index);
    insn_free(insn);
}

/* renumber all the insns */

static void renumber0(struct insns *insns)
{
    struct insn *insn;
    insn_index index;

    index = INSN_INDEX_FIRST;

    INSNS_FOREACH(insn, insns)
        insn->index = index++;

    insns->next_index = index;
}

/* insert a sequence of instructions src before or after an
   insn in another sequence dst. the src sequence is emptied. */

void insns_insert_before(struct insns *dst, struct insn *before,
                         struct insns *src)
{
    struct insn *insn;

    while (insn = INSNS_FIRST(src)) {
        INSNS_REMOVE(src, insn);
        INSNS_INSERT_BEFORE(before, insn);
    }

    INSNS_INIT(src);
    renumber0(dst);
}

void insns_insert_after(struct insns *dst, struct insn *after,
                        struct insns *src)
{
    struct insn *insn;

    while (insn = INSNS_FIRST(src)) {
        INSNS_REMOVE(src, insn);
        INSNS_INSERT_AFTER(dst, after, insn);
        after = insn;
    }

    INSNS_INIT(src);
    renumber0(dst);
}

/* append an instruction to the list */

void insn_append(struct insns *insns, struct insn *insn)
{
    insn->index = (insns->next_index)++;
    INSNS_APPEND(insns, insn);
}

/* prepend an instruction to the list */

void insn_prepend(struct insns *insns, struct insn *insn)
{
    INSNS_PREPEND(insns, insn);
    renumber0(insns);
}

/* append a sequence of insns to the end of another.
   the src list is emptied. */

void insns_append(struct insns *dst, struct insns *src)
{
    INSNS_CONCAT(dst, src);
    INSNS_INIT(src);
    renumber0(dst);
}

/* push n insns from the end of src to dst */

void insns_push(struct insns *dst, struct insns *src, int n)
{
    struct insn *insn;

    while (n--) {
        insn = INSNS_LAST(src);
        INSNS_REMOVE(src, insn);
        INSNS_PREPEND(dst, insn);
    }

    renumber0(src);
    renumber0(dst);
}

/* swap an instruction with the one after it */

void insns_swap(struct insns *insns, struct insn *insn)
{
    struct insn *next;

    next = INSNS_NEXT(insn);
    INSNS_REMOVE(insns, next);
    INSNS_INSERT_BEFORE(insn, next);

    ++(insn->index);
    --(next->index);
}

/* remove all insns from list and free them */

void insns_clear(struct insns *insns)
{
    struct insn *insn;

    while (insn = INSNS_FIRST(insns)) {
        INSNS_REMOVE(insns, insn);
        insn_free(insn);
    }

    insns->next_index = INSN_INDEX_FIRST;
}

/* swap the src operands of a binary insn */

extern void insn_commute(struct insn *insn)
{
    struct operand *tmp;

    tmp = insn->src1;
    insn->src1 = insn->src2;
    insn->src2 = tmp;
}

/* normalize an insn. for now,

   1. pure constants go on the right (src2)

   2. if one of the srcs is the same as dst,
      that src goes on the left (src1)

   these rules will be expanded for sure when
   we implement value numbering et al. */

extern void insn_normalize(struct insn *insn)
{
    if (I_SWAP(insn->op)) {
        if (OPERAND_PURE_CON(insn->src1)) {
            insn_commute(insn);
            return;
        }

        if (operand_is_same(insn->dst, insn->src2)) {
            insn_commute(insn);
            return;
        }
    }
}

/* returns TRUE if this instruction has side effects beyond the
   DEF of the registers (if any) returned by insn_defs_regs(),
   updating the condition codes, or stores to memory. */

bool insn_side_effects(struct insn *insn)
{
    if (insn->flags & INSN_FLAG_VOLATILE)
        return TRUE;

    if (I_TARGET(insn->op))
        return target->insn_side_effects(insn);
    else
        return (I_SIDE(insn->op) != 0);
}

/* returns TRUE if the instruction is a pure register to
   register copy, with no other side effects. if so, then
   *dst and *src are set to the destination and source
   registers, respectively. */

bool insn_copy(struct insn *insn, pseudo_reg *dst, pseudo_reg *src)
{
    if (I_TARGET(insn->op))
        return target->insn_copy(insn, dst, src);
    else {
        if ((insn->op == I_MOVE) && OPERAND_REG(insn->src1)) {
            *dst = insn->dst->reg;
            *src = insn->src1->reg;
            return TRUE;
        } else
            return FALSE;
    }
}

/* if the instruction is merely a load of a constant, then return the
   constant as an operand and set *dst accordingly. otherwise return 0. */

struct operand *insn_con(struct insn *insn, pseudo_reg *dst)
{
    if (I_TARGET(insn->op))
        return target->insn_con(insn, dst);
    else {
        if ((insn->op == I_MOVE) && OPERAND_CON(insn->src1)) {
            *dst = insn->dst->reg;
            return operand_dup(insn->src1);
        } else
            return 0;
    }
}

/* replace occurrences of reg with value, if possible. returns
   TRUE if any substitutions made, FALSE otherwise. */

#define INSN_SUBSTITUTE_CON0(opr)                                           \
    do {                                                                    \
        if (OPERAND_REG(opr) && ((opr)->reg == reg)) {                      \
            operand_free(opr);                                              \
            (opr) = operand_dup(value);                                     \
            changed = TRUE;                                                 \
        }                                                                   \
    } while (0)

bool insn_substitute_con(struct insn *insn, pseudo_reg reg,
                         struct operand *value)
{
    bool changed;

    if (I_TARGET(insn->op))
        return target->insn_substitute_con(insn, reg, value);
    else {
        changed = FALSE;

        if (I_USE_DST(insn->op))
            INSN_SUBSTITUTE_CON0(insn->dst);

        INSN_SUBSTITUTE_CON0(insn->src1);
        INSN_SUBSTITUTE_CON0(insn->src2);

        return changed;
    }
}

/* replace all mentions of the register from with register to.
   returns TRUE if any substitutions made, FALSE otherwise. */

#define INSN_SUBSTITUTE_REG0(OPR)                                           \
    do {                                                                    \
        if (OPERAND_REG(insn->OPR) && (insn->OPR->reg == from)) {           \
            insn->OPR->reg = to;                                            \
            result = TRUE;                                                  \
        }                                                                   \
    } while (0)

bool insn_substitute_reg(struct insn *insn, pseudo_reg from, pseudo_reg to,
                         insn_substitute_flags flags)
{
    bool result = FALSE;

    if (I_TARGET(insn->op))
        return target->insn_substitute_reg(insn, from, to, flags);
    else {
        if ((flags & INSN_SUBSTITUTE_DEFS) && !I_USE_DST(insn->op))
            INSN_SUBSTITUTE_REG0(dst);

        if (flags & INSN_SUBSTITUTE_USES) {
            if (I_USE_DST(insn->op))
                INSN_SUBSTITUTE_REG0(dst);

            INSN_SUBSTITUTE_REG0(src1);
            INSN_SUBSTITUTE_REG0(src2);
        }
    
        return result;
    }
}

/* returns TRUE if the instruction updates the condition codes */

bool insn_defs_cc(struct insn *insn)
{
    if (I_TARGET(insn->op))
        return target->insn_defs_cc(insn);
    else
        return (I_DEF_CC(insn->op) != 0);
}

/* return the set of condition_codes used
   by the insn, (empty if none used. */

ccset insn_uses_cc(struct insn *insn)
{
    ccset ccs;

    if (I_TARGET(insn->op))
        return target->insn_uses_cc(insn);
    else {
        CCSET_CLEAR(ccs);
        
        if (I_IS_SET_CC(insn->op))
            CCSET_SET(ccs, I_CC_FROM_SET_CC(insn->op));
        
        return ccs;
    }
}

/* returns TRUE if the instruction writes to memory */

bool insn_defs_mem(struct insn *insn)
{
    if (I_TARGET(insn->op))
        return target->insn_defs_mem(insn);
    else
        return (I_DEF_MEM(insn->op) != 0);
}

/* returns TRUE if the instruction reads from memory */

bool insn_uses_mem(struct insn *insn)
{
    if (I_TARGET(insn->op))
        return target->insn_uses_mem(insn);
    else
        return (I_USE_MEM(insn->op) != 0);
}

/* strip regs in the insn of their indices */

#define STRIP0(o)                                                       \
    do {                                                                \
        if (OPERAND_REG(o))                                             \
            o->reg = PSEUDO_REG_BASE(o->reg);                           \
    } while (0);

void insn_strip_indices(struct insn *insn)
{
    if (I_TARGET(insn->op))
        target->insn_strip_indices(insn);
    else {
        STRIP0(insn->dst);
        STRIP0(insn->src1);
        STRIP0(insn->src2);
    }
}

/* returns TRUE if the instruction is a test of a register
   against a constant, and has no effect except to set the
   condition codes. if it is, and reg is not 0, then *reg
   is set to indicate which pseudo_reg is subject to test. */

bool insn_test_con(struct insn *insn, pseudo_reg *reg)
{
    if (I_TARGET(insn->op))
        return target->insn_test_con(insn, reg);
    else {
        if ((insn->op == I_CMP) 
          && ((OPERAND_PURE_CON(insn->src1) && OPERAND_REG(insn->src2))
          || (OPERAND_PURE_CON(insn->src2) && OPERAND_REG(insn->src1))))
        {
            if (reg) {
                if (OPERAND_REG(insn->src1))
                    *reg = insn->src1->reg;
                else
                    *reg = insn->src2->reg;
            }

            return TRUE;
        } else
            return FALSE;
    }
}

/* just like insn_test_con(), except that
   the constant in question must be zero. */

bool insn_test_z(struct insn *insn, pseudo_reg *reg)
{
    if (I_TARGET(insn->op))
        return target->insn_test_z(insn, reg);
    else {
        if (insn_test_con(insn, reg))
            return (operand_is_zero(insn->src1)
                    || operand_is_zero(insn->src2));
        else
            return FALSE;
    }
}

/* returns TRUE if the Z flag is set by the insn
   when a register is set to zero by said insn. if
   so and reg is not 0, set *reg to that register;
   otherwise leaves *reg unchanged. */

bool insn_defs_z(struct insn *insn, pseudo_reg *reg)
{
    if (I_TARGET(insn->op))
        return target->insn_defs_z(insn, reg);
    else
        return FALSE;
}

/* these return TRUE if this instruction DEFs (USEs) any registers. if 
   so, and regs is not 0, the DEFd (USEd) registers are added to the set.
   if INSN_DEFSUSES_CC is in flags, then PSEUDO_REG_CC is included in the
   set of registers if the insn DEFs (USEs) condition codes. similarly,
   if INSN_DEFUSES_MEM is in flags, then PSEUDO_REG_MEM is included if the
   insn DEFs (USEs) memory. */

#define INSN_DEFSUSES0(pred, REG, reg)                                      \
    do {                                                                    \
        if ((flags & INSN_DEFSUSES_##REG) && insn_##pred##_##reg(insn)) {   \
            result = TRUE;                                                  \
                                                                            \
            if (regs)                                                       \
                REGS_ADD(regs, PSEUDO_REG_##REG);                           \
        }                                                                   \
    } while (0)

bool insn_defs_regs(struct insn *insn, struct regs *regs,
                    insn_defsuses_flags flags)
{
    bool result = FALSE;

    if (I_TARGET(insn->op))
        result = target->insn_defs_regs(insn, regs);
    else {
        if (insn->dst && OPERAND_REG(insn->dst) && !I_USE_DST(insn->op)) {
            result = TRUE;

            if (regs)
                REGS_ADD(regs, insn->dst->reg);
        }
    }

    INSN_DEFSUSES0(defs, CC, cc);
    INSN_DEFSUSES0(defs, MEM, mem);

    return result;
}

#define INSN_USES_REG(OPR)                                                  \
    do {                                                                    \
        if ((OPR) && OPERAND_REG(OPR)) {                                    \
            result = TRUE;                                                  \
                                                                            \
            if (regs)                                                       \
                REGS_ADD(regs, (OPR)->reg);                                 \
        }                                                                   \
    } while (0)

bool insn_uses_regs(struct insn *insn, struct regs *regs,
                    insn_defsuses_flags flags)
{
    bool result = FALSE;

    if (I_TARGET(insn->op))
        result = target->insn_uses_regs(insn, regs);
    else {
        if (I_USE_DST(insn->op))
            INSN_USES_REG(insn->dst);

        INSN_USES_REG(insn->src1);
        INSN_USES_REG(insn->src2);
    }

    INSN_DEFSUSES0(uses, CC, cc);
    INSN_DEFSUSES0(uses, MEM, mem);

    return result;
}

/* vi: set ts=4 expandtab: */
