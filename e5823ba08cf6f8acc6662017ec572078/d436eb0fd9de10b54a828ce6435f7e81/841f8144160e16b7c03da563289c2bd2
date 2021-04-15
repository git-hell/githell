/* stdarg.h - variadic arguments                           ncc standard library

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

#ifndef _STDARG_H
#define _STDARG_H

#include <ncc.h>

#define __VA_ALIGNMENT  8

#define __VA_PAD(x)     (((x) + (__VA_ALIGNMENT - 1)) & ~(__VA_ALIGNMENT - 1))

#ifndef __VA_LIST
#define __VA_LIST
typedef __va_list va_list;
#endif /* __VA_LIST */

#define va_start(ap, last)          (ap = (((char *) &(last))               \
                                     + __VA_PAD(sizeof(last))))

#define va_arg(ap, type)        ((ap += __VA_PAD(sizeof(type))),            \
                                 *((type *) (ap - __VA_PAD(sizeof(type)))))

#define va_end(ap)

#endif /* _STDARG_H */

/* vi: set ts=4 expandtab: */
