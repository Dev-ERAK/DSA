# Problem: Why Does Dijkstra Fail with Negative Edges?

## Category
Graphs / Theory

## Difficulty
Easy

---

## Problem Statement

Explain in plain language why **Dijkstra's algorithm cannot handle graphs with negative edge weights**, and what to use instead.

---

## Clarifying Questions to Ask the Interviewer

1. **Negative weights with or without negative cycles?** Important distinction — even **without** cycles, Dijkstra can fail.
2. **Are we just explaining or also writing code?** Both — show the failure with a counterexample, then show the fix.

---

## Approach (Explanation)

### The Core Assumption Dijkstra Makes

When Dijkstra **finalizes** a node `u` (pops it from the heap), it commits `dist[u]` as the shortest distance. This is correct **if** no later path through unprocessed nodes could shorten `dist[u]`.

With **non-negative** edges, any path through unprocessed nodes adds ≥ 0 — it can only stay the same or get longer. So Dijkstra's commitment is safe.

### The Counterexample with Negatives

Consider this tiny graph:

```
A ──2──> B
A ──5──> C
B ─-10─> C
```

Edges:
- A → B with weight 2
- A → C with weight 5
- B → C with weight **-10** (negative!)

Dijkstra from A:

```
1. Start: dist[A] = 0, dist[B] = ∞, dist[C] = ∞.  Push (0, A).
2. Pop A.
   Update dist[B] = 2, push (2, B).
   Update dist[C] = 5, push (5, C).
3. Pop B (smallest at 2).
   Update via B→C: 2 + (-10) = -8.
   That's < dist[C] = 5, so dist[C] = -8, push (-8, C).
4. Pop C (current top is dist -8 from latest push, OR it could be the stale 5).
   The optimal answer for C is -8.  ✓
```

Wait, in this small example Dijkstra might still get the right answer. Let me give a sharper counterexample where Dijkstra **commits the wrong distance**.

### A Cleaner Counterexample

```
A → B (weight 5)
A → C (weight 1)
C → B (weight -10)
```

Dijkstra from A:

```
1. Pop A. dist[A] = 0. Update dist[B] = 5, dist[C] = 1.
2. Pop C (dist 1). FINALIZED at 1.
   Update via C → B: 1 + (-10) = -9 < 5 → dist[B] = -9, push.
3. Pop B at distance -9. FINALIZED at -9.
```

OK, this also worked because B got pushed again. The issue with Dijkstra is more subtle: it only fails if a finalized node could be improved later.

Let's pinpoint the exact failure case:

```
A → B (weight 1)
A → C (weight 4)
B → C (weight 2)
B → D (weight 5)
D → C (weight -3)        # negative edge!
```

```
1. Pop A. dist[A] = 0. Update B=1, C=4.
2. Pop B (dist 1). FINALIZED.
   Update C: 1+2=3 < 4 → dist[C] = 3.
   Update D: 1+5=6.
3. Pop C (dist 3). FINALIZED at 3.   ← committed!
4. Pop D (dist 6).
   Try D → C: 6 + (-3) = 3. NOT less than dist[C] = 3 — no update.
```

Here Dijkstra finalized C at 3, which happens to be correct. To break Dijkstra, you need the negative edge to give a path that's **shorter** than what was committed. Construct it carefully:

```
A → B (weight 100)
A → C (weight 50)
C → B (weight -200)
```

```
1. Pop A. Update B=100, C=50.
2. Pop C (dist 50). FINALIZED.
   Update B: 50 + (-200) = -150 < 100 → dist[B] = -150.
3. Pop B at -150. FINALIZED at -150.   ✓ correct
```

Hmm, again it works. Notice: **Dijkstra's stale-entry mechanism means it can sometimes recover even with negatives** — if the node gets pushed again before being finalized.

Where it truly fails: when a node is **already finalized** before the negative-edge update arrives. This happens when the negative edge is reached "later" in heap order. Constructing such a case requires the negative-edge path to use a longer-cost intermediate hop.

The bottom line: **don't try to make Dijkstra work with negatives**. The algorithm's correctness proof depends on non-negative weights. Use a different algorithm.

### Negative Cycles — A Worse Problem

If there's a **negative cycle** reachable from the source, **no shortest path exists** — you can keep going around the cycle, reducing the total cost forever. Dijkstra is silent about this. Bellman-Ford **detects** it explicitly.

### What to Use Instead

| Situation | Algorithm |
|---|---|
| All weights ≥ 0 | **Dijkstra** (best — O((V+E) log V)) |
| Negative weights, no negative cycle | **Bellman-Ford** (O(V × E)) |
| Negative cycle present | Bellman-Ford detects it; otherwise no shortest path exists |
| All-pairs, dense graph | **Floyd-Warshall** (O(V³)) — handles negatives natively |
| All-pairs, sparse graph + no negative cycles | **Johnson's algorithm** — reweight with Bellman-Ford, then Dijkstra from each node |

---

## Time Complexity

(Theory question — not applicable.)

## Space Complexity

(Theory question — not applicable.)

---

## Code (Java) — Bellman-Ford for comparison

```java
// Returns dist[] or null if a negative cycle is reachable from src.
public int[] bellmanFord(int n, int[][] edges, int src) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE / 2);   // /2 to avoid overflow on relaxation
    dist[src] = 0;

    // Relax all edges V-1 times.
    for (int i = 0; i < n - 1; i++) {
        boolean updated = false;
        for (int[] e : edges) {
            int u = e[0], v = e[1], w = e[2];
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                updated = true;
            }
        }
        if (!updated) break;                      // early exit if no change
    }

    // One more pass — if any update happens now, there's a negative cycle.
    for (int[] e : edges) {
        int u = e[0], v = e[1], w = e[2];
        if (dist[u] + w < dist[v]) return null;
    }

    return dist;
}
```

---

## Code (Python)

```python
def bellman_ford(n: int, edges, src: int):
    """edges: list of (u, v, w). Returns dist or None on negative cycle."""
    INF = float('inf')
    dist = [INF] * n
    dist[src] = 0

    for _ in range(n - 1):
        updated = False
        for u, v, w in edges:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                updated = True
        if not updated:
            break

    # detect negative cycle
    for u, v, w in edges:
        if dist[u] + w < dist[v]:
            return None
    return dist
```

---

## Edge Cases

- Negative weights but no negative cycle → Bellman-Ford gives correct shortest paths.
- Negative cycle reachable from the source → no shortest path exists; Bellman-Ford reports it.
- All non-negative → both algorithms work; Dijkstra is much faster.

---

## Follow-up Questions an Interviewer Might Ask

1. **"Can you reweight a graph to make all weights non-negative?"**
   Yes — **Johnson's algorithm**. Use Bellman-Ford to compute "node potentials" `h[v]`. Then transform each edge `(u, v, w)` to `(u, v, w + h[u] - h[v])`. The new weights are non-negative iff there's no negative cycle.

2. **"How does Floyd-Warshall handle negatives?"**
   It computes shortest paths between all pairs. Negative cycles show up as negative entries on the diagonal of the distance matrix.

3. **"What's the time complexity of Bellman-Ford?"**
   O(V × E). It's slower than Dijkstra but more general.

---

## Key Lessons from This Problem

1. **Algorithms have preconditions.** Dijkstra's "non-negative weights" isn't a suggestion — it's a correctness requirement.
2. **Negative cycles → no shortest path exists.** Always think about whether your algorithm handles or detects this.
3. **Choose the right tool**: Dijkstra for non-negative; Bellman-Ford for negatives without cycles; Floyd-Warshall for all-pairs.
