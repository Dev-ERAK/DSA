# Problem: Implement Binary Search

## Category
Searching

## Difficulty
Easy

---

## Problem Statement

Implement **binary search**: given a **sorted** array `nums` and a `target` value, return the **index** of the target in the array, or `-1` if not present.

This is **LeetCode #704** — a fundamental algorithm everyone should know cold.

### Examples

```
nums = [-1, 0, 3, 5, 9, 12], target = 9    →  4
nums = [-1, 0, 3, 5, 9, 12], target = 2    →  -1
nums = [5],                  target = 5    →  0
nums = [],                   target = 1    →  -1
```

---

## Clarifying Questions to Ask the Interviewer

1. **Sorted ascending?** Yes — required.
2. **Duplicates?** Returns *some* matching index. For first/last specifically, see `first-occurrence.md`.
3. **Iterative or recursive?** Iterative is preferred (O(1) space).

---

## Approach

### The Big Idea — Halve, Halve, Halve

We maintain a **search range** `[lo, hi]` (inclusive on both ends). At each step:

1. Compute `mid = lo + (hi - lo) / 2`.
2. Compare `nums[mid]` with `target`:
   - **Match** → return `mid`.
   - **`nums[mid] < target`** → the target is in the right half: `lo = mid + 1`.
   - **`nums[mid] > target`** → the target is in the left half: `hi = mid - 1`.

Continue while `lo ≤ hi`. If we exit the loop without matching, the target isn't in the array.

### Why does this work?

The array is sorted, so we know exactly which half to search. Each step **halves** the remaining range — we go from n down to 1 in `log₂(n)` steps.

For n = 1,000,000, that's about 20 steps. Compare to a linear scan, which would take up to 1 million.

### Why `lo + (hi - lo) / 2` instead of `(lo + hi) / 2`?

If `lo` and `hi` are both very large `int`s, `lo + hi` can **overflow**. The form `lo + (hi - lo) / 2` stays safely in range. Always use the safe form in production.

(In Python, integers are arbitrary-precision, so `(lo + hi) // 2` is fine. But the safe form is still a good habit.)

### Walking through `nums = [-1, 0, 3, 5, 9, 12], target = 9`

```
Step 1: lo=0, hi=5. mid=2. nums[2]=3. 3 < 9 → lo=mid+1=3.
Step 2: lo=3, hi=5. mid=4. nums[4]=9. MATCH! Return 4. ✓
```

### Walking through "target = 2" (not present)

```
Step 1: lo=0, hi=5. mid=2. nums[2]=3. 3 > 2 → hi=1.
Step 2: lo=0, hi=1. mid=0. nums[0]=-1. -1 < 2 → lo=1.
Step 3: lo=1, hi=1. mid=1. nums[1]=0. 0 < 2 → lo=2.
Step 4: lo=2, hi=1. Loop exits. Return -1. ✓
```

---

## Time Complexity

**O(log n)** — the search range halves each step.

## Space Complexity

- Iterative: **O(1)**.
- Recursive: **O(log n)** stack frames.

---

## Code (Java)

```java
public int binarySearch(int[] nums, int target) {
    int lo = 0;
    int hi = nums.length - 1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;        // overflow-safe

        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            lo = mid + 1;                     // search right half
        } else {
            hi = mid - 1;                     // search left half
        }
    }
    return -1;
}
```

---

## Code (Python)

```python
class Solution:
    def binarySearch(self, nums, target: int) -> int:
        lo, hi = 0, len(nums) - 1

        while lo <= hi:
            mid = (lo + hi) // 2

            if nums[mid] == target:
                return mid
            elif nums[mid] < target:
                lo = mid + 1
            else:
                hi = mid - 1
        return -1
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty array | `-1` (loop doesn't execute) |
| Single element matches | 0 |
| Single element doesn't match | -1 |
| Target smaller than min | -1 quickly (just goes left until `hi < 0`) |
| Target larger than max | -1 quickly (just goes right until `lo > n - 1`) |

---

## Follow-up Questions an Interviewer Might Ask

1. **"First or last occurrence with duplicates?"** See `first-occurrence.md`.

2. **"Search in a *rotated* sorted array (LeetCode 33)."**
   The array was sorted, then rotated by some unknown amount. Modify binary search: at each step, identify which half is properly sorted (compare `nums[mid]` to `nums[lo]`), then check if the target lies in that half.

3. **"Search in a 2D matrix (LeetCode 74)."**
   If the matrix is sorted as a single flattened array, binary-search over `0..m*n-1` and convert `mid` to `(row, col) = (mid / n, mid % n)`.

4. **"What does it mean to do 'binary search on the answer'?"**
   Some problems aren't really "find x in this array" — they're "find the smallest x for which some condition holds." If the condition is **monotonic** (once true, always true for larger x), you can binary-search the answer space. See `searching/ship-capacity.md`.

5. **"How would you binary-search a linked list?"**
   You can't, efficiently — finding the middle takes O(n) on a linked list. Each step is O(n), so total is O(n log n) — worse than linear search.

---

## Key Lessons from This Problem

1. **Halving is exponentially powerful.** O(log n) is essentially constant for any practical n.
2. **Pick a convention** — inclusive `[lo, hi]` or exclusive `[lo, hi)` — and stick with it. Off-by-one bugs are the #1 binary search killer.
3. **The overflow-safe formula** `lo + (hi - lo) / 2` is a small habit that avoids real bugs.
