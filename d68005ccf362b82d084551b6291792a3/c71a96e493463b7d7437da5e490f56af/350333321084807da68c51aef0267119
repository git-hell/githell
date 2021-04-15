/* insn.c - AMD64 instructions                          ncc, the new c compiler

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
#include "../../common/util.h"
#include "../cc1.h"
#include "../init.h"
#include "../output.h"
#include "../regs.h"
#include "reg.h"
#include "insn.h"

/* create and initialize a new amd64_operand */

static struct amd64_operand *amd64_operand_new(type_bits ts)
{
    struct amd64_operand *o;
    static int age;

    o = safe_malloc(sizeof(struct amd64_operand));

    o->reg = PSEUDO_REG_NONE;
    o->idx = PSEUDO_REG_NONE;
    o->scale = 1;
    o->ts = ts;
    o->age = age++;

    return o;
}

/* return a duplicate of src */

struct amd64_operand *amd64_operand_dup(struct amd64_operand *src)
{
    struct amd64_operand *dst;

    dst = safe_malloc(sizeof(struct amd64_operand));
    *dst = *src;

    return dst;
}

/* return an amd64_operand referring to the reg */

struct amd64_operand *amd64_operand_reg(type_bits ts, pseudo_reg reg)
{
    struct amd64_operand *o;

    o = amd64_operand_new(ts);
    o->class = AMD64_O_REG;
    o->reg = reg;

    return o;
}

/* return an AMD64_O_CON referring to a symbol */

struct amd64_operand *amd64_operand_sym(struct symbol *sym)
{
    struct amd64_operand *o;

    o = amd64_operand_new(T_ULONG);
    o->class = AMD64_O_CON;
    o->sym = sym;

    return o;
}

/* return an AMD64_O_CON with the given type and value */

struct amd64_operand *amd64_operand_con(type_bits ts, long i,
                                        struct symbol *sym)
{
    struct amd64_operand *o;

    o = amd64_operand_new(ts);
    o->class = AMD64_O_CON;
    o->i = i;
    o->sym = sym;

    return o;
}

/* free an amd64_operand (null safe) */

void amd64_operand_free(struct amd64_operand *o)
{
    if (o)
        free(o);
}

/* create an amd64_operand congruent to the target-independent
   operand src. since there are no such things as floating-point
   immediates, this is where they become memory references. for
   convenience, this function is null-safe. */

struct amd64_operand *amd64_operand_import(struct operand *src)
{
    struct amd64_operand *o = 0;

    if (src != 0) {
        o = amd64_operand_new(src->ts);

        if (OPERAND_REG(src)) {
            o->class = AMD64_O_REG;
            o->reg = src->reg;
        } else if (OPERAND_CON(src)) {
            if (src->ts & T_DISCRETE) {
                o->class = AMD64_O_CON;
                o->i = src->con.i;
                o->sym = src->sym;
            } else {
                o->class = AMD64_O_MEM;
                o->sym = init_constant_symbol(src->ts, src->con);
            }
        }
    }

    return o;
}

/* build an AMD64_O_EFF representing an offset
   from the given base register address */

struct amd64_operand *amd64_operand_based(pseudo_reg reg, int offset)
{
    struct amd64_operand *o;

    o = amd64_operand_new(T_ULONG);
    o->class = AMD64_O_EFF;
    o->reg = reg;
    o->i = offset;

    return o;
}

/* build an AMD64_O_EFF representing a scaled register */

struct amd64_operand *amd64_operand_scaled(type_bits ts, pseudo_reg reg,
                                           int scale)
{
    struct amd64_operand *o;
    
    o = amd64_operand_new(ts);
    o->class = AMD64_O_EFF;
    o->idx = reg;
    o->scale = scale;

    return o;
}

/* returns a temporary reg of the specified type */

struct amd64_operand *amd64_operand_tmp(type_bits ts)
{
    struct amd64_operand *o;
    pseudo_reg reg;

    reg = symbol_temp_reg(ts);
    o = amd64_operand_reg(ts, reg);

    return o;
}

/* adds all registers mentioned in operand o to regs.
   returns TRUE if any registers are mentioned. */

bool amd64_operand_regs(struct amd64_operand *o, struct regs *regs)
{
    bool ret = FALSE;

    if (o->reg != PSEUDO_REG_NONE) {
        REGS_ADD(regs, o->reg);
        ret = TRUE;
    }

    if (o->idx != PSEUDO_REG_NONE) {
        REGS_ADD(regs, o->idx);
        ret = TRUE;
    }

    return ret;
}

/* returns TRUE if the operand is a small constant, i.e., one
   that can be represented as a zero-extended 32-bit value. our
   memory model allows symbols to be negative, so the presence
   of a symbol disqualifies an operand from being "small". */

bool amd64_operand_small(struct amd64_operand *o)
{
    if (AMD64_OPERAND_CON(o) && ((o->i & 0xFFFFFFFF) == o->i)
      && (o->sym == 0))
        return TRUE;

    return FALSE;
}

/* returns TRUE if the operand is a huge constant, i.e., a 64-bit
   constant that can't be represented as sign-extended 32-bit value. */

bool amd64_operand_huge(struct amd64_operand *o)
{
    if ((o->ts & T_LONGS) && AMD64_OPERAND_CON(o) && AMD64_HUGE_CON(o->i))
        return TRUE;

    return FALSE;
}

/* returns a simple measure of the complexity of an operand */

int amd64_operand_degree(struct amd64_operand *o)
{
    int degree = 0;

    if (o->reg != PSEUDO_REG_NONE)
        ++degree;

    if (o->idx != PSEUDO_REG_NONE)
        ++degree;

    if ((o->sym != 0) || (o->i != 0))
        ++degree;

    if (o->scale != 1)
        ++degree;

    return degree;
}

/* normalize an operand */

void amd64_operand_normalize(struct amd64_operand *o)
{
    /* a single, unscaled register is always the base reg */

    if ((o->idx != PSEUDO_REG_NONE) && (o->scale == 1)
      && (o->reg == PSEUDO_REG_NONE)) {
        o->reg = o->idx;
        o->idx = PSEUDO_REG_NONE;
    }

    if (!AMD64_OPERAND_MEM(o)) {
        o->class = AMD64_O_EFF;

        /* an operand with only a constant and/or sym is an AMD64_O_CON */

        if ((o->reg == PSEUDO_REG_NONE) && (o->idx == PSEUDO_REG_NONE))
            o->class = AMD64_O_CON;

        /* an operand with only a base register is an AMD64_O_REG */

        if ((o->reg != PSEUDO_REG_NONE) && (o->idx == PSEUDO_REG_NONE)
          && (o->i == 0) && (o->sym == 0))
            o->class = AMD64_O_REG;
    }
}

/* attempt to combine two operands into one. returns
   the combined operand, or 0 if not possible. */

#define COMBINE0(o)                                                     \
    do {                                                                \
        if ((o)->reg != PSEUDO_REG_NONE)                                \
            ++regs;                                                     \
                                                                        \
        if ((o)->idx != PSEUDO_REG_NONE)                                \
            ++regs;                                                     \
                                                                        \
        if ((o)->scale != 1)                                            \
            ++scales;                                                   \
    } while (0)

struct amd64_operand *amd64_operands_combine(struct amd64_operand *o1,
                                             struct amd64_operand *o2)
{
    struct amd64_operand *combo;
    int regs = 0;
    int scales = 0;
    long i;

    if (AMD64_OPERAND_MEM(o1) || AMD64_OPERAND_MEM(o2))
        return 0;

    if (o1->sym && o2->sym)
        return 0;

    COMBINE0(o1);
    COMBINE0(o2);

    if ((regs > 2) || (scales > 2))
        return 0;

    if (amd64_operand_huge(o1) || amd64_operand_huge(o2))
        return 0;

    i = o1->i;
    i += o2->i;

    if (AMD64_HUGE_CON(i))
        return 0;

    combo = amd64_operand_dup(o1);
    
    if (combo->sym == 0)
        combo->sym = o2->sym;

    if ((combo->reg != PSEUDO_REG_NONE) && (o2->reg != PSEUDO_REG_NONE))
        combo->idx = o2->reg;
    else {
        if (combo->reg == PSEUDO_REG_NONE)
            combo->reg = o2->reg;

        if (combo->idx == PSEUDO_REG_NONE)
            combo->idx = o2->idx;

        if (combo->scale == 1)
            combo->scale = o2->scale;
    }

    combo->i = i;
    combo->age = MIN(combo->age, o2->age);
    amd64_operand_normalize(combo);

    return combo;
}

/* returns TRUE if the operands are identical */

bool amd64_operands_same(struct amd64_operand *o1, struct amd64_operand *o2)
{
    if ((o1->class != o2->class) || (o1->ts != o2->ts)
      || (o1->reg != o2->reg) || (o1->idx != o2->idx)
      || (o1->scale != o2->scale) || (o1->sym != o2->sym)
      || (o1->i != o2->i))
        return FALSE;
    else
        return TRUE;
}

/* replace all occurrences of reg from with reg to in the operand.
   returns TRUE if a replacement occurred, FALSE otherwise. */

#define AMD64_OPERAND_SUBSTITUTE_REG0(r)                                \
    do {                                                                \
        if (o->r == from) {                                             \
            o->r = to;                                                  \
            ret = TRUE;                                                 \
        }                                                               \
    } while (0)

static bool amd64_operand_substitute_reg(struct amd64_operand *o,
                                         pseudo_reg from, pseudo_reg to)
{
    bool ret = FALSE;

    AMD64_OPERAND_SUBSTITUTE_REG0(reg);
    AMD64_OPERAND_SUBSTITUTE_REG0(idx);
    
    return ret;
}

/* output an operand in assembler format.

   because CALL instructions have a slightly different operand syntax, 
   we need the call argument to tell us if it's a call or not. */

static void amd64_operand_output(struct amd64_operand *o, int size,
                                 bool call)
{
    if (AMD64_OPERAND_CON(o)) {
        if (!call)
            output("$");
    } else {
        if (call)
            output("*");
    }
            
    switch (o->class)
    {
    case AMD64_O_REG:   amd64_reg_output(o->reg, size); break;
    case AMD64_O_CON:   output("%G", o->sym, o->i); break;

    case AMD64_O_EFF:
    case AMD64_O_MEM:   if (o->sym || o->i)
                            output("%G", o->sym, o->i);

                        output("(");

                        if (o->sym && (o->reg == PSEUDO_REG_NONE)
                          && (o->idx == PSEUDO_REG_NONE))
                        {
                            /* prefer RIP-relative addressing for efficiency
                               when we're not using any register indices and
                               we have a symbol as an anchor. */

                            output("%%rip");
                        } else {
                            if (o->reg != PSEUDO_REG_NONE)
                                amd64_reg_output(o->reg, AMD64_SIZE_QWORD);

                            if (o->idx != PSEUDO_REG_NONE) {
                                output(",");
                                amd64_reg_output(o->idx, AMD64_SIZE_QWORD);
                            }

                            if (o->scale > 1)
                                output(",%d", o->scale);
                        }

                        output(")");
    }
}

/* complete the construction of an AMD64 insn
   by assigning any applicable operands */

void amd64_insn_construct(struct insn *insn, va_list args)
{
    int n;

    for (n = 0; n < AMD64_I_OPERANDS(insn->op); ++n)
        insn->amd64[n] = va_arg(args, struct amd64_operand *);
}

/* destroy an AMD64 insn by releasing its operands */

void amd64_insn_destruct(struct insn *insn)
{
    amd64_operand_free(insn->amd64[0]);
    amd64_operand_free(insn->amd64[1]);
}

/* return a duplicate of an AMD64 insn */

struct insn *amd64_insn_dup(struct insn *insn)
{
    struct insn *new;

    new = insn_new(I_NOP);
    new->op = insn->op;
    new->amd64[0] = amd64_operand_dup(insn->amd64[0]);
    new->amd64[1] = amd64_operand_dup(insn->amd64[1]);

    return new;
}

/* if the instruction modifies the condition codes */

bool amd64_insn_defs_cc(struct insn *insn)
{
    return (insn->op & AMD64_I_FLAG_DEFCC) != 0;
}

/* return the codes used by the insn. for
   now, be pessimistic and say all of 'em
   if any are used. */

ccset amd64_insn_uses_cc(struct insn *insn)
{
    ccset ccs;

    CCSET_CLEAR(ccs);

    if (insn->op & AMD64_I_FLAG_USECC)
        ccs = CCSET_ALL;

    return ccs;
}

/* if the instruction has side effects. note
   that we count assignments to RBP or RSP. */

bool amd64_insn_side_effects(struct insn *insn)
{
    struct regs regs = REGS_INITIALIZER(regs);
    bool ret = FALSE;
    
    if (insn->op & AMD64_I_FLAG_SIDE)
        ret = TRUE;
    else {
        amd64_insn_defs_regs(insn, &regs);

        if (REGS_CONTAINS(&regs, AMD64_REG_RSP)
          || REGS_CONTAINS(&regs, AMD64_REG_RBP))
            ret = TRUE;
    
        regs_clear(&regs);
    }

    return ret;
}

/* if the instruction compares a reg against 0, 
   return TRUE and (if reg is not 0) set *reg.
   otherwise return FALSE and leave *reg alone. */

bool amd64_insn_test_z(struct insn *insn, pseudo_reg *reg)
{
    struct amd64_operand *o;

    o = 0;

    switch (insn->op)
    {
    case AMD64_I_CMPL:
    case AMD64_I_CMPQ:
        if (AMD64_OPERAND_ZERO(insn->amd64[0]) ||
          AMD64_OPERAND_ZERO(insn->amd64[1])) {
            if (AMD64_OPERAND_REG(insn->amd64[1]))
                o = insn->amd64[0];

            if (AMD64_OPERAND_REG(insn->amd64[1]))
                o = insn->amd64[1];

            if (o) {
                if (reg)
                    *reg = o->reg;

                return TRUE;
            }
        }
    }   

    if (o) {
        if (reg)
            *reg = o->reg;

        return TRUE;
    } else
        return FALSE;
}

/* if the instruction is known to set the Z flag
   based on the resulting value of a reg, then (if
   reg is not 0) set *reg and return TRUE, otherwise
   return FALSE and leave *reg untouched. */

#define DEFS_Z0(n)                                                          \
    do {                                                                    \
        if (AMD64_I_DEFS(insn->op, n) && AMD64_OPERAND_REG(insn->amd64[n])) \
            o = insn->amd64[n];                                             \
    } while (0)

bool amd64_insn_defs_z(struct insn *insn, pseudo_reg *reg)
{
    struct amd64_operand *o;

    o = 0;

    if (AMD64_I_DEF_Z(insn->op)) {
        DEFS_Z0(0);
        DEFS_Z0(1);
    }

    if (o) {
        if (reg)
            *reg = o->reg;
        
        return TRUE;
    } else
        return FALSE;
}

/* if the instruction is a reg to reg copy. */

bool amd64_insn_copy(struct insn *insn, pseudo_reg *dst, pseudo_reg *src)
{
    switch (insn->op)
    {
    case AMD64_I_MOVSD:
    case AMD64_I_MOVSS:
    case AMD64_I_MOVL:
    case AMD64_I_MOVQ:
        
        if (AMD64_OPERAND_REG(insn->amd64[0])
          && AMD64_OPERAND_REG(insn->amd64[1])) {
            *dst = insn->amd64[1]->reg;
            *src = insn->amd64[0]->reg;
            return TRUE;
        }
    }

    return FALSE;
}

/* if the instruction modifies memory */

#define DEFS_MEM0(n)                                                        \
    do {                                                                    \
        if ((AMD64_I_OPERANDS(insn->op) >= (n))                             \
          && AMD64_I_DEFS(insn->op, (n))                                    \
          && AMD64_OPERAND_MEM(insn->amd64[n]))                             \
            return TRUE;                                                    \
    } while (0)

bool amd64_insn_defs_mem(struct insn *insn)
{
    if (insn->op & AMD64_I_FLAG_DEFMEM)
        return TRUE;

    DEFS_MEM0(0);
    DEFS_MEM0(1);

    return FALSE;
}

/* if the instruction reads memory */

#define USES_MEM0(n)                                                        \
    do {                                                                    \
        if ((AMD64_I_OPERANDS(insn->op) >= (n))                             \
          && AMD64_I_USES(insn->op, (n))                                    \
          && AMD64_OPERAND_MEM(insn->amd64[n]))                             \
            return TRUE;                                                    \
    } while (0)

bool amd64_insn_uses_mem(struct insn *insn)
{
    if (insn->op & AMD64_I_FLAG_USEMEM)
        return TRUE;

    USES_MEM0(0);
    USES_MEM0(1);
    
    return FALSE;
}

/* which registers are DEFd by the instruction */

#define DEFS_REGS0(n)                                                       \
    do {                                                                    \
        if ((AMD64_I_OPERANDS(insn->op) >= (n))                             \
          && AMD64_I_DEFS(insn->op, (n))                                    \
          && AMD64_OPERAND_REG(insn->amd64[n])) {                           \
            REGS_ADD(regs, insn->amd64[n]->reg);                            \
            ++count;                                                        \
        }                                                                   \
    } while (0)

bool amd64_insn_defs_regs(struct insn *insn, struct regs *regs)
{
    int count = 0;
    int i;

    DEFS_REGS0(0);
    DEFS_REGS0(1);

    if (AMD64_I_IS_CALL(insn->op)) {
        for (i = 0; i < AMD64_NR_ITRASH; ++i, ++count)
            REGS_ADD(regs, amd64_itrash[i]);

        for (i = 0; i < AMD64_NR_FTRASH; ++i, ++count)
            REGS_ADD(regs, AMD64_REG_XMM(i));
    }

    switch (insn->op)
    {
    case AMD64_I_CLTD:
    case AMD64_I_CQTO:      REGS_ADD(regs, AMD64_REG_RDX); ++count;
                            break;

    case AMD64_I_DIVL:
    case AMD64_I_DIVQ:
    case AMD64_I_IDIVL:
    case AMD64_I_IDIVQ:     REGS_ADD(regs, AMD64_REG_RAX); ++count;
                            REGS_ADD(regs, AMD64_REG_RDX); ++count;
                            break;

    case AMD64_I_MOVSQ:
    case AMD64_I_MOVSL:
    case AMD64_I_MOVSW:
    case AMD64_I_MOVSB:     REGS_ADD(regs, AMD64_REG_RDI); ++count;
                            REGS_ADD(regs, AMD64_REG_RSI); ++count;
                            REGS_ADD(regs, AMD64_REG_RCX); ++count;
                            break;

    case AMD64_I_STOSQ:
    case AMD64_I_STOSB:     REGS_ADD(regs, AMD64_REG_RDI); ++count;
                            REGS_ADD(regs, AMD64_REG_RCX); ++count;
                            break;
    }

    return (count != 0);
}

/* which registers are USEd by the instruction */

#define USES_REGS0(n)                                                       \
    do {                                                                    \
        if ((AMD64_I_OPERANDS(insn->op) >= (n))                             \
          && (AMD64_I_USES(insn->op, (n))                                   \
          || AMD64_OPERAND_MEM(insn->amd64[n]))                             \
          && amd64_operand_regs(insn->amd64[n], regs))                      \
            ++count;                                                        \
    } while (0)

bool amd64_insn_uses_regs(struct insn *insn, struct regs *regs)
{
    int count = 0;
    int i;

    USES_REGS0(0);
    USES_REGS0(1);

    if (AMD64_I_IS_CALL(insn->op)) {
        for (i = 0; i < AMD64_I_IARGS(insn->op); ++i, ++count)
            REGS_ADD(regs, amd64_iargs[i]);

        for (i = 0; i < AMD64_I_FARGS(insn->op); ++i, ++count)
            REGS_ADD(regs, AMD64_REG_XMM(i));
    }

    switch (insn->op)
    {
    case AMD64_I_CLTD:
    case AMD64_I_CQTO:      REGS_ADD(regs, AMD64_REG_RAX); ++count;
                            break;

    case AMD64_I_DIVL:
    case AMD64_I_DIVQ:
    case AMD64_I_IDIVL:
    case AMD64_I_IDIVQ:     REGS_ADD(regs, AMD64_REG_RAX); ++count;
                            REGS_ADD(regs, AMD64_REG_RDX); ++count;
                            break;

    case AMD64_I_MOVSQ:
    case AMD64_I_MOVSL:
    case AMD64_I_MOVSW:
    case AMD64_I_MOVSB:     REGS_ADD(regs, AMD64_REG_RDI); ++count;
                            REGS_ADD(regs, AMD64_REG_RSI); ++count;
                            REGS_ADD(regs, AMD64_REG_RCX); ++count;
                            break;

    case AMD64_I_STOSQ:
    case AMD64_I_STOSB:     REGS_ADD(regs, AMD64_REG_RDI); ++count;
                            REGS_ADD(regs, AMD64_REG_RCX); ++count;
                            REGS_ADD(regs, AMD64_REG_RAX); ++count;
                            break;

    case AMD64_I_RET_I:     REGS_ADD(regs, AMD64_REG_RAX); ++count;
                            break;

    case AMD64_I_RET_F:     REGS_ADD(regs, AMD64_REG_XMM(0)); ++count;
                            break;
    }

    return (count != 0);
}

/* replace occurrences of reg from with reg to in insn. */

bool amd64_insn_substitute_reg(struct insn *insn, pseudo_reg from,
                               pseudo_reg to, insn_substitute_flags flags)
{
    bool ret = FALSE;
    int i;

    for (i = 0; i < AMD64_I_OPERANDS(insn->op); ++i) {
        if ((flags & INSN_SUBSTITUTE_DEFS)
          && AMD64_I_DEFS(insn->op, i)
          && !AMD64_OPERAND_MEM(insn->amd64[i])
          && amd64_operand_substitute_reg(insn->amd64[i], from, to))
            ret = TRUE;
    
        if ((flags & INSN_SUBSTITUTE_USES)
          && (AMD64_I_USES(insn->op, i)
          ||  AMD64_OPERAND_MEM(insn->amd64[i]))
          && amd64_operand_substitute_reg(insn->amd64[i], from, to))
            ret = TRUE;
    }

    return ret;
}

/* output a complete AMD64 instruction for the assembler */

static char *amd64_insn_text[] =        /* keyed to AMD64_I_INDEX() */
{
    /*   0 */   "movb",         "movw",         "movl",         "movq",
    /*   4 */   "movsbl",       "movzbl",       "movsbq",       "movzbq",
    /*   8 */   "movswl",       "movzwl",       "movswq",       "movzwq",
    /*  12 */   "movslq",       "movl",         "movss",        "movsd",
    /*  16 */   "cvtsi2ssl",    "cvtsi2ssq",    "cvtsi2sdl",    "cvtsi2sdq",
    /*  20 */   "cvttss2sil",   "cvttss2siq",   "cvttsd2sil",   "cvttsd2siq",
    /*  24 */   "cvtss2sd",     "cvtsd2ss",     "xorl",         "pushq",
    /*  28 */   "popq",         "leal",         "leaq",         "setz",
    /*  32 */   "setnz",        "setg",         "setle",        "setge",
    /*  36 */   "setl",         "seta",         "setbe",        "setae",
    /*  40 */   "setb",         "ret",          "ret",          "ret",
    /*  44 */   "shll",         "shlq",         "shrl",         "shrq",
    /*  48 */   "sarl",         "sarq",         "addl",         "addq",
    /*  52 */   "addss",        "addsd",        "subl",         "subq",
    /*  56 */   "subss",        "subsd",        "andl",         "andq",
    /*  60 */   "orl",          "orq",          "xorl",         "xorq",
    /*  64 */   "cmpl",         "cmpq",         "ucomiss",      "ucomisd",
    /*  68 */   "notl",         "notq",         "negl",         "negq",
    /*  72 */   "mulss",        "mulsd",        "divss",        "divsd",
    /*  76 */   "imull",        "imulq",        "divl",         "divq",
    /*  80 */   "idivl",        "idivq",        "cltd",         "cqto",
    /*  84 */   "rep",          "movsb",        "stosb",        "call",
    /*  88 */   "addb",         "subb",         "addw",         "subw",
    /*  92 */   "testl",        "testq",        "stosq",        "xorps",
    /*  96 */   "movups",       "movsw",        "movsl",        "movsq"
};

void amd64_insn_output(struct insn *insn)
{
    output("\t%s", amd64_insn_text[AMD64_I_INDEX(insn->op)]);

    if (AMD64_I_OPERANDS(insn->op) > 0) {
        output(" ");
        amd64_operand_output(insn->amd64[0],
                             AMD64_I_SIZE(insn->op, 0),
                             AMD64_I_IS_CALL(insn->op) != 0);

        if (AMD64_I_OPERANDS(insn->op) > 1) {
            output(",");
            amd64_operand_output(insn->amd64[1],
                                 AMD64_I_SIZE(insn->op, 1),
                                 AMD64_I_IS_CALL(insn->op) != 0);
        }
    }
}

/* vi: set ts=4 expandtab: */
