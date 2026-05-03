# Problem: Integer Square Root via Binary Search

## Category
Searching / Math

## Difficulty
Easy

---

## Problem Statement

Given a non-negative integer `x`, return the **floor** of its square root — i.e., the largest integer `r` such that `r × r ≤ x`.

This is **LeetCode #69**.

### Examples

```
x = 4   →  2     (because 2 × 2 = 4)
x = 8   →  2     (because 2 × 2 = 4 ≤ 8, but 3 × 3 = 9 > 8)
x = 16  →  4
x = 0   →  0
x = 1   →  1
```

---

## Clarifying Questions to Ask the Interviewer

1. **Allowed to use `Math.sqrt`?** No — solve it ourselves.
2. **Integer overflow concerns?** Yes — use `long` in Java when squaring.
3. **Negative input?** Undefined; clarify (return -1 or throw).

---

## Approach — Binary Search on the Answer

### The Big Idea — "Find the largest r where r² ≤ x"

The answer `r` lies in `[0, x]`. As `r` increases, `r × r` only grows. So the property `r × r ≤ x` is **monotonic**: once it becomes false, it stays false.

When we have a monotonic property like this, **binary search** can find the boundary efficiently.

### The algorithm

Search `r` in `[1, x]` (we handle `x < 2` as a special case):

```
lo = 1, hi = x, ans = 0
while lo <= hi:
    mid = lo + (hi - lo) / 2
    if mid * mid <= x:
        ans = mid          # remember this candidate
        lo = mid + 1       # try larger
    else:
        hi = mid - 1       # mid was too big, try smaller
```

The largest `mid` for which `mid² ≤ x` is the answer.

### Walking through `x = 8`

```
lo=1, hi=8.

mid=4: 16 > 8. hi=3.
lo=1, hi=3. mid=2: 4 ≤ 8. ans=2, lo=3.
lo=3, hi=3. mid=3: 9 > 8. hi=2.
lo=3, hi=2. Loop ends.

Return ans = 2. ✓
```

### Walking through `x = 16`

```
lo=1, hi=16. mid=8: 64 > 16. hi=7.
lo=1, hi=7. mid=4: 16 ≤ 16. ans=4, lo=5.
lo=5, hi=7. mid=6: 36 > 16. hi=5.
lo=5, hi=5. mid=5: 25 > 16. hi=4.
lo=5, hi=4. Loop ends.

Return ans = 4. ✓
```

### Why "binary search on the answer"?

Because we're not searching an array — we're searching the **answer space** `[0, x]` for the largest value satisfying a monotonic predicate. Any time you can phrase a problem as "find the smallest/largest x where P(x) holds, and P is monotonic in x," binary search works.

This is a hugely useful technique. See also:
- `ship-capacity.md` (find min capacity)
- `koko-eating-bananas.md` (find min speed)

---

## Time Complexity

**O(log x)** — binary search over `[0, x]`.

## Space Complexity

**O(1)**

---

## Code (Java)

```java
public int mySqrt(int x) {
    if (x < 2) return x;                            // 0 → 0, 1 → 1

    long lo = 1, hi = x, ans = 0;

    while (lo <= hi) {
        long mid = lo + (hi - lo) / 2;
        if (mid * mid <= x) {                       // mid is a valid candidate
            ans = mid;
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return (int) ans;
}
```

> Why `long`? For `x = Integer.MAX_VALUE ≈ 2.1 × 10⁹`, `mid` can be ~46340. `mid * mid` is fine in long but overflows int.

---

## Code (Python)

```python
class Solution:
    def mySqrt(self, x: int) -> int:
        if x < 2:
            return x

        lo, hi, ans = 1, x, 0
        while lo <= hi:
            mid = (lo + hi) // 2
            if mid * mid <= x:
                ans = mid
                lo = mid + 1
            else:
                hi = mid - 1
        return ans
```

> Python integers are arbitrary precision — no overflow worries.

---

## Edge Cases

| Case | Result |
|---|---|
| `x = 0` | `0` |
| `x = 1` | `1` |
| Perfect squares (4, 9, 16, ...) | The exact root. |
| `x = Integer.MAX_VALUE` | Use `long` for `mid * mid`, otherwise overflow. |
| Negative `x` | Undefined; throw or return -1. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Floating-point square root?"**
   Use **Newton's method**:
   ```
   guess = x / 2
   while not close enough:
       guess = (guess + x / guess) / 2
   ```
   Converges quadratically. Or binary-search to a precision tolerance.

2. **"N-th root?"**
   Same binary-search idea: find largest `r` with `r^n ≤ x`. Use `pow(mid, n)` (carefully — that can overflow).

3. **"Avoid the `mid * mid` overflow without `long`?"**
   Compare `mid <= x / mid` (with care for `mid == 0`).

4. **"Newton's iteration formula?"**
   `r_{k+1} = (r_k + x / r_k) / 2`. Started near the true root, it doubles the number of correct digits each iteration.

---

## Key Lessons from This Problem

1. **"Binary search on the answer"** is a powerful pattern — recognize when a problem has a monotonic predicate.
2. **Watch for overflow** when squaring numbers near INT_MAX. Use `long`.
3. **Newton's method** is faster in floating-point but harder to get right with integer arithmetic — binary search is the safe choice.
