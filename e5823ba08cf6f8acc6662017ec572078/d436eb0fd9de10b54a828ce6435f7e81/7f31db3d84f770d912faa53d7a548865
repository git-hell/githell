/* ncc.h - library internal definitions                    ncc standard library

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

#ifndef _NCC_H
#define _NCC_H

#define __NULL      ((void *) 0)

typedef int __mode_t;
typedef long __off_t;
typedef int __pid_t;
typedef unsigned long __size_t;
typedef long __ssize_t;
typedef char *__va_list;

#define __SEEK_SET  0
#define __SEEK_CUR  1
#define __SEEK_END  2

#define __WEXITSTATUS(status)   (((status) & 0xff00) >> 8)
#define __WIFEXITED(status)     (__WTERMSIG(status) == 0)
#define __WIFSTOPPED(status)    (((status) & 0xff) == 0x7f)
#define __WIFSIGNALED(status)   (((signed char) (((status)&0x7f)+1)>>1)>0)
#define __WSTOPSIG(status)      __WEXITSTATUS(status)
#define __WTERMSIG(status)      ((status) & 0x7f)

#endif /* _NCC_H */

/* vi: set ts=4 expandtab: */
