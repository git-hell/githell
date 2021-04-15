/* lex.h - lexical analyzer                             ncc, the new c compiler

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

#ifndef LEX_H
#define LEX_H

#include <stdio.h>

struct string;

extern struct string *error_path;
extern int error_line_no;

/* tokens- the use of k to refer
   to a token class is historical. */

typedef int token_class;    /* K_* */

struct token
{
    token_class k;

    union
    {
        long i; 
        unsigned long u; 
        double f;
        struct string *text;
    }; /* C11 */
};

extern struct token token;

extern void lex_init(char *);
extern struct token lex_peek(void);
extern void lex(void);
extern void lex_print_k(FILE *, token_class);
extern void lex_print_token(FILE *, struct token);
extern void lex_expect(token_class);
extern void lex_match(token_class);

    /* the lower bits of a class comprise an ordinal which
       ensures the class is unique, and also provides an
       index for text[] in lex.c. keep them in sync. */

#define K_BASE_MASK         ( 0x000000FF )
#define K_BASE(k)           ((k) & K_BASE_MASK)

    /* bits[16:8] form a powerset that simplifies
       the logic of specifiers() in decl.c */

#define K_SPEC_MASK         ( 0x0001FF00 )
#define K_SPEC(k)           ((k) & K_SPEC_MASK)

#define K_SPEC_VOID         ( 0x00000100 )
#define K_SPEC_CHAR         ( 0x00000200 )
#define K_SPEC_SHORT        ( 0x00000400 )
#define K_SPEC_INT          ( 0x00000800 )
#define K_SPEC_LONG         ( 0x00001000 )
#define K_SPEC_FLOAT        ( 0x00002000 )
#define K_SPEC_DOUBLE       ( 0x00004000 )
#define K_SPEC_UNSIGNED     ( 0x00008000 )
#define K_SPEC_SIGNED       ( 0x00010000 )

    /* token is a declaration-specifier */

#define K_DECL              ( 0x00020000 )

    /* token is in the first-set of a declarator */

#define K_DTOR              ( 0x00040000 )

    /* token is in the follow-set of a declarator in
       a declarator-list */

#define K_DECL_LIST         ( 0x00080000 )

    /* bits[23:20] encode the precedence for binary 
       operators (we use the same trick in cpp) */

#define K_PREC_MASK         ( 0x00F00000 )
#define K_PREC(k)           ((k) & K_PREC_MASK)

#define K_PREC_EPSILON      ( 0x00100000 )
#define K_PREC_NEXT(k)      ((k) - K_PREC_EPSILON)

#define K_PREC_NONE         ( 0x00000000 )
#define K_PREC_MUL          ( 0x00100000 )
#define K_PREC_ADD          ( 0x00200000 )
#define K_PREC_SHIFT        ( 0x00300000 )
#define K_PREC_REL          ( 0x00400000 )
#define K_PREC_EQ           ( 0x00500000 )
#define K_PREC_AND          ( 0x00600000 )
#define K_PREC_XOR          ( 0x00700000 )
#define K_PREC_OR           ( 0x00800000 )
#define K_PREC_LAND         ( 0x00900000 )
#define K_PREC_LOR          ( 0x00A00000 )
#define K_PREC_ASG          ( 0x00B00000 )
    
    /* bits[28:24] hold an index into the
       binary map[] in expr.c. keep in sync. */

#define K_MAP_MASK          ( 0x1F000000 )
#define K_MAP_SHIFT         24
#define K_MAP_IDX(k)        (((k) & K_MAP_MASK) >> K_MAP_SHIFT)
#define K_MAP_ENC(i)        ((i) << K_MAP_SHIFT)

    /* token classes proper */

#define K_NONE      (  0 )                  /* reserve 0 for "none" */

#define K_IDENT     (  1 | K_DTOR )         /* text: identifier */
#define K_STRLIT    (  2 )                  /* text: string literal */
#define K_ICON      (  3 )                  /* i: int constant */
#define K_UCON      (  4 )                  /* u: unsigned constant */
#define K_LCON      (  5 )                  /* i: long constant */
#define K_ULCON     (  6 )                  /* u: unsigned long constant */
#define K_FCON      (  7 )                  /* f: float constant */
#define K_DCON      (  8 )                  /* f: double constant */
#define K_LDCON     (  9 )                  /* f: long double constant */
                        
#define K_HASH      ( 10 )    /* these pseudo-tokens never */   /* # */
#define K_NL        ( 11 )    /* appear outside the lexer */    /* \n */

#define K_LPAREN    ( 12 | K_DTOR )                                 /* ( */
#define K_RPAREN    ( 13 )                                          /* ) */
#define K_LBRACK    ( 14 )                                          /* [ */
#define K_RBRACK    ( 15 )                                          /* ] */
#define K_LBRACE    ( 16 )                                          /* { */
#define K_RBRACE    ( 17 )                                          /* } */
#define K_DOT       ( 18 )                                          /* . */
#define K_ELLIP     ( 19 )                                          /* ... */
#define K_XOR       ( 20 | K_MAP_ENC(11) | K_PREC_XOR )             /* ^ */
#define K_COMMA     ( 21 | K_DECL_LIST )                            /* , */
#define K_COLON     ( 22 | K_MAP_ENC(29) )                          /* : */
#define K_SEMI      ( 23 | K_DECL_LIST )                            /* ; */
#define K_QUEST     ( 24 )                                          /* ? */
#define K_TILDE     ( 25 )                                          /* ~ */
#define K_ARROW     ( 26 )                                          /* -> */
#define K_INC       ( 27 )                                          /* ++ */
#define K_DEC       ( 28 )                                          /* -- */
#define K_NOT       ( 29 )                                          /* ! */
#define K_DIV       ( 30 | K_MAP_ENC(12) | K_PREC_MUL )             /* / */
#define K_MUL       ( 31 | K_MAP_ENC(13) | K_PREC_MUL | K_DTOR )    /* * */
#define K_PLUS      ( 32 | K_MAP_ENC(14) | K_PREC_ADD )             /* + */
#define K_MINUS     ( 33 | K_MAP_ENC(15) | K_PREC_ADD )             /* - */
#define K_GT        ( 34 | K_MAP_ENC(16) | K_PREC_REL )             /* > */
#define K_SHR       ( 35 | K_MAP_ENC(17) | K_PREC_SHIFT )           /* >> */
#define K_GTEQ      ( 36 | K_MAP_ENC(18) | K_PREC_REL )             /* >= */
#define K_SHREQ     ( 37 | K_MAP_ENC( 7) | K_PREC_ASG )             /* >>= */
#define K_LT        ( 38 | K_MAP_ENC(19) | K_PREC_REL )             /* < */
#define K_SHL       ( 39 | K_MAP_ENC(20) | K_PREC_SHIFT )           /* << */
#define K_LTEQ      ( 40 | K_MAP_ENC(21) | K_PREC_REL )             /* <= */
#define K_SHLEQ     ( 41 | K_MAP_ENC( 6) | K_PREC_ASG )             /* <<= */
#define K_AND       ( 42 | K_MAP_ENC(22) | K_PREC_AND )             /* & */
#define K_LAND      ( 43 | K_MAP_ENC(23) | K_PREC_LAND )            /* && */
#define K_ANDEQ     ( 44 | K_MAP_ENC( 8) | K_PREC_ASG )             /* &= */
#define K_OR        ( 45 | K_MAP_ENC(26) | K_PREC_OR )              /* | */
#define K_LOR       ( 46 | K_MAP_ENC(27) | K_PREC_LOR )             /* || */
#define K_OREQ      ( 47 | K_MAP_ENC( 9) | K_PREC_ASG )             /* |= */
#define K_MINUSEQ   ( 48 | K_MAP_ENC( 5) | K_PREC_ASG )             /* -= */
#define K_PLUSEQ    ( 49 | K_MAP_ENC( 4) | K_PREC_ASG )             /* += */
#define K_MULEQ     ( 50 | K_MAP_ENC( 1) | K_PREC_ASG )             /* *= */
#define K_DIVEQ     ( 51 | K_MAP_ENC( 2) | K_PREC_ASG )             /* /= */
#define K_EQEQ      ( 52 | K_MAP_ENC(24) | K_PREC_EQ )              /* == */
#define K_NOTEQ     ( 53 | K_MAP_ENC(25) | K_PREC_EQ )              /* != */
#define K_MOD       ( 54 | K_MAP_ENC(28) | K_PREC_MUL )             /* % */
#define K_MODEQ     ( 55 | K_MAP_ENC( 3) | K_PREC_ASG )             /* %= */
#define K_XOREQ     ( 56 | K_MAP_ENC(10) | K_PREC_ASG )             /* ^= */
#define K_EQ        ( 57 | K_MAP_ENC( 0) | K_PREC_ASG )             /* = */

    /* the base values of keywords should come last, or
       the token-printing logic in lex.c will break */

#define K_ASM       ( 58 )                                      /* __asm */
#define K_AUTO      ( 59 | K_DECL )
#define K_BREAK     ( 60 ) 
#define K_CASE      ( 61 ) 
#define K_CHAR      ( 62 | K_DECL | K_SPEC_CHAR )
#define K_CONST     ( 63 | K_DECL ) 
#define K_CONTINUE  ( 64 ) 
#define K_DEFAULT   ( 65 ) 
#define K_DO        ( 66 ) 
#define K_DOUBLE    ( 67 | K_DECL | K_SPEC_DOUBLE )
#define K_ELSE      ( 68 ) 
#define K_ENUM      ( 69 | K_DECL ) 
#define K_EXTERN    ( 70 | K_DECL )
#define K_FLOAT     ( 71 | K_DECL | K_SPEC_FLOAT )
#define K_FOR       ( 72 ) 
#define K_GOTO      ( 73 ) 
#define K_IF        ( 74 ) 
#define K_INT       ( 75 | K_DECL | K_SPEC_INT )
#define K_LONG      ( 76 | K_DECL | K_SPEC_LONG )
#define K_REGISTER  ( 77 | K_DECL )
#define K_RETURN    ( 78 ) 
#define K_SHORT     ( 79 | K_DECL | K_SPEC_SHORT )
#define K_SIGNED    ( 80 | K_DECL | K_SPEC_SIGNED )
#define K_SIZEOF    ( 81 ) 
#define K_STATIC    ( 82 | K_DECL )
#define K_STRUCT    ( 83 | K_DECL ) 
#define K_SWITCH    ( 84 ) 
#define K_TYPEDEF   ( 85 | K_DECL )
#define K_UNION     ( 86 | K_DECL ) 
#define K_UNSIGNED  ( 87 | K_DECL | K_SPEC_UNSIGNED )
#define K_VOID      ( 88 | K_DECL | K_SPEC_VOID ) 
#define K_VOLATILE  ( 89 | K_DECL ) 
#define K_WHILE     ( 90 )

extern int k_decl(struct token);

#endif /* LEX_H */

/* vi: set ts=4 expandtab: */
