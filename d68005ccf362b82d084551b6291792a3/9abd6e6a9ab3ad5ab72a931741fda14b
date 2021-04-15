/* insn.h - instructions                                ncc, the new c compiler

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

#ifndef INSN_H
#define INSN_H

#include <limits.h>
#include "../common/tailq.h"
#include "cc1.h"
#include "codes.h"
#include "sched.h"
#include "type.h"
#include "amd64/target_insn.h"

struct tree;
struct live;
struct regs;
struct symbol;

/* machine-independent IR operands are simple: they're either constants
   (which may include a symbol reference in addition to the constant value,
   meaning the address of the symbol offset by the constant), or they're
   pseudo_regs (pseudo-registers at this point). only scalar types remain
   in the IR, so type_bits are used instead of a full-blown type. */

typedef int operand_class;  /* O_* */

#define O_NONE      0       /* no operand here */
#define O_CON       1       /* constant value */
#define O_REG       2       /* pseudo_reg */

/* value numbering assigns numbers directly to the IR operands, see slvn.c */

typedef int value_number;   /* VALUE_NUMBER_* */

#define VALUE_NUMBER_NONE   0

#define VALUE_NUMBER_PRINTF "%d"

struct operand
{
    operand_class class;
    type_bits ts;
    value_number number;
    
    union
    {
        struct                          /* O_CON */
        {
            union con con;
            struct symbol *sym; 
        };

        pseudo_reg reg;                 /* O_REG */
    };
};

#define OPERAND_CLASS(o)        ((o) ? ((o)->class) : O_NONE)
#define OPERAND_REG(o)          ((o) && ((o)->class == O_REG))
#define OPERAND_CON(o)          ((o) && ((o)->class == O_CON))
#define OPERAND_PURE_CON(o)     (OPERAND_CON(o) && ((o)->sym == 0))
#define OPERAND_INTEGRAL(o)     ((o) && ((o)->ts & T_INTEGRAL))
#define OPERAND_UNSIGNED(o)     ((o) && ((o)->ts & T_UNSIGNED))

extern struct operand *operand_dup(struct operand *);
extern struct operand *operand_reg(type_bits, pseudo_reg);
extern struct operand *operand_con(type_bits, union con);
extern struct operand *operand_zero(type_bits);
extern struct operand *operand_one(type_bits);
extern struct operand *operand_f(type_bits, double);
extern struct operand *operand_i(type_bits, long, struct symbol *);
extern struct operand *operand_sym(struct symbol *);
extern struct operand *operand_leaf(struct tree *);
extern bool operand_is_zero(struct operand *);
extern bool operand_is_one(struct operand *);
extern bool operand_is_same(struct operand *, struct operand *);
extern void operand_free(struct operand *);

/* IR instructions start their lives out machine-independent, using the
   insn_ops defined below (and the operands defined above). eventually,
   after machine-independent optimizations, the target code rewrites the
   insns for the target architecture using its own ops and operands. */

typedef int insn_op;    /* I_* */

    /* machine-specific instructions have the high bit set. the
       remaining bits are completely up to the target to define */

#define I_FLAG_TARGET       ( 0x80000000 )
#define I_TARGET(op)        ((op) & I_FLAG_TARGET)

    /* these bits indicate which operands are present
       in the following machine-independent ops */

#define I_FLAG_DST          ( 0x40000000 )
#define I_FLAG_SRC1         ( 0x20000000 )
#define I_FLAG_SRC2         ( 0x10000000 )

    /* if the instruction defs the condition codes */

#define I_FLAG_DEF_CC       ( 0x04000000 )
#define I_DEF_CC(i)         ((i) & I_FLAG_DEF_CC)

    /* if the instruction read from (uses) or writes to (defs) memory */

#define I_FLAG_USE_MEM      ( 0x02000000 )
#define I_USE_MEM(i)        ((i) & I_FLAG_USE_MEM)

#define I_FLAG_DEF_MEM      ( 0x01000000 )
#define I_DEF_MEM(i)        ((i) & I_FLAG_DEF_MEM)

    /* if the instruction is a SETcc */

#define I_FLAG_IS_SET_CC    ( 0x00800000 )
#define I_IS_SET_CC(i)      ((i) & I_FLAG_IS_SET_CC)

    /* if the instruction USEs (instead of DEFs) its dst */

#define I_FLAG_USE_DST      ( 0x00400000 )
#define I_USE_DST(i)        ((i) & I_FLAG_USE_DST)

    /* if the instruction has side effects beyond DEFing its
       dst register, setting condition codes, or stores to mem */

#define I_FLAG_SIDE         ( 0x00200000 )
#define I_SIDE(i)           ((i) & I_FLAG_SIDE)

    /* if the instruction is a commutable operation */

#define I_FLAG_SWAP         ( 0x00100000 )
#define I_SWAP(i)           ((i) & I_FLAG_SWAP)

    /* if the instruction is guaranteed to preserve the condition codes */

#define I_FLAG_SAFE_CC      ( 0x00080000 )
#define I_SAFE_CC(i)        ((i) & I_FLAG_SAFE_CC)

    /* if the instruction is a function argument */

#define I_FLAG_ARGUMENT     ( 0x00040000 )
#define I_ARGUMENT(i)       ((i) & I_FLAG_ARGUMENT)

    /* if the instruction is eligible for value numbering */

#define I_FLAG_SLVN         ( 0x00020000 )
#define I_SLVN(i)           ((i) & I_FLAG_SLVN)

    /* if the instruction is eligible for loop-invariant code motion */

#define I_FLAG_LICM         ( 0x00010000 )
#define I_LICM(i)           ((i) & I_FLAG_LICM)

    /* I_TEXT() is used to index insn_text[] in output.c */

#define I_TEXT(op)      ((op) & 0xFF)

#define I_NOP           (  0 | I_FLAG_SAFE_CC )  /* placeholder, must be 0 */

#define I_FRAME         (  1 | I_FLAG_DST | I_FLAG_SRC1                 \
                             | I_FLAG_SAFE_CC | I_FLAG_SLVN             )

#define I_LOAD          (  2 | I_FLAG_DST | I_FLAG_SRC1                 \
                             | I_FLAG_USE_MEM | I_FLAG_SAFE_CC          \
                             | I_FLAG_SLVN | I_FLAG_LICM                )

#define I_STORE         (  3 | I_FLAG_DST | I_FLAG_SRC1                 \
                             | I_FLAG_USE_DST                           \
                             | I_FLAG_DEF_MEM | I_FLAG_SAFE_CC          )

#define I_BLKCOPY       (  4 | I_FLAG_DST | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_USE_DST                           \
                             | I_FLAG_DEF_MEM                           )

#define I_BLKZERO       (  5 | I_FLAG_DST | I_FLAG_SRC1                 \
                             | I_FLAG_USE_DST                           \
                             | I_FLAG_DEF_MEM                           )

#define I_CALL          (  6 | I_FLAG_DST | I_FLAG_SRC1                 \
                             | I_FLAG_SIDE | I_FLAG_DEF_CC              \
                             | I_FLAG_DEF_MEM | I_FLAG_USE_MEM          )

#define I_ARG           (  7              | I_FLAG_SRC1                 \
                             | I_FLAG_SIDE | I_FLAG_ARGUMENT            )

#define I_VARG          (  8              | I_FLAG_SRC1                 \
                             | I_FLAG_SIDE | I_FLAG_ARGUMENT            )

#define I_BLKARG        (  9              | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_SIDE | I_FLAG_ARGUMENT            )

#define I_RETURN        ( 10              | I_FLAG_SRC1                 \
                             | I_FLAG_SIDE                              ) 

#define I_MOVE          ( 11 | I_FLAG_DST | I_FLAG_SRC1                 \
                             | I_FLAG_SAFE_CC | I_FLAG_SLVN             \
                             | I_FLAG_LICM                              )

#define I_CAST          ( 12 | I_FLAG_DST | I_FLAG_SRC1                 \
                             | I_FLAG_SLVN | I_FLAG_LICM                ) 

#define I_NEG           ( 13 | I_FLAG_DST | I_FLAG_SRC1                 \
                             | I_FLAG_SLVN | I_FLAG_LICM                ) 

#define I_COM           ( 14 | I_FLAG_DST | I_FLAG_SRC1                 \
                             | I_FLAG_SLVN | I_FLAG_LICM                ) 

    /* binary operators with their obvious meanings. for non-commutative
       operators, <src1> is the left operand, and <src2> is the right. */

#define I_CMP           ( 15              | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_DEF_CC                            )

#define I_ADD           ( 16 | I_FLAG_DST | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_SWAP | I_FLAG_SLVN                \
                             | I_FLAG_LICM                              )

#define I_SUB           ( 17 | I_FLAG_DST | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_SLVN | I_FLAG_LICM                )

#define I_MUL           ( 18 | I_FLAG_DST | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_SWAP | I_FLAG_SLVN                \
                             | I_FLAG_LICM                              )

#define I_DIV           ( 19 | I_FLAG_DST | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_SLVN | I_FLAG_LICM                )

#define I_MOD           ( 20 | I_FLAG_DST | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_SLVN | I_FLAG_LICM                )

#define I_SHR           ( 21 | I_FLAG_DST | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_SLVN | I_FLAG_LICM                )

#define I_SHL           ( 22 | I_FLAG_DST | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_SLVN | I_FLAG_LICM                )

#define I_XOR           ( 23 | I_FLAG_DST | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_SWAP | I_FLAG_SLVN                \
                             | I_FLAG_LICM                              )

#define I_OR            ( 24 | I_FLAG_DST | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_SWAP | I_FLAG_SLVN                \
                             | I_FLAG_LICM                              )

#define I_AND           ( 25 | I_FLAG_DST | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_SWAP | I_FLAG_SLVN                \
                             | I_FLAG_LICM                              )

    /* SET_cc sets <dst> to 1 if the condition is satisfied, 0 otherwise.
       order is important: same order as CC_INDEX() to make mapping trivial.
       note that CC_ALWAYS and CC_NEVER do not have equivalent I_SET_CCs. */

#define I_SET_CC_FROM_CC(cc)    (I_SET_Z + CC_INDEX(cc))
#define I_CC_FROM_SET_CC(i)     ((i) - I_SET_Z)

#define I_SET_Z         ( 26 | I_FLAG_DST                                   \
                             | I_FLAG_IS_SET_CC | I_FLAG_SAFE_CC            )

#define I_SET_NZ        ( 27 | I_FLAG_DST                                   \
                             | I_FLAG_IS_SET_CC | I_FLAG_SAFE_CC            )

#define I_SET_G         ( 28 | I_FLAG_DST                                   \
                             | I_FLAG_IS_SET_CC | I_FLAG_SAFE_CC            )

#define I_SET_LE        ( 29 | I_FLAG_DST                                   \
                             | I_FLAG_IS_SET_CC | I_FLAG_SAFE_CC            )

#define I_SET_GE        ( 30 | I_FLAG_DST                                   \
                             | I_FLAG_IS_SET_CC | I_FLAG_SAFE_CC            )

#define I_SET_L         ( 31 | I_FLAG_DST                                   \
                             | I_FLAG_IS_SET_CC | I_FLAG_SAFE_CC            )

#define I_SET_A         ( 32 | I_FLAG_DST                                   \
                             | I_FLAG_IS_SET_CC | I_FLAG_SAFE_CC            )

#define I_SET_BE        ( 33 | I_FLAG_DST                                   \
                             | I_FLAG_IS_SET_CC | I_FLAG_SAFE_CC            )

#define I_SET_AE        ( 34 | I_FLAG_DST                                   \
                             | I_FLAG_IS_SET_CC | I_FLAG_SAFE_CC            )

#define I_SET_B         ( 35 | I_FLAG_DST                                   \
                             | I_FLAG_IS_SET_CC | I_FLAG_SAFE_CC            )

    /* value numbering needs a DEF to map registers to numbers,
       so sometimes we have to fake one. these don't appear in
       actual IR blocks - see slvn.c. */

#define I_NUMBER        ( 36 | I_FLAG_DST )

    /* TEST is an AND instruction that tosses the result and is only
       guaranteed to set the Z condition code based on the result */

#define I_TEST          ( 37              | I_FLAG_SRC1 | I_FLAG_SRC2   \
                             | I_FLAG_DEF_CC                            )

    /* instructions in a block are numbered sequentially, starting at
       INSN_INDEX_FIRST and extending up to (unlikely) INSN_INDEX_LAST.
       these indexes are used for, e.g., describing local live ranges.

       a related quantity is insn_count, a count of instructions. */

typedef unsigned insn_index;

#define INSN_INDEX_PRINTF   "%u"

#define INSN_INDEX_BEFORE   0               /* range starts before block */
#define INSN_INDEX_PHI      1               /* range includes phi headers */
#define INSN_INDEX_FIRST    2
#define INSN_INDEX_LAST     (UINT_MAX - 2)
#define INSN_INDEX_BRANCH   (UINT_MAX - 1)  /* range includes the branch */
#define INSN_INDEX_AFTER    UINT_MAX        /* range extends beyond block */

typedef insn_index insn_count;

    /* instruction-related flags:

       INSN_FLAG_VOLATILE is attached to I_LOAD/I_STORE instructions to
       indicate the access is associated with a volatile pointer, thus it
       can't be eliminated/merged/reordered with respect to other accesses.

       INSN_FLAG_SPILL is attached to spill I_LOAD/I_STOREs. this lets us
       know that these reads/writes don't interact with other accesses. */

typedef int insn_flags;

#define INSN_FLAG_VOLATILE      ( 0x00000001 )
#define INSN_FLAG_SPILL         ( 0x00000002 )

struct insns { TAILQ_HEAD(, insn); insn_index next_index; };

#define INSNS_INIT(is)                                                      \
    do {                                                                    \
        TAILQ_INIT(is);                                                     \
        (is)->next_index = INSN_INDEX_FIRST;                                \
    } while(0)

#define INSNS_COUNT(is)         ((is)->next_index - INSN_INDEX_FIRST)
#define INSNS_FIRST(is)         TAILQ_FIRST(is)
#define INSNS_LAST(is)          TAILQ_LAST(is, insns)
#define INSNS_EMPTY(is)         TAILQ_EMPTY(is)
#define INSNS_PREV(i)           TAILQ_PREV(i, insns, links)
#define INSNS_NEXT(i)           TAILQ_NEXT(i, links)
#define INSNS_FOREACH(i, is)    TAILQ_FOREACH(i, is, links)

typedef TAILQ_ENTRY(insn) insns_entry;

struct insn
{
    insn_op op;
    insn_flags flags;
    insn_index index;

    struct sched sched;     /* insn scheduling data: sched.c */

    union
    {
        struct
        {
            struct operand *dst;
            struct operand *src1;
            struct operand *src2;
        };

        union
        {
            struct amd64_operand *amd64[AMD64_INSN_NR_OPERANDS];
        };
    };

    insns_entry links;
};

extern struct insn *insn_new(insn_op, ...);
extern struct insn *insn_dup(struct insn *);
extern void insn_replace(struct insn *, insn_op, ...);
extern void insn_free(struct insn *);
extern void insn_append(struct insns *, struct insn *);
extern void insn_prepend(struct insns *, struct insn *);
extern void insns_remove(struct insns *, struct insn *);
extern void insns_swap(struct insns *, struct insn *);

extern void insns_insert_before(struct insns *, struct insn *,
                                struct insns *);

extern void insns_insert_after(struct insns *, struct insn *,
                               struct insns *);

extern void insns_append(struct insns *, struct insns *);
extern void insns_push(struct insns *, struct insns *, int);
extern void insns_clear(struct insns *);
extern void insn_commute(struct insn *);
extern void insn_normalize(struct insn *);
extern bool insn_copy(struct insn *, pseudo_reg *, pseudo_reg *);

typedef int insn_defsuses_flags;    /* INSN_DEFSUSES_* */

#define INSN_DEFSUSES_CC    ( 0x00000001 )
#define INSN_DEFSUSES_MEM   ( 0x00000002 )

extern bool insn_defs_cc(struct insn *);
extern ccset insn_uses_cc(struct insn *);
extern bool insn_defs_mem(struct insn *);
extern bool insn_uses_mem(struct insn *);
extern bool insn_defs_regs(struct insn *, struct regs *, insn_defsuses_flags);
extern bool insn_uses_regs(struct insn *, struct regs *, insn_defsuses_flags);
extern void insn_strip_indices(struct insn *);

extern bool insn_side_effects(struct insn *);
extern bool insn_test_z(struct insn *, pseudo_reg *);
extern bool insn_test_con(struct insn *, pseudo_reg *);
extern bool insn_defs_z(struct insn *, pseudo_reg *);
extern struct operand *insn_con(struct insn *, pseudo_reg *);
extern bool insn_substitute_con(struct insn *, pseudo_reg, struct operand *);

typedef int insn_substitute_flags;  /* INSN_SUBSTITUTE_* */

#define INSN_SUBSTITUTE_USES    ( 0x00000001 )
#define INSN_SUBSTITUTE_DEFS    ( 0x00000002 )
#define INSN_SUBSTITUTE_ALL     ( 0x00000003 )

extern bool insn_substitute_reg(struct insn *, pseudo_reg, pseudo_reg,
                                insn_substitute_flags);

#endif /* INSN_H */

/* vi: set ts=4 expandtab: */
