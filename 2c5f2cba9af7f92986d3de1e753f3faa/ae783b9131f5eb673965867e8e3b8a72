/* cpp.c - preprocessor main                            ncc, the new c compiler

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
#include "cpp.h"
#include "input.h"
#include "directive.h"
#include "macro.h"

static FILE *out_fp;
static char *out_path;

/* report an error using a printf()-style format string,
   then clean up any partial output and abort. */

void error(char *fmt, ...)
{
    va_list args;

    if (INPUT_STACK) {
        fprintf(stderr, "'%s'", VSTRING_BUF(INPUT_STACK->path));

        if (INPUT_STACK->line_no)
            fprintf(stderr, " (%d)", INPUT_STACK->line_no);
    } else
        fprintf(stderr, "cpp");

    fprintf(stderr, " ERROR: ");
    va_start(args, fmt);
    vfprintf(stderr, fmt, args);
    va_end(args);
    fputc('\n', stderr);

    if (out_fp) {
        fclose(out_fp);
        remove(out_path);
    }

    exit(1);
}

/* simple safe wrapper for malloc.
   allocate or bomb. */

void *safe_malloc(size_t bytes)
{
    void *p;
		
    p = malloc(bytes);
    if (p == 0) error("out of memory");
    return p;
}

/* returns the number of tokens, after the ident, that comprise an
   actual macro argument list. because the opening parenthesis may not
   be on the current line, we might have to read ahead tokens which we
   append to list. if no argument list is found, we return -1 if such
   a readahead occurred, or 0 otherwise. */

static int parentheses(struct list *list, struct token *ident)
{
    int no_args = 0;
    struct token *t;
    int parentheses;
    int count;
    int line_no;

    line_no = INPUT_STACK->line_no;

restart:

    for (;;) {
        t = ident;
        t = list_next(t);
        parentheses = 0;
        count = 0;

        while (t && (t->class == TOKEN_SPACE)) {
            t = list_next(t);
            ++count;
        }

        if (t == 0) {
            if (input_tokens(INPUT_MODE_THIS, list) == -1)
                return no_args;
            
            no_args = -1;
            goto restart;
        }

        if (t->class != TOKEN_LPAREN) return no_args;

        do {
            if (t == 0) {
                if (input_tokens(INPUT_MODE_THIS, list) == -1) {
                    INPUT_STACK->line_no = line_no;
                    error("unterminated macro argument list");
                }

                goto restart;
            }
                
            if (t->class == TOKEN_LPAREN) ++parentheses;
            if (t->class == TOKEN_RPAREN) --parentheses;
            t = list_next(t);
            ++count;
        } while (parentheses);

        return count;
    }
}

/* see if the first token in list is a macro that requires replacement,
   and perform the replacement if so. returns:

        1       a replacement occurred
        0       no replacement occurred
        -1      no replacement occurred, but readahead tokens
                from the next line(s) have been appended to list */

static int replace(struct list *list)
{
    struct list tmp = LIST_INITIALIZER(tmp);
    struct token *t;
    struct macro *m;
    int count = 0;

    t = list_first(list);
    if (t->class != TOKEN_IDENT) return 0;
    m = macro_lookup(&t->u.text);
    if (!m || !MACRO_DEFINED(m)) return 0;

    if (m->flags & MACRO_HAS_ARGS) {
        count = parentheses(list, t);
        if (count <= 0) return count;
    } else
        t = list_next(t);

    list_concat(&tmp, list);
    list_move(list, &tmp, count + 1);
    macro_replace(list);
    list_concat(list, &tmp);

    return 1;
}

/* write the first token of list to the output and discard it.
   a primary job of this function is to insert whitespace where
   necessary to prevent tokens from merging in the output. */

token_class last_class;

static void output(struct list *list)
{
    struct token *t;

    t = list_first(list);
    list_remove(list, t);

    if (t->class & TOKEN_NO_TEXT) 
        error("CPP INTERNAL: attempt to output non-text token %x", t->class);

    if (last_class && token_separate(last_class, t->class))
        fputc(' ', out_fp);

    if (VSTRING_LEN(t->u.text)) {
        fputs(VSTRING_BUF(t->u.text), out_fp);
        last_class = t->class;
    }

    token_free(t);
}

/* synchronize the output file's idea of path/line_no with ours. usually
   this simply emits newlines, but when large (more than SYNC_WINDOW)
   line changes or the path changes, output a #line directive. the flag
   need_sync is a dirty hack; input.c and directive.c set this when the 
   path of the top-of-stack is changed, so we don't have to track it */

char need_sync;

static void sync(void)
{
    static int line_no = 1;
    
    if (need_sync || (line_no > INPUT_STACK->line_no)
      || (line_no < (INPUT_STACK->line_no - SYNC_WINDOW))) {
        line_no = INPUT_STACK->line_no;
        fprintf(out_fp, "\n# %d \"%s\"\n", line_no, 
          VSTRING_BUF(INPUT_STACK->path));
        need_sync = 0;
        last_class = 0;
    }
    
    while (line_no < INPUT_STACK->line_no) {
        fputc('\n', out_fp);
        ++line_no;
        last_class = 0;
    }
}

/* read tokens one line at a time, dispatch any preprocessor directives.
   then copy the tokens to the output, performing macro replacement. this
   otherwise simple loop is complicated by the fact that macro invocations
   may span lines, so we need to do some gymnastics around replacements. */

static void loop(void)
{
    struct list list = LIST_INITIALIZER(list);
    int repl;

    fprintf(out_fp, "# 1 \"%s\"\n", VSTRING_BUF(INPUT_STACK->path));
    need_sync = 0; /* this kludge ensures at least one #line is output */

    for (;;) {
        if (list_empty(&list)
          && (input_tokens(INPUT_MODE_ANY, &list) == -1)) {
            fputc('\n', out_fp);
            return; /* game over */
        }

        while (!list_empty(&list)) {
            directive(&list);
    
            while (!list_empty(&list)) {
                sync();
                repl = replace(&list);
                if (repl == 1) continue;    /* replace again */
                output(&list);
                if (repl == -1) break;      /* check for new directive */
            }
        }
    }
}

/* you know the drill: read the command line arguments
   and set up before heading off to loop() */

int main(int argc, char **argv)
{
    macro_predef();

    --argc;
    ++argv;
    
    while (*argv && (**argv == '-')) {
        switch ((*argv)[1])
        {
        case 'I':
            (*argv) += 2;
            if (**argv == 0) goto usage;
            input_dir(*argv);
            break;
        case 'D':
            (*argv) += 2;
            if (macro_cmdline(*argv) == 0) goto usage;
            break;
            
        default:
            goto usage;
        }

        ++argv;
        --argc;
    }

    if (argc != 2) goto usage;

    out_path = argv[1];
    out_fp = fopen(out_path, "w");
    if (out_fp == 0) error("can't open output '%s'", out_path);

    input_open(argv[0], INPUT_SEARCH_NOWHERE);

    loop();
    fclose(out_fp);
    return 0;

usage:
    fprintf(stderr,
        "cpp: invalid command-line syntax"                              "\n"
                                                                        "\n"
        "usage: cpp {<option>} <input> <output>"                        "\n"
                                                                        "\n"
        "options:"                                                      "\n"
        "   -I<dir>                 add <dir> to system include paths"  "\n"
        "   -D<name>[=<value>]      define macro (default value is 1)"  "\n"
    );

    exit(1);
}

/* vi: set ts=4 expandtab: */
