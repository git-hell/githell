/* field.c - bitfield access optimizations              ncc, the new c compiler

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

#include "cc1.h"
#include "tree.h"
#include "type.h"
#include "field.h"

/* return the mask for a bitfield of the given specification */

long field_mask(int size, int shift)
{
    unsigned long mask;

    mask = -1L;
    mask >>= (sizeof(mask) * BITS_PER_BYTE) - size;
    mask <<= shift;

    return mask;
}

/* get the size and shift of a bitfield fetch */

static void field_fetch_specs(struct tree *tree, int *size, int *shift)
{
    struct type type = TYPE_INITIALIZER(type);

    type_deref(&type, &tree->child->type);

    if (size)
        *size = TYPE_GET_SIZE(&type);

    if (shift)
        *shift = TYPE_GET_SHIFT(&type);

    type_clear(&type);
}

/* convert a bitfield fetch into a straight
   fetch of the host type */

static void field_fetch_unfield(struct tree *tree)
{
    struct type type = TYPE_INITIALIZER(type);

    type_deref(&type, &tree->child->type);
    TYPE_UNFIELD(&type);
    type_clear(&tree->child->type);
    type_ref(&tree->child->type, &type);

    type_clear(&type);
}

/* attempt to optimize some fields accesses. extracting fields
   is an expensive prospect, so we should avoid it when we can. */

struct tree *field_tree_opt(struct tree *tree)
{
    struct type type = TYPE_INITIALIZER(type);
    int size;
    int shift;
    long mask;
    long i;

    tree_normalize(tree);

    if (!TREE_BINARY(tree) || !TREE_FIELD_FETCH(tree->left))
        return tree;

    field_fetch_specs(tree->left, &size, &shift);
    mask = field_mask(size, shift);

    switch (tree->op)
    {
    case E_EQ:
    case E_NEQ:

        /* when testing if the field is equal (or not equal) to zero,
           we fetch the host word and test by ANDing in place. */

        if (!tree_zero(tree->right))
            break;

        field_fetch_unfield(tree->left);

        tree->left = tree_binary(E_AND,
                                 tree->left,
                                 tree_i(TYPE_BASE(&tree->left->type), mask));

        type_copy(&tree->left->type, &tree->left->left->type);
        break;

    case E_ASG:

        /* if assigning a constant that amounts to all ones or all
           zeros, these are OR and AND operations, respectively. */

        if (!TREE_PURE_CON(tree->right))
            break;

        i = tree->right->con.i;
        i <<= shift;
        i &= mask;

        if (i == 0) {
            field_fetch_unfield(tree->left);
            tree->op = E_ANDASG;
            tree->right->con.i = ~mask;
            break;
        }

        if (i == mask) {
            field_fetch_unfield(tree->left);
            tree->op = E_ORASG;
            tree->right->con.i = mask;
            break;
        }
    }

    return tree;
}

/* vi: set ts=4 expandtab: */
