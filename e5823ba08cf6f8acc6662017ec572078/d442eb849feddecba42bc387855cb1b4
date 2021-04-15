/* fopen.c - open stream                                   ncc standard library

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

#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <fcntl.h>
#include <unistd.h>

#define PMODE 0666      /* default mode for file creation */

/* since the O_CREAT flag is not available on all systems, we can't get it
   from the standard library. furthermore, even if we know that <fcntl.h>
   contains such a flag, it's not sure whether it can be used, since we
   might be cross-compiling for another system, which may use an entirely
   different value for O_CREAT (or not support such a mode). the safest
   thing is to just use the v7 semantics for open, and use creat() where
   necessary. another problem is O_APPEND, for which the same holds. when
   "a" open-mode is used, we lseek() to the end before every write().

   NB the above is out of date. should rewrite to use O_CREAT et al. - cey */

FILE *fopen(const char *name, const char *mode)
{
    int i;
    int rwmode = 0;
    int rwflags = 0;
    FILE *fp;
    int fd;
    int flags = 0;

    for (i = 0; __iotab[i] != 0; i++)
        if (i >= (FOPEN_MAX - 1))
            return 0;

    switch (*mode++)
    {
    case 'r':
        flags |= _IOREAD | _IOREADING;
        rwmode = O_RDONLY;
        break;
    case 'w':
        flags |= _IOWRITE | _IOWRITING;
        rwmode = O_WRONLY;
        rwflags = O_CREAT | O_TRUNC;
        break;
    case 'a':
        flags |= _IOWRITE | _IOWRITING | _IOAPPEND;
        rwmode = O_WRONLY;
        rwflags |= O_APPEND | O_CREAT;
        break;
    default:
        return 0;
    }

    while (*mode) {
        switch(*mode++) {
        case 'b':
            continue;
        case '+':
            rwmode = O_RDWR;
            flags |= _IOREAD | _IOWRITE;
            continue;
        /* The sequence may be followed by additional characters */
        default:
            break;
        }
        break;
    }

    /* perform a creat() when the file should be truncated or when
       the file is opened for writing and the open() fails. */

    if ((rwflags & O_TRUNC)
      || (((fd = open(name, rwmode)) < 0)
      && (rwflags & O_CREAT))) {
        if (((fd = creat(name, PMODE)) > 0) && flags  | _IOREAD) {
            close(fd);
            fd = open(name, rwmode);
        }
    }

    if (fd < 0)
        return 0;

    if ((fp = malloc(sizeof(FILE))) == 0) {
        close(fd);
        return 0;
    }

    if ((flags & (_IOREAD | _IOWRITE)) == (_IOREAD | _IOWRITE))
        flags &= ~(_IOREADING | _IOWRITING);

    fp->_count = 0;
    fp->_fd = fd;
    fp->_flags = flags;
    fp->_buf = 0;
    __iotab[i] = fp;
    return fp;
}

/* vi: set ts=4 expandtab: */
