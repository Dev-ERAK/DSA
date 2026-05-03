# Problem: Unique Permutations (with Duplicates in Input)

## Category
Recursion / Backtracking

## Difficulty
Medium

---

## Problem Statement

Given an array `nums` that **may contain duplicates**, return all **unique** permutations.

This is **LeetCode #47**.

### Example

```
Input:  [1, 1, 2]
Output: [[1, 1, 2], [1, 2, 1], [2, 1, 1]]
```

If we used the regular permutations algorithm, we'd produce 6 permutations (3!) but two pairs would be duplicates. We need to deduplicate.

---

## Clarifying Questions to Ask the Interviewer

1. **Duplicates are allowed in the input** but the output must have only unique permutations.
2. **Return order?** Lexicographic is common but not required.

---

## Approach

### The Big Idea — Sort + Skip Rule on `used[]`

Sort the input. Then in the backtracking, **skip a duplicate value if its earlier copy is currently NOT used**.

```
if i > 0 and nums[i] == nums[i - 1] and not used[i - 1]:
    continue
```

### Why "not used[i-1]" (the surprising part)

If `nums[i-1]` is **not used**, then we've already visited the case "pick `nums[i-1]` first" at this depth. Picking `nums[i]` now would create the **same** permutation we already created earlier with the order swapped — a duplicate.

If `nums[i-1]` **is used**, we're deeper down a path that includes it. Picking `nums[i]` now is a legitimate continuation, not a duplicate.

This is subtle. Let's make it concrete.

### Walking through `[1, 1, 2]`

After sort: `[1, 1, 2]`. Call elements `a = nums[0]`, `b = nums[1]`, `c = nums[2]` (so `a` and `b` are both `1`).

```
bt(used=[F,F,F])
  i=0 (val 1, "a"): pick. used=[T,F,F]. path=[1]
    bt
      i=0: skip (used)
      i=1 (val 1, "b"): "i>0 && nums[i]==nums[i-1] && !used[i-1]"
                        used[0]=TRUE so we DON'T skip. pick. used=[T,T,F]. path=[1,1]
        bt
          i=0: skip; i=1: skip
          i=2 (val 2): pick. path=[1,1,2]. emit [1,1,2]. undo
        undo
      i=2 (val 2): pick. path=[1,2]
        bt
          i=0: skip; i=1: pick (used[0]=T, allowed). path=[1,2,1]. emit [1,2,1]. undo
          i=2: skip
        undo
    undo
  i=1 (val 1, "b"): "i>0 && nums[1]==nums[0] && !used[0]"
                    used[0]=FALSE so we SKIP. (avoids duplicate of the path starting with "a")
  i=2 (val 2): pick. path=[2]
    bt
      i=0 (val 1): pick. path=[2,1]
        bt
          i=0: skip; i=1: pick. path=[2,1,1]. emit. undo
          i=2: skip
        undo
      i=1: SKIP (used[0]=F, dup)
      i=2: skip
    undo
```

Output: `[1,1,2], [1,2,1], [2,1,1]` — exactly 3 unique permutations. ✓

### Why this works (intuition)

Imagine the duplicates `1, 1` as `1ₐ, 1ᵦ`. Without the rule, we'd treat them as distinct, generating duplicate permutations. The rule says: **always pick `1ₐ` before `1ᵦ`** at any given depth. This canonicalizes the choice, eliminating the duplicates.

---

## Time Complexity

Worst case: **O(n × n!)** (when there are no duplicates). With duplicates, much faster due to pruning.

## Space Complexity

**O(n)** for recursion + `used[]` + path.

---

## Code (Java)

```java
public List<List<Integer>> permuteUnique(int[] nums) {
    Arrays.sort(nums);                      // crucial — sort to bring duplicates adjacent
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
        if (used[i]) continue;

        // Skip duplicate value at this depth: if the previous identical value
        // is NOT in the current path, picking this one would duplicate work.
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;

        used[i] = true;
        path.add(nums[i]);

        backtrack(nums, used, path, out);

        path.remove(path.size() - 1);
        used[i] = false;
    }
}
```

---

## Code (Python)

```python
class Solution:
    def permuteUnique(self, nums):
        nums.sort()
        out = []
        used = [False] * len(nums)
        path = []

        def bt() -> None:
            if len(path) == len(nums):
                out.append(path[:])
                return
            for i in range(len(nums)):
                if used[i]:
                    continue
                if i > 0 and nums[i] == nums[i - 1] and not used[i - 1]:
                    continue
                used[i] = True
                path.append(nums[i])
                bt()
                path.pop()
                used[i] = False

        bt()
        return out
```

---

## Edge Cases

| Case | Result |
|---|---|
| All identical (`[1, 1, 1]`) | One permutation: `[[1, 1, 1]]`. |
| Empty | `[[]]`. |
| All distinct | Same as the no-duplicate version (the skip rule never fires). |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why does sorting matter?"**
   The skip rule compares `nums[i]` to `nums[i-1]`. For duplicates to be detected, they have to be adjacent — which sorting guarantees.

2. **"Could we deduplicate by storing each permutation in a Set?"**
   Yes — convert each to a tuple and add to a `Set`. Works but uses extra memory and time. The structural skip is more elegant.

3. **"`used[i-1]` — why not `used[i-1]` (the opposite)?"**
   The two cases generate the same set of permutations differently. Either works as a canonicalization rule, but the conventional one is `!used[i-1]`. Be consistent and you'll be fine.

4. **"Compare to subsets-with-duplicates."**
   Same concept, different rule:
   - Subsets: `i > start && nums[i] == nums[i-1]` skip (no `used[]` because subsets don't reorder).
   - Permutations: `i > 0 && nums[i] == nums[i-1] && !used[i-1]` skip.

---

## Key Lessons from This Problem

1. **The skip rule canonicalizes the choice** of which duplicate to use first.
2. **Sort first** — almost every "with duplicates" backtracking problem starts here.
3. **`used[]` plays double duty** — track membership AND drive the deduplication rule.
