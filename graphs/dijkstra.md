# Problem: Dijkstra's Shortest Path Algorithm

## Category
Graphs / Greedy

## Difficulty
Medium

---

## Problem Statement

Given a **weighted graph** with **non-negative edge weights** and a source node `s`, find the **shortest distance** from `s` to every other node.

**Weighted** means each edge has a "cost" or "distance." Examples:
- Roads with travel times.
- Network links with latencies.
- Game maps with terrain costs.

---

## Clarifying Questions to Ask the Interviewer

1. **Are edge weights non-negative?** Yes — Dijkstra **requires** this. For negative edges, use Bellman-Ford.
2. **Directed or undirected?** Either; the algorithm is the same.
3. **Single source-target, or all-pairs from one source?** This problem: single source, all targets. Easy to early-exit if you only need one target.

---

## Approach — Dijkstra with a Min-Heap

### The intuition

Dijkstra is a **greedy** algorithm. At each step, it picks the **unvisited node closest to the source** and marks it as "done" — its shortest distance is finalized. Then it updates the distances of that node's neighbors.

It's like exploring a city by always heading to the next unvisited spot that's closest to home.

### Why it works (with non-negative weights)

When we pick the nearest unvisited node `u`, no other path to `u` can be shorter. Why? Any other path goes through some unvisited node `x` first, but `x` is **at least as far** as `u`. Since edges have non-negative weight, the total path through `x` is at least as long. So `u`'s current distance is final.

This logic **breaks with negative weights** — see `dijkstra-negative.md`.

### The algorithm with a min-heap

```
dist = array of infinity, dist[s] = 0
push (0, s) into min-heap

while heap is not empty:
    (d, u) = pop                    # node with smallest tentative distance
    if d > dist[u]: continue        # stale entry — skip
    for each (v, weight) in adj[u]:
        new_dist = dist[u] + weight
        if new_dist < dist[v]:
            dist[v] = new_dist
            push (new_dist, v)
```

The "stale entry" check is important: a node might be pushed multiple times (each push = a tentative distance discovery). When we pop, we ignore entries that have been improved since.

### Walking through

Graph (directed):
```
0 ──5──> 1 ──2──> 3
│        │
1        4
│        │
v        v
2 ──8──> 3
```

Adjacency: `0: [(1, 5), (2, 1)], 1: [(3, 2)], 2: [(1, 4), (3, 8)], 3: []`. Source = 0.

```
dist = [0, ∞, ∞, ∞]
heap = [(0, 0)]

Pop (0, 0). dist[0] = 0.
  Edge (1, 5): tentative dist 5 < ∞ → dist[1] = 5, push (5, 1)
  Edge (2, 1): tentative dist 1 < ∞ → dist[2] = 1, push (1, 2)
heap = [(1, 2), (5, 1)]

Pop (1, 2). dist[2] = 1.
  Edge (1, 4): tentative dist 1 + 4 = 5. dist[1] is already 5 — not better, don't push.
  Edge (3, 8): tentative dist 9 < ∞ → dist[3] = 9, push (9, 3)
heap = [(5, 1), (9, 3)]

Pop (5, 1). dist[1] = 5.
  Edge (3, 2): tentative dist 5 + 2 = 7 < dist[3]=9 → dist[3] = 7, push (7, 3)
heap = [(7, 3), (9, 3)]

Pop (7, 3). dist[3] = 7. No outgoing edges.

Pop (9, 3). 9 > dist[3]=7 → SKIP (stale entry).

Done. dist = [0, 5, 1, 7]   ✓
```

The shortest path from 0 to 3 has cost 7 (via 0 → 1 → 3).

---

## Time Complexity

**O((V + E) × log V)** with a binary heap. Each edge can trigger one push (O(log V)); each node is popped once (O(log V)).

## Space Complexity

**O(V + E)** — `dist[]` and the heap.

---

## Code (Java)

```java
public int[] dijkstra(int n, List<List<int[]>> adj, int src) {
    // adj[u] is a list of [neighbor, weight]
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    PriorityQueue<int[]> pq = new PriorityQueue<>(
        (a, b) -> Integer.compare(a[0], b[0])      // min-heap by distance
    );
    pq.offer(new int[]{0, src});

    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        int d = cur[0], u = cur[1];

        if (d > dist[u]) continue;                  // stale entry

        for (int[] e : adj.get(u)) {
            int v = e[0], w = e[1];
            // Use long arithmetic if there's any overflow risk:
            int newDist = (dist[u] == Integer.MAX_VALUE) ? Integer.MAX_VALUE : dist[u] + w;
            if (newDist < dist[v]) {
                dist[v] = newDist;
                pq.offer(new int[]{newDist, v});
            }
        }
    }
    return dist;
}
```

---

## Code (Python)

```python
import heapq

class Solution:
    def dijkstra(self, n: int, adj, src: int):
        # adj[u] is a list of (neighbor, weight)
        dist = [float('inf')] * n
        dist[src] = 0
        pq = [(0, src)]                             # (distance, node)

        while pq:
            d, u = heapq.heappop(pq)
            if d > dist[u]:
                continue                             # stale
            for v, w in adj[u]:
                new_dist = d + w
                if new_dist < dist[v]:
                    dist[v] = new_dist
                    heapq.heappush(pq, (new_dist, v))

        return dist
```

> Python's `heapq` provides a min-heap; tuples sort lexicographically (so `(distance, node)` works perfectly as the heap key).

---

## Edge Cases

| Case | Result |
|---|---|
| Disconnected graph | Unreachable nodes stay at `∞`. |
| Source's own distance | 0. |
| Zero-weight edges | Fine — they're non-negative. |
| Stale heap entries | The `if d > dist[u]: continue` line handles them. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Reconstruct the actual path, not just distance."**
   Track `parent[v] = u` whenever you update `dist[v]`. To get the path to `t`, walk back: `t → parent[t] → parent[parent[t]] → ... → s`.

2. **"Single-source single-target — can you stop early?"**
   Yes. Add `if u == target: return dist[target]` after popping.

3. **"What if there are negative weights?"**
   See `dijkstra-negative.md`. Dijkstra fails — use Bellman-Ford.

4. **"All-pairs shortest paths?"**
   Run Dijkstra from each node (O(V × (V+E) log V)), or use Floyd-Warshall (O(V³)). Floyd-Warshall is simpler but worse for sparse graphs.

5. **"What about A*?"**
   A* is Dijkstra + a heuristic that estimates distance to the target — it focuses the search and runs much faster when you have a good heuristic (like geographic distance).

---

## Key Lessons from This Problem

1. **Greedy + min-heap = Dijkstra.** Always pick the closest unvisited node next.
2. **The non-negative weight assumption is critical** — without it, the greedy choice can be wrong.
3. **Stale entries in the heap** are a normal part of the algorithm — just skip them on pop.
