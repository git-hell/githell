/* vstring.h - variable-length strings                  ncc, the new c compiler

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

#ifndef VSTRING_H
#define VSTRING_H

#include <stddef.h>

/* we use so-called small-string optimization in representation of strings:
   a string that is small enough is kept in the struct itself to avoid the
   overhead of allocating an external buffer. this is not very portable:
   in particular, vstring_in.flag must overlap the lsb of vstring_out.cap. */

struct vstring_out
{
    size_t cap;
    size_t len;
    char *buf;
};

#define VSTRING_IN_CAP (sizeof(struct vstring_out) - 1)

struct vstring_in
{
    int flag : 1;
    int len : 7;
    char buf[VSTRING_IN_CAP];
};

struct vstring
{
    union
    {
        struct vstring_in in;
        struct vstring_out out;
    } u;
};

#define VSTRING_MIN_ALLOC (1 << 5)      /* power of 2, please */

#define VSTRING_INITIALIZER { 1 }       /* flag = 1, len/buf 0s */

/* buffers are always NUL terminated for convenience, so VSTRING_BUF()
   can be used where standard c strings are called for. */

#define VSTRING_BUF(vs) (((vs).u.in.flag) ? ((vs).u.in.buf) : ((vs).u.out.buf))
#define VSTRING_LEN(vs) (((vs).u.in.flag) ? ((vs).u.in.len) : ((vs).u.out.len))
#define VSTRING_CAP(vs) (((vs).u.in.flag) ? VSTRING_IN_CAP : ((vs).u.out.cap))

extern void vstring_clear(struct vstring *);
extern void vstring_rubout(struct vstring *);
extern void vstring_put(struct vstring *, char *, size_t);
extern void vstring_putc(struct vstring *, char);
extern void vstring_puts(struct vstring *, char *);
extern void vstring_free(struct vstring *);
extern void vstring_init(struct vstring *);
extern void vstring_concat(struct vstring *, struct vstring *);
extern char vstring_last(struct vstring *);
extern int vstring_same(struct vstring *, struct vstring *);

#endif /* VSTRING_H */

/* vi: set ts=4 expandtab: */
