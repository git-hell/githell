/* evaluate.c - preprocessor expression evaluation      ncc, the new c compiler

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

#include "cpp.h"
#include "macro.h"
#include "token.h"
#include "evaluate.h"

/* rather than using a separate stack, we evaluate expressions
   in place in the token list, replacing (sub)expressions with
   TOKEN_INT/TOKEN_UINT as we go. */

static void expression(struct list *);

static void unary(struct list *list)
{
    struct token *t;

    t = list_first(list);

    if (t) switch (t->class)
    {
    case TOKEN_PLUS:
        list_pop(list, 0);
        unary(list);
        return;

    case TOKEN_MINUS:
        list_pop(list, 0);
        unary(list);
        t = list_first(list);
        t->u.i = -(t->u.i);
        return;

    case TOKEN_TILDE:
        list_pop(list, 0);
        unary(list);
        t = list_first(list);
        t->u.i = ~(t->u.i);
        return;

    case TOKEN_NOT:
        list_pop(list, 0);
        unary(list);
        t = list_first(list);
        t->u.i = !(t->u.i);
        return;

    case TOKEN_LPAREN:
        list_pop(list, 0);
        expression(list);
        t = list_first(list);
        t = list_next(t);

        if (!t || (t->class != TOKEN_RPAREN))
            error("unmatched parentheses in expression");

        list_drop(list, t);
        return;

    case TOKEN_CHAR:
        token_convert_char(t);
        return;
    
    case TOKEN_NUMBER:
        token_convert_number(t);
    case TOKEN_INT:
    case TOKEN_UINT:
        return;
    }

    error("malformed expression");
}

/* all the binary operators of interest to the preprocessor have the
   same associativity, so we deal with them all in one function */

static void binary(int prec, struct list *list)
{
    struct token *result;
    struct token *left;
    struct token *right;
    struct token *op;
    struct token *t;

    if (prec == TOKEN_PREC_NONE) {
        unary(list);
        return;
    }

    binary(TOKEN_PREC_NEXT(prec), list);

    for (;;) {
        left = list_first(list);
        op = list_next(left);
        if (!op || (TOKEN_PREC(op->class) != prec)) return;
        list_pop(list, &left);
        list_pop(list, &op);
        binary(TOKEN_PREC_NEXT(prec), list);
        list_pop(list, &right);
        result = token_int(0);

        if ((left->class == TOKEN_UINT) || (right->class == TOKEN_UINT))
            TOKEN_UNSIGN(result);

        switch (op->class)
        {
        case TOKEN_PLUS:    result->u.i = left->u.i + right->u.i; break;
        case TOKEN_MINUS:   result->u.i = left->u.i - right->u.i; break;
        case TOKEN_MUL:     result->u.i = left->u.i * right->u.i; break;
        case TOKEN_AND:     result->u.i = left->u.i & right->u.i; break;
        case TOKEN_OR:      result->u.i = left->u.i | right->u.i; break;
        case TOKEN_XOR:     result->u.i = left->u.i ^ right->u.i; break;
        case TOKEN_SHL:     result->u.i = left->u.i << right->u.i; break;

        case TOKEN_SHR:
            if (result->class == TOKEN_UINT)
                result->u.u = left->u.u >> right->u.u;
            else
                result->u.i = left->u.i >> right->u.i;

            break;

        case TOKEN_MOD:
            if (right->u.i == 0) error("modulus zero");

            if (result->class == TOKEN_UINT)
                result->u.u = left->u.u % right->u.u;
            else
                result->u.i = left->u.i % right->u.i;

            break;

        case TOKEN_DIV:
            if (right->u.i == 0) error("division by zero");

            if (result->class == TOKEN_UINT)
                result->u.u = left->u.u / right->u.u;
            else
                result->u.i = left->u.i / right->u.i;

            break;

        case TOKEN_GTEQ:
            if (result->class == TOKEN_UINT)
                result->u.i = left->u.u >= right->u.u;
            else
                result->u.i = left->u.i >= right->u.i;

            TOKEN_SIGN(result);
            break;

        case TOKEN_GT:
            if (result->class == TOKEN_UINT)
                result->u.i = left->u.u > right->u.u;
            else
                result->u.i = left->u.i > right->u.i;

            TOKEN_SIGN(result);
            break;

        case TOKEN_LT:
            if (result->class == TOKEN_UINT)
                result->u.i = left->u.u < right->u.u;
            else
                result->u.i = left->u.i < right->u.i;

            TOKEN_SIGN(result);
            break;

        case TOKEN_LTEQ:
            if (result->class == TOKEN_UINT)
                result->u.i = left->u.u <= right->u.u;
            else
                result->u.i = left->u.i <= right->u.i;

            TOKEN_SIGN(result);
            break;

        case TOKEN_EQEQ:
            result->u.i = left->u.i == right->u.i;
            TOKEN_SIGN(result);
            break;

        case TOKEN_NOTEQ:
            result->u.i = left->u.i != right->u.i;
            TOKEN_SIGN(result);
            break;

        case TOKEN_ANDAND:
            result->u.i = left->u.i && right->u.i;
            TOKEN_SIGN(result);
            break;

        case TOKEN_OROR:
            result->u.i = left->u.i || right->u.i;
            TOKEN_SIGN(result);
            break;

        default:
            error("CPP INTERNAL: unknown binary operator");
        }

        list_prepend(list, result);
        token_free(left);
        token_free(op);
        token_free(right);
    }

    unary(list);
}

/* conditional expression */

static void ternary(struct list *list)
{
    struct token *control;
    struct token *left;
    struct token *right;

    binary(TOKEN_PREC_LOR, list);
    control = list_first(list);

    if (list_next_is(list, control, TOKEN_QUEST)) {
        list_pop(list, &control);
        list_pop(list, 0);
        expression(list);
        list_pop(list, &left);

        if (!list_first_is(list, TOKEN_COLON))
            error("missing ':' after '?'");

        list_pop(list, 0);
        ternary(list);
        list_pop(list, &right);

        if (control->u.i) {
            list_prepend(list, left);
            token_free(right);
        } else {
            list_prepend(list, right);
            token_free(left);
        }

        token_free(control);
    }
}

/* top-level expression: comma operator */

static void expression(struct list *list)
{
    struct token *t;

    for (;;) {
        ternary(list);
        t = list_first(list);

        if (list_next_is(list, t, TOKEN_COMMA)) {
            list_pop(list, 0);
            list_pop(list, 0);
        } else
            break;
    }
}

/* replace all identifiers with the value 0 */

static void undefined(struct list *list)
{
    struct token *t;

    t = list_first(list);

    while (t) {
        if (t->class == TOKEN_IDENT) {
            t = list_drop(list, t);
            list_insert(list, t, token_int(0));
        } else
            t = list_next(t);
    }
}

/* evaluate all "defined" operations */

static void defined(struct list *list)
{
    struct macro *m;
    struct token *t;
    int parentheses;

    t = list_first(list);

    while (t) {
        if ((t->class == TOKEN_IDENT) && (m = macro_lookup(&t->u.text))
          && (m->flags & MACRO_PREDEF_DEFINED)) {
            t = list_drop(list, t);

            if (t && (t->class == TOKEN_LPAREN)) {
                t = list_drop(list, t);
                parentheses = 1;
            } else
                parentheses = 0;

            if (!t || (t->class != TOKEN_IDENT))
                error("missing identifier after 'defined'");

            if ((m = macro_lookup(&t->u.text)) && MACRO_DEFINED(m))
                list_insert(list, t, token_int(1));
            else
                list_insert(list, t, token_int(0));

            t = list_drop(list, t);

            if (parentheses) {
                if (!t || (t->class != TOKEN_RPAREN))
                    error("missing closing parenthesis after 'defined'");

                t = list_drop(list, t);
            }
        } else
            t = list_next(t);
    }
}

/* parse an expression and return its truth value */

int evaluate(struct list *list)
{
    struct token *t;
    int true;

    list_strip_all(list);
    defined(list);
    macro_replace_all(list);
    undefined(list);
    list_strip_all(list);
    expression(list);

    if (!list_first_is(list, TOKEN_INT) && !list_first_is(list, TOKEN_UINT))
        error("CPP INTERNAL: botched expression evaluation");

    t = list_first(list);
    true = (t->u.i != 0);
    list_drop(list, t);

    return true;
}

/* vi: set ts=4 expandtab: */
