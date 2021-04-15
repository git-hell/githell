/* macro.c - preprocessor macro table                   ncc, the new c compiler

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

#include <time.h>
#include "../common/util.h"
#include "../common/tailq.h"
#include "cpp.h"
#include "token.h"
#include "vstring.h"
#include "macro.h"
#include "input.h"

/* macros are kept in a simple hash table of NR_MACRO_BUCKETS buckets.
   predefined macros are kept here, too, but marked for special handling.
   'defined' is an oddball here, as it's not really a predefined macro,
   but it's convenient to mark it as such. */

static TAILQ_HEAD(bucket, macro) buckets[NR_MACRO_BUCKETS];

/* this isn't a fabulous hash routine, but it will do for now */

static unsigned hash(struct vstring *vs)
{
    unsigned hash = 0;
    char *s = VSTRING_BUF(*vs);

    while (*s) hash = (hash << 3) ^ *s++;
    return hash;
}

#define BUCKET(name)    (hash(name) % NR_MACRO_BUCKETS)

/* create a new entry in the macro table with the specified name.
   it is the caller's responsibility to ensure it's not a dup. */

static struct macro *insert(struct vstring *name)
{
    int b = BUCKET(name);
    struct macro *m;

    m = safe_malloc(sizeof(struct macro));
    vstring_init(&m->name);
    vstring_concat(&m->name, name);
    list_init(&m->formals);
    list_init(&m->tokens);
    m->flags = 0;
    TAILQ_INSERT_HEAD(&buckets[b], m, links);

    return m;
}

/* seed the macro table with the predefined ones */

static struct { char *name; macro_flags flags; } predefs[] =
{
    { "__STDC__",       MACRO_PREDEF_STDC },
    { "__LINE__",       MACRO_PREDEF_LINE },
    { "__FILE__",       MACRO_PREDEF_FILE },
    { "__DATE__",       MACRO_PREDEF_DATE },
    { "__TIME__",       MACRO_PREDEF_TIME },
    { "defined",        MACRO_PREDEF_DEFINED }
};

void macro_predef(void)
{
    struct vstring vs = VSTRING_INITIALIZER;
    struct macro *m;
    struct token *t;
    int i;

    for (i = 0; i < ARRAY_SIZE(predefs); ++i) {
        vstring_clear(&vs);
        vstring_puts(&vs, predefs[i].name);
        m = insert(&vs);
        m->flags = predefs[i].flags;
        t = 0;
    
        switch (m->flags)
        {
        case MACRO_PREDEF_STDC:
            t = token_number(1);
            break;
        }

        if (t) list_append(&m->tokens, t);
    }

    vstring_free(&vs);
}


/* get an up-to-date copy of the macro's replacement list */

static void tokens(struct macro *m, struct list *dst)
{
    time_t epoch;
    struct token *t;
    char buf[64];

    switch (m->flags & MACRO_PREDEF_MASK)
    {
    case MACRO_PREDEF_LINE:
        t = token_number(INPUT_STACK->line_no);
        break;

    case MACRO_PREDEF_FILE:
        t = token_string(VSTRING_BUF(INPUT_STACK->path));
        break;

    case MACRO_PREDEF_DATE:
        if (list_empty(&m->tokens)) {
            time(&epoch);
            strftime(buf, sizeof(buf), "%b %d %Y", localtime(&epoch));
            t = token_string(buf);
            list_append(&m->tokens, t);
        }

    case MACRO_PREDEF_TIME:
        if (list_empty(&m->tokens)) {
            time(&epoch);
            strftime(buf, sizeof(buf), "%H:%M:%S", localtime(&epoch));
            t = token_string(buf);
            list_append(&m->tokens, t);
        }

    default:
        list_copy(dst, &m->tokens);
        return;
    }

    list_append(dst, t);
}

/* look up a macro in the hash table, and return its entry,
   or 0 if non-existent. the entry is shuffled to the front
   of its bucket to speed subsequent access. */

struct macro *macro_lookup(struct vstring *name)
{
    struct macro *m;
    int b = BUCKET(name);

    for (m = TAILQ_FIRST(&buckets[b]); m; m = TAILQ_NEXT(m, links)) {
        if (vstring_same(&m->name, name)) {
            TAILQ_REMOVE(&buckets[b], m, links);
            TAILQ_INSERT_HEAD(&buckets[b], m, links);
            return m;
        }
    }

    return 0;
}

/* handle a #define directive; the list holds the tokens on the line
   past the 'define' directive itself, and is empty on return. */

void macro_define(struct list *list)
{
    struct macro *m;
    struct token *ident;
    struct token *t;
    struct list formals = LIST_INITIALIZER(formals);
    macro_flags flags = 0;

    list_match(list, TOKEN_IDENT, &ident);

    if (list_first_is(list, TOKEN_LPAREN)) {
        list_pop(list, 0);
        flags = MACRO_HAS_ARGS;
    
        if (!list_first_is(list, TOKEN_RPAREN)) {
            for (;;) {
                list_strip_ends(list);
                list_match(list, TOKEN_IDENT, &t);
                list_append(&formals, t);
                list_strip_ends(list);

                if (list_first_is(list, TOKEN_COMMA))
                    list_pop(list, 0);
                else
                    break;
            }
        }

        list_match(list, TOKEN_RPAREN, 0);
    }

    list_normalize(list, &formals);
    m = macro_lookup(&ident->u.text);
    
    if (m) {
        if (m->flags & MACRO_PREDEF_MASK)
            error("can't #define reserved identifier '%s'", 
              VSTRING_BUF(m->name));

        if (!list_same(&m->formals, &formals)
          || !list_same(&m->tokens, list)
          || (m->flags != flags))
            error("incompatible re-#definition of '%s'",
              VSTRING_BUF(m->name));
        
        list_clear(&formals);
        list_clear(list);
    } else {
        m = insert(&ident->u.text);
        m->flags = flags;
        list_concat(&m->formals, &formals);
        list_concat(&m->tokens, list);
    }

    token_free(ident);
}

/* handle a -D<macro>[=<value>] command line argument.
   true on success, false if the argument is malformed. */

int macro_cmdline(char *s)
{
    struct list list = LIST_INITIALIZER(list);
    struct token *t;

    input_tokenize(&list, s);
    list_strip_ends(&list);
    t = list_first(&list);
    if (!t || (t->class != TOKEN_IDENT)) return 0;
    t = list_next(t);

    if (t) {
        if (t->class != TOKEN_EQ) return 0;
        list_drop(&list, t);
    } else {
        t = token_number(1);
        list_append(&list, t);
    }
        
    macro_define(&list);
    return 1;
}

/* process #undef directive. like macro_define(), the list holds the 
   tokens on the line following '#undef'. we don't fully clear the
   list, though, and leave it to the caller to flag trailing garbage. */

void macro_undef(struct list *list)
{
    struct token *ident;
    struct macro *m;
    int b;

    list_match(list, TOKEN_IDENT, &ident);
    m = macro_lookup(&ident->u.text);
    
    if (m) {
        if (m->flags & MACRO_PREDEF_MASK)
            error("can't #undef reserved identifier '%s'",
              VSTRING_BUF(m->name));

        list_clear(&m->formals);
        list_clear(&m->tokens);
        vstring_free(&m->name);
        b = BUCKET(&ident->u.text);
        TAILQ_REMOVE(&buckets[b], m, links);
        free(m);
    }

    token_free(ident);
}

/* actual arguments are (unsurprisingly) lists of token lists. */

struct arg
{
    struct list tokens;
    TAILQ_ENTRY(arg) links;
};

TAILQ_HEAD(arg_list, arg);          /* struct arg_list */

/* allocate and initialize a new arg, and append it to arg_list */

static struct arg *arg_new(struct arg_list *args)
{
    struct arg *a;
    
    a = safe_malloc(sizeof(struct arg));
    list_init(&a->tokens);
    TAILQ_INSERT_TAIL(args, a, links);
    return a;
}

/* empty the arg_list, freeing up each associated arg */

static void args_clear(struct arg_list *args)
{
    struct arg *a;
    
    while (a = TAILQ_FIRST(args)) {
        list_clear(&a->tokens);
        free(a);
        TAILQ_REMOVE(args, a, links);
    }
}

/* return the nth argument from the argument list */

static struct arg *arg_i(struct arg_list *args, int i)
{
    struct arg *a;

    for (a = TAILQ_FIRST(args); i > 0; --i)
        a = TAILQ_NEXT(a, links);

    if (a == 0) error("CPP INTERNAL: argument out of bounds");
    return a;
}

/* destructively gather the actual arguments to the given macro */

static void gather(struct list *list, struct macro *m, struct arg_list *args)
{
    struct token *formal;
    struct token *t;
    struct arg *a;
    int parentheses;

    list_pop(list, 0); /* ( */

    formal = list_first(&m->formals);
    
    while (formal) {
        a = arg_new(args);
        parentheses = 0;

        while (t = list_first(list)) {
            if ((parentheses == 0) && ((t->class == TOKEN_COMMA)
              || (t->class == TOKEN_RPAREN)))
                break;

            if (t->class == TOKEN_LPAREN) ++parentheses;
            if (t->class == TOKEN_RPAREN) --parentheses;
    
            list_remove(list, t);
            list_append(&a->tokens, t);
        }

        list_strip_ends(&a->tokens);
        list_fold_spaces(&a->tokens);
        list_placeholder(&a->tokens);
        formal = list_next(formal);

        if (list_first_is(list, TOKEN_COMMA))
            list_pop(list, 0);
        else
            break;
    }

    if (formal) 
        error("too few arguments to macro '%s'", VSTRING_BUF(m->name));

    t = list_first(list);
    t = list_skip_spaces(t);

    if (t == 0) 
        error("deformed arguments to macro '%s'", VSTRING_BUF(m->name));

    if (t->class != TOKEN_RPAREN)
        error("too many arguments to macro '%s'", VSTRING_BUF(m->name));

    t = list_next(t);
    list_cut(list, t);
}

/* phase 1 of macro replacement: process stringize (#) operations */

static void stringize(struct list *list, struct arg_list *args)
{
    struct arg *a;
    struct token *t;
    struct token *t2;

    t = list_first(list);

    while (t) {
        if (t->class == TOKEN_STRINGIZE) {
            t = list_drop(list, t);

            while (t && (t->class == TOKEN_SPACE))
                t = list_drop(list, t);

            if (!t || (t->class != TOKEN_ARG))
                error("illegal operand to stringize (#)");

            a = arg_i(args, t->u.i);
            t = list_drop(list, t);
            t2 = list_stringize(&a->tokens);
            list_insert(list, t, t2);
        } else
            t = list_next(t);
    }
}

/* phase 2 of macro replacement: expand all actual arguments. */

static void expand(struct list *list, struct arg_list *args)
{
    struct token *t;
    struct arg *a;
    struct list tmp = LIST_INITIALIZER(tmp);

    t = list_first(list);

    while (t) {
        if (t->class == TOKEN_ARG) {
            a = arg_i(args, t->u.i);
            list_copy(&tmp, &a->tokens);

            if (!list_next_is(list, t, TOKEN_PASTE)
              && !list_prev_is(list, t, TOKEN_PASTE))
                macro_replace_all(&tmp);
            
            t = list_drop(list, t);
            list_insert_list(list, t, &tmp);
        } else
            t = list_next(t);
    }
}

/* phase 3 of macro replacement: perform token pasting (##) operations */

static void paste(struct list *list)
{
    struct token *t;
    struct token *result;
    struct token *left;
    struct token *right;

    t = list_first(list);

    while (t) {
        if (t->class == TOKEN_PASTE) {
            list_strip_around(list, t);
            left = list_prev(t);
            right = list_next(t);

            if (!left || !right)
                error("missing operands to paste (##) operator");

            result = token_paste(left, right);
            list_drop(list, left);
            list_drop(list, right);
            t = list_drop(list, t);
            list_insert(list, t, result);
        } else
            t = list_next(t);
    }
}

/* if the first token is a macro, then it (and any subsequent arguments)
   are replaced with the appropriate expansion, and non-zero is returned. */

int macro_replace(struct list *list)
{
    struct macro *m;
    struct token *t;
    struct arg_list args = TAILQ_HEAD_INITIALIZER(args);
    struct list repl = LIST_INITIALIZER(repl);

    t = list_first(list);
    if (t->class != TOKEN_IDENT) return 0;
    m = macro_lookup(&t->u.text);
    if (!m || !MACRO_DEFINED(m)) return 0;

    if (m->flags & MACRO_HAS_ARGS) {
        t = list_next(t);
        t = list_skip_spaces(t);
        if (!t || (t->class != TOKEN_LPAREN)) return 0;
        list_cut(list, t);
        gather(list, m, &args);
    } else
        list_pop(list, 0);

    tokens(m, &repl);

    stringize(&repl, &args);
    expand(&repl, &args);
    paste(&repl);
    list_ennervate(&repl, &m->name);
    list_concat(&repl, list);
    list_concat(list, &repl);
    args_clear(&args);

    return 1;
}

/* rescan the list until all macro
   substitutions have been performed. */

void macro_replace_all(struct list *list)
{
    struct list tmp = LIST_INITIALIZER(tmp);
    struct token *t;

    list_concat(&tmp, list);

    while (!list_empty(&tmp))
        if (!macro_replace(&tmp)) {
            list_pop(&tmp, &t);
            list_append(list, t);
        }
}

/* vi: set ts=4 expandtab: */
