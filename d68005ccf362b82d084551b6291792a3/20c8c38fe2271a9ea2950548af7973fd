/* bitset.h - large bitsets                             ncc, the new c compiler

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

#ifndef BITSET_H
#define BITSET_H

#include "cc1.h"

struct bitset
{
    unsigned nr_words;
    unsigned *words;
};

extern void bitset_init(struct bitset *, unsigned);
extern void bitset_and(struct bitset *, struct bitset *);
extern void bitset_or(struct bitset *, struct bitset *);
extern void bitset_bic(struct bitset *, struct bitset *);
extern void bitset_zero_all(struct bitset *);
extern void bitset_one_all(struct bitset *);
extern void bitset_copy(struct bitset *, struct bitset *);
extern void bitset_set(struct bitset *, unsigned);
extern void bitset_reset(struct bitset *, unsigned);
extern int bitset_get(struct bitset *, unsigned);
extern bool bitset_same(struct bitset *, struct bitset *);
extern void bitset_clear(struct bitset *);

#endif /* BITSET_H */

/* vi: set ts=4 expandtab: */
