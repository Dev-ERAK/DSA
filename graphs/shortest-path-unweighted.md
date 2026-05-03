# Problem: Shortest Path in an Unweighted Graph

## Category
Graphs / BFS

## Difficulty
Easy

---

## Problem Statement

Given an **unweighted** graph (every edge has the same cost — typically 1) and a source node `s`, find the **shortest distance** (number of edges) from `s` to every other node.

Return `-1` (or `∞`) for nodes that are unreachable from `s`.

### Example

```
Graph:
0 -- 1 -- 2
|    |
3    4

Source: 0
Distances: {0: 0, 1: 1, 3: 1, 2: 2, 4: 2}
```

---

## Clarifying Questions to Ask the Interviewer

1. **Directed or undirected?** Either — the algorithm is the same.
2. **Single-source or all-pairs?** Single-source here. (For all-pairs, run BFS from each node.)
3. **Distance to a specific target, or to all nodes?** Often all; you can early-exit if you only need one.

---

## Approach — BFS

### The Big Idea

In an unweighted graph, **BFS visits nodes in non-decreasing order of distance** from the source. The first time you reach a node, that's the shortest distance.

This is because BFS processes all nodes at distance 1 before any at distance 2, all at distance 2 before any at distance 3, and so on.

### The algorithm

```
dist[s] = 0
push s into queue
while queue is not empty:
    u = pop
    for each neighbor v of u:
        if dist[v] == ∞:                       # not yet reached
            dist[v] = dist[u] + 1
            push v
```

### Walking through

Graph (undirected):
```
0 -- 1 -- 2
|    |
3    4
```

Source = 0.

```
dist = [0, ∞, ∞, ∞, ∞]
queue = [0]

Pop 0. Neighbors: 1, 3.
  dist[1] = 0 + 1 = 1, push 1
  dist[3] = 0 + 1 = 1, push 3
queue = [1, 3]

Pop 1. Neighbors: 0 (already), 2, 4.
  dist[2] = 1 + 1 = 2, push 2
  dist[4] = 1 + 1 = 2, push 4
queue = [3, 2, 4]

Pop 3. Neighbors: 0 (already). Nothing new.

Pop 2. Neighbors: 1 (already). Nothing new.

Pop 4. Neighbors: 1 (already). Nothing new.

Final: dist = [0, 1, 2, 1, 2]   ✓
```

### Why does this work?

Each step from `u` to `v` adds exactly 1 to the distance. The first time we reach `v` is via the shortest path (because BFS processes shorter paths first). Updating `dist[v]` only when it's still `∞` ensures we never overwrite a shorter distance with a longer one.

### Why this fails for weighted graphs

If edges have different weights, the "first to be reached" isn't always the shortest. You need **Dijkstra's algorithm** (a priority queue instead of a plain queue).

---

## Time Complexity

**O(V + E)** — every vertex is dequeued once, every edge examined.

## Space Complexity

**O(V)** — for `dist` and the queue.

---

## Code (Java)

```java
public int[] shortestPath(int s, List<List<Integer>> adj) {
    int n = adj.size();
    int[] dist = new int[n];
    Arrays.fill(dist, -1);                      // -1 means "not reached yet"
    dist[s] = 0;

    Deque<Integer> queue = new ArrayDeque<>();
    queue.offer(s);

    while (!queue.isEmpty()) {
        int u = queue.poll();
        for (int v : adj.get(u)) {
            if (dist[v] == -1) {                 // first time reaching v
                dist[v] = dist[u] + 1;
                queue.offer(v);
            }
        }
    }
    return dist;
}
```

---

## Code (Python)

```python
from collections import deque

class Solution:
    def shortestPath(self, s: int, adj):
        n = len(adj)
        dist = [-1] * n
        dist[s] = 0

        q = deque([s])
        while q:
            u = q.popleft()
            for v in adj[u]:
                if dist[v] == -1:
                    dist[v] = dist[u] + 1
                    q.append(v)

        return dist
```

---

## Edge Cases

| Case | Result |
|---|---|
| Source equals target | Distance 0 |
| Disconnected components | Unreachable nodes stay at -1 |
| Self-loop on source | Ignored (already visited) |
| Graph with 1 node | `dist = [0]` |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Reconstruct the actual path, not just distance."**
   Track parent pointers: when you set `dist[v] = dist[u] + 1`, also record `parent[v] = u`. To rebuild the path to `t`, walk parents from `t` to `s`, then reverse.

2. **"What if some edges are weight 0 and others weight 1?"**
   Use **0/1 BFS** with a deque: push 0-weight edges to the **front**, 1-weight edges to the **back**. O(V + E).

3. **"Weighted graph with positive weights?"**
   **Dijkstra's algorithm** — see `dijkstra.md`.

4. **"Negative weights?"**
   **Bellman-Ford** — handles negative weights and detects negative cycles. O(V × E).

---

## Key Lessons from This Problem

1. **BFS gives shortest paths for free** in unweighted graphs.
2. **The "first time reached"** is always via the shortest path — that's the BFS guarantee.
3. **For weighted graphs, you need Dijkstra** — BFS is no longer correct.
