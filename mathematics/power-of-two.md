# Problem: Power of Two

## Category
Mathematics / Bit Manipulation

## Difficulty
Easy

---

## Problem Statement

Given an integer `n`, return `true` if `n` is a **power of two**, else `false`.

Powers of two: 1, 2, 4, 8, 16, 32, ... (each is `2^k` for some non-negative integer `k`).

This is **LeetCode #231**.

### Examples

```
n = 1   → true   (2^0 = 1)
n = 16  → true   (2^4)
n = 218 → false
n = 0   → false  (no integer k makes 2^k = 0)
n = -8  → false  (negative numbers aren't powers of 2 in this convention)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Is 1 a power of two?** Yes — `2^0 = 1`.
2. **Negative numbers?** False (powers of two are positive).
3. **Type?** Usually `int`. Watch for `INT_MIN` edge cases with bit tricks.

---

## Approach

We'll cover three methods.

### Method 1 — Bit Trick (Optimal — O(1))

A power of two has **exactly one bit set** in its binary representation:

```
1     = 0001
2     = 0010
4     = 0100
8     = 1000
16   = 10000
```

Compare to non-powers:
```
3     = 0011  (two bits set)
6     = 0110  (two bits set)
10   = 1010  (two bits set)
```

The trick: `n & (n - 1)` clears the **lowest set bit**.

If `n` is a power of two (one bit set), clearing it leaves 0.

```
8 & 7 = 1000 & 0111 = 0000  ✓
6 & 5 = 0110 & 0101 = 0100 ≠ 0  ✗ (not a power of two)
```

So:

```
return n > 0 && (n & (n - 1)) == 0;
```

The `n > 0` guard handles 0 and negative numbers.

### Why does `n & (n - 1)` clear the lowest set bit?

Subtracting 1 from `n`:
- Flips the lowest set bit to 0.
- Flips all bits below it from 0 to 1.

Example: `n = 0010 1000` (= 40). `n - 1 = 0010 0111`.

`n & (n - 1) = 0010 0000` (= 32). The lowest set bit (the 8) is gone.

If `n` had only that one set bit (like 8 = 1000), then `n - 1` is all-zeros there (0111), and the AND is 0.

### Method 2 — Repeat Division

Divide by 2 while `n` is even. If we reach 1, it's a power of two.

```
while n % 2 == 0: n /= 2
return n == 1
```

Slower (O(log n)), but easy to explain.

### Method 3 — Math (Avoid)

Use `log2(n)` and check if it's an integer. Floating-point precision makes this **risky** — don't do it in interviews.

---

## Time Complexity

- Bit trick: **O(1)**.
- Division: **O(log n)**.

## Space Complexity

**O(1)**.

---

## Code (Java)

```java
public boolean isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

---

## Code (Python)

```python
class Solution:
    def isPowerOfTwo(self, n: int) -> bool:
        return n > 0 and (n & (n - 1)) == 0
```

---

## Edge Cases

| Case | Result | Why |
|---|---|---|
| `n = 0` | `false` | `n > 0` guard |
| `n = 1` | `true` | `1 & 0 = 0` ✓ |
| Negative `n` | `false` | `n > 0` guard |
| `INT_MIN` (Java) | `false` | Two's-complement weirdness — without `n > 0`, the bit trick would mistakenly say true. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Power of three (LeetCode 326)?"**
   The bit trick doesn't generalize. Either:
   - Repeatedly divide by 3 and check if you reach 1, or
   - For 32-bit ints: `1162261467 % n == 0` (where 1162261467 = `3^19`, the largest power of 3 fitting in int32).

2. **"Power of any base k?"**
   Use repeated division. Floating-point `log` can lie due to precision.

3. **"Round up to the next power of two."**
   "Smear" the high bit:
   ```
   n--;
   n |= n >> 1; n |= n >> 2; n |= n >> 4; n |= n >> 8; n |= n >> 16;
   n++;
   ```
   This sets all bits below the high one, then adds 1 to round up.

4. **"Count the number of set bits to confirm power-of-two?"**
   `Integer.bitCount(n) == 1` works, but `n & (n - 1) == 0` is faster (no popcount needed).

---

## Key Lessons from This Problem

1. **Bit tricks are O(1)** — when applicable, they're unbeatable.
2. **`n & (n - 1)` clears the lowest set bit.** Memorize this — it has many uses.
3. **Always guard against 0 and negatives** when using bit tricks. They sneak past intuition.
