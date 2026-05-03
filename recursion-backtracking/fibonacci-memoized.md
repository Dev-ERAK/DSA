# Problem: Fibonacci with Memoization

## Category
Recursion / Dynamic Programming

## Difficulty
Easy

---

## Problem Statement

Compute the n-th Fibonacci number **efficiently** using **memoization**.

(See `fibonacci-recursive.md` for the naive recursive version that this fixes.)

---

## Clarifying Questions to Ask the Interviewer

1. **Same Fibonacci definition?** `F(0) = 0, F(1) = 1, F(n) = F(n-1) + F(n-2)`.
2. **How big can `n` be?** Memoized recursion is fast for any reasonable n; for huge n, prefer matrix exponentiation.
3. **Worry about integer overflow?** Java `long` overflows at `n = 92`. Python handles big integers natively.

---

## Approach

### What's "memoization"?

**Memoization** = "remember the answer the first time, look it up after that."

You cache the result of each function call, keyed by its arguments. The next call with the same arguments returns the cached value instantly instead of recomputing.

It's a top-down form of **dynamic programming** (DP).

### The fix to naive Fibonacci

The naive version recomputes the same subproblems many times. We add a cache:

```
memo = {}   # arg -> result

fib(n):
    if n < 2: return n
    if n in memo: return memo[n]              # already computed
    result = fib(n-1) + fib(n-2)
    memo[n] = result                          # save it
    return result
```

Each subproblem is now computed **exactly once**. There are O(n) distinct subproblems (`fib(0)` through `fib(n)`), each doing O(1) work after the first call. **Total: O(n).**

### Walking through `fib(5)` with memoization

```
Call fib(5)
  Call fib(4)
    Call fib(3)
      Call fib(2)
        Call fib(1) → 1            (base case)
        Call fib(0) → 0            (base case)
      → 1, memoized
      Call fib(1) → 1               (base case)
    → 2, memoized
    Call fib(2) → 1                 (cache hit!)
  → 3, memoized
  Call fib(3) → 2                   (cache hit!)
→ 5
```

Total recursive calls: **6 unique** (one per value of n) plus 2 cache hits. Compare to ~9 calls without memoization for fib(4) — and the savings explode at higher n.

### Two flavors of DP

#### Top-Down (Memoization)

The code above. You **start at n** and recurse downward, caching as you go.

#### Bottom-Up (Tabulation)

You **start at 0 and 1** and build up. No recursion:

```
dp[0] = 0
dp[1] = 1
for i in 2..n:
    dp[i] = dp[i-1] + dp[i-2]
return dp[n]
```

Both are O(n). Bottom-up has lower constant factors and no recursion-stack risk.

#### Even Better — O(1) Memory

Notice we only ever need the **last two values** to compute the next. So:

```
a, b = 0, 1
for _ in range(n):
    a, b = b, a + b
return a
```

This is O(n) time and **O(1) memory** — the gold standard.

---

## Time Complexity

**O(n)** — each subproblem solved once.

## Space Complexity

- Memoized recursion: **O(n)** memo + **O(n)** stack.
- Iterative tabulation: **O(n)** if you keep the table; **O(1)** with two-variable trick.

---

## Code (Java)

```java
// Memoized recursion
public long fib(int n) {
    long[] memo = new long[n + 1];
    Arrays.fill(memo, -1);             // -1 means "not yet computed"
    return helper(n, memo);
}

private long helper(int n, long[] memo) {
    if (n < 2) return n;
    if (memo[n] != -1) return memo[n];
    return memo[n] = helper(n - 1, memo) + helper(n - 2, memo);
}

// Iterative O(1) — best in practice
public long fibIter(int n) {
    if (n < 2) return n;
    long a = 0, b = 1;
    for (int i = 2; i <= n; i++) {
        long c = a + b;
        a = b;
        b = c;
    }
    return b;
}
```

---

## Code (Python)

```python
from functools import lru_cache

class Solution:
    # The simplest memoization — just decorate.
    @lru_cache(maxsize=None)
    def fib(self, n: int) -> int:
        if n < 2:
            return n
        return self.fib(n - 1) + self.fib(n - 2)

    # Iterative O(1)
    def fib_iter(self, n: int) -> int:
        if n < 2:
            return n
        a, b = 0, 1
        for _ in range(2, n + 1):
            a, b = b, a + b
        return b
```

> Python's `@lru_cache` is the cleanest way to memoize any pure function. `lru_cache` uses an LRU eviction policy; `maxsize=None` means "cache forever."

---

## Edge Cases

| `n` | Result |
|---|---|
| 0 | 0 |
| 1 | 1 |
| Large `n` (>40) | Memoization handles it; naive recursion can't. |
| `n = 92` (Java) | Last value that fits in `long`. F(93) overflows. |
| Recursive memo + huge n | Stack might overflow — prefer iterative. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What's the difference between memoization and tabulation?"**
   - **Memoization** is top-down: recurse from n, cache as you go. Only computes states that are actually reached.
   - **Tabulation** is bottom-up: iterate from 0 to n, fill a table. Computes everything but with lower overhead.

2. **"Can you compute fib(n) in O(log n)?"**
   Yes — **matrix exponentiation**. The recurrence corresponds to a 2×2 matrix that you raise to the n-th power using fast exponentiation.

3. **"What if you need fib(n) under a modulus (mod p)?"**
   Apply the modulus at every addition. Or use matrix exponentiation under modular arithmetic.

4. **"Why might iterative be preferred over memoized recursion?"**
   - No recursion-stack risk for huge n.
   - Lower constant factors (no function-call overhead).
   - Easier to space-optimize.

---

## Key Lessons from This Problem

1. **Memoization turns exponential into linear** when subproblems overlap.
2. **Top-down vs bottom-up** are two valid styles of DP — pick the cleaner one.
3. **Space optimization** comes from noticing how many past values you actually need (here, just the last two).
