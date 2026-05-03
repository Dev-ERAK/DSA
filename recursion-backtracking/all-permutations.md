# Problem: Generate All Permutations

## Category
Recursion / Backtracking

## Difficulty
Medium

---

## Problem Statement

Given an array of **distinct** integers `nums`, return all possible **permutations** (orderings).

For an n-element array, there are `n!` permutations.

This is **LeetCode #46**.

### Example

```
Input:  [1, 2, 3]
Output (3! = 6 permutations):
[1, 2, 3]
[1, 3, 2]
[2, 1, 3]
[2, 3, 1]
[3, 1, 2]
[3, 2, 1]
```

---

## Clarifying Questions to Ask the Interviewer

1. **All distinct?** Yes. Duplicates → `unique-permutations.md`.
2. **Return order?** Doesn't matter usually.

---

## Approach — Backtracking with a "used" Array

At each step, the partial permutation has some elements. Pick an **unused** element, append it, recurse, then undo.

```
backtrack(path, used[]):
    if path.length == n:
        add a copy of `path` to result
        return
    for i in 0..n-1:
        if used[i]: continue                # skip already-picked elements
        used[i] = true; path.append(nums[i])
        backtrack(path, used)
        used[i] = false; path.pop()         # undo
```

### Why we need `used[]`

Unlike subsets (where order doesn't matter), permutations need to know which positions are already filled. The `used[]` array is a constant-time check on whether an element is currently "in" the partial permutation.

### Walking through `[1, 2, 3]`

```
bt([], used=[F,F,F])
  i=0: pick 1. path=[1], used=[T,F,F]
    bt([1], [T,F,F])
      i=1: pick 2. path=[1,2]
        bt([1,2], [T,T,F])
          i=2: pick 3. path=[1,2,3]
            bt → length 3, add [1,2,3]
          undo: path=[1,2]
        undo: path=[1]
      i=2: pick 3. path=[1,3]
        bt → ... → add [1,3,2]
      undo: path=[1]
    undo: path=[], used=[F,F,F]
  i=1: pick 2. path=[2]
    ... → adds [2,1,3] and [2,3,1]
  i=2: pick 3. path=[3]
    ... → adds [3,1,2] and [3,2,1]
```

Total: 6 permutations. ✓

### Visualizing the recursion tree

```
                      []
        /          /          \
      [1]        [2]          [3]
     /  \       /  \          /  \
  [1,2] [1,3] [2,1] [2,3]  [3,1] [3,2]
   /     /     /     /       /     /
[1,2,3] [1,3,2] ...
```

n choices at the first level, n-1 at the second, ..., 1 at the last. Total leaves = n!.

---

## Time Complexity

**O(n × n!)** — there are `n!` permutations, and each takes O(n) to build (the copy).

## Space Complexity

**O(n)** for the recursion stack and current path. The output adds another O(n × n!) but is usually counted separately.

---

## Code (Java)

```java
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> out = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    backtrack(nums, used, new ArrayList<>(), out);
    return out;
}

private void backtrack(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> out) {
    if (path.size() == nums.length) {
        out.add(new ArrayList<>(path));
        return;
    }

    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;            // skip elements already in the permutation

        used[i] = true;
        path.add(nums[i]);

        backtrack(nums, used, path, out);

        // Undo:
        path.remove(path.size() - 1);
        used[i] = false;
    }
}
```

---

## Code (Python)

```python
class Solution:
    def permute(self, nums):
        out = []
        used = [False] * len(nums)
        path = []

        def bt() -> None:
            if len(path) == len(nums):
                out.append(path[:])         # copy
                return
            for i in range(len(nums)):
                if used[i]:
                    continue
                used[i] = True
                path.append(nums[i])
                bt()
                path.pop()                  # undo
                used[i] = False

        bt()
        return out
```

---

## Edge Cases

| Case | Output |
|---|---|
| `[]` (empty) | `[[]]` (just the empty permutation). |
| `[5]` | `[[5]]`. |
| Large n | n! grows extremely fast. n=10 → 3.6 million; n=12 → ~half a billion. Be careful. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if there are duplicates?"** See `unique-permutations.md`. Sort + skip rule based on `used[]`.

2. **"Generate the next permutation in lexicographic order (LeetCode 31)."**
   There's a clever **next-permutation** algorithm:
   1. Walk from the right; find the first descent (`a[i] < a[i+1]`).
   2. Swap `a[i]` with the smallest `a[j] > a[i]` from the right side.
   3. Reverse the suffix after `i`.

3. **"Find the k-th permutation directly without enumerating."**
   Use the factorial number system. Each "digit" of `k - 1` in factorial base picks the next element. O(n²).

4. **"In-place swap-based permutation."**
   Instead of `used[]`, swap elements into position:
   ```
   permute(start):
       if start == n: emit
       for i in start..n-1:
           swap(start, i); permute(start+1); swap(start, i)
   ```
   Saves the `used[]` array but produces a different output order.

---

## Key Lessons from This Problem

1. **Backtracking + `used[]`** is the classic permutation pattern.
2. **Always undo before the next iteration.** Forget the undo and you'll get garbage.
3. **n! grows fast** — be aware of input size limits.
