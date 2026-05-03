# Problem: Greatest Common Divisor (GCD) — Euclidean Algorithm

## Category
Mathematics

## Difficulty
Easy

---

## Problem Statement

Given two non-negative integers `a` and `b`, return their **Greatest Common Divisor (GCD)** — the largest integer that divides both.

### Examples

```
gcd(12, 18) = 6     (6 divides 12 and 18; nothing larger does)
gcd(7, 13)  = 1     (they share no common factors > 1)
gcd(0, 5)   = 5     (every number divides 0; the GCD is the non-zero one)
gcd(0, 0)   = 0     (by convention)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Both non-negative?** Yes. Negatives can be handled by taking absolute values.
2. **`gcd(0, 0)`?** Conventionally 0.
3. **Implementation style?** Iterative or recursive — both work.

---

## Approach — Euclidean Algorithm

### The key identity

Euclid (~300 BC) figured out:

```
gcd(a, b) = gcd(b, a mod b)
```

(Where `a mod b` is the remainder when `a` is divided by `b`.)

Repeat until `b == 0`. Then **the answer is `a`**.

### Why does this work?

If `d` divides both `a` and `b`, then `d` divides `a - q×b = a mod b` (for any integer q). So any common divisor of `(a, b)` is also a common divisor of `(b, a mod b)` — and vice versa. They have the same GCD.

Each step replaces `a` with `b` (smaller, or equal in size) and `b` with `a mod b` (strictly smaller than `b`). The numbers shrink fast → quick termination.

### Walking through `gcd(48, 18)`

```
a = 48, b = 18.    a mod b = 48 - 2×18 = 12.   New: (18, 12).
a = 18, b = 12.    a mod b = 18 - 1×12 = 6.    New: (12, 6).
a = 12, b = 6.     a mod b = 12 - 2×6 = 0.     New: (6, 0).
b = 0 → stop. Return a = 6.
```

GCD(48, 18) = 6. ✓

### How fast does this run?

Surprisingly fast. **Each pair of steps roughly halves the numbers** (Lamé's theorem ties this to the Fibonacci sequence). Total complexity: **O(log min(a, b))**.

For `a = b = 10⁹`, it converges in about 40 iterations. Negligible.

---

## Time Complexity

**O(log min(a, b))**

## Space Complexity

- Iterative: **O(1)**
- Recursive: **O(log min(a, b))** stack

---

## Code (Java)

```java
// Iterative — preferred
public int gcd(int a, int b) {
    while (b != 0) {
        int t = a % b;
        a = b;
        b = t;
    }
    return a;
}

// Recursive — also fine
public int gcdRec(int a, int b) {
    return b == 0 ? a : gcdRec(b, a % b);
}
```

---

## Code (Python)

```python
class Solution:
    def gcd(self, a: int, b: int) -> int:
        while b:
            a, b = b, a % b
        return a

    def gcd_rec(self, a: int, b: int) -> int:
        return a if b == 0 else self.gcd_rec(b, a % b)
```

> Python has `math.gcd(a, b)` built in — use it in real code, but know how to implement it for interviews.

---

## Edge Cases

| Case | Result |
|---|---|
| `gcd(a, 0) = a` | Every number divides 0. |
| `gcd(0, 0) = 0` | Convention. |
| Negative input | Common convention: `gcd(|a|, |b|)`. |
| `a < b` | The algorithm self-corrects: first step swaps them via `a % b = a`. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"How do you compute LCM (Least Common Multiple)?"**
   `lcm(a, b) = a × b / gcd(a, b)`. To avoid overflow: `a / gcd(a, b) * b` (divide first). See `lcm.md`.

2. **"What's the *Extended* Euclidean algorithm?"**
   It also finds integers `x, y` such that `a × x + b × y = gcd(a, b)`. Useful for finding modular inverses (e.g., for RSA).

3. **"GCD of an array?"**
   Fold: `gcd(gcd(a₁, a₂), a₃, ...)`. If you ever hit 1, you can stop — the GCD can't go below 1.

4. **"Binary GCD (Stein's algorithm)?"**
   Avoids `mod` using bit shifts. Useful for big integers where `mod` is expensive.

---

## Key Lessons from This Problem

1. **`gcd(a, b) = gcd(b, a mod b)`** — memorize this identity. It's the foundation of much of number theory.
2. **The Euclidean algorithm is one of the oldest algorithms still in active use** — over 2,000 years and counting.
3. **Iterative is preferred** for production — no recursion-stack overhead.
