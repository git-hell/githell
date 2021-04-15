/* stack.h - value stack                                ncc, the new c compiler

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

#ifndef STACK_H
#define STACK_H

#include "../common/slist.h"

#define STACK_DECLARE(tag, value_type)                                      \
    struct tag##_cell                                                       \
    {                                                                       \
        value_type value;                                                   \
        SLIST_ENTRY(tag##_cell) link;                                       \
    };                                                                      \
                                                                            \
    SLIST_HEAD(tag, tag##_cell);

#define STACK_INITIALIZER(stk)  SLIST_HEAD_INITIALIZER(stk)
#define STACK_INIT(stk)         SLIST_INIT(stk)
#define STACK_EMPTY(stk)        SLIST_EMPTY(stk)
#define STACK_FOREACH(c, stk)   SLIST_FOREACH(c, stk, link)

/* push a value on the stack */

#define STACK_DECLARE_PUSH(tag, value_type)                                 \
    void tag##_push(struct tag *, value_type);

#define STACK_DEFINE_PUSH(tag, value_type)                                  \
    void tag##_push(struct tag *stack, value_type value)                    \
    {                                                                       \
        struct tag##_cell *cell;                                            \
                                                                            \
        cell = safe_malloc(sizeof(struct tag##_cell));                      \
        cell->value = value;                                                \
        SLIST_INSERT_HEAD(stack, cell, link);                               \
    }

/* pop a value from the stack */

#define STACK_DECLARE_POP(tag, value_type)                                  \
    value_type tag##_pop(struct tag *);

#define STACK_DEFINE_POP(tag, value_type)                                   \
    value_type tag##_pop(struct tag *stack)                                 \
    {                                                                       \
        struct tag##_cell *cell;                                            \
        value_type value;                                                   \
                                                                            \
        cell = SLIST_FIRST(stack);                                          \
        SLIST_REMOVE_HEAD(stack, link);                                     \
        value = cell->value;                                                \
        free(cell);                                                         \
                                                                            \
        return value;                                                       \
    }

/* clear the stack */

#define STACK_DECLARE_CLEAR(tag)                                            \
    void tag##_clear(struct tag *);

#define STACK_DEFINE_CLEAR(tag)                                             \
    void tag##_clear(struct tag *stack)                                     \
    {                                                                       \
        while (!STACK_EMPTY(stack))                                         \
            tag##_pop(stack);                                               \
    }

#endif /* STACK_H */

/* vi: set ts=4 expandtab: */
