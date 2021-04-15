/* init.c - initializers                                ncc, the new c compiler

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

#include "../common/util.h"
#include "../common/slist.h"
#include "cc1.h"
#include "lex.h"
#include "expr.h"
#include "gen.h"
#include "type.h"
#include "output.h"
#include "string.h"
#include "symbol.h"
#include "target.h"
#include "tree.h"
#include "init.h"

typedef int init_flags; /* INIT_FLAG_* */

#define INIT_FLAG_OUTER     ( 0x00000001 )      /* outermost invocation */
#define INIT_FLAG_BRACED    ( 0x00000002 )      /* enclosed in braces */

static void init(struct type *, init_flags, int);

/* only used when initializing an automatic variable */

static struct symbol *auto_sym;         /* the symbol being initialized */
static struct tree *auto_list;          /* list of assignments */

/* append an assignment operation to the auto_list */

static void auto_list_append(struct tree *tree)
{
    if (auto_list) {
        tree = tree_binary(E_COMMA, auto_list, tree);
        type_copy(&tree->type, &tree->right->type);
    }

    auto_list = tree;
}

/* when looking for an aggregate initializer (for a struct/union),
   we don't know if an expression initializes the first member of
   the aggregate or initializes the entire aggregate until we know
   its type. we need to implement a pushback buffer for lookahead. */

struct tree *pushback;

/* get the next initializer expression. */

static struct tree *next_value(void)
{
    struct tree *tree;

    if (pushback) {
        tree = pushback;
        pushback = 0;
    } else {
        if (auto_sym)
            tree = init_expression();
        else
            tree = static_expression();
    }

    return tree;
}

/* return a pointer to an object of the specified type
   that lives at offset in the auto_sym. */

static struct tree *auto_ref(struct type *type, int offset)
{
    struct tree *tree;

    tree = tree_sym(auto_sym);
    tree = tree_addrof(tree);
    tree = tree_binary(E_ADD, tree, tree_i(target->ptr_int, offset));
    type_ref(&tree->type, type);

    return tree;
}

/* emit a directive to create a block of uninitialized
   (zero) storage for the specified symbol. */

void init_bss(struct symbol *sym)
{
    size_t size;
    int align;

    size = type_sizeof(&sym->type, TYPE_SIZEOF_NOERROR);

    if (size == 0) 
        error(FATAL, "'%S' has incomplete type %1", sym->id, sym);

    align = type_alignof(&sym->type);

    if (sym->ss & S_STATIC)
        output(".local %g\n", sym);

    output(".comm %g, %z, %d\n", sym, size, align);
    sym->ss &= ~S_TENTATIVE;
    sym->ss |= S_DEFINED;
}

/* output a bitfield value of n bits. we keep a little
   (byte) buffer and output bytes as they fill up. this
   method makes initializing bitfields really slow. */

static void bits(long i, int n)
{
    static char buf;
    static char pos;

    while (n--) {
        buf >>= 1;
        
        if (i & 1)
            buf |= 0x80;
        else
            buf &= 0x7F;

        i >>= 1;
        ++pos;

        if ((pos % BITS_PER_BYTE) == 0)
            output("\t.byte %d\n", buf & 0xFF);
    }
}

/* fill the next n bits with zeros. this is a no-op
   when performing an automatic initialization */

static void init_pad(int n)
{
    if (auto_sym == 0) {
        if (n % BITS_PER_BYTE)
            bits(0, n % BITS_PER_BYTE);

        if (n / BITS_PER_BYTE)
            output("\t.space %d, 0\n", n / BITS_PER_BYTE);
    }
}

/* tree is an E_CON; output its value with
   an appropriate assembler data pseudo-op. */

static void word(struct tree *tree)
{
    type_bits ts;

    ts = type_machine_bits(TYPE_BASE(&tree->type));

    if (TYPE_FLOATING(&tree->type)) {
        union {
            float f;
            int i;
        } u;

        switch (ts)
        {
        case T_FLOAT:       u.f = tree->con.f;
                            output("\t.long %x", u.i);
                            break;

        case T_DOUBLE:      output("\t.quad %X", tree->con.i);
                            break;
        }
    } else {
        switch (ts)
        {
        case T_SCHAR:
        case T_CHAR:
        case T_UCHAR:       output("\t.byte"); break;

        case T_SHORT:
        case T_USHORT:      output("\t.short"); break;

        case T_FLOAT:
        case T_INT:
        case T_UINT:        output("\t.int"); break;
    
        case T_LONG:
        case T_ULONG:       output("\t.quad"); break;
        }

        output(" %G", tree->sym, tree->con.i);
    }

    output("\n");
}

/* we have a scalar value to assign to an object of the
   specified type, so let's do it. first ensure the value
   is appropriate to the type. then, if it's an automatic
   variable, concoct an assignment expression and add it
   to auto_list. otherwise, output the value directly. */

static void init_value(struct type *dst, struct tree *value,
                       init_flags flags, int offset)
{
    struct tree *tree;
    int zero;

    if (auto_sym) {
        zero = tree_zero(value);

        if (offset || !TYPE_SCALAR(&auto_sym->type)) {
            tree = auto_ref(dst, offset);
            tree = tree_fetch(tree, 0);
        } else
            tree = tree_sym(auto_sym);

        tree = assign(tree, value);

        if (zero && !(flags & INIT_FLAG_OUTER)) {
            /* simple optimization: automatic aggregates will be BLKZEROed
               before initialization, so we can skip assigning zero values.
               the optimizer would catch this, but it's so easy to avoid .. */

            tree_free(tree);
        } else 
            auto_list_append(tree);
    } else {
        value = fake_assign(dst, value);
        
        if (TYPE_FIELD(&value->type)) {
            if (value->sym)
                error(FATAL, "invalid bit-field initializer");

            bits(value->con.i, TYPE_GET_SIZE(&value->type));
        } else
            word(value);

        tree_free(value);
    }
}

/* initialize a struct object. cluttered with calculations
   solely to facilitate initializing static bit fields. */

static void init_struct(struct type *dst, init_flags flags, int offset)
{
    struct symbol *tag;
    struct symbol *member;
    int offset_bits;
    int pad_bits;

    tag = TYPE_STRUN_TAG(dst);
    member = MEMBERS_FIRST(&TYPE_STRUN_MEMBERS(dst));

    if (member == 0)
        error(FATAL, "can't initialize incomplete %A", tag);

    offset_bits = 0;

    for (;;)
    {
        if (member == 0)
            error(FATAL, "too many initializers for %A", tag);

        pad_bits = member->offset * BITS_PER_BYTE;
        pad_bits -= offset_bits;
        
        if (TYPE_FIELD(&member->type))
            pad_bits += TYPE_GET_SHIFT(&member->type);

        init_pad(pad_bits);
        offset_bits += pad_bits;

        init(&member->type, 0, offset + member->offset);

        if (TYPE_FIELD(&member->type))
            offset_bits += TYPE_GET_SIZE(&member->type);
        else
            offset_bits += type_sizeof(&member->type, 0) * BITS_PER_BYTE;

        member = MEMBERS_NEXT(member);

        if (TYPE_STRUN_UNION(dst))
            member = 0;

        if (!(flags & INIT_FLAG_BRACED) && (member == 0))
            break;

        if (token.k == K_COMMA)
            lex();
        else
            break;
    }

    init_pad(type_sizeof(dst, 0) * BITS_PER_BYTE - offset_bits);
}

/* if possible, update the bounds of the array type */

static void update_bounds(struct type *dst, int n)
{
    if (TYPE_UNBOUNDED_ARRAY(dst) && !TYPE_FLEXIBLE(dst))
        TYPE_SET_NR_ELEMENTS(dst, n);
}

/* initialize an array object. we don't have to deal with
   bitfields here as we do with structs, so the main loop
   is cleaner. we do need to update the dst type if it is
   incomplete, though (and not a flexible array member) */

static void init_array(struct type *dst, init_flags flags, int offset)
{
    struct type element_type = TYPE_INITIALIZER(element_type);
    size_t size;
    int n;

    type_deref(&element_type, dst);
    size = type_sizeof(&element_type, 0);
    n = 0;

    for (;;)
    {
        if (!TYPE_UNBOUNDED_ARRAY(dst) && (n == TYPE_GET_NR_ELEMENTS(dst)))
            error(FATAL, "too many initializers for array");

        init(&element_type, 0, offset + (n * size));
        ++n;

        if (!(flags & INIT_FLAG_BRACED) && (n == TYPE_GET_NR_ELEMENTS(dst)))
            break;

        if (token.k == K_COMMA)
            lex();
        else
            break;
    }

    update_bounds(dst, n);

    if (!TYPE_UNBOUNDED_ARRAY(dst)) {
        n = TYPE_GET_NR_ELEMENTS(dst) - n;
        init_pad(n * size * BITS_PER_BYTE);
    }

    type_clear(&element_type);
}

/* initialize a char[] from a string literal. */

static void init_strlit(struct type *dst, init_flags flags, int offset)
{
    struct symbol *sym;
    struct tree *tree;
    int n;

    if (TYPE_UNBOUNDED_ARRAY(dst)) {
        n = token.text->len + 1;
        update_bounds(dst, n);
    } else {
        n = TYPE_GET_NR_ELEMENTS(dst);

        if (n < token.text->len)
            error(FATAL, "string literal exceeds length of array");
    }

    if (auto_sym) {
        sym = string_symbol(token.text);
        tree = tree_sym(sym);
        n = MIN(n, token.text->len);
        TYPE_SET_NR_ELEMENTS(&tree->type, n);

        tree = tree_binary(E_BLKASG, auto_ref(dst, offset),
                                     tree_addrof(tree));

        type_copy(&tree->type, &tree->right->type);
        auto_list_append(tree);
    } else
        string_emit(token.text, n);

    lex();
}

/* initialize an object of the specified type. */

static void init(struct type *dst, init_flags flags, int offset)
{
    struct tree *value;

    if (token.k == K_LBRACE) {
        flags |= INIT_FLAG_BRACED;
        lex();
    } else
        flags &= ~INIT_FLAG_BRACED;

    if ((token.k != K_STRLIT) && (token.k != K_LBRACE)) {
        value = next_value();
        pushback = value;
    } else
        value = 0;
    
    if (TYPE_SCALAR(dst) || (TYPE_STRUN(dst)
      && value && TYPE_STRUN(&value->type))) {
        value = next_value();
        init_value(dst, value, flags, offset);
    } else {
        if (TYPE_FLEXIBLE(dst) && auto_sym)
            error(FATAL, "can't initialize automatic flexible array members");

        if (TYPE_CHAR_ARRAY(dst) && (token.k == K_STRLIT))
            init_strlit(dst, flags, offset);
        else {
            if ((flags & INIT_FLAG_OUTER) && !(flags & INIT_FLAG_BRACED))
                error(FATAL, "aggregate initializer missing braces");

            if (TYPE_STRUN(dst))
                init_struct(dst, flags, offset);
            else
                init_array(dst, flags, offset);
        }
    }

    if (flags & INIT_FLAG_BRACED)
        lex_match(K_RBRACE);
}

/* emit an appropriate header before the
   actual data of a static symbol */

static void init_static_header(struct symbol *sym)
{
    output_select(TYPE_CONST(&sym->type)
                  ? OUTPUT_SEG_TEXT
                  : OUTPUT_SEG_DATA);

    output(".align %d\n", type_alignof(&sym->type));
    output("%g:\n", sym);
}

/* perform a static initialization of sym */

static void init_static(struct symbol *sym)
{
    sym->ss |= S_DEFINED;
    sym->ss &= ~S_TENTATIVE;
    symbol_here(sym);

    init_static_header(sym);
    auto_sym = 0;
    init(&sym->type, INIT_FLAG_OUTER, 0);
}

/* called when declaring a global S_STATIC or S_EXTERN object (not a
   function) to either define the object via initialization, or mark
   it tentatively defined, or neither if it is explicitly 'extern'. */

void init_global(struct symbol *sym, storage_class declared_ss)
{
    if (token.k == K_EQ) {
        if (declared_ss & S_EXTERN)
            error(FATAL, "'%S' can't be initialized (declared 'extern')",
            sym->id);

        if (sym->ss & S_DEFINED)
            error(FATAL, "'%S' has already been defined %1", sym->id, sym);

        lex();
        init_static(sym);
    } else {
        if (!(sym->ss & S_DEFINED) && !(declared_ss & S_EXTERN))
            sym->ss |= S_TENTATIVE;
    }
}

/* process the initializer for a local variable, if
   any. called right after parsing the declarator. */

void init_local(struct symbol *sym)
{
    struct tree *tree;
    size_t size;

    if (token.k == K_EQ) {
        lex();

        if (sym->ss & S_STATIC)
            init_static(sym);
        else {
            auto_sym = sym;
            auto_list = 0;

            init(&sym->type, INIT_FLAG_OUTER, 0);
            size = type_sizeof(&sym->type, 0);

            if (!TYPE_SCALAR(&sym->type)) {
                tree = auto_ref(&sym->type, 0);
                tree = gen(tree, GEN_FLAG_ROOT);

                EMIT(insn_new(I_BLKZERO,
                              operand_leaf(tree),
                              operand_i(target->ptr_uint, size, 0)));

                tree_free(tree);
            }

            if (auto_list)
                gen(auto_list, GEN_FLAG_ROOT | GEN_FLAG_DISCARD);
        }
    } else {
        if (sym->ss & S_STATIC)
            init_bss(sym);
        else
            type_sizeof(&sym->type, 0);
    }
}

/* generate an anonymous, static, const symbol of the
   specified type that is initialized with the constant.
   this is used to convert into memory operands those
   constants that can't be immediate on some targets.

   we maintain a pool of such constants so if we encounter
   the same one more than once, we can reuse the data. */

struct constant
{
    type_bits ts;
    union con con;
    asm_label label;
    SLIST_ENTRY(constant) link;
};

static SLIST_HEAD(, constant) constants = SLIST_HEAD_INITIALIZER(constants);

#define CONSTANTS_FOREACH(c)        SLIST_FOREACH(c, &constants, link)
#define CONSTANTS_PREPEND(c)        SLIST_INSERT_HEAD(&constants, c, link)

struct symbol *init_constant_symbol(type_bits ts, union con con)
{
    struct constant *c;
    struct symbol *sym;
    struct tree *tree;

    CONSTANTS_FOREACH(c)
        if ((c->ts == ts) && (((ts & T_FLOATING) && (con.f == c->con.f))
          || (con.i == c->con.i)))
            break;

    if (c == 0) {
        sym = symbol_anonymous(0);
        type_append_bits(&sym->type, ts | T_CONST);
        init_static_header(sym);

        if (ts & T_FLOATING)
            tree = tree_f(ts, con.f);
        else
            tree = tree_i(ts, con.i);

        word(tree);
        tree_free(tree);

        c = safe_malloc(sizeof(struct constant));
        c->ts = ts;
        c->con = con;
        c->label = sym->label;
        CONSTANTS_PREPEND(c);
    } else {
        sym = symbol_anonymous(c->label);
        type_append_bits(&sym->type, ts | T_CONST);
    }

    return sym;
}

/* vi: set ts=4 expandtab: */
