/* sign.c - signedness-related optimizations            ncc, the new c compiler

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
#include "type.h"
#include "tree.h"
#include "sign.h"

/* ultimately, we should formulate some kind of data-flow analysis similar
   to constant propagation that tracks the sign/zero-extended state of regs,
   and uses this information to delete unnecessary additional extensions. at
   the moment this is handled (such as it is) ad hoc in the target code.

   for now, the only sign-related optimization we do is on trees, where we
   rewrite unsigned comparisons against zero: tests for equality are slightly
   cheaper than ordered comparisons. for x of any unsigned type,

                        (x >  0) == (0 <  x) == (x != 0)        (1)
                        (x <= 0) == (0 >= x) == (x == 0)        (2)
                        (x <  0) == (0 >  x) ==    0            (3)
                        (x >= 0) == (0 <= x) ==    1            (4)

   (2) is suspicious, (3) and (4) are certainly bogus. we issue warnings for
   these last three. */

struct tree *sign_tree_opt(struct tree *tree)
{
    switch (tree->op)
    {
    case E_GT:
    case E_GTEQ:
    case E_LT:
    case E_LTEQ:    break;

    default:        return tree;
    }

    if (!TYPE_DISCRETE(&tree->left->type) || TYPE_SIGNED(&tree->left->type))
        return tree;

    if (tree_zero(tree->left)) {            /* normalize: zero on right */
        tree_commute(tree);
    
        switch (tree->op)
        {
        case E_GT:      tree->op = E_LT; break;
        case E_GTEQ:    tree->op = E_LTEQ; break;
        case E_LT:      tree->op = E_GT; break;
        case E_LTEQ:    tree->op = E_GTEQ; break;
        }
    }

    if (!tree_zero(tree->right))
        return tree;

    switch (tree->op)
    {
    case E_GT:      tree->op = E_NEQ; break;        /* (x > 0) == (x != 0) */

    case E_LT:      tree_free(tree);                /* (x < 0) == 0 */
                    tree = tree_i(T_INT, 0);
                    error(WARNING, "unsigned comparison always false");
                    break;

    case E_GTEQ:    tree_free(tree);                /* (x >= 0) == 1 */
                    tree = tree_i(T_INT, 1);
                    error(WARNING, "unsigned comparison always true");
                    break;

    case E_LTEQ:    tree->op = E_EQ;                /* (x <= 0) == (x == 0) */
                    error(WARNING, "suspicious unsigned comparison (use ==)");
                    break;
    }

    return tree;
}

/* vi: set ts=4 expandtab: */
