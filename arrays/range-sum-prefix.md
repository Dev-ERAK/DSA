# Problem: Range Sum Query (Prefix Sum)

## Category
Arrays / Prefix Sum

## Difficulty
Easy

---

## Problem Statement

You're given an array of numbers. You'll be asked many times: "What's the sum of `nums[i]` to `nums[j]` (inclusive)?"

Design a data structure that **preprocesses** the array once, then answers each query in **O(1)**.

This is **LeetCode #303** ("Range Sum Query - Immutable").

### Example

```
nums = [-2, 0, 3, -5, 2, -1]

sumRange(0, 2)  →  1     (-2 + 0 + 3)
sumRange(2, 5)  →  -1    (3 + (-5) + 2 + (-1))
sumRange(0, 5)  →  -3    (sum of everything)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Are the queries inclusive on both endpoints?** Yes (typical convention).
2. **Will the array change between queries?** No — this version assumes the array is **immutable** after construction. (If it changes, you need a Fenwick tree or segment tree — that's a harder problem.)
3. **Can values be negative?** Yes — the technique still works.

---

## Approach

### The Naive Way — O(n) per query

For each query `(i, j)`, sum the elements from `i` to `j`. With `q` queries, total cost is `O(q × n)`. Too slow when both `q` and `n` are large.

### The Big Idea — Precompute Cumulative Sums

If you've ever computed your bank balance from a list of transactions, you've used a prefix sum: at any point, your balance is the sum of all transactions up to that day. To find your spending between Day 5 and Day 10, you just compute `balance[10] - balance[4]`.

#### What is a prefix sum?

`prefix[i]` = sum of the first `i` elements of `nums`.

```
nums   = [-2,  0,  3, -5,  2, -1]
prefix = [ 0, -2, -2,  1, -4, -2, -3]    ← length is n + 1
            ↑   ↑   ↑   ↑   ↑   ↑   ↑
       prefix[0] = 0 (empty prefix)
       prefix[1] = nums[0]              = -2
       prefix[2] = nums[0] + nums[1]    = -2 + 0 = -2
       prefix[3] = -2 + 0 + 3           = 1
       prefix[4] = -2 + 0 + 3 + (-5)    = -4
       prefix[5] = -4 + 2               = -2
       prefix[6] = -2 + (-1)            = -3
```

#### How does this answer range queries?

```
sumRange(i, j) = prefix[j + 1] - prefix[i]
```

Why? `prefix[j+1]` is the sum of `nums[0..j]`. `prefix[i]` is the sum of `nums[0..i-1]`. Subtracting gives the sum of `nums[i..j]`.

#### Quick check with the example

`sumRange(2, 5)`:
- `prefix[6] = -3`
- `prefix[2] = -2`
- Answer: `-3 - (-2) = -1` ✓

#### Why we use length `n + 1`, with `prefix[0] = 0`

That extra zero at the start lets us handle `i = 0` without a special case. `sumRange(0, j) = prefix[j+1] - prefix[0] = prefix[j+1] - 0`. Clean.

---

## Time Complexity

- **Preprocessing:** O(n) — one pass over the array.
- **Each query:** O(1) — one subtraction.

## Space Complexity

**O(n)** — for the prefix array.

---

## Code (Java)

```java
class NumArray {
    private final int[] prefix;

    public NumArray(int[] nums) {
        // prefix has length n + 1; prefix[0] = 0 by default.
        prefix = new int[nums.length + 1];

        // Build the cumulative sum.
        // prefix[i + 1] = prefix[i] + nums[i]
        for (int i = 0; i < nums.length; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }
    }

    public int sumRange(int i, int j) {
        // Sum of nums[i..j] inclusive.
        return prefix[j + 1] - prefix[i];
    }
}
```

---

## Code (Python)

```python
class NumArray:
    def __init__(self, nums):
        # prefix[0] = 0; prefix[i+1] = sum(nums[0..i])
        self.prefix = [0] * (len(nums) + 1)
        for i, x in enumerate(nums):
            self.prefix[i + 1] = self.prefix[i] + x

    def sumRange(self, i: int, j: int) -> int:
        return self.prefix[j + 1] - self.prefix[i]
```

---

## Edge Cases

| Case | What happens |
|---|---|
| `i == j` | Returns just `nums[i]`. The formula still works: `prefix[i+1] - prefix[i] = nums[i]`. |
| `i == 0` | The `prefix[0] = 0` sentinel means we don't need a special case. |
| Empty array | Only `sumRange` calls would be invalid; constructor still works (creates `prefix = [0]`). |
| Negative values | All arithmetic still works — prefix can decrease. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"How would you handle a 2D matrix? sumRegion(r1, c1, r2, c2)?"**
   Use a **2D prefix sum**. Define `pre[i][j]` = sum of the rectangle from `(0,0)` to `(i-1, j-1)`. Then:
   ```
   sumRegion(r1, c1, r2, c2)
       = pre[r2+1][c2+1]
         - pre[r1][c2+1]
         - pre[r2+1][c1]
         + pre[r1][c1]
   ```
   Same idea: subtract overlapping regions, with an inclusion-exclusion correction. Query is still O(1).

2. **"What if the array can be updated?"**
   Prefix sums don't help — a single update would require rebuilding O(n) of the prefix. Use a **Fenwick tree (Binary Indexed Tree)** or **segment tree** for O(log n) per update and per query.

3. **"What if queries are on a range update, point read instead?"**
   Use the dual structure: a **difference array**. Increment `diff[l]` by `v` and decrement `diff[r+1]` by `v`; the value at any point is the prefix sum of `diff`.

4. **"Can prefix sum solve 'subarray sum equals K'?"**
   Yes! Combine with a hashmap. See `arrays/subarray-sum-k.md`.

---

## Key Lessons from This Problem

1. **Trade preprocessing for query speed.** O(n) once, O(1) forever after — this is one of the most useful patterns in array problems.
2. **Use a sentinel `prefix[0] = 0`** to avoid special cases at the boundary.
3. **Prefix sums show up everywhere** — running totals, range queries, even some hashmap problems. Recognize the pattern.
