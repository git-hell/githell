/* assoc.h - associative container                      ncc, the new c compiler

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

#ifndef ASSOC_H
#define ASSOC_H

#include "../common/list.h"
#include "cc1.h"

#define ASSOC_NULL_CONSTRUCT(x)     /* null construct and destruct for ... */
#define ASSOC_NULL_DESTRUCT(x)      /* ... value types that need none */

#define ASSOC_DECLARE(tag, key_type, key_name, value_type, value_name)      \
    struct tag##s                                                           \
    {                                                                       \
        unsigned count;                                                     \
        LIST_HEAD(, tag);                                                   \
    };                                                                      \
                                                                            \
    struct tag                                                              \
    {                                                                       \
        key_type key_name;                                                  \
        value_type value_name;                                              \
        LIST_ENTRY(tag) links;                                              \
    };

#define ASSOC_INIT(assoc)                                                   \
    do {                                                                    \
        LIST_INIT(assoc);                                                   \
        (assoc)->count = 0;                                                 \
    } while (0)

#define ASSOC_INITIALIZER(assoc)        LIST_HEAD_INITIALIZER(assoc)
#define ASSOC_FOREACH(i, assoc)         LIST_FOREACH(i, assoc, links)
#define ASSOC_FIRST(assoc)              LIST_FIRST(assoc)
#define ASSOC_NEXT(i)                   LIST_NEXT(i, links)
#define ASSOC_COUNT(assoc)              ((assoc)->count)

/* lookup returns the entry associated with
   the key, or 0 if no such entry exists. */

#define ASSOC_DECLARE_LOOKUP(tag, key_type)                                 \
    struct tag *tag##s_lookup(struct tag##s *, key_type);

#define ASSOC_DEFINE_LOOKUP(tag, key_type, key_name)                        \
                                                                            \
    struct tag *tag##s_lookup(struct tag##s *assoc, key_type key)           \
    {                                                                       \
        struct tag *entry;                                                  \
                                                                            \
        LIST_FOREACH(entry, assoc, links)                                   \
            if (entry->key_name == key)                                     \
                return entry;                                               \
                                                                            \
        return 0;                                                           \
    }

/* like lookup, but creates a new entry if none exists */

#define ASSOC_DECLARE_INSERT(tag, key_type)                                 \
    struct tag *tag##s_insert(struct tag##s *, key_type);
    
#define ASSOC_DEFINE_INSERT(tag, key_type, key_name, value_name, construct) \
                                                                            \
    struct tag *tag##s_insert(struct tag##s *assoc, key_type key)           \
    {                                                                       \
        struct tag *entry;                                                  \
                                                                            \
        LIST_FOREACH(entry, assoc, links)                                   \
            if (entry->key_name == key)                                     \
                break;                                                      \
                                                                            \
        if (entry == 0) {                                                   \
            entry = safe_malloc(sizeof(struct tag));                        \
            entry->key_name = key;                                          \
            construct(&(entry->value_name));                                \
            LIST_INSERT_HEAD(assoc, entry, links);                          \
            ++(assoc->count);                                               \
        }                                                                   \
                                                                            \
        return entry;                                                       \
    }

/* duplicate an entire container. dst must be initialized and empty */

#define ASSOC_DECLARE_DUP(tag)                                              \
    void tag##s_dup(struct tag##s *, struct tag##s *);

#define ASSOC_DEFINE_DUP(tag, key_type, key_name, value_name, dup)          \
                                                                            \
    void tag##s_dup(struct tag##s *dst, struct tag##s *src)                 \
    {                                                                       \
        struct tag *src_entry;                                              \
        struct tag *dst_entry;                                              \
                                                                            \
        ASSOC_FOREACH(src_entry, src) {                                     \
            dst_entry = safe_malloc(sizeof(struct tag));                    \
            dst_entry->key_name = src_entry->key_name;                      \
            dup(&(dst_entry->value_name), &(src_entry->value_name));        \
            LIST_INSERT_HEAD(dst, dst_entry, links);                        \
            ++(dst->count);                                                 \
        }                                                                   \
    }

/* unset removes any entry associated with key */

#define ASSOC_DECLARE_UNSET(tag, key_type)                                  \
    void tag##s_unset(struct tag##s *, key_type);

#define ASSOC_DEFINE_UNSET(tag, key_type, key_name, value_name, destruct)   \
                                                                            \
    void tag##s_unset(struct tag##s *assoc, key_type key)                   \
    {                                                                       \
        struct tag *entry;                                                  \
                                                                            \
        LIST_FOREACH(entry, assoc, links)                                   \
            if (entry->key_name == key) {                                   \
                --(assoc->count);                                           \
                LIST_REMOVE(entry, links);                                  \
                destruct(&(entry->value_name));                             \
                free(entry);                                                \
                break;                                                      \
            }                                                               \
    }

/* clear removes all associations */

#define ASSOC_DECLARE_CLEAR(tag)                                            \
    void tag##s_clear(struct tag##s *);             

#define ASSOC_DEFINE_CLEAR(tag, value_name, destruct)                       \
                                                                            \
    void tag##s_clear(struct tag##s *assoc)                                 \
    {                                                                       \
        struct tag *entry;                                                  \
                                                                            \
        while (entry = LIST_FIRST(assoc)) {                                 \
            --(assoc->count);                                               \
            LIST_REMOVE(entry, links);                                      \
            destruct(&(entry->value_name));                                 \
            free(entry);                                                    \
        }                                                                   \
    }

/* move all associations from dst to src. src should
   be empty on entry, and dst will be empty on exit. */

#define ASSOC_DECLARE_MOVE(tag)                                             \
    void tag##s_move(struct tag##s *, struct tag##s *);

#define ASSOC_DEFINE_MOVE(tag)                                              \
                                                                            \
    void tag##s_move(struct tag##s *dst, struct tag##s *src)                \
    {                                                                       \
        struct tag *entry;                                                  \
                                                                            \
        while (entry = LIST_FIRST(src)) {                                   \
            --(src->count);                                                 \
            LIST_REMOVE(entry, links);                                      \
            LIST_INSERT_HEAD(dst, entry, links);                            \
           ++(dst->count);                                                  \
        }                                                                   \
    }

/* returns TRUE if both containers contain the same
   elements with the same values, false otherwise. */

#define ASSOC_DECLARE_SAME(tag)                                             \
    bool tag##s_same(struct tag##s *, struct tag##s *);

#define ASSOC_DEFINE_SAME(tag, key_name, value_name, same)                  \
    bool tag##s_same(struct tag##s *x, struct tag##s *y)                    \
    {                                                                       \
        struct tag *x_entry;                                                \
        struct tag *y_entry;                                                \
                                                                            \
        if (x->count != y->count)                                           \
            return FALSE;                                                   \
                                                                            \
        ASSOC_FOREACH(x_entry, x) {                                         \
            y_entry = tag##s_lookup(y, x_entry->key_name);                  \
                                                                            \
            if ((y_entry == 0) || !same(&y_entry->value_name,               \
                                       &x_entry->value_name))               \
                return FALSE;                                               \
        }                                                                   \
                                                                            \
        return TRUE;                                                        \
    }

#endif /* ASSOC_H */

/* vi: set ts=4 expandtab: */
