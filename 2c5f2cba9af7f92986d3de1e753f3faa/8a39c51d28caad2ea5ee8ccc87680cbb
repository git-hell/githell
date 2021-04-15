/* input.c - input/include files                        ncc, the new c compiler

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
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include "cpp.h"
#include "input.h"
#include "directive.h"
#include "vstring.h"

struct input_stack input_stack = SLIST_HEAD_INITIALIZER(input_stack);

/* self-explanatory functions to push and pop
   entries from the input stack */

static void input_push(FILE *fp, char *path)
{
    struct input *tmp;

    tmp = safe_malloc(sizeof(struct input));
    vstring_init(&tmp->path);
    vstring_puts(&tmp->path, path);
    tmp->fp = fp;
    tmp->line_no = 0;
    SLIST_INSERT_HEAD(&input_stack, tmp, link);
    need_sync = 1;
}

static void input_pop(void)
{
    struct input *tmp = INPUT_STACK;

    /* before we close the main file,
       check to be sure we're not in
       an unterminated conditional */

    if (!SLIST_NEXT(tmp, link))
        directive_check();

    SLIST_REMOVE_HEAD(&input_stack, link);
    fclose(tmp->fp);
    vstring_free(&tmp->path);
    free(tmp);
    need_sync = 1;
}

/* overwrite all c-style comments with spaces. since they can lines,
   we maintain in_comment (outside the function for unwind() to use).
   as an easy optimization, we trim any trailing whitespace, too. */

static char in_comment;

static void erase_comments(char *s)
{
    int delim = 0;
    char *space = 0;

    while (*s) {
        if (delim) {
            if (*s == delim) delim = 0;
            if ((*s == '\\') && (s[1] == delim)) ++s;
        } else if (in_comment) {
            if ((*s == '*') && (s[1] == '/')) {
                s[1] = ' ';
                in_comment = 0;
            }
            
            *s = ' ';
        } else {
            if ((*s == '/') && (s[1] == '*')) {
                *s = ' ';
                s[1] = ' ';
                in_comment = 1;
            } else {
                if ((*s == '"') || (*s == '\''))
                    delim = *s;
            }
        }

        if (*s == ' ') {
            if (space == 0)
                space = s;
        } else
            space = 0;

        ++s;
    }

    if (space) *space = 0;
}

/* ensure that the top of the stack has at least one character left to read.
   returns that character, or -1 if we've reached the end of input. mode is
   INPUT_MODE_THIS or INPUT_MODE_ANY; if the latter, we will unwind the stack
   attempting to find input. */

static int unwind(input_mode mode)
{
    int c = -1;

    while (INPUT_STACK) {
        c = getc(INPUT_STACK->fp);

        if (c == -1) {
            if (in_comment) error("end-of-file in comment");
            if (mode == INPUT_MODE_THIS) break;
            input_pop();
        } else {
            ungetc(c, INPUT_STACK->fp);
            break;
        }
    }

    return c;
}

/* read the next logical line from the input stack. as the name indicates,
   this function is responsible for line concatenation. returns a pointer
   to a static NUL-terminated buffer on success, or 0 at end-of-input. */

static char *concat_line(input_mode mode)
{
    static struct vstring buf = VSTRING_INITIALIZER;

    int c = 0;
    int esc;

    if (unwind(mode) == -1) return 0;
    ++(INPUT_STACK->line_no);
    vstring_clear(&buf);

    for (;;) {
        esc = (c == '\\');
        c = getc(INPUT_STACK->fp);

        switch (c)
        {
        case '\n':
            if (esc) {
                vstring_rubout(&buf);
                ++(INPUT_STACK->line_no);
                continue;
            }
        case -1:
            return VSTRING_BUF(buf);
            
        default:
            vstring_putc(&buf, c);
        }
    }
}

/* convert a line of text into a sequence of tokens.
   returns the number of tokens (which might be 0). */

int input_tokenize(struct list *list, char *s)
{
    struct token *t;
    int count = 0;

    while (*s) {
        t = token_scan(s, &s);
        list_append(list, t);
        ++count;
    }

    return count;
}

/* read the next line of input from the input stack and tokenize it.
   the tokens are appended to the end of list. returns the number
   of tokens appended, or -1 if end of input. */

int input_tokens(input_mode mode, struct list *list)
{       
    char *s;

    s = concat_line(mode);
    if (s == 0) return -1;
    erase_comments(s);
    return input_tokenize(list, s);
}

/* system include paths */

struct dir
{
    struct vstring path;
    SLIST_ENTRY(dir) link;
};

SLIST_HEAD(, dir) system_dirs = SLIST_HEAD_INITIALIZER(system_dirs);

/* add an entry to the system include search */

void input_dir(char *path)
{
    struct dir *dir;

    dir = safe_malloc(sizeof(struct dir));
    vstring_init(&dir->path);
    vstring_puts(&dir->path, path);
    SLIST_INSERT_HEAD(&system_dirs, dir, link);
}

/* attempt to find a file, open it, and push it on the input stack.
   this function will try a few different places, depending on search:
   
   INPUT_SEARCH_NOWHERE:    no search is performed, path is tried as-is
   INPUT_SEARCH_LOCAL:      the directory of the current top-of-stack
                            is tried first, and if no file is found ...
   INPUT_SEARCH_SYSTEM:     all the system_dirs are searched */

void input_open(char *path, input_search search)
{
    FILE *fp;
    struct vstring vs;
    struct dir *dir;

    vstring_init(&vs);
    dir = SLIST_FIRST(&system_dirs);

    for (;;) {
        vstring_clear(&vs);

        switch (search)
        {
        case INPUT_SEARCH_NOWHERE:
            vstring_puts(&vs, path);
            goto open;

        case INPUT_SEARCH_LOCAL:
            vstring_concat(&vs, &(INPUT_STACK->path));

            while (VSTRING_LEN(vs) && (vstring_last(&vs) != '/'))
                vstring_rubout(&vs);

            vstring_puts(&vs, path);
            search = INPUT_SEARCH_SYSTEM;
            break;

        case INPUT_SEARCH_SYSTEM:
            if (dir == 0) error("can't find '%s'", path);
            vstring_concat(&vs, &dir->path);
            vstring_putc(&vs, '/');
            vstring_puts(&vs, path);
            dir = SLIST_NEXT(dir, link);
            break;
        }

        if (access(VSTRING_BUF(vs), F_OK) == 0) break;
    }

open:
    fp = fopen(VSTRING_BUF(vs), "r");
    if (fp == 0) error("can't open '%s' for reading", path);
    input_push(fp, VSTRING_BUF(vs));
    vstring_free(&vs);
}

/* vi: set ts=4 expandtab: */
