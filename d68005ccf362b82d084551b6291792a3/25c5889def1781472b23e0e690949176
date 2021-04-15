/* set.h - set container                                ncc, the new c compiler

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

#ifndef SET_H
#define SET_H

#include "../common/tailq.h"
#include "../common/slist.h"

#define SET_DECLARE(tag, element_type, element_name)                        \
    struct tag##s                                                           \
    {                                                                       \
        unsigned count;                                                     \
        TAILQ_HEAD(, tag);                                                  \
    };                                                                      \
                                                                            \
    struct tag                                                              \
    {                                                                       \
        element_type element_name;                                          \
                                                                            \
        union {                                                             \
            TAILQ_ENTRY(tag) links;                                         \
            SLIST_ENTRY(tag) free_link;                                     \
        };                                                                  \
    };                                                                      \
                                                                            \
    SLIST_HEAD(tag##s_free_list, tag);

#define SET_DECLARE_FREELIST(tag)                                           \
    extern struct tag##s_free_list tag##s_free_list;

#define SET_DEFINE_FREELIST(tag)                                            \
    struct tag##s_free_list tag##s_free_list =                              \
            SLIST_HEAD_INITIALIZER(tag##s_free_list);

#define SET_INIT(set)                                                       \
    do {                                                                    \
        TAILQ_INIT(set);                                                    \
        (set)->count = 0;                                                   \
    } while (0)

#define SET_INITIALIZER(set)            { 0, 0, &(set).tqh_first }
#define SET_FOREACH(i, set)             TAILQ_FOREACH(i, set, links)
#define SET_FIRST(set)                  TAILQ_FIRST(set)
#define SET_NEXT(i)                     TAILQ_NEXT(i, links)
#define SET_COUNT(set)                  ((set)->count)
#define SET_EMPTY(set)                  ((set)->count == 0)

/* put an element back on the free list. internal to set implementation. */

#define SET_FREE(tag, i)  SLIST_INSERT_HEAD(&tag##s_free_list, i, free_link)

/* allocate a number of element structs and returns the first one, queuing
   the rest on the free list. internal to the set implementation. */

#define SET_DECLARE_ALLOC(tag)                                              \
    struct tag *tag##s_alloc(void);

#define SET_DEFINE_ALLOC(tag, slab_qty)                                     \
    struct tag *tag##s_alloc(void)                                          \
    {                                                                       \
        struct tag *slab;                                                   \
        int i;                                                              \
                                                                            \
        slab = safe_malloc(sizeof(struct tag) * slab_qty);                  \
                                                                            \
        for (i = 1; i < slab_qty; ++i)                                      \
            SET_FREE(tag, slab + i);                                        \
                                                                            \
        return slab;                                                        \
    }

#define SET_ALLOC(tag, ptr)                                                 \
    do {                                                                    \
        if (ptr = SLIST_FIRST(&tag##s_free_list))                           \
            SLIST_REMOVE_HEAD(&tag##s_free_list, free_link);                \
        else                                                                \
            ptr = tag##s_alloc();                                           \
    } while (0)

/* lookup returns the entry associated with the element.
   if no such element exists and SET_LOOKUP_CREATE, then
   a new one is created and returned, otherwise returns 0. */

typedef int set_lookup_flags;

#define SET_LOOKUP_CREATE   ( 0x00000001 )

#define SET_DECLARE_LOOKUP(tag, element_type)                               \
    struct tag *tag##s_lookup(struct tag##s *, element_type,                \
                              set_lookup_flags);

#define SET_DEFINE_LOOKUP(tag, element_type, element_name)                  \
                                                                            \
    struct tag *tag##s_lookup(struct tag##s *set, element_type element,     \
                              set_lookup_flags flags)                       \
    {                                                                       \
        struct tag *entry;                                                  \
        struct tag *new;                                                    \
                                                                            \
        TAILQ_FOREACH(entry, set, links) {                                  \
            if (entry->element_name == element)                             \
                return entry;                                               \
                                                                            \
            if (entry->element_name > element)                              \
                break;                                                      \
        }                                                                   \
                                                                            \
        if (flags & SET_LOOKUP_CREATE) {                                    \
            SET_ALLOC(tag, new);                                            \
            new->element_name = element;                                    \
                                                                            \
            if (entry)                                                      \
                TAILQ_INSERT_BEFORE(entry, new, links);                     \
            else                                                            \
                TAILQ_INSERT_TAIL(set, new, links);                         \
                                                                            \
            ++(set->count);                                                 \
            return new;                                                     \
        } else                                                              \
            return 0;                                                       \
    }

#define SET_FREE_ENTRY(tag, set, entry)                                     \
    do {                                                                    \
        --((set)->count);                                                   \
        TAILQ_REMOVE(set, entry, links);                                    \
        SET_FREE(tag, entry);                                               \
    } while(0)

/* remove an element from the set (if it exists) */

#define SET_DECLARE_REMOVE(tag, element_type)                               \
    void tag##s_remove(struct tag##s *, element_type);

#define SET_DEFINE_REMOVE(tag, element_type, element_name)                  \
                                                                            \
    void tag##s_remove(struct tag##s *set, element_type element)            \
    {                                                                       \
        struct tag *entry;                                                  \
                                                                            \
        TAILQ_FOREACH(entry, set, links)                                    \
            if (entry->element_name == element) {                           \
                SET_FREE_ENTRY(tag, set, entry);                            \
                break;                                                      \
            }                                                               \
    }

/* set difference: remove all elements in src from dst */

#define SET_DECLARE_DIFF(tag)                                               \
    void tag##s_diff(struct tag##s *, struct tag##s *);

#define SET_DEFINE_DIFF(tag, element_name)                                  \
                                                                            \
    void tag##s_diff(struct tag##s *dst, struct tag##s *src)                \
    {                                                                       \
        struct tag *entry;                                                  \
                                                                            \
        TAILQ_FOREACH(entry, src, links)                                    \
            tag##s_remove(dst, entry->element_name);                        \
    }

/* set union. add all the elements of src into dst. */

#define SET_DECLARE_UNION(tag)                                              \
    void tag##s_union(struct tag##s *, struct tag##s *);

#define SET_DEFINE_UNION(tag, element_name)                                 \
    void tag##s_union(struct tag##s *dst, struct tag##s *src)               \
    {                                                                       \
        struct tag *src_e;                                                  \
        struct tag *dst_e;                                                  \
        struct tag *new_e;                                                  \
                                                                            \
        dst_e = TAILQ_FIRST(dst);                                           \
                                                                            \
        TAILQ_FOREACH(src_e, src, links) {                                  \
            while (dst_e && (dst_e->element_name < src_e->element_name))    \
                dst_e = TAILQ_NEXT(dst_e, links);                           \
                                                                            \
            if (dst_e && (dst_e->element_name == src_e->element_name))      \
                continue;                                                   \
                                                                            \
            SET_ALLOC(tag, new_e);                                          \
            new_e->element_name = src_e->element_name;                      \
                                                                            \
            if (dst_e)                                                      \
                TAILQ_INSERT_BEFORE(dst_e, new_e, links);                   \
            else                                                            \
                TAILQ_INSERT_TAIL(dst, new_e, links);                       \
                                                                            \
            ++(dst->count);                                                 \
        }                                                                   \
    }

/* set intersection. remove elements from dst that are not in src */

#define SET_DECLARE_INTERSECT(tag)                                          \
    void tag##s_intersect(struct tag##s *, struct tag##s *);

#define SET_DEFINE_INTERSECT(tag, element_name)                             \
    void tag##s_intersect(struct tag##s *dst, struct tag##s *src)           \
    {                                                                       \
        struct tag *dst_e;                                                  \
        struct tag *src_e;                                                  \
        struct tag *tmp;                                                    \
                                                                            \
        dst_e = TAILQ_FIRST(dst);                                           \
        src_e = TAILQ_FIRST(src);                                           \
                                                                            \
        while (dst_e && src_e) {                                            \
            if (dst_e->element_name == src_e->element_name) {               \
                dst_e = TAILQ_NEXT(dst_e, links);                           \
                src_e = TAILQ_NEXT(src_e, links);                           \
            } else if (dst_e->element_name < src_e->element_name) {         \
                tmp = TAILQ_NEXT(dst_e, links);                             \
                SET_FREE_ENTRY(tag, dst, dst_e);                            \
                dst_e = tmp;                                                \
            } else if (dst_e->element_name > src_e->element_name)           \
                src_e = TAILQ_NEXT(src_e, links);                           \
        }                                                                   \
                                                                            \
        while (dst_e) {                                                     \
            tmp = TAILQ_NEXT(dst_e, links);                                 \
            SET_FREE_ENTRY(tag, dst, dst_e);                                \
            dst_e = tmp;                                                    \
        }                                                                   \
    }

/* returns TRUE if the sets intersect (does not modify either set) */

#define SET_DECLARE_OVERLAP(tag)                                            \
    bool tag##s_overlap(struct tag##s *, struct tag##s *);

#define SET_DEFINE_OVERLAP(tag, element_name)                               \
    bool tag##s_overlap(struct tag##s *set1, struct tag##s *set2)           \
    {                                                                       \
        struct tag *set1_e;                                                 \
        struct tag *set2_e;                                                 \
                                                                            \
        set1_e = TAILQ_FIRST(set1);                                         \
        set2_e = TAILQ_FIRST(set2);                                         \
                                                                            \
        while (set1_e && set2_e) {                                          \
            if (set1_e->element_name == set2_e->element_name)               \
                return TRUE;                                                \
                                                                            \
            if (set1_e->element_name < set2_e->element_name)                \
                set1_e = TAILQ_NEXT(set1_e, links);                         \
            else                                                            \
                set2_e = TAILQ_NEXT(set2_e, links);                         \
        }                                                                   \
                                                                            \
        return FALSE;                                                       \
    }

/* return TRUE if two sets have the same elements */

#define SET_DECLARE_EQUAL(tag)                                              \
    bool tag##s_equal(struct tag##s *, struct tag##s *);

#define SET_DEFINE_EQUAL(tag, element_name)                                 \
    bool tag##s_equal(struct tag##s *s1, struct tag##s *s2)                 \
    {                                                                       \
        struct tag *e1;                                                     \
        struct tag *e2;                                                     \
                                                                            \
        if (s1->count != s2->count)                                         \
            return FALSE;                                                   \
                                                                            \
        e1 = TAILQ_FIRST(s1);                                               \
        e2 = TAILQ_FIRST(s2);                                               \
                                                                            \
        while (e1) {                                                        \
            if (e1->element_name != e2->element_name)                       \
                return FALSE;                                               \
                                                                            \
            e1 = TAILQ_NEXT(e1, links);                                     \
            e2 = TAILQ_NEXT(e2, links);                                     \
        }                                                                   \
                                                                            \
        return TRUE;                                                        \
    }

/* move all the elements from src to dst. dst must
   empty on entry, and src is empty on exit. */

#define SET_DECLARE_MOVE(tag)                                               \
    void tag##s_move(struct tag##s *, struct tag##s *);

#define SET_DEFINE_MOVE(tag)                                                \
    void tag##s_move(struct tag##s *dst, struct tag##s *src)                \
    {                                                                       \
        TAILQ_CONCAT(dst, src, links);                                      \
        dst->count = src->count;                                            \
        src->count = 0;                                                     \
    }

/* remove all elements from set */

#define SET_DECLARE_CLEAR(tag)                                              \
    void tag##s_clear(struct tag##s *);             

#define SET_DEFINE_CLEAR(tag)                                               \
                                                                            \
    void tag##s_clear(struct tag##s *set)                                   \
    {                                                                       \
        struct tag *entry;                                                  \
                                                                            \
        while (entry = TAILQ_FIRST(set))                                    \
            SET_FREE_ENTRY(tag, set, entry);                                \
    }

#endif /* SET_H */

/* vi: set ts=4 expandtab: */
