/* escape.c - convert character escapes                 ncc, the new c compiler

Copyright (c) The Regents of the University of California. All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

   1. Redistributions of source code must retain the above copyright
      notice, this list of conditions and the following disclaimer.
   2. Redistributions in binary form must reproduce the above copyright
      notice, this list of conditions and the following disclaimer in the
      documentation and/or other materials provided with the distribution.
   3. Neither the name of the University nor the names of its contributors
      may be used to endorse or promote products derived from this software
      without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE REGENTS AND CONTRIBUTORS ``AS IS'' AND ANY
EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED.  IN NO EVENT SHALL THE REGENTS OR CONTRIBUTORS BE LIABLE FOR ANY
DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE. */

#include <string.h>
#include <ctype.h>
#include <limits.h>
#include "escape.h"

/* interpret a character from a character constant or string literal. on
   success, returns the (unsigned) value of the character, and the pointer
   is advanced to the next logical character. returns -1 on failure. */

static char digits[] = "0123456789ABCDEF";

#define ISODIGIT(x)     (isdigit(x) && ((x) != '8') && ((x) != '9'))
#define DIGIT_VALUE(x)  (strchr(digits, toupper(x)) - digits)

int escape(char **buf)
{
    char *cp;
    int c;

    cp = *buf;

    if (*cp == '\\') {
        ++cp;

        if (ISODIGIT(*cp)) {
            c = DIGIT_VALUE(*cp);
            ++cp;

            if (ISODIGIT(*cp)) {
                c <<= 3;
                c += DIGIT_VALUE(*cp);
                ++cp;
                if (ISODIGIT(*cp)) {
                    c <<= 3;
                    c += DIGIT_VALUE(*cp);
                    ++cp;
                    if (c > UCHAR_MAX) return -1;
                }
            }
        } else if (*cp == 'x') {
            ++cp;
            if (!isxdigit(*cp)) return -1;
            c = 0;

            while (isxdigit(*cp)) {
                if (c & 0xF0) return -1;
                c <<= 4;
                c += DIGIT_VALUE(*cp);
                ++cp;
            }
        } else {
            switch (*cp) {
                case 'a':       c = '\a'; break;
                case 'b':       c = '\b'; break;
                case 'f':       c = '\f'; break;
                case 'n':       c = '\n'; break;
                case 'r':       c = '\r'; break;
                case 't':       c = '\t'; break;
                case 'v':       c = '\v'; break;

                case '?':
                case '"':
                case '\\':
                case '\'':      c = *cp; break;

                default:        return -1;
            }

            ++cp;
        }
    } else {
        c = *cp & 0xFF;
        ++cp;
    }

    *buf = cp;
    return c;
}

/* vi: set ts=4 expandtab: */
