# Problem: Frequency of Each Element

## Category
Hashing

## Difficulty
Easy

---

## Problem Statement

Given an array `nums`, return how many times each unique value appears (a **frequency map**).

### Example

```
[1, 2, 2, 3, 3, 3]   →   {1: 1, 2: 2, 3: 3}
["a", "a", "b"]      →   {"a": 2, "b": 1}
[]                    →   {}
```

---

## Clarifying Questions to Ask the Interviewer

1. **Are values bounded (e.g., 0..1000)?** If so, a fixed-size array is faster than a hashmap. Otherwise hashmap.
2. **Need order preserved?** Use `LinkedHashMap` (Java) or `dict` (Python 3.7+ keeps insertion order).
3. **Return type?** Map / dict.

---

## Approach

This is the canonical use case for a hashmap. One pass is all you need.

### Step by step

1. Create an empty hashmap (or `Counter` in Python, which is a hashmap with extras).
2. For each element `x` in the array:
   - If `x` is already in the map, increment its count by 1.
   - Otherwise, set its count to 1.
3. Return the map.

### Walking through `[1, 2, 2, 3, 3, 3]`

```
Start: freq = {}

x = 1: freq = {1: 1}
x = 2: freq = {1: 1, 2: 1}
x = 2: freq = {1: 1, 2: 2}
x = 3: freq = {1: 1, 2: 2, 3: 1}
x = 3: freq = {1: 1, 2: 2, 3: 2}
x = 3: freq = {1: 1, 2: 2, 3: 3}
```

Done.

### Idiomatic shortcuts

- **Java:** `freq.merge(x, 1, Integer::sum)` — adds 1 if present, sets to 1 if absent.
- **Python:** `Counter(nums)` does this in one expression.

### When to use an int array instead of a hashmap

If the values are bounded (e.g., lowercase letters → use `int[26]`; ASCII → `int[128]`), an array is faster. No hashing overhead, better cache behavior.

```
For lowercase letters:
  count[ s.charAt(i) - 'a' ]++;
```

---

## Time Complexity

**O(n)** — one pass, O(1) per hashmap operation.

## Space Complexity

**O(k)** where `k` = number of unique values. Worst case O(n) when all elements are unique.

---

## Code (Java)

```java
public Map<Integer, Integer> frequency(int[] nums) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int x : nums) {
        freq.merge(x, 1, Integer::sum);
        // Equivalent to:  freq.put(x, freq.getOrDefault(x, 0) + 1);
    }
    return freq;
}

// Faster when values are bounded (e.g., 0..max):
public int[] frequencyBounded(int[] nums, int max) {
    int[] f = new int[max + 1];
    for (int x : nums) {
        f[x]++;
    }
    return f;
}
```

---

## Code (Python)

```python
from collections import Counter

class Solution:
    # Cleanest version
    def frequency(self, nums):
        return Counter(nums)

    # Manual version (for understanding)
    def frequency_manual(self, nums):
        f = {}
        for x in nums:
            f[x] = f.get(x, 0) + 1
        return f
```

> `Counter` also has handy methods like `most_common(k)` to get the k most frequent elements.

---

## Edge Cases

| Case | Result |
|---|---|
| Empty array | Empty map `{}`. |
| All same value | One entry: `{x: n}`. |
| All unique | n entries, each with value 1. |
| Negative numbers | Hashmap handles them; an int-array doesn't (would need an offset). |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Find the top-K most frequent elements (LeetCode 347)."**
   Build the frequency map, then either sort by frequency (O(n log n)) or use a min-heap of size K (O(n log K)).

2. **"Find the most frequent element."**
   Walk the map and track max. Or use `Counter.most_common(1)` in Python.

3. **"Streaming version with bounded memory?"**
   Use **Misra-Gries** (for top-k frequent) or **Count-Min Sketch** for approximate counts.

4. **"Ties broken by which element comes first?"**
   Track insertion order (`LinkedHashMap` / standard Python dict).

---

## Key Lessons from This Problem

1. **Hashmaps are the natural tool for "count things."**
2. **Use bounded arrays when possible** — they're faster than hashmaps when the value range is small.
3. **`Counter`** in Python is a one-liner — use it when you can.
