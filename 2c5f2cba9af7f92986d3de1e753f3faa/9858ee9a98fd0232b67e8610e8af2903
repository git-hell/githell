/* token.h - tokens and token lists                     ncc, the new c compiler

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

#ifndef TOKEN_H
#define TOKEN_H

#include "../common/tailq.h"
#include "vstring.h"

/* token classes are encoded as:
       
        [31:12]     various flags
        [11:8]      operator precedence
        [7:0]       ordinal value

   the last field, the ordinal value, exists to guarantee that the
   enumerators be distinct; it also serves for the first group of tokens
   as an index into modifiers[] in token.c... don't reorder recklessly.

   as a convenience, we reserve the value 0 to mean "no token". */

typedef int token_class;

#define TOKEN_NO_TEXT           0x80000000    /* u.text not valid */
#define TOKEN_NOT_C             0x40000000    /* not a valid C token */
#define TOKEN_STATIC            0x20000000    /* token text always the same */

#define TOKEN_PREC_MASK         0x00000F00 
#define TOKEN_PREC_NONE         0x00000000 
#define TOKEN_PREC_MUL          0x00000100      /* careful: TOKEN_PREC_NEXT */
#define TOKEN_PREC_ADD          0x00000200 
#define TOKEN_PREC_SHIFT        0x00000300 
#define TOKEN_PREC_REL          0x00000400 
#define TOKEN_PREC_EQ           0x00000500 
#define TOKEN_PREC_AND          0x00000600 
#define TOKEN_PREC_XOR          0x00000700 
#define TOKEN_PREC_OR           0x00000800 
#define TOKEN_PREC_LAND         0x00000900 
#define TOKEN_PREC_LOR          0x00000A00 

#define TOKEN_PREC(class)       ((class) & TOKEN_PREC_MASK)
#define TOKEN_PREC_NEXT(prec)   ((prec) - TOKEN_PREC_MUL)

#define TOKEN_INDEX_MASK        0x000000FF 
#define TOKEN_INDEX(class)      ((class) & TOKEN_INDEX_MASK)

#define TOKEN_PLUS          (  0 | TOKEN_STATIC | TOKEN_PREC_ADD )    /* + */
#define TOKEN_MINUS         (  1 | TOKEN_STATIC | TOKEN_PREC_ADD )    /* - */
#define TOKEN_MUL           (  2 | TOKEN_STATIC | TOKEN_PREC_MUL )    /* * */
#define TOKEN_DIV           (  3 | TOKEN_STATIC | TOKEN_PREC_MUL )    /* / */
#define TOKEN_MOD           (  4 | TOKEN_STATIC | TOKEN_PREC_MUL )    /* % */
#define TOKEN_AND           (  5 | TOKEN_STATIC | TOKEN_PREC_AND )    /* & */
#define TOKEN_OR            (  6 | TOKEN_STATIC | TOKEN_PREC_OR )     /* | */
#define TOKEN_XOR           (  7 | TOKEN_STATIC | TOKEN_PREC_XOR )    /* ^ */
#define TOKEN_GT            (  8 | TOKEN_STATIC | TOKEN_PREC_REL )    /* > */
#define TOKEN_SHR           (  9 | TOKEN_STATIC | TOKEN_PREC_SHIFT )  /* >> */
#define TOKEN_LT            ( 10 | TOKEN_STATIC | TOKEN_PREC_REL )    /* < */
#define TOKEN_SHL           ( 11 | TOKEN_STATIC | TOKEN_PREC_SHIFT )  /* << */
#define TOKEN_HASH          ( 12 | TOKEN_STATIC | TOKEN_NOT_C )       /* # */
#define TOKEN_EQ            ( 13 | TOKEN_STATIC )                     /* = */
#define TOKEN_NOT           ( 14 | TOKEN_STATIC )                     /* ! */

#define TOKEN_LPAREN        ( 15 | TOKEN_STATIC )                     /* ( */
#define TOKEN_RPAREN        ( 16 | TOKEN_STATIC )                     /* ) */
#define TOKEN_LBRACE        ( 17 | TOKEN_STATIC )                     /* { */
#define TOKEN_RBRACE        ( 18 | TOKEN_STATIC )                     /* } */
#define TOKEN_LBRACK        ( 19 | TOKEN_STATIC )                     /* [ */
#define TOKEN_RBRACK        ( 20 | TOKEN_STATIC )                     /* ] */
#define TOKEN_EQEQ          ( 21 | TOKEN_STATIC | TOKEN_PREC_EQ )     /* == */
#define TOKEN_NOTEQ         ( 22 | TOKEN_STATIC | TOKEN_PREC_EQ )     /* != */
#define TOKEN_INC           ( 23 | TOKEN_STATIC )                     /* ++ */
#define TOKEN_DEC           ( 24 | TOKEN_STATIC )                     /* -- */
#define TOKEN_PLUSEQ        ( 25 | TOKEN_STATIC )                     /* += */
#define TOKEN_MINUSEQ       ( 26 | TOKEN_STATIC )                     /* -= */
#define TOKEN_MULEQ         ( 27 | TOKEN_STATIC )                     /* *= */
#define TOKEN_DIVEQ         ( 28 | TOKEN_STATIC )                     /* /= */
#define TOKEN_MODEQ         ( 29 | TOKEN_STATIC )                     /* %= */
#define TOKEN_ANDEQ         ( 30 | TOKEN_STATIC )                     /* &= */
#define TOKEN_ANDAND        ( 31 | TOKEN_STATIC | TOKEN_PREC_LAND)    /* && */
#define TOKEN_OREQ          ( 32 | TOKEN_STATIC )                     /* |= */
#define TOKEN_OROR          ( 33 | TOKEN_STATIC | TOKEN_PREC_LOR)     /* || */
#define TOKEN_XOREQ         ( 34 | TOKEN_STATIC )                     /* ^= */
#define TOKEN_GTEQ          ( 35 | TOKEN_STATIC | TOKEN_PREC_REL )    /* >= */
#define TOKEN_SHREQ         ( 36 | TOKEN_STATIC )                     /* >>= */
#define TOKEN_LTEQ          ( 37 | TOKEN_STATIC | TOKEN_PREC_REL )    /* <= */
#define TOKEN_SHLEQ         ( 38 | TOKEN_STATIC )                     /* <<= */
#define TOKEN_ARROW         ( 39 | TOKEN_STATIC )                     /* -> */
#define TOKEN_DOT           ( 40 | TOKEN_STATIC )                     /* . */
#define TOKEN_ELLIP         ( 41 | TOKEN_STATIC )                     /* ... */
#define TOKEN_HASHHASH      ( 42 | TOKEN_STATIC | TOKEN_NOT_C )       /* ## */
#define TOKEN_SEMI          ( 43 | TOKEN_STATIC )                     /* ; */
#define TOKEN_COLON         ( 44 | TOKEN_STATIC )                     /* : */
#define TOKEN_QUEST         ( 45 | TOKEN_STATIC )                     /* ? */
#define TOKEN_TILDE         ( 46 | TOKEN_STATIC )                     /* ~ */
#define TOKEN_COMMA         ( 47 | TOKEN_STATIC )                     /* , */

#define TOKEN_SPACE         ( 48 )                    /* whitespace(s) */
#define TOKEN_IDENT         ( 49 )                    /* identifier */
#define TOKEN_INERT_IDENT   ( 50 )                    /* can't be expanded */
#define TOKEN_NUMBER        ( 51 )                    /* preproc number */
#define TOKEN_STRING        ( 52 )                    /* string literal */
#define TOKEN_CHAR          ( 53 )                    /* character constant */

#define TOKEN_STRINGIZE     ( 54 | TOKEN_STATIC | TOKEN_NOT_C )  /* # (op) */
#define TOKEN_PASTE         ( 55 | TOKEN_STATIC | TOKEN_NOT_C )  /* ## (op) */

#define TOKEN_INT           ( 56 | TOKEN_NO_TEXT ) 
#define TOKEN_UINT          ( 57 | TOKEN_NO_TEXT ) 
#define TOKEN_ARG           ( 58 | TOKEN_NO_TEXT )    /* formal argument */

#define TOKEN_OTHER         ( 59 | TOKEN_NOT_C )

/* because the union payload of token has a vstring, which
   requires initialization/finalization, the class should be
   treated read-only by client code (i.e., outside token.c) */

struct token
{
    token_class class;

    union {
        long i;                 /* TOKEN_INT / TOKEN_ARG */
        unsigned long u;        /* TOKEN_UINT */
        struct vstring text;    /* everything else */
    } u;

    TAILQ_ENTRY(token) links;
};

extern void token_free(struct token *);
extern struct token *token_number(int);
extern struct token *token_string(char *);
extern struct token *token_int(long);
extern void token_convert_number(struct token *);
extern void token_convert_char(struct token *);
extern struct token *token_scan(char *, char **);
extern struct token *token_paste(struct token *, struct token *);
extern int token_same(struct token *, struct token *);
extern void token_dequote(struct token *, struct vstring *);
extern void token_text(struct token *, struct vstring *);
extern struct token *token_space(void);
extern struct token *token_copy(struct token *);
extern int token_separate(token_class, token_class);

#define TOKEN_SIGN(t)       ((t)->class = TOKEN_INT)
#define TOKEN_UNSIGN(t)     ((t)->class = TOKEN_UINT)

/* token lists are so central to the logic of
   the preprocessor that we just call them lists */

TAILQ_HEAD(list, token);            /* struct list */

#define LIST_INITIALIZER(l)     TAILQ_HEAD_INITIALIZER(l)

#define list_init(l)                TAILQ_INIT(l)
#define list_empty(l)               TAILQ_EMPTY(l)
#define list_first(l)               TAILQ_FIRST(l)
#define list_last(l)                TAILQ_LAST((l), list)
#define list_remove(l, t)           TAILQ_REMOVE((l), (t), links)
#define list_prepend(l, t)          TAILQ_INSERT_HEAD((l), (t), links)
#define list_append(l, t)           TAILQ_INSERT_TAIL((l), (t), links)
#define list_next(t)                TAILQ_NEXT((t), links)
#define list_prev(t)                TAILQ_PREV((t), list, links)
#define list_concat(dst, src)       TAILQ_CONCAT((dst), (src), links)

#define list_first_is(l, c) (TAILQ_FIRST(l) && (TAILQ_FIRST(l)->class == (c)))

extern int list_next_is(struct list *, struct token *, token_class);
extern int list_prev_is(struct list *, struct token *, token_class);
extern void list_cut(struct list *, struct token *);
extern void list_pop(struct list *, struct token **);
extern struct token *list_drop(struct list *, struct token *);
extern void list_match(struct list *, token_class, struct token **);
extern void list_strip_ends(struct list *);
extern void list_strip_all(struct list *);
extern void list_strip_around(struct list *, struct token *);
extern struct token *list_skip_spaces(struct token *);
extern void list_fold_spaces(struct list *);
extern int list_same(struct list *, struct list *);
extern void list_normalize(struct list *, struct list *);
extern void list_strip_around(struct list *, struct token *);
extern struct token *list_stringize(struct list *);
extern void list_copy(struct list *, struct list *);
extern void list_move(struct list *, struct list *, int);
extern void list_insert(struct list *, struct token *, struct token *);
extern void list_insert_list(struct list *, struct token *, struct list *);
extern void list_ennervate(struct list *, struct vstring *);
extern void list_placeholder(struct list *);

#define list_clear(l)           list_cut((l), 0)

#endif /* TOKEN_H */

/* vi: set ts=4 expandtab: */
