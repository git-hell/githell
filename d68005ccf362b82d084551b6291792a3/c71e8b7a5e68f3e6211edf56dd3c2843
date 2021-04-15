/* kill.c - kill set computation                        ncc, the new c compiler

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

#include "cc1.h"
#include "insn.h"
#include "blks.h"
#include "regs.h"
#include "block.h"
#include "kill.h"

/* initialize/release block kill data */

void kill_init(struct kill *kill)
{
    REGS_INIT(&kill->kill);
    REGS_INIT(&kill->read);
}

void kill_clear(struct kill *kill)
{
    regs_clear(&kill->kill);
    regs_clear(&kill->read);
}

/* do kill/read analysis for a block. */

static blocks_iter_ret analyze0(struct block *b)
{
    struct insn *insn;

    kill_clear(&b->kill);

    INSNS_FOREACH(insn, &b->insns) {
        insn_defs_regs(insn, &b->kill.kill, 0);
        insn_uses_regs(insn, &b->kill.read, 0);
    }

    return BLOCKS_ITER_OK;
}

void kill_analyze(void)
{
    blocks_iter(analyze0);
}

/* gather kill and/or read sets. add the killed regs from
   all blks to kill_regs (if not 0), and the read regs to
   read_regs (if not 0). if blks is 0, then all blocks in
   the function are included. */

static struct blks *gather_blks;
static struct regs *gather_kill_regs;
static struct regs *gather_read_regs;

static blocks_iter_ret gather0(struct block *b)
{
    if ((gather_blks == 0) || BLKS_CONTAINS(gather_blks, b)) {
        if (gather_kill_regs)
            regs_union(gather_kill_regs, &b->kill.kill);

        if (gather_read_regs)
            regs_union(gather_read_regs, &b->kill.read);
    }

    return BLOCKS_ITER_OK;
}

void kill_gather(struct blks *blks, struct regs *kill_regs,
                                    struct regs *read_regs)
{
    gather_blks = blks;
    gather_kill_regs = kill_regs;
    gather_read_regs = read_regs;
    blocks_iter(gather0);
}

/* gather kill and/or read blocks. scan blks, adding each
   block where reg is in READ to read_blks, and where reg
   is in KILL to kill_blks. the caller can express lack of
   interest in either (or both, though that'd be dumb) of
   these sets by passing in null blks. */

void kill_gather_blks(pseudo_reg reg, struct blks *blks,
                      struct blks *kill_blks, struct blks *read_blks)
{
    struct blk *blk;

    BLKS_FOREACH(blk, blks) {
        if (kill_blks && REGS_CONTAINS(&blk->b->kill.kill, reg))
            BLKS_ADD(kill_blks, blk->b);

        if (read_blks && REGS_CONTAINS(&blk->b->kill.read, reg))
            BLKS_ADD(read_blks, blk->b);
    }
}

/* vi: set ts=4 expandtab: */
