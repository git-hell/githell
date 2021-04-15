/* macro.h - preprocessor macro table                   ncc, the new c compiler

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

#ifndef MACRO_H
#define MACRO_H

#include "token.h"
#include "vstring.h"

typedef int macro_flags;    /* MACRO_* */

#define MACRO_HAS_ARGS          0x80000000
#define MACRO_PREDEF_STDC       0x00000020
#define MACRO_PREDEF_LINE       0x00000010
#define MACRO_PREDEF_FILE       0x00000008
#define MACRO_PREDEF_DATE       0x00000004
#define MACRO_PREDEF_TIME       0x00000002
#define MACRO_PREDEF_DEFINED    0x00000001
#define MACRO_PREDEF_MASK       0x0000003F

struct macro
{
    struct vstring name;
    macro_flags flags;
    struct list formals;        /* formal argument names (TOKEN_IDENTs) */
    struct list tokens;         /* replacement token sequence */
    TAILQ_ENTRY(macro) links;
};

extern void macro_predef(void);
extern struct macro *macro_lookup(struct vstring *);
extern void macro_define(struct list *);
extern int macro_cmdline(char *);
extern void macro_undef(struct list *);
extern int macro_replace(struct list *);
extern void macro_replace_all(struct list *);

/* we have to exclude 'defined' because it's not
   technically defined. there's an irony here.. */

#define MACRO_DEFINED(m)    (!((m)->flags & MACRO_PREDEF_DEFINED))

#endif /* MACRO_H */

/* vi: set ts=4 expandtab: */
