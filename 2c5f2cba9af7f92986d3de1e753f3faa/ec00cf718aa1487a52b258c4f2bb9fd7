/* input.h - input/include files                        ncc, the new c compiler

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

#ifndef INPUT_H
#define INPUT_H

#include <stdio.h>
#include "../common/slist.h"
#include "vstring.h"
#include "token.h"

/* the stack formed by the main input file and its nested #includes
   is represented by (... wait for it ...) a stack, using an SLIST. */

struct input
{
    FILE *fp;
    struct vstring path;
    int line_no;
    SLIST_ENTRY(input) link;
};

SLIST_HEAD(input_stack, input);

extern struct input_stack input_stack;

/* the top of the stack is exposed to allow access to the line_no
   and path of the topmost file, and determine when end-of-input is
   reached (when INPUT_STACK is NULL). */

#define INPUT_STACK         SLIST_FIRST(&input_stack)

typedef int input_search;   /* INPUT_SEARCH_* */

#define INPUT_SEARCH_NOWHERE    0
#define INPUT_SEARCH_SYSTEM     1
#define INPUT_SEARCH_LOCAL      2

extern void input_open(char *, input_search);
extern void input_dir(char *);

typedef int input_mode;  /* INPUT_MODE_* */

#define INPUT_MODE_THIS 0
#define INPUT_MODE_ANY  1

extern int input_tokenize(struct list *, char *);
extern int input_tokens(input_mode, struct list *);

#endif /* INPUT_H */

/* vi: set ts=4 expandtab: */
