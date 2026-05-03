# Problem: Kth Largest Element in an Array

## Category
Arrays / Heap / Quickselect

## Difficulty
Medium

---

## Problem Statement

Given an integer array `nums` and an integer `k`, return the **k-th largest** element. Note: this is the k-th largest in **sorted order**, not the k-th *distinct* value.

This is **LeetCode #215**.

### Example

```
Input:  nums = [3, 2, 1, 5, 6, 4], k = 2
Output: 5

Why? Sorted descending: [6, 5, 4, 3, 2, 1]. The 2nd is 5.

Input:  nums = [3, 2, 3, 1, 2, 4, 5, 5, 6], k = 4
Output: 4
```

---

## Clarifying Questions to Ask the Interviewer

1. **K-th largest, or k-th distinct largest?** K-th largest counting duplicates (LeetCode convention).
2. **Is `k` 1-indexed?** Yes — `k = 1` means the maximum.
3. **Do we need worst-case O(n), or is average-case O(n) fine?** Heap is guaranteed O(n log k); quickselect is average O(n).

---

## Approach

We'll cover three approaches, from easiest to fanciest.

### Method 1 — Sort and Index (O(n log n))

Sort ascending, return `nums[n - k]`. Simple but wastefully sorts elements you don't need.

```
nums = [3, 2, 1, 5, 6, 4]
sorted = [1, 2, 3, 4, 5, 6]
n = 6, k = 2
return sorted[6 - 2] = sorted[4] = 5
```

### Method 2 — Min-Heap of Size k (O(n log k))

A **min-heap** is a tree-shaped data structure where the root is always the smallest element. You can add to it and remove the minimum in O(log size).

**The idea:** keep a min-heap that always holds the **top k largest values seen so far**. When a new value arrives:
- If the heap has fewer than k elements, add it.
- Otherwise, if the new value is larger than the heap's minimum, swap them (pop the min, push the new value).

After processing all elements, the heap contains the k largest values, and the **root (the minimum of those)** is exactly the k-th largest.

#### Walking through with `nums = [3, 2, 1, 5, 6, 4]`, k = 2

| step | x | heap before | action | heap after |
|---|---|---|---|---|
| 1 | 3 | [] | size < k, add | [3] |
| 2 | 2 | [3] | size < k, add | [2, 3] |
| 3 | 1 | [2, 3] | 1 < min(2)? no — but heap full, push then pop | [2, 3] (1 was added, 1 was popped) |
| 4 | 5 | [2, 3] | push then pop min | [3, 5] |
| 5 | 6 | [3, 5] | push then pop min | [5, 6] |
| 6 | 4 | [5, 6] | 4 < min(5), nothing changes after push & pop | [5, 6] |

Heap root = 5 = answer. ✓

**Why O(n log k)?** Each of the n elements may cause one push and one pop, each O(log k).

### Method 3 — Quickselect (Average O(n))

Quickselect is to **finding** the k-th smallest what quicksort is to **sorting**. It uses the same partitioning trick — but only recurses on **one side** of the pivot.

#### What's a partition?

Pick a **pivot** value. Rearrange the array so that:
- Everything `< pivot` is on the left.
- Pivot ends up at index `p`.
- Everything `≥ pivot` is on the right.

After partitioning, you know exactly which sorted index the pivot occupies.

#### Quickselect logic

We want the element at sorted index `n - k` (the k-th largest). After partitioning:
- If `p == n - k` → the pivot is our answer. Done.
- If `p < n - k` → answer is on the right side; recurse there.
- If `p > n - k` → answer is on the left side; recurse there.

Because we recurse on only one half, the average work is O(n + n/2 + n/4 + ...) = **O(n)**.

#### The catch — worst case is O(n²)

If you keep picking unlucky pivots (e.g., always the smallest or largest remaining), each partition only shaves off one element → O(n²). Use a **random pivot** to make this exceedingly unlikely.

---

## Time Complexity

| Method | Time | Worst Case |
|---|---|---|
| Sort | O(n log n) | O(n log n) |
| Min-heap of size k | O(n log k) | O(n log k) |
| Quickselect (random pivot) | O(n) average | O(n²) |

## Space Complexity

| Method | Space |
|---|---|
| Sort | O(1) or O(log n) for in-place sorts |
| Heap | O(k) |
| Quickselect | O(1) extra + recursion |

---

## Code (Java) — Min-Heap

```java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> pq = new PriorityQueue<>();   // default = min-heap

    for (int x : nums) {
        pq.offer(x);
        if (pq.size() > k) pq.poll();          // keep only the k largest
    }

    return pq.peek();                           // root = smallest of top k
}
```

### Quickselect

```java
public int findKthLargestQS(int[] nums, int k) {
    int target = nums.length - k;               // sorted index of k-th largest
    int lo = 0, hi = nums.length - 1;
    Random rand = new Random();

    while (lo < hi) {
        int pivotIdx = lo + rand.nextInt(hi - lo + 1);
        int p = partition(nums, lo, hi, pivotIdx);

        if (p == target) return nums[p];
        if (p < target) lo = p + 1;
        else hi = p - 1;
    }
    return nums[lo];
}

private int partition(int[] a, int lo, int hi, int pivotIdx) {
    int pivot = a[pivotIdx];
    swap(a, pivotIdx, hi);                      // park pivot at the end
    int store = lo;
    for (int i = lo; i < hi; i++) {
        if (a[i] < pivot) {
            swap(a, i, store);
            store++;
        }
    }
    swap(a, store, hi);                         // put pivot in its final place
    return store;
}

private void swap(int[] a, int i, int j) {
    int t = a[i]; a[i] = a[j]; a[j] = t;
}
```

---

## Code (Python)

```python
import heapq
import random

class Solution:
    # Heap method — easiest to write
    def findKthLargest(self, nums, k: int) -> int:
        return heapq.nlargest(k, nums)[-1]

    # Manual heap (for understanding)
    def findKthLargest_manual(self, nums, k: int) -> int:
        heap = []
        for x in nums:
            heapq.heappush(heap, x)
            if len(heap) > k:
                heapq.heappop(heap)
        return heap[0]

    # Quickselect
    def findKthLargest_qs(self, nums, k: int) -> int:
        target = len(nums) - k

        def partition(lo, hi, pivot_idx):
            pivot = nums[pivot_idx]
            nums[pivot_idx], nums[hi] = nums[hi], nums[pivot_idx]
            store = lo
            for i in range(lo, hi):
                if nums[i] < pivot:
                    nums[i], nums[store] = nums[store], nums[i]
                    store += 1
            nums[store], nums[hi] = nums[hi], nums[store]
            return store

        lo, hi = 0, len(nums) - 1
        while lo < hi:
            p = partition(lo, hi, random.randint(lo, hi))
            if p == target:
                return nums[p]
            if p < target:
                lo = p + 1
            else:
                hi = p - 1
        return nums[lo]
```

---

## Edge Cases

| Case | Notes |
|---|---|
| `k == 1` | The maximum. |
| `k == n` | The minimum. |
| Duplicates | They count toward `k` (not distinct). |
| Single element with k = 1 | Return it directly. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if the data arrives as a stream and you can't store it all?"**
   Min-heap of size k. As each element comes in, push and (if size > k) pop. After processing, root is the k-th largest. This is the streaming-friendly answer.

2. **"K-th smallest instead of largest?"**
   Use a **max-heap** of size k (keeping the k smallest), or quickselect with target = `k - 1`.

3. **"Why use a random pivot?"**
   Without it, an adversary (or just bad luck) can give you sorted input where every pivot is the worst possible — O(n²). Random pivots make worst-case behavior astronomically unlikely.

4. **"What's the median of an array?"**
   Quickselect for index `n / 2`. There's also a deterministic O(n) algorithm called **median of medians**, but quickselect with random pivots is simpler and almost always fast enough.

---

## Key Lessons from This Problem

1. **A heap is your friend** for "top-k" or "stream of values" problems.
2. **Quickselect is quicksort's smarter cousin** — recurse on only the side that contains your answer.
3. **Sorting is overkill** when you only need *one* element. Don't pay O(n log n) when O(n) is achievable.
