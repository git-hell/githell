/* symbol.h - symbol management                         ncc, the new c compiler

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

#ifndef SYMBOL_H
#define SYMBOL_H

#include "../common/slist.h"
#include "../common/tailq.h"
#include "../common/list.h"
#include "type.h"

/* the symbol table is a layered hash table. each bucket is ordered (in
   decreasing order) by symbol scope. roughly speaking, symbols with a
   scope level equal to or less than the current_scope are visible, but
   but we also define several special scopes that violate this rule. */

typedef int scope_level;

#define SCOPE_LEVEL_PRINTF "%d"

extern scope_level current_scope;

    /* some symbols are kept forever: struct/union/enum tags are
       never freed because prototype arguments might reference them */

#define SCOPE_KEEP          0

    /* 'extern' variables that appear in inner scopes but have
       not been explicitly declared at file scope are lurkers.
       in other words, these are global variables that are not
       visible at SCOPE_GLOBAL. */

#define SCOPE_LURKER        (SCOPE_KEEP + 1)

    /* SCOPE_GLOBAL represents the outermost (file) scope.
       SCOPE_LOCAL..SCOPE_MAX are the localized scopes that
       contain symbols in functions (or function prototypes).

       SCOPE_MAX is an arbitrary number. ANSI requires an
       implementation support a minimum nesting depth of 15. */

#define SCOPE_GLOBAL        (SCOPE_LURKER + 1)
#define SCOPE_LOCAL         (SCOPE_GLOBAL + 1)
#define SCOPE_MAX           1000

    /* labels are unique in that they have function scope,
       so we separate them at their own scope level */

#define SCOPE_LABEL         (SCOPE_MAX + 1)

    /* local symbols get moved to SCOPE_RETIRED when they go
       out of scope until they are ultimately discarded. */

#define SCOPE_RETIRED       (SCOPE_LABEL + 1)

    /* when we exit a function prototype scope, non-argument
       symbols (tags, enum constants) are temporarily moved to
       SCOPE_CONDEMNED. if a definition immediately follows, we
       resurrect them so they're in scope for the definition. */

#define SCOPE_CONDEMNED     (SCOPE_LABEL + 1)

/* a storage_class encompasses the usual c storage class and a few
   other things: the storage_class of a symbol is a basic indicator
   of its kind. the S_* constants form a powerset so that it's easy
   to determine if two storage_classes are in the same namespace. */

typedef int storage_class;  /* S_* */

#define S_NONE          0       /* reserve 0 for "none" */

    /* tags live in the same namespace */

#define S_STRUCT        ( 0x00000001 )
#define S_UNION         ( 0x00000002 )
#define S_ENUM          ( 0x00000004 )

#define S_TAG           ( S_STRUCT | S_UNION | S_ENUM )

    /* most symbols live in the normal namespace. some explanation
       is required here: S_LOCAL is the storage class assigned to a
       block-level symbol without an explicit storage class. if its
       address is taken, it is converted to S_AUTO. if it goes out
       of scope still as an S_LOCAL, it is converted to S_REGISTER.
       this is just a simple way to identify aliased locals. */

#define S_TYPEDEF       ( 0x00000008 )
#define S_STATIC        ( 0x00000010 )
#define S_EXTERN        ( 0x00000020 )
#define S_AUTO          ( 0x00000040 )
#define S_REGISTER      ( 0x00000080 )
#define S_LOCAL         ( 0x00000100 )
#define S_CONST         ( 0x00000200 )      /* enumeration constant */

    /* symbols in the "normal" namespace */

#define S_NORMAL        ( S_TYPEDEF | S_STATIC | S_EXTERN | S_AUTO \
                        | S_REGISTER | S_LOCAL | S_CONST | S_REDIRECT )

    /* block-level symbols */

#define S_BLOCK         ( S_AUTO | S_REGISTER | S_LOCAL )

    /* members and labels live in their own separate worlds */

#define S_MEMBER        ( 0x00000400 )
#define S_LABEL         ( 0x00000800 )

    /* the base storage classes (above) are mutually exclusive */

#define S_BASE_MASK     ( 0x00000FFF )
#define S_BASE(ss)      ((ss) & S_BASE_MASK)

    /* functions which have been declared implicitly at least once */

#define S_IMPLICIT      ( 0x04000000 )

    /* arguments inside a function scope have S_ARG set */

#define S_ARG           ( 0x08000000 )

    /* when a block-level symbol is declared EXTERN, it actually
       references a symbol in either SCOPE_LURKER or SCOPE_GLOBAL,
       and we mark the name in the local scope as an S_REDIRECT. */

#define S_REDIRECT      ( 0x10000000 )

    /* file-scope variable definitions that aren't explicitly
       initialized are S_TENTATIVE; if we get to the end of
       the translation unit without seeing an actual definition,
       we'll issue one then. this is one of those places where
       ANSI really complicated matters (unnecessarily, imho) */

#define S_TENTATIVE     ( 0x20000000 )

    /* once a symbol is actually used in an expression, it is
       marked S_REFERENCED. we only care about this on S_EXTERN
       symbols, so we don't clutter the assembler symbol table. */

#define S_REFERENCED    ( 0x40000000 )

    /* the S_DEFINED flag is used by labels, struct/union tags,
       and file-level objects/functions to indicate that a
       definition has been completed. */

#define S_DEFINED       ( 0x80000000 )

/* symbol table entries cover a lot of ground. the storage_class
   of a symbol determines which fields are valid. we attempt to
   be good citizens by overlapping mutually-exclusive fields. */

SLIST_HEAD(members, symbol);    /* struct members */

/* symbols are assigned a unique number when they're created
   simply to make debugging output intelligible in the face
   of anonymous symbols or ambiguous names */

typedef unsigned symbol_num;

#define SYMBOL_NUM_PRINTF   "#%u"

struct symbol
{
    struct string *id;      /* can be 0 for anonymous symbols */
    scope_level scope;
    storage_class ss;
    symbol_num num;

    /* for better error reporting, we track the first
       appearance or first definition of a symbol here */

    int line_no;
    struct string *path;

    union { /* C11 */

        /* all symbols that are not S_TAGs
           use this first layout */

        struct {
            struct type type;
            pseudo_reg reg;

            /* the last used index for the pseudo-reg */

            int idx;

            /* if this is an argument passed in by register,
               then arg_reg is that [physical] register. */

            pseudo_reg arg_reg;
            
            union {
                asm_label label;    /* S_STATIC */
                int value;          /* S_CONST enumerator value */
                int offset;         /* S_MEMBER/S_AUTO/S_REGISTER/S_LOCAL */
                struct block *to;           /* S_LABEL */
                struct symbol *redirect;    /* S_REDIRECT */
            };

            SLIST_ENTRY(symbol) link;   /* S_MEMBER: member list link */
        };

        /* S_TAGs get their own layout. note that when not S_DEFINED,
           the fields are used as working areas by the parser. */

        struct {
            size_t size;            /* these are in bytes; except that ... */
            int align;              /* ... size is in BITS during parse */
            struct members members;
        };
    };

    TAILQ_ENTRY(symbol) links;
    LIST_ENTRY(symbol) reg_links;
};

extern void symbol_init(void);
extern void symbol_finalize(void);
extern struct symbol *symbol_new(struct string *, storage_class);
extern void symbol_free(struct symbol *);
extern void symbol_here(struct symbol *);
extern void symbol_insert(struct symbol *, scope_level);
extern void symbol_function_class(struct type *, storage_class *);
extern struct symbol *symbol_typename(struct string *);

extern struct symbol *symbol_lookup(scope_level, scope_level,
                                    struct string *, storage_class);

extern struct symbol *symbol_global(struct string *, storage_class,
                                    struct type *, scope_level);

extern struct symbol *symbol_anonymous(asm_label);
extern struct symbol *symbol_temp(struct type *);
extern void symbol_redirect(struct symbol *);
extern pseudo_reg symbol_temp_reg(type_bits);
extern struct symbol *symbol_implicit(struct string *);
extern void symbol_complete_strun(struct symbol *);
extern pseudo_reg symbol_reg(struct symbol *);
extern void symbol_storage(struct symbol *);
extern void symbol_check_labels(void);

/* reset the index counter, and return the next available index */

#define SYMBOL_RESET_IDX(sym)   ((sym)->idx = 0)
#define SYMBOL_NEXT_IDX(sym)    (++((sym)->idx))

/* assume everything that isn't S_REGISTER is aliased. since all locals
   that aren't explicitly declared 'auto' or aliased with '&' are moved
   to S_REGISTER, this isn't as pessimistic as it might at first seem */

#define SYMBOL_ALIASED(sym)     (!((sym)->ss & S_REGISTER))

extern void member_insert(struct symbol *, struct string *, struct type *);
extern struct symbol *member_lookup(struct symbol *, struct string *);

#define MEMBERS_INIT(ms)                SLIST_INIT(ms)
#define MEMBERS_FIRST(ms)               SLIST_FIRST(ms)
#define MEMBERS_NEXT(ms)                SLIST_NEXT(ms, link)
#define MEMBERS_INSERT_HEAD(ms, m)      SLIST_INSERT_HEAD(ms, m, link)
#define MEMBERS_INSERT_AFTER(after, m)  SLIST_INSERT_AFTER(after, m, link)

extern struct symbol *reg_symbol(pseudo_reg);
extern struct symbol *label_lookup(struct string *);
extern struct block *label_define(struct string *);
extern struct block *label_goto(struct string *);

#define REG_RESET_IDX(r)        SYMBOL_RESET_IDX(reg_symbol(r))
#define REG_NEXT_IDX(r)         SYMBOL_NEXT_IDX(reg_symbol(r))

extern void scope_enter(void);
extern void scope_exit(void);
extern void scope_walk_args(void (*)(struct symbol *));
extern void scope_end_func(void);
extern void scope_exit_proto(void);
extern void scope_resurrect(void);
extern void scope_debug(scope_level);

#endif /* SYMBOL_H */

/* vi: set ts=4 expandtab: */
