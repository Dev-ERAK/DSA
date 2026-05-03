# Problem: Contains Duplicate

## Category
Hashing

## Difficulty
Easy

---

## Problem Statement

Given an integer array `nums`, return `true` if **any value** appears at least twice. Return `false` if every value is unique.

This is **LeetCode #217**.

### Examples

```
[1, 2, 3, 1]   →  true   (1 appears twice)
[1, 2, 3, 4]   →  false  (all unique)
[]             →  false
[5]            →  false
```

---

## Clarifying Questions to Ask the Interviewer

1. **Are we allowed extra memory?** Yes — hashset is the standard answer.
2. **Can the array be empty?** Yes — return false.
3. **Negative numbers, large numbers?** Yes, no issue.

---

## Approach

### Method 1 — HashSet (Optimal, O(n))

Walk the array. For each value, check if it's already in a hashset. If yes → duplicate found, return `true`. Otherwise add it.

This is the cleanest solution for any unsorted, unbounded input.

### Method 2 — Sort First (O(n log n), O(1) extra space)

Sort the array. Adjacent duplicates are now next to each other; one scan finds them.

Trade-off: less memory, more time.

### Method 3 — Pythonic shortcut

```python
return len(nums) != len(set(nums))
```

If duplicates exist, the set will be smaller than the list. Same complexity as Method 1.

### Walking through `[1, 2, 3, 1]`

```
seen = {}

x = 1: not in seen → add. seen = {1}
x = 2: not in seen → add. seen = {1, 2}
x = 3: not in seen → add. seen = {1, 2, 3}
x = 1: 1 in seen → return TRUE
```

### Walking through `[1, 2, 3, 4]`

```
seen = {}
x = 1: add. seen = {1}
x = 2: add. seen = {1, 2}
x = 3: add. seen = {1, 2, 3}
x = 4: add. seen = {1, 2, 3, 4}
End of loop → return FALSE
```

---

## Time Complexity

- HashSet: **O(n)** average.
- Sort: **O(n log n)**.
- Pythonic `set(nums)`: **O(n)** average.

## Space Complexity

- HashSet / Pythonic: **O(n)**.
- Sort (in place): **O(1)** extra.

---

## Code (Java)

```java
public boolean containsDuplicate(int[] nums) {
    Set<Integer> seen = new HashSet<>(nums.length);

    for (int x : nums) {
        // HashSet.add returns false if the element was already present.
        if (!seen.add(x)) {
            return true;
        }
    }
    return false;
}
```

> Pre-sizing the hashset to `nums.length` avoids rehashing as it grows — a small constant-factor win.

---

## Code (Python)

```python
class Solution:
    # Cleanest
    def containsDuplicate(self, nums) -> bool:
        return len(nums) != len(set(nums))

    # Explicit early-exit version (slightly faster on average)
    def containsDuplicate_early(self, nums) -> bool:
        seen = set()
        for x in nums:
            if x in seen:
                return True
            seen.add(x)
        return False
```

> The early-exit version is faster when a duplicate appears early — it doesn't have to build the full set.

---

## Edge Cases

| Case | Result |
|---|---|
| Empty array | `false` (no values, can't have duplicates). |
| Single element | `false`. |
| All same | `true` (very early exit). |
| All distinct | `false` (must scan the whole array). |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Contains duplicate within K positions (LeetCode 219)."**
   "Are there `i, j` with `nums[i] == nums[j]` and `|i - j| <= k`?"
   - Maintain a sliding window of the last K elements as a hashset.
   - For each new element, check if it's in the window. Then add it; if window size > K, remove the oldest.

2. **"Contains nearby almost duplicate (LeetCode 220)."**
   `|nums[i] - nums[j]| <= t` and `|i - j| <= k`. Use **bucket sort** by value/(t+1) — O(n).

3. **"Streaming version with bounded memory?"**
   Bloom filter — occasional false positives, fixed memory.

4. **"What's the worst-case time for the hashset version?"**
   O(n) average, but with a bad hash function or adversarial input, O(n²) is possible. For interview purposes, "O(n)" is the expected answer.

---

## Key Lessons from This Problem

1. **HashSet is the simplest tool for "have I seen this?"** Memorize the pattern.
2. **`add()` returns false if the element was already there** — handy for early exit.
3. **The sort-and-scan approach** trades time for space. Useful when memory is tight.
