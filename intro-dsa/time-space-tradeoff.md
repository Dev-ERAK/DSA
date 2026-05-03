# Problem: Time vs Space Trade-off

## Category
Intro to DSA / Complexity

## Difficulty
Easy

---

## Problem Statement

Explain the **time vs space trade-off** in algorithms, and give a concrete example.

The idea: sometimes you can make an algorithm **faster** by using **more memory** (and vice versa). You're being asked to show you understand this fundamental engineering choice.

---

## Clarifying Questions to Ask the Interviewer

1. **Do you want a single problem solved two ways, or a comparison of two different problems?** A single problem solved two ways is more illustrative.
2. **Is memory or time more important here?** In real systems it depends — embedded devices care about memory; web servers usually care more about latency.

---

## Approach (Plain-English Explanation)

The trade-off in one sentence: **you can often save time by remembering things you've computed before, but remembering costs memory.**

We'll explore this with two classic examples.

### Example 1 — Two Sum

**Problem:** Given an array, find two indices whose values add up to `target`.

#### Brute force (O(n²) time, O(1) space)

Try every pair:
```
for i in 0..n-1:
    for j in i+1..n-1:
        if nums[i] + nums[j] == target: return (i, j)
```

We use almost no extra memory — just a few variables. But the time grows as the square of the input size.

#### Hashmap (O(n) time, O(n) space)

Walk the array once. For each value `x`, ask: "Have I already seen `target - x`?" If yes, we have our pair.

```
seen = empty hashmap
for i, x in nums:
    need = target - x
    if need is in seen: return (seen[need], i)
    seen[x] = i
```

Now we use O(n) extra memory (the hashmap). In return, we've gone from O(n²) to O(n) — a massive speed-up.

**The trade:** we spent O(n) memory to save a factor of `n` in time.

### Example 2 — Fibonacci Numbers

The n-th Fibonacci number is defined by `F(n) = F(n-1) + F(n-2)`.

#### Plain recursion (O(2ⁿ) time, O(n) stack space)

```
fib(n) = fib(n-1) + fib(n-2)
```

This is **catastrophically slow** because it computes the same subproblems many times. `fib(40)` takes about a billion calls.

#### Memoized recursion (O(n) time, O(n) memory)

Cache each result the first time it's computed:

```
memo = {}
fib(n):
    if n in memo: return memo[n]
    result = fib(n-1) + fib(n-2)
    memo[n] = result
    return result
```

Each subproblem is solved exactly once. We've traded O(n) memory for an enormous time savings (exponential → linear).

#### Iterative two-variable (O(n) time, O(1) memory)

But wait — do we really need to remember **all** previous values, or just the last two?

```
a, b = 0, 1
for _ in range(n):
    a, b = b, a + b
return a
```

This is the **best of both worlds**: O(n) time and O(1) memory. The trade-off isn't always strict — sometimes a clever rewrite eliminates it.

---

## When you'd choose each option

| Situation | Pick this |
|---|---|
| Memory-constrained device (embedded, mobile) | Slower but lighter algorithm |
| Latency-sensitive (web requests, real-time) | Faster but heavier algorithm |
| Small `n` and tight constants | Brute force (cache effects, simple code) |
| Algorithm runs once, but result is needed many times | Precompute and cache |
| Algorithm runs millions of times on tiny inputs | Whichever has lower constant factors |

---

## Time Complexity / Space Complexity

N/A — this is conceptual. The complexities are shown above per technique.

---

## Code (Java) — Two Sum, both ways

```java
// Brute force: O(n^2) time, O(1) space
public int[] twoSumBrute(int[] nums, int target) {
    for (int i = 0; i < nums.length; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] == target) {
                return new int[] { i, j };
            }
        }
    }
    return new int[0];
}

// Hashmap: O(n) time, O(n) space
public int[] twoSumFast(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int need = target - nums[i];
        if (seen.containsKey(need)) {
            return new int[] { seen.get(need), i };
        }
        seen.put(nums[i], i);
    }
    return new int[0];
}
```

---

## Code (Python)

```python
def two_sum_brute(nums, target):
    for i in range(len(nums)):
        for j in range(i + 1, len(nums)):
            if nums[i] + nums[j] == target:
                return [i, j]
    return []

def two_sum_fast(nums, target):
    seen = {}
    for i, x in enumerate(nums):
        need = target - x
        if need in seen:
            return [seen[need], i]
        seen[x] = i
    return []
```

---

## Edge Cases

- **Tiny arrays.** Brute force can actually be faster than the hashmap version for n < 20 or so, because the overhead of hashing exceeds the cost of a simple loop. Big-O ignores this.
- **Memory-bound systems.** If RAM is precious, the slower algorithm may be the right choice.
- **Cache effects.** Walking through a contiguous array (brute force) has great CPU-cache behavior. A hashmap scatters data around memory and can be slower per operation.

---

## Follow-up Questions an Interviewer Might Ask

1. **"When does memoization actually hurt performance?"**
   When the recomputation is *cheaper* than the cache lookup. For example, `fib` benefits massively, but caching simple arithmetic like `x + 1` would slow you down.

2. **"What's amortized complexity?"**
   Average cost per operation over a sequence. A dynamic array has occasional slow O(n) resizes but, averaged across many inserts, each insert costs O(1). We say it's "amortized O(1)."

3. **"Can you always trade memory for time?"**
   Not always. Some problems have proven lower bounds (e.g., comparison-based sorting needs at least O(n log n) time regardless of memory). Memory only helps when you can use it to **avoid repeated work**.

---

## Key Lessons from This Problem

1. **Cache previously-computed results** to convert exponential time into linear time.
2. **Sometimes you can remove the trade-off** by being clever — e.g., the iterative two-variable Fibonacci is O(n) time AND O(1) space.
3. **The right choice depends on the system**, not just on Big-O. Always ask: what's the bottleneck — time, memory, latency, or simplicity?
