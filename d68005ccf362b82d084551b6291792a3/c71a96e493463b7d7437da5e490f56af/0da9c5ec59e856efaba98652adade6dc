/* gen.c - AMD64 code generator                         ncc, the new c compiler

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
#include "../block.h"
#include "../dead.h"
#include "../symbol.h"
#include "amd64.h"
#include "insn.h"
#include "peep.h"
#include "fuse.h"
#include "reg.h"
#include "gen.h"

/* the only AMD64_O_MEM operands we'll encounter in this phase of the code
   generator are floating-point constants (and those which are brought to
   life briefly and then emitted in load/store). we can safely assume any
   other operand is AMD64_O_REG, AMD64_O_CON, or AMD64_O_EFF. */

/* the selxns cache holds a cache of reg -> AMD64_O_EFF operands, which
   are used to construct the more complex addressing modes of the AMD64.
   the bulk of the work in done in add(), with some help from shift(). */

#define AMD64_SELXN_CONSTRUCT(s)        (*(s) = 0)
#define AMD64_SELXN_DESTRUCT(s)         amd64_operand_free(*(s))
#define AMD64_SELXN_DUP(dst, src)       ((*(dst)) = amd64_operand_dup(*(src)))

static ASSOC_DEFINE_LOOKUP(amd64_selxn, pseudo_reg, reg)
static ASSOC_DEFINE_CLEAR(amd64_selxn, value, AMD64_SELXN_DESTRUCT)
static ASSOC_DEFINE_DUP(amd64_selxn, pseudo_reg, reg, value, AMD64_SELXN_DUP)

static ASSOC_DEFINE_INSERT(amd64_selxn, pseudo_reg, reg,
                           value, AMD64_SELXN_CONSTRUCT)

static ASSOC_DEFINE_UNSET(amd64_selxn, pseudo_reg, reg,
                          value, AMD64_SELXN_DESTRUCT)

#define AMD64_SELXNS_INIT(ss)       ASSOC_INIT(ss)
#define AMD64_SELXNS_FOREACH(s, ss) ASSOC_FOREACH(s, ss)

/* current block's &b->amd64.selxns. we only work one block at
   a time, and this way we don't have to pass the block around. */

static struct amd64_selxns *selxns;

/* at most, one new entry is added into the selxns cache for
   each insn processed. we note which register, if any, was
   cached, in exempt_reg, which protects it from eviction in
   invalidate0(). notice this get-out-of-jail free card only
   protects the entry from the root invocation from gen0():
   when the table is scanned for any dependent entries, the
   previously-exempt reg is again eligible to be evicted. this
   can only happen if the reg is dependent on itself, directly
   or indirectly, and is the desired behavior. */

static pseudo_reg exempt_reg;

/* invalidate the selxn entry for reg (if any) and,
   recursively, any selxn entries that reference reg */

static void invalidate0(pseudo_reg reg)
{
    struct regs regs = REGS_INITIALIZER(regs);
    struct amd64_selxn *selxn;
    struct reg *regs_r;

    if (reg == exempt_reg)
        exempt_reg = PSEUDO_REG_NONE;
    else
        amd64_selxns_unset(selxns, reg);

restart:
    AMD64_SELXNS_FOREACH(selxn, selxns) {
        amd64_operand_regs(selxn->value, &regs);

        REGS_FOREACH(regs_r, &regs) {
            if (regs_r->reg == reg) {
                /* the recursive call may snatch selxn out
                   from under us, so we must start over. */

                invalidate0(selxn->reg);
                regs_clear(&regs);
                goto restart;
            }
        }

        regs_clear(&regs);
    }
}

/* invalidate all registers DEFd in insn */

static void invalidate(struct insn *insn)
{
    struct regs regs = REGS_INITIALIZER(regs);
    struct reg *regs_r;

    insn_defs_regs(insn, &regs, 0);
    
    REGS_FOREACH(regs_r, &regs)
        invalidate0(regs_r->reg);

    regs_clear(&regs);
}

/* stash a copy of this reg/operand pair in the selxns cache.
   if an entry already exists, it's enough for us to simply
   update the entry itself- invalidate() will clean up all
   the dependent entries after this insn is completed. */

static void update(pseudo_reg reg, struct amd64_operand *o)
{
    struct amd64_selxn *selxn;

    selxn = amd64_selxns_insert(selxns, reg);

    if (selxn->value)
        amd64_operand_free(selxn->value);

    selxn->value = amd64_operand_dup(o);
    exempt_reg = reg;
}

/* decide if we should use a selxn operand to compute dst rather that
   computing it directly. if using an LEA on the operand can "do more"
   than we could do with a simple insn, then we go with that.

   this heuristic sometimes fails if the dst register is later coalesced
   with one of the operand registers. the result is correct, but suboptimal.
   any heuristic we choose here is bound to make similar errors, but there's
   no need to worry: we can correct any such mistakes in a peephole pass. */

#define USE_SELXN(o, dst)   ((amd64_operand_degree(o) > 2) ||               \
                            ((amd64_operand_degree(o) == 2) &&              \
                             !AMD64_OPERAND_REFS_REG((o), (dst))))

/* scan the given choices and return the first matching choice.
   an operand matches the template if its type_bits match at
   least one of the template type_bits, or if the operand is
   0 and the template type_bits are 0. table ends with all 0s.
   returns 0 if no match. */

struct choice
{
    type_bits opr0;
    type_bits opr1;
    insn_op op;
};

#define CHOICE_MATCH(o, t)      (((o) && ((o)->ts & (t))) ||                \
                                (((o) == 0) && ((t) == 0)))

static struct choice *match(struct choice *choices,
                            struct amd64_operand *opr0,
                            struct amd64_operand *opr1)
{
    struct choice *choice;

    for (choice = choices; choice->opr0 || choice->opr1; ++choice)
        if (CHOICE_MATCH(opr0, choice->opr0)
          && CHOICE_MATCH(opr1, choice->opr1))
            return choice;

    return 0;
}

/* find the right choice and emit an insn accordingly.
   returns the instruction in case the caller wants it. */

static struct insn *choose(struct choice *choices, struct amd64_operand *opr0,
                                                   struct amd64_operand *opr1)
{
    struct insn *insn;
    struct choice *choice;

    choice = match(choices, opr0, opr1);

    if (choice)
        EMIT(insn = insn_new(choice->op, opr0, opr1));
    else
        error(FATAL, "amd64/gen.c: choose() failure");

    return insn;
}

/* in addition to I_MOVE handling, move() is used by other
   functions which need to move an operand into a register,
   so it properly handles any source operands we'll see. */

static struct choice lea_choices[] =
{
    {   T_ANY,      T_CHARS | T_SHORTS | T_INTS,    AMD64_I_LEAL    },
    {   T_ANY,      T_LONGS,                        AMD64_I_LEAQ    },

    { 0 }
};

static struct choice move_choices[] = 
{
    {   T_ANY,      T_CHARS | T_SHORTS | T_INTS,    AMD64_I_MOVL    },
    {   T_ANY,      T_LONGS,                        AMD64_I_MOVQ    },
    {   T_ANY,      T_FLOAT,                        AMD64_I_MOVSS   },
    {   T_ANY,      T_DOUBLE,                       AMD64_I_MOVSD   },

    { 0 }
};

static void move(struct amd64_operand *src, struct amd64_operand *dst)
{
    if (AMD64_OPERAND_EFF(src))
        choose(lea_choices, src, dst);
    else {
        if ((src->ts & T_LONGS) && amd64_operand_small(src)) {
            /* if we can get away with it,
               use MOVL for efficiency. */

            EMIT(insn_new(AMD64_I_MOVL, src, dst));
        } else
            choose(move_choices, src, dst);
    }
}

/* I_FRAME */

static void frame(struct amd64_operand *ofs, struct amd64_operand *dst)
{
    struct amd64_operand *eff;

    eff = amd64_operand_based(AMD64_REG_RBP, ofs->i);
    amd64_operand_free(ofs);
    update(dst->reg, eff);
    move(eff, dst);
}

/* replace the given operand with its entry in the selxns
   cache, if present; otherwise return the operand unmolested */

static struct amd64_operand *materialize(struct amd64_operand *o)
{
    struct amd64_selxn *selxn;

    if (AMD64_OPERAND_REG(o)
      && (selxn = amd64_selxns_lookup(selxns, o->reg))) {
        amd64_operand_free(o);
        o = amd64_operand_dup(selxn->value);
    }

    return o;
}

/* if the operand is a huge constant, force it into a reg.
   returns the temporary reg if so, the original operand
   untouched if not. */

static struct amd64_operand *dehuge(struct amd64_operand *o)
{
    struct amd64_operand *tmp;

    if (amd64_operand_huge(o)) {
        tmp = amd64_operand_tmp(o->ts);
        move(o, amd64_operand_dup(tmp));
        o = tmp;
    }

    return o;
}

/* the operand is going to be used as a memory address.
   rewrite the operand from the cache, if possible, and
   then mutate it into an AMD64_O_MEM. */

static struct amd64_operand *memorize(struct amd64_operand *o)
{
    o = materialize(o);
    o = dehuge(o);
    AMD64_OPERAND_INDIRECT(o);

    return o;
}

/* if flags says we're volatile, mark the instruction volatile. */

static void apply_flags(struct insn *insn, insn_flags flags)
{
    insn->flags |= flags & (INSN_FLAG_VOLATILE | INSN_FLAG_SPILL);
}

/* I_LOAD */

static struct choice load_choices[] =
{
    {   T_ANY,      T_UCHAR | T_CHAR,       AMD64_I_MOVZBL  },
    {   T_ANY,      T_SCHAR,                AMD64_I_MOVSBL  },
    {   T_ANY,      T_USHORT,               AMD64_I_MOVZWL  },
    {   T_ANY,      T_SHORT,                AMD64_I_MOVSWL  },
    {   T_ANY,      T_INTS,                 AMD64_I_MOVL    },
    {   T_ANY,      T_LONGS,                AMD64_I_MOVQ    },
    {   T_ANY,      T_FLOAT,                AMD64_I_MOVSS   },
    {   T_ANY,      T_DOUBLE,               AMD64_I_MOVSD   },

    { 0 }
};

static void load(struct amd64_operand *src, struct amd64_operand *dst,
                 insn_flags flags)
{
    src = memorize(src);
    apply_flags(choose(load_choices, src, dst), flags);
}

/* if an operand is an AMD64_O_MEM, pluck it out of memory
   into a temporary register, and return that temp. else
   returns the original operand unharmed. */

static struct amd64_operand *demem(struct amd64_operand *o)
{
    struct amd64_operand *tmp;

    if (AMD64_OPERAND_MEM(o)) {
        tmp = amd64_operand_tmp(o->ts);
        move(o, amd64_operand_dup(tmp));
        o = tmp;
    }

    return o;
}

/* if an operand is an AMD64_O_CON, load it into a temporary
   register and return that, or return the original operand. */

static struct amd64_operand *decon(struct amd64_operand *o)
{
    struct amd64_operand *tmp;

    if (AMD64_OPERAND_CON(o)) {
        tmp = amd64_operand_tmp(o->ts);
        move(o, amd64_operand_dup(tmp));
        o = tmp;
    }

    return o;
}

/* I_STORE */

static struct choice store_choices[] =
{
    {   T_CHARS,    T_ANY,      AMD64_I_MOVB    },
    {   T_SHORTS,   T_ANY,      AMD64_I_MOVW    },
    {   T_INTS,     T_ANY,      AMD64_I_MOVL    },
    {   T_LONGS,    T_ANY,      AMD64_I_MOVQ    },
    {   T_FLOAT,    T_ANY,      AMD64_I_MOVSS   },
    {   T_DOUBLE,   T_ANY,      AMD64_I_MOVSD   },
    
    { 0 }
};

static void store(struct amd64_operand *src, struct amd64_operand *dst,
                  insn_flags flags)
{
    dst = memorize(dst);
    src = dehuge(src);
    src = demem(src);

    apply_flags(choose(store_choices, src, dst), flags);
}

/* I_CAST.

   we cover all the bases here, even though it's possible many
   of these combinations will never appear (optimization or no). */

static struct choice cast_choices[] =
{
    {   T_UCHAR | T_CHAR,   T_CHARS | T_SHORTS | T_INTS,    AMD64_I_MOVZBL  },
    {   T_UCHAR | T_CHAR,   T_LONGS,                        AMD64_I_MOVZBQ  },
    {   T_SCHAR,            T_CHARS | T_SHORTS | T_INTS,    AMD64_I_MOVSBL  },
    {   T_SCHAR,            T_LONGS,                        AMD64_I_MOVSBQ  },
                        
    {   T_USHORT,           T_CHARS | T_SHORTS | T_INTS,    AMD64_I_MOVZWL  },
    {   T_USHORT,           T_LONGS,                        AMD64_I_MOVZWQ  },
    {   T_SHORT,            T_CHARS | T_SHORTS | T_INTS,    AMD64_I_MOVSWL  },
    {   T_SHORT,            T_LONGS,                        AMD64_I_MOVSWQ  },

    {   T_INTS,             T_CHARS | T_SHORTS | T_INTS,    AMD64_I_MOVL    },
    {   T_INT,              T_LONGS,                        AMD64_I_MOVSLQ  },
    {   T_UINT,             T_LONGS,                        AMD64_I_MOVZLQ  },
    
    {   T_LONGS,            T_CHARS | T_SHORTS | T_INTS,    AMD64_I_MOVL    },
    {   T_LONGS,            T_LONGS,                        AMD64_I_MOVQ    },

    {   T_CHARS | T_SHORTS | T_INT,     T_FLOAT,    AMD64_I_CVTSI2SSL   },
    {   T_UINT | T_LONGS,               T_FLOAT,    AMD64_I_CVTSI2SSQ   },
    {   T_CHARS | T_SHORTS | T_INT,     T_DOUBLE,   AMD64_I_CVTSI2SDL   },
    {   T_UINT | T_LONGS,               T_DOUBLE,   AMD64_I_CVTSI2SDQ   },

    {   T_FLOAT,    T_FLOAT,                        AMD64_I_MOVSS       },
    {   T_FLOAT,    T_DOUBLE,                       AMD64_I_CVTSS2SD    },
    {   T_DOUBLE,   T_DOUBLE,                       AMD64_I_MOVSD       },
    {   T_DOUBLE,   T_FLOAT,                        AMD64_I_CVTSD2SS    },

    {   T_FLOAT,    T_CHARS | T_SHORTS | T_INT,     AMD64_I_CVTSS2SIL   },
    {   T_FLOAT,    T_UINT | T_LONGS,               AMD64_I_CVTSS2SIQ   },

    {   T_DOUBLE,   T_CHARS | T_SHORTS | T_INT,     AMD64_I_CVTSD2SIL   },
    {   T_DOUBLE,   T_UINT | T_LONGS,               AMD64_I_CVTSD2SIQ   },

    { 0 }
};

static void cast(struct amd64_operand *src1, struct amd64_operand *dst)
{
    struct choice *choice;
    struct amd64_operand *src2;

    choice = match(cast_choices, src1, dst);

    if (choice == 0)
        error(FATAL, "INTERNAL: amd64/gen.c: cast() failure");

    /* MOVQ and MOVL are fine with constant operands, even huge ones
       (with the exception dealt with next). nobody else is, though. */

    if ((choice->op != AMD64_I_MOVQ) && (choice->op != AMD64_I_MOVL))
        src1 = decon(src1);

    /* if the instruction we choose needs a longer source
       operand than we have, we need an intermediate upcast. */

    if ((src1->ts & (T_CHARS | T_SHORTS | T_UINT))
      && (dst->ts & T_FLOATING)) {
        src2 = amd64_operand_dup(src1);
        src2->ts = (src1->ts == T_UINT) ? T_LONG : T_INT;
        cast(amd64_operand_dup(src1), src2);
    }

    EMIT(insn_new(choice->op, src1, dst));
}

/* I_SET_cc: trivial. AMD64 defines this as a byte operation, but
   in our IR it's an int operation, so we zero-extend the result. */

static void setcc(insn_op op, struct amd64_operand *dst)
{
    EMIT(insn_new(op, amd64_operand_dup(dst)));
    EMIT(insn_new(AMD64_I_MOVZBL, amd64_operand_dup(dst), dst));
}

/* I_RETURN: another easy one. synthesize a mov to the appropriate
   return-value register, which we can tell solely by the type of src,
   which is a scalar- luckily the front end deals with struct returns. */

static void retrn(struct amd64_operand *src)
{
    struct amd64_operand *dst;

    if (src) {
        if (src->ts & T_FLOATING)
            dst = amd64_operand_reg(src->ts, AMD64_REG_XMM(0));
        else
            dst = amd64_operand_reg(src->ts, AMD64_REG_RAX);

        move(src, dst);
    }
}

/* standard binary operator processing. three strategies:

   1. if dst is the same as src1, then compute directly into dst
      in one step:
    
                        <op> <src2>, <dst>

   2. otherwise, compute into a temporary and then move to dst:

                        mov <src1>, <tmp>
                        <op> <src2>, <tmp>
                        mov <tmp>, <dst>

   in the second case, the copy propagation or register coalescing will
   determine if tmp can be eliminated and the computation done directly
   into dst.

   I_CMP is a special case, as its result is the condition codes. */

static struct choice mul_choices[] =
{
    {   T_CHARS | T_SHORTS | T_INTS,    T_ANY,          AMD64_I_IMULL   },
    {   T_LONGS,                        T_ANY,          AMD64_I_IMULQ   },
    {   T_FLOAT,                        T_ANY,          AMD64_I_MULSS   },
    {   T_DOUBLE,                       T_ANY,          AMD64_I_MULSD   },

    { 0 }
};

static struct choice div_choices[] =
{
    {   T_FLOAT,                        T_ANY,          AMD64_I_DIVSS   },
    {   T_DOUBLE,                       T_ANY,          AMD64_I_DIVSD   },

    { 0 }
};

static struct choice cmp_choices[] =
{
    {   T_INTS,                         T_ANY,          AMD64_I_CMPL    },
    {   T_LONGS,                        T_ANY,          AMD64_I_CMPQ    },
    {   T_FLOAT,                        T_ANY,          AMD64_I_UCOMISS },
    {   T_DOUBLE,                       T_ANY,          AMD64_I_UCOMISD },

    { 0 }
};

static struct choice test_choices[] =
{
    {   T_CHARS | T_SHORTS | T_INTS,    T_ANY,          AMD64_I_TESTL   },
    {   T_LONGS,                        T_ANY,          AMD64_I_TESTQ   },

    { 0 }
};

static struct choice and_choices[] =
{
    {   T_CHARS | T_SHORTS | T_INTS,    T_ANY,          AMD64_I_ANDL    },
    {   T_LONGS,                        T_ANY,          AMD64_I_ANDQ    },

    { 0 }
};

static struct choice or_choices[] =
{
    {   T_CHARS | T_SHORTS | T_INTS,    T_ANY,          AMD64_I_ORL     },
    {   T_LONGS,                        T_ANY,          AMD64_I_ORQ     },

    { 0 }
};

static struct choice xor_choices[] = 
{
    {   T_CHARS | T_SHORTS | T_INTS,    T_ANY,          AMD64_I_XORL    },
    {   T_LONGS,                        T_ANY,          AMD64_I_XORQ    },

    { 0 }
};

static void binary(struct choice *choices, struct amd64_operand *src1,
                   struct amd64_operand *src2, struct amd64_operand *dst)
{
    struct amd64_operand *tmp;

    src1 = dehuge(src1);
    src2 = dehuge(src2);

    if (AMD64_OPERAND_CON(src1) && AMD64_OPERAND_CON(src2))
        src2 = decon(src2);

    if (AMD64_OPERAND_MEM(src1) && AMD64_OPERAND_MEM(src2))
        src2 = demem(src2);

    if (dst == 0) {
        /* I_CMP or I_TEST */
        src1 = decon(src1);
        choose(choices, src2, src1);
    } else if (AMD64_OPERANDS_SAME_REG(src1, dst)) {
        choose(choices, src2, dst);
        amd64_operand_free(src1);
    } else {
        tmp = amd64_operand_tmp(dst->ts);
        move(src1, amd64_operand_dup(tmp));
        choose(choices, src2, amd64_operand_dup(tmp));
        move(tmp, dst);
    }
}

/* standard unary operator processing. simpler than binary(). */

static void unary(struct choice *choices, struct amd64_operand *src,
                  struct amd64_operand *dst)
{
    src = decon(src);

    if (AMD64_OPERANDS_SAME_REG(src, dst))
        amd64_operand_free(src);
    else
        move(src, amd64_operand_dup(dst));

    choose(choices, dst, 0);
}

/* I_NEG: a unary op for discrete types, but
   multiplication for floating-point types. */

static struct choice neg_choices[] =
{
    {   T_CHARS | T_SHORTS | T_INTS,    0,          AMD64_I_NEGL    },
    {   T_LONGS,                        0,          AMD64_I_NEGQ    },

    { 0 }
};

static void neg(struct amd64_operand *src, struct amd64_operand *dst)
{
    struct operand *src2;
    struct amd64_operand *amd_src2;

    if (dst->ts & T_FLOATING) {
        src2 = operand_f(dst->ts, -1);
        amd_src2 = amd64_operand_import(src2);
        binary(mul_choices, src, amd_src2, dst);
        operand_free(src2);
    } else
        unary(neg_choices, src, dst);
}

/* I_SHL and I_SHR: we deal with the idiosyncrasy of AMD64 that
   non-constant shift counts must be in the count register (CL).
   left shifts also look for the opportunity to cache a selxn.
   then we pass off the rest of the work to binary().

   we choose to sign/zero-extend right shifts rather than using
   sub-int shifts available on the AMD64 for two reasons: first,
   the penalty for a partial update is probably worse than the
   cost of the extension, and second, there is [or will be] an
   optimization pass that will potentially eliminate the extension
   when it knows the state of the most significant bits. */

static struct choice shl_choices[] =
{
    {   T_ANY,  T_CHARS | T_SHORTS | T_INTS,            AMD64_I_SHLL    },
    {   T_ANY,  T_LONGS,                                AMD64_I_SHLQ    },

    { 0 }
};

static struct choice shr_choices[] =
{
    {   T_ANY,  T_CHAR | T_UCHAR | T_USHORT | T_UINT,   AMD64_I_SHRL    },
    {   T_ANY,  T_SCHAR | T_SHORT | T_INT,              AMD64_I_SARL    },
    {   T_ANY,  T_ULONG,                                AMD64_I_SHRQ    },
    {   T_ANY,  T_LONG,                                 AMD64_I_SARQ    },

    { 0 }
};

static void shift(struct choice *choices, struct amd64_operand *src1,
                  struct amd64_operand *src2, struct amd64_operand *dst)
{
    struct amd64_operand *eff;
    struct amd64_operand *rcx;
    struct amd64_operand *ext;
    int scale;

    src1 = decon(src1);

    if ((choices == shl_choices) && AMD64_OPERAND_REG(src1)
      && AMD64_OPERAND_PURE_CON(src2)) {
        scale = 0;

        switch (src2->i)
        {
        case 1:     scale = 2; break;
        case 2:     scale = 4; break;
        case 3:     scale = 8; break;
        }

        if (scale) {
            eff = amd64_operand_scaled(src1->ts, src1->reg, scale);
            update(dst->reg, eff);

            if (USE_SELXN(eff, dst->reg)) {
                move(eff, dst);
                amd64_operand_free(src1);
                amd64_operand_free(src2);
                return;
            } else
                amd64_operand_free(eff);
        }
    }

    if ((choices == shr_choices) && (src1->ts & (T_CHARS | T_SHORTS))) {
        ext = amd64_operand_tmp((src1->ts & (T_CHAR | T_UCHAR | T_USHORT))
                                    ? T_UINT : T_INT);

        cast(src1, amd64_operand_dup(ext));
        src1 = ext;
    }

    if (!AMD64_OPERAND_PURE_CON(src2)) {
        rcx = amd64_operand_reg(src2->ts, AMD64_REG_RCX);
        move(src2, amd64_operand_dup(rcx));
        src2 = rcx;
    }

    binary(choices, src1, src2, dst);
}

/* I_ADD: here is where most of the operand-construction magic happens.

   the strategy is to find the highest-degree operand that results from
   combining src1 and src2 in combination with each other, directly or
   from their selxn cache entries. if we can successfully combine them
   into one operand, we stash that new entry.

   if two combinations result in two new operands with the same degree,
   we use the operand age as a tie-breaker (older entries win). the
   reasoning is that older entries are less likely to be dependent on
   intermediate temporaries.

   notice that we emit all the intermediate calculations, even if, 
   ultimately, they go unused because we've consolidated into one
   effective operand. we depend on a later dead-store elimination. */

static struct choice add_choices[] = 
{
    {   T_CHARS | T_SHORTS | T_INTS,    T_ANY,          AMD64_I_ADDL    },
    {   T_LONGS,                        T_ANY,          AMD64_I_ADDQ    },
    {   T_FLOAT,                        T_ANY,          AMD64_I_ADDSS   },
    {   T_DOUBLE,                       T_ANY,          AMD64_I_ADDSD   },

    { 0 }
};

#define ADD0(o1, o2)                                                    \
    do {                                                                \
        struct amd64_operand *combo;                                    \
        int combo_degree;                                               \
                                                                        \
        combo = amd64_operands_combine((o1), (o2));                     \
                                                                        \
        if (combo) {                                                    \
            combo_degree = amd64_operand_degree(combo);                 \
                                                                        \
            if ((winner == 0)                                           \
              || (combo_degree > degree)                                \
              || ((combo_degree == degree)                              \
              && (winner->age > combo->age))) {                         \
                amd64_operand_free(winner);                             \
                winner = combo;                                         \
                degree = combo_degree;                                  \
            } else                                                      \
                amd64_operand_free(combo);                              \
        }                                                               \
    } while (0)

static void add(struct amd64_operand *src1, struct amd64_operand *src2,
                struct amd64_operand *dst)
{
    struct amd64_operand *src1_m;
    struct amd64_operand *src2_m;
    struct amd64_operand *winner;
    int degree;

    if (!(dst->ts & T_FLOATING)) {
        winner = 0;
        degree = 0;

        src1_m = materialize(amd64_operand_dup(src1));
        src2_m = materialize(amd64_operand_dup(src2));

        ADD0(src1, src2);
        ADD0(src1_m, src2);
        ADD0(src1, src2_m);
        ADD0(src1_m, src2_m);

        amd64_operand_free(src1_m);
        amd64_operand_free(src2_m);

        if (winner)
            update(dst->reg, winner);

        if (USE_SELXN(winner, dst->reg)) {
            move(winner, dst);
            amd64_operand_free(src1);
            amd64_operand_free(src2);
            return;
        } else
            amd64_operand_free(winner);
    }

    binary(add_choices, src1, src2, dst);
}

/* we intercept subtraction, just in case it's subtraction of an
   integer constant, in which case it's really an add and there's
   a chance it could end up in the selxn cache, or combined with
   other operands- so we convert it to an add and call add(). */

static struct choice sub_choices[] =
{
    {   T_CHARS | T_SHORTS | T_INTS,    T_ANY,          AMD64_I_SUBL    },
    {   T_LONGS,                        T_ANY,          AMD64_I_SUBQ    },
    {   T_FLOAT,                        T_ANY,          AMD64_I_SUBSS   },
    {   T_DOUBLE,                       T_ANY,          AMD64_I_SUBSD   },

    { 0 }
};

static void sub(struct amd64_operand *src1, struct amd64_operand *src2,
                struct amd64_operand *dst)
{
    if ((dst->ts & T_DISCRETE) && AMD64_OPERAND_PURE_CON(src2)
      && !AMD64_HUGE_CON(src2->i)) {
        src2->i = -(src2->i);
        add(src1, src2, dst);
    } else
        binary(sub_choices, src1, src2, dst);
}

/* division/modulo operations. floating-point division is fobbed off
   on binary(), but integer division requires some hoop-jumping. */

static void divmod(pseudo_reg reg, struct amd64_operand *src1,
                   struct amd64_operand *src2, struct amd64_operand *dst)
{
    struct amd64_operand *result;

    if (dst->ts & T_FLOATING)
        binary(div_choices, src1, src2, dst);
    else {
        result = amd64_operand_reg(dst->ts, reg);

        src1 = decon(src1);
        src2 = decon(src2);

        move(src1, amd64_operand_reg(src1->ts, AMD64_REG_RAX));

        switch (T_BASE(src1->ts))
        {
        case T_ULONG:
            EMIT(insn_new(AMD64_I_ZERO, 
                          amd64_operand_reg(T_ULONG, AMD64_REG_RDX),
                          amd64_operand_reg(T_ULONG, AMD64_REG_RDX)));
            EMIT(insn_new(AMD64_I_DIVQ, src2));
            break;
            
        case T_LONG:
            EMIT(insn_new(AMD64_I_CQTO));
            EMIT(insn_new(AMD64_I_IDIVQ, src2));
            break;

        case T_UINT:
            EMIT(insn_new(AMD64_I_ZERO, 
                          amd64_operand_reg(T_UINT, AMD64_REG_RDX),
                          amd64_operand_reg(T_UINT, AMD64_REG_RDX)));
            EMIT(insn_new(AMD64_I_DIVL, src2));
            break;

        case T_INT:
            EMIT(insn_new(AMD64_I_CLTD));
            EMIT(insn_new(AMD64_I_IDIVL, src2));
            break;
        }

        move(result, dst);
    }
}

/* I_BLKCOPY: the IR does not tell us the alignment of the regions, so
   we assume 8-byte alignment and hope for the best. for small copies-
   those that would involve BLKCOPY_MAX load/store cycles or fewer- we
   do the copies manually. otherwise we fall back to REP MOVS with an
   operand size that results in an integral number of cycles; this is
   probably a decent (not: optimal) choice at least 95% of the time.

   newer CPUs just Do the Right Thing(tm) when handed a REP MOVSB. if
   you're going to be CISC, that's the obvious way to do it. i have no
   idea what took them so long to figure that out. */

#define BLKCOPY_MAX     4       /* max 64 bytes, 4 x MOVUPS */

#define BLKCOPY0(op, n, r)                                                  \
    do {                                                                    \
        EMIT(insn_new(op, amd64_operand_dup(src1), amd64_operand_dup(r)));  \
        EMIT(insn_new(op, amd64_operand_dup(r), amd64_operand_dup(dst)));   \
        bytes -= n;                                                         \
        dst->i += n;                                                        \
        src1->i += n;                                                       \
    } while (0)

static void blkcopy(struct amd64_operand *src1, struct amd64_operand *src2,
                    struct amd64_operand *dst)
{
    struct amd64_operand *tmp;
    struct amd64_operand *tmp_xmm;
    size_t bytes;
    insn_op op;
    int copies;

    bytes = src2->i;
    copies = bytes >> 4;            /* number of XMM copies */
    copies += (bytes >> 3) & 1;     /* number of qword copies */
    copies += (bytes >> 2) & 1;     /* ... dword ... */
    copies += (bytes >> 1) & 1;     /* ... word ... */
    copies += bytes & 1;            /* ... byte ... */

    if (copies > BLKCOPY_MAX) {
        if (bytes & 1)
            op = AMD64_I_MOVSB;
        else if (bytes & 2) {
            op = AMD64_I_MOVSW;
            src2->i >>= 1;
        } else if (bytes & 4) {
            op = AMD64_I_MOVSL;
            src2->i >>= 2;
        } else {
            op = AMD64_I_MOVSQ;
            src2->i >>= 3;
        }

        move(src1, amd64_operand_reg(T_ULONG, AMD64_REG_RSI));
        move(dst, amd64_operand_reg(T_ULONG, AMD64_REG_RDI));
        move(src2, amd64_operand_reg(T_ULONG, AMD64_REG_RCX));
        EMIT(insn_new(AMD64_I_REP));
        EMIT(insn_new(op));
    } else {
        tmp = 0;
        tmp_xmm = 0;
        src1 = memorize(src1);
        dst = memorize(dst);

        while (bytes) {
            if (bytes >= 16) {
                if (tmp_xmm == 0)
                    tmp_xmm = amd64_operand_tmp(T_DOUBLE);

                BLKCOPY0(AMD64_I_MOVUPS, 16, tmp_xmm);
            } else {
                if (tmp == 0)
                    tmp = amd64_operand_tmp(T_LONG);

                if (bytes >= 8)
                    BLKCOPY0(AMD64_I_MOVQ, 8, tmp);
                else if (bytes >= 4)
                    BLKCOPY0(AMD64_I_MOVL, 4, tmp);
                else if (bytes >= 2)
                    BLKCOPY0(AMD64_I_MOVW, 2, tmp);
                else
                    BLKCOPY0(AMD64_I_MOVB, 1, tmp);
            }
        }

        amd64_operand_free(src1);
        amd64_operand_free(src2);
        amd64_operand_free(dst);
        amd64_operand_free(tmp);
        amd64_operand_free(tmp_xmm);
    }
}

/* I_BLKZERO: we cheat a little here- at present, I_BLKZERO is only issued
   to zero stack space for aggregate automatics, and that is likely to
   remain the case (unless we go down the built-in memset()/bzero() path).
   as such, we know that the region we are going to zero is 8-byte aligned,
   and- more importantly- we know it's padded an 8-byte boundary.

   if the region is larger than BLKZERO_MAX bytes, then we fall back to
   REP/MOVSQ, which is reasonable since the microarchitecture is unknown. */

#define BLKZERO_MAX 64

static void blkzero(struct amd64_operand *src, struct amd64_operand *dst)
{
    struct amd64_operand *tmp_xmm;
    unsigned long bytes;

    bytes = src->i;
    bytes = ROUND_UP(bytes, AMD64_STACK_ALIGN);

    if (bytes > BLKZERO_MAX) {
        src->i = bytes / AMD64_STACK_ALIGN;
        move(dst, amd64_operand_reg(T_ULONG, AMD64_REG_RDI));
        move(src, amd64_operand_reg(T_ULONG, AMD64_REG_RCX));

        EMIT(insn_new(AMD64_I_ZERO, amd64_operand_reg(T_INT, AMD64_REG_RAX), 
                                    amd64_operand_reg(T_INT, AMD64_REG_RAX)));

        EMIT(insn_new(AMD64_I_REP));
        EMIT(insn_new(AMD64_I_STOSQ));
    } else {
        tmp_xmm = 0;
        dst = memorize(dst);

        while (bytes) {
            if (bytes >= 16) {
                if (tmp_xmm == 0) {
                    tmp_xmm = amd64_operand_tmp(T_DOUBLE);
                    EMIT(insn_new(AMD64_I_XMMZERO,
                                  amd64_operand_dup(tmp_xmm),
                                  amd64_operand_dup(tmp_xmm)));
                }

                EMIT(insn_new(AMD64_I_MOVUPS, amd64_operand_dup(tmp_xmm),
                                              amd64_operand_dup(dst)));

                dst->i += 16;
                bytes -= 16;
            } else {
                EMIT(insn_new(AMD64_I_MOVQ, amd64_operand_con(T_LONG, 0, 0),
                                            amd64_operand_dup(dst)));
                bytes -= 8;
            }
        }

        amd64_operand_free(dst);
    }
}

/* when we encounter I_ARG/I_VARG/I_BLKARG, we simply
   record the first; I_CALL will reset it when done. */

static struct insn *first_arg_insn;

static void arg(struct insn *insn, struct amd64_operand *src1,
                                   struct amd64_operand *src2)
{
    if (first_arg_insn == 0)
        first_arg_insn = insn;

    amd64_operand_free(src1);
    amd64_operand_free(src2);
}

/* iterate forward over the arguments, demoting any
   I_ARGs that don't fit in registers to I_VARGs. 
   this is also where we set nr_iargs/nr_fargs. */

static int nr_iargs;
static int nr_fargs;

#define CALL0(nr_args, max_args)                                            \
    do {                                                                    \
        if ((nr_args) == (max_args))                                        \
           arg_insn->op = I_VARG;                                           \
        else                                                                \
            ++(nr_args);                                                    \
    } while (0)

static void call0(void)
{
    struct insn *arg_insn;

    arg_insn = first_arg_insn;

    while (I_ARGUMENT(arg_insn->op)) {
        if (arg_insn->op == I_ARG) {
            if (arg_insn->src1->ts & T_FLOATING)
                CALL0(nr_fargs, AMD64_NR_FARGS);
            else
                CALL0(nr_iargs, AMD64_NR_IARGS);
        }

        arg_insn = INSNS_NEXT(arg_insn);
    }
}

/* we must track how many bytes we've pushed onto the
   stack so we can clean up after the a call. */

static int stack_allocked;

/* allocate bytes on the stack for a call: this function
   handles rounding to AMD64_STACK_ALIGN and adjusting RSP. */

static void stack_alloc(int bytes)
{
    bytes = ROUND_UP(bytes, AMD64_STACK_ALIGN);

    EMIT(insn_new(AMD64_I_SUBQ,
                  amd64_operand_con(T_LONG, bytes, 0),
                  amd64_operand_reg(T_LONG, AMD64_REG_RSP)));

    stack_allocked += bytes;
}

/* iterate backwards over the arguments, pushing stack arguments.
   there is a lot of room for improvement here, but stacked args
   are pretty infrequent, so it can wait. */

static void call1(struct insn *call_insn)
{
    struct amd64_operand *o;
    struct insn *arg_insn;

    arg_insn = call_insn;

    do {
        arg_insn = INSNS_PREV(arg_insn);

        if (arg_insn->op == I_VARG) {
            o = amd64_operand_import(arg_insn->src1);

            if (o->ts & T_DISCRETE) {
                o = dehuge(o);
                EMIT(insn_new(AMD64_I_PUSHQ, o));
                stack_allocked += 8;
            } else {
                stack_alloc(type_bits_sizeof(o->ts));
                store(o, amd64_operand_reg(T_LONG, AMD64_REG_RSP), 0);
            }
        } else if (arg_insn->op == I_BLKARG) {
            stack_alloc(arg_insn->src2->con.i);
            o = amd64_operand_import(arg_insn->src1);
            blkcopy(o, amd64_operand_con(T_LONG, arg_insn->src2->con.i, 0),
                       amd64_operand_reg(T_LONG, AMD64_REG_RSP));
        }
    } while (arg_insn != first_arg_insn);
}

/* iterate forward over the arguments,
   loading register arguments */

static void call2(void)
{
    struct insn *arg_insn;
    struct amd64_operand *arg;
    int i = 0;
    int f = 0;

    arg_insn = first_arg_insn;

    while (I_ARGUMENT(arg_insn->op)) {
        if (arg_insn->op == I_ARG) {
            arg = amd64_operand_import(arg_insn->src1);

            if (arg->ts & T_FLOATING) {
                move(arg, amd64_operand_reg(arg->ts, AMD64_REG_XMM(f)));
                ++f;
            } else
                move(arg, amd64_operand_reg(arg->ts, amd64_iargs[i++]));
        }

        arg_insn = INSNS_NEXT(arg_insn);
    }
}

static void call(struct insn *call_insn, struct amd64_operand *src,
                                         struct amd64_operand *dst)
{
    if (first_arg_insn) {
        call0();
        call1(call_insn);
        call2();
    }

    src = dehuge(src);
    EMIT(insn_new(AMD64_I_CALL(nr_iargs, nr_fargs), src));
    
    if (stack_allocked)
        EMIT(insn_new(AMD64_I_ADDQ,
                      amd64_operand_con(T_LONG, stack_allocked, 0),
                      amd64_operand_reg(T_LONG, AMD64_REG_RSP)));

    if (dst) {
        if (dst->ts & T_FLOATING)
            move(amd64_operand_reg(dst->ts, AMD64_REG_XMM(0)), dst);
        else
            move(amd64_operand_reg(dst->ts, AMD64_REG_RAX), dst);
    }

    first_arg_insn = 0;
    nr_iargs = 0;
    nr_fargs = 0;
    stack_allocked = 0;
}

/* if block b has exactly one predecessor, then
   we're part of the same extended basic block
   and can inherit the predecessor's selxns. */

static void inherit(struct block *b)
{
    struct cessor *pred;

    if (block_nr_predecessors(b) == 1) {
        pred = block_get_predecessor_n(b, 0);
        amd64_selxns_dup(selxns, &pred->b->amd64.selxns);
    }
}

static struct choice com_choices[] =
{
    {   T_CHARS | T_SHORTS | T_INTS,    0,          AMD64_I_NOTL    },
    {   T_LONGS,                        0,          AMD64_I_NOTQ    },

    { 0 }
};

/* generate AMD64 insns for a block */

static void gen0(struct block *b)
{
    struct insn *insn;
    struct amd64_operand *dst;
    struct amd64_operand *src1;
    struct amd64_operand *src2;

    selxns = &b->amd64.selxns;
    inherit(b);

    INSNS_FOREACH(insn, &b->insns) {
        exempt_reg = PSEUDO_REG_NONE;
        insn_normalize(insn);

        dst = amd64_operand_import(insn->dst);
        src1 = amd64_operand_import(insn->src1);
        src2 = amd64_operand_import(insn->src2);

        switch (insn->op)
        {
        case I_FRAME:       frame(src1, dst); break;
        case I_LOAD:        load(src1, dst, insn->flags); break;
        case I_STORE:       store(src1, dst, insn->flags); break;
        case I_BLKCOPY:     blkcopy(src1, src2, dst); break;
        case I_BLKZERO:     blkzero(src1, dst); break;

        case I_ARG:
        case I_VARG:        
        case I_BLKARG:      arg(insn, src1, src2); break;

        case I_CALL:        call(insn, src1, dst); break;

        case I_RETURN:      retrn(src1); break;
        case I_MOVE:        move(src1, dst); break;
        case I_CAST:        cast(src1, dst); break;
        case I_NEG:         neg(src1, dst); break;
        case I_COM:         unary(com_choices, src1, dst); break;

        case I_CMP:         binary(cmp_choices, src1, src2, dst); break;
        case I_ADD:         add(src1, src2, dst); break;
        case I_SUB:         sub(src1, src2, dst); break;
        case I_MUL:         binary(mul_choices, src1, src2, dst); break;
        case I_DIV:         divmod(AMD64_REG_RAX, src1, src2, dst); break;
        case I_MOD:         divmod(AMD64_REG_RDX, src1, src2, dst); break;
        case I_SHR:         shift(shr_choices, src1, src2, dst); break;
        case I_SHL:         shift(shl_choices, src1, src2, dst); break;
        case I_XOR:         binary(xor_choices, src1, src2, dst); break;
        case I_OR:          binary(or_choices, src1, src2, dst); break;
        case I_AND:         binary(and_choices, src1, src2, dst); break;

        case I_SET_Z:       setcc(AMD64_I_SETZ, dst); break;
        case I_SET_NZ:      setcc(AMD64_I_SETNZ, dst); break;
        case I_SET_G:       setcc(AMD64_I_SETG, dst); break;
        case I_SET_LE:      setcc(AMD64_I_SETLE, dst); break;
        case I_SET_GE:      setcc(AMD64_I_SETGE, dst); break;
        case I_SET_L:       setcc(AMD64_I_SETL, dst); break;
        case I_SET_A:       setcc(AMD64_I_SETA, dst); break;
        case I_SET_BE:      setcc(AMD64_I_SETBE, dst); break;
        case I_SET_AE:      setcc(AMD64_I_SETAE, dst); break;
        case I_SET_B:       setcc(AMD64_I_SETB, dst); break;

        case I_TEST:        binary(test_choices, src1, src2, dst); break;
        }

        invalidate(insn);
    }

    insns_clear(&b->insns);
    insns_append(&b->insns, &current_block->insns);
}

/* discard selxn caches */

static blocks_iter_ret gen1(struct block *b)
{
    amd64_selxns_clear(&b->amd64.selxns);
    return BLOCKS_ITER_OK;
}

/* generate AMD64 code for a function. */

void amd64_gen(void)
{
    current_block = block_new();
    blocks_walk(gen0, 0);
    blocks_iter(gen1);
    block_free(current_block);

    /* add an appropriate RET to the exit block 
       now, so any dependency on RAX or XMM0 is
       known to subsequent analysis. */

    current_block = exit_block;

    if (TYPE_DISCRETE(&func_ret_type))
        EMIT(insn_new(AMD64_I_RET_I));
    else if (TYPE_FLOATING(&func_ret_type))
        EMIT(insn_new(AMD64_I_RET_F));
    else
        EMIT(insn_new(AMD64_I_RET));

    /* we generate a lot of dead stores building
       operands, so eliminate them now. */

    dead();

    /* pre-allocation machine-specific optimizations */

    amd64_peep();
    amd64_fuse();
}

/* called by the register allocator to generate spills
   to current_block. spill_in loads the register from its
   spill storage; spill_out does the reverse. (the front
   end has already ensured the symbol has storage.) 

   in the current formulation, this spill code will never
   see static/externs (as they're all aliased), but ... */

static struct amd64_operand *spill0(struct symbol *sym)
{
    struct amd64_operand *o;

    if (sym->ss & (S_STATIC | S_EXTERN))
        o = amd64_operand_sym(sym);
    else
        o = amd64_operand_based(AMD64_REG_RBP, sym->offset);

    AMD64_OPERAND_INDIRECT(o);
    o->ts = TYPE_BASE(&sym->type);
    o->ts = type_machine_bits(o->ts);

    return o;
}

void amd64_gen_spill_in(pseudo_reg reg, struct symbol *sym)
{
    struct amd64_operand *mem;

    mem = spill0(sym);
    apply_flags(choose(load_choices, mem, amd64_operand_reg(mem->ts, reg)),
                INSN_FLAG_SPILL);
}

void amd64_gen_spill_out(pseudo_reg reg, struct symbol *sym)
{
    struct amd64_operand *mem;

    mem = spill0(sym);
    apply_flags(choose(store_choices, amd64_operand_reg(mem->ts, reg), mem),
                INSN_FLAG_SPILL);
}

/* vi: set ts=4 expandtab: */
