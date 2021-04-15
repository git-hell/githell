/* string.c - immutable string table                    ncc, the new c compiler

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
#include <string.h>
#include "../common/util.h"
#include "../common/tailq.h"
#include "cc1.h"
#include "lex.h"
#include "output.h"
#include "symbol.h"
#include "type.h"
#include "string.h"

static TAILQ_HEAD(bucket, string) buckets[STRING_NR_BUCKETS];

#define BUCKET(hash)    &(buckets[(hash) % STRING_NR_BUCKETS])

/* return the string table entry associated with the
   given buffer, creating a new one if necessary. */

struct string *string_new(char *buf, size_t len)
{
    struct bucket *b;
    struct string *s;
    string_hash hash;
    size_t i;

    for (hash = 0, i = 0; i < len; ++i)
        hash = (hash << 4) ^ buf[i];

    b = BUCKET(hash);

    for (s = TAILQ_FIRST(b); s; s = TAILQ_NEXT(s, links)) {
        if ((s->len == len) && (s->hash == hash) && !memcmp(s->s, buf, len)) {
            /* match: remove the string from the bucket: the
               following code will put it back at the front */
            TAILQ_REMOVE(b, s, links);
            break;
        }
    }

    if (s == 0) {
        i = sizeof(struct string) + len + 1;
        s = safe_malloc(i);
        s->hash = hash;
        s->len = len;
        s->k = K_IDENT;
        s->label = 0;
        memcpy(s->s, buf, len);
        s->s[len] = 0;
    }

    TAILQ_INSERT_HEAD(b, s, links);
    return s;
}

/* initialize the hash table buckets and
   seed the string table with c keywords. */

#define MAX_KEYWORD_LEN 8       /* longest keyword is .. */

struct { token_class k; char text[MAX_KEYWORD_LEN+1]; } keywords[] =
{
    { K_ASM,        "__asm"     },  { K_AUTO,       "auto"      },
    { K_BREAK,      "break"     },  { K_CASE,       "case"      },
    { K_CHAR,       "char"      },  { K_CONST,      "const"     },
    { K_CONTINUE,   "continue"  },  { K_DEFAULT,    "default"   },
    { K_DO,         "do"        },  { K_DOUBLE,     "double"    },
    { K_ELSE,       "else"      },  { K_ENUM,       "enum"      },
    { K_EXTERN,     "extern"    },  { K_FLOAT,      "float"     },
    { K_FOR,        "for"       },  { K_GOTO,       "goto"      },
    { K_IF,         "if"        },  { K_INT,        "int"       },
    { K_LONG,       "long"      },  { K_REGISTER,   "register"  },  
    { K_RETURN,     "return"    },  { K_SHORT,      "short"     },
    { K_SIGNED,     "signed"    },  { K_SIZEOF,     "sizeof"    },
    { K_STATIC,     "static"    },  { K_STRUCT,     "struct"    },
    { K_SWITCH,     "switch"    },  { K_TYPEDEF,    "typedef"   },
    { K_UNION,      "union"     },  { K_UNSIGNED,   "unsigned"  },
    { K_VOID,       "void"      },  { K_VOLATILE,   "volatile"  }, 
    { K_WHILE,      "while"     }
};

void string_init(void)
{
    struct string *s;
    int i;

    for (i = 0; i < STRING_NR_BUCKETS; ++i)
        TAILQ_INIT(&buckets[i]);
        
    for (i = 0; i < ARRAY_SIZE(keywords); ++i) {
        s = string_new(keywords[i].text, strlen(keywords[i].text));
        s->k = keywords[i].k;
    }
}

/* print the token associated with the keyword
   token class to the file specified. */

void string_print_k(FILE *fp, token_class k)
{
    int i;

    for (i = 0; i < ARRAY_SIZE(keywords); ++i) {
        if (keywords[i].k == k) {
            fprintf(fp, "'%s'", keywords[i].text);
            return;
        }
    }
}

/* output bytes of a string literal. if
   bytes is longer than the string, the
   string is extended with zero pads. */

void string_emit(struct string *s, size_t bytes)
{
    size_t i;
    char c;

    for (i = 0; i < bytes; ++i) {
        if (i < s->len)
            c = s->s[i];
        else
            c = 0;
    
        if ((i % 8) == 0) {
            if (i != 0) output("\n");
            output("\t.byte ");
        } else
            output(",");

        output("%d", c & 0xFF);
    }

    output("\n");
}

/* call before exiting to emit any strings
   referenced as anonymous literals */

void string_emit_literals(void)
{
    struct string *s;
    int i;

    for (i = 0; i < STRING_NR_BUCKETS; ++i) {
        for (s = TAILQ_FIRST(&buckets[i]); s; s = TAILQ_NEXT(s, links)) {
            if (s->label) {
                output_select(OUTPUT_SEG_TEXT);
                output("%L:\n", s->label);
                string_emit(s, s->len + 1);
            }
        }
    }
}

/* create an anonymous S_STATIC symbol in the
   current scope that references a string literal */

struct symbol *string_symbol(struct string *s)
{
    struct symbol *sym;
    struct type_node *tn;

    sym = symbol_anonymous(s->label);
    s->label = sym->label;
    tn = type_append_bits(&sym->type, T_ARRAY);
    tn->nr_elements = s->len + 1;
    type_append_bits(&sym->type, T_CHAR);

    return sym;
}

/* vi: set ts=4 expandtab: */
