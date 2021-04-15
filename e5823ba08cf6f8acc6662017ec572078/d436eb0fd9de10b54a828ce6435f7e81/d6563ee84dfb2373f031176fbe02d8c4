/* stdio.h - standard i/o library                          ncc standard library

Copyright (c) 1987, 1997, 2001 Prentice Hall. All rights reserved.
Copyright (c) 1987 Vrije Universiteit, Amsterdam, The Netherlands.
Copyright (c) 2021 Charles E. Youse (charles@gnuless.org).

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of Prentice Hall nor the names of the software authors or
  contributors may be used to endorse or promote products derived from this
  software without specific prior written permission.

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

#ifndef _STDIO_H
#define _STDIO_H

#include <ncc.h>

#define NULL __NULL

typedef __off_t fpos_t;

#ifndef __SIZE_T
#define __SIZE_T
typedef __size_t size_t;
#endif /* __SIZE_T */

#define SEEK_SET        __SEEK_SET
#define SEEK_CUR        __SEEK_CUR
#define SEEK_END        __SEEK_END

/* focus point of all stdio activity */

typedef struct __iobuf
{
    int _count;
    int _fd;
    int _flags;
    int _bufsiz;
    unsigned char *_buf;
    unsigned char *_ptr;
} FILE;

/* __iobuf._flags */

#define _IOFBF          0x000           /* fully buffered */
#define _IOREAD         0x001           /* open for reading */
#define _IOWRITE        0x002           /* open for writing */
#define _IONBF          0x004           /* unbuffered */
#define _IOMYBUF        0x008           /* stdio owns the buffer */
#define _IOEOF          0x010           /* at end of file */
#define _IOERR          0x020           /* error encountered */
#define _IOLBF          0x040           /* line buffered */
#define _IOREADING      0x080           /* currently buffering input */
#define _IOWRITING      0x100           /* currently buffering output */
#define _IOAPPEND       0x200           /* open in append mode */

#define FOPEN_MAX       20

extern FILE *__iotab[FOPEN_MAX];
extern FILE __stdin, __stdout, __stderr;

#define stdin           (&__stdin)
#define stdout          (&__stdout)
#define stderr          (&__stderr)

#define EOF             (-1)
#define BUFSIZ          1024

extern int __fillbuf(FILE *);
extern int __flushbuf(int, FILE *);

extern void clearerr(FILE *);
extern int fclose(FILE *);
extern int fflush(FILE *);
extern int fileno(FILE *);
extern char *fgets(char *, int n, FILE *);
extern int fgetpos(FILE *, fpos_t *);
extern int fsetpos(FILE *, fpos_t *);
extern int fprintf(FILE *, const char *, ...);
extern int fputc(int, FILE *);
extern int fputs(const char *, FILE *);
extern FILE *fopen(const char *, const char *);
extern size_t fread(void *, size_t, size_t, FILE *);
extern int fscanf(FILE *, const char *, ...);
extern int fseek(FILE *, long, int);
extern long ftell(FILE *);
extern size_t fwrite(const void *, size_t, size_t, FILE *);
extern char *gets(char *);
extern int printf(const char *, ...);
extern int puts(const char *);
extern int remove(const char *);
extern void rewind(FILE *);
extern int rename(const char *, const char *);
extern int scanf(const char *, ...);
extern int sscanf(const char *, const char *, ...);
extern void setbuf(FILE *, char *);
extern int setvbuf(FILE *, char *, int, size_t);
extern int sprintf(char *, const char *, ...);
extern int ungetc(int, FILE *);
extern int vfprintf(FILE *, const char *, __va_list);
extern int vfscanf(FILE *, const char *, __va_list);
extern int vsprintf(char *, const char *, __va_list);

#define fileno(fp)      ((fp)->_fd)
#define getchar()       getc(stdin)
#define putchar(c)      putc(c,stdout)
#define feof(p)         (((p)->_flags & _IOEOF) != 0)
#define ferror(p)       (((p)->_flags & _IOERR) != 0)
#define clearerr(p)     ((p)->_flags &= ~(_IOERR|_IOEOF))

#define getc(p)         (--(p)->_count >= 0 ? (int) (*(p)->_ptr++) :        \
                                __fillbuf(p))

#define putc(c, p)      (--(p)->_count >= 0 ?                               \
                         (int) (*(p)->_ptr++ = (c)) :                       \
                         __flushbuf((c),(p)))

#endif /* _STDIO_H */

/* vi: set ts=4 expandtab: */
