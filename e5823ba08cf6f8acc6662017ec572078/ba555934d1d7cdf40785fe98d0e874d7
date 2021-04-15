/* vfprintf.c - formatted output                           ncc standard library

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
#include <stdlib.h>
#include <stdarg.h>
#include <limits.h>
#include <math.h>

/* ANSI says any conversion item can be up to 509 characters. the
   only case where cbuf needs to be big is for floating point (%f). */

#define CBUFMAX     512         /* conversion buffer size */
#define NDIGITS     23          /* max digits in octal long + NUL */
#define PRINTNULL   "{NULL}"    /* for %s with NULL arg */
#define PPREC       16          /* precision for %p format */

static const char digits[] = {
    '0', '1', '2', '3', '4', '5', '6', '7',
    '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'
};

static const char ldigits[] = {
    '0', '1', '2', '3', '4', '5', '6', '7',
    '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'
};

/* convert unsigned long n to ascii in base b for conversion c.
   store the result through cp and return a pointer past the end. */

static char *convert(char *cp, unsigned long n, unsigned b, int c)
{
    char *ep;
    const char *dp;
    char pbuf[NDIGITS];

    dp = (c == 'x') ? ldigits : digits;
    ep = &pbuf[NDIGITS-1];
    *ep = '\0';

    do {
        *--ep = dp[n%b];
    } while ((n /= b) != 0);

    while (*ep)
        *cp++ = *ep++;

    return cp;
}

int vfprintf(FILE *fp, const char *format, va_list args)
{
    char cbuf[CBUFMAX];
    char *cbp;
    char *cbs;
    char *s;
    int count, c;
    long l;
    int fwidth, prec, base, len;
    int leftjustify, plusflag, spaceflag, altflag, longflag;
    int padchar, padwidth, issigned, prefix, ispfx, nzeros;
    double d;

    count = 0;

    for (;;) {

        /* nonconversion characters */

        while ((c = *format++) != '%') {
            if (c == '\0')
                return count;

            ++count;
            putc(c, fp);
        }

        /* optional flags "-+ #0" */

        leftjustify = plusflag = spaceflag = altflag = 0;
        padchar = ' ';

        for (;;) {
            switch(c = *format++) {
            case '-':   ++leftjustify; continue;
            case '+':   ++plusflag; continue;
            case ' ':   ++spaceflag; continue;
            case '#':   ++altflag; continue;
            case '0':   padchar = '0'; continue;
            default:    break;
            }
            break;
        }

        /* optional field width ('*' or decimal integer) */

        if (c == '*') {
            if ((fwidth = va_arg(args, int)) < 0) {
                leftjustify = 1;
                fwidth = -fwidth;
            }
            c = *format++;
        } else
            for (fwidth = 0; c >= '0' && c <= '9'; c = *format++)
                fwidth = fwidth*10 + c-'0';

        /* optional precision ('.' followed by '*' or decimal integer) */

        if (c == '.') {
            c = *format++;
            if (c == '*') {
                prec = va_arg(args, int);

                if (prec < 0)
                    prec = -1;

                c = *format++;
            } else
                for (prec=0; c>='0' && c<='9'; c = *format++)
                    prec = prec*10 + c-'0';
        } else
            prec = -1;

        /* optional 'h', 'l', 'L' or 'z' flag */

        if (c == 'l' || c == 'h' || c == 'L' || c == 'z') {
            if (c == 'z')
                c = (sizeof(size_t) == 8) ? 'l' : 0;
            else
                longflag = c;

            c = *format++;
        } else
            longflag = 0;

        /* convert an item */

        cbp = cbs = cbuf;
        issigned = nzeros = prefix = ispfx = 0;

        switch (c) {
        case 'd':
        case 'i':
            base = 10;

            if (longflag=='l')
                l = va_arg(args, long);
            else
                l = (long) va_arg(args, int);

            if (longflag == 'h')
                l = (short) l;

            if (l < 0L) {
                l = -l;
                --issigned;     /* -1 for negative */
            } else
                ++issigned;     /* +1 for positive */

            goto conv;

        case 'o':
            base = 8;
            goto unsconv;

        case 'u':
            base = 10;
            goto unsconv;
    
        case 'x':
        case 'X':
            base = 16;

unsconv:
            if (longflag=='l')
                l = va_arg(args, long);
            else
                l = (unsigned long) va_arg(args, int);

            if (longflag == 'h')
                l = (unsigned short) l;

            if (altflag && ((l != 0L && base == 8) || base == 16))
                prefix = c;     /* 'o', 'x' or 'X' */

conv:
            if (prec == 0 && l == 0L)
                break;  /* ANSI says so */

            if (prec != -1)
                padchar = ' ';  /* ignore 0 flag */

            cbp = convert(cbp, l, base, c);

            if ((nzeros = prec - (cbp - cbs)) < 0)  /* number of leading 0s */
                nzeros = 0;

            break;

        case 'f':
        case 'e':
        case 'E':
        case 'g':
        case 'G':
            d = va_arg(args, double);
            cbp = __dtefg(cbp, &d, c, prec, altflag, &issigned);
            break;

        case 'c':
            *cbp++ = (unsigned char) va_arg(args, int);
            break;
    
        case 's':
            if ((s = va_arg(args, char *)) == NULL)
                s = PRINTNULL;  /* not strictly ANSI */

            cbp = cbs = s;

            while (*cbp++ != '\0')
                if ((prec >= 0) && ((cbp - s) > prec))
                    break;

            cbp--;
            break;

        case 'p':
            longflag = 'l';
            prec = PPREC;
            ++altflag;
            c = 'X';
            base = 16;
            goto unsconv;

        case 'n':
            if (longflag == 'h')
                *(va_arg(args, short *)) = (short)count;
            else if (longflag == 'l')
                *(va_arg(args, long *)) = (long)count;
            else
                *(va_arg(args, int *)) = count;
            break;

        default:
            ++count;
            putc(c, fp);
            continue;
        }

        /* output an item */

        len = cbp - cbs;        /* length of converted item */

        if (issigned && (issigned == -1 || plusflag || spaceflag)) {
            ++len;          /* for '-', '+', ' ' */
            ++ispfx;
        }

        if (prefix) {
            ++len;          /* for '0' */
            ++ispfx;

            if (prefix != 'o')
                ++len;      /* for 'x', 'X' */
        }

        if ((padwidth = fwidth - nzeros - len) < 0)
            padwidth = 0;       /* length of padding required */

        count += len + padwidth + nzeros;

        if (!leftjustify && padwidth > 0) {
            if (ispfx && padchar == '0')
                nzeros += padwidth; /* prefix before 0-padding */
            else
                while (padwidth-- > 0)  /* pad on the left */
                    putc(padchar, fp);
        }

        if (issigned) {
            if (issigned == -1)
                putc('-', fp);
            else if (plusflag)
                putc('+', fp);
            else if (spaceflag)
                putc(' ', fp);
        }

        if (prefix) {
            putc('0', fp);

            if (prefix != 'o')
                putc(prefix, fp);
        }

        while (nzeros-- > 0)
            putc('0', fp);      /* leading 0s */

        len = cbp - cbs;

        while (len-- > 0)
            putc(*cbs++, fp);   /* converted item */

        if (leftjustify)
            while (padwidth-- > 0)
                putc(' ', fp);  /* pad on the right */
    }
}

/* vi: set ts=4 expandtab: */
