# Problem: 0/1 Knapsack (Bottom-Up DP)

## Category
Dynamic Programming

## Difficulty
Medium

---

## Problem Statement

You have:
- `n` items, each with a **weight** `w[i]` and a **value** `v[i]`.
- A knapsack with capacity `W` (max total weight).

Choose a subset of items to **maximize total value** without exceeding `W`. Each item can be taken **at most once** (hence "0/1").

This is a classic DP problem.

### Example

```
weights = [1, 3, 4, 5]
values  = [1, 4, 5, 7]
W = 7

Best choice: items 2 and 3 (weights 3 + 4 = 7, values 4 + 5 = 9).
```

---

## Clarifying Questions to Ask the Interviewer

1. **Integer weights and capacity?** Yes — that's required for this DP. (Real-valued capacities use a different approach.)
2. **Each item used at most once?** Yes (0/1 knapsack). Unbounded version uses each item any number of times.
3. **Maximize value, not the number of items?** Yes.

---

## Approach — Bottom-Up DP

### The DP table

`dp[i][c]` = maximum value achievable using the **first `i` items** with **capacity `c`**.

We want `dp[n][W]` at the end.

### The recurrence — for each item, two choices

For item `i` (1-indexed):

1. **Skip item i:** `dp[i][c] = dp[i-1][c]`. Same value as not having seen item i.

2. **Take item i** (only if `w[i] ≤ c`): `dp[i][c] = dp[i-1][c - w[i]] + v[i]`. We use up `w[i]` capacity and gain `v[i]` value.

Take whichever is larger:

```
dp[i][c] = max(dp[i-1][c],                      # skip
               dp[i-1][c - w[i]] + v[i])        # take (if it fits)
```

### Walking through the example

```
weights = [1, 3, 4, 5], values = [1, 4, 5, 7], W = 7
```

Build the DP table (rows are number of items considered; columns are capacity 0..7):

```
        c: 0   1   2   3   4   5   6   7
   i=0      0   0   0   0   0   0   0   0      (no items → 0 value)
   i=1 (w=1, v=1):
            0   1   1   1   1   1   1   1      (only item 1 available)
   i=2 (w=3, v=4):
            0   1   1   4   5   5   5   5      (can take item 2 from c=3)
   i=3 (w=4, v=5):
            0   1   1   4   5   6   6   9      (item 3 fits from c=4)
   i=4 (w=5, v=7):
            0   1   1   4   5   7   8   9
```

Reading some cells:
- `dp[2][3]`: capacity 3, items 1 and 2 available. Take item 2 (w=3, v=4). dp[1][0] + 4 = 4. Skip = dp[1][3] = 1. Max = 4. ✓
- `dp[3][7]`: capacity 7. Skip item 3 → dp[2][7] = 5. Take item 3 (w=4, v=5) → dp[2][3] + 5 = 9. Max = 9. ✓
- `dp[4][7] = 9`. The best.

Final answer: **9** (items 2 and 3, weights 3 + 4, values 4 + 5).

### Space optimization — 1D array

We only ever read from the previous row (`dp[i-1]`). So we can use a single 1D array of length `W + 1`, **iterating capacity from W down to w[i]** (so we don't reuse the new value during the same item's iteration).

The "from high to low" direction is key. Going low to high would let an item "be taken twice" within one iteration — that's the unbounded knapsack.

---

## Time Complexity

**O(n × W)** — pseudo-polynomial (in the *value* of W, not its bit-length).

## Space Complexity

**O(n × W)**, reducible to **O(W)**.

---

## Code (Java)

```java
public int knapsack(int[] w, int[] v, int W) {
    int n = w.length;
    int[][] dp = new int[n + 1][W + 1];

    for (int i = 1; i <= n; i++) {
        for (int c = 0; c <= W; c++) {
            // Option 1: skip item i.
            dp[i][c] = dp[i - 1][c];

            // Option 2: take item i (if it fits).
            if (w[i - 1] <= c) {
                dp[i][c] = Math.max(dp[i][c], dp[i - 1][c - w[i - 1]] + v[i - 1]);
            }
        }
    }
    return dp[n][W];
}

// Space-optimized to O(W)
public int knapsack1D(int[] w, int[] v, int W) {
    int[] dp = new int[W + 1];

    for (int i = 0; i < w.length; i++) {
        // Iterate capacity HIGH to LOW so we don't re-take the same item.
        for (int c = W; c >= w[i]; c--) {
            dp[c] = Math.max(dp[c], dp[c - w[i]] + v[i]);
        }
    }
    return dp[W];
}
```

---

## Code (Python)

```python
class Solution:
    def knapsack(self, w, v, W: int) -> int:
        n = len(w)
        dp = [[0] * (W + 1) for _ in range(n + 1)]

        for i in range(1, n + 1):
            for c in range(W + 1):
                dp[i][c] = dp[i - 1][c]                  # skip
                if w[i - 1] <= c:                         # take
                    dp[i][c] = max(dp[i][c], dp[i - 1][c - w[i - 1]] + v[i - 1])
        return dp[n][W]

    def knapsack_1d(self, w, v, W: int) -> int:
        dp = [0] * (W + 1)
        for i in range(len(w)):
            for c in range(W, w[i] - 1, -1):              # high to low
                dp[c] = max(dp[c], dp[c - w[i]] + v[i])
        return dp[W]
```

---

## Edge Cases

| Case | Result |
|---|---|
| Capacity 0 | 0 (can't take anything) |
| All items heavier than W | 0 |
| Single item that fits | Value of that item |
| Negative values | Possibly negative answer; problem usually assumes non-negative |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Reconstruct which items were chosen."**
   Walk back through the DP table from `dp[n][W]`. At each cell, decide if the item was taken or skipped:
   - If `dp[i][c] != dp[i-1][c]`, item i was taken — record it, move to `(i-1, c - w[i])`.
   - Else, item i was skipped — move to `(i-1, c)`.

2. **"Unbounded knapsack — each item can be used unlimited times."**
   Iterate capacity **low to high** in the 1D version. That allows an item to be "re-taken" within the same iteration.

3. **"Fractional knapsack — you can take any fraction of an item."**
   Greedy: sort items by value/weight ratio, take from the highest down. Works because there's no integer constraint.

4. **"Why pseudo-polynomial?"**
   The complexity is O(n × W), but `W` can be huge (e.g., 10⁹). It's polynomial in `W`, not in `log W` (the number of bits to write W). For very large W, the problem becomes intractable.

---

## Key Lessons from This Problem

1. **"Skip or take"** — the universal binary choice in 0/1 DP problems.
2. **Iterating high-to-low** in 1D matters: it prevents using an item twice.
3. **2D DP is clearer; 1D is a memory optimization.** Write the 2D version first, then collapse if needed.
