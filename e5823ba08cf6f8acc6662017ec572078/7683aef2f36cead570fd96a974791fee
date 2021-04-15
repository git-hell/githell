/* ctime.c - time conversions                              ncc standard library

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

#include <stdlib.h>
#include <time.h>

/* pseudo System V, employs TIMEZONE environment for gmt offset,
   timezone abbreviations, and daylight savings time information.

        TIMEZONE=SSS:mmm:DDD:n.d.m:n.d.m:h:m

   SSS - standard timezone abbreviation up to 31 characters long
   mmm - minutes west of GMT
   DDD - daylight timezone abbreviation also 31 characters
   n.d.m - n'th occurrence of d'th day of week in m'th month of start
        and end of daylight savings time. negative n indicates counting
        backwards from end of month. zero n indicates absolute date,
        d'th day of m'th month. days and months from 1 to n.
   h - hour of change from standard to daylight
   m - minutes of adjustment at change.

   example - Central standard in current US conventions:

        TIMEZONE=CST:360:CDT:-1.1.4:-1.1.10:2:60

   only the first two fields are required;

   if no daylight timezone is specified, then no daylight conversion is done.
   if no dates for daylight are given, they default to 83-05-10 US standard.
   if no hour of daylight time change is specified, it defaults to 2AM.
   if no minutes of adjustment is specified, it defaults to 60.

   these defaults can be changed by overwriting tzdstdef[]. */

static char tzdstdef[] = "1.1.4:-1.1.10:2:60......";

#define FEB         1
#define NWDAY       7               /* number of weekdays */
#define NMON        12              /* number of months */
#define NTZNAME     31              /* max time zone name size */

static char daynames[3 * NWDAY + 1] = "SunMonTueWedThuFriSat";
static char dpm[] = { 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };
static int dstadjust = 60 * 60;   /* in seconds */
static char dsthour = 2;

static struct dsttimes {
    char dst_occur;
    char dst_day;
    char dst_month;
} dsttimes[2] = {
    {  1, 0, 3 },                   /* first sunday in april */
    { -1, 0, 9 }                    /* last sunday in october */
};

static char months[3 * NMON + 1] = "JanFebMarAprMayJunJulAugSepOctNovDec";
static char timestr[] = "AAA AAA DD DD:DD:DD DDDD\n";
static struct tm tm;

static char tz0[NTZNAME + 1] = "GMT";
static char tz1[NTZNAME + 1] = "";

long timezone;
char *tzname[2] = { tz0, tz1 };

/* return 1 on leap years; 0 otherwise */

static int isleap(int yr)
{
    if (yr % 4000 == 0)
        return 0;

    return (yr % 400 == 0 || (yr % 100 != 0 && yr % 4 == 0));
}

/* compute which day of the month is
   the nth occurrence of a weekday */

static int nthday(struct dsttimes *dp)
{
    int nthday;
    int nth;

    if ((nth = dp->dst_occur) == 0)
        return dp->dst_day;

    nthday = tm.tm_mday - tm.tm_wday + dp->dst_day;

    if (nth > 0) {
        while (nthday > 0)
            nthday -= 7;

        do
            nthday += 7;
        while (--nth > 0);

    } else {
        while (nthday < dpm[tm.tm_mon])
            nthday += 7;

        do
            nthday -= 7;
        while (++nth < 0);
    }

    return nthday;
}

/* check for daylight savings time.

   watch out for the southern hemisphere, where start month > end month.
   this does not handle the case start == end correctly for all cases. */

static int isdaylight(void)
{
    int month, start, end, xday;

    if (tzname[1][0] == 0)      /* no name, no daylight time */
        return 0;

    month = tm.tm_mon;
    start = dsttimes[0].dst_month;
    end = dsttimes[1].dst_month;

    if ((start <= end && (month < start || month > end))
      || (start > end && (month < start && month > end)))
        return 0;                       /* current month is never DST */
    else if (month == start) {          /* DST starts this month */
        xday = nthday(&dsttimes[0]);    /* DST starts on xday */

        if (tm.tm_mday != xday)
            return (tm.tm_mday > xday);

        return (tm.tm_hour >= dsthour);
    } else if (month == end) {          /* DST ends this month */
        xday = nthday(&dsttimes[1]);    /* DST ends on xday */

        if (tm.tm_mday != xday)
            return (tm.tm_mday < xday);

        return (tm.tm_hour < dsthour-1);
    } else
        return 1;                       /* current month is always DST */
}

/* parse and set dst parameters from string */

#define SKIPTO(x)  while (*cp1 && *cp1++ != (x))

static void setdst(char *cp1)
{
    /* get optional start of daylight time */

    if (*cp1) {
        dsttimes[0].dst_occur = atoi(cp1); SKIPTO('.');
        dsttimes[0].dst_day = atoi(cp1)-1; SKIPTO('.');
        dsttimes[0].dst_month = atoi(cp1)-1; SKIPTO(':');
    }

    /* get optional end of daylight time */

    if (*cp1) {
        dsttimes[1].dst_occur = atoi(cp1); SKIPTO('.');
        dsttimes[1].dst_day = atoi(cp1) - 1; SKIPTO('.');
        dsttimes[1].dst_month = atoi(cp1) - 1; SKIPTO(':');
    }

    /* get optional hour of daylight time advance */

    if (*cp1) {
        dsthour = atoi(cp1); SKIPTO(':');
    }

    /* get optional minutes of adjustment */

    if (*cp1)
        dstadjust = atoi(cp1) * 60;
}

/* set the timezone parameters from the environment */

void tzset(void)
{
    char *cp1, *cp2;
    static int tzset = 0;

    if (tzset)  /* once is enough */
        return;

    tzset = 1;

    if ((cp1 = getenv("TIMEZONE")) == 0)
        return;

    timezone = 0;

    /* read primary timezone name and nul terminate */

    cp2 = tzname[0];

    while (*cp1 && *cp1 != ':' && cp2 < &tzname[0][NTZNAME])
        *cp2++ = *cp1++;

    *cp2++ = '\0';

    while (*cp1 && *cp1++ != ':')
        ;

    /* read timezone offset and convert to seconds */

    timezone = atoi(cp1) * 60L;

    while (*cp1 && *cp1++ != ':')
        ;

    /* read daylight timezone name and nul terminate */

    cp2 = tzname[1];

    while (*cp1 && *cp1 != ':' && cp2 < &tzname[1][NTZNAME])
        *cp2++ = *cp1++;

    *cp2++ = '\0';

    while (*cp1 && *cp1++ != ':')
        ;

    /* exit if no daylight time */

    if (tzname[1][0] == '\0')
        return;

    setdst(tzdstdef);   /* set default dst parameters */
    setdst(cp1);        /* set supplied dst parameters */
}

/* do conversions from GMT to the tm structure. */

struct tm *gmtime(const time_t *tp)
{
    long xtime;
    unsigned days;
    long secs;
    int year;
    int ydays;
    int wday;
    char *mp;

    if ((xtime = *tp) < 0)
        xtime = 0;

    days = xtime / (60L * 60L * 24L);
    secs = xtime % (60L * 60L * 24L);
    tm.tm_hour = secs / (60L * 60L);
    secs = secs % (60L * 60L);
    tm.tm_min = secs / 60;
    tm.tm_sec = secs % 60;

    /*
     * start at thursday (wday=4) Jan 1, 1970 - 
     * the unix epoch - and calculate from there.
     */

    wday = (4 + days) % NWDAY;
    year = 1970;

    for (;;) {
        ydays = isleap(year) ? 366 : 365;

        if (days < ydays)
            break;

        year++;
        days -= ydays;
    }

    tm.tm_year = year - 1900;
    tm.tm_yday = days;
    tm.tm_wday = wday;

    /*
     * setup february's #days now.
     */

    if (isleap(year))
        dpm[FEB] = 29;
    else
        dpm[FEB] = 28;

    for (mp = &dpm[0]; mp < &dpm[NMON] && days >= *mp; mp++)
        days -= *mp;

    tm.tm_mon = mp - dpm;
    tm.tm_mday = days + 1;
    return &tm;
}

/* do what gmtime does, but for the local
   timezone, and correct for daylight time. */

struct tm *localtime(const time_t *tp)
{
    time_t ltime;

    tzset();
    ltime = *tp - timezone;
    gmtime(&ltime);

    if (isdaylight()) {
        ltime = *tp - timezone + dstadjust;
        gmtime(&ltime);
        tm.tm_isdst = 1;
    } else
        tm.tm_isdst = 0;

    return &tm;
}

/* returns a printable version of the time which
   has been broken down as in the tm structure. */

#define TODIGIT(c) ((c)+'0')

char *asctime(const struct tm *tmp)
{
    char *cp, *xp;
    unsigned i;

    cp = timestr;

    /* day of week */

    if ((i = tmp->tm_wday) >= NWDAY)
        i = 0;

    xp = &daynames[i * 3];
    *cp++ = *xp++;
    *cp++ = *xp++;
    *cp++ = *xp++;
    *cp++ = ' ';

    /* month */

    if ((i = tmp->tm_mon) >= NMON)
        i = 0;

    xp = &months[i * 3];
    *cp++ = *xp++;
    *cp++ = *xp++;
    *cp++ = *xp++;
    *cp++ = ' ';

    /* day of month */

    if ((i = tmp->tm_mday) >= 10)
        *cp++ = TODIGIT(i / 10);
    else
        *cp++ = ' ';

    *cp++ = TODIGIT(i % 10);
    *cp++ = ' ';

    /* hours:mins:seconds */

    *cp++ = TODIGIT((i = tmp->tm_hour) / 10);
    *cp++ = TODIGIT(i % 10);
    *cp++ = ':';
    *cp++ = TODIGIT((i = tmp->tm_min) / 10);
    *cp++ = TODIGIT(i % 10);
    *cp++ = ':';
    *cp++ = TODIGIT((i = tmp->tm_sec) / 10);
    *cp++ = TODIGIT(i % 10);
    *cp++ = ' ';

    /* year */

    i = tmp->tm_year + 1900;
    *cp++ = TODIGIT(i / 1000);
    i = i % 1000;
    *cp++ = TODIGIT(i / 100);
    i = i % 100;
    *cp++ = TODIGIT(i / 10);
    *cp++ = TODIGIT(i % 10);
    *cp++ = '\n';
    *cp++ = '\0';

    return timestr;
}

/* most common interface, returns a static string
   which is a printable version of the time and date. */

char *ctime(const time_t *tp)
{
    return asctime(localtime(tp));
}

/* vi: set ts=4 expandtab: */
