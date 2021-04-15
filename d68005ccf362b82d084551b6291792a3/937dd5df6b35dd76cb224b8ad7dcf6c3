/* output.c - output file                               ncc, the new c compiler

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

#include <stdio.h>
#include <stdarg.h>
#include "cc1.h"
#include "dom.h"
#include "opt.h"
#include "type.h"
#include "string.h"
#include "symbol.h"
#include "block.h"
#include "insn.h"
#include "live.h"
#include "regs.h"
#include "blks.h"
#include "loop.h"
#include "target.h"
#include "output.h"

static FILE *out_fp;
static char *out_path;

/* open output file */

void output_open(char *path)
{
    out_fp = fopen(path, "w");

    if (out_fp == 0)
        error(FATAL, "can't open output '%s' (%E)", path);

    out_path = path;
}

/* call before exiting, with either OUTPUT_STATUS_OK (to preserve
   the output) or OUTPUT_STATUS_ABORT (to discard it). */

void output_close(output_status status)
{
    if (out_fp) {
        fclose(out_fp);

        if (status == OUTPUT_STATUS_ABORT)
            remove(out_path);
    }
}

/* output the name of a "global" variable */

static void output_global(struct symbol *sym)
{
    if ((sym->scope > SCOPE_GLOBAL) || (sym->id == 0))
        fprintf(out_fp, ASM_LABEL_PRINTF, sym->label);
    else
        fprintf(out_fp, "_%s", sym->id->s);
}

/* print to the output file using a printf()-style format string. */

void output(char *fmt, ...)
{
    struct symbol *sym;
    struct string *s;
    insn_index i;
    pseudo_reg reg;
    type_bits ts;
    va_list args;
    union con con;
    int idx;

    va_start(args, fmt);

    while (*fmt) {
        if (*fmt != '%')
            putc(*fmt, out_fp);
        else {
            ++fmt;
            switch (*fmt)
            {
            case '%':   fputc('%', out_fp);
                        break;

            case 'c':   fprintf(out_fp, "%c", va_arg(args, int));
                        break;

            case 'd':   fprintf(out_fp, "%d", va_arg(args, int));
                        break;

            case 'f':   fprintf(out_fp, "%f", va_arg(args, double));
                        break;

            case 'g':   sym = va_arg(args, struct symbol *);
                        output_global(sym);
                        break;

            case 'i':   i = va_arg(args, insn_index);
                    
                        switch (i)
                        {
                        case INSN_INDEX_BEFORE: fputs("BEF", out_fp); break;
                        case INSN_INDEX_BRANCH: fputs("BRN", out_fp); break;
                        case INSN_INDEX_AFTER:  fputs("AFT", out_fp); break;

                        default: fprintf(out_fp, INSN_INDEX_PRINTF, i);
                        }
                        
                        break;

            case 'r':   reg = va_arg(args, pseudo_reg);
                        idx = PSEUDO_REG_IDX(reg);
                        reg = PSEUDO_REG_BASE(reg);
                        fputc('$', out_fp);

                        switch (reg)
                        {
                        case PSEUDO_REG_CC:     fputs("CC", out_fp); break;
                        case PSEUDO_REG_NONE:   fputs("NONE", out_fp); break;
                        case PSEUDO_REG_MEM:    fputs("MEM", out_fp); break;
                        default:                target->output_reg(reg);
                        }
        
                        if (idx)
                            fprintf(out_fp, ".%d", idx);

                        break;
    
            case 's':   fputs(va_arg(args, char *), out_fp);
                        break;

            case 't':   ts = va_arg(args, type_bits);
                        type_output_bits(ts);
                        break;

            case 'u':   fprintf(out_fp, "%u", va_arg(args, unsigned));
                        break;

            case 'v':   fprintf(out_fp, VALUE_NUMBER_PRINTF,
                                        va_arg(args, value_number));
                        break;

            case 'x':   fprintf(out_fp, "0x%x", va_arg(args, int));
                        break;

            case 'z':   fprintf(out_fp, "%zu", va_arg(args, size_t));
                        break;

            case 'B':   blks_output(va_arg(args, struct blks *));
                        break;

            case 'C':   ts = va_arg(args, type_bits);
                        con = va_arg(args, union con);

                        if (ts & T_FLOATING)
                            fprintf(out_fp, "%f", con.f);
                        else if (ts & T_SIGNED)
                            fprintf(out_fp, "%ld", con.i);
                        else
                            fprintf(out_fp, "%lu", con.u);
                        
                        break;

            case 'D':   fprintf(out_fp, "%ld", va_arg(args, long));
                        break;

            case 'G':   sym = va_arg(args, struct symbol *);
                        con.i = va_arg(args, long);

                        if (sym) {
                            output_global(sym);
                        
                            if (con.i)
                                fprintf(out_fp, "%+ld", con.i);
                        } else
                            fprintf(out_fp, "%ld", con.i);

                        break;

            case 'L':   fprintf(out_fp, ASM_LABEL_PRINTF,
                            va_arg(args, asm_label));
                        break;

            case 'N':   fprintf(out_fp, SCOPE_LEVEL_PRINTF,
                            va_arg(args, scope_level));
                        break;

            case 'R':   regs_output(va_arg(args, struct regs *));
                        break;

            case 'S':   s = va_arg(args, struct string *);
                        fputs(s->s, out_fp);
                        break;

            case 'T':   type_output(va_arg(args, struct type *));
                        break;

            case 'U':   fprintf(out_fp, "%lu", va_arg(args, unsigned long));
                        break;

            case 'X':   fprintf(out_fp, "0x%lx", va_arg(args, long));
                        break;

            case 'Z':   sym = va_arg(args, struct symbol *);
                        fprintf(out_fp, SYMBOL_NUM_PRINTF, sym->num);

                        if (sym->id)
                            fprintf(out_fp, "(%s)", sym->id->s);

                        break;
            }
        }

        ++fmt;
    }
    
    va_end(args);
}

/* select a new output segment */

void output_select(output_seg new_seg)
{
    static output_seg seg = OUTPUT_SEG_NONE;

    if (new_seg != seg) {
        switch (new_seg)
        {
        case OUTPUT_SEG_TEXT:   fputs(".text\n", out_fp); break;
        case OUTPUT_SEG_DATA:   fputs(".data\n", out_fp); break;
        }
        
        seg = new_seg;
    }
}

/* output operand. used only when debugging,
   since machine-independent operands will not
   occur in actual target code */

static void output_operand(struct operand *opr)
{
    if (debug_flag_v && (opr->number != VALUE_NUMBER_NONE))
        output("@%v ", opr->number);

    output("<%t> ", opr->ts);

    if (opr->class == O_CON) {
        output("%C", opr->ts, opr->con);

        if (opr->sym)
            output("+%Z", opr->sym);
    } else
        output("%r", opr->reg);
}

/* output an instruction. usually this just invokes the target
   insn_output(), but if we're debugging, we'll still have the
   machine-independent instructions to output. */

static char *insn_text[] =          /* indexed by I_TEXT() (see code.h) */
{
    /*  0 */    "NOP",          "FRAME",        "LOAD",         "STORE",
    /*  4 */    "BLKCOPY",      "BLKZERO",      "CALL",         "ARG",
    /*  8 */    "VARG",         "BLKARG",       "RETURN",       "MOVE",
    /* 12 */    "CAST",         "NEG",          "COM",          "CMP",
    /* 16 */    "ADD",          "SUB",          "MUL",          "DIV",
    /* 20 */    "MOD",          "SHR",          "SHL",          "XOR",
    /* 24 */    "OR",           "AND",          "SET_Z",        "SET_NZ",
    /* 28 */    "SET_G",        "SET_LE",       "SET_GE",       "SET_L",
    /* 32 */    "SET_A",        "SET_BE",       "SET_AE",       "SET_B",
    /* 36 */    "NUMBER",       "TEST"
};

static void output_insn(struct insn *insn)
{
    if (insn->op & I_FLAG_TARGET)
        target->insn_output(insn);
    else {
        output("# (%i)\t", insn->index);

        if (insn->dst) {
            output_operand(insn->dst);
            output(" := ");
        }

        output("%s ", insn_text[I_TEXT(insn->op)]);

        if (insn->src1) {
            output_operand(insn->src1);

            if (insn->src2) {
                output(", ");
                output_operand(insn->src2);
            }
        }
    }

    if (insn->flags & INSN_FLAG_VOLATILE)
        output("\t # volatile");

    if (insn->flags & INSN_FLAG_SPILL)
        output("\t # spill");

    output("\n");
}

/* output the current function */

void output_func(void)
{
    struct insn *insn;
    struct block *b;
    struct block *next;
    struct cessor *succ;
    int n;

    output_select(OUTPUT_SEG_TEXT);
    output("%g:\n", func_sym);

    if (debug_flag_s) {
        scope_debug(SCOPE_RETIRED);
        scope_debug(SCOPE_LABEL);
    }

    if (debug_flag_d)
        loop_analyze();

    if (debug_flag_l)
        live_analyze();

    for (b = entry_block; b; b = next) {
        if (debug_flag_d)
            output("# DOMINATORS: %B, loop_depth = %d\n", &b->dominators,
                                                          b->loop.depth);

        if (debug_flag_l)
            live_debug(&b->live);

        output("%L:\n", b->label);

        for (insn = INSNS_FIRST(&b->insns); insn; insn = INSNS_NEXT(insn))
            output_insn(insn);

        next = BLOCKS_NEXT(b);

        for (n = 0; succ = block_get_successor_n(b, n); ++n) {
            switch (succ->cc)
            {
            case CC_SWITCH:
                output("#\tSWITCH[%U,%U] -> %L\n",
                            succ->min,
                            succ->max,
                            succ->b->label);
                break;

            case CC_DEFAULT:
                output("#\tSWITCH ");
                output_operand(b->control);
                output("\n#\tDEFAULT -> %L\n", succ->b->label);
                break;

            default:
                if (succ->b != next)
                    target->insn_branch(succ->cc, succ->b);
            }
        }
    }
}

/* vi: set ts=4 expandtab: */
