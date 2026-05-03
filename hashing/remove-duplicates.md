# Problem: Remove Duplicates from Array

## Category
Hashing / Two Pointers

## Difficulty
Easy

---

## Problem Statement

Given an integer array, remove duplicates so each value appears only once. Return the deduplicated array.

There are **two common variants**:

1. **Unsorted array** — preserve first-occurrence order, return a new (smaller) array.
2. **Sorted array** — remove duplicates in place, return the new length (LeetCode #26).

We'll cover both.

### Examples

```
Variant 1 — unsorted:
  [3, 1, 2, 1, 3, 5, 1]   →   [3, 1, 2, 5]   (first-occurrence order)

Variant 2 — sorted, in place (LeetCode 26):
  [1, 1, 2, 3, 3]   →   length = 3
                          first 3 elements are now [1, 2, 3]
                          (the rest of the array can hold anything)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Is the input sorted?** Different algorithms apply.
2. **Preserve order?** Almost always yes — first-occurrence order.
3. **Modify in place, or return a new array?** Sorted variant is in place.
4. **Return the deduplicated array, or just the length?** Sorted-in-place returns length.

---

## Approach

### Variant 1 — Unsorted, HashSet (preserve first-occurrence order)

Walk the array. For each value, if you haven't seen it before, append to the output and mark it seen.

- **`LinkedHashSet`** in Java preserves insertion order.
- In Python, you can use a regular `set` for membership and a `list` for order.

This is **O(n)** time, **O(n)** space.

### Variant 2 — Sorted, Two Pointers (in place)

When the array is sorted, **duplicates are adjacent**. We can rewrite in place:

- Use a "write" pointer `w` for where to write the next unique value.
- Use a "read" pointer `r` to scan through.
- Whenever `nums[r] != nums[w-1]` (a new unique value), write it at `nums[w]` and advance `w`.

This is **O(n)** time, **O(1)** extra space — no hashmap needed!

#### Walking through Variant 2 — `[1, 1, 2, 3, 3]`

```
Initial:  nums = [1, 1, 2, 3, 3], w = 1
                   ↑w
                  (the first element is automatically "kept")

r = 1: nums[1] = 1, nums[w-1] = 1 → same, skip
r = 2: nums[2] = 2, nums[w-1] = 1 → new! write nums[1] = 2, w = 2
       nums = [1, 2, 2, 3, 3]
r = 3: nums[3] = 3, nums[w-1] = 2 → new! write nums[2] = 3, w = 3
       nums = [1, 2, 3, 3, 3]
r = 4: nums[4] = 3, nums[w-1] = 3 → same, skip

Final: w = 3.  First 3 elements [1, 2, 3] are unique. The rest is undefined.
```

#### Why it works

Since the array is sorted, equal elements form contiguous blocks. As we scan, the only thing we need to compare against is the **most recent unique value** (which lives at `nums[w-1]`). Anything different from that must be a new unique.

---

## Time Complexity

Both variants: **O(n)**.

## Space Complexity

- Variant 1 (unsorted, HashSet): **O(n)**.
- Variant 2 (sorted, in place): **O(1)**.

---

## Code (Java)

### Variant 1 — Unsorted, preserve order

```java
public int[] removeDuplicates(int[] nums) {
    Set<Integer> seen = new LinkedHashSet<>();    // keeps insertion order
    for (int x : nums) {
        seen.add(x);
    }
    int[] out = new int[seen.size()];
    int i = 0;
    for (int x : seen) {
        out[i++] = x;
    }
    return out;
}
```

### Variant 2 — Sorted, in place (LeetCode 26)

```java
public int removeDuplicatesSorted(int[] nums) {
    if (nums.length == 0) return 0;
    int w = 1;                         // first element is kept

    for (int r = 1; r < nums.length; r++) {
        if (nums[r] != nums[w - 1]) {
            nums[w] = nums[r];
            w++;
        }
    }
    return w;                          // new length
}
```

---

## Code (Python)

```python
class Solution:
    # Variant 1 — unsorted, preserve order
    def removeDuplicates(self, nums):
        seen = set()
        out = []
        for x in nums:
            if x not in seen:
                seen.add(x)
                out.append(x)
        return out

    # Variant 2 — sorted, in place
    def removeDuplicatesSorted(self, nums):
        if not nums:
            return 0
        w = 1
        for r in range(1, len(nums)):
            if nums[r] != nums[w - 1]:
                nums[w] = nums[r]
                w += 1
        return w
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty array | New length 0; output empty. |
| All same value | New length 1. |
| All unique | New length n; array unchanged. |
| Two-element duplicate (`[1, 1]`) | New length 1. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Allow each element at most K times (LeetCode 80)."**
   Modify Variant 2: replace `nums[r] != nums[w - 1]` with `r < k || nums[r] != nums[w - k]`. This compares against the value `k` positions back, allowing up to K duplicates in a row.

2. **"Linked list version (LeetCode 83 / 82)."**
   Walk and skip equal consecutive nodes. For "remove all that have duplicates" (LeetCode 82), you need a slightly more careful traversal.

3. **"Memory-bounded streaming dedup."**
   Use a **Bloom filter** for probabilistic membership tests (occasional false positives, fixed memory).

4. **"Why doesn't the in-place trick work on unsorted arrays?"**
   Because duplicates aren't adjacent. To dedup an unsorted array in place without extra memory, you'd need to sort first (O(n log n)) — defeating the point.

---

## Key Lessons from This Problem

1. **Sorted arrays unlock O(1) extra space** — duplicates being adjacent is a powerful property.
2. **Two pointers (read + write)** is a recurring pattern for in-place rewriting of arrays.
3. **`LinkedHashSet` preserves insertion order** in Java — useful when the regular `HashSet` would scramble it.
