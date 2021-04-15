/* list.h - LIST macros                                 ncc, the new c compiler

Copyright (c) The Regents of the University of California. All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

   1. Redistributions of source code must retain the above copyright
      notice, this list of conditions and the following disclaimer.
   2. Redistributions in binary form must reproduce the above copyright
      notice, this list of conditions and the following disclaimer in the
      documentation and/or other materials provided with the distribution.
   3. Neither the name of the University nor the names of its contributors
      may be used to endorse or promote products derived from this software
      without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE REGENTS AND CONTRIBUTORS ``AS IS'' AND ANY
EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED.  IN NO EVENT SHALL THE REGENTS OR CONTRIBUTORS BE LIABLE FOR ANY
DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE. */

#ifndef LIST_H
#define LIST_H

/* a list is headed by a single forward pointer (or an array of forward
   pointers for a hash table header). the elements are doubly linked so
   that an arbitrary element can be removed without a need to traverse
   the list. New elements can be added to the list before or after an
   existing element or at the head of the list. a list may only be 
   traversed in the forward direction. */

#define LIST_HEAD(name, type)                                               \
    struct name {                                                           \
        struct type *lh_first;  /* first element */                         \
    }

#define LIST_HEAD_INITIALIZER(head)     { 0 }

#define LIST_ENTRY(type)                                                    \
    struct {                                                                \
        struct type *le_next;   /* next element */                          \
        struct type **le_prev;  /* address of previous next element */      \
    }

#define LIST_INIT(head)                                                     \
    do {                                                                    \
        (head)->lh_first = 0;                                               \
    } while (0)

#define LIST_INSERT_AFTER(listelm, elm, field)                              \
    do {                                                                    \
        if (((elm)->field.le_next = (listelm)->field.le_next) != 0)         \
            (listelm)->field.le_next->field.le_prev =                       \
                &(elm)->field.le_next;                                      \
        (listelm)->field.le_next = (elm);                                   \
        (elm)->field.le_prev = &(listelm)->field.le_next;                   \
    } while (0)

#define LIST_INSERT_BEFORE(listelm, elm, field)                             \
    do {                                                                    \
        (elm)->field.le_prev = (listelm)->field.le_prev;                    \
        (elm)->field.le_next = (listelm);                                   \
        *(listelm)->field.le_prev = (elm);                                  \
        (listelm)->field.le_prev = &(elm)->field.le_next;                   \
    } while (0)

#define LIST_INSERT_HEAD(head, elm, field)                                  \
    do {                                                                    \
        if (((elm)->field.le_next = (head)->lh_first) != 0)                 \
            (head)->lh_first->field.le_prev = &(elm)->field.le_next;        \
        (head)->lh_first = (elm);                                           \
        (elm)->field.le_prev = &(head)->lh_first;                           \
    } while (0)

#define LIST_REMOVE(elm, field)                                             \
    do {                                                                    \
        if ((elm)->field.le_next != 0)                                      \
            (elm)->field.le_next->field.le_prev =                           \
                (elm)->field.le_prev;                                       \
        *(elm)->field.le_prev = (elm)->field.le_next;                       \
    } while (0)

#define LIST_FOREACH(var, head, field)                                      \
    for ((var) = ((head)->lh_first);                                        \
        (var);                                                              \
        (var) = ((var)->field.le_next))

#define LIST_EMPTY(head)            ((head)->lh_first == 0)
#define LIST_FIRST(head)            ((head)->lh_first)
#define LIST_NEXT(elm, field)       ((elm)->field.le_next)

#endif/* LIST_H */

/* vi: set ts=4 expandtab: */
