/* cc.c - compiler driver                               ncc, the new c compiler

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

#include <stdio.h>
#include <stdarg.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/wait.h>

/* override the default root of the compiler installation
   by defining ROOT, or override the paths individually. */

#ifndef ROOT
#define ROOT "/usr"
#endif

#ifndef CPP
#define CPP ROOT "/bin/cpp"
#endif

#ifndef CC1
#define CC1 ROOT "/lib/cc1"
#endif

#ifndef AS
#define AS ROOT "/bin/as"
#endif

#ifndef LD
#define LD ROOT "/bin/ld"
#endif

#ifndef CRT0
#define CRT0 ROOT "/lib/crt0.o"
#endif

#ifndef LIBC
#define LIBC ROOT "/lib/libc.a"
#endif

#ifndef INC
#define INC ROOT "/include"
#endif

/* struct list is an expandable container used to build argument
   vectors for exec(). we grow the vectors in LIST_INC increments.
   lists do not own the strings they point to, only the pointers. */

#define LIST_INC    10      

struct list
{
    int capacity;
    int len;
    char **s;
};

struct list cpp;        /* templates */
struct list cc1;
struct list as;
struct list ld;
struct list args;       /* arguments built for current command */
struct list temps;      /* list of temp files (delete before exit) */

#define EXEC_FILE   0           /* file types */
#define C_FILE      'c'
#define CC1_FILE    'i'
#define AS_FILE     's'
#define LD_FILE     'o'
#define LIB_FILE    'a'

int goal = EXEC_FILE;       /* file type for result(s) */
char *a_out;                /* target executable name */
int n_flag;                 /* no standard include files */

/* call before exit to axe all the temp files */

void remove_temps(void)
{
    int i;

    for (i = 0; i < temps.len; ++i)
        unlink(temps.s[i]);
}

/* print an error message, clean up and abort */

void error(char *fmt, ...)
{
    va_list args;

    va_start(args, fmt);
    fprintf(stderr, "cc: ");
    vfprintf(stderr, fmt, args);
    va_end(args);
    fputc('\n', stderr);

    remove_temps();
    exit(1);
}

/* allocate memory or bomb */

void *mem(size_t len)
{
    void *p = malloc(len);

    if (p == 0)
        error("out of memory");

    return p;
}

/* add strings to the end of list, growing
   it as necessary. terminate with 0. */

void add(struct list *list, ...)
{
    va_list ss;
    char **new;
    char *s;

    va_start(ss, list);

    while (s = va_arg(ss, char *)) {
        if (list->len == list->capacity) {
            new = mem(sizeof(char *) * (list->capacity + LIST_INC + 1));

            if (list->s) {
                memcpy(new, list->s, sizeof(char *) * (list->capacity));
                free(list->s);
            }

            list->s = new;
            list->capacity += LIST_INC;
        }

        list->s[list->len++] = s;
        list->s[list->len] = 0;
    }
}

/* truncate dst and make it a copy of src.
   this could be more efficient, but why? */

void copy(struct list *dst, struct list *src)
{
    int i;

    dst->len = 0;

    if (dst->capacity)
        dst->s[0] = 0;

    for (i = 0; i < src->len; ++i)
        add(dst, src->s[i], 0);
}

/* return the type of file based on its extension */

int type(char *path)
{
    char *dot;

    dot = strrchr(path, '.');

    if (dot) {
        ++dot;

        switch (*dot)
        {
        case C_FILE:
        case CC1_FILE:
        case AS_FILE:
        case LD_FILE:
        case LIB_FILE:
            return *dot;
        }
    }

    error("'%s': unknown file type", path);
}

/* return a copy of the name in question with its
   extension changed to the file type specified.
   notice that we make no attempt to track these
   allocations, we are dirty, dirty, lazy leakers. */

char *morph(char *path, int type)
{
    char *dot;
    char *new;

    new = mem(strlen(path) + 1);
    strcpy(new, path);
    dot = strrchr(new, '.');
    *++dot = type;

    return new;
}

/* run the command indicated by args, which
   will output to out_path. abort on error. */

void run(struct list *args, char *out_path)
{
    pid_t pid;
    int status;

    if ((pid = fork()) == 0) {
        execvp(args->s[0], args->s);
        error("can't exec '%s': %s", args->s[0], strerror(errno));
    }
        
    if (pid == -1)
        error("can't fork: %s", strerror(errno));

    while (pid != wait(&status))
        ;

    if (status != 0) {
        add(&temps, out_path, 0);
        error("compilation terminated abnormally");
    }
}

int main(int argc, char **argv)
{
    char *src;
    char *new;

    add(&cpp, CPP, 0);
    add(&cc1, CC1, 0);
    add(&as, AS, "-o", 0);
    add(&ld, LD, "-e", "cstart", "-o", 0);

    ++argv;

    while (*argv && (*argv[0] == '-')) {
        switch ((*argv)[1]) {
        case 'D':
        case 'I':   
            add(&cpp, *argv, 0);
            break;
        
        case 'd':
            add(&cc1, *argv, 0);
            break;

        case 'n':
            ++n_flag;
            break;

        case 'S':
        case 'P':
        case 'c':
            if ((*argv)[2])
                error("malformed goal option");

            if (goal != EXEC_FILE)
                error("conflicting goal options");

            switch ((*argv)[1])
            {
            case 'S':   goal = AS_FILE; break;
            case 'P':   goal = CC1_FILE; break;
            case 'c':   goal = LD_FILE; break;
            }
            
            break;

        case 'o':
            if ((*argv)[2] || (argv[1] == 0))
                error("malformed output option (-o)");

            if (a_out)
                error("duplicate output specified (-o)");

            ++argv;
            a_out = *argv;
            break;

        default:
            error("unrecognized option: -%c\n", (*argv)[1]);
        }

        ++argv;
    }

    if (a_out == 0)
        a_out = "a.out";

    if (n_flag == 0)
        add(&cpp, "-I" INC, 0);

    add(&ld, a_out, CRT0, 0);

    if (*argv == 0)
        error("no input files");

    while (*argv) {
        src = *argv;

        switch (type(src)) {
        case C_FILE:
            new = morph(src, CC1_FILE);
            copy(&args, &cpp);
            add(&args, src, 0);
            add(&args, new, 0);
            run(&args, new);

            if (goal == CC1_FILE)
                break;

            add(&temps, new, 0);
            src = new;

        case CC1_FILE:
            new = morph(src, AS_FILE);
            copy(&args, &cc1);
            add(&args, src, 0);
            add(&args, new, 0);
            run(&args, new);

            if (goal == AS_FILE)
                break;

            src = new;
            add(&temps, new, 0);

        case AS_FILE:
            new = morph(src, LD_FILE);
            copy(&args, &as);
            add(&args, new, 0);
            add(&args, src, 0);
            run(&args, new);

            if (goal == LD_FILE)
                break;

            add(&temps, new, 0);
            src = new;

        case LD_FILE:
            add(&ld, src, 0);
        }

        argv++;
    }

    if (goal == EXEC_FILE) {
        add(&ld, LIBC, 0);
        run(&ld, a_out);
    }

    remove_temps();
    return 0;
}

/* vi: set ts=4 expandtab: */
