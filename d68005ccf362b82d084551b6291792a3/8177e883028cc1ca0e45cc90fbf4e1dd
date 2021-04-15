/* decl.c - declaration parsing                         ncc, the new c compiler

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

#include "cc1.h"
#include "lex.h"
#include "symbol.h"
#include "type.h"
#include "expr.h"
#include "init.h"
#include "stmt.h"
#include "insn.h"
#include "block.h"
#include "dealias.h"
#include "opt.h"
#include "gen.h"
#include "output.h"
#include "target.h"
#include "graph.h"
#include "switch.h"
#include "decl.h"

static void declarator(struct type *, struct string **);
static void strun_specifier(struct type *);

/* parse type qualifiers (if any), and add
   their type_bits to the quals set. */

static void qualifiers(type_bits *quals)
{
    for (;;)
    {
        switch (token.k)
        {
        case K_CONST:       *quals |= T_CONST; break;
        case K_VOLATILE:    *quals |= T_VOLATILE; break;

        default:            return;
        }

        lex();
    }
}

/* parse storage class specifier(s), if any. */

static void storage_specifiers(storage_class *ss)
{
    storage_class bit;

    for (;;)
    {
        switch (token.k)
        {
        case K_AUTO:        bit = S_AUTO; break;
        case K_REGISTER:    bit = S_REGISTER; break;
        case K_TYPEDEF:     bit = S_TYPEDEF; break;
        case K_EXTERN:      bit = S_EXTERN; break;
        case K_STATIC:      bit = S_STATIC; break;

        default:            return;
        }

        if (*ss) error(FATAL, "multiple storage class specifiers");
        *ss |= bit;
        lex();
    }
}

/* given a token_class that consists (only) of K_SPEC_* bits,
   append a node with the corresponding type_bits to t. */

static void map_specs(struct type *t, token_class specs)
{
    type_bits ts;

    switch (specs)
    {
    case K_SPEC_VOID:                                   ts = T_VOID; break;
    case K_SPEC_CHAR:                                   ts = T_CHAR; break;
    case K_SPEC_CHAR | K_SPEC_SIGNED:                   ts = T_SCHAR; break;
    case K_SPEC_CHAR | K_SPEC_UNSIGNED:                 ts = T_UCHAR; break;

    case K_SPEC_SHORT:
    case K_SPEC_SHORT | K_SPEC_SIGNED:
    case K_SPEC_SHORT | K_SPEC_INT:
    case K_SPEC_SHORT | K_SPEC_SIGNED | K_SPEC_INT:     ts = T_SHORT; break;

    case K_SPEC_SHORT | K_SPEC_UNSIGNED:
    case K_SPEC_SHORT | K_SPEC_UNSIGNED | K_SPEC_INT:   ts = T_USHORT; break;

    case K_SPEC_INT:
    case K_SPEC_SIGNED:
    case K_SPEC_SIGNED | K_SPEC_INT:                    ts = T_INT; break;

    case K_SPEC_UNSIGNED:
    case K_SPEC_UNSIGNED | K_SPEC_INT:                  ts = T_UINT; break;

    case K_SPEC_LONG: 
    case K_SPEC_LONG | K_SPEC_SIGNED:
    case K_SPEC_LONG | K_SPEC_INT:
    case K_SPEC_LONG | K_SPEC_SIGNED | K_SPEC_INT:      ts = T_LONG; break;

    case K_SPEC_UNSIGNED | K_SPEC_LONG:    
    case K_SPEC_UNSIGNED | K_SPEC_LONG | K_SPEC_INT:    ts = T_ULONG; break; 

    case K_SPEC_FLOAT:                                  ts = T_FLOAT; break;
    case K_SPEC_DOUBLE:                                 ts = T_DOUBLE; break;
    case K_SPEC_DOUBLE | K_SPEC_LONG:                   ts = T_LDOUBLE; break;

    default:
        error(FATAL, "illegal type specification");
    }

    type_append_bits(t, ts);
}

/* the parser is positioned at a tag specifier. returns
   the tag symbol table entry for the tag, after creating
   one in the current scope if necessary. an appropriate
   type_node (T_STRUN or T_INT) is appended to type. */

struct symbol *tag_specifier(struct type *type)
{
    struct symbol *tag = 0;
    struct string *id = 0;
    storage_class ss;
    struct type_node *tn;

    switch (token.k)
    {
    case K_STRUCT:  ss = S_STRUCT; break;
    case K_UNION:   ss = S_UNION; break;
    case K_ENUM:    ss = S_ENUM; break;
    }

    lex();

    if (token.k == K_IDENT) {
        id = token.text;
        lex();

        if ((token.k == K_LBRACE) || (token.k == K_SEMI))
            tag = symbol_lookup(current_scope, current_scope, id, S_TAG);
        else
            tag = symbol_lookup(current_scope, SCOPE_GLOBAL, id, S_TAG);
    }
    
    if (tag == 0) {
        tag = symbol_new(id, ss);
        symbol_insert(tag, current_scope);
    }

    if (!(tag->ss & ss))
        error(FATAL, "tag identifier already used in %A %1", tag, tag);

    if ((tag->ss & S_DEFINED) && (token.k == K_LBRACE))
        error(FATAL, "%A is already defined %1", tag, tag);

    if (token.k == K_LBRACE) symbol_here(tag);
    
    if (ss == S_ENUM)
        tn = type_append_bits(type, T_INT);
    else {
        tn = type_append_bits(type, T_STRUN);
        tn->tag = tag;
    }

    return tag;
}

/* process an enumeration specifier and
   appends a T_INT to the type. */

static void enum_specifier(struct type *type)
{
    struct symbol *tag = 0;
    struct symbol *sym;
    struct string *id;
    int value;

    tag = tag_specifier(type);
        
    if (token.k == K_LBRACE) {
        lex();
        value = 0;

        for (;;) {
            lex_expect(K_IDENT);
            id = token.text;

            sym = symbol_lookup(current_scope, current_scope, id, S_NORMAL);

            if (sym)
                error(FATAL, "can't redeclare enumerator '%S' %1", id, sym);

            sym = symbol_new(id, S_CONST);
            lex();

            if (token.k == K_EQ) {
                lex();
                value = int_expression(T_INT | T_UINT, INT_EXPRESSION_WARN);
            }

            sym->value = value++;
            symbol_insert(sym, current_scope);

            if (token.k == K_COMMA)
                lex();
            else
                break;
        }

        lex_match(K_RBRACE);
        tag->ss |= S_DEFINED;
    }

    if ((tag->ss & S_DEFINED) == 0)
        error(FATAL, "incomplete enumeration");
}

/* parse specifiers, and populate type accordingly. storage class is
   returned by setting *ss (to 0, if no storage class given). if no
   type specifiers are seen, we default to int. */

static void specifiers(struct type *type, storage_class *ss)
{
    token_class specs = 0;
    type_bits quals = 0;
    struct symbol *typename;

    if (ss) *ss = 0;

    for (;;)
    {
        switch (token.k)
        {
        case K_AUTO:        case K_EXTERN:
        case K_REGISTER:    case K_STATIC:      case K_TYPEDEF:

            if (ss) {
                storage_specifiers(ss);
                break;
            } else
                error(FATAL, "storage class not permitted here");

        case K_CONST:       case K_VOLATILE:

            qualifiers(&quals);
            break;

        case K_SIGNED:      case K_UNSIGNED:    case K_CHAR:
        case K_SHORT:       case K_INT:         case K_LONG:
        case K_FLOAT:       case K_DOUBLE:      case K_VOID:
            
            if (!TYPE_EMPTY(type) || (specs & K_SPEC(token.k)))
                goto multiple_types;

            specs |= K_SPEC(token.k);
            lex();
            break;

        case K_STRUCT:      case K_UNION:       case K_ENUM:

            if (!TYPE_EMPTY(type) || specs)
                goto multiple_types;

            if (token.k == K_ENUM)
                enum_specifier(type);
            else
                strun_specifier(type);

            break;

        case K_IDENT:

            if (TYPE_EMPTY(type) && (specs == 0)
              && (typename = symbol_typename(token.text))) {
                type_copy(type, &typename->type);
                lex();
                break;
            }

            /* .. not a type name, fall through ... */
            
        default:
            if (specs) map_specs(type, specs);
            if (TYPE_EMPTY(type)) type_append_bits(type, T_INT);
            type_qualify(type, quals);
            return;
        }
    }

multiple_types:
    error(FATAL, "too many type specifiers");
}

/* parse an old-style argument list and append
   an appropriate function type_node to type. */

static void old_arguments(struct type *type)
{
    struct type_node *tn;
    struct formal *f;

    tn = type_append_bits(type, T_FUNC | T_OLD_STYLE);
    lex_match(K_LPAREN);

    if (token.k == K_IDENT) {
        for (;;) {
            lex_expect(K_IDENT);

            /* ANSI 6.7.1 prohibits redeclaring a typedef name 
               as a parameter in an old-style argument list. */

            if (symbol_typename(token.text))
                error(FATAL, "can't redeclare typedef name as argument");

            f = formal_new();
            f->id = token.text;
            formals_append(&tn->formals, f);
            lex();

            if (token.k == K_COMMA)
                lex();
            else
                break;
        }
    }

    lex_match(K_RPAREN);
}

/* parse a prototype argument list, and append
   an appropriate function type_node to type. */

static void proto_arguments(struct type *type)
{
    struct type base = TYPE_INITIALIZER(base);
    struct type_node *tn;
    storage_class ss;
    struct formal *f;

    tn = type_append_bits(type, T_FUNC);
    lex_match(K_LPAREN);
    scope_enter();

    for (;;) {
        if (token.k == K_ELLIP) {   /* we don't need to check that this */
            lex();                  /* isn't first, because this function */
            tn->ts |= T_VARIADIC;   /* is only called when an initial */
            break;                  /* declaration specifier was seen. */
        }

        /* As a pure matter of syntax (see BNF), ANSI requires that
           a parameter-declaration include at least one specifier. */

        if (!k_decl(token))
            error(FATAL, "declaration specifier(s) missing");

        f = formal_new();
        ss = 0;
        specifiers(&base, &ss);

        if (ss) 
            if (ss == S_REGISTER)
                ++(f->is_register);
            else
                error(FATAL, "illegal storage class in prototype");
        
        declarator(&f->type, &f->id);
        TYPE_CONCAT(&f->type, &base);

        if ((TYPE_BASE(&f->type) == T_VOID) && (f->id == 0)
          && FORMALS_EMPTY(&tn->formals)) {
            formal_free(f);
            break; /* one unnamed void argument */
        }

        type_validate(&f->type, 0);
        type_fix_pointer(&f->type);
        formals_append(&tn->formals, f);

        if (token.k == K_COMMA)
            lex();
        else
            break;
    }

    lex_match(K_RPAREN);
    scope_exit_proto();
}

/* parse a declarator. type is populated with type_nodes as appropriate.
   if id is 0, then the declarator must be abstract; otherwise *id is
   set to the declared identifier, which may itself be 0 if absent. */

static void declarator0(struct type *type, struct string **id)
{
    struct token peek;
    struct type_node *tn;

    if (token.k == K_LPAREN) {
        peek = lex_peek();

        if ((peek.k != K_RPAREN) && !k_decl(peek)) {
            lex();
            declarator(type, id);
            lex_match(K_RPAREN);
        }
    } else if (token.k == K_IDENT) {
        if (id) {
            *id = token.text;
            lex();
        } else
            error(FATAL, "abstract type required");
    } else 
        if (id) *id = 0;

    for (;;) {
        switch (token.k)
        {
        case K_LBRACK:
            lex();
            tn = type_append_bits(type, T_ARRAY);

            if (token.k != K_RBRACK) {
                tn->nr_elements = size_expression();

                if (tn->nr_elements == 0)
                    error(FATAL, "zero dimension not permitted");
            } else
                tn->nr_elements = 0;
            
            lex_match(K_RBRACK);
            break;

        case K_LPAREN:
            peek = lex_peek();
        
            if (k_decl(peek))
                proto_arguments(type);
            else
                old_arguments(type);

            break;

        default:
            return;
        }
    }
}

static void declarator(struct type *type, struct string **id)
{
    type_bits quals = 0;

    if (token.k == K_MUL) {
        lex();
        qualifiers(&quals);
        declarator(type, id);
        type_append_bits(type, T_PTR | quals);
    } else
        declarator0(type, id);
}

/* parse the bit-field specification following
   a declarator. modifies type accordingly. */

static void bitfield(struct string *id, struct type *type)
{
    size_t bits;

    if (!(TYPE_INTEGRAL(type)))
        error(FATAL, "bit fields must have integral type");
        
    lex_match(K_COLON);
    bits = size_expression();

    if ((bits == 0) && id)
        error(FATAL, "alignment fields must be anonymous");

    if (bits > (type_sizeof(type, 0) * BITS_PER_BYTE))
        error(FATAL, "bit field too long for type");
    
    TYPE_SET_SIZE(type, bits);
}

typedef int declarations_flags; /* DECLARATIONS_FLAG_ */

#define DECLARATIONS_FLAG_FIRST     ( 0x00000001 )  /* is first declarator */
#define DECLARATIONS_FLAG_ARGSOK    ( 0x00000002 )  /* allow old-style args */
#define DECLARATIONS_FLAG_FIELDS    ( 0x00000004 )  /* allow bitfields */

/* returned by callback functions used by declarations(). */

typedef int declarations_ret;   /* DECLARATIONS_RET_ */

#define DECLARATIONS_RET_CONTINUE   0   /* continue looking for declarators */
#define DECLARATIONS_RET_ABORT      1   /* no more declarators, next line */

/* process a line of declarations according to the flags
   given, calling the function for each declarator. */

static void declarations(declarations_flags flags, void *arg,
             declarations_ret callback(storage_class, struct string *,
                                       struct type *, void *,
                                       declarations_flags))
{
    struct type base = TYPE_INITIALIZER(base);
    struct type type = TYPE_INITIALIZER(type);
    type_validate_flags validate_flags;
    declarations_flags callback_flags;
    declarations_ret ret;
    storage_class ss;
    struct string *id;

    callback_flags = DECLARATIONS_FLAG_FIRST;
    validate_flags = (flags & DECLARATIONS_FLAG_ARGSOK)
                        ? TYPE_VALIDATE_ARGSOK : 0;

    specifiers(&base, &ss);

    for (;;) {
        declarator(&type, &id);
        type_copy(&type, &base);
        type_validate(&type, validate_flags);

        if ((token.k == K_COLON) && (flags & DECLARATIONS_FLAG_FIELDS))
            bitfield(id, &type);

        ret = callback(ss, id, &type, arg, callback_flags);
        type_clear(&type);

        if (ret == DECLARATIONS_RET_ABORT)
            goto abort;

        callback_flags = 0;
        validate_flags = 0;

        if (token.k == K_COMMA)
            lex();
        else
            break;
    }

    lex_match(K_SEMI);
abort:
    type_clear(&base);
}

/* declarations callback for struct/union members */

static declarations_ret member_declaration(storage_class ss,
                                           struct string *id,
                                           struct type *type, void *arg,
                                           declarations_flags flags)
{
    struct symbol *tag = arg;

    if (ss) error(FATAL, "members can't have storage class");

    if ((id == 0) && !TYPE_FIELD(type) && !((flags & DECLARATIONS_FLAG_FIRST)
      && TYPE_ANONYMOUS_STRUN(type)))
        error(FATAL, "member declaration missing identifier");

    if (TYPE_FUNC(type))
        error(FATAL, "function '%S' can't be a member", id);

    member_insert(tag, id, type);

    if ((id == 0) && TYPE_ANONYMOUS_STRUN(type)) {
        lex_match(K_SEMI);
        return DECLARATIONS_RET_ABORT;
    } else
        return DECLARATIONS_RET_CONTINUE;
}

/* deal with struct or union specifiers. appends
   an appropriate T_STRUN type_node to type. */

static void strun_specifier(struct type *tag_type)
{
    struct symbol *tag;

    tag = tag_specifier(tag_type);
    
    if (token.k == K_LBRACE) {
        lex();

        while (k_decl(token))
            declarations(DECLARATIONS_FLAG_FIELDS, tag, member_declaration);

        lex_match(K_RBRACE);
        symbol_complete_strun(tag);
    }
}

/* distinguish between empty and non-empty declarations
   and legal and non-legal empty declarations. */

static int empty_declaration(struct string *id, struct type *type,
                             declarations_flags flags)
{
    if (id == 0) {
        if ((flags & DECLARATIONS_FLAG_FIRST) && TYPE_TERMINAL(type)) {
            /* the declaration list can be (completely) empty; using */
            /* TYPE_TERMINAL ensures there is no declarator at all */

            lex_match(K_SEMI);
            return 1;
        } else
            error(FATAL, "declaration missing identifier");
    }
            
    return 0;
}

/* process local declarations - those found
   in function-level scopes */

static declarations_ret local_declaration(storage_class ss,
                                          struct string *id,
                                          struct type *type, void *arg,
                                          declarations_flags flags)
{
    struct symbol *sym;

    if (empty_declaration(id, type, flags))
        return DECLARATIONS_RET_ABORT;
    
    sym = symbol_lookup(current_scope, current_scope, id, S_NORMAL);

    if (sym)
        error(FATAL, "'%S' already declared in this scope %1", sym->id, sym);

    symbol_function_class(type, &ss);
    if (ss == 0) ss = S_LOCAL;

    if (ss & S_EXTERN) {
        sym = symbol_global(id, ss, type, SCOPE_LURKER);
        symbol_redirect(sym);
    } else {
        sym = symbol_new(id, ss);
        type_copy(&sym->type, type);
        symbol_insert(sym, current_scope);

        if (!(ss & S_TYPEDEF))
            init_local(sym);
    }

    return DECLARATIONS_RET_CONTINUE;
}

void local_declarations(void)
{
    while (k_decl(token))
        declarations(0, 0, local_declaration);
}

/* handle the argument declarations at the head
   of an old-style function definition */

static declarations_ret argument_declaration(storage_class ss,
                                             struct string *id,
                                             struct type *type, void *arg,
                                             declarations_flags flags)
{
    struct formals *formals = arg;
    struct formal *f;

    if (id == 0)
        error(FATAL, "missing argument name");

    f = formal_lookup(formals, id);

    if (f == 0)
        error(FATAL, "'%S' is not an argument", id);

    if (!TYPE_EMPTY(&f->type))
        error(FATAL, "'%S' already declared", id);
    
    type_copy(&f->type, type);
    type_fix_pointer(&f->type);
    
    switch (ss)
    {
    case S_REGISTER:    f->is_register = 1; break;
    case 0:             break;

    default:   
        error(FATAL, "illegal storage class for argument");
    }

    if (TYPE_FLOAT(&f->type))
        f->downcast_float = 1;

    return DECLARATIONS_RET_CONTINUE;
}

static void function_header(struct type *type)
{
    struct formals *formals;

    formals = &TYPE_FORMALS(type);

    while (k_decl(token))
        declarations(0, formals, argument_declaration);

    formals_default(formals);
}

/* handle a function definition. this not only parses the function, but
   also drives the optimization, code generation, and output phases. */

static void function_definition(struct symbol *sym, struct type *type)
{
    if (TYPE_OLD_DEF(type))
        function_header(type);
    else
        scope_resurrect();

    if (sym->ss & S_DEFINED)
        error(FATAL, "function '%S' already defined %1", sym->id, sym);

    sym->ss |= S_DEFINED;
    scope_enter();
    func_new(sym);
    formals_declare(&TYPE_FORMALS(type));
    compound_statement();
    scope_exit();
    symbol_check_labels();

    block_add_successor(current_block, CC_ALWAYS, exit_block);

    dealias();
    opt_early();
    switch_gen();

    if (!debug_flag_i) {
        gen_args();
        target->gen();
        opt_late();

        if (!debug_flag_g) {
            graph_alloc();
            target->opt();
        }

        target->logues();
    }

    opt_post();
    output_func();
    func_free();
    scope_end_func();
}

/* a c translation unit boils down to a simple
   sequence of external definitions. */

static declarations_ret external_definition(storage_class ss,
                                            struct string *id,
                                            struct type *type, void *arg,
                                            declarations_flags flags)
{
    storage_class effective_ss;
    struct symbol *sym;

    if (ss & (S_AUTO | S_REGISTER))
        error(FATAL, "external definition can't be %k or %k",
                    K_AUTO, K_REGISTER);

    if (empty_declaration(id, type, flags))
        return DECLARATIONS_RET_ABORT;

    effective_ss = ss;
    symbol_function_class(type, &effective_ss);
    if (effective_ss == 0) effective_ss = S_EXTERN;
    sym = symbol_global(id, effective_ss, type, SCOPE_GLOBAL);

    if (!(effective_ss & S_TYPEDEF)) {
        if (TYPE_BASE(type) == T_FUNC) {
            if (TYPE_OLD_DEF(type) || !(token.k & K_DECL_LIST)) {
                function_definition(sym, type);
                return DECLARATIONS_RET_ABORT;
            }
        } else
            init_global(sym, ss);
    }

    return DECLARATIONS_RET_CONTINUE;
}

extern void translation_unit(void)
{
    while (k_decl(token) || (token.k & K_DTOR))
        declarations(DECLARATIONS_FLAG_ARGSOK, 0, external_definition);

    if (token.k) error(FATAL, "external definition expected");
}

/* parse an abstract declarator and
   populate the type accordingly */

void abstract_declarator(struct type *type)
{
    struct type base = TYPE_INITIALIZER(base);

    specifiers(&base, 0);
    declarator(type, 0);
    TYPE_CONCAT(type, &base);
    type_validate(type, TYPE_VALIDATE_VOIDOK);
}

/* vi: set ts=4 expandtab: */
