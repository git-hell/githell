/* vfscanf.c - formatted input conversion                  ncc standard library

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

#include <stdio.h>
#include <stdarg.h>
#include <ctype.h>
#include <stdlib.h>
#include <string.h>

/* read an ASCII floating point number containing at most width characters
   from fp into buf. convert the result using strtod() and store it to *dp.
   return the number of characters read.

   use the finite state machine shown below. in the diagram,
        states are in (parens)
        inputs are in [brackets]
   for all inputs not shown, transition is to (end) state.
                                   --
                                 /    \
                                v      \
                  ----------> [0-9] -> (3)
                /               ^       | \
               /               /        |  \
              /               /         v   \
             /               /         [.]   \               ->[+-] -
            /               /           |     \             /         \
           /               /            |      \           /           \
          /               /             v       v         /             v
       (0) --> [+-] -> (1)             (4) --> [Ee] -> (6) -> [0-9] -> (7)
          \           /               /         ^               ^      /
           \         /               /         /                 \    /
            \       /               /         /                    --
             \     /               /         /
              \   /               /         /
               \ /               /         /
                v               v         /
               [.] --> (2) -> [0-9] -> (5)
                                ^      /
                                 \    /
                                   --
   this cannot be done correctly with the single character pushback
   guaranteed by ungetc(); for example, rejecting "+y", "1Ex" or "1E+x"
   requires two or three character pushback. */

static int dscan(char *buf, FILE *fp, int width, double *dp)
{
    int c, state, count;
    char *cp;

    cp = buf;

    for (state = count = 0; count < width; ++count) {
        *cp++ = c = getc(fp);
        switch (c) {
        case '+': case '-':

            if (state != 0 && state != 6)
                break;

            state++;
            continue;

        case '0': case '1': case '2': case '3': case '4':
        case '5': case '6': case '7': case '8': case '9':

            if (state == 0 || state == 1 || state == 3)
                state = 3;
            else if (state == 2 || state == 4 || state == 5)
                state = 5;
            else if (state == 6 || state == 7)
                state = 7;
            else
                break;

            continue;

        case 'E': case 'e':

            if (state < 3 || 5 < state)
                break;

            state = 6;
            continue;

        default:
            if (c != '.')
                break;

            if (state <= 1)
                state = 2;
            else if (state == 3)
                state++;
            else
                break;

            continue;
        }

        --cp;
        ungetc(c, fp);
        break;
    }

    *cp = '\0';
    *dp = strtod(buf, 0);
    return ((int)(cp - buf));
}

/* read an ASCII number in given base containing at most width characters
   from fp into buf. evaluate it using strtol()/strtoul() and store value
   through *lp. return the number of characters read.

   ANSI 4.9.6.2 does not explicitly say anything about overflow, but it
   implies that integer conversions work like strtol()/strtoul(), so this
   scans the number into a buffer and uses the function.

   the routine implements a finite state machine with 8 states:
        (0)     initial state, base == 0
        (1)     after sign, base == 0
        (2)     after '0', base == 0
        (3)     initial state, base != 0 && base != 16
        (4)     after sign, base != 0 && base != 16
        (5)     after digit, base != 0
        (6)     initial state, base == 16
        (7)     after sign, base == 16
        (8)     after '0', base == 16

   this cannot be done correctly with the single character pushback
   guaranteed by ungetc(); for example, rejecting "+y", "0xy" or "+0xy"
   requires two character pushback. */

static int lscan(char *buf, FILE *fp, int base, int width, int flag, long *lp)
{
    int c, state, count;
    char *cp;

    cp = buf;
    state = (base == 0) ? 0 : (base == 16) ? 6 : 3;

    for (count = 0; count < width; ++count) {
        *cp++ = c = getc(fp);

        switch(c) {
        case '+':
        case '-':
            if (state != 0 && state != 3 && state != 6)
                break;
            ++state;
            continue;

        case '0':
            if (state <= 1)
                state = 2;
            else if (state == 2) {
                base = 8;
                state = 5;
            } else if (state <= 5 || state == 8)
                state = 5;
            else
                state = 8;

            continue;

        case 'x':
        case 'X':
            if (state == 2) {
                base = 16;
                state = 5;
            } else if (state == 8)
                state = 5;
            else
                break;

            continue;

        default:
            if (state <= 2) {
                if (isdigit(c)) {
                    base = 10;
                    state = 5;
                } else
                    break;

                continue;
            }

            if ((isdigit(c) && c-'0' < base)
              || (islower(c) && c-'a'+10 < base)
              || (isupper(c) && c-'A'+10 < base))
                state = 5;
            else
                break;

            continue;
        }

        --cp;
        ungetc(c, fp);
        break;
    }

    *cp = '\0';

    if (flag)
        *lp = strtol(buf, 0, base);
    else
        *lp = strtoul(buf, 0, base);

    return (int)(cp - buf);
}

#define PPREC           16      /* precision for %p format */
#define MAX_WIDTH       128     /* max width of conversion item */

int vfscanf(FILE *fp, const char *format, va_list args)
{
    int fc, gc;
    char *cp;
    int base, width, count, nitems, n, sflag, lflag, flag;
    long val;
    double d;
    char buf[MAX_WIDTH];

    for (nitems = count = 0; fc = *format++; ) {
        if (isspace(fc)) { /* white space directive */
            while (isspace(gc = getc(fp)))
                ++count;

            if (gc == EOF)
                break;

            ungetc(gc, fp);
            continue;
        } else if (fc != '%') {
matchin:
            if ((gc=getc(fp)) != fc) {
                ungetc(gc, fp);
                break;
            }

            count++;
            continue;
        } else { 
            /* process *, field width, hlL flags */

            flag = sflag = lflag = 0;

            if ((fc = *format++) == '*') {
                ++sflag;
                fc = *format++;
            }

            if (isdigit(fc))
                for (width = 0; isdigit(fc); fc = *format++)
                    width = width*10 + fc - '0';
            else
                width = -1;

            if (fc == 'h' || fc == 'l' || fc == 'L') {
                lflag = fc;
                fc = *format++;
            }

            /* skip leading white space */

            if (fc != '[' && fc != 'c' && fc != 'n') {
                while (isspace(gc = getc(fp)))
                    ++count;

                ungetc(gc, fp);
            }

            /* perform a conversion */
            /* three generic cases: fixed, float, string */

            switch (fc) {
            default:
            case '\0':
                break;

            case 'd':
                base = 10;
                ++flag;                 /* signed */
                goto fixed;

            case 'i':
                base = 0;
                ++flag;                 /* signed */
                goto fixed;

            case 'o':
                base = 8;
                goto fixed;

            case 'u':
                base = 10;
                goto fixed;

            case 'X':
            case 'x':
                base = 16;
fixed:
                if (width == -1 || width > MAX_WIDTH)
                    width = MAX_WIDTH;

                if ((n = lscan(buf, fp, base, width, flag, &val)) == 0)
                    break;

                count += n;

                if (sflag)
                    continue;

                if (lflag == 'p')
                    *(va_arg(args, void **)) = (void *)val;
                else if (lflag == 'l')
                    *(va_arg(args, long *)) = val;
                else if (lflag == 'h')
                    *(va_arg(args, short *)) = (short)val;
                else
                    *(va_arg(args, int *)) = (int)val;

                nitems++;
                continue;

            case 'e':
            case 'f':
            case 'g':
            case 'E':
            case 'G':
                if (width == -1 || width > MAX_WIDTH)
                    width = MAX_WIDTH;

                if ((n = dscan(buf, fp, width, &d)) == 0)
                    break;

                count += n;

                if (sflag)
                    continue;

                if (lflag == 'l' || lflag == 'L')
                    *(va_arg(args, double *)) = d;
                else
                    *(va_arg(args, float *)) = d;

                nitems++;
                continue;

            case 's':
scanin:
                if (!sflag)
                    cp = va_arg(args, char *);

                for (n = 0; width < 0 || n < width; n++) {
                    if ((gc = getc(fp)) == EOF)
                        break;

                    if ((fc == 's' && isspace(gc))
                      || (fc == '['
                      && flag != (strchr(buf, gc)==0))) {
                        ungetc(gc, fp);
                        break;
                    }

                    if (!sflag)
                        *cp++ = gc;
                }

                if (n == 0)
                    break;

                count += n;

                if (sflag)
                    continue;

                if (fc != 'c')
                    *cp = '\0';

                nitems++;
                continue;

            case '[':
                /* set flag iff ^ specified */

                flag = 0;

                if ((fc = *format++) == '^') {
                    ++flag;
                    fc = *format++;
                }

                /* copy scanlist into buf */

                cp = buf;

                if (fc == ']') {
                    *cp++ = fc;
                    fc = *format++;
                }

                while (fc != '\0' && fc != ']') {
                    /* accept character ranges */

                    if (fc == '-' && cp != buf && *format != ']') {
                        gc = *(cp-1);   /* start */
                        fc = *format++; /* end */

                        if (gc > fc)
                            --cp;   /* empty */
                        else while (++gc <= fc)
                            *cp++ = gc;
                    } else
                        *cp++ = fc;

                    fc = *format++;
                }

                *cp = '\0';
                fc = '[';
                goto scanin;

            case 'c':
                if (width == -1)
                    width = 1;

                goto scanin;

            case 'p':
                width = PPREC + 2;      /* "0x"+PPREC hex digits */
                base = 16;
                lflag = 'p';
                goto fixed;

            case 'n':
                if (lflag == 'l')
                    *(va_arg(args, long *)) = (long)count;
                else if (lflag == 'h')
                    *(va_arg(args, short *)) = (short)count;
                else
                    *(va_arg(args, int *)) = count;

                continue;

            case '%':
                goto matchin;
            }

            break;
        }
        break;
    }

    return (nitems==0 && feof(fp)) ? EOF : nitems;
}

/* vi: set ts=4 expandtab: */
