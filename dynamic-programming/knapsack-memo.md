# Problem: 0/1 Knapsack (Top-Down Memoization)

## Category
Dynamic Programming / Recursion

## Difficulty
Medium

---

## Problem Statement

Same as `knapsack-dp.md` — find the maximum value of items that fit in a knapsack of capacity `W`. Each item is taken at most once.

This time, solve it with **top-down recursion + memoization** instead of bottom-up tabulation.

---

## Clarifying Questions to Ask the Interviewer

Same as the bottom-up version. Integer weights, each item used at most once, maximize value.

---

## Approach

### The recursive idea — "skip or take"

Define `solve(i, c)` = maximum value using items from index `i` onwards with capacity `c` remaining.

Base case: when `i == n` (no more items) or `c == 0` (no capacity), return 0.

For each item, two choices:

1. **Skip it:** `solve(i + 1, c)`.
2. **Take it** (if it fits): `v[i] + solve(i + 1, c - w[i])`.

Return the max.

```
solve(i, c):
    if i == n or c == 0: return 0
    skip = solve(i + 1, c)
    take = 0
    if w[i] <= c:
        take = v[i] + solve(i + 1, c - w[i])
    return max(skip, take)
```

### Why naive recursion is too slow — exponential

Each call branches into two more. There are 2ⁿ leaf calls. Catastrophic for n > 30.

### The fix — Memoize by `(i, c)`

Each subproblem is uniquely identified by `(i, c)`. There are only `n × (W + 1)` distinct subproblems. Cache each one.

```
memo[(i, c)] = result    # cache it
```

After memoization, each subproblem is solved once → **O(n × W)** time, same as bottom-up.

### Walking through with the example from `knapsack-dp.md`

```
weights = [1, 3, 4, 5], values = [1, 4, 5, 7], W = 7
```

```
solve(0, 7):
  skip → solve(1, 7) → ...
  take → 1 + solve(1, 6) → ...
  return max
```

The recursion tree has many overlapping calls (e.g., `solve(2, 4)` might be reached by multiple paths). Memoization caches them.

### Top-down vs Bottom-up

|   | Top-down (memoization) | Bottom-up (tabulation) |
|---|---|---|
| Style | Recursive, lazy | Iterative, eager |
| Visits all states? | Only reachable ones | All states |
| Constant factors | Higher (recursion overhead) | Lower |
| Risk of stack overflow | Yes for deep recursion | No |
| Easier to write? | Often yes — mirrors the problem definition | Sometimes more setup |

In an interview, top-down is often the easier first version to write. Convert to bottom-up if asked or if performance matters.

---

## Time Complexity

**O(n × W)** — same as bottom-up.

## Space Complexity

**O(n × W)** memo + **O(n)** recursion stack.

---

## Code (Java)

```java
public int knapsack(int[] w, int[] v, int W) {
    Integer[][] memo = new Integer[w.length][W + 1];
    return solve(0, W, w, v, memo);
}

private int solve(int i, int c, int[] w, int[] v, Integer[][] memo) {
    if (i == w.length || c == 0) return 0;

    if (memo[i][c] != null) return memo[i][c];

    int skip = solve(i + 1, c, w, v, memo);
    int take = 0;
    if (w[i] <= c) {
        take = v[i] + solve(i + 1, c - w[i], w, v, memo);
    }

    return memo[i][c] = Math.max(skip, take);
}
```

> Why `Integer[][]` (boxed) instead of `int[][]`? So we can use `null` to mean "not computed yet." With `int`, you'd need a sentinel like -1 — but valid answers can be 0, so distinguishing "not computed" from "0" is awkward.

---

## Code (Python)

```python
from functools import lru_cache

class Solution:
    def knapsack(self, w, v, W: int) -> int:
        n = len(w)

        @lru_cache(maxsize=None)
        def solve(i: int, c: int) -> int:
            if i == n or c == 0:
                return 0
            skip = solve(i + 1, c)
            take = v[i] + solve(i + 1, c - w[i]) if w[i] <= c else 0
            return max(skip, take)

        return solve(0, W)
```

> `@lru_cache` is the cleanest memoization in Python — just decorate any pure function with it.

---

## Edge Cases

| Case | Result |
|---|---|
| Capacity 0 | 0 |
| All items heavier than W | 0 |
| Single item that fits | Its value |
| Stack overflow on huge n | Switch to bottom-up |

---

## Follow-up Questions an Interviewer Might Ask

1. **"When would you prefer top-down over bottom-up?"**
   - Top-down is easier to write when the recurrence is natural.
   - Top-down only computes states that are actually reached.
   - Bottom-up has lower overhead but visits all states.

2. **"Why does memoization work here?"**
   The recursion has **overlapping subproblems** (the same `(i, c)` is reached from many paths) and **optimal substructure** (the answer for a subproblem is built from smaller subproblems). These are the two requirements for DP.

3. **"How would you do unbounded knapsack with memoization?"**
   Replace `solve(i + 1, c - w[i])` with `solve(i, c - w[i])` in the "take" branch — don't increment `i`, allowing the same item to be taken again.

4. **"Reconstruct the chosen items."**
   Re-run the recurrence with caching, but at each state, decide whether `skip` or `take` produced the cached value. Recover the chosen items.

---

## Key Lessons from This Problem

1. **Memoization = recursion + cache.** It transforms exponential into polynomial when subproblems overlap.
2. **Top-down vs bottom-up are equivalent in complexity** but differ in style and overhead.
3. **`@lru_cache` in Python** is one line away from making any recursive function fast.
