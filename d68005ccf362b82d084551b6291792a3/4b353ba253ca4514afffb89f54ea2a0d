/* type.h - data types                                  ncc, the new c compiler

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

#ifndef TYPE_H
#define TYPE_H

#include <stddef.h>
#include "../common/stailq.h"
#include "../common/slist.h"

/* types are represented as linked lists of type_nodes, which
   when read head to tail quite naturally describe the type. */

STAILQ_HEAD(type, type_node);       /* struct type */

SLIST_HEAD(formals, formal);        /* struct formals */

typedef long type_bits;             /* T_* (below) */

struct type_node
{
    type_bits ts;

    union /* C11 */
    {
        size_t nr_elements;         /* T_ARRAY (0 == unbounded) */
        struct symbol *tag;         /* T_STRUN */
        struct formals formals;     /* T_FUNC */
    };
    
    STAILQ_ENTRY(type_node) link;
};

#define TYPE_INITIALIZER(ty)    STAILQ_HEAD_INITIALIZER(ty)
#define TYPE_INIT(ty)           STAILQ_INIT(ty)
#define TYPE_FIRST(ty)          STAILQ_FIRST(ty)
#define TYPE_NEXT(ty)           STAILQ_NEXT(ty, link)
#define TYPE_PREPEND(ty, tn)    STAILQ_INSERT_HEAD(ty, tn, link)
#define TYPE_APPEND(ty, tn)     STAILQ_INSERT_TAIL(ty, tn, link)
#define TYPE_REMOVE_HEAD(ty)    STAILQ_REMOVE_HEAD(ty, link)
#define TYPE_EMPTY(ty)          STAILQ_EMPTY(ty)
#define TYPE_CONCAT(dst, src)   STAILQ_CONCAT(dst, src)

extern struct type_node *type_append_bits(struct type *, type_bits);
extern struct type_node *type_prepend_bits(struct type *, type_bits);
extern void type_qualify(struct type *, type_bits);
extern void type_requalify(struct type *, struct type *);
extern void type_clear(struct type *);
extern void type_copy(struct type *, struct type *);
extern void type_ref(struct type *, struct type *);
extern void type_deref(struct type *, struct type *);
extern void type_fix_pointer(struct type *);
extern void type_fix_array(struct type *);
extern int type_alignof(struct type *);
extern type_bits type_machine_bits(type_bits);
extern size_t type_bits_sizeof(type_bits);

typedef int type_sizeof_flags; /* TYPE_SIZEOF_* */

#define TYPE_SIZEOF_TARGET  ( 0x00000001 )  /* size of the type pointed to */
#define TYPE_SIZEOF_NOERROR ( 0x00000002 )  /* return 0 instead of error */

extern size_t type_sizeof(struct type *, type_sizeof_flags);

extern void type_output(struct type *);
extern void type_output_bits(type_bits);

#define TYPE_BITS(ty)           (TYPE_FIRST(ty)->ts)
#define TYPE_BASE(ty)           (T_BASE(TYPE_FIRST(ty)->ts))

/* if the type is an old-style function with arguments */

#define TYPE_OLD_DEF(ty)                                                    \
        ((((TYPE_FIRST(ty)->ts) & (T_FUNC | T_OLD_STYLE))                   \
            == (T_FUNC | T_OLD_STYLE))                                      \
        && !FORMALS_EMPTY(&(TYPE_FIRST(ty)->formals)))

#define TYPE_VOID(ty)       (TYPE_BASE(ty) & T_VOID)
#define TYPE_PTR(ty)        (TYPE_BASE(ty) & T_PTR)
#define TYPE_FUNC(ty)       (TYPE_BASE(ty) & T_FUNC)
#define TYPE_FORMALS(ty)    (TYPE_FIRST(ty)->formals)
#define TYPE_FLOAT(ty)      (TYPE_BASE(ty) & T_FLOAT)
#define TYPE_FLOATING(ty)   (TYPE_BASE(ty) & T_FLOATING)
#define TYPE_ARITH(ty)      (TYPE_BASE(ty) & T_ARITH)
#define TYPE_CHARS(ty)      (TYPE_BASE(ty) & T_CHARS)
#define TYPE_SHORTS(ty)     (TYPE_BASE(ty) & T_SHORTS)
#define TYPE_INTEGRAL(ty)   (TYPE_BASE(ty) & T_INTEGRAL)
#define TYPE_SIGNED(ty)     (TYPE_BASE(ty) & T_SIGNED)
#define TYPE_UNSIGNED(ty)   (TYPE_BASE(ty) & T_UNSIGNED)
#define TYPE_DISCRETE(ty)   (TYPE_BASE(ty) & T_DISCRETE)
#define TYPE_SCALAR(ty)     (TYPE_BASE(ty) & T_SCALAR)
#define TYPE_ARRAY(ty)      (TYPE_BASE(ty) & T_ARRAY)
#define TYPE_STRUN(ty)      (TYPE_BASE(ty) & T_STRUN)
#define TYPE_TERMINAL(ty)   (TYPE_BASE(ty) & T_TERMINAL)

#define TYPE_CONST(ty)      (TYPE_BITS(ty) & T_CONST)
#define TYPE_VOLATILE(ty)   (TYPE_BITS(ty) & T_VOLATILE)
#define TYPE_VARIADIC(ty)   (TYPE_BITS(ty) & T_VARIADIC)
#define TYPE_OLD_STYLE(ty)  (TYPE_BITS(ty) & T_OLD_STYLE)

#define TYPE_QUALS(ty)      (TYPE_BITS(ty) & T_QUAL_MASK)

#define TYPE_SAME_CLASS(ty1, ty2) T_SAME_CLASS(TYPE_BITS(ty1), TYPE_BITS(ty2))

/* properties of struct/union aggregate */

#define TYPE_STRUN_TAG(ty)      (TYPE_FIRST(ty)->tag)
#define TYPE_STRUN_UNION(ty)    (TYPE_STRUN_TAG(ty)->ss & S_UNION)
#define TYPE_STRUN_MEMBERS(ty)  (TYPE_FIRST(ty)->tag->members)

#define TYPE_STRUN_PTR(ty)      (TYPE_PTR(ty) &&                        \
                                (TYPE_NEXT(TYPE_FIRST(ty))->ts & T_STRUN))

#define TYPE_STRUN_PTR_TAG(ty)  (TYPE_NEXT(TYPE_FIRST(ty))->tag)

#define TYPE_ANONYMOUS_STRUN(ty)    (TYPE_STRUN(ty) &&                      \
                                    (TYPE_STRUN_TAG(ty)->id == 0))

/* arrays and properties of arrays */

#define TYPE_CHAR_ARRAY(ty)     (TYPE_ARRAY(ty) &&                          \
                                (TYPE_NEXT(TYPE_FIRST(ty))->ts & T_CHARS))

#define TYPE_UNBOUNDED_ARRAY(ty)        (TYPE_ARRAY(ty) &&                  \
                                        (TYPE_FIRST(ty)->nr_elements == 0))

#define TYPE_FLEXIBLE(ty)               (TYPE_BITS(ty) & T_FLEXIBLE)
#define TYPE_MARK_FLEXIBLE(ty)          (TYPE_BITS(ty) |= T_FLEXIBLE)

#define TYPE_GET_NR_ELEMENTS(ty)        (TYPE_FIRST(ty)->nr_elements)
#define TYPE_SET_NR_ELEMENTS(ty, n)     (TYPE_FIRST(ty)->nr_elements = (n))

/* remove all top-level qualifiers */

#define TYPE_UNQUAL(ty)     (TYPE_FIRST(ty)->ts &= ~T_QUAL_MASK)

/* type is pointer to func */

#define TYPE_FUNC_PTR(ty)   (TYPE_PTR(ty) &&                                \
                            (TYPE_NEXT(TYPE_FIRST(ty))->ts & T_FUNC))

#define TYPE_VARIADIC_FUNC_PTR(ty)                                          \
                            (TYPE_FUNC_PTR(ty) &&                           \
                            (TYPE_NEXT(TYPE_FIRST(ty))->ts & T_VARIADIC))

/* qualifiers of the type pointed to */

#define TYPE_PTR_QUALS(ty)      T_QUALS(TYPE_NEXT(TYPE_FIRST(ty))->ts)
#define TYPE_VOLATILE_PTR(ty)   (TYPE_PTR_QUALS(ty) & T_VOLATILE)

/* type is pointer to void */

#define TYPE_VOID_PTR(ty)   (TYPE_PTR(ty) &&                                \
                            (TYPE_NEXT(TYPE_FIRST(ty))->ts & T_VOID))

/* if the type is a bitfield, or pointer to a bitfield */

#define TYPE_FIELD(ty)          (TYPE_FIRST(ty)->ts & T_FIELD)
#define TYPE_GET_SIZE(ty)       (T_GET_SIZE(TYPE_FIRST(ty)->ts))
#define TYPE_GET_SHIFT(ty)      (T_GET_SHIFT(TYPE_FIRST(ty)->ts))

#define TYPE_FIELD_PTR(ty)      (TYPE_PTR(ty) &&                            \
                                (TYPE_NEXT(TYPE_FIRST(ty))->ts & T_FIELD))

#define TYPE_FIELD_PTR_SIZE(ty)   (T_GET_SIZE(TYPE_NEXT(TYPE_FIRST(ty))->ts))
#define TYPE_FIELD_PTR_SHIFT(ty)  (T_GET_SHIFT(TYPE_NEXT(TYPE_FIRST(ty))->ts))

#define TYPE_SET_SIZE(ty, sz)                                               \
    do {                                                                    \
        T_SET_SIZE(TYPE_FIRST(ty)->ts, (sz));                               \
        TYPE_FIRST(ty)->ts |= T_FIELD;                                      \
    } while (0)

#define TYPE_SET_SHIFT(ty, ofs)                                             \
    (T_SET_SHIFT(TYPE_FIRST(ty)->ts, (ofs)))

/* remove bitfield information */

#define TYPE_UNFIELD(ty)        (TYPE_FIRST(ty)->ts &= ~T_FIELD)

typedef int type_compat_flags;  /* TYPE_COMPAT_FLAG_* */

#define TYPE_COMPAT_FLAG_COMPOSE    ( 0x00000001 )
#define TYPE_COMPAT_FLAG_QUALS      ( 0x00000002 )
#define TYPE_COMPAT_FLAG_SKIP       ( 0x00000004 )
#define TYPE_COMPAT_FLAG_MERGEQUALS ( 0x00000008 )
#define TYPE_COMPAT_FLAG_MOREQUALS  ( 0x00000010 )

typedef int type_compat_ret;    /* TYPE_COMPAT_* */

#define TYPE_COMPAT_OK              ( 0 )
#define TYPE_COMPAT_ERROR           ( 1 )
#define TYPE_COMPAT_DISCARD         ( 2 )

extern type_compat_ret type_compat(struct type *, struct type *,
                                   type_compat_flags);

typedef int type_validate_flags;        /* TYPE_VALIDATE_* */

#define TYPE_VALIDATE_ARGSOK        ( 0x00000001 )
#define TYPE_VALIDATE_VOIDOK        ( 0x00000002 )

extern void type_validate(struct type *, type_validate_flags);

    /* for T_OLD_STYLE functions, formals only exist in the type
       briefly, as a convenience during parsing of the old-style
       definitions; outside of this brief window, formals is empty.

       for new-style functions, an empty formals means it takes
       no arguments (there is no explicit VOID argument). */

struct formal
{
    char is_register;           /* declared 'register' */
    char downcast_float;        /* must downcast double to float */
    struct string *id;          /* can be 0 */
    struct type type;           /* can be empty if T_OLD_STYLE */

    SLIST_ENTRY(formal) link;
};

#define FORMALS_INITIALIZER(fs)         SLIST_HEAD_INITIALIZER(fs)
#define FORMALS_INIT(fs)                SLIST_INIT(fs)
#define FORMALS_FIRST(f)                SLIST_FIRST(f)
#define FORMALS_NEXT(f)                 SLIST_NEXT(f, link)
#define FORMALS_INSERT_HEAD(fs, f)      SLIST_INSERT_HEAD(fs, f, link)
#define FORMALS_INSERT_AFTER(after, f)  SLIST_INSERT_AFTER(after, f, link)
#define FORMALS_REMOVE_HEAD(fs)         SLIST_REMOVE_HEAD(fs, link)
#define FORMALS_EMPTY(fs)               SLIST_EMPTY(fs)

extern struct formal *formal_new(void);
extern void formal_free(struct formal *);
extern void formals_append(struct formals *, struct formal *);
extern struct formal *formal_lookup(struct formals *, struct string *);
extern void formals_default(struct formals *);
extern void formals_declare(struct formals *);

    /* the base type bits are mutually exclusive. they're arranged
       as a powerset so we can easily match groups of types. note:
       the code which handles the usual promotions (expr.c) cheats
       by relying on the values of T_INT .. T_LDOUBLE. */
                        
#define T_VOID          ( 0x0000000000000001L ) 
#define T_CHAR          ( 0x0000000000000002L ) 
#define T_SCHAR         ( 0x0000000000000004L ) 
#define T_UCHAR         ( 0x0000000000000008L ) 
#define T_SHORT         ( 0x0000000000000010L ) 
#define T_USHORT        ( 0x0000000000000020L ) 
#define T_INT           ( 0x0000000000000040L ) 
#define T_UINT          ( 0x0000000000000080L ) 
#define T_LONG          ( 0x0000000000000100L ) 
#define T_ULONG         ( 0x0000000000000200L ) 
#define T_FLOAT         ( 0x0000000000000400L ) 
#define T_DOUBLE        ( 0x0000000000000800L ) 
#define T_LDOUBLE       ( 0x0000000000001000L ) 
#define T_ARRAY         ( 0x0000000000002000L ) 
#define T_FUNC          ( 0x0000000000004000L ) 
#define T_PTR           ( 0x0000000000008000L ) 
#define T_STRUN         ( 0x0000000000010000L ) 

#define T_BASE_MASK     ( 0x000000000001FFFFL ) 
#define T_BASE(ts)      ((ts) & T_BASE_MASK)
    
    /* type qualifiers are obviously not mutually exclusive */

#define T_CONST         ( 0x0000000000020000L ) 
#define T_VOLATILE      ( 0x0000000000040000L ) 

#define T_QUAL_MASK     ( 0x0000000000060000L ) 
#define T_QUALS(ts)     ((ts) & T_QUAL_MASK)
#define T_UNQUAL(ts)    ((ts) & ~T_QUAL_MASK)

    /* if T_FIELD is set, then the type is a bit-field.
       the base type (which is integral) gives the size
       of the word in which the field is embedded. the
       two embedded fields covered by T_SHIFT_MASK and 
       T_SIZE_MASK give the shift (in bits from the lsb)
       and size (in bits) of the field in the word. */

#define T_FIELD         ( 0x0000000080000000L ) 
#define T_SHIFT_MASK    ( 0x0000003F00000000L )     /* 0-63 */
#define T_SIZE_MASK     ( 0x00001FC000000000L )     /* 0-64 */

#define T_GET_SHIFT(ts)         (((ts) & T_SHIFT_MASK) >> 32)
#define T_SET_SHIFT(ts, ofs)    ((ts) |= (((ofs) << 32) & T_SHIFT_MASK))
#define T_GET_SIZE(ts)          (((ts) & T_SIZE_MASK) >> 38)
#define T_SET_SIZE(ts, sz)      ((ts) |= (((sz) << 38) & T_SIZE_MASK))

    /* T_ARRAY is a flexible array member */

#define T_FLEXIBLE      ( 0x2000000000000000L )

    /* T_FUNC is not a prototype */
    
#define T_OLD_STYLE     ( 0x4000000000000000L )

    /* T_FUNC has a variadic argument list */

#define T_VARIADIC      ( 0x8000000000000000L )

    /* useful classes */

#define T_CHARS         ( T_CHAR | T_SCHAR | T_UCHAR )
#define T_SHORTS        ( T_SHORT | T_USHORT )
#define T_INTS          ( T_INT | T_UINT )
#define T_LONGS         ( T_LONG | T_ULONG )
#define T_FLOATING      ( T_FLOAT | T_DOUBLE | T_LDOUBLE)

#define T_SIGNED        ( T_SCHAR | T_SHORT | T_INT | T_LONG )
#define T_UNSIGNED      ( T_CHAR | T_UCHAR | T_USHORT | T_UINT | T_ULONG )

#define T_INTEGRAL      ( T_CHARS | T_SHORTS | T_INTS | T_LONGS )
#define T_ARITH         ( T_INTEGRAL | T_FLOATING )
#define T_DISCRETE      ( T_INTEGRAL | T_PTR )
#define T_SCALAR        ( T_ARITH | T_PTR )

#define T_ANY           ( ~(type_bits) 0 )

    /* terminals only appear in the final type_node */

#define T_TERMINAL      ( T_INTEGRAL | T_FLOATING | T_VOID | T_STRUN )

    /* true if both sets of type_bits are in the same class,
       that is, both discrete or both floating-point. */

#define T_SAME_CLASS(t1,t2)                                                 \
        ((((t1) & T_FLOATING) && ((t2) & T_FLOATING))                       \
        || (((t1) & T_DISCRETE) && ((t2) & T_DISCRETE)))

#endif /* TYPE_H */

/* vi: set ts=4 expandtab: */
