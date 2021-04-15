/* lex.c - lexical analyzer                             ncc, the new c compiler

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

#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <ctype.h>
#include <errno.h>
#include <limits.h>
#include "../common/escape.h"
#include "../common/util.h"
#include "cc1.h"
#include "lex.h"
#include "string.h"
#include "symbol.h"

/* the parser's current token */

struct token token;

/* tracked for reporting errors only */

struct string *error_path;
int error_line_no;

/* token recognition is done in a sliding buffer of LEX_BUFSZ bytes.
   making the buffer larger will speed the lexer to a degree and
   also increase the length of the longest recognizable token. */

#define LEX_BUFSZ 4096

static int fd;

static char buf[LEX_BUFSZ];

static char *begin = buf + LEX_BUFSZ;   /* start of current token */
static char *pos = buf + LEX_BUFSZ;     /* current position */
static char *invalid = buf + LEX_BUFSZ; /* first invalid character */

/* guarantee that pos[0]..pos[LOOKAHEAD-3] are
   valid, or are at least NUL-terminated. */

#define LOOKAHEAD 3

static void refill(void)
{
    size_t offset;
    size_t bytes;
    ssize_t bytes_read;

    if (((invalid - pos) < LOOKAHEAD) && (fd != -1)) {
        offset = begin - buf;                               /* slide buffer */
        bytes = invalid - begin;
        memmove(buf, begin, bytes);
        begin -= offset;
        pos -= offset;
        invalid -= offset;

        bytes = (buf + LEX_BUFSZ) - invalid;                /* now refill */
        if (bytes == 0) error(FATAL, "token too long");
        bytes_read = read(fd, invalid, bytes);
        if (bytes_read == -1) error(FATAL, "input I/O error (%E)");
        invalid += bytes_read;

        if (bytes_read < bytes) {
            *invalid = 0;
            close(fd);
            fd = -1;
        }
    }
}

/* use when incrementing pos might
   bust the LOOKAHEAD window */

#define ADVANCE()                                                       \
    do {                                                                \
        ++pos;                                                          \
                                                                        \
        if ((invalid - pos) < LOOKAHEAD)                                \
            refill();                                                   \
    } while (0)

/* identifiers/keywords. the string table
   is kind enough to tell us whether an
   identifier is a keyword or not. */

static token_class ident(void)
{
    while (isalnum(*pos) || (*pos == '_'))
        ADVANCE();

    token.text = string_new(begin, pos - begin);
    return token.text->k;
}

/* we're currently positioned over a quote of
   some kind; advance until we encompass the
   entirety of the delimited token */

static void delimit(void)
{
    int delimiter = *pos;
    int escaped = 1;
    int c;

    while (c = *pos) {
        ADVANCE();

        if ((c == delimiter) && !escaped)
            return;

        if (escaped)
            escaped = 0;
        else
            escaped = (c == '\\');
    }

    if (delimiter == '\'')
        error(FATAL, "unterminated character constant");
    else
        error(FATAL, "unterminated string literal");
}

/* string literals. we take advantage of the fact
   that the length of a literal token is always
   less than the length of what it represents, so
   we overwrite the token with its interpretation.

   we concatenate strings here. this is technically
   incorrect, because this should happen after all
   directives disappear; a #line or #pragma between
   string literals will expose this flaw. if this
   poses real-world problems, we can fix later. */

static token_class strlit(void)
{
    size_t len;
    char *head;
    char *tail;
    int c;

    while (*pos == '\"') {
        delimit();
        len = pos - begin;
        while (isspace(*pos)) ADVANCE();
    }

    pos = begin + len; /* don't consume trailing whitespace */
    tail = begin;
    head = begin + 1;

    while (head != pos) {
        while (*head != '"') {
            c = escape(&head);

            if (c == -1)
                error(FATAL, "malformed escape sequence in string literal");

            *tail++ = c;
        }

        while ((++head != pos) && isspace(*head))
            if (*head == '\n')
                ++error_line_no;
    }

    token.text = string_new(begin, tail - begin);
    return K_STRLIT;
}

/* character constants. we support neither wide
   character nor multi-character constants. */

static token_class ccon(void)
{
    delimit();

    ++begin;
    token.i = escape(&begin);
    
    if (token.i == -1)
        error(FATAL, "malformed escape sequence in character constant");

    if (*begin != '\'')
        error(FATAL, "multi-character constant");

    return K_ICON;
}

/* numeric constants. ANSI really made a
   mess of these (for dubious reasons), but
   luckily we can fob off most of the dirty
   work on the standard library. */

static token_class number(void)
{
    char *endptr;
    char is_unsigned = 0;
    char is_long = 0;

    while (isalnum(*pos) || (*pos == '.') || (*pos == '_')) {
        if ((toupper(*pos) == 'E') && ((pos[1] == '+') || (pos[1] == '-')))
            ++pos;

        ADVANCE();
    }

    errno = 0;
    token.u = strtoul(begin, &endptr, 0);
    if (toupper(*endptr) == 'U') { is_unsigned = 1; ++endptr; }
    if (toupper(*endptr) == 'L') { is_long = 1; ++endptr; }
    if (toupper(*endptr) == 'U') { is_unsigned = 1; ++endptr; }
    if ((endptr != pos) || errno) goto floating;

    if (is_long && is_unsigned)
        return K_ULCON;
    else if (is_long) {
        if (token.u > LONG_MAX)
            return K_ULCON;
        else
            return K_LCON;
    } else if (is_unsigned) {               /* surely there is a */
        if (token.u > UINT_MAX)             /* more elegant way to */
            return K_ULCON;                 /* determine the type */
        else                                /* of an integral constant */
            return K_UCON;                  /* but i am way too lazy */
    } else {                                /* right now to think */
        if (token.u > LONG_MAX)             /* about it */
            return K_ULCON;
        else if (token.u > UINT_MAX)
            return K_LCON;
        else if (token.u > INT_MAX) {
            if (*begin == '0')
                return K_UCON;
            else
                return K_LCON;
        } else
            return K_ICON;
    }

floating:
    errno = 0;
    token.f = strtod(begin, &endptr);

    switch (toupper(*endptr))
    {
    case 'L':   token.k = K_LDCON; 
                ++endptr;
                break;

    case 'F':   token.f = strtof(begin, &endptr);
                token.k = K_FCON;
                ++endptr;
                break;

    default:    token.k = K_DCON;
                break;
    }

    if (endptr != pos) error(FATAL, "malformed numeric constant");
    if (errno) error(FATAL, "floating-point constant out of range");

    return token.k;
}

/* the first pipeline stage does the
   heavy lifting, recognizing tokens */

#define OPERATOR(self, dup, eq, dupeq)                                  \
    do {                                                                \
        if (dupeq && (pos[1] == *pos) && (pos[2] == '=')) {             \
            pos += 3;                                                   \
            return dupeq;                                               \
        }                                                               \
                                                                        \
        if (dup && (pos[1] == *pos)) {                                  \
            pos += 2;                                                   \
            return dup;                                                 \
        }                                                               \
                                                                        \
        if (eq && (pos[1] == '=')) {                                    \
            pos += 2;                                                   \
            return eq;                                                  \
        }                                                               \
                                                                        \
        ++pos;                                                          \
        return self;                                                    \
    } while (0)

static token_class lex0(void)
{
    begin = pos;
    refill();

    while (isspace(*pos) && (*pos != '\n')) {
        ADVANCE();
        begin = pos;
    }

    switch (*pos)
    {
    case '\n':  ++pos; return K_NL;
    case '#':   ++pos; return K_HASH;
    case '?':   ++pos; return K_QUEST;
    case ':':   ++pos; return K_COLON;
    case ';':   ++pos; return K_SEMI;
    case ',':   ++pos; return K_COMMA;
    case '(':   ++pos; return K_LPAREN;
    case ')':   ++pos; return K_RPAREN;
    case '{':   ++pos; return K_LBRACE;
    case '}':   ++pos; return K_RBRACE;
    case '[':   ++pos; return K_LBRACK;
    case ']':   ++pos; return K_RBRACK;
    case '~':   ++pos; return K_TILDE;

    case '\'':  return ccon();
    case '\"':  return strlit();

    case '=':   OPERATOR(K_EQ, K_EQEQ, 0, 0);
    case '!':   OPERATOR(K_NOT, 0, K_NOTEQ, 0);
    case '<':   OPERATOR(K_LT, K_SHL, K_LTEQ, K_SHLEQ);
    case '>':   OPERATOR(K_GT, K_SHR, K_GTEQ, K_SHREQ);
    case '^':   OPERATOR(K_XOR, 0, K_XOREQ, 0);
    case '|':   OPERATOR(K_OR, K_LOR, K_OREQ, 0);
    case '&':   OPERATOR(K_AND, K_LAND, K_ANDEQ, 0);
    case '*':   OPERATOR(K_MUL, 0, K_MULEQ, 0);
    case '/':   OPERATOR(K_DIV, 0, K_DIVEQ, 0);
    case '%':   OPERATOR(K_MOD, 0, K_MODEQ, 0);
    case '+':   OPERATOR(K_PLUS, K_INC, K_PLUSEQ, 0);

    case '-':   if (pos[1] == '>') {
                    pos += 2;
                    return K_ARROW;
                }

                OPERATOR(K_MINUS, K_DEC, K_MINUSEQ, 0);

    case '.':   if ((pos[1] == '.') && (pos[2] == '.')) {
                    pos += 3;
                    return K_ELLIP;
                }

                if (!isdigit(pos[1])) {
                    ++pos;
                    return K_DOT;
                }
                
                /* fall thru */

    case '0': case '1': case '2': case '3': case '4':
    case '5': case '6': case '7': case '8': case '9':

                return number();

    case 'A': case 'B': case 'C': case 'D': case 'E':
    case 'F': case 'G': case 'H': case 'I': case 'J':
    case 'K': case 'L': case 'M': case 'N': case 'O':
    case 'P': case 'Q': case 'R': case 'S': case 'T':
    case 'U': case 'V': case 'W': case 'X': case 'Y':
    case 'Z': case '_': case 'a': case 'b': case 'c':
    case 'd': case 'e': case 'f': case 'g': case 'h':
    case 'i': case 'j': case 'k': case 'l': case 'm':
    case 'n': case 'o': case 'p': case 'q': case 'r':
    case 's': case 't': case 'u': case 'v': case 'w':
    case 'x': case 'y': case 'z':

                return ident();

    case 0:
        if (pos == invalid) return 0;
    default:
        error(FATAL, "invalid character (ASCII %d)", *pos & 0xff);
    }
}

/* the middle pipeline stage reinjects tokens
   pushed back by lex_peek(). the initial K_NL
   ensures the logic in the outer lexer works
   properly on the first call. */

static struct token next = { K_NL };

static void lex1(void)
{
    if (next.k == 0)
        token.k = lex0();
    else {
        token = next;
        next.k = 0;
    }
}

/* look ahead one token without
   advancing the lexer. */

struct token lex_peek(void)
{
    struct token tmp;

    if (next.k == 0) {
        tmp = token;
        lex();
        next = token;
        token = tmp;
    }

    return next;
}

/* the outer lexer (which is the client interface) is responsible
   for error location tracking, which includes interpreting (and
   then filtering out) pseudo-tokens and # line directives. */

void lex(void)
{
    lex1();

    while (token.k == K_NL) {
        ++error_line_no;
        lex1();

        if (token.k == K_HASH) {
            lex1();
            if (token.k == K_ICON) {
                error_line_no = token.i;
                lex1();
                if (token.k == K_STRLIT) {
                    error_path = token.text;
                    lex1();
                }
            }

            if (token.k != K_NL) error(FATAL, "malformed directive");
            lex1();
        }
    }
}

/* get the party started */

void lex_init(char *path)
{
    fd = open(path, O_RDONLY);
    if (fd == -1) error(FATAL, "can't open '%s' (%E)", path);
    error_path = string_new(path, strlen(path));
    lex();
}

/* text[] table maps the token class base values to printable text,
   solely for the purposes of error reporting. to avoid unnecessary
   duplication, keywords are not included here - we use the helper
   from string.c instead. nevermind the duplicated strings in THIS
   table: any compiler worth its salt will consolidate them.

   obviously these values must be kept in sync with token_class. */

static char *text[] =
{
    "end-of-file",
    "identifier",
    "string literal",
    "integral constant",
    "integral constant",
    "integral constant",
    "integral constant",
    "floating-point constant",
    "floating-point constant",
    "floating-point constant",
    
    0, 0,   /* pseudo tokens deliberately omitted: should not happen */

    "(",    ")",    "[",    "]",    "{",    "}",    ".",    "...",  
    "^",    ",",    ":",    ";",    "?",    "~",    "->",   "++",   
    "--",   "!",    "/",    "*",    "+",    "-",    ">",    ">>",   
    ">=",   ">>=",  "<",    "<<",   "<=",   "<<=",  "&",    "&&",   
    "&=",   "|",    "||",   "|=",   "-=",   "+=",   "*=",   "/=",   
    "==",   "!=",   "%",    "%=",   "^=",   "="
};

/* print human-readable description of token
   class to the specified file */

void lex_print_k(FILE *fp, token_class k)
{
    int base = K_BASE(k);

    if (base < ARRAY_SIZE(text)) {
        if (text[base]) {
            if (isalnum(*text[base]))
                fputs(text[base], fp);
            else
                fprintf(fp, "'%s'", text[base]);
        }
    } else
        string_print_k(fp, k);  /* keyword */
}

/* just like lex_print_k(), except we have the whole token
   and so can potentially provide a wee bit more about it. */

void lex_print_token(FILE *fp, struct token t)
{
    lex_print_k(fp, token.k);

    switch (token.k)
    {
    case K_ICON:
    case K_LCON:    fprintf(fp, " [%ld]", token.i); break;

    case K_UCON:
    case K_ULCON:   fprintf(fp, " [%lu]", token.u); break;

    case K_FCON:
    case K_DCON:
    case K_LDCON:   fprintf(fp, " [%f]", token.f); break;

    case K_IDENT:   fprintf(fp, " '%s'", token.text->s); break;
    }
}

/* simple parsing helpers */

void lex_expect(token_class k)
{
    if (token.k != k)
        error(FATAL, "expected %k (found %t)", k, token);
}

void lex_match(token_class k)
{
    lex_expect(k);
    lex();
}

/* returns true if the token could be the start of a declaration-
   either a declaration-specifier or an identifier that names a type. */

int k_decl(struct token tok)
{
    if ((tok.k & K_DECL) || ((tok.k == K_IDENT)
      && symbol_typename(tok.text)))
        return 1;
    
    return 0;
}

/* vi: set ts=4 expandtab: */
