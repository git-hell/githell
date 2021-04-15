/* fseek.c - move stream position                          ncc standard library

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
#include <unistd.h>

int fseek(FILE *fp, long offset, int whence)
{
    int adjust = 0;
    long pos;

    fp->_flags &= ~(_IOEOF | _IOERR);

    if (fp->_flags & _IOREADING) {
        if (whence == SEEK_CUR && fp->_buf && !(fp->_flags & _IONBF))
            adjust = fp->_count;

        fp->_count = 0;
    } else if (fp->_flags & _IOWRITING) {
        fflush(fp);
    } else /* neither reading nor writing. The buffer must be empty */
        /* EMPTY */ ;

    pos = lseek(fileno(fp), offset - adjust, whence);

    if ((fp->_flags & _IOREAD) && (fp->_flags &  _IOWRITE))
        fp->_flags &= ~(_IOREADING | _IOWRITING);

    fp->_ptr = fp->_buf;

    return ((pos == -1) ? -1 : 0);
}

/* vi: set ts=4 expandtab: */
