/* execvpe.c - execute new binary                          ncc standard library

Copyright (c) 1977-1995 by Robert Swartz. All rights reserved.
Copyright (c) 2021 Charles E. Youse (charles@gnuless.org).

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of the copyright holder nor the names of the contributors 
  may be used to endorse or promote products derived from this software
  without specific prior written permission.

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

#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <stdlib.h>
#include <limits.h>

/* per Posix, we are supposed to invoke the shell to process scripts
   if execve() fails with ENOEXEC, but we do not support that- it is
   legacy behavior and quite limited. users should use #!/bin/sh or
   whatever for executable scripts, and we let the kernel handle it.
   we also don't embed a default path- set the PATH variable. */

#define DEFAULT_PATH ""

int execvpe(const char *name, char *const argv[], char *const envp[])
{
    char fname[PATH_MAX];
    const char *p1;
    char *p2, *p3, *sp;
    int isabs;

    if ((sp = getenv("PATH")) == 0)     /* find PATH */
        sp = DEFAULT_PATH;              /* or use an empty one */

    isabs = (strchr(name, '/') != 0);   /* if pathname is absolute */

    for (;;) {
        if (isabs)
            strcpy(fname, name);
        else {
            for (p2 = fname; *sp && (*sp != ':'); )
                *p2++ = *sp++;  /* copy element from PATH */
    
            if (p2 != fname)            /* append / if not empty */
                *p2++ = '/';    

            for (p1 = name; *p1; )      /* append name */
                *p2++ = *p1++;
            
            *p2 = 0;                    /* and NUL terminate */
        }

        execve(fname, argv, envp);

        if (isabs)                      /* absolute path, fail */
            break;

        if (*sp == 0)                   /* end of PATH, fail */
            break;

        if (*sp == ':')
            ++sp;
    }

    return -1;
}

/* vi: set ts=4 expandtab: */
