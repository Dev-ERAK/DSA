# Problem: Detect a Cycle in a Graph

## Category
Graphs / DFS

## Difficulty
Medium

---

## Problem Statement

Given a graph, return `true` if it contains a **cycle** (a path that returns to its starting node).

The algorithm differs based on whether the graph is **directed** or **undirected** — we'll cover both.

### Examples

Undirected, has cycle:
```
1 -- 2
|    |
3 -- 4
```
(There's a cycle 1 → 2 → 4 → 3 → 1.)

Directed, has cycle:
```
1 → 2 → 3
↑       │
└───────┘
```

---

## Clarifying Questions to Ask the Interviewer

1. **Directed or undirected?** They need different algorithms.
2. **Self-loops count as cycles?** Yes, by convention.
3. **Disconnected graph?** Iterate over all components.

---

## Approach

### Undirected — DFS with Parent Tracking

In an undirected graph, every edge `(u, v)` is also `(v, u)`. So when DFS visits `v` from `u`, it'll naturally see `u` again — but that's just the edge we came in on, **not** a cycle.

**Rule:** while DFSing from `u` to `v`, track `u` as `v`'s "parent." If we visit a neighbor `w` that's **already visited but isn't the parent**, we've found a cycle.

```
dfs(u, parent):
    mark u visited
    for each neighbor v of u:
        if not visited[v]:
            if dfs(v, u): return true       # cycle found below
        elif v != parent:
            return true                     # cycle!
    return false
```

### Directed — DFS with Three Colors

In a directed graph, a cycle = a **back edge** (an edge from a node to one of its ancestors in the DFS tree).

Track each node in one of three states:

- **WHITE (0)** — not yet visited.
- **GRAY (1)** — currently in the DFS recursion stack (we're exploring through it now).
- **BLACK (2)** — fully processed; we're done with this node.

If during DFS you see a **GRAY** neighbor, that's a back edge → cycle.

```
dfs(u):
    color[u] = GRAY
    for each neighbor v of u:
        if color[v] == GRAY: return true        # back edge — cycle!
        if color[v] == WHITE and dfs(v): return true
    color[u] = BLACK
    return false
```

### Why three colors for directed graphs?

Because once you're done exploring a node (BLACK), revisiting it from elsewhere isn't a cycle — it's just a "cross edge" or "forward edge." Only revisiting a node still in your active DFS stack (GRAY) means you've gone in a circle.

### Walking through an undirected example

Graph: `1 -- 2 -- 3`

```
dfs(1, parent=-1)
  visit 1, mark visited
  neighbor 2: not visited → dfs(2, parent=1)
    visit 2, mark visited
    neighbor 1: visited, but it's the parent → ignore
    neighbor 3: not visited → dfs(3, parent=2)
      visit 3, mark visited
      neighbor 2: visited, but it's the parent → ignore
      return false
    return false
  return false
No cycle. ✓
```

Now add an edge `1 -- 3`:
```
dfs(1, -1)
  visit 1
  neighbor 2: dfs(2, 1)
    visit 2
    neighbor 1: parent, ignore
    neighbor 3: dfs(3, 2)
      visit 3
      neighbor 1: visited, NOT parent → CYCLE
```

### Walking through a directed example

Graph: `1 → 2 → 3 → 1`

```
dfs(1)
  color[1] = GRAY
  neighbor 2: dfs(2)
    color[2] = GRAY
    neighbor 3: dfs(3)
      color[3] = GRAY
      neighbor 1: color[1] = GRAY → CYCLE ✓
```

---

## Time Complexity

**O(V + E)** in both cases.

## Space Complexity

**O(V)** — for the visited/color array and recursion stack.

---

## Code (Java) — Undirected

```java
public boolean hasCycleUndirected(int n, List<List<Integer>> adj) {
    boolean[] visited = new boolean[n];

    for (int u = 0; u < n; u++) {
        if (!visited[u] && dfsU(u, -1, adj, visited)) {
            return true;
        }
    }
    return false;
}

private boolean dfsU(int u, int parent, List<List<Integer>> adj, boolean[] visited) {
    visited[u] = true;
    for (int v : adj.get(u)) {
        if (!visited[v]) {
            if (dfsU(v, u, adj, visited)) return true;
        } else if (v != parent) {
            return true;
        }
    }
    return false;
}
```

### Directed

```java
public boolean hasCycleDirected(int n, List<List<Integer>> adj) {
    int[] color = new int[n];                   // 0 white, 1 gray, 2 black

    for (int u = 0; u < n; u++) {
        if (color[u] == 0 && dfsD(u, adj, color)) {
            return true;
        }
    }
    return false;
}

private boolean dfsD(int u, List<List<Integer>> adj, int[] color) {
    color[u] = 1;                                // GRAY
    for (int v : adj.get(u)) {
        if (color[v] == 1) return true;          // back edge → cycle
        if (color[v] == 0 && dfsD(v, adj, color)) return true;
    }
    color[u] = 2;                                // BLACK
    return false;
}
```

---

## Code (Python)

```python
class Solution:
    # Undirected
    def has_cycle_undirected(self, n: int, adj) -> bool:
        visited = [False] * n

        def dfs(u, parent):
            visited[u] = True
            for v in adj[u]:
                if not visited[v]:
                    if dfs(v, u):
                        return True
                elif v != parent:
                    return True
            return False

        for u in range(n):
            if not visited[u] and dfs(u, -1):
                return True
        return False

    # Directed
    def has_cycle_directed(self, n: int, adj) -> bool:
        color = [0] * n        # 0 white, 1 gray, 2 black

        def dfs(u):
            color[u] = 1
            for v in adj[u]:
                if color[v] == 1:
                    return True
                if color[v] == 0 and dfs(v):
                    return True
            color[u] = 2
            return False

        for u in range(n):
            if color[u] == 0 and dfs(u):
                return True
        return False
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty graph | No cycle |
| Single node, no edges | No cycle |
| Self-loop on a node | Cycle |
| Disconnected graph | Iterate every component |
| Multigraph (two edges between same pair, undirected) | The simple parent check fails — track edge IDs instead |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Find the actual nodes of the cycle, not just whether one exists."**
   Track parent pointers during DFS. When you detect a cycle, walk back from the current node via parents until you reach the duplicate.

2. **"What's a topological sort?"**
   For directed graphs without cycles (DAGs), a topological sort is an ordering where every edge `(u, v)` has `u` before `v`. **Kahn's algorithm** uses BFS with in-degrees; DFS post-order also works.

3. **"Detect cycle using Union-Find for undirected graphs."**
   See `cycle-detection-dsu.md`. For each edge, if both endpoints are already in the same component, adding this edge creates a cycle.

4. **"What's a strongly connected component (SCC)?"**
   In a directed graph, an SCC is a set of nodes where each is reachable from every other. **Tarjan's** or **Kosaraju's** algorithm finds them.

---

## Key Lessons from This Problem

1. **Directed and undirected cycle detection are different beasts.** Don't confuse them.
2. **Three-color DFS** is the canonical approach for directed cycles.
3. **Parent tracking** handles the "edge back to where we came from" issue in undirected graphs.
