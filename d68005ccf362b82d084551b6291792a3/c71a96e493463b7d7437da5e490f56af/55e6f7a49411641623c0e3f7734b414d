/* reg.h - AMD64 registers                              ncc, the new c compiler

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

#ifndef AMD64_REG_H
#define AMD64_REG_H

#include "../cc1.h"
#include "../target.h"

struct regs;

extern void amd64_reg_output(pseudo_reg, int);
extern void amd64_output_reg(pseudo_reg);
extern bool amd64_reg_physical(pseudo_reg);

/* for pseudo_reg_class: AMD64 has integer
   and floating-point (XMM/SSE) registers */

#define AMD64_REG_CLASS_INT         0
#define AMD64_REG_CLASS_FP          1

extern pseudo_reg_class amd64_reg_class(pseudo_reg);

/* all registers are available to their respective register
   classes, except of course for the frame and stack pointers,
   and two reserved colors for spilled ranges. */

#define AMD64_NR_ICOLORS            12
#define AMD64_NR_FCOLORS            14

extern int amd64_reg_k(pseudo_reg);
extern void amd64_reg_colors(pseudo_reg, struct regs *);

/* we reserve two registers in each class for spills */

#define AMD64_NR_ISPILLS            2
#define AMD64_NR_FSPILLS            2

extern void amd64_reg_spills(pseudo_reg, struct regs *);

/* the first AMD64_NR_IARGS discrete arguments are passed to
   functions in registers amd64_iargs[0..AMD64_NR_IARGS-1]. */

#define AMD64_NR_IARGS      6

extern pseudo_reg amd64_iargs[];

/* the first AMD64_NR_FARGS floating-point arguments are passed
   to functions in registers AMD64_REG_XMM(0..AMD64_NR_FARGS-1). */

#define AMD64_NR_FARGS      6

/* registers in amd64_itrash[] are the volatile (caller-save) iregs */

#define AMD64_NR_ITRASH     9

extern pseudo_reg amd64_itrash[];

/* the first AMD64_NR_FTRASH XMM registers are caller-save (XMM0-XMM7) */

#define AMD64_NR_FTRASH     8

extern void amd64_regs_remove_trash(struct regs *);

/* AMD64 divides pseudo_reg into two classes, the integer registers and
   the floating-point registers (SSE). the classes are distinguished by
   the AMD64_REG_FLAG_SSE bit. */

#define AMD64_REG_FLAG_SSE      ( 0x00100000 )
#define AMD64_IS_IREG(r)        (!((r) & AMD64_REG_FLAG_SSE))
#define AMD64_IS_FREG(r)        ((r) & AMD64_REG_FLAG_SSE)

/* in each class, bits[19:0] are simple ordinals. the first 16 registers
   correspond to the physical registers, and the rest are pseudo registers. */

#define AMD64_REG_INDEX_MASK    ( 0x000FFFFF )
#define AMD64_REG_INDEX(r)      ((r) & AMD64_REG_INDEX_MASK)

#define AMD64_IREG(i)           (i)
#define AMD64_FREG(i)           ((i) | AMD64_REG_FLAG_SSE)

#define AMD64_REG_PSEUDO        16
#define AMD64_REG_IS_PSEUDO(r)  (AMD64_REG_INDEX(r) >= AMD64_REG_PSEUDO)

/* we name the integer registers. the SSE registers are just numbered.
   the numerical values are significant, as the register allocator will
   assign the lowest-valued available register to a node: put callee-save
   registers at the back of the line. do not change these indexes without
   changing the tables in reg.c. */

#define AMD64_REG_RSI       AMD64_IREG(0)
#define AMD64_REG_RDI       AMD64_IREG(1)
#define AMD64_REG_RAX       AMD64_IREG(2)
#define AMD64_REG_RCX       AMD64_IREG(3)
#define AMD64_REG_RDX       AMD64_IREG(4)
#define AMD64_REG_RBP       AMD64_IREG(5)
#define AMD64_REG_RSP       AMD64_IREG(6)
#define AMD64_REG_R8        AMD64_IREG(7)
#define AMD64_REG_R9        AMD64_IREG(8)
#define AMD64_REG_R10       AMD64_IREG(9)
#define AMD64_REG_R11       AMD64_IREG(10)
#define AMD64_REG_RBX       AMD64_IREG(11)
#define AMD64_REG_R12       AMD64_IREG(12)
#define AMD64_REG_R13       AMD64_IREG(13)
#define AMD64_REG_R14       AMD64_IREG(14)
#define AMD64_REG_R15       AMD64_IREG(15)

#define AMD64_REG_XMM(i)    AMD64_FREG(i)       /* %xmm0 - %xmm15 */

#endif /* AMD64_REG_H */

/* vi: set ts=4 expandtab: */
