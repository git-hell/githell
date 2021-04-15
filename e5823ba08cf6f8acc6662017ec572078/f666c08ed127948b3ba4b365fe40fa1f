/* modf.c - extract integer/fractional components          ncc standard library

   Copyright (C) 1993 by Sun Microsystems, Inc. All rights reserved.
   Copyright (c) 2021 Charles E. Youse (charles@gnuless.org).
  
   Developed at SunPro, a Sun Microsystems, Inc. business. Permission
   to use, copy, modify, and distribute this software is freely granted,
   provided that this notice is preserved. */

#include <math.h>

/* get two 32 bit ints from a double */

#define EXTRACT_WORDS(ix0,ix1,d)                                            \
    do {                                                                    \
        union __ieee_double ew_u;                                           \
        ew_u.value = (d);                                                   \
        (ix0) = ew_u.words.msw;                                             \
        (ix1) = ew_u.words.lsw;                                             \
    } while (0)

/* get the more significant 32 bit int from a double */

#define GET_HIGH_WORD(i,d)                                                  \
    do {                                                                    \
        union __ieee_double gh_u;                                           \
        gh_u.value = (d);                                                   \
        (i) = gh_u.words.msw;                                               \
    } while (0)

/* create a double from two 32 bit ints */

#define INSERT_WORDS(d,ix0,ix1)                                             \
    do {                                                                    \
        union __ieee_double iw_u;                                           \
        iw_u.words.msw = (ix0);                                             \
        iw_u.words.lsw = (ix1);                                             \
        (d) = iw_u.value;                                                   \
    } while (0)

static const double one = 1.0;

double modf(double x, double *iptr)
{
    int i0, i1, j0;
    unsigned i;
    unsigned high;

    EXTRACT_WORDS(i0,i1,x);
    j0 = ((i0 >> 20) & 0x7ff) - 0x3ff;      /* exponent of x */

    if (j0 < 20) {                          /* integer part in high x */
        if (j0 < 0) {                       /* |x|<1 */
            INSERT_WORDS(*iptr, i0 & 0x80000000, 0);    /* *iptr = +-0 */
            return x;
        } else {
            i = (0x000fffff) >> j0;

            if(((i0 & i) | i1) == 0) {                  /* x is integral */
                *iptr = x;
                GET_HIGH_WORD(high, x);
                INSERT_WORDS(x, high & 0x80000000, 0);  /* return +-0 */
                return x;
            } else {
                INSERT_WORDS(*iptr, i0 & (~i), 0);
                return x - *iptr;
            }
        }
    } else if (j0 > 51) {               /* no fraction part */
        if (j0 == 0x400) {              /* inf/NaN */
            *iptr = x;
            return 0.0 / x;
        }

        *iptr = x * one;
        GET_HIGH_WORD(high,x);
        INSERT_WORDS(x, high & 0x80000000, 0);      /* return +-0 */
        return x;
    } else {                        /* fraction part in low x */
        i = ((unsigned) (0xffffffff)) >> (j0 - 20);

        if((i1 & i) == 0) {                         /* x is integral */
            *iptr = x;
            GET_HIGH_WORD(high,x);
            INSERT_WORDS(x, high & 0x80000000,0);   /* return +-0 */
            return x;
        } else {
            INSERT_WORDS(*iptr, i0, i1 & (~i));
            return x - *iptr;
        }
    }
}

/* vi: set ts=4 expandtab: */
