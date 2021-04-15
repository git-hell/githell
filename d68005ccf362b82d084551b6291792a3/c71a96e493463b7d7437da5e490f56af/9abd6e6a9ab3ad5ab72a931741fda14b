/* insn.h - AMD64 instructions                          ncc, the new c compiler

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

#ifndef AMD64_INSN_H
#define AMD64_INSN_H

#include <limits.h>
#include "../cc1.h"
#include "../type.h"
#include "../codes.h"
#include "../insn.h"

struct regs;
struct symbol;

/* AMD64 operands. note that type_bits gives the type as we interpret it,
   not the size of the operand as it is rendered in assembly. the operand
   sizes in the output are dictated by the attached insn_op (since we use
   AT&T syntax.) address sizes are always 64-bit. */

typedef int amd64_operand_class;    /* AMD64_O_* */

    /* AMD64_O_NONE is a place holder */

#define AMD64_O_NONE    0

    /* AMD64_O_REG means the value contained in a register (reg) */

#define AMD64_O_REG     1

    /* AMD64_O_CON is an assembler constant: a global symbol (sym)
       offset by a value (i). the symbol may be (usually is) absent. */

#define AMD64_O_CON     2

    /* AMD64_O_MEM represents the value at the memory location:

                    (reg, idx * scale) + (sym) + (i)
       
       reg, idx, scale, sym can be absent, only i is required. */

#define AMD64_O_MEM     3

    /* AMD64_O_EFF has the same format as AMD64_O_MEM, but its value is
       the effective address rather than the value at that address. the
       assembler output is identical, but AMD64_O_EFF only appears as an
       operand to an LEA instruction. */

#define AMD64_O_EFF     4

struct amd64_operand
{
    amd64_operand_class class;
    type_bits ts;
    pseudo_reg reg;
    pseudo_reg idx;
    int scale;
    struct symbol *sym;
    long i;

    /* when we allocate a new operand, we assign it a sequential number
       which we call its age (though lower numbers are older). this only
       has one use, which is tie-breaking operand combinations during
       instruction selection: see add() in gen.c. */

    int age;
};

extern void amd64_operand_free(struct amd64_operand *);
extern struct amd64_operand *amd64_operand_dup(struct amd64_operand *);
extern struct amd64_operand *amd64_operand_import(struct operand *);
extern struct amd64_operand *amd64_operand_based(pseudo_reg, int);

extern struct amd64_operand *amd64_operand_con(type_bits, long,
                                               struct symbol *);

extern struct amd64_operand *amd64_operand_sym(struct symbol *);
extern struct amd64_operand *amd64_operand_reg(type_bits, pseudo_reg);
extern struct amd64_operand *amd64_operand_scaled(type_bits, pseudo_reg, int);
extern struct amd64_operand *amd64_operand_tmp(type_bits);
extern bool amd64_operand_regs(struct amd64_operand *, struct regs *);
extern bool amd64_operand_small(struct amd64_operand *);
extern bool amd64_operand_huge(struct amd64_operand *);
extern int amd64_operand_degree(struct amd64_operand *);
extern void amd64_operand_normalize(struct amd64_operand *);

extern struct amd64_operand *amd64_operands_combine(struct amd64_operand *,
                                                    struct amd64_operand *);

extern bool amd64_operands_same(struct amd64_operand *,
                               struct amd64_operand *);

#define AMD64_OPERAND_CON(o)        ((o) && ((o)->class == AMD64_O_CON))
#define AMD64_OPERAND_REG(o)        ((o) && ((o)->class == AMD64_O_REG))
#define AMD64_OPERAND_EFF(o)        ((o) && ((o)->class == AMD64_O_EFF))
#define AMD64_OPERAND_MEM(o)        ((o) && ((o)->class == AMD64_O_MEM))

#define AMD64_OPERAND_PURE_CON(o) (AMD64_OPERAND_CON(o) && ((o)->sym == 0))
#define AMD64_OPERAND_ZERO(o)     (AMD64_OPERAND_PURE_CON(o) && ((o)->i == 0))

#define AMD64_HUGE_CON(i)   (((i) < INT_MIN) || ((i) > INT_MAX))

#define AMD64_OPERANDS_SAME_REG(o1, o2)     (AMD64_OPERAND_REG(o1) &&       \
                                             AMD64_OPERAND_REG(o2) &&       \
                                             ((o1)->reg == (o2)->reg))

#define AMD64_OPERAND_REFS_REG(o, r)  (((o)->reg == (r)) || ((o)->idx == (r)))

/* perform indirection on the operand */

#define AMD64_OPERAND_INDIRECT(o)   ((o)->class = AMD64_O_MEM)

/* each insn has an index into amd64_insn_text[] (in amd64/output.c) */

#define AMD64_I_INDEX_MASK      ( 0x0000007F )              /* bits[6:0] */
#define AMD64_I_INDEX(i)        ((i) & AMD64_I_INDEX_MASK)

/* each AMD64 instruction has up to two operands, the sizes of which
   are inherent to the instruction itself, since we use AT&T syntax. */

#define AMD64_SIZE_BYTE         0           /* 1 byte */
#define AMD64_SIZE_WORD         1           /* 2 bytes */
#define AMD64_SIZE_DWORD        2           /* 4 bytes */
#define AMD64_SIZE_QWORD        3           /* 8 bytes */
#define AMD64_SIZE_OWORD        4           /* 16 bytes - XMM only */

#define AMD64_I_OPERANDS_MASK       ( 0x00000003 )          /* bits[8:7] */
#define AMD64_I_OPERANDS_SHIFT      7

/* the AMD64_SIZE_* of the first operand is encoded
   in bits[10:9], the second in bits[12:11]. */

#define AMD64_I_SIZE_MASK           ( 0x00000003 )          /* bits[12:9] */
#define AMD64_I_SIZE_SHIFT          9
#define AMD64_I_SIZE_SHIFT_N(n)     (AMD64_I_SIZE_SHIFT + ((n) * 2))

/* bits 13 and 14 indicate whether operand
   0 or 1 (respectively) is modified by insn */

#define AMD64_I_DEFS_MASK           ( 0x00000001 )          /* bits[14:13] */
#define AMD64_I_DEFS_SHIFT          13
#define AMD64_I_DEFS_SHIFT_N(n)     (AMD64_I_DEFS_SHIFT + (n))

/* and bits 15 and 16 indicate whether
   operand 0 or 1 is read by an insn */

#define AMD64_I_USES_MASK           ( 0x00000001 )          /* bits[16:15] */
#define AMD64_I_USES_SHIFT          15
#define AMD64_I_USES_SHIFT_N(n)     (AMD64_I_USES_SHIFT + (n))

/* bits[19:17] encode the number of integer
   arguments for AMD64_I_CALL (max 7) */

#define AMD64_I_IARGS_MASK          ( 0x00000007 )          /* bits[19:17] */
#define AMD64_I_IARGS_SHIFT         17

/* bits[22:20] encode the number of
   floating-point arguments (max 7) */

#define AMD64_I_FARGS_MASK          ( 0x00000007 )          /* bits[22:20] */
#define AMD64_I_FARGS_SHIFT         20

/* bit 23 is on when the insn sets the Z flag
   properly if the register DEFd is zero */

#define AMD64_I_FLAG_DEF_Z          ( 0x00800000 )
#define AMD64_I_DEF_Z(i)            ((i) & AMD64_I_FLAG_DEF_Z)

/* and bits 24 and 25 indicate whether
   operand 0 or 1 can be a memory reference */

#define AMD64_I_FUSE_MASK           ( 0x00000001 )          /* bits[25:24] */
#define AMD64_I_FUSE_SHIFT          24
#define AMD64_I_FUSE_SHIFT_N(n)     (AMD64_I_FUSE_SHIFT + (n))

/* if the insn is modifies the condition codes */

#define AMD64_I_FLAG_DEFCC          ( 0x04000000 )          /* bit[26] */

/* if the insn is dependent on the condition codes */

#define AMD64_I_FLAG_USECC          ( 0x08000000 )          /* bit[27] */

/* if the insn has side effects */

#define AMD64_I_FLAG_SIDE           ( 0x10000000 )          /* bit[28] */

/* if the insn defs mem */

#define AMD64_I_FLAG_DEFMEM         ( 0x20000000 )          /* bit[29] */

/* if the insn uses mem */

#define AMD64_I_FLAG_USEMEM         ( 0x40000000 )          /* bit[30] */

/* use to query how many operands an insn_op has */

#define AMD64_I_OPERANDS(i)         (((i) >> AMD64_I_OPERANDS_SHIFT)        \
                                        & AMD64_I_OPERANDS_MASK)

/* use to encode an insn that has n operands */

#define AMD64_I_ENC_OPERANDS(n)     ((n) << AMD64_I_OPERANDS_SHIFT)

/* use to query the size of operand n */

#define AMD64_I_SIZE(i, n)          (((i) >> AMD64_I_SIZE_SHIFT_N(n))      \
                                            & AMD64_I_SIZE_MASK)

/* use to encode the size of operand n in an insn */

#define AMD64_I_ENC_SIZE(n, s)      ((s) << AMD64_I_SIZE_SHIFT_N(n))

/* use to query whether an operand is DEFd or USEd (or both) */

#define AMD64_I_DEFS(i, n)          (((i) >> AMD64_I_DEFS_SHIFT_N(n))       \
                                            & AMD64_I_DEFS_MASK)

#define AMD64_I_USES(i, n)          (((i) >> AMD64_I_USES_SHIFT_N(n))       \
                                            & AMD64_I_USES_MASK)

/* use to encode where operand n is DEFd or USEd (or both) */

#define AMD64_I_ENC_DEFS(n)         (1 << AMD64_I_DEFS_SHIFT_N(n))
#define AMD64_I_ENC_USES(n)         (1 << AMD64_I_USES_SHIFT_N(n))

/* use to query whether an operand can be replaced with a memory reference */

#define AMD64_I_FUSE(i, n)          (((i) >> AMD64_I_FUSE_SHIFT_N(n))       \
                                            & AMD64_I_FUSE_MASK)

/* use to encode whether an operand can be replaced with a memory reference */

#define AMD64_I_ENC_FUSE(n)         (1 << AMD64_I_FUSE_SHIFT_N(n))

/* use to query how many integer arguments an AMD64_I_CALL has */

#define AMD64_I_IARGS(i)            (((i) >> AMD64_I_IARGS_SHIFT)           \
                                        & AMD64_I_IARGS_MASK)

/* use to encode how many integer arguments */

#define AMD64_I_ENC_IARGS(n)        ((n) << AMD64_I_IARGS_SHIFT)

/* use to query how many floating-point arguments an AMD64_I_CALL has */

#define AMD64_I_FARGS(i)            (((i) >> AMD64_I_FARGS_SHIFT)           \
                                        & AMD64_I_FARGS_MASK)

/* use to encode how many floating-point arguments */

#define AMD64_I_ENC_FARGS(n)        ((n) << AMD64_I_FARGS_SHIFT)

/* AMD64 insn_ops */

#define AMD64_I(i)              ((i) | I_FLAG_TARGET)

#define AMD64_I_MOVB        AMD64_I ( 0                                     \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_MOVW        AMD64_I ( 1                                     \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_WORD)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_WORD)  \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_MOVL        AMD64_I ( 2                                     \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_MOVQ        AMD64_I ( 3                                     \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_MOVSBL      AMD64_I ( 4                                     \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_MOVZBL      AMD64_I ( 5                                     \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_MOVSBQ      AMD64_I ( 6                                     \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_MOVZBQ      AMD64_I ( 7                                     \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_MOVSWL      AMD64_I ( 8                                     \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_WORD)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_MOVZWL      AMD64_I ( 9                                     \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_WORD)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_MOVSWQ      AMD64_I ( 10                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_WORD)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_MOVZWQ      AMD64_I ( 11                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_WORD)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_MOVSLQ      AMD64_I ( 12                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

        /* movzlq isn't real (it comes out as movl), but we distinguish it
           here so it doesn't look like just a register-register copy */

#define AMD64_I_MOVZLQ      AMD64_I ( 13                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_MOVSS       AMD64_I ( 14                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_MOVSD       AMD64_I ( 15                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_CVTSI2SSL   AMD64_I ( 16                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_CVTSI2SSQ   AMD64_I ( 17                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_CVTSI2SDL   AMD64_I ( 18                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_CVTSI2SDQ   AMD64_I ( 19                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_CVTSS2SIL   AMD64_I ( 20                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_CVTSS2SIQ   AMD64_I ( 21                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_CVTSD2SIL   AMD64_I ( 22                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_CVTSD2SIQ   AMD64_I ( 23                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_CVTSS2SD    AMD64_I ( 24                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_CVTSD2SS    AMD64_I ( 25                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

    /* synthetic instruction to load a 0 integral operand.
       these are output as self-XORs, but we can't use XOR
       because those appear to USE the register value. */

#define AMD64_I_ZERO        AMD64_I ( 26                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_PUSHQ       AMD64_I ( 27                                    \
                                    | AMD64_I_FLAG_SIDE                     \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   )

#define AMD64_I_POPQ        AMD64_I ( 28                                    \
                                    | AMD64_I_FLAG_SIDE                     \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_LEAL        AMD64_I ( 29                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_LEAQ        AMD64_I ( 30                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_SETZ        AMD64_I ( 31                                    \
                                    | AMD64_I_FLAG_USECC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_SETNZ       AMD64_I ( 32                                    \
                                    | AMD64_I_FLAG_USECC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_SETG        AMD64_I ( 33                                    \
                                    | AMD64_I_FLAG_USECC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_SETLE       AMD64_I ( 34                                    \
                                    | AMD64_I_FLAG_USECC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_SETGE       AMD64_I ( 35                                    \
                                    | AMD64_I_FLAG_USECC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_SETL        AMD64_I ( 36                                    \
                                    | AMD64_I_FLAG_USECC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_SETA        AMD64_I ( 37                                    \
                                    | AMD64_I_FLAG_USECC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_SETBE       AMD64_I ( 38                                    \
                                    | AMD64_I_FLAG_USECC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_SETAE       AMD64_I ( 39                                    \
                                    | AMD64_I_FLAG_USECC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_SETB        AMD64_I ( 40                                    \
                                    | AMD64_I_FLAG_USECC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_DEFS(0)                   )

    /* we use three versions of RET. the difference is in the dependencies:
       RET_I depends on RAX, RET_F on XMM0, and RET depends on nothing. */

#define AMD64_I_RET         AMD64_I ( 41 | AMD64_I_FLAG_SIDE )
#define AMD64_I_RET_I       AMD64_I ( 42 | AMD64_I_FLAG_SIDE )
#define AMD64_I_RET_F       AMD64_I ( 43 | AMD64_I_FLAG_SIDE )

#define AMD64_I_SHLL        AMD64_I ( 44                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_SHLQ        AMD64_I ( 45                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_SHRL        AMD64_I ( 46                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_SHRQ        AMD64_I ( 47                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_SARL        AMD64_I ( 48                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_SARQ        AMD64_I ( 49                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_ADDL        AMD64_I ( 50                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_ADDQ        AMD64_I ( 51                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_ADDSS       AMD64_I ( 52                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_ADDSD       AMD64_I ( 53                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_SUBL        AMD64_I ( 54                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_SUBQ        AMD64_I ( 55                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_SUBSS       AMD64_I ( 56                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_SUBSD       AMD64_I ( 57                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_ANDL        AMD64_I ( 58                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_ANDQ        AMD64_I ( 59                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_ORL         AMD64_I ( 60                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_ORQ         AMD64_I ( 61                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_XORL        AMD64_I ( 62                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_XORQ        AMD64_I ( 63                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_FLAG_DEF_Z                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_CMPL        AMD64_I ( 64                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_CMPQ        AMD64_I ( 65                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_UCOMISS     AMD64_I ( 66                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   )

#define AMD64_I_UCOMISD     AMD64_I ( 67                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   )

#define AMD64_I_NOTL        AMD64_I ( 68                                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_NOTQ        AMD64_I ( 69                                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_NEGL        AMD64_I ( 70                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_NEGQ        AMD64_I ( 71                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_DEFS(0)                   )

#define AMD64_I_MULSS       AMD64_I ( 72                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_MULSD       AMD64_I ( 73                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_DIVSS       AMD64_I ( 74                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_DIVSD       AMD64_I ( 75                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_IMULL       AMD64_I ( 76                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_IMULQ       AMD64_I ( 77                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_DIVL        AMD64_I ( 78                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   )

#define AMD64_I_DIVQ        AMD64_I ( 79                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   )

#define AMD64_I_IDIVL       AMD64_I ( 80                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   )

#define AMD64_I_IDIVQ       AMD64_I ( 81                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   )

#define AMD64_I_CLTD        AMD64_I ( 82 )
#define AMD64_I_CQTO        AMD64_I ( 83 )
#define AMD64_I_REP         AMD64_I ( 84 | AMD64_I_FLAG_SIDE )

#define AMD64_I_MOVSB       AMD64_I ( 85 | AMD64_I_FLAG_DEFMEM              \
                                         | AMD64_I_FLAG_USEMEM              )

#define AMD64_I_STOSB       AMD64_I ( 86 | AMD64_I_FLAG_DEFMEM )

    /* AMD64_I_CALL is a family of instructions, each of which
       encodes the number of integer and floating-point arguments.
       this is necessary so analysis can tell which registers are
       actually used by the called function. */

#define AMD64_I_IS_CALL(op)     (AMD64_I_INDEX(op) ==                       \
                                 AMD64_I_INDEX(AMD64_I_CALL(0, 0)))

#define AMD64_I_CALL(iargs, fargs)                                          \
                            AMD64_I ( 87                                    \
                                    | AMD64_I_FLAG_USEMEM                   \
                                    | AMD64_I_FLAG_DEFMEM                   \
                                    | AMD64_I_FLAG_SIDE                     \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(1)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_IARGS(iargs)              \
                                    | AMD64_I_ENC_FARGS(fargs)              )

#define AMD64_I_ADDB        AMD64_I ( 88                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_SUBB        AMD64_I ( 89                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_BYTE)  \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_ADDW        AMD64_I ( 90                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_WORD)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_WORD)  \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_SUBW        AMD64_I ( 91                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_WORD)  \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_WORD)  \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_TESTL       AMD64_I ( 92                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_DWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_TESTQ       AMD64_I ( 93                                    \
                                    | AMD64_I_FLAG_DEFCC                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_FUSE(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_QWORD) \
                                    | AMD64_I_ENC_USES(1)                   \
                                    | AMD64_I_ENC_FUSE(1)                   )

#define AMD64_I_STOSQ       AMD64_I ( 94 | AMD64_I_FLAG_DEFMEM )

    /* just like AMD64_I_ZERO, except for an XMM register. */

#define AMD64_I_XMMZERO     AMD64_I ( 95                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_OWORD) \
                                    | AMD64_I_ENC_DEFS(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_OWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_MOVUPS      AMD64_I ( 96                                    \
                                    | AMD64_I_ENC_OPERANDS(2)               \
                                    | AMD64_I_ENC_SIZE(0, AMD64_SIZE_OWORD) \
                                    | AMD64_I_ENC_USES(0)                   \
                                    | AMD64_I_ENC_SIZE(1, AMD64_SIZE_OWORD) \
                                    | AMD64_I_ENC_DEFS(1)                   )

#define AMD64_I_MOVSW       AMD64_I ( 97 | AMD64_I_FLAG_DEFMEM              \
                                         | AMD64_I_FLAG_USEMEM              )

#define AMD64_I_MOVSL       AMD64_I ( 98 | AMD64_I_FLAG_DEFMEM              \
                                         | AMD64_I_FLAG_USEMEM              )

#define AMD64_I_MOVSQ       AMD64_I ( 99 | AMD64_I_FLAG_DEFMEM              \
                                         | AMD64_I_FLAG_USEMEM              )

extern struct insn *amd64_insn_dup(struct insn *);
extern void amd64_insn_construct(struct insn *, va_list);
extern void amd64_insn_destruct(struct insn *);
extern void amd64_insn_output(struct insn *);
extern bool amd64_insn_defs_cc(struct insn *);
extern ccset amd64_insn_uses_cc(struct insn *);
extern bool amd64_insn_side_effects(struct insn *);
extern bool amd64_insn_test_z(struct insn *, pseudo_reg *);
extern bool amd64_insn_defs_z(struct insn *, pseudo_reg *);
extern bool amd64_insn_defs_mem(struct insn *);
extern bool amd64_insn_uses_mem(struct insn *);
extern bool amd64_insn_defs_regs(struct insn *, struct regs *);
extern bool amd64_insn_uses_regs(struct insn *, struct regs *);
extern bool amd64_insn_copy(struct insn *, pseudo_reg *, pseudo_reg *);

extern bool amd64_insn_substitute_reg(struct insn *, pseudo_reg,
                                      pseudo_reg, insn_substitute_flags);

#endif /* AMD64_INSN_H */

/* vi: set ts=4 expandtab: */
