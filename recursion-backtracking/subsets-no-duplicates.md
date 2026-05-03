# Problem: Subsets When Input Has Duplicates

## Category
Recursion / Backtracking

## Difficulty
Medium

---

## Problem Statement

Given an array `nums` that **may contain duplicate values**, return all **unique subsets** (no two output subsets should have the same multiset of values).

This is **LeetCode #90**.

### Example

```
Input:  [1, 2, 2]
Output: [[],  [1],  [2],  [1, 2],  [2, 2],  [1, 2, 2]]
```

If we used the same algorithm as for distinct inputs, we'd get duplicates like `[1, 2(a)]` and `[1, 2(b)]` (using both 2's individually) — which look the same. We have to deduplicate.

---

## Clarifying Questions to Ask the Interviewer

1. **Inner order matters?** No — `[1, 2]` and `[2, 1]` are the same subset (multiset).
2. **Output order?** Doesn't matter; lexicographic is common.
3. **Are values guaranteed sorted?** Often not — sort them first.

---

## Approach

### The Big Idea — Sort, Then Skip Same-Value Choices at the Same Depth

Sort the input. Then, during backtracking, when we're choosing the **next** element to add at a given recursion depth, **skip duplicates** that we've already considered at this depth.

### The skip rule

Inside the for-loop at each `start`:

```
if i > start and nums[i] == nums[i-1]:
    continue          # already tried this value at this depth
```

The `i > start` check is the key — it ensures we still allow consecutive duplicates **deeper** in the recursion (i.e., we can include both 2's in the same subset), but we don't re-pick them as the "next element to add" at the same level.

### Walking through `[1, 2, 2]`

After sorting (already sorted): `[1, 2, 2]`.

```
bt(0, [])
  add []
  i = 0 (value 1): path = [1]
    bt(1, [1])
      add [1]
      i = 1 (value 2): path = [1, 2]
        bt(2, [1, 2])
          add [1, 2]
          i = 2 (value 2): path = [1, 2, 2]
            bt(3, [1, 2, 2])
              add [1, 2, 2]
            undo
          (no more i)
        undo
      i = 2 (value 2): SKIP (same as nums[1] at this depth)
    undo
  i = 1 (value 2): path = [2]
    bt(2, [2])
      add [2]
      i = 2 (value 2): path = [2, 2]
        bt(3, [2, 2])
          add [2, 2]
        undo
      (no more i)
    undo
  i = 2 (value 2): SKIP (i > start AND nums[2] == nums[1])
```

Final output:
```
[]
[1]
[1, 2]
[1, 2, 2]
[2]
[2, 2]
```

That's 6 unique subsets — exactly right.

### Why "i > start"?

Consider when `i = start`. That means we're picking the **first** element at this depth — there's nothing "previous" at the same depth to compare against. Skipping here would be wrong.

When `i > start`, we've already tried `nums[start]` (which equals `nums[i-1]` if they're duplicates). Skipping `nums[i]` avoids generating the same subset shape twice.

Equivalent picture: at each depth, you choose **one representative per duplicate value**. Sort + the skip rule guarantees this.

---

## Time Complexity

Worst case (no duplicates): **O(n × 2ⁿ)** — same as distinct subsets. With duplicates, faster because of pruning.

## Space Complexity

**O(n)** recursion + output.

---

## Code (Java)

```java
public List<List<Integer>> subsetsWithDup(int[] nums) {
    Arrays.sort(nums);                       // sort to put duplicates next to each other
    List<List<Integer>> out = new ArrayList<>();
    backtrack(0, nums, new ArrayList<>(), out);
    return out;
}

private void backtrack(int start, int[] nums, List<Integer> path, List<List<Integer>> out) {
    out.add(new ArrayList<>(path));

    for (int i = start; i < nums.length; i++) {
        // Skip duplicates: same value as the previous one we just processed at this depth.
        if (i > start && nums[i] == nums[i - 1]) continue;

        path.add(nums[i]);
        backtrack(i + 1, nums, path, out);
        path.remove(path.size() - 1);
    }
}
```

---

## Code (Python)

```python
class Solution:
    def subsetsWithDup(self, nums):
        nums.sort()
        out, path = [], []

        def bt(start: int) -> None:
            out.append(path[:])
            for i in range(start, len(nums)):
                if i > start and nums[i] == nums[i - 1]:
                    continue
                path.append(nums[i])
                bt(i + 1)
                path.pop()

        bt(0)
        return out
```

---

## Edge Cases

| Case | Result |
|---|---|
| All duplicates `[2, 2, 2]` | `[[], [2], [2,2], [2,2,2]]` — only 4 subsets. |
| Single element `[5]` | `[[], [5]]`. |
| Empty input | `[[]]`. |
| All distinct (no duplicates triggered) | Same as the "no duplicates" version — skip rule never fires. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if I forgot the sort?"**
   The skip rule `nums[i] == nums[i-1]` only catches **adjacent** duplicates. Without sorting, a `[2, 1, 2]` input would produce duplicate subsets. **Sorting is critical.**

2. **"How does this generalize to permutations with duplicates?"**
   See `unique-permutations.md`. Same idea: sort + skip-at-same-depth rule, but with a slightly different condition (involving a `used[]` array).

3. **"Combination sum II?"**
   Same skip rule applies — choose distinct values, but each input element can only be used once. (LeetCode 40.)

4. **"Without sorting, is there any way?"**
   Yes — store output subsets in a `Set<List<Integer>>` keyed on a canonical form (e.g., sorted tuple). Slower and less elegant than the structural skip.

---

## Key Lessons from This Problem

1. **Sort to bring duplicates adjacent**, then skip in the recursion.
2. **`i > start`** is the magic guard — letting duplicates extend a subset while preventing re-choice at the same depth.
3. **The same template** (sort + skip-at-same-depth) handles permutations and combinations with duplicates — just adapt the condition.
