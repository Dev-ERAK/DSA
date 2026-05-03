# Problem: Maximum Sum Circular Subarray

## Category
Arrays / Kadane's Algorithm

## Difficulty
Medium

---

## Problem Statement

Given a **circular** integer array `nums`, return the maximum possible sum of a non-empty contiguous subarray.

"Circular" means the subarray is allowed to **wrap around** from the end of the array back to the beginning.

This is **LeetCode #918**.

### Example

```
Input:  nums = [5, -3, 5]
Output: 10
Why? The wrapping subarray [5, 5] (last and first elements) sums to 10.
That's better than the non-wrapping [5, -3, 5] which sums to 7.
```

```
Input:  nums = [1, -2, 3, -2]
Output: 3      (subarray [3] alone — non-wrapping is best here)
```

```
Input:  nums = [-3, -2, -3]
Output: -2     (must return some subarray; the single element -2 is best)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Must the subarray be non-empty?** Yes (otherwise the answer would always be 0 by picking the empty one).
2. **Can the subarray wrap around more than once?** No — it can wrap at most once. (Wrapping more means re-using elements, which isn't allowed.)
3. **All negatives — what's the answer?** The maximum element (least negative).

---

## Approach

### Two Kinds of Optimal Subarrays

Any optimal subarray is either:

1. **Non-wrapping** — fits within `nums[0..n-1]` like a normal subarray.
2. **Wrapping** — uses some prefix `nums[0..i]` and some suffix `nums[j..n-1]`.

We compute the answer to **both cases** and take the larger.

### Case 1: Non-Wrapping — Standard Kadane

Run regular Kadane on the array. Result: `maxKadane`.

### Case 2: Wrapping — Total Minus Worst Middle

Here's the clever bit. A wrapping subarray uses some prefix and some suffix — equivalently, it **excludes** some contiguous middle slice.

```
[ prefix ] [ middle excluded ] [ suffix ]
```

The wrapping sum = `total_sum - sum(middle)`. To **maximize** the wrapping sum, we want to **minimize** the middle. So:

```
maxWrapping = total - minSubarraySum
```

`minSubarraySum` is found by — surprise! — running Kadane in **min mode** on the original array.

### The All-Negative Edge Case

If every element is negative, the "min subarray" is the entire array, and `total - minSum = total - total = 0`. But that would correspond to picking the **empty** wrapping subarray, which isn't allowed!

In that case, just return `maxKadane` (the least-negative single element).

So our final answer:
```
if maxKadane < 0: return maxKadane         # all negatives
else: return max(maxKadane, total - minKadane)
```

### Walking through `[5, -3, 5]`

- Total: `5 + (-3) + 5 = 7`.
- Max Kadane (non-wrap): the whole array sums to 7. So `maxKadane = 7`.
- Min Kadane (worst contiguous slice): `[-3]` alone, sum `-3`. So `minKadane = -3`.
- Wrapping candidate: `total - minKadane = 7 - (-3) = 10`.
- Answer: `max(7, 10) = 10` ✓

That `10` corresponds to dropping the `-3` and keeping `[5, 5]`.

### Walking through `[-3, -2, -3]`

- Total: `-8`.
- Max Kadane: `-2` (best single element).
- Min Kadane: `-8` (whole array is also the worst).
- Wrapping candidate: `-8 - (-8) = 0` — but this would be the empty subarray, not allowed.
- Since `maxKadane = -2 < 0`, return `maxKadane = -2` ✓

---

## Time Complexity

**O(n)** — one pass that computes Kadane both ways and the total.

## Space Complexity

**O(1)**.

---

## Code (Java)

```java
public int maxSubarraySumCircular(int[] nums) {
    int total = 0;

    // Track running max-Kadane and min-Kadane.
    int curMax = 0, maxSum = nums[0];
    int curMin = 0, minSum = nums[0];

    for (int x : nums) {
        // Standard Kadane for the maximum.
        curMax = Math.max(curMax + x, x);
        maxSum = Math.max(maxSum, curMax);

        // Same shape, but tracking the minimum.
        curMin = Math.min(curMin + x, x);
        minSum = Math.min(minSum, curMin);

        total += x;
    }

    // All-negative guard: if the best non-wrapping sum is negative,
    // there's no valid wrapping answer (it would be empty).
    if (maxSum < 0) return maxSum;

    // Otherwise pick the larger of: best non-wrapping, total - worst middle.
    return Math.max(maxSum, total - minSum);
}
```

---

## Code (Python)

```python
class Solution:
    def maxSubarraySumCircular(self, nums) -> int:
        total = 0
        cur_max = max_sum = nums[0]
        cur_min = min_sum = nums[0]

        for i, x in enumerate(nums):
            if i == 0:
                total = x
                continue
            # Max Kadane
            cur_max = max(cur_max + x, x)
            max_sum = max(max_sum, cur_max)
            # Min Kadane
            cur_min = min(cur_min + x, x)
            min_sum = min(min_sum, cur_min)
            total += x

        if max_sum < 0:
            return max_sum                      # all-negative case

        return max(max_sum, total - min_sum)
```

---

## Edge Cases

| Case | Why it works |
|---|---|
| Single element | Both `maxSum` and `minSum` equal that element; `total - minSum = 0`, but the all-negative guard kicks in if needed. |
| All negative | We return `maxSum` (the least-negative element). |
| All positive | Wrapping = total = max Kadane. They coincide. |
| `[1, -1]` | maxSum = 1, total = 0, minSum = -1, wrapping = 0 - (-1) = 1. Answer 1. ✓ |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why doesn't running Kadane on the doubled array `nums + nums` work directly?"**
   It almost does, but you'd need to limit the subarray length to `n` to avoid using the same element twice. With careful constraints it works (a deque-based sliding window over the doubled array), but the total-minus-min-middle trick is cleaner.

2. **"What if the array could wrap multiple times?"**
   Then with all positives the answer would be infinite. The problem typically restricts to one wrap.

3. **"Find the actual subarray, not just its sum."**
   Track start/end indices for both the Kadane max and min. If the wrapping case wins, the answer is the **complement** of the min subarray.

4. **"Can you do it without computing the total?"**
   Yes, but harder — the cleanest solutions all use `total - minSum`. The total is O(n), so it's free.

---

## Key Lessons from This Problem

1. **"Wrapping = total − worst middle slice"** is a beautiful inversion. Remember this trick — it generalizes to many circular problems.
2. **Run Kadane twice** — once for max, once for min — when a problem has a "min/max-of-something" twist.
3. **Always check the all-negative edge case** when your formula could produce an "empty subarray" answer.
