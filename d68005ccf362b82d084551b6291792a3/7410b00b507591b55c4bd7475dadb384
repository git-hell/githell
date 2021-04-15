/* tree.h - expression trees                            ncc, the new c compiler

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

#ifndef TREE_H
#define TREE_H

#include "../common/tailq.h"
#include "cc1.h"
#include "type.h"

struct symbol;

/* expressions are represented in trees as one would expect. when
   trees are strung together (e.g., as function call arguments) we
   call it a forest. */

TAILQ_HEAD(forest, tree);  /* struct forest */

typedef int tree_op;    /* E_* below */

struct tree
{
    tree_op op;
    struct type type;

    union
    {
        struct                          /* E_CON / E_SYM */
        {
            union con con;
            struct symbol *sym;
        };

        struct                          /* unary ops */
        {
            struct tree *child;
            struct forest args;
        };

        struct                          /* binary ops */
        {
            struct tree *left;
            struct tree *right;
        };
    };

    TAILQ_ENTRY(tree) links;    /* in a forest */
};

/* tree represents the null constant */

#define TREE_NULL_CON(t)        (((t)->op == E_CON)                 \
                                && (TYPE_INTEGRAL(&(t)->type))      \
                                && ((t)->con.i == 0)                \
                                && ((t)->sym == 0))

/* tree is a fetch of a bitfield */

#define TREE_FIELD_FETCH(t)     (((t)->op == E_FETCH) &&            \
                                 TYPE_FIELD_PTR(&(t)->child->type))

/* tree is an lvalue */

#define TREE_LVALUE(t)          (((t)->op == E_SYM) || ((t)->op == E_FETCH))

#define TREE_LEAF(t)            ((t)->op & E_LEAF)
#define TREE_UNARY(t)           (!TREE_LEAF(t) && ((t)->op & E_UNARY))
#define TREE_BINARY(t)          (!TREE_LEAF(t) && !TREE_UNARY(t))

#define TREE_CON(t)             ((t)->op == E_CON)
#define TREE_PURE_CON(t)        (TREE_CON(t) && ((t)->sym == 0))

extern struct tree *tree_v(void);
extern struct tree *tree_i(type_bits, long);
extern struct tree *tree_f(type_bits, double);
extern struct tree *tree_sym(struct symbol *);
extern struct tree *tree_cast(struct tree *, struct type *);
extern struct tree *tree_cast_bits(struct tree *, type_bits);
extern struct tree *tree_unary(tree_op, struct tree *);
extern struct tree *tree_binary(tree_op, struct tree *, struct tree *);
extern struct tree *tree_addrof(struct tree *);
extern struct tree *tree_rvalue(struct tree *);
extern struct tree *tree_chop_unary(struct tree *);
extern struct tree *tree_chop_binary(struct tree *);
extern int tree_zero(struct tree *);
extern int tree_nonzero(struct tree *);
extern void tree_commute(struct tree *);
extern void tree_free(struct tree *);
extern void tree_normalize(struct tree *);
extern struct tree *tree_simplify(struct tree *);
extern struct tree *tree_rewrite_volatile(struct tree *);
extern struct tree *tree_opt(struct tree *);

typedef int tree_fetch_flags; /* TREE_FETCH_* */

#define TREE_FETCH_FORCE    ( 0x00000001 )

extern struct tree *tree_fetch(struct tree *, tree_fetch_flags);

#define FOREST_INITIALIZER(f)       TAILQ_HEAD_INITIALIZER(f)
#define FOREST_INIT(f)              TAILQ_INIT(f)
#define FOREST_FIRST(f)             TAILQ_FIRST(f)
#define FOREST_LAST(f)              TAILQ_LAST(f, forest)
#define FOREST_NEXT(t)              TAILQ_NEXT(t, links)
#define FOREST_PREPEND(f, t)        TAILQ_INSERT_HEAD(f, t, links)
#define FOREST_APPEND(f, t)         TAILQ_INSERT_TAIL(f, t, links)
#define FOREST_REMOVE(f, t)         TAILQ_REMOVE(f, t, links)
#define FOREST_CONCAT(dst, src)     TAILQ_CONCAT(dst, src, links)

extern void forest_clear(struct forest *);
extern void tree_debug(struct tree *, int);

/* flags to distinguish between union variants */

#define E_LEAF          ( 0x80000000 )
#define E_UNARY         ( 0x40000000 )

/* E_SWAP indicates if a binary operator is commutative */

#define E_SWAP          ( 0x20000000 )

/* E_SCALE means a binary operator is subject to pointer scaling */

#define E_SCALE         ( 0x10000000 )

/* E_UNUSUAL indicates the operands of a binary operator 
   are not subject to the usual promotions */

#define E_UNUSUAL       ( 0x08000000 )

/* E_NULLPTR means the operands of a binary operator may be null
   constants (0), and void pointers are compatible with others. */

#define E_NULLPTR       ( 0x04000000 )

/* E_INT means the result of a binary operation is always int */

#define E_INT           ( 0x02000000 )

/* E_LOG marks binary logical operators */

#define E_LOG           ( 0x01000000 )

/* E_2S (2's complement) marks operators that are
   friendly to cast elimination, see cast.c */

#define E_2S            ( 0x00800000 )

/* E_ASSOC marks associative operators */

#define E_ASSOC         ( 0x00400000 )

/* bits[11:8] hold indices into the bin[] table in gen.c. */

#define E_BIN_MASK      ( 0x00000F00 )
#define E_BIN_SHIFT     8
#define E_BIN_IDX(op)   (((op) & E_BIN_MASK) >> E_BIN_SHIFT)
#define E_BIN_ENC(i)    ((i) << E_BIN_SHIFT)

/* E_TEXT() is used to index the tree_op_text[] table in tree.c. */

#define E_TEXT(op)      ((op) & 0xFF)

#define E_NONE      (  0 | E_LEAF )
#define E_CON       (  1 | E_LEAF )
#define E_SYM       (  2 | E_LEAF )

#define E_CALL      (  3 | E_UNARY )                    /* function call */
#define E_CAST      (  4 | E_UNARY )                        /* ( cast )  */
#define E_FETCH     (  5 | E_UNARY )                        /*    *      */
#define E_ADDROF    (  6 | E_UNARY )                        /*    &      */
#define E_NEG       (  7 | E_UNARY | E_2S )                 /*    -      */
#define E_RVALUE    (  8 | E_UNARY )                        /*  barrier  */
#define E_COM       (  9 | E_UNARY | E_2S )                 /*    ~      */

#define E_ASG       ( 10 | E_UNUSUAL | E_NULLPTR )              /*  =    */
#define E_MULASG    ( 11 | E_BIN_ENC(2) | E_UNUSUAL )           /*  *=   */
#define E_DIVASG    ( 12 | E_BIN_ENC(1) | E_UNUSUAL )           /*  /=   */
#define E_MODASG    ( 13 | E_BIN_ENC(9) | E_UNUSUAL )           /*  %=   */
#define E_ADDASG    ( 14 | E_BIN_ENC(3) | E_UNUSUAL | E_SCALE ) /*  +=   */
#define E_SUBASG    ( 15 | E_BIN_ENC(4) | E_UNUSUAL | E_SCALE ) /*  -=   */
#define E_SHLASG    ( 16 | E_BIN_ENC(6) | E_UNUSUAL )           /*  <<=  */
#define E_SHRASG    ( 17 | E_BIN_ENC(5) | E_UNUSUAL )           /*  >>=  */
#define E_ANDASG    ( 18 | E_BIN_ENC(7) | E_UNUSUAL )           /*  &=   */
#define E_ORASG     ( 19 | E_BIN_ENC(8) | E_UNUSUAL )           /*  |=   */
#define E_XORASG    ( 20 | E_BIN_ENC(0) | E_UNUSUAL )           /*  ^=   */

#define E_XOR       ( 21 | E_BIN_ENC(0) | E_SWAP | E_2S | E_ASSOC ) /*  ^   */
#define E_DIV       ( 22 | E_BIN_ENC(1) )                           /*  /   */
#define E_MUL       ( 23 | E_BIN_ENC(2) | E_SWAP | E_2S | E_ASSOC ) /*  *   */

#define E_ADD       ( 24 | E_BIN_ENC(3) | E_SCALE | E_SWAP                  \
                         | E_2S | E_ASSOC )                         /*  +   */

#define E_SUB       ( 25 | E_BIN_ENC(4) | E_SCALE | E_2S )          /*  -   */
#define E_GT        ( 26 | E_INT )                                  /*  >   */
#define E_SHR       ( 27 | E_BIN_ENC(5) | E_UNUSUAL )               /*  >>  */
#define E_GTEQ      ( 28 | E_INT )                                  /*  >=  */
#define E_LT        ( 29 | E_INT )                                  /*  <   */
#define E_SHL       ( 30 | E_BIN_ENC(6) | E_UNUSUAL )               /*  <<  */
#define E_LTEQ      ( 31 | E_INT )                                  /*  <=  */
#define E_AND       ( 32 | E_BIN_ENC(7) | E_SWAP | E_2S | E_ASSOC ) /*  &   */
#define E_LAND      ( 33 | E_UNUSUAL | E_INT | E_LOG )              /*  &&  */
#define E_EQ        ( 34 | E_SWAP | E_NULLPTR | E_INT )             /*  ==  */
#define E_NEQ       ( 35 | E_SWAP | E_NULLPTR | E_INT )             /*  !=  */
#define E_OR        ( 36 | E_BIN_ENC(8) | E_SWAP | E_2S | E_ASSOC ) /*  |   */
#define E_LOR       ( 37 | E_UNUSUAL | E_INT | E_LOG )              /*  ||  */
#define E_MOD       ( 38 | E_BIN_ENC(9) )                           /*  %   */

    /* the ternary operator is represented by two tree nodes,
       an E_QUEST node with the controlling expression on the
       left, and an E_COLON node on the right. */

#define E_QUEST     ( 39 )                                      /*  ?   */
#define E_COLON     ( 40 | E_NULLPTR )                          /*  :   */

    /* the E_POST op is just like E_ADDASG, except that
       its result is the value before the addition. */

#define E_POST      ( 41 | E_BIN_ENC(3) | E_SCALE )           /* post ++/-- */

#define E_COMMA     ( 42 )                                      /*  ,   */

    /* block assignment: the children are pointers, and the tree type is
       a pointer whose target's size dictates the size of the copy; the
       result has the type of the tree and the value of the left child. */

#define E_BLKASG    ( 43 )                                    /* blk = */

#endif /* TREE_H */

/* vi: set ts=4 expandtab: */
