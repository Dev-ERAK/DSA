# Problem: Subarray Sum Equals K

## Category
Arrays / Prefix Sum / Hashing

## Difficulty
Medium

---

## Problem Statement

Given an integer array `nums` and an integer `k`, return the **number of contiguous subarrays** whose sum equals `k`.

A **subarray** is a contiguous slice — like `nums[2..4]`. (Not the same as a *subset*, which can skip elements.)

### Example

```
Input:  nums = [1, 1, 1], k = 2
Output: 2

Why? Subarrays summing to 2:
- nums[0..1] = [1, 1]
- nums[1..2] = [1, 1]
That's 2 subarrays.
```

```
Input:  nums = [1, 2, 3], k = 3
Output: 2     ([1,2] and [3])
```

This is **LeetCode #560**.

---

## Clarifying Questions to Ask the Interviewer

1. **Can `nums` contain negative numbers?** Yes. **This matters a lot** — negatives break the sliding-window approach you might be tempted to use.
2. **Can `k` be 0 or negative?** Yes — the technique handles both.
3. **Do we count the *number* of subarrays, or list them?** Count.

---

## Approach

### Why brute force is O(n²)

Try every starting index `i`, then every ending index `j ≥ i`, and check if the sum is `k`. That's `n²/2` subarrays. For `n = 10⁵` that's 5 billion checks — too slow.

### Why sliding window doesn't work here

If all numbers were **positive**, you could use a sliding window: grow it when the sum is too small, shrink when too big. But with negatives, the sum is **not monotonic** as you grow the window — so you can't decide whether to grow or shrink.

Example: `nums = [1, -1, 1, -1, 1]`, `k = 1`. As you slide a window forward, the sum bounces around. Sliding window can't handle this.

### The Big Idea — Prefix Sums + Hashmap

Recall **prefix sums**: `prefix[i]` = sum of `nums[0..i-1]`. The sum of any subarray `nums[l..r]` is:

```
sum(l..r) = prefix[r+1] - prefix[l]
```

We want subarrays where this equals `k`:

```
prefix[r+1] - prefix[l] = k
prefix[l] = prefix[r+1] - k
```

So: **for each ending position `r+1`, count how many earlier prefix values equal `prefix[r+1] - k`.**

To count efficiently, store prefix-sum frequencies in a hashmap as you go.

### The algorithm

1. Initialize a hashmap `count = {0: 1}`. (We seed it with `prefix=0` having frequency 1, to count subarrays that start at index 0.)
2. Walk through `nums`, maintaining a running prefix sum.
3. For each new prefix value, look up `prefix - k` in the hashmap. Add that frequency to the answer.
4. Then add the current prefix to the hashmap.

### Walking through an example

`nums = [1, 2, 3]`, `k = 3`:

| step | x | prefix | look up `prefix - k` = ? | count[?] | answer | hashmap after |
|---|---|---|---|---|---|---|
| start | - | 0 | - | - | 0 | `{0: 1}` |
| 1 | 1 | 1 | -2 | 0 | 0 | `{0:1, 1:1}` |
| 2 | 2 | 3 | 0 | 1 | 1 | `{0:1, 1:1, 3:1}` |
| 3 | 3 | 6 | 3 | 1 | 2 | `{0:1, 1:1, 3:1, 6:1}` |

Answer: **2** ✓

The "+1" at step 2 corresponds to the subarray `[1, 2]` (prefix went from 0 to 3, so `[1, 2]` sums to `3 - 0 = 3`). The "+1" at step 3 is for `[3]` (prefix went from 3 to 6, so just the third element sums to 3).

### Why we seed `count = {0: 1}`

Without it, we'd miss subarrays starting at index 0. The "empty prefix" has sum 0 and exists once — that's exactly `count[0] = 1` at start.

---

## Time Complexity

**O(n)** — one pass; each hashmap operation is O(1) average.

## Space Complexity

**O(n)** — the hashmap can grow to hold n distinct prefix sums.

---

## Code (Java)

```java
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> count = new HashMap<>();
    count.put(0, 1);                  // seed: empty prefix has sum 0

    int prefix = 0;
    int answer = 0;

    for (int x : nums) {
        prefix += x;

        // How many earlier prefixes equal (prefix - k)?
        // Each such earlier prefix means there's a subarray ending here
        // whose sum is k.
        answer += count.getOrDefault(prefix - k, 0);

        // Record this prefix value.
        count.merge(prefix, 1, Integer::sum);
    }
    return answer;
}
```

---

## Code (Python)

```python
from collections import defaultdict

class Solution:
    def subarraySum(self, nums, k: int) -> int:
        count = defaultdict(int)
        count[0] = 1                  # seed for subarrays starting at index 0

        prefix = 0
        answer = 0

        for x in nums:
            prefix += x
            answer += count[prefix - k]
            count[prefix] += 1

        return answer
```

---

## Edge Cases

| Case | What happens |
|---|---|
| Single element equal to k | The seed `{0: 1}` means we find it: prefix=k, look up `k - k = 0`, find frequency 1. |
| `k = 0` and array has zeros | Each zero-sum window contributes — handled correctly. |
| All negatives | Same algorithm — prefix can decrease, hashmap doesn't care. |
| Empty array | Returns 0 (loop never executes). |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if k could be very large but the answer fits in int?"** No change needed — prefix sums use whatever integer type you have. Just use `long` if values can overflow `int`.

2. **"What if you only need to know whether *any* such subarray exists?"** Same algorithm — return `true` as soon as `count[prefix - k] > 0`.

3. **"Find the longest subarray with sum k."** Slight variation: store the **first index** where each prefix value occurred (instead of frequency). When you see `prefix - k` in the map, length is `current_index - first_index`.

4. **"What about subarrays whose sum is divisible by k?"** Hash on `prefix % k` (carefully handle negatives with `((prefix % k) + k) % k`). LeetCode 974.

---

## Key Lessons from This Problem

1. **Prefix sums + hashmap** is one of the most useful combos in array problems. Memorize this pattern.
2. **Sliding window only works on monotonic sums** — usually meaning all positive numbers. Negatives break it.
3. **The seed `{0: 1}`** is the trick that saves a special case for subarrays starting at index 0.
