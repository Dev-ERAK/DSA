# Problem: Longest Increasing Subsequence (LIS) — O(n²)

## Category
Dynamic Programming

## Difficulty
Medium

---

## Problem Statement

Given an integer array `nums`, return the **length** of the longest **strictly increasing subsequence**.

A **subsequence** preserves order but doesn't have to be contiguous (you can skip elements).

This is **LeetCode #300**.

### Examples

```
[10, 9, 2, 5, 3, 7, 101, 18]   →  4
   The LIS is [2, 3, 7, 101] or [2, 3, 7, 18] or [2, 5, 7, 101] (any of length 4).

[0, 1, 0, 3, 2, 3]   →  4   ([0, 1, 2, 3])
[7, 7, 7, 7]         →  1   (strict — duplicates don't count)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Strictly increasing or non-decreasing?** Strictly (LeetCode 300).
2. **Length only, or the subsequence itself?** Length here. Tracking the subsequence requires extra bookkeeping.
3. **Need O(n log n)?** Different problem — see `lis-nlogn.md`.

---

## Approach — DP O(n²)

### The DP idea

Define `dp[i]` = length of the LIS that **ends** at index `i`.

For each `i`, we consider every earlier index `j < i`. If `nums[j] < nums[i]`, then we can extend the LIS ending at `j` by appending `nums[i]`:

```
dp[i] = 1 + max(dp[j] for j in 0..i-1 if nums[j] < nums[i])
```

If no such `j` exists, `dp[i] = 1` (the subsequence is just `[nums[i]]`).

The final answer is `max(dp)` — the longest LIS ending anywhere.

### Walking through `[10, 9, 2, 5, 3, 7, 101, 18]`

```
i = 0: nums[0] = 10. No earlier elements. dp[0] = 1.
i = 1: nums[1] = 9.  9 > 10? No. dp[1] = 1.
i = 2: nums[2] = 2.  2 > anything before? No. dp[2] = 1.
i = 3: nums[3] = 5.  5 > 2 (j=2, dp=1). dp[3] = 1 + 1 = 2.
i = 4: nums[4] = 3.  3 > 2 (j=2, dp=1). dp[4] = 1 + 1 = 2.
i = 5: nums[5] = 7.  7 > 2(dp=1), 5(dp=2), 3(dp=2). max=2. dp[5] = 3.
i = 6: nums[6] = 101. > 10,9,2,5,3,7. max(dp) over those = 3. dp[6] = 4.
i = 7: nums[7] = 18.  > 10,9,2,5,3,7. max = 3 (at index 5). dp[7] = 4.

dp = [1, 1, 1, 2, 2, 3, 4, 4]
max(dp) = 4.   ✓
```

### Why O(n²)?

For each of the `n` indices, we look at all earlier indices — that's `O(n)` per index → **O(n²)** total.

For `n` up to maybe 2000, this is fine. For larger `n`, see `lis-nlogn.md`.

---

## Time Complexity

**O(n²)**

## Space Complexity

**O(n)** for the `dp` array.

---

## Code (Java)

```java
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    if (n == 0) return 0;

    int[] dp = new int[n];
    Arrays.fill(dp, 1);                     // every element is its own length-1 LIS

    int best = 1;
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        best = Math.max(best, dp[i]);
    }
    return best;
}
```

---

## Code (Python)

```python
class Solution:
    def lengthOfLIS(self, nums) -> int:
        if not nums:
            return 0
        dp = [1] * len(nums)
        for i in range(1, len(nums)):
            for j in range(i):
                if nums[j] < nums[i]:
                    dp[i] = max(dp[i], dp[j] + 1)
        return max(dp)
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty array | 0 |
| Strictly decreasing | 1 (no element extends another) |
| All equal | 1 (strict comparison disqualifies duplicates) |
| Strictly increasing | n (every element extends) |
| Single element | 1 |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Can you do it in O(n log n)?"**
   Yes — see `lis-nlogn.md`. Uses binary search.

2. **"Reconstruct the actual LIS."**
   Track a parent pointer for each `i`: store the `j` that gave the best `dp[i]`. After finding the index with max `dp`, walk back via parents.

3. **"Number of LIS sequences (LeetCode 673)?"**
   Track both length and count: `count[i]` = number of distinct LIS ending at `i`. Update count appropriately when extending.

4. **"Longest *non-decreasing* subsequence?"**
   Replace `nums[j] < nums[i]` with `nums[j] <= nums[i]`.

---

## Key Lessons from This Problem

1. **"LIS ending here"** is a useful subproblem definition.
2. **Inner loop scans all previous indices** — that's where the O(n²) comes from.
3. **`dp[i] = 1` initial value** ensures every element is at least its own length-1 LIS.
