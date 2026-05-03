# Problem: Time Complexity of a Nested Loop (j = i to n)

## Category
Intro to DSA / Complexity

## Difficulty
Easy

---

## Problem Statement

What is the time complexity of this code?

```java
for (int i = 0; i < n; i++) {
    for (int j = i; j < n; j++) {
        // O(1) work, e.g. a comparison or counter increment
    }
}
```

You're being asked: **how many times does the inner statement run, as a function of `n`?**

---

## Clarifying Questions to Ask the Interviewer

1. **Does the inner loop start at `i` or `i + 1`?** It only changes the constant, not the asymptotic class — both give O(n²).
2. **Is the body O(1)?** Assume yes. If the body itself were a loop, we'd multiply.

---

## Approach (Plain-English Counting)

Let's count exactly how many times the inner loop body runs.

- When `i = 0`: inner runs for `j = 0, 1, 2, ..., n-1` → **n iterations**.
- When `i = 1`: inner runs for `j = 1, 2, ..., n-1` → **n-1 iterations**.
- When `i = 2`: inner runs for `j = 2, 3, ..., n-1` → **n-2 iterations**.
- ...
- When `i = n-1`: inner runs for `j = n-1` only → **1 iteration**.

Adding it up:
```
n + (n-1) + (n-2) + ... + 2 + 1
```

This is a famous sum (Gauss's trick) and equals:
```
n(n + 1) / 2
```

Expanding: `(n² + n) / 2`. The dominant term is `n² / 2`.

### Why we throw away the constants

In Big-O, we care about how the cost **scales** with `n`, not exact constants. We say:

```
n² / 2 is in O(n²)
```

because for large `n`, the `1/2` factor doesn't change the *shape* of the curve — it just shifts it.

### A picture helps

Imagine an `n × n` grid. The pairs `(i, j)` that the loops visit are the cells where `j ≥ i` — the **upper triangle** including the diagonal.

```
n = 5:
. . . . .  ← row 0 (i=0): 5 cells visited (j=0..4)
. X X X X  ← row 1 (i=1): 4 cells visited (j=1..4)
. . X X X  ← row 2 (i=2): 3 cells visited
. . . X X  ← row 3 (i=3): 2 cells
. . . . X  ← row 4 (i=4): 1 cell
```

Half the cells of an `n × n` grid is roughly `n² / 2` — confirming O(n²).

---

## Time Complexity

**O(n²)** — the inner statement runs `n(n+1)/2 ≈ n²/2` times.

## Space Complexity

**O(1)** — we only use a counter.

---

## Code (Java)

```java
public int countOps(int n) {
    int count = 0;
    for (int i = 0; i < n; i++) {
        for (int j = i; j < n; j++) {
            count++;
        }
    }
    return count;   // equals n*(n+1)/2
}
```

Quick sanity check: `countOps(5)` should return `5+4+3+2+1 = 15`. And `5 * 6 / 2 = 15`. ✓

---

## Code (Python)

```python
def count_ops(n: int) -> int:
    count = 0
    for i in range(n):
        for j in range(i, n):
            count += 1
    return count        # == n*(n+1) // 2
```

---

## Edge Cases

| `n` | Iterations | Why |
|---|---|---|
| 0 | 0 | Outer loop doesn't run. |
| 1 | 1 | Inner runs once with i=0, j=0. |
| 2 | 3 | i=0: 2 runs; i=1: 1 run. Total 3. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if the inner loop was `for (j = i*i; j < n; j++)`?"**
   The inner loop runs `n - i²` times when `i² < n`, otherwise 0. The total is dominated by smaller `i` values and still works out to **O(n²)** asymptotically. (You can prove this with calculus or careful counting.)

2. **"What if the inner loop was `for (j = 1; j < n; j *= 2)`?"**
   The inner loop now doubles `j` each time, so it runs **log₂(n)** times. Combined with the outer O(n) loop, total is **O(n log n)**.

3. **"What's the difference between O(n²) and Θ(n²)?"**
   `O(n²)` means "at most n² (up to constants)" — an upper bound. `Θ(n²)` means "exactly n² up to constants" — both upper and lower bound. In interviews, people use Big-O loosely to mean the tight bound.

---

## Key Lessons from This Problem

1. **Sum of 1 + 2 + ... + n = n(n+1)/2.** This identity comes up constantly in algorithm analysis.
2. **Drop constants and lower-order terms** in Big-O. `n²/2 + n/2` becomes `O(n²)`.
3. **Visualize loops as grids** when counting iterations — it's much easier than chasing the math symbolically.
