/* cast.c - cast optimizations                          ncc, the new c compiler

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
#include "con.h"
#include "tree.h"
#include "cast.h"

/* a completely useless cast, no change of type */

static bool cast_useless(struct tree *tree)
{
    if (tree->op == E_CAST)
        return TYPE_BASE(&tree->type) == TYPE_BASE(&tree->child->type);
    else
        return FALSE;
}

/* involves merely a change of interpretation, not value.
   these would be casts from one discrete type to another
   of the same size. note this encompasses useless casts. */

static bool cast_reinterpret(struct tree *tree)
{
    if (tree->op == E_CAST)
        return 
          TYPE_DISCRETE(&tree->type)
          && TYPE_DISCRETE(&tree->child->type)
          && (type_sizeof(&tree->type, 0)
                == type_sizeof(&tree->child->type, 0));
    else
        return FALSE;
}

/* widens the representation within a class:
   e.g., int to long, float to double. */

static bool cast_widen(struct tree *tree)
{
    if (tree->op == E_CAST)
        return
          TYPE_SAME_CLASS(&tree->type, &tree->child->type)
          && (type_sizeof(&tree->type, 0)
                > type_sizeof(&tree->child->type, 0));
    else
        return FALSE;
}

/* narrows the representation within a class:
   e.g., long to int, double to float. */

bool cast_narrow(struct tree *tree)
{
    if (tree->op == E_CAST)
        return
          TYPE_SAME_CLASS(&tree->type, &tree->child->type)
          && (type_sizeof(&tree->type, 0)
                < type_sizeof(&tree->child->type, 0));
    else
        return FALSE;
}

/* changes the type class of the tree, i.e.,
   floating-point to discrete or vice-versa. */

static bool cast_reclass(struct tree *tree)
{
    if (tree->op == E_CAST)
        return TYPE_SAME_CLASS(&tree->type, &tree->child->type) == 0;
    else
        return FALSE;
}

/* if we're widening the integral operand of ~ or
   unary -, just to discard the upper bits anyway,
   we can save ourselves the cast. e.g.,

         char c; c = ~c;

   by language rules (and as translated by the front
   end) is equivalent to

         char c; c = (char) ~ (int) c;

   but the promotion to int is pointless. instead, we
   migrate the cast from the result to the operand:

         char c; c = ~ (char) (int) c;
 
   and, in this case, cast_tree_opt() will eliminate both.

   it's not always a win, but this transformation never
   results in more casts than we started with. */

static struct tree *cast_opt_unary(struct tree *tree)
{
    if (cast_narrow(tree) && TYPE_INTEGRAL(&tree->type)
      && TREE_UNARY(tree->child) && (tree->child->op & E_2S)) {
        tree = tree_chop_unary(tree);
        tree->child = tree_cast(tree->child, &tree->type);
        tree->child = cast_tree_opt(tree->child);
    }

    return tree;
}

/* the situation described in cast_opt_unary() extends
   to some binary operators in a straightfoward manner.
   we check to be sure that at least one of the operands
   widened or narrowed to ensure that the transformation
   is not a loser (with more casts). */

static struct tree *cast_opt_binary(struct tree *tree)
{
    if (cast_narrow(tree) && TYPE_INTEGRAL(&tree->type)
      && TREE_BINARY(tree->child) && (tree->child->op & E_2S)
      && (cast_narrow(tree->child->left) || cast_narrow(tree->child->right)
      || cast_widen(tree->child->left) || cast_widen(tree->child->right))) {
        tree = tree_chop_unary(tree);
        tree->left = tree_cast(tree->left, &tree->type);
        tree->left = cast_tree_opt(tree->left);
        tree->right = tree_cast(tree->right, &tree->type);
        tree->right = cast_tree_opt(tree->right);
    }

    return tree;
}

/* called to attempt to optimize a tree headed by E_CAST.
   tree_opt() calls this bottom-up so there is no need to
   descend, but we do need to loop in case of eliminations. */

struct tree *cast_tree_opt(struct tree *tree)
{
    for (;;) {
        /* useless casts are ... useless:  int x; (int) x; */

        if (cast_useless(tree))
            goto chop;

        /* we can eliminate reinterpretation casts, e.g.,

                        unsigned x; (int) x;
                        long y; (void *) y;     if pointers are 64-bit
                        int z; (void *) z;      if pointers are 32-bit

          except if the child is a fetch of a bitfield: extracting the
          bitfield must be done with the field's native type. */

        if (cast_reinterpret(tree) && !TREE_FIELD_FETCH(tree->child))
            goto chop;

        /* if we're widening a widening cast:

                    char c; ((long) (int) c) == ((long) c);

           or narrowing a widening cast:
        
                    float f; ((float) (double) f) == ((float) f);
                                i.e., just f

           or narrowing a narrowing cast:

                    long l; ((char) (int) l) == ((char) l)

           we can skip the intervening cast. the second example is a
           good illustration of why we need to loop. */

        if (cast_widen(tree) && cast_widen(tree->child))
            goto chop;

        if (cast_narrow(tree) && cast_widen(tree->child))
            goto chop;

        if (cast_narrow(tree) && cast_narrow(tree->child))
            goto chop;

        /* if we're widening a value just to reclass it,

                    char c; ((float) (int) c) == ((float) c);
                    int i; ((double) (long) i) == ((double) i);

           we can skip the intervening cast. (for the first case above, many
           backends will reintroduce the cast because they can't cast sub-int
           values, but the second elimination will be a win.) */

        if (cast_reclass(tree) && cast_widen(tree->child))
            goto chop;

        tree = cast_opt_unary(tree);
        return cast_opt_binary(tree);

chop:
        tree = tree_chop_unary(tree);

        if (tree->op == E_CON)
            con_normalize(TYPE_BITS(&tree->type), &tree->con);
    }
}

/* vi: set ts=4 expandtab: */
