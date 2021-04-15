/* graph.c - register allocator                         ncc, the new c compiler

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

#include <limits.h>
#include "../common/util.h"
#include "../common/tailq.h"
#include "../common/list.h"
#include "cc1.h"
#include "opt.h"
#include "dead.h"
#include "block.h"
#include "symbol.h"
#include "output.h"
#include "live.h"
#include "loop.h"
#include "webs.h"
#include "stack.h"
#include "target.h"
#include "graph.h"

static bool changed;

/* straight out of your discrete math textbook- our graph is
   a collection of nodes connected by [undirected] edges. we
   keep the nodes sorted in descending order of degree. */

static TAILQ_HEAD(nodes, node) graph = TAILQ_HEAD_INITIALIZER(graph);

#define GRAPH_APPEND(n)             TAILQ_INSERT_TAIL(&graph, n, links)
#define GRAPH_FIRST()               TAILQ_FIRST(&graph)
#define GRAPH_LAST()                TAILQ_LAST(&graph, nodes)
#define GRAPH_NEXT(n)               TAILQ_NEXT(n, links)
#define GRAPH_PREV(n)               TAILQ_PREV(n, nodes, links)
#define GRAPH_FOREACH(n)            TAILQ_FOREACH(n, &graph, links)
#define GRAPH_REMOVE(n)             TAILQ_REMOVE(&graph, n, links)
#define GRAPH_INSERT_BEFORE(bef, n) TAILQ_INSERT_BEFORE(bef, n, links)
#define GRAPH_INSERT_AFTER(aft, n)  TAILQ_INSERT_AFTER(&graph, aft, n, links)

LIST_HEAD(edges, edge);     /* struct edges */

#define EDGES_INIT(es)          LIST_INIT(es)
#define EDGES_PREPEND(es, e)    LIST_INSERT_HEAD(es, e, links)
#define EDGES_FIRST(es)         LIST_FIRST(es)
#define EDGES_REMOVE(e)         LIST_REMOVE(e, links)
#define EDGES_FOREACH(e, es)    LIST_FOREACH(e, es, links)

/* we represent edges as a pair of pointers, one in each node,
   pointing to the other node. when disconnected, a node retains
   its half edges so we know where to reconnect it. */

struct edge
{
    struct node *n;
    LIST_ENTRY(edge) links;
};

struct node
{
    pseudo_reg reg;         /* register as appears in IR */
    pseudo_reg color;       /* assigned machine reg, or PSEUDO_REG_NONE */
    unsigned cost;          /* spill cost of this [pseudo] register */
    int degree;
    struct edges edges;

    TAILQ_ENTRY(node) links;
};

/* we need a stack to store the nodes we disconnect
   from the graph, so we can pop them out LIFO. */

STACK_DECLARE(node_stack, struct node *)

static STACK_DEFINE_PUSH(node_stack, struct node *)
static STACK_DEFINE_POP(node_stack, struct node *)

static struct node_stack node_stack = STACK_INITIALIZER(node_stack);

/* find the node for an IR reg in the graph. if none is found, we
   create a new one if GRAPH_LOOKUP_CREATE, otherwise return 0. */

typedef int graph_lookup_flags;  /* GRAPH_LOOKUP_* */

#define GRAPH_LOOKUP_CREATE     ( 0x00000001 )

static struct node *graph_lookup(pseudo_reg reg, graph_lookup_flags flags)
{
    struct node *n;

    GRAPH_FOREACH(n)
        if (n->reg == reg)
            break;

    if ((n == 0) && (flags & GRAPH_LOOKUP_CREATE)) {
        n = safe_malloc(sizeof(struct node));
        n->reg = reg;
        n->color = PSEUDO_REG_NONE;
        EDGES_INIT(&n->edges);
        GRAPH_APPEND(n);
    }

    return n;
}

/* move a node up the in graph node list
   until it's properly sorted by degree */

#define BUBBLE_UP(n)                                                        \
    do {                                                                    \
        struct node *prev;                                                  \
                                                                            \
        while ((prev = GRAPH_PREV(n)) && (prev->degree < (n)->degree)) {    \
            GRAPH_REMOVE(n);                                                \
            GRAPH_INSERT_BEFORE(prev, n);                                   \
        }                                                                   \
    } while(0)

/* add a half-edge between nodes from and to.
   resort from in the graph list accordingly */

static void graph_add_half_edge(struct node *from, struct node *to)
{
    struct edge *e;

    e = safe_malloc(sizeof(struct edge));
    e->n = to;
    EDGES_PREPEND(&from->edges, e);
    ++(from->degree);

    BUBBLE_UP(from);
}

/* add a [complete] edge between nodes n1
   and n2 if no such edge already exists. */

static void graph_add_edge(struct node *n1, struct node *n2)
{
    struct edge *e;

    EDGES_FOREACH(e, &n1->edges)
        if (e->n == n2)
            return;

    graph_add_half_edge(n1, n2);
    graph_add_half_edge(n2, n1);
}

/* remove the half edge between from and to.
   resort from in the graph list accordingly */

static void graph_remove_half_edge(struct node *from, struct node *to)
{
    struct node *next;
    struct edge *e;

    EDGES_FOREACH(e, &from->edges)
        if (e->n == to)
            break;

    EDGES_REMOVE(e);
    free(e);
    --(from->degree);

    while ((next = GRAPH_NEXT(from)) && (next->degree > from->degree)) {
        GRAPH_REMOVE(from);
        GRAPH_INSERT_AFTER(next, from);
    }
}

/* remove the [complete] edge between n1 and n2. */

static void graph_remove_edge(struct node *n1, struct node *n2)
{
    graph_remove_half_edge(n1, n2);
    graph_remove_half_edge(n2, n1);
}

/* returns TRUE if there is an edge between n1 and n2 */

static bool graph_connected(struct node *n1, struct node *n2)
{
    struct edge *e;

    EDGES_FOREACH(e, &n1->edges)
        if (e->n == n2)
            return TRUE;

    return FALSE;
}

/* remove a node from the graph and discard it */

static void graph_remove_node(struct node *n)
{
    struct edge *e;

    while (e = EDGES_FIRST(&n->edges))
        graph_remove_edge(n, e->n);

    GRAPH_REMOVE(n);
    free(n);
}

/* disconnect a node from the graph - remove the half-edges that
   lead from its neighbors and remove it from the global node list. */

static void graph_disconnect(struct node *n)
{
    struct edge *e;

    EDGES_FOREACH(e, &n->edges)
        graph_remove_half_edge(e->n, n);
    
    GRAPH_REMOVE(n);
}

/* reconnect a node to the graph - put half-edges from its
   neighbors back in and re-insert it into the global node list. */

static void graph_reconnect(struct node *n)
{
    struct node *n2;
    struct edge *e;

    EDGES_FOREACH(e, &n->edges)
        graph_add_half_edge(e->n, n);
        
    GRAPH_APPEND(n);
    BUBBLE_UP(n);
}

/* merge n2 into n1:
   1. move all edges from n2 to n1.
   2. rewrite all IR references to n2->reg to n1->reg
   3. redirect all disconnected nodes pointing to n2 to n1
   4. finally, remove n2 from the graph and discard it */

static void graph_merge(struct node *n1, struct node *n2)
{
    struct node_stack_cell *cell;
    struct edge *e;

    EDGES_FOREACH(e, &n2->edges)
        graph_add_edge(n1, e->n);

    blocks_substitute_reg(n2->reg, n1->reg);

    STACK_FOREACH(cell, &node_stack)
        EDGES_FOREACH(e, &cell->value->edges)
            if (e->n == n2)
                e->n = n1;

    graph_remove_node(n2);
}

/* discard the contents of a graph (by brute force) */

void graph_clear(void)
{
    struct node *n;
    struct edge *e;

    while (n = GRAPH_FIRST()) {
        GRAPH_REMOVE(n);

        while (e = EDGES_FIRST(&n->edges)) {
            EDGES_REMOVE(e);
            free(e);
        }

        free(n);
    }
}

/* add the regs of all of n's neighbors to regs */

static void graph_neighbor_regs(struct node *n, struct regs *regs)
{
    struct edge *e;
    
    EDGES_FOREACH(e, &n->edges)
        REGS_ADD(regs, e->n->reg);
}

/* add the colors of all of n's neighbors to regs */

static void graph_neighbor_colors(struct node *n, struct regs *colors)
{
    struct edge *e;
    
    EDGES_FOREACH(e, &n->edges)
        if (e->n->color != PSEUDO_REG_NONE)
            REGS_ADD(colors, e->n->color);
}

/* determine if an insn is a copy that is eligible for coalescing. if
   so, populates dst and src with the eligible nodes - dst being the
   node that should survive the merge. */

static bool coal_eligible(struct insn *insn, struct node **dst,
                                             struct node **src)
{
    struct node *src_n;
    struct node *dst_n;
    struct symbol *src_sym;
    struct symbol *dst_sym;
    pseudo_reg src_reg;
    pseudo_reg dst_reg;

    if (!insn_copy(insn, &dst_reg, &src_reg))
        return FALSE;

    if (dst_reg == src_reg)
        return FALSE;

    if (!(dst_n = graph_lookup(dst_reg, 0))
      || !(src_n = graph_lookup(src_reg, 0)))
        return FALSE;

    if (graph_connected(dst_n, src_n))
        return FALSE;

    if (target->reg_physical(src_n->reg))
        SWAP(struct node *, dst_n, src_n); /* physical reg always survives */

    if (target->reg_physical(src_n->reg))
        return FALSE;   /* can't coalesce two physical regs */

    if (!target->reg_physical(dst_n->reg)) {    /* both non-physical */
        /* it's possible that the two registers represent symbols
           with different types. if so, the larger survives, so the
           spill region is guaranteed to be large enough for either.
           (physical registers never spill, so we don't check them.) */

        src_sym = reg_symbol(src_n->reg);
        dst_sym = reg_symbol(dst_n->reg);

        if (type_sizeof(&src_sym->type, 0) > type_sizeof(&dst_sym->type, 0))
            SWAP(struct node *, dst_n, src_n);
    }

    *dst = dst_n;
    *src = src_n;

    return TRUE;
}

/* build the interference graph, one block
   at a time, using the live variables data. */

static blocks_iter_ret interf0(struct block *b)
{
    struct regs all = REGS_INITIALIZER(all);
    struct regs interfs = REGS_INITIALIZER(interfs);
    struct node *all_n;
    struct node *interfs_n;
    struct reg *all_r;
    struct reg *interfs_r;

    regs_union(&all, &b->live.def);
    regs_union(&all, &b->live.use);
    
    REGS_FOREACH(all_r, &all) {
        all_n = graph_lookup(all_r->reg, GRAPH_LOOKUP_CREATE);
        live_interf(&b->live, all_r->reg, &interfs);

        REGS_FOREACH(interfs_r, &interfs) {
            if (target->reg_class(all_r->reg)
              == target->reg_class(interfs_r->reg)) {
                interfs_n = graph_lookup(interfs_r->reg, GRAPH_LOOKUP_CREATE);
                graph_add_edge(all_n, interfs_n);
            }
        }

        regs_clear(&interfs);
    }

    regs_clear(&all);
    return BLOCKS_ITER_OK;
}

/* scan the IR and compute spill costs for each pseudo register. we
   use a basic heuristic: 1 point for each predicted load or store,
   multiplied by 100 * loop depth. */

#define COST0(x)                                                        \
    do {                                                                \
        struct regs regs;                                               \
        struct reg *r;                                                  \
        struct node *n;                                                 \
                                                                        \
        REGS_INIT(&regs);                                               \
        insn_##x##_regs(insn, &regs, 0);                                \
                                                                        \
        REGS_FOREACH(r, &regs) {                                        \
            if (target->reg_physical(r->reg))                           \
                continue;                                               \
                                                                        \
            n = graph_lookup(r->reg, 0);                                \
            n->cost += cost;                                            \
        }                                                               \
                                                                        \
        regs_clear(&regs);                                              \
    } while (0)

static blocks_iter_ret cost0(struct block *b)
{
    struct insn *insn;
    int i;
    int cost = 1;

    for (i = 0; i < b->loop.depth; ++i)
        cost *= 10;

    INSNS_FOREACH(insn, &b->insns) {
        COST0(uses);
        COST0(defs);
    }

    return BLOCKS_ITER_OK;
}

/* early coalescing. before we being coloring the graph, we coalesce nodes
   when (theoretically) such merging does not affect the colorability of
   the graph. in our case, we will merge Ni and Nj if the result Nij would
   have less than k neighbors whose degree is greater than or equal to k. */

static blocks_iter_ret early0(struct block *b)
{
    struct regs neighbors = REGS_INITIALIZER(neighbors);
    struct reg *neighbors_r;
    struct node *neighbor_n;
    struct insn *insn;
    struct node *dst_n;
    struct node *src_n;
    int k;
    int count;
    
    INSNS_FOREACH(insn, &b->insns) {
        if (!coal_eligible(insn, &dst_n, &src_n))
            continue;

        count = 0;
        k = target->reg_k(dst_n->reg);
        graph_neighbor_regs(src_n, &neighbors);
        graph_neighbor_regs(dst_n, &neighbors);

        if (REGS_COUNT(&neighbors) >= k) {
            REGS_FOREACH(neighbors_r, &neighbors) {
                neighbor_n = graph_lookup(neighbors_r->reg, 0);

                if (neighbor_n->degree >= k)
                    ++count;
            }
        }

        regs_clear(&neighbors);

        if (count < k) {
            graph_merge(dst_n, src_n);
            changed = TRUE;
        }
    }

    return BLOCKS_ITER_OK;
}

/* later coalescing. if it looks like we might have to spill, we try
   coalescing the partially-completed graph instead, as it may lower
   the degrees of the nodes and perhaps allow us to continue. this is
   a conservative so-called biased approach: we will coalesce nodes if
   we can preserve the color of the dst node in the merged node. */

static blocks_iter_ret later0(struct block *b)
{
    struct node *dst_n;
    struct node *src_n;
    struct insn *insn;
    struct regs colors = REGS_INITIALIZER(colors);
    
    INSNS_FOREACH(insn, &b->insns) {
        if (!coal_eligible(insn, &dst_n, &src_n))
            continue;

        if ((dst_n->color == PSEUDO_REG_NONE)
          || (src_n->color == PSEUDO_REG_NONE))
            continue;   /* only unspilled nodes already in graph */

        graph_neighbor_colors(src_n, &colors);

        if (REGS_CONTAINS(&colors, dst_n->color))
            goto skip;

        graph_merge(dst_n, src_n);
        changed = TRUE;
skip:
        regs_clear(&colors);
    }

    return BLOCKS_ITER_OK;
}

/* precolor the graph- physical registers are given their
   own colors, everyone else is PSEUDO_REG_NONE. */

static void precolor(void)
{
    struct node *n;

    GRAPH_FOREACH(n)
        if (target->reg_physical(n->reg))
            n->color = n->reg;
        else
            n->color = PSEUDO_REG_NONE;
}

/* pick the next node to disconnect from the stack. first we choose nodes
   with degree < k, in lowest degree order first. when that fails, we do
   late coalescing and try again; if still out of luck, we pick a node
   using a basic heuristic from Muchnick, which is the node with minimal
   spill cost divided by its degree. */

static struct node *graph_pick(void)
{
    struct node *n;
    struct node *pick;
    unsigned cost;

    do {
        n = GRAPH_LAST();

        while (n && (n->color != PSEUDO_REG_NONE))
            n = GRAPH_PREV(n);

        if (n && (n->degree < target->reg_k(n->reg)))
            return n;

        changed = FALSE;
        blocks_iter(later0);
    } while (changed);

    pick = 0;
    cost = UINT_MAX;

    GRAPH_FOREACH(n) {
        if (n->color != PSEUDO_REG_NONE)
            continue;

        if ((n->cost / n->degree) < cost) {
            pick = n;
            cost = n->cost / n->degree;
        }
    }

    return pick;
}

/* color the graph */

static void color(void)
{
    struct regs colors = REGS_INITIALIZER(colors);
    struct regs used = REGS_INITIALIZER(used);
    struct reg *color_r;
    struct node *n;

    GRAPH_FOREACH(n)
        if (target->reg_physical(n->reg))
            n->color = n->reg;
        else
            n->color = PSEUDO_REG_NONE;

    while (n = graph_pick()) {
        graph_disconnect(n);
        node_stack_push(&node_stack, n);
    }

    while (!STACK_EMPTY(&node_stack)) {
        n = node_stack_pop(&node_stack);
        graph_reconnect(n);

        target->reg_colors(n->reg, &colors);
        graph_neighbor_colors(n, &used);
        regs_diff(&colors, &used);

        color_r = REGS_FIRST(&colors);

        if (color_r)
            n->color = color_r->reg;

        regs_clear(&colors);
        regs_clear(&used);
    }
}

/* we've encountered an insn that references a spilled pseudo-register.
   find a spill register and invoke the back end to generate spill insns. */

static void spill(struct node *n, struct block *b, struct insn *insn)
{
    struct regs defs = REGS_INITIALIZER(defs);
    struct regs uses = REGS_INITIALIZER(uses);
    struct regs spills = REGS_INITIALIZER(spills);
    struct reg *regs_r;
    pseudo_reg spill_reg;
    struct symbol *sym;

    sym = reg_symbol(n->reg);
    symbol_storage(sym);

    insn_uses_regs(insn, &uses, 0);
    insn_defs_regs(insn, &defs, 0);

    target->reg_spills(n->reg, &spills);
    regs_diff(&spills, &uses);
    regs_diff(&spills, &defs);
    regs_r = REGS_FIRST(&spills);
    spill_reg = regs_r->reg;

    if (REGS_CONTAINS(&uses, n->reg)) {
        target->gen_spill_in(spill_reg, sym);
        insns_insert_before(&b->insns, insn, &current_block->insns);
    }

    if (REGS_CONTAINS(&defs, n->reg)) {
        target->gen_spill_out(spill_reg, sym);
        insns_insert_after(&b->insns, insn, &current_block->insns);
    }

    insn_substitute_reg(insn, n->reg, spill_reg, INSN_SUBSTITUTE_ALL);
        
    regs_clear(&spills);
    regs_clear(&defs);
    regs_clear(&uses);
}

/* rewrite all the pseudo-registers in the block with those
   chosen during coloring, inserting spills as necessary. */

static blocks_iter_ret rewrite0(struct block *b)
{
    struct regs regs = REGS_INITIALIZER(regs);
    struct symbol *sym;
    struct reg *regs_r;
    struct node *n;
    struct insn *insn;

    INSNS_FOREACH(insn, &b->insns) {
        insn_uses_regs(insn, &regs, 0);
        insn_defs_regs(insn, &regs, 0);

        REGS_FOREACH(regs_r, &regs) {
            if (target->reg_physical(regs_r->reg))
                continue;

            n = graph_lookup(regs_r->reg, 0);
            
            if (n->color == PSEUDO_REG_NONE)
                spill(n, b, insn);
            else
                insn_substitute_reg(insn, n->reg, n->color,
                                    INSN_SUBSTITUTE_ALL);
        }

        regs_clear(&regs);

        insn_uses_regs(insn, &func_regs, 0);
        insn_defs_regs(insn, &func_regs, 0);
    }

    return BLOCKS_ITER_OK;
}

static void rewrite(void)
{
    current_block = block_new();
    blocks_iter(rewrite0);
}

void graph_alloc(void)
{
    webs_analyze();

    /* the interference graph is updated incrementally during
       early coalescing, but too conservatively, so we repeat
       to find additional opportunities. */

    do {
        changed = FALSE;

        live_analyze();
        blocks_iter(interf0);
        blocks_iter(early0);

        if (changed) {
            graph_clear();
            nop();
        }
    } while (changed);

    loop_analyze();
    blocks_iter(cost0);
    color();
    rewrite();
    graph_clear();

    coal();
    nop();
    dead();
}

/* vi: set ts=4 expandtab: */
