/* setvbuf.c - stream buffer control                       ncc standard library

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

int setvbuf(FILE *fp, char *buf, int mode, size_t size)
{
    int retval = 0;
    
    __exit_cleanup = __stdio_cleanup;

    if (mode != _IOFBF && mode != _IOLBF && mode != _IONBF)
        return EOF;

    if (fp->_buf && (fp->_flags & _IOMYBUF))
        free(fp->_buf);

    fp->_flags &= ~(_IOMYBUF | _IONBF | _IOLBF);

    if (buf && size == 0)
        retval = EOF;

    if (!buf && (mode != _IONBF)) {
        if (size == 0 || (buf = malloc(size)) == 0)
            retval = EOF;
        else
            fp->_flags |= _IOMYBUF;
    }

    fp->_buf = (unsigned char *) buf;
    fp->_count = 0;
    fp->_flags |= mode;
    fp->_ptr = fp->_buf;

    if (!buf) {
        fp->_bufsiz = 1;
    } else {
        fp->_bufsiz = size;
    }

    return retval;
}

/* vi: set ts=4 expandtab: */
