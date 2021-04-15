/* gen.c - expression code generator                    ncc, the new c compiler

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
#include "symbol.h"
#include "target.h"
#include "insn.h"
#include "block.h"
#include "field.h"
#include "tree.h"
#include "cast.h"
#include "gen.h"

/* given an integer leaf, generate a branch
   to false if it's zero, or true otherwise. */

void gen_branch(struct tree *tree, struct block *true, struct block *false)
{
    EMIT(insn_new(I_CMP, operand_leaf(tree),
                         operand_i(T_INT, 0, 0)));

    block_add_successor(current_block, CC_Z, false);
    block_add_successor(current_block, CC_NZ, true);
}

/* helper for gen_args0(): build the frame address of
   a variable into a temp reg, and return that. */

static pseudo_reg gen_frame(struct symbol *sym)
{
    pseudo_reg reg;

    symbol_storage(sym);
    reg = symbol_temp_reg(target->ptr_uint);

    EMIT(insn_new(I_FRAME, operand_reg(target->ptr_uint, reg),
                           operand_i(T_INT, sym->offset, 0)));

    return reg;
}

/* generate the sequence to move arguments from where they are passed
   by the caller to where we expect them to be. this must happen after
   target-independent optimization (but before target code generation)
   because these insns reference physical registers.
  
   for normal arguments, we transfer the value from its arrival reg or
   stack frame location to the appropriate pseudoregister. volatile and
   aliased arguments are spilled out if they arrived in registers. */

static void gen_args0(struct symbol *sym)
{
    pseudo_reg reg;

    if (TYPE_STRUN(&sym->type))     /* only applies to scalars */
        return;

    if (!SYMBOL_ALIASED(sym) && !TYPE_VOLATILE(&sym->type)) {
        if (sym->arg_reg != PSEUDO_REG_NONE) {
            EMIT(insn_new(I_MOVE,
                          operand_reg(TYPE_BITS(&sym->type), symbol_reg(sym)),
                          operand_reg(TYPE_BITS(&sym->type), sym->arg_reg)));
        } else {
            reg = gen_frame(sym);

            EMIT(insn_new(I_LOAD,
                          operand_reg(TYPE_BITS(&sym->type), symbol_reg(sym)),
                          operand_reg(target->ptr_uint, reg)));
        }
    } else {
        if (sym->arg_reg != PSEUDO_REG_NONE) {
            reg = gen_frame(sym);

            EMIT(insn_new(I_STORE,
                          operand_reg(target->ptr_uint, reg),
                          operand_reg(TYPE_BITS(&sym->type), sym->arg_reg)));
        }
    }
}

void gen_args(void)
{
    struct cessor *succ;

    succ = block_always_successor(entry_block);
    current_block = block_split_edge(entry_block, succ);
    scope_walk_args(gen_args0);
}

/* gen_load and gen_store wrap the generation of all I_LOAD/I_STOREs 
   to ensure that accesses through volatile pointers are so marked */

static void gen_load(struct symbol *dst, struct tree *ptr)
{
    struct insn *insn;

    insn = insn_new(I_LOAD, operand_sym(dst), operand_leaf(ptr));

    if (TYPE_VOLATILE_PTR(&ptr->type))
        insn->flags |= INSN_FLAG_VOLATILE;
    
    EMIT(insn);
}

static void gen_store(struct tree *ptr, struct tree *src)
{
    struct insn *insn;

    insn = insn_new(I_STORE, operand_leaf(ptr), operand_leaf(src));

    if (TYPE_VOLATILE_PTR(&ptr->type))
        insn->flags |= INSN_FLAG_VOLATILE;

    EMIT(insn);
}

/* an E_ADDROF can ultimately only have E_SYM children, and only E_SYMs
   that refer to stack-allocated variables. the front-end will generate
   E_ADDROF with an E_CALL child in the case of a function that returns
   a struct; we must call gen() on the child because of this case, but
   it reduces to the E_SYM of the local temporary.

   other semantically-possible children of E_ADDROF - global/static E_SYMs
   and E_FETCH - are eliminated by tree_addrof() during construction. */

static struct tree *gen_addrof(struct tree *tree)
{
    struct symbol *sym;
    struct symbol *temp;

    tree->child = gen(tree->child, 0);
    temp = symbol_temp(&tree->type);
    sym = tree->child->sym;
    symbol_storage(sym);

    if (sym->ss & S_LOCAL) {
        sym->ss &= ~S_LOCAL;
        sym->ss |= S_AUTO;
    }

    EMIT(insn_new(I_FRAME, operand_sym(temp),
                           operand_i(T_INT, sym->offset, 0)));

    tree_free(tree);
    return tree_sym(temp);
}

/* block assignment is simple, since the
   hard work is done by the back ends. */

static struct tree *gen_blkasg(struct tree *tree)
{
    size_t bytes;

    tree->left = gen(tree->left, 0);
    tree->right = gen(tree->right, 0);
    bytes = type_sizeof(&tree->type, TYPE_SIZEOF_TARGET);

    EMIT(insn_new(I_BLKCOPY, operand_leaf(tree->left),
                             operand_leaf(tree->right),
                             operand_i(target->ptr_uint, bytes, 0)));

    tree = tree_chop_binary(tree);
    return tree;
}

/* extract the bitfield of the given size and shift in sym back
   into sym. there are better sequences than shift-shift in some
   special cases (e.g., an unsigned field at offset 0 can simply
   be masked off) but we'll deal with that later. */

static void extract(struct symbol *sym, int size, int shift)
{
    int host_bits;
    int left_shift;
    int right_shift;

    host_bits = type_sizeof(&sym->type, 0);
    host_bits *= BITS_PER_BYTE;

    left_shift = host_bits - (size + shift);
    right_shift = host_bits - size;

    EMIT(insn_new(I_SHL, operand_sym(sym),
                         operand_sym(sym),
                         operand_i(T_INT, left_shift, 0)));

    EMIT(insn_new(I_SHR, operand_sym(sym),
                         operand_sym(sym),
                         operand_i(T_INT, right_shift, 0)));
}

/* process an E_FETCH. this is usually a simple issue of a load
   to a temporary, but there are two wrinkles: bit field fetches
   must have their values extracted, and fetches of struct types
   are no-ops (yielding void trees). if mode is GEN_FETCH_RETAIN,
   the original tree is not tossed: gen_compound() needs to keep
   its left-hand side intact for gen_asg(). (a bit of a hack.) */

typedef int gen_fetch_mode;     /* GEN_FETCH_* */

#define GEN_FETCH_NORMAL    0
#define GEN_FETCH_RETAIN    1

static struct tree *gen_fetch(struct tree *tree, gen_fetch_mode mode)
{
    struct symbol *temp;

    temp = 0;
    tree->child = gen(tree->child, 0);

    if (!TYPE_STRUN(&tree->type)) {
        temp = symbol_temp(&tree->type);

        gen_load(temp, tree->child);

        if (TYPE_FIELD_PTR(&tree->child->type))
            extract(temp, TYPE_FIELD_PTR_SIZE(&tree->child->type),
                          TYPE_FIELD_PTR_SHIFT(&tree->child->type));

    }

    if (mode != GEN_FETCH_RETAIN)
        tree_free(tree);

    if (temp)
        return tree_sym(temp);
    else
        return tree_v();
}

/* a bitfield (described by size and shift) with the given value
   must be inserted into the word at ptr. this function does the 
   loading/masking/shifting and rewrites the value tree so the
   caller can do the store. just as with extract(), there is some
   improvement to be made here. optimizing bitfield manipulation
   is not high on the list, but this should be revisited at some
   point, once it becomes clear what later passes can't improve.

   this function relies on the compiler implementation to shift in
   zeros from the left when right-shifting unsigned values. */

static struct tree *insert(struct tree *ptr, struct tree *value,
                           int size, int shift)
{
    struct symbol *new;
    struct symbol *shifted;
    unsigned long mask;
    unsigned long mask_mask;
    int host_bits;

    host_bits = type_sizeof(&value->type, 0);
    host_bits *= BITS_PER_BYTE;

    mask_mask = -1L;
    mask_mask >>= (sizeof(mask) * BITS_PER_BYTE) - host_bits;

    mask = field_mask(size, shift);

    new = symbol_temp(&value->type);
    shifted = symbol_temp(&value->type);

    EMIT(insn_new(I_SHL, operand_sym(shifted),
                         operand_leaf(value),
                         operand_i(T_INT, shift, 0)));

    EMIT(insn_new(I_AND, operand_sym(shifted),
                         operand_sym(shifted),
                         operand_i(TYPE_BASE(&value->type), mask, 0)));

    mask = ~mask;
    mask &= mask_mask;

    gen_load(new, ptr);

    EMIT(insn_new(I_AND, operand_sym(new),
                         operand_sym(new),
                         operand_i(TYPE_BASE(&value->type), mask, 0)));

    EMIT(insn_new(I_OR, operand_sym(new),
                        operand_sym(new),
                        operand_sym(shifted)));

    tree_free(value);
    return tree_sym(new);
}

/* handle scalar assignments. like gen_fetch(), we must handle the
   messy details of bitfields (and they're even worse for stores).
   this function is not just for E_ASG trees, but is a helper for
   gen_compound(), which adjusts the right side first.

   the value of an assignment operation is the value of the left
   operand after assignment; we yield the value of the right side
   instead, which we've already computed into a leaf anyway. this
   means we need to fix up the right when bitfields are involved,
   but avoids issuing a spurious load when the left-hand-side is
   an E_FETCH through a volatile pointer: such a load could not be
   optimized out (as it is marked INSN_FLAG_VOLATILE) and is not
   correct behavior (imagine the target is a hardware register). */

static struct tree *gen_asg(struct tree *tree)
{
    struct symbol *temp;

    tree->right = gen(tree->right, 0);

    if (tree->left->op == E_FETCH) {
        tree->left->child = gen(tree->left->child, 0);

        if (TYPE_FIELD_PTR(&tree->left->child->type))
            tree->right = insert(tree->left->child, tree->right,
                             TYPE_FIELD_PTR_SIZE(&tree->left->child->type),
                             TYPE_FIELD_PTR_SHIFT(&tree->left->child->type));

        gen_store(tree->left->child, tree->right);

        if (TYPE_FIELD_PTR(&tree->left->child->type)) {
            temp = symbol_temp(&tree->right->type);

            EMIT(insn_new(I_MOVE, operand_sym(temp),
                                  operand_leaf(tree->right)));

            extract(temp, TYPE_FIELD_PTR_SIZE(&tree->left->child->type),
                          TYPE_FIELD_PTR_SHIFT(&tree->left->child->type));

            tree_free(tree->right);
            tree->right = tree_sym(temp);
        }
    } else {
        EMIT(insn_new(I_MOVE, operand_leaf(tree->left),
                              operand_leaf(tree->right)));
    }

    tree_commute(tree);
    tree = tree_chop_binary(tree);
    return gen(tree, 0);
}

/* there are two special cases to look out for with casts:

   1. casts to void: no-op, but convert to void nodes
   2. discrete downcasts: since the values do not change (we
      merely disregard the upper bits), we emit I_MOVE insns
      instead of I_CASTs, taking care to rewrite the source
      type, since src/dst must have the same type. */

static struct tree *gen_cast(struct tree *tree)
{
    struct insn *insn;
    struct symbol *temp;

    tree->child = gen(tree->child, 0);

    if (TYPE_VOID(&tree->type)) {
        tree_free(tree);
        return tree_v();
    } else {
        temp = symbol_temp(&tree->type);

        if (TYPE_DISCRETE(&tree->type) && cast_narrow(tree)) {
            insn = insn_new(I_MOVE, operand_sym(temp),
                                    operand_leaf(tree->child));

            insn->src1->ts = insn->dst->ts;
            EMIT(insn);
        } else
            EMIT(insn_new(I_CAST, operand_sym(temp),
                                  operand_leaf(tree->child)));
    
        tree_free(tree);
        return tree_sym(temp);
    }
}

/* unary and binary operators are trivial: map a tree_op to
   an insn_op, calculate into a temporary and return that. */

static struct tree *gen_unary(struct tree *tree)
{
    struct symbol *temp;
    insn_op op;

    op = (tree->op == E_NEG) ? I_NEG : I_COM;
    
    tree->child = gen(tree->child, 0);
    temp = symbol_temp(&tree->type);

    EMIT(insn_new(op, operand_sym(temp),
                      operand_leaf(tree->child)));

    tree_free(tree);
    return tree_sym(temp);
}

static insn_op bin[] =      /* must match E_BIN_* in tree.h */
{
    I_XOR,  I_DIV,  I_MUL,  I_ADD,  I_SUB,
    I_SHR,  I_SHL,  I_AND,  I_OR,   I_MOD
};

static struct tree *gen_binary(struct tree *tree)
{
    struct symbol *temp;

    tree->left = gen(tree->left, 0);
    tree->right = gen(tree->right, 0);

    temp = symbol_temp(&tree->type);

    EMIT(insn_new(bin[E_BIN_IDX(tree->op)], operand_sym(temp),
                                            operand_leaf(tree->left),
                                            operand_leaf(tree->right)));

    tree_free(tree);
    return tree_sym(temp);
}

/* compound operators are somewhat tricky, for two reasons. first, the
   left side can only be evaluated once, necessitating the dirty hack
   to gen_fetch(). secondly, in the case of E_POST, our result is the
   value before the operation took place, rather than after. luckily,
   we can pawn off the bitfield chaos to gen_fetch() and gen_asg(). */

static struct tree *gen_compound(struct tree *tree)
{
    struct tree *temp;
    struct tree *orig;
    int post;

    post = (tree->op == E_POST);
    orig = tree_sym(symbol_temp(&tree->type));
    tree->right = gen(tree->right, 0);

    if (tree->left->op == E_SYM) {
        EMIT(insn_new(I_MOVE, operand_leaf(orig),
                              operand_leaf(tree->left)));

        EMIT(insn_new(bin[E_BIN_IDX(tree->op)], operand_leaf(tree->left),
                                                operand_leaf(tree->left),
                                                operand_leaf(tree->right)));
    
        tree = tree_chop_binary(tree);
    } else {
        temp = gen_fetch(tree->left, GEN_FETCH_RETAIN);

        EMIT(insn_new(I_MOVE, operand_leaf(orig),
                              operand_leaf(temp)));

        EMIT(insn_new(bin[E_BIN_IDX(tree->op)], operand_leaf(temp),
                                                operand_leaf(temp),
                                                operand_leaf(tree->right)));

        tree_free(tree->right);
        tree->right = temp;
        tree = gen_asg(tree);
    }

    if (post) {
        tree_free(tree);
        return orig;
    } else {
        tree_free(orig);
        return tree;
    }
}

/* the real work in the relational operators is determining which
   condition_codes correspond to the relation in question. */

static struct tree *gen_rel(struct tree *tree)
{
    struct symbol *temp;
    condition_code cc;
    
    temp = symbol_temp(&tree->type);
    
    tree->left = gen(tree->left, 0);
    tree->right = gen(tree->right, 0);

    EMIT(insn_new(I_CMP, operand_leaf(tree->left),
                         operand_leaf(tree->right)));

    switch (tree->op)
    {
    case E_EQ:   cc = CC_Z; break;
    case E_NEQ:  cc = CC_NZ; break;
    case E_GT:   cc = (TYPE_SIGNED(&tree->left->type)) ? CC_G : CC_A; break;
    case E_LT:   cc = (TYPE_SIGNED(&tree->left->type)) ? CC_L : CC_B; break;
    case E_GTEQ: cc = (TYPE_SIGNED(&tree->left->type)) ? CC_GE : CC_AE; break;
    case E_LTEQ: cc = (TYPE_SIGNED(&tree->left->type)) ? CC_LE : CC_BE; break;
    }

    EMIT(insn_new(I_SET_CC_FROM_CC(cc), operand_sym(temp)));

    tree_free(tree);
    return tree_sym(temp);
}

/* we don't have enough context to do a good job with the logical
   operators. what we emit here is a big mess that is (hopefully)
   optimized down to something sane. */

static struct tree *gen_log(struct tree *tree)
{
    struct symbol *temp;
    struct block *right_block;
    struct block *true_block;
    struct block *false_block;
    struct block *join_block;

    temp = symbol_temp(&tree->type);

    right_block = block_new();
    true_block = block_new();
    false_block = block_new();
    join_block = block_new();

    BLOCK_APPEND_INSN(true_block, insn_new(I_MOVE, operand_sym(temp),
                                                   operand_i(T_INT, 1, 0)));

    BLOCK_APPEND_INSN(false_block, insn_new(I_MOVE, operand_sym(temp),
                                                    operand_i(T_INT, 0, 0)));

    block_add_successor(true_block, CC_ALWAYS, join_block);
    block_add_successor(false_block, CC_ALWAYS, join_block);

    tree->left = gen(tree->left, 0);
    
    if (tree->op == E_LOR) 
        gen_branch(tree->left, true_block, right_block);
    else
        gen_branch(tree->left, right_block, false_block);

    current_block = right_block;
    tree->right = gen(tree->right, 0);
    gen_branch(tree->right, true_block, false_block);
    current_block = join_block;

    tree_free(tree);
    return tree_sym(temp);
}

/* like the logical operators, the output for the conditional
   operator is convoluted, but the optimizer will clean it up. */

static struct tree *gen_quest(struct tree *tree)
{
    struct symbol *temp;
    struct block *true_block;
    struct block *false_block;
    struct block *join_block;
    
    if (TYPE_VOID(&tree->type))
        temp = 0;
    else
        temp = symbol_temp(&tree->type);
    
    true_block = block_new();
    false_block = block_new();
    join_block = block_new();

    tree->left = gen(tree->left, 0);
    gen_branch(tree->left, true_block, false_block);

    current_block = true_block;
    tree->right->left = gen(tree->right->left, 0);
    
    if (temp)
        EMIT(insn_new(I_MOVE, operand_sym(temp),
                              operand_leaf(tree->right->left)));

    block_add_successor(current_block, CC_ALWAYS, join_block);

    current_block = false_block;
    tree->right->right = gen(tree->right->right, 0);

    if (temp)
        EMIT(insn_new(I_MOVE, operand_sym(temp),
                              operand_leaf(tree->right->right)));

    block_add_successor(current_block, CC_ALWAYS, join_block);

    current_block = join_block;
    tree_free(tree);

    return temp ? tree_sym(temp) : tree_v();
}

/* commas are easy: just evaluate them
   in the right order. */

static struct tree *gen_comma(struct tree *tree)
{
    tree->left = gen(tree->left, 0);
    tree_commute(tree);
    tree = tree_chop_binary(tree);
    return gen(tree, 0);
}

/* generating calls has some complexity because of struct arguments, and
   because we need to keep the I_ARGUMENTs contiguous with the I_CALL. */

static struct tree *gen_call(struct tree *tree)
{
    struct insns arg_insns;
    struct insn *call_insn;
    struct symbol *temp;
    struct tree *arg_tree;
    struct tree *ret_tree;
    size_t bytes;
    insn_op arg_op;

    INSNS_INIT(&arg_insns);
    tree->child = gen(tree->child, 0);
    arg_op = TYPE_VARIADIC_FUNC_PTR(&tree->child->type) ? I_VARG : I_ARG;

    if (TYPE_VOID(&tree->type))
        ret_tree = tree_v();
    else {
        temp = symbol_temp(&tree->type);
        ret_tree = tree_sym(temp);
    }

    if (TYPE_STRUN(&tree->type)) {
        arg_tree = tree_sym(temp);
        arg_tree = tree_addrof(arg_tree);
        FOREST_PREPEND(&tree->args, arg_tree);
    }

    while (arg_tree = FOREST_LAST(&tree->args)) {
        FOREST_REMOVE(&tree->args, arg_tree);

        if (TYPE_STRUN(&arg_tree->type)) {
            bytes = type_sizeof(&arg_tree->type, 0);
            arg_tree = tree_addrof(arg_tree);
            arg_tree = gen(arg_tree, 0);

            insn_prepend(&arg_insns,
                insn_new(I_BLKARG, operand_leaf(arg_tree),
                                   operand_i(target->ptr_uint, bytes, 0)));
        } else {
            arg_tree = gen(arg_tree, 0);
            insn_prepend(&arg_insns, insn_new(arg_op,
                                              operand_leaf(arg_tree)));
        }

        tree_free(arg_tree);
    }

    if (TYPE_VOID(&tree->type) || TYPE_STRUN(&tree->type))
        EMIT(call_insn = insn_new(I_CALL, 0, operand_leaf(tree->child)));
    else
        EMIT(call_insn = insn_new(I_CALL, operand_sym(temp),
                                          operand_leaf(tree->child)));

    insns_insert_before(&current_block->insns, call_insn, &arg_insns);

    tree_free(tree);
    return ret_tree;
}

/* expression tree code generation: given a tree,
   reduce it to a leaf: E_NONE, E_CON or E_SYM. */

struct tree *gen(struct tree *tree, gen_flags flags)
{
    if (flags & GEN_FLAG_ROOT) {
        tree = tree_rewrite_volatile(tree);
        tree = tree_opt(tree);
        tree = tree_simplify(tree);

        if (debug_flag_e)
            tree_debug(tree, 0);
    }

    switch (tree->op)
    {
    case E_ADDROF:      tree = gen_addrof(tree); break;
    case E_BLKASG:      tree = gen_blkasg(tree); break;
    case E_CAST:        tree = gen_cast(tree); break;
    case E_FETCH:       tree = gen_fetch(tree, GEN_FETCH_NORMAL); break;
    case E_QUEST:       tree = gen_quest(tree); break;
    case E_COMMA:       tree = gen_comma(tree); break;
    case E_CALL:        tree = gen_call(tree); break;

    case E_CON:     case E_NONE:    case E_SYM:

            break;

    case E_NEG:     case E_COM:

            tree = gen_unary(tree);
            break;

    case E_ADD:     case E_SUB:     case E_MUL:     case E_DIV:
    case E_MOD:     case E_AND:     case E_OR:      case E_XOR:
    case E_SHR:     case E_SHL:

            tree = gen_binary(tree);
            break;

    case E_ASG:

            tree = gen_asg(tree);
            break;

    case E_ADDASG:  case E_SUBASG:  case E_MULASG:  case E_DIVASG:
    case E_MODASG:  case E_ANDASG:  case E_ORASG:   case E_XORASG:
    case E_SHRASG:  case E_SHLASG:  case E_POST:

            tree = gen_compound(tree);
            break;

    case E_GT:      case E_GTEQ:    case E_LT:      case E_LTEQ:
    case E_EQ:      case E_NEQ:

            tree = gen_rel(tree);
            break;

    case E_LOR:     case E_LAND:

            tree = gen_log(tree);
            break;
    }
    
    if (flags & GEN_FLAG_DISCARD) {
        tree_free(tree);
        return 0;
    } else
        return tree;
}

/* vi: set ts=4 expandtab: */
