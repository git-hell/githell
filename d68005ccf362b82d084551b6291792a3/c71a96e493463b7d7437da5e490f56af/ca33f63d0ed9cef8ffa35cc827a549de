/* target.c - AMD64 target                              ncc, the new c compiler 

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

#include "../target.h"
#include "amd64.h"
#include "insn.h"
#include "reg.h"
#include "gen.h"
#include "peep.h"

struct target amd64_target =
{
    2,                      /* short_align */
    4,                      /* int_align */
    8,                      /* long_align */
    4,                      /* float_align */
    8,                      /* double_align */
    T_LONG,                 /* ptr_int */
    T_ULONG,                /* ptr_uint */

    amd64_gen,
    amd64_func_new,
    amd64_insn_construct,
    amd64_insn_destruct,
    amd64_insn_dup,
    amd64_symbol_reg,
    amd64_symbol_storage,
    amd64_formal_declare,
    amd64_output_reg,
    amd64_insn_output,
    amd64_insn_branch,
    amd64_insn_copy,
    0 /* insn_con */,
    0 /* insn_substitute_con */,
    amd64_insn_substitute_reg,
    amd64_insn_defs_regs,
    amd64_insn_uses_regs,
    amd64_insn_defs_cc,
    amd64_insn_uses_cc,
    amd64_insn_defs_mem,
    amd64_insn_uses_mem,
    0 /* insn_strip_indices */,
    amd64_insn_side_effects,
    amd64_insn_test_z,
    0 /* insn_test_con */,
    amd64_insn_defs_z,
    amd64_reg_physical,
    amd64_reg_class,
    amd64_reg_k,
    amd64_reg_colors,
    amd64_reg_spills,
    amd64_gen_spill_in,
    amd64_gen_spill_out,
    amd64_logues,
    amd64_opt
};

/* vi: set ts=4 expandtab: */
