/* token.c - tokens and token lists                     ncc, the new c compiler

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
#include <string.h>
#include <ctype.h>
#include <errno.h>
#include "../common/util.h"
#include "../common/escape.h"
#include "cpp.h"
#include "token.h"

/* because of the churn rate of tokens, we keep a local pool
   instead of passing off every request to the system allocator */

static struct list pool = LIST_INITIALIZER(pool);

/* grab a token from the pool, or allocate another if
   none remain. initialize as appropriate to the class. */

static struct token *alloc(token_class class)
{
    struct token *t;

    if (list_empty(&pool))
        t = safe_malloc(sizeof(struct token));
    else {
        t = list_first(&pool);
        list_remove(&pool, t);
    }

    t->class = class;
    if (!(class & TOKEN_NO_TEXT)) vstring_init(&t->u.text);
    
    return t;
}

/* create a new TOKEN_NUMBER for an integer of the given value */

struct token *token_number(int i)
{
    char buf[21];       /* 19 digits (2^63), possible sign, and NUL */
    struct token *t;

    sprintf(buf, "%d", i);
    t = alloc(TOKEN_NUMBER);
    vstring_puts(&t->u.text, buf);

    return t;
}

/* add a character to a vstring, escaping quotes and backslashes */

static void backslash(struct vstring *vs, int c)
{
    if ((c == '\\') || (c == '"'))
        vstring_putc(vs, '\\');

    vstring_putc(vs, c);
}

/* create a TOKEN_STRING which contains the given text.
   (the text is escaped to ensure the literal is valid) */

struct token *token_string(char *s)
{
    struct token *t;

    t = alloc(TOKEN_STRING);
    vstring_putc(&t->u.text, '"');
    while (*s) backslash(&t->u.text, *s++);
    vstring_putc(&t->u.text, '"');

    return t;
}

/* convenience function for creating TOKEN_INT */

struct token *token_int(long i)
{
    struct token *t;

    t = alloc(TOKEN_INT);
    t->u.i = i;
    return t;
}

/* convert a TOKEN_NUMBER into a TOKEN_INT or TOKEN_UINT */

void token_convert_number(struct token *t)
{
    unsigned long u;
    char *endptr;
    token_class class;
    
    class = TOKEN_INT;
    errno = 0;
    u = strtoul(VSTRING_BUF(t->u.text), &endptr, 0);
    if (toupper(*endptr) == 'L') ++endptr;
    if (toupper(*endptr) == 'U') class = TOKEN_UINT;
    if (toupper(*endptr) == 'L') ++endptr;
    if (*endptr || errno) error("malformed integer constant");

    vstring_free(&t->u.text);
    t->class = class;
    t->u.i = u;
}

/* convert a TOKEN_CHAR into a TOKEN_INT */

void token_convert_char(struct token *t)
{
    char *cp;
    int c;

    cp = VSTRING_BUF(t->u.text);
    ++cp; /* opening quote */
    c = escape(&cp);
    if (c == -1) error("invalid escape sequence");
    if (*cp != '\'') error("multi-character constants unsupported");
    vstring_free(&t->u.text);
    t->class = TOKEN_INT;
    t->u.i = c;
}

/* release a token's resources 
   move it to the free pool. */

void token_free(struct token *t)
{
    if (!(t->class & TOKEN_NO_TEXT))
        vstring_free(&t->u.text);

    list_prepend(&pool, t);
}

/* return a token with a single space in it */

struct token *token_space(void)
{
    struct token *t;
    
    t = alloc(TOKEN_SPACE);
    vstring_putc(&t->u.text, ' ');
    
    return t;
}

/* keep the modifiers[] table in sync with token_class */

static struct { token_class dup; token_class eq; } modifiers[] = {
    { TOKEN_INC,        TOKEN_PLUSEQ },         /* TOKEN_PLUS */
    { TOKEN_DEC,        TOKEN_MINUSEQ },        /* TOKEN_MINUS */
    { 0,                TOKEN_MULEQ },          /* TOKEN_MUL */
    { 0,                TOKEN_DIVEQ },          /* TOKEN_DIV */
    { 0,                TOKEN_MODEQ },          /* TOKEN_MOD */
    { TOKEN_ANDAND,     TOKEN_ANDEQ },          /* TOKEN_AND */
    { TOKEN_OROR,       TOKEN_OREQ },           /* TOKEN_OR */
    { 0,                TOKEN_XOREQ },          /* TOKEN_XOR */
    { TOKEN_SHR,        TOKEN_GTEQ },           /* TOKEN_GT */
    { 0,                TOKEN_SHREQ },          /* TOKEN_SHR */
    { TOKEN_SHL,        TOKEN_LTEQ },           /* TOKEN_LT */
    { 0,                TOKEN_SHLEQ },          /* TOKEN_SHL */
    { TOKEN_HASHHASH,   0          },           /* TOKEN_HASH */
    { 0,                TOKEN_EQEQ },           /* TOKEN_EQ */
    { 0,                TOKEN_NOTEQ }           /* TOKEN_NOT */
};

/* returns true if whitespace is required to separate a token
   of the first class from a token of the second class. */

int token_separate(token_class first, token_class second)
{
    int index;

    switch (first)
    {
    case TOKEN_IDENT:
        if ((second == TOKEN_IDENT) || (second == TOKEN_NUMBER))
            return 1;

    case TOKEN_NUMBER:
        if ((second == TOKEN_IDENT) || (second == TOKEN_NUMBER)
          || (second == TOKEN_DOT) || (second == TOKEN_ELLIP))
            return 1;
        
        break;

    case TOKEN_DOT:
        if ((second == TOKEN_DOT) || (second == TOKEN_ELLIP)
          || (second == TOKEN_NUMBER))
            return 1;

        break;

    case TOKEN_MINUS:
        if ((second == TOKEN_GT) || (second == TOKEN_GTEQ)
          || (second == TOKEN_SHR) || (second == TOKEN_SHREQ))
            return 1;

    default:
        index = TOKEN_INDEX(first);
        if (index >= ARRAY_SIZE(modifiers)) break;

        if (modifiers[index].eq && ((second == TOKEN_EQ)
          || (second == TOKEN_EQEQ)))
            return 1;

        if (modifiers[index].dup) {
            if ((second == first) || (second == modifiers[index].dup)
              || (second == modifiers[index].eq))
                return 1;

            index = TOKEN_INDEX(modifiers[index].dup);
            if (index >= ARRAY_SIZE(modifiers)) break;
        
            if ((second == modifiers[index].dup)
              || (second == modifiers[index].eq))
                return 1;
        }
    }

    return 0;
}

/* approximation of token class based on the first character. note that
   only printable ASCII is accepted; this is perhaps too restrictive. */

static token_class classes[] =
{
    /*   0 */   0,              0,              0,              0,
    /*   4 */   0,              0,              0,              0,
    /*   8 */   0,              TOKEN_SPACE,    0,              TOKEN_SPACE,
    /*  12 */   TOKEN_SPACE,    TOKEN_SPACE,    0,              0,
    /*  16 */   0,              0,              0,              0,
    /*  20 */   0,              0,              0,              0,
    /*  24 */   0,              0,              0,              0,
    /*  28 */   0,              0,              0,              0,
    /*  32 */   TOKEN_SPACE,    TOKEN_NOT,      TOKEN_STRING,   TOKEN_HASH,
    /*  36 */   TOKEN_OTHER,    TOKEN_MOD,      TOKEN_AND,      TOKEN_CHAR,
    /*  40 */   TOKEN_LPAREN,   TOKEN_RPAREN,   TOKEN_MUL,      TOKEN_PLUS,
    /*  44 */   TOKEN_COMMA,    TOKEN_MINUS,    TOKEN_DOT,      TOKEN_DIV,
    /*  48 */   TOKEN_NUMBER,   TOKEN_NUMBER,   TOKEN_NUMBER,   TOKEN_NUMBER,
    /*  52 */   TOKEN_NUMBER,   TOKEN_NUMBER,   TOKEN_NUMBER,   TOKEN_NUMBER,
    /*  56 */   TOKEN_NUMBER,   TOKEN_NUMBER,   TOKEN_COLON,    TOKEN_SEMI,
    /*  60 */   TOKEN_LT,       TOKEN_EQ,       TOKEN_GT,       TOKEN_QUEST,
    /*  64 */   TOKEN_OTHER,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,
    /*  68 */   TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,
    /*  72 */   TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,
    /*  76 */   TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,
    /*  80 */   TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,
    /*  84 */   TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,
    /*  88 */   TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_LBRACK,
    /*  92 */   TOKEN_OTHER,    TOKEN_RBRACK,   TOKEN_XOR,      TOKEN_IDENT,
    /*  96 */   TOKEN_OTHER,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,
    /* 100 */   TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,
    /* 104 */   TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,
    /* 108 */   TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,
    /* 112 */   TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,
    /* 116 */   TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,
    /* 120 */   TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_IDENT,    TOKEN_LBRACE,
    /* 124 */   TOKEN_OR,       TOKEN_RBRACE,   TOKEN_TILDE,    0
};

/* scan buf and return the next token. buf must be NUL-terminated.
   endptr is updated to point to the character after the token. */

struct token *token_scan(char *buf, char **endptr)
{
    struct token *t;
    char *cp = buf;
    token_class class;

    if ((*cp > 0) && (*cp < ARRAY_SIZE(classes)))
        class = classes[*cp];
    else
        class = 0;

    switch (class)
    {
    case TOKEN_CHAR:
    case TOKEN_STRING:
        {
            int esc = 1;

            for (;;) {
                if (!*cp) {
                    if (class == TOKEN_STRING)
                        error("unterminated string literal");
                    else
                        error("unterminated char constant");
                }
    
                if ((*cp++ == *buf) && !esc) break;
                esc = !esc && (cp[-1] == '\\');
            }
        }
        break;

    case TOKEN_DOT:
        if ((cp[1] == '.') && (cp[2] == '.')) {
            cp += 3;
            class = TOKEN_ELLIP;
            break;
        } else if (!isdigit(*cp)) {
            ++cp;
            break;
        }
    case TOKEN_NUMBER:
        while (isalnum(*cp) || (*cp == '.') || (*cp == '_')) {
            if ((toupper(*cp) == 'E') && ((cp[1] == '-') || (cp[1] == '+')))
                ++cp;

            ++cp;
        }
        break;

    case TOKEN_SPACE:
        while (isspace(*cp)) ++cp;
        break;

    case TOKEN_IDENT:
        while (isalnum(*cp) || (*cp == '_')) ++cp;
        break;

    case TOKEN_MINUS:
        if (cp[1] == '>') {
            class = TOKEN_ARROW;
            cp += 2;
            break;
        }
    default:
        {
            int index;

            ++cp;

            while ((index = TOKEN_INDEX(class)) < ARRAY_SIZE(modifiers)) {
                if ((*cp == *buf) && (modifiers[index].dup)) {
                    class = modifiers[index].dup;
                    ++cp;
                } else if ((*cp == '=') && (modifiers[index].eq)) {
                    class = modifiers[index].eq;
                    ++cp;
                } else
                    break;
            }
        }
        break;

    case 0:
        error("invalid character (ASCII %d) in input", *cp & 0xFF);
    }

    t = alloc(class);
    vstring_put(&t->u.text, buf, cp - buf);
    *endptr = cp;
    return t;
}

/* returns the token that results from
   pasting the text of t1 and t2. */

struct token *token_paste(struct token *t1, struct token *t2)
{
    struct vstring vs = VSTRING_INITIALIZER;
    struct token *t;
    char *endptr;
    
    token_text(t1, &vs);
    token_text(t2, &vs);
    t = token_scan(VSTRING_BUF(vs), &endptr);

    if ((*endptr) || (t->class & TOKEN_NOT_C))
        error("result of paste (##) '%s' is not a token", VSTRING_BUF(vs));
    
    return t;
}

/* returns non-zero if two tokens represent the same thing */

int token_same(struct token *t1, struct token *t2)
{
    if (t1->class == t2->class) {
        if (t1->class & TOKEN_STATIC) return 1;
        if (t1->class & TOKEN_NO_TEXT) return (t1->u.i == t2->u.i);
        return vstring_same(&t1->u.text, &t2->u.text);
    }

    return 0;
}

/* make a copy of a token */

struct token *token_copy(struct token *t)
{
    struct token *t2;

    t2 = alloc(t->class);

    if (t2->class & TOKEN_NO_TEXT)
        t2->u.i = t->u.i;
    else
        vstring_concat(&t2->u.text, &t->u.text);

    return t2;
}

/* appends the text of the token to the vstring */

void token_text(struct token *token, struct vstring *vs)
{
    if (token->class & TOKEN_NO_TEXT)
        error("CPP INTERNAL: can't get text of non-text token");

    vstring_put(vs, VSTRING_BUF(token->u.text), VSTRING_LEN(token->u.text));
}

/* appends the text of a TOKEN_STRING to a vstring, without the
   opening and closing quotes. */

void token_dequote(struct token *token, struct vstring *vs)
{
    size_t i;
    size_t len;
    char *buf;

    if (token->class != TOKEN_STRING)
        error("CPP INTERNAL: can't dequote non-string token");
    
    len = VSTRING_LEN(token->u.text);
    buf = VSTRING_BUF(token->u.text);

    for (i = 1; i < (len - 1); ++i)
            vstring_putc(vs, *++buf);
}

/* discard all the tokens in the list before where. where
   can be 0, in which case the entire list is discarded. */

void list_cut(struct list *list, struct token *where)
{
    struct token *t;

    while (!list_empty(list)) {
        t = list_first(list);
        if (t == where) break;
        list_remove(list, t);
        token_free(t);
    }
}

/* return the first token from cursor that is
   not a TOKEN_SPACE, or 0 if there isn't one. */

struct token *list_skip_spaces(struct token *t)
{
    while (t && (t->class == TOKEN_SPACE))
        t = list_next(t);

    return t;
}

/* fold all sequences of whitespace to a single space */

void list_fold_spaces(struct list *list)
{
    struct token *t;
    struct token *t2;

    for (t = list_first(list); t; t = list_next(t)) {
        if (t->class == TOKEN_SPACE) {
            vstring_free(&t->u.text);
            vstring_init(&t->u.text);
            vstring_putc(&t->u.text, ' ');

            while ((t2 = list_next(t)) && (t2->class == TOKEN_SPACE)) {
                list_remove(list, t2);
                token_free(t2);
            }
        }
    }
}

/* trim trailing and leading spaces from the list */

void list_strip_ends(struct list *list)
{
    struct token *t;

    while (((t = list_first(list)) && (t->class == TOKEN_SPACE))
      || ((t = list_last(list)) && (t->class == TOKEN_SPACE)))
    {
        list_remove(list, t);
        token_free(t);
    }
}

/* remove all TOKEN_SPACEs and empty TOKEN_OTHERs from the list */

void list_strip_all(struct list *list)
{
    struct token *t;
    struct token *t2;

    t = list_first(list);

    while (t) {
        t2 = list_next(t);

        if ((t->class == TOKEN_SPACE) || ((t->class == TOKEN_OTHER)
          && (VSTRING_LEN(t->u.text) == 0))) {
            list_remove(list, t);
            token_free(t);
        }

        t = t2;
    }
}

/* remove TOKEN_SPACEs from around a token */

void list_strip_around(struct list *list, struct token *around)
{
    struct token *t;
    
    while ((t = list_prev(around)) && (t->class == TOKEN_SPACE))
        list_drop(list, t);

    while ((t = list_next(around)) && (t->class == TOKEN_SPACE))
        list_drop(list, t);
}

/* remove the head of the list. if tp isn't 0, then then
   token is returned to the caller, otherwise it's freed. */

void list_pop(struct list *list, struct token **tp)
{
    struct token *t;

    t = list_first(list);
    list_remove(list, t);

    if (tp)
        *tp = t;
    else
        token_free(t);
}

/* removes and frees the specified token from
   list. returns the element following. */

struct token *list_drop(struct list *list, struct token *token)
{
    struct token *t;

    t = list_next(token);
    list_remove(list, token);
    token_free(token);

    return t;
}

/* ensure that the head of the list is of the given class,
   and remove it. if tp is not 0, then the token is returned
   to the caller, otherwise it is freed. */

void list_match(struct list *list, token_class class, struct token **tp)
{
    struct token *t;

    t = list_first(list);

    if (t && (t->class == class))
        list_pop(list, tp);
    else
        error("syntax");
}

/* returns true if two lists represent the same token sequences */

int list_same(struct list *l1, struct list *l2)
{
    struct token *t1 = list_first(l1);
    struct token *t2 = list_first(l2);

    while (t1 && t2 && token_same(t1, t2)) {
        t1 = list_next(t1);
        t2 = list_next(t2);
    }

    if (t1 || t2)
        return 0;
    else
        return 1;
}

/* normalize a macro replacement list:

   (1) leading and trailing whitespace is tossed,
   (2) internal spaces are consolidated to a single space,
   (3) TOKEN_HASH and TOKEN_HASHHASH are promoted to the operators
       TOKEN_STRINGIZE and TOKEN_PASTE (respectively),
   (4) appearances of the identifiers corresponding to the formal
       arguments are replaced with TOKEN_ARGs. */

void list_normalize(struct list *list, struct list *formals)
{
    struct token *t;
    struct token *t2;
    int i;

    list_strip_ends(list);
    list_fold_spaces(list);

    for (t = list_first(list); t; t = list_next(t)) {
        if (t->class == TOKEN_HASH) t->class = TOKEN_STRINGIZE;
        if (t->class == TOKEN_HASHHASH) t->class = TOKEN_PASTE;
    }

    for (t = list_first(formals), i = 0; t; t = list_next(t), ++i) {
        for (t2 = list_first(list); t2; t2 = list_next(t2)) {
            if (token_same(t, t2)) {
                vstring_free(&t2->u.text);
                t2->class = TOKEN_ARG;
                t2->u.i = i;
            }
        }
    }
}

/* create a TOKEN_STRING out of the text of the tokens in list,
   escaping special characters as necessary. */

struct token *list_stringize(struct list *list)
{
    struct token *str;
    struct token *t;
    char *s;

    str = alloc(TOKEN_STRING);
    vstring_putc(&str->u.text, '"');

    for (t = list_first(list); t; t = list_next(t)) {
        if (t->class & TOKEN_NO_TEXT)
            error("CPP INTERNAL: can't stringize a textless token");

        if ((t->class == TOKEN_SPACE) && ((t == list_first(list))
          || (list_next(t) == 0)))
            continue;
    
        s = VSTRING_BUF(t->u.text);

        while (*s) {
            if ((t->class == TOKEN_STRING) || (t->class == TOKEN_CHAR))
                backslash(&str->u.text, *s++);
            else
                vstring_putc(&str->u.text, *s++);
        }
    }
    
    vstring_putc(&str->u.text, '"');
    return str;
}

/* demote all occurrences TOKEN_IDENT with the specified name to
   TOKEN_OTHER, (so it is not a candidate for macro expansion) */

void list_ennervate(struct list *list, struct vstring *name)
{
    struct token *t;

    for (t = list_first(list); t; t = list_next(t))
        if ((t->class == TOKEN_IDENT) && vstring_same(&t->u.text, name))
            t->class = TOKEN_OTHER;
}

/* copy the tokens from src to the end of dst */

void list_copy(struct list *dst, struct list *src)
{
    struct token *t;    
    struct token *t2;

    for (t = list_first(src); t; t = list_next(t)) {
        t2 = token_copy(t);
        list_append(dst, t2);
    }
}

/* move n tokens from (front of) src to (end of) dst */

void list_move(struct list *dst, struct list *src, int n)
{
    struct token *t;

    while (n--) {
        t = list_first(src);
        if (t == 0) error("CPP INTERNAL: list_move");
        list_remove(src, t);
        list_append(dst, t);
    }
}

/* return true if the next (previous) non-space token
   in list after (before) t has the given class */

int list_next_is(struct list *list, struct token *after, token_class class)
{
    after = list_next(after);

    while (after && (after->class == TOKEN_SPACE))
        after = list_next(after);

    if (after && (after->class == class))
        return 1;
    else
        return 0;
}

int list_prev_is(struct list *list, struct token *before, token_class class)
{
    before = list_prev(before);

    while (before && (before->class == TOKEN_SPACE))
        before = list_prev(before);

    if (before && (before->class == class))
        return 1;
    else
        return 0;
}

/* insert a token into list before the specified token.
   if before is 0, it means "the end of the list" */

void list_insert(struct list *list, struct token *before, struct token *t)
{
    if (before)
        TAILQ_INSERT_BEFORE(before, t, links);
    else
        list_append(list, t);
}


/* move the contents of the src list to another list dst,
   before the specified token. again, if before is 0, it
   means the tokens should be appended to the end of dst*/

void list_insert_list(struct list *dst, struct token *before, struct list *src)
{
    struct token *t;

    while (t = list_first(src)) {
        list_remove(src, t);
        list_insert(dst, before, t);
    }
}

/* ensure that the list is not empty by adding an
   empty TOKEN_OTHER if necessary */

void list_placeholder(struct list *list)
{
    struct token *t;

    if (list_empty(list)) {
        t = alloc(TOKEN_OTHER);
        list_append(list, t);
    }
}

/* vi: set ts=4 expandtab: */
