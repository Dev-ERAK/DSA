# Problem: Check if a Number is Prime

## Category
Mathematics

## Difficulty
Easy

---

## Problem Statement

Given a positive integer `n`, return `true` if `n` is **prime**, else `false`.

A **prime number** has exactly two distinct divisors: 1 and itself. Examples: 2, 3, 5, 7, 11, 13, ...

By convention, **1 is NOT prime** (it has only one distinct divisor).

### Examples

```
2  → true   (smallest prime)
17 → true
1  → false
0  → false
9  → false  (= 3 × 3)
```

---

## Clarifying Questions to Ask the Interviewer

1. **What about 1?** Not prime by definition.
2. **Negative numbers?** Not prime by definition.
3. **Single number test, or many primes up to N?** Single test here. For "all primes up to N," use the Sieve of Eratosthenes — see `count-primes.md`.

---

## Approach — Trial Division up to √n

### The key observation

If `n` has a divisor `d > 1`, then `n / d` is also a divisor. **At least one of these two divisors is ≤ √n** — because if both were > √n, their product would exceed n.

So if we don't find any divisor up to `√n`, there are no proper divisors, and `n` is prime.

This reduces the work from O(n) (checking every number) to **O(√n)**.

### Optimizations

1. Handle 2 and 3 explicitly (they're the only "even and prime" / "divisible by 3 and prime" cases).
2. Check divisibility by 2 and 3 quickly.
3. After that, only check candidates of the form `6k ± 1` — i.e., 5, 7, 11, 13, 17, 19, ...

#### Why `6k ± 1`?

Every integer is one of these mod 6: 0, 1, 2, 3, 4, 5.
- Numbers `6k`, `6k+2`, `6k+4` are even → divisible by 2.
- Number `6k+3` is divisible by 3.
- Only `6k+1` and `6k+5` (= `6(k+1) - 1`) can possibly be prime (other than 2 and 3 themselves).

So skipping the rest cuts the work by ~3x.

### The algorithm

```
if n < 2: return false
if n == 2 or n == 3: return true
if n % 2 == 0 or n % 3 == 0: return false

i = 5
while i * i <= n:
    if n % i == 0 or n % (i + 2) == 0: return false
    i += 6
return true
```

### Walking through `n = 29`

```
29 < 2? No.
29 in {2, 3}? No.
29 % 2 = 1, 29 % 3 = 2. Not divisible.

i = 5: 5*5 = 25 ≤ 29. Check 29 % 5 = 4 (no), 29 % 7 = 1 (no). i = 11.
i = 11: 11*11 = 121 > 29. Loop exits.

Return true. ✓
```

### Walking through `n = 49`

```
49 % 2 = 1, 49 % 3 = 1.
i = 5: 25 ≤ 49. 49 % 5 = 4, 49 % 7 = 0. Found a divisor! Return false. ✓
```

---

## Time Complexity

**O(√n)** — we check up to √n candidates.

## Space Complexity

**O(1)**

---

## Code (Java)

```java
public boolean isPrime(int n) {
    if (n < 2) return false;
    if (n == 2 || n == 3) return true;
    if (n % 2 == 0 || n % 3 == 0) return false;

    for (int i = 5; (long) i * i <= n; i += 6) {
        if (n % i == 0 || n % (i + 2) == 0) return false;
    }
    return true;
}
```

> Why `(long) i * i`? In Java, for n up to ~2.1 × 10⁹, `i` can be up to ~46,340. `i * i` is fine in `long`, but cast to be safe. Also, we never check past `√n`.

---

## Code (Python)

```python
class Solution:
    def isPrime(self, n: int) -> bool:
        if n < 2:
            return False
        if n in (2, 3):
            return True
        if n % 2 == 0 or n % 3 == 0:
            return False

        i = 5
        while i * i <= n:
            if n % i == 0 or n % (i + 2) == 0:
                return False
            i += 6

        return True
```

---

## Edge Cases

| Case | Result |
|---|---|
| `n = 0`, `n = 1` | `false` |
| `n = 2` | `true` (smallest prime) |
| `n = 3` | `true` |
| `n = 4` | `false` (= 2 × 2) |
| Very large `n` | Use `long` for `i * i` to avoid overflow in Java. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"How would you find all primes up to N?"**
   Use the **Sieve of Eratosthenes** — see `count-primes.md`. Much faster than trial-dividing each number.

2. **"How do you test primality of huge numbers (e.g., 100+ digits)?"**
   Use **Miller-Rabin** — a probabilistic test. With enough witnesses, it's correct for all numbers in standard ranges.

3. **"How do you generate the next prime ≥ n?"**
   Increment `n` and call `isPrime(n)` until it's true. Decent in practice.

4. **"Why isn't 1 prime?"**
   Modern definition: a prime has exactly **two distinct positive divisors**. 1 has only one. Excluding 1 also keeps the **fundamental theorem of arithmetic** (unique factorization) clean.

---

## Key Lessons from This Problem

1. **Only check up to √n** — anything past that is redundant.
2. **The `6k ± 1` optimization** is a nice trick that buys you a 3x speed-up.
3. **Watch for overflow** in `i * i` — use `long` or compare `i ≤ n / i` carefully.
