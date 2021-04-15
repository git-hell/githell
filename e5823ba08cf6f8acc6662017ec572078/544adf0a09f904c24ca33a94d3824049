/* __fillbuf.c - fill a buffer                             ncc standard library

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
#include <unistd.h>

int __fillbuf(FILE *fp)
{
    static unsigned char ch[FOPEN_MAX];   /* "buffers" for the unbuffered */
    int i;
    
    fp->_count = 0;

    if (fp->_fd < 0)
        return EOF;

    if (fp->_flags & (_IOEOF | _IOERR))
        return EOF;

    if (!(fp->_flags & _IOREAD)) {
        fp->_flags |= _IOERR;
        return EOF;
    }

    if (fp->_flags & _IOWRITING) {
        fp->_flags |= _IOERR;
        return EOF;
    }

    if (!(fp->_flags & _IOREADING))
        fp->_flags |= _IOREADING;

    if (!(fp->_flags & _IONBF) && !fp->_buf) {
        fp->_buf = malloc(BUFSIZ);

        if (!fp->_buf)
            fp->_flags |= _IONBF;
        else {
            fp->_flags |= _IOMYBUF;
            fp->_bufsiz = BUFSIZ;
        }
    }

    /* flush line-buffered output when filling an input buffer */

    for (i = 0; i < FOPEN_MAX; i++) {
        if (__iotab[i] && (__iotab[i]->_flags & _IOLBF))
            if (__iotab[i]->_flags & _IOWRITING)
                fflush(__iotab[i]);
    }

    if (!fp->_buf) {
        fp->_buf = &ch[fp->_fd];
        fp->_bufsiz = 1;
    }

    fp->_ptr = fp->_buf;
    fp->_count = read(fp->_fd, fp->_buf, fp->_bufsiz);

    if (fp->_count <= 0) {
        if (fp->_count == 0)
            fp->_flags |= _IOEOF;
        else
            fp->_flags |= _IOERR;

        return EOF;
    }

    fp->_count--;

    return *fp->_ptr++;
}

/* vi: set ts=4 expandtab: */
