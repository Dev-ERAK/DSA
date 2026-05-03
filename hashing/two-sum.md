# Problem: Two Sum

## Category
Hashing / Arrays

## Difficulty
Easy

---

## Problem Statement

Given an array of integers `nums` and a `target`, return the **indices** of the two numbers that add up to the target.

You may assume:
- Exactly **one** valid answer exists.
- You can't use the same element twice.

This is **LeetCode #1** — possibly the most famous coding interview question.

### Examples

```
nums = [2, 7, 11, 15], target = 9   →  [0, 1]
        ↑  ↑
        2 + 7 = 9

nums = [3, 2, 4], target = 6        →  [1, 2]
nums = [3, 3], target = 6            →  [0, 1]
```

---

## Clarifying Questions to Ask the Interviewer

1. **Exactly one solution?** Yes — LeetCode 1 guarantees it.
2. **Can the array have duplicate values?** Yes (see `[3, 3]` example).
3. **Return values or indices?** Indices.
4. **Is the array sorted?** No (it's not, in this version).

---

## Approach

### Brute Force — O(n²)

Try every pair of indices.

```
for i in 0..n-1:
    for j in i+1..n-1:
        if nums[i] + nums[j] == target:
            return [i, j]
```

For each `nums[i]`, we look at every `nums[j]` after it. That's `n × n / 2 ≈ n² / 2` operations. For `n = 10⁵`, that's 5 billion — way too slow.

### Optimal — Hashmap (O(n))

For each value `x`, we need to find `target - x` somewhere in the array. A hashmap gives us **O(1) lookup**.

#### The trick — One pass

We don't need two passes. As we walk the array, we **check then store**:

1. For each element `x` at index `i`, compute `need = target - x`.
2. Check if `need` is already in the hashmap. If yes, return `[seen_index_of_need, i]`.
3. If not, put `x → i` in the hashmap.

By the time we encounter the second member of the pair, the first one is already in the map. The first member's "look up" returns nothing — but storing it lets the second one find it later.

### Walking through the example

`nums = [2, 7, 11, 15]`, `target = 9`.

| step | i | x | need = 9 - x | hashmap before | found? | hashmap after |
|---|---|---|---|---|---|---|
| 1 | 0 | 2 | 7 | {} | no | {2: 0} |
| 2 | 1 | 7 | 2 | {2: 0} | **yes! at index 0** | — return [0, 1] |

Done.

### Why "check then store" instead of "store then check"

If we stored first, then checked, we might match an element with **itself** when `target = 2 × nums[i]`. The check-first pattern avoids using the same element twice — a constraint of the problem.

For `nums = [3, 3], target = 6`:

```
i = 0, x = 3, need = 3.
  Check: 3 in hashmap? No.
  Store: {3: 0}.

i = 1, x = 3, need = 3.
  Check: 3 in hashmap? YES, at index 0.
  Return [0, 1].   ✓
```

If we'd stored first, the first iteration would have stored `3 → 0`, then the check `3 in hashmap` would return true — and we'd return `[0, 0]`, using the same element twice. Bug!

---

## Time Complexity

**O(n)** — one pass; each hashmap operation is O(1) average.

## Space Complexity

**O(n)** — the hashmap holds up to n - 1 entries (we always find a match before adding the n-th).

---

## Code (Java)

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();

    for (int i = 0; i < nums.length; i++) {
        int need = target - nums[i];

        // Check first — does the complement exist?
        if (seen.containsKey(need)) {
            return new int[] { seen.get(need), i };
        }

        // Then store.
        seen.put(nums[i], i);
    }
    return new int[0];   // unreachable per problem spec
}
```

---

## Code (Python)

```python
class Solution:
    def twoSum(self, nums, target: int):
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

| Case | Handled because... |
|---|---|
| Duplicates that form the answer (`[3, 3]`) | Check-then-store ensures we don't match the same element. |
| Negative numbers | Hashmaps handle any int. |
| `target = 0` | Same algorithm — works. |
| Very large numbers | Use `long` if `target - x` could overflow `int`. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if the array were sorted?"**
   Use **two pointers** — start `l = 0`, `r = n - 1`. If `nums[l] + nums[r] < target`, increment `l`; if `> target`, decrement `r`. **O(n) time, O(1) extra space.**

2. **"Find *all unique* pairs that sum to target."**
   Sort first; use two pointers; skip duplicates on both sides. (LeetCode 167-flavored.)

3. **"Three sum?"**
   For each `i`, fix `nums[i]` and run the two-sum-in-sorted-array on the rest. **O(n²)**.

4. **"Streaming version — values arrive one at a time, can't store the full array."**
   Bloom filters or count-min sketches give probabilistic answers; exact solution needs the hashmap.

---

## Key Lessons from This Problem

1. **Hashmap-as-memory** — the most common pattern in array problems. Whenever you find yourself "looking back" at previous elements, ask: "could a hashmap help?"
2. **Check before store** when avoiding self-pairs.
3. **One pass is enough** for many "find pair" problems — don't default to nested loops without thinking.
