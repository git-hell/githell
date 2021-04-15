/* cc1.c - compiler main                                ncc, the new c compiler

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
#include <stdlib.h>
#include <stdarg.h>
#include <string.h>
#include <errno.h>
#include <ctype.h>
#include "../common/util.h"
#include "cc1.h"
#include "opt.h"
#include "lex.h"
#include "string.h"
#include "symbol.h"
#include "output.h"
#include "type.h"
#include "decl.h"

asm_label last_asm_label;       /* for ASM_LABEL_NEW() */

/* compiler debugging is enabled with the -d option (not to
   be confused with enabling debugging symbols with -g). */

bool debug_flag_d;  /* dominators: dump dominators before each block */
bool debug_flag_e;  /* expressions: dump root trees at top of gen() */
bool debug_flag_g;  /* graph allocator: do not run */
bool debug_flag_i;  /* intermediate code: do not invoke target generator */
bool debug_flag_l;  /* live variable analysis: dump before each block */
bool debug_flag_r;  /* reaching definitions: dump before discarding */
bool debug_flag_s;  /* symbols: dump before each function and at end */
bool debug_flag_v;  /* value numbers: show on IR instructions */

/* report a warning or error to the user. using
   a printf()-style format string. if it's FATAL,
   clean up any partial output and abort. */

void error(error_type type, char *fmt, ...)
{
    struct symbol *sym;
    va_list args;

    if (error_path) {
        fprintf(stderr, "%s", error_path->s);
        if (error_line_no) fprintf(stderr, " (%d)", error_line_no);
        fprintf(stderr, ": ");
    } else
        fprintf(stderr, "cc1: ");

    switch (type)
    {
    case WARNING:   fprintf(stderr, "WARNING: "); break;
    case FATAL:     fprintf(stderr, "FATAL: "); break;
    }

    va_start(args, fmt);

    while (*fmt) {
        if (*fmt != '%')
            fputc(*fmt, stderr);
        else {
            ++fmt;

            switch (*fmt)
            {
            case '1':   sym = va_arg(args, struct symbol *);

                        fprintf(stderr, "\n\t\t(%s at %s line %d)",
                                        (sym->ss & S_DEFINED)
                                            ? "defined"
                                            : "first seen",
                                        sym->path->s, sym->line_no);
                        break;

            case 'A':   sym = va_arg(args, struct symbol *);

                        switch (sym->ss & (S_STRUCT | S_UNION | S_ENUM))
                        {
                        case S_STRUCT:  fprintf(stderr, "struct"); break;
                        case S_UNION:   fprintf(stderr, "union"); break;
                        case S_ENUM:    fprintf(stderr, "enum"); break;
                        }

                        if (sym->id) fprintf(stderr, " %s", sym->id->s);
                        break;

            case 'd':   fprintf(stderr, "%d", va_arg(args, int));
                        break;

            case 'E':   fputs(strerror(errno), stderr);
                        break;

            case 'k':   lex_print_k(stderr, va_arg(args, token_class));
                        break;

            case 'S':   fputs(va_arg(args, struct string *)->s, stderr);
                        break;

            case 's':   fputs(va_arg(args, char *), stderr);
                        break;

            case 't':   lex_print_token(stderr, va_arg(args, struct token));
                        break;

            }
        }

        ++fmt;
    }

    va_end(args);
    fputc('\n', stderr);

    if (type > WARNING) {
        output_close(OUTPUT_STATUS_ABORT);
        exit(1);
    }
}

/* give me memory, or give me death */

void *safe_malloc(size_t bytes)
{
    void *p;

    p = malloc(bytes);
    if (p == 0) error(FATAL, "out of memory");
    memset(p, 0, bytes);

    return p;
}

/* entry point. you know the drill: parse the command
   line arguments, initialize state, and then go. */

int main(int argc, char **argv)
{
    char *p;
    int i;

    string_init();
    symbol_init();

    --argc;
    ++argv;

    while (*argv && (**argv == '-')) {
        ++(*argv);

        switch (**argv)
        {
        case 'd':
            p = *argv;

            while (*++p)
            {
                switch (*p)
                {
                case 'd':   debug_flag_d = TRUE; break;
                case 'e':   debug_flag_e = TRUE; break;
                case 'g':   debug_flag_g = TRUE; break;
                case 'i':   debug_flag_i = TRUE; break;
                case 'l':   debug_flag_l = TRUE; break;
                case 'r':   debug_flag_r = TRUE; break;
                case 's':   debug_flag_s = TRUE; break;
                case 'v':   debug_flag_v = TRUE; break;

                default:    goto usage;
                }
            }

            break;

        default:
            goto usage;
        }

        --argc;
        ++argv;
    }

    if (argc != 2) goto usage;
    output_open(argv[1]);
    lex_init(argv[0]);

    translation_unit();

    string_emit_literals();
    symbol_finalize();
    output_close(OUTPUT_STATUS_OK);
    return 0;

usage:
    fprintf(stderr,
        "cc1: invalid command-line syntax"                              "\n"
                                                                        "\n"
        "usage: cc1 {<option>} <input> <output>"                        "\n"
                                                                        "\n"
        "options:"                                                      "\n"
                                                                        "\n"
        "    -d<flags>        compiler debugging options"               "\n"
    );

    return 1;
}

/* vi: set ts=4 expandtab: */
