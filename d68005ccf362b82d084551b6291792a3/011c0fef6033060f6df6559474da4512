/* stmt.c - statements                                  ncc, the new c compiler

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
#include "cc1.h"
#include "decl.h"
#include "expr.h"
#include "gen.h"
#include "lex.h"
#include "con.h"
#include "symbol.h"
#include "tree.h"
#include "target.h"
#include "stmt.h"

static void statement(void);

static struct block *switch_block;
static struct block *break_block;
static struct block *continue_block;
static struct block *switch_block;
static struct block *default_block;
static bool saw_default;

/* compound-statement   :
        '{' [ declaration-list ] [ statement-list ] '}'

   note that the caller is responsible for entering and exiting the scope */

void compound_statement(void)
{
    lex_match(K_LBRACE);
    local_declarations();

    while (token.k != K_RBRACE)
        statement();

    lex_match(K_RBRACE);
}

/* read a boolean expression in parentheses and generate
   a branch to one of two blocks based on its value. */

static void condition(struct block *true, struct block *false)
{
    struct tree *tree;

    lex_match(K_LPAREN);
    tree = expression();
    tree = test_expression(tree, K_NOTEQ);
    tree = gen(tree, GEN_FLAG_ROOT);
    gen_branch(tree, true, false);
    tree_free(tree);
    lex_match(K_RPAREN);
}

/* do-statement : 'do' statement while '(' expression ')' ';' */

static void do_statement(void)
{
    struct block *saved_continue = continue_block;
    struct block *saved_break = break_block;
    struct block *body_block = block_new();

    break_block = block_new();
    continue_block = block_new();
    block_add_successor(current_block, CC_ALWAYS, body_block);
    current_block = body_block;

    lex();
    statement();
    block_add_successor(current_block, CC_ALWAYS, continue_block);
    current_block = continue_block;

    lex_match(K_WHILE);
    condition(body_block, break_block);
    lex_match(K_SEMI);
    current_block = break_block;

    continue_block = saved_continue;
    break_block = saved_break;
}

/* for-statement :
    'for' '(' [expression] ';' [expression] ';' [expression] ')' statement */

static void for_statement(void)
{
    struct block *saved_continue = continue_block;
    struct block *saved_break = break_block;
    struct block *test_block = block_new();
    struct block *body_block = block_new();
    struct tree *initial_tree = 0;
    struct tree *test_tree = 0;
    struct tree *step_tree = 0;

    continue_block = block_new();
    break_block = block_new();
    
    lex();
    lex_match(K_LPAREN);

    if (token.k != K_SEMI)
        initial_tree = expression();
    
    lex_match(K_SEMI);

    if (token.k != K_SEMI) {
        test_tree = expression();
        test_tree = test_expression(test_tree, K_NOTEQ);
    }

    lex_match(K_SEMI);

    if (token.k != K_RPAREN)
        step_tree = expression();

    lex_match(K_RPAREN);

    if (initial_tree)
        gen(initial_tree, GEN_FLAG_ROOT | GEN_FLAG_DISCARD);

    block_add_successor(current_block, CC_ALWAYS, test_block);
    current_block = test_block;

    if (test_tree) {
        test_tree = gen(test_tree, GEN_FLAG_ROOT);
        gen_branch(test_tree, body_block, break_block);
        tree_free(test_tree);
    } else
        block_add_successor(current_block, CC_ALWAYS, body_block);

    current_block = body_block;
    statement();
    block_add_successor(current_block, CC_ALWAYS, continue_block);
    current_block = continue_block;
    
    if (step_tree)
        gen(step_tree, GEN_FLAG_ROOT | GEN_FLAG_DISCARD);

    block_add_successor(current_block, CC_ALWAYS, test_block);

    current_block = break_block;
    continue_block = saved_continue;
    break_block = saved_break;
}

/* if-statement : 'if' '(' expression ')' statement [ 'else' statement ] */

static void if_statement(void)
{
    struct block *true_block = block_new();
    struct block *else_block = block_new();
    struct block *join_block = block_new();

    lex();
    condition(true_block, else_block);

    current_block = true_block;
    statement();
    block_add_successor(current_block, CC_ALWAYS, join_block);

    if (token.k == K_ELSE) {
        lex();
        current_block = else_block;
        statement();
        else_block = current_block;
    }

    block_add_successor(else_block, CC_ALWAYS, join_block);
    current_block = join_block;
}

/* while-statement  :   'while' '(' expression ')' statement */

static void while_statement(void)
{
    struct block *saved_break = break_block;
    struct block *saved_continue = continue_block;
    struct block *test_block = block_new();
    struct block *body_block = block_new();
    
    break_block = block_new();
    continue_block = test_block;
    
    block_add_successor(current_block, CC_ALWAYS, test_block);
    current_block = test_block;

    lex();
    condition(body_block, break_block);

    current_block = body_block;
    statement();
    block_add_successor(current_block, CC_ALWAYS, test_block);

    current_block = break_block;
    continue_block = saved_continue;
    break_block = saved_break;
}

/* return-statement :   'return' [ expression ] ';' */

static void return_statement(void)
{
    struct tree *tree = 0;
    size_t size;

    lex();

    if (token.k != K_SEMI) {
        tree = expression();
        tree = fake_assign(&func_ret_type, tree);
    } else {
        if (!TYPE_VOID(&func_ret_type))
            error(WARNING, "return value missing in non-void function");
    }

    if (TYPE_STRUN(&func_ret_type)) {
        size = type_sizeof(&func_ret_type, 0);
        tree = tree_addrof(tree);
        tree = gen(tree, GEN_FLAG_ROOT);
        EMIT(insn_new(I_BLKCOPY, operand_sym(func_strun_ret),
                                 operand_leaf(tree),
                                 operand_i(target->ptr_uint, size, 0)));
    } else {
        if (tree)
            tree = gen(tree, GEN_FLAG_ROOT);

        EMIT(insn_new(I_RETURN, tree ? operand_leaf(tree) : 0));
    }

    block_add_successor(current_block, CC_ALWAYS, exit_block);
    current_block = block_new();
    lex_match(K_SEMI);
    tree_free(tree);
}

/* common code for 'break' and 'continue' statements */

static void loop_control(struct block *block)
{
    if (block == 0)
        error(FATAL, "misplaced %k statement", token.k);

    block_add_successor(current_block, CC_ALWAYS, block);
    current_block = block_new();
    lex();
    lex_match(K_SEMI);
}

/* switch-statement :   'switch' '(' expression ')' statement */

static void switch_statement(void)
{
    struct block *saved_switch = switch_block;
    struct block *saved_default = default_block;
    struct block *saved_break = break_block;
    bool saved_saw_default = saw_default;
    struct block *body_block = block_new();
    struct tree *tree;

    lex();
    lex_match(K_LPAREN);
    tree = expression();
        
    if (!TYPE_INTEGRAL(&tree->type))
        error(FATAL, "switch expression must be integral");

    if (TYPE_CHARS(&tree->type) || TYPE_SHORTS(&tree->type))
        tree = tree_cast_bits(tree, T_INT);

    lex_match(K_RPAREN);

    tree = gen(tree, GEN_FLAG_ROOT);

    switch_block = current_block;
    saw_default = FALSE;
    default_block = block_new();
    break_block = block_new();
    body_block = block_new();

    block_switch(current_block, tree, default_block);
    tree_free(tree);
    
    current_block = body_block;
    statement();
    block_add_successor(current_block, CC_ALWAYS, break_block);

    if (!saw_default)
        block_add_successor(default_block, CC_ALWAYS, break_block);

    block_switch_done(switch_block);

    current_block = break_block;
    switch_block = saved_switch;
    break_block = saved_break;
    default_block = saved_default;
    saw_default = saved_saw_default;
}

/* ensure we're in a switch statement,
   or error out. */

static void in_switch(void)
{
    if (switch_block == 0)
        error(FATAL, "misplaced %k (not in switch)", token.k);
}

/* case-label   :   'case' expression ':' statement */

static void case_label(void)
{
    struct block *block;
    union con con;

    lex();
    con.i = int_expression(switch_block->control->ts, INT_EXPRESSION_WARN);
    con_normalize(switch_block->control->ts, &con);
    lex_match(K_COLON);
    block = block_new();
    block_switch_case(switch_block, con.i, block);
    block_add_successor(current_block, CC_ALWAYS, block);
    current_block = block;
}

/* default-case :   'default' ':' statement */

static void default_case(void)
{
    if (saw_default)
        error(FATAL, "duplicate default case");

    lex();
    lex_match(K_COLON);
    saw_default = TRUE;
    block_add_successor(current_block, CC_ALWAYS, default_block);
    current_block = default_block;
}

/* goto-statement   :   'goto' identifier ';' */

static void goto_statement(void)
{
    struct block *label;

    lex();
    lex_expect(K_IDENT);
    label = label_goto(token.text);
    block_add_successor(current_block, CC_ALWAYS, label);
    current_block = block_new();
    lex();
    lex_match(K_SEMI);
}

static void statement(void)
{
    struct block *label;
    struct token peek;

again:

    switch (token.k)
    {
    case K_LBRACE:
        scope_enter();
        compound_statement();
        scope_exit();
        break;

    case K_IDENT:
        peek = lex_peek();

        if (peek.k == K_COLON) {
            label = label_define(token.text);
            lex();
            lex();
            block_add_successor(current_block, CC_ALWAYS, label);
            current_block = label;
            goto again;
        }

    default:
        gen(expression(), GEN_FLAG_ROOT | GEN_FLAG_DISCARD);
        
    case K_SEMI:
        lex_match(K_SEMI);
        break;

    case K_GOTO:            goto_statement(); break;
    case K_DO:              do_statement(); break;
    case K_FOR:             for_statement(); break;
    case K_IF:              if_statement(); break;
    case K_WHILE:           while_statement(); break;
    case K_BREAK:           loop_control(break_block); break;
    case K_CONTINUE:        loop_control(continue_block); break;
    case K_RETURN:          return_statement(); break;

    case K_SWITCH:          switch_statement(); break;
    case K_DEFAULT:         in_switch(); default_case(); goto again; 
    case K_CASE:            in_switch(); case_label(); goto again;
    }
}

/* vi: set ts=4 expandtab: */
