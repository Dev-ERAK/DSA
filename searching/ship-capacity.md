# Problem: Capacity to Ship Packages Within D Days

## Category
Searching / Binary Search on Answer

## Difficulty
Medium

---

## Problem Statement

You're shipping packages by boat. The packages must be loaded **in order** (you can't reorder them). Each day, the boat can carry packages up to its **capacity**.

Given:
- `weights[]` — list of package weights (in order).
- `days` — number of days available.

Return the **minimum boat capacity** that lets us ship all packages within `days` days.

This is **LeetCode #1011**.

### Example

```
weights = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10], days = 5
Output: 15

Why? With capacity 15, we can ship:
  Day 1: 1 + 2 + 3 + 4 + 5 = 15
  Day 2: 6 + 7 = 13
  Day 3: 8
  Day 4: 9
  Day 5: 10
That's 5 days exactly. Any smaller capacity would need more days.
```

---

## Clarifying Questions to Ask the Interviewer

1. **Packages in fixed order?** Yes — you can't reorder.
2. **Boat capacity must hold the **largest single package**?** Yes — that's the lower bound for the capacity.
3. **Are weights and days positive integers?** Yes.

---

## Approach — Binary Search on the Capacity

### The key insight

For any capacity `C`, ask: "**Can we ship everything in ≤ days days using capacity C?**"

The function `canShip(C)` is **monotonic**:
- If `canShip(C)` is true, then `canShip(C + 1)` is also true (more capacity is at least as good).
- If `canShip(C)` is false, then `canShip(C - 1)` is also false.

So there's a **threshold** below which it's impossible and at-or-above which it's possible. We want to find that threshold — the smallest C with `canShip(C) = true`.

This is a perfect setup for **binary search on the answer**.

### The bounds for the binary search

- **Lower bound:** `max(weights)` — we must at least be able to fit the heaviest package on a single day.
- **Upper bound:** `sum(weights)` — at this capacity, we ship everything in one day (the worst case).

### The `canShip` predicate

Walk through weights, greedily packing them onto each day's load. When adding the next package would exceed `C`, start a new day.

```
canShip(C):
    days_used = 1; load = 0
    for w in weights:
        if load + w > C:                  # doesn't fit today
            days_used += 1
            load = 0
        load += w
    return days_used <= days
```

### Walking through

`weights = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]`, `days = 5`.

Bounds: `lo = max(weights) = 10`, `hi = sum(weights) = 55`.

```
mid = 32. canShip(32)?
  Day 1: 1+2+3+4+5+6+7 = 28; +8=36 > 32 → new day.
  Day 2: 8+9+10 = 27.
  Total days: 2 ≤ 5 → true.
hi = 32 (try smaller).

mid = 21. canShip(21)?
  Day 1: 1+2+3+4+5 = 15; +6 = 21 ≤ 21 → keep loading. 6 added. Total 21. +7 = 28 > 21 → new day.
  Day 2: 7+8 = 15; +9 = 24 > 21 → new day.
  Day 3: 9+10 = 19.
  Total days: 3 ≤ 5 → true.
hi = 21.

mid = 15. canShip(15)?
  Day 1: 1+2+3+4+5 = 15. +6 = 21 > 15 → new.
  Day 2: 6+7 = 13. +8 = 21 > 15 → new.
  Day 3: 8. +9 = 17 > 15 → new.
  Day 4: 9. +10 = 19 > 15 → new.
  Day 5: 10.
  Total days: 5 ≤ 5 → true.
hi = 15.

mid = 12. canShip(12)?
  Day 1: 1+2+3+4 = 10. +5 = 15 > 12 → new.
  Day 2: 5+6 = 11. +7 = 18 > 12 → new.
  Day 3: 7. +8 = 15 > 12 → new.
  Day 4: 8. +9 = 17 > 12 → new.
  Day 5: 9. +10 = 19 > 12 → new.
  Day 6: 10.
  Total days: 6 > 5 → false.
lo = 13.

(continue narrowing...)

Eventually lo = hi = 15. Answer: 15. ✓
```

---

## Time Complexity

**O(n × log(sum - max))** — log range × O(n) per `canShip` call.

## Space Complexity

**O(1)**

---

## Code (Java)

```java
public int shipWithinDays(int[] weights, int days) {
    int lo = 0, hi = 0;
    for (int w : weights) {
        lo = Math.max(lo, w);                       // largest single package
        hi += w;                                    // total weight
    }

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (canShip(weights, mid, days)) {
            hi = mid;                               // mid works — try smaller
        } else {
            lo = mid + 1;                           // mid too small — go bigger
        }
    }
    return lo;
}

private boolean canShip(int[] weights, int cap, int days) {
    int used = 1, load = 0;
    for (int w : weights) {
        if (load + w > cap) {                       // doesn't fit — start new day
            used++;
            load = 0;
        }
        load += w;
    }
    return used <= days;
}
```

---

## Code (Python)

```python
class Solution:
    def shipWithinDays(self, weights, days: int) -> int:
        def can_ship(cap: int) -> bool:
            used, load = 1, 0
            for w in weights:
                if load + w > cap:
                    used += 1
                    load = 0
                load += w
            return used <= days

        lo, hi = max(weights), sum(weights)
        while lo < hi:
            mid = (lo + hi) // 2
            if can_ship(mid):
                hi = mid
            else:
                lo = mid + 1
        return lo
```

---

## Edge Cases

| Case | Result |
|---|---|
| `days == 1` | Capacity = `sum(weights)` (must ship everything in one day). |
| `days >= n` | Capacity = `max(weights)` (one package per day). |
| Single weight | That weight. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why does binary search work?"**
   The `canShip(cap)` predicate is monotonic in `cap`. Whenever you have a monotonic property over a numeric domain, you can binary-search the threshold.

2. **"Why these specific bounds?"**
   - `lo = max(weights)`: tighter than 1; below this, we can't ship the heaviest package on any day.
   - `hi = sum(weights)`: at this capacity, the answer is achievable (1 day). No need to search higher.
   These bounds make the binary search converge fast.

3. **"Related problems?"**
   - **Split Array Largest Sum (LC410)** — same pattern.
   - **Koko Eating Bananas (LC875)** — see `koko-eating-bananas.md`.

4. **"What if the packages can be reordered?"**
   Different problem. Could be solved as a knapsack/scheduling variant.

---

## Key Lessons from This Problem

1. **"Binary search on the answer"** when the predicate is monotonic.
2. **Tighten the bounds** — using `max` and `sum` instead of 1 and a huge number speeds up convergence.
3. **The greedy `canShip` predicate** is correct because packing must be in order — the greedy is forced.
