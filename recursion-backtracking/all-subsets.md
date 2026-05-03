# Problem: Generate All Subsets (Power Set)

## Category
Recursion / Backtracking

## Difficulty
Medium

---

## Problem Statement

Given an array of distinct integers `nums`, return **all possible subsets** (the **power set**).

A **subset** is any selection of elements (including the empty selection and the full set). For an n-element set, there are exactly **2ⁿ** subsets.

This is **LeetCode #78**.

### Example

```
Input: [1, 2, 3]
Output (8 subsets):
[]
[1]
[2]
[3]
[1, 2]
[1, 3]
[2, 3]
[1, 2, 3]
```

(Order doesn't matter for the output.)

---

## Clarifying Questions to Ask the Interviewer

1. **All distinct?** Yes for this problem. Duplicates are a separate problem (`subsets-no-duplicates.md`).
2. **Include the empty subset?** Yes — that's part of the power set.
3. **Output order?** Usually any order is fine.

---

## Approach

We'll cover three approaches.

### Method 1 — Backtracking (Recommended, Most Common)

Walk through the array. At each index, you have two choices: **include** this element in the current subset, or **don't**.

The cleanest way to write it: for each subset, decide which elements to add by iterating over remaining indices.

```
backtrack(start, path):
    add a copy of `path` to the result    # every state is a valid subset
    for i in [start..n):
        path.add(nums[i])
        backtrack(i + 1, path)
        path.remove(last)                 # undo, try the next i
```

Each call adds the current `path` as a subset, then tries adding each remaining element.

#### Walking through `[1, 2, 3]`

```
backtrack(0, [])
  add []
  i = 0: path = [1]
    backtrack(1, [1])
      add [1]
      i = 1: path = [1, 2]
        backtrack(2, [1, 2])
          add [1, 2]
          i = 2: path = [1, 2, 3]
            backtrack(3, [1, 2, 3])
              add [1, 2, 3]
            (no more i)
          undo: path = [1, 2]
        (no more i)
      undo: path = [1]
      i = 2: path = [1, 3]
        backtrack(3, [1, 3])
          add [1, 3]
        undo: path = [1]
    undo: path = []
  i = 1: path = [2]
    ... (similar)
  i = 2: path = [3]
    ... (similar)
```

Final output: 8 subsets. ✓

### Method 2 — Bitmask (Iterative)

Each subset corresponds to a binary number from 0 to 2ⁿ - 1. Bit `i` in the number = "is `nums[i]` in this subset?"

```
For [1, 2, 3] (n = 3):
  mask 0 = 000 → []
  mask 1 = 001 → [1]
  mask 2 = 010 → [2]
  mask 3 = 011 → [1, 2]
  ...
  mask 7 = 111 → [1, 2, 3]
```

For each `mask` in `0..2ⁿ - 1`, build the subset by checking which bits are set. Concise and iterative — no recursion.

### Method 3 — Iterative Build-Up

Start with `[[]]`. For each element `x`:
- Take every subset already in the result.
- Make a new subset by appending `x` to it.
- Add the new subsets to the result.

```
Start: [[]]
After 1: [[], [1]]
After 2: [[], [1], [2], [1,2]]
After 3: [[], [1], [2], [1,2], [3], [1,3], [2,3], [1,2,3]]
```

Each step doubles the result.

---

## Time Complexity

**O(n × 2ⁿ)** — there are 2ⁿ subsets, and copying each one costs up to O(n).

## Space Complexity

**O(n × 2ⁿ)** for the output.
**O(n)** for the recursion stack and current path (excluding output).

---

## Code (Java)

```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> out = new ArrayList<>();
    backtrack(0, nums, new ArrayList<>(), out);
    return out;
}

private void backtrack(int start, int[] nums, List<Integer> path, List<List<Integer>> out) {
    out.add(new ArrayList<>(path));         // copy! (path mutates)

    for (int i = start; i < nums.length; i++) {
        path.add(nums[i]);                  // include nums[i]
        backtrack(i + 1, nums, path, out);  // recurse on the rest
        path.remove(path.size() - 1);       // undo (backtrack!)
    }
}

// Bitmask alternative
public List<List<Integer>> subsetsBitmask(int[] nums) {
    int n = nums.length;
    List<List<Integer>> out = new ArrayList<>(1 << n);

    for (int mask = 0; mask < (1 << n); mask++) {
        List<Integer> sub = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) {
                sub.add(nums[i]);
            }
        }
        out.add(sub);
    }
    return out;
}
```

> Why the copy `new ArrayList<>(path)`? Because `path` is the same object across the whole recursion — we mutate it. Without copying, every subset in `out` would point to the same (eventually empty) list.

---

## Code (Python)

```python
class Solution:
    # Backtracking
    def subsets(self, nums):
        out = []
        path = []

        def bt(start: int) -> None:
            out.append(path[:])         # copy (slice)
            for i in range(start, len(nums)):
                path.append(nums[i])
                bt(i + 1)
                path.pop()              # undo

        bt(0)
        return out

    # Iterative build-up
    def subsets_build(self, nums):
        out = [[]]
        for x in nums:
            out += [sub + [x] for sub in out]
        return out
```

---

## Edge Cases

| Case | Output |
|---|---|
| Empty input `[]` | `[[]]` (just the empty subset). |
| Single element `[5]` | `[[], [5]]`. |
| Result count | Exactly **2ⁿ** subsets. For n = 20, that's about a million; for n = 25, 33 million. Be careful with large n. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if the array has duplicates?"**
   See `subsets-no-duplicates.md`. You sort first and skip "duplicate at the same depth" branches.

2. **"Generate subsets with sum equal to k?"**
   Add a parameter for current sum. Add to result when sum == k. Prune when sum exceeds k (if all positive).

3. **"What's the relationship between backtracking and bitmask?"**
   They're equivalent — backtracking explicitly tries each bit's "in/out" decision; bitmask enumerates all combinations directly. Bitmask is faster in practice for small n; backtracking is more flexible (e.g., for pruning).

4. **"Lexicographic order?"**
   Sort the input first. The backtracking solution above naturally emits in lex order.

---

## Key Lessons from This Problem

1. **Backtracking** = "try a choice, recurse, then undo." It's the workhorse for combinatorial enumeration.
2. **Always copy when adding to the result** if your "path" variable is mutable.
3. **2ⁿ explodes fast** — be aware of input size limits in subset / power-set problems.
