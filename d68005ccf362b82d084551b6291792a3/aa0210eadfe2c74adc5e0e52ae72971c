/* tree.c - expression trees                            ncc, the new c compiler

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

#include <stdlib.h>
#include "cc1.h"
#include "con.h"
#include "cast.h"
#include "field.h"
#include "symbol.h"
#include "output.h"
#include "type.h"
#include "sign.h"
#include "algebra.h"
#include "tree.h"

/* allocate a new tree node properly
   initialized for the given op */

static struct tree *tree_new(tree_op op)
{
    struct tree *tree;

    tree = safe_malloc(sizeof(struct tree));

    tree->op = op;
    TYPE_INIT(&tree->type);

    if (TREE_UNARY(tree))
        FOREST_INIT(&tree->args);
    
    return tree;
}

/* release a tree's resources and free it */

void tree_free(struct tree *tree)
{
    if (tree) {
        if (TREE_UNARY(tree)) {
            tree_free(tree->child);
            forest_clear(&tree->args);
        } else if (TREE_BINARY(tree)) {
            tree_free(tree->left);
            tree_free(tree->right);
        }

        type_clear(&tree->type);
        free(tree);
    }
}

/* empty out a forest */

void forest_clear(struct forest *f)
{
    struct tree *tree;

    while (tree = FOREST_FIRST(f)) {
        FOREST_REMOVE(f, tree);
        tree_free(tree);
    }
}

/* swap the left and right operands of a tree. */

void tree_commute(struct tree *tree)
{
    struct tree *tmp;

    tmp = tree->left;
    tree->left = tree->right;
    tree->right = tmp;
}

/* convenience functions for creating E_CON
   tree nodes. the type_bits (if present) must
   be appropriate to the type of the argument. */

struct tree *tree_v(void)
{
    struct tree *tree;
    
    tree = tree_new(E_NONE);
    type_append_bits(&tree->type, T_VOID);

    return tree;
}

struct tree *tree_i(type_bits ts, long i)
{
    struct tree *tree;

    tree = tree_new(E_CON);
    tree->con.i = i;
    type_append_bits(&tree->type, ts);

    return tree;
}

struct tree *tree_f(type_bits ts, double f)
{
    struct tree *tree;

    tree = tree_new(E_CON);
    tree->con.f = f;
    type_append_bits(&tree->type, ts);

    return tree;
}

/* another convenience to create an E_SYM that
   references the given symbol. the symbol is
   presumed to be an object or function. */

struct tree *tree_sym(struct symbol *sym)
{
    struct tree *tree;

    tree = tree_new(E_SYM);
    tree->sym = sym;
    type_copy(&tree->type, &sym->type);

    return tree;
}

/* create a tree for a unary op with
   the specified child */

struct tree *tree_unary(tree_op op, struct tree *child)
{
    struct tree *tree;

    tree = tree_new(op);
    tree->child = child;

    return tree;
}

/* create a tree for a binary op with
   the specified descendants */

struct tree *tree_binary(tree_op op, struct tree *left, 
                         struct tree *right)
{
    struct tree *tree;

    tree = tree_new(op);
    tree->left = left;
    tree->right = right;

    return tree;
}

/* place an rvalue barrier on top of tree */

struct tree *tree_rvalue(struct tree *tree)
{
    tree = tree_unary(E_RVALUE, tree);
    type_copy(&tree->type, &tree->child->type);

    return tree;
}

/* assigns the type of the root of the tree to its
   child, then chops the root off and frees it. */

struct tree *tree_chop_unary(struct tree *tree)
{
    struct tree *child;

    child = tree->child;
    tree->child = 0;
    type_clear(&child->type);
    TYPE_CONCAT(&child->type, &tree->type);

    tree_free(tree);
    return child;
}

/* assigns the type of the root of the tree to its 
   left child, then chops off the root and right
   child and frees them. */

struct tree *tree_chop_binary(struct tree *tree)
{
    struct tree *left;

    left = tree->left;
    tree->left = 0;
    type_clear(&left->type);
    TYPE_CONCAT(&left->type, &tree->type);
    tree_free(tree);
    return left;
}

/* fetch a value through a pointer with E_FETCH,
   then normalize:

        1. if the child is an E_CON tree that was
           created by tree_addrof(), turn it back
           into an E_SYM.
        2. if the child is an E_ADDROF, squash the
           E_FETCH and E_ADDROF undoing them.

   normalization is suppressed if TREE_FETCH_FORCE:
   this is only used to force fetches for volatiles. */

struct tree *tree_fetch(struct tree *tree, tree_fetch_flags flags)
{
    tree = tree_unary(E_FETCH, tree);
    type_deref(&tree->type, &tree->child->type);
    TYPE_UNFIELD(&tree->type);

    if (!(flags & TREE_FETCH_FORCE)) {
        if ((tree->child->op == E_CON)
          && (tree->child->sym)
          && (tree->child->con.i == 0)) {
            tree = tree_chop_unary(tree);
            tree->op = E_SYM;
        } else if (tree->child->op == E_ADDROF) {
            tree = tree_chop_unary(tree);
            tree = tree_chop_unary(tree);
        }
    }

    return tree;
}

/* take the address of a tree with E_ADDROF,
   then normalize:

        1. if the child is a E_FETCH tree, squash
           the E_ADDROF and the E_FETCH, or
        2. if the child represents a symbol known to
           the assembler, rewrite it as an E_CON. */

struct tree *tree_addrof(struct tree *tree)
{
    tree = tree_unary(E_ADDROF, tree);
    type_ref(&tree->type, &tree->child->type);

    if ((tree->child->op == E_SYM)
      && (tree->child->sym->ss & (S_EXTERN | S_STATIC))) {
        tree = tree_chop_unary(tree);
        tree->op = E_CON;
        con_normalize(TYPE_BASE(&tree->type), &tree->con);
    } else if (tree->child->op == E_FETCH) {
        tree = tree_chop_unary(tree);
        tree = tree_chop_unary(tree);
    }

    return tree;
}

/* create an E_CAST node to cast the
   tree to the type specified */

struct tree *tree_cast(struct tree *tree, struct type *type)
{
    tree = tree_unary(E_CAST, tree);
    type_copy(&tree->type, type);

    return tree;
}

/* create an E_CAST node to cast the tree to
   the type indicated by the type_bits */

struct tree *tree_cast_bits(struct tree *tree, type_bits ts)
{
    tree = tree_unary(E_CAST, tree);
    type_append_bits(&tree->type, ts);
    
    return tree;
}

/* return true if the tree is guaranteed to be
   zero or nonzero, respectively. note that
   tree_zero(x) is not always !tree_nonzero(x). */

int tree_nonzero(struct tree *tree)
{
    if (TREE_PURE_CON(tree)) {
        if (TYPE_FLOATING(&tree->type))
            return (tree->con.f != 0.0);
        else
            return (tree->con.i != 0);
    } else
        return 0;
}

int tree_zero(struct tree *tree)
{
    if (TREE_PURE_CON(tree)) {
        if (TYPE_FLOATING(&tree->type))
            return (tree->con.f == 0.0);
        else
            return (tree->con.i == 0);
    } else
        return 0;
}

/* normalize a tree. for the moment, this just
   means putting pure constants on the right of
   commutative binary operators. */

void tree_normalize(struct tree *tree)
{
    if ((tree->op & E_SWAP) && TREE_PURE_CON(tree->left))
        tree_commute(tree);
}

/* simplifying a tree does constant folding
   and removes E_RVALUE barriers. */

#define FOLD_UNARY_I(tr, OP)                                                \
    do {                                                                    \
        CON_FOLD_UNARY_I(TYPE_BITS(&(tr)->type),                            \
                         &(tr)->child->con,                                 \
                         &(tr)->child->con,                                 \
                         OP);                                               \
                                                                            \
        (tr) = tree_chop_unary(tr);                                         \
    } while (0)

#define FOLD_UNARY_ANY(tr, OP)                                              \
    do {                                                                    \
        CON_FOLD_UNARY(TYPE_BITS(&(tr)->type),                              \
                       &(tr)->child->con,                                   \
                       &(tr)->child->con,                                   \
                       OP);                                                 \
                                                                            \
        (tr) = tree_chop_unary(tr);                                         \
    } while (0)

#define FOLD_BINARY_ANY(tr, OP)                                             \
    do {                                                                    \
        CON_FOLD_BINARY_ANY(TYPE_BITS(&(tr)->type),                         \
                            &(tr)->left->con,                               \
                            &(tr)->left->con,                               \
                            &(tr)->right->con,                              \
                            OP);                                            \
                                                                            \
        (tr) = tree_chop_binary(tr);                                        \
    } while (0)

#define FOLD_BINARY_I(tr, OP)                                               \
    do {                                                                    \
        CON_FOLD_BINARY_I(TYPE_BITS(&(tr)->type),                           \
                            &(tr)->left->con,                               \
                            &(tr)->left->con,                               \
                            &(tr)->right->con,                              \
                            OP);                                            \
                                                                            \
        (tr) = tree_chop_binary(tr);                                        \
    } while (0)

#define FOLD_EQUALITY(tr, OP)                                               \
    do {                                                                    \
        int RESULT;                                                         \
                                                                            \
        if (TYPE_FLOATING(&((tr)->type)))                                   \
            RESULT = ((tr)->left->con.f OP (tr)->right->con.f);             \
        else                                                                \
            RESULT = ((tr)->left->con.i OP (tr)->right->con.i);             \
                                                                            \
        tree_free(tr);                                                      \
        (tr) = tree_i(T_INT, RESULT);                                       \
    } while (0)

#define FOLD_RELATIONAL(tr, OP)                                             \
    do {                                                                    \
        int RESULT;                                                         \
                                                                            \
        if (TYPE_FLOATING(&((tr)->type)))                                   \
            RESULT = ((tr)->left->con.f OP (tr)->right->con.f);             \
        else if (TYPE_SIGNED(&((tr)->type)))                                \
            RESULT = ((tr)->left->con.i OP (tr)->right->con.i);             \
        else                                                                \
            RESULT = ((tr)->left->con.u OP (tr)->right->con.u);             \
                                                                            \
        tree_free(tr);                                                      \
        (tr) = tree_i(T_INT, RESULT);                                       \
    } while (0)

static struct tree *simplify0(struct tree *tree)
{
    if (TREE_UNARY(tree)) {
        switch (tree->op)
        {
        case E_RVALUE:  tree = tree_chop_unary(tree);
                        return tree;
        }

        if (TREE_PURE_CON(tree->child)) {
            switch (tree->op)
            {
            case E_NEG:     FOLD_UNARY_ANY(tree, -); return tree;
            case E_COM:     FOLD_UNARY_I(tree, ~); return tree;

            case E_CAST:    con_cast(TYPE_BITS(&tree->type),
                                     &tree->child->con,
                                     TYPE_BITS(&tree->child->type),
                                     &tree->child->con);

                            tree = tree_chop_unary(tree);
                            return tree;
            }
        }
    } else if (TREE_BINARY(tree)) {
        tree_normalize(tree);

        if (tree_nonzero(tree->left)) {
            switch (tree->op)
            {
            case E_QUEST:   tree_commute(tree);
                            tree = tree_chop_binary(tree);
                            return tree_chop_binary(tree);

            case E_LAND:    tree_commute(tree);
                            return tree_chop_binary(tree);

            case E_LOR:     tree_free(tree);
                            return tree_i(T_INT, 1);
            }
        }

        if (tree_zero(tree->left)) {
            switch (tree->op)
            {
            case E_QUEST:   tree_commute(tree);
                            tree = tree_chop_binary(tree);
                            tree_commute(tree);
                            return tree_chop_binary(tree);

            case E_LAND:    tree_free(tree);
                            return tree_i(T_INT, 0);

            case E_LOR:     tree_commute(tree);
                            return tree_chop_binary(tree);
            }
        }

        /* see fold.c for an explanation of the mechanics behind the
           addition/subtraction of mixed pure and impure constants */ 

        if ((tree->op == E_SUB) && TREE_CON(tree->left)
          && TREE_CON(tree->right) && (tree->left->sym == tree->right->sym))
            tree->left->sym = tree->right->sym = 0;

        if (TREE_PURE_CON(tree->right)) {
            if (TREE_CON(tree->left)) {
                switch (tree->op)
                {
                case E_ADD:     FOLD_BINARY_ANY(tree, +); return tree;
                case E_SUB:     FOLD_BINARY_ANY(tree, -); return tree;
                }
            }

            if (TREE_PURE_CON(tree->left)) {
                switch (tree->op)
                {
                case E_DIV:     if (!tree_zero(tree->right))
                                    FOLD_BINARY_ANY(tree, /);

                                return tree;

                case E_MOD:     if (!tree_zero(tree->right))
                                    FOLD_BINARY_I(tree, %);

                                return tree;

                case E_MUL:     FOLD_BINARY_ANY(tree, *); return tree;
                case E_SHL:     FOLD_BINARY_I(tree, <<); return tree;
                case E_SHR:     FOLD_BINARY_I(tree, >>); return tree;
                case E_AND:     FOLD_BINARY_I(tree, &); return tree;
                case E_OR:      FOLD_BINARY_I(tree, |); return tree;
                case E_XOR:     FOLD_BINARY_I(tree, ^); return tree;
                case E_EQ:      FOLD_EQUALITY(tree, ==); return tree;
                case E_NEQ:     FOLD_EQUALITY(tree, !=); return tree;
                case E_GT:      FOLD_RELATIONAL(tree, >); return tree;
                case E_GTEQ:    FOLD_RELATIONAL(tree, >=); return tree;
                case E_LT:      FOLD_RELATIONAL(tree, <); return tree;
                case E_LTEQ:    FOLD_RELATIONAL(tree, <=); return tree;
                }
            }
        }
    }

    return tree;
}

/* descend through TREE depth-first,
   invoking F on each node */

#define TREE_DESCEND(TREE, F)                                               \
    do {                                                                    \
        struct forest ARGS = FOREST_INITIALIZER(ARGS);                      \
        struct tree *ARG;                                                   \
                                                                            \
        if (TREE_UNARY(TREE)) {                                             \
            TREE->child = F(TREE->child);                                   \
                                                                            \
            while (ARG = FOREST_FIRST(&tree->args)) {                       \
                FOREST_REMOVE(&tree->args, ARG);                            \
                ARG = F(ARG);                                               \
                FOREST_APPEND(&ARGS, ARG);                                  \
            }                                                               \
                                                                            \
            FOREST_CONCAT(&TREE->args, &ARGS);                              \
        } else if (TREE_BINARY(TREE)) {                                     \
            TREE->left = F(TREE->left);                                     \
            TREE->right = F(TREE->right);                                   \
        }                                                                   \
    } while (0)

/* a simple depth-first post-order
   walk to simplify the whole tree */

struct tree *tree_simplify(struct tree *tree)
{
    TREE_DESCEND(tree, tree_simplify);
    return simplify0(tree);
}

/* right before gen() starts generating for an expression, we
   rewrite all remaining E_SYM nodes that refer to volatiles
   as E_FETCH/E_ADDROF/E_SYM trees, to force memory accesses. */

struct tree *tree_rewrite_volatile(struct tree *tree)
{
    if (tree->op == E_ADDROF)
        return tree;

    if ((tree->op == E_SYM) && TYPE_VOLATILE(&tree->type)) {
        tree = tree_addrof(tree);
        tree = tree_simplify(tree);
        tree = tree_fetch(tree, TREE_FETCH_FORCE);
        return tree;
    }

    TREE_DESCEND(tree, tree_rewrite_volatile);

    return tree;
}

/* called right before gen() processes a tree. we
   take the opportunity to do some optimizations
   that are easier on trees than linear IR. */

struct tree *tree_opt(struct tree *tree)
{
    TREE_DESCEND(tree, tree_opt);

    if (tree->op == E_CAST)
        tree = cast_tree_opt(tree);

    tree = field_tree_opt(tree);
    tree = sign_tree_opt(tree);
    tree = algebra_tree_opt(tree);

    return tree;
}

/* print a human-readable version of a tree to the output
   file (as a comment). keep the tree_op_text[] table in
   sync with the E_TEXT() values in tree.h. */

static char *tree_op_text[] =
{
    /*  0 */    "NONE",     "CON",      "SYM",      "CALL",     "CAST",
    /*  5 */    "FETCH",    "ADDROF",   "NEG",      "RVALUE",   "COM",
    /* 10 */    "ASG",      "MULASG",   "DIVASG",   "MODASG",   "ADDASG",
    /* 15 */    "SUBASG",   "SHLASG",   "SHRASG",   "ANDASG",   "ORASG",
    /* 20 */    "XORASG",   "XOR",      "DIV",      "MUL",      "ADD",
    /* 25 */    "SUB",      "GT",       "SHR",      "GTEQ",     "LT",
    /* 30 */    "SHL",      "LTEQ",     "AND",      "LAND",     "EQ",
    /* 35 */    "NEQ",      "OR",       "LOR",      "MOD",      "QUEST",
    /* 40 */    "COLON",    "POST",     "COMMA",    "BLKASG"
};

void tree_debug(struct tree *tree, int depth)
{
    struct tree *arg;
    int i;
    
    output("%s# ", (depth == 0) ? "\n" : "");
    
    for (i = 0; i < depth; ++i)
        output("    ");

    output("%s <%T> ", tree_op_text[E_TEXT(tree->op)], &tree->type);

    switch (tree->op)
    {
    case E_CON:     output("%C ", TYPE_BITS(&tree->type), tree->con);

    case E_SYM:     if (tree->sym)
                        output("%Z", tree->sym);

                    output("\n");
                    break;

    default:        output("\n");

                    if (TREE_UNARY(tree)) {
                        tree_debug(tree->child, depth + 1);
                            
                        for (arg = FOREST_FIRST(&tree->args); arg;
                          arg = FOREST_NEXT(arg))
                            tree_debug(arg, depth + 2);
                    } else if (TREE_BINARY(tree)) {
                        tree_debug(tree->left, depth + 1);
                        tree_debug(tree->right, depth + 1);
                    }
    }
}

/* vi: set ts=4 expandtab: */
