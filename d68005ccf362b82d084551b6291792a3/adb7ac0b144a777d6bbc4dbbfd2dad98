/* type.c - data types                                  ncc, the new c compiler

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
#include "gen.h"
#include "expr.h"
#include "string.h"
#include "symbol.h"
#include "output.h"
#include "target.h"
#include "tree.h"
#include "type.h"

/* allocate and initialize
   a new formal argument */

struct formal *formal_new(void)
{
    struct formal *f;

    f = safe_malloc(sizeof(struct formal));
    TYPE_INIT(&f->type);

    return f;
}

/* allocate a new formal argument
   and make it a copy of src */

struct formal *formal_copy(struct formal *src)
{
    struct formal *dst;

    dst = formal_new();
    dst->is_register = src->is_register;
    dst->id = src->id;
    type_copy(&dst->type, &src->type);

    return dst;
}

/* release formal argument resources
   and free it */

void formal_free(struct formal *f)
{
    type_clear(&f->type);
    free(f);
}

/* free all formals in a list */

static void formals_clear(struct formals *formals)
{
    struct formal *f;

    while (f = FORMALS_FIRST(formals)) {
        FORMALS_REMOVE_HEAD(formals);
        type_clear(&f->type);
        free(f);
    }
}

/* append a formal to a formals list. ensure that
   the identifier (if any) is not duplicated. */

void formals_append(struct formals *formals, struct formal *formal)
{
    struct formal *f;

    f = FORMALS_FIRST(formals);

    if (f == 0)
        FORMALS_INSERT_HEAD(formals, formal);
    else {
        for (;;) {
            if (formal->id && (f->id == formal->id))
                error(FATAL, "duplicate argument name '%S'", formal->id);

            if (FORMALS_NEXT(f))
                f = FORMALS_NEXT(f);
            else
                break;
        }

        FORMALS_INSERT_AFTER(f, formal);
    }
}

/* append of copy of the formals from
   src to the end of dst */

static void formals_copy(struct formals *dst, struct formals *src)
{
    struct formal *f;

    for (f = FORMALS_FIRST(src); f; f = FORMALS_NEXT(f))
        formals_append(dst, formal_copy(f));
}

/* search the formals for the given id
   and return it, or 0 if not present */

struct formal *formal_lookup(struct formals *formals, struct string *id)
{
    struct formal *f;

    for (f = FORMALS_FIRST(formals); f; f = FORMALS_NEXT(f))
        if (f->id == id)
            return f;

    return 0;
}

/* default any as-yet undeclared formals to int */

void formals_default(struct formals *formals)
{
    struct formal *f;

    for (f = FORMALS_FIRST(formals); f; f = FORMALS_NEXT(f))
        if (TYPE_EMPTY(&f->type))
            type_append_bits(&f->type, T_INT);
}

/* declare the formals in the current scope. this is called
   at function entry to make the arguments visible. */

void formals_declare(struct formals *formals)
{
    struct symbol *anon;
    struct symbol *sym;
    struct formal *f;

    for (f = FORMALS_FIRST(formals); f; f = FORMALS_NEXT(f)) {
        if (f->id == 0)
            error(FATAL, "anonymous arguments not allowed");

        sym = symbol_new(f->id, f->is_register ? S_REGISTER : S_LOCAL);
        type_copy(&sym->type, &f->type);
        symbol_insert(sym, current_scope);

        if (f->downcast_float) {
            /* the actual argument will be a double, so
               we declare an anonymous double argument
               and downcast it to the declared float */
            
            anon = symbol_new(0, S_LOCAL | S_ARG);
            type_append_bits(&anon->type, T_DOUBLE);
            symbol_insert(anon, current_scope);
            target->formal_declare(anon);
            gen(assign(tree_sym(sym), tree_sym(anon)), GEN_FLAG_DISCARD);
        } else {
            sym->ss |= S_ARG;
            target->formal_declare(sym);
        }
    }
}

/* check two lists of prototype formal arguments for compatibility. helper
   for type_compat()- see that function for flags and return codes. */

static type_compat_ret formals_compat(struct formals *dst,
                                      struct formals *src,
                                      type_compat_flags flags)
{
    struct formal *dst_f;
    struct formal *src_f;
    type_compat_ret ret;

    for (dst_f = FORMALS_FIRST(dst), src_f = FORMALS_FIRST(src); 
      dst_f && src_f;
      dst_f = FORMALS_NEXT(dst_f), src_f = FORMALS_NEXT(src_f))
    {
        ret = type_compat(&dst_f->type, &src_f->type,
                                        (flags & TYPE_COMPAT_FLAG_COMPOSE)
                                        | TYPE_COMPAT_FLAG_QUALS
                                        | TYPE_COMPAT_FLAG_SKIP);

        if (ret != TYPE_COMPAT_OK) return ret;
    }

    if (dst_f || src_f)
        return TYPE_COMPAT_ERROR;
    else
        return TYPE_COMPAT_OK;
}

/* allocate a new type_node labeled with the
   specified bits, and attempt to initialize
   it with sane values. */

static struct type_node *tn_new(type_bits ts)
{
    struct type_node *tn;

    tn = safe_malloc(sizeof(struct type_node));
    tn->ts = ts;

    if (tn->ts & T_ARRAY) tn->nr_elements = 0;
    if (tn->ts & T_STRUN) tn->tag = 0;
    if (tn->ts & T_FUNC) FORMALS_INIT(&tn->formals);

    return tn;
}

/* allocate a new type node, make it a copy of src. it is
   key that we never copy (or attempt to copy) old-style
   arguments. they are not part of the type and it is
   the parser's problem to track the originals. */

static struct type_node *tn_copy(struct type_node *src)
{
    struct type_node *tn;

    tn = tn_new(src->ts);

    switch (tn->ts & (T_ARRAY | T_STRUN | T_FUNC))
    {
    case T_ARRAY:   tn->nr_elements = src->nr_elements; break;
    case T_STRUN:   tn->tag = src->tag; break;

    case T_FUNC:    if (!(tn->ts & T_OLD_STYLE))
                        formals_copy(&tn->formals, &src->formals);
    }

    return tn;
}

/* check two type_nodes for compatibility. helper for type_compat(),
   see that function for the meaning of flags and return codes. */

static type_compat_ret tn_compat(struct type_node *dst, struct type_node *src,
                                 type_compat_flags flags)
{
    if (T_BASE(dst->ts) != T_BASE(src->ts))
        return TYPE_COMPAT_ERROR;

    switch (T_BASE(dst->ts))
    {
    case T_STRUN:
        if (dst->tag != src->tag)
            return TYPE_COMPAT_ERROR;

        break;

    case T_ARRAY:
        if (dst->nr_elements && src->nr_elements
          && (dst->nr_elements != src->nr_elements))
            return TYPE_COMPAT_ERROR;

        if ((flags & TYPE_COMPAT_FLAG_COMPOSE) && (dst->nr_elements == 0))
            dst->nr_elements = src->nr_elements;

        break;

    case T_FUNC:
        if ((dst->ts & T_OLD_STYLE) && !(src->ts & T_OLD_STYLE)) {
            if (flags & TYPE_COMPAT_FLAG_COMPOSE) {
                dst->ts &= ~T_OLD_STYLE;
                formals_copy(&dst->formals, &src->formals);
            }
        } else if (!(dst->ts & T_OLD_STYLE) && !(src->ts & T_OLD_STYLE)) {
            if ((dst->ts & T_VARIADIC) != (src->ts & T_VARIADIC))
                return TYPE_COMPAT_ERROR;

            return formals_compat(&dst->formals, &src->formals,
                                  (flags & TYPE_COMPAT_FLAG_COMPOSE));
        }
    }

    return TYPE_COMPAT_OK;
}

/* release type_node resources and free it */

static void tn_free(struct type_node *tn)
{
    if (tn->ts & T_FUNC) formals_clear(&tn->formals);
    free(tn);
}

/* type_compat() checks type compatibility, and handles many slightly
   different but closely-related operations. complexity here illustrates
   how poorly type qualifiers and prototypes actually fit into c.

   TYPE_COMPAT_FLAG_COMPOSE:    compose types, update dst with additional
                                type information from src
   TYPE_COMPAT_FLAG_QUALS:      ensure dst and src are identically qualified
   TYPE_COMPAT_FLAG_MERGEQUALS: merge qualifiers from src into dst
   TYPE_COMPAT_FLAG_MOREQUALS:  ensure dst is at least as qualified as src
   TYPE_COMPAT_FLAG_SKIP:       ignore qualifiers at the top level

   return codes:

   TYPE_COMPAT_OK:          all is well, types are compatible
   TYPE_COMPAT_ERROR:       types are not compatible
   TYPE_COMPAT_DISCARD:     types are compatible, but assignment from
                            src to dst would discard qualifiers (only
                            when TYPE_COMPAT_FLAG_MOREQUALS specified) */

type_compat_ret type_compat(struct type *dst, struct type *src,
                            type_compat_flags flags)
{
    struct type_node *dst_tn;
    struct type_node *src_tn;
    type_compat_ret ret;
    char disqualified = 0;

    for (dst_tn = TYPE_FIRST(dst), src_tn = TYPE_FIRST(src); dst_tn && src_tn;
      dst_tn = TYPE_NEXT(dst_tn), src_tn = TYPE_NEXT(src_tn))
    {
        if ((dst_tn != TYPE_FIRST(dst)) || !(flags & TYPE_COMPAT_FLAG_SKIP)) {
            if ((flags & TYPE_COMPAT_FLAG_QUALS) 
              && (T_QUALS(dst_tn->ts) != T_QUALS(src_tn->ts)))
                return TYPE_COMPAT_ERROR;

            if ((flags & TYPE_COMPAT_FLAG_MOREQUALS) &&
              ((T_QUALS(dst_tn->ts) & T_QUALS(src_tn->ts))
              != T_QUALS(src_tn->ts)))
                disqualified = 1;

            if (flags & TYPE_COMPAT_FLAG_MERGEQUALS)
                dst_tn->ts |= T_QUALS(src_tn->ts);
        }

        ret = tn_compat(dst_tn, src_tn, flags);
        if (ret != TYPE_COMPAT_OK) return ret;
    }

    if (dst_tn || src_tn)
        return TYPE_COMPAT_ERROR;

    if (disqualified)
        return TYPE_COMPAT_DISCARD;

    return TYPE_COMPAT_OK;
}

/* allocate a new type_node labeled with the specified bits,
   and prepend or append it to the end of the type. a pointer
   to the new type_node is returned. */

struct type_node *type_prepend_bits(struct type *type, type_bits ts)
{
    struct type_node *tn;

    tn = tn_new(ts);
    TYPE_PREPEND(type, tn);

    return tn;
}

struct type_node *type_append_bits(struct type *type, type_bits ts)
{
    struct type_node *tn;

    tn = tn_new(ts);
    TYPE_APPEND(type, tn);

    return tn;
}

/* free all the type_nodes in a type */

void type_clear(struct type *type)
{
    struct type_node *tn;

    while (tn = TYPE_FIRST(type)) {
        TYPE_REMOVE_HEAD(type);
        tn_free(tn);
    }
}

/* copy the elements of src onto the end of dst */

void type_copy(struct type *dst, struct type *src)
{
    struct type_node *tn;
    struct type_node *tn2;

    for (tn = TYPE_FIRST(src); tn; tn = TYPE_NEXT(tn)) {
        tn2 = tn_copy(tn);
        TYPE_APPEND(dst, tn2);
    }
}

/* fill dst with the dereferenced type of src,
   e.g., src = T_PTR T_INT -> dst = T_INT. also
   valid if the first node is T_ARRAY or T_FUNC. */

void type_deref(struct type *dst, struct type *src)
{
    struct type_node *tn;
    struct type_node *tn2;

    tn = TYPE_FIRST(src);

    while (tn = TYPE_NEXT(tn)) {
        tn2 = tn_copy(tn);
        TYPE_APPEND(dst, tn2);
    }
}

/* fill dst with the referenced type of src,
   e.g., src = T_INT -> dst = T_PTR T_INT */

void type_ref(struct type *dst, struct type *src)
{
    type_copy(dst, src);
    type_prepend_bits(dst, T_PTR);
}

/* apply the qualifiers given to the specified type.
   the only complexity here is that qualifiers on
   arrays apply to the elements, not the arrays. */

void type_qualify(struct type *type, type_bits quals)
{
    struct type_node *tn;

    for (tn = TYPE_FIRST(type);
      tn && (tn->ts & T_ARRAY);
      tn = TYPE_NEXT(tn))
        /* null */;

    tn->ts |= T_QUALS(quals);
}

/* given two (usually different) pointer types, cleanse the qualifiers
   from dst and reapply only the qualifiers present in src. used when
   converting between void and non-void pointers. */

void type_requalify(struct type *dst, struct type *src)
{
    struct type_node *dst_tn;
    struct type_node *src_tn;

    for (dst_tn = TYPE_FIRST(dst),
      src_tn = TYPE_FIRST(src);
      dst_tn; dst_tn = TYPE_NEXT(dst_tn)) {
        dst_tn->ts = T_UNQUAL(dst_tn->ts);
    
        if (src_tn) {
            dst_tn->ts |= T_QUALS(src_tn->ts);
            src_tn = TYPE_NEXT(src_tn);
        }
    }
}

/* adjust the type of functions and arrays to be
   pointers-to-function and pointers-to-element,
   respectively. */

void type_fix_pointer(struct type *type)
{
    struct type_node *tn = TYPE_FIRST(type);

    switch (T_BASE(tn->ts))
    {
    case T_ARRAY:
        tn->ts &= ~T_ARRAY;
        tn->ts |= T_PTR;
        break;
    case T_FUNC:
        type_prepend_bits(type, T_PTR);
        break;
    }
}

/* adjust a pointer-to-array-of-x to be pointer-to-x, as
   happens when an array name appears in an expression. */

void type_fix_array(struct type *type)
{
    struct type_node *tn;

    tn = TYPE_FIRST(type);
    TYPE_REMOVE_HEAD(type);
    tn_free(tn);

    type_fix_pointer(type);
}

/* sanity-check a type. call after parsing a declarator
   and gluing that to a base type. we ensure that:

   1. only first dimension of an array is unbounded
   2. there are no arrays of functions or voids
   3. functions don't return arrays or functions
   4. there are no old-style arguments

   if flags includes TYPE_VALIDATE_ARGSOK then we relax (4)
   somewhat, permitting arguments on the first type_node.

   if flags includes TYPE_VALIDATE_VOIDOK, then plain 'void'
   is permitted, otherwise it's an error. */

void type_validate(struct type *type, type_validate_flags flags)
{
    struct type_node *tn;
    struct type_node *next;

    for (tn = TYPE_FIRST(type); tn; tn = next) {
        next = TYPE_NEXT(tn);

        if (tn->ts & T_ARRAY) {
            if ((next->ts & T_ARRAY) && (next->nr_elements == 0))
                error(FATAL, "illegal array specification");

            if (next->ts & (T_FUNC | T_VOID))
                error(FATAL, "illegal array components");
        }

        if (tn->ts & T_FUNC) {
            if (next->ts & (T_ARRAY | T_FUNC))
                error(FATAL, "illegal return type");

            if ((tn->ts & T_OLD_STYLE) && !FORMALS_EMPTY(&tn->formals)
              && (!(flags & TYPE_VALIDATE_ARGSOK) || (tn != TYPE_FIRST(type))))
                error(FATAL, "old-style argument list not permitted here");
        }

        if ((tn->ts & T_VOID) && (tn == TYPE_FIRST(type))
          && !(flags & TYPE_VALIDATE_VOIDOK))
            error(FATAL, "illegal use of 'void' type");
    }
}

/* return the type_bits adjusted to reflect the machine
   representation: pointers become target-specific ints,
   long doubles shrink to doubles. */

type_bits type_machine_bits(type_bits ts)
{
    if (ts & T_PTR) {
        ts &= ~T_PTR;
        ts |= target->ptr_uint;
    }

    if (ts & T_LDOUBLE) {
        ts &= ~T_LDOUBLE;
        ts |= T_DOUBLE;
    }

    return ts;
}

/* returns the size of the scalar type indicated by ts */

size_t type_bits_sizeof(type_bits ts)
{
    ts = T_BASE(ts);
    ts = type_machine_bits(ts);

    switch (ts)
    {
    case T_UCHAR:
    case T_SCHAR:
    case T_CHAR:        return 1;

    case T_USHORT:
    case T_SHORT:       return 2;

    case T_FLOAT:
    case T_UINT:
    case T_INT:         return 4;

    case T_DOUBLE:
    case T_ULONG:
    case T_LONG:        return 8;
    }
}

/* returns the size of the type. if the TYPE_SIZEOF_TARGET flag is
   given, then the type must be a pointer or array and we return the
   size of the type it points to or its elements (respectively). if
   TYPE_SIZEOF_NOERROR is specified, incomplete types do not result
   in errors (0 is returned instead). */

size_t type_sizeof(struct type *type, type_sizeof_flags flags)
{
    struct type_node *tn;
    long size = 1;

    for (tn = TYPE_FIRST(type); tn; tn = TYPE_NEXT(tn)) {
        if ((flags & TYPE_SIZEOF_TARGET) && (tn == TYPE_FIRST(type)))
            continue;

        switch (T_BASE(tn->ts))
        {
        case T_STRUN:
            if (tn->tag->ss & S_DEFINED)
                size *= tn->tag->size;
            else
                goto incomplete;

            break;

        case T_ARRAY:
            if (tn->nr_elements == 0)
                goto incomplete;

            size *= tn->nr_elements;
            break;

        case T_FUNC:    error(FATAL, "can't take size of function");
        case T_VOID:    error(FATAL, "void type has no size");

        default:
            size *= type_bits_sizeof(tn->ts);
        }

        if (size > MAX_OBJECT_SIZE) error(FATAL, "type/object too large");
        if (T_BASE(tn->ts) == T_PTR) break;
    }
    
    return size;

incomplete:
    if (flags & TYPE_SIZEOF_NOERROR)
        return 0;
    else {
        if (tn->ts & T_ARRAY)
            error(FATAL, "incomplete array type");
        else 
            error(FATAL, "%A is an incomplete type", tn->tag);
    }
}

/* returns the required alignment of the type (in bytes) */

int type_alignof(struct type *type)
{
    struct type_node *tn;
    type_bits ts;

    for (tn = TYPE_FIRST(type); tn; tn = TYPE_NEXT(tn)) {
        ts = type_machine_bits(tn->ts);

        switch (T_BASE(ts))
        {
        case T_ARRAY:       break;
        case T_SCHAR:
        case T_UCHAR:
        case T_CHAR:        return 1;
        case T_USHORT:
        case T_SHORT:       return target->short_align;
        case T_UINT:
        case T_INT:         return target->int_align;
        case T_ULONG:
        case T_LONG:        return target->long_align;
        case T_FLOAT:       return target->float_align;

        case T_DOUBLE:      return target->double_align;

        case T_STRUN:
            if (tn->tag->ss & S_DEFINED)
                return tn->tag->align;
            else
                error(FATAL, "%A is incomplete", tn->tag);
        }
    }
}

/* print human-readable description of type (or type_bits) to the
   output. we spew types out left-to-right, rather than trying to
   reconstruct the c syntax. this is only used debugging. */

static struct { type_bits ts; char *text; } type_bits_text[] =
{
    { T_CONST,      "const "            },
    { T_VOLATILE,   "volatile "         },
    { T_VOID,       "void"              },
    { T_CHAR,       "char"              },
    { T_SCHAR,      "signed char"       },
    { T_UCHAR,      "unsigned char"     },
    { T_SHORT,      "short"             },
    { T_USHORT,     "unsigned short"    },
    { T_INT,        "int"               },
    { T_UINT,       "unsigned"          },
    { T_LONG,       "long"              },
    { T_ULONG,      "unsigned long"     },
    { T_FLOAT,      "float"             },
    { T_DOUBLE,     "double"            },
    { T_LDOUBLE,    "long double"       },
    { T_PTR,        "pointer to "       },
    { T_FLEXIBLE,   "flexible "         },
    { T_ARRAY,      "array ["           },
    { T_FUNC,       "function("         }
};

void type_output_bits(type_bits ts)
{
    int i;

    for (i = 0; i < ARRAY_SIZE(type_bits_text); ++i)
        if (ts & type_bits_text[i].ts)
            output(type_bits_text[i].text);

    if (ts & T_FIELD)
        output(" [size %d shift %d]",
            T_GET_SIZE(ts),
            T_GET_SHIFT(ts));
}

void type_output(struct type *type)
{
    struct type_node *tn;
    struct formal *f;

    for (tn = TYPE_FIRST(type); tn; tn = TYPE_NEXT(tn)) {
        type_output_bits(tn->ts);

        if (tn->ts & T_FUNC) {
            if (!(tn->ts & T_OLD_STYLE)) {
                f = FORMALS_FIRST(&tn->formals);
                if (f == 0) output("void");

                while (f) {
                    type_output(&f->type);

                    if (f = FORMALS_NEXT(f))
                        output(", ");
                }
            }

            if (tn->ts & T_VARIADIC)
                output(", ...");

            output(") returning ");
        }

        if (tn->ts & T_ARRAY) {
            if (tn->nr_elements)
                output("%U", tn->nr_elements);

             output("] of ");
        }

        if (tn->ts & T_STRUN) {
            switch (tn->tag->ss & (S_STRUCT | S_UNION))
            {
            case S_STRUCT:  output("struct"); break;
            case S_UNION:   output("union"); break;
            }

            if (tn->tag->id) output(" %s", tn->tag->id->s);
        }
    }
}

/* vi: set ts=4 expandtab: */
