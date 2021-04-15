/* __dtefg.c - convert double to ascii                     ncc standard library

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
#include <float.h>
#include <math.h>

#define LOG10B2  0.33219280948873623e+01

/* copy ASCII number from cp to buf in %f format with
   precision prec and decimal exponent decexp. the fmt
   determines whether trailing zeros are suppressed.

   returns a pointer past the last character. */

static char *dtof(char *buf, char *cp, int prec,
                  int decexp, int fmt, int aflag)
{
    if (decexp < 0)
        *buf++ = '0';                       /* units digit '0' */
    else
        do
            *buf++ = *cp ? *cp++ : '0';     /* integral part */
        while (decexp--);

    if (!aflag && (prec == 0 || ((fmt=='g'|| fmt=='G') && *cp == '\0')))
        return buf;

    *buf++ = '.';

    while (prec-- > 0) {
        if ((fmt=='g' || fmt=='G') && *cp == '\0' && !aflag)
            break;  /* suppress trailing zeros */

        if (++decexp < 0)
            *buf++ = '0';                   /* put leading zero */
        else
            *buf++ = *cp ? *cp++ : '0';
    }

    return buf;
}

/* convert the mantissa of nonnegative double d to a
   string of ASCII digits in the supplied buffer buf.
   store the decimal exponent indirectly through decexpp.

   the first digit of the mantissa is implicitly followed by '.'
   the result has no leading zeros (unless d=0.0) and no trailing zeros.
   the precision prec and format fmt determine the digit count.
   the maximum length of the result is DBL_DIG + 1 (for the NUL).

   for example, if *dp == 123.456789 and prec == 3:

        fmt=='e'        "1234"          decexp==2       1.234e+02
        fmt=='f'        "123456"        decexp==2       123.456
        fmt=='g'        "123"           decexp==2       123 */

static void dtoa(int fmt, double *dp, int prec, int *decexpp, char *buf)
{
    char *cp;
    int digit;
    int decexp;
    int ndigits;
    int binexp;
    double d;
    double dexp;

    /* handle 0.0 as a special case */

    cp = buf;
    if ((d = *dp) == 0.0) {
ret0:
        *decexpp = 0;
        *cp++ = '0';
        *cp ='\0';
        return;
    }

    /* reduce d to range [1., 10) and set decexp accordingly.
       approximate the decimal exponent from the binary exponent.
       obscure but it makes floating output much more efficient. */

    frexp(d, &binexp);                      /* find binary exponent */

    if (modf((--binexp)/LOG10B2, &dexp) < 0.0)
        dexp -= 1.0;                        /* scale, take integer part */

    decexp = dexp;                          /* convert to integer */
    d *= __pow10(-decexp);                  /* reduce d by power of 10 */

    if (d >= 10) {                          /* may be off by 1 place */
        ++decexp;
        d *= 0.10;
    }

    *decexpp = decexp;                      /* store the decimal exponent */

    /* compute the desired number of result digits */

    if (fmt == 'e' || fmt == 'E')
        ndigits = prec + 1;                 /* for 'e' or 'E' format */
    else if (fmt == 'f')
        ndigits = prec + decexp + 1;        /* for 'f' format */
    else
        ndigits = prec;                     /* for 'g' or 'G' format */
    if (ndigits <= 0) {                     /* no significant digits */
        if (ndigits == 0 && d > 5.0)        /* round up to one digit */
            goto ret1;                      /* and return "1" */
        else
            goto ret0;                      /* return "0" */
    } else if (ndigits > DBL_DIG)
        ndigits = DBL_DIG;                  /* maximum precision */

    /* compute the result digits */

    for ( ; cp < &buf[ndigits] && d != 0.0; ) {
        digit = (int) d;
        *cp++ = digit + '0';                /* store next digit */
        d = 10.0 * (d-digit);               /* and reduce d accordingly */
    }

    *cp = '\0';                             /* NUL-terminate result */

    /* round up the result if appropriate */

    if (d <= 5.0) {                         /* do not round up */
        while (--cp != buf && *cp == '0')
            *cp = '\0';                     /* strip a trailing '0' */

        return;
    }

    while (cp-- != buf) {                   /* round up */
        if (++*cp <= '9')                   /* bump last digit */
            return;

        *cp = '\0';                         /* strip a trailing '0' */
    }

    ++cp;                                   /* point to buf again */

ret1:
    ++*decexpp;                             /* and return "1" */
    *cp++ = '1';
    *cp = '\0';
}

/* convert a floating point number d from binary into 'e',
   'E', 'f', 'g', or 'G' ASCII format into the buffer cp.

   fmt is the conversion type.
   prec is the precision.
   aflag is the '#' flag from the conversion specication.
   signp points to the returned sign (-1 negative, 1 nonnegative).

   returns a pointer past the last character.
   called from printf() for fp conversions. */

char *__dtefg(char *cp, double *dp, int fmt, int prec, int aflag, int *signp)
{
    int eflag, decexp;
    char tbuf[DBL_DIG+1];
    double d;

    d = *dp;

    if (prec == 0 && (fmt == 'g' || fmt == 'G'))
        prec = 1;
    else if (prec == -1)
        prec = 6;                       /* Default precision */

    if (d < 0.0) {
        d = -d;                         /* Force d nonnegative */
        *signp = -1;
    } else
        *signp = 1;

    dtoa(fmt, &d, prec, &decexp, tbuf);

    eflag = (fmt=='e' || fmt=='E'
          || ((fmt=='g' || fmt=='G') && (decexp < -4 || decexp >= prec)));

    cp = dtof(cp, tbuf, prec, eflag ? 0 : decexp, fmt, aflag);

    if (eflag) {
        *cp++ = (fmt == 'E' || fmt == 'G') ? 'E' : 'e';
        cp += sprintf(cp, "%+03d", decexp);
    }

    return cp;
}

/* vi: set ts=4 expandtab: */
