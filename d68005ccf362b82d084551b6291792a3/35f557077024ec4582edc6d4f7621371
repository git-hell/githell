/* regs.h - register sets                               ncc, the new c compiler

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

#ifndef REGS_H
#define REGS_H

#include "set.h"
#include "cc1.h"

SET_DECLARE(reg, pseudo_reg, reg)
SET_DECLARE_FREELIST(reg)
SET_DECLARE_ALLOC(reg)
SET_DECLARE_LOOKUP(reg, pseudo_reg)
SET_DECLARE_REMOVE(reg, pseudo_reg)
SET_DECLARE_OVERLAP(reg)
SET_DECLARE_DIFF(reg)
SET_DECLARE_UNION(reg)
SET_DECLARE_MOVE(reg)
SET_DECLARE_EQUAL(reg)
SET_DECLARE_CLEAR(reg)

#define REGS_INIT(r)            SET_INIT(r)
#define REGS_INITIALIZER(r)     SET_INITIALIZER(r)
#define REGS_FIRST(r)           SET_FIRST(r)
#define REGS_NEXT(r)            SET_NEXT(r)
#define REGS_FOREACH(v, r)      SET_FOREACH(v, r)
#define REGS_EMPTY(r)           SET_EMPTY(r)
#define REGS_COUNT(r)           SET_COUNT(r)

#define REGS_ADD(rs, r)         regs_lookup((rs), (r), SET_LOOKUP_CREATE)
#define REGS_CONTAINS(rs, r)    (regs_lookup((rs), (r), 0) != 0)

extern void regs_output(struct regs *);
extern void regs_eliminate_base(struct regs *, pseudo_reg);
extern void regs_eliminate_bases(struct regs *, struct regs *);
extern void regs_replace_base(struct regs *, pseudo_reg);
extern void regs_select_base(struct regs *, struct regs *, pseudo_reg);
extern void regs_select_bases(struct regs *, struct regs *, struct regs *);
extern void regs_intersect_bases(struct regs *, struct regs *);

#endif /* REGS_H */

/* vi: set ts=4 expandtab: */
