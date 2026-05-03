# Problem: Cycle Detection Using DSU (Union-Find)

## Category
Graphs / Union-Find

## Difficulty
Medium

---

## Problem Statement

Given an **undirected** graph (as a list of edges), determine whether it contains a cycle. Use **Disjoint Set Union (DSU)** to do it.

(For directed cycle detection, use DFS with three colors — see `detect-cycle-graph.md`.)

---

## Clarifying Questions to Ask the Interviewer

1. **Undirected only?** Yes — DSU naturally fits. Directed cycles need a different approach.
2. **Self-loops count as cycles?** Yes.
3. **Multigraph (duplicate edges between same pair)?** A duplicate edge between same pair is itself a cycle (length 2). Make sure your input is what you expect.

---

## Approach

### The Big Idea

Walk through the edges one by one. For each edge `(u, v)`:

- Check whether `u` and `v` are **already in the same component** (DSU's `find(u) == find(v)`).
- If yes → this edge would form a cycle. Return `true`.
- If no → `union(u, v)` to merge them.

If you finish all edges without finding such a duplicate-component case, there's no cycle.

### Why this works

DSU's `find` tells you which "group" each node belongs to. Two nodes have the same root **iff** they're connected via some chain of edges already processed. Adding a new edge between two already-connected nodes creates a cycle.

### Walking through

Edges: `(0, 1), (1, 2), (2, 0)`. (Triangle — should detect a cycle.)

```
DSU init: parent = [0, 1, 2]

Edge (0, 1): find(0) = 0, find(1) = 1. Different. Union.
            parent = [1, 1, 2]   (or similar after rank)

Edge (1, 2): find(1) = 1, find(2) = 2. Different. Union.
            parent = [1, 1, 1]

Edge (2, 0): find(2) = 1, find(0) = 1. SAME! → CYCLE detected.
```

Without the third edge, no cycle.

### Comparison to DFS-based detection

- **DSU**: O((V + E) × α(V)) ≈ O(V + E). Conceptually simple. Doesn't visit nodes — just processes edges.
- **DFS**: O(V + E). Same asymptotic time. Better when you also want to know **which** nodes are in the cycle.

DSU is preferred when edges arrive **online** (one at a time) and you want to answer "does this form a cycle?" instantly.

---

## Time Complexity

**O((V + E) × α(V))** ≈ O(V + E) — DSU operations are essentially O(1).

## Space Complexity

**O(V)**.

---

## Code (Java)

```java
public boolean hasCycleDSU(int n, int[][] edges) {
    DSU dsu = new DSU(n);
    for (int[] e : edges) {
        if (!dsu.union(e[0], e[1])) {
            return true;             // already connected → cycle
        }
    }
    return false;
}

// (DSU class as in graphs/union-find.md)
```

---

## Code (Python)

```python
class Solution:
    def has_cycle_dsu(self, n: int, edges) -> bool:
        dsu = DSU(n)
        for u, v in edges:
            if not dsu.union(u, v):
                return True
        return False

# DSU class as in graphs/union-find.md
```

---

## Edge Cases

| Case | Result |
|---|---|
| Self-loop `(u, u)` | `find(u) == find(u)` → cycle detected immediately. |
| Multigraph with duplicate edges | The second copy of the same edge triggers cycle detection. |
| Disconnected graph | DSU handles it naturally; cycle in any component returns true. |
| Empty graph or no edges | No cycle. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why doesn't this work for directed graphs?"**
   In directed graphs, "same component" doesn't imply a cycle. Two trees pointing to a common sink share a "component" but have no cycle. Use DFS three-coloring instead.

2. **"Find the actual cycle nodes, not just whether one exists."**
   When DSU detects the cycle, run a DFS from one endpoint to find the path to the other — that completes the cycle.

3. **"Kruskal's MST uses this same idea."**
   Yes — sort edges, take them in order, but reject any edge that would form a cycle (i.e., where `find(u) == find(v)`).

4. **"How does this compare to DFS in performance?"**
   Same asymptotic complexity. DSU has lower constant factors and fits the "edges-arriving-online" model better; DFS can give you more info (cycle path, traversal order) for free.

---

## Key Lessons from This Problem

1. **DSU is the ideal data structure for "are these connected yet?"** — both querying and joining components.
2. **Cycle in undirected = trying to union two already-same-component nodes.**
3. **Different cycle problems need different tools** — DSU for undirected, DFS three-coloring for directed.
