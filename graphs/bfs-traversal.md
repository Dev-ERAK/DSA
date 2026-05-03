# Problem: BFS Traversal of a Graph

## Category
Graphs / BFS

## Difficulty
Easy

---

## Problem Statement

Given a graph (as an adjacency list) and a starting node, perform a **Breadth-First Search (BFS)** and return the order in which nodes are visited.

### What's a graph?

A graph is a collection of **nodes** (vertices) connected by **edges**. Examples:
- A network of computers (nodes) connected by cables (edges).
- A map of cities (nodes) connected by roads (edges).
- Friendships on a social network.

### What's BFS?

**Breadth-First Search** explores the graph **layer by layer**:
- First visit the start node.
- Then visit all its **direct neighbors**.
- Then their neighbors.
- And so on.

Like ripples spreading outward from a stone dropped in water.

---

## Clarifying Questions to Ask the Interviewer

1. **Directed or undirected?** BFS works the same for both (the adjacency list reflects the difference).
2. **Connected graph?** If not, you'll only reach nodes in the start's component. Loop over all nodes if you want full coverage.
3. **Output the order, or some derived info (like distance)?** Order, here.

---

## Approach — Queue-Based BFS

### The algorithm

```
mark start as visited
push start onto queue

while queue is not empty:
    u = dequeue
    output u
    for each neighbor v of u:
        if v is not visited:
            mark v as visited
            enqueue v
```

The **queue** ensures FIFO order — we process all nodes at distance 1 before any at distance 2, etc.

### Why mark as visited *before* enqueueing?

If you mark when popping (instead of when adding to the queue), the same node might be enqueued multiple times by different neighbors. That works for correctness but bloats the queue.

### Walking through an example

Graph (undirected):
```
0 -- 1
|    |
2 -- 3
|
4
```

Adjacency list:
```
0: [1, 2]
1: [0, 3]
2: [0, 3, 4]
3: [1, 2]
4: [2]
```

BFS from node 0:

```
visited = {0}, queue = [0]

Step 1: dequeue 0. Output 0. Neighbors: 1, 2 → enqueue both.
        visited = {0, 1, 2}, queue = [1, 2]

Step 2: dequeue 1. Output 1. Neighbors: 0 (visited), 3 → enqueue 3.
        visited = {0, 1, 2, 3}, queue = [2, 3]

Step 3: dequeue 2. Output 2. Neighbors: 0 (V), 3 (V), 4 → enqueue 4.
        visited = {0, 1, 2, 3, 4}, queue = [3, 4]

Step 4: dequeue 3. Output 3. All neighbors visited.
        queue = [4]

Step 5: dequeue 4. Output 4. All neighbors visited.
        queue = []
```

Output order: `[0, 1, 2, 3, 4]`. Layers: 0 (distance 0), 1 & 2 (distance 1), 3 & 4 (distance 2).

---

## Time Complexity

**O(V + E)** — every vertex is enqueued once, every edge examined twice (once from each endpoint, in undirected) or once (directed).

## Space Complexity

**O(V)** — for the visited set + queue.

---

## Code (Java)

```java
public List<Integer> bfs(int start, List<List<Integer>> adj) {
    int n = adj.size();
    boolean[] visited = new boolean[n];
    List<Integer> order = new ArrayList<>();

    Deque<Integer> queue = new ArrayDeque<>();
    queue.offer(start);
    visited[start] = true;

    while (!queue.isEmpty()) {
        int u = queue.poll();
        order.add(u);

        for (int v : adj.get(u)) {
            if (!visited[v]) {
                visited[v] = true;
                queue.offer(v);
            }
        }
    }
    return order;
}
```

---

## Code (Python)

```python
from collections import deque

class Solution:
    def bfs(self, start: int, adj):
        n = len(adj)
        visited = [False] * n
        order = []

        q = deque([start])
        visited[start] = True

        while q:
            u = q.popleft()
            order.append(u)
            for v in adj[u]:
                if not visited[v]:
                    visited[v] = True
                    q.append(v)
        return order
```

---

## Edge Cases

| Case | What happens |
|---|---|
| Single node, no edges | Returns just `[start]`. |
| Disconnected graph | BFS only reaches the start's component. Wrap in a loop over all nodes for full coverage. |
| Self-loops or parallel edges | Handled by the `visited` check. |
| Cycles | BFS naturally handles cycles via `visited`. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Find the shortest path from start to every node (unweighted graph)."**
   See `shortest-path-unweighted.md`. Track distance during BFS.

2. **"Multi-source BFS — many starts at once."**
   Enqueue **all sources** with distance 0 at the start. Useful for problems like "rotting oranges."

3. **"BFS on a 2D grid."**
   Treat each cell as a node; the 4-directional neighbors are edges. Same algorithm.

4. **"DFS or BFS — when to use which?"**
   - BFS finds shortest paths in unweighted graphs.
   - DFS is better for "is there a path?" / topological sort / detecting cycles.

---

## Key Lessons from This Problem

1. **BFS = queue + visited set.** That's the whole template.
2. **Mark visited when enqueueing**, not when dequeuing — avoids duplicate enqueues.
3. **BFS naturally explores by distance** — that's why it's the algorithm of choice for shortest paths in unweighted graphs.
