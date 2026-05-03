# Problem: Count Set Bits (Hamming Weight)

## Category
Mathematics / Bit Manipulation

## Difficulty
Easy

---

## Problem Statement

Given an integer `n`, return the number of `1` bits in its binary representation.

This count is called the **Hamming weight** (or popcount).

This is **LeetCode #191**.

### Examples

```
n = 11   → 3   (binary 1011 has three 1s)
n = 128  → 1   (binary 10000000 has one 1)
n = 0    → 0
n = -1   → 32  (in 32-bit two's complement, all bits are 1)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Treat `n` as unsigned 32-bit?** LeetCode 191 treats it that way.
2. **Negative numbers?** Treated as their two's complement bit pattern.

---

## Approach

We'll cover three methods.

### Method 1 — Naive Bit-by-Bit

Loop through all 32 bits. For each bit, check if it's set.

```
count = 0
for i in 0..31:
    if (n >> i) & 1: count++
```

**O(32) = O(1)** — constant work, but always 32 iterations.

### Method 2 — Brian Kernighan's Trick (Optimal)

`n & (n - 1)` clears the **lowest set bit**.

So we can count by repeatedly clearing the lowest set bit until `n` is 0:

```
count = 0
while n != 0:
    n = n & (n - 1)
    count++
return count
```

The loop runs **once per set bit** — so it's `O(k)` where `k` is the count of set bits. Best case 0 iterations (n = 0), worst case 32 iterations (all bits set).

### Why `n & (n - 1)` clears the lowest set bit

Subtracting 1 from `n`:
- Flips the lowest set bit from 1 to 0.
- Flips all bits below it from 0 to 1.

Then ANDing zeros out exactly the lowest set bit.

Example: `n = 1010 1000` (= 168). `n - 1 = 1010 0111`. `n & (n-1) = 1010 0000` (= 160). Lowest 1 (the 8) is gone.

### Method 3 — Built-in `popcount`

Most modern CPUs have a hardware `POPCNT` instruction. The built-in functions use it:

- Java: `Integer.bitCount(n)`
- Python: `bin(n).count('1')` or `n.bit_count()` (Python 3.10+)
- C++: `__builtin_popcount(n)`

These are typically the fastest in production.

### Walking through `n = 11` (`1011`)

Brian Kernighan's:
```
n = 1011, count = 0
n & (n-1) = 1011 & 1010 = 1010. n = 1010, count = 1.
n & (n-1) = 1010 & 1001 = 1000. n = 1000, count = 2.
n & (n-1) = 1000 & 0111 = 0000. n = 0000, count = 3.
n is 0 → done. Return 3. ✓
```

3 iterations for 3 set bits.

---

## Time Complexity

- Naive: **O(32)** = O(1).
- Brian Kernighan: **O(k)** where k = number of set bits. Worst-case O(32).

## Space Complexity

**O(1)**

---

## Code (Java)

```java
// Brian Kernighan's
public int hammingWeight(int n) {
    int count = 0;
    while (n != 0) {
        n &= (n - 1);            // clear the lowest set bit
        count++;
    }
    return count;
}

// Built-in (fastest in practice)
public int hammingWeightBuiltin(int n) {
    return Integer.bitCount(n);
}
```

---

## Code (Python)

```python
class Solution:
    # Brian Kernighan's
    def hammingWeight(self, n: int) -> int:
        count = 0
        # Python ints are arbitrary precision; mask to 32 bits if needed.
        n = n & 0xFFFFFFFF       # treat as unsigned 32-bit
        while n:
            n &= n - 1
            count += 1
        return count

    # Built-in
    def hammingWeight_builtin(self, n: int) -> int:
        return bin(n & 0xFFFFFFFF).count('1')
```

> Python integers can be negative and arbitrary precision. To match the LeetCode 32-bit unsigned convention, AND with `0xFFFFFFFF` first.

---

## Edge Cases

| Case | Result |
|---|---|
| `n = 0` | 0 |
| `n = -1` (in 32-bit) | 32 |
| `n = INT_MIN` (`0x80000000`) | 1 |
| Java/C++ `n & (n-1)` for negative `n` | Works correctly — two's complement makes the trick valid. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Count set bits for all numbers `0..n` (LeetCode 338)."**
   Use DP: `dp[i] = dp[i >> 1] + (i & 1)`. Each value uses the previously-computed answer for `i / 2`. O(n).

2. **"Hamming distance between two integers (LeetCode 461)."**
   Number of bits where they differ = `bitCount(a XOR b)`.

3. **"Why does `n & (n - 1)` clear the lowest set bit?"**
   See above — subtracting 1 flips the lowest set bit and flips all bits below it.

4. **"Hardware support?"**
   Yes — modern x86 CPUs have a `POPCNT` instruction that does this in 1-3 clock cycles. Java's `Integer.bitCount` will use it on platforms that support it.

---

## Key Lessons from This Problem

1. **Brian Kernighan's trick** is one of the most cited bit-manipulation patterns. Memorize it.
2. **Hardware popcount is fastest** — use built-ins in production code.
3. **Two's complement makes negative numbers work** for the bit trick — but mind the conventions in different languages.
