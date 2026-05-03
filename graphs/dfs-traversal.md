# Problem: DFS Traversal of a Graph

## Category
Graphs / DFS

## Difficulty
Easy

---

## Problem Statement

Given a graph (as an adjacency list) and a starting node, perform a **Depth-First Search (DFS)** and return the order in which nodes are visited.

### What's DFS?

**Depth-First Search** explores as far as possible along one branch **before backtracking**:

- Visit a node.
- Pick an unvisited neighbor; recurse into it.
- Continue diving until you hit a dead end.
- Then backtrack and try other branches.

It's like exploring a maze by always going forward whenever possible, only backing up when stuck.

---

## Clarifying Questions to Ask the Interviewer

1. **Directed or undirected?** Either — the code is identical.
2. **Recursive or iterative?** Both shown.
3. **What if the graph is disconnected?** Loop over all nodes and DFS from each unvisited one for full coverage.

---

## Approach

### Method 1 — Recursive DFS (most natural)

```
dfs(u):
    mark u visited
    output u
    for each neighbor v of u:
        if v not visited: dfs(v)
```

The call stack does the work — each recursive call dives one level deeper.

### Method 2 — Iterative DFS (with explicit stack)

```
push start onto stack
while stack not empty:
    u = pop
    if visited[u]: continue
    mark u visited
    output u
    for each neighbor v of u (in reverse order to match recursive output):
        if not visited[v]: push v
```

The explicit stack mimics what recursion does behind the scenes. Useful for very deep graphs that would blow the recursion stack.

### Walking through an example

Graph:
```
1 -- 2
|    |
3 -- 4
```

Adjacency list:
```
1: [2, 3]
2: [1, 4]
3: [1, 4]
4: [2, 3]
```

DFS from 1 (recursive):

```
dfs(1)
  mark 1, output 1
  neighbor 2:
    dfs(2)
      mark 2, output 2
      neighbor 1: already visited
      neighbor 4:
        dfs(4)
          mark 4, output 4
          neighbor 2: already visited
          neighbor 3:
            dfs(3)
              mark 3, output 3
              all neighbors visited
  neighbor 3: already visited
```

Output: `[1, 2, 4, 3]`. (Order depends on how neighbors are listed.)

Compare with BFS, which would give `[1, 2, 3, 4]` — exploring layer by layer.

---

## Time Complexity

**O(V + E)** — every node visited once, every edge examined.

## Space Complexity

- Recursive: **O(V)** — recursion stack depth (worst case all nodes).
- Iterative: **O(V)** — explicit stack.

---

## Code (Java)

```java
// Recursive
public List<Integer> dfs(int start, List<List<Integer>> adj) {
    int n = adj.size();
    boolean[] visited = new boolean[n];
    List<Integer> order = new ArrayList<>();
    dfsHelper(start, adj, visited, order);
    return order;
}

private void dfsHelper(int u, List<List<Integer>> adj, boolean[] visited, List<Integer> order) {
    visited[u] = true;
    order.add(u);
    for (int v : adj.get(u)) {
        if (!visited[v]) {
            dfsHelper(v, adj, visited, order);
        }
    }
}

// Iterative
public List<Integer> dfsIterative(int start, List<List<Integer>> adj) {
    boolean[] visited = new boolean[adj.size()];
    List<Integer> order = new ArrayList<>();
    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(start);

    while (!stack.isEmpty()) {
        int u = stack.pop();
        if (visited[u]) continue;
        visited[u] = true;
        order.add(u);

        // Push neighbors in REVERSE order so we pop them in the original order
        // (matching the recursive version's output).
        List<Integer> nbrs = adj.get(u);
        for (int i = nbrs.size() - 1; i >= 0; i--) {
            if (!visited[nbrs.get(i)]) stack.push(nbrs.get(i));
        }
    }
    return order;
}
```

---

## Code (Python)

```python
class Solution:
    def dfs(self, start: int, adj):
        n = len(adj)
        visited = [False] * n
        order = []

        def go(u):
            visited[u] = True
            order.append(u)
            for v in adj[u]:
                if not visited[v]:
                    go(v)

        go(start)
        return order

    def dfs_iter(self, start: int, adj):
        visited = [False] * len(adj)
        order = []
        stack = [start]
        while stack:
            u = stack.pop()
            if visited[u]:
                continue
            visited[u] = True
            order.append(u)
            for v in reversed(adj[u]):
                if not visited[v]:
                    stack.append(v)
        return order
```

---

## Edge Cases

| Case | What happens |
|---|---|
| Cycle in the graph | The `visited` set prevents infinite recursion. |
| Disconnected graph | Only the start's component is visited. Loop `for u in 0..n-1: if not visited[u]: dfs(u)` for full coverage. |
| Deeply nested graph | Recursive DFS may stack-overflow — switch to iterative. |
| Single node, no edges | Just outputs `[start]`. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Find connected components."**
   Loop over all nodes; for each unvisited one, DFS to label everything reachable from it. Each DFS run = one component.

2. **"Topological sort."**
   For DAGs (Directed Acyclic Graphs): DFS, and on each node's *post-visit* (after all descendants), prepend it to the output list. The result is a valid topological order.

3. **"Detect cycle in a graph."**
   See `detect-cycle-graph.md`. DFS with three colors (white, gray, black) catches back-edges.

4. **"DFS or BFS — when to use which?"**
   - DFS: cycle detection, topological sort, "is there a path?", strongly connected components.
   - BFS: shortest paths in unweighted graphs, level-order processing.

---

## Key Lessons from This Problem

1. **DFS = stack (or recursion).** Whichever you prefer, the logic is identical.
2. **`visited` is non-negotiable** — without it, cycles cause infinite loops.
3. **DFS vs BFS** is one of the foundational choices in graph algorithms — pick based on what you need.
