# Problem: Count Primes Less Than n (Sieve of Eratosthenes)

## Category
Mathematics

## Difficulty
Medium

---

## Problem Statement

Given an integer `n`, return the **number of prime numbers strictly less than `n`**.

This is **LeetCode #204**.

### Examples

```
n = 10  →  4   (primes: 2, 3, 5, 7)
n = 0   →  0
n = 2   →  0   (no primes less than 2)
n = 100 → 25
```

---

## Clarifying Questions to Ask the Interviewer

1. **Strictly less than `n`?** Yes — LeetCode 204.
2. **Memory bound?** O(n) is fine for `n ≤ 10⁷`.

---

## Approach — Sieve of Eratosthenes

### The Big Idea

Instead of checking each number's primality individually (slow), we **eliminate composites in bulk**.

Start with a boolean array marking everything as "potentially prime." Then for each prime we discover, **mark all its multiples as composite**. What remains is the list of primes.

This is the **Sieve of Eratosthenes**, named after the Greek mathematician (~200 BC).

### The algorithm

1. Create `is_prime[0..n-1]`, all `true`. Mark `is_prime[0] = is_prime[1] = false` (0 and 1 aren't prime).
2. For each `i` from 2 while `i × i < n`:
   - If `is_prime[i]` is still true, then `i` is prime.
   - Mark all multiples of `i`, starting from `i × i`, as composite.
3. Count how many `is_prime[i]` are still true (for `2 ≤ i < n`).

### Why start marking from `i × i`?

Smaller multiples of `i` (`2i, 3i, ..., (i-1)×i`) have already been marked composite by smaller primes. For example, when `i = 5`, the values `10, 15, 20` were already marked by `i = 2, 3, 2` respectively. Starting at `25` saves redundant work.

### Walking through `n = 30`

Initial: all of `2..29` marked prime.

```
i = 2 (prime). Mark 4, 6, 8, 10, ..., 28 as composite.
i = 3 (prime). Mark 9, 15, 21, 27. (6 already marked, etc.)
i = 4 (composite — was marked by 2). Skip.
i = 5 (prime). Mark 25.

i × i > 30 once i = 6, so the loop ends here.

Primes < 30: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29 — total 10.
```

### Why O(n log log n)?

The total work is `n/2 + n/3 + n/5 + n/7 + ...` (summing over primes up to √n). This sum is bounded by `n × log log n` — a famous result. So the sieve is **almost linear**.

For `n = 10⁷`, that's about 10⁷ × 3 ≈ 3 × 10⁷ operations — fast.

---

## Time Complexity

**O(n log log n)**

## Space Complexity

**O(n)** — the boolean array.

---

## Code (Java)

```java
public int countPrimes(int n) {
    if (n < 2) return 0;

    boolean[] composite = new boolean[n];     // composite[i] = true means i is NOT prime

    for (int i = 2; (long) i * i < n; i++) {
        if (!composite[i]) {                  // i is prime
            for (long j = (long) i * i; j < n; j += i) {
                composite[(int) j] = true;
            }
        }
    }

    int count = 0;
    for (int i = 2; i < n; i++) {
        if (!composite[i]) count++;
    }
    return count;
}
```

> Use `long` for `i * i` and the loop counter to avoid overflow at large `n`.

---

## Code (Python)

```python
class Solution:
    def countPrimes(self, n: int) -> int:
        if n < 2:
            return 0

        is_prime = [True] * n
        is_prime[0] = is_prime[1] = False

        for i in range(2, int(n ** 0.5) + 1):
            if is_prime[i]:
                # Mark multiples of i, starting from i*i.
                for j in range(i * i, n, i):
                    is_prime[j] = False

        return sum(is_prime)
```

> Pythonic shortcut: `is_prime[i*i::i] = [False] * len(is_prime[i*i::i])`. Slightly faster.

---

## Edge Cases

| Case | Result |
|---|---|
| `n ≤ 2` | 0 |
| `n = 3` | 1 (just 2) |
| `n = 10⁷+` | Memory becomes the concern — use a bitset for half the space. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Reduce memory using a bitset."**
   Each `bool` in Java/Python is roughly 1 byte. A `BitSet` (Java) or `bytearray` (Python) uses 1 bit per entry — 8x smaller.

2. **"Linear sieve in O(n)?"**
   Each composite gets marked exactly once — by its **smallest prime factor**. Slightly more complex but achieves O(n).

3. **"Find primes in a range [L, R] where R is huge but R - L is small?"**
   **Segmented sieve**: precompute primes up to √R using a regular sieve. Then sieve the range [L, R] using those primes — only marks composites in the small range.

4. **"How do I know `i × i` is the right starting point?"**
   Earlier multiples like `2i`, `3i`, ..., `(i-1)i` have a prime factor smaller than i, so they were already marked. The first un-marked composite multiple of i is `i × i` itself.

---

## Key Lessons from This Problem

1. **The sieve is one of the most beautiful algorithms in CS** — eliminate composites in bulk rather than testing each.
2. **O(n log log n) is essentially linear** for any practical n.
3. **Starting at `i × i`** is a small optimization with a real impact on speed.
