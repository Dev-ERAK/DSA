# Problem: Implement Merge Sort

## Category
Arrays / Sorting / Divide & Conquer

## Difficulty
Medium

---

## Problem Statement

Implement **Merge Sort** — a sorting algorithm that:
- Runs in **O(n log n)** in the worst case (no bad surprises like quicksort).
- Is **stable** (preserves the original order of equal elements).
- Uses **O(n) extra memory** for the merging step.

### Why merge sort matters

It's the canonical example of **divide & conquer** — a problem-solving strategy that splits work into halves, solves each half, and combines the results. Once you understand merge sort, many other algorithms become easy.

---

## Clarifying Questions to Ask the Interviewer

1. **Sorting integers, or generic objects?** Both work; the comparison function changes.
2. **In-place or allowed extra memory?** Classic merge sort uses O(n) extra. True in-place merge sort is much harder (and slower in practice).
3. **Stable required?** Yes, by convention.

---

## Approach

### Divide & Conquer in three steps

1. **Divide.** Split the array into two halves at the middle.
2. **Conquer.** Recursively sort each half.
3. **Combine (merge).** Walk both sorted halves with two pointers, picking the smaller current element each time.

### Why is the merge step O(n)?

Each of the two sorted halves has its own pointer (`i` for the left, `j` for the right). At each step, we compare `left[i]` and `right[j]` and put the smaller one into the output. Then advance the pointer we took from. Each element is moved exactly once → O(total size).

### The recursion tree — why O(n log n)

Each level of recursion does a total of O(n) work (the merging across all subarrays at that level sums to n).

There are **log₂ n** levels because we keep halving. So total: O(n × log n) = **O(n log n)**.

### Walking through merge sort on `[5, 2, 8, 1, 9, 3]`

```
Step 1 - Divide:
[5, 2, 8] | [1, 9, 3]

Step 2 - Recurse left half [5, 2, 8]:
   Divide: [5] | [2, 8]
   Recurse left: [5]  (already sorted, single element)
   Recurse right [2, 8]:
       Divide: [2] | [8]
       Both single elements, sorted.
       Merge: [2, 8]
   Merge [5] and [2, 8]:
       compare 5 vs 2 → take 2 → [2]
       compare 5 vs 8 → take 5 → [2, 5]
       drain right: [2, 5, 8]

Step 2 - Recurse right half [1, 9, 3]:
   Divide: [1] | [9, 3]
   Recurse left: [1]
   Recurse right [9, 3]:
       Divide: [9] | [3]
       Merge: [3, 9]
   Merge [1] and [3, 9]:
       1 < 3 → [1]
       drain right: [1, 3, 9]

Step 3 - Merge [2, 5, 8] and [1, 3, 9]:
   2 vs 1 → 1 → [1]
   2 vs 3 → 2 → [1, 2]
   5 vs 3 → 3 → [1, 2, 3]
   5 vs 9 → 5 → [1, 2, 3, 5]
   8 vs 9 → 8 → [1, 2, 3, 5, 8]
   drain right: [1, 2, 3, 5, 8, 9]
```

Final answer: `[1, 2, 3, 5, 8, 9]` ✓

### Why is merge sort stable?

When `left[i] == right[j]`, we take from the **left** first (using `<=` in the comparison). This means equal elements from the left half stay before equal elements from the right half — preserving their original relative order.

---

## Time Complexity

**O(n log n)** — best, average, and worst case all the same. Predictable performance.

## Space Complexity

**O(n)** — for the auxiliary array used during merging.
**O(log n)** — for the recursion stack.

---

## Code (Java)

```java
public void mergeSort(int[] a) {
    if (a.length < 2) return;            // already sorted
    int[] aux = new int[a.length];       // shared scratch space
    sort(a, aux, 0, a.length - 1);
}

// Recursively sort a[lo..hi] (inclusive on both ends).
private void sort(int[] a, int[] aux, int lo, int hi) {
    if (lo >= hi) return;
    int mid = lo + (hi - lo) / 2;
    sort(a, aux, lo, mid);               // sort left half
    sort(a, aux, mid + 1, hi);           // sort right half
    merge(a, aux, lo, mid, hi);          // combine
}

// Merge a[lo..mid] and a[mid+1..hi] into a[lo..hi].
private void merge(int[] a, int[] aux, int lo, int mid, int hi) {
    // Copy the section we're about to overwrite into aux.
    for (int k = lo; k <= hi; k++) aux[k] = a[k];

    int i = lo;          // pointer into left half (in aux)
    int j = mid + 1;     // pointer into right half (in aux)

    for (int k = lo; k <= hi; k++) {
        if (i > mid) {
            a[k] = aux[j++];                 // left exhausted
        } else if (j > hi) {
            a[k] = aux[i++];                 // right exhausted
        } else if (aux[i] <= aux[j]) {
            a[k] = aux[i++];                 // <= for stability
        } else {
            a[k] = aux[j++];
        }
    }
}
```

---

## Code (Python)

```python
def merge_sort(a):
    if len(a) < 2:
        return a
    mid = len(a) // 2
    left = merge_sort(a[:mid])
    right = merge_sort(a[mid:])
    return _merge(left, right)

def _merge(left, right):
    out = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:              # <= keeps it stable
            out.append(left[i]); i += 1
        else:
            out.append(right[j]); j += 1
    out.extend(left[i:])
    out.extend(right[j:])
    return out

# Usage:
# a = [5, 2, 8, 1, 9, 3]
# sorted_a = merge_sort(a)
```

---

## Edge Cases

| Case | What happens |
|---|---|
| Empty array | Length 0 — base case returns immediately. |
| Single element | Length 1 — base case returns immediately. |
| Already sorted | Still does O(n log n) work — merge sort isn't adaptive. |
| Reverse-sorted | Same O(n log n). The merging happens to take the maximum number of comparisons, but still linear. |
| Many duplicates | Stable sort keeps their original order. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What's the difference between merge sort and quicksort?"**
   - Merge sort: O(n log n) **always**. Stable. Uses O(n) extra memory.
   - Quicksort: O(n log n) **average**, O(n²) **worst case**. Usually not stable. In-place.
   - Most language standard libraries use a hybrid (e.g., Timsort in Python, dual-pivot quicksort in Java for primitives).

2. **"Why is merge sort preferred for linked lists?"**
   - Linked lists don't support fast random access, so quicksort's partition step is awkward.
   - Merge sort just walks the lists with pointers — no random access needed.
   - And the merge step doesn't need extra memory for linked lists (just rewire pointers).

3. **"How would you count inversions while sorting?"**
   An *inversion* is a pair `(i, j)` with `i < j` but `a[i] > a[j]`. During the merge step, when you take from the right half (`a[j]`), every remaining element in the left half is bigger than `a[j]` → that's `mid - i + 1` inversions. Add to the running count. Total: O(n log n).

4. **"Bottom-up merge sort?"**
   Instead of recursion, iterate `width = 1, 2, 4, 8, ...` and merge pairs of adjacent runs of that width. Same O(n log n), no recursion overhead.

---

## Key Lessons from This Problem

1. **Divide & conquer** = recurse on smaller subproblems, then combine.
2. **Stability matters** when sorting by a "secondary key" — the original order of equal items is preserved.
3. **The recurrence T(n) = 2T(n/2) + O(n) → O(n log n)** is one of the most important results in algorithms. Memorize it.
