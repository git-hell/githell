/* reg.c - AMD64 registers                              ncc, the new c compiler

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
#include "../output.h"
#include "../target.h"
#include "../regs.h"
#include "insn.h"
#include "reg.h"

pseudo_reg amd64_iargs[AMD64_NR_IARGS] =
{
    AMD64_REG_RDI, AMD64_REG_RSI, AMD64_REG_RDX,
    AMD64_REG_RCX, AMD64_REG_R8,  AMD64_REG_R9
};

pseudo_reg amd64_itrash[AMD64_NR_ITRASH] =
{
    AMD64_REG_RAX, AMD64_REG_RCX, AMD64_REG_RDX, AMD64_REG_RSI,
    AMD64_REG_RDI, AMD64_REG_R8,  AMD64_REG_R9,  AMD64_REG_R10,
    AMD64_REG_R11
};

pseudo_reg amd64_icolors[AMD64_NR_ICOLORS] =
{
    AMD64_REG_RAX, AMD64_REG_RCX, AMD64_REG_RDX, AMD64_REG_RSI,
    AMD64_REG_RDI, AMD64_REG_R8,  AMD64_REG_R9,  AMD64_REG_RBX,
    AMD64_REG_R12, AMD64_REG_R13, AMD64_REG_R14, AMD64_REG_R15
};

pseudo_reg amd64_ispills[AMD64_NR_ISPILLS] =
{
    AMD64_REG_R10, AMD64_REG_R11
};

pseudo_reg amd64_fspills[AMD64_NR_FSPILLS] =
{
    AMD64_REG_XMM(6), AMD64_REG_XMM(7)
};

void amd64_regs_remove_trash(struct regs *regs)
{
    int i;

    for (i = 0; i < AMD64_NR_ITRASH; ++i)
        regs_remove(regs, amd64_itrash[i]);

    for (i = 0; i < AMD64_NR_FTRASH; ++i)
        regs_remove(regs, AMD64_REG_XMM(i));
}

bool amd64_reg_physical(pseudo_reg reg)
{
    if (AMD64_REG_IS_PSEUDO(reg))
        return FALSE;
    else
        return TRUE;
}

pseudo_reg_class amd64_reg_class(pseudo_reg reg)
{
    if (AMD64_IS_IREG(reg))
        return AMD64_REG_CLASS_INT;
    else
        return AMD64_REG_CLASS_FP;
}

int amd64_reg_k(pseudo_reg reg)
{
    if (AMD64_IS_IREG(reg))
        return AMD64_NR_ICOLORS;
    else
        return AMD64_NR_FCOLORS;
}

#define REGS0(array, n, regs)                                           \
    do {                                                                \
        int i;                                                          \
                                                                        \
        for (i = 0; i < (n); ++i)                                       \
            REGS_ADD((regs), (array)[i]);                               \
                                                                        \
    } while(0)

void amd64_reg_colors(pseudo_reg reg, struct regs *regs)
{
    int i;

    if (AMD64_IS_IREG(reg))
        REGS0(amd64_icolors, AMD64_NR_ICOLORS, regs);
    else
        for (i = 0; i < AMD64_NR_FCOLORS; ++i)
            REGS_ADD(regs, AMD64_REG_XMM(i));
}

void amd64_reg_spills(pseudo_reg reg, struct regs *regs)
{
    if (AMD64_IS_IREG(reg))
        REGS0(amd64_ispills, AMD64_NR_ISPILLS, regs);
    else
        REGS0(amd64_fspills, AMD64_NR_FSPILLS, regs);
}

/* these tables are keyed to the AMD64_REG_* values
   defined in reg.h, so keep in sync with those */

static char *byte_iregs[] =
{
    "sil",  "dil",  "al",   "cl",   "dl",   0,      0,      "r8b",
    "r9b",  "r10b", "r11b", "bl",   "r12b", "r13b", "r14b", "r15b"
};

static char *word_iregs[] = 
{
    "si",   "di",   "ax",   "cx",   "dx",   0,      0,      "r8w",
    "r9w",  "r10w", "r11w", "bx",   "r12w", "r13w", "r14w", "r15w"
};

static char *dword_iregs[] = 
{
    "esi",  "edi",  "eax",  "ecx",  "edx",  0,      0,      "r8d",
    "r9d",  "r10d", "r11d", "ebx",  "r12d", "r13d", "r14d", "r15d"
};

static char *qword_iregs[] =
{
    "rsi",  "rdi",  "rax",  "rcx",  "rdx",  "rbp",  "rsp",  "r8",
    "r9",   "r10",  "r11",  "rbx",  "r12",  "r13",  "r14",  "r15"
};

/* output an AMD64 register in assembler format.
   size is one of the AMD64_SIZE_* constants.

   this is not to be confused with amd64_output_reg(),
   which is for outputting pseudo-registers in target-
   independent insns. for debugging purposes, we still
   handle pseudo-registers here (albeit differently) 
   in case we do this before register allocation. */

void amd64_reg_output(pseudo_reg reg, int size)
{
    char **iregs;

    output("%%");

    if (AMD64_REG_IS_PSEUDO(reg)) {
        output(AMD64_IS_IREG(reg) ? "i" : "f");
        output("%d", AMD64_REG_INDEX(reg));

        switch (size)
        {
        case AMD64_SIZE_BYTE:       output("b"); break;
        case AMD64_SIZE_WORD:       output("w"); break;
        case AMD64_SIZE_DWORD:      output("d"); break;
        case AMD64_SIZE_QWORD:      output("q"); break;
        }
    } else {
        if (AMD64_IS_IREG(reg)) {
            switch (size)
            {
            case AMD64_SIZE_BYTE:       iregs = byte_iregs; break;
            case AMD64_SIZE_WORD:       iregs = word_iregs; break;
            case AMD64_SIZE_DWORD:      iregs = dword_iregs; break;
            case AMD64_SIZE_QWORD:      iregs = qword_iregs; break;
            }

            output("%s", iregs[AMD64_REG_INDEX(reg)]);
        } else
            output("xmm%d", AMD64_REG_INDEX(reg));
    }

    if (PSEUDO_REG_IDX(reg))
        output(".%d", PSEUDO_REG_IDX(reg));
}

/* output a register in human-readable format.
   this is for debugging output, not assembly. */

void amd64_output_reg(pseudo_reg reg)
{
    if (AMD64_REG_IS_PSEUDO(reg))
        output("%c%d", AMD64_IS_FREG(reg) ? 'F' : 'I',
                       AMD64_REG_INDEX(reg));
    else {
        if (AMD64_IS_IREG(reg))
            output(qword_iregs[AMD64_REG_INDEX(reg)]);
        else
            output("xmm%d", AMD64_REG_INDEX(reg));
    }
}

/* vi: set ts=4 expandtab: */
