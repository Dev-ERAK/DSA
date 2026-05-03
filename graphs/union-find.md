# Problem: Union-Find (Disjoint Set Union — DSU)

## Category
Graphs / Data Structures

## Difficulty
Medium

---

## Problem Statement

Implement a **Disjoint Set Union (DSU)** data structure (also called **Union-Find**) supporting two operations, both in (almost) **O(1)**:

- `find(x)` — return the **representative** (root) of the set containing `x`.
- `union(x, y)` — merge the sets containing `x` and `y`.

### What's it good for?

DSU answers "**are these two elements in the same group?**" efficiently. Classic uses:
- **Kruskal's MST** (Minimum Spanning Tree).
- **Cycle detection** in undirected graphs.
- **Connected components** in dynamic graphs (edges added one at a time).
- **Network connectivity** queries.

---

## Clarifying Questions to Ask the Interviewer

1. **Number of elements known up front?** Yes, usually — you allocate a parent array of size `n`.
2. **0-indexed or 1-indexed?** Either; just be consistent.

---

## Approach

### The Big Idea — A Forest of Trees

Each element points to a **parent**. Following parent pointers leads you to a **root** — the representative of the set.

Initially, every element is its own parent (each is its own set):

```
Element:  0   1   2   3   4
Parent:   0   1   2   3   4
```

### find(x) — walk to the root

```
find(x):
    while parent[x] != x:
        x = parent[x]
    return x
```

### union(x, y) — merge two trees

Find both roots. If they differ, point one root at the other:

```
union(x, y):
    rx = find(x)
    ry = find(y)
    if rx == ry: return false        # already in the same set
    parent[rx] = ry                  # merge
    return true
```

### The Naive Version Is Slow

Without optimizations, trees can become tall (n-deep), making `find` O(n). Two simple tricks fix this.

### Optimization 1 — Union by Rank (or Size)

Always attach the **shorter** tree under the **taller** one. This keeps the trees shallow.

```
union(x, y):
    rx = find(x); ry = find(y)
    if rx == ry: return false
    if rank[rx] < rank[ry]: swap(rx, ry)
    parent[ry] = rx                       # attach shorter under taller
    if rank[rx] == rank[ry]: rank[rx]++   # tied: tree grew
```

### Optimization 2 — Path Compression

During `find`, **flatten** the path so that every node we touched points directly at the root.

```
find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])      # recursively flatten
    return parent[x]
```

Equivalent iterative form: walk up to the root, then walk up again setting each node's parent to the root.

### With Both Optimizations — Effectively O(1)

Combined, these give an **amortized time of O(α(n))** per operation, where `α` is the inverse Ackermann function. For any practical n (less than the number of atoms in the universe), `α(n) ≤ 4`. So in practice it's **constant time**.

### Walking through

Start with 5 isolated elements:
```
parent = [0, 1, 2, 3, 4]
```

`union(0, 1)`:
```
rx = find(0) = 0; ry = find(1) = 1. Different. Merge.
parent = [1, 1, 2, 3, 4]   (0's parent is now 1)
```

`union(2, 3)`:
```
parent = [1, 1, 3, 3, 4]
```

`union(0, 2)`:
```
rx = find(0) = 1; ry = find(2) = 3. Different. Merge.
parent = [1, 3, 3, 3, 4]
```

Now `find(0)`:
```
parent[0] = 1, parent[1] = 3, parent[3] = 3 → root is 3.
With path compression: parent[0] = 3, parent[1] = 3.
parent = [3, 3, 3, 3, 4]
```

So `0`, `1`, `2`, `3` are all in one component; `4` is alone.

---

## Time Complexity

- `find`, `union`: **amortized O(α(n))** ≈ effectively O(1).

## Space Complexity

**O(n)** — the parent and rank arrays.

---

## Code (Java)

```java
class DSU {
    private final int[] parent;
    private final int[] rank;
    private int components;

    public DSU(int n) {
        parent = new int[n];
        rank = new int[n];
        components = n;
        for (int i = 0; i < n; i++) parent[i] = i;
    }

    public int find(int x) {
        // Walk to root, then compress (path halving).
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];      // halving — point to grandparent
            x = parent[x];
        }
        return x;
    }

    public boolean union(int a, int b) {
        int ra = find(a);
        int rb = find(b);
        if (ra == rb) return false;             // already same set

        // Union by rank.
        if (rank[ra] < rank[rb]) {
            int t = ra; ra = rb; rb = t;
        }
        parent[rb] = ra;
        if (rank[ra] == rank[rb]) rank[ra]++;

        components--;
        return true;
    }

    public int components() { return components; }
}
```

---

## Code (Python)

```python
class DSU:
    def __init__(self, n: int):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.components = n

    def find(self, x: int) -> int:
        # Find root.
        root = x
        while self.parent[root] != root:
            root = self.parent[root]
        # Path compression — make every node on the path point to root.
        while self.parent[x] != root:
            self.parent[x], x = root, self.parent[x]
        return root

    def union(self, a: int, b: int) -> bool:
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False

        if self.rank[ra] < self.rank[rb]:
            ra, rb = rb, ra
        self.parent[rb] = ra
        if self.rank[ra] == self.rank[rb]:
            self.rank[ra] += 1

        self.components -= 1
        return True
```

---

## Edge Cases

| Case | Result |
|---|---|
| `union(x, x)` | Returns false (same set). |
| Operations on the same element | Trivially fine. |
| Without path compression | Trees can grow tall — operations become O(n) worst case. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Detect cycles in an undirected graph using DSU."**
   See `cycle-detection-dsu.md`. For each edge, if both endpoints are already in the same component, this edge would close a cycle.

2. **"Implement Kruskal's MST."**
   Sort edges by weight ascending. For each edge, try to `union` its endpoints. If they're in different components, include this edge in the MST.

3. **"DSU with rollback."**
   Don't use path compression (it makes rollback hard). Instead, push each union onto a stack so you can undo later. Useful for offline algorithms.

4. **"Online connectivity in a directed graph."**
   Much harder than the undirected case. Use **link-cut trees** or **Euler tour trees**.

---

## Key Lessons from This Problem

1. **Path compression + union by rank** = nearly O(1) per operation.
2. **DSU is a workhorse** for connectivity problems on graphs.
3. **The data structure is small** (just two arrays) but the analysis is famously deep — inverse Ackermann is one of the most beautiful results in algorithms.
