/* blks.h - block sets                                  ncc, the new c compiler

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

#ifndef BLKS_H
#define BLKS_H

#include "set.h"

struct block;

SET_DECLARE(blk, struct block *, b)
SET_DECLARE_FREELIST(blk)
SET_DECLARE_ALLOC(blk)
SET_DECLARE_LOOKUP(blk, struct block *)
SET_DECLARE_REMOVE(blk, struct block *)
SET_DECLARE_UNION(blk)
SET_DECLARE_DIFF(blk)
SET_DECLARE_INTERSECT(blk)
SET_DECLARE_MOVE(blk)
SET_DECLARE_CLEAR(blk)

#define BLKS_INITIALIZER(bs)    SET_INITIALIZER(bs)
#define BLKS_INIT(bs)           SET_INIT(bs)
#define BLKS_FIRST(bs)          SET_FIRST(bs)
#define BLKS_COUNT(bs)          SET_COUNT(bs)
#define BLKS_EMPTY(r)           SET_EMPTY(r)
#define BLKS_FOREACH(b, bs)     SET_FOREACH(b, bs)

#define BLKS_ADD(bs, b)         blks_lookup((bs), (b), SET_LOOKUP_CREATE)
#define BLKS_CONTAINS(bs, b)    (blks_lookup((bs), (b), 0) != 0)

extern void blks_output(struct blks *);
extern void blks_all(struct blks *);

#endif /* BLKS_H */

/* vi: set ts=4 expandtab: */
