/* bitset.c - large bitsets                             ncc, the new c compiler

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

#include <string.h>
#include "../common/util.h"
#include "cc1.h"
#include "bitset.h"

#define BITS_PER_WORD   (sizeof(unsigned) * BITS_PER_BYTE)
#define BYTES_PER_WORD  (BITS_PER_WORD / BITS_PER_BYTE)

#define WORD_INDEX(n)   (((int) (n)) / BITS_PER_WORD)
#define BIT_INDEX(n)    (((int) (n)) % BITS_PER_WORD)

/* initialize a new bitset large enough to hold n bits */

void bitset_init(struct bitset *bitset, unsigned n)
{
    n = ROUND_UP(n, BITS_PER_WORD);

    bitset->nr_words = n / BITS_PER_WORD;
    bitset->words = safe_malloc(n / BITS_PER_BYTE);
    bitset_zero_all(bitset);
}

/* release bitset resources */

void bitset_clear(struct bitset *bitset)
{
    free(bitset->words);
}

/* set all bits to zero or one */

void bitset_zero_all(struct bitset *bitset)
{
    memset(bitset->words, 0, bitset->nr_words * BYTES_PER_WORD);
}

void bitset_one_all(struct bitset *bitset)
{
    memset(bitset->words, 0xFF, bitset->nr_words * BYTES_PER_WORD);
}

/* set or reset the nth bit */

void bitset_set(struct bitset *bitset, unsigned n)
{
    bitset->words[WORD_INDEX(n)] |= (1 << BIT_INDEX(n));
}

void bitset_reset(struct bitset *bitset, unsigned n)
{
    bitset->words[WORD_INDEX(n)] &= ~(1 << BIT_INDEX(n));
}

/* returns the value of bit n in the set */

int bitset_get(struct bitset *bitset, unsigned n)
{
    return bitset->words[WORD_INDEX(n)] & (1 << BIT_INDEX(n));
}

/* make dst a copy of src */

void bitset_copy(struct bitset *dst, struct bitset *src)
{
    int i;

    for (i = 0; i < dst->nr_words; ++i)
        dst->words[i] = src->words[i];
}

/* bitwise and/or/bic of src and dst, result in dst.
   (bic = bit clear a la PDP-11, and with inverse of src). */

void bitset_and(struct bitset *dst, struct bitset *src)
{
    int i;

    for (i = 0; i < dst->nr_words; ++i)
        dst->words[i] &= src->words[i];
}

void bitset_or(struct bitset *dst, struct bitset *src)
{
    int i;

    for (i = 0; i < dst->nr_words; ++i)
        dst->words[i] |= src->words[i];
}

void bitset_bic(struct bitset *dst, struct bitset *src)
{
    int i;

    for (i = 0; i < dst->nr_words; ++i)
        dst->words[i] &= ~(src->words[i]);
}

/* returns TRUE if two bitsets are equivalent */

bool bitset_same(struct bitset *bitset1, struct bitset *bitset2)
{
    return memcmp(bitset1->words, bitset2->words,
                  bitset1->nr_words * BYTES_PER_WORD) == 0;
}

/* vi: set ts=4 expandtab: */
