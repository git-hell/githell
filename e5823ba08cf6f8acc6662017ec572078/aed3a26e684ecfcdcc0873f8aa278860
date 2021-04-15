/* strerror.c - map error numbers to strings               ncc standard library

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

#include <string.h>

static const char * const errors[] =
{
    /*  0 */    "no error",
    /*  1 */    0,
    /*  2 */    0,
    /*  3 */    0,
    /*  4 */    0,
    /*  5 */    0,
    /*  6 */    0,
    /*  7 */    0,
    /*  8 */    0,
    /*  9 */    0,
    /* 10 */    0,
    /* 11 */    0,
    /* 12 */    "out of memory",                            /* ENOMEM */
    /* 13 */    0,
    /* 14 */    0,
    /* 15 */    0,
    /* 16 */    0,
    /* 17 */    0,
    /* 18 */    0,
    /* 19 */    0,
    /* 20 */    0,
    /* 21 */    0,
    /* 22 */    0,
    /* 23 */    0,
    /* 24 */    0,
    /* 25 */    "not a typewriter",                         /* ENOTTY */
    /* 26 */    0,
    /* 27 */    0,
    /* 28 */    0,
    /* 29 */    0,
    /* 30 */    0,
    /* 31 */    0,
    /* 32 */    0,
    /* 33 */    "math argument out of domain of func",      /* EDOM */
    /* 34 */    "math result not representable"             /* ERANGE */
};

#define NR_ERRORS (sizeof(errors) / sizeof(*errors))

char *strerror(int errno)
{
    if ((errno < 0) || (errno >= NR_ERRORS) || (errors[errno] == 0))
        return "unknown error";
    else
        return (char *) errors[errno];
}

/* vi: set ts=4 expandtab: */
