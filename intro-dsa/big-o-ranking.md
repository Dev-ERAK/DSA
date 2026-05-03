# Problem: Rank Big-O Complexities

## Category
Intro to DSA / Complexity

## Difficulty
Easy

---

## Problem Statement

Rank these four complexities in **increasing** order of growth (slowest-growing first):

```
O(n),  O(log n),  O(n log n),  O(n²)
```

Big-O describes how an algorithm's time (or memory) grows as the input size `n` grows. **Slower-growing functions are better** — they scale to larger inputs.

---

## Clarifying Questions to Ask the Interviewer

1. **Are we comparing for large n?** Yes — Big-O is about asymptotic behavior (how things behave as `n` gets large).
2. **Do constants matter?** No. Big-O ignores constants and lower-order terms. `100n` and `n` are both O(n).

---

## Approach

### The Ranking

```
O(log n)  <  O(n)  <  O(n log n)  <  O(n²)
```

(slowest growth → fastest growth)

### Concrete Numbers

The easiest way to understand Big-O is to plug in real values for `n`:

| n | log₂ n | n | n log n | n² |
|---|---|---|---|---|
| 10 | ~3 | 10 | ~33 | 100 |
| 100 | ~7 | 100 | ~700 | 10,000 |
| 1,000 | ~10 | 1,000 | ~10,000 | 1,000,000 |
| 1,000,000 | ~20 | 1,000,000 | ~20 million | **1 trillion** |

A few takeaways:
- **O(log n)** barely grows. Doubling `n` only adds 1 to log n.
- **O(n)** grows in step with the input.
- **O(n log n)** grows slightly faster than linear — the log factor is small.
- **O(n²)** grows **dramatically** — for n = 1 million, it's a trillion operations. That's hours or days.

### Where each one comes from

| Class | Typical algorithm | Why |
|---|---|---|
| O(log n) | **Binary search** | Each step halves the search space. After k steps you've narrowed `n` to `n / 2ᵏ` — this hits 1 when `k = log₂ n`. |
| O(n) | **Linear scan** | Touch every element once. |
| O(n log n) | **Merge sort, heap sort** | Divide & conquer: `n` elements processed in `log n` levels. |
| O(n²) | **Bubble sort, brute-force pair check** | Compare every pair — there are roughly n²/2 pairs. |

### Visualizing growth rates

For n on the x-axis and operations on the y-axis:

```
Operations
   ↑
   |             /  ← n²
   |          /
   |        /
   |      /        ← n log n
   |    /         ← n
   |  /         ← log n (almost flat)
   +─────────────→ n
```

`log n` is almost flat — it barely moves even as `n` blows up. `n²` shoots upward dramatically.

### Rule of thumb for what's "fast enough"

For about a 1-second budget on a typical machine, here's roughly how big an `n` each complexity can handle:

| Complexity | Reasonable max n |
|---|---|
| O(n!) | 11 |
| O(2ⁿ) | ~25 |
| O(n³) | ~500 |
| O(n²) | ~10,000 |
| O(n log n) | ~10,000,000 |
| O(n) | ~100,000,000 |
| O(log n) | basically anything |

If you see n = 10⁵ in a problem, you probably need O(n log n) or better.

---

## Time Complexity / Space Complexity

N/A — this is theory.

---

## Code (Java) — examples for each class

```java
// O(log n) — binary search
int binarySearch(int[] a, int target) {
    int lo = 0, hi = a.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] == target) return mid;
        if (a[mid] < target) lo = mid + 1;
        else                 hi = mid - 1;
    }
    return -1;
}

// O(n) — sum the array
int sum(int[] a) {
    int s = 0;
    for (int x : a) s += x;
    return s;
}

// O(n log n) — sort
void sortIt(int[] a) {
    Arrays.sort(a);
}

// O(n²) — count all pairs
int pairCount(int[] a) {
    int count = 0;
    for (int i = 0; i < a.length; i++) {
        for (int j = i + 1; j < a.length; j++) {
            count++;
        }
    }
    return count;
}
```

---

## Code (Python)

```python
def binary_search(a, target):
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == target: return mid
        if a[mid] < target: lo = mid + 1
        else: hi = mid - 1
    return -1

def total(a):       return sum(a)        # O(n)
def sort_it(a):     a.sort()             # O(n log n)

def pair_count(a):                       # O(n^2)
    count = 0
    for i in range(len(a)):
        for j in range(i + 1, len(a)):
            count += 1
    return count
```

---

## Edge Cases

- **Small `n` is misleading.** O(n²) with tiny constants can beat O(n log n) with large constants when `n < 20`. That's why some standard libraries fall back to insertion sort for small subarrays inside merge/quick sort.
- **Logarithm base doesn't matter** in Big-O. log₂ n and log₁₀ n differ by a constant factor.

---

## Follow-up Questions an Interviewer Might Ask

1. **"Where does O(2ⁿ) fit in?"** Worse than O(n²) — it explodes very fast. Naive recursive Fibonacci, subset enumeration.

2. **"Where does O(n!) fit in?"** Worse than O(2ⁿ). Permutations of `n` items. n! grows so fast that 12! is already half a billion.

3. **"Full ordering, smallest to largest?"**
   ```
   O(1) < O(log n) < O(√n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!)
   ```

4. **"What's the difference between O(1) and 'constant time'?"** Same thing. O(1) means the runtime doesn't depend on the input size — could be 1 instruction, could be 1,000, but it doesn't grow.

---

## Key Lessons from This Problem

1. **Big-O describes growth, not exact time.** Two O(n) algorithms can differ by 100x in real-world speed.
2. **Think in orders of magnitude.** Knowing whether your algorithm is O(n) or O(n²) usually matters far more than micro-optimizations.
3. **Memorize the typical complexities** for each algorithm — they come up constantly.
