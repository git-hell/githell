/* directive.c - preprocessor directives                ncc, the new c compiler

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

#include <string.h>
#include "../common/util.h"
#include "../common/slist.h"
#include "cpp.h"
#include "input.h"
#include "macro.h"
#include "evaluate.h"
#include "directive.h"

typedef int directive_index;    /* D_ */

#define D_UNKNOWN   -1
#define D_DEFINE    0
#define D_ELIF      1
#define D_ELSE      2
#define D_ENDIF     3
#define D_ERROR     4
#define D_IF        5
#define D_IFDEF     6
#define D_IFNDEF    7
#define D_INCLUDE   8
#define D_LINE      9
#define D_PRAGMA    10
#define D_UNDEF     11
#define D_EMPTY     12

#define DIRECTIVE_MAX_LEN   7

static const char directives[][DIRECTIVE_MAX_LEN+1] =
{
    { "define" },     { "elif" },       { "else" },     { "endif" },
    { "error" },      { "if" },         { "ifdef" },      { "ifndef" },
    { "include" },    { "line" },       { "pragma" },     { "undef" }
};

static directive_index lookup(struct token *t)
{
    directive_index i;

    if (t->class == TOKEN_IDENT) {
        for (i = 0; i < ARRAY_SIZE(directives); ++i)
            if (!strcmp(directives[i], VSTRING_BUF(t->u.text))) 
                return i;
    }
    
    return D_UNKNOWN;
}

/* #if/#ifdef/etc/#endif state is tracked on a stack. */

struct state
{
    char copying;
    char saw_true;
    char saw_else;

    SLIST_ENTRY(state) link;
};

static SLIST_HEAD(, state) state_stack;

#define STATE           SLIST_FIRST(&state_stack)
#define COPYING         (STATE ? (STATE->copying) : 1)

/* enter a new #if/#ifdef/#ifndef. the copying flag here indicates
   whether the directive has determined its condition to be true. */

static void state_push(int copying)
{
    struct state *st;

    st = safe_malloc(sizeof(struct state));

    if (COPYING) {
        st->saw_true = copying;
        st->copying = copying;
    } else {
        st->copying = 0;
        st->saw_true = 1;
    }

    st->saw_else = 0;
    SLIST_INSERT_HEAD(&state_stack, st, link);
}

static void state_pop(void)
{
    struct state *c;

    c = STATE;
    SLIST_REMOVE_HEAD(&state_stack, link);
    free(c);
}

/* each of the do_*() functions receives a list which
   contains the tokens on the line after the line has
   been cut just beyond the directive itself. */

static void do_ifdef(token_class which, struct list *list)
{
    struct token *ident;
    struct macro *m;
    int defined;

    list_match(list, TOKEN_IDENT, &ident);
    m = macro_lookup(&ident->u.text);
    defined = m && MACRO_DEFINED(m);

    if (which == D_IFDEF)
        state_push(defined);
    else
         state_push(!defined);

    token_free(ident);
}

static void do_if(struct list *list)
{
    if (COPYING)
        state_push(evaluate(list));
    else 
        state_push(0);
}

static void do_elif(struct list *list)
{
    if (!STATE) error("#elif without #if");
    if (STATE->saw_else) error("#elif after #else");

    if (!STATE->saw_true) {
        STATE->saw_true = evaluate(list);
        STATE->copying = STATE->saw_true;
    }
}

static void do_else(struct list *list)
{
    if (!STATE) error("#else without #if");
    if (STATE->saw_else) error("duplicate #else");
    STATE->copying = !STATE->saw_true;
    STATE->saw_else = 1;
}

static void do_endif(void)
{
    if (!STATE) error("#endif without #if");
    state_pop();
}

static void do_line(struct list *list)
{
    struct token *line_no;
    struct token *path = 0;

    list_strip_all(list);
    list_match(list, TOKEN_NUMBER, &line_no);
    if (!list_empty(list)) list_match(list, TOKEN_STRING, &path);
    token_convert_number(line_no);
    INPUT_STACK->line_no = line_no->u.i - 1;

    if (path) {
        vstring_clear(&INPUT_STACK->path);
        vstring_concat(&INPUT_STACK->path, &path->u.text);
        token_free(path);
    }

    token_free(line_no);
    need_sync = 1;
}

static void do_error(struct list *list)
{
    struct token *t;

    t = list_stringize(list);
    error("#error directive: %s", VSTRING_BUF(t->u.text));
}

static void do_include(struct list *list)
{
    struct vstring path = VSTRING_INITIALIZER;
    struct token *t;
    input_search search;

    if (!list_first_is(list, TOKEN_STRING)
      && !list_first_is(list, TOKEN_LT)) {
        macro_replace_all(list);
        list_strip_ends(list);
    }

    if (list_first_is(list, TOKEN_STRING)) {
        list_pop(list, &t);
        token_dequote(t, &path);
        token_free(t);
        search = INPUT_SEARCH_LOCAL;
    } else if (list_first_is(list, TOKEN_LT)) {
        list_pop(list, 0);

        while (!list_empty(list) && !list_first_is(list, TOKEN_GT)) {
            list_pop(list, &t);
            token_text(t, &path);
            token_free(t);
        }

        list_match(list, TOKEN_GT, 0);
        search = INPUT_SEARCH_SYSTEM;
    } else
        error("expected file name after #include");
    
    input_open(VSTRING_BUF(path), search);
    vstring_free(&path);
}

/* called by the main loop after a fresh line; it not only dispatches
   directives, but- this is easy to miss- it also effects conditional
   compilation by emptying the tokens from the line when appropriate. */

void directive(struct list *list)
{
    struct token *t;
    directive_index d;
    int defined;

    t = list_first(list);
    t = list_skip_spaces(t);

    if (t && (t->class == TOKEN_HASH)) {
        t = list_next(t);
        t = list_skip_spaces(t);

        if (t) {
            d = lookup(t);
            t = list_next(t);
        } else
            d = D_EMPTY;

        if (d == D_PRAGMA) goto out;  /* passes through verbatim */

        t = list_skip_spaces(t);
        list_cut(list, t);

        switch (d)
        {
        case D_INCLUDE: do_include(list); break;

        case D_IFDEF:   
        case D_IFNDEF:  do_ifdef(d, list); break;

        case D_IF:      do_if(list); break;
        case D_ELIF:    do_elif(list); break;
        case D_ELSE:    do_else(list); break;
        case D_ENDIF:   do_endif(); break;

        case D_DEFINE:  if (COPYING) macro_define(list); break;
        case D_UNDEF:   if (COPYING) macro_undef(list); break;
        case D_UNKNOWN: if (COPYING) error("unknown directive"); break;

        case D_LINE:    do_line(list); break;
        case D_ERROR:   if (COPYING) do_error(list); break;

        case D_EMPTY:   break;
        }

        if (COPYING && !list_empty(list))
            error("trailing garbage after directive");
    }

out:
    if (!COPYING) list_clear(list);
}

/* called at the end of a file */

void directive_check(void)
{
    if (STATE) error("end-of-file in #if/ifdef/ifndef");
}

/* vi: set ts=4 expandtab: */
