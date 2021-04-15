/* target.h - back-end interface                        ncc, the new c compiler

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

#ifndef TARGET_H
#define TARGET_H

#include "cc1.h"
#include "codes.h"
#include "block.h"
#include "insn.h"
#include "type.h"

struct regs;
struct symbol;

/* the target may define several distinct register
   classes for the purposes of allocation */

typedef int pseudo_reg_class;

/* the target structure is the descriptor
   of, and interface to, a back end. */

struct target
{
    char short_align;       /* type geometry */
    char int_align; 
    char long_align;
    char float_align;
    char double_align;

    type_bits ptr_int;      /* pointer -> int mapping for arithmetic */
    type_bits ptr_uint;     /* pointer -> uint mapping for comparisons */

    void (*gen)(void);
    void (*func_new)(void);
    void (*insn_construct)(struct insn *, va_list);
    void (*insn_destruct)(struct insn *);
    struct insn *(*insn_dup)(struct insn *);
    void (*symbol_reg)(struct symbol *);
    void (*symbol_storage)(struct symbol *);
    void (*formal_declare)(struct symbol *);
    void (*output_reg)(pseudo_reg);
    void (*insn_output)(struct insn *);
    void (*insn_branch)(condition_code, struct block *);
    bool (*insn_copy)(struct insn *, pseudo_reg *, pseudo_reg *);
    struct operand *(*insn_con)(struct insn *, pseudo_reg *);
    bool (*insn_substitute_con)(struct insn *, pseudo_reg, struct operand *);

    bool (*insn_substitute_reg)(struct insn *, pseudo_reg, pseudo_reg,
                                insn_substitute_flags);

    bool (*insn_defs_regs)(struct insn *, struct regs *);
    bool (*insn_uses_regs)(struct insn *, struct regs *);
    bool (*insn_defs_cc)(struct insn *);
    ccset (*insn_uses_cc)(struct insn *);
    bool (*insn_defs_mem)(struct insn *);
    bool (*insn_uses_mem)(struct insn *);
    void (*insn_strip_indices)(struct insn *);
    bool (*insn_side_effects)(struct insn *);
    bool (*insn_test_z)(struct insn *, pseudo_reg *);
    bool (*insn_test_con)(struct insn *, pseudo_reg *);
    bool (*insn_defs_z)(struct insn *, pseudo_reg *);
    bool (*reg_physical)(pseudo_reg);
    pseudo_reg_class (*reg_class)(pseudo_reg);
    int (*reg_k)(pseudo_reg);
    void (*reg_colors)(pseudo_reg, struct regs *);
    void (*reg_spills)(pseudo_reg, struct regs *);
    void (*gen_spill_in)(pseudo_reg, struct symbol *);
    void (*gen_spill_out)(pseudo_reg, struct symbol *);
    void (*logues)(void);
    void (*opt)(void);
};

extern struct target *target;       /* the selected target */

extern struct target amd64_target;

#endif /* TARGET_H */

/* vi: set ts=4 expandtab: */
