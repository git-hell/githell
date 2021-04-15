/* live.h - live variable analysis                      ncc, the new c compiler

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

#ifndef LIVE_H
#define LIVE_H

#include "../common/list.h"
#include "cc1.h"
#include "codes.h"
#include "regs.h"
#include "insn.h"

struct block;

/* a range describes a single live range within a block. the basic
   interpretation is that the definition of reg in def reaches all
   uses up to and including last_use, after which point it is dead.
   the seemingly nonsense range (def == last_use) represents a def
   with no uses, i.e., in the absence of aliasing, a dead store.

   def may be INSN_INDEX_BEFORE, which indicates the definition occured
   before the beginning of the block, that is, the register is live-in.
   similarly last_use may be INSN_INDEX_AFTER, meaning the last_use is
   after the end of this block, and thus the register is live-out. 

   when describing the live range of condition_codes (with PSEUDO_REG_CC 
   for reg) last_use may be INSN_INDEX_BRANCH, indicating the use of the
   condition_codes in a conditional branch at the end of the block. */

struct range
{
    pseudo_reg reg;
    insn_index def;
    insn_index last;
    int uses;
    TAILQ_ENTRY(range) links;
};

#define RANGE_DEAD(r)       ((r)->uses == 0)

/* and a struct live holds the live-variable analysis data for a block.
   this includes the sets of live-in and live-out registers, as well as
   a list of all the live ranges for all registers in the block. ranges
   are ordered by (reg, def), such that all ranges for a given register
   are adjacent, and earlier definitions precede later ones. */

struct live
{
    TAILQ_HEAD(, range);
    struct regs in;         /* live in */
    struct regs out;        /* live out */
    struct regs def;        /* registers whose first appearance is a def */
    struct regs use;        /* ..................................... use */
    struct regs kill;       /* registers DEFd in this block */
};

extern void live_init(struct live *);
extern void live_clear(struct live *);
extern void live_analyze_ccs(struct block *);
extern void live_analyze(void);
extern struct range *range_by_def(struct live *, pseudo_reg, insn_index);
extern struct range *range_by_use(struct live *, pseudo_reg, insn_index);
extern void live_interf(struct live *, pseudo_reg, struct regs *);
extern bool live_kill_insn(struct live *, insn_index);
extern ccset live_ccs(struct block *, struct insn *);

extern void live_debug(struct live *);

#endif /* LIVE_H */

/* vi: set ts=4 expandtab: */
