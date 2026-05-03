# Problem: Why is Binary Search O(log n)?

## Category
Intro to DSA / Complexity

## Difficulty
Easy

---

## Problem Statement

Explain — in plain language and with math — why **binary search** has time complexity **O(log n)**.

---

## Clarifying Questions to Ask the Interviewer

1. **Is the array sorted?** Yes — binary search requires a sorted array. (On unsorted data, you'd be back to linear search.)
2. **Iterative or recursive implementation?** Doesn't matter for the complexity — both are O(log n).

---

## Approach (Plain-English Explanation)

### What does binary search do?

You have a sorted array. You want to find a target value. The trick: instead of checking each element one by one (linear search, O(n)), you **check the middle element**:

- If the middle equals the target, you're done.
- If the target is **smaller** than the middle, throw away the right half — the target can only be on the left.
- If the target is **larger** than the middle, throw away the left half.

Then you repeat on the half that's left.

### Why this is fast — each step halves the work

Suppose you start with `n` elements. After:

- **Step 1:** at most `n / 2` elements remain.
- **Step 2:** at most `n / 4` remain.
- **Step 3:** at most `n / 8` remain.
- ...
- **Step k:** at most `n / 2ᵏ` remain.

You stop when only **1 element** is left. Setting `n / 2ᵏ = 1`:

```
2ᵏ = n
k  = log₂(n)
```

So binary search takes **at most log₂(n) steps**. That's the definition of O(log n).

### Concrete numbers

For n = 1,000,000:

- Linear search: up to **1,000,000** comparisons.
- Binary search: at most **20** comparisons (because 2²⁰ > 10⁶).

For n = 1,000,000,000:

- Linear search: up to **1 billion** comparisons.
- Binary search: at most **30** comparisons.

This is why binary search is one of the most powerful ideas in CS — it scales effortlessly.

### Walking through an example

Find `7` in `[1, 3, 5, 7, 9, 11, 13]` (n = 7):

```
Step 1: range [1, 3, 5, 7, 9, 11, 13]    middle = 7  ← FOUND!
```

Find `3`:

```
Step 1: range [1, 3, 5, 7, 9, 11, 13]    middle = 7   3 < 7, go left
Step 2: range [1, 3, 5]                  middle = 3  ← FOUND!
```

Find `12` (not present):

```
Step 1: range [1, 3, 5, 7, 9, 11, 13]    middle = 7    12 > 7, go right
Step 2: range [9, 11, 13]                middle = 11   12 > 11, go right
Step 3: range [13]                       middle = 13   12 < 13, go left
Step 4: range []                         empty — return -1
```

Notice: only **4 comparisons** needed in the worst case for 7 elements. log₂(7) ≈ 2.8, and we need **at most ⌈log₂(n)⌉ + 1 = 3 to 4** steps.

---

## Time Complexity

**O(log n)** — proof above.

## Space Complexity

- **Iterative:** O(1) — just a few index variables.
- **Recursive:** O(log n) — one stack frame per step.

---

## Code (Java)

```java
int binarySearch(int[] a, int target) {
    int lo = 0, hi = a.length - 1;     // search range [lo, hi]

    while (lo <= hi) {
        // Why "lo + (hi - lo) / 2" instead of "(lo + hi) / 2"?
        // Because (lo + hi) can overflow int when lo and hi are huge.
        // The subtraction-first form stays safely within int range.
        int mid = lo + (hi - lo) / 2;

        if (a[mid] == target) {
            return mid;                 // found it
        } else if (a[mid] < target) {
            lo = mid + 1;               // target is in the right half
        } else {
            hi = mid - 1;               // target is in the left half
        }
    }
    return -1;                          // exhausted the range — not found
}
```

### Common bug — off-by-one

If you write `while (lo < hi)` (instead of `<=`), you skip checking the case where `lo == hi`. That misses the target if it's the very last remaining element. Always be deliberate about the loop condition.

---

## Code (Python)

```python
def binary_search(a, target):
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == target:
            return mid
        if a[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

(Python ints are arbitrary precision, so the overflow trick isn't strictly needed. But it's still a good habit.)

---

## Edge Cases

| Case | What happens |
|---|---|
| Empty array | Loop doesn't execute. Return `-1`. ✓ |
| Target is the first / last element | Found within the first few steps. |
| Target is smaller than min / larger than max | One side runs out, loop exits with `-1`. |
| Duplicate values | Returns *some* matching index (not necessarily the first). To find the first/last, you need a modified binary search — see `searching/first-occurrence.md`. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why won't binary search work efficiently on a linked list?"**
   Finding the middle element of a linked list takes O(n) (you have to walk from the head). So each "step" of binary search costs O(n), and the total is O(n log n) — worse than linear search!

2. **"How would you find the first occurrence of a duplicate value?"**
   When `a[mid] == target`, don't stop — instead set `hi = mid - 1` and remember `mid` as the best found so far. Keep searching the left half.

3. **"What does 'binary search on the answer' mean?"**
   Some problems aren't about searching an array — they're about finding the smallest (or largest) value of `x` for which a condition is true. If the condition is **monotonic** (once true, stays true), you can binary-search the answer space. See `searching/ship-capacity.md` for an example.

4. **"Why does the base of the logarithm not matter in O(log n)?"**
   `log₂(n) = log₁₀(n) / log₁₀(2)`. The conversion is a constant factor, and Big-O ignores constants.

---

## Key Lessons from This Problem

1. **Halving is exponentially powerful.** Cutting the work by half each step turns 10⁹ into 30 — that's the magic of logarithms.
2. **Binary search needs a sorted (or "monotonic") structure.** Without that, the "throw away half" idea doesn't work.
3. **Beware off-by-one bugs.** Inclusive vs exclusive bounds, `<` vs `<=` — these are the most common sources of binary-search bugs. Pick a convention and stick with it.
