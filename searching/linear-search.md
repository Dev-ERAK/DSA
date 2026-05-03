# Problem: Linear Search

## Category
Searching

## Difficulty
Easy

---

## Problem Statement

Given an array `nums` and a `target` value, return the **index** of the target if it appears in the array, or `-1` if it doesn't.

This is the most basic search algorithm — useful when the array isn't sorted (so binary search doesn't apply).

### Examples

```
nums = [3, 5, 1, 7, 9], target = 7   →  3
nums = [3, 5, 1, 7, 9], target = 4   →  -1
nums = [], target = 1                →  -1
```

---

## Clarifying Questions to Ask the Interviewer

1. **Is the array sorted?** If yes, prefer binary search (O(log n) vs O(n)). Linear search works regardless.
2. **Multiple occurrences — return first or last?** Usually first.
3. **Allowed to use built-ins?** Often no — the question is about implementing the algorithm.

---

## Approach

Walk through the array, comparing each element with the target. Return the index of the first match. If no match, return `-1`.

```
for i in 0..n-1:
    if nums[i] == target: return i
return -1
```

That's it. Linear search is the simplest possible algorithm — but it has O(n) cost because each call may have to scan the whole array.

### Why "linear"?

The number of operations grows **linearly** with input size: `n` array elements → up to `n` comparisons.

---

## Time Complexity

**O(n)** — worst case, must scan everything.
**O(1)** best case — target is at index 0.
**O(n/2)** average — target is somewhere in the middle.

(For Big-O, average and worst are both O(n).)

## Space Complexity

**O(1)** — no extra storage.

---

## Code (Java)

```java
public int linearSearch(int[] nums, int target) {
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] == target) return i;
    }
    return -1;
}
```

---

## Code (Python)

```python
class Solution:
    def linearSearch(self, nums, target: int) -> int:
        for i, x in enumerate(nums):
            if x == target:
                return i
        return -1
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty array | `-1` |
| Target absent | `-1` |
| Duplicates | Returns the first occurrence |
| Single element matches | 0 |

---

## Follow-up Questions an Interviewer Might Ask

1. **"First or last occurrence?"**
   - First: return on the first match (the code above).
   - Last: scan from the right, or scan left-to-right and track the latest match.

2. **"Sorted array?"**
   Use binary search — O(log n) instead of O(n). See `binary-search.md`.

3. **"Search a 2D grid?"**
   Either flatten to 1D or scan row by row — O(m × n).

4. **"Streaming search — values arrive one at a time, can't restart?"**
   You can only do linear search. Track the answer as values arrive.

5. **"Search with custom comparator (e.g., find nearest below target)?"**
   Track the best candidate as you scan; return it at the end.

---

## Key Lessons from This Problem

1. **Linear search is the universal fallback** — works on any list, regardless of structure.
2. **It's O(n)** — fine for small inputs, too slow for huge ones.
3. **For sorted data, do better** — binary search drops to O(log n).
