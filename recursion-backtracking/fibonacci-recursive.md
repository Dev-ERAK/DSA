# Problem: Fibonacci Numbers (Naive Recursion)

## Category
Recursion

## Difficulty
Easy

---

## Problem Statement

The **Fibonacci sequence** is defined by:

```
F(0) = 0
F(1) = 1
F(n) = F(n-1) + F(n-2)   for n ≥ 2
```

So the sequence starts: **0, 1, 1, 2, 3, 5, 8, 13, 21, ...**

Given `n`, compute and return `F(n)` using **naive recursion** (no caching).

This problem is the classic "see how recursion works" — and the classic "see why naive recursion can be terrible."

---

## Clarifying Questions to Ask the Interviewer

1. **0-indexed?** Yes — `F(0) = 0`.
2. **How big can `n` be?** Naive recursion gets unusable past `n ≈ 35-40`. Memoization (next problem) fixes that.
3. **Use big integers?** Java's `long` overflows at `n = 92`. Python handles big integers natively.

---

## Approach

### The Naive Recursive Definition

The math definition translates straight into code:

```
fib(n):
    if n < 2: return n      # base cases: F(0)=0, F(1)=1
    return fib(n-1) + fib(n-2)
```

Beautifully simple. But...

### Why This Is Catastrophically Slow

Each call branches into **two more calls**. The recursion tree for `fib(5)` looks like:

```
                    fib(5)
                  /        \
              fib(4)       fib(3)
              /   \         /   \
          fib(3) fib(2)   fib(2) fib(1)
          /  \    / \      /  \
       fib(2) fib(1) fib(1) fib(0) ...
       /  \
    fib(1) fib(0)
```

Notice `fib(3)` is computed twice. `fib(2)` is computed three times. `fib(1)` and `fib(0)` are computed many times. **The same subproblems are solved over and over.**

### The complexity

The number of nodes in this tree grows roughly like `2ⁿ`. More precisely, it's `O(φⁿ)` where `φ ≈ 1.618` (the golden ratio). The point is: **exponential**.

Concrete numbers:
- `fib(10)` — ~177 calls
- `fib(20)` — ~21,891 calls
- `fib(30)` — ~2.7 million calls
- `fib(40)` — ~331 million calls (takes seconds)
- `fib(50)` — ~40 billion calls (takes hours)

This is why **memoization** matters — see `fibonacci-memoized.md`.

### Walking through `fib(4)`

```
fib(4)
  fib(3)
    fib(2)
      fib(1) → 1
      fib(0) → 0
    → 1
    fib(1) → 1
  → 2
  fib(2)
    fib(1) → 1
    fib(0) → 0
  → 1
→ 3
```

`F(4) = 3`. Total calls: 9 (for one value of n=4). Already wasteful.

---

## Time Complexity

**O(2ⁿ)** — exponential.

## Space Complexity

**O(n)** — recursion stack depth.

---

## Code (Java)

```java
public long fib(int n) {
    if (n < 2) return n;             // F(0) = 0, F(1) = 1
    return fib(n - 1) + fib(n - 2);
}
```

---

## Code (Python)

```python
class Solution:
    def fib(self, n: int) -> int:
        if n < 2:
            return n
        return self.fib(n - 1) + self.fib(n - 2)
```

---

## Edge Cases

| `n` | Result |
|---|---|
| 0 | 0 |
| 1 | 1 |
| 2 | 1 (= 1 + 0) |
| Negative `n` | Undefined; throw or clarify. |
| Large `n` (>40) | Too slow — must memoize. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"This is too slow. How would you fix it?"**
   Use **memoization** — cache each result the first time it's computed. See `fibonacci-memoized.md`. Brings it down to **O(n)**.

2. **"Can you do it iteratively with O(1) memory?"**
   Yes — only the last two values matter:
   ```
   a, b = 0, 1
   for _ in range(n):
       a, b = b, a + b
   return a
   ```
   O(n) time, O(1) memory.

3. **"O(log n) time?"**
   **Matrix exponentiation**. The Fibonacci recurrence corresponds to multiplying:
   ```
   [F(n+1)]   [1 1]^n   [F(1)]
   [F(n)  ] = [1 0]   * [F(0)]
   ```
   Using fast exponentiation, you compute the matrix power in O(log n).

4. **"What's the closed-form formula?"**
   **Binet's formula:** `F(n) = (φⁿ - ψⁿ) / √5` where `φ = (1 + √5)/2` and `ψ = (1 - √5)/2`. Exact in math, but floating-point error makes it unreliable for large `n`.

---

## Key Lessons from This Problem

1. **Naive recursion can be exponential** when subproblems overlap.
2. **The recursion tree shape** matters — branching factor and overlap drive the complexity.
3. **Always think: "am I solving the same thing twice?"** If yes, memoize.
