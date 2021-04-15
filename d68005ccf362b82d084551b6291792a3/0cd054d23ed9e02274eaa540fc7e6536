/* symbol.c - symbol management                         ncc, the new c compiler

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
#include "../common/util.h"
#include "cc1.h"
#include "init.h"
#include "string.h"
#include "output.h"
#include "block.h"
#include "gen.h"
#include "target.h"
#include "symbol.h"

scope_level current_scope = SCOPE_GLOBAL;

/* note that we keep an extra bucket around, which is used for
   anonymous symbols, since we'll never search for them by name. */

#define UNNAMED_BUCKET      SYMBOL_NR_BUCKETS

static TAILQ_HEAD(bucket, symbol) buckets[SYMBOL_NR_BUCKETS+1];

#define BUCKET(id)      (((id))                                             \
                        ? &buckets[(id)->hash % SYMBOL_NR_BUCKETS]          \
                        : &buckets[UNNAMED_BUCKET])

#define BUCKET_INIT(b)                  TAILQ_INIT(b)
#define BUCKET_FIRST(b)                 TAILQ_FIRST(b)
#define BUCKET_NEXT(s)                  TAILQ_NEXT(s, links)
#define BUCKET_INSERT_TAIL(b, s)        TAILQ_INSERT_TAIL(b, s, links)
#define BUCKET_INSERT_BEFORE(bef, s)    TAILQ_INSERT_BEFORE(bef, s, links)
#define BUCKET_REMOVE(b, s)             TAILQ_REMOVE(b, s, links)

/* in addition to the main symbol table above, which maps identifiers to
   symbols, we also need a mechanism to map pseudo_regs to symbols. */

static LIST_HEAD(reg_bucket, symbol) reg_buckets[SYMBOL_NR_REG_BUCKETS];

#define REG_BUCKET(reg)     (&reg_buckets[((reg) % SYMBOL_NR_REG_BUCKETS)])

#define REG_BUCKET_INIT(rb)             LIST_INIT(rb)
#define REG_BUCKET_FIRST(rb)            LIST_FIRST(rb)
#define REG_BUCKET_NEXT(s)              LIST_NEXT(s, reg_links)
#define REG_BUCKET_INSERT_HEAD(rb, s)   LIST_INSERT_HEAD(rb, s, reg_links)
#define REG_BUCKET_REMOVE(s)            LIST_REMOVE(s, reg_links)

/* the usual new/free pair for symbols. the default
   values here aren't necessarily written in stone. */

struct symbol *symbol_new(struct string *id, storage_class ss)
{
    struct symbol *sym;
    static symbol_num last_num;

    sym = safe_malloc(sizeof(struct symbol));
    sym->id = id;
    sym->ss = ss;
    sym->scope = current_scope;
    sym->path = error_path;
    sym->line_no = error_line_no;
    sym->num = ++last_num;
    sym->reg = PSEUDO_REG_NONE;
    
    switch (ss)
    {
    case S_ENUM:
    case S_STRUCT:    
    case S_UNION:       MEMBERS_INIT(&sym->members);
                        break;

    case S_LABEL:       sym->to = block_new();
                        break;

    case S_STATIC:      sym->label = ASM_LABEL_NEW();
    default:            TYPE_INIT(&sym->type);
                        sym->reg = PSEUDO_REG_NONE;
                        sym->arg_reg = PSEUDO_REG_NONE;
                        break;
    }

    return sym;
}

void symbol_free(struct symbol *sym)
{
    if (!(sym->ss & S_TAG)) {
        type_clear(&sym->type);

        if (sym->reg != PSEUDO_REG_NONE)
            REG_BUCKET_REMOVE(sym);
    }

    free(sym);
}

/* update the path/line_no for a symbol
   to the current position */

void symbol_here(struct symbol *sym)
{
    sym->path = error_path;
    sym->line_no = error_line_no;
}

/* append a member to the members list, and inserts
   it into the symbol table at the appropriate scope.
   ensures that the member name is not a duplicate.
   unnamed members are simply discarded. */

static void members_append(struct members *members, struct symbol *member)
{
    struct symbol *m;

    if (member->id == 0) {
        symbol_free(member);
        return;
    }

    m = MEMBERS_FIRST(members);

    if (m == 0)
        MEMBERS_INSERT_HEAD(members, member);
    else {
        for (;;) {
            if (m->id == member->id)
                error(FATAL, "duplicate member '%S' %1", m->id, m);

            if (TYPE_FLEXIBLE(&m->type))
                error(FATAL, "no members allowed after flexible array");

            if (MEMBERS_NEXT(m))
                m = MEMBERS_NEXT(m);
            else
                break;
        }

        MEMBERS_INSERT_AFTER(m, member);
    }

    symbol_insert(member, current_scope);
}

/* absorb the members of the anonymous struct or union in
   src into the struct or union in dst, starting at offset. */

static void members_absorb(struct symbol *dst, struct symbol *src,
                           size_t offset)
{
    struct symbol *m;
    struct symbol *dst_m;

    for (m = MEMBERS_FIRST(&src->members); m; m = MEMBERS_NEXT(m)) {
        dst_m = symbol_new(m->id, S_MEMBER);
        type_copy(&dst_m->type, &m->type);
        dst_m->offset = offset + m->offset;
        members_append(&dst->members, dst_m);
    }
}

/* insert a member into an S_STRUCT or S_UNION. the caller
   is responsible for ensuring that that the member is prima
   facie valid (has a valid type and an identifer if required),
   but this function and its friends will do the heavy lifting
   of computing offsets and absorbing anonymous structs/unions.

   before an aggregate is S_DEFINED, its size is in BITS
   and is a running counter: for S_STRUCT, of the current
   offset, and for S_UNION, of the largest member seen.
   similarly the alignment is computed as-we-go. */

void member_insert(struct symbol *tag, struct string *id, struct type *type)
{
    struct symbol *member;
    long offset_bits;
    size_t type_bits;
    size_t member_bits;
    int align_bytes;

    member = symbol_new(id, S_MEMBER);
    type_copy(&member->type, type);

    if (TYPE_UNBOUNDED_ARRAY(type)) {
        type_bits = 0;
        TYPE_MARK_FLEXIBLE(&member->type);
    } else
        type_bits = type_sizeof(type, 0) * BITS_PER_BYTE;

    align_bytes = type_alignof(type);

    if (tag->ss & S_UNION)
        offset_bits = 0;
    else
        offset_bits = tag->size;

    if (TYPE_FIELD(type)) {
        member_bits = TYPE_GET_SIZE(type);

        if ((member_bits == 0) || (ROUND_DOWN(offset_bits, type_bits)
          != ROUND_DOWN(offset_bits + member_bits - 1, type_bits)))
            offset_bits = ROUND_UP(offset_bits, type_bits);

        TYPE_SET_SHIFT(&member->type, offset_bits % type_bits);
    } else {
        offset_bits = ROUND_UP(offset_bits, align_bytes * BITS_PER_BYTE);
        member_bits = type_bits;
    }

    member->offset = ROUND_DOWN(offset_bits / BITS_PER_BYTE, align_bytes);

    if (tag->ss & S_UNION)
        tag->size = MAX(tag->size, member_bits);
    else {
        offset_bits += member_bits;
        tag->size = offset_bits;
    }

    if (offset_bits > (MAX_OBJECT_SIZE * BITS_PER_BYTE))
        error(FATAL, "%A exceeds maximum object size", tag);

    tag->align = MAX(tag->align, align_bytes);

    if (TYPE_ANONYMOUS_STRUN(type) && (id == 0))
        members_absorb(tag, TYPE_FIRST(type)->tag, member->offset);

    members_append(&tag->members, member);
}

/* find a member in a struct or union and
   returns its symbol, or 0 if not found */

struct symbol *member_lookup(struct symbol *tag, struct string *id)
{
    struct symbol *m;

    for (m = MEMBERS_FIRST(&tag->members); m; m = MEMBERS_NEXT(m))
        if (m->id == id)
            break;

    return m;
}

/* finished defining a struct or union,
   so finalize the tag entry. */

void symbol_complete_strun(struct symbol *tag)
{
    if (tag->size == 0) 
        error(FATAL, "%A has zero size", tag);

    tag->size = ROUND_UP(tag->size, tag->align * BITS_PER_BYTE);
    tag->size /= BITS_PER_BYTE;
    tag->ss |= S_DEFINED;
}

/* insert a symbol into the symbol table
   at the specified scope_level. */

void symbol_insert(struct symbol *sym, scope_level scope)
{
    struct symbol *p;
    struct bucket *b;

    sym->scope = scope;
    b = BUCKET(sym->id);

    for (p = BUCKET_FIRST(b); p; p = BUCKET_NEXT(p)) {
        if (p->scope <= scope) {
            BUCKET_INSERT_BEFORE(p, sym);
            return;
        }
    }

    BUCKET_INSERT_TAIL(b, sym);
}

/* remove a symbol from the symbol table */

static void symbol_remove(struct symbol *sym)
{
    struct bucket *b;

    b = BUCKET(sym->id);
    BUCKET_REMOVE(b, sym);
}

/* move the symbol from its current scope to
   the specified scope. */

static void symbol_move(struct symbol *sym, scope_level scope)
{
    symbol_remove(sym);
    symbol_insert(sym, scope);
}

/* prime the symbol table.
   call early from main(). */

void symbol_init(void)
{
    int i;

    for (i = 0; i <= UNNAMED_BUCKET; ++i)
        BUCKET_INIT(&buckets[i]);

    for (i = 0; i < SYMBOL_NR_REG_BUCKETS; ++i)
        REG_BUCKET_INIT(&reg_buckets[i]);
}

/* entering a new scope is as simple
   as bumping the level up. */

void scope_enter(void)
{
    if (current_scope == SCOPE_MAX)
        error(FATAL, "scopes too deeply nested");

    ++current_scope;
}

/* walk the symbol table and perform the action on all
   symbols in scopes between start and end (inclusive).
   use SCOPE_WALK() to walk the named buckets only, or
   SCOPE_WALK_ALL() to include UNNAMED_BUCKET. */

#define SCOPE_WALK0(start, end, last, action)                               \
    do {                                                                    \
        struct symbol *THIS;                                                \
        struct symbol *NEXT;                                                \
        int I;                                                              \
                                                                            \
        for (I = 0; I <= (last); ++I) {                                     \
            for (THIS = BUCKET_FIRST(&buckets[I]); THIS; THIS = NEXT) {     \
                NEXT = BUCKET_NEXT(THIS);                                   \
                if (THIS->scope < (end)) break;                             \
                if (THIS->scope > (start)) continue;                        \
                { action }                                                  \
            }                                                               \
        }                                                                   \
    } while (0)

#define SCOPE_WALK(start, end, action) \
        SCOPE_WALK0(start, end, SYMBOL_NR_BUCKETS - 1, action)

#define SCOPE_WALK_ALL(start, end, action) \
        SCOPE_WALK0(start, end, UNNAMED_BUCKET, action)

/* look for a symbol with (one of) the specified storage
   class(es) in the scopes between start and end (inclusive),
   and return its entry, or 0 if not found. */

struct symbol *symbol_lookup(scope_level start, scope_level end,
                             struct string *id, storage_class ss)
{
    struct bucket *b;
    struct symbol *sym;
    struct symbol *next;

    b = BUCKET(id);

    for (sym = BUCKET_FIRST(b); sym; sym = next) {
        next = BUCKET_NEXT(sym);
        if (sym->scope < (end)) break;
        if (sym->scope > (start)) continue;

        if (sym->ss & S_REDIRECT)
            sym = sym->redirect;
        
        if ((sym->id == id) && (sym->ss & ss))
            return sym;
    }

    return 0;
}

/* if the identifier is a type name visible in the current
   scope, return its symbol, or 0 otherwise. it's tempting
   to just use symbol_lookup() and look for S_TYPEDEF, but
   we can't- other names in S_NORMAL can hide type names. */

struct symbol *symbol_typename(struct string *id)
{
    struct symbol *sym;

    sym = symbol_lookup(current_scope, SCOPE_GLOBAL, id, S_NORMAL);

    if (sym && (sym->ss & S_TYPEDEF))
        return sym;
    
    return 0;
}

/* called after parsing a function to ensure that all
   referenced labels have been defined. */

void symbol_check_labels(void)
{
    SCOPE_WALK(SCOPE_LABEL, SCOPE_LABEL,
        if (!(THIS->ss & S_DEFINED))
            error(FATAL, "label '%S' never defined %1", THIS->id, THIS);
    );
}

/* called right before compiler exits to
   perform any final actions. */

void symbol_finalize(void)
{
    if (debug_flag_s) {
        scope_debug(SCOPE_KEEP);
        scope_debug(SCOPE_LURKER);
        scope_debug(SCOPE_GLOBAL);
    }

    SCOPE_WALK(SCOPE_GLOBAL, SCOPE_LURKER, 
        if (THIS->ss & S_TENTATIVE)
            init_bss(THIS);

        if ((THIS->ss & S_EXTERN) && (THIS->ss & (S_DEFINED | S_REFERENCED)))
            output(".globl %g\n", THIS);
    );
}

/* validate (and possibly adjust) the storage_class of a symbol
   declared with the specified type. this is called before the
   default storage class associated with a scope is assigned, to
   enforce rules that pertain to function storage classes. */

void symbol_function_class(struct type *type, storage_class *ss)
{
    if (TYPE_BASE(type) == T_FUNC) {
        if (((*ss & (S_AUTO | S_REGISTER))) ||
          ((*ss & S_STATIC) && (current_scope != SCOPE_GLOBAL)))
            error(FATAL, "illegal storage class for function");

        if ((*ss & (S_STATIC | S_EXTERN)) == 0) 
            *ss |= S_EXTERN;
    }
}

/* declare S_STATIC/S_EXTERN/S_TYPEDEF symbol in the global scope with
   the specified id, storage_class, and type, and return the symbol.
   this function handles the business of composing duplicates and
   exposing previously-lurking global variables if necessary. when
   calling from a block-level scope (which should only happen for
   S_EXTERNs), pass in SCOPE_LURKER for scope, otherwise SCOPE_GLOBAL. */

struct symbol *symbol_global(struct string *id, storage_class ss,
                             struct type *type, scope_level scope)
{
    struct symbol *sym;

    sym = symbol_lookup(SCOPE_GLOBAL, SCOPE_LURKER, id, S_NORMAL);

    if (sym) {
        if ((sym->ss & (S_TYPEDEF | S_CONST)) || (ss == S_TYPEDEF)
          || (type_compat(&sym->type, type, TYPE_COMPAT_FLAG_COMPOSE
                        | TYPE_COMPAT_FLAG_QUALS) != TYPE_COMPAT_OK))
            error(FATAL, "incompatible redeclaration of '%S' %1",
                         sym->id, sym);

        if ((sym->ss & S_EXTERN) && (ss & S_STATIC))
            error(FATAL, "'%S' previously declared 'extern' %1",
                         sym->id, sym);

        if (scope > sym->scope) symbol_move(sym, scope);
    } else {
        sym = symbol_new(id, ss);
        type_copy(&sym->type, type);
        symbol_insert(sym, scope);
    }
    
    return sym;
}

/* put an S_REDIRECT entry in the current scope
   that redirects to the global sym */

void symbol_redirect(struct symbol *sym)
{
    struct symbol *new;

    new = symbol_new(sym->id, S_REDIRECT);
    new->redirect = sym;
    symbol_insert(new, current_scope);
}

/* perform an implicit function declaration
   of id and return its symbol */

struct symbol *symbol_implicit(struct string *id)
{
    struct symbol *sym;
    struct type type = TYPE_INITIALIZER(type);

    type_append_bits(&type, T_FUNC | T_OLD_STYLE);
    type_append_bits(&type, T_INT);
    sym = symbol_global(id, S_EXTERN, &type, SCOPE_LURKER);

    if (!(sym->ss & S_IMPLICIT)) {
        error(WARNING, "implicit declaration of %s()", id->s);
        sym->ss |= S_IMPLICIT;
    }
    
    type_clear(&type);
    symbol_redirect(sym);

    return sym;
}

/* create an anonymous symbol in the current scope
   which references the label. if label is 0, a new
   label is allocated. */

struct symbol *symbol_anonymous(asm_label label)
{
    struct symbol *sym;

    sym = symbol_new(0, S_STATIC);

    if (label)
        sym->label = label;

    symbol_insert(sym, SCOPE_RETIRED);
    return sym;
}

/* create a compiler temporary with the
   specified type (stripped of qualifiers). */

struct symbol *symbol_temp(struct type *type)
{
    struct symbol *sym;

    sym = symbol_new(0, S_LOCAL);
    type_copy(&sym->type, type);
    TYPE_UNQUAL(&sym->type);
    symbol_insert(sym, current_scope);
    
    return sym;
}

/* create a compiler temporary reg with specified type.
   where symbol_temp() is used during high-level code
   generation (making trees and such), this is used in
   low-level code generation, after parsing is complete. */

pseudo_reg symbol_temp_reg(type_bits ts)
{
    struct symbol *sym;

    sym = symbol_new(0, S_REGISTER);
    type_append_bits(&sym->type, ts);
    symbol_insert(sym, SCOPE_RETIRED);
    
    return symbol_reg(sym);
}

/* return the [pseudo] register associated with
   the symbol, allocating one if needed */

pseudo_reg symbol_reg(struct symbol *sym)
{
    if (sym->reg == PSEUDO_REG_NONE) {
        target->symbol_reg(sym);
        REG_BUCKET_INSERT_HEAD(REG_BUCKET(sym->reg), sym);
    }
        
    return sym->reg;
}

/* ensure that physical storage is allocated
   for the specified symbol. */

void symbol_storage(struct symbol *sym)
{
    if ((sym->ss & S_BLOCK) && (sym->offset == 0))
        target->symbol_storage(sym);
}

/* return the symbol entry associated with the pseudo_reg, or
   0 if none. only physical machine registers and truly fake
   registers (e.g., PSEUDO_REG_CC) will not have entries. */

struct symbol *reg_symbol(pseudo_reg reg)
{
    struct symbol *sym;
    
    reg = PSEUDO_REG_BASE(reg);
    sym = REG_BUCKET_FIRST(REG_BUCKET(reg));

    while (sym && (sym->reg != reg))
        sym = REG_BUCKET_NEXT(sym);

    return sym;
}

/* look up a label with the given name,
   creating it if it doesn't already exist */

struct symbol *label_lookup(struct string *id)
{
    struct symbol *sym;

    sym = symbol_lookup(SCOPE_LABEL, SCOPE_LABEL, id, S_LABEL);

    if (sym == 0) {
        sym = symbol_new(id, S_LABEL);
        symbol_insert(sym, SCOPE_LABEL);
    }

    return sym;
}

/* caller wants to target a label with the given id.
   return the block it should branch to */

struct block *label_goto(struct string *id)
{
    struct symbol *sym;

    sym = label_lookup(id);
    return sym->to;
}

/* caller wants to define a label with the given
   name. check that the label's not already defined
   and return the block target. */

struct block *label_define(struct string *id)
{
    struct symbol *sym;

    sym = label_lookup(id);
    
    if (sym->ss & S_DEFINED)
        error(FATAL, "label '%S' already defined %1", id, sym);

    sym->ss |= S_DEFINED;
    symbol_here(sym);

    return sym->to;
}

/* clearing a scope discards all the symbols in it. the
   tag symbols are neutered but retained at SCOPE_KEEP,
   just in case a function prototype references them. */

static void scope_clear(scope_level scope)
{
    SCOPE_WALK_ALL(scope, scope, 
        symbol_remove(THIS);

        if (THIS->ss & S_TAG) {
            MEMBERS_INIT(&THIS->members);
            symbol_insert(THIS, SCOPE_KEEP);
        } else {
            symbol_free(THIS);
        }
    );
}

/* move all the symbols from one scope level to another. */

static void scope_move(scope_level from, scope_level to)
{
    SCOPE_WALK_ALL(from, from, symbol_move(THIS, to); );
}

/* print human-readable scope contents to
   the output file for debugging purposes. */

static struct { storage_class ss; char *text; } storage_class_text[] =
{
    { S_STRUCT,     "STRUCT"        },  { S_UNION,      "UNION"         },
    { S_ENUM,       "ENUM"          },  { S_TYPEDEF,    "TYPEDEF"       },
    { S_STATIC,     "STATIC"        },  { S_EXTERN,     "EXTERN"        },
    { S_AUTO,       "AUTO"          },  { S_REGISTER,   "REGISTER"      },
    { S_LOCAL,      "LOCAL"         },  { S_CONST,      "CONST"         },
    { S_MEMBER,     "MEMBER"        },  { S_LABEL,      "LABEL"         },
    { S_REDIRECT,   "REDIRECT"      },  { S_TENTATIVE,  " (tentative)"  },  
    { S_REFERENCED, " (referenced)" },  { S_DEFINED,    " (defined)"    },
    { S_ARG,        " (arg)"        },  { S_IMPLICIT,   " (implicit)"   }
};

static void debug0(struct symbol *sym)
{
    struct symbol *member;
    int i;

    output("# %Z ", sym);

    if (sym->ss & (S_STATIC | S_EXTERN))
        output("[%g] ", sym);

    for (i = 0; i < ARRAY_SIZE(storage_class_text); ++i)
        if (sym->ss & storage_class_text[i].ss)
            output(storage_class_text[i].text);

    if (sym->ss & S_TAG) {
        if (sym->ss & S_DEFINED)
            output(" size = %z, align = %d\n", sym->size, sym->align);
        
        for (member = MEMBERS_FIRST(&sym->members); member;
          member = MEMBERS_NEXT(member))
            debug0(member);
    } else {
        if (sym->reg != PSEUDO_REG_NONE) 
            output(" %r", sym->reg);

        output(" %T", &sym->type);
        
        if (sym->ss & S_CONST) output(" value = %d", sym->value);

        if ((sym->ss & (S_MEMBER | S_AUTO | S_REGISTER | S_LOCAL))
          && (sym->offset))
            output(" offset = %d", sym->offset);
            
        output("\n");
    }
}

void scope_debug(scope_level level)
{
    output("\n# scope ", level);

    switch (level)
    {
    case SCOPE_KEEP:    output("KEEP\n"); break;
    case SCOPE_LURKER:  output("LURKER\n"); break;
    case SCOPE_GLOBAL:  output("GLOBAL\n"); break;
    case SCOPE_RETIRED: output("RETIRED\n"); break;
    case SCOPE_LABEL:   output("LABEL\n"); break;
    default:            output("level %N\n", level);
    }
   
    SCOPE_WALK_ALL(level, level, 
        if (!(THIS->ss & S_MEMBER))
            debug0(THIS); 
    );
}

/* exit a local scope. symbols from current_scope are stashed in
   SCOPE_RETIRED until scope_end_func(). we promote any S_LOCALs
   to S_REGISTER, since we know their addresses won't be taken. */

void scope_exit(void)
{
    SCOPE_WALK_ALL(current_scope, current_scope, 
        if (THIS->ss & S_LOCAL) {
            THIS->ss &= ~S_LOCAL;
            THIS->ss |= S_REGISTER;
        }

        symbol_move(THIS, SCOPE_RETIRED);
    );

    --current_scope;
}

/* find all the formal arguments declared in this function
   and invoke f for each one (in no particular order). */

void scope_walk_args(void (*f)(struct symbol *))
{
    SCOPE_WALK_ALL(SCOPE_RETIRED, SCOPE_RETIRED,
        if (THIS->ss & S_ARG)
            f(THIS);
    );
}

/* finished translating the current function,
   so we can discard its symbols. */

void scope_end_func(void)
{
    scope_clear(SCOPE_RETIRED);
    scope_clear(SCOPE_LABEL);
}

/* exit a prototype scope. such a scope will only contain
   type definitions and their constituent members/constants,
   which we temporarily cache in SCOPE_CONDEMNED, in case
   the prototype is actually a function definition- then we
   scope_resurrect() them to make them accessible again. */

void scope_exit_proto(void)
{
    scope_clear(SCOPE_CONDEMNED);
    scope_move(current_scope, SCOPE_CONDEMNED);
    --current_scope;
}

void scope_resurrect(void)
{
    scope_move(SCOPE_CONDEMNED, current_scope);
}

/* vi: set ts=4 expandtab: */
