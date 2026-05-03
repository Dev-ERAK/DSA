# Problem: Maximum Sum Subarray of Size K

## Category
Arrays / Sliding Window

## Difficulty
Easy

---

## Problem Statement

Given an array of integers and a number `k`, find the **maximum sum** among all contiguous subarrays of **exactly size k**.

This is the canonical introduction to **sliding-window** problems.

### Example

```
Input:  nums = [2, 1, 5, 1, 3, 2], k = 3
Output: 9

Why? The subarrays of size 3 are:
[2,1,5] sum = 8
[1,5,1] sum = 7
[5,1,3] sum = 9   ← max
[1,3,2] sum = 6
```

---

## Clarifying Questions to Ask the Interviewer

1. **What if `k > n`?** Usually invalid — return 0, throw, or assume the input always has `k ≤ n`. Confirm with the interviewer.
2. **Can elements be negative?** Yes — sliding window for **fixed size** still works regardless of sign. (For variable-size windows, signs sometimes matter.)
3. **Return the sum or the subarray?** Sum, by default.

---

## Approach

### Brute force — O(n × k)

For each starting index `i` from 0 to `n - k`, sum up `nums[i..i+k-1]`. That's `(n - k + 1) × k ≈ n×k` operations.

Why it's wasteful: between window `[i..i+k-1]` and the next window `[i+1..i+k]`, only **one element changed at each end**. The middle `k - 2` elements are identical. We're re-summing them needlessly.

### The Big Idea — Sliding Window

A "sliding window" is a fixed-size range that you move by **one position at a time**, updating the sum incrementally.

```
window: [a, b, c]  sum = a + b + c
        ↓ slide right by 1
window:    [b, c, d]  sum = (a + b + c) - a + d
```

**Key observation:** when the window slides right by 1, you add the new right element and subtract the old left element. That's O(1) work per slide.

### The algorithm

1. Compute the sum of the first `k` elements. Call this `window`.
2. Set `best = window`.
3. For each new index `i` from `k` to `n-1`:
   - `window += nums[i]` (add the entering element)
   - `window -= nums[i - k]` (subtract the leaving element)
   - Update `best` if `window` is larger.
4. Return `best`.

### Walking through the example

`nums = [2, 1, 5, 1, 3, 2]`, `k = 3`.

Initial window (first 3): `2 + 1 + 5 = 8`. `best = 8`.

| i | nums[i] entering | nums[i-k] leaving | new window | best |
|---|---|---|---|---|
| 3 | 1 | 2 | 8 + 1 - 2 = 7 | 8 |
| 4 | 3 | 1 | 7 + 3 - 1 = 9 | **9** |
| 5 | 2 | 5 | 9 + 2 - 5 = 6 | 9 |

Answer: **9** ✓

---

## Time Complexity

**O(n)** — each element enters and leaves the window exactly once.

## Space Complexity

**O(1)** — just a few variables.

---

## Code (Java)

```java
public int maxSumK(int[] nums, int k) {
    if (nums.length < k) return 0;       // or throw — clarify with interviewer

    // Step 1: sum of the first k elements.
    int window = 0;
    for (int i = 0; i < k; i++) {
        window += nums[i];
    }
    int best = window;

    // Step 2: slide the window across the rest of the array.
    for (int i = k; i < nums.length; i++) {
        window += nums[i];               // add new right element
        window -= nums[i - k];           // remove old left element
        best = Math.max(best, window);
    }
    return best;
}
```

---

## Code (Python)

```python
class Solution:
    def maxSumK(self, nums, k: int) -> int:
        if len(nums) < k:
            return 0

        window = sum(nums[:k])           # sum of the first k elements
        best = window

        for i in range(k, len(nums)):
            window += nums[i]            # entering element
            window -= nums[i - k]        # leaving element
            best = max(best, window)

        return best
```

---

## Edge Cases

| Case | What to do |
|---|---|
| `k == n` | The whole array is the only window. Return total sum. |
| `k == 1` | Each "window" is one element — answer is the maximum element. |
| `k > n` | No valid window. Return 0 or throw. |
| All-negative array | Answer is the **least-negative** window (since we still must have a window). |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if k can vary? Find the smallest window with sum ≥ S."**
   That's a **variable-size sliding window**. Grow the right side until the sum is enough, then shrink the left side as much as possible. Both pointers only move right → O(n).

2. **"What if you want the window with maximum *product* of size k?"**
   Trickier — sliding doesn't work cleanly because zeros and negatives change behavior. Use a **monotonic deque** or recompute when zeros appear.

3. **"Return the actual subarray, not just the sum."**
   Track the start index when you update `best`. The subarray is `nums[bestStart..bestStart+k-1]`.

4. **"Can it be done in O(1) extra space?"**
   Yes — see code above. We only use a few integers.

---

## Key Lessons from This Problem

1. **Sliding window** turns O(n × k) into O(n) by reusing work between adjacent windows.
2. **Add the entering element, subtract the leaving element** — that's the heart of the technique.
3. **Fixed-size vs variable-size windows** are two different patterns. Master the fixed-size first; variable-size builds on it.
