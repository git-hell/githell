/* unistd.h - standard UNIX definitions                    ncc standard library

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

#ifndef _UNISTD_H
#define _UNISTD_H

#include <ncc.h>

#define NULL __NULL

#ifndef __SIZE_T
#define __SIZE_T
typedef __size_t size_t;
#endif /* __SIZE_T */

#ifndef __SSIZE_T
#define __SSIZE_T
typedef __ssize_t ssize_t;
#endif /* __SSIZE_T */

#ifndef __PID_T
#define __PID_T
typedef __pid_t pid_t;
#endif /* __PID_T */

#define STDIN_FILENO    0
#define STDOUT_FILENO   1
#define STDERR_FILENO   2

#define F_OK  0
#define X_OK  1
#define W_OK  2
#define R_OK  4

extern int access(const char *, int);

/* the program break is always manipulated with the __brk
   system call. the brk() and sbrk() library functions are
   wrappers that provide the posix semantics. */

extern void *__brk(void *);     /* system call */

extern int brk(void *);
extern void *sbrk(__ssize_t);

extern int close(int);
extern int execve(const char *, char *const [], char *const []);
extern int execvp(const char *, char *const []);
extern int execvpe(const char *, char *const [], char *const []);
extern pid_t fork(void);
extern pid_t getpid(void);
extern int isatty(int);

#define SEEK_SET        __SEEK_SET
#define SEEK_CUR        __SEEK_CUR
#define SEEK_END        __SEEK_END

extern __off_t lseek(int, __off_t, int);

extern __ssize_t read(int, void *, __size_t);
extern int unlink(const char *);
extern __ssize_t write(int, const void *, __size_t);

extern char *optarg;
extern int optind;
extern int opterr;
extern int optopt;

extern int getopt(int, char *const[], const char *);

#endif /* _UNISTD_H */

/* vi: set ts=4 expandtab: */
