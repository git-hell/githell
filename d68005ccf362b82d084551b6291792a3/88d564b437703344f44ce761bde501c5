/* expr.c - expression parsing                          ncc, the new c compiler

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

#include <limits.h>
#include "../common/util.h"
#include "cc1.h"
#include "decl.h"
#include "lex.h"
#include "tree.h"
#include "string.h"
#include "symbol.h"
#include "target.h"
#include "expr.h"

static struct tree *comma(void);
static struct tree *assignment(void);

/* perform standard promotions:

   1. functions become pointers to function,
   2. arrays become pointers to their first elements,
   3. char/short are promoted to full integer,
   4. type qualifiers are discarded.

   if PROMOTE_OLD is specified, then:

   6. floats are promoted to doubles. */

typedef int promote_mode; /* PROMOTE_* */

#define PROMOTE_NORMAL      0
#define PROMOTE_OLD         1

static struct tree *promote(struct tree *tree, promote_mode mode)
{
    type_bits base = TYPE_BASE(&tree->type);

    if (base & T_FUNC)
        tree = tree_addrof(tree);

    if (base & (T_CHARS | T_SHORTS))
        tree = tree_cast_bits(tree, T_INT);

    if ((mode == PROMOTE_OLD) && (base & T_FLOAT))
        tree = tree_cast_bits(tree, T_DOUBLE);

    if (TYPE_QUALS(&tree->type)) {
        tree = tree_cast(tree, &tree->type);
        TYPE_UNQUAL(&tree->type);
    }

    if (base & T_ARRAY) {
        tree = tree_addrof(tree);
        type_fix_array(&tree->type);
    }

    return tree;
}

/* perform the usual arithmetic conversions. this
   relies on the type_bits T_INT..T_LDOUBLE being
   adjacent and ordered appropriately. see type.h. */

static void usuals(struct tree *tree)
{
    type_bits ts;

    if (TYPE_ARITH(&tree->left->type) && TYPE_ARITH(&tree->right->type)) {
        for (ts = T_LDOUBLE; ts != T_INT; ts >>= 1)
            if ((TYPE_BASE(&tree->left->type) & ts)
              || (TYPE_BASE(&tree->right->type) & ts))
                break;

        if (!(TYPE_BASE(&tree->left->type) & ts)) 
            tree->left = tree_cast(tree->left, &tree->right->type);
        if (!(TYPE_BASE(&tree->right->type) & ts))
            tree->right = tree_cast(tree->right, &tree->left->type);
    }
}

/* ensure that the tree is an lvalue.
   the token_class is only used for
   reporting errors. */

typedef int lvalue_mode;        /* LVALUE_* */

#define LVALUE_ANY          0       /* any lvalue is fine */
#define LVALUE_MODIFIABLE   1       /* lvalue must be non-const */
#define LVALUE_NOT_REGISTER 2       /* lvalue can't be register */

static void lvalue(struct tree *tree, token_class k, lvalue_mode mode)
{
    if (!TREE_LVALUE(tree))
        error(FATAL, "operator %k requires an lvalue", k);

    if (((mode == LVALUE_NOT_REGISTER) && (tree->op == E_SYM)
      && (tree->sym->ss & S_REGISTER)))
        error(FATAL, "can't apply %k to register variable", k);

    if ((mode == LVALUE_MODIFIABLE) && TYPE_CONST(&tree->type))
        error(FATAL, "operator %k requires non-const operand", k);
}

/* if the tree is commutable and there's exactly one pointer
   operand, ensure that it is on the left. this is just a 
   form of canonicalization to simplify logic. */

static void pointer_left(struct tree *tree)
{
    if ((tree->op & E_SWAP) && TYPE_PTR(&tree->right->type))
        tree_commute(tree);
}

/* perform null constant conversion: if the left is 
   pointer and the right is the integral constant 0,
   cast the integral constant to the pointer type. */

static void null_constant(struct tree *tree)
{
    if (TYPE_PTR(&tree->left->type) && TYPE_INTEGRAL(&tree->right->type)) {
        tree->right = tree_simplify(tree->right);

        if (TREE_NULL_CON(tree->right)) 
            tree->right = tree_cast(tree->right, &tree->left->type);
    }
}

/* if one child is a void pointer, and the other is also 
   a pointer, bring them to a common pointer type: either
   the type of the left (PERMUTE_VOIDS_LEFT) or both to void
   pointer (PERMUTE_VOIDS_VOID). in either case, qualifiers
   are preserved as much as possible. */

typedef int permute_voids_mode; /* PERMUTE_VOIDS_* */

#define PERMUTE_VOIDS_LEFT      0
#define PERMUTE_VOIDS_VOID      1

static struct tree *permute0(struct tree *tree, struct type *type)
{
    tree = tree_cast(tree, type);
    type_requalify(&tree->type, &tree->child->type);

    return tree;
}

static void permute_voids(struct tree *tree, permute_voids_mode mode)
{
    struct tree *left = tree->left;
    struct tree *right = tree->right;
    struct type voidptr = TYPE_INITIALIZER(voidptr);

    if ((TYPE_PTR(&left->type) && TYPE_PTR(&right->type))
      && (TYPE_VOID_PTR(&left->type) || TYPE_VOID_PTR(&right->type))) {
        if (mode == PERMUTE_VOIDS_LEFT)
            tree->right = permute0(tree->right, &left->type);
        else {
            type_append_bits(&voidptr, T_PTR);
            type_append_bits(&voidptr, T_VOID);
            tree->left = permute0(tree->left, &voidptr);
            tree->right = permute0(tree->right, &voidptr);
            type_clear(&voidptr);
        }
    }
}

/* perform pointer scaling (if applicable) and assign an
   appropriate result type. if no pointers are involved,
   the result inherits its type from the left operand. */

static struct tree *scale(struct tree *tree)
{
    struct tree *left = tree->left;
    struct tree *right = tree->right;
    struct tree *icon;
    size_t size;

    pointer_left(tree);

    if ((tree->op & E_SCALE) && TYPE_PTR(&left->type)) {
        size = type_sizeof(&left->type, TYPE_SIZEOF_TARGET);
        icon = tree_i(target->ptr_int, size);

        if (!TYPE_PTR(&right->type)) {
            /* pointer +/- int */
            right = tree_cast_bits(right, target->ptr_int);
            right = tree_binary(E_MUL, right, icon);
            type_append_bits(&right->type, target->ptr_int);
            tree->right = right;
            type_copy(&tree->type, &left->type);
        } else {
            /* pointer - pointer */
            type_append_bits(&tree->type, target->ptr_int);
            tree = tree_binary(E_DIV, tree, icon);
            type_append_bits(&tree->type, target->ptr_int);
        }
    } else
        type_copy(&tree->type, &left->type);

    return tree;
}

/* rewrite an E_ASG node if it is a struct assignment.
   (left = right) becomes *(&left [BLKASG]= &right). */

static struct tree *fix_struct_assign(struct tree *tree)
{
    if (TYPE_STRUN(&tree->type))
    {
        tree->left = tree_addrof(tree->left);
        tree->right = tree_addrof(tree->right);

        tree->op = E_BLKASG;
        type_clear(&tree->type);
        type_copy(&tree->type, &tree->left->type);

        tree = tree_fetch(tree, 0);
        tree = tree_rvalue(tree);
    }

    return tree;
}

/* this map is indexed by K_MAP_IDX() of the operator's token
   class; keep it in sync with the definitions in lex.h. the
   idx and len are references to the operands[] array below. */

static struct { tree_op op; int idx; int len; } map[] =
{                                   
    /*  0 */    { E_ASG,        2,  3   },  { E_MULASG,     2,  1   },
    /*  2 */    { E_DIVASG,     2,  1   },  { E_MODASG,     0,  1   },
    /*  4 */    { E_ADDASG,     1,  2   },  { E_SUBASG,     1,  2   },
    /*  6 */    { E_SHLASG,     0,  1   },  { E_SHRASG,     0,  1   },
    /*  8 */    { E_ANDASG,     0,  1   },  { E_ORASG,      0,  1   },
    /* 10 */    { E_XORASG,     0,  1   },  { E_XOR,        0,  1   },
    /* 12 */    { E_DIV,        2,  1   },  { E_MUL,        2,  1   },
    /* 14 */    { E_ADD,        1,  2   },  { E_SUB,        1,  3   },
    /* 16 */    { E_GT,         2,  2   },  { E_SHR,        0,  1   },
    /* 18 */    { E_GTEQ,       2,  2   },  { E_LT,         2,  2   },
    /* 20 */    { E_SHL,        0,  1   },  { E_LTEQ,       2,  2   },
    /* 22 */    { E_AND,        0,  1   },  { E_LAND,       6,  1   },
    /* 24 */    { E_EQ,         2,  2   },  { E_NEQ,        2,  2   },
    /* 26 */    { E_OR,         0,  1   },  { E_LOR,        6,  1   },
    /* 28 */    { E_MOD,        0,  1   },  { E_COLON,      2,  4   }
};

/* valid operand combinations. these are used in (idx, len) tuples
   which are associated with operators in the above map[] */

static struct { type_bits left; type_bits right; } operands[] =
{
    /* 0 */     { T_INTEGRAL,               T_INTEGRAL                  },
    /* 1 */     { T_PTR,                    T_INTEGRAL                  },
    /* 2 */     { T_ARITH,                  T_ARITH                     },
    /* 3 */     { T_PTR,                    T_PTR                       },
    /* 4 */     { T_STRUN,                  T_STRUN                     },
    /* 5 */     { T_VOID,                   T_VOID                      },
    /* 6 */     { T_SCALAR,                 T_SCALAR                    }
};

static void check_operands(struct tree *left, struct tree *right,
                           token_class k)
{
    int idx = map[K_MAP_IDX(k)].idx;
    int len = map[K_MAP_IDX(k)].len;
    type_compat_ret ret = TYPE_COMPAT_OK;
    type_compat_flags flags = 0;
    int i;

    switch (k)
    {
    case K_COLON:   flags = TYPE_COMPAT_FLAG_MERGEQUALS; break;
    case K_EQ:      flags = TYPE_COMPAT_FLAG_MOREQUALS
                          | TYPE_COMPAT_FLAG_SKIP; break;
    default:        flags = 0;
    }

    for (i = 0; i < len; ++i)
        if ((TYPE_BASE(&left->type) & operands[idx + i].left)
          && (TYPE_BASE(&right->type) & operands[idx + i].right))
            break;

    if (i == len)
        ret = TYPE_COMPAT_ERROR;
    else if ((TYPE_PTR(&left->type) && TYPE_PTR(&right->type))
          || (TYPE_STRUN(&left->type) && TYPE_STRUN(&right->type)))
        ret = type_compat(&left->type, &right->type, flags);

    /* we need to jump through a few hoops here, because initializers,
       arguments, and return values are processed as faked assignments.
       we don't want to issue confusing/nonsensical error messages. */
    
    switch (ret)
    {
    case TYPE_COMPAT_ERROR:
        if (k == K_EQ)
            error(FATAL, "incompatible type");
        else
            error(FATAL, "incompatible operands to binary %k", k);

    case TYPE_COMPAT_DISCARD:
        error(FATAL, "discards qualifiers");
    }
}

/* given a binary operator and two operands, construct the
   appropriate tree. the caller -- usually binary() -- is
   concerned merely with syntax; we deal with semantics here */

static struct tree *map_binary(struct tree *left, struct tree *right,
                               token_class k)
{
    struct tree *tree;
    tree_op op = map[K_MAP_IDX(k)].op;

    right = promote(right, PROMOTE_NORMAL);
    tree = tree_binary(op, left, right);
    pointer_left(tree);

    if (op & E_NULLPTR) {
        null_constant(tree);
        permute_voids(tree, (k == K_COLON) ? PERMUTE_VOIDS_VOID
                                           : PERMUTE_VOIDS_LEFT);
    }
    
    check_operands(tree->left, tree->right, k);

    if (op & E_LOG) {
        tree->left = test_expression(tree->left, K_NOTEQ);
        tree->right = test_expression(tree->right, K_NOTEQ);
    }

    if (K_PREC(k) == K_PREC_ASG)
        if (TYPE_ARITH(&tree->left->type))
            tree->right = tree_cast_bits(tree->right,
                                         TYPE_BASE(&tree->left->type));

    if (!(op & E_UNUSUAL)) 
        usuals(tree);

    if (op & E_INT)
        type_append_bits(&tree->type, T_INT);
    else
        tree = scale(tree);

    return tree;
}

/* process pre- or post-increment/decrement operators.
   the op is either E_ADD or E_POST (pre- or post-) and
   k is either K_INC or K_DEC (increment or decrement). */

static struct tree *crement(struct tree *tree, tree_op op, token_class k)
{
    struct tree *addend;
    int inc = (k == K_INC) ? 1 : -1;

    lvalue(tree, k, LVALUE_MODIFIABLE);

    if (!TYPE_SCALAR(&tree->type))
        error(FATAL, "operator %k requires scalar type", k);

    if (TYPE_PTR(&tree->type))
        addend = tree_i(target->ptr_int, inc);
    else if (TYPE_INTEGRAL(&tree->type))
        addend = tree_i(TYPE_BASE(&tree->type), inc);
    else /* floating */
        addend = tree_f(TYPE_BASE(&tree->type), inc);

    tree = tree_binary(op, tree, addend);
    tree = scale(tree);
    return tree;
}

/* primary-expression
            :   <identifier>
            |   <constant>
            |   <string-literal>
            |   '(' expression ')'      */

static struct tree *primary(void)
{
    struct string *id;
    struct symbol *sym;
    struct tree *tree;

    switch (token.k)
    {
    case K_ICON:        tree = tree_i(T_INT, token.i); lex(); break;
    case K_UCON:        tree = tree_i(T_UINT, token.i); lex(); break;
    case K_LCON:        tree = tree_i(T_LONG, token.i); lex(); break;
    case K_ULCON:       tree = tree_i(T_ULONG, token.i); lex(); break;
    case K_FCON:        tree = tree_f(T_FLOAT, token.f); lex(); break;
    case K_DCON:        tree = tree_f(T_DOUBLE, token.f); lex(); break;
    case K_LDCON:       tree = tree_f(T_LDOUBLE, token.f); lex(); break;

    case K_STRLIT:      sym = string_symbol(token.text);
                        tree = tree_sym(sym);
                        lex();
                        break;

    case K_LPAREN:      lex();
                        tree = comma();
                        lex_match(K_RPAREN);
                        break;

    case K_IDENT:       id = token.text;
                        lex();

                        sym = symbol_lookup(current_scope, SCOPE_GLOBAL,
                                            token.text, S_NORMAL);
                        if (sym) {
                            if (sym->ss & S_CONST) {
                                tree = tree_i(T_INT, sym->value);
                                break;
                            }
                        } else {
                            if (token.k != K_LPAREN)
                                error(FATAL, "'%S' is unknown", id);

                            sym = symbol_implicit(id);
                        }
                
                        if (sym->ss & S_TYPEDEF)
                            error(FATAL, "'%S' is a typedef", id);
                    
                        sym->ss |= S_REFERENCED;
                        tree = tree_sym(sym);
                        break;

    default:            error(FATAL, "expression missing (got %k)", token.k);
    }

    return tree;
}

/* postfix-expression   
            :   primary-expression
            |   postfix-expression '[' expression ']'
            |   postfix-expression '(' [ argument-expression-list ] ')'
            |   postfix-expression '.' <identifier>
            |   postfix-expression '->' <identifier>
            |   postfix-expression '++'
            |   postfix-expression '--'     */

static struct tree *access(struct tree *tree)       /* member access */
{
    struct type member_type = TYPE_INITIALIZER(member_type);
    struct symbol *tag;
    struct symbol *member;
    int lvalue;

    tree = promote(tree, PROMOTE_NORMAL);

    if (token.k == K_DOT) {
        lvalue = TREE_LVALUE(tree);
        tree = tree_addrof(tree);
    } else
        lvalue = 1;

    if (TYPE_STRUN_PTR(&tree->type))
        tag = TYPE_STRUN_PTR_TAG(&tree->type);
    else
        error(FATAL, "illegal left side of %k operator", token.k);

    if (!(tag->ss & S_DEFINED))
        error(FATAL, "%A is an incomplete type", tag);
                        
    lex();
    lex_expect(K_IDENT);
    member = member_lookup(tag, token.text);

    if (member == 0)
        error(FATAL, "member '%S' not in %A", token.text, tag);

    type_copy(&member_type, &member->type);
    type_qualify(&member_type, TYPE_PTR_QUALS(&tree->type));
                        
    tree = tree_binary(E_ADD, tree,
                        tree_i(target->ptr_int, member->offset));

    type_ref(&tree->type, &member_type);
    type_clear(&member_type);
    lex();
    tree = tree_fetch(tree, 0);

    if (!lvalue) 
        tree = tree_rvalue(tree);

    return tree;
}

static struct tree *array(struct tree *tree)        /* array indexing */
{
    struct tree *index;

    tree = promote(tree, PROMOTE_NORMAL);
    lex_match(K_LBRACK);
    tree = tree_binary(E_ADD, tree, comma());
    lex_match(K_RBRACK);
    pointer_left(tree);

    if (!TYPE_PTR(&tree->left->type) || !TYPE_INTEGRAL(&tree->right->type))
        error(FATAL, "invalid operands to []");

    tree = scale(tree);
    tree = tree_fetch(tree, 0);
    return tree;
}

static struct tree *call(struct tree *tree)         /* function call */
{
    struct type func_type = TYPE_INITIALIZER(func_type);
    struct tree *arg;
    struct formal *formal;

    tree = promote(tree, PROMOTE_NORMAL);

    if (!TYPE_FUNC_PTR(&tree->type))
        error(FATAL, "() requires function or pointer-to-function");

    type_deref(&func_type, &tree->type);
    /* XXX - make sure function and all its arguments are complete */
    tree = tree_unary(E_CALL, tree);
    type_deref(&tree->type, &func_type);

    lex();
    formal = FORMALS_FIRST(&TYPE_FORMALS(&func_type));

    if (token.k != K_RPAREN) {
        for (;;) {
            arg = assignment();

            if (formal == 0) {
                if (TYPE_OLD_STYLE(&func_type) || TYPE_VARIADIC(&func_type))
                    arg = promote(arg, PROMOTE_OLD);
                else
                    error(FATAL, "too many function arguments");
            } else {
                arg = fake_assign(&formal->type, arg);
                formal = FORMALS_NEXT(formal);
            }

            FOREST_APPEND(&tree->args, arg);

            if (token.k == K_RPAREN)
                break;
            else
                lex_match(K_COMMA);
        }
    }

    lex_match(K_RPAREN);

    if (formal)
        error(FATAL, "not enough function arguments");

    type_clear(&func_type);
    return tree;
}

static struct tree *postfix(void)
{
    struct tree *tree;

    tree = primary();

    for (;;)
    {
        switch (token.k)
        {
        case K_INC: 
        case K_DEC:     tree = crement(tree, E_POST, token.k);
                        lex();
                        break;

        case K_DOT:     
        case K_ARROW:   tree = access(tree);
                        break;

        case K_LBRACK:  tree = array(tree);
                        break;

        case K_LPAREN:  tree = call(tree);
                        break;

        default:        return tree;
        }
    }
}

/* unary-expression
            :   postfix-expression
            |   '++' unary-expression
            |   '--' unary-expression
            |   unary-operator cast-expression
            |   'sizeof' unary-expression
            |   'sizeof' '(' type-name ')'

   unary-operator   :   '&' | '*' | '+' | '-' | '~' | '!'   */

static struct tree *cast(void);

static struct tree *unary(void)
{
    struct type type = TYPE_INITIALIZER(type);
    size_t size;
    struct tree *tree;
    token_class k;
    type_bits ts;
    tree_op op;

    switch (k = token.k)
    {
    case K_MINUS:   op = E_NEG; ts = T_ARITH; break;
    case K_PLUS:    op = E_RVALUE; ts = T_ARITH; break;
    case K_TILDE:   op = E_COM; ts = T_INTEGRAL; break;

    case K_MUL:     lex();
                    tree = cast();
                    tree = promote(tree, PROMOTE_NORMAL);

                    if (!TYPE_PTR(&tree->type) || TYPE_VOID_PTR(&tree->type))
                        error(FATAL, "can't dereference that");

                    return tree_fetch(tree, 0);

    case K_NOT:     lex();
                    tree = cast();
                    tree = test_expression(tree, K_EQEQ);
                    return tree;

    case K_AND:     lex();
                    tree = cast();
                    lvalue(tree, K_AND, LVALUE_NOT_REGISTER);

                    if ((tree->op == E_FETCH)
                      && (TYPE_FIELD_PTR(&tree->child->type)))
                        error(FATAL, "can't take address of bit field");

                    return tree_addrof(tree);

    case K_INC:
    case K_DEC:     lex();
                    return crement(unary(), E_ADDASG, k);

    case K_SIZEOF:  lex();

                    if (token.k == K_LPAREN) {
                        if (!k_decl(lex_peek())) {
                            tree = unary();
                            TYPE_CONCAT(&type, &tree->type);
                            tree_free(tree);
                        } else {
                            lex();
                            abstract_declarator(&type);
                            lex_match(K_RPAREN);
                        }
                    }

                    size = type_sizeof(&type, 0);
                    tree = tree_i(target->ptr_uint, size);
                    type_clear(&type);
                    return tree;

    default:        return postfix();
    }

    lex();
    tree = cast();
    tree = promote(tree, PROMOTE_NORMAL);

    if ((TYPE_BITS(&tree->type) & ts) == 0)
        error(FATAL, "illegal operand to unary %k", k);

    tree = tree_unary(op, tree);
    type_copy(&tree->type, &tree->child->type);
    return tree;
}

/* cast-expression
            :   unary-expression
            |   '(' type-name ')' cast-expression       */

static struct tree *cast(void)
{
    struct type type = TYPE_INITIALIZER(type);
    struct tree *tree;

    if ((token.k == K_LPAREN) && k_decl(lex_peek())) {
        lex_match(K_LPAREN);
        abstract_declarator(&type);
        lex_match(K_RPAREN);
        tree = cast();
        tree = promote(tree, PROMOTE_NORMAL);

        if (((!TYPE_SCALAR(&tree->type) || !TYPE_SCALAR(&type))
          || (TYPE_FLOATING(&tree->type) && TYPE_PTR(&type))
          || (TYPE_PTR(&tree->type) && TYPE_FLOATING(&type)))
          && !TYPE_VOID(&type))
            error(FATAL, "invalid cast");

        tree = tree_cast(tree, &type);
        type_clear(&type);
        return tree;
    } else
        return unary();
}

/* binary-expression
            :   cast-expression
            |   binary-expression binary-operator cast-expression

   binary-operator  :   '*' | '/' | '%' | '+' | '-'
                    |   '<<' | '>>' | '<' | '>' | '>='
                    |   '<=' | '==' | '!=' | '&' | '^'
                    |   '|' | '&&' | '||'

   obviously this short grammar does not account for precedence. */

struct tree *binary(int prec)
{
    struct tree *left;
    struct tree *right;
    token_class k;

    if (prec == K_PREC_NONE)
        return cast();

    left = binary(K_PREC_NEXT(prec));

    while (K_PREC(token.k) == prec) {
        k = token.k;
        lex();
        right = binary(K_PREC_NEXT(prec));
        left = promote(left, PROMOTE_NORMAL);
        left = map_binary(left, right, k);
    }

    return left;
}

/* ternary-expression
            :   binary-expression
            |   binary-expression '?' expression ':' ternary-expression

   to avoid generating aggregate temporaries, when b and c are
   aggregates, we rewrite (a ? b : c) as *(a ? &b : &c). */

static struct tree *ternary(void)
{
    struct tree *tree;
    struct tree *left;
    struct tree *right;
    struct tree *colon;
    int strun = 0;

    tree = binary(K_PREC_LOR);
    
    if (token.k == K_QUEST) {
        tree = test_expression(tree, K_NOTEQ);
        lex();
        left = comma();
        left = promote(left, PROMOTE_NORMAL);
        lex_match(K_COLON);
        right = ternary();
        colon = map_binary(left, right, K_COLON);

        if (TYPE_STRUN(&colon->left->type)) {
            colon->left = tree_addrof(colon->left);
            colon->right = tree_addrof(colon->right);
            type_clear(&colon->type);
            type_copy(&colon->type, &colon->left->type);
            strun = 1;
        }

        tree = tree_binary(E_QUEST, tree, colon);
        type_copy(&tree->type, &colon->type);

        if (strun) {
            tree = tree_fetch(tree, 0);
            tree = tree_rvalue(tree);
        }
    }

    return tree;
}

/* assignment-expression
            :   ternary-expression
            |   unary-expression assignment-operator assignment-expression

   assignment-operator  :   '=' | '*=' | '/=' | '%=' | '+=' | '-='
                        |   '<<=' | '>>=' | '&=' | '|=' | '^='   */

static struct tree *assignment(void)
{
    struct tree *tree;
    struct tree *right;
    token_class k;

    tree = ternary();

    if (K_PREC(token.k) == K_PREC_ASG) {
        k = token.k;
        lex();
        right = assignment();
        lvalue(tree, k, LVALUE_MODIFIABLE);
        tree = map_binary(tree, right, k);
        tree = fix_struct_assign(tree);
    }

    return tree;
}

/* comma-expression
            :   assignment-expression
            |   comma-expression ',' assignment-expression

    if the right operand (the result) is an aggregate type, we rewrite
    the expression (a, b) as *(a, &b) to avoid burying the E_FETCH: the
    code generator will choke on the resulting E_ADDROF/E_FETCH stack. */

static struct tree *comma(void)
{
    struct tree *left;
    struct tree *right;
    bool rewrite;

    left = assignment();

    if (token.k == K_COMMA) {
        lex();
        right = comma();

        if (TYPE_STRUN(&right->type)) {
            right = tree_addrof(right);
            rewrite = TRUE;
        } else
            rewrite = FALSE;
            
        left = tree_binary(E_COMMA, left, right);
        type_copy(&left->type, &right->type);

        if (rewrite)
            left = tree_fetch(left, 0);
    }

    return left;
}

/* parse an expression: this is the primary
   interface to the rest of the system */

struct tree *expression(void)
{
    struct tree *tree;

    tree = comma();
    tree = tree_simplify(tree);

    return tree;
}

/* parse an initializer expression, that is, an
   assignment expression (no commas allowed). */

struct tree *init_expression(void)
{
    return assignment();
}

/* parse a constant expression, as needed for
   static initializers and such. */

struct tree *static_expression(void)
{
    struct tree *tree;

    tree = ternary();
    tree = promote(tree, PROMOTE_NORMAL);
    tree = tree_simplify(tree);

    if (!TREE_CON(tree))
        error(FATAL, "constant expression required");

    return tree;
}

/* parse an integral constant expression and return its value. a cursory
   (and hardly fool-proof) check is made to see that the value is in a
   reasonable range for the type specified in ts (T_INT, T_UINT, T_LONG,
   or T_ULONG). if the INT_EXPRESSION_WARN flag is specified, then issue
   a warning instead of an error when the range check fails. */

long int_expression(type_bits ts, int_expression_flags flags)
{
    struct tree *tree;
    error_type type;
    long i;

    type = (flags & INT_EXPRESSION_WARN) ? WARNING : FATAL;
    tree = static_expression();

    if (!TREE_PURE_CON(tree) || !TYPE_INTEGRAL(&tree->type))
        error(FATAL, "integer constant expression required");

    i = tree->con.i;

    if ((ts & T_INT) && (i >= INT_MIN) && (i <= INT_MAX))
        goto success;

    if ((ts & (T_UINT | T_INT)) && (i >= 0) && (i <= UINT_MAX))
        goto success;

    if (ts & (T_LONG | T_ULONG))
        goto success;

    error(type, "integer expression out of range");
success:
    tree_free(tree);
    return i;
}

/* parse a constant expression used for bit
   field sizes and array dimensions */

int size_expression(void)
{
    int i;

    i = int_expression(T_UINT, 0);

    if (i > MAX_OBJECT_SIZE)
        error(FATAL, "size expression out of range");

    return i;
}

/* test the tree against zero to get its truth value.
   with k == K_NOTEQ, this means true if not zero.
   alternatively k == K_EQEQ yields true if zero. */

struct tree *test_expression(struct tree *tree, token_class k)
{
    tree = promote(tree, PROMOTE_NORMAL);

    if (!TYPE_SCALAR(&tree->type))
        error(FATAL, "scalar expression required");
    
    tree = map_binary(tree, tree_i(T_INT, 0), k);
    return tree;
}

/* generate a tree that assigns the src tree to the dst
   tree, or error out if not possible. this is used for
   automatic initializers (amongst other things), so we
   can (and should) skip the check for modifiable lvalue. */

struct tree *assign(struct tree *dst, struct tree *src)
{
    struct tree *tree;

    tree = map_binary(dst, src, K_EQ);
    tree = fix_struct_assign(tree);
    return tree;
}

/* adjust the src tree to enable assigning it to the dst
   type, or error out if not possible. this is used for
   static initializers and function arguments, so again,
   we skip the check for a modifiable lvalue. */

struct tree *fake_assign(struct type *dst, struct tree *src)
{
    struct symbol *sym;
    struct tree *tree;

    sym = symbol_new(0, S_REGISTER);
    type_copy(&sym->type, dst);
    tree = map_binary(tree_sym(sym), src, K_EQ);
    tree_commute(tree);
    tree = tree_chop_binary(tree);
    symbol_free(sym);

    return tree_simplify(tree);
}

/* vi: set ts=4 expandtab: */
