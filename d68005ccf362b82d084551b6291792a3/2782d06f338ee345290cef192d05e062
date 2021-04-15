/* string.h - immutable string table                    ncc, the new c compiler

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

#ifndef STRING_H
#define STRING_H

#include <stdio.h>
#include <stddef.h>
#include "../common/tailq.h"
#include "cc1.h"
#include "lex.h"

struct symbol;

/* all lexical strings encountered (identifiers and string literals)
   are kept in a hash table for the lifetime of the translation. this
   simplifies memory management and speeds comparisons.

   the buckets are kept in LRU order to mitigate the overhead associated
   with passing every identifier through the string table.

   if label is non-zero, then this is a string literal that must be
   emitted at the end of compilation - see string_emit_literals().

   if k is not K_IDENT, it's a keyword with that token class.

   s is guaranteed to be NUL-terminated, so can be used as a C string.
   just keep in mind that string literals may have embedded NULs. */

typedef unsigned string_hash;

struct string
{
    string_hash hash;
    size_t len;
    token_class k;
    asm_label label;
    TAILQ_ENTRY(string) links;
    
    char s[]; /* C11 */
};

extern void string_init(void);
extern struct string *string_new(char *, size_t);
extern void string_emit(struct string *, size_t);
extern void string_emit_literals(void);
extern void string_print_k(FILE *, token_class);
extern struct symbol *string_symbol(struct string *);

#endif /* STRING_H */

/* vi: set ts=4 expandtab: */
