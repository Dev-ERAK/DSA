# Problem: Least Common Multiple (LCM)

## Category
Mathematics

## Difficulty
Easy

---

## Problem Statement

Given two non-negative integers `a` and `b`, return their **Least Common Multiple (LCM)** — the smallest positive integer that is a multiple of both.

### Examples

```
lcm(4, 6)   = 12       (12 is divisible by both 4 and 6; 12 is the smallest such)
lcm(7, 13)  = 91       (since gcd is 1, lcm is just the product)
lcm(0, 5)   = 0        (by convention)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Both non-negative?** Yes. `lcm(0, x) = 0` by convention.
2. **Possible overflow?** `a × b` can overflow `int` if both are near `Integer.MAX_VALUE`. Use `long`.

---

## Approach

### The Key Identity

```
lcm(a, b) = a × b / gcd(a, b)
```

This is one of the most useful identities in number theory.

### Why does this work?

Both sides equal the smallest number divisible by both `a` and `b`. The intuition: if you write `a` and `b` as prime factorizations:
- `a = p₁^x₁ × p₂^x₂ × ...`
- `b = p₁^y₁ × p₂^y₂ × ...`

Then:
- `gcd(a, b) = ∏ pᵢ^min(xᵢ, yᵢ)` — take the minimum exponent of each prime.
- `lcm(a, b) = ∏ pᵢ^max(xᵢ, yᵢ)` — take the maximum.

And `min + max = x + y`, so `gcd × lcm = a × b`. Rearranging: `lcm = a × b / gcd`.

### Avoiding overflow

Compute as `a / gcd(a, b) * b` (divide first, then multiply). The intermediate stays smaller. Same mathematical result.

### Walking through

`lcm(4, 6)`:
- `gcd(4, 6) = 2`.
- `4 / 2 × 6 = 2 × 6 = 12`. ✓

`lcm(15, 25)`:
- `gcd(15, 25) = 5`.
- `15 / 5 × 25 = 3 × 25 = 75`. ✓

---

## Time Complexity

**O(log min(a, b))** — dominated by the GCD computation.

## Space Complexity

**O(1)** iterative.

---

## Code (Java)

```java
public long lcm(long a, long b) {
    if (a == 0 || b == 0) return 0;
    return a / gcd(a, b) * b;             // divide before multiplying
}

private long gcd(long a, long b) {
    while (b != 0) {
        long t = a % b;
        a = b;
        b = t;
    }
    return a;
}
```

---

## Code (Python)

```python
from math import gcd

class Solution:
    def lcm(self, a: int, b: int) -> int:
        if a == 0 or b == 0:
            return 0
        return a // gcd(a, b) * b
```

> Python 3.9+ has `math.lcm` built-in. Useful in real code; in interviews, show you know the formula.

---

## Edge Cases

| Case | Result |
|---|---|
| `a == 0` or `b == 0` | 0 |
| `a == b` | That value |
| Coprime (`gcd(a, b) = 1`) | `a × b` |
| Large products | Use `long` and divide-before-multiply to avoid overflow. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"LCM of more than two numbers?"**
   Fold: `lcm(lcm(a, b), c, ...)`. The result can grow very fast — consider `BigInteger` for chains.

2. **"LCM of fractions?"**
   `lcm(p/q, r/s) = lcm(p, r) / gcd(q, s)`. Used in some music-theory and physics applications.

3. **"Why isn't LCM just `a × b`?"**
   Because if `a` and `b` share factors, you'd be over-counting. Dividing by `gcd(a, b)` removes the duplication.

4. **"What's the time complexity of LCM compared to GCD?"**
   Same — O(log min(a, b)). The multiplication and division are O(1) on bounded integers.

---

## Key Lessons from This Problem

1. **`lcm(a, b) = a × b / gcd(a, b)`** — memorize this identity.
2. **Divide before multiplying** to avoid overflow.
3. **Prime-factorization view** of GCD and LCM makes them feel natural and easy to remember.
