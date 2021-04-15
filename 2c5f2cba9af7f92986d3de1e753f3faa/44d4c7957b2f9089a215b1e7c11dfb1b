/* vstring.c - variable-length strings                  ncc, the new c compiler

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

#include <stdlib.h>
#include <string.h>
#include "cpp.h"
#include "vstring.h"

/* empty a vstring by truncating its length to zero. note
   that we deliberately do not release external storage if
   it was allocated- that's what vstring_free() is for. */

void vstring_clear(struct vstring *vs)
{
    if (vs->u.in.flag) {
        vs->u.in.len = 0;
        vs->u.in.buf[0] = 0;
    } else {
        vs->u.out.len = 0;
        vs->u.out.buf[0] = 0;
    }
}

/* initialize a vstring; must be called before the first use
   (unless otherwise initialized, e.g., VSTRING_INITIALIZER). */

void vstring_init(struct vstring *vs)
{
    vs->u.in.flag = 1;
    vs->u.in.len = 0;
    vs->u.in.buf[0] = 0;
}

/* empty a vstring, releasing any allocated storage. the
   state is invalidated, so vstring_init() must be called
   if the string is to be used again. */

void vstring_free(struct vstring *vs)
{
    if (vs->u.in.flag == 0)
        free(vs->u.out.buf);
}

/* remove the last character from the vstring, or
   whoopsie if there is no character to remove. */

void vstring_rubout(struct vstring *vs)
{
    if (VSTRING_LEN(*vs) == 0)
        error("CPP INTERNAL: vstring underflow");

    if (vs->u.in.flag)
        --(vs->u.in.len);
    else
        --(vs->u.out.len);
}

/* return the last character of the vstring,
   or NUL if the string is empty */

char vstring_last(struct vstring *vs)
{
    if (VSTRING_LEN(*vs) == 0) return 0;

    if (vs->u.in.flag)
        return vs->u.in.buf[vs->u.in.len - 1];
    else
        return vs->u.out.buf[vs->u.out.len - 1];
}

/* append len bytes from buf to the vstring, growing
   the string if the capacity is insufficient.
   
   we are a bit repetitive here rather than relying on
   the VSTRING macros, to avoid excessive branching. */

void vstring_put(struct vstring *vs, char *buf, size_t len)
{
    size_t min_cap;
    size_t new_cap;
    char *new_buf;

    min_cap = VSTRING_LEN(*vs);
    min_cap += len + 1;
    if (min_cap < len) error("CPP INTERNAL: vstring overflow");
    
    if (min_cap > VSTRING_CAP(*vs)) {
        new_cap = VSTRING_MIN_ALLOC;
        while (new_cap && (new_cap < min_cap)) new_cap <<= 1;
        if (new_cap == 0) error("CPP INTERNAL: vstring overflow");
        new_buf = safe_malloc(new_cap);

        if (vs->u.in.flag) {
            memcpy(new_buf, vs->u.in.buf, vs->u.in.len);
            vs->u.out.len = vs->u.in.len;
        } else {
            memcpy(new_buf, vs->u.out.buf, vs->u.out.len);
            free(vs->u.out.buf);
        }
        
        vs->u.out.cap = new_cap;
        vs->u.out.buf = new_buf;
    }

    if (vs->u.in.flag) {
        memcpy(vs->u.in.buf + vs->u.in.len, buf, len);
        vs->u.in.len += len;
        vs->u.in.buf[vs->u.in.len] = 0;
    } else {
        memcpy(vs->u.out.buf + vs->u.out.len, buf, len);
        vs->u.out.len += len;
        vs->u.out.buf[vs->u.out.len] = 0;
    }
}

/* append one character to the end of the vstring.
   we attempt to optimize the common easy case,
   but hand off anything harder to vstring_put(). */

void vstring_putc(struct vstring *vs, char c)
{
    if (VSTRING_LEN(*vs) <= (VSTRING_CAP(*vs) - 2)) {
        /* in other words, there's room for another
           character plus the NUL terminator. */

        if (vs->u.in.flag) {
            vs->u.in.buf[vs->u.in.len++] = c;
            vs->u.in.buf[vs->u.in.len] = 0;
        } else {
            vs->u.out.buf[vs->u.out.len++] = c;
            vs->u.out.buf[vs->u.out.len] = 0;
        }
    } else
        vstring_put(vs, &c, 1);
}

/* copy the contents of src to the end of dst */

void vstring_concat(struct vstring *dst, struct vstring *src)
{
    vstring_put(dst, VSTRING_BUF(*src), VSTRING_LEN(*src));
}

/* copy the contents of c string to end of dst */

void vstring_puts(struct vstring *dst, char *s)
{
    vstring_put(dst, s, strlen(s));
}

/* returns true if two vstrings have the same contents */

int vstring_same(struct vstring *vs1, struct vstring *vs2)
{
    if ((VSTRING_LEN(*vs1) == VSTRING_LEN(*vs2))
      && !memcmp(VSTRING_BUF(*vs1), VSTRING_BUF(*vs2), VSTRING_LEN(*vs1)))
        return 1;
    else
        return 0;
}

/* vi: set ts=4 expandtab: */
