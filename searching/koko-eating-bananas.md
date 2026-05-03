# Problem: Koko Eating Bananas (Minimum Eating Speed)

## Category
Searching / Binary Search on Answer

## Difficulty
Medium

---

## Problem Statement

Koko has `n` piles of bananas. Pile `i` has `piles[i]` bananas. She has `h` hours to eat all the bananas. Her **eating speed** is `k` bananas per hour, where:

- Each hour, she picks **one pile** and eats up to `k` bananas from it.
- If a pile has fewer than `k` bananas, she eats them all and **rests for the rest of that hour** (so she can't start a new pile until next hour).

Return the **minimum integer eating speed** `k` that lets her finish all the bananas within `h` hours.

This is **LeetCode #875**.

### Example

```
piles = [3, 6, 7, 11], h = 8   →  k = 4

Why? At speed 4:
  Pile 3: 1 hour
  Pile 6: 2 hours (6 / 4 = 1.5, rounded up)
  Pile 7: 2 hours
  Pile 11: 3 hours (11 / 4 = 2.75, rounded up)
  Total: 1 + 2 + 2 + 3 = 8 hours. ✓
```

---

## Clarifying Questions to Ask the Interviewer

1. **One pile per hour, no spillover?** Yes — that's the rule.
2. **`h ≥ n` always?** Yes — otherwise impossible (you can't finish n piles in fewer than n hours).
3. **Integer `k` only?** Yes — the answer is an integer.

---

## Approach — Binary Search on the Speed

### The key insight

For any speed `k`, define `canFinish(k)` = "can Koko finish in `h` hours at speed `k`?"

This is **monotonic**: faster speeds finish in fewer or equal hours. So if `canFinish(k)` is true, then `canFinish(k + 1)` is also true.

We want the **smallest `k`** with `canFinish(k) = true`.

### How many hours at speed k?

For each pile of size `p`, hours required = `ceil(p / k)`. (Because she can eat at most `k` per hour; a leftover smaller-than-k chunk still takes a full hour.)

Total hours = sum of those over all piles.

```
canFinish(k):
    hours = sum(ceil(p / k) for p in piles)
    return hours <= h
```

### How to compute `ceil(p / k)` with integer math

`ceil(p / k) = (p + k - 1) // k`. (Or in Python: `-(-p // k)`.)

### Bounds for binary search

- **Lower bound:** `k = 1` (slowest possible).
- **Upper bound:** `k = max(piles)` — at this speed, every pile takes exactly 1 hour, total `n` hours, which is the fastest possible.

### Walking through

`piles = [3, 6, 7, 11]`, `h = 8`. Bounds: `lo = 1`, `hi = 11`.

```
mid = 6: hours = ceil(3/6) + ceil(6/6) + ceil(7/6) + ceil(11/6)
              = 1 + 1 + 2 + 2 = 6 ≤ 8 → true. hi = 6.

mid = 3: hours = ceil(3/3) + ceil(6/3) + ceil(7/3) + ceil(11/3)
              = 1 + 2 + 3 + 4 = 10 > 8 → false. lo = 4.

mid = 5: hours = 1 + 2 + 2 + 3 = 8 ≤ 8 → true. hi = 5.

mid = 4: hours = 1 + 2 + 2 + 3 = 8 ≤ 8 → true. hi = 4.

lo = hi = 4. Return 4. ✓
```

---

## Time Complexity

**O(n × log(max(piles)))**.

## Space Complexity

**O(1)**

---

## Code (Java)

```java
public int minEatingSpeed(int[] piles, int h) {
    int lo = 1, hi = 0;
    for (int p : piles) hi = Math.max(hi, p);

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (canFinish(piles, mid, h)) {
            hi = mid;                                // try slower
        } else {
            lo = mid + 1;                            // need to be faster
        }
    }
    return lo;
}

private boolean canFinish(int[] piles, int k, int h) {
    long hours = 0;
    for (int p : piles) {
        hours += (p + k - 1) / k;                   // ceil(p / k)
        if (hours > h) return false;                 // early exit
    }
    return true;
}
```

> `long` for `hours` because in extreme cases the sum can exceed `Integer.MAX_VALUE`.

---

## Code (Python)

```python
class Solution:
    def minEatingSpeed(self, piles, h: int) -> int:
        def can_finish(k: int) -> bool:
            hours = 0
            for p in piles:
                hours += -(-p // k)                   # ceil division
                if hours > h:
                    return False
            return True

        lo, hi = 1, max(piles)
        while lo < hi:
            mid = (lo + hi) // 2
            if can_finish(mid):
                hi = mid
            else:
                lo = mid + 1
        return lo
```

---

## Edge Cases

| Case | Result |
|---|---|
| `h == n` | `k = max(piles)` (one pile per hour). |
| All piles size 1 | `k = 1` (one banana per hour suffices). |
| Very large piles | Use `long` (Java) for the hours total. |
| `h < n` | Impossible — but the problem guarantees `h ≥ n`. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why does binary search work here?"**
   The "hours required" function is monotonically **non-increasing** in `k`. Find the smallest `k` such that hours ≤ `h`.

2. **"Why `k = max(piles)` is enough as the upper bound?"**
   At that speed, every pile takes exactly 1 hour. So total = `n` hours. Since `h ≥ n` is guaranteed, this is sufficient.

3. **"Related patterns?"**
   - **Capacity to Ship Packages (LC1011)** — same pattern, see `ship-capacity.md`.
   - **Split Array Largest Sum (LC410)** — same template.

4. **"What if Koko can switch between piles within an hour?"**
   That's a different problem and probably the answer is just `ceil(sum(piles) / h)` because no waste.

---

## Key Lessons from This Problem

1. **"Binary search on the answer" works when the predicate is monotonic** — recognize this pattern.
2. **`ceil(a / b)` in integer math** = `(a + b - 1) / b`. Memorize this trick.
3. **Tight bounds matter** — `max(piles)` as the upper bound is much better than 10⁹.
