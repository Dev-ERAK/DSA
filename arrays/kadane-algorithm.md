# Problem: Maximum Subarray Sum (Kadane's Algorithm)

## Category
Arrays / Dynamic Programming

## Difficulty
Medium

---

## Problem Statement

Given an integer array `nums`, find the **contiguous subarray** with the **largest sum** and return that sum.

This is **LeetCode #53** — a classic, and a great introduction to dynamic programming.

### Example

```
Input:  nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Output: 6     (the subarray [4, -1, 2, 1])

Input:  nums = [1]
Output: 1

Input:  nums = [-1, -2, -3]
Output: -1    (best we can do — must return some non-empty subarray)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Must the subarray be non-empty?** Almost always yes. So even if all elements are negative, we have to return something.
2. **What if all elements are negative?** Return the maximum (least negative) element.
3. **Return the sum, or the subarray itself?** Sum is standard. Reconstruction is a follow-up.

---

## Approach

### Brute force — O(n³) or O(n²)

Try every starting index, every ending index, sum the elements between. O(n²) with prefix sums; O(n³) without. Too slow for `n = 10⁵`.

### Kadane's Insight — make a local decision at each step

At each index `i`, ask: **"For a subarray ending here, what's the best sum?"**

There are exactly two choices:

- **Extend** the subarray we were building: `cur + nums[i]`.
- **Restart** with just `nums[i]` (because previous total is dragging us down).

Pick whichever is larger:

```
cur = max(nums[i], cur + nums[i])
```

Then `best = max(best, cur)` — the global answer is the largest "local best" seen so far.

### Why does this work?

The optimal subarray must end **somewhere**. So if we know the best subarray ending at every index, the answer is the maximum of those.

For "best ending at index `i`":
- It either includes `i-1`'s best (extends it), or it doesn't.
- If extending makes the sum smaller than just starting fresh at `nums[i]`, then "best ending at i" is just `nums[i]`.

This is **dynamic programming** — solving a problem by combining solutions to overlapping subproblems.

### Walking through the example

`nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]`

| i | nums[i] | cur = max(nums[i], cur + nums[i]) | best |
|---|---|---|---|
| 0 | -2 | -2 (start) | -2 |
| 1 | 1 | max(1, -2 + 1) = max(1, -1) = **1** (restart) | 1 |
| 2 | -3 | max(-3, 1 + -3) = max(-3, -2) = **-2** (extend) | 1 |
| 3 | 4 | max(4, -2 + 4) = max(4, 2) = **4** (restart) | 4 |
| 4 | -1 | max(-1, 4 + -1) = max(-1, 3) = **3** (extend) | 4 |
| 5 | 2 | max(2, 3 + 2) = max(2, 5) = **5** (extend) | 5 |
| 6 | 1 | max(1, 5 + 1) = max(1, 6) = **6** (extend) | **6** |
| 7 | -5 | max(-5, 6 + -5) = max(-5, 1) = **1** (extend) | 6 |
| 8 | 4 | max(4, 1 + 4) = max(4, 5) = **5** (extend) | 6 |

Answer: **6** ✓

The "restart" decisions tell you where the optimal subarray begins; the "extend" decisions tell you where it grows.

### Visualizing "extend or restart"

```
Running sum:    -2  ->  1  -> -2  ->  4  ->  3  ->  5  ->  6  ->  1  ->  5
Decision:      start RESTART ext RESTART ext  ext  ext   ext  ext
```

The two restarts cleanly mark the boundaries of the optimal subarray.

---

## Time Complexity

**O(n)** — one pass.

## Space Complexity

**O(1)** — only two integers (`cur` and `best`).

---

## Code (Java)

```java
public int maxSubArray(int[] nums) {
    int cur = nums[0];                 // best sum ending at index 0
    int best = nums[0];                // best sum seen anywhere

    for (int i = 1; i < nums.length; i++) {
        // Either extend the previous subarray, or start fresh here.
        cur = Math.max(nums[i], cur + nums[i]);
        best = Math.max(best, cur);
    }
    return best;
}
```

---

## Code (Python)

```python
class Solution:
    def maxSubArray(self, nums) -> int:
        cur = best = nums[0]
        for x in nums[1:]:
            cur = max(x, cur + x)
            best = max(best, cur)
        return best
```

---

## Edge Cases

| Case | What happens |
|---|---|
| Single element | Returned as-is (positive or negative). |
| All negatives | The "max element" wins (we always extend or restart, but the best is the least negative single element). |
| All positives | We always extend; answer is the total sum. |
| Empty array | Ill-defined — clarify (return 0 or throw). The starter `nums[0]` would crash on empty input, so guard if needed. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Return the subarray indices, not just the sum."**
   Track a `start` pointer. Reset it when you "restart" (when `nums[i] > cur + nums[i]`). Track `end = i` whenever `cur` becomes the new `best`.

2. **"Maximum sum *circular* subarray (LeetCode 918)."**
   See `arrays/max-circular-subarray.md`. Compute Kadane normally, *and* compute Kadane on the negated array (to find the minimum subarray sum), then `total - minSum` is the wrapping case.

3. **"Maximum product subarray."**
   Kadane-like, but track both **max** and **min** ending here (because multiplying a negative number flips them). LeetCode 152.

4. **"Maximum sum submatrix (2D)."**
   For each pair of rows `(r1, r2)`, sum the columns between them and run Kadane on that 1D array. Total: O(n³).

---

## Key Lessons from This Problem

1. **"Extend or restart"** is a beautifully simple idea — it's the heart of Kadane.
2. **Track best-ending-here AND best-overall** as separate variables. They serve different purposes.
3. **Many array problems have a "Kadane shape"** — when you see "best contiguous something," try the local-decision approach first.
